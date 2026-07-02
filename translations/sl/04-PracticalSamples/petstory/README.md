# Vodnik za ustvarjalnik zgodb o hišnih ljubljenčkih za začetnike

## Kazalo vsebine

- [Predpogoji](#predpogoji)
- [Razumevanje strukture projekta](#razumevanje-strukture-projekta)
- [Razlaga glavnih komponent](#razlaga-glavnih-komponent)
  - [1. Glavna aplikacija](#1-glavna-aplikacija)
  - [2. Spletni kontroler](#2-spletni-kontroler)
  - [3. Storitev za zgodbe](#3-storitev-za-zgodbe)
  - [4. Spletne predloge](#4-spletne-predloge)
  - [5. Konfiguracija](#5-konfiguracija)
- [Zagon aplikacije](#zagon-aplikacije)
- [Kako vse skupaj deluje](#kako-vse-skupaj-deluje)
- [Razumevanje AI integracije](#razumevanje-ai-integracije)
- [Naslednji koraki](#naslednji-koraki)

## Predpogoji

Pred začetkom se prepričajte, da imate:
- Nameščeno Javo 21 ali novejšo
- Maven za upravljanje odvisnosti
- Namestitev modela Azure AI Foundry (zagotovite jo z `azd up` — glejte [Poglavje 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), prijavljeni z `az login` (avtentikacija brez ključa)
- Osnovno znanje Jave, Spring Boota in spletnega razvoja

## Razumevanje strukture projekta

Projekt z zgodbo o hišnih ljubljenčkih vsebuje več pomembnih datotek:

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

## Razlaga glavnih komponent

### 1. Glavna aplikacija

**Datoteka:** `PetStoryApplication.java`

To je vhodna točka naše Spring Boot aplikacije:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Kaj to počne:**
- Anotacija `@SpringBootApplication` omogoča samodejno konfiguracijo in iskanje komponent
- Zažene vgrajeni spletni strežnik (Tomcat) na vratih 8080
- Samodejno ustvari vse potrebne Spring beane in storitve

### 2. Spletni kontroler

**Datoteka:** `PetController.java`

Obravnava vse spletne zahteve in interakcije uporabnikov:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Vrne predlogo index.html
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Preverjanje veljavnosti vnosa
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Očisti vnos za varnost
        String sanitizedDescription = sanitizeInput(description);
        
        // Generiraj zgodbo z obravnavo napak
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Vrne predlogo result.html
            
        } catch (Exception e) {
            // Uporabi nadomestno zgodbo, če AI ne uspe
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Omeji dolžino
    }
}
```

**Glavne funkcije:**

1. **Ravnanje usmeritev**: `@GetMapping("/")` prikazuje obrazec za nalaganje, `@PostMapping("/generate-story")` obdeluje oddaje
2. **Preverjanje vnosa**: Preverja prazne opise in omejitve dolžine
3. **Varnost**: Čisti uporabniški vnos, da prepreči XSS napade
4. **Obravnava napak**: Nudi rezervne zgodbe, ko AI storitev ne deluje
5. **Povezava modela**: Posreduje podatke v HTML predloge prek Springovega `Model`

**Sistem rezervnih zgodb:**
Kontroler vključuje vnaprej napisane predloge zgodb, ki se uporabijo, ko AI storitev ni dosegljiva:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Uporabite hash opisa za dosledne odgovore
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Storitev za zgodbe

**Datoteka:** `StoryService.java`

Ta storitev komunicira z Azure AI Foundry za generiranje zgodb z avtentikacijo brez ključa:

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
        
        // Končna točka, združljiva z OpenAI, Foundry živi pod /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Avtentikacija brez ključa z Microsoft Entra ID (brez API ključa)
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
        
        // Konfigurirajte zahtevo AI
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Omejite dolžino odgovora
                .temperature(0.8)          // Nadzor kreativnosti (0.0-1.0)
                .build();
        
        // Pošljite zahtevo in pridobite odgovor
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Ključne komponente:**

1. **OpenAI odjemalec**: Uporablja uradni OpenAI Java SDK, konfiguriran za Azure AI Foundry (brez ključa)
2. **Sistemski poziv**: Nastavi vedenje AI za pisanje družinam prijaznih zgodb o hišnih ljubljenčkih
3. **Uporabniški poziv**: Natančno naroči AI, kakšno zgodbo naj napiše na podlagi opisa
4. **Parametri**: Nadzorujejo dolžino zgodbe in raven ustvarjalnosti
5. **Obravnava napak**: Vrže izjeme, ki jih kontroler ujame in obdela

### 4. Spletne predloge

**Datoteka:** `index.html` (obrazec za nalaganje)

Glavna stran, kjer uporabniki opisujejo svoje hišne ljubljenčke:

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

**Datoteka:** `result.html` (prikaz zgodbe)

Prikaže ustvarjeno zgodbo:

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

**Značilnosti predloge:**

1. **Integracija Thymeleaf**: Uporablja atribute `th:` za dinamično vsebino
2. **Prilagodljiv dizajn**: CSS slog za mobilne in namizne naprave
3. **Obravnava napak**: Prikazuje napake preverjanja uporabnikom
4. **Obdelava na odjemalcu**: JavaScript za analizo slik (s pomočjo Transformers.js)

### 5. Konfiguracija

**Datoteka:** `application.properties`

Nastavitve konfiguracije aplikacije:

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

**Razlaga konfiguracije:**

1. **Nalaganje datotek**: Dovoli slike do velikosti 10MB
2. **Zapisovanje dnevnikov**: Nadzira, katere informacije se zapisujejo med izvajanjem
3. **Azure AI Foundry**: Določa priključek in namestitev modela za uporabo (brez ključa)
4. **Varnost**: Nastavitve za obravnavo napak, da se ne razkrije občutljivih informacij

## Zagon aplikacije

### 1. korak: Prijava in nastavitev priključka

Avtentikacija je brez ključa (Microsoft Entra ID), zato ni API ključa. Prijavite se in nastavite svoj Foundry priključek:

**Windows (Ukazna vrstica):**
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

**Zakaj je to potrebno:**
- Azure AI Foundry uporablja Microsoft Entra ID za overjanje zahtevkov za sklepanje
- Avtentikacija brez ključa pomeni, da ni skrivnosti v vaši izvorni kodi ali okolju
- Vaš račun mora imeti vlogo **Cognitive Services OpenAI User** na viru

### 2. korak: Gradnja in zagon

Pomaknite se v imenik projekta:
```bash
cd 04-PracticalSamples/petstory
```

Zgradite aplikacijo:
```bash
mvn clean compile
```

Zaženite strežnik:
```bash
mvn spring-boot:run
```

Aplikacija se bo zagnala na `http://localhost:8080`.

### 3. korak: Preizkus aplikacije

1. **Odprite** `http://localhost:8080` v brskalniku
2. **Opišite** svojega hišnega ljubljenčka v besedilno polje (npr. "Igriv zlat retriver, ki rad prinaša")
3. **Kliknite** "Generate Story" za prejem zgodbe, ustvarjene z AI
4. **Lahko pa** naložite sliko hišnega ljubljenčka za samodejno generiranje opisa
5. **Ogledate si** ustvarjalno zgodbo na podlagi opisa vašega ljubljenčka

## Kako vse skupaj deluje

Tukaj je celoten potek, ko ustvarite zgodbo o hišnem ljubljenčku:

1. **Uporabniški vnos**: Opisujete svojega ljubljenčka v spletnem obrazcu
2. **Oddaja obrazca**: Brskalnik pošlje POST zahtevek na `/generate-story`
3. **Obdelava v kontrolerju**: `PetController` preveri in očisti vnos
4. **Klic AI storitve**: `StoryService` pošlje zahtevek modelu Azure AI Foundry
5. **Ustvarjanje zgodbe**: AI ustvari ustvarjalno zgodbo na podlagi opisa
6. **Obravnava odgovora**: Kontroler prejme zgodbo in jo doda modelu
7. **Upodobitev predloge**: Thymeleaf izriše `result.html` s zgodbo
8. **Prikaz**: Uporabnik vidi ustvarjeno zgodbo v brskalniku

**Potek obravnave napak:**
Če AI storitev ne uspe:
1. Kontroler ujame izjemo
2. Ustvari rezervno zgodbo iz vnaprej napisanih predlog
3. Prikaže rezervno zgodbo z obvestilom o nedostopnosti AI
4. Uporabnik vseeno dobi zgodbo, kar zagotavlja dobro uporabniško izkušnjo

## Razumevanje AI integracije

### Azure AI Foundry (brez ključa)
Aplikacija uporablja Azure AI Foundry z avtentikacijo brez ključa (Microsoft Entra ID):

```java
// Avtentikacija brez ključa - brez API ključa
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Inženiring pozivov
Storitev uporablja skrbno pripravljene pozive za dobre rezultate:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Obdelava odgovora
Odgovor AI se izloči in preveri:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Naslednji koraki

Za več primerov glejte [Poglavje 04: Praktični primeri](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Omejitev odgovornosti**:
Ta dokument je bil preveden z uporabo AI prevajalske storitve [Co-op Translator](https://github.com/Azure/co-op-translator). Čeprav si prizadevamo za natančnost, vas prosimo, da upoštevate, da avtomatizirani prevodi lahko vsebujejo napake ali netočnosti. Izvirni dokument v njegovem izvirnem jeziku je treba obravnavati kot avtoritativni vir. Za kritične informacije je priporočljiv strokovni človeški prevod. Ne odgovarjamo za morebitna nesporazume ali napačne interpretacije, ki izhajajo iz uporabe tega prevoda.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->