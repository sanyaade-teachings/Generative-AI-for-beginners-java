# MCP Ръководство за начинаещи

## Съдържание

- [Какво ще научите](#какво-ще-научите)
- [Изисквания](#изисквания)
- [Разбиране на структурата на проекта](#разбиране-на-структурата-на-проекта)
- [Обяснение на основните компоненти](#обяснение-на-основните-компоненти)
  - [1. Основно приложение](#1-основно-приложение)
  - [2. Календарната услуга](#2-календарната-услуга)
  - [3. Директен MCP клиент](#3-директен-mcp-клиент)
  - [4. Клиент с изкуствен интелект](#4-клиент-с-изкуствен-интелект)
- [Стартиране на примерите](#стартиране-на-примерите)
- [Как всичко работи заедно](#как-всичко-работи-заедно)
- [Следващи стъпки](#следващи-стъпки)

## Какво ще научите

В това ръководство се обяснява как да изградите калкулаторна услуга, използвайки Model Context Protocol (MCP). Ще разберете:

- Как да създадете услуга, която изкуственият интелект може да използва като инструмент
- Как да настроите директна комуникация с MCP услуги
- Как AI моделите могат автоматично да избират кои инструменти да използват
- Разликата между директни повиквания на протокола и взаимодействия с помощта на AI

## Изисквания

Преди да започнете, уверете се, че имате:
- Инсталиран Java 21 или по-нова версия
- Maven за управление на зависимости
- Разгръщане на Azure AI Foundry модел (осигурете го с `azd up` — вижте [Глава 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), влезли сте с `az login` (авторизация без ключ)
- Основни познания по Java и Spring Boot

## Разбиране на структурата на проекта

Проектът калкулатор съдържа няколко важни файла:

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

## Обяснение на основните компоненти

### 1. Основно приложение

**Файл:** `McpServerApplication.java`

Това е входната точка на нашата калкулаторна услуга. Това е стандартно Spring Boot приложение с една специална добавка:

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

**Какво прави това:**
- Стартира Spring Boot уеб сървър на порт 8080
- Създава `ToolCallbackProvider`, който прави нашите калкулаторни методи достъпни като MCP инструменти
- Маркерът `@Bean` казва на Spring да управлява това като компонент, който други части могат да използват

### 2. Календарната услуга

**Файл:** `CalculatorService.java`

Тук се извършват всички математически операции. Всеки метод е маркиран с `@Tool`, за да бъде достъпен чрез MCP:

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
    
    // Още изчислителни операции...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Основни характеристики:**

1. **Анотация `@Tool`**: Указва на MCP, че този метод може да бъде извикан от външни клиенти
2. **Ясни описания**: Всеки инструмент има описание, което помага на AI моделите да разберат кога да го използват
3. **Консистентен формат на връщаните стойности**: Всички операции връщат четими за хора резултати като "5.00 + 3.00 = 8.00"
4. **Обработка на грешки**: Деление на нула и отрицателен квадратен корен връщат съобщения за грешка

**Налични операции:**
- `add(a, b)` - Събира две числа
- `subtract(a, b)` - Изважда второто от първото
- `multiply(a, b)` - Умножава две числа
- `divide(a, b)` - Деление на първото число на второто (с проверка за нула)
- `power(base, exponent)` - Повдига основата на степента на показателя
- `squareRoot(number)` - Изчислява квадратен корен (с проверка за отрицателни числа)
- `modulus(a, b)` - Връща остатъка от деление
- `absolute(number)` - Връща абсолютната стойност
- `help()` - Връща информация за всички операции

### 3. Директен MCP клиент

**Файл:** `SDKClient.java`

Този клиент се свързва директно със сървъра MCP без използване на AI. Той ръчно извиква конкретни калкулаторни функции:

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
        
        // Изброяване на наличните инструменти
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Извикване на конкретни функции на калкулатора
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

**Какво прави това:**
1. **Свързва се** със сървъра на калкулатора на `http://localhost:8080` използвайки builder pattern
2. **Изброява** всички налични инструменти (нашите калкулаторни функции)
3. **Извиква** конкретни функции с точни параметри
4. **Отпечатва** резултатите директно

**Забележка:** Този пример използва Spring AI 1.1.0-SNAPSHOT зависимост, която въведе builder pattern за `WebFluxSseClientTransport`. Ако използвате по-стабилна по-стара версия, може да се наложи да използвате директния конструктор.

**Кога да използвате това:** Когато точно знаете какви изчисления искате да изпълните и искате да ги извикате програмно.

### 4. Клиент с изкуствен интелект

**Файл:** `LangChain4jClient.java`

Този клиент използва AI модел (GPT-4o-mini), който автоматично решава кои калкулаторни инструменти да използва:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Настройте AI модела (Azure AI Foundry, безключова автентикация чрез Microsoft Entra ID)
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

        // Свържете се с нашия сървър за калкулатор MCP
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Показва какво прави AI-то
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Дайте на AI достъп до нашите калкулаторни инструменти
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Създайте AI бот, който може да използва нашия калкулатор
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Сега можем да помолим AI-то да извършва изчисления на естествен език
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Какво прави това:**
1. **Създава** връзка с AI модел с авторизация без ключ (Microsoft Entra ID)
2. **Свързва** AI със сървъра на калкулатора MCP
3. **Дае** на AI достъп до всички наши калкулаторни инструменти
4. **Позволява** заявки на естествен език като "Изчисли сумата на 24.5 и 17.3"

**AI автоматично:**
- Разбира, че искате да съберете числа
- Избира инструмента `add`
- Извиква `add(24.5, 17.3)`
- Връща резултата с естествен отговор

## Стартиране на примерите

### Стъпка 1: Стартиране на сървъра на калкулатора

Първо влезте и задайте крайна точка на Azure AI Foundry (необходима за AI клиента — безключова авторизация, без API ключ):

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

Стартирайте сървъра:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

Сървърът ще стартира на `http://localhost:8080`. Трябва да видите:
```
Started McpServerApplication in X.XXX seconds
```

### Стъпка 2: Тествайте с директния клиент

В **НОВ** терминал, като сървърът все още работи, стартирайте директния MCP клиент:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Ще видите изход като:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Стъпка 3: Тествайте с AI клиента

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Ще видите AI автоматично да използва инструментите:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Стъпка 4: Затворете MCP сървъра

Когато приключите с тестовете, можете да спрете AI клиента с `Ctrl+C` в терминала му. MCP сървърът ще продължи да работи, докато не го спрете.
За да спрете сървъра, натиснете `Ctrl+C` в терминала, където е стартиран.

## Как всичко работи заедно

Ето пълния процес, когато попитате AI "Колко е 5 + 3?":

1. **Вие** задавате въпроса на AI на естествен език
2. **AI** анализира заявката и разбира, че искате събиране
3. **AI** извиква MCP сървъра: `add(5.0, 3.0)`
4. **Калкулаторната услуга** изчислява: `5.0 + 3.0 = 8.0`
5. **Калкулаторната услуга** връща: `"5.00 + 3.00 = 8.00"`
6. **AI** получава резултата и форматира естествен отговор
7. **Вие** получавате: "Сумата на 5 и 3 е 8"

## Следващи стъпки

За повече примери вижте [Глава 04: Практически примери](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Отказ от отговорност**:
Този документ е преведен с помощта на AI преводачески услуга [Co-op Translator](https://github.com/Azure/co-op-translator). Въпреки че се стремим към точност, моля имайте предвид, че автоматизираните преводи могат да съдържат грешки или неточности. Оригиналният документ на неговия роден език трябва да се счита за авторитетен източник. За критична информация се препоръчва професионален човешки превод. Ние не носим отговорност за каквито и да е недоразумения или неправилни тълкувания, произтичащи от използването на този превод.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->