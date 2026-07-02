# ਪੇਟ ਸਟੋਰੀ ਜੈਨਰੇਟਰ ਟਿਊਟੋਰੀਅਲ ਸ਼ੁਰੂਆਤੀ ਲਈ

## ਸੂਚੀ

- [ਪੂਰਵ ਅਵਸ਼ਕਤਾ](#ਪੂਰਵ-ਅਵਸ਼ਕਤਾ)
- [ਪ੍ਰੋਜੈਕਟ ਸਟ੍ਰੱਕਚਰ ਨੂੰ ਸਮਝਣਾ](#ਪ੍ਰੋਜੈਕਟ-ਸਟ੍ਰੱਕਚਰ-ਨੂੰ-ਸਮਝਣਾ)
- [ਮੁੱਖ ਘਟਕਾਂ ਦੀ ਵਿਆਖਿਆ](#ਮੁੱਖ-ਘਟਕਾਂ-ਦੀ-ਵਿਆਖਿਆ)
  - [1. ਮੁੱਖ ਐਪਲੀਕੇਸ਼ਨ](#1-ਮੁੱਖ-ਐਪਲੀਕੇਸ਼ਨ)
  - [2. ਵੈੱਬ ਕੰਟਰੋਲਰ](#2-ਵੈੱਬ-ਕੰਟਰੋਲਰ)
  - [3. ਸਟੋਰੀ ਸਰਵਿਸ](#3-ਸਟੋਰੀ-ਸਰਵਿਸ)
  - [4. ਵੈੱਬ ਟੈਂਪਲੇਟਸ](#4-ਵੈੱਬ-ਟੈਂਪਲੇਟਸ)
  - [5. ਕਾਂਫਿਗਰੇਸ਼ਨ](#5-ਕਾਂਫਿਗਰੇਸ਼ਨ)
- [ਐਪਲੀਕੇਸ਼ਨ ਚਲਾਉਣਾ](#ਐਪਲੀਕੇਸ਼ਨ-ਚਲਾਉਣਾ)
- [ਇਹ ਸਾਰਾ ਕੰਮ ਕਿਵੇਂ ਕਰਦਾ ਹੈ](#ਇਹ-ਸਾਰਾ-ਕੰਮ-ਕਿਵੇਂ-ਕਰਦਾ-ਹੈ)
- [AI ਇੰਟਿਗ੍ਰੇਸ਼ਨ ਨੂੰ ਸਮਝਣਾ](#ai-ਇੰਟਿਗ੍ਰੇਸ਼ਨ-ਨੂੰ-ਸਮਝਣਾ)
- [ਅਗਲੇ ਕਦਮ](#ਅਗਲੇ-ਕਦਮ)

## ਪੂਰਵ ਅਵਸ਼ਕਤਾ

ਸ਼ੁਰੂ ਕਰਨ ਤੋਂ ਪਹਿਲਾਂ, ਇਹ ਯਕੀਨੀ ਬਣਾਓ ਕਿ ਤੁਹਾਡੇ ਕੋਲ ਹੈ:
- ਜਾਵਾ 21 ਜਾਂ ਉੱਪਰ ਇੰਸਟਾਲਡ
- ਡਿਪੈਂਡੇਨਸੀ ਪ੍ਰਬੰਧਨ ਲਈ ਮਾਵੇਨ
- ਏਜ਼ੂਰ AI ਫਾਊਂਡਰੀ ਮਾਡਲ ਡਿਪਲੋਇਮੈਂਟ (ਉਦਾਹਰਨ ਲਈ `azd up` ਨਾਲ ਪ੍ਰੋਵਿਜ਼ਨ ਕਰੋ — ਦੇਖੋ [ਚੈਪਟਰ 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), `az login` (ਕੀਲੈੱਸ ਅਥੈਂਟੀਕੇਸ਼ਨ) ਨਾਲ ਸਾਇਨ ਇਨ
- ਜਾਵਾ, ਸਪ੍ਰਿੰਗ ਬੂਟ ਅਤੇ ਵੈੱਬ ਡਿਵੈਲਪਮੈਂਟ ਦੀ ਬੁਨਿਆਦੀ ਸਮਝ

## ਪ੍ਰੋਜੈਕਟ ਸਟ੍ਰੱਕਚਰ ਨੂੰ ਸਮਝਣਾ

ਪੇਟ ਸਟੋਰੀ ਪ੍ਰੋਜੈਕਟ ਵਿੱਚ ਕੁਝ ਮਹੱਤਵਪੂਰਣ ਫਾਇਲਾਂ ਹਨ:

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

## ਮੁੱਖ ਘਟਕਾਂ ਦੀ ਵਿਆਖਿਆ

### 1. ਮੁੱਖ ਐਪਲੀਕੇਸ਼ਨ

**ਫਾਇਲ:** `PetStoryApplication.java`

ਇਹ ਸਾਡੇ ਸਪ੍ਰਿੰਗ ਬੂਟ ਐਪਲੀਕੇਸ਼ਨ ਲਈ ਐਂਟ੍ਰੀ ਪੁਆਇੰਟ ਹੈ:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**ਇਹ ਕੀ ਕਰਦਾ ਹੈ:**
- `@SpringBootApplication` ਐਨੋਟੇਸ਼ਨ ਆਟੋ-ਕਾਂਫਿਗਰੇਸ਼ਨ ਅਤੇ ਕੰਪੋਨੈਂਟ ਸਕੈਨਿੰਗ ਨੂੰ ਯੋਗ ਕਰਦਾ ਹੈ
- ਪੋਰਟ 8080 'ਤੇ ਐੰਬੈਡਡ ਵੈੱਬ ਸਰਵਰ (ਟੋਮਕੈਟ) ਚਲਾਉਂਦਾ ਹੈ
- ਸਾਰੀਆਂ ਲੋੜੀਂਦੀਆਂ ਸਪ੍ਰਿੰਗ ਬੀਨਜ਼ ਅਤੇ ਸਰਵਿਸਜ਼ ਆਪਣੇ ਆਪ ਬਣਾਂਦਾ ਹੈ

### 2. ਵੈੱਬ ਕੰਟਰੋਲਰ

**ਫਾਇਲ:** `PetController.java`

ਇਹ ਸਾਰੇ ਵੈੱਬ ਰਿਕਵੈਸਟਸ ਅਤੇ ਯੂਜ਼ਰ ਇੰਟਰੈਕਸ਼ਨ ਸੰਭਾਲਦਾ ਹੈ:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // index.html ਟੈਮਪਲੇਟ ਵਾਪਸ ਕਰਦਾ ਹੈ
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // ਇਨਪੁੱਟ ਵੈਰੀਫਿਕੇਸ਼ਨ
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // ਸੁਰੱਖਿਆ ਲਈ ਇਨਪੁੱਟ ਸਾਫ਼ ਕਰੋ
        String sanitizedDescription = sanitizeInput(description);
        
        // ਗਲਤੀ ਸੰਭਾਲ ਨਾਲ ਕਹਾਣੀ ਬਣਾਓ
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // result.html ਟੈਮਪਲੇਟ ਵਾਪਸ ਕਰਦਾ ਹੈ
            
        } catch (Exception e) {
            // ਜੇ AI ਅਸਫਲ ਰਹਿੰਦਾ ਹੈ ਤਾਂ ਬੈਕਅੱਪ ਕਹਾਣੀ ਵਰਤੋ
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // ਲੰਬਾਈ ਸੀਮਿਤ ਕਰੋ
    }
}
```

**ਮੁੱਖ ਵਿਸ਼ੇਸ਼ਤਾਵਾਂ:**

1. **ਰੂਟ ਹੈਂਡਲਿੰਗ**: `@GetMapping("/")` ਅਪਲੋਡ ਫਾਰਮ ਦਿਖਾਉਂਦਾ ਹੈ, `@PostMapping("/generate-story")` ਸਬਮੀਸ਼ਨਾਂ ਨੂੰ ਪ੍ਰੋਸੈਸ ਕਰਦਾ ਹੈ
2. **ਇਨਪੁੱਟ ਵੈਰੀਫਿਕੇਸ਼ਨ**: ਖਾਲੀ ਡਿਸਕ੍ਰਿਪਸ਼ਨ ਅਤੇ ਲੰਬਾਈ ਸੀਮਾਵਾਂ ਦੀ ਜਾਂਚ ਕਰਦਾ ਹੈ
3. **ਸੁਰੱਖਿਆ**: XSS ਹਮਲਿਆਂ ਨੂੰ ਰੋਕਣ ਲਈ ਯੂਜ਼ਰ ਇੰਪੁੱਟ ਨੂੰ ਸੈਨਿਟਾਈਜ਼ ਕਰਦਾ ਹੈ
4. **ਏਰਰ ਹੈਂਡਲਿੰਗ**: ਜਦੋਂ AI ਸਰਵਿਸ ਨਾਕਾਮ ਰਹਿੰਦੀ ਹੈ ਤਾਂ ਫਾਲਬੈਕ ਕਹਾਣੀਆਂ ਪ੍ਰਦਾਨ ਕਰਦਾ ਹੈ
5. **ਮਾਡਲ ਬਾਇੰਡਿੰਗ**: ਸਪ੍ਰਿੰਗ ਦੇ `Model` ਦੀ ਵਰਤੋਂ ਕਰਕੇ ਡੇਟਾ HTML ਟੈਂਪਲੇਟਸ ਨੂੰ ਭੇਜਦਾ ਹੈ

**ਫਾਲਬੈਕ ਸਿਸਟਮ:**
ਕੰਟਰੋਲਰ ਵਿੱਚ ਪਹਿਲਾਂ ਲਿਖੀਆਂ ਕਹਾਣੀਆਂ ਦੇ ਟੈਂਪਲੇਟ ਸ਼ਾਮਲ ਹੁੰਦੇ ਹਨ ਜੋ AI ਸੇਵਾ ਦੇ ਅਣਉਪਲਬਧ ਹੋਣ 'ਤੇ ਵਰਤੇ ਜਾਂਦੇ ਹਨ:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // ਲਗਾਤਾਰ ਜਵਾਬਾਂ ਲਈ ਵਰਣਨ ਹੈਸ਼ ਦੀ ਵਰਤੋਂ ਕਰੋ
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. ਸਟੋਰੀ ਸਰਵਿਸ

**ਫਾਇਲ:** `StoryService.java`

ਇਹ ਸਰਵਿਸ Azure AI Foundry ਨਾਲ ਸੰਪਰਕ ਕਰਦਾ ਹੈ ਅਤੇ ਕੀ-ਲੈੱਸ ਅਥੈਂਟੀਕੇਸ਼ਨ ਦੀ ਵਰਤੋਂ ਕਰਕੇ ਕਹਾਣੀਆਂ ਬਣਾਉਂਦਾ ਹੈ:

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
        
        // Foundry ਦਾ OpenAI-ਅਨੁਰੂਪ ਐਂਡਪੌਇੰਟ /openai/v1/ ਦੇ ਹੇਠਾਂ ਹੈ
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Microsoft Entra ID ਨਾਲ ਬਿਨਾਂ ਕੁੰਜੀ ਦੇ ਪ੍ਰਮਾਣੀਕਰਨ (ਕੋਈ API ਕੁੰਜੀ ਨਹੀਂ)
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
        
        // AI ਬੇਨਤੀ ਨੂੰ ਸੰਰਚਿਤ ਕਰੋ
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // ਜਵਾਬ ਦੀ ਲੰਬਾਈ ਸੀਮਤ ਕਰੋ
                .temperature(0.8)          // ਰਚਨਾਤਮਕਤਾ ਨੂੰ ਨਿਯੰਤਰਿਤ ਕਰੋ (0.0-1.0)
                .build();
        
        // ਬੇਨਤੀ ਭੇਜੋ ਅਤੇ ਜਵਾਬ ਪ੍ਰਾਪਤ ਕਰੋ
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**ਮੁੱਖ ਘਟਕ:**

1. **OpenAI ਕਲਾਇੰਟ**: ਅਧਿਕਾਰਿਤ OpenAI ਜਾਵਾ SDK ਜੋ Azure AI Foundry (ਕੀਲੈੱਸ) ਲਈ ਕੰਫਿਗਰ ਕੀਤਾ ਗਿਆ ਹੈ
2. **ਸਿਸਟਮ ਪ੍ਰਾਂਪਟ**: AI ਦੇ ਵਿਵਹਾਰ ਨੂੰ ਪਰਿਵਾਰ-ਮੈਤਰ ਸਟੋਰੀਜ਼ ਲਿਖਣ ਲਈ ਸੈੱਟ ਕਰਦਾ ਹੈ
3. **ਯੂਜ਼ਰ ਪ੍ਰਾਂਪਟ**: AI ਨੂੰ ਉਹ ਕਹਾਣੀ ਲਿਖਣ ਲਈ ਤਕਲੀਫ਼ ਦਿੰਦਾ ਹੈ ਜੋ ਵਰਣਨ 'ਤੇ ਅਧਾਰਿਤ ਹੈ
4. **ਪੈਰਾਮੀਟਰਸ**: ਕਹਾਣੀ ਦੀ ਲੰਬਾਈ ਅਤੇ ਕ੍ਰੀਏਟਿਵਿਟੀ ਲੈਵਲ ਨੂੰ ਨਿਯੰਤਰਿਤ ਕਰਦਾ ਹੈ
5. **ਏਰਰ ਹੈਂਡਲਿੰਗ**: ਐਸਪਸ਼ਨਾਂ ਨੂੰ ਥ੍ਰੋ ਕਰਦਾ ਹੈ ਜਿਨ੍ਹਾਂ ਨੂੰ ਕੰਟਰੋਲਰ ਫੜਦਾ ਅਤੇ ਸੰਭਾਲਦਾ ਹੈ

### 4. ਵੈੱਬ ਟੈਂਪਲੇਟਸ

**ਫਾਇਲ:** `index.html` (ਅਪਲੋਡ ਫਾਰਮ)

ਮੁੱਖ ਸਫ਼ਾ ਇੱਥੇ ਯੂਜ਼ਰ ਆਪਣੇ ਪੈਟ ਦਾ ਵੇਰਵਾ ਦਿੰਦੇ ਹਨ:

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

**ਫਾਇਲ:** `result.html` (ਸਟੋਰੀ ਦਿਖਾਉਣਾ)

ਉਤਪੰਨ ਕਹਾਣੀ ਦਿਖਾਉਂਦਾ ਹੈ:

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

**ਟੈਂਪਲੇਟ ਵਿਸ਼ੇਸ਼ਤਾਵਾਂ:**

1. **ਥਾਈਮਲੀਫ਼ ਇੰਟਿਗ੍ਰੇਸ਼ਨ**: ਡਾਇਨਾਮਿਕ ਸਮੱਗਰੀ ਲਈ `th:` ਐਟਰਿਬਿਊਟ ਦੀ ਵਰਤੋਂ
2. **ਰੈਸਪਾਂਸਿਵ ਡਿਜ਼ਾਈਨ**: ਮੋਬਾਈਲ ਅਤੇ ਡੈਸਕਟਾਪ ਲਈ CSS ਸਟਾਈਲਿੰਗ
3. **ਏਰਰ ਹੈਂਡਲਿੰਗ**: ਯੂਜ਼ਰਾਂ ਨੂੰ ਵੈਰੀਫਿਕੇਸ਼ਨ ਦੇ ਤ੍ਰੁੱਟੀਆਂ ਦਿਖਾਉਂਦਾ ਹੈ
4. **ਕਲਾਇੰਟ-ਸਾਈਡ ਪ੍ਰੋਸੈਸਿੰਗ**: ਜਾਵਾਸਕ੍ਰਿਪਟ ਇਮੇਜ ਵਿਸ਼ਲੇਸ਼ਣ ਲਈ (Transformers.js ਦੀ ਵਰਤੋਂ ਨਾਲ)

### 5. ਕਾਂਫਿਗਰੇਸ਼ਨ

**ਫਾਇਲ:** `application.properties`

ਐਪਲੀਕੇਸ਼ਨ ਲਈ ਕਾਂਫਿਗਰੇਸ਼ਨ ਸੈਟਿੰਗਜ਼:

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

**ਕਾਂਫਿਗਰੇਸ਼ਨ ਦੀ ਵਿਆਖਿਆ:**

1. **ਫਾਇਲ ਅਪਲੋਡ**: 10MB ਤੱਕ ਦੀਆਂ ਇਮੇਜਾਂ ਦੀ ਆਗਿਆ
2. **ਲਾਗਿੰਗ**: ਚਲਾਉਣ ਦੌਰਾਨ ਕਿੜ੍ਹਾ ਜਾਣਕਾਰੀ ਲਾਗ ਕੀਤੀ ਜਾਵੇ
3. **Azure AI Foundry**: ਵਰਤਣ ਲਈ ਐਂਡਪੌਇੰਟ ਅਤੇ ਮਾਡਲ ਡਿਪਲੋਇਮੈਂਟ ਦਰਸਾਉਂਦਾ ਹੈ (ਕੀਲੈੱਸ ਅਥੈਂਟੀਕੇਸ਼ਨ)
4. **ਸੁਰੱਖਿਆ**: ਸੈਂਸੀਟਿਵ ਜਾਣਕਾਰੀ ਪ੍ਰਗਟ ਨਾ ਹੋਣ ਨਾਲ ਸਮੰਬੰਧਤ ਏਰਰ ਹੈਂਡਲਿੰਗ

## ਐਪਲੀਕੇਸ਼ਨ ਚਲਾਉਣਾ

### ਕਦਮ 1: ਸਾਇਨ ਇਨ ਕਰੋ ਅਤੇ ਆਪਣਾ ਐਂਡਪੌਇੰਟ ਸੈੱਟ ਕਰੋ

ਅਥੈਂਟੀਕੇਸ਼ਨ ਕੀਲੈੱਸ (Microsoft Entra ID) ਹੈ, ਇਸ ਲਈ ਕੋਈ API ਕੀ ਨਹੀਂ। ਸਾਇਨ ਇਨ ਕਰੋ ਅਤੇ ਆਪਣਾ Foundry ਐਂਡਪੌਇੰਟ ਸੈੱਟ ਕਰੋ:

**ਵਿੰਡੋਜ਼ (ਕਮਾਂਡ ਪ੍ਰੰਪਟ):**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**ਵਿੰਡੋਜ਼ (ਪਾਵਰਸ਼ੈੱਲ):**
```powershell
az login
$env:AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
```

**ਲਿਨਕਸ/ਮੈਕਓਐਸ:**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**ਇਸ ਦੀ ਲੋੜ ਕਿਉਂ:**
- Azure AI Foundry Microsoft Entra ID ਨਾਲ ਇਨਫਰੰਸ ਰਿਕਵੈਸਟਾਂ ਦੀ ਪ੍ਰਮਾਣਿਕਤਾ ਕਰਦਾ ਹੈ
- ਕੀਲੈੱਸ ਅਥੈਂਟੀਕੇਸ਼ਨ ਦਾ ਅਰਥ ਹੈ ਕਿ ਤੁਹਾਡੇ ਸੋਰਸ ਕੋਡ ਜਾਂ ਵਾਤਾਵਰਨ ਵਿੱਚ ਕੋਈ ਗੁਪਤਚਰ ਨਹੀਂ ਹੁੰਦੇ
- ਤੁਹਾਡੇ ਖਾਤੇ ਨੂੰ ਸਰੋਤ 'ਤੇ **Cognitive Services OpenAI User** ਰੋਲ ਦੀ ਲੋੜ ਹੁੰਦੀ ਹੈ

### ਕਦਮ 2: ਬਿਲਡ ਕਰੋ ਅਤੇ ਚਲਾਓ

ਪ੍ਰੋਜੈਕਟ ਡਾਇਰੈਕਟਰੀ ਵਿੱਚ ਜਾਓ:
```bash
cd 04-PracticalSamples/petstory
```

ਐਪਲੀਕੇਸ਼ਨ ਬਣਾਓ:
```bash
mvn clean compile
```

ਸਰਵਰ ਸ਼ੁਰੂ ਕਰੋ:
```bash
mvn spring-boot:run
```

ਐਪਲੀਕੇਸ਼ਨ `http://localhost:8080` 'ਤੇ ਚਲ਼ੇਗਾ।

### ਕਦਮ 3: ਐਪਲੀਕੇਸ਼ਨ ਟੈਸਟ ਕਰੋ

1. ਆਪਣੇ ਬ੍ਰਾਊਜ਼ਰ ਵਿੱਚ `http://localhost:8080` ਖੋਲ੍ਹੋ
2. ਟੈਕਸਟ ਏਰੀਆ ਵਿੱਚ ਆਪਣੇ ਪੈਟ ਦਾ ਵੇਰਵਾ ਦਿਓ (ਜਿਵੇਂ, "ਇੱਕ ਖੇਡਾਂ ਵਾਲਾ ਗੋਲਡਨ ਰੀਟਰੀਵਰ ਜੋ ਲੈ ਜਾਨਾ ਪਸੰਦ ਕਰਦਾ ਹੈ")
3. "Generate Story" 'ਤੇ ਕਲਿਕ ਕਰੋ ਤਾਂ ਜੋ AI ਵੱਲੋਂ ਕਹਾਣੀ ਬਣਾਈ ਜਾਵੇ
4. ਅਲਟਰਨੇਟਿਵ ਤੌਰ 'ਤੇ, ਆਪਣੇ ਪੈਟ ਦੀ ਇਮੇਜ ਅਪਲੋਡ ਕਰੋ ਤਾਂ ਜੋ ਆਪਣੇ ਆਪ ਡਿਸਕ੍ਰਿਪਸ਼ਨ ਬਣ ਜਾਵੇ
5. ਆਪਣੇ ਪੈਟ ਦੇ ਵੇਰਵੇ ਬਿਨਾ ਕ੍ਰੀਏਟਿਵ ਕਹਾਣੀ ਵੇਖੋ

## ਇਹ ਸਾਰਾ ਕੰਮ ਕਿਵੇਂ ਕਰਦਾ ਹੈ

ਇਹ ਰਹੀ ਪੂਰੀ ਪ੍ਰਕਿਰਿਆ ਜਦੋਂ ਤੁਸੀਂ ਪੈਟ ਦੀ ਕਹਾਣੀ ਬਣਾਉਂਦੇ ਹੋ:

1. **ਯੂਜ਼ਰ ਇਨਪੁੱਟ**: ਤੁਸੀਂ ਵੈੱਬ ਫਾਰਮ ਤੇ ਆਪਣੇ ਪੈਟ ਦਾ ਵੇਰਵਾ ਦਿੰਦੇ ਹੋ
2. **ਫਾਰਮ ਸਬਮੀਸ਼ਨ**: ਬ੍ਰਾਊਜ਼ਰ `/generate-story` ਨੂੰ POST ਬੇਨਤੀ ਭੇਜਦਾ ਹੈ
3. **ਕੰਟਰੋਲਰ ਪ੍ਰੋਸੈਸਿੰਗ**: `PetController` ਇਨਪੁੱਟ ਨੂੰ ਵੈਲੀਡੇਟ ਅਤੇ ਸੈਨਿਟਾਈਜ਼ ਕਰਦਾ ਹੈ
4. **AI ਸਰਵਿਸ ਕਾਲ**: `StoryService` Azure AI Foundry ਮਾਡਲ ਨੂੰ ਬੇਨਤੀ ਭੇਜਦਾ ਹੈ
5. **ਸਟੋਰੀ ਜੈਨਰੇਸ਼ਨ**: AI ਵੇਰਵੇ ਦੇ ਅਧਾਰ 'ਤੇ ਇੱਕ ਕ੍ਰੀਏਟਿਵ ਕਹਾਣੀ ਬਣਾਉਂਦਾ ਹੈ
6. **ਰਿਸਪਾਂਸ ਹੈਂਡਲਿੰਗ**: ਕੰਟਰੋਲਰ ਕਹਾਣੀ ਪ੍ਰਾਪਤ ਕਰਕੇ ਇਸ ਨੂੰ ਮਾਡਲ ਵਿੱਚ ਸ਼ਾਮਲ ਕਰਦਾ ਹੈ
7. **ਟੈਂਪਲੇਟ ਰੈਂਡਰਿੰਗ**: ਥਾਈਮਲੀਫ਼ `result.html` ਨੂੰ ਕਹਾਣੀ ਨਾਲ ਰੇਂਡਰ ਕਰਦਾ ਹੈ
8. **ਦਿਖਾਈ ਦੇਣਾ**: ਯੂਜ਼ਰ ਆਪਣੇ ਬ੍ਰਾਊਜ਼ਰ ਵਿੱਚ ਬਣਾਈ ਗਈ ਕਹਾਣੀ ਵੇਖਦਾ ਹੈ

**ਏਰਰ ਹੈਂਡਲਿੰਗ ਫਲੋ:**
ਜੇ AI ਸਰਵਿਸ ਫੇਲ ਹੁੰਦੀ ਹੈ:
1. ਕੰਟਰੋਲਰ ਐਕਸਪਸ਼ਨ ਫੜਦਾ ਹੈ
2. ਪਹਿਲਾਂ ਲਿਖੇ ਟੈਂਪਲੇਟਾਂ ਨਾਲ ਫਾਲਬੈਕ ਕਹਾਣੀ ਬਣਾਉਂਦਾ ਹੈ
3. AI ਅਣਉਪਲਬਧਤਾ ਬਾਰੇ ਨੋਟ ਦੇ ਨਾਲ ਫਾਲਬੈਕ ਕਹਾਣੀ ਦਿਖਾਉਂਦਾ ਹੈ
4. ਯੂਜ਼ਰ ਨੂੰ ਫਿਰ ਵੀ ਕਹਾਣੀ ਮਿਲਦੀ ਹੈ, ਜਿਸ ਨਾਲ ਉਨ੍ਹਾਂ ਦਾ ਬਿਹਤਰ ਅਨੁਭਵ ਯਕੀਨੀ ਬਣਦਾ ਹੈ

## AI ਇੰਟਿਗ੍ਰੇਸ਼ਨ ਨੂੰ ਸਮਝਣਾ

### Azure AI Foundry (ਕੀਲੈੱਸ)
ਐਪਲੀਕੇਸ਼ਨ Microsoft Entra ID ਦੇ ਨਾਲ ਕੀਲੈੱਸ ਅਥੈਂਟੀਕੇਸ਼ਨ ਵਰਤਦਾ Azure AI Foundry ਵਰਤਦਾ ਹੈ:

```java
// ਕੀਲ ਰਹਿਤ ਪ੍ਰਮਾਣਿਕਤਾ - ਕੋਈ API ਕੀ ਨਹੀਂ
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### ਪ੍ਰਾਂਪਟ ਇੰਜੀਨੀਅਰਿੰਗ
ਸਰਵਿਸ ਚੰਗੇ ਨਤੀਜੇ ਲਈ ਧਿਆਨ ਸਹਿਤ ਪ੍ਰਾਂਪਟ ਵਰਤਦੀ ਹੈ:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### ਰਿਸਪਾਂਸ ਪ੍ਰੋਸੈਸਿੰਗ
AI ਦਾ ਰਿਸਪਾਂਸ ਕੱਢਿਆ ਅਤੇ ਵੈਰੀਫਾਈ ਕੀਤਾ ਜਾਂਦਾ ਹੈ:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## ਅਗਲੇ ਕਦਮ

ਹੋਰ ਉਦਾਹਰਨਾਂ ਲਈ, ਦੇਖੋ [ਚੈਪਟਰ 04: ਪ੍ਰੈਕਟਿਕਲ ਨਮੂਨੇ](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ਅਸਵੀਕਾਰੋਪਣ**:
ਇਸ ਦਸਤਾਵੇਜ਼ ਦਾ ਅਨੁਵਾਦ ਏਆਈ ਅਨੁਵਾਦ ਸੇਵਾ [Co-op Translator](https://github.com/Azure/co-op-translator) ਦੀ ਵਰਤੋਂ ਕਰਕੇ ਕੀਤਾ ਗਿਆ ਹੈ। ਜਦੋਂ ਕਿ ਅਸੀਂ ਸਹੀਤਾਵਾਂ ਲਈ ਯਤਨਸ਼ੀਲ ਹਾਂ, ਕਿਰਪਾ ਕਰਕੇ ਧਿਆਨ ਰੱਖੋ ਕਿ ਸਵੈਚਾਲਿਤ ਅਨੁਵਾਦਾਂ ਵਿੱਚ ਗਲਤੀਆਂ ਜਾਂ ਅਸਮੱਤਿਆਵਾਂ ਹੋ ਸਕਦੀਆਂ ਹਨ। ਮੂਲ ਦਸਤਾਵੇਜ਼ ਆਪਣੀ ਮੂਲ ਭਾਸ਼ਾ ਵਿੱਚ ਅਧਿਕਾਰਕ ਸਰੋਤ ਮੰਨਿਆ ਜਾਣਾ ਚਾਹੀਦਾ ਹੈ। ਜਰੂਰੀ ਜਾਣਕਾਰੀ ਲਈ, ਪੇਸ਼ੇਵਰ ਮਨੁੱਖੀ ਅਨੁਵਾਦ ਦੀ ਸਿਫ਼ਾਰਸ਼ ਕੀਤੀ ਜਾਂਦੀ ਹੈ। ਅਸੀਂ ਇਸ ਅਨੁਵਾਦ ਦੇ ਉਪਯੋਗ ਤੋਂ ਪੈਦਾ ਹੋਣ ਵਾਲੀਆਂ ਕਿਸੇ ਵੀ ਗਲਤਫਹਿਮੀਆਂ ਜਾਂ ਗਲਤ ਵਿਆਖਿਆਵਾਂ ਲਈ ਜਵਾਬਦੇਹ ਨਹੀਂ ਹਾਂ।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->