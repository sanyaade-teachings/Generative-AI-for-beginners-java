# Pet Story Generator Tutorial for Beginners

## Table of Contents

- [Prerequisites](#prerequisites)
- [Understanding the Project Structure](#understanding-the-project-structure)
- [Core Components Explained](#core-components-explained)
  - [1. Main Application](#1-main-application)
  - [2. Web Controller](#2-web-controller)
  - [3. Story Service](#3-story-service)
  - [4. Web Templates](#4-web-templates)
  - [5. Configuration](#5-configuration)
- [Running the Application](#running-the-application)
- [How It All Works Together](#how-it-all-works-together)
- [Understanding the AI Integration](#understanding-the-ai-integration)
- [Next Steps](#next-steps)

## Prerequisites

शुरू करने से पहले, सुनिश्चित करें कि आपके पास निम्नलिखित हैं:
- Java 21 या इससे ऊपर संस्करण स्थापित है
- निर्भरता प्रबंधन के लिए Maven
- एक Azure AI Foundry मॉडल डिप्लॉयमेंट (इसे `azd up` के साथ प्रदान करें — देखें [Chapter 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), `az login` के साथ साइन इन करें (कीलेस प्रमाणीकरण)
- Java, Spring Boot, और वेब विकास की बुनियादी समझ

## Understanding the Project Structure

पेट स्टोरी प्रोजेक्ट में कई महत्वपूर्ण फ़ाइलें हैं:

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

## Core Components Explained

### 1. Main Application

**फ़ाइल:** `PetStoryApplication.java`

यह हमारे Spring Boot एप्लिकेशन का एंट्री पॉइंट है:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**यह क्या करता है:**
- `@SpringBootApplication` एनोटेशन ऑटो-कॉन्फ़िगरेशन और कंपोनेंट स्कैनिंग सक्षम करता है
- पोर्ट 8080 पर एक एम्बेडेड वेब सर्वर (Tomcat) शुरू करता है
- सभी आवश्यक Spring बीन्स और सर्विसेज़ को स्वतः बनाता है

### 2. Web Controller

**फ़ाइल:** `PetController.java`

यह सभी वेब अनुरोधों और उपयोगकर्ता इंटरैक्शन को संभालता है:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // index.html टेम्पलेट लौटाता है
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // इनपुट सत्यापन
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // सुरक्षा के लिए इनपुट को शुद्ध करें
        String sanitizedDescription = sanitizeInput(description);
        
        // त्रुटि संभालने के साथ कहानी जनरेट करें
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // result.html टेम्पलेट लौटाता है
            
        } catch (Exception e) {
            // यदि AI असफल होता है तो फॉलबैक कहानी का उपयोग करें
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // लंबाई सीमित करें
    }
}
```

**मुख्य विशेषताएं:**

1. **रूट हैंडलिंग**: `@GetMapping("/")` अपलोड फॉर्म दिखाता है, `@PostMapping("/generate-story")` सबमिशन को प्रोसेस करता है
2. **इनपुट सत्यापन**: खाली विवरण और लम्बाई सीमाओं की जांच करता है
3. **सुरक्षा**: XSS हमलों को रोकने के लिए उपयोगकर्ता इनपुट को साफ करता है
4. **त्रुटि प्रबंधन**: AI सेवा विफल होने पर बैकअप कहानियां प्रदान करता है
5. **मॉडल बाइंडिंग**: Spring के `Model` का उपयोग करके डेटा को HTML टेम्प्लेट्स को भेजता है

**बैकअप सिस्टम:**
कंट्रोलर में पहले से लिखे गए कहानी टेम्प्लेट शामिल हैं, जो AI सेवा अनुपलब्ध होने पर उपयोग किए जाते हैं:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // लगातार प्रतिक्रियाओं के लिए विवरण हैश का उपयोग करें
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Story Service

**फ़ाइल:** `StoryService.java`

यह सेवा Azure AI Foundry के साथ बिना कुंजी प्रमाणीकरण के कहानियाँ उत्पन्न करने के लिए संवाद करती है:

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
        
        // Foundry का OpenAI-समतुल्य एंडपॉइंट /openai/v1/ के अंतर्गत है
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Microsoft Entra ID के साथ कीलेस प्रमाणीकरण (कोई API कुंजी नहीं)
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
        
        // AI अनुरोध कॉन्फ़िगर करें
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // प्रतिक्रिया की लंबाई सीमित करें
                .temperature(0.8)          // रचनात्मकता नियंत्रित करें (0.0-1.0)
                .build();
        
        // अनुरोध भेजें और प्रतिक्रिया प्राप्त करें
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**मुख्य घटक:**

1. **OpenAI क्लाइंट**: Azure AI Foundry (कीलेस) के लिए आधिकारिक OpenAI Java SDK का उपयोग करता है
2. **सिस्टम प्रॉम्प्ट**: AI के व्यवहार को पारिवारिक-मित्रवत पालतू कहानियाँ लिखने के लिए निर्धारित करता है
3. **यूजर प्रॉम्प्ट**: विवरण के आधार पर AI को ठीक वही कहानी लिखने के लिए कहता है
4. **पैरामीटर्स**: कहानी की लंबाई और रचनात्मकता स्तर को नियंत्रित करता है
5. **त्रुटि प्रबंधन**: अपवाद फेंकता है जिन्हें कंट्रोलर पकड़ता और प्रबंधित करता है

### 4. Web Templates

**फ़ाइल:** `index.html` (अपलोड फॉर्म)

मुख्य पृष्ठ जहाँ उपयोगकर्ता अपने पालतू के बारे में बताते हैं:

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

**फ़ाइल:** `result.html` (कहानी प्रदर्शन)

उत्पन्न की गई कहानी दिखाता है:

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

**टेम्प्लेट विशेषताएं:**

1. **Thymeleaf इंटीग्रेशन**: डायनामिक कंटेंट के लिए `th:` एट्रिब्यूट्स का उपयोग करता है
2. **रिस्पॉन्सिव डिज़ाइन**: मोबाइल और डेस्कटॉप के लिए CSS स्टाइलिंग
3. **त्रुटि प्रबंधन**: उपयोगकर्ताओं को सत्यापन त्रुटियां दिखाता है
4. **क्लाइंट-साइड प्रोसेसिंग**: इमेज एनालिसिस के लिए JavaScript (Transformers.js का उपयोग)

### 5. Configuration

**फ़ाइल:** `application.properties`

एप्लिकेशन के कॉन्फ़िगरेशन सेटिंग्स:

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

**कॉन्फ़िगरेशन समझाया गया:**

1. **फ़ाइल अपलोड**: 10MB तक की छवियों की अनुमति देता है
2. **लॉगिंग**: निष्पादन के दौरान कौन-सी जानकारी लॉग होती है यह नियंत्रित करता है
3. **Azure AI Foundry**: उपयोग के लिए एन्डपॉइंट और मॉडल डिप्लॉयमेंट निर्दिष्ट करता है (कीलेस प्रमाणीकरण)
4. **सुरक्षा**: संवेदनशील जानकारी उजागर न होने देने के लिए त्रुटि प्रबंधन कॉन्फ़िगरेशन

## Running the Application

### Step 1: Sign In and Set Your Endpoint

प्रमाणीकरण कीलेस है (Microsoft Entra ID), इसलिए कोई API key नहीं है। साइन इन करें और अपने Foundry Endpoint सेट करें:

**Windows (Command Prompt):**
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

**यह आवश्यक क्यों है:**
- Azure AI Foundry Microsoft Entra ID का उपयोग करता है इन्फरेंस अनुरोधों को प्रमाणित करने के लिए
- कीलेस प्रमाणीकरण का मतलब है कि आपके स्रोत कोड या वातावरण में कोई सीक्रेट्स नहीं
- आपके खाते को संसाधन पर **Cognitive Services OpenAI User** भूमिका की आवश्यकता है

### Step 2: Build and Run

प्रोजेक्ट डायरेक्टरी पर जाएं:
```bash
cd 04-PracticalSamples/petstory
```

एप्लिकेशन बनाएं:
```bash
mvn clean compile
```

सर्वर शुरू करें:
```bash
mvn spring-boot:run
```

एप्लिकेशन `http://localhost:8080` पर शुरू होगा।

### Step 3: Test the Application

1. **ब्राउज़र में खोलें** `http://localhost:8080`
2. **अपने पालतू का वर्णन करें** टेक्स्ट क्षेत्र में (उदा., "एक खिलंदड़ गोल्डन रिट्रीवर जो फ़ेच करना पसंद करता है")
3. **"Generate Story" पर क्लिक करें** AI उत्पन्न कहानी पाने के लिए
4. **या**, पालतू की तस्वीर अपलोड करें स्वतः विवरण उत्पन्न करने के लिए
5. **रचनात्मक कहानी देखें** आपके पालतू के विवरण के आधार पर

## How It All Works Together

यहाँ वह संपूर्ण प्रक्रिया है जब आप पालतू कहानी उत्पन्न करते हैं:

1. **उपयोगकर्ता इनपुट**: आप वेब फॉर्म पर अपने पालतू का वर्णन करते हैं
2. **फॉर्म सबमिशन**: ब्राउज़र `/generate-story` पर POST अनुरोध भेजता है
3. **कंट्रोलर प्रोसेसिंग**: `PetController` इनपुट को सत्यापित और साफ करता है
4. **AI सेवा कॉल**: `StoryService` Azure AI Foundry मॉडल को अनुरोध भेजता है
5. **कहानी उत्पादन**: AI विवरण के आधार पर रचनात्मक कहानी बनाता है
6. **प्रतिक्रिया प्रबंधन**: कंट्रोलर कहानी प्राप्त करता है और मॉडल में जोड़ता है
7. **टेम्प्लेट रेंडरिंग**: Thymeleaf `result.html` को कहानी के साथ रेंडर करता है
8. **प्रदर्शन**: उपयोगकर्ता ब्राउज़र में उत्पन्न कहानी देखता है

**त्रुटि प्रबंधन प्रवाह:**
यदि AI सेवा विफल होती है:
1. कंट्रोलर अपवाद पकड़ता है
2. पहले से लिखे गए टेम्प्लेट का उपयोग कर बैकअप कहानी बनाता है
3. AI अनुपलब्धता के बारे में नोट के साथ बेकअप कहानी दिखाता है
4. उपयोगकर्ता को फिर भी एक कहानी मिलती है, जिससे अच्छा उपयोगकर्ता अनुभव सुनिश्चित होता है

## Understanding the AI Integration

### Azure AI Foundry (keyless)
एप्लिकेशन Azure AI Foundry का उपयोग करता है कीलेस प्रमाणीकरण (Microsoft Entra ID) के साथ:

```java
// बिना कुंजी प्रमाणीकरण - कोई API कुंजी नहीं
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Prompt Engineering
सेवा अच्छे परिणाम पाने के लिए सावधानीपूर्वक बनाए गए प्रॉम्प्ट्स का उपयोग करती है:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Response Processing
AI प्रतिक्रिया को निकाला और सत्यापित किया जाता है:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Next Steps

अधिक उदाहरणों के लिए देखें [Chapter 04: Practical samples](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**अस्वीकरण**:
इस दस्तावेज़ का अनुवाद AI अनुवाद सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) का उपयोग करके किया गया है। जबकि हम सटीकता के लिए प्रयास करते हैं, कृपया ध्यान दें कि स्वचालित अनुवादों में त्रुटियाँ या अशुद्धियाँ हो सकती हैं। मूल दस्तावेज़ अपनी मूल भाषा में ही प्रामाणिक स्रोत माना जाना चाहिए। महत्वपूर्ण जानकारी के लिए, पेशेवर मानव अनुवाद की सिफारिश की जाती है। इस अनुवाद के उपयोग से उत्पन्न किसी भी गलतफहमी या गलत व्याख्या के लिए हम उत्तरदायी नहीं हैं।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->