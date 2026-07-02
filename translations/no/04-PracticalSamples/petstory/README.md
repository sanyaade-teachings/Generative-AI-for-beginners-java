# Veilednings for nybegynnere i Pet Story Generator

## Innholdsfortegnelse

- [Forutsetninger](#forutsetninger)
- [Forstå prosjektstrukturen](#forstå-prosjektstrukturen)
- [Kjernekomponenter forklart](#kjernekomponenter-forklart)
  - [1. Hovedapplikasjon](#1-hovedapplikasjon)
  - [2. Web-kontroller](#2-web-kontroller)
  - [3. Historietjeneste](#3-historietjeneste)
  - [4. Webmaler](#4-webmaler)
  - [5. Konfigurasjon](#5-konfigurasjon)
- [Kjøre applikasjonen](#kjøre-applikasjonen)
- [Hvordan det hele fungerer sammen](#hvordan-det-hele-fungerer-sammen)
- [Forstå AI-integrasjonen](#forstå-ai-integrasjonen)
- [Neste steg](#neste-steg)

## Forutsetninger

Før du begynner, sørg for at du har:
- Java 21 eller nyere installert
- Maven for avhengighetsstyring
- En Azure AI Foundry-modellutplassering (provisjoner den med `azd up` — se [Kapittel 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), logget inn med `az login` (nøkkelfri autentisering)
- Grunnleggende forståelse av Java, Spring Boot og webutvikling

## Forstå prosjektstrukturen

Pet story-prosjektet har flere viktige filer:

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

## Kjernekomponenter forklart

### 1. Hovedapplikasjon

**Fil:** `PetStoryApplication.java`

Dette er inngangspunktet for vår Spring Boot-applikasjon:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Hva dette gjør:**
- `@SpringBootApplication`-annotasjonen aktiverer automatisk konfigurasjon og komponent-skanning
- Starter en innebygd webserver (Tomcat) på port 8080
- Oppretter alle nødvendige Spring beans og tjenester automatisk

### 2. Web-kontroller

**Fil:** `PetController.java`

Denne håndterer alle webforespørsler og brukerinteraksjoner:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Returnerer index.html-mal
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Inndata validering
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Rens inndata for sikkerhet
        String sanitizedDescription = sanitizeInput(description);
        
        // Generer historie med feilhåndtering
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Returnerer result.html-mal
            
        } catch (Exception e) {
            // Bruk reservehistorie hvis AI mislykkes
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Begrens lengde
    }
}
```

**Nøkkelfunksjoner:**

1. **Rutehåndtering**: `@GetMapping("/")` viser opplastingsskjemaet, `@PostMapping("/generate-story")` behandler innsendinger
2. **Inndatavalidering**: Sjekker for tomme beskrivelser og lengdebegrensninger
3. **Sikkerhet**: Rengjør brukerinput for å forhindre XSS-angrep
4. **Feilhåndtering**: Tilbyr reservehistorier når AI-tjenesten svikter
5. **Model Binding**: Sender data til HTML-maler ved hjelp av Springs `Model`

**Reservesystem:**
Kontrolleren inkluderer forhåndsskrevede historietemplater som brukes når AI-tjenesten er utilgjengelig:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Bruk beskrivelseshash for konsistente svar
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Historietjeneste

**Fil:** `StoryService.java`

Denne tjenesten kommuniserer med Azure AI Foundry for å generere historier ved bruk av nøkkelfri autentisering:

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
        
        // Foundrys OpenAI-kompatible endepunkt finnes under /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Nøkkelfri autentisering med Microsoft Entra ID (ingen API-nøkkel)
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
        
        // Konfigurer AI-forespørselen
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Begrens svarlengde
                .temperature(0.8)          // Kontroller kreativitet (0.0-1.0)
                .build();
        
        // Send forespørsel og få svar
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Viktige komponenter:**

1. **OpenAI-klient**: Bruker den offisielle OpenAI Java SDK konfigurert for Azure AI Foundry (nøkkelfri)
2. **Systemprompt**: Setter AI-ens oppførsel til å skrive familievennlige dyrehistorier
3. **Brukerprompt**: Forteller AI nøyaktig hvilken historie som skal skrives basert på beskrivelsen
4. **Parametere**: Kontrollere lengde og kreativitetsnivå på historien
5. **Feilhåndtering**: Kaster unntak som kontrolleren fanger og håndterer

### 4. Webmaler

**Fil:** `index.html` (Opplastingsskjema)

Hovedsiden hvor brukere beskriver kjæledyrene sine:

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

Viser den genererte historien:

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

**Malens egenskaper:**

1. **Thymeleaf-integrasjon**: Bruker `th:`-attributter for dynamisk innhold
2. **Responsivt design**: CSS-styling for mobil og desktop
3. **Feilhåndtering**: Viser valideringsfeil til brukerne
4. **Klientsidig behandling**: JavaScript for bildeanalyse (ved bruk av Transformers.js)

### 5. Konfigurasjon

**Fil:** `application.properties`

Konfigurasjonsinnstillinger for applikasjonen:

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

**Konfigurasjon forklart:**

1. **Filopplasting**: Tillater bilder opptil 10MB
2. **Logging**: Styrer hva slags informasjon som logges under kjøring
3. **Azure AI Foundry**: Spesifiserer endepunkt og modellutplassering som skal brukes (nøkkelfri autentisering)
4. **Sikkerhet**: Feilhåndteringskonfigurasjon for å unngå eksponering av sensitiv informasjon

## Kjøre applikasjonen

### Steg 1: Logg inn og sett endepunktet ditt

Autentiseringen er nøkkelfri (Microsoft Entra ID), så det kreves ingen API-nøkkel. Logg inn og sett Foundry-endepunktet ditt:

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

**Hvorfor dette er nødvendig:**
- Azure AI Foundry bruker Microsoft Entra ID for å autentisere inferensforespørsler
- Nøkkelfri autentisering betyr ingen hemmeligheter i kildekode eller miljø
- Kontoen din må ha rollen **Cognitive Services OpenAI User** på ressursen

### Steg 2: Bygg og kjør

Naviger til prosjektmappen:
```bash
cd 04-PracticalSamples/petstory
```

Bygg applikasjonen:
```bash
mvn clean compile
```

Start serveren:
```bash
mvn spring-boot:run
```

Applikasjonen vil starte på `http://localhost:8080`.

### Steg 3: Test applikasjonen

1. **Åpne** `http://localhost:8080` i nettleseren din
2. **Beskriv** kjæledyret ditt i tekstfeltet (f.eks. "En leken golden retriever som elsker å hente")
3. **Klikk** på "Generate Story" for å få en AI-generert historie
4. **Alternativt**, last opp et bilde av kjæledyret for automatisk å generere en beskrivelse
5. **Se** den kreative historien basert på kjæledyrets beskrivelse

## Hvordan det hele fungerer sammen

Her er den komplette flyten når du genererer en kjæledyrhistorie:

1. **Brukerinndata**: Du beskriver kjæledyret ditt i webskjemaet
2. **Skjemainnsending**: Nettleseren sender POST-forespørsel til `/generate-story`
3. **Kontrollerbehandling**: `PetController` validerer og renser inndata
4. **AI-tjenestekall**: `StoryService` sender forespørsel til Azure AI Foundry-modellen
5. **Historiegenerering**: AI genererer en kreativ historie basert på beskrivelsen
6. **Responsbehandling**: Kontrolleren mottar historien og legger den til i modellen
7. **Malerendering**: Thymeleaf gjengir `result.html` med historien
8. **Visning**: Brukeren ser den genererte historien i nettleseren

**Feilhåndteringsflyt:**
Hvis AI-tjenesten skulle feile:
1. Kontrolleren fanger unntaket
2. Genererer en reservehistorie ved hjelp av forhåndsskrevede maler
3. Viser reservehistorien med en merknad om at AI ikke var tilgjengelig
4. Brukeren får fortsatt en historie, noe som sikrer en god brukeropplevelse

## Forstå AI-integrasjonen

### Azure AI Foundry (nøkkelfri)
Applikasjonen bruker Azure AI Foundry med nøkkelfri autentisering (Microsoft Entra ID):

```java
// Nøkkelfri autentisering - ingen API-nøkkel
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Prompt Engineering
Tjenesten bruker nøye utformede prompts for å oppnå gode resultater:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Responsbehandling
AI-responsen blir hentet ut og validert:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Neste steg

For flere eksempler, se [Kapittel 04: Praktiske eksempler](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Ansvarsfraskrivelse**:
Dette dokumentet er oversatt ved hjelp av AI-oversettelsestjenesten [Co-op Translator](https://github.com/Azure/co-op-translator). Selv om vi streber etter nøyaktighet, vær oppmerksom på at automatiske oversettelser kan inneholde feil eller unøyaktigheter. Det opprinnelige dokumentet på originalspråket skal betraktes som den autoritative kilden. For kritisk informasjon anbefales profesjonell menneskelig oversettelse. Vi er ikke ansvarlige for eventuelle misforståelser eller feiltolkninger som oppstår ved bruk av denne oversettelsen.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->