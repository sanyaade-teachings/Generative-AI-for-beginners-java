# പാറ്റ് സ്റ്റോറി ജനറേറ്റർ ട്യുട്ടോറിയൽ ആരംഭക്കാർക്കുള്ളത്

## ഉള്ളടക്ക പട്ടിക

- [ആവശ്യമായ മുൻപരിചയങ്ങൾ](#ആവശ്യമുള്ള-മുൻപരിചയങ്ങൾ)
- [പ്രോജക്റ്റ് ഘടന മനസിലാക്കുക](#പ്രോജക്റ്റ്-ഘടന-മനസിലാക്കുക)
- [പ്രധാന ഘടകങ്ങൾ വിശദീകരണം](#പ്രധാന-ഘടകങ്ങൾ-വിശദീകരണം)
  - [1. പ്രധാന ആപ്ലിക്കേഷൻ](#1-പ്രധാന-ആപ്ലിക്കേഷൻ)
  - [2. വെബ് കൺട്രോളർ](#2-വെബ്-കൺട്രോളർ)
  - [3. സ്റ്റോറി സർവീസ്](#3-സ്റ്റോറി-സർവീസ്)
  - [4. വെബ് ടെംപ്ലേറ്റുകൾ](#4-വെബ്-ടെംപ്ലേറ്റുകൾ)
  - [5. കോൺഫിഗറേഷൻ](#5-കോൺഫിഗറേഷൻ)
- [ആപ്ലിക്കേഷൻ റൺ ചെയ്‌தൽ](#ആപ്ലിക്കേഷൻ-റൺ-ചെയ്യൽ)
- [എങ്ങനെയാണ് എല്ലാം ചേർന്ന് പ്രവർത്തിക്കുന്നത്](#എങ്ങനെ-എല്ലാം-ചേർന്ന്-പ്രവർത്തിക്കുന്നു)
- [എഐ സംയോജനം മനസിലാക്കുക](#എഐ-സംയോജനം-മനസിലാക്കുക)
- [അടുത്ത നടപടികൾ](#അടുത്ത-നടപടികൾ)

## ആവശ്യമുള്ള മുൻപരിചയങ്ങൾ

ആരംഭിക്കുന്നതിന് മുമ്പ്, നിങ്ങൾക്കുണ്ടാകണം:
- ജാവ 21 അല്ലെങ്കിൽ അതിന് മുകളിൽ ഇൻസ്റ്റാൾ ചെയ്ത്
- ഡിപ്പെൻഡൻസി മാനേജ്മെന്റിനായി മേവൻ
- ഒരു Azure AI Foundry മോഡൽ ഡിപ്ലോയ്‌മെന്റ് (`azd up` ഉപയോഗിച്ച് പ്രൊവിഷൻ ചെയ്യുക — [Chapter 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md) കാണുക), `az login` (കീലെസ് ഒതെന്റിക്കേഷൻ) ഉപയോഗിച്ച് സൈൻ ഇൻ ചെയ്ത
- ജാവ, സ്പ്രിംഗ് ബൂട്ട്, വെബ് ഡെവലപ്‌മെന്റ് എന്നിവയുടെ അടിസ്ഥാന പരിജ്ഞാനം

## പ്രോജക്റ്റ് ഘടന മനസിലാക്കുക

പാറ്റ് സ്റ്റോറി പ്രോജക്റ്റിന് താഴെപ്പറയുന്ന പ്രധാന ഫയലുകൾ ഉണ്ട്:

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

## പ്രധാന ഘടകങ്ങൾ വിശദീകരണം

### 1. പ്രധാന ആപ്ലിക്കേഷൻ

**ഫയൽ:** `PetStoryApplication.java`

നമ്മുടെ സ്പ്രിംഗ് ബൂട്ട് ആപ്ലിക്കേഷന്റെ എൻട്രി പോയിന്റ് ഇതാണ്:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**ഇത് ചെയ്യുന്നത്:**
- `@SpringBootApplication` അനോടേഷൻ ഓട്ടോ-കൺഫിഗറേഷൻയും കമ്പോണന്റ് സ്കാനിങും സജ്ജീകരിക്കുന്നു
- 8080 പോർട്ടിൽ ഒരു എംബെഡഡ് വെബ് സർവർ (ടോമ്കാറ്റ്) തുടങ്ങുന്നു
- ആവശ്യമായ എല്ലാ സ്പ്രിംഗ് ബീൻസ്, സർവ്വീസുകൾ സ്വയം സൃഷ്ടിക്കുന്നു

### 2. വെബ് കൺട്രോളർ

**ഫയൽ:** `PetController.java`

ഇങ്ങനെ വെബ് അഭ്യർത്ഥനകളും ഉപയോക്തൃ ഇടപെടലുകളും കൈകാര്യം ചെയ്യുന്നു:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // index.html ടെംപ്ലേറ്റ് മടക്കുന്നു
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // ഇൻപുട്ട് സാധുത പരിശോധിക്കൽ
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // സുരക്ഷയ്ക്ക് വേണ്ടി ഇൻപുട്ട് ശുദ്ധീകരിക്കുക
        String sanitizedDescription = sanitizeInput(description);
        
        // പിശക് കൈകാര്യം ചെയ്ത് കഥ സൃഷ്ടിക്കുക
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // result.html ടെംപ്ലേറ്റ് മടക്കുന്നു
            
        } catch (Exception e) {
            // AI പരാജയപ്പെട്ടാൽ ഫല്ബാക്ക് കഥ ഉപയോഗിക്കുക
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // നീളം നിയന്ത്രിക്കുക
    }
}
```

**പ്രധാന സവിശേഷതകൾ:**

1. **റൂട്ടിംഗ് കൈകാര്യം ചെയ്യൽ**: `@GetMapping("/")` അപ്‌ലോഡ് ഫോം കാണിക്കുന്നു, `@PostMapping("/generate-story")` സമർപ്പണങ്ങൾ പ്രോസസ് ചെയ്യുന്നു
2. **ഇന്പുട്ട് പരിശോധന**: ഒഴിവുള്ള വിവരണങ്ങൾക്കും നീള പരിധികൾക്കും പരിശോധന
3. **സുരക്ഷ**: ഉപയോക്തൃ ഇൻപുട്ട് XSS ആക്രമണങ്ങളിൽ നിന്നും സംരക്ഷിക്കുന്നതിനായി ശുദ്ധീകരിക്കുന്നു
4. **പിശക് കൈകാര്യം ചെയ്യൽ**: AI സർവീസ് പരാജയപ്പെട്ടാൽ ഫാൾബാക്ക് കഥകൾ നൽകുന്നു
5. **മോഡൽ ബൈൻഡിംഗ്**: സ്പ്രിംഗ് `Model` ഉപയോഗിച്ച് ഡേറ്റ HTML ടെംപ്ലേറ്റുകൾക്കായി പാസ് ചെയ്യുന്നു

**ഫാൾബാക്ക് സംവിധാനം:**
AI സർവീസ് ലഭ്യമല്ലെങ്കിൽ ഉപയോഗിക്കുന്ന മുൻകൂട്ടി എഴുതിയ സ്റ്റോറി ടെംപ്ലേറ്റുകൾ ഉൾക്കൊള്ളുന്നു:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // സ്ഥിരമായ പ്രതികരണങ്ങള്‍ക്കായി വിവരണ ഹാഷ് ഉപയോഗിക്കുക
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. സ്റ്റോറി സർവീസ്

**ഫയൽ:** `StoryService.java`

കീലെസ് ഒതെന്റിക്കേഷൻ ഉപയോഗിച്ച് Azure AI Foundry-യുമായി സംവാദം നടത്തിയാണ് കഥകൾ സൃഷ്ടിക്കുന്നത്:

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
        
        // Foundryയുടെ OpenAI-സാമ്യം ഉള്ള എന്റ്പോയിന്റ് /openai/v1/അടിസ്ഥാനത്തിലാണ്
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Microsoft Entra ID ഉപയോഗിച്ച് കീ ഇല്ലാതെയുള്ള ഓതന്റിക്കേഷൻ (API കീ കൂടാതെ)
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
        
        // AI അഭ്യർത്ഥന ക്രമീകരിക്കുക
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // പ്രതികരണത്തിന്റെ നീളം പരിമിതപ്പെടുത്തുക
                .temperature(0.8)          // സൃഷ്ടിപരത്വം നിയന്ത്രിക്കുക (0.0-1.0)
                .build();
        
        // അഭ്യർത്ഥന അയച്ച് പ്രതികരണം მიიღുക
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**പ്രധാന ഘടകങ്ങൾ:**

1. **OpenAI ക്ലയന്റ്**: ഫീസ് ഔദ്യോഗിക OpenAI ജावा SDK Azure AI Foundry (കീലെസ്)ക്കായി ക്രമീകരിച്ചിരിക്കുന്നു
2. **സിസ്റ്റം പ്രോംപ്റ്റ്**: AI-നെ കുടുംബസന്തതമകളെ വേണ്ടി അനുയോജ്യമായ പാറ്റ് കഥകൾ എഴുതാൻ നിയോഗിക്കുന്നു
3. **ഉപയോക്തൃ പ്രോംപ്റ്റ്**: വിവരണത്തെ ആശ്രയിച്ച് ഏത് കഥ എഴുതണമെന്ന് AI-ന് വ്യക്തമാക്കുന്നു
4. **പാരാമീറ്ററുകൾ**: കഥയുടെ നീളവും സൃഷ്ടിപ്രവർത്തനപ്പരിസരം നിയന്ത്രിക്കുന്നു
5. **പിശക് കൈകാര്യം ചെയ്യൽ**: നിയന്ത്രകൻ പിടിക്കുന്ന വ്യത്യാസങ്ങൾ Throw ചെയ്യുന്നു

### 4. വെബ് ടെംപ്ലേറ്റുകൾ

**ഫയൽ:** `index.html` (അപ്‌ലോഡ് ഫോം)

ഉപയോക്താക്കൾ തങ്ങളുടെ പാറ്റ് വിവരണം നൽകുന്ന പ്രധാന പേജ്:

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

**ഫയൽ:** `result.html` (സ്റ്റോറി പ്രദർശനം)

സൃഷ്ടിച്ച കഥ പ്രദർശിപ്പിക്കുന്നു:

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

**ടെംപ്ലേറ്റ് സവിശേഷതകൾ:**

1. **തയ്മ്ലീഫ് ഇന്റഗ്രേഷൻ**: ഡൈനാമിക് ഉള്ളടക്കത്തിനായി `th:` ഗുണധർമ്മങ്ങൾ ഉപയോഗിക്കുന്നു
2. **റിസ്പോൺസീവ് ഡിസൈൻ**: മൊബൈൽ, ഡെസ്ക്ടോപ് ഇരുത്തും CSS സ്റ്റയിലിങ്ങ്
3. **പിശക് കൈകാര്യം ചെയ്യൽ**: ഉപയോക്താക്കൾക്ക് പരിശോധന പിശകുകൾ പ്രദർശിപ്പിക്കുന്നു
4. **ക്ലയന്റ്-സൈഡ് പ്രോസസ്സിംഗ്**: ഇമേജ് അനാലിസിസ് വേണ്ടിയുള്ള ജാവാസ്ക്രിപ്റ്റ് (Transformers.js ഉപയോഗിച്ച്)

### 5. കോൺഫിഗറേഷൻ

**ഫയൽ:** `application.properties`

ആപ്ലിക്കേഷന്റെ കോൺഫിഗറേഷൻ സെറ്റിങ്ങുകൾ:

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

**കോണ്ഫിഗർ വിശദീകരണം:**

1. **ഫയൽ അപ്‌ലോഡ്**: 10MB വരെയുള്ള ചിത്രങ്ങൾ അനുവദിക്കുന്നു
2. **ലോഗിംഗ്**: എക്സിക്യൂഷനിൽ ലോഗ് ചെയ്യേണ്ട വിവരങ്ങൾ നിയന്ത്രിക്കുന്നു
3. **Azure AI Foundry**: ഉപയോഗിക്കുവാനുള്ള എൻഡ്പോയിന്റും മോഡൽ ഡിപ്ലോയ്‌മെന്റും സജ്ജീകരിക്കുന്നു (കീലെസ് ഒതെന്റിക്കേഷൻ)
4. **സുരക്ഷ**: എയർ ഹാന്റ്ലിംഗ് കോൺഫിഗറേഷൻ ലഘുലേഖകൾ വെളിപ്പെടുത്താതിരിക്കാൻ

## ആപ്ലിക്കേഷൻ റൺ ചെയ്യൽ

### ഘട്ടം 1: സൈൻ ഇൻ ചെയ്‌തോയും എൻഡ്പോയിന്റ് സജ്ജീകരിച്ചോ

ഒതെന്റിക്കേഷൻ കീലെസ് (Microsoft Entra ID) ആണ്, അതിനാൽ API കീ ഇല്ല. സൈൻ ഇൻ ചെയ്ത് നിങ്ങളുടെ ഫൗണ്ട്രി എൻഡ്പോയിന്റ് സജ്ജീകരിക്കുക:

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

**ഇത് ആവശ്യമുള്ള കാരണം:**
- Azure AI Foundry Microsoft Entra ID ഉപയോഗിച്ച് ഇൻഫറൻസ് അഭ്യർത്ഥനകൾ ഒതെന്റിക്കേറ്റ് ചെയ്യുന്നു
- കീലെസ് ഒതെന്റിക്കേഷൻ അടിസ്ഥാനത്തിൽ നിങ്ങളുടെ സോഴ്‌സ് കോഡിലും എങ്ക്‌ളോയ്‌മെന്റിലും രഹസ്യങ്ങൾ വരുകയില്ല
- നിങ്ങളുടെ അക്കൗണ്ടിന് **Cognitive Services OpenAI User** റോളും റിസോഴ്സിലും വേണം

### ഘട്ടം 2: ബിൽഡ് ചെയ്യുകയും റൺ ചെയ്യുകയും ചെയ്യുക

പ്രോജക്റ്റ് ഡയറക്ടറിയിലേക്ക് പോകുക:
```bash
cd 04-PracticalSamples/petstory
```

ആപ്ലിക്കേഷൻ നിർമ്മിക്കുക:
```bash
mvn clean compile
```

സർവർ ആരംഭിക്കുക:
```bash
mvn spring-boot:run
```

ആപ്ലിക്കേഷൻ ആരംഭിക്കും `http://localhost:8080` ൽ.

### ഘട്ടം 3: ആപ്ലിക്കേഷൻ പരിശോധന

1. ബ്രൗസറിൽ `http://localhost:8080` തുറക്കുക
2. നിങ്ങളുടെ പാറ്റ് വിവരിക്കുക (ഉദാ: "ഫെച്ച് ചെയ്യാൻ ഇഷ്ടമുള്ള ഒരു ആവേശമുള്ള ഗോൾഡൺ റെട്രീവർ")
3. "Generate Story" ക്ലിക്ക് ചെയ്ത് AI സൃഷ്ടിച്ച കഥ ലഭിക്കുക
4. അല്ലെങ്കിൽ, പാറ്റ് ചിത്രം അപ്‌ലോഡ് ചെയ്ത് ഓട്ടോമാറ്റിക്കായി വിവരണം സൃഷ്ടിക്കുക
5. നിങ്ങളുടെ പാറ്റിന്റെ വിവരണത്തിന്റെ അടിസ്ഥാനത്തിൽ സൃഷ്ടിപരമായ കഥ കാണുക

## എങ്ങനെ എല്ലാം ചേർന്ന് പ്രവർത്തിക്കുന്നു

പാറ്റ് കഥ സൃഷ്‌ടിക്കുമ്പോൾ ഉള്ള സമ്പൂർണ പ്രവാഹം:

1. **ഉപയോക്തൃ ഇൻപുട്ട്**: നിങ്ങൾ വെബ് ഫോമിൽ നിങ്ങളുടെ പാറ്റ് വിവരിക്കുന്നു
2. **ഫോം സമർപ്പണം**: ബ്രൗസർ POST അഭ്യർത്ഥന `/generate-story` ലേക്ക് അയക്കും
3. **കൺട്രോളർപ്രവര്ത്തനം**: `PetController` ഇൻപുട്ട് സ്ഥിരീകരിക്കുകയും ശുദ്ധീകരിക്കുകയും ചെയ്യുന്നു
4. **AI സർവീസ് കോൾ**: `StoryService` Azure AI Foundry മോഡലിലേക്ക് അഭ്യർത്ഥന അയയ്ക്കുന്നു
5. **കഥ സൃഷ്ടി**: AI വിവരണത്തെ ആശ്രയിച്ച് സൃഷ്ടിപരമായ ഒരു കഥ എഴുതുന്നു
6. **റസ്പോൺസ് കൈകാര്യം ചെയ്യൽ**: കൺട്രോളർ കഥ സ്വീകരിച്ച് മോഡലിൽ ചേർക്കുന്നു
7. **ടെംപ്ലേറ്റ് റെൻഡറിംഗും**: തയ്മ്ലീഫ് `result.html` ഉണ്ടായ കഥയോടുകൂടി റണ്ടർ ചെയ്യുന്നു
8. **പ്രദർശനം**: ഉപയോക്തൃ ബ്രൗസറിൽ സൃഷ്ടിച്ച കഥ കാണുന്നു

**പിശക് കൈകാര്യം പ്രവാഹം:**
AI സർവീസ് പരാജയപ്പെട്ടാൽ:
1. കൺട്രോളർ wyjątion പിടിക്കും
2. മുൻകൂട്ടി എഴുതിയ ഫാൾബാക്ക് കഥ ഉപയോഗിച്ച് ഒരു കഥ സൃഷ്ടിക്കും
3. AI ലഭ്യമല്ലെന്ന കുറിപ്പോടുകൂടി ഫാൾബാക്ക് കഥ പ്രദർശിപ്പിക്കും
4. ഉപയോക്താവ് കഥ ലഭിക്കും; നല്ല ഉപയോക്തൃ അനുഭവം ഉറപ്പ് വരുത്തും

## എഐ സംയോജനം മനസിലാക്കുക

### Azure AI Foundry (കീലെസ്)
ആപ്ലിക്കേഷൻ Azure AI Foundry കീലെസ് ഒതെന്റിക്കേഷൻ (Microsoft Entra ID) ഉപയോഗിച്ച് പ്രവർത്തിക്കുന്നു:

```java
// കീലെസ്സ് ഓത്തന്റിക്കേഷൻ - API കീ ഇല്ലാതെ
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### പ്രോംപ്റ്റ് എൻജിനിയറിംഗ്
നല്ല ഫലങ്ങൾ നേടാൻ സർവീസ് സൂക്ഷ്മമായി രൂപകൽപ്പന ചെയ്ത പ്രോംപ്റ്റുകൾ ഉപയോഗിക്കുന്നു:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### പ്രതികരണം പ്രോസസ്സിംഗ്
AI ന്റെ പ്രതികരണം എടുക്കുകയും ശരിയെന്ന് പരിശോധിക്കുകയും ചെയ്യുന്നു:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## അടുത്ത നടപടികൾ

കൂടുതൽ ഉദാഹരണങ്ങൾക്ക് [Chapter 04: Practical samples](../README.md) കാണുക

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**അറിയിപ്പ്**:
ഈ രേഖ AI പരിഭാഷാ സേവനം [Co-op Translator](https://github.com/Azure/co-op-translator) ഉപയോഗിച്ച് പരിഭാഷപ്പെടുത്തിയതാണ്. ഞങ്ങൾ കൃത്യതയ്ക്കായി ശ്രമിക്കുന്നുവെങ്കിലും, ഓട്ടോമേറ്റഡ് പരിഭാഷകളിൽ പിഴവുകൾ അല്ലെങ്കിൽ തെറ്റായ വിവരങ്ങൾ ഉണ്ടാകാൻ സാധ്യതയുണ്ട്. അതിന്റെ സ്വാഭാവിക ഭാഷയിലുള്ള അസൽ രേഖയാണ് പ്രാമാണികമായ ഉറവിടമായി പരിഗണിക്കേണ്ടത്. നിർണായകമായ വിവരങ്ങൾക്ക്, പ്രൊഫഷണൽ മനുഷ്യ പരിഭാഷ ശുപാർശ ചെയ്യുന്നു. ഈ പരിഭാഷ ഉപയോഗിച്ച് ഉണ്ടാകുന്ന തെറ്റിദ്ധാരണകൾ അല്ലെങ്കിൽ തെറ്റായ വ്യാഖ്യാനങ്ങൾക്കായി ഞങ്ങൾ ഉത്തരവാദികളല്ല.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->