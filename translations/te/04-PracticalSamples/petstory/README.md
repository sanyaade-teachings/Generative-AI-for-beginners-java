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

ప్రారంభించడానికి ముందు, మీరు కలిగి ఉండాలి:
- Java 21 లేదా అంతకంతలో ఉన్న సంస్కరణ ఇన్‌స్టాల్ చేయబడింది
- డిపెండెన్సీ నిర్వహణ కోసం Maven
- ఒక Azure AI Foundry మోడల్ డిప్లాయ్‌మెంట్ (దాన్ని `azd up` తో ప్రొవిజన్ చేయండి — [Chapter 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md) చూడండి), `az login` తో సైన్ ఇన్ అయి ఉండు (కీ లెస్ ఆథ్)
- Java, Spring Boot మరియు వెబ్ అభివృద్ధి యొక్క ప్రాథమిక అవగాహన

## Understanding the Project Structure

పెట్ స్టోరి ప్రాజెక్ట్‌లో కొన్ని ముఖ్యమైన ఫైళ్ళున్నాయి:

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

**ఫైల్:** `PetStoryApplication.java`

ఇది మా Spring Boot అప్లికేషన్ యొక్క ప్రవేశద్వారం:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**ఇది చేస్తున్నది:**
- `@SpringBootApplication` అనోటేషన్ ఆటో-కన్ఫిగరేషన్ మరియు కంపోనెంట్ స్కానింగ్‌ని ఎనేబుల్ చేస్తుంది
- 8080 పోర్టులో ఒక ఎంబెడెడ్ వెబ్ సర్వర్ (Tomcat) ప్రారంభిస్తుంది
- అవసరమైన అన్ని Spring బీన్‌లు మరియు సర్వీసులను ఆటోమేటిక్గా సృష్టిస్తుంది

### 2. Web Controller

**ఫైల్:** `PetController.java`

ఇది అన్ని వెబ్ అభ్యర్థనలు మరియు యూజర్ ఇంటరాక్షన్‌లను నిర్వహిస్తుంది:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // index.html టెంప్లేట్‌ని తిరిగి ఇస్తుంది
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // ఇన్‌పుట్ ధృవీకరణ
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // భద్రత కొరకు ఇన్‌పుట్‌ను శుద్ధి చేయండి
        String sanitizedDescription = sanitizeInput(description);
        
        // లోపాలను నిర్వహిస్తూ కథను రూపొందించండి
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // result.html టెంప్లేట్‌ని తిరిగి ఇస్తుంది
            
        } catch (Exception e) {
            // AI విఫలమైతే ప్రత్యామ్నాయ కథను ఉపయోగించండి
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // పొడవును పరిమితం చేయండి
    }
}
```

**ప్రధాన లక్షణాలు:**

1. **రూట్ హ్యాండ్లింగ్**: `@GetMapping("/")` అప్‌లోడ్ ఫారమ్ ప్రదర్శిస్తుంది, `@PostMapping("/generate-story")` సబ్మిషన్స్‌ను ప్రాసెస్ చేస్తుంది
2. **ఇన్‌పుట్ చెల్లింపు**: ఖాళీ వివరణలు మరియు పొడవు పరిమితుల కోసం తనిఖీలు చేస్తుంది
3. **భద్రత**: XSS దాడులను నివారించేందుకు యూజర్ ఇన్‌పుట్‌ను శుభ్రపరుస్తుంది
4. **లోప నిర్వహణ**: AI సర్వీస్ విఫలమైనప్పుడు fallback కథలను అందిస్తుంది
5. **మోడల్ బైండింగ్**: Spring యొక్క `Model` ఉపయోగించి డేటాను HTML టెంప్లేట్లకు పాస్ చేస్తుంది

**Fallback సిస్టమ్:**
కంట్రోలర్ AI సర్వీస్ అందుబాటులో లేకపోయినప్పుడు ఉపయోగించే ముందుగా రాసిన కథా టెంప్లేట్లను కలిగి ఉంటుంది:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // సహనశీల ప్రతిస్పందనలకు వివరణ హాష్ వాడండి
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Story Service

**ఫైల్:** `StoryService.java`

ఈ సర్వీస్ Azure AI Foundryతో కమ్యూనికేట్ చేస్తూ కీ లెస్ ఆథెంటికేషన్ ఉపయోగించి కథలు రూపొందిస్తుంది:

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
        
        // Foundry యొక్క OpenAI-అనుకూల ఎండ్పాయింట్ /openai/v1/ కింద ఉంటుంది
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Microsoft Entra ID తో కీఫ్లీ లాగిన్ (ఏపీ ఐ కీ అవసరం లేదు)
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
        
        // AI అభ్యర్థనను అమర్చండి
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // ప్రతిస్పందన పొడవుని పరిమితం చేయండి
                .temperature(0.8)          // సృజనాత్మకతను నియంత్రించండి (0.0-1.0)
                .build();
        
        // అభ్యర్థన పంపండి మరియు ప్రతిస్పందన పొందండి
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**ప్రధాన భాగాలు:**

1. **OpenAI క్లయింట్**: అధికారిక OpenAI Java SDK ఉపయోగించి Azure AI Foundry (కీ లెస్) కొరకు కాన్ఫిగర్ చేయబడింది
2. **సిస్టమ్ ప్రాంప్ట్**: AIకి కుటుంబ స్నేహపూర్వకమైన పెట్ కథలు వ్రాయమని ప్రవర్తనను సెట్ చేస్తుంది
3. **యూజర్ ప్రాంప్ట్**: వివరణ ఆధారంగా AIకి ఎలాంటి కథ వ్రాయమని చెప్పారు
4. **పరామితులు**: కథ పొడవు మరియు సృజనాత్మకత స్థాయిని నియంత్రిస్తుంది
5. **లోప నిర్వహణ**: ఎక్స్‌సెప్షన్లు విరిగి కంట్రోలర్ వాటిని పట్టుకుని నిర్వహిస్తుంది

### 4. Web Templates

**ఫైల్:** `index.html` (అప్‌లోడ్ ఫార్మ్)

ఇది ప్రధాన పేజీ, యూజర్లు అక్కడ తమ పెట్లను వివరించగలరు:

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

**ఫైల్:** `result.html` (కథ ప్రదర్శన)

పుట్టిన కథను ప్రదర్శిస్తుంది:

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

**టెంప్లేట్ లక్షణాలు:**

1. **Thymeleaf ఇంటిగ్రేషన్**: డైనమిక్ కంటెంట్ కోసం `th:` అట్రిబ్యూట్లను ఉపయోగిస్తుంది
2. **రిస్పాన్సివ్ డిజైన్**: మొబైల్ మరియు డెస్క్‌టాప్ కొరకు CSS శైలీకరణ
3. **లోప నిర్వహణ**: వాలిడేషన్ లోపాలను యూజర్లకు చూపిస్తుంది
4. **క్లయింట్-సైడ్ ప్రాసెసింగ్**: చిత్రం విశ్లేషణకు జావాస్క్రిప్ట్ (Transformers.js ఉపయోగించి)

### 5. Configuration

**ఫైల్:** `application.properties`

అప్లికేషన్ కోసం కాన్ఫిగరేషన్ సెట్టింగ్స్:

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

**కాన్ఫిగరేషన్ వివరణ:**

1. **ఫైల్ అప్‌లోడ్**: 10MB వరకు చిత్రాలు అప్‌లోడ్ చేసుకునేందుకు అనుమతిస్తుంది
2. **లాగింగ్**: అమలు సమయంలో ఏ సమాచారం లాగ్ చేయబడిందో నియంత్రిస్తుంది
3. **Azure AI Foundry**: ఉపయోగించాల్సిన ఎండ్‌పాయింట్ మరియు మోడల్ డిప్లాయ్‌మెంట్‌ను (కీ లెస్ ఆథ్) సూచిస్తుంది
4. **భద్రత**: సున్నితమైన సమాచారాన్ని బయటపెట్టకుండా లోప నిర్వహణని నియంత్రిస్తుంది

## Running the Application

### Step 1: Sign In and Set Your Endpoint

ఆథెంటికేషన్ కీ లెస్ (Microsoft Entra ID)గా ఉంటుంది, అందుకే API కీ అవసరం లేదు. సైన్ ఇన్ అయి మీ Foundry ఎండ్‌పాయింట్ సెట్ చేయండి:

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

**దీని అవసరం ఎందుకు:**
- Azure AI Foundry మైక్రోసాఫ్ట్ ఎత్రా ID ఉపయోగించి ఇన్ఫరెన్స్ అభ్యర్థనలు ఆథెంటికేట్ చేస్తుంది
- కీ లెస్ ఆథ్ అంటే మీ సోర్స్ కోడ్ లేదా ఎన్విరాన్‌మెంట్‌లో ఏ రహస్యాలు ఉండవు
- మీ అకౌంటుకు రీసోర్స్‌పై **Cognitive Services OpenAI User** పాత్ర అవసరం

### Step 2: Build and Run

ప్రాజెక్ట్ డైరెక్టరీకి వెళ్ళండి:
```bash
cd 04-PracticalSamples/petstory
```

అప్లికేషన్ బిల్డ్ చేయండి:
```bash
mvn clean compile
```

సర్వర్ స్టార్ట్ చేసుకోండి:
```bash
mvn spring-boot:run
```

అప్లికేషన్ `http://localhost:8080` పై ప్రారంభమవుతుంది.

### Step 3: Test the Application

1. బ్రౌజర్లో `http://localhost:8080` ను ఓపెన్ చేయండి
2. మీరు పెట్ వివరించే టెక్స్ట్ ఏరియా (ఉదా: "ప్రియమైన గోల్డెన్ రెట్రీవర్, ఫెట్చ్ చేయడం ఇష్టపడే") లో రాయండి
3. "Generate Story" క్లిక్ చేసి AI ద్వారా రూపొందించిన కథ పొందండి
4. లేదా, పెట్ చిత్రం అప్‌లోడ్ చేసి వివరణ ఆటోమేటిగ తయారు చేయండి
5. మీ పెట్ వివరణ ఆధారంగా సృజనాత్మకమైన కథను చూడండి

## How It All Works Together

పెట్స్ కథను రూపొందించే క్రమం ఈ విధంగా ఉంటుంది:

1. **యూజర్ ఇన్‌పుట్**: మీరు వెబ్ ఫార్మ్‌లో మీ పెట్‌ను వివరించండి
2. **ఫార్మ్ సబ్మిషన్**: బ్రౌజర్ `/generate-story` కు POST అభ్యర్థన పంపుతుంది
3. **కంట్రోలర్ ప్రాసెసింగ్**: `PetController` ఇన్‌పుట్‌ను చెల్లింపుతో కూడినది మరియు శుభ్రపరచబడినది గా పరిశీలిస్తుంది
4. **AI సర్వీస్ కాల్**: `StoryService` Azure AI Foundry మోడల్‌కు అభ్యర్థనకు పంపుతుంది
5. **కథ సృష్టి**: వివరణ ఆధారంగా AI సృజనాత్మకమైన కథను రూపొందిస్తుంది
6. **ప్రతిస్పందన నిర్వహణ**: కంట్రోలర్ కథను స్వీకరించి మోడల్‌కు జత చేస్తుంది
7. **టెంప్లేట్ రేండరింగ్**: Thymeleaf `result.html` ను కథతో రేండర్ చేస్తుంది
8. **ప్రదర్శన**: యూజర్ బ్రౌజర్లో తయారైన కథను చూస్తారు

**లోప నిర్వహణ ఫ్లో:**
AI సర్వీస్ విఫలమైనప్పుడు:
1. కంట్రోలర్ ఎక్స్‌సెప్షన్‌ను పట్టుకోగలదు
2. ముందుగా పెట్టిన fallback కథలు రూపొందిస్తుంది
3. AI అందుబాటులో లేదు అన్న నోట్లతో fallback కథను చూపిస్తుంది
4. యూజర్‌కు కథ మాత్రం అందుతుంది, మంచి అనుభవం కోసం

## Understanding the AI Integration

### Azure AI Foundry (keyless)
అప్లికేషన్ Azure AI Foundryని కీ లెస్ ఆథెంటికేషన్ (Microsoft Entra ID)తో ఉపయోగిస్తింది:

```java
// కీ లేకుండా ప్రమాణీకరణ - ఏపీ ఐ కీ అవసరం లేదు
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Prompt Engineering
భద్రతగా మరియు మంచి ఫలితాల కోసం సజాగ్రతతో prompts తయారు చేస్తారు:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Response Processing
AI స్పందనను తేచి, దాన్ని పరిశీలించి ప్రామాణికత నిర్ధారిస్తుంది:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Next Steps

అనేక ఇతర ఉదాహరణల కోసం, చూడండి [Chapter 04: Practical samples](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**అస్వీకరణ**:
ఈ పత్రం AI అనువాద సేవ [Co-op Translator](https://github.com/Azure/co-op-translator) ఉపయోగించి అనువదించబడింది. మేము ఖచ్చితత్వానికి ప్రయత్నిస్తున్నప్పటికీ, ఆటోమేటెడ్ అనువాదాలు తప్పులు లేదా అసమగ్రతలను కలిగి ఉండవచ్చు. దాని స్వదేశ భాషలో ఉన్న అసలు పత్రాన్ని అధికారం కలిగిన మూలంగా పరిగణించాలి. కీలకమైన సమాచారం కోసం, ప్రొఫెషనల్ మానవ అనువాదాన్ని సిఫారసు చేస్తాము. ఈ అనువాదం ఉపయోగం వల్ల కలిగే ఏవైనా అపార్థాలు లేదా తప్పుదారులు కోసం మేము బాధ్యత వహించము.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->