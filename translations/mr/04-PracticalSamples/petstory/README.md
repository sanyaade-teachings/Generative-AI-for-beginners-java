# प्राणी कथा जनरेटर ट्युटोरियल नवशिक्यांसाठी

## अनुक्रमणिका

- [पूर्वशर्ता](#पूर्वशर्ता)
- [प्रकल्प रचनेचे आकलन](#प्रकल्प-रचनेचे-आकलन)
- [मुख्य घटकांचे स्पष्टीकरण](#मुख्य-घटकांचे-स्पष्टीकरण)
  - [1. मुख्य अनुप्रयोग](#1-मुख्य-अनुप्रयोग)
  - [2. वेब कंट्रोलर](#2-वेब-कंट्रोलर)
  - [3. कथा सेवा](#3-कथा-सेवा)
  - [4. वेब टेम्प्लेट्स](#4-वेब-टेम्प्लेट्स)
  - [5. संयोजन](#5-संयोजन)
- [अॅप्लिकेशन चालविणे](#अॅप्लिकेशन-चालविणे)
- [हे कसे एकत्र काम करते](#हे-कसे-एकत्र-काम-करते)
- [एआय समाकलन समजून घेणे](#एआय-समाकलन-समजून-घेणे)
- [पुढील पावले](#पुढील-पावले)

## पूर्वशर्ता

प्रारंभ करण्यापूर्वी, खालील गोष्टींची खात्री करा:
- जावा 21 किंवा त्यापेक्षा उच्च आवृत्ती स्थापित आहे
- अवलंबित्व व्यवस्थापनासाठी मावेन वापरले आहे
- Azure AI Foundry मॉडेल तैनात केले आहे (ते `azd up` ने तयार करा — पाहा [अध्याय 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), `az login` (कीलेस प्रमाणीकरण) वापरून साइन इन केले आहे
- जावा, स्प्रिंग बूट आणि वेब विकासाची मूलभूत समज

## प्रकल्प रचनेचे आकलन

पाळीव प्राणी कथा प्रकल्पामध्ये काही महत्त्वाच्या फाइल्स आहेत:

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

## मुख्य घटकांचे स्पष्टीकरण

### 1. मुख्य अनुप्रयोग

**फाइल:** `PetStoryApplication.java`

हा आमच्या स्प्रिंग बूट अनुप्रयोगाचा प्रवेश बिंदू आहे:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**हे काय करते:**
- `@SpringBootApplication` अ‍ॅनोटेशन स्वयंचलित संरचना आणि घटक स्कॅनिंग सक्षम करते
- पोर्ट 8080 वर एंबेडेड वेब सर्व्हर (टॉमकॅट) चालू करते
- आवश्यक असलेले सर्व स्प्रिंग बीन आणि सेवा आपोआप तयार करते

### 2. वेब कंट्रोलर

**फाइल:** `PetController.java`

हे सर्व वेब विनंत्या आणि वापरकर्ता संवाद हाताळते:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // index.html टेम्प्लेट परत करते
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // इनपुट वैधता तपासणी
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // सुरक्षिततेसाठी इनपुट स्वच्छ करा
        String sanitizedDescription = sanitizeInput(description);
        
        // त्रुटी हाताळणीसह कथा तयार करा
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // result.html टेम्प्लेट परत करते
            
        } catch (Exception e) {
            // AI अयशस्वी झाल्यास फॉलबॅक कथा वापरा
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // लांबी मर्यादित करा
    }
}
```

**मुख्य वैशिष्ट्ये:**

1. **मार्ग हाताळणी**: `@GetMapping("/")` अपलोड फॉर्म दर्शवतो, `@PostMapping("/generate-story")` सबमिशन्स प्रक्रिया करतो
2. **इनपुट पडताळणी**: वर्णन रिक्त आहे की नाही आणि लांबी मर्यादा तपासते
3. **सुरक्षा**: वापरकर्त्याच्या इनपुटचे एक्सएसएस हल्ले टाळण्यासाठी शुद्धीकरण करते
4. **त्रुटी हाताळणी**: जेव्हा एआय सेवा अयशस्वी होते तेव्हा पर्यायी कहाण्या प्रदान करते
5. **मॉडेल बाइंडिंग**: स्प्रिंगच्या `Model` चा वापर करून HTML टेम्प्लेट्सना डेटा पाठवते

**पर्यायी व्यवस्था:**
कंट्रोलरमध्ये पूर्वलिखित कथा टेम्प्लेट्स असतात जे AI सेवा अनुपलब्ध असताना वापरले जातात:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // सुसंगत प्रतिसादांसाठी वर्णन हॅश वापरा
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. कथा सेवा

**फाइल:** `StoryService.java`

ही सेवा Azure AI Foundry शी संवाद साधून कीलेस प्रमाणीकरण वापरून कथा तयार करते:

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
        
        // Foundry चा OpenAI-सुसंगत एंडपॉइंट /openai/v1/ अंतर्गत आहे
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Microsoft Entra ID सह कीलेस प्रमाणीकरण (कोणतीही API की नाही)
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
        
        // AI विनंती कॉन्फिगर करा
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // प्रतिसादाची लांबाई मर्यादित करा
                .temperature(0.8)          // सर्जनशीलता नियंत्रित करा (0.0-1.0)
                .build();
        
        // विनंती पाठवा आणि प्रतिसाद मिळवा
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**मुख्य घटक:**

1. **OpenAI क्लायंट**: Azure AI Foundry साठी अधिकृत OpenAI जावा SDK वापरते (कीलेस)
2. **सिस्टम प्रॉम्प्ट**: AI चे वर्तन पाळीव प्राण्यांच्या कुटुंब-सुसंगत कथा लिहिण्यासाठी सेट करते
3. **वापरकर्ता प्रॉम्प्ट**: वर्णनानुसार AI ला नेमकी कोणती कथा लिहावी हे सांगते
4. **पॅरामीटर्स**: कथांची लांबी आणि सर्जनशीलता नियंत्रित करते
5. **त्रुटी हाताळणी**: अपवाद टाकते जे कंट्रोलर पकडते आणि हाताळते

### 4. वेब टेम्प्लेट्स

**फाइल:** `index.html` (अपलोड फॉर्म)

मुख्य पृष्ठ जिथे वापरकर्ते त्यांच्या प्राण्यांचे वर्णन करतात:

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

**फाइल:** `result.html` (कथा प्रदर्शन)

निर्मित कथा दर्शवते:

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

**टेम्प्लेट वैशिष्ट्ये:**

1. **थाइमलीफ समाकलन**: गतिशील सामग्रीसाठी `th:` अ‍ॅट्रीब्युट्स वापरले जातात
2. **उत्तरदायी डिझाईन**: मोबाइल आणि डेस्कटॉपसाठी CSS शैलीकरण
3. **त्रुटी हाताळणी**: वापरकर्त्यांना पडताळणी त्रुटी दाखविते
4. **क्लायंट-साइड प्रक्रिया**: इमेज विश्लेषणासाठी JavaScript वापर (Transformers.js वापरून)

### 5. संयोजन

**फाइल:** `application.properties`

अॅप्लिकेशनसाठी संयोजन सेटिंग्ज:

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

**संयोजन स्पष्टीकरण:**

1. **फाइल अपलोड**: 10MB पर्यंत इमेजेसची अनुमती देते
2. **लॉगिंग**: कार्यान्वयनादरम्यान कोणती माहिती लॉग करायची ते नियंत्रित करते
3. **Azure AI Foundry**: वापरण्यासाठी एंडपॉइंट आणि मॉडेल तैनात निर्दिष्ट करते (कीलेस प्रमाणीकरण)
4. **सुरक्षा**: संवेदनशील माहिती उघड न होण्यासाठी त्रुटी हाताळणी संयोजन

## अॅप्लिकेशन चालविणे

### पाऊल 1: साइन इन करा आणि तुमचा एंडपॉइंट सेट करा

प्रमाणीकरण कीलेस आहे (Microsoft Entra ID), त्यामुळे कोणतीही API की नाही. साइन इन करा आणि तुमचा Foundry एंडपॉइंट सेट करा:

**विंडोज (कमांड प्रॉम्प्ट):**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**विंडोज (पॉवरशेल):**
```powershell
az login
$env:AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
```

**लिनक्स/macOS:**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**हे का आवश्यक आहे:**
- Azure AI Foundry Microsoft Entra ID वापरून इन्फरन्स विनंत्या प्रमाणीकरण करते
- कीलेस प्रमाणीकरण म्हणजे तुमच्या स्रोत कोड किंवा वातावरणात कोणतेही गुपित नाहीत
- तुमच्या खात्याला त्या संसाधनावर **Cognitive Services OpenAI User** भूमिका हवी आहे

### पाऊल 2: बिल्ड आणि चालवा

प्रकल्प डिरेक्टरीमध्ये बदला:
```bash
cd 04-PracticalSamples/petstory
```

अॅप्लिकेशन बिल्ड करा:
```bash
mvn clean compile
```

सर्व्हर सुरू करा:
```bash
mvn spring-boot:run
```

अॅप्लिकेशन `http://localhost:8080` वर सुरू होईल.

### पाऊल 3: अॅप्लिकेशन चाचणी करा

1. **ब्राउझरमध्ये उघडा** `http://localhost:8080`
2. **तुमच्या प्राण्याचे वर्णन करा** (उदा., "एक खेळकर गोल्डन रिट्रीव्हर जो फेच खेळायला आवडतो")
3. **"कथा तयार करा" वर क्लिक करा** AI-निर्मित कथा प्राप्त करण्यासाठी
4. **पर्यायीपणे**, पाळीव प्राण्याचा फोटो अपलोड करा आणि वर्णन आपोआप तयार करा
5. **तयार केलेली सर्जनशील कथा पाहा** तुमच्या प्राण्याच्या वर्णनावर आधारित

## हे कसे एकत्र काम करते

पाळीव प्राणी कथा तयार करताना पूर्ण प्रक्रिया अशी आहे:

1. **वापरकर्ता इनपुट**: तुम्ही वेब फॉर्मवर तुमच्या प्राण्याचे वर्णन करता
2. **फॉर्म सबमिशन**: ब्राउझर POST विनंती `/generate-story` कडे पाठवतो
3. **कंट्रोलर प्रक्रिया**: `PetController` इनपुटची पडताळणी आणि शुद्धीकरण करतो
4. **एआय सेवा कॉल**: `StoryService` Azure AI Foundry मॉडेलला विनंती पाठवतो
5. **कथा निर्मिती**: AI वर्णनानुसार सर्जनशील कथा तयार करतो
6. **प्रतिसाद हाताळणी**: कंट्रोलर कथाअंगीकृत करून मॉडेलमध्ये जोडतो
7. **टेम्प्लेट रेंडरिंग**: थाइमलीफ `result.html` कथेसह रेंडर करतो
8. **प्रदर्शन**: वापरकर्त्याला ब्राउझरमध्ये तयार कथा दिसते

**त्रुटी हाताळणी प्रक्रिया:**
जर AI सेवा अयशस्वी झाली:
1. कंट्रोलर अपवाद पकडतो
2. पूर्वलिखित टेम्प्लेट्स वापरून पर्यायी कथा तयार करतो
3. AI अनुपलब्धतेबाबत सूचना असलेली पर्यायी कथा दर्शवतो
4. वापरकर्त्याला अजूनही कथा मिळते, चांगला वापरकर्ता अनुभव सुनिश्चित होतो

## एआय समाकलन समजून घेणे

### Azure AI Foundry (कीलेस)
अॅप्लिकेशन Microsoft Entra ID सह कीलेस प्रमाणीकरण वापरून Azure AI Foundry वापरते:

```java
// कीलेस प्रमाणीकरण - कोणतीही API की नाही
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### प्रॉम्प्ट अभियांत्रिकी
सेवा चांगले परिणाम मिळवण्यासाठी काळजीपूर्वक तयार केलेले प्रॉम्प्ट वापरते:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### प्रतिसाद प्रक्रिया
AI प्रतिसाद काढून घेऊन पडताळला जातो:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## पुढील पावले

अधिक उदाहरणांसाठी, पहा [अध्याय 04: व्यावहारिक नमुने](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**अस्वीकरण**:
हा दस्तऐवज AI भाषांतर सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) चा वापर करून अनुवादित केला आहे. जरी आम्ही अचूकतेसाठी प्रयत्न करतो, तरी कृपया लक्षात घ्या की स्वयंचलित भाषांतरांमध्ये त्रुटी किंवा अचूकतेची कमतरता असू शकते. मूळ दस्तऐवज त्याच्या मूळ भाषेत अधिकृत स्रोत मानला पाहिजे. महत्त्वाची माहिती असल्यास, व्यावसायिक मानवी भाषांतराची शिफारस केली जाते. या भाषांतराच्या वापरामुळे उद्भवणाऱ्या कोणत्याही गैरसमज किंवा चुकीच्या अर्थलावणीसाठी आम्ही जबाबदार नाही.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->