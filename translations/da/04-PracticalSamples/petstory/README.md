# Pet Story Generator Tutorial for Begyndere

## Indholdsfortegnelse

- [Forudsætninger](#forudsætninger)
- [Forståelse af Projektstrukturen](#forståelse-af-projektstrukturen)
- [Hovedkomponenter Forklaret](#hovedkomponenter-forklaret)
  - [1. Hovedapplikation](#1-hovedapplikation)
  - [2. Web Controller](#2-web-controller)
  - [3. Story Service](#3-story-service)
  - [4. Web Skabeloner](#4-web-skabeloner)
  - [5. Konfiguration](#5-konfiguration)
- [Køre Applikationen](#køre-applikationen)
- [Hvordan Det Hele Fungerer Sammen](#hvordan-det-hele-fungerer-sammen)
- [Forståelse af AI-Integrationen](#forståelse-af-ai-integrationen)
- [Næste Skridt](#næste-skridt)

## Forudsætninger

Før du går i gang, skal du sikre dig, at du har:
- Java 21 eller nyere installeret
- Maven til afhængighedsstyring
- En Azure AI Foundry modeludrulning (opret den med `azd up` — se [Kapitel 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), logget ind med `az login` (nøglefri autentificering)
- Grundlæggende forståelse af Java, Spring Boot og webudvikling

## Forståelse af Projektstrukturen

Pet story-projektet har flere vigtige filer:

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

## Hovedkomponenter Forklaret

### 1. Hovedapplikation

**Fil:** `PetStoryApplication.java`

Dette er indgangspunktet for vores Spring Boot-applikation:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Hvad dette gør:**
- `@SpringBootApplication` annotationen aktiverer auto-konfiguration og komponent-scanning
- Starter en indlejret webserver (Tomcat) på port 8080
- Opretter automatisk alle nødvendige Spring beans og services

### 2. Web Controller

**Fil:** `PetController.java`

Denne håndterer alle webanmodninger og brugerinteraktioner:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Returnerer index.html skabelon
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Indtastningsvalidering
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Rens input for sikkerhed
        String sanitizedDescription = sanitizeInput(description);
        
        // Generer historie med fejlhåndtering
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Returnerer result.html skabelon
            
        } catch (Exception e) {
            // Brug fallback-historie hvis AI fejler
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Begræns længde
    }
}
```

**Nøglefunktioner:**

1. **Rutehåndtering**: `@GetMapping("/")` viser upload-formularen, `@PostMapping("/generate-story")` behandler indsendelser
2. **Inputvalidering**: Tjekker for tomme beskrivelser og længdebegrænsninger
3. **Sikkerhed**: Rensker brugerinput for at forhindre XSS-angreb
4. **Fejlhåndtering**: Tilbyder fallback-historier, når AI-service fejler
5. **Model Binding**: Overfører data til HTML-skabeloner ved hjælp af Springs `Model`

**Fallback System:**
Controlleren inkluderer forudskrevne historie-skabeloner, der bruges, når AI-servicen ikke er tilgængelig:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Brug beskrivelses-hash for konsistente svar
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Story Service

**Fil:** `StoryService.java`

Denne service kommunikerer med Azure AI Foundry for at generere historier med nøglefri autentificering:

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
        
        // Foundrys OpenAI-kompatible endpoint findes under /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Nøglefri autentificering med Microsoft Entra ID (ingen API-nøgle)
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
        
        // Konfigurer AI-forespørgslen
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Begræns svarlængde
                .temperature(0.8)          // Kontroller kreativitet (0,0-1,0)
                .build();
        
        // Send forespørgsel og modtag svar
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Nøglekomponenter:**

1. **OpenAI Client**: Bruger den officielle OpenAI Java SDK konfigureret til Azure AI Foundry (nøglefri)
2. **System Prompt**: Sætter AIs adfærd til at skrive familievenlige dyrehistorier
3. **User Prompt**: Fortæller AI præcis hvilken historie der skal skrives baseret på beskrivelsen
4. **Parametre**: Kontrollerer historiens længde og kreativitet
5. **Fejlhåndtering**: Smider undtagelser, som controlleren fanger og håndterer

### 4. Web Skabeloner

**Fil:** `index.html` (Upload Formular)

Hovedsiden, hvor brugerne beskriver deres kæledyr:

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

**Fil:** `result.html` (Historievisning)

Viser den genererede historie:

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

**Skabelonfunktioner:**

1. **Thymeleaf Integration**: Bruger `th:` attributter til dynamisk indhold
2. **Responsivt Design**: CSS-styling til mobil og desktop
3. **Fejlhåndtering**: Viser valideringsfejl til brugerne
4. **Klientsidebehandling**: JavaScript til billedanalyse (ved brug af Transformers.js)

### 5. Konfiguration

**Fil:** `application.properties`

Konfigurationsindstillinger til applikationen:

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

**Konfiguration forklaret:**

1. **Fil Upload**: Tillader billeder op til 10MB
2. **Logging**: Kontrollerer hvilken information der logges under kørsel
3. **Azure AI Foundry**: Angiver endpoint og modeludrulning der skal bruges (nøglefri autentificering)
4. **Sikkerhed**: Fejlhåndteringskonfiguration for at undgå at afsløre følsomme oplysninger

## Køre Applikationen

### Trin 1: Log Ind og Sæt Dit Endpoint

Autentificering er nøglefri (Microsoft Entra ID), så der er ingen API-nøgle. Log ind og angiv dit Foundry-endpoint:

**Windows (Kommandoprompt):**
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

**Hvorfor dette er nødvendigt:**
- Azure AI Foundry bruger Microsoft Entra ID til at autentificere inferensanmodninger
- Nøglefri autentificering betyder ingen hemmeligheder i din kildekode eller miljø
- Din konto skal have rollen **Cognitive Services OpenAI User** på ressourcen

### Trin 2: Byg og Kør

Skift til projektmappen:
```bash
cd 04-PracticalSamples/petstory
```

Byg applikationen:
```bash
mvn clean compile
```

Start serveren:
```bash
mvn spring-boot:run
```

Applikationen starter på `http://localhost:8080`.

### Trin 3: Test Applikationen

1. **Åbn** `http://localhost:8080` i din browser
2. **Beskriv** dit kæledyr i tekstfeltet (f.eks. "En legesyg golden retriever, der elsker at hente ting")
3. **Klik** på "Generate Story" for at få en AI-genereret historie
4. **Alternativt**, upload et kæledyrsbillede for automatisk at generere en beskrivelse
5. **Se** den kreative historie baseret på din kæledyrsbeskrivelse

## Hvordan Det Hele Fungerer Sammen

Her er den komplette proces, når du genererer en kæledyrshistorie:

1. **Brugerinput**: Du beskriver dit kæledyr i webformularen
2. **Formularindsendelse**: Browseren sender en POST-anmodning til `/generate-story`
3. **Controllerbehandling**: `PetController` validerer og renser input
4. **AI Servicekald**: `StoryService` sender anmodning til Azure AI Foundry modellen
5. **Historiegenerering**: AI genererer en kreativ historie baseret på beskrivelsen
6. **Svarhåndtering**: Controller modtager historien og tilføjer den til modellen
7. **Skabelonrendering**: Thymeleaf render `result.html` med historien
8. **Visning**: Brugeren ser den genererede historie i browseren

**Fejlhåndteringsflow:**
Hvis AI-servicen fejler:
1. Controlleren fanger undtagelsen
2. Genererer en fallback-historie ved hjælp af forudskrevne skabeloner
3. Viser fallback-historien med en note om AI-ustabilitet
4. Brugeren får stadig en historie, hvilket sikrer en god brugeroplevelse

## Forståelse af AI-Integrationen

### Azure AI Foundry (nøglefri)
Applikationen bruger Azure AI Foundry med nøglefri autentificering (Microsoft Entra ID):

```java
// Nøglefri autentificering - ingen API-nøgle
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Prompt Engineering
Servicen bruger nøje udformede prompts til at opnå gode resultater:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Svarbehandling
AI-svaret trækkes ud og valideres:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Næste Skridt

For flere eksempler, se [Kapitel 04: Praktiske eksempler](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Ansvarsfraskrivelse**:
Dette dokument er blevet oversat ved hjælp af AI-oversættelsestjenesten [Co-op Translator](https://github.com/Azure/co-op-translator). Selvom vi bestræber os på nøjagtighed, skal du være opmærksom på, at automatiserede oversættelser kan indeholde fejl eller unøjagtigheder. Det originale dokument på dets oprindelige sprog bør betragtes som den autoritative kilde. For kritisk information anbefales professionel menneskelig oversættelse. Vi påtager os intet ansvar for misforståelser eller fejltolkninger, der opstår som følge af brugen af denne oversættelse.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->