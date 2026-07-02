# Pet Story Generator Tutorial för nybörjare

## Innehållsförteckning

- [Förutsättningar](#förutsättningar)
- [Förstå projektstrukturen](#förstå-projektstrukturen)
- [Kärnkomponenter förklarade](#kärnkomponenter-förklarade)
  - [1. Huvudapplikation](#1-huvudapplikation)
  - [2. Webbkontroller](#2-webbkontroller)
  - [3. Story Service](#3-story-service)
  - [4. Webbmallar](#4-webbmallar)
  - [5. Konfiguration](#5-konfiguration)
- [Köra applikationen](#köra-applikationen)
- [Hur allt fungerar tillsammans](#hur-allt-fungerar-tillsammans)
- [Förstå AI-integrationen](#förstå-ai-integrationen)
- [Nästa steg](#nästa-steg)

## Förutsättningar

Innan du börjar, se till att du har:
- Java 21 eller högre installerat
- Maven för beroendehantering
- En Azure AI Foundry-modellutplacering (provisionera med `azd up` — se [Kapitel 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), inloggad med `az login` (keyless auth)
- Grundläggande kunskaper i Java, Spring Boot och webbutveckling

## Förstå projektstrukturen

Pet story-projektet innehåller flera viktiga filer:

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

## Kärnkomponenter förklarade

### 1. Huvudapplikation

**Fil:** `PetStoryApplication.java`

Det här är ingångspunkten för vår Spring Boot-applikation:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Vad detta gör:**
- `@SpringBootApplication`-annoteringen möjliggör automatisk konfiguration och komponent-scanning
- Startar en inbäddad webbserver (Tomcat) på port 8080
- Skapar automatiskt alla nödvändiga Spring beans och tjänster

### 2. Webbkontroller

**Fil:** `PetController.java`

Denna hanterar alla webbförfrågningar och användarinteraktioner:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Returnerar index.html-mallen
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Inmatningsvalidering
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Rensa inmatning för säkerhet
        String sanitizedDescription = sanitizeInput(description);
        
        // Generera berättelse med felhantering
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Returnerar result.html-mallen
            
        } catch (Exception e) {
            // Använd reservberättelse om AI misslyckas
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Begränsa längd
    }
}
```

**Viktiga funktioner:**

1. **Routning**: `@GetMapping("/")` visar uppladdningsformuläret, `@PostMapping("/generate-story")` behandlar inskickningar
2. **Indatavalidering**: Kontrollerar tomma beskrivningar och längdbegränsningar
3. **Säkerhet**: Sanerar användarinput för att förhindra XSS-attacker
4. **Felhantering**: Erbjuder fallback-berättelser när AI-tjänsten misslyckas
5. **Model Binding**: Skickar data till HTML-mallar med hjälp av Spring's `Model`

**Fallback-system:**
Kontrollern inkluderar förskrivna berättelsemallar som används när AI-tjänsten inte är tillgänglig:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Använd beskrivningshash för konsekventa svar
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Story Service

**Fil:** `StoryService.java`

Denna tjänst kommunicerar med Azure AI Foundry för att generera berättelser med keyless autentisering:

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
        
        // Foundrys OpenAI-kompatibla slutpunkt finns under /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Autentisering utan nyckel med Microsoft Entra ID (ingen API-nyckel)
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
        
        // Konfigurera AI-förfrågan
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Begränsa svarslängden
                .temperature(0.8)          // Kontrollera kreativitet (0.0-1.0)
                .build();
        
        // Skicka förfrågan och få svar
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Viktiga komponenter:**

1. **OpenAI-klient**: Använder den officiella OpenAI Java SDK konfigurerad för Azure AI Foundry (keyless)
2. **Systemprompt**: Sätter AI:ns beteende för att skriva familjevänliga djurberättelser
3. **Användarprompt**: Ber AI:n att exakt skriva den berättelse som beskrivningen kräver
4. **Parametrar**: Styr berättelsens längd och kreativitet
5. **Felhantering**: Kastar undantag som kontrollern fångar och hanterar

### 4. Webbmallar

**Fil:** `index.html` (Uppladdningsformulär)

Huvudsidan där användare beskriver sina husdjur:

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

**Fil:** `result.html` (Visning av berättelse)

Visar den genererade berättelsen:

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

**Mallfunktioner:**

1. **Thymeleaf-integration**: Använder `th:`-attribut för dynamiskt innehåll
2. **Responsiv design**: CSS-styling för mobil och desktop
3. **Felhantering**: Visar valideringsfel för användare
4. **Klientsidebehandling**: JavaScript för bildanalys (med Transformers.js)

### 5. Konfiguration

**Fil:** `application.properties`

Konfigurationsinställningar för applikationen:

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

**Konfiguration förklarad:**

1. **Filuppladdning**: Tillåter bilder upp till 10MB
2. **Loggning**: Styr vilken information som loggas under körning
3. **Azure AI Foundry**: Specificerar endpoint och modellutplacering som ska användas (keyless auth)
4. **Säkerhet**: Felhanteringskonfiguration för att undvika exponering av känslig information

## Köra applikationen

### Steg 1: Logga in och sätt din endpoint

Autentisering är keyless (Microsoft Entra ID), så ingen API-nyckel behövs. Logga in och ställ in din Foundry-endpoint:

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

**Varför detta behövs:**
- Azure AI Foundry använder Microsoft Entra ID för att autentisera inferensförfrågningar
- Keyless-autentisering betyder inga hemligheter i din källkod eller miljö
- Ditt konto behöver rollen **Cognitive Services OpenAI User** på resursen

### Steg 2: Bygg och kör

Navigera till projektkatalogen:
```bash
cd 04-PracticalSamples/petstory
```

Bygg applikationen:
```bash
mvn clean compile
```

Starta servern:
```bash
mvn spring-boot:run
```

Applikationen startar på `http://localhost:8080`.

### Steg 3: Testa applikationen

1. **Öppna** `http://localhost:8080` i din webbläsare
2. **Beskriv** ditt husdjur i textfältet (t.ex. "En lekfull golden retriever som älskar att apportera")
3. **Klicka på** "Generate Story" för att få en AI-genererad berättelse
4. **Alternativt**, ladda upp en bild på ett husdjur för att automatiskt generera en beskrivning
5. **Se** den kreativa berättelsen baserat på din husdjursbeskrivning

## Hur allt fungerar tillsammans

Här är det fullständiga flödet när du genererar en djurberättelse:

1. **Användarinmatning**: Du beskriver ditt husdjur i webbformuläret
2. **Formulärinlämning**: Webbläsaren skickar en POST-förfrågan till `/generate-story`
3. **Kontrollerbearbetning**: `PetController` validerar och sanerar inmatningen
4. **AI-tjänstförfrågan**: `StoryService` skickar förfrågan till Azure AI Foundry-modellen
5. **Berättelsegenerering**: AI genererar en kreativ berättelse baserad på beskrivningen
6. **Svarshantering**: Kontroller tar emot berättelsen och lägger till den i modellen
7. **Mallrendering**: Thymeleaf renderar `result.html` med berättelsen
8. **Visning**: Användaren ser den genererade berättelsen i sin webbläsare

**Felhanteringsflöde:**
Om AI-tjänsten misslyckas:
1. Kontroller fångar undantaget
2. Genererar en fallback-berättelse med hjälp av förskrivna mallar
3. Visar fallback-berättelsen med en notis om AI:s otillgänglighet
4. Användaren får ändå en berättelse, vilket säkerställer en bra användarupplevelse

## Förstå AI-integrationen

### Azure AI Foundry (keyless)
Applikationen använder Azure AI Foundry med keyless autentisering (Microsoft Entra ID):

```java
// Nyckelfri autentisering - ingen API-nyckel
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Prompt Engineering
Tjänsten använder omsorgsfullt utformade prompts för att få bra resultat:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Svarshantering
AI-svaret extraheras och valideras:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Nästa steg

För fler exempel, se [Kapitel 04: Praktiska exempel](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Ansvarsfriskrivning**:
Detta dokument har översatts med hjälp av AI-översättningstjänsten [Co-op Translator](https://github.com/Azure/co-op-translator). Även om vi strävar efter noggrannhet, var vänlig notera att automatiska översättningar kan innehålla fel eller brister. Det ursprungliga dokumentet på dess modersmål bör betraktas som den auktoritativa källan. För kritisk information rekommenderas professionell mänsklig översättning. Vi ansvarar inte för några missförstånd eller feltolkningar som uppstår till följd av användningen av denna översättning.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->