# MCP கணக்கிடும் கருவி பயிற்சி முறையறை

## உள்ளடக்கம்

- [நீங்கள் படிக்கப்போகும் விஷயங்கள்](#நீங்கள்-படிக்கப்போகும்-விஷயங்கள்)
- [முந்தைய அறிவு](#முந்தைய-அறிவு)
- [திட்ட அமைப்பை புரிந்துகொள்வது](#திட்ட-அமைப்பை-புரிந்துகொள்வது)
- [முக்கிய கூறுகள் விளக்கம்](#முக்கிய-கூறுகள்-விளக்கம்)
  - [1. முக்கிய பயன்பாடு](#1-முக்கிய-பயன்பாடு)
  - [2. கணக்கிடும் சேவை](#2-கணக்கிடும்-சேவை)
  - [3. நேரடி MCP கிளையன்ட்](#3-நேரடி-mcp-கிளையன்ட்)
  - [4. AI இயக்கும் கிளையன்ட்](#4-ai-இயக்கும்-கிளையன்ட்)
- [எடுத்துக்காட்டு நிரல்கள் இயக்குவது](#எடுத்துக்காட்டு-நிரல்கள்-இயக்குவது)
- [அனைத்து ஒன்றாக எவ்வாறு வேலை செய்கிறது](#அனைத்து-ஒன்றாக-எவ்வாறு-வேலை-செய்கிறது)
- [அடுத்த படிகள்](#அடுத்த-படிகள்)

## நீங்கள் படிக்கப்போகும் விஷயங்கள்

இந்த பயிற்சி முறையில் Model Context Protocol (MCP) பயன்படுத்தி கணக்கிடும் சேவையை உருவாக்குவது எப்படி என்று விளக்கப்படுகிறது. நீங்கள் புரிந்துகொள்ளப்போகிறது:

- AI கருவியாகப் பயன்படுத்தக்கூடிய சேவையை உருவாக்குவது எப்படி
- MCP சேவைகளுடன் நேரடி தொடர்பை அமைக்குவது எப்படி
- AI மாதிரிகள் தானாக திறன்களை எடுத்து பயன்படுத்துவது எப்படி
- நேரடி பிராடோகால் அழைப்புகளும் AI உதவியுடன் நடக்கும் தொடர்புகளும் எப்படி வேறுபடுகின்றன

## முந்தைய அறிவு

தொடங்குவதற்கு முன், பின்வருவன இருக்க வேண்டும்:
- Java 21 அல்லது அதற்கு மேல் பதிப்பு நிறுவியிருத்தல்
- Maven மூலம் சார்பு மேலாண்மை
- Azure AI Foundry மாதிரி தோற்றம் (azd up கொண்டு உருவாக்கவும் — [பகுதி 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md) பார்க்கவும்)
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) நிறுவியிருத்தல் மற்றும் `az login` மூலம் உள்நுழைவு (keyless auth)
- Java மற்றும் Spring Boot அடிப்படைக் கற்றல்

## திட்ட அமைப்பை புரிந்துகொள்வது

கணக்கிடும் திட்டத்தில் முக்கியமான பல கோப்புகள் உள்ளன:

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

## முக்கிய கூறுகள் விளக்கம்

### 1. முக்கிய பயன்பாடு

**கோப்பு:** `McpServerApplication.java`

இது எங்கள் கணக்கிடும் சேவையின் முகப்புக் கோப்பாகும். இது மொத்தமாக Spring Boot பயன்பாடு ஆகும், ஆனால் இதில் ஒரு சிறப்பு சேர்க்கை உள்ளது:

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

**இது என்ன செய்கிறது:**
- 8080 போர்டில் Spring Boot வலை சேவையகத்தை துவக்குகிறது
- எங்கள் கணக்கிடும் முறைமைகளை MCP கருவிகளாக கிடைக்கும் வகையில் `ToolCallbackProvider` உருவாக்குகிறது
- `@Bean` குறிப்பு இதை Spring மற்ற கூறுகள் பயன்படுத்தும் பொருட்டு நிர்வகிக்கும் என்பதை குறிக்கிறது

### 2. கணக்கிடும் சேவை

**கோப்பு:** `CalculatorService.java`

இங்கு எல்லா கணிதச் செயல்பாடுகளும் நடக்கின்றன. ஒவ்வொரு முறைமையும் `@Tool` குறியீட்டுடன் குறிக்கப்பட்டுள்ளது அன்று MCP மூலம் அழைக்கப்படுகிறது:

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
    
    // இன்னும் கணக்கீடு செயல்பாடுகள்...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**முக்கிய அம்சங்கள்:**

1. **`@Tool` குறிப்பு**: இந்த முறைமை MCPக்கு வெளியில் இருந்து அழைக்கப்பட வாய்ப்பு என அறிவிக்கிறது  
2. **தெளிவான விளக்கங்கள்**: ஒவ்வொரு கருவிக்கும் AI மாதிரிகள் எப்போது பயன்படுத்துவது என்று புரிய இருக்கும் விளக்கம் கொடுக்கப்பட்டுள்ளது  
3. **ஒரே மாதிரியில் திரும்ப வழங்கல்**: அனைத்து கணக்கீடுகளும் "5.00 + 3.00 = 8.00" போன்ற மனிதர்களுக்கு வாசிக்கக்கூடிய சரங்களைத் திருப்புகின்றன  
4. **பிழை கையாளல்**: பூஜ்யத்தில் வகுக்குதல் மற்றும் எதிர்மறை வேர்க்கில்கள் பிழை செய்திகளைத் தருகின்றன  

**கிடைக்கும் செயல்பாடுகள்:**
- `add(a, b)` - இரண்டு எண்களை கூட்டும்  
- `subtract(a, b)` - இரண்டாவது எணால் முதல் எண்ணை கழிக்கும்  
- `multiply(a, b)` - இரண்டு எண்களை பெருக்குகிறது  
- `divide(a, b)` - முதல் எண்ணை இரண்டாவது எண்ணால் வகுக்கிறது (பூஜ்யம் பரிசோதனை உடன்)  
- `power(base, exponent)` - அடிப்படை எண்ணை சக்தி எணுக்கு உயர்த்துகிறது  
- `squareRoot(number)` - வேர்க்கிளை கணக்கிடுகிறது (எதிர்மறை பரிசோதனை உடன்)  
- `modulus(a, b)` - வகுத்தல் மீதி பெறும்  
- `absolute(number)` - முழு மதிப்பை திருப்புகிறது  
- `help()` - பணி அனைத்து செயல்பாடுகளிக்கும் விவரத்தை தருகிறது  

### 3. நேரடி MCP கிளையன்ட்

**கோப்பு:** `SDKClient.java`

இந்த கிளையன்ட் AIயை பயன்படுத்தாமை MCP சேவையகத்துடன் நேரடியாக பேசுகிறது. கணக்கிடும் குறிப்பிட்ட செயல்பாடுகளை தனி கையால் அழைக்கிறது:

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
        
        // பயன்படுத்தக்கூடிய உபகரணங்களை பட்டியலிடு
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // குறிப்பிட்ட கணக்கு செயல்பாடுகளை அழைக்கவும்
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

**இது என்ன செய்கிறது:**
1. `http://localhost:8080` முகவரியில் உள்ள கணக்கிடும் சேவையகத்துடன் புரவல் வடிவமைப்பை பயன்படுத்தித் தொடர்பு கொள்கிறது  
2. அனைத்து கிடைக்கும் கருவிப்பட்டியலைப் பெறுகிறது  
3. கண்டிப்பான அளவுருக்களைப் பயன்படுத்தி தேர்ந்தெடுக்கப்பட்ட செயல்பாடுகளை அழைக்கிறது  
4. முடிவுகளை நேரடியாக அச்சிடுகிறது  

**குறிப்பு:** இந்த எடுத்துக்காட்டு Spring AI 1.1.0-SNAPSHOT சார்பு பயன்படுத்துகிறது, இது `WebFluxSseClientTransport`க்கு புரவல் வடிவமைப்பை அறிமுகப்படுத்தியுள்ளது. பழைய நிரந்தர பதிப்பு பயன்படுத்தினால் நேரடியான கட்டமைப்பை பயன்படுத்த வேண்டியிருக்கும்.  

**எப்போது பயன்படுத்துவது:** நீங்கள் எந்த கணக்கீட்டைக் கணக்கிடவேண்டும் என்று நிச்சயமாக தெரிந்திருக்கும்போது மற்றும் அதை நிரலாக அழைக்க விரும்பும்போது.  

### 4. AI இயக்கும் கிளையன்ட்

**கோப்பு:** `LangChain4jClient.java`

இந்த கிளையன்ட் GPT-4o-mini போன்ற AI மாதிரியை பயன்படுத்தி எந்த கணக்கிடும் கருவிகளைப் பயன்படுத்த வேண்டும் என்று தானாக முடிவு செய்கிறது:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // AI மாதிரியை அமைக்கவும் (Azure AI Foundry, Microsoft Entra ID வழியாக விசைமூன்றற்ற அங்கீகாரம்)
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

        // எங்கள் கணக்காய்வாளர் MCP சேவையகத்துடன் இணைக்கவும்
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // AI என்ன செய்கிறது என்பதை பார்க்கவும்
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // AIக்கு எங்கள் கணக்காய்வாளர் கருவிகளுக்கு அணுகல் கொடுக்கவும்
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // எங்கள் கணக்காய்வாளரை பயன்படுத்தக்கூடிய AI வாட் உருவாக்கவும்
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // இப்போது நாம் இயல்பான மொழியில் கணக்குகளை செய்ய AIஐ கேட்கலாம்
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**இது என்ன செய்கிறது:**
1. உங்கள் GitHub டோக்கனை பயன்படுத்தி AI மாதிரி இணைப்பை உருவாக்குகிறது  
2. AIயை எங்கள் MCP கணக்கிடும் சேவையகத்துடன் இணைக்கிறது  
3. AIக்கு அனைத்து கணக்கிடும் கருவிகளுக்கும் அணுகலை வழங்குகிறது  
4. "24.5 மற்றும் 17.3 கூட்டுவதைக் கணக்கிடு" போன்ற இயற்கை மொழி கோரிக்கைகளை அனுமதிக்கிறது  

**AI தானாகச் செய்கிறது:**
- நீங்கள் எண்களை கூட்ட விரும்புகிறீர்கள் என்பதை புரிந்து கொள்கிறது  
- `add` கருவியை தேர்வு செய்கிறது  
- `add(24.5, 17.3)` அழைக்கிறது  
- முடிவை இயற்கையான பதிலாக அளிக்கிறது  

## எடுத்துக்காட்டு நிரல்கள் இயக்குவது

### படி 1: கணக்கிடும் சேவையகத்தைக் துவக்குங்கள்

முதலில், உள்நுழைந்து உங்கள் Azure AI Foundry முகவரியை அமைக்கவும் (AI கிளையன்டிற்குக் தேவையானது — keyless auth, API விசையில்லை):

**விண்டோஸ்:**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**லினக்ஸ்/மெக்OS:**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

சேவையகத்தை துவக்கவும்:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

சேவையகம் `http://localhost:8080`இல் துவங்கும். இதைப் போல காண்பீர்கள்:
```
Started McpServerApplication in X.XXX seconds
```

### படி 2: நேரடி கிளையன்டுடன் சோதனை

ஒரு **புதிய** டெர்மினலில் சேவையகம் இயங்கிக்கொண்டே இருந்தபோது, நேரடி MCP கிளையன்டை இயக்கவும்:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

இதுபோன்ற வெளியீடு காணப்படும்:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### படி 3: AI கிளையன்டுடன் சோதனை

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

AI தானாக கருவிகளைப் பயன்படுத்திக் கொண்டிருப்பதைப் பார்க்கலாம்:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### படி 4: MCP சேவையகத்தைப் பற்றிக்கொள்ளுங்கள்

சோதனை முடிந்ததும் AI கிளையன்டை `Ctrl+C` அழுத்தி நிறுத்தலாம். MCP சேவையகம் நீங்கள் நிறுத்தும் வரை இயங்கிக் கொண்டிருக்கும்.  
சேவையகத்தை நிறுத்த, சேவையகம் இயங்கும் டெர்மினலில் `Ctrl+C` அழுத்தவும்.  

## அனைத்து ஒன்றாக எவ்வாறு வேலை செய்கிறது

AIயிடம் "5 + 3 என்ன?" என்று கேட்டபோது முழு செயற்தெரிவுச்செயல் இவ்வாறு இருக்கும்:

1. **நீங்கள்** இயற்கை மொழியில் AIயிடம் கேட்கிறீர்கள்  
2. **AI** உங்கள் கோரிக்கையை பகுப்பாய்வு செய்து கூடுதலே தேவையானது என்பதை அறிகிறது  
3. **AI** MCP சேவையகத்தை அழைக்கிறது: `add(5.0, 3.0)`  
4. **கணக்கிடும் சேவை** செய்வதை செய்கிறது: `5.0 + 3.0 = 8.0`  
5. **கணக்கிடும் சேவை** பதிலளிக்கிறது: `"5.00 + 3.00 = 8.00"`  
6. **AI** முடிவை பெறுகிறது மற்றும் இயற்கை பதிலாக வடிகிறது  
7. **நீங்கள்** பெறுவது: "5 மற்றும் 3 கூட்டல் 8 ஆகும்"  

## அடுத்த படிகள்

மேலும் எடுத்துக்காட்டுகளுக்கு, [பகுதி 04: நடைமுறை மாதிரிகள்](../README.md) பார்க்கவும்

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**மறுப்பு**:
இந்த ஆவணம் AI மொழிபெயர்ப்பு சேவை [Co-op Translator](https://github.com/Azure/co-op-translator) பயன்படுத்தி மொழிபெயர்க்கப்பட்டுள்ளது. நாங்கள் துல்லியத்திற்காக முயற்சி செய்துள்ளோம், ஆனால் தானாக செய்யப்படும் மொழிபெயர்ப்புகளில் பிழைகள் அல்லது தவறுகள் இருக்கலாம் என்பதை கவனத்தில் கொள்ளவும். அசல் ஆவணம் அதன் தாய்மொழியில் அதிகாரப்பூர்வ ஆதாரமாக கருதப்பட வேண்டும். முக்கியமான தகவல்களுக்கு, தொழில்நுட்பமான மனித மொழிபெயர்ப்பு பரிந்துரைக்கப்படுகிறது. இந்த மொழிபெயர்ப்பைப் பயன்படுத்துவதால் ஏற்படும் எந்த தவறான புரிதல்கள் அல்லது தவறான விளக்கத்திற்கும் நாங்கள் பொறுப்பில்வில்லை.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->