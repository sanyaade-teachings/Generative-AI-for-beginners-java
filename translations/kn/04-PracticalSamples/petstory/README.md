# ಪ್ರಾಣಿಗಳ ಕಥा ರಚನೆ ಜನಕ ಗೈಡ್ ಬೇಗಿನ್‌ರ್ಸ್‌ಗೆ

## ವಿಷಯ ಸೂಚಿ

- [ಆವಶ್ಯಕತೆಗಳು](#ಆವಶ್ಯಕತೆಗಳು)
- [ಯೋಜನೆ ಉತ್ತುಂಗವನ್ನು ಅರ್ಥಮಾಡಿಕೊಳ್ಳುವುದು](#ಯೋಚನೆಯ-ಉತ್ತುಂಗವನ್ನು-ಅರ್ಥಮಾಡಿಕೊಳ್ಳುವುದು)
- [ಮೂಲಭೂತ ಭಾಗಗಳ ವಿವರಣೆ](#ಮೂಲಭೂತ-ಭಾಗಗಳ-ವಿವರಣೆ)
  - [1. ಮುಖ್ಯ ಅನ್ವಯಿಕೆ](#1-ಮುಖ್ಯ-ಅನ್ವಯಿಕೆ)
  - [2. ವೆಬ್ ನಿಯಂತ್ರಕ](#2-ವೆಬ್-ನಿಯಂತ್ರಕ)
  - [3. ಕಥಾ ಸೇವೆ](#3-ಕಥಾ-ಸೇವೆ)
  - [4. ವೆಬ್ ಟೆಂಪ್ಲೇಟ್ಗಳು](#4-ವೆಬ್-ಟೆಂಪ್ಲೇಟ್ಗಳು)
  - [5. ಸಂರಚನೆ](#5-ಸಂರಚನೆ)
- [ಅನ್ವಯಿಕೆಯನ್ನು ಚಾಲನೆ ಮಾಡುವುದು](#ಅನ್ವಯಿಕೆಯನ್ನು-ಚಾಲನೆ-ಮಾಡುವುದು)
- [ಇಡೀ ವ್ಯವಸ್ಥೆ ಹೇಗೆ ಕೆಲಸ ಮಾಡುತ್ತದೆ](#ಇಡೀ-ವ್ಯವಸ್ಥೆ-ಹೇಗೆ-ಕೆಲಸ-ಮಾಡುತ್ತದೆ)
- [ಕೃತಕ ಬುದ್ಧಿಮತ್ತೆ ಸಂಯೋಜನೆಯನ್ನು ಅರ್ಥಮಾಡಿಕೊಳ್ಳುವುದು](#ಕೃತಕ-ಬುದ್ಧಿಮತ್ತೆ-ಸಂಯೋಜನೆಯನ್ನು-ಅರ್ಥಮಾಡಿಕೊಳ್ಳುವುದು)
- [ಮುಂದಿನ ಹಂತಗಳು](#ಮುಂದಿನ-ಹಂತಗಳು)

## ಆವಶ್ಯಕತೆಗಳು

ಆರಂಭಿಸುವ ಮುನ್ನ, ನೀವು ಹೊಂದಿರಬೇಕು:
- ಜಾವಾ 21 ಅಥವಾ ಮೇಲಿನ ಸಂಸ್ಕರಣೆ
- ಅವಲಂಬನೆ ನಿರ್ವಹಣೆಗೆ ಮೇವನ
- ಏಜೂರ್ AI ಫೌಂಡ್ರಿ ಮಾದರಿ ನಿಯೋಜನೆ (ನೀವು ಅದನ್ನು `azd up` ಮೂಲಕ ವ್ಯವಸ್ಥೆ ಮಾಡಬೇಕು — [ಅಧ್ಯಾಯ 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md) ನೋಡಿ), `az login` ಮೂಲಕ ಲಾಗಿನ್ (ಕೀಲೆಸ್ ಮಾನ್ಯತೆ)
- ಜಾವಾ, ಸ್ಪ್ರಿಂಗ್ ಬೂಟ್ ಮತ್ತು ವೆಬ್ ಅಭಿವೃದ್ಧಿಯ ಮೂಲಭೂತ ತಿಳಿವು

## ಯೋಚನೆಯ ಉತ್ತುಂಗವನ್ನು ಅರ್ಥಮಾಡಿಕೊಳ್ಳುವುದು

ಪ್ರಾಣಿ ಕಥಾ ಯೋಜನೆಯಲ್ಲಿರುವ ಕೆಲವು ಪ್ರಮುಖ ಕಡತಗಳು:

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

## ಮೂಲಭೂತ ಭಾಗಗಳ ವಿವರಣೆ

### 1. ಮುಖ್ಯ ಅನ್ವಯಿಕೆ

**ಕಡತ:** `PetStoryApplication.java`

ಇದು ನಮ್ಮ ಸ್ಪ್ರಿಂಗ್ ಬೂಟ್ ಅನ್ವಯಿಕೆಗೆ ಪ್ರವೇಶ ಬಿಂದು:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**ಇದರಿಂದ ಏನು ಮಾಡುತ್ತೆ:**
- `@SpringBootApplication` ಟ್ಯಾಗ್ ಸ್ವಯಂಚಾಲಿತ ಸಂರಚನೆ ಮತ್ತು ಘಟಕ ಸ್ಕ್ಯಾನಿಂಗ್ ಸಕ್ರಿಯಗೊಳಿಸುತ್ತದೆ
- ಪೋರ್ಟ್ 8080 ನಲ್ಲಿ ಅಳವಡಿಸಲಾದ ವೆಬ್ ಸರ್ವರ್ (ಟೋಂಕ್ಯಾಟ್) ಅನ್ನು ಪ್ರಾರಂಭಿಸುತ್ತದೆ
- ಎಲ್ಲಾ ಅಗತ್ಯ ಸ್ಪ್ರಿಂಗ್ ಬೀನ್ಸ್ ಮತ್ತು ಸೇವೆಗಳನ್ನು ಸ್ವಯಂಚಾಲಿತವಾಗಿ ಸೃಷ್ಟಿಸುತ್ತದೆ

### 2. ವೆಬ್ ನಿಯಂತ್ರಕ

**ಕಡತ:** `PetController.java`

ಇದು ಎಲ್ಲಾ ವೆಬ್ ವಿನಂತಿಗಳನ್ನೂ ಮತ್ತು ಬಳಕೆದಾರ ಸಂವಾದಗಳನ್ನೂ ನಿರ್ವಹಿಸುತ್ತದೆ:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // index.html ಟೆಂಪ್ಲೇಟ್ ಅನ್ನು ಹಿಂತಿರುಗಿಸುತ್ತದೆ
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // ಇನ್ಪುಟ್ ಪರಿಶೀಲನೆ
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // ಭದ್ರತೆಗಾಗಿ ಇನ್ಪುಟ್ ಸ್ವಚ್ಛಗೊಳಿಸಿ
        String sanitizedDescription = sanitizeInput(description);
        
        // ದೋಷ ನಿರ್ವಹಣೆಯೊಂದಿಗೆ ಕಥೆ ರಚಿಸಿ
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // result.html ಟೆಂಪ್ಲೇಟ್ ಅನ್ನು ಹಿಂತಿರುಗಿಸುತ್ತದೆ
            
        } catch (Exception e) {
            // AI ವಿಫಲವಾದರೆFallback ಕಥೆಯನ್ನು ಬಳಸಿ
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // ಉದ್ದವನ್ನು मर्यादಿತ ಮಾಡಿ
    }
}
```

**ಮುಖ್ಯ ವೈಶಿಷ್ಟ್ಯಗಳು:**

1. **ಮಾರ್ಗ ನಿರ್ವಹಣೆ**: `@GetMapping("/")` ಅಪ್‌ಲೋಡ್ ಫಾರ್ಮ್ ಪ್ರದರ್ಶಿಸುತ್ತದೆ, `@PostMapping("/generate-story")` ಸಲ್ಲಿಕೆಯನ್ನು ಪ್ರಕ್ರಿಯೆ ಪಡಿಸುತ್ತದೆ
2. **ಇನ್‌ಪುಟ್ ಪರಿಶೀಲನೆ**: ಖಾಲಿ ವಿವರಣೆಗಳು ಮತ್ತು ಉದ್ದದ ಮಿತಿಗಳನ್ನು ಪರಿಶೀಲಿಸುತ್ತದೆ
3. **ಭದ್ರತೆ**: ಬಳಕೆದಾರ ಇನ್‌ಪುಟ್ ಅನ್ನು XSS ದಾಳಿಗಳಿಂದ ರಕ್ಷಿಸುವಂತೆ ಸ್ವಚ್ಛಗೊಳಿಸುತ್ತದೆ
4. **ದೋಷ ನಿರ್ವಹಣೆ**: AI ಸೇವೆ ವಿಫಲವಾದಾಗ ಪರ್ಯಾಯ ಕಥೆಗಳನ್ನು ಒದಗಿಸುತ್ತದೆ
5. **ಮಾದರಿ ಬಂಧನ**: ಸ್ಪ್ರಿಂಗ್‌ನ `Model` ಬಳಸಿ HTML ಟೆಂಪ್ಲೇಟ್ಗೆ ಡೇಟಾ ಒದಗಿಸುತ್ತದೆ

**ಪರ್ಯಾಯ ವ್ಯವಸ್ಥೆ:**
ನಿಯಂತ್ರಕ AI ಸೇವೆ ಲಭ್ಯವಿಲ್ಲದಾಗ ಬಳಸಿ ಕೊಳ್ಳುವ ಪೂರ್ವ-ಬರೆದ ಕಥಾ ಟೆಂಪ್ಲೇಟ್ಗಳನ್ನು ಒಳಗೊಂಡಿದೆ:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // ಸತತ ಪ್ರತಿಕ್ರಿಯೆಗಳಿಗೆ ವಿವರಣೆ ಹ್ಯಾಶ್ ಅನ್ನು ಉಪಯೋಗಿಸಿ
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. ಕಥಾ ಸೇವೆ

**ಕಡತ:** `StoryService.java`

ಈ ಸೇವೆ ಕೀಲೆಸ್ ಮಾನ್ಯತೆಯೊಂದಿಗೆ ಏಜೂರ್ AI ಫೌಂಡ್ರಿಗೆ ಸಂಪರ್ಕಿಸುವ ಮೂಲಕ ಕಥೆಗಳನ್ನು ರಚಿಸುತ್ತದೆ:

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
        
        // ಫೌಂಡ್ರಿಯ OpenAI-ಸಾದೃಶ್ಯ ಎಂಡ್‌ಪಾಯಿಂಟ್ /openai/v1/ ಅಡಿ ಇರುತ್ತದೆ
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // ಮೈಕ್ರೋಸಾಫ್ಟ್ ಎಂಟ್ರಾ ID ನೊಂದಿಗೆ ಕೀಲಿಕಾರಣವಿಲ್ಲದ ಪ್ರಾಮಾಣೀಕರಣ (ಯಾವುದೇ API ಕೀಲಿಯಿಲ್ಲ)
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
        
        // AI ವಿನಂತಿಯನ್ನು ರಚಿಸು
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // ಪ್ರತಿಕ್ರಿಯೆಯ ಉದ್ದವನ್ನು ಮಿತಿ ಹಾಕಿ
                .temperature(0.8)          // ಸೃಜನಶೀಲತೆ ನಿಯಂತ್ರಿಸು (0.0-1.0)
                .build();
        
        // ವಿನಂತಿಯನ್ನು ಕಳುಹಿಸಿ ಮತ್ತು ಪ್ರತಿಕ್ರಿಯೆ ಪಡೆಯಿರಿ
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**ಪ್ರಮುಖ ಘಟಕಗಳು:**

1. **OpenAI ಕ್ಲೈಂಟ್**: ಅಧಿಕೃತ OpenAI ಜಾವಾ SDK ಬಳಸಿ ಏಜೂರ್ AI ಫೌಂಡ್ರಿಗಾಗಿ ಕೀಲೆಸ್ ಮಾದರಿಯನ್ನು ಸಂರಚಿಸಲಾಗಿದೆ
2. **ಸಿಸ್ಟಂ ಪ್ರಾಂಪ್ಟ್**: AI ಅವರನ್ನು ಕುಟುಂಬ ಸ್ನೇಹಿ ಪ್ರಾಣಿ ಕಥೆಗಳನ್ನು ಬರೆಯಲು ನಿರ್ದಿಷ್ಟಪಡಿಸುತ್ತದೆ
3. **ಬಳಕೆದಾರ ಪ್ರಾಂಪ್ಟ್**: ವಿವರಣೆಯ ಆಧಾರದ ಮೇಲೆ ಕಥೆಯನ್ನು AIಗೆ ಹೇಳುತ್ತದೆ
4. **ಪ್ಯಾರಾಮೀಟರ್ಗಳು**: ಕಥಾ ಉದ್ದ ಮತ್ತು ಸೃಜನಶೀಲತೆಯನ್ನು ನಿಯಂತ್ರಿಸುತ್ತದೆ
5. **ದೋಷ ನಿರ್ವಹಣೆ**: ನಿಯಂತ್ರಕ ಹಿಡಿಯುವ ಮತ್ತು ನಿಭಾಯಿಸುವ ಹೊರಟಿರುವ Exception ಗಳನ್ನು ತள்ளುವದು

### 4. ವೆಬ್ ಟೆಂಪ್ಲೇಟ್ಗಳು

**ಕಡತ:** `index.html` (ಅಪ್‌ಲೋಡ್ ಫಾರ್ಮ್)

ಪ್ರಧಾನ ಪುಟ, ಇಲ್ಲಿ ಬಳಕೆದಾರರು ತಮ್ಮ ಪ್ರಾಣಿಗಳನ್ನು ವರ್ಣಿಸುತ್ತಾರೆ:

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

**ಕಡತ:** `result.html` (ಕಥಾವೇಳಗಿನ ಪ್ರದರ್ಶನ)

ರಚಿಸಿದ ಕಥೆಯನ್ನು ತೋರಿಸುತ್ತದೆ:

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

**ಟೆಂಪ್ಲೇಟ್ ವೈಶಿಷ್ಟ್ಯಗಳು:**

1. **ಥೈಮ್‍ಲೀಫ್ ಸಂಯೋಜನೆ**: ಡೈನಾಮಿಕ್ ವಿಷಯಕ್ಕಾಗಿ `th:` ಗುಣಲಕ್ಷಣಗಳನ್ನು ಬಳಸುತ್ತದೆ
2. **ಪ್ರತಿಕ್ರಿಯಾಶೀಲ ವಿನ್ಯಾಸ**: ಮೊಬೈಲ್ ಮತ್ತು ಡೆಸ್ಕ್‌ಟಾಪ್‌ನಿಗಾಗಿ CSS ಶೈಲಿ
3. **ದೋಷ ನಿರ್ವಹಣೆ**: ಬಳಕೆದಾರರಿಗೆ ಮಾನ್ಯತೆ ದೋಷಗಳನ್ನು ಪ್ರದರ್ಶಿಸುತ್ತದೆ
4. **ಗ್ರಾಹಕಪಾರ್ಶ್ವ ಪ್ರಕ್ರಿಯೆ**: ಇಮೇಜ್ ವಿಶ್ಲೇಷಣೆಗೆ ಜಾವಾಸ್ಕ್ರಿಪ್ಟ್ (Transformers.js ಬಳಸಿ)

### 5. ಸಂರಚನೆ

**ಕಡತ:** `application.properties`

ಅನ್ವಯಿಕೆಯ ಸಂರಚನಾ ಸೆಟ್ಟಿಂಗ್ಸ್:

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

**ಸಂರಚನೆಯ ವಿವರಣೆ:**

1. **ಫೈಲ್ ಅಪ್‌ಲೋಡ್**: 10MB ವರೆಗೆ ಚಿತ್ರದ ಅಪ್‌ಲೋಡ್‌ಗೆ ಅನುಮತಿಸುತ್ತದೆ
2. **ಲಾಗಿಂಗ್**: ನಿರ್ವಹಣೆಯ ಸಮಯದಲ್ಲಿ ಲಾಗ್ ನೋಂದಣಿಯನ್ನು ನಿಯಂತ್ರಿಸುತ್ತದೆ
3. **ಏಜೂರ್ AI ಫೌಂಡ್ರಿ**: ಕೀಲೆಸ್ ಮಾನ್ಯತೆಯೊಂದಿಗೆ ಬಳಸಬೇಕಾದ ಎಂಡ್‌ಪಾಯಿಂಟ್ ಮತ್ತು ಮಾದರಿ ನಿಯೋಜನೆ ಸೂಚಿಸುತ್ತದೆ
4. **ಭದ್ರತೆ**: ಸಂವೇದನಾಶೀಲ ಮಾಹಿತಿಯನ್ನು ಬಾಹ್ಯಪಡಿಸುವುದನ್ನು ತಪ್ಪಿಸಲು ದೋಷ ನಿರ್ವಹಣೆ ಸಂರಚನೆ

## ಅನ್ವಯಿಕೆಯನ್ನು ಚಾಲನೆ ಮಾಡುವುದು

### ಹಂತ 1: ಸೈನ್ ಇನ್ ಮಾಡಿ ಮತ್ತು ನಿಮ್ಮ ಎಂಡ್‌ಪಾಯಿಂಟ್ ಅನ್ನು ಸೆಟ್ ಮಾಡಿ

ಮಾನ್ಯತೆ ಕೀಲೆಸ್ ಆಗಿದ್ದು (Microsoft Entra ID), ಆಗ API ಕೀ ಅಗತ್ಯವಿಲ್ಲ. ಸೈನ್ ಇನ್ ಮಾಡಿ ಮತ್ತು ಫೌಂಡ್ರಿ ಎಂಡ್‌ಪಾಯಿಂಟ್ ಅನ್ನು ಹೊಂದಿಸಿ:

**ವಿಂಡೋಸ್ (ಕಮಾಂಡ್ ಪ್ರಾಂಪ್ಟ್):**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**ವಿಂಡೋಸ್ (ಪವರ್ ಶೆಲ್):**
```powershell
az login
$env:AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
```

**ಲಿನಕ್ಸ್ನ/ಮ್ಯಾಕ್ಗಳಲ್ಲಿ:**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**ಯಾಕೆ ಇದಕ್ಕಿದೆ ಅವಶ್ಯಕತೆ:**
- ಏಜೂರ್ AI ಫೌಂಡ್ರಿ Microsoft Entra ID ಮೂಲಕ ಮಾನ್ಯತೆ ನೀಡುತ್ತದೆ
- ಕೀಲೆಸ್ ಮಾನ್ಯತೆ ಅಂದರೆ ಸ್ರೋತ ಕೋಡ್ ಅಥವಾ ಪರಿಸರದಲ್ಲಿ ಯಾವುದೇ ರಹಸ್ಯ ಇರುವುದಿಲ್ಲ
- ನಿಮ್ಮ ಖಾತೆಗೆ **Cognitive Services OpenAI User** ಪಾತ್ರ ಕಳುಹಿಸಬೇಕಾಗುತ್ತದೆ

### ಹಂತ 2: ನಿರ್ಮಿಸಿ ಮತ್ತು ಚಾಲನೆ ಮಾಡು

ಯೋಜನೆ ಡೈರೆಕ್ಟರಿಗಾಗಿ ಭೇಟಿ ನೀಡಿ:
```bash
cd 04-PracticalSamples/petstory
```

ಅನ್ವಯಿಕೆಯನ್ನು ನಿರ್ಮಿಸಿ:
```bash
mvn clean compile
```

ಸರ್ವರ್ ಪ್ರಾರಂಭಿಸಿ:
```bash
mvn spring-boot:run
```

ಅನ್ವಯಿಕೆ ಮುಂದೆ `http://localhost:8080` ನಲ್ಲಿ ಚಾಲನೆಯಲ್ಲಿರುತ್ತದೆ.

### ಹಂತ 3: ಅನ್ವಯಿಕೆಯನ್ನು ಪರೀಕ್ಷಿಸಿ

1. **ತೆರೆಯಿರಿ** `http://localhost:8080` ಅನ್ನು ನಿಮ್ಮ ಬ್ರೌಸರ್‌ನಲ್ಲಿ
2. **ನಿಮ್ಮ ಪ್ರಾಣಿ ವಿವರಣೆ ನೀಡಿ** (ಉದಾಹರಣೆಗೆ, "ಫೆಚ್ಚು ಹಿಡಿಯಲು ಇಷ್ಟಪಡುವ ಆಟಗಾರ ಗೋಲ್ಡನ್ ರಿಟ್ಟ್ರೀವರ್")
3. **"Generate Story" ಕ್ಲಿಕ್ ಮಾಡಿ** AI-ರಚಿಸಲಾದ ಕಥೆಯನ್ನು ಪಡೆಯಿರಿ
4. **ಮತ್ತೊಂದು ವಿಧಾನವಾಗಿ**, ಪ್ರಾಣಿ ಚಿತ್ರ ಅಪ್‌ಲೋಡ್ ಮಾಡಿ ಸ್ವಯಂಚಾಲಿತ ವಿವರಣೆ ಪಡೆಯಿರಿ
5. **ನೋಡಿ** ನಿಮ್ಮ ಪ್ರಾಣಿಯ ವಿವರಣೆಯ ಆಧಾರದ ಮೇಲೆ ಸೃಜನಶೀಲ ಕಥೆಯನ್ನು

## ಇಡೀ ವ್ಯವಸ್ಥೆ ಹೇಗೆ ಕೆಲಸ ಮಾಡುತ್ತದೆ

ನೀವು ಪ್ರಾಣಿ ಕಥೆ ರಚಿಸುವಾಗ ಪೂರ್ಣ ಪ್ರಕ್ರಿಯೆ:

1. **ಬಳಕೆದಾರ ಇನ್‌ಪುಟ್**: ನೀವು ವೆಬ್ ಫಾರ್ಮ್‌ನಲ್ಲಿ ಪ್ರಾಣಿಯ ವಿವರಣೆ ನೀಡುತ್ತಾರೆ
2. **ಫಾರ್ಮ್ ಸಲ್ಲಿಕೆ**: ಬ್ರೌಸರ್ POST ವಿನಂತಿ `/generate-story` ಗೆ ಕಳುಹಿಸುತ್ತದೆ
3. **ನಿಯಂತ್ರಕ ಪ್ರಕ್ರಿಯೆ**: `PetController` ಇನ್‌ಪುಟ್ ಪರಿಶೀಲನೆ ಮತ್ತು ಸ್ವಚ್ಛಗೊಳಿಸುತ್ತದು
4. **AI ಸೇವೆ ಕರೆ**: `StoryService` ಏಜೂರ್ AI ಫೌಂಡ್ರಿ ಮಾದರಿಗೆ ವಿನಂತಿ ಕಳುಹಿಸುತ್ತದೆ
5. **ಕಥೆ ರಚನೆ**: AI ವಿವರಣೆಯ ಆಧಾರದ ಮೇಲೆ ಸೃಜನಾತ್ಮಕ ಕಥೆ ಸೃಷ್ಟಿಸುತ್ತದೆ
6. **ಪ್ರತಿಕ್ರಿಯೆ ಸ್ವೀಕಾರ**: ನಿಯಂತ್ರಕ ಕಥೆಯನ್ನು ಪಡೆದು ಮಾದರಿಗೆ ಸೇರಿಸುತ್ತದೆ
7. **ಟೆಂಪ್ಲೇಟ್ ರೆಂಡರಿಂಗ್**: ಥೈಮ್‍ಲೀಫ್ `result.html` ಅನ್ನು ಕಥೆಯೊಂದಿಗೆ ರೆಂಡರ್ ಮಾಡುತ್ತದೆ
8. **ಪ್ರದರ್ಶನ**: ಬಳಕೆದಾರನು ತನ್ನ ಬ್ರೌಸರ್‌ನಲ್ಲಿ ಕಥೆಯನ್ನು ನೋಡುತ್ತಾನೆ

**ದೋಷ ನಿರ್ವಹಣೆ ಪ್ರಕ್ರಿಯೆ:**
AI ಸೇವೆ ವಿಫಲವಾದರೆ:
1. ನಿಯಂತ್ರಕ Exception ಹಿಡಿಯುತ್ತದೆ
2. ಪೂರ್ವ-ಬರೆದ ಟೆಂಪ್ಲೇಟುಗಳಿಂದ ಪರ್ಯಾಯ ಕಥೆಯನ್ನು ರಚಿಸುತ್ತದೆ
3. AI ಲಭ್ಯವಿಲ್ಲದ ಬಗ್ಗೆ ಸೂಚನೆಯೊಂದಿಗೆ ಪರ್ಯಾಯ ಕಥೆಯನ್ನು ತೋರಿಸುತ್ತದೆ
4. ಬಳಕೆದಾರಕ್ಕೆ ಸರಿಯ ಆದರ್ಶ ಅನುಭವ ಒದಗಿಸುತ್ತದೆ

## ಕೃತಕ ಬುದ್ಧಿಮತ್ತೆ ಸಂಯೋಜನೆಯನ್ನು ಅರ್ಥಮಾಡಿಕೊಳ್ಳುವುದು

### ಏಜೂರ್ AI ಫೌಂಡ್ರಿ (ಕೀಲೆಸ್)
ಅನ್ವಯಿಕೆ Microsoft Entra ID ಮೂಲಕ ಕೀಲೆಸ್ ಮಾನ್ಯತೆಯೊಂದಿಗೆ ಏಜೂರ್ AI ಫೌಂಡ್ರಿ ಬಳಸуда:

```java
// ಕೀಲಿವಿಲ್ಲದ ಪ್ರಾಮಾಣೀಕರಣ - API ಕೀ ಇಲ್ಲದೆ
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### ಪ್ರಾಂಪ್ಟ್ ಇಂಜಿನಿಯರಿಂಗ್
ಸೇವೆಯನ್ನು ಉತ್ತಮ ಫಲಿತಾಂಶ ಪಡೆಯಲು ಜಾಗರೂಕತೆಯಿಂದ ವಿನ್ಯಾಸಗೊಳಿಸಿದ ಪ್ರಾಂಪ್ಟ್‌ಗಳನ್ನು ಬಳಸುತ್ತದೆ:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### ಪ್ರತಿಕ್ರಿಯೆ ಸಂಸ್ಕರಣೆ
AI ಪ್ರತಿಕ್ರಿಯೆಯನ್ನು ತೆಗೆಯಲಾಗುತ್ತಿತ್ತು ಮತ್ತು ಪರಿಶೀಲಿಸಲಾಗುತ್ತಿತ್ತು:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## ಮುಂದಿನ ಹಂತಗಳು

ಹೆಚ್ಚು ಉದಾಹರಣೆಗಳಿಗೆ, [ಅಧ್ಯಾಯ 04: ಪ್ರಾಯೋಗಿಕ ಮಾದರಿಗಳು](../README.md) ನೋಡಿ

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ಅಸ್ವೀಕಾರ**:
ಈ ದಸ್ತಾವೇಜು AI ಅನುವಾದ ಸೇವೆ [Co-op Translator](https://github.com/Azure/co-op-translator) ಬಳಸಿ ಅನುವಾದಿಸಲಾಗಿದೆ. ನಾವು ನಿಖರತೆಯನ್ನು ಸಾಧಿಸಲು ಪ್ರಯತ್ನಿಸುತ್ತಿದ್ದರೂ, ದಯವಿಟ್ಟು ಗಮನಿಸಿ, ಸ್ವಯಂಚಾಲಿತ ಅನುವಾದಗಳಲ್ಲಿ ದೋಷಗಳು ಅಥವಾ ಅಸಡ್ಡೆಗಳು ಇರಬಹುದು. ಮೂಲ ಭಾಷೆಯಲ್ಲಿರುವ ಮೂಲ ದಸ್ತಾವೇಜು ಪ್ರಾಮಾಣಿಕ ಮೂಲವೆಂದು ಪರಿಗಣಿಸಬೇಕು. ಪ್ರಮುಖ ಮಾಹಿತಿಗಾಗಿ, ವೃತ್ತಿಪರ ಮಾನವ ಅನುವಾದವನ್ನು ಶಿಫಾರಸು ಮಾಡಲಾಗುತ್ತದೆ. ಈ ಅನುವಾದವನ್ನು ಬಳಸುವ ಮೂಲಕ ಉಂಟಾಗುವ ಯಾವುದೇ ತಪ್ಪು ಅರ್ಥಗಳ ಅಥವಾ ತಪ್ಪು ವ್ಯಾಖ್ಯಾನಗಳ ಬಗ್ಗೆ ನಾವು ಹೊಣೆಗಾರರಲ್ಲ.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->