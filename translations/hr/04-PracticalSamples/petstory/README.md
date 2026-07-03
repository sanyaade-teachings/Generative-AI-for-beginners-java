# Vodič za generiranje priča o kućnim ljubimcima za početnike

## Sadržaj

- [Preduvjeti](#preduvjeti)
- [Razumijevanje strukture projekta](#razumijevanje-strukture-projekta)
- [Objašnjenje osnovnih komponenti](#objašnjenje-osnovnih-komponenti)
  - [1. Glavna aplikacija](#1-glavna-aplikacija)
  - [2. Web kontroler](#2-web-kontroler)
  - [3. Servis za priče](#3-servis-za-priče)
  - [4. Web predlošci](#4-web-predlošci)
  - [5. Konfiguracija](#5-konfiguracija)
- [Pokretanje aplikacije](#pokretanje-aplikacije)
- [Kako sve to funkcionira zajedno](#kako-sve-to-funkcionira-zajedno)
- [Razumijevanje AI integracije](#razumijevanje-ai-integracije)
- [Sljedeći koraci](#sljedeći-koraci)

## Preduvjeti

Prije početka, provjerite imate li:
- Instaliranu Javu 21 ili noviju verziju
- Maven za upravljanje ovisnostima
- Azure AI Foundry model deployment (postavite ga s `azd up` — vidi [Poglavlje 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), prijavljeni putem `az login` (autentifikacija bez ključa)
- Osnovno znanje Jave, Spring Boota i web razvoja

## Razumijevanje strukture projekta

Projekt priče o kućnim ljubimcima sadrži nekoliko važnih datoteka:

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

## Objašnjenje osnovnih komponenti

### 1. Glavna aplikacija

**Datoteka:** `PetStoryApplication.java`

Ovo je ulazna točka naše Spring Boot aplikacije:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Što ovo radi:**
- `@SpringBootApplication` anotacija omogućuje automatsku konfiguraciju i skeniranje komponenti
- Pokreće ugrađeni web poslužitelj (Tomcat) na portu 8080
- Automatski kreira sve potrebne Spring beane i servise

### 2. Web kontroler

**Datoteka:** `PetController.java`

Ovaj kontroler obrađuje sve web zahtjeve i interakcije s korisnikom:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Vraća predložak index.html
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Validacija unosa
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Očistite unos radi sigurnosti
        String sanitizedDescription = sanitizeInput(description);
        
        // Generiraj priču s rukovanjem pogreškama
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Vraća predložak result.html
            
        } catch (Exception e) {
            // Koristi rezervnu priču ako AI zakaže
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Ograniči duljinu
    }
}
```

**Ključne značajke:**

1. **Rukovanje rutama**: `@GetMapping("/")` prikazuje obrazac za upload, `@PostMapping("/generate-story")` obrađuje predaje
2. **Validacija unosa**: Provjerava prazne opise i ograničenja duljine
3. **Sigurnost**: Sanitizira korisnički unos kako bi spriječio XSS napade
4. **Rukovanje pogreškama**: Pruža zamjenske priče kad AI servis zakaže
5. **Povezivanje modela**: Prosljeđuje podatke u HTML predloške koristeći Springov `Model`

**Sustav zamjene:**
Kontroler uključuje unaprijed napisane predloške priča koji se koriste kad AI servis nije dostupan:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Koristite hash opisa za dosljedne odgovore
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Servis za priče

**Datoteka:** `StoryService.java`

Ovaj servis komunicira s Azure AI Foundry za generiranje priča koristeći autentifikaciju bez ključa:

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
        
        // OpenAI-kompatibilna točka kraja Foundryja nalazi se pod /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Autentifikacija bez ključa s Microsoft Entra ID (bez API ključa)
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
        
        // Konfigurirajte AI zahtjev
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Ograniči duljinu odgovora
                .temperature(0.8)          // Kontroliraj kreativnost (0.0-1.0)
                .build();
        
        // Pošalji zahtjev i dobij odgovor
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Ključne komponente:**

1. **OpenAI klijent**: Koristi službeni OpenAI Java SDK konfiguriran za Azure AI Foundry (autentifikacija bez ključa)
2. **Sistemski prompt**: Postavlja ponašanje AI-a za pisanje obiteljskih priča o kućnim ljubimcima
3. **Kornički prompt**: Precizno govori AI-u koju priču napisati na temelju opisa
4. **Parametri**: Kontrolira duljinu i razinu kreativnosti priče
5. **Rukovanje pogreškama**: Bacaju se iznimke koje kontroler hvata i obrađuje

### 4. Web predlošci

**Datoteka:** `index.html` (obrazac za upload)

Glavna stranica gdje korisnici opisuju svoje ljubimce:

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

**Datoteka:** `result.html` (prikaz priče)

Prikazuje generiranu priču:

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

**Značajke predloška:**

1. **Integracija s Thymeleafom**: Koristi `th:` atribute za dinamički sadržaj
2. **Odazivni dizajn**: CSS stilovi za mobilne uređaje i desktop računala
3. **Rukovanje pogreškama**: Prikazuje korisničke poruke o neuspjehu validacije
4. **Obrada na strani klijenta**: JavaScript za analizu slika (koristeći Transformers.js)

### 5. Konfiguracija

**Datoteka:** `application.properties`

Postavke konfiguracije aplikacije:

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

**Objašnjenje konfiguracije:**

1. **Upload datoteka**: Dozvoljava slike veličine do 10MB
2. **Logiranje**: Kontrolira što se zapisuje tijekom izvođenja
3. **Azure AI Foundry**: Navodi endpoint i model deployment za korištenje (autentifikacija bez ključa)
4. **Sigurnost**: Konfiguracija rukovanja pogreškama kako ne bi izložila osjetljive informacije

## Pokretanje aplikacije

### Korak 1: Prijavite se i postavite endpoint

Autentifikacija je bez ključa (Microsoft Entra ID), stoga nema API ključa. Prijavite se i postavite svoj Foundry endpoint:

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

**Zašto je ovo potrebno:**
- Azure AI Foundry koristi Microsoft Entra ID za autentifikaciju zahtjeva za predviđanje
- Autentifikacija bez ključa znači da nema tajni u izvornom kodu ili okruženju
- Vaš račun mora imati ulogu **Cognitive Services OpenAI User** na resursu

### Korak 2: Izgradite i pokrenite

Idite u direktorij projekta:
```bash
cd 04-PracticalSamples/petstory
```

Izgradite aplikaciju:
```bash
mvn clean compile
```

Pokrenite poslužitelj:
```bash
mvn spring-boot:run
```

Aplikacija će se pokrenuti na `http://localhost:8080`.

### Korak 3: Testirajte aplikaciju

1. **Otvorite** `http://localhost:8080` u pregledniku
2. **Opširno opišite** svog ljubimca u tekstualnom polju (npr. "Razigrani zlatni retriver koji voli donositi lopticu")
3. **Kliknite** "Generate Story" za dobivanje priče koju je generirao AI
4. **Alternativno**, učitajte sliku ljubimca za automatsko generiranje opisa
5. **Pogledajte** kreativnu priču baziranu na opisu vašeg ljubimca

## Kako sve to funkcionira zajedno

Evo kompletnog toka kada generirate priču o kućnom ljubimcu:

1. **Korisnički unos**: Opisujete svog ljubimca u web obrascu
2. **Slanje obrasca**: Preglednik šalje POST zahtjev na `/generate-story`
3. **Obrada u kontroleru**: `PetController` validira i sanitizira unos
4. **Poziv AI servisu**: `StoryService` šalje zahtjev modelu Azure AI Foundry
5. **Generiranje priče**: AI kreira kreativnu priču na temelju opisa
6. **Rukovanje odgovorom**: Kontroler prima priču i dodaje je u model
7. **Prikaz predloška**: Thymeleaf renderira `result.html` s pričom
8. **Prikaz korisniku**: Korisnik vidi generiranu priču u pregledniku

**Tijek rukovanja pogreškama:**
Ako AI servis zakaže:
1. Kontroler hvata iznimku
2. Generira zamjensku priču koristeći unaprijed napisane predloške
3. Prikazuje zamjensku priču uz napomenu o nedostupnosti AI usluge
4. Korisnik i dalje dobiva priču, osiguravajući dobar korisnički doživljaj

## Razumijevanje AI integracije

### Azure AI Foundry (bez ključa)
Aplikacija koristi Azure AI Foundry s autentifikacijom bez ključa (Microsoft Entra ID):

```java
// Autentifikacija bez ključa - bez API ključa
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Inženjering prompta
Servis koristi pažljivo kreirane promptove za dobre rezultate:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Obrada odgovora
AI odgovor se ekstrahira i validira:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Sljedeći koraci

Za više primjera, pogledajte [Poglavlje 04: Praktični primjeri](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Napomena**:
Ovaj dokument je preveden korištenjem AI prevoditeljskog servisa [Co-op Translator](https://github.com/Azure/co-op-translator). Iako težimo točnosti, imajte na umu da automatski prijevodi mogu sadržavati greške ili netočnosti. Izvorni dokument na izvornom jeziku treba smatrati autoritativnim izvorom. Za važne informacije preporuča se profesionalni ljudski prijevod. Nismo odgovorni za bilo kakva nesporazumevanja ili pogrešne interpretacije koje proizlaze iz korištenja ovog prijevoda.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->