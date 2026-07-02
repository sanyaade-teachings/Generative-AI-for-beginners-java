# Pet Story Generator Tutorial para sa mga Baguhan

## Table of Contents

- [Mga Kinakailangan](#mga-kinakailangan)
- [Pag-unawa sa Istruktura ng Proyekto](#pag-unawa-sa-istruktura-ng-proyekto)
- [Paliwanag ng Mga Pangunahing Bahagi](#paliwanag-ng-mga-pangunahing-bahagi)
  - [1. Pangunahing Aplikasyon](#1-pangunahing-aplikasyon)
  - [2. Web Controller](#2-web-controller)
  - [3. Story Service](#3-story-service)
  - [4. Mga Web Template](#4-mga-web-template)
  - [5. Pag-configure](#5-pag-configure)
- [Pagpapatakbo ng Aplikasyon](#pagpapatakbo-ng-aplikasyon)
- [Paano Ito Nagtutulungan](#paano-ito-nagtutulungan)
- [Pag-unawa sa AI Integration](#pag-unawa-sa-ai-integration)
- [Mga Susunod na Hakbang](#mga-susunod-na-hakbang)

## Mga Kinakailangan

Bago magsimula, siguraduhing mayroon ka ng:
- Java 21 o mas mataas na naka-install
- Maven para sa pamamahala ng dependency
- Isang Azure AI Foundry model deployment (i-provision gamit ang `azd up` — tingnan ang [Kabanata 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), naka-sign in gamit ang `az login` (keyless auth)
- Pangunahing kaalaman sa Java, Spring Boot, at web development

## Pag-unawa sa Istruktura ng Proyekto

Ang pet story project ay may ilang mahahalagang files:

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

## Paliwanag ng Mga Pangunahing Bahagi

### 1. Pangunahing Aplikasyon

**File:** `PetStoryApplication.java`

Ito ang entry point para sa ating Spring Boot application:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Ano ang ginagawa nito:**
- Ang `@SpringBootApplication` annotation ay nagpapagana ng auto-configuration at component scanning
- Nagpapasimula ng embedded web server (Tomcat) sa port 8080
- Awtomatikong lumilikha ng lahat ng kinakailangang Spring beans at services

### 2. Web Controller

**File:** `PetController.java`

Ito ang humahawak sa lahat ng web requests at user interactions:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Nagbabalik ng index.html na template
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Pag-validate ng input
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Linisin ang input para sa seguridad
        String sanitizedDescription = sanitizeInput(description);
        
        // Bumuo ng kuwento na may paghawak ng error
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Nagbabalik ng result.html na template
            
        } catch (Exception e) {
            // Gumamit ng fallback na kuwento kung pumalya ang AI
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Limitahan ang haba
    }
}
```

**Pangunahing tampok:**

1. **Route Handling**: `@GetMapping("/")` ay nagpapakita ng upload form, `@PostMapping("/generate-story")` ay nagpoproseso ng mga submission
2. **Input Validation**: Sinusuri kung walang laman ang descriptions at mga limitasyon sa haba
3. **Security**: Nililinis ang user input upang maiwasan ang XSS attacks
4. **Error Handling**: Nagbibigay ng fallback stories kapag nabigo ang AI service
5. **Model Binding**: Pinapasa ang data sa mga HTML template gamit ang Spring's `Model`

**Fallback System:**
Kasama sa controller ang mga pre-written na story templates na ginagamit kapag hindi available ang AI service:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Gamitin ang hash ng paglalarawan para sa mga pare-parehong tugon
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Story Service

**File:** `StoryService.java`

Ang serbisyo na ito ay nakikipag-ugnayan sa Azure AI Foundry para bumuo ng mga kwento gamit ang keyless authentication:

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
        
        // Ang OpenAI-compatible na endpoint ng Foundry ay matatagpuan sa ilalim ng /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Keyless authentication gamit ang Microsoft Entra ID (walang API key)
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
        
        // I-configure ang AI request
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Limitahan ang haba ng sagot
                .temperature(0.8)          // Kontrolin ang pagiging malikhain (0.0-1.0)
                .build();
        
        // Ipadala ang request at kunin ang tugon
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Pangunahing bahagi:**

1. **OpenAI Client**: Ginagamit ang opisyal na OpenAI Java SDK na naka-configure para sa Azure AI Foundry (keyless)
2. **System Prompt**: Itinakda ang pag-uugali ng AI upang magsulat ng mga kwentong pang-pamilya tungkol sa mga alagang hayop
3. **User Prompt**: Sinasabi sa AI kung anong kwento ang isusulat base sa description
4. **Parameters**: Kinokontrol ang haba ng kwento at antas ng pagkamalikhain
5. **Error Handling**: Nagbubuga ng mga exceptions na hinahawakan at pinamamahalaan ng controller

### 4. Mga Web Template

**File:** `index.html` (Upload Form)

Pangunahing pahina kung saan inilalarawan ng mga user ang kanilang mga alaga:

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

**File:** `result.html` (Pagpapakita ng Kwento)

Ipinapakita ang na-generate na kwento:

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

**Mga tampok ng template:**

1. **Thymeleaf Integration**: Gumagamit ng `th:` na attributes para sa dynamic na nilalaman
2. **Responsive Design**: CSS styling para sa mobile at desktop
3. **Error Handling**: Ipinapakita ang mga validation errors sa mga user
4. **Client-side Processing**: JavaScript para sa pagsusuri ng imahe (gamit ang Transformers.js)

### 5. Pag-configure

**File:** `application.properties`

Mga settings para sa aplikasyon:

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

**Paliwanag ng configuration:**

1. **File Upload**: Pinapayagan ang mga imahe hanggang 10MB
2. **Logging**: Kinokontrol kung anong impormasyon ang nilalagay sa log habang tumatakbo
3. **Azure AI Foundry**: Tinukoy ang endpoint at model deployment na gagamitin (keyless auth)
4. **Security**: Pag-configure ng error handling upang hindi mailantad ang sensitibong impormasyon

## Pagpapatakbo ng Aplikasyon

### Hakbang 1: Mag-sign In at I-set ang Iyong Endpoint

Ang authentication ay keyless (Microsoft Entra ID), kaya walang API key. Mag-sign in at i-set ang iyong Foundry endpoint:

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

**Bakit ito kailangan:**
- Ginagamit ng Azure AI Foundry ang Microsoft Entra ID para i-authenticate ang mga inference requests
- Ibig sabihin ng keyless auth na walang mga secret sa iyong source code o environment
- Kailangan ng iyong account ang **Cognitive Services OpenAI User** role sa resource

### Hakbang 2: I-build at Patakbuhin

Pumunta sa direktoryo ng proyekto:
```bash
cd 04-PracticalSamples/petstory
```

I-build ang aplikasyon:
```bash
mvn clean compile
```

Simulan ang server:
```bash
mvn spring-boot:run
```

Mag-uumpisa ang aplikasyon sa `http://localhost:8080`.

### Hakbang 3: Subukan ang Aplikasyon

1. **Buksan** ang `http://localhost:8080` sa iyong browser
2. **Ilarawan** ang iyong alaga sa text area (halimbawa, "Isang masiglang golden retriever na mahilig mag-fetch")
3. **I-click** ang "Generate Story" para makakuha ng AI-generated na kwento
4. **O kaya naman**, mag-upload ng larawan ng alaga upang awtomatikong makabuo ng description
5. **Tingnan** ang malikhaing kwento base sa iyong paglalarawan ng alaga

## Paano Ito Nagtutulungan

Narito ang buong daloy kapag nag-generate ka ng kwento ng alaga:

1. **User Input**: Ikinukwento mo ang iyong alaga sa web form
2. **Form Submission**: Nagpapadala ang browser ng POST request sa `/generate-story`
3. **Controller Processing**: Tine-validate at nililinis ng `PetController` ang input
4. **AI Service Call**: Nagpapadala ang `StoryService` ng request sa Azure AI Foundry model
5. **Story Generation**: Gumagawa ang AI ng malikhaing kwento base sa paglalarawan
6. **Response Handling**: Tinatanggap ng controller ang kwento at idinadagdag ito sa model
7. **Template Rendering**: Ipinapakita ng Thymeleaf ang `result.html` na may kwento
8. **Display**: Nakikita ng user ang na-generate na kwento sa kanilang browser

**Daloy ng Pag-handle ng Error:**
Kung mabigo ang AI service:
1. Nahuhuli ng controller ang exception
2. Gumagawa ng fallback story gamit ang mga pre-written na template
3. Ipinapakita ang fallback story kasama ang paalala tungkol sa hindi availability ng AI
4. Nakakakuha pa rin ang user ng kwento, na nagpapanatili ng magandang karanasan

## Pag-unawa sa AI Integration

### Azure AI Foundry (keyless)
Gumagamit ang aplikasyon ng Azure AI Foundry gamit ang keyless authentication (Microsoft Entra ID):

```java
// Walang susi na pagpapatunay - walang API key
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Prompt Engineering
Maingat na ginawa ang mga prompt ng serbisyo upang makamit ang magagandang resulta:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Response Processing
Ina-extract at tines-triktura ang sagot ng AI:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Mga Susunod na Hakbang

Para sa higit pang mga halimbawa, tingnan ang [Kabanata 04: Praktikal na mga halimbawa](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Pagtatanggi**:
Ang dokumentong ito ay isinalin gamit ang serbisyo ng AI translation na [Co-op Translator](https://github.com/Azure/co-op-translator). Bagama't nagsusumikap kami para sa katumpakan, pakatandaan na ang awtomatikong pagsasalin ay maaaring maglaman ng mga pagkakamali o hindi pagkakatugma. Ang orihinal na dokumento sa orihinal nitong wika ang dapat ituring na pangunahing sanggunian. Para sa mahahalagang impormasyon, inirerekomenda ang propesyonal na pagsasalin ng tao. Hindi kami mananagot sa anumang maling pagkakaintindi o maling interpretasyon na nagmula sa paggamit ng pagsasaling ito.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->