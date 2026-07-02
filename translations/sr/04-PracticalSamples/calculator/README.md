# MCP Пррачунски Туторијал за Почетнике

## Садржај

- [Шта ћете научити](#шта-ћете-научити)
- [Предуслови](#предуслови)
- [Разумевање структуре пројекта](#разумевање-структуре-пројекта)
- [Објашњење основних компоненти](#објашњење-основних-компоненти)
  - [1. Главна апликација](#1-главна-апликација)
  - [2. Сервис калкулатора](#2-сервис-калкулатора)
  - [3. Директни MCP Клијент](#3-директни-mcp-клијент)
  - [4. Клијент покретан вештачком интелигенцијом](#4-клијент-покретан-вештачком-интелигенцијом)
- [Покретање примера](#покретање-примера)
- [Како све ради заједно](#како-све-ради-заједно)
- [Следећи кораци](#следећи-кораци)

## Шта ћете научити

Овај туторијал објашњава како да направите сервис калкулатора користећи Model Context Protocol (MCP). Разумете:

- Како направити сервис који вештачка интелигенција може да користи као алат
- Како поставити директну комуникацију са MCP сервисима
- Како модели вештачке интелигенције могу аутоматски изабрати које алате користити
- Разлику између директних позива протокола и интеракција уз помоћ вештачке интелигенције

## Предуслови

Пре почетка, уверите се да имате:
- Инсталиран Java 21 или новији
- Maven за управљање зависностима
- Deploy Azure AI Foundry модела (поставите га са `azd up` — видети [Поглавље 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), пријављен са `az login` (аутентификација без кључа)
- Основно разумевање Јаве и Spring Boot-а

## Разумевање структуре пројекта

Пројекат калкулатора има неколико важних фајлова:

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

## Објашњење основних компоненти

### 1. Главна апликација

**Фајл:** `McpServerApplication.java`

Ово је улазна тачка нашег сервиса калкулатора. То је стандардна Spring Boot апликација са једним посебним додатком:

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
- Креира `ToolCallbackProvider` који наше методе калкулатора чини доступним као MCP алате
- Аннотација `@Bean` каже Spring-у да управља овом компоненетом како би је други делови могли користити

### 2. Сервис калкулатора

**Фајл:** `CalculatorService.java`

Овде се одвија сва математика. Сваки метод је означен са `@Tool` да би био доступан преко MCP:

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
    
    // Више операција калкулатора...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Кључне одлике:**

1. **Аннотација `@Tool`**: Ово говори MCP-у да овај метод може бити позван од стране спољашњих клијената
2. **Јасни описи**: Сваки алат има опис који помаже AI моделима да разумеју када га користити
3. **Доследан формат повратка**: Све операције враћају људски читљиве стрингове као што је "5.00 + 3.00 = 8.00"
4. **Обрада грешака**: Дељење са нулом и корени негативних бројева враћају поруке о грешци

**Доступне операције:**
- `add(a, b)` - Сабира два броја
- `subtract(a, b)` - Одузима други од првог
- `multiply(a, b)` - Множи два броја
- `divide(a, b)` - Дели први са другим (са провером на нулу)
- `power(base, exponent)` - Подиже базу на степен експонента
- `squareRoot(number)` - Израчунава корен (са провером на негативно)
- `modulus(a, b)` - Враћа остатак дељења
- `absolute(number)` - Враћа апсолутну вредност
- `help()` - Враћа информације о свим операцијама

### 3. Директни MCP Клијент

**Фајл:** `SDKClient.java`

Овај клијент директно комуницира са MCP сервером без коришћења AI. Он ручно позива одређене функције калкулатора:

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
        
        // Наведи доступне алате
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Позови одређене функције калкулатора
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
1. **Повезује се** на сервер калкулатора на `http://localhost:8080` користећи builder pattern
2. **Листa** све доступне алате (наше калкулаторске функције)
3. **Позива** одређене функције са тачним параметрима
4. **Исписује** резултате директно

**Напомена:** Овај пример користи Spring AI 1.1.0-SNAPSHOT зависност, која је увела builder pattern за `WebFluxSseClientTransport`. Ако користите старију стабилну верзију, можда ћете морати да користите директан конструктор.

**Када користити ово:** Када тачно знате који израчун желите да извршите и желите да га позовете програмски.

### 4. Клијент покретан вештачком интелигенцијом

**Фајл:** `LangChain4jClient.java`

Овај клијент користи AI модел (GPT-4o-mini) који аутоматски одлучује које калкулаторске алате да користи:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Подесите АИ модел (Azure AI Foundry, аутентификација без кључа преко Microsoft Entra ID)
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

        // Повежите се са нашим сервером калкулатора MCP
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Приказује шта АИ ради
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Дозволите АИ приступ нашим алатима за калкулатор
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Креирајте АИ бота који може користити наш калкулатор
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Сада можемо тражити од АИ да обави прорачуне у природном језику
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Шта ово ради:**
1. **Креира** везу са AI моделом користећи ваш GitHub token
2. **Повезује** AI са нашим MCP сервером калкулатора
3. **Дозвољава** AI приступ свим нашим калкулаторским алатима
4. **Омогућава** природне захтеве као што су "Израчунај зброј 24.5 и 17.3"

**AI аутоматски:**
- Разумева да желите да саберете бројеве
- Изабире `add` алат
- Позива `add(24.5, 17.3)`
- Враћа резултат у природном одговору

## Покретање примера

### Корак 1: Покрените калкулатор сервер

Прво, пријавите се и поставите ваш Azure AI Foundry endpoint (потребно за AI клијент — аутентификација без кључа, без API кључа):

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

Сервер ће се покренути на `http://localhost:8080`. Требало би да видите:
```
Started McpServerApplication in X.XXX seconds
```

### Корак 2: Тестирајте са директним клијентом

У **НОВОМ** терминалу, док сервер још ради, покрените директни MCP клијент:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Видећете излаз као:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Корак 3: Тестирајте са AI клијентом

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Видећете да AI аутоматски користи алате:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Корак 4: Затворите MCP сервер

Када завршите тестирање, можете зауставити AI клијент притиском `Ctrl+C` у његовом терминалу. MCP сервер ће наставити да ради док га не зауставите.
За заустављање сервера, притисните `Ctrl+C` у терминалу где се покреће.

## Како све ради заједно

Ево комплетног тока када питате AI „Колико је 5 + 3?“:

1. **Ви** питате AI природним језиком
2. **AI** анализира ваш захтев и схвата да желите сабирање
3. **AI** позива MCP сервер: `add(5.0, 3.0)`
4. **Сервис калкулатора** извршава: `5.0 + 3.0 = 8.0`
5. **Сервис калкулатора** враћа: `"5.00 + 3.00 = 8.00"`
6. **AI** прими резултат и форматира природан одговор
7. **Ви добијате:** "Зброј бројева 5 и 3 је 8"

## Следећи кораци

За више примера, видети [Поглавље 04: Практични примери](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Изјава о одрицању одговорности**:
Овај документ је преведен коришћењем услуге за аутоматски превод [Co-op Translator](https://github.com/Azure/co-op-translator). Иако тежимо тачности, имајте у виду да аутоматски преводи могу садржати грешке или нетачности. Оригинални документ на његовом изворном језику треба сматрати ауторитативним извором. За критичне информације препоручује се професионални људски превод. Нисмо одговорни за било каква неспоразума или погрешна тумачења која произилазе из коришћења овог превода.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->