# सुरु गर्नेहरूको लागि पेट कथा निर्माणकर्ता ट्युटोरियल

## सcontents

- [पूर्वआवश्यकताहरू](#पूर्वआवश्यकताहरू)
- [परियोजना संरचना बुझ्न](#परियोजना-संरचना-बुझ्न)
- [कोर कम्पोनेंटहरू व्याख्या गरिएको](#कोर-कम्पोनेंटहरू-व्याख्या-गरिएको)
  - [१. मुख्य अनुप्रयोग](#१-मुख्य-अनुप्रयोग)
  - [२. वेब कन्ट्रोलर](#२-वेब-कन्ट्रोलर)
  - [३. कथा सेवा](#३-कथा-सेवा)
  - [४. वेब टेम्प्लेटहरू](#४-वेब-टेम्प्लेटहरू)
  - [५. कन्फिगरेसन](#५-कन्फिगरेसन)
- [अनुप्रयोग चलाउने तरिका](#अनुप्रयोग-चलाउने-तरिका)
- [सबै कसरी सँगै काम गर्छ](#सबै-कसरी-सँगै-काम-गर्छ)
- [AI एकीकरण बुझ्न](#ai-एकीकरण-बुझ्न)
- [अर्को कदमहरू](#अर्को-कदमहरू)

## पूर्वआवश्यकताहरू

सुरु गर्नुअघि, सुनिश्चित गर्नुहोस् तपाईंले:
- जाभा २१ वा माथि स्थापना गरेका हुनुहुन्छ
- डिपेन्डेन्सी व्यवस्थापनका लागि म्याभेन
- Azure AI Foundry मोडेल डिप्लोयमेन्ट (यसलाई `azd up` सँग प्रोभिजन गर्नुहोस् — हेर्नुहोस् [अध्याय २](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), `az login` (किललेस प्रमाणीकरण) द्वारा साइन इन गरेको
- जाभा, स्प्रिंग बूट र वेब विकासको आधारभूत समझ

## परियोजना संरचना बुझ्न

पेट कथा परियोजनामा केहि महत्वपूर्ण फाइलहरू छन्:

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

## कोर कम्पोनेंटहरू व्याख्या गरिएको

### १. मुख्य अनुप्रयोग

**फाइल:** `PetStoryApplication.java`

यो हाम्रो स्प्रिंग बूट अनुप्रयोगको प्रवेश बिन्दु हो:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**यसले के गर्छ:**
- `@SpringBootApplication` एनोटेशनले अटो-कन्फिगरेसन र कम्पोनेन्ट स्क्यानिङ सक्षम पार्दछ
- पोर्ट ८०८० मा एम्बेडेड वेब सर्भर (टम्क्याट) सुरु गर्दछ
- आवश्यक सबै स्प्रिंग बीनहरू र सेवाहरू स्वचालित रूपमा सिर्जना गर्दछ

### २. वेब कन्ट्रोलर

**फाइल:** `PetController.java`

यसले सबै वेब अनुरोध र प्रयोगकर्ता अन्तरक्रियाहरूको सम्हाल गर्छ:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // index.html टेम्प्लेट फर्काउँछ
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // इनपुट प्रमाणीकरण
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // सुरक्षा का लागि इनपुट सफा गर्नुहोस्
        String sanitizedDescription = sanitizeInput(description);
        
        // त्रुटि व्यवस्थापनसँग कथा सिर्जना गर्नुहोस्
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // result.html टेम्प्लेट फर्काउँछ
            
        } catch (Exception e) {
            // AI असफल भएमा वैकल्पिक कथा प्रयोग गर्नुहोस्
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // लम्बाइ सीमित गर्नुहोस्
    }
}
```

**प्रमुख विशेषताहरू:**

१. **रूट ह्यान्डलिङ**: `@GetMapping("/")` अपलोड फारम देखाउँछ, `@PostMapping("/generate-story")` सबमिशनहरू प्रक्रिया गर्छ
२. **इनपुट जाँच**: खाली विवरण र लम्बाइ सीमाहरू जाँच गर्दछ
३. **सुरक्षा**: प्रयोगकर्ताको इनपुटलाई XSS आक्रमणबाट बचाउन सफा पार्दछ
४. **त्रुटि ह्यान्डलिङ**: AI सेवा असफल हुँदा फ्यालब्याक कथाहरू प्रदान गर्दछ
५. **मोडेल बाँध्ने**: स्प्रिंगको `Model` प्रयोग गरेर HTML टेम्प्लेटमा डाटा पठाउँछ

**फ्यालब्याक प्रणाली:**
कन्ट्रोलरले पहिले देखि लेखिएका कथा टेम्प्लेटहरू समावेश गर्दछ जुन AI सेवा अनुपलब्ध हुँदा प्रयोग गरिन्छ:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // निरन्तर प्रतिक्रिया को लागी वर्णन ह्यास प्रयोग गर्नुहोस्
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### ३. कथा सेवा

**फाइल:** `StoryService.java`

यस सेवाले Azure AI Foundry सँग मुख्य प्रमाणीकरण (किललेस) प्रयोग गरेर कथा सिर्जना गर्न संवाद गर्छ:

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
        
        // फाउन्ड्रीको OpenAI-अनुकूल अन्तबिन्दु /openai/v1/ अन्तर्गत छ
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Microsoft Entra ID सँग कुञ्जीबिहीन प्रमाणीकरण (API कुञ्जी छैन)
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
        
        // AI अनुरोध कन्फिगर गर्नुहोस्
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // प्रतिक्रिया लम्बाइ सीमित गर्नुहोस्
                .temperature(0.8)          // सिर्जनशीलता नियन्त्रण गर्नुहोस् (0.0-1.0)
                .build();
        
        // अनुरोध पठाउनुहोस् र प्रतिक्रिया प्राप्त गर्नुहोस्
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**प्रमुख कम्पोनेंटहरू:**

१. **OpenAI क्लाइन्ट**: Azure AI Foundry को लागि आधिकारिक OpenAI जाभा SDK कन्फिगर गरिएको (किललेस)
२. **सिस्टम प्राम्प्ट**: AI को व्यवहारलाई पारिवारिक-मित्रवत पेट कथाहरू लेख्न सेट गर्दछ
३. **प्रयोगकर्ता प्राम्प्ट**: विवरणको आधारमा AI लाई कथा के लेख्ने हो स्पष्ट बताउँछ
४. **प्यारामिटरहरू**: कथा लामो र सिर्जनात्मक स्तर नियन्त्रण गर्दछ
५. **त्रुटि ह्यान्डलिङ**: अपवादहरू फ्याँक्छ जसलाई कन्ट्रोलरले समात्छ र ह्यान्डल गर्छ

### ४. वेब टेम्प्लेटहरू

**फाइल:** `index.html` (अपलोड फारम)

मुख्य पृष्ठ जहाँ प्रयोगकर्ताले आफ्नो पेट वर्णन गर्छन्:

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

निर्मित कथा देखाउँछ:

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

**टेम्प्लेट सुविधाहरू:**

१. **थाइमलिफ इंटीग्रेशन**: गतिशील सामग्रीका लागि `th:` विशेषताहरू प्रयोग गर्दछ
२. **रिस्पोन्सिभ डिजाइन**: मोबाइल र डेस्कटपका लागि CSS स्टाइलिङ
३. **त्रुटि ह्यान्डलिङ**: प्रयोगकर्तालाई प्रमाणीकरण त्रुटिहरू देखाउँछ
४. **क्लाइन्ट-साइड प्रोसेसिङ**: छवि विश्लेषणका लागि जाभास्क्रिप्ट (Transformers.js प्रयोग गरी)

### ५. कन्फिगरेसन

**फाइल:** `application.properties`

अनुप्रयोगका कन्फिगरेसन सेटिङहरू:

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

**कन्फिगरेसन व्याख्या:**

१. **फाइल अपलोड**: १०एमबी सम्मका छविहरू अनुमति दिन्छ
२. **लगिङ**: कार्यान्वयनको समयमा के सूचना लग गरिन्छ नियन्त्रण गर्दछ
३. **Azure AI Foundry**: प्रयोग गर्ने अन्तिम बिन्दु र मोडेल डिप्लोयमेन्ट निर्दिष्ट गर्दै (किललेस प्रमाणीकरण)
४. **सुरक्षा**: संवेदनशील जानकारी प्रकट नगर्ने गरी त्रुटि ह्यान्डलिङ कन्फिगरेसन

## अनुप्रयोग चलाउने तरिका

### चरण १: साइन इन र अन्तिम बिन्दु सेट गर्नुहोस्

प्रमाणीकरण किललेस (Microsoft Entra ID) छ, त्यसैले कुनै API कुञ्जी छैन। साइन इन गर्नुहोस् र Foundry अन्तिम बिन्दु सेट गर्नुहोस्:

**विन्डोज (कमाण्ड प्रॉम्प्ट):**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**विन्डोज (पावरशेल):**
```powershell
az login
$env:AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
```

**लिनक्स/म्याकओएस:**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**किन आवश्यक छ:**
- Azure AI Foundry ले Microsoft Entra ID प्रयोग गरेर अनुमोदन अनुरोधहरू गर्छ
- किललेस प्रमाणीकरणको अर्थ स्रोत कोड वा वातावरणमा कुनै गोप्य कुरा छैन
- तपाईंको एकाउन्टलाई स्रोतमा **Cognitive Services OpenAI User** भूमिका आवश्यक छ

### चरण २: बिल्ड र चलाउनुहोस्

परियोजना निर्देशिकामा जानुहोस्:
```bash
cd 04-PracticalSamples/petstory
```

अनुप्रयोग बिल्ड गर्नुहोस्:
```bash
mvn clean compile
```

सर्भर सुरु गर्नुहोस्:
```bash
mvn spring-boot:run
```

अनुप्रयोग `http://localhost:8080` मा सुरू हुनेछ।

### चरण ३: अनुप्रयोग परीक्षण गर्नुहोस्

१. आफ्नो ब्राउजरमा `http://localhost:8080` खोल्नुहोस्  
२. पाठ क्षेत्रमा तपाईंको पेट वर्णन गर्नुहोस् (जस्तै, "फेच खेल्न मन पर्ने एक खेलाउ सुनौलो retriever")  
३. "Generate Story" क्लिक गर्नुहोस् AI द्वारा सिर्जित कथा प्राप्त गर्न  
४. विकल्प स्वरूप, पेटको तस्वीर अपलोड गरी स्वतः विवरण प्राप्त गर्नुहोस्  
५. तपाईंको पेटको विवरण आधारमा सिर्जनात्मक कथा हेर्नुहोस्  

## सबै कसरी सँगै काम गर्छ

पेट कथा सिर्जना गर्दा सम्पूर्ण प्रवाह यस्तो हुन्छ:

१. **प्रयोगकर्ता इनपुट**: तपाईं वेब फारममा आफ्नो पेट वर्णन गर्नुहुन्छ  
२. **फारम सबमिशन**: ब्राउजरले POST अनुरोध `/generate-story` मा पठाउँछ  
३. **कन्ट्रोलर प्रक्रिया**: `PetController` इनपुटलाई प्रमाणीकरण र सफा गर्दछ  
४. **AI सेवा कल**: `StoryService` ले Azure AI Foundry मोडेललाई अनुरोध पठाउँछ  
५. **कथा सिर्जना**: AI ले विवरण आधारित सिर्जनात्मक कथा तयार पार्छ  
६. **उत्तर प्रक्रिया**: कन्ट्रोलरले कथालाई प्राप्त गरी मोडेलमा थप्छ  
७. **टेम्प्लेट रेंडरिङ**: थाइमलिफ `result.html` टेम्प्लेट कथासहित रेंडर गर्छ  
८. **प्रदर्शन**: प्रयोगकर्ताले ब्राउजरमा कथालाई हेर्छ  

**त्रुटि ह्यान्डलिङ प्रवाह:**  
AI सेवा असफल भए:  
१. कन्ट्रोलरले अपवाद समात्छ  
२. पहिले लेखिएका टेम्प्लेट प्रयोग गरी फ्यालब्याक कथा सिर्जना गर्छ  
३. AI अनुपलब्धता सम्बन्धी नोटसहित फ्यालब्याक कथा देखाउँछ  
४. प्रयोगकर्ताले अझै कथा पाउँछ, राम्रो प्रयोगकर्ता अनुभव सुनिश्चित गर्दै

## AI एकीकरण बुझ्न

### Azure AI Foundry (किललेस)
अनुप्रयोग Azure AI Foundry सँग किललेस प्रमाणीकरण (Microsoft Entra ID) प्रयोग गर्दछ:

```java
// कुञ्जीविनाको प्रमाणीकरण - कुनै API कुञ्जी छैन
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### प्राम्प्ट इन्जिनियरिङ
सेवाले राम्रो नतिजा लिन सावधानीपूर्वक बनेका प्राम्प्टहरू प्रयोग गर्दछ:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### प्रतिक्रिया प्रक्रिया
AI प्रतिक्रियालाई निकालेर प्रमाणीकरण गर्दछ:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## अर्को कदमहरू

थप उदाहरणहरूका लागि हेर्नुहोस् [अध्याय ०४: व्यावहारिक नमूना](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**अस्वीकरण**:
यो दस्तावेज़ AI अनुवाद सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) प्रयोग गरेर अनुवाद गरिएको हो। हामी सही हुन प्रयास गर्छौं, तर कृपया जानकार हुनुस् कि स्वचालित अनुवादमा त्रुटिहरू वा अशुद्धताहरू हुन सक्छन्। मूल दस्तावेज़ यसको मूल भाषामा आधिकारिक स्रोत मानिनुपर्छ। महत्वपूर्ण जानकारीका लागि व्यावसायिक मानव अनुवाद सिफारिस गरिन्छ। यस अनुवादको प्रयोगबाट उत्पन्न कुनै पनि गलत बुझाइ वा त्रुटिको लागि हामी जिम्मेवार छैनौं।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->