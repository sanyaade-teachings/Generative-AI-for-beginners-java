# آموزش محاسبه‌گر MCP برای مبتدیان

## فهرست مطالب

- [آنچه یاد خواهید گرفت](#آنچه-یاد-خواهید-گرفت)
- [پیش‌نیازها](#پیش‌نیازها)
- [درک ساختار پروژه](#درک-ساختار-پروژه)
- [توضیح اجزای اصلی](#توضیح-اجزای-اصلی)
  - [1. برنامه اصلی](#1-برنامه-اصلی)
  - [2. سرویس محاسبه‌گر](#2-سرویس-محاسبه‌گر)
  - [3. کلاینت مستقیم MCP](#3-کلاینت-مستقیم-mcp)
  - [4. کلاینت مبتنی بر هوش مصنوعی](#4-کلاینت-مبتنی-بر-هوش-مصنوعی)
- [اجرای نمونه‌ها](#اجرای-نمونه‌ها)
- [چگونگی عملکرد همه با هم](#چگونگی-عملکرد-همه-با-هم)
- [گام‌های بعدی](#گام‌های-بعدی)

## آنچه یاد خواهید گرفت

این آموزش توضیح می‌دهد چگونه یک سرویس محاسبه‌گر با استفاده از پروتکل کانتکست مدل (MCP) بسازید. شما خواهید فهمید:

- چگونه یک سرویسی بسازید که هوش مصنوعی بتواند از آن به عنوان ابزاری استفاده کند
- چگونه ارتباط مستقیم با سرویس‌های MCP برقرار کنید
- چگونه مدل‌های هوش مصنوعی می‌توانند به صورت خودکار ابزارهای مورد نیاز را انتخاب کنند
- تفاوت تماس‌های مستقیم پروتکل با تعاملات پشتیبانی شده توسط هوش مصنوعی چیست

## پیش‌نیازها

قبل از شروع، اطمینان حاصل کنید که:
- جاوا نسخه 21 یا بالاتر نصب باشد
- Maven برای مدیریت وابستگی‌ها داشته باشید
- یک استقرار مدل Azure AI Foundry داشته باشید (با `azd up` آن را فراهم کنید — نگاه کنید به [فصل 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) را نصب کرده و با `az login` وارد شده باشید (احراز هویت بدون کلید)
- درک پایه‌ای از جاوا و Spring Boot داشته باشید

## درک ساختار پروژه

پروژه محاسبه‌گر شامل چند فایل مهم است:

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

### 1. برنامه اصلی

**فایل:** `McpServerApplication.java`

این نقطه ورود سرویس محاسبه‌گر ما است. این برنامه استاندارد Spring Boot با یک اضافه خاص است:

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

**کار این بخش:**
- راه‌اندازی یک وب‌سرور Spring Boot روی پورت 8080
- ایجاد یک `ToolCallbackProvider` که روش‌های محاسبه‌گر ما را به عنوان ابزارهای MCP در دسترس قرار می‌دهد
- نشانه `@Bean` به Spring می‌گوید این را به عنوان یک کامپوننت مدیریت کنید تا بخش‌های دیگر بتوانند از آن استفاده کنند

### 2. سرویس محاسبه‌گر

**فایل:** `CalculatorService.java`

اینجاست که همه محاسبات انجام می‌شود. هر متد با `@Tool` علامت‌گذاری شده تا از طریق MCP در دسترس باشد:

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
    
    // عملیات بیشتر ماشین حساب...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**ویژگی‌های کلیدی:**

1. **نشانه `@Tool`**: به MCP می‌گوید که این متد می‌تواند توسط کلاینت‌های خارجی فراخوانی شود
2. **توضیحات واضح**: هر ابزار توضیحی دارد که به مدل‌های هوش مصنوعی کمک می‌کند بفهمند کی باید از آن استفاده کنند
3. **فرمت خروجی یکسان**: تمام عملیات رشته‌هایی قابل فهم برای انسان مثل "5.00 + 3.00 = 8.00" را برمی‌گرداند
4. **مدیریت خطا**: تقسیم بر صفر و ریشه مربع منفی پیام خطا برمی‌گرداند

**عملیات موجود:**
- `add(a, b)` - جمع دو عدد
- `subtract(a, b)` - تفریق عدد دوم از اول
- `multiply(a, b)` - ضرب دو عدد
- `divide(a, b)` - تقسیم عدد اول بر دوم (چک صفر)
- `power(base, exponent)` - توان عدد پایه به توان نمایی
- `squareRoot(number)` - محاسبه ریشه دوم (با بررسی منفی بودن)
- `modulus(a, b)` - باقی‌مانده تقسیم
- `absolute(number)` - مقدار مطلق
- `help()` - برگشت اطلاعات درباره همه عملیات‌ها

### 3. کلاینت مستقیم MCP

**فایل:** `SDKClient.java`

این کلاینت به طور مستقیم با سرور MCP صحبت می‌کند بدون استفاده از هوش مصنوعی. به صورت دستی توابع خاص محاسبه‌گر را فراخوانی می‌کند:

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
        
        // فراخوانی توابع محاسبه‌گر خاص
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

**کار این بخش:**
1. **اتصال** به سرور محاسبه‌گر در `http://localhost:8080` با استفاده از الگوی سازنده
2. **نمایش** همه ابزارهای موجود (توابع محاسبه‌گر ما)
3. **فراخوانی** توابع خاص با پارامترهای دقیق
4. **چاپ** نتایج به طور مستقیم

**توجه:** این نمونه وابستگی Spring AI 1.1.0-SNAPSHOT را استفاده می‌کند که الگوی سازنده برای `WebFluxSseClientTransport` معرفی کرده است. اگر از نسخه قدیمی‌تر پایدار استفاده می‌کنید، ممکن است نیاز باشد از سازنده مستقیم استفاده کنید.

**چه زمانی استفاده کنیم؟** وقتی دقیقاً می‌دانید چه محاسبه‌ای می‌خواهید انجام دهید و می‌خواهید برنامه‌نویسی آن را انجام دهید.

### 4. کلاینت مبتنی بر هوش مصنوعی

**فایل:** `LangChain4jClient.java`

این کلاینت از یک مدل هوش مصنوعی (GPT-4o-mini) استفاده می‌کند که می‌تواند به طور خودکار تعیین کند کدام ابزارهای محاسبه‌گر را استفاده کند:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // راه‌اندازی مدل هوش مصنوعی (Azure AI Foundry، احراز هویت بدون کلید از طریق Microsoft Entra ID)
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

        // دسترسی دادن به هوش مصنوعی برای استفاده از ابزارهای ماشین حساب ما
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // ساخت یک ربات هوش مصنوعی که قادر به استفاده از ماشین حساب ما باشد
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // اکنون می‌توانیم از هوش مصنوعی بخواهیم که محاسبات را به زبان طبیعی انجام دهد
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**کار این بخش:**
1. **ایجاد** اتصال مدل هوش مصنوعی با احراز هویت بدون کلید (Microsoft Entra ID)
2. **اتصال** هوش مصنوعی به سرور MCP محاسبه‌گر ما
3. **دادن** دسترسی به همه ابزارهای محاسبه‌گر به هوش مصنوعی
4. **اجازه** درخواست‌های زبان طبیعی مثل "جمع 24.5 و 17.3 را حساب کن"

**هوش مصنوعی به صورت خودکار:**
- متوجه می‌شود که می‌خواهید اعداد را جمع کنید
- ابزار `add` را انتخاب می‌کند
- فراخوانی `add(24.5, 17.3)`
- نتیجه را در پاسخ طبیعی برمی‌گرداند

## اجرای نمونه‌ها

### گام 1: راه‌اندازی سرور محاسبه‌گر

ابتدا، وارد شوید و نقطه انتهایی Azure AI Foundry را تنظیم کنید (لازم برای کلاینت هوش مصنوعی — احراز هویت بدون کلید، بدون کلید API):

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

سرور روی `http://localhost:8080` راه‌اندازی می‌شود. باید ببینید:
```
Started McpServerApplication in X.XXX seconds
```

### گام 2: تست با کلاینت مستقیم

در یک ترمینال **جدید** در حالی که سرور هنوز در حال اجراست، کلاینت مستقیم MCP را اجرا کنید:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

نتایجی شبیه این می‌بینید:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### گام 3: تست با کلاینت هوش مصنوعی

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

می‌بینید که هوش مصنوعی به صورت خودکار از ابزارها استفاده می‌کند:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### گام 4: بستن سرور MCP

وقتی آزمایش‌ها تمام شد، می‌توانید کلاینت هوش مصنوعی را با فشار دادن `Ctrl+C` در ترمینال آن متوقف کنید. سرور MCP تا زمانی که خودتان آن را متوقف کنید به کار ادامه می‌دهد.
برای متوقف کردن سرور، `Ctrl+C` را در ترمینالی که سرور اجرا می‌شود فشار دهید.

## چگونگی عملکرد همه با هم

در اینجا جریان کامل وقتی از هوش مصنوعی می‌پرسید «5 + 3 چقدر است؟»:

1. **شما** درخواست را به زبان طبیعی از هوش مصنوعی می‌پرسید
2. **هوش مصنوعی** درخواست شما را تحلیل می‌کند و متوجه می‌شود که جمع می‌خواهید
3. **هوش مصنوعی** سرور MCP را با `add(5.0, 3.0)` فراخوانی می‌کند
4. **سرویس محاسبه‌گر** انجام می‌دهد: `5.0 + 3.0 = 8.0`
5. **سرویس محاسبه‌گر** برمی‌گرداند: `"5.00 + 3.00 = 8.00"`
6. **هوش مصنوعی** نتیجه را دریافت کرده و پاسخ طبیعی ارائه می‌دهد
7. **شما** دریافت می‌کنید: «جمع 5 و 3 برابر با 8 است»

## گام‌های بعدی

برای نمونه‌های بیشتر، به [فصل 04: نمونه‌های عملی](../README.md) مراجعه کنید

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**سلب مسئولیت**:
این سند با استفاده از سرویس ترجمه هوش مصنوعی [Co-op Translator](https://github.com/Azure/co-op-translator) ترجمه شده است. در حالی که ما در تلاش برای دقت هستیم، لطفاً توجه داشته باشید که ترجمه‌های خودکار ممکن است شامل خطاها یا نادرستی‌هایی باشند. سند اصلی به زبان مادری خود باید به عنوان منبع معتبر در نظر گرفته شود. برای اطلاعات حیاتی، ترجمه حرفه‌ای انسانی توصیه می‌شود. ما در قبال هرگونه سوء تفاهم یا برداشت نادرست ناشی از استفاده از این ترجمه مسئولیتی نداریم.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->