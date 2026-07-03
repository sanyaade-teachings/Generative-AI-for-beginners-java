# MCP ক্যালকুলেটর টিউটোরিয়াল শুরু করার জন্য

## বিষয়সূচী

- [আপনি কী শিখবেন](#আপনি-কী-শিখবেন)
- [পূর্বশর্ত](#পূর্বশর্ত)
- [প্রকল্পের কাঠামো বুঝুন](#প্রকল্পের-কাঠামো-বুঝুন)
- [কোর উপাদান ব্যাখ্যা](#কোর-উপাদান-ব্যাখ্যা)
  - [১। মেইন অ্যাপ্লিকেশন](#১।-মেইন-অ্যাপ্লিকেশন)
  - [২। ক্যালকুলেটর সার্ভিস](#২।-ক্যালকুলেটর-সার্ভিস)
  - [৩। ডাইরেক্ট MCP ক্লায়েন্ট](#৩।-ডাইরেক্ট-mcp-ক্লায়েন্ট)
  - [৪। AI-পাওয়ার্ড ক্লায়েন্ট](#৪।-ai-পাওয়ার্ড-ক্লায়েন্ট)
- [উদাহরণগুলি চালানো](#উদাহরণগুলি-চালানো)
- [সবমিলিয়ে কীভাবে কাজ করে](#সবমিলিয়ে-কীভাবে-কাজ-করে)
- [পরবর্তী ধাপসমূহ](#পরবর্তী-ধাপসমূহ)

## আপনি কী শিখবেন

এই টিউটোরিয়ালে ব্যাখ্যা করা হবে কীভাবে Model Context Protocol (MCP) ব্যবহার করে একটি ক্যালকুলেটর সার্ভিস তৈরি করবেন। আপনি বুঝবেন:

- কীভাবে একটি সার্ভিস তৈরি করবেন যা AI কে একটি টুল হিসেবে ব্যবহার করতে দেয়
- MCP সার্ভিসের সাথে সরাসরি যোগাযোগ কিভাবে সেটআপ করবেন
- কীভাবে AI মডেলগুলি স্বয়ংক্রিয়ভাবে কোন টুলগুলো ব্যবহার করতে হবে তা নির্বাচন করে
- সরাসরি প্রোটোকল কল এবং AI-সহায়ক ইন্টারঅ্যাকশনের মধ্যে পার্থক্য

## পূর্বশর্ত

শুরু করার আগে নিশ্চিত করুন আপনার কাছে আছে:
- Java ২১ বা তার উপরে ইন্সটল করা
- Maven ডিপেন্ডেন্সি ম্যানেজমেন্টের জন্য
- একটি Azure AI Foundry মডেল ডিপ্লয়মেন্ট (এটি প্রোভিশন করতে `azd up` ব্যবহার করুন — দেখুন [Chapter 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), `az login` এর মাধ্যমে সাইন ইন (কীলেস অথ)
- Java এবং Spring Boot এর মৌলিক জ্ঞান

## প্রকল্পের কাঠামো বুঝুন

ক্যালকুলেটর প্রকল্পে বেশ কয়েকটি গুরুত্বপূর্ণ ফাইল রয়েছে:

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

## কোর উপাদান ব্যাখ্যা

### ১। মেইন অ্যাপ্লিকেশন

**ফাইল:** `McpServerApplication.java`

এটি আমাদের ক্যালকুলেটর সার্ভিসের এন্ট্রি পয়েন্ট। এটি একটি স্ট্যান্ডার্ড Spring Boot অ্যাপ্লিকেশন, তবে একটি বিশেষ সংযোজন সহ:

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

**এটি কী করে:**
- পোর্ট ৮০৮০ এ একটি Spring Boot ওয়েব সার্ভার শুরু করে
- একটি `ToolCallbackProvider` তৈরি করে যা আমাদের ক্যালকুলেটর মেথডগুলোকে MCP টুল হিসেবে উপলব্ধ করে
- `@Bean` অ্যানোটেশন Spring কে বলে এটি এমন একটি কম্পোনেন্ট হিসেবে পরিচালনা করতে যা অন্য অংশগুলো ব্যবহার করতে পারে

### ২। ক্যালকুলেটর সার্ভিস

**ফাইল:** `CalculatorService.java`

এখানেই সব গাণিতিক কাজ হয়। প্রতিটি মেথড `@Tool` দিয়ে চিহ্নিত, যাতে MCP এর মাধ্যমে এটি উপলব্ধ হয়:

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

**মুখ্য বৈশিষ্ট্য:**

১. **`@Tool` অ্যানোটেশন**: MCP কে বলে যে এই মেথডটি বাহ্যিক ক্লায়েন্ট দ্বারা কল করা যাবে
২. **স্পষ্ট বর্ণনা**: প্রতিটি টুলের একটি বর্ণনা আছে যা AI মডেলকে বুঝতে সাহায্য করে কখন এটি ব্যবহার করতে হবে
৩. **সঙ্গতিপূর্ণ রিটার্ন ফরম্যাট**: সব অপারেশন হিউম্যান-রিডেবল স্ট্রিং রিটার্ন করে যেমন "5.00 + 3.00 = 8.00"
৪. **ত্রুটিমুক্ত হ্যান্ডলিং**: শূন্য দিয়ে ভাগ করা এবং নেতিবাচক বর্গমূলের ক্ষেত্রে ত্রুটি মেসেজ রিটার্ন করে

**উপলব্ধ অপারেশনসমূহ:**
- `add(a, b)` - দুইটি সংখ্যার যোগফল
- `subtract(a, b)` - দ্বিতীয় সংখ্যাটি থেকে প্রথম সংখ্যাটি বিয়োগ
- `multiply(a, b)` - দুইটি সংখ্যার গুণফল
- `divide(a, b)` - প্রথম সংখ্যাকে দ্বিতীয় দ্বারা ভাগ (শূন্য পরীক্ষা সহ)
- `power(base, exponent)` - বেসকে পাওয়ারে উত্তোলন
- `squareRoot(number)` - বর্গমূল নির্ণয় (নেতিবাচক পরীক্ষা সহ)
- `modulus(a, b)` - ভাগশেষ প্রদান
- `absolute(number)` - পরম মান প্রদান
- `help()` - সব অপারেশন সংক্রান্ত তথ্য প্রদান

### ৩। ডাইরেক্ট MCP ক্লায়েন্ট

**ফাইল:** `SDKClient.java`

এই ক্লায়েন্ট সরাসরি MCP সার্ভার এর সাথে যোগাযোগ করে AI ছাড়া। এটি ম্যানুয়ালি নির্দিষ্ট ক্যালকুলেটর ফাংশনগুলো কল করে:

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
        
        // উপলব্ধ সরঞ্জামগুলি তালিকাভুক্ত করুন
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // নির্দিষ্ট ক্যালকুলেটর ফাংশনগুলি কল করুন
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

**এটি কী করে:**
১. বিল্ডার প্যাটার্ন ব্যবহার করে `http://localhost:8080` এ ক্যালকুলেটর সার্ভারের সাথে কানেক্ট করে
২. সব উপলব্ধ টুলের তালিকা নেয় (আমাদের ক্যালকুলেটর ফাংশনসমূহ)
৩. সুনির্দিষ্ট প্যারামিটার সহ নির্দিষ্ট ফাংশনগুলো কল করে
৪. সরাসরি ফলাফল প্রিন্ট করে

**দ্রষ্টব্য:** এই উদাহরণে Spring AI 1.1.0-SNAPSHOT ডিপেন্ডেন্সি ব্যবহার করা হয়েছে, যা `WebFluxSseClientTransport` এর জন্য বিল্ডার প্যাটার্ন চালু করেছে। আপনি যদি কোনও পুরানো স্টেবল ভার্সন ব্যবহার করেন, তাহলে সরাসরি কনস্ট্রাক্টর ব্যবহার করতে হতে পারে।

**কখন ব্যবহার করবেন:** যখন আপনি জানেন ঠিক কোন হিসাব করতে চান এবং প্রোগ্রাম্যাটিক্যালি কল করতে চান।

### ৪। AI-পাওয়ার্ড ক্লায়েন্ট

**ফাইল:** `LangChain4jClient.java`

এই ক্লায়েন্ট একটি AI মডেল (GPT-4o-mini) ব্যবহার করে যা স্বয়ংক্রিয়ভাবে সিদ্ধান্ত নেয় কোন ক্যালকুলেটর টুল গুলো ব্যবহার করবে:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // এআই মডেল সেট আপ করুন (Azure AI Foundry, Microsoft Entra ID এর মাধ্যমে কীলেস অথেনটিকেশন)
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

        // আমাদের ক্যালকুলেটর MCP সার্ভারের সাথে সংযুক্ত হন
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // কী করছে এআই তা দেখায়
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // এআই কে আমাদের ক্যালকুলেটর টুলস অ্যাক্সেস দিন
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // একটি এআই বট তৈরি করুন যা আমাদের ক্যালকুলেটর ব্যবহার করতে পারে
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // এখন আমরা এআই কে স্বাভাবিক ভাষায় গণনা করতে বলতে পারি
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**এটি কী করে:**
১. আপনার GitHub টোকেন ব্যবহার করে একটি AI মডেল সংযোগ তৈরি করে
২. AI কে আমাদের ক্যালকুলেটর MCP সার্ভারে কানেক্ট করে
৩. AI কে আমাদের সব ক্যালকুলেটর টুলে অ্যাক্সেস দেয়
৪. "24.5 এবং 17.3 এর যোগফল হিসাব করুন" এর মতো প্রাকৃতিক ভাষা অনুরোধ গ্রহণ করে

**AI স্বয়ংক্রিয়ভাবে:**
- বুঝে আপনি সংখ্যাগুলোর যোগ চান
- `add` টুল নির্বাচন করে
- `add(24.5, 17.3)` কল করে
- ফলাফল প্রাকৃতিক রেসপন্সে ফিরিয়ে দেয়

## উদাহরণগুলি চালানো

### ধাপ ১: ক্যালকুলেটর সার্ভার শুরু করুন

প্রথমে, সাইন ইন করুন এবং আপনার Azure AI Foundry এন্ডপয়েন্ট সেট করুন (AI ক্লায়েন্টের জন্য প্রয়োজন — কীলেস অথ, API কী নয়):

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

সার্ভার শুরু করুন:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

সার্ভার `http://localhost:8080` এ শুরু হবে। আপনি দেখতে পাবেন:
```
Started McpServerApplication in X.XXX seconds
```

### ধাপ ২: ডাইরেক্ট ক্লায়েন্ট দিয়ে টেস্ট করুন

একটি **নতুন** টার্মিনালে যেখানে সার্ভার চলছে, ডাইরেক্ট MCP ক্লায়েন্ট চালান:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

আপনি নিম্নরূপ আউটপুট দেখতে পাবেন:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### ধাপ ৩: AI ক্লায়েন্ট দিয়ে টেস্ট করুন

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

আপনি দেখতে পাবেন AI স্বয়ংক্রিয়ভাবে টুল ব্যবহারের মাধ্যমে কাজ করছে:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### ধাপ ৪: MCP সার্ভার বন্ধ করুন

পরীক্ষা শেষ হলে, AI ক্লায়েন্টের টার্মিনালে `Ctrl+C` চাপ দিয়ে বন্ধ করতে পারবেন। MCP সার্ভার তখন চলতেই থাকবে যতক্ষণ না আপনি নিজে এটি বন্ধ করেন।
সার্ভার বন্ধ করতে, যেই টার্মিনালে চলছে সেখানে `Ctrl+C` চাপ দিন।

## সবমিলিয়ে কীভাবে কাজ করে

এখানে সম্পূর্ণ প্রবাহ যখন আপনি AI কে জিজ্ঞাসা করেন "5 + 3 কত?":

১. **আপনি** প্রাকৃতিক ভাষায় AI কে প্রশ্ন করেন  
২. **AI** আপনার অনুরোধ বিশ্লেষণ করে বুঝে আপনি যোগফল চাচ্ছেন  
৩. **AI** MCP সার্ভারে কল করে: `add(5.0, 3.0)`  
৪. **ক্যালকুলেটর সার্ভিস** গাণিতিক কাজ করে: `5.0 + 3.0 = 8.0`  
৫. **ক্যালকুলেটর সার্ভিস** রিটার্ন করে: `"5.00 + 3.00 = 8.00"`  
৬. **AI** ফলাফল পেয়ে একটি প্রাকৃতিক রেসপন্স গঠন করে  
৭. **আপনি** পান: "5 এবং 3 এর যোগফল হল 8"

## পরবর্তী ধাপসমূহ

আরও উদাহরণের জন্য দেখুন [Chapter 04: Practical samples](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**অস্বীকৃতি**:
এই নথিটি AI অনুবাদ পরিষেবা [Co-op Translator](https://github.com/Azure/co-op-translator) ব্যবহার করে অনূদিত হয়েছে। যদিও আমরা শুদ্ধতার জন্য চেষ্টা করি, অনুগ্রহ করে মনে রাখবেন যে স্বয়ংক্রিয় অনুবাদে ত্রুটি বা অসঙ্গতি থাকতে পারে। মূল নথিটি তার স্বভাষায় কর্তৃত্বপূর্ণ উৎস হিসেবে বিবেচিত হওয়া উচিত। গুরুত্বপূর্ণ তথ্যের জন্য পেশাদার মানব অনুবাদ সুপারিশ করা হয়। এই অনুবাদের ব্যবহারে প্রয়োজনীয় ভুল বোঝাবুঝি বা ভুল ব্যাখ্যার জন্য আমরা দায়বদ্ধ নই।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->