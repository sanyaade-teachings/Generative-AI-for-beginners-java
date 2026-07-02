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

முதல் தொடங்குவதற்கு முன், நீங்கள் கீழ்காணும் விஷயங்களை உறுதி செய்திருக்க வேண்டும்:
- Java 21 அல்லது அதற்கு மேற்பட்ட பதிப்பு நிறுவப்பட்டிருக்க வேண்டும்
- சார்பு மேலாண்மைக்காக Maven
- Azure AI Foundry மாதிரி விவரத்தை (மாடல்) உருவாக்கியிருத்தல் (அதை `azd up` மூலம் செயற்படுத்தவும் — [அத்தியாயம் 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md) காண்க), `az login` (கீ இல்லாத அங்கீகாரம்) கொண்டு உள்நுழைந்திருத்தல்
- Java, Spring Boot மற்றும் வலை மேம்பாட்டில் அடிப்படை புரிதல்

## Understanding the Project Structure

pet story திட்டத்தில் முக்கியமான சில கோப்புகள் உள்ளன:

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

**கோப்பு:** `PetStoryApplication.java`

இது நமது Spring Boot பயன்பாட்டின் நுழைவு புள்ளி:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**இது என்ன செய்கிறது:**
- `@SpringBootApplication` என்ற குறியீடு தானாக கட்டமைப்பு மற்றும் கூறுகள் சுரங்கம் செயல்படுத்துகிறது
- தொகுக்கப்பட்ட வலை சேவையகம் (Tomcat) ஐ 8080 போர்டில் துவக்குகிறது
- தேவையான அனைத்து Spring பூச்சிகள் மற்றும் சேவைகளை தானாக உருவாக்குகிறது

### 2. Web Controller

**கோப்பு:** `PetController.java`

இது அனைத்து வலை கோரிக்கைகள் மற்றும் பயனர் தொடர்புகளை கையாள்கிறது:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // index.html ஊடகச்சாரம்பை 반환ுகிறது
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // உள்ளீட்டை சரிபார்த்தல்
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // பாதுகாப்புக்காக உள்ளீட்டை சுத்திகரிக்கவும்
        String sanitizedDescription = sanitizeInput(description);
        
        // பிழை처리 உடன் கதை உருவாக்கவும்
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // result.html ஊடகச்சாரம்பை 반환ுகிறது
            
        } catch (Exception e) {
            // AI தோல்வியடைந்தால் மாற்று கதையைப் பயன்படுத்தவும்
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // நீண்டதன்மையை கட்டுப்படுத்து
    }
}
```

**முக்கிய அம்சங்கள்:**

1. **வழித்தொடர் கையாளல்**: `@GetMapping("/")` பதிவேற்ற படிவத்தை காட்டுகிறது, `@PostMapping("/generate-story")` சமர்ப்பிப்புகளை செயலாக்குகிறது
2. **உள்ளீடு சரிபார்ப்பு**: Description காலியாக இருக்க கூடாது மற்றும் நீளம் வரம்பில் இருக்க வேண்டும்
3. **பாதுகாப்பு**: பயனர் உள்ளீட்டை XSS தாக்குதல்களைத் தடுப்பதற்கான சுத்திகரிப்பு
4. **பிழை கையாளல்**: AI சேவையின் தோல்வியால்Fallback கதைகள் வழங்குகிறது
5. **மாதிரி கட்டமைப்பை கட்டமைத்தல்**: Spring இன் `Model` பயன்படுத்தி HTML வார்ப்புருக்களுக்கு தரவு அனுப்புகிறது

**Fallback அமைப்பு:**
AI சேவை கிடைக்காதபோது முதலில் எழுதப்பட்ட கதையாளர்கள் பயன்படுத்தப்படுகின்றன:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // ஒரே மாதிரியான பதில்களுக்கு விவரங்கள் ஹாஷ் பயன்படுத்தவும்
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Story Service

**கோப்பு:** `StoryService.java`

இந்த சேவை Azure AI Foundry உடன் தொடர்பு கொண்டு கதைகளை உருவாக்குகிறது, கீ இல்லாத அங்கீகாரத்தை பயன்படுத்துகிறது:

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
        
        // Foundry இன் OpenAI-க்கு இணக்கமான குடிவரை /openai/v1/ இல் உள்ளது
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Microsoft Entra ID உடன் முக்கியமில்லா அங்கீகாரம் (API விசை இல்லை)
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
        
        // AI கோரிக்கையை அமைக்கவும்
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // பதிலின் நீளத்தை வரம்பிடு
                .temperature(0.8)          // படைப்பாற்றலை கட்டுப்படுத்து (0.0-1.0)
                .build();
        
        // கோரிக்கை அனுப்பி பதிலை பெறு
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**முக்கிய கூறுகள்:**

1. **OpenAI கிளைண்ட்**: Azure AI Foundry க்கு அமைக்கப்பட்ட OpenAI Java SDK பயன்படுத்துகிறது (கீ இல்லாதது)
2. **கணினி (System) தூண்டுகோள்**: குடும்ப நண்பர் விலங்கு கதைகளை எழுத AI நடத்தையை அமைக்கிறது
3. **பயனர் தூண்டுகோள்**: AIக்கு சரியான கதையை விவரத்தின் அடிப்படையில் எழுத சொல்கிறது
4. **அளவுருக்கள்**: கதை நீளம் மற்றும் படைப்பாற்றல் மட்டத்தை கட்டுப்படுத்துகிறது
5. **பிழை கையாளல்**: கட்டுப்படுத்தப்படாத விதிவிலக்குகளை உருவாக்குகிறது, அதனை கட்டுப்படுத்துகிறது

### 4. Web Templates

**கோப்பு:** `index.html` (பதிவேற்ற படிவம்)

பயனர்கள் தங்கள் விலங்குகளை விவரிக்கும் முதன்மை பக்கம்:

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

**கோப்பு:** `result.html` (கதை காட்சி)

உருவாக்கிய கதையை காட்சி படுத்துகிறது:

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

**வார்ப்புரு அம்சங்கள்:**

1. **Thymeleaf ஒருங்கிணைப்பு**: தானாக உள்ளடக்கத்துக்கு `th:` பண்புகளை பயன்படுத்துகிறது
2. **அதிரடி வடிவமைப்பு**: CSS ஸ்டைலிங் மொபைல் மற்றும் டெஸ்க்டாப்புக்காக
3. **பிழை கையாளல்**: பயனர்களுக்கு சரிபார்ப்பு பிழைகள் காட்டுகிறது
4. **வாடிக்கையாளர் பக்கம் செயலாக்கம்**: படங்களை பகுப்பாய்வு செய்ய JavaScript (Transformers.js பயன்படுத்துகிறது)

### 5. Configuration

**கோப்பு:** `application.properties`

பயன்பாட்டிற்கான கட்டமைப்பு அமைப்புகள்:

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

**கட்டமைப்பின் விளக்கம்:**

1. **கோப்பு பதிவேற்றம்**: 10MB வரை படங்கள் அனுமதிக்கிறது
2. **பதிவேற்றம்**: இயக்கத்தைப் போது எந்த தகவல்கள் பதிவேற்றப்படும் என்பதைக் கட்டுப்படுத்துகிறது
3. **Azure AI Foundry**: பயன்பாட்டிற்கு பயன்படுத்த வேண்டிய தொடுப்புக் குறிப்பு மற்றும் மாடல் விவரங்கள் (கீ இல்லாத அங்கீகாரம்)
4. **பாதுகாப்பு**: செயல்பாட்டின்போது அடையாள தகவல்களை வெளிப்பட வேண்டாமென பிழை கையாளல்

## Running the Application

### படி 1: உள்நுழைந்து உங்கள் தொடுப்புக் குறிப்பை அமைக்கவும்

அங்கீகாரம் Microsoft Entra ID மூலம் கீ இல்லாமல் நடக்கும், அதனால் API கீ இல்லை. உள்நுழையவும் உங்கள் Foundry தொடுப்புக் குறிப்பை அமைக்கவும்:

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

**ஏன் இது தேவையானது:**
- Azure AI Foundry Microsoft Entra ID மூலம் inference கோரிக்கைகளை அங்கீகரிக்கிறது
- கீ இல்லாத அங்கீகாரம் என்பது உங்கள் மூலக் குறியீட்டிலும் சுற்றுச்சூழலும் ரகசியங்கள் இல்லாமல் இருப்பதை குறிக்கும்
- உங்கள் கணக்கிற்கு அந்த வளத்தில் **Cognitive Services OpenAI User** பங்கு வழங்கப்பட்டிருக்க வேண்டும்

### படி 2: கட்டமைக்கவும் இயக்கவும்

திட்ட அடைவு செல்லவும்:
```bash
cd 04-PracticalSamples/petstory
```

பயன்பாட்டை கட்டமைக்கவும்:
```bash
mvn clean compile
```

சேவையகத்தை துவங்கவும்:
```bash
mvn spring-boot:run
```

பயன்பாடு `http://localhost:8080` இல் துவங்கும்.

### படி 3: பயன்பாட்டை சோதனை செய்தல்

1. உலாவியில் `http://localhost:8080` ஐ திறக்கவும்
2. உங்கள் விலங்கைக் குறித்த விவரத்தை உள்ளிடவும் (உதா., "பிடித்தவாறான பொன் நிறம் கொண்ட ரெட்ரீவர்")
3. "Generate Story" பொத்தானை அழுத்தி AI உருவாக்கிய கதையை பெறவும்
4. மாற்றாக, விலங்கின் படத்தை பதிவேற்றவும், அது தானாகவும் விளக்கம் உருவாக்கும்
5. உங்கள் விலங்கின் விளக்கத்தின் அடிப்படையில் படைப்பாற்றலான கதை காண்க

## How It All Works Together

நீங்கள் கதை உருவாக்கும் போது முழு ஓட்டம் இதுவாக இருக்கும்:

1. **பயனர் உள்ளீடு**: நீங்கள் வலைப் படிவத்தில் உங்கள் விலங்கைக் குறிக்கிறீர்கள்
2. **படிவ தாக்கல்**: உலாவி POST கோரிக்கையை `/generate-story` தளத்திற்கு அனுப்புகிறது
3. **கட்டுப்படுத்தல் செயல்முறை**: `PetController` உள்ளீட்டை சரிபார்த்து சுத்திகரிக்கிறது
4. **AI சேவை அழைப்பு**: `StoryService` Azure AI Foundry மாடலை அழைக்கிறது
5. **கதை உருவாக்கம்**: AI தரப்பட்ட விவரத்தின் அடிப்படையில் படைப்பாற்றலான கதை எழுதுகிறது
6. **பதில் கையாளல்**: கட்டுப்படுத்தல் கதையை பெற்று மாதிரியில் சேர்க்கிறது
7. **வார்ப்புரு வரைபடல்**: Thymeleaf `result.html` கதை கொண்டு வடிவமைக்கிறது
8. **காட்சி**: பயனர் உலாவியில் உருவாக்கிய கதையைப் பார்க்கிறார்

**பிழை கையாளும் ஓட்டம்:**
AI சேவை தோன்றாவிட்டால்:
1. கட்டுப்படுத்தல் விதிவிலக்கை பிடிக்கிறது
2. முன்பே எழுதப்பட்டFallback கதைகளை உருவாக்குகிறது
3. AI சேவை கிடையாது என்பதை குறிக்கும் குறிப்புடன்Fallback கதை த்தை காண்பிக்கிறது
4. பயனருக்கு எப்போதும் கதை கிடைக்கச் செய்கிறது, நல்ல பயனர் அனுபவம் உறுதி செய்யப்படுகிறது

## Understanding the AI Integration

### Azure AI Foundry (keyless)
பயன்பாடு Microsoft Entra ID கீ இல்லாத அங்கீகாரம் கொண்டு Azure AI Foundry ஐ பயன்படுத்துகிறது:

```java
// சாவி இல்லாத அங்கீகாரம் - API சாவி இல்லை
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Prompt Engineering
சேவை நல்ல முடிவுகளை பெற கவனமாக வடிவமைக்கப்பட்ட தூண்டுகோள்களை பயன்படுத்துகிறது:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Response Processing
AI இன் பதிலை இறக்குமதி செய்து சரிபார்க்கிறது:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Next Steps

மேலும் உதாரணங்களுக்கு, [அத்தியாயம் 04: நடைமுறை எடுத்துக்காட்டுகள்](../README.md) காணவும்

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**மறுப்பு**:
இந்த ஆவணம் AI மொழிபெயர்ப்பு சேவை [Co-op Translator](https://github.com/Azure/co-op-translator) பயன்படுத்தி மொழிபெயர்க்கப்பட்டுள்ளது. நாங்கள் துல்லியத்திற்காக முயற்சி செய்துள்ளோம், ஆனால் தானாக செய்யப்படும் மொழிபெயர்ப்புகளில் பிழைகள் அல்லது தவறுகள் இருக்கலாம் என்பதை கவனத்தில் கொள்ளவும். அசல் ஆவணம் அதன் தாய்மொழியில் அதிகாரப்பூர்வ ஆதாரமாக கருதப்பட வேண்டும். முக்கியமான தகவல்களுக்கு, தொழில்நுட்பமான மனித மொழிபெயர்ப்பு பரிந்துரைக்கப்படுகிறது. இந்த மொழிபெயர்ப்பைப் பயன்படுத்துவதால் ஏற்படும் எந்த தவறான புரிதல்கள் அல்லது தவறான விளக்கத்திற்கும் நாங்கள் பொறுப்பில்வில்லை.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->