# Учебное пособие по калькулятору MCP для начинающих

## Содержание

- [Чему вы научитесь](#чему-вы-научитесь)
- [Требования](#требования)
- [Понимание структуры проекта](#понимание-структуры-проекта)
- [Объяснение основных компонентов](#объяснение-основных-компонентов)
  - [1. Основное приложение](#1-основное-приложение)
  - [2. Сервис калькулятора](#2-сервис-калькулятора)
  - [3. Прямой клиент MCP](#3-прямой-клиент-mcp)
  - [4. Клиент с ИИ](#4-клиент-с-ии)
- [Запуск примеров](#запуск-примеров)
- [Как все работает вместе](#как-все-работает-вместе)
- [Следующие шаги](#следующие-шаги)

## Чему вы научитесь

В этом руководстве объясняется, как создать сервис калькулятора с использованием протокола Model Context Protocol (MCP). Вы узнаете:

- Как создать сервис, который ИИ может использовать в качестве инструмента
- Как настроить прямую связь с сервисами MCP
- Как ИИ-модели могут автоматически выбирать, какие инструменты использовать
- Различия между прямыми вызовами протокола и взаимодействием с помощью ИИ

## Требования

Перед началом убедитесь, что у вас установлены:
- Java 21 или выше
- Maven для управления зависимостями
- Развернутая модель Azure AI Foundry (провизия с помощью `azd up` — см. [Глава 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), выполнен вход через `az login` (авторизация без ключа)
- Базовые знания Java и Spring Boot

## Понимание структуры проекта

Проект калькулятора содержит несколько важных файлов:

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

Это точка входа в наш сервис калькулятора. Стандартное Spring Boot приложение с одним особенным дополнением:

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

**Что оно делает:**
- Запускает веб-сервер Spring Boot на порту 8080
- Создаёт `ToolCallbackProvider`, который делает наши методы калькулятора доступными как инструменты MCP
- Аннотация `@Bean` сообщает Spring управлять этим компонентом для использования другими частями

### 2. Сервис калькулятора

**Файл:** `CalculatorService.java`

Здесь происходит вся математика. Каждый метод помечен `@Tool`, делая его доступным через MCP:

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

**Ключевые особенности:**

1. **Аннотация `@Tool`**: сообщает MCP, что этот метод могут вызывать внешние клиенты
2. **Чёткие описания**: каждый инструмент содержит описание, помогающее ИИ-моделям понять, когда его использовать
3. **Единый формат возвращаемого значения**: все операции возвращают удобочитаемые строки, например "5.00 + 3.00 = 8.00"
4. **Обработка ошибок**: деление на ноль и отрицательные квадратные корни возвращают сообщения об ошибке

**Доступные операции:**
- `add(a, b)` — складывает два числа
- `subtract(a, b)` — вычитает второе из первого
- `multiply(a, b)` — умножает два числа
- `divide(a, b)` — делит первое на второе (с проверкой на ноль)
- `power(base, exponent)` — возводит основание в степень
- `squareRoot(number)` — вычисляет квадратный корень (с проверкой на отрицательное число)
- `modulus(a, b)` — возвращает остаток от деления
- `absolute(number)` — возвращает абсолютное значение
- `help()` — возвращает информацию о всех операциях

### 3. Прямой клиент MCP

**Файл:** `SDKClient.java`

Этот клиент общается напрямую с сервером MCP без использования ИИ. Он вручную вызывает конкретные функции калькулятора:

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
        
        // Перечислить доступные инструменты
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Вызвать конкретные функции калькулятора
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
1. **Подключается** к серверу калькулятора по адресу `http://localhost:8080` с использованием паттерна строителя
2. **Выводит список** всех доступных инструментов (наших функций калькулятора)
3. **Вызывает** конкретные функции с точными параметрами
4. **Выводит** результаты напрямую

**Примечание:** Этот пример использует зависимость Spring AI 1.1.0-SNAPSHOT, которая ввела паттерн строителя для `WebFluxSseClientTransport`. Если вы используете более старую стабильную версию, возможно, потребуется использовать прямой конструктор.

**Когда использовать:** когда вы точно знаете, какое вычисление хотите выполнить и хотите вызвать его программно.

### 4. Клиент с ИИ

**Файл:** `LangChain4jClient.java`

Этот клиент использует модель ИИ (GPT-4o-mini), которая может автоматически выбирать, какие инструменты калькулятора использовать:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Настройка модели ИИ (Azure AI Foundry, аутентификация без ключа через Microsoft Entra ID)
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

        // Предоставить ИИ доступ к нашим калькуляторным инструментам
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Создать ИИ-бота, который может использовать наш калькулятор
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Теперь мы можем просить ИИ выполнять вычисления на естественном языке
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Что это делает:**
1. **Создаёт** подключение к ИИ-модели с вашим GitHub токеном
2. **Подключает** ИИ к нашему серверу MCP калькулятора
3. **Дает** ИИ доступ ко всем инструментам калькулятора
4. **Позволяет** использовать запросы на естественном языке, например "Вычисли сумму 24.5 и 17.3"

**ИИ автоматически:**
- Понимает, что вы хотите сложить числа
- Выбирает инструмент `add`
- Вызывает `add(24.5, 17.3)`
- Возвращает результат в естественном ответе

## Запуск примеров

### Шаг 1: Запуск сервера калькулятора

Сначала войдите в систему и задайте конечную точку Azure AI Foundry (нужно для клиента ИИ — авторизация без ключа, без API ключа):

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

### Шаг 2: Тестирование с помощью прямого клиента

В **НОВОМ** терминале при работающем сервере запустите прямой клиент MCP:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Вы увидите вывод, похожий на:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Шаг 3: Тестирование с помощью клиента ИИ

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Вы увидите, как ИИ автоматически использует инструменты:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Шаг 4: Остановка сервера MCP

Когда завершите тестирование, вы можете остановить клиент ИИ, нажав `Ctrl+C` в его терминале. MCP сервер будет работать, пока вы его не остановите.
Чтобы остановить сервер, нажмите `Ctrl+C` в терминале, где он запущен.

## Как все работает вместе

Вот полный процесс, когда вы спрашиваете ИИ «Что такое 5 + 3?»:

1. **Вы** задаёте вопрос ИИ на естественном языке
2. **ИИ** анализирует запрос и понимает, что вы хотите сложить
3. **ИИ** вызывает сервер MCP: `add(5.0, 3.0)`
4. **Сервис калькулятора** выполняет: `5.0 + 3.0 = 8.0`
5. **Сервис калькулятора** возвращает: `"5.00 + 3.00 = 8.00"`
6. **ИИ** получает результат и формирует естественный ответ
7. **Вы** получаете: "Сумма 5 и 3 равна 8"

## Следующие шаги

Для дополнительных примеров смотрите [Глава 04: Практические примеры](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Отказ от ответственности**:
Этот документ был переведен с использованием сервиса машинного перевода [Co-op Translator](https://github.com/Azure/co-op-translator). Несмотря на наши усилия по обеспечению точности, имейте в виду, что автоматический перевод может содержать ошибки или неточности. Оригинальный документ на его исходном языке следует считать авторитетным источником. Для получения критически важной информации рекомендуется обратиться к профессиональному человеческому переводу. Мы не несем ответственности за любые недоразумения или неправильные толкования, возникшие в результате использования этого перевода.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->