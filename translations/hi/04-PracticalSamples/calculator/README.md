# शुरुआती लोगों के लिए MCP कैलकुलेटर ट्यूटोरियल

## सामग्री तालिका

- [आप क्या सीखेंगे](#आप-क्या-सीखेंगे)
- [पूर्वशर्तें](#पूर्वशर्तें)
- [परियोजना संरचना को समझना](#परियोजना-संरचना-को-समझना)
- [मुख्य घटकों की व्याख्या](#मुख्य-घटकों-की-व्याख्या)
  - [1. मुख्य एप्लीकेशन](#1-मुख्य-एप्लीकेशन)
  - [2. कैलकुलेटर सेवा](#2-कैलकुलेटर-सेवा)
  - [3. सीधे MCP क्लाइंट](#3-सीधे-mcp-क्लाइंट)
  - [4. एआई-संचालित क्लाइंट](#4-एआई-संचालित-क्लाइंट)
- [उदाहरण चलाना](#उदाहरण-चलाना)
- [यह सब कैसे काम करता है](#यह-सब-कैसे-काम-करता-है)
- [अगले कदम](#अगले-कदम)

## आप क्या सीखेंगे

यह ट्यूटोरियल मॉडल कंटेक्स्ट प्रोटोकॉल (MCP) का उपयोग करके एक कैलकुलेटर सेवा कैसे बनाएँ, इसके बारे में बताता है। आप समझेंगे:

- कैसे एक सेवा बनाएं जिसका AI उपकरण के रूप में उपयोग कर सके
- MCP सेवाओं के साथ सीधे संचार कैसे स्थापित करें
- कैसे AI मॉडल स्वचालित रूप से चुन सकते हैं कि कौन से उपकरण उपयोग करने हैं
- सीधे प्रोटोकॉल कॉल्स और AI-सहायता प्राप्त इंटरैक्शन्स के बीच अंतर

## पूर्वशर्तें

शुरू करने से पहले, सुनिश्चित करें कि आपके पास है:
- Java 21 या उससे ऊपर स्थापित
- Maven डिपेंडेंसी प्रबंधन के लिए
- एक Azure AI Foundry मॉडल डिप्लॉयमेंट (इसे `azd up` से तैयार करें — देखें [चैप्टर 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), `az login` के साथ साइन इन (कीलेस प्रमाणीकरण)
- Java और Spring Boot की मूल समझ

## परियोजना संरचना को समझना

कैलकुलेटर प्रोजेक्ट में कई महत्वपूर्ण फाइलें हैं:

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

## मुख्य घटकों की व्याख्या

### 1. मुख्य एप्लीकेशन

**फाइल:** `McpServerApplication.java`

यह हमारी कैलकुलेटर सेवा का प्रवेश बिंदु है। यह एक मानक Spring Boot एप्लीकेशन है जिसमें एक विशेष जोड़ है:

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

**यह क्या करता है:**
- पोर्ट 8080 पर एक Spring Boot वेब सर्वर शुरू करता है
- `ToolCallbackProvider` बनाता है जो हमारे कैलकुलेटर के मेथड्स को MCP उपकरणों के रूप में उपलब्ध कराता है
- `@Bean` एनोटेशन बताता है कि Spring इसे एक कॉम्पोनेंट के रूप में प्रबंधित करेगा जिसे अन्य भाग उपयोग कर सकते हैं

### 2. कैलकुलेटर सेवा

**फाइल:** `CalculatorService.java`

यहां सभी गणितीय कार्य होते हैं। प्रत्येक मेथड को `@Tool` के साथ चिन्हित किया गया है ताकि इसे MCP के माध्यम से उपलब्ध कराया जा सके:

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
    
    // और अधिक कैलकुलेटर ऑपरेशन...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**मुख्य विशेषताएं:**

1. **`@Tool` एनोटेशन**: यह MCP को बताता है कि इस मेथड को बाहरी क्लाइंट्स कॉल कर सकते हैं
2. **स्पष्ट विवरण**: प्रत्येक उपकरण के लिए एक विवरण होता है जो AI मॉडलों को समझने में मदद करता है कि इसे कब उपयोग करना चाहिए
3. **सुसंगत रिटर्न फॉर्मेट**: सभी ऑपरेशन मानव-पठनीय स्ट्रिंग्स जैसे "5.00 + 3.00 = 8.00" लौटाते हैं
4. **त्रुटि प्रबंधन**: शून्य से भाग देना और नकारात्मक वर्गमूल पर त्रुटि संदेश लौटाए जाते हैं

**उपलब्ध ऑपरेशन:**
- `add(a, b)` - दो संख्याओं को जोड़ता है
- `subtract(a, b)` - दूसरी संख्या को पहली से घटाता है
- `multiply(a, b)` - दो संख्याओं का गुणा करता है
- `divide(a, b)` - पहली संख्या को दूसरी से विभाजित करता है (शून्य जाँच के साथ)
- `power(base, exponent)` - बेस को उपाधि की शक्ति में बढ़ाता है
- `squareRoot(number)` - वर्गमूल निकालता है (नकारात्मक जाँच के साथ)
- `modulus(a, b)` - विभाजन के शेष को लौटाता है
- `absolute(number)` - पूर्णांक मान लौटाता है
- `help()` - सभी ऑपरेशन्स के बारे में जानकारी देता है

### 3. सीधे MCP क्लाइंट

**फाइल:** `SDKClient.java`

यह क्लाइंट AI का उपयोग किए बिना सीधे MCP सर्वर से बात करता है। यह मैन्युअल रूप से विशिष्ट कैलकुलेटर फंक्शन्स को कॉल करता है:

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
        
        // उपलब्ध उपकरणों की सूची बनाएँ
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // विशिष्ट कैलकुलेटर फ़ंक्शंस को कॉल करें
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

**यह क्या करता है:**
1. बिल्डर पैटर्न का उपयोग करते हुए `http://localhost:8080` पर कैलकुलेटर सर्वर से कनेक्ट करता है
2. सभी उपलब्ध उपकरणों (हमारे कैलकुलेटर फंक्शन्स) को सूचीबद्ध करता है
3. सटीक पैरामीटर के साथ विशिष्ट फंक्शन्स को कॉल करता है
4. परिणामों को सीधे प्रिंट करता है

**टिप:** यह उदाहरण Spring AI 1.1.0-SNAPSHOT डिपेंडेंसी का उपयोग करता है, जिसने `WebFluxSseClientTransport` के लिए बिल्डर पैटर्न पेश किया है। अगर आप पुराना स्थिर संस्करण उपयोग कर रहे हैं, तो आपको सीधे कंस्ट्रक्टर का उपयोग करना पड़ सकता है।

**इसे कब उपयोग करें:** जब आप जानते हों कि आप कौन सा कैलकुलेशन करना चाहते हैं और इसे प्रोग्रामेटिक तरीके से कॉल करना चाहते हैं।

### 4. एआई-संचालित क्लाइंट

**फाइल:** `LangChain4jClient.java`

यह क्लाइंट AI मॉडल (GPT-4o-mini) का उपयोग करता है जो स्वचालित रूप से चुन सकता है कि कौन से कैलकुलेटर उपकरण उपयोग करने हैं:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // AI मॉडल सेट अप करें (Azure AI Foundry, Microsoft Entra ID के माध्यम से बिना कुंजी के प्रमाणीकरण)
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

        // हमारे कैलकुलेटर MCP सर्वर से कनेक्ट करें
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // दिखाता है कि AI क्या कर रहा है
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // AI को हमारे कैलकुलेटर टूल्स तक पहुंच दें
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // एक AI बोट बनाएं जो हमारे कैलकुलेटर का उपयोग कर सके
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // अब हम AI से प्राकृतिक भाषा में गणनाएं करने को कह सकते हैं
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**यह क्या करता है:**
1. आपके GitHub टोकन का उपयोग करके AI मॉडल कनेक्शन बनाता है
2. AI को हमारे कैलकुलेटर MCP सर्वर से जोड़ता है
3. AI को हमारे सभी कैलकुलेटर उपकरणों तक पहुँच देता है
4. "24.5 और 17.3 का योग निकालो" जैसे प्राकृतिक भाषा अनुरोध स्वीकार करता है

**AI स्वचालित रूप से:**
- समझता है कि आप संख्याओं को जोड़ना चाहते हैं
- `add` उपकरण चुनता है
- `add(24.5, 17.3)` कॉल करता है
- प्राकृतिक प्रतिक्रिया में परिणाम लौटाता है

## उदाहरण चलाना

### चरण 1: कैलकुलेटर सर्वर शुरू करें

सबसे पहले, साइन इन करें और अपना Azure AI Foundry एंडपॉइंट सेट करें (AI क्लाइंट के लिए आवश्यक — कीलेस प्रमाणीकरण, कोई API कुंजी नहीं):

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

सर्वर स्टार्ट करें:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

सर्वर `http://localhost:8080` पर शुरू होगा। आप देखेंगे:
```
Started McpServerApplication in X.XXX seconds
```

### चरण 2: सीधे क्लाइंट के साथ परीक्षण करें

**नया** टर्मिनल खोलें जहाँ सर्वर चल रहा हो, सीधे MCP क्लाइंट चलाएं:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

आपको आउटपुट इस तरह दिखेगा:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### चरण 3: AI क्लाइंट के साथ परीक्षण करें

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

AI उपकरणों का स्वचालित उपयोग करते हुए दिखेगा:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### चरण 4: MCP सर्वर बंद करें

जब आप परीक्षण समाप्त कर लें, AI क्लाइंट को बंद करने के लिए उसके टर्मिनल में `Ctrl+C` दबाएं। MCP सर्वर तब तक चलता रहेगा जब तक आप इसे बंद न करें।
सर्वर बंद करने के लिए, उस टर्मिनल में `Ctrl+C` दबाएं जहाँ यह चल रहा हो।

## यह सब कैसे काम करता है

यहाँ पूरी प्रक्रिया है जब आप AI से पूछते हैं "5 + 3 क्या है?":

1. **आप** प्राकृतिक भाषा में AI से पूछते हैं
2. **AI** आपके अनुरोध का विश्लेषण करता है और समझता है कि आप जोड़ना चाहते हैं
3. **AI** MCP सर्वर को कॉल करता है: `add(5.0, 3.0)`
4. **कैलकुलेटर सेवा** प्रदर्शन करती है: `5.0 + 3.0 = 8.0`
5. **कैलकुलेटर सेवा** लौटाती है: `"5.00 + 3.00 = 8.00"`
6. **AI** परिणाम प्राप्त करता है और प्राकृतिक प्रतिक्रिया तैयार करता है
7. **आप** पाते हैं: "5 और 3 का योग 8 है"

## अगले कदम

अधिक उदाहरणों के लिए देखें [अध्याय 04: व्यावहारिक उदाहरण](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**अस्वीकरण**:
इस दस्तावेज़ का अनुवाद AI अनुवाद सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) का उपयोग करके किया गया है। जबकि हम सटीकता के लिए प्रयास करते हैं, कृपया ध्यान दें कि स्वचालित अनुवादों में त्रुटियाँ या अशुद्धियाँ हो सकती हैं। मूल दस्तावेज़ अपनी मूल भाषा में ही प्रामाणिक स्रोत माना जाना चाहिए। महत्वपूर्ण जानकारी के लिए, पेशेवर मानव अनुवाद की सिफारिश की जाती है। इस अनुवाद के उपयोग से उत्पन्न किसी भी गलतफहमी या गलत व्याख्या के लिए हम उत्तरदायी नहीं हैं।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->