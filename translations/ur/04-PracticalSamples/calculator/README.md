# MCP کیلکولیٹر ٹیوٹوریل برائے مبتدی

## فہرست مضامین

- [آپ کیا سیکھیں گے](#آپ-کیا-سیکھیں-گے)
- [ضروریات](#ضروریات)
- [پروجیکٹ کی ساخت کو سمجھنا](#پروجیکٹ-کی-ساخت-کو-سمجھنا)
- [اہم اجزاء کی وضاحت](#اہم-اجزاء-کی-وضاحت)
  - [1. مین ایپلیکیشن](#1-مین-ایپلیکیشن)
  - [2. کیلکولیٹر سروس](#2-کیلکولیٹر-سروس)
  - [3. ڈائریکٹ MCP کلائنٹ](#3-ڈائریکٹ-mcp-کلائنٹ)
  - [4. AI سے مربوط کلائنٹ](#4-ai-سے-مربوط-کلائنٹ)
- [مثالیں چلانا](#مثالیں-چلانا)
- [یہ سب کیسے کام کرتا ہے](#یہ-سب-کیسے-کام-کرتا-ہے)
- [اگلے اقدامات](#اگلے-اقدامات)

## آپ کیا سیکھیں گے

یہ ٹیوٹوریل بتاتا ہے کہ کیسے ماڈل کانٹیکسٹ پروٹوکول (MCP) کا استعمال کرتے ہوئے کیلکولیٹر سروس بنائی جائے۔ آپ سمجھیں گے:

- کیسے ایسی سروس بنائیں جو AI ایک ٹول کے طور پر استعمال کر سکتا ہے
- MCP سروسز کے ساتھ براہ راست مواصلت کس طرح قائم کی جائے
- AI ماڈلز خودکار طور پر کون سے ٹول استعمال کریں منتخب کر سکتے ہیں
- براہ راست پروٹوکول کالز اور AI کی معاونت والی انٹریکشنز میں فرق

## ضروریات

شروع کرنے سے پہلے، یہ یقینی بنائیں کہ آپ کے پاس ہے:
- Java 21 یا اس سے اوپر ورژن انسٹال شدہ
- Maven برائے انحصار کے انتظام کے لئے
- Azure AI Foundry ماڈل کی تعیناتی (اسے `azd up` کے ذریعے فراہم کریں — ملاحظہ کریں [باب 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli)، `az login` سے سائن ان کیا ہوا (بغیر کلید کے توثیق)
- Java اور Spring Boot کی بنیادی سمجھ

## پروجیکٹ کی ساخت کو سمجھنا

کیلکولیٹر پروجیکٹ میں کئی اہم فائلیں ہیں:

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

## اہم اجزاء کی وضاحت

### 1. مین ایپلیکیشن

**فائل:** `McpServerApplication.java`

یہ ہمارے کیلکولیٹر سروس کا انٹری پوائنٹ ہے۔ یہ ایک عام Spring Boot ایپلیکیشن ہے جس میں ایک خاص اضافہ کیا گیا ہے:

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

**یہ کیا کرتی ہے:**
- پورٹ 8080 پر Spring Boot ویب سرور شروع کرتی ہے
- `ToolCallbackProvider` بناتی ہے جو ہماری کیلکولیٹر طریقوں کو MCP ٹول کے طور پر دستیاب بناتا ہے
- `@Bean` تشریح Spring کو بتاتی ہے کہ اسے ایک کمپونینٹ کے طور پر منظم کرے جو دیگر حصے استعمال کرسکیں

### 2. کیلکولیٹر سروس

**فائل:** `CalculatorService.java`

یہاں تمام ریاضیاتی عملیات انجام دی جاتی ہیں۔ ہر طریقہ `@Tool` سے نشان زد ہے تاکہ MCP کے ذریعے دستیاب ہو:

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
    
    // مزید کیلکولیٹر کے عمل...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**اہم خصوصیات:**

1. **`@Tool` تشریح**: MCP کو بتاتی ہے کہ یہ طریقہ بیرونی کلائنٹس کال کر سکتے ہیں  
2. **واضح وضاحتیں**: ہر ٹول کی تفصیل ہوتی ہے جو AI ماڈلز کو سمجھنے میں مدد دیتی ہے کہ کب اسے استعمال کرنا ہے  
3. **ہم آہنگ ریٹرن فارمیٹ**: تمام عملیات ایسے انسانی پڑھنے کے قابل سٹرنگز لوٹاتے ہیں جیسے "5.00 + 3.00 = 8.00"  
4. **خرابی سے نمٹنا**: صفر سے تقسیم اور منفی مربع جذر کو خرابی کے پیغامات دئیے جاتے ہیں  

**دستیاب عملیات:**
- `add(a, b)` - دو عدد جمع کرتا ہے  
- `subtract(a, b)` - دوسرا عدد پہلے سے گھٹاتا ہے  
- `multiply(a, b)` - دو عدد ضرب دیتا ہے  
- `divide(a, b)` - پہلے کو دوسرے سے تقسیم کرتا ہے (صفر چیک کے ساتھ)  
- `power(base, exponent)` - بیس کو طاقت پر اٹھاتا ہے  
- `squareRoot(number)` - مربع جذر نکالتا ہے (منفی چیک کے ساتھ)  
- `modulus(a, b)` - تقسیم کا باقی دیتا ہے  
- `absolute(number)` - مکمل قدر دیتا ہے  
- `help()` - تمام عملیات کے متعلق معلومات دیتا ہے

### 3. ڈائریکٹ MCP کلائنٹ

**فائل:** `SDKClient.java`

یہ کلائنٹ بغیر AI کے MCP سرور سے براہ راست بات کرتا ہے۔ یہ مخصوص کیلکولیٹر فنکشنز کو دستی طور پر کال کرتا ہے:

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
        
        // دستیاب آلات کی فہرست بنائیں
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // مخصوص کیلکولیٹر فنکشنز کو کال کریں
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

**یہ کیا کرتا ہے:**
1. بلڈر پیٹرن کا استعمال کرتے ہوئے `http://localhost:8080` پر کیلکولیٹر سرور سے **متصل** ہوتا ہے  
2. تمام دستیاب ٹولز (ہمارے کیلکولیٹر فنکشنز) **فہرست** کرتا ہے  
3. مخصوص فنکشنز کو درست پیرامیٹرز کے ساتھ **کال** کرتا ہے  
4. نتائج کو براہ راست **پرنٹ** کرتا ہے  

**نوٹ:** اس مثال میں Spring AI 1.1.0-SNAPSHOT ڈیپینڈینسی استعمال ہوئی ہے، جس نے `WebFluxSseClientTransport` کے لئے بلڈر پیٹرن متعارف کرایا ہے۔ اگر آپ پرانا مستحکم ورژن استعمال کر رہے ہیں تو آپ کو براہ راست کنسٹرکٹر استعمال کرنا پڑ سکتا ہے۔

**یہ کب استعمال کریں:** جب آپ کو بالکل معلوم ہو کہ کون سا حساب کرنا ہے اور اسے پروگرام کے ذریعے کال کرنا ہو۔

### 4. AI سے مربوط کلائنٹ

**فائل:** `LangChain4jClient.java`

یہ کلائنٹ AI ماڈل (GPT-4o-mini) استعمال کرتا ہے جو خودکار طور پر منتخب کر سکتا ہے کہ کون سے کیلکولیٹر ٹولز استعمال کرنا ہیں:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // اے آئی ماڈل مرتب کریں (Azure AI Foundry، Microsoft Entra ID کے ذریعے بغیر کلید کی تصدیق)
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

        // ہمارے کیلکولیٹر MCP سرور سے جڑیں
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // دکھاتا ہے کہ AI کیا کر رہا ہے
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // AI کو ہمارے کیلکولیٹر ٹولز تک رسائی دیں
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // ایک AI بوٹ بنائیں جو ہمارے کیلکولیٹر کا استعمال کر سکے
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // اب ہم AI سے قدرتی زبان میں حساب کتاب کرنے کو کہہ سکتے ہیں
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**یہ کیا کرتا ہے:**
1. کلید کے بغیر تصدیق (Microsoft Entra ID) کے ساتھ AI ماڈل کنکشن **بناتا ہے**  
2. AI کو ہمارے کیلکولیٹر MCP سرور سے **متصل** کرتا ہے  
3. AI کو ہمارے تمام کیلکولیٹر ٹولز تک رسائی **دیتا ہے**  
4. قدرتی زبان کی درخواستوں کی اجازت دیتا ہے جیسے "24.5 اور 17.3 کا مجموعہ نکالیں"  

**AI خودکار طور پر:**
- سمجھتا ہے کہ آپ نمبر جمع کرنا چاہتے ہیں  
- `add` ٹول منتخب کرتا ہے  
- `add(24.5, 17.3)` کال کرتا ہے  
- نتیجہ قدرتی ردعمل میں واپس دیتا ہے

## مثالیں چلانا

### مرحلہ 1: کیلکولیٹر سرور شروع کریں

سب سے پہلے، سائن ان کریں اور اپنا Azure AI Foundry اینڈپوائنٹ سیٹ کریں (AI کلائنٹ کے لیے ضروری — کلید کے بغیر توثیق):

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

سرور شروع کریں:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

سرور `http://localhost:8080` پر شروع ہو جائے گا۔ آپ کو یہ دیکھنا چاہیے:
```
Started McpServerApplication in X.XXX seconds
```

### مرحلہ 2: ڈائریکٹ کلائنٹ سے ٹیسٹ کریں

ایک **نئے** ٹرمینل میں، جب سرور چل رہا ہو، ڈائریکٹ MCP کلائنٹ چلائیں:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

آپ کو ایسی آؤٹ پٹ نظر آئے گی:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### مرحلہ 3: AI کلائنٹ سے ٹیسٹ کریں

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

آپ دیکھیں گے کہ AI خودکار طور پر ٹولز استعمال کر رہا ہے:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### مرحلہ 4: MCP سرور بند کریں

جب آپ ٹیسٹ مکمل کر لیں، تو AI کلائنٹ کو اپنے ٹرمینل میں `Ctrl+C` دباکر بند کر سکتے ہیں۔ MCP سرور تب تک چلتا رہے گا جب تک آپ اسے خود بند نہ کریں۔

سرور بند کرنے کے لئے، ٹرمینل میں جہاں چل رہا ہے، `Ctrl+C` دبائیں۔

## یہ سب کیسے کام کرتا ہے

یہاں مکمل فلور ہے جب آپ AI سے پوچھتے ہیں "5 + 3 کیا ہے؟":

1. **آپ** AI سے قدرتی زبان میں سوال کرتے ہیں  
2. **AI** آپ کی درخواست کا تجزیہ کرتا ہے اور سمجھتا ہے کہ آپ جمع چاہتے ہیں  
3. **AI** MCP سرور کو کال کرتا ہے: `add(5.0, 3.0)`  
4. **کیلکولیٹر سروس** عمل انجام دیتا ہے: `5.0 + 3.0 = 8.0`  
5. **کیلکولیٹر سروس** واپس کرتا ہے: `"5.00 + 3.00 = 8.00"`  
6. **AI** نتیجہ وصول کر کے قدرتی جواب تیار کرتا ہے  
7. **آپ** کو ملتا ہے: "5 اور 3 کا مجموعہ 8 ہے"

## اگلے اقدامات

مزید مثالوں کے لئے، ملاحظہ کریں [باب 04: عملی نمونے](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ڈس کلیمر**:
یہ دستاویز AI ترجمہ سروس [Co-op Translator](https://github.com/Azure/co-op-translator) کے ذریعے ترجمہ کی گئی ہے۔ جبکہ ہم درستگی کے لیے کوشاں ہیں، براہ کرم اس بات سے آگاہ رہیں کہ خودکار ترجمے میں غلطیاں یا عدم درستیاں ہو سکتی ہیں۔ اصل دستاویز اپنے مادری زبان میں مستند ماخذ سمجھی جائے گی۔ حساس معلومات کے لیے پیشہ ور انسانی ترجمہ کی سفارش کی جاتی ہے۔ اس ترجمے کے استعمال سے پیدا ہونے والی کسی بھی غلط فہمی یا غلط تشریح کی ذمہ داری ہم قبول نہیں کرتے۔
<!-- CO-OP TRANSLATOR DISCLAIMER END -->