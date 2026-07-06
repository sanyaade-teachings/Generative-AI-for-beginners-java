# MCP कैलकुलेटर ट्यूटोरियल शुरुआती के लिए

## विषय-सूची

- [आप क्या सीखेंगे](#आप-क्या-सीखेंगे)
- [पूर्वापेक्षाएँ](#पूर्वापेक्षाएँ)
- [परियोजना संरचना को समझना](#परियोजना-संरचना-को-समझना)
- [मुख्य घटक समझाए गए](#मुख्य-घटक-समझाए-गए)
  - [1. मुख्य एप्लिकेशन](#1-मुख्य-एप्लिकेशन)
  - [2. कैलकुलेटर सेवा](#2-कैलकुलेटर-सेवा)
  - [3. डायरेक्ट MCP क्लाइंट](#3-डायरेक्ट-mcp-क्लाइंट)
  - [4. AI-चालित क्लाइंट](#4-ai-चालित-क्लाइंट)
- [उदाहरण चलाना](#उदाहरण-चलाना)
- [यह सब कैसे साथ में काम करता है](#यह-सब-कैसे-साथ-में-काम-करता-है)
- [अगले कदम](#अगले-कदम)

## आप क्या सीखेंगे

यह ट्यूटोरियल Model Context Protocol (MCP) का उपयोग करके एक कैलकुलेटर सेवा बनाने की प्रक्रिया समझाता है। आप जानेंगे:

- कैसे एक ऐसी सेवा बनाएं जिसका AI उपकरण के रूप में उपयोग कर सके
- MCP सेवाओं के साथ डायरेक्ट संचार कैसे स्थापित करें
- कैसे AI मॉडल अपने आप तय करते हैं कि कौन से उपकरणों का उपयोग करना है
- डायरेक्ट प्रोटोकॉल कॉल और AI-समर्थित इंटरैक्शन के बीच का अंतर

## पूर्वापेक्षाएँ

शुरू करने से पहले सुनिश्चित करें कि आपके पास है:
- Java 21 या उससे ऊपर संस्करण इंस्टॉल किया हुआ
- Maven डिपेंडेंसी प्रबंधन के लिए
- एक Azure AI Foundry मॉडल डिप्लॉयमेंट (इसे `azd up` से प्रावधानित करें — देखें [अध्याय 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), `az login` के साथ साइन इन (कीलेस प्रमाणीकरण)
- Java और Spring Boot की बुनियादी समझ

## परियोजना संरचना को समझना

कैलकुलेटर परियोजना में कई महत्वपूर्ण फाइलें हैं:

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

## मुख्य घटक समझाए गए

### 1. मुख्य एप्लिकेशन

**फाइल:** `McpServerApplication.java`

यह हमारे कैलकुलेटर सेवा का प्रविष्टि बिंदु है। यह एक मानक Spring Boot एप्लिकेशन है जिसमें एक विशेष वृद्धि है:

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
- एक `ToolCallbackProvider` बनाता है जो हमारे कैलकुलेटर मेथड्स को MCP उपकरणों के रूप में उपलब्ध कराता है
- `@Bean` एनोटेशन Spring को बताता है कि इसे एक घटक के रूप में मैनेज करें जिसे अन्य हिस्से उपयोग कर सकें

### 2. कैलकुलेटर सेवा

**फाइल:** `CalculatorService.java`

यहाँ सारी गणितीय प्रक्रिया होती है। प्रत्येक मेथड `@Tool` के साथ चिह्नित है ताकि इसे MCP के माध्यम से उपलब्ध कराया जा सके:

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
    
    // और गणक क्रियाएं...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**मुख्य विशेषताएँ:**

1. **`@Tool` एनोटेशन**: यह MCP को बताता है कि यह मेथड बाहरी क्लाइंट द्वारा कॉल किया जा सकता है
2. **स्पष्ट विवरण**: प्रत्येक टूल का एक विवरण है जो AI मॉडल को बताता है कि इसे कब उपयोग करना है
3. **संगत रिटर्न फॉर्मेट**: सभी ऑपरेशन मानव-पठन योग्य स्ट्रिंग लौटाते हैं जैसे "5.00 + 3.00 = 8.00"
4. **त्रुटि प्रबंधन**: शून्य से भाग और ऋणात्मक वर्गमूल पर त्रुटि संदेश लौटाते हैं

**उपलब्ध ऑपरेशन:**
- `add(a, b)` - दो नंबर जोड़ता है
- `subtract(a, b)` - दूसरे से पहले को घटाता है
- `multiply(a, b)` - दो नंबरों को गुणा करता है
- `divide(a, b)` - पहले को दूसरे से विभाजित करता है (शून्य जाँच के साथ)
- `power(base, exponent)` - बेस को घातांक तक उठाता है
- `squareRoot(number)` - वर्गमूल निकालता है (ऋण जाँच के साथ)
- `modulus(a, b)` - भागफल का शेषांश लौटाता है
- `absolute(number)` - पूर्णांक मान लौटाता है
- `help()` - सभी ऑपरेशन के बारे में जानकारी लौटाता है

### 3. डायरेक्ट MCP क्लाइंट

**फाइल:** `SDKClient.java`

यह क्लाइंट सीधे MCP सर्वर से बात करता है बिना AI का उपयोग किए। यह विशेष कैलकुलेटर फ़ंक्शंस को मैन्युअली कॉल करता है:

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
        
        // उपलब्ध उपकरणों की सूची बनाएं
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // विशिष्ट कैलकुलेटर कार्यों को कॉल करें
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
1. बिल्डर पैटर्न का उपयोग कर `http://localhost:8080` पर कैलकुलेटर सर्वर से कनेक्ट होता है
2. सभी उपलब्ध टूल्स (हमारे कैलकुलेटर फ़ंक्शंस) को सूचीबद्ध करता है
3. सटीक पैरामीटर के साथ विशिष्ट फ़ंक्शंस को कॉल करता है
4. परिणामों को सीधे प्रिंट करता है

**टिप्पणी:** यह उदाहरण Spring AI 1.1.0-SNAPSHOT डिपेंडेंसी का उपयोग करता है, जिसने `WebFluxSseClientTransport` के लिए बिल्डर पैटर्न पेश किया है। यदि आप पुराना स्थिर संस्करण उपयोग कर रहे हैं, तो आपको सीधे कंस्ट्रक्टर का उपयोग करना पड़ सकता है।

**कब उपयोग करें:** जब आप जानते हैं कि कौन सा कैलकुलेशन करना है और इसे प्रोग्रामेटिक रूप से कॉल करना चाहते हैं।

### 4. AI-चालित क्लाइंट

**फाइल:** `LangChain4jClient.java`

यह क्लाइंट AI मॉडल (GPT-4o-mini) का उपयोग करता है जो स्वचालित रूप से तय करता है कि कौन से कैलकुलेटर टूल्स का उपयोग करना है:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // एआई मॉडल सेट अप करें (Azure AI Foundry, Microsoft Entra ID के माध्यम से keyless प्रमाणीकरण)
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
                .logRequests(true)  // दिखाता है कि एआई क्या कर रहा है
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // एआई को हमारे कैलकुलेटर टूल्स तक पहुंच दें
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // एक एआई बॉट बनाएं जो हमारे कैलकुलेटर का उपयोग कर सके
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // अब हम प्राकृतिक भाषा में एआई से गणनाएं करने के लिए पूछ सकते हैं
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**यह क्या करता है:**
1. कीलेस प्रमाणीकरण (Microsoft Entra ID) का उपयोग कर AI मॉडल कनेक्शन बनाता है
2. AI को हमारे कैलकुलेटर MCP सर्वर से जोड़ता है
3. AI को हमारे सभी कैलकुलेटर टूल्स तक पहुँच देता है
4. "24.5 और 17.3 का योग निकालो" जैसी प्राकृतिक भाषा अनुरोधों की अनुमति देता है

**AI स्वचालित रूप से:**
- समझता है कि आप नंबर जोड़ना चाहते हैं
- `add` टूल चुनता है
- `add(24.5, 17.3)` कॉल करता है
- परिणाम को प्राकृतिक प्रतिक्रिया में लौटाता है

## उदाहरण चलाना

### चरण 1: कैलकुलेटर सर्वर शुरू करें

सबसे पहले, साइन इन करें और अपना Azure AI Foundry endpoint सेट करें (AI क्लाइंट के लिए आवश्यक — कीलेस प्रमाणीकरण, कोई API कुंजी नहीं):

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

सर्वर शुरू करें:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

सर्वर `http://localhost:8080` पर शुरू होगा। आपको दिखेगा:
```
Started McpServerApplication in X.XXX seconds
```

### चरण 2: डायरेक्ट क्लाइंट के साथ परीक्षण करें

सर्वर चालू रखते हुए एक **नई** टर्मिनल में डायरेक्ट MCP क्लाइंट चलाएं:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

आपको इस तरह का आउटपुट दिखेगा:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### चरण 3: AI क्लाइंट के साथ परीक्षण करें

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

AI स्वचालित रूप से टूल्स का उपयोग करता दिखेगा:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### चरण 4: MCP सर्वर बंद करें

जब आप परीक्षण समाप्त कर लें, तो AI क्लाइंट को रोकने के लिए उसके टर्मिनल में `Ctrl+C` दबाएं। MCP सर्वर तब तक चलता रहेगा जब तक आप उसे नहीं बंद करते।
सर्वर को बंद करने के लिए, उस टर्मिनल में `Ctrl+C` दबाएं जहाँ यह चल रहा है।

## यह सब कैसे साथ में काम करता है

यहाँ पूरा प्रवाह है जब आप AI से पूछते हैं "5 + 3 क्या है?":

1. **आप** AI से प्राकृतिक भाषा में पूछते हैं
2. **AI** आपकी अनुरोध का विश्लेषण करता है और समझता है कि आप जोड़ चाहते हैं
3. **AI** MCP सर्वर को कॉल करता है: `add(5.0, 3.0)`
4. **कैलकुलेटर सेवा** करता है: `5.0 + 3.0 = 8.0`
5. **कैलकुलेटर सेवा** लौटाता है: `"5.00 + 3.00 = 8.00"`
6. **AI** परिणाम प्राप्त करता है और एक प्राकृतिक प्रतिक्रिया तैयार करता है
7. **आप** पाते हैं: "5 और 3 का योग 8 है"

## अगले कदम

अधिक उदाहरणों के लिए देखें [अध्याय 04: व्यावहारिक नमूने](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**अस्वीकरण**:
इस दस्तावेज़ का अनुवाद AI अनुवाद सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) का उपयोग करके किया गया है। जबकि हम सटीकता के लिए प्रयास करते हैं, कृपया ध्यान दें कि स्वचालित अनुवादों में त्रुटियाँ या अशुद्धियाँ हो सकती हैं। मूल दस्तावेज़ अपनी मूल भाषा में ही प्रामाणिक स्रोत माना जाना चाहिए। महत्वपूर्ण जानकारी के लिए, पेशेवर मानव अनुवाद की सिफारिश की जाती है। इस अनुवाद के उपयोग से उत्पन्न किसी भी गलतफहमी या गलत व्याख्या के लिए हम उत्तरदायी नहीं हैं।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->