# Pet Story Generator oktatóanyag kezdőknek

## Tartalomjegyzék

- [Előfeltételek](#előfeltételek)
- [A projekt struktúrájának megértése](#a-projekt-struktúrájának-megértése)
- [A fő komponensek magyarázata](#a-fő-komponensek-magyarázata)
  - [1. Főalkalmazás](#1-főalkalmazás)
  - [2. Web vezérlő](#2-web-vezérlő)
  - [3. Történet szolgáltatás](#3-történet-szolgáltatás)
  - [4. Web sablonok](#4-web-sablonok)
  - [5. Konfiguráció](#5-konfiguráció)
- [Az alkalmazás futtatása](#az-alkalmazás-futtatása)
- [Hogyan működik minden együtt](#hogyan-működik-minden-együtt)
- [Az AI integráció megértése](#az-ai-integráció-megértése)
- [Következő lépések](#következő-lépések)

## Előfeltételek

A kezdés előtt győződjön meg arról, hogy rendelkezik:
- Telepített Java 21 vagy újabb verzióval
- Maven a függőségkezeléshez
- Azure AI Foundry modell üzembe helyezéssel (províziós `azd up` parancsal — lásd [2. fejezet](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), bejelentkezve `az login`-nal (kulcs nélküli hitelesítés)
- Alapvető Java, Spring Boot és webfejlesztési ismeretekkel

## A projekt struktúrájának megértése

A pet story projekt több fontos fájlt tartalmaz:

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

## A fő komponensek magyarázata

### 1. Főalkalmazás

**Fájl:** `PetStoryApplication.java`

Ez a belépési pont a Spring Boot alkalmazásunkhoz:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Mit csinál ez:**
- Az `@SpringBootApplication` annotáció engedélyezi az automatikus konfigurációt és a komponens-keresést
- Elindít egy beágyazott webszervert (Tomcat) a 8080-as porton
- Automatikusan létrehozza az összes szükséges Spring bean-t és szolgáltatást

### 2. Web vezérlő

**Fájl:** `PetController.java`

Ez kezeli az összes webes kérést és felhasználói interakciót:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Visszaadja az index.html sablont
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Bemeneti érvényesítés
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Bemenet megtisztítása a biztonság érdekében
        String sanitizedDescription = sanitizeInput(description);
        
        // Történet generálása hibakezeléssel
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Visszaadja a result.html sablont
            
        } catch (Exception e) {
            // Visszaesési történet használata, ha az AI meghibásodik
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Hossz korlátozása
    }
}
```

**Fő jellemzők:**

1. **Útvonal kezelés**: A `@GetMapping("/")` megjeleníti a feltöltési űrlapot, a `@PostMapping("/generate-story")` feldolgozza a beküldéseket
2. **Bemenet érvényesítés**: Ellenőrzi az üres leírásokat és a hosszkorlátokat
3. **Biztonság**: Kitisztítja a felhasználói bemenetet XSS támadások ellen
4. **Hiba kezelés**: Tartalék történeteket biztosít, ha az AI szolgáltatás nem működik
5. **Modellek kötése**: Adatokat továbbít a HTML sablonok számára Spring `Model` segítségével

**Tartalék rendszer:**
A vezérlő előre megírt történet sablonokat tartalmaz, amelyeket akkor használ, amikor az AI szolgáltatás nem elérhető:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Használja a leírás hash-t az egységes válaszokhoz
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Történet szolgáltatás

**Fájl:** `StoryService.java`

Ez a szolgáltatás kommunikál az Azure AI Foundry-val, hogy kulcs nélküli hitelesítéssel generáljon történeteket:

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
        
        // A Foundry OpenAI-kompatibilis végpontja az /openai/v1/ útvonalon található
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Kulcs nélküli hitelesítés Microsoft Entra ID-vel (API kulcs nélkül)
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
        
        // Az AI kérés konfigurálása
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Válasz hosszának korlátozása
                .temperature(0.8)          // Kreativitás szabályozása (0.0-1.0)
                .build();
        
        // Kérés elküldése és válasz fogadása
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Fő összetevők:**

1. **OpenAI kliens**: Használja az hivatalos OpenAI Java SDK-t, Azure AI Foundry konfigurációval (kulcs nélküli)
2. **Rendszer prompt**: Beállítja az AI viselkedését családbarát kisállat történetek írására
3. **Felhasználói prompt**: Pontosan megmondja az AI-nak, milyen történetet írjon a leírás alapján
4. **Paraméterek**: Szabályozza a történet hosszát és kreativitási szintjét
5. **Hiba kezelés**: Kivételt dob, amelyet a vezérlő elkap és kezel

### 4. Web sablonok

**Fájl:** `index.html` (feltöltési űrlap)

A fő oldal, ahol a felhasználók leírják a kisállataikat:

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

**Fájl:** `result.html` (történet megjelenítése)

Megjeleníti a generált történetet:

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

**Sablon funkciók:**

1. **Thymeleaf integráció**: `th:` attribútumokat használ dinamukus tartalomhoz
2. **Reszponzív design**: CSS stílusok mobilra és asztali gépre
3. **Hiba kezelés**: Megjeleníti az érvényesítési hibákat a felhasználóknak
4. **Kliens oldali feldolgozás**: JavaScript kép elemzéshez (Transformers.js használatával)

### 5. Konfiguráció

**Fájl:** `application.properties`

Az alkalmazás beállításai:

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

**A konfiguráció magyarázata:**

1. **Fájl feltöltés**: Engedélyezi a maximum 10MB méretű képfeltöltést
2. **Naplózás**: Szabályozza a futás közbeni információk naplózását
3. **Azure AI Foundry**: Megadja a végpontot és a modell bevetést (kulcs nélküli hitelesítés)
4. **Biztonság**: Hiba kezelés konfiguráció, hogy elkerülje az érzékeny adatok nyilvánosságra kerülését

## Az alkalmazás futtatása

### 1. lépés: Jelentkezzen be és állítsa be a végpontját

A hitelesítés kulcs nélküli (Microsoft Entra ID), így nincs API kulcs. Jelentkezzen be és állítsa be a Foundry végpontját:

**Windows (Parancssor):**
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

**Miért van erre szükség:**
- Az Azure AI Foundry Microsoft Entra ID-t használ a kérések hitelesítéséhez
- Kulcs nélküli hitelesítés azt jelenti, hogy nincs titok a forráskódban vagy a környezetben
- Fiókjának rendelkeznie kell a **Cognitive Services OpenAI User** szerepkörrel az erőforráson

### 2. lépés: Build és futtatás

Lépjen a projekt mappába:
```bash
cd 04-PracticalSamples/petstory
```

Építse meg az alkalmazást:
```bash
mvn clean compile
```

Indítsa el a szervert:
```bash
mvn spring-boot:run
```

Az alkalmazás a `http://localhost:8080` címen indul el.

### 3. lépés: Tesztelje az alkalmazást

1. **Nyissa meg** a `http://localhost:8080` címet a böngészőjében
2. **Írja le** a kisállatát a szövegdobozba (pl. "Egy játékos golden retriever, aki imád apportírozni")
3. **Kattintson** a "Generate Story" gombra, hogy AI által generált történetet kapjon
4. **Vagy** töltsön fel egy kisállat képet, hogy automatikusan generálódjon leírás
5. **Nézze meg** a kreatív történetet a kisállat leírása alapján

## Hogyan működik minden együtt

Íme a teljes folyamat, amikor kisállat történetet generál:

1. **Felhasználói bemenet**: Ön leírja a kisállatát a webes űrlapon
2. **Űrlap beküldése**: A böngésző POST kérést küld a `/generate-story` címre
3. **Vezérlő feldolgozás**: A `PetController` érvényesíti és kitisztítja a bemenetet
4. **AI szolgáltatás hívás**: A `StoryService` kérést küld az Azure AI Foundry modellnek
5. **Történet generálás**: Az AI kreatív történetet generál a leírás alapján
6. **Válasz feldolgozás**: A vezérlő megkapja a történetet és hozzáadja a modellhez
7. **Sablon feldolgozás**: A Thymeleaf kirajzolja a `result.html`-t a történettel
8. **Megjelenítés**: A felhasználó látja a generált történetet a böngészőjében

**Hiba kezelési folyamat:**
Ha az AI szolgáltatás hibát jelez:
1. A vezérlő elkapja a kivételt
2. Tartalék történetet generál előre megírt sablonokból
3. Megjeleníti a tartalék történetet egy megjegyzéssel az AI elérhetetlenségéről
4. A felhasználó így is kap történetet, ami jó felhasználói élményt biztosít

## Az AI integráció megértése

### Azure AI Foundry (kulcs nélküli)
Az alkalmazás Azure AI Foundry-t használ kulcs nélküli hitelesítéssel (Microsoft Entra ID):

```java
// Kulcs nélküli hitelesítés - nincs API-kulcs
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Prompt készítés
A szolgáltatás gondosan megírt promptokat használ a jó eredmények érdekében:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Válasz feldolgozás
Az AI választ kinyeri és ellenőrzi:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Következő lépések

További példákért lásd [4. fejezet: Gyakorlati példák](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Jogi nyilatkozat**:
Ez a dokumentum az AI fordítási szolgáltatás, a [Co-op Translator](https://github.com/Azure/co-op-translator) segítségével készült. Bár az pontosságra törekszünk, kérjük, vegye figyelembe, hogy az automatikus fordítások hibákat vagy pontatlanságokat tartalmazhatnak. Az eredeti dokumentum az anyanyelvén tekintendő hiteles forrásnak. Fontos információk esetén professzionális emberi fordítást javasolunk. Nem vállalunk felelősséget semmilyen félreértésért vagy téves értelmezésért, amely ebből a fordításból ered.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->