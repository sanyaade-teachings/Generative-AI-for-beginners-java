# MCP کیلکولیٹر ٹیوٹوریل ابتدائیوں کے لیے

## فہرست مضامین

- [آپ کیا سیکھیں گے](#آپ-کیا-سیکھیں-گے)
- [ضروریات](#ضروریات)
- [پروجیکٹ اسٹرکچر کو سمجھنا](#پروجیکٹ-اسٹرکچر-کو-سمجھنا)
- [مرکزی اجزاء کی وضاحت](#مرکزی-اجزاء-کی-وضاحت)
  - [1. مین ایپلیکیشن](#1-مین-ایپلیکیشن)
  - [2. کیلکولیٹر سروس](#2-کیلکولیٹر-سروس)
  - [3. ڈائریکٹ MCP کلائنٹ](#3-ڈائریکٹ-mcp-کلائنٹ)
  - [4. AI سے چلنے والا کلائنٹ](#4-ai-سے-چلنے-والا-کلائنٹ)
- [مثالیں چلانا](#مثالیں-چلانا)
- [یہ سب کیسے ساتھ کام کرتے ہیں](#یہ-سب-کیسے-ساتھ-کام-کرتے-ہیں)
- [اگلے اقدامات](#اگلے-اقدامات)

## آپ کیا سیکھیں گے

یہ ٹیوٹوریل بتاتا ہے کہ کیسے ماڈل کانٹیکسٹ پروٹوکول (MCP) استعمال کرتے ہوئے ایک کیلکولیٹر سروس بنائی جائے۔ آپ کو سمجھ آئے گا:

- کیسے ایک ایسی سروس بنائیں جو AI ایک ٹول کی طرح استعمال کر سکے
- کیسے MCP خدمات کے ساتھ براہ راست مواصلت قائم کی جائے
- کیسے AI ماڈلز خودکار طور پر منتخب کرتے ہیں کہ کون سے ٹولز استعمال کرنے ہیں
- براہ راست پروٹوکول کالز اور AI مدد یافتہ تعاملات کے درمیان فرق

## ضروریات

شروع کرنے سے پہلے، یقینی بنائیں کہ آپ کے پاس ہے:
- جاوا 21 یا اس سے جدید ورژن انسٹالڈ ہے
- میون برائے انحصاری انتظام
- Azure AI Foundry ماڈل کی تعیناتی (اسے `azd up` کے ساتھ تیار کریں — دیکھیں [باب 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) جو `az login` سے سائن ان ہو (بغیر کلید کے توثیق)
- جاوا اور اسپرنگ بوٹ کی بنیادی سمجھ

## پروجیکٹ اسٹرکچر کو سمجھنا

کیلکولیٹر پروجیکٹ میں کئی اہم فائلیں شامل ہیں:

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

## مرکزی اجزاء کی وضاحت

### 1. مین ایپلیکیشن

**فائل:** `McpServerApplication.java`

یہ ہمارے کیلکولیٹر سروس کا داخلی نقطہ ہے۔ یہ ایک معیاری اسپرنگ بوٹ ایپلیکیشن ہے جس میں ایک خاص اضافہ موجود ہے:

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

**یہ کیا کرتا ہے:**
- 8080 پورٹ پر اسپرنگ بوٹ ویب سرور شروع کرتا ہے
- ایک `ToolCallbackProvider` بناتا ہے جو ہمارے کیلکولیٹر میتھڈز کو MCP ٹولز کے طور پر دستیاب کرتا ہے
- `@Bean` اینوٹیشن بتاتی ہے کہ اسپرنگ کو اسے ایک ایسا کمپونینٹ بنا کر منظم کرنا ہے جسے دوسرے حصے استعمال کر سکیں

### 2. کیلکولیٹر سروس

**فائل:** `CalculatorService.java`

یہاں تمام ریاضی کا حساب ہوتا ہے۔ ہر میتھڈ کو `@Tool` سے نشان زد کیا گیا ہے تاکہ یہ MCP کے ذریعے دستیاب ہو:

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
    
    // مزید کیلکولیٹر کی کارروائیاں...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**اہم خصوصیات:**

1. **`@Tool` اینوٹیشن**: یہ MCP کو بتاتا ہے کہ یہ میتھڈ بیرونی کلائنٹس سے کال کیا جا سکتا ہے
2. **واضح وضاحتیں**: ہر ٹول کی ایک تفصیل ہے جو AI ماڈلز کو سمجھنے میں مدد دیتی ہے کہ اسے کب استعمال کرنا ہے
3. **مستقل ریٹرن فارمٹ**: تمام آپریشنز قابل فہم اسٹرنگز لوٹاتے ہیں جیسے "5.00 + 3.00 = 8.00"
4. **ایرر ہینڈلنگ**: صفر سے تقسیم اور منفی جذر نکالنے پر ایرر میسجز لوٹاتے ہیں

**دستیاب عملیات:**
- `add(a, b)` - دو نمبرز کا جمع کرتا ہے
- `subtract(a, b)` - دوسرے نمبر کو پہلے سے منفی کرتا ہے
- `multiply(a, b)` - دو نمبرز کا ضرب دیتا ہے
- `divide(a, b)` - پہلے نمبر کو دوسرے سے تقسیم کرتا ہے (صفر کی جانچ کے ساتھ)
- `power(base, exponent)` - بیس کو پاور پر اٹھاتا ہے
- `squareRoot(number)` - جذر نکالتا ہے (منفی کی جانچ کے ساتھ)
- `modulus(a, b)` - تقسیم کا باقی دیتا ہے
- `absolute(number)` - مطلق قدر لوٹاتا ہے
- `help()` - تمام عملیات کی معلومات لوٹاتا ہے

### 3. ڈائریکٹ MCP کلائنٹ

**فائل:** `SDKClient.java`

یہ کلائنٹ AI استعمال کیے بغیر براہ راست MCP سرور سے بات کرتا ہے۔ یہ مخصوص کیلکولیٹر فنکشنز کو دستی طور پر کال کرتا ہے:

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
        
        // دستیاب آلات کی فہرست
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // مخصوص کیلکولیٹر کے افعال کو کال کریں
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
1. `http://localhost:8080` پر کیلکولیٹر سرور سے کنیکٹ کرتا ہے، بلڈر پیٹرن کا استعمال کرتے ہوئے
2. دستیاب تمام ٹولز (ہمارے کیلکولیٹر فنکشنز) کی فہرست بناتا ہے
3. مخصوص فنکشنز کو صحیح پیرا میٹرز کے ساتھ کال کرتا ہے
4. نتائج کو براہ راست پرنٹ کرتا ہے

**نوٹ:** یہ مثال اسپرنگ AI 1.1.0-SNAPSHOT انحصار استعمال کرتی ہے، جس نے `WebFluxSseClientTransport` کے لیے بلڈر پیٹرن متعارف کروایا ہے۔ اگر آپ پرانا مستحکم ورژن استعمال کر رہے ہیں، تو آپ کو براہ راست کنسٹرکٹر استعمال کرنا پڑ سکتا ہے۔

**کب استعمال کریں:** جب آپ کو معلوم ہو کہ بالکل کون سا حساب کرنا ہے اور اسے پروگرام کے ذریعے کال کرنا ہو۔

### 4. AI سے چلنے والا کلائنٹ

**فائل:** `LangChain4jClient.java`

یہ کلائنٹ ایک AI ماڈل (GPT-4o-mini) استعمال کرتا ہے جو خودکار طور پر منتخب کر سکتا ہے کہ کون سے کیلکولیٹر ٹولز استعمال کرنے ہیں:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // AI ماڈل کو سیٹ اپ کریں (Azure AI Foundry، Microsoft Entra ID کے ذریعے بغیر کلید کی تصدیق)
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

        // ہمارے کیلکولیٹر MCP سرور سے کنکٹ کریں
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

        // ایک AI بوٹ بنائیں جو ہمارے کیلکولیٹر کو استعمال کر سکے
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
1. آپ کے GitHub ٹوکن کے ذریعے AI ماڈل کنیکشن بناتا ہے
2. AI کو ہمارے کیلکولیٹر MCP سرور سے کنیکٹ کرتا ہے
3. AI کو ہمارے تمام کیلکولیٹر ٹولز تک رسائی دیتا ہے
4. قدرتی زبان میں درخواستیں جیسے "24.5 اور 17.3 کا مجموعہ معلوم کریں" کی اجازت دیتا ہے

**AI خودکار طور پر:**
- سمجھتا ہے کہ آپ عدد جمع کرنا چاہتے ہیں
- `add` ٹول کا انتخاب کرتا ہے
- `add(24.5, 17.3)` کال کرتا ہے
- قدرتی جواب میں نتیجہ لوٹاتا ہے

## مثالیں چلانا

### مرحلہ 1: کیلکولیٹر سرور شروع کریں

سب سے پہلے، سائن ان کریں اور اپنا Azure AI Foundry اینڈپوائنٹ سیٹ کریں (AI کلائنٹ کے لیے ضروری — بغیر API کلید کے توثیق):

**ونڈوز:**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**لینکس/macOS:**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

سرور شروع کریں:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

سرور `http://localhost:8080` پر شروع ہو جائے گا۔ آپ دیکھیں گے:
```
Started McpServerApplication in X.XXX seconds
```

### مرحلہ 2: ڈائریکٹ کلائنٹ کے ساتھ ٹیسٹ کریں

سرور چل رہا ہو تو ایک **نیا** ٹرمینل کھولیں اور ڈائریکٹ MCP کلائنٹ چلائیں:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

آپ کو اس طرح کا آؤٹ پٹ نظر آئے گا:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### مرحلہ 3: AI کلائنٹ کے ساتھ ٹیسٹ کریں

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

AI خودکار طور پر ٹولز استعمال کرتے ہوئے دیکھا جائے گا:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### مرحلہ 4: MCP سرور بند کریں

جب آپ ٹیسٹنگ ختم کر لیں، تو AI کلائنٹ کو اپنے ٹرمینل میں `Ctrl+C` دبا کر بند کریں۔ MCP سرور چلتا رہے گا جب تک آپ اسے روک نہ دیں۔
سرور روکنے کے لیے، وہ ٹرمینل کھولیں جہاں سرور چل رہا ہے اور `Ctrl+C` دبائیں۔

## یہ سب کیسے ساتھ کام کرتے ہیں

جب آپ AI سے پوچھیں "5 + 3 کیا ہے؟"، تو پورا عمل یہ ہے:

1. **آپ** AI کو قدرتی زبان میں سوال کرتے ہیں
2. **AI** آپ کی درخواست کا تجزیہ کرتا ہے اور سمجھتا ہے کہ آپ جمع چاہتے ہیں
3. **AI** MCP سرور کو کال کرتا ہے: `add(5.0, 3.0)`
4. **کیلکولیٹر سروس** کرتا ہے: `5.0 + 3.0 = 8.0`
5. **کیلکولیٹر سروس** لوٹاتا ہے: `"5.00 + 3.00 = 8.00"`
6. **AI** نتیجہ وصول کر کے قدرتی جواب تیار کرتا ہے
7. **آپ** کو ملتا ہے: "5 اور 3 کا مجموعہ 8 ہے"

## اگلے اقدامات

مزید مثالوں کے لیے، دیکھیں [باب 04: عملی نمونے](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ڈس کلیمر**:
یہ دستاویز AI ترجمہ سروس [Co-op Translator](https://github.com/Azure/co-op-translator) کے ذریعے ترجمہ کی گئی ہے۔ جبکہ ہم درستگی کے لیے کوشاں ہیں، براہ کرم اس بات سے آگاہ رہیں کہ خودکار ترجمے میں غلطیاں یا عدم درستیاں ہو سکتی ہیں۔ اصل دستاویز اپنے مادری زبان میں مستند ماخذ سمجھی جائے گی۔ حساس معلومات کے لیے پیشہ ور انسانی ترجمہ کی سفارش کی جاتی ہے۔ اس ترجمے کے استعمال سے پیدا ہونے والی کسی بھی غلط فہمی یا غلط تشریح کی ذمہ داری ہم قبول نہیں کرتے۔
<!-- CO-OP TRANSLATOR DISCLAIMER END -->