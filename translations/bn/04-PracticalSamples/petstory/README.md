# পেট গল্প জেনারেটর টিউটোরিয়াল শুরু করার জন্য

## বিষয়সূচী

- [প্রয়োজনীয়তা](#প্রয়োজনীয়তা)
- [প্রকল্পের কাঠামো বোঝা](#প্রকল্পের-কাঠামো-বোঝা)
- [কোর কম্পোনেন্ট ব্যাখ্যা](#কোর-কম্পোনেন্ট-ব্যাখ্যা)
  - [১। প্রধান অ্যাপ্লিকেশন](#১।-প্রধান-অ্যাপ্লিকেশন)
  - [২। ওয়েব কন্ট্রোলার](#২।-ওয়েব-কন্ট্রোলার)
  - [৩। গল্প সেবা](#৩।-গল্প-সেবা)
  - [৪। ওয়েব টেমপ্লেট](#৪।-ওয়েব-টেমপ্লেট)
  - [৫। কনফিগারেশন](#৫।-কনফিগারেশন)
- [অ্যাপ্লিকেশন চালানো](#অ্যাপ্লিকেশন-চালানো)
- [সবকিছু কিভাবে একসাথে কাজ করে](#সবকিছু-কিভাবে-একসাথে-কাজ-করে)
- [AI অন্তর্ভুক্তিকরণ বোঝা](#ai-অন্তর্ভুক্তিকরণ-বোঝা)
- [পরবর্তী ধাপ](#পরবর্তী-ধাপ)

## প্রয়োজনীয়তা

শুরু করার আগে নিশ্চিত করুন আপনার কাছে রয়েছে:
- Java 21 বা তার উপরের সংস্করণ ইন্সটল করা
- Maven ডিপেনডেন্সি ম্যানেজমেন্টের জন্য
- একটি Azure AI Foundry মডেল ডিপ্লয়মেন্ট (প্রোভিশন করতে `azd up` কমান্ড ব্যবহার করুন — দেখুন [অধ্যায় ২](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), `az login` দিয়ে সাইন ইন করুন (কী ছাড়া অথেন্টিকেশন)
- Java, Spring Boot, এবং ওয়েব ডেভেলপমেন্টের প্রাথমিক ধারণা

## প্রকল্পের কাঠামো বোঝা

পেট গল্প প্রকল্পে কয়েকটি গুরুত্বপূর্ণ ফাইল আছে:

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

## কোর কম্পোনেন্ট ব্যাখ্যা

### ১। প্রধান অ্যাপ্লিকেশন

**ফাইল:** `PetStoryApplication.java`

এটি আমাদের Spring Boot অ্যাপ্লিকেশনের এন্ট্রি পয়েন্ট:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**এটি কি করে:**
- `@SpringBootApplication` অ্যানোটেশন অটো-কনফিগারেশন এবং কম্পোনেন্ট স্ক্যানিং সক্ষম করে
- পোর্ট ৮০৮০ এ এম্বেডেড ওয়েব সার্ভার (Tomcat) চালু করে
- সব প্রয়োজনীয় Spring bean এবং সেবা স্বয়ংক্রিয়ভাবে তৈরি করে

### ২। ওয়েব কন্ট্রোলার

**ফাইল:** `PetController.java`

এটি সমস্ত ওয়েব অনুরোধ এবং ইউজার ইন্টারঅ্যাকশনগুলি পরিচালনা করে:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // index.html টেমপ্লেট ফেরত দেয়
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // ইনপুট যাচাই
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // নিরাপত্তার জন্য ইনপুট পরিষ্কার করুন
        String sanitizedDescription = sanitizeInput(description);
        
        // ত্রুটি ব্যবস্থাপনার সাথে গল্প তৈরি করুন
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // result.html টেমপ্লেট ফেরত দেয়
            
        } catch (Exception e) {
            // AI ব্যর্থ হলে বিকল্প গল্প ব্যবহার করুন
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // দৈর্ঘ্য সীমাবদ্ধ করুন
    }
}
```

**মূল বৈশিষ্ট্য:**

১. **রুট হ্যান্ডলিং**: `@GetMapping("/")` আপলোড ফর্ম দেখায়, `@PostMapping("/generate-story")` সাবমিশন প্রক্রিয়াজাত করে  
২. **ইনপুট যাচাই**: ফাঁকা বিবরণ এবং দৈর্ঘ্য সীমা চেক করে  
৩. **সিকিউরিটি**: XSS আক্রমণ প্রতিরোধে ইউজার ইনপুট স্যানিটাইজ করে  
৪. **এরর হ্যান্ডলিং**: AI সেবা ব্যর্থ হলে বিকল্প গল্প দেয়  
৫. **মডেল বাইন্ডিং**: Spring এর `Model` ব্যবহার করে HTML টেমপ্লেটে ডাটা পাঠায়

**বিকল্প ব্যবস্থা:**
কন্ট্রোলার প্রি-লিখিত গল্পের টেমপ্লেট অন্তর্ভুক্ত করে যা AI সেবা অনুপল‌ব্ধ হলে ব্যবহৃত হয়:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // সঙ্গতিপূর্ণ উত্তরগুলির জন্য বিবরণ হ্যাশ ব্যবহার করুন
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### ৩। গল্প সেবা

**ফাইল:** `StoryService.java`

এই সেবা Azure AI Foundry এর সাথে যোগাযোগ করে কী ছাড়া অথেন্টিকেশন দিয়ে গল্প তৈরি করে:

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
        
        // ফাউন্ড্রির OpenAI-সঙ্গত এন্ডপয়েন্ট /openai/v1/ এর নিচে অবস্থিত
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Microsoft Entra ID এর মাধ্যমে কীহীন প্রমাণীকরণ (কোনো API কী নেই)
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
        
        // AI অনুরোধ কনফিগার করুন
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // প্রতিক্রিয়ার দৈর্ঘ্য সীমাবদ্ধ করুন
                .temperature(0.8)          // সৃজনশীলতা নিয়ন্ত্রণ করুন (0.0-1.0)
                .build();
        
        // অনুরোধ পাঠান এবং প্রতিক্রিয়া পান
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**মূল উপাদান:**

১. **OpenAI ক্লায়েন্ট**: অফিসিয়াল OpenAI Java SDK ব্যবহার করে Azure AI Foundry এর জন্য কনফিগার করা (কী ছাড়া)  
২. **সিস্টেম প্রম্পট**: AI কে পারিবারিকভিত্তিক পোষা প্রাণীর গল্প লেখার জন্য নির্দেশ দেয়  
৩. **ইউজার প্রম্পট**: AI কে স্পষ্ট করে বলে কি গল্প লিখতে হবে বিবরণ অনুযায়ী  
৪. **প্যারামিটার**: গল্পের দৈর্ঘ্য এবং সৃজনশীলতার মাত্রা নিয়ন্ত্রণ করে  
৫. **এরর হ্যান্ডলিং**: কন্ট্রোলারে ক্যাচ করার জন্য এক্সসেপশন ছোঁড়ে

### ৪। ওয়েব টেমপ্লেট

**ফাইল:** `index.html` (আপলোড ফর্ম)

প্রধান পাতা যেখানে ব্যবহারকারী তাদের পোষা প্রাণীর বর্ণনা দেয়:

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

**ফাইল:** `result.html` (গল্প প্রদর্শন)

উত্পন্ন গল্প প্রদর্শন করে:

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

**টেমপ্লেট বৈশিষ্ট্য:**

১. **Thymeleaf ইন্টিগ্রেশন**: ডায়নামিক কন্টেন্টের জন্য `th:` অ্যাট্রিবিউট ব্যবহার করে  
২. **প্রতিক্রিয়াশীল ডিজাইন**: মোবাইল ও ডেস্কটপের জন্য CSS স্টাইলিং  
৩. **এরর হ্যান্ডলিং**: ইউজারের কাছে যাচাইকরণ ত্রুটি দেখায়  
৪. **ক্লায়েন্ট-সাইড প্রসেসিং**: ইমেজ বিশ্লেষণের জন্য JavaScript (Transformers.js ব্যবহার করে)

### ৫। কনফিগারেশন

**ফাইল:** `application.properties`

অ্যাপ্লিকেশনের কনফিগারেশন সেটিংস:

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

**কনফিগারেশন ব্যাখ্যা:**

১. **ফাইল আপলোড**: ১০MB পর্যন্ত চিত্র আপলোড করার অনুমতি দেয়  
২. **লগিং**: এক্সেকিউশনের সময় কোন তথ্য লগ হবে তা নিয়ন্ত্রণ করে  
৩. **Azure AI Foundry**: কী ছাড়া অথেন্টিকেশনের জন্য এন্ডপয়েন্ট ও মডেল ডিপ্লয়মেন্ট নির্ধারণ করে  
৪. **সিকিউরিটি**: সংবেদনশীল তথ্য ফাঁস রোধে এরর হ্যান্ডলিং কনফিগারেশন

## অ্যাপ্লিকেশন চালানো

### ধাপ ১: সাইন ইন করুন এবং এন্ডপয়েন্ট সেট করুন

কী ছাড়া অথেন্টিকেশন (Microsoft Entra ID) হওয়ায় API কী লাগে না। সাইন ইন করুন এবং Foundry এন্ডপয়েন্ট নির্ধারণ করুন:

**Windows (কমান্ড প্রম্পট):**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**Windows (PowerShell):**
```powershell
az login
$env:AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
```

**Linux/macOS:**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**কেন এটি প্রয়োজন:**
- Azure AI Foundry ইনফারেন্স অনুরোধ যাচাই করতে Microsoft Entra ID ব্যবহার করে  
- কী ছাড়া অথেন্টিকেশন মানে সোর্স কোড বা এনভায়রনমেন্টে কোনো সিক্রেট নেই  
- আপনার অ্যাকাউন্টে **Cognitive Services OpenAI User** রোল থাকা আবশ্যক রিসোর্সে

### ধাপ ২: বিল্ড এবং চালান

প্রকল্প ডিরেক্টরিতে যান:
```bash
cd 04-PracticalSamples/petstory
```

অ্যাপ্লিকেশন বিল্ড করুন:
```bash
mvn clean compile
```

সার্ভার শুরু করুন:
```bash
mvn spring-boot:run
```

অ্যাপ্লিকেশন চালু হবে `http://localhost:8080` এ।

### ধাপ ৩: অ্যাপ্লিকেশন পরীক্ষা করুন

১. ব্রাউজারে খুলুন `http://localhost:8080`  
২. টেক্সট এরিয়ায় আপনার পোষা প্রাণীর বর্ণনা দিন (যেমন, "একটি খেলাধুলাপ্রিয় গোল্ডেন রিট্রিভার যাকে ফেচ করা খুব ভালো লাগে")  
৩. "Generate Story" ক্লিক করুন AI-প্রস্তুত গল্প পেতে  
৪. অথবা, এক পোষা প্রাণীর ছবি আপলোড করুন স্বয়ংক্রিয়ভাবে বিবরণ তৈরি করার জন্য  
৫. আপনার পোষা প্রাণীর বর্ণনার ওপর ভিত্তি করে সৃজনশীল গল্প দেখুন

## সবকিছু কিভাবে একসাথে কাজ করে

পেট গল্প তৈরি করার সম্পূর্ণ প্রবাহ:

১. **ইউজার ইনপুট**: আপনি ওয়েব ফর্মে আপনার পোষা বর্ণনা করেন  
২. **ফর্ম সাবমিশন**: ব্রাউজার POST রিকোয়েস্ট পাঠায় `/generate-story` এ  
৩. **কন্ট্রোলার প্রসেসিং**: `PetController` ইনপুট যাচাই ও স্যানিটাইজ করে  
৪. **AI সেবা কল**: `StoryService` Azure AI Foundry মডেলে রিকোয়েস্ট পাঠায়  
৫. **গল্প তৈরী**: AI বর্ণনার ভিত্তিতে সৃজনশীল গল্প তৈরি করে  
৬. **রেসপন্স হ্যান্ডলিং**: কন্ট্রোলার গল্প গ্রহণ করে মডেলে যোগ করে  
৭. **টেমপ্লেট রেন্ডারিং**: Thymeleaf `result.html` গল্পসহ রেন্ডার করে  
৮. **প্রদর্শন**: ব্যবহারকারী ব্রাউজারে তৈরি গল্প দেখে

**এরর হ্যান্ডলিং ফ্লো:**
যদি AI সেবা ব্যর্থ হয়:  
১. কন্ট্রোলার এক্সসেপশন ধরে  
২. পূর্বে লেখা টেমপ্লেট ব্যবহার করে বিকল্প গল্প তৈরি করে  
৩. AI অনুপল‌ব্ধতা সম্পর্কিত নোট সহ বিকল্প গল্প দেখায়  
৪. ব্যবহারকারী তবুও ভালো অভিজ্ঞতা পায়

## AI অন্তর্ভুক্তিকরণ বোঝা

### Azure AI Foundry (কী ছাড়া)
অ্যাপ্লিকেশনটি Azure AI Foundry ব্যবহার করে কী ছাড়া অথেন্টিকেশন (Microsoft Entra ID):

```java
// কি ছাড়া প্রমাণীকরণ - কোন API কী নেই
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### প্রম্পট ইঞ্জিনিয়ারিং
সেবা ভালো ফলাফল পেতে সাবধানে প্রম্পট তৈরি করে:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### রেসপন্স প্রসেসিং
AI রেসপন্স বের করা এবং যাচাই করা হয়:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## পরবর্তী ধাপ

আরও উদাহরণের জন্য দেখুন [অধ্যায় ০৪: ব্যবহারিক নমুনা](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**অস্বীকৃতি**:
এই নথিটি AI অনুবাদ পরিষেবা [Co-op Translator](https://github.com/Azure/co-op-translator) ব্যবহার করে অনূদিত হয়েছে। যদিও আমরা শুদ্ধতার জন্য চেষ্টা করি, অনুগ্রহ করে মনে রাখবেন যে স্বয়ংক্রিয় অনুবাদে ত্রুটি বা অসঙ্গতি থাকতে পারে। মূল নথিটি তার স্বভাষায় কর্তৃত্বপূর্ণ উৎস হিসেবে বিবেচিত হওয়া উচিত। গুরুত্বপূর্ণ তথ্যের জন্য পেশাদার মানব অনুবাদ সুপারিশ করা হয়। এই অনুবাদের ব্যবহারে প্রয়োজনীয় ভুল বোঝাবুঝি বা ভুল ব্যাখ্যার জন্য আমরা দায়বদ্ধ নই।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->