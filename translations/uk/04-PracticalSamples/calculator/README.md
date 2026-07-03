# Посібник з калькулятора MCP для початківців

## Зміст

- [Чому ви навчитеся](#чому-ви-навчитеся)
- [Вимоги](#вимоги)
- [Розуміння структури проєкту](#розуміння-структури-проєкту)
- [Пояснення основних компонентів](#пояснення-основних-компонентів)
  - [1. Головна програма](#1-головна-програма)
  - [2. Служба калькулятора](#2-служба-калькулятора)
  - [3. Прямий клієнт MCP](#3-прямий-клієнт-mcp)
  - [4. Клієнт на основі ШІ](#4-клієнт-на-основі-ші)
- [Запуск прикладів](#запуск-прикладів)
- [Як це все працює разом](#як-це-все-працює-разом)
- [Наступні кроки](#наступні-кроки)

## Чому ви навчитеся

У цьому посібнику пояснюється, як створити службу калькулятора за допомогою протоколу Model Context Protocol (MCP). Ви дізнаєтесь:

- Як створити сервіс, який ШІ може використовувати як інструмент
- Як налаштувати прямий зв’язок із службами MCP
- Як моделі ШІ можуть автоматично вибирати, які інструменти використовувати
- Різницю між прямими викликами протоколу і взаємодією за допомогою ШІ

## Вимоги

Перед початком переконайтеся, що у вас є:
- Встановлена Java 21 або новіша версія
- Maven для керування залежностями
- Розгортання моделі Azure AI Foundry (створіть його за допомогою `azd up` — див. [Розділ 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), авторизований через `az login` (аутентифікація без ключа)
- Базове розуміння Java та Spring Boot

## Розуміння структури проєкту

Проєкт калькулятора має кілька важливих файлів:

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

### 1. Головна програма

**Файл:** `McpServerApplication.java`

Це точка входу нашого сервісу калькулятора. Це стандартний додаток Spring Boot з одним особливим доповненням:

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
- Запускає вебсервер Spring Boot на порту 8080
- Створює `ToolCallbackProvider`, який робить наші методи калькулятора доступними як інструменти MCP
- Анотація `@Bean` каже Spring керувати цим як компонентом, який можуть використовувати інші частини

### 2. Служба калькулятора

**Файл:** `CalculatorService.java`

Тут відбуваються всі обчислення. Кожен метод позначений `@Tool`, щоб бути доступним через MCP:

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

**Ключові особливості:**

1. **Анотація `@Tool`**: інформує MCP, що цей метод може викликатися зовнішніми клієнтами
2. **Чіткі описи**: кожен інструмент має опис, що допомагає моделям ШІ зрозуміти, коли його використовувати
3. **Послідовний формат повернення**: усі операції повертають зрозумілі рядки, наприклад "5.00 + 3.00 = 8.00"
4. **Обробка помилок**: ділення на нуль і від’ємний корінь повертають повідомлення про помилку

**Доступні операції:**
- `add(a, b)` - додавання двох чисел
- `subtract(a, b)` - віднімання другого від першого
- `multiply(a, b)` - множення двох чисел
- `divide(a, b)` - ділення першого на друге (з перевіркою на нуль)
- `power(base, exponent)` - піднесення до степеня
- `squareRoot(number)` - обчислення квадратного кореня (з перевіркою на від’ємне)
- `modulus(a, b)` - остача від ділення
- `absolute(number)` - абсолютне значення
- `help()` - інформація про всі операції

### 3. Прямий клієнт MCP

**Файл:** `SDKClient.java`

Цей клієнт спілкується безпосередньо з сервером MCP без участі ШІ. Він вручну викликає конкретні функції калькулятора:

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
1. **Підключається** до сервера калькулятора за адресою `http://localhost:8080` з використанням патерну builder
2. **Перелічує** всі доступні інструменти (наші функції калькулятора)
3. **Викликає** конкретні функції з точними параметрами
4. **Виводить** результати напряму

**Примітка:** цей приклад використовує залежність Spring AI 1.1.0-SNAPSHOT, яка додала патерн builder для `WebFluxSseClientTransport`. Якщо ви використовуєте старішу версію, можливо, доведеться застосувати безпосередній конструктор.

**Коли використовувати:** коли ви точно знаєте, яке обчислення потрібно виконати, і хочете викликати його програмно.

### 4. Клієнт на основі ШІ

**Файл:** `LangChain4jClient.java`

Цей клієнт використовує модель ШІ (GPT-4o-mini), яка може автоматично вибирати, які інструменти калькулятора застосовувати:

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

        // Надайте ШІ доступ до наших інструментів калькулятора
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Створіть бота ШІ, який може користуватися нашим калькулятором
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Тепер ми можемо просити ШІ виконувати обчислення природною мовою
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Що це робить:**
1. **Створює** з’єднання зі ШІ за допомогою вашого GitHub токена
2. **Підключає** ШІ до нашого сервера MCP калькулятора
3. **Надає** ШІ доступ до всіх наших інструментів калькулятора
4. **Дозволяє** природні мовні запити, наприклад: "Обчисли суму 24.5 і 17.3"

**ШІ автоматично:**
- Розуміє, що ви хочете додати числа
- Обирає інструмент `add`
- Викликає `add(24.5, 17.3)`
- Повертає результат у природній відповіді

## Запуск прикладів

### Крок 1: Запустіть сервер калькулятора

Спочатку увійдіть і встановіть кінцеву точку Azure AI Foundry (потрібно для клієнта ШІ — аутентифікація без ключа, без API ключа):

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

Сервер запуститься на `http://localhost:8080`. Ви повинні побачити:
```
Started McpServerApplication in X.XXX seconds
```

### Крок 2: Перевірка з прямим клієнтом

У **НОВОМУ** терміналі, поки сервер працює, запустіть прямий клієнт MCP:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Ви побачите вивід приблизно такого виду:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Крок 3: Перевірка з клієнтом ШІ

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Ви побачите, як ШІ автоматично використовує інструменти:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Крок 4: Закриття сервера MCP

Коли закінчите тестування, ви можете зупинити клієнт ШІ, натиснувши `Ctrl+C` у його терміналі. Сервер MCP продовжуватиме працювати, поки ви його не зупините.
Щоб зупинити сервер, натисніть `Ctrl+C` у терміналі, де він запущений.

## Як це все працює разом

Ось повний процес, коли ви питаєте ШІ: "Скільки буде 5 + 3?":

1. **Ви** звертаєтесь до ШІ природною мовою
2. **ШІ** аналізує ваше запитання і розуміє, що потрібно додати
3. **ШІ** викликає сервер MCP: `add(5.0, 3.0)`
4. **Служба калькулятора** виконує: `5.0 + 3.0 = 8.0`
5. **Служба калькулятора** повертає: `"5.00 + 3.00 = 8.00"`
6. **ШІ** отримує результат і формує природну відповідь
7. **Ви** отримуєте: "Сума 5 і 3 дорівнює 8"

## Наступні кроки

Для отримання інших прикладів дивіться [Розділ 04: Практичні зразки](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Відмова від відповідальності**:
Цей документ було перекладено за допомогою сервісу штучного інтелекту для перекладу [Co-op Translator](https://github.com/Azure/co-op-translator). Хоча ми прагнемо до точності, будь ласка, майте на увазі, що автоматичні переклади можуть містити помилки або неточності. Оригінальний документ рідною мовою слід вважати авторитетним джерелом. Для критично важливої інформації рекомендується професійний людський переклад. Ми не несемо відповідальності за будь-які непорозуміння або неправильні тлумачення, що виникли внаслідок використання цього перекладу.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->