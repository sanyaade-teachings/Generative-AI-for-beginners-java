# آموزش ماشین‌حساب MCP برای مبتدی‌ها

## فهرست مطالب

- [آنچه یاد خواهید گرفت](#آنچه-یاد-خواهید-گرفت)
- [پیش‌نیازها](#پیش‌نیازها)
- [درک ساختار پروژه](#درک-ساختار-پروژه)
- [توضیح اجزای اصلی](#توضیح-اجزای-اصلی)
  - [۱. برنامه اصلی](#۱-برنامه-اصلی)
  - [۲. سرویس ماشین‌حساب](#۲-سرویس-ماشین‌حساب)
  - [۳. کلاینت مستقیم MCP](#۳-کلاینت-مستقیم-mcp)
  - [۴. کلاینت مجهز به هوش مصنوعی](#۴-کلاینت-مجهز-به-هوش-مصنوعی)
- [اجرای مثال‌ها](#اجرای-مثال‌ها)
- [چگونگی کارکرد همه با هم](#چگونگی-کارکرد-همه-با-هم)
- [گام‌های بعدی](#گام‌های-بعدی)

## آنچه یاد خواهید گرفت

این آموزش نحوه ساخت سرویس ماشین‌حساب با استفاده از پروتکل زمینه مدل (MCP) را توضیح می‌دهد. شما خواهید فهمید:

- چگونه سرویسی بسازید که هوش مصنوعی بتواند به عنوان ابزاری از آن استفاده کند
- چگونه ارتباط مستقیم با سرویس‌های MCP برقرار کنید
- چگونه مدل‌های هوش مصنوعی به صورت خودکار ابزارهای مناسب را انتخاب کنند
- تفاوت بین تماس‌های مستقیم پروتکل و تعاملات با کمک هوش مصنوعی

## پیش‌نیازها

قبل از شروع، مطمئن شوید که:
- جاوا نسخه ۲۱ یا بالاتر نصب است
- Maven برای مدیریت وابستگی‌ها را دارید
- یک استقرار مدل Azure AI Foundry دارید (با دستور `azd up` راه‌اندازی کنید — نگاه کنید به [فصل ۲](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- رابط خط فرمان Azure CLI را نصب و با دستور `az login` وارد شده‌اید (احراز هویت بدون کلید)
- دانش پایه در Java و Spring Boot دارید

## درک ساختار پروژه

پروژه ماشین‌حساب چندین فایل مهم دارد:

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

## توضیح اجزای اصلی

### ۱. برنامه اصلی

**فایل:** `McpServerApplication.java`

این نقطه شروع سرویس ماشین‌حساب ماست. یک برنامه استاندارد Spring Boot با یکّ افزودنی ویژه:

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

**کارهایی که انجام می‌دهد:**
- راه‌اندازی سرور وب Spring Boot روی پورت ۸۰۸۰
- ایجاد `ToolCallbackProvider` که متدهای ماشین‌حساب ما را به عنوان ابزار MCP در دسترس قرار می‌دهد
- نشانه‌گذاری با `@Bean` به Spring می‌گوید این را مدیریت و به قسمت‌های دیگر ارائه کند

### ۲. سرویس ماشین‌حساب

**فایل:** `CalculatorService.java`

اینجاست که همه محاسبات انجام می‌شوند. هر متد با `@Tool` علامت‌گذاری شده تا از طریق MCP در دسترس باشد:

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
    
    // عملیات‌های بیشتر ماشین‌حساب...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**ویژگی‌های کلیدی:**

۱. **علامت‌گذاری `@Tool`**: به MCP می‌گوید این متد را کلاینت‌های خارجی می‌توانند فراخوانی کنند  
۲. **توضیحات واضح**: هر ابزار توضیحی دارد که به مدل‌های هوش مصنوعی کمک می‌کند زمان استفاده را بفهمند  
۳. **فرمت بازگشتی یکنواخت**: تمام عملیات رشته‌هایی به صورت خوانا مانند "5.00 + 3.00 = 8.00" برمی‌گردانند  
۴. **مدیریت خطا**: تقسیم بر صفر و جذر اعداد منفی پیام خطا برمی‌گردانند

**عملیات‌های موجود:**
- `add(a, b)` - جمع دو عدد  
- `subtract(a, b)` - تفریق عدد دوم از اول  
- `multiply(a, b)` - ضرب دو عدد  
- `divide(a, b)` - تقسیم عدد اول بر دوم (با چک صفر)  
- `power(base, exponent)` - توان رسانی عدد پایه  
- `squareRoot(number)` - محاسبه جذر (با چک منفی)  
- `modulus(a, b)` - باقی‌مانده تقسیم  
- `absolute(number)` - قدرمطلق  
- `help()` - اطلاعات درباره تمام عملیات‌ها

### ۳. کلاینت مستقیم MCP

**فایل:** `SDKClient.java`

این کلاینت مستقیماً با سرور MCP ارتباط دارد و از هوش مصنوعی استفاده نمی‌کند. به صورت دستی توابع خاص ماشین‌حساب را فراخوانی می‌کند:

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
        
        // فهرست ابزارهای موجود
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // فراخوانی توابع خاص ماشین حساب
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

**کارهایی که انجام می‌دهد:**
۱. با الگوی سازنده به سرور ماشین‌حساب در `http://localhost:8080` متصل می‌شود  
۲. لیست تمام ابزارهای در دسترس (توابع ماشین‌حساب ما) را دریافت می‌کند  
۳. توابع خاص را با پارامترهای دقیق فراخوانی می‌کند  
۴. نتایج را مستقیم چاپ می‌کند

**توجه:** این مثال از وابستگی Spring AI نسخه ۱.۱.۰-SNAPSHOT استفاده می‌کند که الگوی سازنده برای `WebFluxSseClientTransport` معرفی کرده است. اگر از نسخه‌ی پایدار قدیمی‌تر استفاده می‌کنید، ممکن است نیاز باشد مستقیماً سازنده را فراخوانی کنید.

**زمان استفاده:** وقتی دقیقاً می‌دانید چه محاسبه‌ای می‌خواهید انجام دهید و می‌خواهید برنامه‌نویسی شده آن را فراخوانی کنید.

### ۴. کلاینت مجهز به هوش مصنوعی

**فایل:** `LangChain4jClient.java`

این کلاینت از مدل هوش مصنوعی (GPT-4o-mini) استفاده می‌کند که می‌تواند به‌طور خودکار انتخاب کند کدام ابزارهای ماشین‌حساب را استفاده کند:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // تنظیم مدل هوش مصنوعی (Azure AI Foundry، احراز هویت بدون کلید از طریق Microsoft Entra ID)
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

        // اتصال به سرور ماشین حساب MCP ما
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // نمایش کاری که هوش مصنوعی انجام می‌دهد
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // اجازه دادن به هوش مصنوعی برای دسترسی به ابزارهای ماشین حساب ما
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // ایجاد ربات هوش مصنوعی که بتواند از ماشین حساب ما استفاده کند
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // حالا می‌توانیم از هوش مصنوعی بخواهیم محاسبات را به زبان طبیعی انجام دهد
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**کارهایی که انجام می‌دهد:**
۱. اتصال به مدل هوش مصنوعی را با استفاده از توکن گیت‌هاب شما ایجاد می‌کند  
۲. هوش مصنوعی را به سرور MCP ماشین‌حساب ما متصل می‌کند  
۳. همه ابزارهای ماشین‌حسابمان را برای هوش مصنوعی فراهم می‌کند  
۴. اجازه می‌دهد درخواست‌های زبان طبیعی مثل "مجموع ۲۴.۵ و ۱۷.۳ را حساب کن" داده شود

**هوش مصنوعی به طور خودکار:**
- متوجه می‌شود شما می‌خواهید جمع کنید  
- ابزار `add` را انتخاب می‌کند  
- `add(24.5, 17.3)` را فراخوانی می‌کند  
- نتیجه را به صورت پاسخ طبیعی برمی‌گرداند

## اجرای مثال‌ها

### مرحله ۱: راه‌اندازی سرور ماشین‌حساب

ابتدا وارد حساب شده و نقطه پایان Azure AI Foundry را تنظیم کنید (برای کلاینت هوش مصنوعی لازم است — احراز هویت بدون کلید، بدون کلید API):

**ویندوز:**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**لینوکس/macOS:**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

سرور را راه‌اندازی کنید:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

سرور روی آدرس `http://localhost:8080` شروع به کار می‌کند. باید ببینید:
```
Started McpServerApplication in X.XXX seconds
```

### مرحله ۲: تست با کلاینت مستقیم

در یک ترمینال **جدید** و با سرور هنوز کار می‌کند، کلاینت مستقیم MCP را اجرا کنید:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

خروجی مشابه زیر مشاهده می‌کنید:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### مرحله ۳: تست با کلاینت هوش مصنوعی

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

خواهید دید که هوش مصنوعی به طور خودکار از ابزارها استفاده می‌کند:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### مرحله ۴: خاموش کردن سرور MCP

بعد از اتمام آزمایش، می‌توانید کلاینت هوش مصنوعی را با فشار دادن `Ctrl+C` در ترمینال آن قطع کنید. سرور MCP تا زمانی که آن را متوقف کنید به کار ادامه خواهد داد.  
برای توقف سرور، در ترمینالی که در حال اجرا است `Ctrl+C` را بزنید.

## چگونگی کارکرد همه با هم

در اینجا جریان کامل وقتی از هوش مصنوعی می‌پرسید "۵ + ۳ چقدر است؟" آمده است:

۱. **شما** سوال را به زبان طبیعی به هوش مصنوعی می‌دهید  
۲. **هوش مصنوعی** درخواست شما را تحلیل می‌کند و متوجه می‌شود شما جمع می‌خواهید  
۳. **هوش مصنوعی** سرور MCP را فراخوانی می‌کند: `add(5.0, 3.0)`  
۴. **سرویس ماشین‌حساب** عملیات `۵.۰ + ۳.۰ = ۸.۰` را انجام می‌دهد  
۵. **سرویس ماشین‌حساب** نتیجه `"5.00 + 3.00 = 8.00"` را برمی‌گرداند  
۶. **هوش مصنوعی** نتیجه را دریافت کرده و پاسخ طبیعی می‌سازد  
۷. **شما** پاسخ می‌گیرید: "مجموع ۵ و ۳ برابر با ۸ است"

## گام‌های بعدی

برای مثال‌های بیشتر، به [فصل ۰۴: نمونه‌های عملی](../README.md) مراجعه کنید

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**سلب مسئولیت**:
این سند با استفاده از سرویس ترجمه هوش مصنوعی [Co-op Translator](https://github.com/Azure/co-op-translator) ترجمه شده است. در حالی که ما در تلاش برای دقت هستیم، لطفاً توجه داشته باشید که ترجمه‌های خودکار ممکن است شامل خطاها یا نادرستی‌هایی باشند. سند اصلی به زبان مادری خود باید به عنوان منبع معتبر در نظر گرفته شود. برای اطلاعات حیاتی، ترجمه حرفه‌ای انسانی توصیه می‌شود. ما در قبال هرگونه سوء تفاهم یا برداشت نادرست ناشی از استفاده از این ترجمه مسئولیتی نداریم.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->