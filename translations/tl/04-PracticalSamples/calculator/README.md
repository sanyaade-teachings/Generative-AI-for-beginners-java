# MCP Calculator Tutorial para sa mga Baguhan

## Table of Contents

- [Ano ang Matututunan Mo](#ano-ang-matututunan-mo)
- [Mga Kinakailangan](#mga-kinakailangan)
- [Pag-unawa sa Istruktura ng Proyekto](#pag-unawa-sa-istruktura-ng-proyekto)
- [Paliwanag sa Mga Pangunahing Bahagi](#paliwanag-sa-mga-pangunahing-bahagi)
  - [1. Pangunahing Aplikasyon](#1-pangunahing-aplikasyon)
  - [2. Calculator Service](#2-calculator-service)
  - [3. Direktang MCP Client](#3-direktang-mcp-client)
  - [4. AI-Powered Client](#4-ai-powered-client)
- [Pagpapatakbo ng Mga Halimbawa](#pagpapatakbo-ng-mga-halimbawa)
- [Paano Ito Nagtratrabaho Nang Sama-sama](#paano-ito-nagtratrabaho-nang-sama-sama)
- [Mga Susunod na Hakbang](#mga-susunod-na-hakbang)

## Ano ang Matututunan Mo

Ipinaliwanag sa tutorial na ito kung paano bumuo ng calculator service gamit ang Model Context Protocol (MCP). Maiintindihan mo kung paano:

- Gumawa ng serbisyo na magagamit ng AI bilang isang tool
- Mag-setup ng diretsong komunikasyon sa mga MCP services
- Paano awtomatikong pumipili ng mga tool ang mga AI model
- Ang pagkakaiba ng direktang tawag ng protocol at pag-interak gamit ang AI

## Mga Kinakailangan

Bago magsimula, siguraduhing mayroon kang:
- Java 21 o mas mataas na naka-install
- Maven para sa pamamahala ng dependency
- Isang Azure AI Foundry model deployment (iprovide gamit ang `azd up` — tingnan ang [Kabanata 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- Ang [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), naka-sign in gamit ang `az login` (keyless auth)
- Pangunahing kaalaman sa Java at Spring Boot

## Pag-unawa sa Istruktura ng Proyekto

Ang proyekto ng calculator ay may ilang mahahalagang files:

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

## Paliwanag sa Mga Pangunahing Bahagi

### 1. Pangunahing Aplikasyon

**File:** `McpServerApplication.java`

Ito ang entry point ng aming calculator service. Isa itong standard Spring Boot application na may isang espesyal na dagdag:

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

**Ano ang ginagawa nito:**
- Pinapatakbo ang Spring Boot web server sa port 8080
- Lumilikha ng `ToolCallbackProvider` na ginagawang available ang mga method ng calculator bilang mga MCP tool
- Ang `@Bean` annotation ay nagsasabi sa Spring na pangasiwaan ito bilang isang component na magagamit ng iba pang bahagi

### 2. Calculator Service

**File:** `CalculatorService.java`

Dito nangyayari lahat ng matematika. Bawat method ay may markang `@Tool` para maging available sa MCP:

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

**Pangunahing katangian:**

1. **`@Tool` Annotation**: Sinusabi nito sa MCP na maaaring tawagin ang method na ito ng mga external client
2. **Malinaw na Deskripsyon**: Bawat tool ay may paliwanag na tumutulong sa AI models na maintindihan kung kailan ito gagamitin
3. **Pare-parehong Format ng Output**: Lahat ng operations ay nagbabalik ng human-readable na strings gaya ng "5.00 + 3.00 = 8.00"
4. **Pag-handle ng Error**: Ang paghahati sa zero at negatibong square roots ay nagbabalik ng mga mensahe ng error

**Mga Available na Operation:**
- `add(a, b)` - Nagdadagdag ng dalawang numero
- `subtract(a, b)` - Binabawas ang pangalawa sa unang numero
- `multiply(a, b)` - Nagmumultiply ng dalawang numero
- `divide(a, b)` - Hinahati ang una sa pangalawa (may check sa zero)
- `power(base, exponent)` - Inaangat ang base sa power ng exponent
- `squareRoot(number)` - Kinakalkula ang square root (may check sa negatibo)
- `modulus(a, b)` - Nagbabalik ng remainder ng paghati
- `absolute(number)` - Nagbabalik ng absolute value
- `help()` - Nagbabalik ng impormasyon tungkol sa lahat ng operasyon

### 3. Direktang MCP Client

**File:** `SDKClient.java`

Ang client na ito ay direktang nakikipag-usap sa MCP server nang hindi gumagamit ng AI. Pinapatawag nito nang mano-mano ang mga partikular na function ng calculator:

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
        
        // Tawagin ang mga partikular na function ng kalkuladora
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

**Ano ang ginagawa nito:**
1. **Kinokonekta** sa calculator server sa `http://localhost:8080` gamit ang builder pattern
2. **Naililista** ang lahat ng magagamit na tools (mga function ng calculator)
3. **Tinutawag** ang mga partikular na function na may eksaktong parameters
4. **Piniprint** ang resulta nang direkta

**Tandaan:** Ginagamit ng halimbawang ito ang Spring AI 1.1.0-SNAPSHOT dependency, na nagpakilala ng builder pattern para sa `WebFluxSseClientTransport`. Kung gumagamit ka ng mas lumang stable na bersyon, baka kailangan mong gamitin ang direktang constructor.

**Kailan ito gagamitin:** Kapag alam mo nang eksakto kung anong kalkulasyon ang gusto mong gawin at gusto mo itong tawagin programmatically.

### 4. AI-Powered Client

**File:** `LangChain4jClient.java`

Ang client na ito ay gumagamit ng AI model (GPT-4o-mini) na awtomatikong nagpapasya kung aling mga calculator tools ang gagamitin:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Isaayos ang AI model (Azure AI Foundry, keyless auth sa pamamagitan ng Microsoft Entra ID)
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

        // Bigyan ang AI ng akses sa aming mga calculator tool
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Gumawa ng AI bot na kayang gumamit ng aming calculator
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Ngayon maaari na nating hilingin sa AI na magsagawa ng mga kalkulasyon gamit ang natural na wika
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Ano ang ginagawa nito:**
1. **Lumilikha** ng koneksyon sa AI model gamit ang iyong GitHub token
2. **Kinokonekta** ang AI sa aming calculator MCP server
3. **Binibigyan** ang AI ng access sa lahat ng calculator tools namin
4. **Pinapayagan** ang natural na mga kahilingan sa wika gaya ng "Calculate the sum of 24.5 and 17.3"

**Awtomatikong ginagawa ng AI:**
- Naiintindihan na gusto mong magdagdag ng mga numero
- Pinipili ang `add` na tool
- Tinutawag ang `add(24.5, 17.3)`
- Nagbabalik ng resulta sa natural na tugon

## Pagpapatakbo ng Mga Halimbawa

### Hakbang 1: Simulan ang Calculator Server

Una, mag-sign in at itakda ang iyong Azure AI Foundry endpoint (kailangan para sa AI client — keyless auth, walang API key):

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

Ang server ay magsisimula sa `http://localhost:8080`. Makikita mo ang:
```
Started McpServerApplication in X.XXX seconds
```

### Hakbang 2: Subukan gamit ang Direktang Client

Sa isang **BAGONG** terminal habang patuloy na tumatakbo ang Server, patakbuhin ang direktang MCP client:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Makikita mo ang output na:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Hakbang 3: Subukan gamit ang AI Client

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Makikita mong awtomatikong ginagamit ng AI ang mga tool:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Hakbang 4: Isara ang MCP Server

Kapag natapos mo na ang pagsubok, maaari mong itigil ang AI client sa pamamagitan ng pagpindot ng `Ctrl+C` sa terminal nito. Patuloy na tatakbo ang MCP server hanggang itigil mo ito.
Para itigil ang server, pindutin ang `Ctrl+C` sa terminal kung saan ito tumatakbo.

## Paano Ito Nagtratrabaho Nang Sama-sama

Narito ang buong daloy kapag tinanong mo ang AI ng "Ano ang 5 + 3?":

1. **Ikaw** ang nagtatanong sa AI sa natural na wika
2. **AI** sinusuri ang iyong kahilingan at namamalayan na gusto mong magdagdag
3. **AI** tinatawag ang MCP server: `add(5.0, 3.0)`
4. **Calculator Service** ginagampanan ang: `5.0 + 3.0 = 8.0`
5. **Calculator Service** nagbabalik: `"5.00 + 3.00 = 8.00"`
6. **AI** natatanggap ang resulta at nagfo-format ng natural na tugon
7. **Ikaw** ay nakakatanggap ng: "Ang kabuuan ng 5 at 3 ay 8"

## Mga Susunod na Hakbang

Para sa karagdagang mga halimbawa, tingnan ang [Kabanata 04: Mga Praktikal na Halimbawa](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Pagtatanggi**:
Ang dokumentong ito ay isinalin gamit ang serbisyo ng AI translation na [Co-op Translator](https://github.com/Azure/co-op-translator). Bagama't nagsusumikap kami para sa katumpakan, pakatandaan na ang awtomatikong pagsasalin ay maaaring maglaman ng mga pagkakamali o hindi pagkakatugma. Ang orihinal na dokumento sa orihinal nitong wika ang dapat ituring na pangunahing sanggunian. Para sa mahahalagang impormasyon, inirerekomenda ang propesyonal na pagsasalin ng tao. Hindi kami mananagot sa anumang maling pagkakaintindi o maling interpretasyon na nagmula sa paggamit ng pagsasaling ito.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->