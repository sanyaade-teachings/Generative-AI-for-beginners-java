# MCP калькулятор: Посібник для початківців

## Зміст

- [Чого ви навчитесь](#чого-ви-навчитесь)
- [Вимоги](#вимоги)
- [Розуміння структури проєкту](#розуміння-структури-проєкту)
- [Пояснення основних компонентів](#пояснення-основних-компонентів)
  - [1. Головний додаток](#1-головний-додаток)
  - [2. Сервіс калькулятора](#2-сервіс-калькулятора)
  - [3. Прямий MCP клієнт](#3-прямий-mcp-клієнт)
  - [4. Клієнт на базі ШІ](#4-клієнт-на-базі-ші)
- [Запуск прикладів](#запуск-прикладів)
- [Як все це працює разом](#як-все-це-працює-разом)
- [Наступні кроки](#наступні-кроки)

## Чого ви навчитесь

У цьому посібнику пояснюється, як створити сервіс калькулятора з використанням Model Context Protocol (MCP). Ви дізнаєтесь:

- Як створити сервіс, який ШІ може використовувати як інструмент
- Як налаштувати прямий зв'язок із MCP сервісами
- Як ШІ-моделі можуть автоматично обирати інструменти для використання
- Різницю між прямими викликами протоколу та взаємодією за допомогою ШІ

## Вимоги

Перед початком переконайтеся, що у вас встановлено:

- Java 21 або новіша
- Maven для керування залежностями
- Розгортання моделі Azure AI Foundry (налаштуйте за допомогою `azd up` — див. [Розділ 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), увійшли в систему за допомогою `az login` (авторизація без ключа)
- Базове розуміння Java та Spring Boot

## Розуміння структури проєкту

Проєкт калькулятора містить кілька важливих файлів:

```
calculator/
├── src/main/java/com/microsoft/mcp/sample/server/
│   ├── McpServerApplication.java          # Main Spring Boot app
│   └── service/CalculatorService.java     # Calculator operations
└── src/test/java/com/microsoft/mcp/sample/client/
    ├── SDKClient.java                     # Direct MCP communication
    ├── LangChain4jClient.java            # AI-powered client
    └── Bot.java                          # Simple chat interface
```

## Пояснення основних компонентів

### 1. Головний додаток

**Файл:** `McpServerApplication.java`

Це точка входу в наш сервіс калькулятора. Це стандартний Spring Boot додаток з одним особливим доповненням:

```java
@SpringBootApplication
public class McpServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(McpServerApplication.class, args);
    }
    
    @Bean
    public ToolCallbackProvider calculatorTools(CalculatorService calculator) {
        return MethodToolCallbackProvider.builder().toolObjects(calculator).build();
    }
}
```

**Що це робить:**
- Запускає Spring Boot веб-сервер на порту 8080
- Створює `ToolCallbackProvider`, який робить методи калькулятора доступними як MCP інструменти
- Анотація `@Bean` вказує Spring керувати цим компонентом, щоб інші частини могли його використовувати

### 2. Сервіс калькулятора

**Файл:** `CalculatorService.java`

Тут відбуваються всі обчислення. Кожен метод позначено `@Tool`, щоб він був доступний через MCP:

```java
@Service
public class CalculatorService {

    @Tool(description = "Add two numbers together")
    public String add(double a, double b) {
        double result = a + b;
        return formatResult(a, "+", b, result);
    }

    @Tool(description = "Subtract the second number from the first number")
    public String subtract(double a, double b) {
        double result = a - b;
        return formatResult(a, "-", b, result);
    }
    
    // Більше операцій калькулятора...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Основні особливості:**

1. **Анотація `@Tool`**: повідомляє MCP, що цей метод можна викликати зовнішнім клієнтам
2. **Чіткі описи**: кожен інструмент має опис, що допомагає ШІ-моделям зрозуміти, коли його використовувати
3. **Єдиний формат повернення**: всі операції повертають зрозумілі рядки, наприклад "5.00 + 3.00 = 8.00"
4. **Обробка помилок**: ділення на нуль та негативні квадратні корені повертають повідомлення про помилку

**Доступні операції:**
- `add(a, b)` - додає два числа
- `subtract(a, b)` - віднімає друге з першого
- `multiply(a, b)` - множить два числа
- `divide(a, b)` - ділить перше на друге (з перевіркою на нуль)
- `power(base, exponent)` - підносить основу до степеня
- `squareRoot(number)` - обчислює квадратний корінь (з перевіркою на негативність)
- `modulus(a, b)` - повертає остачу від ділення
- `absolute(number)` - повертає абсолютне значення
- `help()` - повертає інформацію про всі операції

### 3. Прямий MCP клієнт

**Файл:** `SDKClient.java`

Цей клієнт безпосередньо спілкується з MCP сервером без ШІ. Він вручну викликає конкретні функції калькулятора:

```java
public class SDKClient {
    
    public static void main(String[] args) {
        McpClientTransport transport = WebFluxSseClientTransport.builder(
            WebClient.builder().baseUrl("http://localhost:8080")
        ).build();
        new SDKClient(transport).run();
    }
    
    public void run() {
        var client = McpClient.sync(this.transport).build();
        client.initialize();
        
        // Перелічити доступні інструменти
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Викликати конкретні функції калькулятора
        CallToolResult resultAdd = client.callTool(
            new CallToolRequest("add", Map.of("a", 5.0, "b", 3.0))
        );
        System.out.println("Add Result = " + resultAdd);
        
        CallToolResult resultSqrt = client.callTool(
            new CallToolRequest("squareRoot", Map.of("number", 16.0))
        );
        System.out.println("Square Root Result = " + resultSqrt);
        
        client.closeGracefully();
    }
}
```

**Що це робить:**
1. **Підключається** до калькулятора за адресою `http://localhost:8080` використовуючи builder pattern
2. **Відображає** всі доступні інструменти (наші функції калькулятора)
3. **Викликає** конкретні функції з точними параметрами
4. **Виводить** результати безпосередньо

**Примітка:** У цьому прикладі використовується залежність Spring AI 1.1.0-SNAPSHOT, яка ввела builder pattern для `WebFluxSseClientTransport`. Якщо ви використовуєте старішу стабільну версію, можливо, доведеться використовувати безпосередній конструктор.

**Коли використовувати:** Якщо ви точно знаєте який розрахунок потрібно виконати і хочете викликати його програмно.

### 4. Клієнт на базі ШІ

**Файл:** `LangChain4jClient.java`

Цей клієнт використовує ШІ модель (GPT-4o-mini), яка може автоматично обирати, які інструменти калькулятора застосовувати:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Налаштуйте модель ШІ (Azure AI Foundry, автентифікація без ключа через Microsoft Entra ID)
        String endpoint = System.getenv("AZURE_OPENAI_ENDPOINT");
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1";
        String token = new DefaultAzureCredentialBuilder().build()
                .getToken(new TokenRequestContext().addScopes("https://ai.azure.com/.default"))
                .block().getToken();
        ChatLanguageModel model = OpenAiOfficialChatModel.builder()
                .baseUrl(baseUrl)
                .apiKey(token)
                .modelName("gpt-4o-mini")
                .build();

        // Підключіться до нашого сервера калькулятора MCP
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Показує, що робить ШІ
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Надати ШІ доступ до наших інструментів калькулятора
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Створіть бота ШІ, який може користуватися нашим калькулятором
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Тепер ми можемо просити ШІ робити обчислення природною мовою
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Що це робить:**
1. **Створює** підключення до ШІ моделі з авторизацією без ключа (Microsoft Entra ID)
2. **Підключає** ШІ до нашого MCP калькулятора
3. **Надає** ШІ доступ до всіх наших інструментів калькулятора
4. **Дозволяє** природньомовні запити, наприклад "Обчисли суму 24.5 і 17.3"

**ШІ автоматично:**
- Розуміє, що ви хочете додати числа
- Обирає інструмент `add`
- Викликає `add(24.5, 17.3)`
- Повертає результат у природній формі

## Запуск прикладів

### Крок 1: Запустіть сервер калькулятора

Спершу увійдіть і встановіть кінцеву точку Azure AI Foundry (потрібно для клієнта ШІ — авторизація без ключа, без API ключа):

**Windows:**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**Linux/macOS:**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

Запустіть сервер:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

Сервер запускається за адресою `http://localhost:8080`. Ви побачите:
```
Started McpServerApplication in X.XXX seconds
```

### Крок 2: Тест із прямим клієнтом

В **НОВОМУ** терміналі, поки сервер працює, запустіть прямий MCP клієнт:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Ви побачите приблизно таке:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Крок 3: Тест із клієнтом на базі ШІ

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Ви побачите як ШІ автоматично використовує інструменти:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Крок 4: Закрийте MCP сервер

Після тестування можна зупинити клієнт ШІ, натиснувши `Ctrl+C` у відповідному терміналі. MCP сервер працюватиме, поки ви його не зупините.
Щоб зупинити сервер, натисніть `Ctrl+C` у терміналі, де він запущений.

## Як все це працює разом

Ось повний процес, коли ви питаєте ШІ: "Скільки буде 5 + 3?":

1. **Ви** ставите запит ШІ природною мовою
2. **ШІ** аналізує запит і розуміє, що вам потрібне додавання
3. **ШІ** викликає MCP сервер: `add(5.0, 3.0)`
4. **Сервіс калькулятора** виконує: `5.0 + 3.0 = 8.0`
5. **Сервіс калькулятора** повертає: `"5.00 + 3.00 = 8.00"`
6. **ШІ** отримує результат і формує природну відповідь
7. **Ви** отримуєте: "Сума 5 і 3 дорівнює 8"

## Наступні кроки

Для додаткових прикладів дивіться [Розділ 04: Практичні приклади](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Відмова від відповідальності**:
Цей документ було перекладено за допомогою сервісу штучного інтелекту для перекладу [Co-op Translator](https://github.com/Azure/co-op-translator). Хоча ми прагнемо до точності, будь ласка, майте на увазі, що автоматичні переклади можуть містити помилки або неточності. Оригінальний документ рідною мовою слід вважати авторитетним джерелом. Для критично важливої інформації рекомендується професійний людський переклад. Ми не несемо відповідальності за будь-які непорозуміння або неправильні тлумачення, що виникли внаслідок використання цього перекладу.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->