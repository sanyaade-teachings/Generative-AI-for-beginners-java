# MCP Calculator Tutorial para sa mga Nagsisimula

## Talaan ng Nilalaman

- [Ano ang Iyong Matututunan](#ano-ang-iyong-matututunan)
- [Mga Kinakailangan](#mga-kinakailangan)
- [Pag-unawa sa Istruktura ng Proyekto](#pag-unawa-sa-istruktura-ng-proyekto)
- [Paliwanag ng Pangunahing Mga Bahagi](#paliwanag-ng-pangunahing-mga-bahagi)
  - [1. Pangunahing Aplikasyon](#1-pangunahing-aplikasyon)
  - [2. Calculator Service](#2-calculator-service)
  - [3. Direktang MCP Client](#3-direktang-mcp-client)
  - [4. AI-Powered Client](#4-ai-powered-client)
- [Pagpapatakbo ng mga Halimbawa](#pagpapatakbo-ng-mga-halaman)
- [Paano Ito Gumagana nang Sama-sama](#paano-ito-gumagana-nang-sama-sama)
- [Mga Susunod na Hakbang](#mga-susunod-na-hakbang)

## Ano ang Iyong Matututunan

Ipinaliwanag ng tutorial na ito kung paano gumawa ng calculator service gamit ang Model Context Protocol (MCP). Maiintindihan mo:

- Paano gumawa ng service na maaaring gamitin ng AI bilang kasangkapan
- Paano mag-set up ng direktang komunikasyon sa mga MCP service
- Paano awtomatikong pipili ang mga AI model kung aling mga kasangkapan ang gagamitin
- Ang pagkakaiba ng direktang mga tawag sa protocol at mga interaksyon na may tulong ng AI

## Mga Kinakailangan

Bago magsimula, tiyakin na mayroon kang:
- Java 21 o mas mataas na naka-install
- Maven para sa pamamahala ng dependencies
- Isang Azure AI Foundry model deployment (i-provision gamit ang `azd up` — tingnan ang [Kabanata 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- Ang [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), naka-login gamit ang `az login` (keyless auth)
- Pangunahing kaalaman sa Java at Spring Boot

## Pag-unawa sa Istruktura ng Proyekto

May ilang mahahalagang file ang proyekto ng calculator:

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

## Paliwanag ng Pangunahing Mga Bahagi

### 1. Pangunahing Aplikasyon

**File:** `McpServerApplication.java`

Ito ang entry point ng ating calculator service. Isa itong karaniwang Spring Boot application na may isang espesyal na dagdag:

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

**Ginagawa nito:**
- Pinapatakbo ang isang Spring Boot web server sa port 8080
- Lumilikha ng `ToolCallbackProvider` na ginagawa ang mga method ng calculator nating magagamit bilang mga MCP tool
- Ang `@Bean` annotation ay nagsasabi sa Spring na pamahalaan ito bilang isang bahagi na magagamit ng ibang bahagi

### 2. Calculator Service

**File:** `CalculatorService.java`

Dito nangyayari lahat ng matematika. Bawat method ay may markang `@Tool` para magamit sa pamamagitan ng MCP:

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
    
    // Maraming mga operasyon ng calculator pa...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Mga Pangunahing Tampok:**

1. **`@Tool` Annotation**: Sinasabi nito sa MCP na maaaring tawagin ang method na ito ng panlabas na mga client
2. **Maliwanag na mga Deskripsyon**: Ang bawat tool ay may deskripsyon na tumutulong sa AI models na maintindihan kung kailan ito gagamitin
3. **Pare-parehong Format ng Pagbabalik**: Lahat ng operasyon ay nagbabalik ng mga string na madaling maintindihan, tulad ng "5.00 + 3.00 = 8.00"
4. **Pag-handle ng Error**: Ang paghahati sa zero at mga negatibong square root ay nagbabalik ng mga mensahe ng error

**Mga Magagamit na Operasyon:**
- `add(a, b)` - Nagdaragdag ng dalawang numero
- `subtract(a, b)` - Nagbabawas ng pangalawa mula sa una
- `multiply(a, b)` - Nagmumultiply ng dalawang numero
- `divide(a, b)` - Naghahati ng una sa pangalawa (may check kung zero)
- `power(base, exponent)` - Itinaas ang base sa exponent
- `squareRoot(number)` - Kinakalkula ang square root (may check kung negatibo)
- `modulus(a, b)` - Nagbabalik ng remainder ng paghahati
- `absolute(number)` - Nagbabalik ng absolute value
- `help()` - Nagbabalik ng impormasyon tungkol sa lahat ng operasyon

### 3. Direktang MCP Client

**File:** `SDKClient.java`

Ang client na ito ay nakikipag-usap nang direkta sa MCP server nang hindi gumagamit ng AI. Manu-mano nitong tinatawagan ang mga partikular na calculator function:

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
        
        // Ilista ang mga magagamit na kasangkapan
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Tawagan ang mga tiyak na function ng calculator
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

**Ginagawa nito:**
1. **Kumokonekta** sa calculator server sa `http://localhost:8080` gamit ang builder pattern
2. **Ipinapakita** lahat ng magagamit na tools (mga function ng calculator natin)
3. **Tinutawag** ang mga partikular na function na may eksaktong mga parameter
4. **Ipiniprinta** ang mga resulta nang direkta

**Tandaan:** Ang halimbawang ito ay gumagamit ng Spring AI 1.1.0-SNAPSHOT dependency, na nagpakilala ng builder pattern para sa `WebFluxSseClientTransport`. Kung gumagamit ka ng mas luma at stable na bersyon, maaaring kailanganin mong gamitin ang direktang constructor.

**Kailan gagamitin ito:** Kapag alam mo na eksakto kung ano ang kalkulasyon na gusto mong gawin at gusto mo itong tawagin nang programmatically.

### 4. AI-Powered Client

**File:** `LangChain4jClient.java`

Ang client na ito ay gumagamit ng AI model (GPT-4o-mini) na kayang awtomatikong pumili kung anong mga calculator tool ang gagamitin:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // I-set up ang AI model (Azure AI Foundry, keyless auth sa pamamagitan ng Microsoft Entra ID)
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

        // Kumonekta sa aming calculator MCP server
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Ipinapakita kung ano ang ginagawa ng AI
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Bigyan ang AI ng access sa aming mga calculator tools
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Gumawa ng AI bot na kayang gumamit ng aming calculator
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Ngayon maaari na nating hilingin sa AI na gumawa ng mga kalkulasyon sa natural na wika
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Ginagawa nito:**
1. **Lumilikha** ng AI model connection gamit ang keyless authentication (Microsoft Entra ID)
2. **Kinokonekta** ang AI sa ating calculator MCP server
3. **Pinapayagan** ang AI na ma-access ang lahat ng calculator tools natin
4. **Pinapayagan** ang mga natural na kahilingan sa wika tulad ng "Calculate the sum of 24.5 and 17.3"

**Awtomatikong ginagawa ng AI:**
- Naiintindihan nitong gusto mong magdagdag ng mga numero
- Pinipili ang `add` tool
- Tinutawag ang `add(24.5, 17.3)`
- Ibinabalik ang resulta sa natural na tugon

## Pagpapatakbo ng mga Halimbawa

### Hakbang 1: Simulan ang Calculator Server

Una, mag-login at itakda ang iyong Azure AI Foundry endpoint (kailangan para sa AI client — keyless auth, walang API key):

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

Simulan ang server:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

Magsisimula ang server sa `http://localhost:8080`. Makikita mo:
```
Started McpServerApplication in X.XXX seconds
```

### Hakbang 2: Subukan gamit ang Direktang Client

Sa isang **BAGONG** terminal habang tumatakbo pa ang Server, patakbuhin ang direktang MCP client:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Makikita mo ang output tulad ng:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Hakbang 3: Subukan gamit ang AI Client

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Makikita mong awtomatikong gumagamit ang AI ng mga tools:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Hakbang 4: Isara ang MCP Server

Kapag tapos ka na sa pagsubok, maaari mong itigil ang AI client sa pamamagitan ng pagpindot ng `Ctrl+C` sa terminal nito. Mananatiling tumatakbo ang MCP server hanggang mapatay mo ito.
Para itigil ang server, pindutin ang `Ctrl+C` sa terminal kung saan ito tumatakbo.

## Paano Ito Gumagana nang Sama-sama

Narito ang kumpletong daloy kapag tinanong mo ang AI ng "Ano ang 5 + 3?":

1. **Ikaw** ang nagtatanong sa AI gamit ang natural na wika
2. **AI** ang nagsusuri ng iyong kahilingan at napagtanto mo gusto mo ng addition
3. **AI** ang tumatawag sa MCP server: `add(5.0, 3.0)`
4. **Calculator Service** ang nagsasagawa: `5.0 + 3.0 = 8.0`
5. **Calculator Service** ay nagbabalik ng: `"5.00 + 3.00 = 8.00"`
6. **AI** ay tumatanggap ng resulta at bumubuo ng natural na tugon
7. **Ikaw** ay makakuha ng: "Ang suma ng 5 at 3 ay 8"

## Mga Susunod na Hakbang

Para sa higit pang mga halimbawa, tingnan ang [Kabanata 04: Praktikal na mga halimbawa](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Pagtatanggi**:
Ang dokumentong ito ay isinalin gamit ang serbisyo ng AI translation na [Co-op Translator](https://github.com/Azure/co-op-translator). Bagama't nagsusumikap kami para sa katumpakan, pakatandaan na ang awtomatikong pagsasalin ay maaaring maglaman ng mga pagkakamali o hindi pagkakatugma. Ang orihinal na dokumento sa orihinal nitong wika ang dapat ituring na pangunahing sanggunian. Para sa mahahalagang impormasyon, inirerekomenda ang propesyonal na pagsasalin ng tao. Hindi kami mananagot sa anumang maling pagkakaintindi o maling interpretasyon na nagmula sa paggamit ng pagsasaling ito.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->