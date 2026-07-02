# Lemmikkitarinageneraattorin opas aloittelijoille

## Sisällysluettelo

- [Esivaatimukset](#esivaatimukset)
- [Projektirakenteen ymmärtäminen](#projektirakenteen-ymmärtäminen)
- [Keskeiset komponentit selitettynä](#keskeiset-komponentit-selitettynä)
  - [1. Pääsovellus](#1-pääsovellus)
  - [2. Web-kontrolleri](#2-web-kontrolleri)
  - [3. Tarinapalvelu](#3-tarinapalvelu)
  - [4. Web-mallit](#4-web-mallit)
  - [5. Konfiguraatio](#5-konfiguraatio)
- [Sovelluksen suorittaminen](#sovelluksen-suorittaminen)
- [Kuinka kaikki toimii yhdessä](#kuinka-kaikki-toimii-yhdessä)
- [AI-integraation ymmärtäminen](#ai-integraation-ymmärtäminen)
- [Seuraavat askeleet](#seuraavat-askeleet)

## Esivaatimukset

Ennen aloittamista varmista, että sinulla on:
- Java 21 tai uudempi asennettuna
- Maven riippuvuuksien hallintaan
- Azure AI Foundry -mallin käyttöönotto (provisionoi komennolla `azd up` — katso [Luku 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), kirjaudu sisään `az login` (avaimeton autentikointi)
- Perusteet Javasta, Spring Bootista ja web-kehityksestä

## Projektirakenteen ymmärtäminen

Lemmikkitarinaprojektissa on useita tärkeitä tiedostoja:

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

## Keskeiset komponentit selitettynä

### 1. Pääsovellus

**Tiedosto:** `PetStoryApplication.java`

Tämä on Spring Boot -sovelluksen pääsisäänkäynti:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Mitä tämä tekee:**
- `@SpringBootApplication`-annotaatio mahdollistaa automaattisen konfiguroinnin ja komponenttien skannauksen
- Käynnistää sisäisen web-palvelimen (Tomcat) portissa 8080
- Luo kaikki tarvittavat Spring-beanit ja -palvelut automaattisesti

### 2. Web-kontrolleri

**Tiedosto:** `PetController.java`

Tämä hoitaa kaikki web-pyynnöt ja käyttäjän vuorovaikutukset:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Palauttaa index.html-mallin
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Syötteen validointi
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Puhdista syöte turvallisuuden vuoksi
        String sanitizedDescription = sanitizeInput(description);
        
        // Luo tarina virheenkäsittelyllä
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Palauttaa result.html-mallin
            
        } catch (Exception e) {
            // Käytä varatarinaa, jos tekoäly epäonnistuu
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Rajoita pituus
    }
}
```

**Keskeiset ominaisuudet:**

1. **Reittien käsittely**: `@GetMapping("/")` näyttää latauslomakkeen, `@PostMapping("/generate-story")` käsittelee lähetykset
2. **Syötteen validointi**: Tarkistaa tyhjät kuvaukset ja pituusrajat
3. **Turvallisuus**: Siivoaa käyttäjän syötteen estääkseen XSS-hyökkäyksiä
4. **Virheenkäsittely**: Tarjoaa varatarinoita, jos AI-palvelu epäonnistuu
5. **Mallin sitominen**: Välittää dataa HTML-malleille Springin `Model`-objektin avulla

**Varajärjestelmä:**
Kontrolleri sisältää valmiiksi kirjoitettuja tarinamekintöjä, joita käytetään, kun AI-palvelu ei ole käytettävissä:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Käytä kuvaushashia johdonmukaisiin vastauksiin
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Tarinapalvelu

**Tiedosto:** `StoryService.java`

Tämä palvelu kommunikoi Azure AI Foundryn kanssa tuottaakseen tarinoita avaimettomalla autentikoinnilla:

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
        
        // Foundryn OpenAI-yhteensopiva päätepiste sijaitsee polussa /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Avaineton todennus Microsoft Entra ID:llä (ei API-avainta)
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
        
        // Määritä tekoälypyyntö
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Rajoita vastauspituus
                .temperature(0.8)          // Säädä luovuutta (0.0-1.0)
                .build();
        
        // Lähetä pyyntö ja vastaanota vastaus
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Keskeiset osat:**

1. **OpenAI-asiakas**: Käyttää virallista OpenAI Java SDK:ta, joka on konfiguroitu Azure AI Foundrylle (avaimeton)
2. **Järjestelmäkehotus**: Asettaa AI:n käyttäytymään niin, että se kirjoittaa perheystävällisiä lemmikkitarinoita
3. **Käyttäjäkehotus**: Kertoo AI:lle tarkalleen, minkä tarinan kirjoittaa kuvauksen perusteella
4. **Parametrit**: Ohjaa tarinan pituutta ja luovuustasoa
5. **Virheenkäsittely**: Heittää poikkeuksia, jotka kontrolleri nappaa ja käsittelee

### 4. Web-mallit

**Tiedosto:** `index.html` (Latauslomake)

Pääsivu, jossa käyttäjät kuvailevat lemmikkejään:

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

**Tiedosto:** `result.html` (Tarinan näyttö)

Näyttää generoidun tarinan:

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

**Mallin ominaisuudet:**

1. **Thymeleaf-integraatio**: Käyttää `th:`-attribuutteja dynaamiseen sisältöön
2. **Responsiivinen muotoilu**: CSS-tyylit mobiili- ja työpöytäkäyttöön
3. **Virheenkäsittely**: Näyttää validointivirheet käyttäjille
4. **Asiakaspään käsittely**: JavaScript kuvien analyysiin (Transformers.js-kirjastoa hyödyntäen)

### 5. Konfiguraatio

**Tiedosto:** `application.properties`

Sovelluksen konfiguraatioasetukset:

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

**Konfiguraation selitys:**

1. **Tiedostojen lataus**: Sallii korkeintaan 10MB kuvat
2. **Lokitus**: Määrittää, mitä tietoja kirjataan suorituksen aikana
3. **Azure AI Foundry**: Määrittää käytettävän päätepisteen ja mallin käyttöönoton (avaimeton autentikointi)
4. **Turvallisuus**: Virheenkäsittelyn asetukset, jotta arkaluontoisia tietoja ei vuoda

## Sovelluksen suorittaminen

### Vaihe 1: Kirjaudu sisään ja määritä päätepisteesi

Autentikointi on avaimetonta (Microsoft Entra ID), joten API-avainta ei tarvita. Kirjaudu sisään ja määritä Foundry-päätepisteesi:

**Windows (Komentokehote):**
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

**Miksi tämä tarvitaan:**
- Azure AI Foundry käyttää Microsoft Entra ID:tä autentikoimaan inference-pyynnöt
- Avaimeton autentikointi tarkoittaa, ettei salasanoja tai avaimia ole lähdekoodissa tai ympäristössä
- Tililläsi pitää olla **Cognitive Services OpenAI User** -rooli resurssissa

### Vaihe 2: Käännä ja suorita

Siirry projektihakemistoon:
```bash
cd 04-PracticalSamples/petstory
```

Käännä sovellus:
```bash
mvn clean compile
```

Käynnistä palvelin:
```bash
mvn spring-boot:run
```

Sovellus käynnistyy osoitteessa `http://localhost:8080`.

### Vaihe 3: Testaa sovellusta

1. **Avaa** `http://localhost:8080` selaimessasi
2. **Kuvaile** lemmikkiäsi tekstikentässä (esim. "Leikkisä kultainennoutaja, joka rakastaa noutamista")
3. **Klikkaa** "Generate Story" saadaksesi tekoälyn luoman tarinan
4. **Vaihtoehtoisesti** lataa lemmikkisi kuva, jolloin kuvaus generoidaan automaattisesti
5. **Katso** luova tarina lemmikkisi kuvauksen pohjalta

## Kuinka kaikki toimii yhdessä

Tässä on koko prosessi, kun luot lemmikkitarinan:

1. **Käyttäjän syöte**: Kuvailet lemmikkisi web-lomakkeella
2. **Lomakkeen lähetys**: Selain lähettää POST-pyynnön osoitteeseen `/generate-story`
3. **Kontrollerin käsittely**: `PetController` validoi ja puhdistaa syötteen
4. **AI-palvelun kutsu**: `StoryService` lähettää pyynnön Azure AI Foundry -malliin
5. **Tarinan luonti**: AI tuottaa luovan tarinan kuvauksen perusteella
6. **Vastauksen käsittely**: Kontrolleri vastaanottaa tarinan ja lisää sen malliin
7. **Mallin renderöinti**: Thymeleaf renderöi `result.html`-sivun tarinalla
8. **Näyttö**: Käyttäjä näkee generoidun tarinan selaimessa

**Virheenkäsittely:**
Jos AI-palvelu epäonnistuu:
1. Kontrolleri nappaa poikkeuksen
2. Tuottaa varatarinan valmiiden mallien avulla
3. Näyttää varatarinan ja huomautuksen AI-palvelun tilapäisestä poissaolosta
4. Käyttäjälle tarjotaan silti tarina, mikä takaa hyvän käyttökokemuksen

## AI-integraation ymmärtäminen

### Azure AI Foundry (avaimeton)
Sovellus käyttää Azure AI Foundryta avaimettomalla autentikoinnilla (Microsoft Entra ID):

```java
// Avaineton todennus - ei API-avainta
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Kehotetekniikat
Palvelu käyttää huolellisesti suunniteltuja kehotteita hyvien tulosten saamiseksi:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Vastauksen käsittely
AI:n vastaus poimitaan ja validoidaan:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Seuraavat askeleet

Lisää esimerkkejä löydät kohdasta [Luku 04: Käytännön esimerkit](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Vastuuvapauslauseke**:
Tämä asiakirja on käännetty käyttämällä tekoälypohjaista käännöspalvelua [Co-op Translator](https://github.com/Azure/co-op-translator). Vaikka pyrimme tarkkuuteen, otathan huomioon, että automaattiset käännökset saattavat sisältää virheitä tai epätarkkuuksia. Alkuperäinen asiakirja sen alkuperäiskielellä on virallinen lähde. Tärkeissä asioissa suositellaan ammattimaista ihmiskäännöstä. Emme ole vastuussa tämän käännöksen käytöstä aiheutuvista väärinymmärryksistä tai tulkinnoista.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->