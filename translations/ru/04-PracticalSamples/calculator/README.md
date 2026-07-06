# Руководство по калькулятору MCP для начинающих

## Содержание

- [Чему вы научитесь](#чему-вы-научитесь)
- [Требования](#требования)
- [Понимание структуры проекта](#понимание-структуры-проекта)
- [Объяснение основных компонентов](#объяснение-основных-компонентов)
  - [1. Основное приложение](#1-основное-приложение)
  - [2. Сервис калькулятора](#2-сервис-калькулятора)
  - [3. Прямой клиент MCP](#3-прямой-клиент-mcp)
  - [4. Клиент с поддержкой ИИ](#4-клиент-с-поддержкой-и)
- [Запуск примеров](#запуск-примеров)
- [Как всё работает вместе](#как-всё-работает-вместе)
- [Следующие шаги](#следующие-шаги)

## Чему вы научитесь

В этом руководстве объясняется, как создать сервис калькулятора с использованием протокола Model Context Protocol (MCP). Вы узнаете:

- Как создать сервис, который ИИ может использовать в качестве инструмента
- Как настроить прямую коммуникацию с сервисами MCP
- Как ИИ-модели могут автоматически выбирать необходимые инструменты
- В чём разница между прямыми вызовами протокола и взаимодействием с помощью ИИ

## Требования

Перед началом убедитесь, что у вас есть:
- Установленная Java 21 или выше
- Maven для управления зависимостями
- Развёртывание модели Azure AI Foundry (создайте с помощью `azd up` — см. [Главу 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), выполненный вход с помощью `az login` (аутентификация без ключа)
- Базовое понимание Java и Spring Boot

## Понимание структуры проекта

В проекте калькулятора есть несколько важных файлов:

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

## Объяснение основных компонентов

### 1. Основное приложение

**Файл:** `McpServerApplication.java`

Это точка входа нашего сервиса калькулятора. Это стандартное Spring Boot приложение с одним особым дополнением:

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

**Что это делает:**
- Запускает веб-сервер Spring Boot на порту 8080
- Создаёт `ToolCallbackProvider`, который делает методы нашего калькулятора доступными как инструменты MCP
- Аннотация `@Bean` сообщает Spring, что это компонент, который могут использовать другие части

### 2. Сервис калькулятора

**Файл:** `CalculatorService.java`

Здесь выполняются все вычисления. Каждый метод помечен `@Tool`, чтобы сделать его доступным через MCP:

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
    
    // Больше операций калькулятора...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Основные особенности:**

1. **Аннотация `@Tool`**: Сообщает MCP, что этот метод может быть вызван внешними клиентами
2. **Чёткие описания**: Каждый инструмент имеет описание, которое помогает ИИ моделям понять, когда его использовать
3. **Единый формат возврата**: Все операции возвращают человекочитаемые строки, например "5.00 + 3.00 = 8.00"
4. **Обработка ошибок**: Деление на ноль и отрицательный квадратный корень возвращают сообщения об ошибках

**Доступные операции:**
- `add(a, b)` - Сложение двух чисел
- `subtract(a, b)` - Вычитание второго из первого
- `multiply(a, b)` - Умножение двух чисел
- `divide(a, b)` - Деление первого на второе (с проверкой на ноль)
- `power(base, exponent)` - Возведение основания в степень
- `squareRoot(number)` - Вычисление квадратного корня (с проверкой на отрицательное число)
- `modulus(a, b)` - Остаток от деления
- `absolute(number)` - Модуль числа
- `help()` - Возвращает информацию обо всех операциях

### 3. Прямой клиент MCP

**Файл:** `SDKClient.java`

Этот клиент напрямую обращается к серверу MCP без использования ИИ. Он вручную вызывает конкретные функции калькулятора:

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
        
        // Список доступных инструментов
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Вызов конкретных функций калькулятора
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

**Что это делает:**
1. **Подключается** к серверу калькулятора по адресу `http://localhost:8080` с использованием шаблона builder
2. **Выводит список** всех доступных инструментов (функций калькулятора)
3. **Вызывает** конкретные функции с точными параметрами
4. **Выводит** результаты напрямую

**Примечание:** Этот пример использует зависимость Spring AI версии 1.1.0-SNAPSHOT, которая ввела шаблон builder для `WebFluxSseClientTransport`. Если вы используете более старую стабильную версию, возможно, придётся использовать конструктор напрямую.

**Когда использовать:** Когда вы точно знаете, какое вычисление хотите выполнить, и хотите вызывать его программно.

### 4. Клиент с поддержкой ИИ

**Файл:** `LangChain4jClient.java`

Этот клиент использует ИИ-модель (GPT-4o-mini), которая может автоматически выбирать инструменты калькулятора:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Настройка модели ИИ (Azure AI Foundry, авторизация без ключа через Microsoft Entra ID)
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

        // Подключение к нашему серверу калькулятора MCP
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Показывает, что делает ИИ
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Предоставить ИИ доступ к нашим инструментам калькулятора
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Создать ИИ-бота, который может использовать наш калькулятор
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Теперь мы можем попросить ИИ выполнять вычисления на естественном языке
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Что это делает:**
1. **Создаёт** подключение к ИИ модели с аутентификацией без ключа (Microsoft Entra ID)
2. **Подключает** ИИ к нашему серверу MCP калькулятора
3. **Дает** ИИ доступ ко всем инструментам калькулятора
4. **Позволяет** использовать запросы на естественном языке, например "Вычисли сумму 24.5 и 17.3"

**ИИ автоматически:**
- Понимает, что нужно сложить числа
- Выбирает инструмент `add`
- Вызывает `add(24.5, 17.3)`
- Возвращает результат в естественной форме ответа

## Запуск примеров

### Шаг 1: Запустите сервер калькулятора

Сначала выполните вход и установите конечную точку Azure AI Foundry (нужно для клиента ИИ — аутентификация без ключа, API ключ не нужен):

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

Запустите сервер:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

Сервер запустится на `http://localhost:8080`. Вы должны увидеть:
```
Started McpServerApplication in X.XXX seconds
```

### Шаг 2: Тест с прямым клиентом

В **НОВОМ** терминале, при работающем сервере, запустите прямой MCP клиент:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Вы увидите вывод вроде:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Шаг 3: Тест с клиентом ИИ

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Вы увидите, как ИИ автоматически использует инструменты:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Шаг 4: Закройте сервер MCP

Когда закончите тестирование, можно остановить клиент ИИ, нажав `Ctrl+C` в его терминале. Сервер MCP будет продолжать работу, пока вы его не остановите.
Чтобы остановить сервер, нажмите `Ctrl+C` в терминале, где он запущен.

## Как всё работает вместе

Полный процесс, когда вы спрашиваете ИИ: "Сколько будет 5 + 3?":

1. **Вы** задаёте вопрос ИИ на естественном языке
2. **ИИ** анализирует ваш запрос и понимает, что вы хотите сложить
3. **ИИ** вызывает сервер MCP: `add(5.0, 3.0)`
4. **Сервис калькулятора** выполняет: `5.0 + 3.0 = 8.0`
5. **Сервис калькулятора** возвращает: `"5.00 + 3.00 = 8.00"`
6. **ИИ** получает результат и формирует естественный ответ
7. **Вы** получаете: "Сумма 5 и 3 равна 8"

## Следующие шаги

Для дополнительных примеров смотрите [Главу 04: Практические примеры](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Отказ от ответственности**:
Этот документ был переведен с использованием сервиса машинного перевода [Co-op Translator](https://github.com/Azure/co-op-translator). Несмотря на наши усилия по обеспечению точности, имейте в виду, что автоматический перевод может содержать ошибки или неточности. Оригинальный документ на его исходном языке следует считать авторитетным источником. Для получения критически важной информации рекомендуется обратиться к профессиональному человеческому переводу. Мы не несем ответственности за любые недоразумения или неправильные толкования, возникшие в результате использования этого перевода.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->