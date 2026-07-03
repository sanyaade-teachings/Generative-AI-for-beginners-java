# درس حساب MCP للمبتدئين

## جدول المحتويات

- [ما الذي ستتعلمه](#ما-الذي-ستتعلمه)
- [المتطلبات الأساسية](#المتطلبات-الأساسية)
- [فهم هيكل المشروع](#فهم-هيكل-المشروع)
- [شرح المكونات الأساسية](#شرح-المكونات-الأساسية)
  - [1. التطبيق الرئيسي](#1-التطبيق-الرئيسي)
  - [2. خدمة الآلة الحاسبة](#2-خدمة-الآلة-الحاسبة)
  - [3. عميل MCP المباشر](#3-عميل-mcp-المباشر)
  - [4. العميل المدعوم بالذكاء الاصطناعي](#4-العميل-المدعوم-بالذكاء-الاصطناعي)
- [تشغيل الأمثلة](#تشغيل-الأمثلة)
- [كيف تعمل معا](#كيف-تعمل-معا)
- [الخطوات المقبلة](#الخطوات-المقبلة)

## ما الذي ستتعلمه

يشرح هذا الدرس كيفية بناء خدمة آلة حاسبة باستخدام بروتوكول سياق النموذج (MCP). ستفهم:

- كيفية إنشاء خدمة يمكن للذكاء الاصطناعي استخدامها كأداة
- كيفية إعداد التواصل المباشر مع خدمات MCP
- كيف يمكن لنماذج الذكاء الاصطناعي اختيار الأدوات التي تستخدمها تلقائيًا
- الفرق بين مكالمات البروتوكول المباشرة والتفاعلات المدعومة بالذكاء الاصطناعي

## المتطلبات الأساسية

قبل البدء، تأكد من وجود:
- جافا 21 أو أعلى مثبتة
- مافن لإدارة التبعيات
- نشر نموذج Azure AI Foundry (قم بتوفيره باستخدام `azd up` — انظر [الفصل 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [واجهة أوامر Azure](https://learn.microsoft.com/cli/azure/install-azure-cli) مسجل دخول باستخدام الأمر `az login` (مصادقة بدون مفتاح)
- فهم أساسي لجافا وSpring Boot

## فهم هيكل المشروع

مشروع الآلة الحاسبة يحتوي على عدة ملفات مهمة:

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

هذه نقطة الدخول لخدمة الآلة الحاسبة الخاصة بنا. هو تطبيق Spring Boot قياسي مع إضافة واحدة خاصة:

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
- ينشئ `ToolCallbackProvider` لجعل طرق الآلة الحاسبة متاحة كأدوات MCP
- تعليمة `@Bean` تخبر Spring بإدارة هذا كمكون يمكن للأجزاء الأخرى استخدامه

### 2. خدمة الآلة الحاسبة

**الملف:** `CalculatorService.java`

هنا تقع كل العمليات الحسابية. كل طريقة مؤشرة بـ `@Tool` لجعلها متاحة عبر MCP:

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
    
    // المزيد من عمليات الحاسبة...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**الميزات الرئيسية:**

1. **تعليمة `@Tool`**: تخبر MCP أن هذه الطريقة يمكن استدعاؤها من قبل العملاء الخارجيين
2. **وصف واضح**: كل أداة لها وصف يساعد نماذج الذكاء الاصطناعي على فهم متى تستخدمها
3. **صيغة إرجاع متسقة**: كل العمليات ترجع سلاسل نصية مفهومة مثل "5.00 + 3.00 = 8.00"
4. **معالجة الأخطاء**: القسمة على الصفر والجذر التربيعي للأعداد السالبة ترجع رسائل خطأ

**العمليات المتاحة:**
- `add(a, b)` - يضيف عددين
- `subtract(a, b)` - يطرح الثاني من الأول
- `multiply(a, b)` - يضرب عددين
- `divide(a, b)` - يقسم الأول على الثاني (مع التحقق من الصفر)
- `power(base, exponent)` - يرفع الأساس إلى قوة الأس
- `squareRoot(number)` - يحسب الجذر التربيعي (مع التحقق من السالب)
- `modulus(a, b)` - يعيد باقي القسمة
- `absolute(number)` - يعيد القيمة المطلقة
- `help()` - يعيد معلومات عن كل العمليات

### 3. عميل MCP المباشر

**الملف:** `SDKClient.java`

هذا العميل يتحدث مباشرة إلى خادم MCP بدون استخدام الذكاء الاصطناعي. يستدعي وظائف الآلة الحاسبة المحددة يدويًا:

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
        
        // استدعاء دوال الحاسبة المحددة
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
1. **يتصل** بخادم الآلة الحاسبة على `http://localhost:8080` باستخدام نمط المنشئ
2. **يسرد** كل الأدوات المتاحة (وظائف الآلة الحاسبة)
3. **ينادي** الوظائف المحددة بالمعاملات الدقيقة
4. **يطبع** النتائج مباشرة

**ملاحظة:** هذا المثال يستخدم تبعية Spring AI 1.1.0-SNAPSHOT، التي قدمت نمط المنشئ لـ `WebFluxSseClientTransport`. إذا كنت تستخدم نسخة مستقرة أقدم، قد تحتاج إلى استخدام المنشئ المباشر بدلاً من ذلك.

**متى تستخدم هذا:** عندما تعرف بالضبط العملية الحسابية التي تريد إجراءها وترغب في نداءها برمجياً.

### 4. العميل المدعوم بالذكاء الاصطناعي

**الملف:** `LangChain4jClient.java`

هذا العميل يستخدم نموذج ذكاء اصطناعي (GPT-4o-mini) يمكنه اختيار أدوات الآلة الحاسبة التي يستخدمها تلقائيًا:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // إعداد نموذج الذكاء الاصطناعي (Azure AI Foundry، مصادقة بدون مفتاح عبر Microsoft Entra ID)
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

        // الاتصال بخادم الحاسبة MCP الخاص بنا
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // يعرض ما يفعله الذكاء الاصطناعي
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // منح الذكاء الاصطناعي حق الوصول إلى أدوات الحاسبة الخاصة بنا
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // إنشاء بوت ذكاء اصطناعي يمكنه استخدام حاسبتنا
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // الآن يمكننا أن نطلب من الذكاء الاصطناعي إجراء الحسابات باللغة الطبيعية
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**ما يقوم به هذا:**
1. **ينشئ** اتصالًا بنموذج ذكاء اصطناعي باستخدام مصادقة بدون مفتاح (Microsoft Entra ID)
2. **يوصل** الذكاء الاصطناعي بخادم MCP الخاص بالآلة الحاسبة
3. **يعطي** الذكاء الاصطناعي حق الوصول لكل أدوات الآلة الحاسبة
4. **يسمح** بطلبات لغة طبيعية مثل "احسب مجموع 24.5 و 17.3"

**الذكاء الاصطناعي تلقائياً:**
- يفهم أنك تريد جمع الأعداد
- يختار أداة `add`
- ينادي `add(24.5, 17.3)`
- يعيد النتيجة برد طبيعي

## تشغيل الأمثلة

### الخطوة 1: تشغيل خادم الآلة الحاسبة

أولاً، سجل دخولك وعيّن نقطة نهاية Azure AI Foundry (مطلوب للعميل الذكي — مصادقة بدون مفتاح، بدون مفتاح API):

**ويندوز:**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**لينكس/ماك:**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

شغل الخادم:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

سيبدأ الخادم على `http://localhost:8080`. يجب أن ترى:
```
Started McpServerApplication in X.XXX seconds
```

### الخطوة 2: اختبار مع العميل المباشر

في نافذة طرفية **جديدة** بينما الخادم يعمل، شغل عميل MCP المباشر:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

سترى ناتج مثل:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### الخطوة 3: اختبار مع العميل المدعوم بالذكاء الاصطناعي

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

سترى الذكاء الاصطناعي يستخدم الأدوات تلقائيًا:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### الخطوة 4: إيقاف خادم MCP

عند الانتهاء من الاختبار، يمكنك إيقاف العميل الذكي بالضغط على `Ctrl+C` في نافذته الطرفية. سيظل خادم MCP يعمل حتى توقفه.
لإيقاف الخادم، اضغط `Ctrl+C` في نافذة الطرفية التي يشغلها.

## كيف تعمل معا

إليك التدفق الكامل عندما تسأل الذكاء الاصطناعي "ما هو 5 + 3؟":

1. **أنت** تسأل الذكاء الاصطناعي بلغة طبيعية
2. **الذكاء الاصطناعي** يحلل طلبك ويدرك أنك تريد الجمع
3. **الذكاء الاصطناعي** ينادي خادم MCP: `add(5.0, 3.0)`
4. **خدمة الآلة الحاسبة** تنفذ: `5.0 + 3.0 = 8.0`
5. **خدمة الآلة الحاسبة** تعيد: `"5.00 + 3.00 = 8.00"`
6. **الذكاء الاصطناعي** يستلم النتيجة ويُنسق ردًا طبيعيًا
7. **أنت** تحصل على: "مجموع 5 و 3 هو 8"

## الخطوات المقبلة

لمزيد من الأمثلة، راجع [الفصل 04: عينات عملية](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**تنويه**:
تمت ترجمة هذا المستند باستخدام خدمة الترجمة بالذكاء الاصطناعي [Co-op Translator](https://github.com/Azure/co-op-translator). بينما نسعى للدقة، يرجى العلم أن الترجمات الآلية قد تحتوي على أخطاء أو عدم دقة. يجب اعتبار المستند الأصلي بلغته الأصلية المصدر الرسمي والمعتمد. للمعلومات الهامة، يُنصح بالاستعانة بترجمة بشرية محترفة. نحن غير مسؤولين عن أي سوء فهم أو تفسير ناتج عن استخدام هذه الترجمة.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->