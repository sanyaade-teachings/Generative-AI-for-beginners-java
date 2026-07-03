# MCP Туторијал за Израчунавање за Почетнике

## Садржај

- [Шта ћете научити](#шта-ћете-научити)
- [Претходни предуслови](#претходни-предуслови)
- [Разумевање Структуре Пројекта](#разумевање-структуре-пројекта)
- [Објашњење Кључних Компоненти](#објашњење-кључних-компоненти)
  - [1. Главна Апликација](#1-главна-апликација)
  - [2. Сервис за Израчунавање](#2-сервис-за-израчунавање)
  - [3. Директни MCP Клијент](#3-директни-mcp-клијент)
  - [4. Клијент са Вештачком Интелигенцијом](#4-клијент-са-вештачком-интелигенцијом)
- [Покретање Примера](#покретање-примера)
- [Како Све Ради Зajедно](#како-све-ради-зajедно)
- [Следећи Корaци](#следећи-корaци)

## Шта ћете научити

Овај туторијал објашњава како да направите сервис за израчунавање користећи Model Context Protocol (MCP). Разумете:

- Како да направите сервис који вештачка интелигенција може користити као алат
- Како да поставите директну комуникацију са MCP сервисима
- Како AI модели могу аутоматски изабрати које алате да користе
- Разлику између директних позива протокола и интеракције уз помоћ AI

## Претходни предуслови

Пре него што почнете, уверите се да имате:
- Инсталиран Java 21 или новији
- Maven за управљање зависностима
- Деплојмент модела у Azure AI Foundry (поставите га помоћу `azd up` — погледајте [Поглавље 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), пријављен корисник са `az login` (ауторизација без кључа)
- Основно разумевање Јаве и Spring Boot-а

## Разумевање Структуре Пројекта

Пројекат калкулатора садржи неколико важних фајлова:

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

## Објашњење Кључних Компоненти

### 1. Главна Апликација

**Фајл:** `McpServerApplication.java`

Ово је улазна тачка нашег сервиса за израчунавање. То је стандардна Spring Boot апликација са једним посебним додатком:

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

**Шта ово ради:**
- Покреће Spring Boot веб сервер на порту 8080
- Креира `ToolCallbackProvider` који наше калкулаторске методе чини доступним као MCP алате
- Анотација `@Bean` говори Spring-у да ова компонента буде управљана и доступна другим деловима

### 2. Сервис за Израчунавање

**Фајл:** `CalculatorService.java`

Овде се обавља свако математичко рачунање. Свака метода је означена са `@Tool` да би била доступна преко MCP-а:

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
    
    // Још операција калкулатора...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Кључне карактеристике:**

1. **`@Tool` Анотација**: Каже MCP-у да ова метода може бити позвана од стране спољних клијената
2. **Јасни Описи**: Сваки алат има опис који помаже AI моделима да разумеју када да га користе
3. **Уједначен Формат Резултата**: Све операције враћају људски читљиве стрингове као "5.00 + 3.00 = 8.00"
4. **Обрада Грешака**: Дељење са нулом и негативан корен квадратни враћају поруке о грешкама

**Доступне Операције:**
- `add(a, b)` - Сабира два броја
- `subtract(a, b)` - Одузима други од првог
- `multiply(a, b)` - Множи два броја
- `divide(a, b)` - Делить први број са другим (са провером на нулу)
- `power(base, exponent)` - Подиже базу на степен експонента
- `squareRoot(number)` - Израчунава квадратни корен (са провером на негативе)
- `modulus(a, b)` - Враћа остатак при дељењу
- `absolute(number)` - Враћа апсолутну вредност
- `help()` - Враћа информације о свим операцијама

### 3. Директни MCP Клијент

**Фајл:** `SDKClient.java`

Овај клијент директно комуницира са MCP сервером без употребе AI. Ручно позива специфичне калкулаторске функције:

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
        
        // Листа доступних алата
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Позови специфичне функције калкулатора
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

**Шта ово ради:**
1. Повезује се са сервером калкулатора на `http://localhost:8080` користећи builder pattern
2. Листa све расположиве алате (наше калкулаторске функције)
3. Позива специфичне функције са тачним параметрима
4. Исписује резултате директно

**Напомена:** Овај пример користи Spring AI 1.1.0-SNAPSHOT зависност, која је увела builder pattern за `WebFluxSseClientTransport`. Ако користите старију стабилну верзију, можда ћете морати користити директан конструктор.

**Када користити овако:** Када тачно знате коју рачунање желите да извршите и желите да га позовете програмски.

### 4. Клијент са Вештачком Интелигенцијом

**Фајл:** `LangChain4jClient.java`

Овај клијент користи AI модел (GPT-4o-mini) који може аутоматски одлучити које калкулаторске алате да користи:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Подесите АИ модел (Azure AI Foundry, аутентификација без кључа путем Microsoft Entra ID)
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

        // Повежите се са нашим MCP сервером калкулатора
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Приказује шта АИ ради
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Дајте АИ приступ нашим алаткама калкулатора
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Направите АИ бота који може користити наш калкулатор
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Сада можемо тражити од АИ да израчунава на природном језику
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Шта ово ради:**
1. Креира везу са AI моделом користећи аутентификацију без кључа (Microsoft Entra ID)
2. Повезује AI са нашим MCP сервером калкулатора
3. Дозвољава AI приступ свим нашим калкулаторским алатима
4. Омогућава природне захтеве као што је "Израчунај суму 24.5 и 17.3"

**AI аутоматски:**
- Разуме да желите да саберете бројеве
- Бира алат `add`
- Позива `add(24.5, 17.3)`
- Враћа резултат у природном одговору

## Покретање Примера

### Корак 1: Покрените Сервер Калкулатора

Прво се пријавите и подесите свој Azure AI Foundry endpoint (потребно за AI клијента — ауторизација без кључа, без API кључа):

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

Покрените сервер:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

Сервер ће почети на `http://localhost:8080`. Требало би да видите:
```
Started McpServerApplication in X.XXX seconds
```

### Корак 2: Тестирајте са Директним Клијентом

У **НОВОМ** терминалу, док сервер ради, покрените директни MCP клијент:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Видећете излаз попут:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Корак 3: Тестирајте са AI Клијентом

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Видећете да AI аутоматски користи алате:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Корак 4: Затворите MCP Сервер

Када завршите са тестирањем, можете зауставити AI клијента притиском на `Ctrl+C` у његовом терминалу. MCP сервер остаје покренут док га не зауставите.
Да бисте зауставили сервер, притисните `Ctrl+C` у терминалу у коме ради.

## Како Све Ради Заједно

Ево комплетног процеса када питате AI "Колико је 5 + 3?":

1. **Ви** питате AI у природном језику
2. **AI** анализира ваш захтев и схвата да желите сабирање
3. **AI** позива MCP сервер: `add(5.0, 3.0)`
4. **Сервис за Калкулатор** рачуна: `5.0 + 3.0 = 8.0`
5. **Сервис за Калкулатор** враћа: `"5.00 + 3.00 = 8.00"`
6. **AI** прими резултат и формира природни одговор
7. **Ви** добијате: "Сума од 5 и 3 је 8"

## Следећи Корaци

За више примера, погледајте [Поглавље 04: Практични Примери](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Изјава о одрицању одговорности**:
Овај документ је преведен коришћењем услуге за аутоматски превод [Co-op Translator](https://github.com/Azure/co-op-translator). Иако тежимо тачности, имајте у виду да аутоматски преводи могу садржати грешке или нетачности. Оригинални документ на његовом изворном језику треба сматрати ауторитативним извором. За критичне информације препоручује се професионални људски превод. Нисмо одговорни за било каква неспоразума или погрешна тумачења која произилазе из коришћења овог превода.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->