# پالتو جانوروں کی کہانی بنانے والے ٹیوٹوریل برائے نو آموز

## فہرست مضامین

- [درکار چیزیں](#درکار-چیزیں)
- [پروجیکٹ کی ساخت کو سمجھنا](#پروجیکٹ-کی-ساخت-کو-سمجھنا)
- [اہم اجزاء کی وضاحت](#اہم-اجزاء-کی-وضاحت)
  - [1. مین ایپلیکیشن](#1-مین-ایپلیکیشن)
  - [2. ویب کنٹرولر](#2-ویب-کنٹرولر)
  - [3. کہانی سروس](#3-کہانی-سروس)
  - [4. ویب ٹیمپلیٹس](#4-ویب-ٹیمپلیٹس)
  - [5. ترتیب](#5-ترتیب)
- [ایپلیکیشن چلانا](#ایپلیکیشن-چلانا)
- [یہ سب کیسے مل کر کام کرتا ہے](#یہ-سب-کیسے-مل-کر-کام-کرتا-ہے)
- [AI انٹیگریشن کو سمجھنا](#ai-انٹیگریشن-کو-سمجھنا)
- [اگلے اقدامات](#اگلے-اقدامات)

## درکار چیزیں

شروع کرنے سے پہلے، یقینی بنائیں کہ آپ کے پاس ہے:

- جاوا 21 یا اس سے اعلیٰ ورژن انسٹال شدہ ہے
- Maven برائے انحصارات کا انتظام
- Azure AI Foundry ماڈل کی تعیناتی ( `azd up` کے ساتھ فراہم کریں — دیکھیں [باب 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))، `az login` کے ذریعے سائن ان (بغیر کی کی تصدیق)
- Java، Spring Boot، اور ویب ڈویلپمنٹ کی بنیادی سمجھ

## پروجیکٹ کی ساخت کو سمجھنا

پالتو جانوروں کی کہانی کے پروجیکٹ میں چند اہم فائلز ہیں:

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

## اہم اجزاء کی وضاحت

### 1. مین ایپلیکیشن

**فائل:** `PetStoryApplication.java`

یہ ہمارے Spring Boot ایپلیکیشن کا داخلہ نقطہ ہے:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**یہ کیا کرتا ہے:**
- `@SpringBootApplication` اینوٹیشن خودکار ترتیب اور کمپونینٹ سکیننگ کو فعال کرتا ہے
- پورٹ 8080 پر ایمبیڈڈ ویب سرور (Tomcat) شروع کرتا ہے
- تمام ضروری Spring بینز اور سروسز خود بخود بناتا ہے

### 2. ویب کنٹرولر

**فائل:** `PetController.java`

یہ تمام ویب درخواستوں اور صارف کے تعاملات کو سنبھالتا ہے:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // انڈیکس.html ٹیمپلیٹ واپس کرتا ہے
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // ان پٹ کی تصدیق
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // سیکیورٹی کے لیے ان پٹ کو صاف کریں
        String sanitizedDescription = sanitizeInput(description);
        
        // غلطی کی ہینڈلنگ کے ساتھ کہانی تیار کریں
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // رزلٹ.html ٹیمپلیٹ واپس کرتا ہے
            
        } catch (Exception e) {
            // اگر AI ناکام ہو جائے تو بیک اپ کہانی استعمال کریں
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // لمبائی کو محدود کریں
    }
}
```

**اہم خصوصیات:**

1. **روٹ ہینڈلنگ**: `@GetMapping("/")` اپ لوڈ فارم دکھاتا ہے، `@PostMapping("/generate-story")` سبمیشن پروسیس کرتا ہے
2. **ان پٹ ویلیڈیشن**: خالی تفصیل اور لمبائی کی حدوں کی جانچ کرتا ہے
3. **سیکیورٹی**: XSS حملوں سے بچاؤ کے لئے صارف کی ان پٹ کو صاف کرتا ہے
4. **خرابی کا انتظام**: AI سروس ناکام ہو جانے پر بیک اپ کہانیاں فراہم کرتا ہے
5. **ماڈل بائنڈنگ**: Spring کی `Model` کے ذریعے HTML ٹیمپلیٹس کو ڈیٹا فراہم کرتا ہے

**بیک اپ سسٹم:**
کنٹرولر میں پہلے سے لکھی گئی کہانی ٹیمپلیٹس شامل ہیں جو AI سروس دستیاب نہ ہونے پر استعمال ہوتی ہیں:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // مستقل جوابات کے لیے تفصیل کا ہیش استعمال کریں
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. کہانی سروس

**فائل:** `StoryService.java`

یہ سروس Azure AI Foundry کے ساتھ رابطہ کرتی ہے تاکہ بغیر کی کی تصدیق کے کہانیاں تیار کرے:

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
        
        // فاؤنڈری کا اوپن اے آئی کے ساتھ ہم آہنگ اینڈپوائنٹ /openai/v1/ کے تحت واقع ہے
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // مائیکروسافٹ انٹرا آئی ڈی کے ساتھ بغیر کلید کی توثیق (کوئی API کلید نہیں)
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
        
        // اے آئی کی درخواست کو ترتیب دیں
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // ردعمل کی لمبائی محدود کریں
                .temperature(0.8)          // تخلیقی صلاحیت کو کنٹرول کریں (0.0-1.0)
                .build();
        
        // درخواست بھیجیں اور ردعمل حاصل کریں
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**اہم اجزاء:**

1. **OpenAI کلائنٹ**: Azure AI Foundry کے لئے سرکاری OpenAI جاوا SDK کو استعمال کرتا ہے (کی لیس)
2. **سسٹم پرامپٹ**: AI رویہ کو سیٹ کرتا ہے کہ وہ خاندانی دوستانہ پالتو جانوروں کی کہانیاں لکھے
3. **صارف پرامپٹ**: AI کو بتاتا ہے کہ تفصیل کی بنیاد پر کونسی کہانی لکھنی ہے
4. **پیرامیٹرز**: کہانی کی لمبائی اور تخلیقی صلاحیت کو کنٹرول کرتا ہے
5. **خرابی کا انتظام**: ایسی استثناء پھینکتا ہے جنہیں کنٹرولر پکڑتا اور سنبھالتا ہے

### 4. ویب ٹیمپلیٹس

**فائل:** `index.html` (اپ لوڈ فارم)

مین پیج جہاں صارفین اپنے پالتو جانور کی تفصیل دیتے ہیں:

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

**فائل:** `result.html` (کہانی کی نمائش)

تخلیق کردہ کہانی دکھاتا ہے:

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

**ٹیمپلیٹ خصوصیات:**

1. **Thymeleaf انٹیگریشن**: متحرک مواد کے لئے `th:` اتریبیوٹ استعمال کرتا ہے
2. **ریسپانسیو ڈیزائن**: موبائل اور ڈیسک ٹاپ کے لئے CSS اسٹائلنگ
3. **خرابی کا انتظام**: صارفین کو جانچ کی غلطیاں دکھاتا ہے
4. **کلائنٹ سائیڈ پروسیسنگ**: تصویری تجزیے کے لیے جاوا اسکرپٹ (Transformers.js کا استعمال)

### 5. ترتیب

**فائل:** `application.properties`

ایپلیکیشن کی ترتیب کی ترتیبات:

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

**ترتیب کی وضاحت:**

1. **فائل اپ لوڈ**: 10MB تک کی تصاویر کی اجازت دیتا ہے
2. **لاگنگ**: عملدرآمد کے دوران لاگ ہونے والی معلومات کو کنٹرول کرتا ہے
3. **Azure AI Foundry**: استعمال کے لئے اینڈ پوائنٹ اور ماڈل تعیناتی کی وضاحت کرتا ہے (کی لیس تصدیق)
4. **سیکیورٹی**: حساس معلومات کو ظاہر کرنے سے بچانے کے لئے خرابی کی ترتیب

## ایپلیکیشن چلانا

### مرحلہ 1: سائن ان کریں اور اپنا اینڈ پوائنٹ مقرر کریں

تصدیق کی کی نہیں ہے (Microsoft Entra ID)، اس لیے کوئی API کلید نہیں ہے۔ سائن ان کریں اور اپنا Foundry اینڈ پوائنٹ سیٹ کریں:

**ونڈوز (کمانڈ پرامپٹ):**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**ونڈوز (پاور شیل):**
```powershell
az login
$env:AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
```

**لینکس/میک او ایس:**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**یہ ضروری کیوں ہے:**
- Azure AI Foundry Microsoft Entra ID کو استعمال کرتا ہے inference درخواستیں مستند کرنے کے لیے
- کی لیس تصدیق کا مطلب ہے آپ کے سورس کوڈ یا ماحول میں کوئی خفیہ معلومات نہیں
- آپ کے اکاؤنٹ کو resource پر **Cognitive Services OpenAI User** کا کردار درکار ہے

### مرحلہ 2: کمپائل کریں اور چلائیں

پروجیکٹ ڈائریکٹری میں جائیں:
```bash
cd 04-PracticalSamples/petstory
```

ایپلیکیشن بنائیں:
```bash
mvn clean compile
```

سرور شروع کریں:
```bash
mvn spring-boot:run
```

ایپلیکیشن `http://localhost:8080` پر شروع ہو جائے گی۔

### مرحلہ 3: ایپلیکیشن کا تجربہ کریں

1. براؤزر میں `http://localhost:8080` کھولیں
2. اپنے پالتو جانور کی تفصیل درج کریں (مثلاً: "ایک خوش مزاج گولڈن ریٹریور جو گیند لانے میں پسند کرتا ہے")
3. "Generate Story" پر کلک کریں تاکہ AI سے کہانی حاصل ہو
4. متبادل طور پر، پالتو جانور کی تصویر اپ لوڈ کریں تاکہ خود بخود تفصیل تیار ہو
5. اپنے پالتو جانور کی تفصیل پر مبنی تخلیقی کہانی دیکھیں

## یہ سب کیسے مل کر کام کرتا ہے

جب آپ پالتو جانور کی کہانی بناتے ہیں تو مکمل عمل کچھ یوں ہوتا ہے:

1. **صارف کی ان پٹ**: آپ ویب فارم پر اپنے پالتو جانور کی تفصیل دیتے ہیں
2. **فارم کی جمع کرائی**: براؤزر POST درخواست `/generate-story` پر بھیجتا ہے
3. **کنٹرولر کا عمل**: `PetController` ان پٹ کی جانچ پڑتال اور صفائی کرتا ہے
4. **AI سروس کال**: `StoryService` Azure AI Foundry ماڈل کو درخواست بھیجتا ہے
5. **کہانی کی تخلیق**: AI تفصیل کی بنیاد پر تخلیقی کہانی بناتا ہے
6. **جواب کی ہینڈلنگ**: کنٹرولر کہانی حاصل کر کے ماڈل میں شامل کرتا ہے
7. **ٹیمپلیٹ رینڈرنگ**: Thymeleaf `result.html` میں کہانی کو رینڈر کرتا ہے
8. **نمائش**: صارف اپنے براؤزر میں کہانی کو دیکھتا ہے

**خرابی کے انتظام کا عمل:**
اگر AI سروس ناکام ہو جاتی ہے:

1. کنٹرولر استثناء پکڑتا ہے
2. پہلے سے لکھی گئی ٹیمپلیٹس سے متبادل کہانی بناتا ہے
3. ایک نوٹ کے ساتھ متبادل کہانی دکھاتا ہے کہ AI دستیاب نہیں تھا
4. صارف کو پھر بھی کہانی ملتی ہے، جس سے اچھا صارف تجربہ یقینی بنتا ہے

## AI انٹیگریشن کو سمجھنا

### Azure AI Foundry (کی لیس)
ایپلیکیشن کی تصدیق Microsoft Entra ID کے ذریعے بغیر کسی کلید کے Azure AI Foundry کا استعمال کرتی ہے:

```java
// بغیر چابی کی توثیق - کوئی API کی نہیں
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### پرامپٹ انجنیئرنگ
سروس اچھے نتائج کے لیے منفرد پرامپٹس استعمال کرتی ہے:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### جواب کا عملدرآمد
AI سے حاصل جواب کو نکالا اور تصدیق کیا جاتا ہے:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## اگلے اقدامات

مزید مثالوں کے لیے دیکھیں [باب 04: عملی نمونے](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ڈس کلیمر**:
یہ دستاویز AI ترجمہ سروس [Co-op Translator](https://github.com/Azure/co-op-translator) کے ذریعے ترجمہ کی گئی ہے۔ جبکہ ہم درستگی کے لیے کوشاں ہیں، براہ کرم اس بات سے آگاہ رہیں کہ خودکار ترجمے میں غلطیاں یا عدم درستیاں ہو سکتی ہیں۔ اصل دستاویز اپنے مادری زبان میں مستند ماخذ سمجھی جائے گی۔ حساس معلومات کے لیے پیشہ ور انسانی ترجمہ کی سفارش کی جاتی ہے۔ اس ترجمے کے استعمال سے پیدا ہونے والی کسی بھی غلط فہمی یا غلط تشریح کی ذمہ داری ہم قبول نہیں کرتے۔
<!-- CO-OP TRANSLATOR DISCLAIMER END -->