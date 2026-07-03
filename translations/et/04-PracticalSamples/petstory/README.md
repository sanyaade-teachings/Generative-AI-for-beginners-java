# Lemmiklooma loo generaatori juhend algajatele

## Sisukord

- [Eeldused](#eeldused)
- [Projekti struktuuri mõistmine](#projekti-struktuuri-mõistmine)
- [Põhikompontendid selgitatud](#põhikompontendid-selgitatud)
  - [1. Põhirakendus](#1-põhirakendus)
  - [2. Veebikontroller](#2-veebikontroller)
  - [3. Loo teenus](#3-loo-teenus)
  - [4. Veebimallid](#4-veebimallid)
  - [5. Konfiguratsioon](#5-konfiguratsioon)
- [Rakenduse käivitamine](#rakenduse-käivitamine)
- [Kuidas see kõik koos töötab](#kuidas-see-kõik-koos-töötab)
- [AI integratsiooni mõistmine](#ai-integratsiooni-mõistmine)
- [Järgmised sammud](#järgmised-sammud)

## Eeldused

Enne alustamist veendu, et sul on olemas:
- Java 21 või uuem versioon
- Maven sõltuvuste halduseks
- Azure AI Foundry mudeli juurutus (provisioneeritav käsuga `azd up` — vt [2. peatükk](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), sisse logitud käsklusega `az login` (võtmeta autentimine)
- Põhilised teadmised Javast, Spring Bootist ja veebiarendusest

## Projekti struktuuri mõistmine

Lemmiklooma loo projekt sisaldab mitmeid olulisi faile:

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

## Põhikompontendid selgitatud

### 1. Põhirakendus

**Fail:** `PetStoryApplication.java`

See on meie Spring Boot rakenduse sisenemispunkt:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Mis see teeb:**
- `@SpringBootApplication` annotatsioon võimaldab automaatset konfiguratsiooni ja komponentide skannimist
- Käivitab manustatud veebi serveri (Tomcat) pordil 8080
- Loob automaatselt kõik vajalikud Springi bean’id ja teenused

### 2. Veebikontroller

**Fail:** `PetController.java`

See haldab kõiki veebipäringuid ja kasutajate interaktsioone:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Tagastab index.html malli
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Sisendi valideerimine
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Turvalisuse tagamiseks puhasta sisend
        String sanitizedDescription = sanitizeInput(description);
        
        // Loo lugu koos veakäsitlusega
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Tagastab result.html malli
            
        } catch (Exception e) {
            // Kasuta varuplaani lugu, kui tehisintellekt ebaõnnestub
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Pikkuse piiramine
    }
}
```

**Peamised omadused:**

1. **Marsruudi haldus**: `@GetMapping("/")` kuvab üleslaadimise vormi, `@PostMapping("/generate-story")` töötleb esitatud andmeid
2. **Sisendi valideerimine**: Kontrollib tühje kirjeldusi ja pikkuse piiranguid
3. **Turvalisus**: Saniteerib kasutaja sisendi XSS rünnakute vältimiseks
4. **Veakäsitlus**: Pakkub varuplaanina lugusid, kui AI teenus ei tööta
5. **Mudeli sidumine**: Edastab andmed HTML mallidele Springi `Model` abil

**Varuplaani süsteem:**
Kontroller sisaldab eelkirjutatud lugude malle, mida kasutatakse, kui AI teenus pole kättesaadav:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Kasuta järjepidevate vastuste jaoks kirjelduste räsi
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Loo teenus

**Fail:** `StoryService.java`

See teenus suhtleb Azure AI Foundry-ga, et lugusid genereerida võtmeta autentimisega:

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
        
        // Foundry OpenAI-ga ühilduv ühenduspunkt asub aadressil /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Võtmeta autentimine Microsoft Entra ID-ga (ilma API võtmeta)
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
        
        // AI-päringu seadistamine
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Vastuse pikkuse piiramine
                .temperature(0.8)          // Loovuse juhtimine (0.0-1.0)
                .build();
        
        // Saada päring ja saa vastus
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Olulised komponendid:**

1. **OpenAI klient**: Kasutab ametlikku OpenAI Java SDK-d, mis on konfigureeritud Azure AI Foundry jaoks (võtmeta)
2. **Süsteemi prompt**: Seadistab AI käitumiseks perekonnasõbralike lemmikloomalugude kirjutamise
3. **Kasutajaprompt**: Selgitab AI-le täpselt, millist lugu kirjeldusest kirjutada
4. **Parameetrid**: Kontrollib loo pikkust ja loovusastet
5. **Veakäsitlus**: Visab erandeid, mida kontroller püüab ja töötleb

### 4. Veebimallid

**Fail:** `index.html` (Üleslaadimise vorm)

Põhileht, kus kasutajad kirjeldavad oma lemmiklooma:

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

**Fail:** `result.html` (Lugude kuvamine)

Kuvab genereeritud loo:

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

**Mallide omadused:**

1. **Thymeleaf integratsioon**: Kasutab `th:` atribuute dünaamilise sisu jaoks
2. **Reageeriv disain**: CSS stiilid nii mobiilile kui ka lauaarvutile
3. **Veakäsitlus**: Kuvab valideerimisvead kasutajatele
4. **Kliendipoolne töötlemine**: JavaScript pildianalüüsiks (kasutades Transformers.js)

### 5. Konfiguratsioon

**Fail:** `application.properties`

Rakenduse konfiguratsiooniseaded:

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

**Konfiguratsiooni selgitus:**

1. **Faili üleslaadimine**: Lubab pilte kuni 10MB
2. **Logimine**: Kontrollib, millist infot täitmise ajal logitakse
3. **Azure AI Foundry**: Määratleb kasutatava lõpp-punkti ja mudeli juurutuse (võtmeta autentimine)
4. **Turvalisus**: Veahaldus konfiguratsioon, et vältida tundliku info lekkimist

## Rakenduse käivitamine

### 1. samm: Logi sisse ja määra oma lõpp-punkt

Autentimine on võtmeta (Microsoft Entra ID), seega API võtit pole. Logi sisse ja määra oma Foundry lõpp-punkt:

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

**Miks see on vajalik:**
- Azure AI Foundry kasutab päringute autentimiseks Microsoft Entra ID-d
- Võtmeta autentimise puhul pole saladusi sinu lähtekoodis ega keskkonnas
- Sinu kontol peab olema **Cognitive Services OpenAI User** roll ressursil

### 2. samm: Koosta ja käivita

Mine projekti kausta:
```bash
cd 04-PracticalSamples/petstory
```

Koosta rakendus:
```bash
mvn clean compile
```

Käivita server:
```bash
mvn spring-boot:run
```

Rakendus jookseb aadressil `http://localhost:8080`.

### 3. samm: Testi rakendust

1. **Ava** `http://localhost:8080` oma brauseris
2. **Kirjelda** oma lemmikut tekstialal (näiteks “Mänguhimuline kuldne retriiver, kes armastab toob palliga”)
3. **Vajuta** "Generate Story", et saada AI genereeritud lugu
4. **Või** lae üles lemmiklooma pilt automaatseks kirjelduseks
5. **Vaata** loomingulist lugu, mis põhineb su kirjelduse põhjal

## Kuidas see kõik koos töötab

Lemmiklooma loo genereerimise täielik voog:

1. **Kasutaja sisend**: Sa kirjeldad oma lemmikut veebivormil
2. **Vormi esitamine**: Brauser saadab POST päringu `/generate-story` aadressile
3. **Kontrolleri töötlemine**: `PetController` valideerib ja puhastab sisendi
4. **AI teenuse kutse**: `StoryService` saadab päringu Azure AI Foundry mudelile
5. **Loo genereerimine**: AI loob kirjelduse põhjal loomingulise loo
6. **Vastuse töötlemine**: Kontroller võtab loo vastu ja lisab mudeleid
7. **Malli renderdamine**: Thymeleaf kuvab `result.html` mallis loo
8. **Kuvamine**: Kasutaja näeb oma brauseris genereeritud lugu

**Veakäsitluse voog:**
Kui AI teenus ebaõnnestub:
1. Kontroller püüab erandi kinni
2. Genereerib varuplaaniloo eelkirjutatud mallidest
3. Kuvab varuloa koos märgiga, et AI teenus pole saadaval
4. Kasutaja saab siiski loo, tagades hea kasutajakogemuse

## AI integratsiooni mõistmine

### Azure AI Foundry (võtmeta)
Rakendus kasutab Azure AI Foundryt võtmeta autentimisega (Microsoft Entra ID):

```java
// Võtmeta autentimine - puudub API võti
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Promptide koostamine
Teenusel on hoolikalt loodud promptid, et saada häid tulemusi:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Vastuse töötlemine
AI vastus eraldatakse ja valideeritakse:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Järgmised sammud

Rohkem näidete jaoks vaata [4. peatükk: Praktilised näited](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Lahtiütlus**:
See dokument on tõlgitud kasutades AI tõlketeenust [Co-op Translator](https://github.com/Azure/co-op-translator). Kuigi me püüdleme täpsuse poole, palun pange tähele, et automatiseeritud tõlgetes võib esineda vigu või ebatäpsusi. Originaaldokument selle emakeeles tuleks pidada autoriteetseks allikaks. Olulise teabe puhul soovitatakse kasutada professionaalset inimtõlget. Me ei vastuta selle tõlkega seotud eksimustest või valesti mõistmistest.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->