# MCP క్యాలిక్యులేటర్ ట్యుటోరియల్ ప్రారంభకర్తలకు

## పట్టిక

- [మీరు ఏమి నేర్చుకుంటారు](#మీరు-ఏమి-నేర్చుకుంటారు)
- [అవసరాలు](#అవసరాలు)
- [ప్రాజెక్ట్ గঠনాన్ని అర్థం చేసుకోవడం](#ప్రాజెక్ట్-గঠনాన్ని-అర్థం-చేసుకోవడం)
- [ప్రధాన భాగాల వివరణ](#ప్రధాన-భాగాల-వివరణ)
  - [1. ప్రధాన అనువర్తనం](#1-ప్రధాన-అనువర్తనం)
  - [2. క్యాలక్యులేటర్ సేవ](#2-క్యాలిక్యులేటర్-సేవ)
  - [3. డైరెక్ట్ MCP క్లయింట్](#3-డైరెక్ట్-mcp-క్లయింట్)
  - [4. AI-పవర్డ్ క్లయింట్](#4-ai-పవర్డ్-క్లయింట్)
- [ఉదాహరణలు రన్ చేయడం](#ఉదాహరణలు-రన్-చేయడం)
- [ఇది ఎలా కలిసి పని చేస్తుంది](#ఇది-ఎలా-కలిసి-పని-చేస్తుంది)
- [తర్వాతి దశలు](#తర్వాతి-దశలు)

## మీరు ఏమి నేర్చుకుంటారు

ఈ ట్యుటోరియల్ Model Context Protocol (MCP) ఉపయోగించి క్యాలిక్యులేటర్ సేవను ఎలా నిర్మించాలో వివరిస్తుంది. మీరు అర్థం చేసుకుంటారు:

- AI టూల్ గా ఉపయోగించగల సేవను ఎలా సృష్టించాలో
- MCP సేవలతో నేరుగా కమ్యూనికేషన్ ఎలా ఏర్పాటు చేయాలో
- AI నమూనాలు ఆటోమేటిక్ గా ఏ టూల్స్ ఉపయోగించాలో ఎంచుకునే విధానం
- నేరుగా ప్రోటోకాల్ కాల్స్ మరియు AI-ఎస్‌సిస్టెడ్ ఇంటరాక్షన్స్ మధ్య తేడా

## అవసరాలు

ప్రారంభించే ముందు, మీరు కలిగి ఉండాలి:
- Java 21 లేదా అంతకంటే ఎక్కువ వెర్షన్ ఇన్‌స్టాల్ చేయబడినది
- డిపెండెన్సీ మేనేజ్‌మెంట్‌ కోసం Maven
- Azure AI Foundry మోడల్ డిప్లాయ్‌మెంట్ (దాన్ని `azd up` తో ప్రొవిజన్ చేయండి — [అధ్యాయం 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md) చూడండి)
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), `az login` తో సైన్ ఇన్ అయి ఉండాలి (కీ లేకుండా ప్రామాణీకరణ)
- Java మరియు Spring Boot యొక్క మౌలిక అవగాహన

## ప్రాజెక్ట్ గঠনాన్ని అర్థం చేసుకోవడం

క్యాలిక్యులేటర్ ప్రాజెక్ట్ లో కొన్ని ముఖ్యమైన ఫైళ్లున్నాయి:

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

## ప్రధాన భాగాల వివరణ

### 1. ప్రధాన అనువర్తనం

**ఫైల్:** `McpServerApplication.java`

ఇది మా క్యాలిక్యులేటర్ సేవకు ప్రవేశ పాయింట్. ఇది ఒక సాధారణ Spring Boot అప్లికేషన్, కానీ ఒక ప్రత్యేక అదనంతో:

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

**ఇది చేస్తుంది:**
- పోర్టు 8080 పై Spring Boot వెబ్ సర్వర్ ప్రారంభిస్తుంది
- మా క్యాలిక్యులేటర్ పద్ధతులను MCP టూల్స్ గా అందించే `ToolCallbackProvider` సృష్టిస్తుంది
- `@Bean` అనోటేషన్ Spring కి ఇది ఒక భాగంగా నిర్వహించమని సూచిస్తుంది, తద్వారా ఇతర భాగాలు దీనిని ఉపయోగించగలవు

### 2. క్యాలిక్యులేటర్ సేవ

**ఫైల్:** `CalculatorService.java`

అక్కడే గణిత సంబంధిత అన్ని లాజిక్ జరుగుతాయి. ప్రతి పద్ధతి `@Tool` తో గుర్తింపు పొందింది, అంతగా MCP ద్వారా అందుబాటులో ఉంటుంది:

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
    
    // మరిన్ని కేలిక్యులేటర్ ఆపరేషన్లు...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**ప్రధాన లక్షణాలు:**

1. **`@Tool` అనోటేషన్**: MCP కి ఈ పద్ధతి బహిర్గత కస్టమర్ల ద్వారా కాల్ చేయవచ్చని తెలియజేస్తుంది
2. **స్పష్ట వివరణలు**: AI మోడల్స్ ఎప్పుడు ఏ టూల్ ఉపయోగించాలో అర్థం చేసుకునేందుకు వివరణలు ఉంటాయి
3. **సామంజస్యం గల రిటర్న్ ఫార్మాట్**: అన్ని ఆపరేషన్స్ "5.00 + 3.00 = 8.00" లాంటి మానవ పఠనీయ స్ట్రింగ్స్‌గా రిటర్న్ చేస్తాయి
4. **పొరపాటు హ్యాండ్లింగ్**: జీరోతో భాగించడంలో పొరపాటు, నెగటివ్ స్క్వేర్ రూట్స్ లో తప్పిద సందేశాలు అందిస్తుంది

**ఉపలబ్ధి ఆపరేషన్స్:**
- `add(a, b)` - రెండు సంఖ్యలను జత చేస్తుంది
- `subtract(a, b)` - రెండవ నుండి మొదటిదాన్ని తీసివేస్తుంది
- `multiply(a, b)` - రెండు సంఖ్యలను గుణిస్తుంది
- `divide(a, b)` - మొదటిని రెండవతో భాగిస్తుంది (జీరోచక్రంతో పరీక్షతో)
- `power(base, exponent)` - బేసును ఎగ్జిపోనెంట్ శక్తిలో పెంచుతుంది
- `squareRoot(number)` - స్క్వేర్ రూట్ లెక్కిస్తుంది (నెగటివ్ తనిఖీతో)
- `modulus(a, b)` - భాగించాక మిగిలిన మొత్తాన్ని ఇస్తుంది
- `absolute(number)` - సంపూర్ణ విలువ ఇస్తుంది
- `help()` - అన్ని ఆపరేషన్ల సమాచారం ఇస్తుంది

### 3. డైరెక్ట్ MCP క్లయింట్

**ఫైల్:** `SDKClient.java`

ఈ క్లయింట్ AI ఉపయోగించకుండా నేరుగా MCP సర్వర్ తో మాటాడుతుంది. ఇది ప్రత్యేక క్యాలిక్యులేటర్ ఫంక్షన్లను చేతితో కాల్ చేస్తుంది:

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
        
        // అందుబాటులో ఉన్న సాధనాల జాబితా
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // నిర్దిష్ట లెక్కింపు ఫంక్షన్లను పిలవండి
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

**ఇది చేస్తుంది:**
1. బిల్డర్ ప్యాటర్న్ ఉపయోగించి `http://localhost:8080` వద్ద క్యాలిక్యులేటర్ సర్వర్‌కి కనెక్ట్ అవుతుంది
2. అందుబాటులో ఉన్న అన్ని టూల్స్ (మా క్యాలిక్యులేటర్ ఫంక్షన్లు) జాబితా చేయబడతాయి
3. ఖచ్చితమైన పారామీటర్లతో స్పెసిఫిక్ ఫంక్షన్లను కాల్ చేస్తుంది
4. ఫలితాలను నేరుగా ప్రింట్ చేస్తుంది

**గమనిక:** ఈ ఉదాహరణ Spring AI 1.1.0-SNAPSHOT డిపెండెన్సీ ఉపయోగించింది, ఇందులో `WebFluxSseClientTransport` కోసం బిల్డర్ ప్యాటర్న్ పరిచయం చేయబడింది. మీరు పాత స్థిర వెర్షన్ ఉపయోగిస్తుంటే నేరుగా కన్‌స్ట్రక్టర్ ఉపయోగించవలసి ఉంటుంది.

**ఈ విధంగా ఉపయోగించాలి:** మీరు ఏ గణన చేయాలో ఖచ్చితంగా తెలుసుకుని దాన్ని ప్రోగ్రామాటిక్గా కాల్ చేయాలనుకుంటే.

### 4. AI-పవర్డ్ క్లయింట్

**ఫైల్:** `LangChain4jClient.java`

ఈ క్లయింట్ GPT-4o-mini AI మోడల్ ఉపయోగించి ఆటోమేటిక్ గా ఏ క్యాలిక్యులేటర్ టూల్స్ వాడాలో నిర్ణయిస్తుంది:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // AI మోడల్‌ను సెటప్ చేయండి (Azure AI Foundry, Microsoft Entra ID ద్వారా keyless auth)
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

        // మా క్యాల్క్యులేటర్ MCP సర్వర్‌కు కనెక్ట్ అవ్వండి
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // AI ఏమి చేస్తున్నదో చూపిస్తుంది
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // AIకి మా క్యాల్క్యులేటర్ టూల్స్‌కు ప్రాప్తి ఇవ్వండి
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // మా క్యాల్క్యులేటర్ ఉపయోగించగల AI బాట్ తయారు చేయండి
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // ఇప్పుడు మేము సహజ భాషలో గణన చేయమని AIని అడగచ్చు
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**ఇది చేస్తుంది:**
1. మీ GitHub టోకెన్ ఉపయోగించి AI మోడల్ కనెక్షన్ సృష్టిస్తుంది
2. AI ని మా MCP క్యాలిక్యులేటర్ సర్వర్‌కు కనెక్ట్ చేస్తుంది
3. AI కి మా అన్ని క్యాలిక్యులేటర్ టూల్స్ యాక్సెస్ ఇస్తుంది
4. "24.5 మరియు 17.3 యొక్క మొత్తం లెక్కించు" లాంటి సహజ భాషా అభ్యర్థనలను అనుమతిస్తుంది

**AI ఆటోమేటిక్ గా:**
- మీరు సంఖ్యలను జోడించమని అర్థం చేసుకుంటుంది
- `add` టూల్ ను ఎంచుకుంటుంది
- `add(24.5, 17.3)` ను కాల్ చేస్తుంది
- ఫలితాన్ని సహజ సమాధానంగా తిరిగి ఇస్తుంది

## ఉదాహరణలు రన్ చేయడం

### దశ 1: క్యాలిక్యులేటర్ సర్వర్ ప్రారంభించడం

మొదట, సైన్ ఇన్ చేసి Azure AI Foundry ఎండ్పాయింట్ సెట్ చేయండి (AI క్లయింట్ కోసం అవసరం — కీలో లేకుండా, API కీ అవసరం లేదు):

**విండోస్:**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**లినక్స్/మాక్‌ఓఎస్:**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

సర్వర్‌ను ప్రారంభించండి:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

సర్వర్ `http://localhost:8080` పై ప్రారంభమవుతుంది. మీరు ఈ విధంగా చూస్తారు:
```
Started McpServerApplication in X.XXX seconds
```

### దశ 2: డైరెక్ట్ క్లయింట్ తో పరీక్షించండి

పுதிய టెర్మినల్ లో, సర్వర్ ఇంకా పని చేస్తున్నపుడు, డైరెక్ట్ MCP క్లయింట్ నడపండి:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

మీకు ఈ విధంగా అవుట్పుట్ వస్తుంది:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### దశ 3: AI క్లయింట్ తో పరీక్షించండి

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

మీరు AI ఆటోమేటిక్ గా టూల్స్ ఉపయోగిస్తున్నట్లు చూస్తారు:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### దశ 4: MCP సర్వర్ ను మూసివేయండి

పరీక్ష పూర్తి చేసిన తర్వాత, AI క్లయింట్ టెర్మినల్ లో `Ctrl+C` నొక్కి నిలిపివేయండి. MCP సర్వర్ ఆపకపోతే కొనసాగుతుంది.
సర్వర్ ఆపడానికి, ఆ టెర్మినల్ లో `Ctrl+C` నొక్కండి.

## ఇది ఎలా కలిసి పని చేస్తుంది

AI ని మీరు "5 + 3 ఎంత?" అని అడిగినపుడు పూర్తి ప్రాసెస్ ఇలాగే ఉంటుంది:

1. **మీరు** సహజ భాషలో AI ని అడుగుతారు
2. **AI** మీ అభ్యర్థన విశ్లేషించి మీరు జోడింపు కోరుతారని తెలుసుకుంటుంది
3. **AI** MCP సర్వర్‌కు కాల్ చేస్తుంది: `add(5.0, 3.0)`
4. **క్యాలిక్యులేటర్ సేవ** చేస్తుంది: `5.0 + 3.0 = 8.0`
5. **క్యాలిక్యులేటర్ సేవ** తిరిగి ఇస్తుంది: `"5.00 + 3.00 = 8.00"`
6. **AI** ఫలితాన్ని స్వభావసిద్ధ సమాధానంగా మార్చుకుంటుంది
7. **మీకు** వస్తుంది: "5 మరియు 3 యొక్క మొత్తం 8"

## తర్వాతి దశలు

మరింత ఉదాహరణల కొరకు, [Chapter 04: Practical samples](../README.md) చూడండి

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**అస్వీకరణ**:
ఈ పత్రం AI అనువాద సేవ [Co-op Translator](https://github.com/Azure/co-op-translator) ఉపయోగించి అనువదించబడింది. మేము ఖచ్చితత్వానికి ప్రయత్నిస్తున్నప్పటికీ, ఆటోమేటెడ్ అనువాదాలు తప్పులు లేదా అసమగ్రతలను కలిగి ఉండవచ్చు. దాని స్వదేశ భాషలో ఉన్న అసలు పత్రాన్ని అధికారం కలిగిన మూలంగా పరిగణించాలి. కీలకమైన సమాచారం కోసం, ప్రొఫెషనల్ మానవ అనువాదాన్ని సిఫారసు చేస్తాము. ఈ అనువాదం ఉపయోగం వల్ల కలిగే ఏవైనా అపార్థాలు లేదా తప్పుదారులు కోసం మేము బాధ్యత వహించము.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->