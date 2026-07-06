# MCP क्यालकुलेटर ट्युटोरियल बिगिनरहरूको लागि

## सामग्री

- [तपाईं के सिक्नुहुनेछ](#तपाईं-के-सिक्नुहुनेछ)
- [पूर्वआवश्यकताहरु](#पूर्वआवश्यकताहरु)
- [प्रोजेक्ट संरचना बुझ्नुहोस्](#प्रोजेक्ट-संरचना-बुझ्नुहोस्)
- [मूल कम्पोनेन्टहरू व्याख्या गरियो](#मूल-कम्पोनेन्टहरू-व्याख्या-गरियो)
  - [१. मुख्य एप्लिकेसन](#१-मुख्य-एप्लिकेसन)
  - [२. क्यालकुलेटर सेवा](#२-क्यालकुलेटर-सेवा)
  - [३. प्रत्यक्ष MCP क्लाइन्ट](#३-प्रत्यक्ष-mcp-क्लाइन्ट)
  - [४. एआई-समर्थित क्लाइन्ट](#४-एआई-समर्थित-क्लाइन्ट)
- [उदाहरणहरू चलाउने तरिका](#उदाहरणहरू-चलाउने-तरिका)
- [तिनीहरू कसरी सँगै काम गर्छन्](#तिनीहरू-कसरी-सँगै-काम-गर्छन्)
- [अर्को कदमहरू](#अर्को-कदमहरू)

## तपाईं के सिक्नुहुनेछ

यस ट्युटोरियलले मोडल कन्टेक्ट प्रोटोकल (MCP) प्रयोग गरेर क्यालकुलेटर सेवा कसरी बनाउने व्याख्या गर्छ। तपाईंले बुझ्नुहुनेछ:

- एआईलाई उपकरणको रूपमा प्रयोग गर्न सेवा कसरी बनाउने
- MCP सेवाहरू सँग प्रत्यक्ष संवाद कसरी सेटअप गर्ने
- एआई मोडेलहरूले कुन उपकरण प्रयोग गर्ने स्वचालित रूपमा कसरी छनौट गर्छन्
- प्रत्यक्ष प्रोटोकल कल र एआई-सहायता प्राप्त अन्तरक्रियाको बीच फरक

## पूर्वआवश्यकताहरु

सुरू गर्नु अघि, सुनिश्चित गर्नुहोस् तपाईंले:
- Java 21 वा माथि इन्स्टल गर्नुभएको छ
- Maven निर्भरता व्यवस्थापनका लागि
- Azure AI Foundry मोडल डिप्लोइमेन्ट गरिएको छ (`azd up` ले प्रावधान गर्नुस् — हेर्नुहोस् [अध्याय २](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) इन्स्टल र `az login` सँग साईन इन (keyless authentication)
- Java र Spring Boot को आधारभूत बुझाइ

## प्रोजेक्ट संरचना बुझ्नुहोस्

क्यालकुलेटर प्रोजेक्टमा केही महत्वपूर्ण फाइलहरू छन्:

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

## मूल कम्पोनेन्टहरू व्याख्या गरियो

### १. मुख्य एप्लिकेसन

**फाइल:** `McpServerApplication.java`

यो हाम्रो क्यालकुलेटर सेवाको प्रवेश बिन्दु हो। यो एउटा मानक Spring Boot एप्लिकेसन हो जसमा एउटा विशेष थप हुन्छ:

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

**यसले के गर्छ:**
- पोर्ट 8080 मा Spring Boot वेब सर्भर सुरु गर्छ
- `ToolCallbackProvider` बनाउँछ जसले हाम्रो क्यालकुलेटर मेथडहरूलाई MCP उपकरणको रूपमा उपलब्ध गराउँछ
- `@Bean` एनोटेसनले Spring लाई यसको प्रबन्धन गर्न र अन्य भागहरूले प्रयोग गर्न दिन्छ

### २. क्यालकुलेटर सेवा

**फाइल:** `CalculatorService.java`

यहाँ सबै गणितीय कामहरू हुन्छन्। प्रत्येक मेथड `@Tool` ले चिन्हित गरिएको छ ताकि यो MCP मार्फत उपलब्ध होस्:

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
    
    // थप क्याल्कुलेटर अपरेसनहरू...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**मुख्य विशेषताहरू:**

1. **`@Tool` एनोटेसन**: यसले MCP लाई भन्छ कि यो मेथड बाह्य क्लाइन्टहरूले कल गर्न सक्छन्
2. **स्पष्ट वर्णनहरू**: प्रत्येक उपकरणसँग यस्तो वर्णन हुन्छ जसले एआई मोडेललाई कहिले प्रयोग गर्ने भन्ने बुझ्न सहयोग गर्छ
3. **सुसंगत रिटर्न फारम्याट**: सबै अपरेसनहरूले मानिसले पढ्न सक्ने स्ट्रिङहरू फर्काउँछन् जस्तै "5.00 + 3.00 = 8.00"
4. **त्रुटि ह्यान्डलिङ**: शून्यले विभाजन गर्दा र ऋणात्मक बेउ जड निकाल्दा त्रुटि सन्देश फर्काउँछ

**उपलब्ध अपरेसनहरू:**
- `add(a, b)` - दुई संख्या जोड्दछ
- `subtract(a, b)` - दोस्रो बाट पहिलो घटाउँछ
- `multiply(a, b)` - दुई संख्या गुणा गर्दछ
- `divide(a, b)` - पहिलोलाई दोस्रोले विभाजन गर्दछ (शून्य जाँच सहित)
- `power(base, exponent)` - आधारलाई घातांकमा उठाउँछ
- `squareRoot(number)` - वर्गमूल निकाल्छ (ऋणात्मक जाँच सहित)
- `modulus(a, b)` - भागशेष फर्काउँछ
- `absolute(number)` - पूर्णांक मान फर्काउँछ
- `help()` - सबै अपरेसनहरूको जानकारी फर्काउँछ

### ३. प्रत्यक्ष MCP क्लाइन्ट

**फाइल:** `SDKClient.java`

यो क्लाइन्ट एआई प्रयोग नगरी प्रत्यक्ष MCP सर्भरसँग कुराकानी गर्छ। यो म्यानुअली विशेष क्यालकुलेटर फङ्सनहरू कल गर्छ:

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
        
        // उपलब्ध उपकरणहरू सूचीबद्ध गर्नुहोस्
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // विशेष क्याल्कुलेटर कार्यहरू कल गर्नुहोस्
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

**यसले के गर्छ:**
1. `http://localhost:8080` मा क्यालकुलेटर सर्भरसँग बिल्डर प्याटर्न प्रयोग गरी जडान गर्छ
2. सबै उपलब्ध उपकरणहरू (हाम्रो क्यालकुलेटर फङ्सनहरू) सूचीबद्ध गर्छ
3. सही प्यारामिटरहरूसँग विशेष फङ्सनहरू कल गर्छ
4. नतिजाहरू सिधै प्रिन्ट गर्छ

**नोट:** यो उदाहरणमा Spring AI 1.1.0-SNAPSHOT निर्भरता प्रयोग गरिएको छ जसले `WebFluxSseClientTransport` को लागि बिल्डर प्याटर्न प्रस्तुत गरेको हो। यदि तपाईं पुरानो स्थिर संस्करण प्रयोग गर्दै हुनुहुन्छ भने सिधा कन्स्ट्रक्टर प्रयोग गर्नुपर्ने हुन सक्छ।

**यो कहिले प्रयोग गर्ने:** जब तपाईंलाई ठ्याक्कै कुन गणना गर्नुपर्ने थाहा छ र प्रोग्रामिङ मार्फत कल गर्न चाहनुहुन्छ।

### ४. एआई-समर्थित क्लाइन्ट

**फाइल:** `LangChain4jClient.java`

यो क्लाइन्टले एआई मोडल (GPT-4o-mini) प्रयोग गर्छ जुन स्वचालित रूपमा कुन क्यालकुलेटर उपकरणहरू प्रयोग गर्ने निर्णय गर्न सक्छ:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // एआई मोडेल सेट अप गर्नुहोस् (Azure AI Foundry, Microsoft Entra ID मार्फत keyless प्रमाणीकरण)
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

        // हाम्रो क्याल्क्युलेटर MCP सर्भरसँग जडान गर्नुहोस्
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // एआई के गर्दैछ देखाउँछ
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // एआईलाई हाम्रो क्याल्क्युलेटर उपकरणहरूको पहुँच दिनुहोस्
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // हाम्रो क्याल्क्युलेटर प्रयोग गर्न सक्ने एआई बोट बनाउनुहोस्
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // अब हामी एआईलाई प्राकृतिक भाषामा गणना गर्न भन्न सक्छौं
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**यसले के गर्छ:**
1. keyless authentication (Microsoft Entra ID) प्रयोग गरेर एआई मोडल कनेक्सन सिर्जना गर्छ
2. एआईलाई हाम्रो क्यालकुलेटर MCP सर्भरसँग जोड्छ
3. एआईलाई हाम्रो सबै क्यालकुलेटर उपकरणहरूको पहुँच दिन्छ
4. "24.5 र 17.3 को योगफल गणना गर" जस्ता स्वाभाविक भाषाका अनुरोधहरूलाई अनुमति दिन्छ

**एआई स्वचालित रूपमा:**
- तपाईंले जोङ्ग गर्न चाहनु भएको बुझ्छ
- `add` उपकरण छनौट गर्छ
- `add(24.5, 17.3)` कल गर्छ
- नतिजालाई स्वाभाविक जवाफमा फर्काउँछ

## उदाहरणहरू चलाउने तरिका

### चरण १: क्यालकुलेटर सर्भर सुरु गर्नुहोस्

पहिले, साइन इन गर्नुहोस् र Azure AI Foundry अन्त बिन्दु सेट गर्नुहोस् (AI क्लाइन्टका लागि आवश्यक — keyless authentication, API कुञ्जी आवश्यक छैन):

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

सर्भर सुरु गर्नुहोस्:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

सर्भर `http://localhost:8080` मा सुरु हुनेछ। तपाईंले देख्नुहुन्छ:
```
Started McpServerApplication in X.XXX seconds
```

### चरण २: प्रत्यक्ष क्लाइन्टसँग परीक्षण गर्नुहोस्

सर्भर चलिरहेको अवस्थामा **नयाँ** टर्मिनल खोल्नुहोस् र प्रत्यक्ष MCP क्लाइन्ट चलाउनुहोस्:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

यो आउटपुट जस्तै देखिनेछ:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### चरण ३: एआई क्लाइन्टसँग परीक्षण गर्नुहोस्

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

एआईले स्वचालित रूपमा उपकरणहरूको प्रयोग गर्न सुरु गर्छ:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### चरण ४: MCP सर्भर बन्द गर्नुहोस्

परीक्षण सम्पन्न भएपछि, एआई क्लाइन्टलाई बन्द गर्न `Ctrl+C` थिच्न सक्नुहुन्छ। MCP सर्भर तबसम्म चलिरहनेछ जबसम्म तपाईं यसलाई बन्द गर्नुहुन्न।
सर्भर बन्द गर्न, जहाँ चलिरहेको टर्मिनलमा छ त्यहाँ `Ctrl+C` थिच्नुहोस्।

## तिनीहरू कसरी सँगै काम गर्छन्

यहाँ सम्पूर्ण प्रक्रिया छ जब तपाईं एआईलाई "5 + 3 के हो?" सोध्नुहुन्छ:

1. **तपाईं** एआईलाई स्वाभाविक भाषामा सोध्नुहुन्छ
2. **एआई** तपाईंको अनुरोध विश्लेषण गर्छ र बुझ्छ कि तपाईंले जोड चाहनुहुन्छ
3. **एआई** MCP सर्भरलाई कल गर्छ: `add(5.0, 3.0)`
4. **क्यालकुलेटर सेवा** गर्दछ: `5.0 + 3.0 = 8.0`
5. **क्यालकुलेटर सेवा** फर्काउँछ: `"5.00 + 3.00 = 8.00"`
6. **एआई** नतिजा प्राप्त गरी स्वाभाविक प्रतिक्रिया तयार पार्छ
7. **तपाईं** पाउनुहुन्छ: "5 र 3 को योगफल 8 हो"

## अर्को कदमहरू

थप उदाहरणका लागि, हेर्नुहोस् [अध्याय ०४: व्यवहारिक नमूना](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**अस्वीकरण**:
यो दस्तावेज़ AI अनुवाद सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) प्रयोग गरेर अनुवाद गरिएको हो। हामी सही हुन प्रयास गर्छौं, तर कृपया जानकार हुनुस् कि स्वचालित अनुवादमा त्रुटिहरू वा अशुद्धताहरू हुन सक्छन्। मूल दस्तावेज़ यसको मूल भाषामा आधिकारिक स्रोत मानिनुपर्छ। महत्वपूर्ण जानकारीका लागि व्यावसायिक मानव अनुवाद सिफारिस गरिन्छ। यस अनुवादको प्रयोगबाट उत्पन्न कुनै पनि गलत बुझाइ वा त्रुटिको लागि हामी जिम्मेवार छैनौं।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->