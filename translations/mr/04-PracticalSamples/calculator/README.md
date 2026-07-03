# नवीनांसाठी MCP कॅल्क्युलेटर ट्युटोरियल

## विषय सूची

- [तुम्हाला काय शिकायला मिळेल](#तुम्हाला-काय-शिकायला-मिळेल)
- [पूर्वअटी](#पूर्वअटी)
- [प्रोजेक्ट रचना समजून घेणे](#प्रोजेक्ट-रचना-समजून-घेणे)
- [मुख्य घटकांचे स्पष्टीकरण](#मुख्य-घटकांचे-स्पष्टीकरण)
  - [1. मुख्य अनुप्रयोग](#1-मुख्य-अनुप्रयोग)
  - [2. कॅल्क्युलेटर सेवा](#2-कॅल्क्युलेटर-सेवा)
  - [3. थेट MCP क्लायंट](#3-थेट-mcp-क्लायंट)
  - [4. AI-चालित क्लायंट](#4-ai-चालित-क्लायंट)
- [उदाहरण चालविणे](#उदाहरण-चालविणे)
- [हे सगळी कशी एकत्र काम करतात](#हे-सगळी-कशी-एकत्र-काम-करतात)
- [पुढील पायऱ्या](#पुढील-पायऱ्या)

## तुम्हाला काय शिकायला मिळेल

हा ट्युटोरियल Model Context Protocol (MCP) वापरून कॅल्क्युलेटर सेवा तयार करण्याची प्रक्रिया समजावतो. तुम्हाला याचा अर्थ समजेल:

- AI एका साधनाप्रमाणे वापरू शकणारी सेवा कशी तयार करायची
- MCP सेवांशी थेट संवाद कसा सेट करायचा
- AI मॉडेल्स स्वयंचलितपणे कोणते साधन वापरायचे ते कसे निवडतात
- थेट प्रोटोकॉल कॉल्स आणि AI-सहाय्यक संवाद यामधील फरक

## पूर्वअटी

सुरू करण्यापूर्वी, खात्री करा की तुमच्याकडे:

- Java 21 किंवा त्यापुढील आवृत्ती स्थापित
- Maven dependency management साठी
- Azure AI Foundry मॉडेल डिप्लॉयमेंट (ते `azd up` वापरून तयार करा — पाहा [Chapter 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), `az login` ने साइन इन केलेले (keyless auth)
- Java आणि Spring Boot ची मूलभूत समज

## प्रोजेक्ट रचना समजून घेणे

कॅल्क्युलेटर प्रोजेक्टमध्ये काही महत्त्वाची फाइल्स आहेत:

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

## मुख्य घटकांचे स्पष्टीकरण

### 1. मुख्य अनुप्रयोग

**फाइल:** `McpServerApplication.java`

हे आपल्या कॅल्क्युलेटर सेवेसाठी एंट्री पॉइंट आहे. हे एक सामान्य Spring Boot अनुप्रयोग आहे ज्यात एक विशेष भर आहे:

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

**हे काय करते:**
- 8080 पोर्टवर Spring Boot वेब सर्व्हर सुरु करते
- `ToolCallbackProvider` तयार करते ज्यामुळे आमची कॅल्क्युलेटर फंक्शन्स MCP साधनांप्रमाणे वापरता येतात
- `@Bean` ही annotation सांगते की Spring हे कॉम्पोनंट म्हणून व्यवस्थापित करेल ज्याचा इतर भाग वापर करू शकतात

### 2. कॅल्क्युलेटर सेवा

**फाइल:** `CalculatorService.java`

येथे सर्व गणित होते. प्रत्येक मेथड `@Tool` ने चिन्हांकित आहे ज्यामुळे ते MCP द्वारे उपलब्ध होतात:

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
    
    // अधिक कॅल्क्युलेटर ऑपरेशन्स...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**महत्वाचे वैशिष्ट्ये:**

1. **`@Tool` Annotation**: MCP ला कळवते की हा मेथड बाह्य क्लायंटमधून कॉल करता येतो
2. **स्पष्ट वर्णने**: प्रत्येक साधनाचे स्पष्टीकरण आहे जे AI मॉडेल्सना वापर कधी करायचा हे समजण्यास मदत करते
3. **सुसंगत परतावा स्वरूप**: सर्व ऑपरेशन्स मानवी वाचनीय स्ट्रिंगस् परत करतात जसे "5.00 + 3.00 = 8.00"
4. **त्रुटी हाताळणी**: शून्यावर भागाकार आणि नकारात्मक वर्गमुळ या संदर्भातील त्रुटी संदेश परत करतो

**उपलब्ध ऑपरेशन्स:**
- `add(a, b)` - दोन संख्या जमवते
- `subtract(a, b)` - दुसरी संख्या पहिल्या मधून वजाबाकी करते
- `multiply(a, b)` - दोन संख्या गुणाकार करते
- `divide(a, b)` - पहिल्या संख्येचे दुसऱ्याने भाग करते (शून्य तपासणीसह)
- `power(base, exponent)` - बेसला एक्स्पोनंट ची शक्ती देते
- `squareRoot(number)` - वर्गमुळ काढते (नकारात्मक तपासणीसह)
- `modulus(a, b)` - भागाकाराचा शिल्लक भाग परत करतो
- `absolute(number)` - संख्येची पूर्णमान (absolute value) परत करते
- `help()` - सर्व ऑपरेशन्स बद्दल माहिती परत करते

### 3. थेट MCP क्लायंट

**फाइल:** `SDKClient.java`

हा क्लायंट AI न वापरता थेट MCP सर्व्हरसोबत बोलतो. तो विशिष्ट कॅल्क्युलेटर फंक्शन्स मॅन्युअली कॉल करतो:

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
        
        // उपलब्ध साधने यादी करा
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // विशिष्ट कॅल्क्युलेटर फंक्शन्स कॉल करा
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

**हे काय करते:**
1. बिल्डर पॅटर्न वापरून `http://localhost:8080` या कॅल्क्युलेटर सर्व्हरशी कनेक्ट होते
2. सर्व उपलब्ध साधनांची (आमचे कॅल्क्युलेटर फंक्शन्स) यादी करतो
3. नेमक्या पॅरामीटर्ससह विशिष्ट फंक्शन्स कॉल करतो
4. निकाल थेट प्रिंट करतो

**टीप:** हे उदाहरण Spring AI 1.1.0-SNAPSHOT डिपेंडेंसी वापरते, ज्यात `WebFluxSseClientTransport` साठी बिल्डर पॅटर्न जोडले गेले आहे. जर तुम्ही जुनी स्थिर आवृत्ती वापरत असाल, तर तुम्हाला थेट कन्स्ट्रक्टर वापरावा लागेल.

**केव्हा वापरावे:** जेव्हा तुम्हाला नेमके कोणते गणित करायचे आहे माहित असते आणि ते प्रोग्राममधून कॉल करायचे असते.

### 4. AI-चालित क्लायंट

**फाइल:** `LangChain4jClient.java`

हा क्लायंट GPT-4o-mini AI मॉडेल वापरतो जे स्वयंचलितपणे कोणते कॅल्क्युलेटर साधन वापरायचे ते ठरवते:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // AI मॉडेल सेट अप करा (Azure AI Foundry, Microsoft Entra ID द्वारे कीलेस प्रमाणीकरण)
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

        // आमच्या कॅल्क्युलेटर MCP सर्व्हरसह कनेक्ट करा
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // दाखवते AI काय करत आहे
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // AI ला आमच्या कॅल्क्युलेटर साधनांमध्ये प्रवेश द्या
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // असा AI बॉट तयार करा जो आमचा कॅल्क्युलेटर वापरू शकेल
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // आता आपण AI ला नैसर्गिक भाषेत गणिते करण्यास सांगू शकतो
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**हे काय करते:**
1. GitHub टोकन वापरून AI मॉडेल कनेक्शन तयार करते
2. AI ला आमच्या कॅल्क्युलेटर MCP सर्व्हरशी जोडते
3. AI ला आमच्या सर्व कॅल्क्युलेटर साधनांचा प्रवेश देते
4. "24.5 आणि 17.3 ची बेरीज करा" सारख्या नैसर्गिक भाषा विनंत्या स्वीकारते

**AI आपोआप:**
- समजतो की तुम्हाला संख्या जोडायच्या आहेत
- `add` साधन निवडतो
- `add(24.5, 17.3)` कॉल करतो
- नैसर्गिक प्रतिसादात निकाल परत करतो

## उदाहरण चालविणे

### पाऊल 1: कॅल्क्युलेटर सर्व्हर सुरु करा

प्रथम, साइन इन करा आणि तुमचा Azure AI Foundry endpoint सेट करा (AI क्लायंटसाठी आवश्यक — keyless auth, API की नाही):

**विंडोज:**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**Linux/macOS:**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

सर्व्हर सुरु करा:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

सर्व्हर `http://localhost:8080` वर सुरु होईल. तुम्हाला हे दिसेल:
```
Started McpServerApplication in X.XXX seconds
```

### पाऊल 2: थेट क्लायंटसह चाचणी करा

**नव्या** टर्मिनलमध्ये (सर्व्हर चालू आहे त्या स्थितीत) थेट MCP क्लायंट चालवा:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

तुम्हाला अशी आउटपुट दिसेल:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### पाऊल 3: AI क्लायंटसह चाचणी करा

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

AI आपोआप साधने वापरताना दिसेल:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### पाऊल 4: MCP सर्व्हर बंद करा

चाचणी पूर्ण झाल्यावर AI क्लायंटच्या टर्मिनलमध्ये `Ctrl+C` दाबून थांबवा. MCP सर्व्हर चालू राहील जोपर्यंत तुम्ही ते स्वतः थांबवत नाही.
सर्व्हर थांबवण्यासाठी चालू असलेल्या टर्मिनलमध्ये `Ctrl+C` दाबा.

## हे सगळी कशी एकत्र काम करतात

जेव्हा तुम्ही AI ला विचारता "5 + 3 काय आहे?" तर संपूर्ण प्रक्रिया अशी आहे:

1. **तुम्ही** नैसर्गिक भाषेत AI ला प्रश्न विचारता
2. **AI** तुमचा विनंती विश्लेषित करते आणि समजते की तुम्हाला बेरीज करायची आहे
3. **AI** MCP सर्व्हरला कॉल करते: `add(5.0, 3.0)`
4. **कॅल्क्युलेटर सेवा** करतो: `5.0 + 3.0 = 8.0`
5. **कॅल्क्युलेटर सेवा** परत करते: `"5.00 + 3.00 = 8.00"`
6. **AI** निकाल घेतो आणि नैसर्गिक प्रतिसाद म्हणून फॉरमॅट करतो
7. **तुम्हाला** मिळते: "5 आणि 3 ची बेरीज 8 आहे"

## पुढील पायऱ्या

अधिक उदाहरणांसाठी, पाहा [Chapter 04: Practical samples](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**अस्वीकरण**:
हा दस्तऐवज AI भाषांतर सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) चा वापर करून अनुवादित केला आहे. जरी आम्ही अचूकतेसाठी प्रयत्न करतो, तरी कृपया लक्षात घ्या की स्वयंचलित भाषांतरांमध्ये त्रुटी किंवा अचूकतेची कमतरता असू शकते. मूळ दस्तऐवज त्याच्या मूळ भाषेत अधिकृत स्रोत मानला पाहिजे. महत्त्वाची माहिती असल्यास, व्यावसायिक मानवी भाषांतराची शिफारस केली जाते. या भाषांतराच्या वापरामुळे उद्भवणाऱ्या कोणत्याही गैरसमज किंवा चुकीच्या अर्थलावणीसाठी आम्ही जबाबदार नाही.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->