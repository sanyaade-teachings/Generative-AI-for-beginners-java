# Handleiding Pet Story Generator voor Beginners

## Inhoudsopgave

- [Vereisten](#vereisten)
- [Het Projectstructuur Begrijpen](#het-projectstructuur-begrijpen)
- [Uitleg van de Kernonderdelen](#uitleg-van-de-kernonderdelen)
  - [1. Hoofd Applicatie](#1-hoofd-applicatie)
  - [2. Webcontroller](#2-webcontroller)
  - [3. Story Service](#3-story-service)
  - [4. Web Templates](#4-web-templates)
  - [5. Configuratie](#5-configuratie)
- [De Applicatie Uitvoeren](#de-applicatie-uitvoeren)
- [Hoe Het Alles Samenwerkt](#hoe-het-alles-samenwerkt)
- [De AI-integratie Begrijpen](#de-ai-integratie-begrijpen)
- [Volgende Stappen](#volgende-stappen)

## Vereisten

Voordat je begint, zorg ervoor dat je:
- Java 21 of hoger geïnstalleerd hebt
- Maven voor dependency management
- Een Azure AI Foundry model deployment (zet dit op met `azd up` — zie [Hoofdstuk 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), aangemeld met `az login` (keyless authenticatie)
- Basiskennis van Java, Spring Boot en webontwikkeling

## Het Projectstructuur Begrijpen

Het pet story project bevat meerdere belangrijke bestanden:

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

## Uitleg van de Kernonderdelen

### 1. Hoofd Applicatie

**Bestand:** `PetStoryApplication.java`

Dit is het startpunt van onze Spring Boot applicatie:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Dit doet het:**
- `@SpringBootApplication` annotatie maakt auto-configuratie en component scanning mogelijk
- Start een ingebouwde webserver (Tomcat) op poort 8080
- Maakt automatisch alle benodigde Spring beans en services aan

### 2. Webcontroller

**Bestand:** `PetController.java`

Dit behandelt alle webverzoeken en gebruikersinteracties:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Geeft index.html template terug
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Invoer validatie
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Maak invoer veilig voor beveiliging
        String sanitizedDescription = sanitizeInput(description);
        
        // Genereer verhaal met foutafhandeling
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Geeft result.html template terug
            
        } catch (Exception e) {
            // Gebruik fallback verhaal als AI faalt
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Beperk lengte
    }
}
```

**Belangrijkste kenmerken:**

1. **Route-afhandeling**: `@GetMapping("/")` toont het uploadformulier, `@PostMapping("/generate-story")` verwerkt inzendingen
2. **Inputvalidatie**: Controleert op lege beschrijvingen en lengtebeperkingen
3. **Beveiliging**: Sanitizeert gebruikersinvoer om XSS-aanvallen te voorkomen
4. **Foutafhandeling**: Biedt fallback-verhalen aan wanneer AI-service faalt
5. **Model Binding**: Geeft data door aan HTML-templates via Spring's `Model`

**Fallback Systeem:**
De controller bevat vooraf geschreven verhaaltemplates die gebruikt worden wanneer de AI-service niet beschikbaar is:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Gebruik description hash voor consistente reacties
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Story Service

**Bestand:** `StoryService.java`

Deze service communiceert met Azure AI Foundry om verhalen te genereren met keyless authenticatie:

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
        
        // De OpenAI-compatibele endpoint van Foundry bevindt zich onder /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Sleutelloze authenticatie met Microsoft Entra ID (geen API-sleutel)
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
        
        // Configureer het AI-verzoek
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Beperk de responslengte
                .temperature(0.8)          // Beheer creativiteit (0.0-1.0)
                .build();
        
        // Verstuur verzoek en ontvang antwoord
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Belangrijke onderdelen:**

1. **OpenAI Client**: Gebruikt de officiële OpenAI Java SDK geconfigureerd voor Azure AI Foundry (keyless)
2. **Systeem Prompt**: Stelt het gedrag van de AI in om gezinsvriendelijke dierenverhalen te schrijven
3. **Gebruikersprompt**: Geeft de AI precies aan welk verhaal ze moeten schrijven op basis van de beschrijving
4. **Parameters**: Beheert de verhaal lengte en creativiteitsniveau
5. **Foutafhandeling**: Gooit excepties die de controller opvangt en afhandelt

### 4. Web Templates

**Bestand:** `index.html` (Uploadformulier)

De hoofdpagina waar gebruikers hun huisdieren beschrijven:

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

**Bestand:** `result.html` (Verhaalweergave)

Toont het gegenereerde verhaal:

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

**Kenmerken van de templates:**

1. **Thymeleaf Integratie**: Gebruikt `th:` attributen voor dynamische inhoud
2. **Responsief Ontwerp**: CSS-styling voor mobiel en desktop
3. **Foutafhandeling**: Toont validatiefouten aan gebruikers
4. **Client-side Verwerking**: JavaScript voor beeldanalyse (met Transformers.js)

### 5. Configuratie

**Bestand:** `application.properties`

Configuratie-instellingen voor de applicatie:

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

**Toelichting op de configuratie:**

1. **Bestand Upload**: Staat afbeeldingen toe tot 10MB
2. **Logging**: Bepaalt welke informatie wordt gelogd tijdens uitvoering
3. **Azure AI Foundry**: Specificeert de endpoint en model deployment (keyless authenticatie)
4. **Beveiliging**: Foutafhandelingsconfiguratie om blootstelling van gevoelige informatie te voorkomen

## De Applicatie Uitvoeren

### Stap 1: Aanmelden en Stel Jouw Endpoint In

Authenticatie is keyless (Microsoft Entra ID), dus er is geen API-sleutel. Meld je aan en stel de Foundry endpoint in:

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

**Waarom dit nodig is:**
- Azure AI Foundry gebruikt Microsoft Entra ID voor authenticatie van inference-verzoeken
- Keyless authenticatie betekent geen geheimen in je broncode of omgeving
- Je account moet de rol **Cognitive Services OpenAI User** op de resource hebben

### Stap 2: Bouwen en Uitvoeren

Ga naar de projectmap:
```bash
cd 04-PracticalSamples/petstory
```

Bouw de applicatie:
```bash
mvn clean compile
```

Start de server:
```bash
mvn spring-boot:run
```

De applicatie start op `http://localhost:8080`.

### Stap 3: Test de Applicatie

1. **Open** `http://localhost:8080` in je browser
2. **Beschrijf** je huisdier in het tekstvak (bijv. "Een speelse golden retriever die graag apporteren speelt")
3. **Klik** op "Generate Story" om een AI-gegenereerd verhaal te krijgen
4. **Of**, upload een afbeelding van een huisdier om automatisch een beschrijving te genereren
5. **Bekijk** het creatieve verhaal gebaseerd op de beschrijving van je huisdier

## Hoe Het Alles Samenwerkt

Dit is het volledige proces wanneer je een huisdierenverhaal genereert:

1. **Gebruikersinput**: Je beschrijft je huisdier op het webformulier
2. **Formulierverzending**: Browser stuurt POST-verzoek naar `/generate-story`
3. **Controllerverwerking**: `PetController` valideert en sanitizeert de invoer
4. **AI Service Aanroep**: `StoryService` stuurt verzoek naar het Azure AI Foundry model
5. **Verhaalgeneratie**: AI genereert een creatief verhaal op basis van de beschrijving
6. **Responsafhandeling**: Controller ontvangt het verhaal en voegt het toe aan het model
7. **Template Rendering**: Thymeleaf rendert `result.html` met het verhaal
8. **Weergave**: Gebruiker ziet het gegenereerde verhaal in de browser

**Foutafhandelingsstroom:**
Als de AI-service faalt:
1. Controller vangt de exceptie op
2. Genereert een fallback-verhaal met vooraf geschreven templates
3. Toont het fallback-verhaal met een melding over AI niet beschikbaar
4. Gebruiker krijgt nog steeds een verhaal, voor een goede gebruikerservaring

## De AI-integratie Begrijpen

### Azure AI Foundry (keyless)
De applicatie maakt gebruik van Azure AI Foundry met keyless authenticatie (Microsoft Entra ID):

```java
// Sleutelvrije authenticatie - geen API-sleutel
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Prompt Engineering
De service gebruikt zorgvuldig samengestelde prompts om goede resultaten te verkrijgen:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Response Processing
De AI-respons wordt geëxtraheerd en gevalideerd:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Volgende Stappen

Voor meer voorbeelden, zie [Hoofdstuk 04: Praktische voorbeelden](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Disclaimer**:
Dit document is vertaald met behulp van de AI vertaaldienst [Co-op Translator](https://github.com/Azure/co-op-translator). Hoewel we streven naar nauwkeurigheid, dient u er rekening mee te houden dat geautomatiseerde vertalingen fouten of onnauwkeurigheden kunnen bevatten. Het originele document in de oorspronkelijke taal moet worden beschouwd als de gezaghebbende bron. Voor kritieke informatie wordt professionele menselijke vertaling aanbevolen. Wij zijn niet aansprakelijk voor eventuele misverstanden of verkeerde interpretaties die voortvloeien uit het gebruik van deze vertaling.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->