# آموزش ساخت داستان حیوان خانگی برای مبتدیان

## فهرست مطالب

- [پیش‌نیازها](#پیش‌نیازها)
- [درک ساختار پروژه](#درک-ساختار-پروژه)
- [توضیح اجزای اصلی](#توضیح-اجزای-اصلی)
  - [1. برنامه اصلی](#1-برنامه-اصلی)
  - [2. کنترل‌کننده وب](#2-کنترل‌کننده-وب)
  - [3. سرویس داستان](#3-سرویس-داستان)
  - [4. قالب‌های وب](#4-قالب‌های-وب)
  - [5. پیکربندی](#5-پیکربندی)
- [اجرای برنامه](#اجرای-برنامه)
- [نحوه کارکرد همه‌چیز با هم](#نحوه-کارکرد-همه‌چیز-با-هم)
- [درک ادغام هوش مصنوعی](#درک-ادغام-هوش-مصنوعی)
- [گام‌های بعدی](#گام‌های-بعدی)

## پیش‌نیازها

قبل از شروع، مطمئن شوید که:
- جاوا ۲۱ یا بالاتر نصب شده است
- Maven برای مدیریت وابستگی‌ها
- یک استقرار مدل Azure AI Foundry دارید (با دستور `azd up` آن را فراهم کنید — مراجعه کنید به [فصل ۲](../../02-SetupDevEnvironment/getting-started-azure-openai.md))، به وسیله `az login` وارد شده‌اید (احراز هویت بدون کلید)
- دانش پایه‌ای از جاوا، Spring Boot، و توسعه وب دارید

## درک ساختار پروژه

پروژه داستان حیوان خانگی شامل چندین فایل مهم است:

```
petstory/
├── src/main/java/com/example/petstory/
│   ├── PetStoryApplication.java       # Main Spring Boot application
│   ├── PetController.java             # Web request handler
│   ├── StoryService.java              # AI story generation service
│   └── SecurityConfig.java            # Security configuration
├── src/main/resources/
│   ├── application.properties         # App configuration
│   └── templates/
│       ├── index.html                 # Upload form page
│       └── result.html               # Story display page
└── pom.xml                           # Maven dependencies
```

## توضیح اجزای اصلی

### 1. برنامه اصلی

**فایل:** `PetStoryApplication.java`

این نقطه شروع برنامه Spring Boot ماست:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**کارکرد این بخش:**
- صفت `@SpringBootApplication` پیکربندی خودکار و جستجوی کامپوننت‌ها را فعال می‌کند
- یک وب‌سرور داخلی (Tomcat) را روی پورت ۸۰۸۰ راه‌اندازی می‌کند
- به صورت خودکار تمام Beanها و سرویس‌های Spring مورد نیاز را می‌سازد

### 2. کنترل‌کننده وب

**فایل:** `PetController.java`

این کنترل‌کننده تمام درخواست‌های وب و تعاملات کاربر را مدیریت می‌کند:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // قالب index.html را باز می‌گرداند
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // اعتبارسنجی ورودی
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // ورودی را برای امنیت پاک‌سازی کنید
        String sanitizedDescription = sanitizeInput(description);
        
        // تولید داستان با مدیریت خطا
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // قالب result.html را باز می‌گرداند
            
        } catch (Exception e) {
            // در صورت شکست هوش مصنوعی از داستان جایگزین استفاده کنید
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // محدود کردن طول
    }
}
```

**ویژگی‌های کلیدی:**

1. **مدیریت مسیرها:** `@GetMapping("/")` فرم بارگذاری را نمایش می‌دهد، `@PostMapping("/generate-story")` ارسال‌ها را پردازش می‌کند
2. **اعتبارسنجی ورودی:** بررسی توصیف‌های خالی و محدودیت طول
3. **امنیت:** پاکسازی ورودی‌های کاربر برای جلوگیری از حملات XSS
4. **مدیریت خطا:** ارائه داستان‌های جایگزین هنگام بروز اشکال در سرویس هوش مصنوعی
5. **اتصال مدل:** ارسال داده‌ها به قالب‌های HTML با استفاده از Model در Spring

**سیستم جایگزین:**
کنترل‌کننده قالب‌های داستان از پیش نوشته شده دارد که وقتی سرویس هوش مصنوعی در دسترس نباشد، استفاده می‌شوند:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // استفاده از هش توضیحات برای پاسخ‌های منسجم
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. سرویس داستان

**فایل:** `StoryService.java`

این سرویس برای تولید داستان‌ها با استفاده از احراز هویت بدون کلید با Azure AI Foundry ارتباط برقرار می‌کند:

```java
@Service
public class StoryService {
    
    private final OpenAIClient openAIClient;
    private final String modelName;
    
    public StoryService(@Value("${azure.openai.endpoint:}") String endpoint,
                       @Value("${azure.openai.deployment:gpt-4o-mini}") String modelName) {
        this.modelName = modelName;
        if (endpoint == null || endpoint.isBlank()) {
            endpoint = System.getenv("AZURE_OPENAI_ENDPOINT");
        }
        
        // نقطه انتهایی سازگار با OpenAI Foundry زیر /openai/v1/ قرار دارد
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // احراز هویت بدون کلید با Microsoft Entra ID (بدون کلید API)
        DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
        this.openAIClient = OpenAIOkHttpClient.builder()
                .baseUrl(baseUrl)
                .credential(BearerTokenCredential.create(
                        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
                .build();
    }
    
    public String generateStory(String description) {
        String systemPrompt = "You are a creative storyteller who writes fun, " +
                             "family-friendly short stories about pets. " +
                             "Keep stories under 500 words and appropriate for all ages.";
        
        String userPrompt = "Write a fun short story about a pet described as: " + description;
        
        // پیکربندی درخواست هوش مصنوعی
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // محدود کردن طول پاسخ
                .temperature(0.8)          // کنترل خلاقیت (۰.۰-۱.۰)
                .build();
        
        // ارسال درخواست و دریافت پاسخ
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**اجزای کلیدی:**

1. **کلاینت OpenAI:** از SDK رسمی OpenAI Java استفاده می‌کند که برای Azure AI Foundry (بدون کلید) پیکربندی شده است
2. **دستور سیستم:** رفتار هوش مصنوعی را برای نوشتن داستان‌های مناسب خانواده درباره حیوانات خانگی تنظیم می‌کند
3. **دستور کاربر:** به هوش مصنوعی دقیقا می‌گوید بر اساس توصیف چه داستانی بنویسد
4. **پارامترها:** کنترل طول داستان و سطح خلاقیت
5. **مدیریت خطا:** استثناهایی که کنترل‌کننده آن‌ها را دریافت و مدیریت می‌کند، پرتاب می‌کند

### 4. قالب‌های وب

**فایل:** `index.html` (فرم بارگذاری)

صفحه اصلی که کاربران در آن حیوانات خانگی خود را توصیف می‌کنند:

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Pet Story Generator</title>
    <!-- CSS styling -->
</head>
<body>
    <div class="container">
        <h1>Pet Story Generator</h1>
        <p>Describe your pet and we'll create a fun story about them!</p>
        
        <!-- Error message display -->
        <div th:if="${error}" class="error" th:text="${error}"></div>
        
        <!-- Story generation form -->
        <form action="/generate-story" method="post">
            <div class="form-group">
                <label for="description">Describe your pet:</label>
                <textarea id="description" name="description" 
                         placeholder="Tell us about your pet - what they look like, their personality, favorite activities..."
                         maxlength="1000" required></textarea>
            </div>
            <button type="submit" class="btn btn-primary">Generate Story</button>
        </form>
        
        <!-- Image upload section with client-side processing -->
        <div class="upload-section">
            <h2>Or Upload a Photo</h2>
            <input type="file" id="imageInput" accept="image/*" />
            <button onclick="analyzeImage()" class="upload-btn">Analyze Image</button>
        </div>
        
        <script>
            // Client-side image analysis using Transformers.js
            async function analyzeImage() {
                // Image processing code here
                // Generates description automatically from uploaded image
            }
        </script>
    </div>
</body>
</html>
```

**فایل:** `result.html` (نمایش داستان)

داستان تولید شده را نمایش می‌دهد:

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Pet Story Result</title>
</head>
<body>
    <div class="container">
        <h1>Your Pet's Story</h1>
        
        <div class="result-section">
            <div class="result-label">Pet Description:</div>
            <div class="result-content" th:text="${caption}"></div>
        </div>
        
        <div class="result-section">
            <div class="result-label">Generated Story:</div>
            <div class="result-content" th:text="${story}"></div>
        </div>
        
        <div class="result-section" th:if="${analysisType}">
            <div class="result-label">Analysis Type:</div>
            <div class="result-content" th:text="${analysisType}"></div>
        </div>
        
        <a href="/" class="back-link">Generate Another Story</a>
    </div>
</body>
</html>
```

**ویژگی‌های قالب:**

1. **ادغام Thymeleaf:** استفاده از ویژگی‌های `th:` برای محتوای پویا
2. **طراحی پاسخگو:** استایل CSS برای موبایل و دسکتاپ
3. **مدیریت خطا:** نمایش خطاهای اعتبارسنجی به کاربران
4. **پردازش سمت کلاینت:** استفاده از جاوااسکریپت برای تحلیل تصویر (با استفاده از Transformers.js)

### 5. پیکربندی

**فایل:** `application.properties`

تنظیمات پیکربندی برنامه:

```properties
spring.application.name=pet-story-app

# File upload limits
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=10MB

# Logging configuration
logging.level.com.example.petstory=INFO

# Azure AI Foundry (keyless) configuration
azure.openai.endpoint=${AZURE_OPENAI_ENDPOINT:}
azure.openai.deployment=${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}
```

**توضیح پیکربندی:**

1. **بارگذاری فایل:** اجازه بارگذاری تصاویر تا حجم ۱۰ مگابایت را می‌دهد
2. **ثبت لاگ:** کنترل اطلاعاتی که هنگام اجرا ثبت می‌شود
3. **Azure AI Foundry:** مشخص کردن نقطه انتهایی و مدل استقرار برای استفاده (احراز هویت بدون کلید)
4. **امنیت:** پیکربندی مدیریت خطا برای جلوگیری از افشای اطلاعات حساس

## اجرای برنامه

### گام ۱: ورود و تعیین نقطه انتهایی

احراز هویت بدون کلید است (Microsoft Entra ID)، پس کلید API وجود ندارد. وارد شوید و نقطه انتهایی Foundry خود را تنظیم کنید:

**ویندوز (Command Prompt):**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**ویندوز (PowerShell):**
```powershell
az login
$env:AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
```

**لینوکس/macOS:**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**چرا این کار لازم است:**
- Azure AI Foundry برای احراز هویت درخواست‌های استنتاج از Microsoft Entra ID استفاده می‌کند
- احراز هویت بدون کلید یعنی هیچ راز یا کلیدی در سورس کد یا محیط وجود ندارد
- حساب شما باید دارای نقش **Cognitive Services OpenAI User** روی منبع باشد

### گام ۲: ساخت و اجرا

به دایرکتوری پروژه بروید:
```bash
cd 04-PracticalSamples/petstory
```

برنامه را بسازید:
```bash
mvn clean compile
```

سرور را راه‌اندازی کنید:
```bash
mvn spring-boot:run
```

برنامه روی `http://localhost:8080` اجرا خواهد شد.

### گام ۳: آزمایش برنامه

1. **باز کردن** `http://localhost:8080` در مرورگر
2. **توصیف** حیوان خانگی خود در کادر متن (مثلا: «یک سگ گلدن رتریور بازیگوش که دوست دارد توپ بیاورد»)
3. **کلیک** روی "Generate Story" برای دریافت داستان تولید شده توسط هوش مصنوعی
4. **یا** تصویری از حیوان خانگی بارگذاری کنید تا توصیف به صورت خودکار تولید شود
5. **مشاهده** داستان خلاقانه بر اساس توصیف حیوان خانگی شما

## نحوه کارکرد همه‌چیز با هم

در اینجا جریان کامل زمانی که داستان حیوان خانگی تولید می‌کنید آمده است:

1. **ورودی کاربر:** شما حیوان خانگی خود را در فرم وب توصیف می‌کنید
2. **ارسال فرم:** مرورگر درخواست POST به `/generate-story` می‌فرستد
3. **پردازش کنترل‌کننده:** `PetController` ورودی را اعتبارسنجی و پاکسازی می‌کند
4. **تماس با سرویس هوش مصنوعی:** `StoryService` درخواست به مدل Azure AI Foundry ارسال می‌کند
5. **تولید داستان:** هوش مصنوعی داستان خلاقانه‌ای بر اساس توصیف می‌نویسد
6. **مدیریت پاسخ:** کنترل‌کننده داستان را دریافت و به مدل اضافه می‌کند
7. **رندر قالب:** Thymeleaf قالب `result.html` را با داستان نمایش می‌دهد
8. **نمایش:** کاربر داستان تولید شده را در مرورگر خود می‌بیند

**جریان مدیریت خطا:**
اگر سرویس هوش مصنوعی خطا دهد:
1. کنترل‌کننده استثنا را می‌گیرد
2. با استفاده از قالب‌های از پیش نوشته شده یک داستان جایگزین تولید می‌کند
3. داستان جایگزین را همراه با پیامی درباره عدم دسترسی AI نمایش می‌دهد
4. کاربر همچنان یک داستان دریافت می‌کند تا تجربه کاربری خوبی حفظ شود

## درک ادغام هوش مصنوعی

### Azure AI Foundry (بدون کلید)

برنامه از Azure AI Foundry با احراز هویت بدون کلید (Microsoft Entra ID) استفاده می‌کند:

```java
// احراز هویت بدون کلید - بدون کلید API
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### مهندسی پرامپت

سرویس از پرامپت‌های به دقت طراحی شده برای گرفتن نتایج خوب استفاده می‌کند:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### پردازش پاسخ

پاسخ هوش مصنوعی استخراج و اعتبارسنجی می‌شود:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## گام‌های بعدی

برای نمونه‌های بیشتر، به [فصل ۰۴: نمونه‌های عملی](../README.md) مراجعه کنید.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**سلب مسئولیت**:
این سند با استفاده از سرویس ترجمه هوش مصنوعی [Co-op Translator](https://github.com/Azure/co-op-translator) ترجمه شده است. در حالی که ما در تلاش برای دقت هستیم، لطفاً توجه داشته باشید که ترجمه‌های خودکار ممکن است شامل خطاها یا نادرستی‌هایی باشند. سند اصلی به زبان مادری خود باید به عنوان منبع معتبر در نظر گرفته شود. برای اطلاعات حیاتی، ترجمه حرفه‌ای انسانی توصیه می‌شود. ما در قبال هرگونه سوء تفاهم یا برداشت نادرست ناشی از استفاده از این ترجمه مسئولیتی نداریم.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->