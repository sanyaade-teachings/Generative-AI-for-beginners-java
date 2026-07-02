# Mafunzo ya Mkuzaji wa Hadithi za Wanyama wa Kifugo kwa Waanzilishi

## Jedwali la Yaliyomo

- [Mambo Yanayohitajika](#mambo-yanayohitajika)
- [Kuelewa Muundo wa Mradi](#kuelewa-muundo-wa-mradi)
- [Vipengele Vikuu Vimefafanuliwa](#vipengele-vikuu-vimefafanuliwa)
  - [1. Programu Kuu](#1-programu-kuu)
  - [2. Kidhibiti Mtandao](#2-kidhibiti-mtandao)
  - [3. Huduma ya Hadithi](#3-huduma-ya-hadithi)
  - [4. Miongozo ya Mtandao](#4-miongozo-ya-mtandao)
  - [5. Usanidi](#5-usanidi)
- [Kukimbia Programu](#kukimbia-programu)
- [Jinsi Kila Kipengele Kinavyofanya Kazi Pamoja](#jinsi-kila-kipengele-kinavyofanya-kazi-pamoja)
- [Kuelewa Uingizaji wa AI](#kuelewa-uingizaji-wa-ai)
- [Hatua Zinazo Fuata](#hatua-zinazo-fuata)

## Mambo Yanayohitajika

Kabla ya kuanza, hakikisha una:
- Java 21 au juu zaidi imewekwa
- Maven kwa usimamizi wa utegemezi
- Utekelezaji wa mfano wa Azure AI Foundry (uishe na `azd up` — ona [Sura 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), umeingia kwa kutumia `az login` (uthibitishaji bila funguo)
- Uelewa wa msingi wa Java, Spring Boot, na maendeleo ya mtandao

## Kuelewa Muundo wa Mradi

Mradi wa hadithi za wanyama wa kufugo una faili kadhaa muhimu:

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

## Vipengele Vikuu Vimefafanuliwa

### 1. Programu Kuu

**Faili:** `PetStoryApplication.java`

Hii ni njia ya kuingia kwa programu yetu ya Spring Boot:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Hii hufanya nini:**
- Kielezi cha `@SpringBootApplication` kinawezesha usanidi wa moja kwa moja na skanning ya vipengele
- Inaendesha seva ya mtandao iliyojumuishwa (Tomcat) kwenye bandari 8080
- Huunda viumbe vyote vya Spring na huduma moja kwa moja

### 2. Kidhibiti Mtandao

**Faili:** `PetController.java`

Hii inashughulikia maombi yote ya mtandao na mwingiliano wa mtumiaji:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Hurejesha kiolezo cha index.html
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Uthibitishaji wa ingizo
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Safisha ingizo kwa usalama
        String sanitizedDescription = sanitizeInput(description);
        
        // Tengeneza hadithi na kushughulikia makosa
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Hurejesha kiolezo cha result.html
            
        } catch (Exception e) {
            // Tumia hadithi mbadala ikiwa AI itashindwa
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Kizuizi cha urefu
    }
}
```

**Vipengele muhimu:**

1. **Shughulikia Njia**: `@GetMapping("/")` inaonyesha fomu ya kupakia, `@PostMapping("/generate-story")` inashughulikia maombi
2. **Uhakiki wa Ingizo**: Hukagua maelezo yaliyo tupu na mpaka wa urefu
3. **Usalama**: Hufuta vipengele hatari katika ingizo la mtumiaji kuzuia mashambulizi ya XSS
4. **Udhibiti wa Makosa**: Hutoa hadithi mbadala wakati huduma ya AI inashindwa
5. **Ufungaji wa Mfano**: Hupitisha data kwa templates za HTML kwa kutumia `Model` ya Spring

**Mfumo wa Kurejesha:**
Kidhibiti kina templates za hadithi zilizotangulia ambazo hutumika wakati huduma ya AI haipo:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Tumia kryptografia ya maelezo kwa majibu thabiti
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Huduma ya Hadithi

**Faili:** `StoryService.java`

Huduma hii huwasiliana na Azure AI Foundry kuunda hadithi kwa kutumia uthibitishaji bila funguo:

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
        
        // Kituo cha OpenAI kinacholingana cha Foundry kiko chini ya /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Uthibitishaji bila ufunguo kwa Microsoft Entra ID (hakuna ufunguo wa API)
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
        
        // Sanidi ombi la AI
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Punguza urefu wa jibu
                .temperature(0.8)          // Dhibiti ubunifu (0.0-1.0)
                .build();
        
        // Tuma ombi na upate jibu
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Vipengele vikuu:**

1. **Mteja wa OpenAI**: Inatumia SDK rasmi ya OpenAI ya Java imewekwa kwa Azure AI Foundry (bila funguo)
2. **Amri ya Mfumo**: Hua na tabia ya AI kuandika hadithi za wanyama wa kufugo zinazofaa familia
3. **Amri ya Mtumiaji**: Hueleza AI hadithi gani kuandika kulingana na maelezo
4. **Vigezo**: Hukagua urefu na kiwango cha ubunifu wa hadithi
5. **Udhibiti wa Makosa**: Hutupa makosa ambayo kidhibiti hupokea na kushughulikia

### 4. Miongozo ya Mtandao

**Faili:** `index.html` (Fomu ya Kupakia)

Ukurasa mkuu ambapo watumiaji huelezea wanyama wao:

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

**Faili:** `result.html` (Onyesho la Hadithi)

Inaonyesha hadithi iliyozalishwa:

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

**Vipengele vya Miongozo:**

1. **Uunganisho wa Thymeleaf**: Inatumia sifa za `th:` kwa maudhui yanayobadilika
2. **Muundo Unaojibua**: Uwekaji mtindo wa CSS kwa simu na mezani
3. **Udhibiti wa Makosa**: Inaonyesha makosa ya uhakiki kwa watumiaji
4. **Usindikaji Kando ya Mteja**: JavaScript kwa uchambuzi wa picha (kutumia Transformers.js)

### 5. Usanidi

**Faili:** `application.properties`

Mipangilio ya usanidi kwa programu:

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

**Maelezo ya usanidi:**

1. **Kupakia Faili**: Ruhusu picha hadi ukubwa wa 10MB
2. **Ufuatiliaji**: Hukagua taarifa gani zinaandikwa wakati wa utekelezaji
3. **Azure AI Foundry**: Inaeleza kiungo na utekelezaji wa mfano (uthibitishaji bila funguo)
4. **Usalama**: Usanidi wa udhibiti wa makosa ili kuepuka kufichua maelezo muhimu

## Kukimbia Programu

### Hatua ya 1: Ingia na Weka Kiungo Chako

Uthibitishaji ni wa bila funguo (Microsoft Entra ID), hivyo haina funguo ya API. Ingia na weka kiungo cha Foundry:

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

**Kwa nini hii inahitajika:**
- Azure AI Foundry hutumia Microsoft Entra ID kuidhinisha maombi ya inference
- Uthibitishaji bila funguo hamaanishi hakuna siri katika msimbo wako au mazingira
- Akaunti yako inahitaji nafasi ya **Cognitive Services OpenAI User** kwenye rasilimali

### Hatua ya 2: Jenga na Kimbia

Nenda kwenye saraka ya mradi:
```bash
cd 04-PracticalSamples/petstory
```

Jenga programu:
```bash
mvn clean compile
```

Anzisha seva:
```bash
mvn spring-boot:run
```

Programu itaanzishwa kwenye `http://localhost:8080`.

### Hatua ya 3: Jaribu Programu

1. **Fungua** `http://localhost:8080` kwenye kivinjari chako
2. **Eleza** mnyama wako kwenye sehemu ya maandishi (mfano, "Mbwa wa rangi ya dhahabu mwenye kucheza sana anayeipenda kubeba vitu")
3. **Bofya** "Generate Story" kupata hadithi iliyotengenezwa na AI
4. **Mbali na hilo**, pakia picha ya mnyama kupata maelezo moja kwa moja
5. **Tazama** hadithi za ubunifu kulingana na maelezo ya mnyama wako

## Jinsi Kila Kipengele Kinavyofanya Kazi Pamoja

Hapa ni mtiririko kamili unapotengeneza hadithi za wanyama wa kufugo:

1. **Ingizo la Mtumiaji**: Unamwelezea mnyama wako kwenye fomu ya mtandao
2. **Uwasilishaji wa Fomu**: Kivinjari kinatuma ombi la POST kwa `/generate-story`
3. **Usindikaji wa Kidhibiti**: `PetController` huhakiki na kusafisha ingizo
4. **Mawasiliano na Huduma ya AI**: `StoryService` hutuma ombi kwa mfano wa Azure AI Foundry
5. **Uundaji wa Hadithi**: AI huunda hadithi ya ubunifu kulingana na maelezo
6. **Udhibiti wa Majibu**: Kidhibiti hupokea hadithi na kuiweka kwenye mfano
7. **Uwasilishaji wa Miongozo**: Thymeleaf huonyesha `result.html` na hadithi
8. **Onyesho**: Mtumiaji anaona hadithi iliyozalishwa kwenye kivinjari

**Mtiririko wa Udhibiti wa Makosa:**
Ikiwa huduma ya AI inashindwa:
1. Kidhibiti hukamata kosa
2. Hutengeneza hadithi mbadala kwa kutumia templates zilizotanguliwa
3. Inaonyesha hadithi mbadala na ujumbe kuhusu upatikanaji wa AI
4. Mtumiaji bado anapata hadithi, kuhakikisha uzoefu mzuri wa mtumiaji

## Kuelewa Uingizaji wa AI

### Azure AI Foundry (bila funguo)
Programu hii inatumia Azure AI Foundry kwa uthibitishaji bila funguo (Microsoft Entra ID):

```java
// Uthibitisho bila ufunguo - hakuna ufunguo wa API
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Uhandisi wa Amri (Prompt Engineering)
Huduma hutumia maelekezo kwa makini kupata matokeo mazuri:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Usindikaji wa Majibu
Jibu la AI huchujwa na kuhakikiwa:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Hatua Zinazo Fuata

Kwa mifano zaidi, angalia [Sura 04: Sampuli za vitendo](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Kionyozo**:
Hati hii imetafsiriwa kwa kutumia huduma ya tafsiri ya AI [Co-op Translator](https://github.com/Azure/co-op-translator). Ingawa tunajitahidi kupata usahihi, tafadhali fahamu kwamba tafsiri za kiotomatiki zinaweza kuwa na makosa au upungufu wa usahihi. Hati ya asili katika lugha yake halisi inapaswa kuchukuliwa kama chanzo cha mamlaka. Kwa taarifa muhimu, tafsiri ya kitaalamu inayofanywa na binadamu inapendekezwa. Hatutojibu kwa kuelewa vibaya au tafsiri potofu zinazotokea kutokana na matumizi ya tafsiri hii.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->