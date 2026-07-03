# MCP ক্যালকুলেটর টিউটোরিয়াল শুরু করার জন্য

## বিষয়বস্তু

- [আপনি যা শিখবেন](#আপনি-যা-শিখবেন)
- [প্রয়োজনীয়তাসমূহ](#প্রয়োজনীয়তাসমূহ)
- [প্রকল্প কাঠামো বোঝা](#প্রকল্প-কাঠামো-বোঝা)
- [কোর উপাদানগুলি ব্যাখ্যা](#কোর-উপাদানগুলি-ব্যাখ্যা)
  - [১। প্রধান অ্যাপ্লিকেশন](#১।-প্রধান-অ্যাপ্লিকেশন)
  - [২। ক্যালকুলেটর সার্ভিস](#২।-ক্যালকুলেটর-সার্ভিস)
  - [৩। ডিরেক্ট MCP ক্লায়েন্ট](#৩।-ডিরেক্ট-mcp-ক্লায়েন্ট)
  - [৪। AI-চালিত ক্লায়েন্ট](#৪।-ai-চালিত-ক্লায়েন্ট)
- [উদাহরণগুলি চালানো](#উদাহরণগুলি-চালানো)
- [কিভাবে সব একসঙ্গে কাজ করে](#কিভাবে-সব-একসঙ্গে-কাজ-করে)
- [পরবর্তী ধাপ](#পরবর্তী-ধাপ)

## আপনি যা শিখবেন

এই টিউটোরিয়ালে ব্যাখ্যা করা হয়েছে কিভাবে Model Context Protocol (MCP) ব্যবহার করে একটি ক্যালকুলেটর সার্ভিস তৈরি করতে হয়। আপনি বুঝতে পারবেন:

- কিভাবে একটি সার্ভিস তৈরি করবেন যা AI একটি টুল হিসেবে ব্যবহার করতে পারে
- MCP সার্ভিসের সাথে সরাসরি যোগাযোগ স্থাপন করার পদ্ধতি
- কিভাবে AI মডেলগুলি স্বয়ংক্রিয়ভাবে কোন টুল ব্যবহার করবে তা বেছে নেয়
- সরাসরি প্রোটোকল কল এবং AI-সহায়তাধীন ইন্টারঅ্যাকশনের মধ্যে পার্থক্য

## প্রয়োজনীয়তাসমূহ

শুরু করার আগে নিশ্চিত করুন আপনার কাছে আছে:
- Java 21 বা তার বেশি ইনস্টল করা
- Maven ডিপেনডেন্সি ব্যবস্থাপনার জন্য
- একটি Azure AI Foundry মডেল ডিপ্লয়মেন্ট (প্রদান করুন `azd up` দিয়ে — দেখুন [অধ্যায় ২](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), `az login` দিয়ে সাইন ইন করা (কি ছাড়া প্রমাণীকরণ)
- Java এবং Spring Boot সম্পর্কে প্রাথমিক ধারণা

## প্রকল্প কাঠামো বোঝা

ক্যালকুলেটর প্রকল্পের বেশ কিছু গুরুত্বপূর্ণ ফাইল রয়েছে:

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

## কোর উপাদানগুলি ব্যাখ্যা

### ১। প্রধান অ্যাপ্লিকেশন

**ফাইল:** `McpServerApplication.java`

এটি আমাদের ক্যালকুলেটর সার্ভিসের প্রবেশদ্বার। এটি একটি স্ট্যান্ডার্ড Spring Boot অ্যাপ্লিকেশন যার মধ্যে একটি বিশেষ সংযোজন আছে:

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

**এটি যা করে:**
- ৮০৮০ পোর্টে Spring Boot ওয়েব সার্ভার চালু করে
- একটি `ToolCallbackProvider` তৈরি করে যা আমাদের ক্যালকুলেটর মেথডগুলো MCP টুল হিসেবে উপলব্ধ করে
- `@Bean` অ্যানোটেশন Spring কে বলে এটি এমন একটি কম্পোনেন্ট যা অন্যান্য অংশ ব্যবহার করতে পারবে

### ২। ক্যালকুলেটর সার্ভিস

**ফাইল:** `CalculatorService.java`

এখানে সব গণনা ঘটে। প্রতিটি মেথডে `@Tool` চিহ্ন দেয়া হয়েছে যাতে MCP এর মাধ্যমে এটি উপলব্ধ হয়:

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
    
    // আরও ক্যালকুলেটর অপারেশন...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**মূল বৈশিষ্ট্য:**

১. **`@Tool` অ্যানোটেশন**: এটি MCP কে বলে যে এই মেথডটি বহিরাগত ক্লায়েন্ট দ্বারা ডাকা যেতে পারে  
২. **স্পষ্ট বর্ণনা**: প্রতিটি টুলের এমন একটি বর্ণনা আছে যা AI মডেলকে বোঝাতে সাহায্য করে কখন ব্যবহার করতে হবে  
৩. **সঙ্গত ফরম্যাটে রিটার্ন**: সব অপারেশন মানুষ পড়তে পারা স্ট্রিং যেমন "5.00 + 3.00 = 8.00" রিটার্ন করে  
৪. **ত্রুটি হ্যান্ডলিং**: শূন্য দিয়ে ভাগ এবং ঋণাত্মক বর্গমূলের জন্য ত্রুটির বার্তা রিটার্ন করে

**উপলব্ধ অপারেশনসমূহ:**  
- `add(a, b)` - দুই সংখ্যার যোগফল  
- `subtract(a, b)` - প্রথম থেকে দ্বিতীয় বিয়োগ  
- `multiply(a, b)` - দুই সংখ্যার গুণফল  
- `divide(a, b)` - প্রথম সংখ্যাকে দ্বিতীয় দ্বারা ভাগ (শূন্য পরীক্ষা সহ)  
- `power(base, exponent)` - বেসের শক্তি নির্ণয়  
- `squareRoot(number)` - বর্গমূল হিসাব (ঋণাত্মক পরীক্ষা সহ)  
- `modulus(a, b)` - ভাগশেষ নির্ণয়  
- `absolute(number)` - পরম মান নির্ণয়  
- `help()` - সব অপারেশন সম্পর্কে তথ্য প্রদান

### ৩। ডিরেক্ট MCP ক্লায়েন্ট

**ফাইল:** `SDKClient.java`

এই ক্লায়েন্ট AI ব্যবহার না করে সরাসরি MCP সার্ভারের সাথে কথা বলে। এটি নির্দিষ্ট ক্যালকুলেটর ফাংশন ম্যানুয়ালি কল করে:

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
        
        // উপলব্ধ সরঞ্জামগুলির তালিকা দিন
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // নির্দিষ্ট ক্যালকুলেটর ফাংশনগুলো কল করুন
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

**এটি যা করে:**
১. বিল্ডার প্যাটার্ন ব্যবহার করে `http://localhost:8080` এ ক্যালকুলেটর সার্ভারের সাথে সংযোগ করে  
২. সব উপলব্ধ টুলের তালিকা দেখায় (আমাদের ক্যালকুলেটর ফাংশনগুলি)  
৩. নির্দিষ্ট ফাংশন নির্দিষ্ট প্যারামিটার নিয়ে কল করে  
৪. ফলাফল সরাসরি প্রিন্ট করে

**মনে রাখবেন:** এই উদাহরণটি Spring AI 1.1.0-SNAPSHOT ডিপেনডেন্সি ব্যবহার করে, যা `WebFluxSseClientTransport` এর বিল্ডার প্যাটার্ন প্রবর্তন করেছে। যদি আপনি পুরানো স্থিতিশীল সংস্করণ ব্যবহার করেন, তাহলে সরাসরি কনস্ট্রাক্টর ব্যবহার করতে হতে পারে।

**কখন ব্যবহার করবেন:** যখন আপনি সুনির্দিষ্ট গণনা জানতে চান এবং প্রোগ্রাম্যাটিকভাবে কল করতে চান।

### ৪। AI-চালিত ক্লায়েন্ট

**ফাইল:** `LangChain4jClient.java`

এই ক্লায়েন্ট AI মডেল (GPT-4o-mini) ব্যবহার করে, যা স্বয়ংক্রিয়ভাবে কোন ক্যালকুলেটর টুল ব্যবহার করবে তা নির্ধারণ করে:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // AI মডেল সেট আপ করুন (Azure AI Foundry, Microsoft Entra ID এর মাধ্যমে কীলেস প্রমাণীকরণ)
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

        // আমাদের ক্যালকুলেটর MCP সার্ভারের সাথে সংযোগ করুন
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // প্রদর্শন করে AI কী করছে
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // AI কে আমাদের ক্যালকুলেটর টুলগুলোর অ্যাক্সেস দিন
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // একটি AI বট তৈরি করুন যা আমাদের ক্যালকুলেটর ব্যবহার করতে পারে
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // এখন আমরা AI কে প্রাকৃতিক ভাষায় গণনা করার জন্য অনুরোধ করতে পারি
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**এটি যা করে:**
১. কী ছাড়া প্রমাণীকরণ ব্যবহার করে AI মডেল সংযোগ তৈরি করে (Microsoft Entra ID)  
২. AI কে আমাদের ক্যালকুলেটর MCP সার্ভারের সাথে সংযোগ করায়  
৩. AI কে সব ক্যালকুলেটর টুল ব্যবহার করার অনুমতি দেয়  
৪. স্বাভাবিক ভাষায় অনুরোধ গ্রহণ করে যেমন "24.5 এবং 17.3 এর যোগফল গণনা করো"

**AI স্বয়ংক্রিয়ভাবে:**
- বুঝে আপনি যোগফল চান  
- `add` টুল বেছে নেয়  
- `add(24.5, 17.3)` কল করে  
- ফলাফল স্বাভাবিক ভাষায় প্রদান করে

## উদাহরণগুলি চালানো

### ধাপ ১: ক্যালকুলেটর সার্ভার চালু করা

প্রথমে সাইন ইন করুন এবং আপনার Azure AI Foundry এন্ডপয়েন্ট সেট করুন (AI ক্লায়েন্টের জন্য প্রয়োজন — কী ছাড়া প্রমাণীকরণ, কোনো API কী নয়):

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

সার্ভার চালু করুন:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

সার্ভার `http://localhost:8080` এ চালু হবে। আপনি দেখতে পাবেন:
```
Started McpServerApplication in X.XXX seconds
```

### ধাপ ২: সরাসরি ক্লায়েন্ট দিয়ে পরীক্ষা

**নতুন** একটি টার্মিনাল খুলে, সার্ভার চালু রেখে, ডিরেক্ট MCP ক্লায়েন্ট চালান:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

আপনি এমন আউটপুট পাবেন:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### ধাপ ৩: AI ক্লায়েন্ট দিয়ে পরীক্ষা

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

AI স্বয়ংক্রিয়ভাবে টুল ব্যবহার করছে দেখতে পাবেন:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### ধাপ ৪: MCP সার্ভার বন্ধ করা

পরীক্ষা শেষ হলে, AI ক্লায়েন্ট বন্ধ করতে তার টার্মিনালে `Ctrl+C` চাপুন। MCP সার্ভার চালু থাকবে যতক্ষণ না আপনি এটি বন্ধ করেন।  
সার্ভার বন্ধ করতে, যেখানে চালু করেছেন ঐ টার্মিনালে `Ctrl+C` চাপুন।

## কিভাবে সব একসঙ্গে কাজ করে

যখন আপনি AI কে জিজ্ঞাসা করবেন "5 + 3 কত?" তখন সম্পূর্ণ প্রবাহ হবে:

১. **আপনি** স্বাভাবিক ভাষায় AI কে প্রশ্ন করেন  
২. **AI** আপনার অনুরোধ বিশ্লেষণ করে বুঝে আপনি যোগ করতে চান  
৩. **AI** MCP সার্ভারে কল দেয়: `add(5.0, 3.0)`  
৪. **ক্যালকুলেটর সার্ভিস** গণনা করে: `5.0 + 3.0 = 8.0`  
৫. **ক্যালকুলেটর সার্ভিস** রিটার্ন করে: `"5.00 + 3.00 = 8.00"`  
৬. **AI** ফলাফল গ্রহণ করে একটি প্রাকৃতিক প্রতিক্রিয়া তৈরি করে  
৭. **আপনি** পান: "5 এবং 3 এর যোগফল 8"

## পরবর্তী ধাপ

আরও উদাহরণের জন্য দেখুন [অধ্যায় ০৪: ব্যবহারিক নমুনা](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**অস্বীকৃতি**:
এই নথিটি AI অনুবাদ পরিষেবা [Co-op Translator](https://github.com/Azure/co-op-translator) ব্যবহার করে অনূদিত হয়েছে। যদিও আমরা শুদ্ধতার জন্য চেষ্টা করি, অনুগ্রহ করে মনে রাখবেন যে স্বয়ংক্রিয় অনুবাদে ত্রুটি বা অসঙ্গতি থাকতে পারে। মূল নথিটি তার স্বভাষায় কর্তৃত্বপূর্ণ উৎস হিসেবে বিবেচিত হওয়া উচিত। গুরুত্বপূর্ণ তথ্যের জন্য পেশাদার মানব অনুবাদ সুপারিশ করা হয়। এই অনুবাদের ব্যবহারে প্রয়োজনীয় ভুল বোঝাবুঝি বা ভুল ব্যাখ্যার জন্য আমরা দায়বদ্ধ নই।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->