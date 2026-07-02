# دليل مولد قصص الحيوانات الأليفة للمبتدئين

## جدول المحتويات

- [المتطلبات الأساسية](#المتطلبات-الأساسية)
- [فهم هيكل المشروع](#فهم-هيكل-المشروع)
- [شرح المكونات الأساسية](#شرح-المكونات-الأساسية)
  - [1. التطبيق الرئيسي](#1-التطبيق-الرئيسي)
  - [2. متحكم الويب](#2-متحكم-الويب)
  - [3. خدمة القصة](#3-خدمة-القصة)
  - [4. قوالب الويب](#4-قوالب-الويب)
  - [5. التكوين](#5-التكوين)
- [تشغيل التطبيق](#تشغيل-التطبيق)
- [كيف تعمل الأمور معًا](#كيف-تعمل-الأمور-معًا)
- [فهم دمج الذكاء الاصطناعي](#فهم-دمج-الذكاء-الاصطناعي)
- [الخطوات التالية](#الخطوات-التالية)

## المتطلبات الأساسية

قبل البدء، تأكد من أن لديك:
- جافا 21 أو أعلى مثبتة
- Maven لإدارة التبعيات
- نشر نموذج Azure AI Foundry (قم بتوفيره باستخدام `azd up` — راجع [الفصل 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))، وقم بتسجيل الدخول عبر `az login` (المصادقة بدون مفتاح)
- فهم أساسي لجافا، Spring Boot، وتطوير الويب

## فهم هيكل المشروع

يحتوي مشروع قصة الحيوانات الأليفة على عدة ملفات مهمة:

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

## شرح المكونات الأساسية

### 1. التطبيق الرئيسي

**الملف:** `PetStoryApplication.java`

هذا هو نقطة الدخول لتطبيق Spring Boot الخاص بنا:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**ما يقوم به:**
- التعليمة `@SpringBootApplication` تُمكّن التكوين التلقائي ومسح المكونات
- تشغيل خادم ويب مدمج (Tomcat) على المنفذ 8080
- إنشاء جميع حبات وخدمات Spring اللازمة تلقائيًا

### 2. متحكم الويب

**الملف:** `PetController.java`

هذا يتعامل مع جميع طلبات الويب وتفاعلات المستخدم:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // يعيد قالب index.html
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // التحقق من صحة الإدخال
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // تنظيف الإدخال لأسباب أمنية
        String sanitizedDescription = sanitizeInput(description);
        
        // توليد القصة مع معالجة الأخطاء
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // يعيد قالب result.html
            
        } catch (Exception e) {
            // استخدام قصة احتياطية في حال فشل الذكاء الاصطناعي
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // تحديد الطول
    }
}
```

**الميزات الرئيسية:**

1. **معالجة المسارات:** `@GetMapping("/")` يعرض نموذج الرفع، و `@PostMapping("/generate-story")` يعالج الإرساليات
2. **التحقق من الإدخال:** يفحص الوصف الفارغ وحدود الطول
3. **الأمان:** تنقية مدخلات المستخدم لمنع هجمات XSS
4. **معالجة الأخطاء:** يوفر قصص بديلة عند فشل خدمة الذكاء الاصطناعي
5. **ربط النماذج:** يمرر البيانات إلى قوالب HTML باستخدام نموذج Spring

**نظام الاسترجاع:**
يحتوي المتحكم على قوالب قصص مكتوبة مسبقًا تُستخدم عند عدم توفر خدمة الذكاء الاصطناعي:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // استخدم تجزئة الوصف للحصول على استجابات متسقة
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. خدمة القصة

**الملف:** `StoryService.java`

تتواصل هذه الخدمة مع Azure AI Foundry لتوليد القصص باستخدام المصادقة بدون مفتاح:

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
        
        // نقطة النهاية المتوافقة مع OpenAI الخاصة بـ Foundry تقع تحت /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // المصادقة بدون مفتاح مع Microsoft Entra ID (بدون مفتاح API)
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
        
        // تكوين طلب الذكاء الاصطناعي
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // تحديد طول الاستجابة
                .temperature(0.8)          // التحكم في الإبداع (0.0-1.0)
                .build();
        
        // إرسال الطلب والحصول على الاستجابة
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**المكونات الرئيسية:**

1. **عميل OpenAI:** يستخدم SDK الرسمي لجافا الخاص بـ OpenAI مهيأ لـ Azure AI Foundry (بدون مفتاح)
2. **موجه النظام:** يحدد سلوك الذكاء الاصطناعي لكتابة قصص حيوانات أليفة مناسبة للعائلة
3. **موجه المستخدم:** يخبر الذكاء الاصطناعي بما يجب كتابته بناءً على الوصف
4. **المعلمات:** تتحكم في طول القصة ومستوى الإبداع
5. **معالجة الأخطاء:** يرمي استثناءات يلتقطها المتحكم ويتعامل معها

### 4. قوالب الويب

**الملف:** `index.html` (نموذج الرفع)

الصفحة الرئيسية حيث يصف المستخدمون حيواناتهم الأليفة:

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

**الملف:** `result.html` (عرض القصة)

يعرض القصة المولدة:

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

**ميزات القالب:**

1. **تكامل Thymeleaf:** يستخدم سمات `th:` للمحتوى الديناميكي
2. **تصميم متجاوب:** تنسيق CSS للهاتف والكمبيوتر المكتبي
3. **معالجة الأخطاء:** يعرض أخطاء التحقق للمستخدمين
4. **المعالجة على جانب العميل:** جافا سكريبت لتحليل الصور (باستخدام Transformers.js)

### 5. التكوين

**الملف:** `application.properties`

إعدادات التكوين للتطبيق:

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

**شرح التكوين:**

1. **رفع الملفات:** يسمح بصور تصل حتى 10MB
2. **السجلات:** يتحكم بما يتم تسجيله أثناء التنفيذ
3. **Azure AI Foundry:** يحدد نقطة النهاية ونموذج النشر المستخدم (بدون مفتاح)
4. **الأمان:** إعدادات معالجة الأخطاء لتجنب كشف معلومات حساسة

## تشغيل التطبيق

### الخطوة 1: تسجيل الدخول وضبط نقطة النهاية الخاصة بك

المصادقة بدون مفتاح (Microsoft Entra ID)، لذا لا يوجد مفتاح API. قم بتسجيل الدخول واضبط نقطة نهاية Foundry الخاصة بك:

**ويندوز (موجه الأوامر):**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**ويندوز (PowerShell):**
```powershell
az login
$env:AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
```

**لينكس/ماك أو إس:**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**لماذا هذا ضروري:**
- يستخدم Azure AI Foundry معرف Microsoft Entra للمصادقة على طلبات الاستدلال
- المصادقة بدون مفتاح تعني عدم وجود أسرار في شفرتك المصدرية أو البيئة
- يحتاج حسابك إلى دور **Cognitive Services OpenAI User** على المورد

### الخطوة 2: البناء والتشغيل

انتقل إلى دليل المشروع:
```bash
cd 04-PracticalSamples/petstory
```

ابنِ التطبيق:
```bash
mvn clean compile
```

شغّل الخادم:
```bash
mvn spring-boot:run
```

سيبدأ التطبيق على `http://localhost:8080`.

### الخطوة 3: اختبار التطبيق

1. **افتح** `http://localhost:8080` في متصفحك
2. **وصف** حيوانك الأليف في مربع النص (مثلاً، "كلب جولدن ريتريفر لعوب يحب الاسترجاع")
3. **انقر** على "توليد القصة" للحصول على قصة مولدة بواسطة الذكاء الاصطناعي
4. **بدلاً من ذلك**، ارفع صورة للحيوان الأليف لتوليد وصف تلقائيًا
5. **شاهد** القصة الإبداعية المبنية على وصف حيوانك الأليف

## كيف تعمل الأمور معًا

إليك التدفق الكامل عند توليد قصة للحيوان الأليف:

1. **إدخال المستخدم:** تصف حيوانك الأليف في نموذج الويب
2. **إرسال النموذج:** يرسل المتصفح طلب POST إلى `/generate-story`
3. **معالجة المتحكم:** يتحقق `PetController` من صحة وتنقية الإدخال
4. **نداء خدمة الذكاء الاصطناعي:** يرسل `StoryService` الطلب إلى نموذج Azure AI Foundry
5. **توليد القصة:** يُولّد الذكاء الاصطناعي قصة إبداعية بناءً على الوصف
6. **معالجة الاستجابة:** يستقبل المتحكم القصة ويضيفها إلى النموذج
7. **تصيير القالب:** تعرض Thymeleaf ملف `result.html` مع القصة
8. **العرض:** يرى المستخدم القصة المولدة في المتصفح

**تدفق معالجة الأخطاء:**
إذا فشلت خدمة الذكاء الاصطناعي:
1. يلتقط المتحكم الاستثناء
2. يُولّد قصة بديلة باستخدام القوالب المكتوبة مسبقًا
3. يعرض القصة البديلة مع ملاحظة عن عدم توفر الذكاء الاصطناعي
4. لا يزال المستخدم يحصل على قصة، مما يضمن تجربة مستخدم جيدة

## فهم دمج الذكاء الاصطناعي

### Azure AI Foundry (بدون مفتاح)
يستخدم التطبيق Azure AI Foundry مع مصادقة بدون مفتاح (Microsoft Entra ID):

```java
// مصادقة بدون مفتاح - لا مفتاح API
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### هندسة الموجهات
تستخدم الخدمة موجهات مصممة بعناية للحصول على نتائج جيدة:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### معالجة الاستجابة
يتم استخراج استجابة الذكاء الاصطناعي والتحقق منها:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## الخطوات التالية

للمزيد من الأمثلة، راجع [الفصل 04: نماذج عملية](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**تنويه**:
تمت ترجمة هذا المستند باستخدام خدمة الترجمة بالذكاء الاصطناعي [Co-op Translator](https://github.com/Azure/co-op-translator). بينما نسعى للدقة، يرجى العلم أن الترجمات الآلية قد تحتوي على أخطاء أو عدم دقة. يجب اعتبار المستند الأصلي بلغته الأصلية المصدر الرسمي والمعتمد. للمعلومات الهامة، يُنصح بالاستعانة بترجمة بشرية محترفة. نحن غير مسؤولين عن أي سوء فهم أو تفسير ناتج عن استخدام هذه الترجمة.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->