# MCP கால்குலேட்டர் தொடக்கக் கையேடு

## உள்ளடக்கப்பட்டியல்

- [நீங்கள் என்ன கற்றுக்கொள்வீர்கள்](#நீங்கள்-என்ன-கற்றுக்கொள்வீர்கள்)
- [முன்னோட்டங்கள்](#முன்னோட்டங்கள்)
- [திட்ட அமைப்பை புரிந்துகொள்ளுதல்](#திட்ட-அமைப்பை-புரிந்துகொள்ளுதல்)
- [முக்கிய கூறுகள் விளக்கம்](#முக்கிய-கூறுகள்-விளக்கம்)
  - [1. பிரதான பயன்பாடு](#1-பிரதான-பயன்பாடு)
  - [2. கால்குலேட்டர் சேவை](#2-கால்குலேட்டர்-சேவை)
  - [3. நேரடி MCP கிளையன்ட்](#3-நேரடி-mcp-கிளையன்ட்)
  - [4. AI இயக்கும் கிளையன்ட்](#4-ai-இயக்கும்-கிளையன்ட்)
- [உதாரணங்களை இயக்குதல்](#உதாரணங்களை-இயக்குதல்)
- [எல்லாம் எப்படி சேர்ந்து செயல்படுகிறது](#எல்லாம்-எப்படி-சேர்ந்து-செயல்படுகிறது)
- [அடுத்த படிகள்](#அடுத்த-படிகள்)

## நீங்கள் என்ன கற்றுக்கொள்வீர்கள்

இந்த கையேடு Model Context Protocol (MCP) பயன்படுத்தி ஒரு கால்குலேட்டர் சேவையை உருவாக்குவது எப்படி என்பதைக் விளக்குகிறது. நீங்கள் புரிந்து கொள்வீர்கள்:

- AI கருவியாக பயன்படுத்தக்கூடிய சேவையை உருவாக்குவது எப்படி
- MCP சேவைகளுடன் நேரடி தொடர்பை அமைப்பது எப்படி
- AI மாடல்கள் தானாக எந்த கருவிகளை பயன்படுத்த வேண்டும் என்பதைத் தேர்ந்தெடுப்பது எப்படி
- நேரடி நெறிமுறை அழைப்புகளுடன் AI-ஆல் உதவியுள்ள தொடர்புகளுக்கான வித்தியாசம்

## முன்னோட்டங்கள்

தொடங்குவதற்கு முன்பு, நீங்கள் எடுத்துக் கொள்ளவேண்டியது:

- Java 21 அல்லது அதற்கால் மேல் பதிப்பு நிறுவப்பட்டிருத்தல்
- சார்பு மேலாண்மைக்காக Maven
- Azure AI Foundry மாடல் இயக்கம் (பயன்பாடு `azd up` மூலம் ஏற்படுத்தவும் — [அத்தியாயம் 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md) காண்க)
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), `az login` இல் உள்நுழைந்திருத்தல் (keyless அங்கீகாரம்)
- Java மற்றும் Spring Boot பற்றிய அடிப்படைக் புரிதல்

## திட்ட அமைப்பை புரிந்துகொள்ளுதல்

கால்குலேட்டர் திட்டத்தில் அத்தியாவசியமான சில கோப்புகள் உள்ளன:

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

### 1. பிரதான பயன்பாடு

**கோப்பு:** `McpServerApplication.java`

இது நமது கால்குலேட்டர் சேவையின் நுழைவு புள்ளி. இது ஒரு குறைந்த பட்ச Spring Boot பயன்பாடு, அதில் ஒரு சிறப்பு அம்சம் சேர்க்கப்பட்டுள்ளது:

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
- 8080 போர்டில் ஒரு Spring Boot வலை சேவையகத்தை துவக்கிறது
- நமது கால்குலேட்டர் முறைகளை MCP கருவியாகக் கிடைக்க செய்யும் `ToolCallbackProvider` ஐ உருவாக்குகிறது
- `@Bean` குறிப்பு Spring-ஐ இந்த அங்கமாக நிர்வகிக்கவும் மற்ற பகுதிகள் பயன்படுத்தவும் சொல்கிறது

### 2. கால்குலேட்டர் சேவை

**கோப்பு:** `CalculatorService.java`

இங்கே அனைத்து கணக்குகள் நிகழ்கின்றன. ஒவ்வொரு முறையும் MCP வழியாகக் கொண்டு வர அனுமதிக்கும் `@Tool` மூலம் குறிக்கப்படுகிறது:

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
    
    // மேலும் கனக்க இயந்திர செயல்பாடுகள்...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```


**முக்கிய அம்சங்கள்:**

1. **`@Tool` குறிப்பு**: இந்த முறை வெளி கிளையண்டுகளால் அழைக்கப்படலாம் என்பதைக் காட்டுகிறது
2. **தெளிவான விளக்கங்கள்**: ஒவ்வொரு கருவிக்கும் AI மாடல்கள் எப்போது பயன்படுத்த வேண்டும் என்பதை புரிந்து கொள்ள உதவும் விளக்கம் உள்ளது
3. **ஒத்த நிறைவு வடிவமைப்பு**: அனைத்து செயல்பாடுகளும் மனிதர் புரிந்துகொள்ளக்கூடிய "5.00 + 3.00 = 8.00" போன்ற தொடர்களை திருப்புகின்றன
4. **தவறு கையாளுதல்**: பூஜ்யத்தால் பகுக்குதல் மற்றும் பர்மாணவத் தோற்றத்தை தவிர்க்கும் செய்திகளை திருப்புகிறது

**கிடைக்கக்கூடிய செயல்பாடுகள்:**
- `add(a, b)` - இரண்டு எண்களை கூட்டுகிறது
- `subtract(a, b)` - இரண்டாவது எண்ணை முதலில் இருந்து கழிக்கிறது
- `multiply(a, b)` - இரண்டு எண்களை பெருக்குகிறது
- `divide(a, b)` - முதலில் எண்ணை இரண்டாவது என்றால் பாகுபடுத்துகிறது (பூஜ்யம் பரிசோதனை உண்டு)
- `power(base, exponent)` - அடிப்படை எண்ணினை சக்தி எண்ணாக உயர்த்துகிறது
- `squareRoot(number)` - வரிசை வேரை கணக்கிடுகிறது (கேட்டுள்ளமை இருப்பின் தவறு அளிக்கிறது)
- `modulus(a, b)` - பகுப்பின் மீதியை திருப்புகிறது
- `absolute(number)` - நிறைய மதிப்பை திருப்புகிறது
- `help()` - அனைத்து செயல்களும் பற்றிய தகவல்களை திருப்புகிறது

### 3. நேரடி MCP கிளையன்ட்

**கோப்பு:** `SDKClient.java`

இந்த கிளையன்ட் AI பயன்படுத்தாமல் நேரடியாக MCP சேவையகத்துடன் பேசுகிறது. குறிப்பிட்ட கால்குலேட்டர் செயல்பாடுகளை கையால் அழைக்கிறது:

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
        
        // கிடைக்கும் கருவிகள் பட்டியல்
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


**இது என்ன செய்கிறது:**
1. கட்டுமான வீரியத்தைப் பயன்படுத்தி `http://localhost:8080` என்ற முகவரியில் சேவையகத்தை இணைக்கிறது
2. அனைத்து கிடைக்கும் கருவிகளை பட்டியலிடுகிறது (நமது கால்குலேட்டர் செயல்பாடுகள்)
3. குறிப்பிட்ட செயல்களை சரியான அளவுருக்களுடன் அழைக்கிறது
4. முடிவுகளை நேரடியாக அச்சிடுகிறது

**குறிப்பு:** இந்த உதாரணம் Spring AI 1.1.0-SNAPSHOT சார்ப்பை பயன்படுத்துகிறது, இது `WebFluxSseClientTransport` க்கான கட்டுமான வீரியத்தை அறிமுகப்படுத்தியது. பழைய நிலைத்த பதிப்புகளைப் பயன்படுத்தினால் நேரடி கட்டுமான கருவியைப் பயன்படுத்த வேண்டியிருக்கலாம்.

**எப்போது பயன்படுத்த வேண்டும்:** நீங்கள் செய்ய வேண்டிய கணக்கை தெளிவாக அறிந்திருக்கும் போது அதை நிரலாக்க முறையில் அழைக்க.

### 4. AI இயக்கும் கிளையன்ட்

**கோப்பு:** `LangChain4jClient.java`

இந்த கிளையன்ட் GPT-4o-mini என்ற AI மாடலைப் பயன்படுத்தி தானாகவே எந்த கால்குலேட்டர் கருவிகளைப் பயன்படுத்த வேண்டும் என்று முடிவு செய்கிறது:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // AI மodel ஐ அமைக்கவும் (Azure AI Foundry, Microsoft Entra ID வழியாக விசை இல்லாத அங்கீகாரம்)
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

        // எங்கள் கணக்கிடும் MCP சேவையகத்துடன் இணையுங்கள்
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // AI என்ன செய்கிறதோ அதைக் காட்டுகிறது
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // எங்கள் கணக்கிடும் கருவிகளுக்கு AI அணுகலை வழங்கவும்
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // எங்கள் கணக்கிடும் கருவிகளைப் பயன்படுத்தக்கூடிய AI பாட்டை உருவாக்கவும்
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // இப்போது நாம AIக்கு இயல்பான மொழியில் கணக்கீடுகளை செய்யக் கேட்கலாம்
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```


**இது என்ன செய்கிறது:**
1. கீ இல்லாத அங்கீகாரத்தைப் பயன்படுத்தி (Microsoft Entra ID) AI மாடல் தொடர்பு உருவாக்குகிறது
2. AI-ஐ நமது கால்குலேட்டர் MCP சேவையகத்துடன் இணைக்கிறது
3. AI-க்கு எங்கள் அனைத்து கால்குலேட்டர் கருவிகளையும் அணுகலில் வழங்குகிறது
4. "24.5 மற்றும் 17.3-இன் கூட்டை கணக்கிடு" போன்ற இயல்பான மொழி கோரிக்கைகளை அனுமதிக்கிறது

**AI தானாகவே:**
- நீங்கள் எண்களை கூட்ட விரும்புவதாக புரிகிறது
- `add` கருவியை தேர்ந்தெடுக்கிறது
- `add(24.5, 17.3)` ஐ அழைக்கிறது
- முடிவை இயல்பான பதிலில் அளிக்கிறது

## உதாரணங்களை இயக்குதல்

### படி 1: கால்குலேட்டர் சேவையகத்தை துவக்கவும்

முதலில் உள்நுழைந்து உங்கள் Azure AI Foundry வழிப்புள்ளியை அமைக்கவும் (AI கிளையன்டுக்கான தேவையானது — keyless அங்கீகாரம், API விசை தேவையில்லை):

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


சேவையகத்தை துவக்கவும்:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```


சேவையகம் `http://localhost:8080` இல் துவங்கும். அதைப் பார்க்கலாம்:
```
Started McpServerApplication in X.XXX seconds
```


### படி 2: நேரடி கிளையன்ட் மூலம் சோதனை செய்க

சேவையகம் இயங்கிக் கொண்டிருக்கும் நிலையில் **புதிய** டெர்மினலில் நேரடி MCP கிளையன்டை இயக்கவும்:
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


### படி 3: AI கிளையன்ட் மூலம் சோதனை செய்க

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```


AI தானாக கருவிகளை பயன்படுத்தியதைக் காணலாம்:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```


### படி 4: MCP சேவையகத்தை நிறுத்துக

சோதனை முடிந்ததும், AI கிளையன்டை அதன் டெர்மினலில் `Ctrl+C` அழுத்தி நிறுத்தவும். MCP சேவையகம் இயங்கிவிருக்கும் நிலையிலும் நின்றிருக்கும்.
சேவையகத்தை நிறுத்த, அது இயங்கும் டெர்மினலில் `Ctrl+C` ஐ அழுத்தவும்.

## எல்லாம் எப்படி சேர்ந்து செயல்படுகிறது

நீங்கள் AIக்கு "5 + 3 என்ன?" என்று கேட்டபோது நிகழும் முழு சிலை:

1. **நீங்கள்** இயல்பான மொழியில் AI-யிடம் கேட்கிறீர்கள்
2. **AI** உங்கள் கோரிக்கையை பரிசீலித்து, நீங்கள் கூட்டல் விரும்புகிறீர்கள் என்பதை உணர்கிறது
3. **AI** MCP சேவையகத்தை அழைக்கிறது: `add(5.0, 3.0)`
4. **கால்குலேட்டர் சேவை** செய்கிறது: `5.0 + 3.0 = 8.0`
5. **கால்குலேட்டர் சேவை** திருப்புகிறது: `"5.00 + 3.00 = 8.00"`
6. **AI** முடிவைப் பெறுகிறது மற்றும் இயல்பான பதிலை வடிவமைக்கிறது
7. **நீங்கள்** பெறுகிறீர்கள்: "5 மற்றும் 3-ன் தொகை 8 ஆகும்"

## அடுத்த படிகள்

மேலும் உதாரணங்களுக்கு, [அத்தியாயம் 04: நடைமுறை மாதிரிகள்](../README.md) பார்வையிடவும்

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**மறுப்பு**:
இந்த ஆவணம் AI மொழிபெயர்ப்பு சேவை [Co-op Translator](https://github.com/Azure/co-op-translator) பயன்படுத்தி மொழிபெயர்க்கப்பட்டுள்ளது. நாங்கள் துல்லியத்திற்காக முயற்சி செய்துள்ளோம், ஆனால் தானாக செய்யப்படும் மொழிபெயர்ப்புகளில் பிழைகள் அல்லது தவறுகள் இருக்கலாம் என்பதை கவனத்தில் கொள்ளவும். அசல் ஆவணம் அதன் தாய்மொழியில் அதிகாரப்பூர்வ ஆதாரமாக கருதப்பட வேண்டும். முக்கியமான தகவல்களுக்கு, தொழில்நுட்பமான மனித மொழிபெயர்ப்பு பரிந்துரைக்கப்படுகிறது. இந்த மொழிபெயர்ப்பைப் பயன்படுத்துவதால் ஏற்படும் எந்த தவறான புரிதல்கள் அல்லது தவறான விளக்கத்திற்கும் நாங்கள் பொறுப்பில்வில்லை.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->