# Průvodce generátorem příběhů o mazlíčcích pro začátečníky

## Obsah

- [Předpoklady](#prerekvizity)
- [Pochopení struktury projektu](#pochopeni-struktury-projektu)
- [Vysvětlení hlavních komponent](#vysvetleni-hlavnich-komponent)
  - [1. Hlavní aplikace](#1-hlavni-aplikace)
  - [2. Webový kontroler](#2-webovy-kontroler)
  - [3. Služba pro příběhy](#3-sluzba-pro-pribehy)
  - [4. Webové šablony](#4-webove-sablony)
  - [5. Konfigurace](#5-konfigurace)
- [Spuštění aplikace](#spusteni-aplikace)
- [Jak to vše spolu funguje](#jak-to-vse-spolu-funguje)
- [Pochopení AI integrace](#pochopeni-ai-integrace)
- [Další kroky](#dalsi-kroky)

## Prerekvizity

Před zahájením se ujistěte, že máte:
- Nainstalováno Java 21 nebo vyšší
- Maven pro správu závislostí
- Nasazený model Azure AI Foundry (zprovozněte pomocí `azd up` — viz [Kapitola 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), přihlášený přes `az login` (autentizace bez klíče)
- Základní znalosti Javy, Spring Boot a webového vývoje

## Pochopení struktury projektu

Projekt příběhů o mazlíčcích obsahuje několik důležitých souborů:

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

## Vysvětlení hlavních komponent

### 1. Hlavní aplikace

**Soubor:** `PetStoryApplication.java`

Toto je vstupní bod naší Spring Boot aplikace:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Co toto dělá:**
- Anotace `@SpringBootApplication` povoluje automatickou konfiguraci a skenování komponent
- Spouští zabudovaný webový server (Tomcat) na portu 8080
- Automaticky vytváří všechny potřebné Spring beany a služby

### 2. Webový kontroler

**Soubor:** `PetController.java`

Tento zpracovává všechny webové požadavky a interakce uživatele:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Vrací šablonu index.html
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Kontrola vstupních dat
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Očistit vstup pro bezpečnost
        String sanitizedDescription = sanitizeInput(description);
        
        // Generovat příběh s ošetřením chyb
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Vrací šablonu result.html
            
        } catch (Exception e) {
            // Použít záložní příběh, pokud AI selže
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Omezit délku
    }
}
```

**Klíčové vlastnosti:**

1. **Zpracování tras:** `@GetMapping("/")` zobrazí formulář pro nahrání, `@PostMapping("/generate-story")` zpracuje odeslaná data
2. **Validace vstupů:** Kontroluje prázdné popisy a omezení délky
3. **Bezpečnost:** Čistí uživatelský vstup, aby předešel útokům XSS
4. **Zpracování chyb:** Poskytuje záložní příběhy, když AI služba selže
5. **Vazba modelu:** Předává data do HTML šablon pomocí Spring `Model`

**Záložní systém:**
Kontroler obsahuje předpřipravené šablony příběhů, které se použijí, pokud není AI služba dostupná:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Použijte hash popisu pro konzistentní odpovědi
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Služba pro příběhy

**Soubor:** `StoryService.java`

Tato služba komunikuje s Azure AI Foundry pro generování příběhů s autentizací bez klíče:

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
        
        // OpenAI-kompatibilní konec Foundry se nachází pod /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Autentizace bez klíče pomocí Microsoft Entra ID (žádný API klíč)
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
        
        // Nakonfigurujte požadavek AI
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Omezte délku odpovědi
                .temperature(0.8)          // Řiďte kreativitu (0.0-1.0)
                .build();
        
        // Odešlete požadavek a získejte odpověď
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Klíčové složky:**

1. **OpenAI klient:** Používá oficiální OpenAI Java SDK nakonfigurované pro Azure AI Foundry (bezklíčová autentizace)
2. **Systémový prompt:** Určuje chování AI, aby psala příběhy o mazlíčcích vhodné pro rodiny
3. **Uživatelský prompt:** Říká AI, jaký příběh má napsat na základě popisu
4. **Parametry:** Řídí délku a kreativitu příběhu
5. **Zpracování chyb:** Vyvolává výjimky, které kontroler zachytí a zpracuje

### 4. Webové šablony

**Soubor:** `index.html` (Formulář pro nahrání)

Hlavní stránka, kde uživatelé popisují své mazlíčky:

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

**Soubor:** `result.html` (Zobrazení příběhu)

Zobrazuje vygenerovaný příběh:

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

**Vlastnosti šablony:**

1. **Integrace Thymeleaf:** Používá atributy `th:` pro dynamický obsah
2. **Responzivní design:** CSS stylování pro mobily i desktop
3. **Zpracování chyb:** Zobrazuje uživateli chyby validace
4. **Zpracování na klientovi:** JavaScript pro analýzu obrázků (pomocí Transformers.js)

### 5. Konfigurace

**Soubor:** `application.properties`

Nastavení konfigurace aplikace:

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

**Vysvětlení konfigurace:**

1. **Nahrávání souborů:** Povolení obrázků do 10MB
2. **Protokolování:** Řízení, jaké informace se logují při běhu
3. **Azure AI Foundry:** Specifikace endpointu a modelu (bezklíčová autentizace)
4. **Bezpečnost:** Konfigurace zpracování chyb, aby se nezobrazovaly citlivé informace

## Spuštění aplikace

### Krok 1: Přihlaste se a nastavte endpoint

Autentizace probíhá bez klíče (Microsoft Entra ID), proto se nepoužívá API klíč. Přihlaste se a nastavte svůj Foundry endpoint:

**Windows (Příkazový řádek):**
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

**Proč je to potřeba:**
- Azure AI Foundry používá Microsoft Entra ID pro autentizaci požadavků na inference
- Bezklíčová autentizace znamená, že v kódu ani prostředí nejsou žádná tajemství
- Váš účet potřebuje roli **Cognitive Services OpenAI User** na zdroji

### Krok 2: Sestavte a spusťte

Přejděte do složky projektu:
```bash
cd 04-PracticalSamples/petstory
```

Sestavte aplikaci:
```bash
mvn clean compile
```

Spusťte server:
```bash
mvn spring-boot:run
```

Aplikace se spustí na adrese `http://localhost:8080`.

### Krok 3: Testujte aplikaci

1. **Otevřete** `http://localhost:8080` ve svém prohlížeči
2. **Popište** svého mazlíčka do textového pole (např. "Hrníček hravý zlatý retrívr, který rád aportuje")
3. **Klikněte** na "Generate Story", aby vám AI vytvořila příběh
4. **Alternativně** nahrajte obrázek mazlíčka pro automatické vygenerování popisu
5. **Prohlédněte** si kreativní příběh založený na vámi zadaném popisu mazlíčka

## Jak to vše spolu funguje

Tady je celý proces, když vygenerujete příběh o mazlíčkovi:

1. **Uživatelský vstup:** Vy popíšete mazlíčka ve webovém formuláři
2. **Odeslání formuláře:** Prohlížeč pošle POST požadavek na `/generate-story`
3. **Zpracování kontrolerem:** `PetController` vstup validuje a čistí
4. **Volání AI služby:** `StoryService` odešle požadavek modelu Azure AI Foundry
5. **Generování příběhu:** AI vygeneruje kreativní příběh podle popisu
6. **Zpracování odpovědi:** Kontroler obdrží příběh a vloží ho do modelu
7. **Rendrování šablony:** Thymeleaf vykreslí `result.html` s příběhem
8. **Zobrazení:** Uživatel uvidí vygenerovaný příběh ve svém prohlížeči

**Zpracování chyb:**
Pokud AI služba selže:
1. Kontroler zachytí výjimku
2. Vygeneruje záložní příběh pomocí předpřipravených šablon
3. Zobrazí záložní příběh s upozorněním, že AI není dostupná
4. Uživatel stále obdrží příběh, aby byla zajištěna dobrá uživatelská zkušenost

## Pochopení AI integrace

### Azure AI Foundry (bez klíče)
Aplikace používá Azure AI Foundry s bezklíčovou autentizací (Microsoft Entra ID):

```java
// Autentizace bez klíče - bez API klíče
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Tvorba promptů
Služba používá pečlivě navržené prompty pro dosažení dobrých výsledků:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Zpracování odpovědi
Odpověď AI se extrahuje a validuje:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Další kroky

Pro více příkladů viz [Kapitola 04: Praktické ukázky](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Prohlášení o omezení odpovědnosti**:
Tento dokument byl přeložen pomocí AI překladatelské služby [Co-op Translator](https://github.com/Azure/co-op-translator). Přestože usilujeme o co největší přesnost, mějte prosím na paměti, že automatizované překlady mohou obsahovat chyby nebo nepřesnosti. Originální dokument v jeho mateřském jazyce by měl být považován za autoritativní zdroj. Pro kritické informace se doporučuje profesionální lidský překlad. Nejsme odpovědní za jakékoli nedorozumění nebo nesprávné interpretace vzniklé použitím tohoto překladu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->