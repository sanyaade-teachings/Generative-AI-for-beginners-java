# MCP कॅल्क्युलेटर ट्युटोरियल शुरुआतीसाठी

## अनुक्रमणिका

- [तुम्ही काय शिकाल](#तुम्ही-काय-शिकाल)
- [पूर्वअट](#पूर्वअट)
- [प्रकल्प रचना समजून घेणे](#प्रकल्प-रचना-समजून-घेणे)
- [मुख्य घटक स्पष्ट केले](#मुख्य-घटक-स्पष्ट-केले)
  - [1. मुख्य अनुप्रयोग](#1-मुख्य-अनुप्रयोग)
  - [2. कॅल्क्युलेटर सेवा](#2-कॅल्क्युलेटर-सेवा)
  - [3. डायरेक्ट MCP क्लायंट](#3-डायरेक्ट-mcp-क्लायंट)
  - [4. AI-शक्तीप्राप्त क्लायंट](#4-ai-शक्तीप्राप्त-क्लायंट)
- [उदाहरणे चालवा](#उदाहरणे-चालवा)
- [हे कसे सर्व एकत्र कार्य करते](#हे-कसे-सर्व-एकत्र-कार्य-करते)
- [पुढील पावले](#पुढील-पावले)

## तुम्ही काय शिकाल

हा ट्युटोरियल Model Context Protocol (MCP) वापरून कॅल्क्युलेटर सेवा कशी तयार करायची हे स्पष्ट करतो. तुम्हाला समजेल:

- AI वापरासाठी साधन म्हणून सेवा कशी तयार करायची
- MCP सेवांसोबत थेट संवाद कसा सेट करायचा
- AI मॉडेल्स कोणती साधने वापरायची हे आपोआप कसे निवडतात
- थेट प्रोटोकॉल कॉल आणि AI-योग्य संवाद यामधला फरक काय आहे

## पूर्वअट

सुरू करण्यापूर्वी, खात्री करा की तुमच्याकडे आहे:
- Java 21 किंवा त्याहून अधिक स्थापित केलेले
- Maven निर्भरता व्यवस्थापनासाठी
- Azure AI Foundry मॉडेल डिप्लॉयमेंट (त्यासाठी `azd up` वापरा — पहा [Chapter 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), `az login` ने साइन इन केलेले (कीलेस प्रमाणीकरण)
- Java आणि Spring Boot ची प्राथमिक समज

## प्रकल्प रचना समजून घेणे

कॅल्क्युलेटर प्रकल्पात काही महत्त्वाची फाइल्स आहेत:

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

## मुख्य घटक स्पष्ट केले

### 1. मुख्य अनुप्रयोग

**फाइल:** `McpServerApplication.java`

हे आपल्या कॅल्क्युलेटर सेवेचे प्रवेश बिंदू आहे. हे एक सामान्य Spring Boot अनुप्रयोग आहे ज्यात एक विशेष भर आहे:

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

**हे काय करतो:**
- पोर्ट 8080 वर Spring Boot वेब सर्व्हर सुरू करतो
- `ToolCallbackProvider` तयार करतो जे आपले कॅल्क्युलेटर पद्धती MCP साधनांप्रमाणे उपलब्ध करतो
- `@Bean` संज्ञा Spring ला हे घटक म्हणून व्यवस्थापित करण्यास सांगते जे अन्य भाग वापरू शकतात

### 2. कॅल्क्युलेटर सेवा

**फाइल:** `CalculatorService.java`

इथे सर्व गणित होते. प्रत्येक पद्धत `@Tool` ने चिन्हांकित आहे जी MCP द्वारे उपलब्ध होते:

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
    
    // अधिक कॅलक्युलेटर ऑपरेशन्स...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**महत्त्वाचे मुद्दे:**

1. **`@Tool` अ‍ॅनोटेशन**: MCP ला सांगते की ही पद्धत बाह्य क्लायंटने कॉल केली जाऊ शकते
2. **स्पष्ट वर्णने**: प्रत्येक साधनाचे वर्णन आहे जे AI मॉडेल्सना समजावते की ते कधी वापरायचे
3. **सुसंगत परिणाम स्वरूप**: सर्व ऑपरेशन्स मानवी-सुलभ स्ट्रिंग्स परत करतात जसे "5.00 + 3.00 = 8.00"
4. **त्रुटी हाताळणी**: शून्याने भागाकार आणि नकारात्मक वर्गमुळ त्रुटी संदेश परत करतात

**उपलब्ध ऑपरेशन्स:**
- `add(a, b)` - दोन संख्या जमवते
- `subtract(a, b)` - दुसरी संख्या पहिल्यापासून वजा करते
- `multiply(a, b)` - दोन संख्या गुणाकार करते
- `divide(a, b)` - पहिल्याला दुसऱ्याने भाग देतो (शून्य तपासणीसह)
- `power(base, exponent)` - बेसला एक्स्पोनंटच्या शक्तीत उन्चावते
- `squareRoot(number)` - वर्गमुळ काढतो (नकारात्मक तपासणीसह)
- `modulus(a, b)` - भागाकाराचा उराचा भाग परत करतो
- `absolute(number)` - मूल्याचा अभाज्य भाग परत करतो
- `help()` - सर्व ऑपरेशन्सची माहिती देते

### 3. डायरेक्ट MCP क्लायंट

**फाइल:** `SDKClient.java`

हा क्लायंट AI वापरल्याशिवाय थेट MCP सर्व्हरशी बोलतो. तो विशिष्ट कॅल्क्युलेटर फंक्शन्स मॅन्युअली कॉल करतो:

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
        
        // उपलब्ध साधने सूचीबद्ध करा
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
1. बिल्डर पॅटर्न वापरून `http://localhost:8080` वर कॅल्क्युलेटर सर्व्हरशी कनेक्ट होतो
2. सर्व उपलब्ध साधने यादी करतो (आपल्या कॅल्क्युलेटर फंक्शन्स)
3. अचूक पॅरामीटर्ससह विशिष्ट फंक्शन्स कॉल करतो
4. परिणाम थेट प्रिंट करतो

**सूचना:** या उदाहरणात Spring AI 1.1.0-SNAPSHOT निर्भरता वापरलेली आहे, ज्यात `WebFluxSseClientTransport` साठी बिल्डर पॅटर्न आणले गेले आहे. जुनी स्थिर आवृत्ती वापरत असल्यास थेट कंस्ट्रक्टर वापरणे आवश्यक असू शकते.

**केव्हा वापरायचे:** जेव्हा तुम्हाला नक्की कोणते गणित करायचे आहे हे माहित असेल आणि प्रोग्रामॅटिकली कॉल करायचे असेल.

### 4. AI-शक्तीप्राप्त क्लायंट

**फाइल:** `LangChain4jClient.java`

हा क्लायंट GPT-4o-mini AI मॉडेल वापरतो जे आपोआप कोणती कॅल्क्युलेटर साधने वापरायची ते ठरवू शकते:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // एआय मॉडेल सेट करा (Azure AI Foundry, Microsoft Entra ID द्वारे कीलेस प्रमाणीकरण)
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

        // आपल्या कॅल्क्युलेटर MCP सर्व्हरशी कनेक्ट करा
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // एआय काय करत आहे ते दर्शविते
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // एआयला आपल्या कॅल्क्युलेटर साधनांची प्रवेश द्या
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // एआय बोट तयार करा जो आपल्या कॅल्क्युलेटरचा वापर करू शकेल
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // आता आपण एआयला नैसर्गिक भाषेत गणना करण्यास सांगू शकतो
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**हे काय करते:**
1. कीलेस प्रमाणीकरण (Microsoft Entra ID) वापरून AI मॉडेल कनेक्शन तयार करतो
2. AI ला आमच्या कॅल्क्युलेटर MCP सर्व्हरशी जोडतो
3. AI ला सर्व कॅल्क्युलेटर साधनांवर प्रवेश देतो
4. "24.5 आणि 17.3 चा बेरीज करा" सारख्या नैसर्गिक भाषा विनंत्या मानतो

**AI आपोआप:**
- तुम्हाला संख्या जमा करायचे आहे हे समजतो
- `add` साधन निवडतो
- `add(24.5, 17.3)` कॉल करतो
- परिणाम नैसर्गिक प्रतिसादात परत करतो

## उदाहरणे चालवा

### पाऊल 1: कॅल्क्युलेटर सर्व्हर सुरू करा

सर्वप्रथम, साइन इन करा व तुमचा Azure AI Foundry endpoint सेट करा (AI क्लायंटसाठी आवश्यक — कीलेस प्रमाणीकरण, API की नाही):

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

सर्व्हर सुरू करा:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

सर्व्हर `http://localhost:8080` वर सुरू होईल. तुम्हाला दिसेल:
```
Started McpServerApplication in X.XXX seconds
```

### पाऊल 2: डायरेक्ट क्लायंटसह चाचणी करा

एखाद्या **नवीन** टर्मिनलमध्ये, सर्व्हर चालू असताना डायरेक्ट MCP क्लायंट चालवा:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

तुम्हाला असे आउटपुट दिसेल:
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

चाचणी पूर्ण झाल्यावर, AI क्लायंट टर्मिनलमध्ये `Ctrl+C` दाबून थांबवा. MCP सर्व्हर तुम्ही थांबवल्याशिवाय चालू राहील.
सर्व्हर बंद करण्यासाठी, तो चालू असलेल्या टर्मिनलमध्ये `Ctrl+C` दाबा.

## हे कसे सर्व एकत्र कार्य करते

तुम्ही AI ला "5 + 3 काय आहे?" असे विचारल्यावर पूर्ण प्रक्रिया अशी असते:

1. **तुम्ही** AI ला नैसर्गिक भाषेत प्रश्न विचारता
2. **AI** तुमची विनंती विश्लेषण करतो आणि तुम्हाला बेरीज करायची आहे हे ओळखतो
3. **AI** MCP सर्व्हरला कॉल करतो: `add(5.0, 3.0)`
4. **कॅल्क्युलेटर सेवा** करते: `5.0 + 3.0 = 8.0`
5. **कॅल्क्युलेटर सेवा** परत करते: `"5.00 + 3.00 = 8.00"`
6. **AI** परिणाम प्राप्त करतो आणि नैसर्गिक प्रतिसाद तयार करतो
7. **तुम्हाला** मिळते: "5 आणि 3 ची बेरीज 8 आहे"

## पुढील पावले

अधिक उदाहरणांसाठी, पहा [Chapter 04: Practical samples](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**अस्वीकरण**:
हा दस्तऐवज AI भाषांतर सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) चा वापर करून अनुवादित केला आहे. जरी आम्ही अचूकतेसाठी प्रयत्न करतो, तरी कृपया लक्षात घ्या की स्वयंचलित भाषांतरांमध्ये त्रुटी किंवा अचूकतेची कमतरता असू शकते. मूळ दस्तऐवज त्याच्या मूळ भाषेत अधिकृत स्रोत मानला पाहिजे. महत्त्वाची माहिती असल्यास, व्यावसायिक मानवी भाषांतराची शिफारस केली जाते. या भाषांतराच्या वापरामुळे उद्भवणाऱ्या कोणत्याही गैरसमज किंवा चुकीच्या अर्थलावणीसाठी आम्ही जबाबदार नाही.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->