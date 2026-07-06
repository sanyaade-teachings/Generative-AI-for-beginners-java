# ஆரம்பக்காரர்களுக்கான MCP கணக்கிதாரி படிக்கட்டு

## உள்ளடக்கம்

- [நீங்கள் கற்பது என்ன](#நீங்கள்-கற்பது-என்ன)
- [முன்னிருப்பு செயற்பாடுகள்](#முன்னிருப்பு-செயற்பாடுகள்)
- [திட்ட அமைப்பை புரிவது](#திட்ட-அமைப்பை-புரிவது)
- [முக்கிய கூறுகள் விளக்கம்](#முக்கிய-கூறுகள்-விளக்கம்)
  - [1. முதன்மை செயலி](#1-முதன்மை-செயலி)
  - [2. கணக்கிதாரி சேவை](#2-கணக்கிதாரி-சேவை)
  - [3. நேரடி MCP கிளையண்ட்](#3-நேரடி-mcp-கிளையண்ட்)
  - [4. AI இயக்கக் கிளையண்ட்](#4-ai-இயக்கக்-கிளையண்ட்)
- [உதாரணங்களை இயக்குதல்](#உதாரணங்களை-இயக்குதல்)
- [எப்படி எல்லாம் ஒன்றாக வேலை செய்கிறது](#எப்படி-எல்லாம்-ஒன்றாக-வேலை-செய்கிறது)
- [அடுத்த படிகள்](#அடுத்த-படிகள்)

## நீங்கள் கற்பது என்ன

இந்த படிக்கட்டு Model Context Protocol (MCP) பயன்படுத்தி ஒரு கணக்கிதாரி சேவையை எப்படி உருவாக்குவது என்பதை விளக்குகிறது. நீங்கள் புரிந்துகொள்வீர்கள்:

- AI ஒரு கருவியாக பயன்படுத்தக்கூடிய சேவையை எப்படி உருவாக்குவது
- MCP சேவைகள் உடன் நேரடி தொடர்பை எப்படி அமைக்க வேண்டும்
- AI மாடல்கள் தானாகவே எந்த கருவிகளை பயன்படுத்த வேண்டும் என்பதை எவ்வாறு தேர்வு செய்கிறது
- நேரடி நடைமுறை அழைப்புகள் மற்றும் AI உதவியுடன் இடையிலான வேறுபாடு

## முன்னிருப்பு செயற்பாடுகள்

தொடங்குவதற்கு முன், நீங்கள் கீழ்கண்டவை வைத்திருக்க வேண்டும்:
- Java 21 அல்லது அதற்கும் மேல் நிறுவப்பட்டிருப்பதற்காக
- சார்பு மேலாண்மை க்காக Maven
- Azure AI Foundry மாடல் பிம்பம் (தளர்வதற்கு `azd up` வெளிப்படுத்து — [அத்தியாயம் 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md) காண்க)
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), `az login` உடன் கையொப்பமிட்டல் (கீ இல்லாத அங்கீகாரம்)
- Java மற்றும் Spring Boot பற்றிய அடிப்படை அறிவு

## திட்ட அமைப்பை புரிவது

கணக்கிதாரி திட்டத்தில் சில முக்கியமான கோப்புகள் உள்ளன:

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

### 1. முதன்மை செயலி

**கோப்பு:** `McpServerApplication.java`

இது எங்கள் கணக்கிதாரி சேவையின் நுழைவு புள்ளி. இது ஒரு சாதாரண Spring Boot செயலி ஒரு சிறப்பு கூடுதல் உடன்:

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

**இது செய்கிறது:**
- போர்ட் 8080 இல் Spring Boot வலை சேவையகம் துவக்குகிறது
- எங்கள் கணக்கிதாரி நுட்பங்கள் MCP கருவிகளாக கிடைக்க ஒரு `ToolCallbackProvider` உருவாக்குகிறது
- `@Bean` குறிப்பு Spring இல் இந்த கூறு மற்ற பகுதிகள் பயன்படுத்துவதற்கு மேற்கொண்டிருப்பதாக நிர்வகிக்க சொல்கிறது

### 2. கணக்கிதாரி சேவை

**கோப்பு:** `CalculatorService.java`

இதுதான் அனைத்து கணிதக் கணக்கீடுகளும் நடக்கும் இடம். ஒவ்வொரு முறை மேற்கொள்ளும் செயல்கள் `@Tool` குறிச்சொல்லுடன் குறிக்கப்பட்டுள்ளன MCP வழியாக அழைக்கக்கூடியதாக இருக்க:

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
    
    // மேலும் கணினி செயல்பாடுகள்...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**முக்கிய அம்சங்கள்:**

1. **`@Tool` குறிச்சொல்**: இந்த முறை வெளிப்புற கிளையண்டுகளால் அழைக்கப்பட முடியும் என்று MCP அறிவுறுத்துகிறது
2. **தெளிவான விளக்கங்கள்**: ஒவ்வொரு கருவிக்கும் AI மாடல்கள் எப்போது அதை பயன்படுத்துவது என புரிய உதவும் விளக்கம் உள்ளது
3. **இணக்கம் ஒரே மாதிரியில்**: அனைத்து செயல்பாடுகளும் மனிதருக்கு வாசிக்க எளிதான "5.00 + 3.00 = 8.00" போன்ற சரங்கள் கொடுக்கின்றன
4. **பிழை கையாளல்**: பத்து பட்டிலிருந்து வகுத்தல் மற்றும் நெகட்டிவ் வேர்க்கோடு பிழை செய்திகள் தருகிறது

**கிடைக்கும் செயல்பாடுகள்:**
- `add(a, b)` - இரண்டு எண்களை கூட்டுகிறது
- `subtract(a, b)` - இரண்டாவது எணிலிருந்து முதல்வத்தை கழிக்கிறது
- `multiply(a, b)` - இரண்டு எண்களை பெருக்குகிறது
- `divide(a, b)` - முதன்மையான எண்ணை இரண்டாவது எண்ணால் வகுக்கிறது (பூஜ்ஜியச் சோதனை உடன்)
- `power(base, exponent)` - அடிப்படை எண்ணை உயர்த்துகிறது
- `squareRoot(number)` - விரைவு வேர்க்கோடு கணக்கிடுகிறது (நெகட்டிவ் சோதனை உடன்)
- `modulus(a, b)` - வகுப்பின் மீதி எண்ணை தருகிறது
- `absolute(number)` - மூல மதிப்பை திருப்புகிறது
- `help()` - அனைத்து செயல்பாடுகளையும் குறித்த தகவலை திருப்புகிறது

### 3. நேரடி MCP கிளையண்ட்

**கோப்பு:** `SDKClient.java`

இந்த கிளையண்ட் AI பயன்படுத்தாமல் MCP சர்வருக்கு நேரடி தொடர்பு கொள்கிறது. இது குறிப்பிட்ட கணக்கிதாரி செயல்பாடுகளை மானுவல் அழைக்கிறது:

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
        
        // கிடைக்கும் கருவிகளை பட்டியலிடுக
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // குறிப்பிட்ட கணக்கீட்டு செயல்பாடுகளை அழைக்கவும்
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

**இது செய்கிறது:**
1. தொகுப்பாளர் முறையில் `http://localhost:8080` என்ற கணக்கிதாரி சர்வருக்கு இணைக்கிறது
2. கிடைக்கக்கூடிய அனைத்து கருவிகளையும் பட்டியலிடுகிறது (எங்கள் கணக்கிதாரி செயல்பாடுகள்)
3. குறிப்பிட்ட செயல்பாடுகளை துல்லியமான அளவுருக்களுடன் அழைக்கிறது
4. முடிவுகளை நேரடியாக அச்சிடுகிறது

**குறிப்பு:** இந்த உதாரணம் Spring AI 1.1.0-SNAPSHOT சார்பை பயன்படுத்துகிறது, இது `WebFluxSseClientTransport` க்கான தொகுப்பாளர் முறையை அறிமுகம் செய்தது. பழைய நிலையாகிய பதிப்புகளை நீங்கள் பயன்படுத்தினால் நேரடி கட்டமைப்பாளர் பயன்படுத்த வேண்டியிருக்கலாம்.

**எப்பொழுது பயன்படுத்துவது:** கோட்படுத்தி நீங்கள் சேர்க்க வேண்டிய கணக்கீட்டை நிச்சயம் தெரிந்தால் மற்றும் அதை நிரலாக அழைக்க விரும்பினால்.

### 4. AI இயக்கக் கிளையண்ட்

**கோப்பு:** `LangChain4jClient.java`

இந்த கிளையண்ட் AI மாடல் (GPT-4o-mini) பயன்படுத்தி தானாக எந்த கணக்கிதாரி கருவிகளை தேர்ந்தெடுக்க முடியும்:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // AI மாதிரியை அமைக்கவும் (Azure AI Foundry, Microsoft Entra ID மூலம் விசையில்லா அங்கீகாரம்)
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

        // எங்கள் கணக்கீட்டு MCP சேவையகத்துடன் இணைக்கவும்
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // AI என்ன செய்கிறது என்பதை காட்டுகிறது
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // AIக்கு எங்கள் கணக்கீட்டு கருவிகளுக்கு அணுகல் வழங்கவும்
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // எங்கள் கணக்கீடு பயன்படுத்தக்கூடிய AI பாட்டைப் படைக்கவும்
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // இப்போது இயற்கை மொழியில் கணக்கீடுகளை செய்ய AIக்கு கேட்கலாம்
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**இது செய்கிறது:**
1. கீ இல்லாத அங்கீகாரப்படி (Microsoft Entra ID) AI மாடல் தொடர்பை உருவாக்குகிறது
2. AI-ஐ எங்கள் MCP கணக்கிதாரியுடன் இணைக்கிறது
3. AIக்கு எங்கள் அனைத்து கருவிகளுக்கும் அணுகலாக அனுமதிக்கிறது
4. "24.5 மற்றும் 17.3 இரண்டின் கூட்டை கணக்கிடு" போன்ற இயற்கை மொழி கோரிக்கைகளை அனுமதிக்கிறது

**AI தானாக:**
- நீங்கள் எண்களை கூட்ட விரும்புகிறீர்கள் என்று புரிந்து கொள்கிறது
- `add` கருவியை தேர்ந்தெடுக்கிறது
- `add(24.5, 17.3)` அழைக்கிறது
- இயற்கை பதிலில் முடிவை திருப்புகிறது

## உதாரணங்களை இயக்குதல்

### படி 1: கணக்கிதாரி சர்வரை துவக்கவும்

முதலில், உள்நுழைந்து Azure AI Foundry முடிவுப் புள்ளியை அமைக்கவும் (AI கிளையண்ட் பயன்பாட்டிற்கு தேவையானது — கீ இல்லாத அங்கீகாரம்):

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

சர்வரை துவக்கு:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

சர்வர் `http://localhost:8080` இல் துவங்கும். நீங்கள் காண்பீர்கள்:
```
Started McpServerApplication in X.XXX seconds
```

### படி 2: நேரடி கிளையண்டுடன் சோதனை

சர்வர் இயங்கிவிருக்கும் நிலையில் புதிய டெர்மினலில் நேரடி MCP கிளையண்டை இயக்கவும்:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

உங்களுக்குப் பின்வரும் வெளியீடு வரும்:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### படி 3: AI கிளையண்டுடன் சோதனை

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

AI தானாக கருவிகளை பயன்படுத்துவது உங்களுக்குப் பின்வரும் பதிவைக் காண்பிக்கும்:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### படி 4: MCP சர்வரை மூடவும்

சோதனை முடிந்தபோது, AI கிளையண்டை அதன் டெர்மினலில் `Ctrl+C` அழுத்தி நிறுத்தவும். MCP சர்வர் நீங்கள் நிறுத்தும் வரை இயங்கிக் கொண்டிருக்கும்.
சர்வரை நிறுத்த, இயங்கி இருக்கும் டெர்மினலில் `Ctrl+C` ஐ அழுத்தவும்.

## எப்படி எல்லாம் ஒன்றாக வேலை செய்கிறது

AIயிடம் நீங்கள் "5 + 3 என்ன?" என்று கேட்டால் முழு வேலைப்பாச்சல்:

1. **நீங்கள்** இயற்கை மொழியில் கேட்டீர்கள்
2. **AI** உங்கள் கோரிக்கையை பகுப்பாய்வு செய்து கூட்டல்தான் தேவையென்பதை உணர்ந்தது
3. **AI** MCP சர்வருக்கு அழைக்கிறான்: `add(5.0, 3.0)`
4. **கணக்கிதாரி சேவை** செய்கிறது: `5.0 + 3.0 = 8.0`
5. **கணக்கிதாரி சேவை** திருப்புகிறது: `"5.00 + 3.00 = 8.00"`
6. **AI** முடிவை பெற்று இயற்கை பதிலெழுது
7. **நீங்கள்** பெறுகிறீர்கள்: "5 மற்றும் 3 இன் கூட்டு 8 ஆகும்"

## அடுத்த படிகள்

மேலும் உதாரணங்களுக்கு, [அத்தியாயம் 04: நடைமுறை மாதிரிகள்](../README.md) காண்க

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**மறுப்பு**:
இந்த ஆவணம் AI மொழிபெயர்ப்பு சேவை [Co-op Translator](https://github.com/Azure/co-op-translator) பயன்படுத்தி மொழிபெயர்க்கப்பட்டுள்ளது. நாங்கள் துல்லியத்திற்காக முயற்சி செய்துள்ளோம், ஆனால் தானாக செய்யப்படும் மொழிபெயர்ப்புகளில் பிழைகள் அல்லது தவறுகள் இருக்கலாம் என்பதை கவனத்தில் கொள்ளவும். அசல் ஆவணம் அதன் தாய்மொழியில் அதிகாரப்பூர்வ ஆதாரமாக கருதப்பட வேண்டும். முக்கியமான தகவல்களுக்கு, தொழில்நுட்பமான மனித மொழிபெயர்ப்பு பரிந்துரைக்கப்படுகிறது. இந்த மொழிபெயர்ப்பைப் பயன்படுத்துவதால் ஏற்படும் எந்த தவறான புரிதல்கள் அல்லது தவறான விளக்கத்திற்கும் நாங்கள் பொறுப்பில்வில்லை.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->