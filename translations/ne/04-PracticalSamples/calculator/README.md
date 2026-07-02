# MCP क्यालकुलेटर ट्युटोरियल शुरुवातकर्ताका लागि

## विषय सूची

- [तपाईं के सिक्नेछ](#तपाईं-के-सिक्नेछ)
- [आवश्यकताहरू](#आवश्यकताहरू)
- [परियोजनाको संरचना बुझ्नुहोस्](#परियोजनाको-संरचना-बुझ्नुहोस्)
- [प्रमुख कम्पोनेन्टहरू व्याख्या](#प्रमुख-कम्पोनेन्टहरू-व्याख्या)
  - [1. मुख्य एप्लिकेशन](#1-मुख्य-एप्लिकेशन)
  - [2. क्यालकुलेटर सेवा](#2-क्यालकुलेटर-सेवा)
  - [3. सिधा MCP क्लाइन्ट](#3-सिधा-mcp-क्लाइन्ट)
  - [4. एआई-संचालित क्लाइन्ट](#4-एआई-संचालित-क्लाइन्ट)
- [उदाहरणहरू चलाउने तरिका](#उदाहरणहरू-चलाउने-तरिका)
- [सबै एकसाथ कसरी काम गर्छ](#सबै-एकसाथ-कसरी-काम-गर्छ)
- [अगाडि के गर्ने](#अगाडि-के-गर्ने)

## तपाईं के सिक्नेछ

यो ट्युटोरियलले तपाईंलाई Model Context Protocol (MCP) प्रयोग गरी क्यालकुलेटर सेवा निर्माण गर्ने तरिका बताउँछ। तपाईं बुझ्ने हुनुहुनेछ:

- कसरी एआईले उपकरणको रूपमा प्रयोग गर्न सकिने सेवा बनाउने
- कसरी MCP सेवाहरूसँग सिधा सञ्चार सेट अप गर्ने
- कसरी एआई मोडेलहरूले आफैँ आवश्यक उपकरणहरू छान्छन्
- सिधा प्रोटोकल कलहरू र एआई सहायता गरिएको अन्तरक्रियाहरू बीचको फरक

## आवश्यकताहरू

सुरु गर्नु अघि, सुनिश्चित गर्नुहोस् तपाईंसँग:
- Java 21 वा त्यो भन्दा माथि संस्करण स्थापना गरिएको
- Maven निर्भरता व्यवस्थापनका लागि
- Azure AI Foundry मोडेल परिनियोजन ( `azd up` प्रयोग गरी प्रावधान गर्नुस् — हेर्नुहोस् [अध्याय २](../../02-SetupDevEnvironment/getting-started-azure-openai.md) )
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), `az login` सँग हस्ताक्षर गरिएको (की बिना प्रमाणित)
- Java र Spring Boot को आधारभूत ज्ञान

## परियोजनाको संरचना बुझ्नुहोस्

क्यालकुलेटर परियोजनामा केही महत्वपूर्ण फाइलहरू छन्:

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

## प्रमुख कम्पोनेन्टहरू व्याख्या

### 1. मुख्य एप्लिकेशन

**फाइल:** `McpServerApplication.java`

यो हाम्रो क्यालकुलेटर सेवाको प्रवेश बिन्दु हो। यो एक सामान्य Spring Boot एप्लिकेशन हो जसमा एउटा विशेष थप सुविधा छ:

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

**यो के गर्दछ:**
- पर्ट 8080 मा Spring Boot वेब सर्भर सुरु गर्छ
- एक `ToolCallbackProvider` बनाउँछ जसले हाम्रो क्यालकुलेटर विधिहरूलाई MCP उपकरणको रूपमा उपलब्ध गराउँछ
- `@Bean` एनोटेशनले Spring लाई यो कम्पोनेन्टको रूपमा व्यवस्थापन गर्न बताउँछ जुन अरू भागहरूले प्रयोग गर्न सक्छन्

### 2. क्यालकुलेटर सेवा

**फाइल:** `CalculatorService.java`

यहाँ सबै गणित संचालनहरू हुन्छन्। प्रत्येक विधि `@Tool` ले चिन्हित गरिएको छ ताकि यो MCP मार्फत उपलब्ध हुन सकोस्:

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
    
    // थप क्याल्कुलेटर अपरेशन्स...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**मूल विशेषताहरू:**

1. **`@Tool` एनोटेशन**: यसले MCP लाई यो विधि बाह्य क्लाइन्टहरूले कल गर्न सक्ने बताउँछ
2. **स्पष्ट विवरणहरू**: प्रत्येक उपकरणसँग एउटा विवरण हुन्छ जसले एआई मोडेललाई कहिले प्रयोग गर्ने भनेर बुझ्न मद्दत गर्छ
3. **समान फर्कने ढाँचा**: सबै अपरेसनहरूले मानव-पठनीय स्ट्रिङहरू फर्काउँछन् जस्तै "5.00 + 3.00 = 8.00"
4. **त्रुटि ह्यान्डलिंग**: शून्यले भाग गर्दा र नकारात्मक वर्गमुलको लागि त्रुटि सन्देशहरू फर्काउँछ

**उपलब्ध अपरेसनहरू:**
- `add(a, b)` - दुई संख्या जोड्दछ
- `subtract(a, b)` - दोस्रोलाई पहिलोबाट घटाउँछ
- `multiply(a, b)` - दुई संख्या गुणा गर्दछ
- `divide(a, b)` - पहिलोलाई दोस्रोले भाग गर्दछ (शून्य जाँच सहित)
- `power(base, exponent)` - आधारलाई घातमा उठाउँछ
- `squareRoot(number)` - वर्गमूल गणना गर्दछ (नकारात्मक जाँच सहित)
- `modulus(a, b)` - भागफाँटका बाँकी भाग फर्काउँछ
- `absolute(number)` - पूर्णांक मान फर्काउँछ
- `help()` - सबै अपरेसनहरूको जानकारी फर्काउँछ

### 3. सिधा MCP क्लाइन्ट

**फाइल:** `SDKClient.java`

यो क्लाइन्ट एआई प्रयोग नगरी सीधे MCP सर्भरसँग कुरा गर्छ। यसले निश्चित क्यालकुलेटर कार्यहरूलाई म्यानुअली कल गर्दछ:

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
        
        // उपलब्ध उपकरणहरूको सूची बनाउनुहोस्
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

**यो के गर्दछ:**
1. बिल्डर पद्धति प्रयोग गरी `http://localhost:8080` मा क्यालकुलेटर सर्भरसँग जडान गर्छ
2. उपलब्ध सबै उपकरणहरूको सूची देखाउँछ (हाम्रो क्यालकुलेटर कार्यहरू)
3. निश्चित कार्यहरूलाई ठ्याक्कै प्यारेमिटरहरूसँग कल गर्छ
4. नतिजाहरू सोझै प्रिन्ट गर्छ

**ध्यान दिनुहोस्:** यो उदाहरणले Spring AI 1.1.0-SNAPSHOT निर्भरता प्रयोग गरेको छ, जसले `WebFluxSseClientTransport` को लागि बिल्डर पद्धति प्रस्तुत गरेको छ। यदि तपाईं पुरानो स्थिर संस्करण प्रयोग गर्नुहुन्छ भने सिधा कन्स्ट्रक्टर प्रयोग गर्नुपर्ने हुन सक्छ।

**यो कहिले प्रयोग गर्ने:** जब तपाईंलाई ठ्याक्कै कुन गणना गर्नुपर्ने थाहा छ र प्रोग्रामिङ्गबाट कल गर्न चाहनुहुन्छ।

### 4. एआई-संचालित क्लाइन्ट

**फाइल:** `LangChain4jClient.java`

यो क्लाइन्टले एआई मोडेल (GPT-4o-mini) प्रयोग गर्दछ जसले स्वतः क्यालकुलेटर उपकरणहरू छनौट गर्न सक्छ:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // एआई मोडेल सेट अप गर्नुहोस् (एजुर एआई फाउन्ड्री, माइक्रोसफ्ट एन्ट्रा आईडी मार्फत कीलेस प्रमाणीकरण)
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

        // हाम्रो क्याल्कुलेटर MCP सर्भरमा जडान गर्नुहोस्
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // एआई के गर्दैछ देखाउँछ
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // एआईलाई हाम्रो क्याल्कुलेटर उपकरणमा पहुँच दिनुहोस्
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // यस्तो एआई बॉट बनाउनुहोस् जुन हाम्रो क्याल्कुलेटर प्रयोग गर्न सक्छ
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // अब हामी प्राकृतिक भाषामा गणना गर्न एआईलाई सोध्न सक्छौं
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**यो के गर्दछ:**
1. तपाईंको GitHub टोकन प्रयोग गरी एआई मोडेल जडान बनाउँछ
2. एआईलाई हाम्रो क्यालकुलेटर MCP सर्भरसँग जडान गर्छ
3. एआईलाई हाम्रो सबै क्यालकुलेटर उपकरणहरूको पहुँच दिन्छ
4. "24.5 र 17.3 को योगफल गणना गर" जस्ता प्राकृतिक भाषा अनुरोध स्वीकार्छ

**एआईले स्वतः:**
- तपाईंले संख्या जोड्न चाहनुहुन्छ भनेर बुझ्छ
- `add` उपकरण छान्दछ
- `add(24.5, 17.3)` कल गर्दछ
- नतिजा प्राकृतिक जवाफमा फर्काउँछ

## उदाहरणहरू चलाउने तरिका

### चरण 1: क्यालकुलेटर सर्भर सुरु गर्नुहोस्

पहिले, साइन इन गर्नुहोस् र Azure AI Foundry एन्डप्वाइन्ट सेट गर्नुहोस् (एआई क्लाइन्टका लागि आवश्यक — की बिना प्रमाणिकरण, API किजस्तो आवश्यक छैन):

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

सर्भर `http://localhost:8080` मा सुरु हुनेछ। तपाईंले यस्तो देख्नुहुनेछ:
```
Started McpServerApplication in X.XXX seconds
```

### चरण 2: सिधा क्लाइन्टसँग परीक्षण गर्नुहोस्

सर्भर चलिरहँदा **नयाँ** टर्मिनल खोल्नुहोस् र सिधा MCP क्लाइन्ट चलाउनुहोस्:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

तपाईं यस्तो आउटपुट देख्नुहुनेछ:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### चरण 3: एआई क्लाइन्टसँग परीक्षण गर्नुहोस्

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

तपाईंले एआईले स्वतः उपकरणहरू प्रयोग गरेको देख्नुहुनेछ:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### चरण 4: MCP सर्भर बन्द गर्नुहोस्

परीक्षण सकिएपछि, एआई क्लाइन्टलाई बन्द गर्न यसको टर्मिनलमा `Ctrl+C` थिच्नुहोस्। MCP सर्भर तबसम्म चलिरहन्छ जबसम्म तपाईं त्यसलाई बन्द गर्नुहुन्न।
सर्भर बन्द गर्न, सर्भर चलिरहेको टर्मिनलमा `Ctrl+C` थिच्नुहोस्।

## सबै एकसाथ कसरी काम गर्छ

यहाँ पूर्ण प्रक्रिया छ जब तपाईंले एआईलाई "5 + 3 कति हो?" सोध्नुहुन्छ:

1. **तपाईं** एआईलाई प्राकृतिक भाषामा सोध्नुहुन्छ
2. **एआई** तपाईँको अनुरोध विश्लेषण गर्दछ र बुझ्छ कि तपाईंलाई जोड चाहिन्छ
3. **एआई** MCP सर्भरलाई कल गर्छ: `add(5.0, 3.0)`
4. **क्यालकुलेटर सेवा** कार्यान्वयन गर्दछ: `5.0 + 3.0 = 8.0`
5. **क्यालकुलेटर सेवा** फर्काउँछ: `"5.00 + 3.00 = 8.00"`
6. **एआई** नतिजा प्राप्त गरी प्राकृतिक जवाफ तयार पार्छ
7. **तपाईं** पाउनुहुन्छ: "5 र 3 को योगफल 8 हो"

## अगाडि के गर्ने

थप उदाहरणहरूको लागि, हेर्नुहोस् [अध्याय ०४: व्यावहारिक नमूनाहरू](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**अस्वीकरण**:
यो दस्तावेज़ AI अनुवाद सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) प्रयोग गरेर अनुवाद गरिएको हो। हामी सही हुन प्रयास गर्छौं, तर कृपया जानकार हुनुस् कि स्वचालित अनुवादमा त्रुटिहरू वा अशुद्धताहरू हुन सक्छन्। मूल दस्तावेज़ यसको मूल भाषामा आधिकारिक स्रोत मानिनुपर्छ। महत्वपूर्ण जानकारीका लागि व्यावसायिक मानव अनुवाद सिफारिस गरिन्छ। यस अनुवादको प्रयोगबाट उत्पन्न कुनै पनि गलत बुझाइ वा त्रुटिको लागि हामी जिम्मेवार छैनौं।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->