# ప్రారంభ దశలకి MCP గణక యంత్రం ట్యుటోరియల్

## విషయ సూచిక

- [మీరు నేర్చుకునే విషయాలు](#మీరు-నేర్చుకునే-విషయాలు)
- [ముందస్తు అవసరాలు](#ముందస్తు-అవసరాలు)
- [ప్రాజెక్ట్ నిర్మాణం గురించి అవగాహన](#ప్రాజెక్ట్-నిర్మాణం-గురించి-అవగాహన)
- [ప్రధాన భాగాల వివరణ](#ప్రధాన-భాగాల-వివరణ)
  - [1. ప్రధాన అనువర్తనం](#1-ప్రధాన-అనువర్తనం)
  - [2. గణక సేవ](#2-గణక-సేవ)
  - [3. ప్రత్యక్ష MCP క్లయింట్](#3-ప్రత్యక్ష-mcp-క్లయింట్)
  - [4. AI ఆధారిత క్లయింట్](#4-ai-ఆధారిత-క్లయింట్)
- [ఉదాహరణలు నడిపించడం](#ఉదాహరణలు-నడిపించడం)
- [ఇవి ఎలా కలిసి పనిచేస్తాయి](#ఇవి-ఎలా-కలిసి-పనిచేస్తాయి)
- [తర్వాతి అడుగులు](#తరువాతి-అడుగులు)

## మీరు నేర్చుకునే విషయాలు

ఈ ట్యుటోరియల్ Model Context Protocol (MCP) ఉపయోగించి గణక సేవను ఎలా తయారు చేయాలో వివరించబడుతుంది. మీరు అర్థం చేసుకుంటారు:

- AI ఒక టూల్ గా ఉపయోగించే సర్వీస్ ఎలా సృష్టించాలి
- MCP సేవలతో ప్రత్యక్ష సంభాషణ ఎలా ఏర్పరుచుకోవాలి
- AI మోడళ్లు స్వయంచాలకంగా ఏ టూల్స్ ఉపయోగించాలో ఎలా ఎంచుకుంటాయో
- ప్రత్యక్ష ప్రోటోకాల్ కాల్స్ మరియు AI-సహాయక పరస్పర చర్యల మధ్య తేడా

## ముందస్తు అవసరాలు

ప్రారంభించక ముందు, మీరు కలిగి ఉండాలి:
- Java 21 లేదా అంతకుపైగా ఇన్‌స్టాల్ చేయబడినది
- డిపెండెన్సీ మేనేజ్‌మెంట్ కోసం Maven
- Azure AI Foundry మోడల్ డిప్లాయ్‌మెంట్ (దీనిని `azd up` తో ప్రావిజన్ చేయండి — [అధ్యాయం 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md) చూడండి)
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), `az login` తో సైన్ ఇన్ (కీ లెస్ ప్రామాణీకరణ)
- Java మరియు Spring Boot పై ప్రాథమిక అవగాహన

## ప్రాజెక్ట్ నిర్మాణం గురించి అవగాహన

గణక ప్రాజెక్ట్‌లో కొన్ని ముఖ్యమైన ఫైల్స్ ఉన్నాయి:

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

ఇది మన గణక సేవ యొక్క ప్రవేశ పాయింట్. ఇది ఒక సాధారణ Spring Boot అనువర్తనం, ఒక ప్రత్యేక అంశంతో:

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

**ఇది చేసినది:**
- 8080 పోర్ట్‌లో Spring Boot వెబ్ సర్వర్ ప్రారంభిస్తుంది
- మన గణక విధానాలను MCP టూల్స్ గా అందించే `ToolCallbackProvider` సృష్టిస్తుంది
- `@Bean` అనోటేషన్ Springకి దీనిని ఒక భాగంగా నిర్వహించాలని తెలియజేస్తుంది, దీన్ని ఇతర భాగాలు ఉపయోగించవచ్చు

### 2. గణక సేవ

**ఫైల్:** `CalculatorService.java`

ఇక్కడ అన్ని గణిత సూత్రాలు నిర్వర్తిస్తాయి. ప్రతి విధానం `@Tool` తో గుర్తింపబడింది, MCP ద్వారా అందుబాటులో ఉండటానికి:

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
    
    // మరింత కాల్క్యులేటర్ ఆపరేషన్లు...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**ప్రధాన లక్షణాలు:**

1. **`@Tool` అనోటేషన్**: MCPకి ఈ విధానం బాహ్య క్లయింట్లు పిలవగలదని తెలియజేస్తుంది
2. **స్పష్ట వివరణలు**: ప్రతి టూల్‌కు వివరణ ఉంటుంది, దీని ద్వారా AI మోడళ్లు ఎప్పుడు దీనిని ఉపయోగించాలో అర్థం చేసుకుంటాయి
3. **స్థిరమైన ఫలిత రూపం**: అన్ని ఆపరేషన్లు "5.00 + 3.00 = 8.00" వంటి మనుషుల చదవగల స్ట్రింగ్స్ తిరిగి ఇస్తాయి
4. **లోప నిర్వహణ**: 0తో భాగమొగడం లేదా రుణాత్మక వర్గమూలం పొరపాట్లు సందేశాలుగా తిరిగి వస్తాయి

**అందుబాటులో ఉన్న ఆపరేషన్లు:**
- `add(a, b)` - రెండు సంఖ్యలను జోడిస్తుంది
- `subtract(a, b)` - రెండవ సంఖ్యను మొదటిది నుండి తీసివేస్తుంది
- `multiply(a, b)` - రెండు సంఖ్యలను గుణించు
- `divide(a, b)` - మొదటి సంఖ్యను రెండవదితో భాగించు (సున్నా తనిఖీతో)
- `power(base, exponent)` - బేస్ ను ఎక్స్‌పోనెంట్ శక్తికి ఎత్తు
- `squareRoot(number)` - వర్గమూలం లెక్కించు (నెగటివ్ తనిఖీతో)
- `modulus(a, b)` - భాగించిన తర్వాత మిగిలినది (రిమైండర్) ఇస్తుంది
- `absolute(number)` - పరమ శూన్య విలువ ఇస్తుంది
- `help()` - అన్ని ఆపరేషన్ల గురించి సమాచారం ఇస్తుంది

### 3. ప్రత్యక్ష MCP క్లయింట్

**ఫైల్:** `SDKClient.java`

ఈ క్లయింట్ AI పంపిణీ లేకుండా MCP సర్వర్‌తో నేరుగా మాట్లాడుతుంది. ఇది ప్రత్యేక గణక ఫంక్షన్లను మానవీయంగా పిలుస్తుంది:

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
        
        // అందుబాటులో ఉన్న సాధనాలను జాబితా చేయండి
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // నిర్దిష్ట గణన యంత్ర ఫంక్షన్లను పిలవండి
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

**ఇది చేసినది:**
1. `http://localhost:8080` సర్వర్‌తో బిల్డర్ ప్యాటర్న్ ఉపయోగించి కనెక్ట్ అవుతుంది
2. అందుబాటులో ఉన్న అన్ని టూల్స్ (మన గణక ఫంక్షన్లు) ను జాబితా చేస్తుంది
3. నిర్దిష్ట ఫంక్షన్లను ఖచ్చితమైన పారామితులతో పిలుస్తుంది
4. ఫలితాలను నేరుగా ప్రింట్ చేస్తుంది

**గమనిక:** ఈ ఉదాహరణ Spring AI 1.1.0-SNAPSHOT డిపెండెన్సీ ఉపయోగిస్తుంది, ఇది WebFluxSseClientTransport కోసం బిల్డర్ ప్యాటర్న్ ప్రవేశపెట్టింది. మీరు పాత, స్థిర వెర్షన్ ఉపయోగిస్తుంటే నేరుగా కాన్స్ట్రక్టర్ ఉపయోగించాల్సి ఉంటుంది.

**ఏ సమయంలో ఉపయోగించాలి:** మీరు ఖచ్చితంగా ఏ లెక్కింపును చేయాలని అనుకుంటున్నారో తెలిసి, దానిని ప్రోగ్రామాటిక్ గా పిలవాలనుకున్నప్పుడు.

### 4. AI ఆధారిత క్లయింట్

**ఫైల్:** `LangChain4jClient.java`

ఈ క్లయింట్ AI మోడల్ (GPT-4o-mini) ఉపయోగిస్తుంది, ఇది ఏ గణక టూల్ ఉపయోగించాలో ఆటోమేటిక్‌గా నిర్ణయిస్తుంది:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // AI మోడల్‌ను సెటప్ చేయండి (Azure AI Foundry, Microsoft Entra ID ద్వారా కీలు రహిత గుర్తింపు)
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

        // మా కాల్క్యులేటర్ MCP సర్వర్కు కనెక్ట్ అవ్వండి
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // AI ఏమి చేస్తున్నదో చూపిస్తుంది
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // AIకి మా కాల్క్యులేటర్ టూల్స్ యాక్సెస్ ఇవ్వండి
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // మా కాల్క్యులేటర్ ఉపయోగించగల AI బాట్ తయారు చేయండి
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // ఇప్పుడు AIకి సహజ భాషలో గణనలు చేయమని అడగవచ్చు
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**ఇది చేసినది:**
1. కీ లెస్ ప్రామాణీకరణ (Microsoft Entra ID) ద్వారా AI మోడల్ కనెక్షన్ సృష్టిస్తుంది
2. AIని మన గణక MCP సర్వర్‌కు కనెక్ట్ చేస్తుంది
3. AIకి మన గణక టూల్స్ అందించబడతాయి
4. "24.5 మరియు 17.3 యొక్క మొత్తం లెక్కించు" లాంటి సహజ భాషా అభ్యర్థనలు ఆమోదిస్తుంది

**AI ఆటోమేటిక్‌గా:**
- మీరు సంఖ్యలు జోడించాలని అర్థం చేసుకుంటుంది
- `add` టూల్ ఎంచుకుంటుంది
- `add(24.5, 17.3)` ను పిలుస్తుంది
- ఫలితాన్ని సహజ ప్రతిస్పందనగా తిరిగిస్తుంది

## ఉదాహరణలు నడిపించడం

### దశ 1: గణన సర్వర్ ప్రారంభించండి

మొదట, సైన్ ఇన్ అయి Azure AI Foundry ఎండ్పాయింట్ సెట్ చేయండి (AI క్లయింట్ కోసం అవసరం — కీ లెస్, API కీ అవసరం లేదు):

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

సర్వర్ ప్రారంభించండి:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

సర్వర్ `http://localhost:8080` వద్ద ప్రారంభమవుతుంది. మీరు దీన్ని చూడగలరు:
```
Started McpServerApplication in X.XXX seconds
```

### దశ 2: ప్రత్యక్ష క్లయింట్‌తో పరీక్షించండి

సర్వర్ నడుస్తున్న సమయంలో మరో టెర్మినల్‌లో ప్రత్యక్ష MCP క్లయింట్ నడపండి:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

త్వరగా ఈ విధంగా అవుట్పుట్ వస్తుంది:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### దశ 3: AI క్లయింట్‌తో పరీక్షించండి

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

AI స్వయంచాలకంగా టూల్స్ ఉపయోగిస్తుందని చూడగలరు:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### దశ 4: MCP సర్వర్ నిలిపివేయండి

పరీక్ష పూర్తయిన తర్వాత కంట్రోల్+C తో AI క్లయింట్ ఆపండి. MCP సర్వర్ మీరు ఆపేవరకు నడుస్తూనే ఉంటుంది.
సర్వర్ నిలిపేయడానికి అది నడుస్తున్న టెర్మినల్‌లో Ctrl+C నొక్కండి.

## ఇవి ఎలా కలిసి పనిచేస్తాయి

మీరు AIకి "5 + 3 అంటే ఎంత?" అని అడిగినప్పుడు పూర్తయిన ప్రక్రియ:

1. **మీరు** సహజ భాషలో AIని అడుగుతారు
2. **AI** మీ అభ్యర్థనను విశ్లేషించి మీరు జోడింపులు కోరుతున్నారని తెలుసుకుంటుంది
3. **AI** MCP సర్వర్‌ను పిలుస్తుంది: `add(5.0, 3.0)`
4. **గణక సేవ**.perform చేస్తుంది: `5.0 + 3.0 = 8.0`
5. **గణక సేవ** తిరిగి ఇచ్చేది: `"5.00 + 3.00 = 8.00"`
6. **AI** ఫలితాన్ని అందుకొని సహజ ప్రతిస్పందనగా మార్చుతుంది
7. **మీకు** మలిరేది: "5 మరియు 3 యొక్క మొత్తం 8"

## తరువాతి అడుగులు

ఇంకా ఉదాహరణల కోసం, చూడండి [అధ్యాయం 04: ప్రత్యక్ష నమూనాలు](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**అస్వీకరణ**:
ఈ పత్రం AI అనువాద సేవ [Co-op Translator](https://github.com/Azure/co-op-translator) ఉపయోగించి అనువదించబడింది. మేము ఖచ్చితత్వానికి ప్రయత్నిస్తున్నప్పటికీ, ఆటోమేటెడ్ అనువాదాలు తప్పులు లేదా అసమగ్రతలను కలిగి ఉండవచ్చు. దాని స్వదేశ భాషలో ఉన్న అసలు పత్రాన్ని అధికారం కలిగిన మూలంగా పరిగణించాలి. కీలకమైన సమాచారం కోసం, ప్రొఫెషనల్ మానవ అనువాదాన్ని సిఫారసు చేస్తాము. ఈ అనువాదం ఉపయోగం వల్ల కలిగే ఏవైనా అపార్థాలు లేదా తప్పుదారులు కోసం మేము బాధ్యత వహించము.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->