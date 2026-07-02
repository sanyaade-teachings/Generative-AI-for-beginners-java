# دليل حساب MCP للمبتدئين

## فهرس المحتويات

- [ما ستتعلمه](#ما-ستتعلمه)
- [المتطلبات الأساسية](#المتطلبات-الأولية)
- [فهم هيكل المشروع](#فهم-هيكل-المشروع)
- [شرح المكونات الأساسية](#شرح-المكونات-الأساسية)
  - [1. التطبيق الرئيسي](#1-التطبيق-الرئيسي)
  - [2. خدمة الآلة الحاسبة](#2-خدمة-الآلة-الحاسبة)
  - [3. عميل MCP مباشر](#3-عميل-mcp-مباشر)
  - [4. عميل مدعوم بالذكاء الاصطناعي](#4-عميل-مدعوم-بالذكاء-الاصطناعي)
- [تشغيل الأمثلة](#تشغيل-الأمثلة)
- [كيف يعمل كل شيء معًا](#كيف-يعمل-كل-شيء-معًا)
- [الخطوات التالية](#الخطوات-التالية)

## ما ستتعلمه

يشرح هذا الدليل كيفية بناء خدمة آلة حاسبة باستخدام بروتوكول سياق النموذج (MCP). ستفهم:

- كيفية إنشاء خدمة يمكن للذكاء الاصطناعي استخدامها كأداة
- كيفية إعداد اتصال مباشر مع خدمات MCP
- كيف يمكن لنماذج الذكاء الاصطناعي اختيار الأدوات التي تستخدمها تلقائيًا
- الفرق بين المكالمات المباشرة للبروتوكول والتفاعلات المدعومة بالذكاء الاصطناعي

## المتطلبات الأساسية

قبل البدء، تأكد من توفر لديك:
- جافا 21 أو أحدث مثبتة
- Maven لإدارة التبعيات
- نشر نموذج Azure AI Foundry (وفّره باستخدام `azd up` — راجع [الفصل 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli)، مسجل الدخول بـ `az login` (مصادقة بدون مفتاح)
- فهم أساسي لجافا وSpring Boot

## فهم هيكل المشروع

يحتوي مشروع الآلة الحاسبة على عدة ملفات هامة:

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

## شرح المكونات الأساسية

### 1. التطبيق الرئيسي

**الملف:** `McpServerApplication.java`

هذه هي نقطة الدخول لخدمة الآلة الحاسبة الخاصة بنا. إنه تطبيق Spring Boot قياسي مع إضافة خاصة واحدة:

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

**ما يقوم به هذا:**
- يبدأ خادم ويب Spring Boot على المنفذ 8080
- ينشئ `ToolCallbackProvider` الذي يجعل طرق الآلة الحاسبة متاحة كأدوات MCP
- توضح التعليمة `@Bean` لـ Spring إدارة هذا كعنصر يمكن للأجزاء الأخرى استخدامه

### 2. خدمة الآلة الحاسبة

**الملف:** `CalculatorService.java`

هنا يحدث كل الحساب. كل طريقة موسومة بـ `@Tool` لجعلها متاحة عبر MCP:

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
    
    // المزيد من عمليات الآلة الحاسبة...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**الميزات الأساسية:**

1. **تعليمة `@Tool`**: تخبر MCP أن هذه الطريقة يمكن استدعاؤها بواسطة العملاء الخارجيين
2. **وصف واضح**: كل أداة لها وصف يساعد نماذج الذكاء الاصطناعي على فهم متى تستخدمها
3. **تنسيق إرجاع ثابت**: جميع العمليات تعيد سلاسل نصية مقروءة للبشر مثل "5.00 + 3.00 = 8.00"
4. **معالجة الأخطاء**: القسمة على صفر والجذر التربيعي للأعداد السالبة تعيد رسائل خطأ

**العمليات المتاحة:**
- `add(a, b)` - جمع رقمين
- `subtract(a, b)` - طرح الثاني من الأول
- `multiply(a, b)` - ضرب رقمين
- `divide(a, b)` - قسمة الأول على الثاني (مع التحقق من القسمة على الصفر)
- `power(base, exponent)` - رفع الأساس إلى قوة الأس
- `squareRoot(number)` - حساب الجذر التربيعي (مع التحقق من السالب)
- `modulus(a, b)` - إرجاع باقي القسمة
- `absolute(number)` - إرجاع القيمة المطلقة
- `help()` - إعطاء معلومات عن كل العمليات

### 3. عميل MCP مباشر

**الملف:** `SDKClient.java`

هذا العميل يتحدث مباشرة إلى خادم MCP دون استخدام الذكاء الاصطناعي. يستدعي وظائف الآلة الحاسبة المحددة يدويًا:

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
        
        // سرد الأدوات المتاحة
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // استدعاء دوال الآلة الحاسبة المحددة
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

**ما يقوم به هذا:**
1. **يتصل** بخادم الآلة الحاسبة على `http://localhost:8080` باستخدام نمط البناء
2. **يعرض قائمة** بجميع الأدوات المتاحة (وظائف الآلة الحاسبة لدينا)
3. **يستدعي** وظائف محددة بمعاملات دقيقة
4. **يطبع** النتائج مباشرة

**ملحوظة:** هذا المثال يستخدم تبعية Spring AI 1.1.0-SNAPSHOT، التي أدخلت نمط البناء لـ `WebFluxSseClientTransport`. إذا كنت تستخدم إصدارًا ثابتًا أقدم، قد تحتاج لاستخدام المُنشئ المباشر بدلاً من ذلك.

**متى تستخدم هذا:** عندما تعرف بالضبط الحساب الذي تريد إجراؤه وتريد استدعاءه برمجيًا.

### 4. عميل مدعوم بالذكاء الاصطناعي

**الملف:** `LangChain4jClient.java`

هذا العميل يستخدم نموذج ذكاء اصطناعي (GPT-4o-mini) يمكنه اختيار أدوات الآلة الحاسبة تلقائيًا:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // إعداد نموذج الذكاء الاصطناعي (Azure AI Foundry، المصادقة بدون مفتاح عبر Microsoft Entra ID)
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

        // الاتصال بخادم MCP للحاسبة الخاصة بنا
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // يعرض ما يفعله الذكاء الاصطناعي
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // منح الذكاء الاصطناعي الوصول إلى أدوات الحاسبة الخاصة بنا
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // إنشاء روبوت ذكاء اصطناعي يمكنه استخدام الحاسبة الخاصة بنا
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // الآن يمكننا طلب من الذكاء الاصطناعي إجراء الحسابات بلغة طبيعية
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**ما يقوم به هذا:**
1. **ينشئ** اتصال نموذج ذكاء اصطناعي باستخدام رمز GitHub الخاص بك
2. **يربط** الذكاء الاصطناعي بخادم MCP الخاص بنا للآلة الحاسبة
3. **يمنح** الذكاء الاصطناعي الوصول إلى كل أدوات الآلة الحاسبة
4. **يسمح** بطلبات اللغة الطبيعية مثل "احسب مجموع 24.5 و 17.3"

**يتم تلقائيًا من قبل الذكاء الاصطناعي:**
- يفهم أنك تريد جمع الأرقام
- يختار أداة `add`
- يستدعي `add(24.5, 17.3)`
- يعيد النتيجة برد طبيعي

## تشغيل الأمثلة

### الخطوة 1: تشغيل خادم الآلة الحاسبة

أولاً، سجّل الدخول واضبط نقطة نهاية Azure AI Foundry الخاصة بك (مطلوب لعميل الذكاء الاصطناعي — مصادقة بدون مفتاح، بدون مفتاح API):

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

شغّل الخادم:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

سيبدأ الخادم على `http://localhost:8080`. يجب أن ترى:
```
Started McpServerApplication in X.XXX seconds
```

### الخطوة 2: الاختبار مع العميل المباشر

في نافذة طرفية **جديدة** مع استمرار تشغيل الخادم، شغّل عميل MCP المباشر:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

سترى مخرجات مثل:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### الخطوة 3: الاختبار مع عميل الذكاء الاصطناعي

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

سترى الذكاء الاصطناعي يستخدم الأدوات تلقائيًا:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### الخطوة 4: إغلاق خادم MCP

عندما تنتهي من الاختبار، يمكنك إيقاف عميل الذكاء الاصطناعي بالضغط على `Ctrl+C` في نافذة الطرفية الخاصة به. سيستمر خادم MCP في العمل حتى توقفه.
لإيقاف الخادم، اضغط `Ctrl+C` في الطرفية التي يعمل بها.

## كيف يعمل كل شيء معًا

إليك التدفق الكامل عندما تسأل الذكاء الاصطناعي "كم حاصل 5 + 3؟":

1. **أنت** تسأل الذكاء الاصطناعي بلغة طبيعية
2. **الذكاء الاصطناعي** يحلل طلبك ويفهم أنك تريد الجمع
3. **الذكاء الاصطناعي** يستدعي خادم MCP: `add(5.0, 3.0)`
4. **خدمة الآلة الحاسبة** تنفذ: `5.0 + 3.0 = 8.0`
5. **خدمة الآلة الحاسبة** تعيد: `"5.00 + 3.00 = 8.00"`
6. **الذكاء الاصطناعي** يستلم النتيجة ويصيغ ردًا طبيعيًا
7. **أنت** تحصل على: "مجموع 5 و 3 هو 8"

## الخطوات التالية

لمزيد من الأمثلة، راجع [الفصل 04: عينات عملية](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**تنويه**:
تمت ترجمة هذا المستند باستخدام خدمة الترجمة بالذكاء الاصطناعي [Co-op Translator](https://github.com/Azure/co-op-translator). بينما نسعى للدقة، يرجى العلم أن الترجمات الآلية قد تحتوي على أخطاء أو عدم دقة. يجب اعتبار المستند الأصلي بلغته الأصلية المصدر الرسمي والمعتمد. للمعلومات الهامة، يُنصح بالاستعانة بترجمة بشرية محترفة. نحن غير مسؤولين عن أي سوء فهم أو تفسير ناتج عن استخدام هذه الترجمة.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->