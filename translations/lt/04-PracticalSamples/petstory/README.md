# Naminių gyvūnėlių istorijų generatoriaus pamoka pradedantiesiems

## Turinys

- [Reikalavimai](#reikalavimai)
- [Projekto struktūros supratimas](#projekto-struktūros-supratimas)
- [Pagrindinių komponentų paaiškinimas](#pagrindinių-komponentų-paaiškinimas)
  - [1. Pagrindinė programa](#1-pagrindinė-programa)
  - [2. Tinklo valdiklis](#2-tinklo-valdiklis)
  - [3. Istorijų paslauga](#3-istorijų-paslauga)
  - [4. Interneto šablonai](#4-interneto-šablonai)
  - [5. Konfigūracija](#5-konfigūracija)
- [Programos paleidimas](#programos-paleidimas)
- [Kaip visa tai veikia kartu](#kaip-visa-tai-veikia-kartu)
- [AI integracijos supratimas](#ai-integracijos-supratimas)
- [Kiti žingsniai](#kiti-žingsniai)

## Reikalavimai

Prieš pradėdami, įsitikinkite, kad turite:
- Įdiegtą Java 21 arba naujesnę versiją
- Maven priklausomybių valdymui
- Azure AI Foundry modelio diegimą (paleiskite su `azd up` — žr. [2 skyrių](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), prisijungę su `az login` (autentifikacija be rakto)
- Pagrindines žinias apie Java, Spring Boot ir tinklo programavimą

## Projekto struktūros supratimas

Naminių gyvūnėlių istorijų projekte yra keli svarbūs failai:

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

## Pagrindinių komponentų paaiškinimas

### 1. Pagrindinė programa

**Failas:** `PetStoryApplication.java`

Tai yra įėjimo taškas mūsų Spring Boot programai:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Ką tai daro:**
- `@SpringBootApplication` anotacija leidžia automatinį konfigūravimą ir komponentų nuskaitymą
- Paleidžia įterptą tinklo serverį (Tomcat) 8080 prievade
- Automatiškai sukuria visus reikalingus Spring bean'us ir paslaugas

### 2. Tinklo valdiklis

**Failas:** `PetController.java`

Šis valdo visus tinklo užklausimus ir vartotojo sąveikas:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Grąžina index.html šabloną
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Įvesties tikrinimas
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Išvalyti įvestį dėl saugumo
        String sanitizedDescription = sanitizeInput(description);
        
        // Generuoti istoriją su klaidų tvarkymu
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Grąžina result.html šabloną
            
        } catch (Exception e) {
            // Naudoti atsarginę istoriją, jei DI nepavyksta
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Apriboti ilgį
    }
}
```

**Pagrindinės savybės:**

1. **Maršruto valdymas**: `@GetMapping("/")` rodo įkėlimo formą, `@PostMapping("/generate-story")` apdoroja pateikimus
2. **Įvesties patikra**: Tikrina ar aprašymai nėra tušti ir neviršija ilgio ribos
3. **Sauga**: Išvalo vartotojo įvestį, kad būtų išvengta XSS atakų
4. **Klaidų valdymas**: Teikia atsarginę istoriją, jei AI paslauga nepasiekiama
5. **Modelio susiejimas**: Perduoda duomenis į HTML šablonus naudojant Spring `Model`

**Atsarginė sistema:**
Valdiklis turi iš anksto parašytus istorijų šablonus, kurie naudojami, kai AI paslauga neveikia:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Naudokite aprašymo maišą nuoseklioms atsakoms
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Istorijų paslauga

**Failas:** `StoryService.java`

Ši paslauga bendrauja su Azure AI Foundry, kad generuotų istorijas be rakto autentifikacijos:

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
        
        // Foundry suderinamas su OpenAI galinis taškas gyvena po /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Prisijungimas be rakto su Microsoft Entra ID (be API rakto)
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
        
        // Konfigūruokite AI užklausą
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Apriboti atsakymo ilgį
                .temperature(0.8)          // Valdykite kūrybiškumą (0.0-1.0)
                .build();
        
        // Siųsti užklausą ir gauti atsakymą
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Svarbūs komponentai:**

1. **OpenAI klientas**: Naudoja oficialų OpenAI Java SDK, konfigūruotą Azure AI Foundry (be rakto)
2. **Sistemos pakvietimas**: Nustato AI elgesį rašyti šeimai draugiškas naminių gyvūnėlių istorijas
3. **Vartotojo pakvietimas**: Nurodo AI tiksliai kokią istoriją rašyti pagal aprašymą
4. **Parametrai**: Valdo istorijos ilgį ir kūrybingumo lygį
5. **Klaidų valdymas**: Metamas išimtis, kurias pagauna ir apdoroja valdiklis

### 4. Interneto šablonai

**Failas:** `index.html` (Įkėlimo forma)

Pagrindinis puslapis, kuriame vartotojai aprašo savo gyvūnus:

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

**Failas:** `result.html` (Istorijos rodymas)

Rodo sugeneruotą istoriją:

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

**Šablono savybės:**

1. **Thymeleaf integracija**: Naudoja `th:` atributus dinamiškam turiniui
2. **Reaguojantis dizainas**: CSS stiliai mobiliesiems ir staliniams įrenginiams
3. **Klaidų rodymas**: Rodo patikros klaidas vartotojams
4. **Kliento pusės apdorojimas**: JavaScript vaizdų analizei (naudojant Transformers.js)

### 5. Konfigūracija

**Failas:** `application.properties`

Programos konfigūracijos nustatymai:

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

**Konfigūracijos paaiškinimas:**

1. **Failų įkėlimas**: Leidžia įkelti paveikslėlius iki 10MB
2. **Registravimas**: Kontroliuoja, kokia informacija fiksuojama vykdymo metu
3. **Azure AI Foundry**: Nurodo galinio taško adresą ir naudojamą modelio diegimą (be rakto)
4. **Sauga**: Klaidų valdymo nustatymai, kad nebūtų išduodama jautri informacija

## Programos paleidimas

### 1 žingsnis: Prisijunkite ir nustatykite galinį tašką

Autentifikacija vyksta be rakto (Microsoft Entra ID), tad API rakto nereikia. Prisijunkite ir nurodykite Foundry galinį tašką:

**Windows (Komandinė eilutė):**
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

**Kodėl tai reikalinga:**
- Azure AI Foundry naudoja Microsoft Entra ID autentifikuoti užklausas
- Be rakto autentifikacija reiškia, kad šaltinio kode ar aplinkoje nėra slaptažodžių
- Jūsų paskyra turi turėti **Cognitive Services OpenAI User** vaidmenį resurse

### 2 žingsnis: Kompiliavimas ir paleidimas

Eikite į projekto katalogą:
```bash
cd 04-PracticalSamples/petstory
```

Sukurkite programą:
```bash
mvn clean compile
```

Paleiskite serverį:
```bash
mvn spring-boot:run
```

Programa pradės veikti adresu `http://localhost:8080`.

### 3 žingsnis: Testavimas

1. **Atidarykite** `http://localhost:8080` naršyklėje
2. **Aprašykite** savo gyvūną teksto lauke (pvz., „Žaismingas auksaspalvis retriveris, mėgstantis nešti daiktus“)
3. **Paspauskite** „Generate Story“ (Sukurti istoriją), kad gautumėte AI sukurtą pasakojimą
4. **Arba** įkelkite gyvūno nuotrauką, kad automatiškai sugeneruotumėte aprašymą
5. **Peržiūrėkite** kūrybingą istoriją, sukurtą pagal jūsų aprašymą

## Kaip visa tai veikia kartu

Štai visas procesas, generuojant naminės gyvūnėlio istoriją:

1. **Vartotojo įvestis**: Jūs aprašote savo gyvūną internete
2. **Formos pateikimas**: Naršyklė siunčia POST užklausą į `/generate-story`
3. **Valdiklio apdorojimas**: `PetController` patikrina ir išvalo įvestį
4. **AI paslaugos užklausa**: `StoryService` siunčia užklausą Azure AI Foundry modelio atpažinimui
5. **Istorijos kūrimas**: AI sugeneruoja kūrybingą pasakojimą pagal aprašymą
6. **Atsakymo apdorojimas**: Valdiklis gauna istoriją ir prideda ją modelyje
7. **Šablono atvaizdavimas**: Thymeleaf generuoja `result.html` su istorija
8. **Rodymas**: Vartotojas mato sugeneruotą istoriją savo naršyklėje

**Klaidų apdorojimo eiga:**
Jei AI paslauga neveikia:
1. Valdiklis pagauna išimtį
2. Sukuria atsarginę istoriją naudojant iš anksto paruoštus šablonus
3. Parodo atsarginę istoriją su pranešimu apie AI paslaugos nebuvimą
4. Vartotojas vis tiek gauna istoriją, užtikrinant gerą naudotojo patirtį

## AI integracijos supratimas

### Azure AI Foundry (be rakto)
Programa naudoja Azure AI Foundry su autentifikacija be rakto (Microsoft Entra ID):

```java
// Autentifikacija be rakto - joks API raktas
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Pakvietimų kūrimas
Paslauga naudoja kruopščiai parinktus pakvietimus, kad gautų gerus rezultatus:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Atsakymo apdorojimas
AI atsakymas ištraukiamas ir patikrinamas:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Kiti žingsniai

Daugiau pavyzdžių rasite [4 skyriuje: Praktiniai pavyzdžiai](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Atsakomybės apribojimas**:
Šis dokumentas buvo išverstas naudojant dirbtinio intelekto vertimo paslaugą [Co-op Translator](https://github.com/Azure/co-op-translator). Nors siekiame tikslumo, prašome atkreipti dėmesį, kad automatiniai vertimai gali turėti klaidų ar netikslumų. Originalus dokumentas jo gimtąja kalba laikomas autoritetingu šaltiniu. Svarbiai informacijai rekomenduojama naudoti profesionalų žmogiškąjį vertimą. Mes neatsakome už jokius nesusipratimus ar neteisingą interpretaciją, kilusią naudojantis šiuo vertimu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->