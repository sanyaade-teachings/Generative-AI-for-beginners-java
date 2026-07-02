# Návod na generovanie príbehu o zvieratku pre začiatočníkov

## Obsah

- [Predpoklady](#prerekvizity)
- [Pochopenie štruktúry projektu](#pochopenie-štruktúry-projektu)
- [Vysvetlenie hlavných komponentov](#vysvetlenie-hlavných-komponentov)
  - [1. Hlavná aplikácia](#1-hlavná-aplikácia)
  - [2. Webový kontrolér](#2-webový-kontrolér)
  - [3. Servis príbehov](#3-servis-príbehov)
  - [4. Webové šablóny](#4-webové-šablóny)
  - [5. Konfigurácia](#5-konfigurácia)
- [Spustenie aplikácie](#spustenie-aplikácie)
- [Ako to všetko spolu funguje](#ako-to-všetko-spolu-funguje)
- [Pochopenie integrácie AI](#pochopenie-integrácie-ai)
- [Ďalšie kroky](#ďalšie-kroky)

## Prerekvizity

Pred začatím sa uistite, že máte:
- Nainštalované Java 21 alebo novšiu verziu
- Maven na správu závislostí
- Nasadenie modelu Azure AI Foundry (zabezpečené pomocou `azd up` — pozri [Kapitolu 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), prihlásenie cez `az login` (autentifikácia bez kľúča)
- Základné znalosti Java, Spring Boot a webového vývoja

## Pochopenie štruktúry projektu

Projekt príbehu o zvieratku obsahuje niekoľko dôležitých súborov:

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

## Vysvetlenie hlavných komponentov

### 1. Hlavná aplikácia

**Súbor:** `PetStoryApplication.java`

Toto je vstupný bod našej aplikácie Spring Boot:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Čo to robí:**
- Anotácia `@SpringBootApplication` umožňuje automatickú konfiguráciu a skenovanie komponentov
- Spúšťa zabudovaný webový server (Tomcat) na porte 8080
- Automaticky vytvára všetky potrebné Spring bean-y a služby

### 2. Webový kontrolér

**Súbor:** `PetController.java`

Zodpovedá za všetky webové požiadavky a interakciu používateľa:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Vracia šablónu index.html
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Validácia vstupu
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Sanitizovať vstup pre bezpečnosť
        String sanitizedDescription = sanitizeInput(description);
        
        // Generovať príbeh s ošetrením chýb
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Vracia šablónu result.html
            
        } catch (Exception e) {
            // Použiť náhradný príbeh, ak AI zlyhá
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Obmedziť dĺžku
    }
}
```

**Kľúčové vlastnosti:**

1. **Spracovanie trás:** `@GetMapping("/")` zobrazuje formulár na nahranie, `@PostMapping("/generate-story")` spracováva odoslania
2. **Validácia vstupu:** Kontroluje prázdne popisy a dĺžkové limity
3. **Bezpečnosť:** Sanitizuje vstup používateľa na prevenciu XSS útokov
4. **Spracovanie chýb:** Poskytuje náhradné príbehy, ak AI služba zlyhá
5. **Väzba modelu:** Posiela dáta do HTML šablón pomocou Spring `Model`

**Náhradný systém:**
Kontrolér obsahuje predpripravené šablóny príbehov, ktoré sa používajú, keď AI služba nie je dostupná:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Použite hash popisu pre konzistentné odpovede
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Servis príbehov

**Súbor:** `StoryService.java`

Táto služba komunikuje s Azure AI Foundry a generuje príbehy pomocou autentifikácie bez kľúča:

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
        
        // Endpoint kompatibilný s OpenAI vo Foundry sa nachádza pod /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Overenie bez kľúča pomocou Microsoft Entra ID (žiadny API kľúč)
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
        
        // Nakonfigurujte požiadavku AI
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Obmedziť dĺžku odpovede
                .temperature(0.8)          // Ovládajte kreativitu (0.0-1.0)
                .build();
        
        // Odoslať požiadavku a získať odpoveď
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Hlavné komponenty:**

1. **OpenAI klient:** Používa oficiálne OpenAI Java SDK nakonfigurované pre Azure AI Foundry (bez kľúča)
2. **Systémový prompt:** Nastavuje správanie AI na písanie priateľských rodinných príbehov o zvieratkách
3. **Používateľský prompt:** Povedá AI presne, aký príbeh má napísať na základe popisu
4. **Parametre:** Riadi dĺžku príbehu a úroveň kreativity
5. **Spracovanie chýb:** Vyhadzuje výnimky, ktoré kontrolér zachytáva a spracováva

### 4. Webové šablóny

**Súbor:** `index.html` (Formulár na nahratie)

Hlavná stránka, kde používatelia opisujú svoje zvieratká:

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

**Súbor:** `result.html` (Zobrazenie príbehu)

Zobrazuje vygenerovaný príbeh:

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

**Vlastnosti šablóny:**

1. **Integrácia s Thymeleaf:** Používa atribúty `th:` pre dynamický obsah
2. **Responzívny dizajn:** CSS pre mobilné a desktopové zariadenia
3. **Spracovanie chýb:** Zobrazuje používateľom validačné chyby
4. **Spracovanie na klientovi:** JavaScript pre analýzu obrázkov (používajúc Transformers.js)

### 5. Konfigurácia

**Súbor:** `application.properties`

Nastavenia konfigurácie aplikácie:

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

**Vysvetlenie konfigurácie:**

1. **Nahrávanie súborov:** Povolené obrázky do veľkosti 10 MB
2. **Zaznamenávanie:** Riadi, aké informácie sa logujú počas behu
3. **Azure AI Foundry:** Určuje endpoint a nasadenie modelu na použitie (autentifikácia bez kľúča)
4. **Bezpečnosť:** Nastavenie spracovania chýb, aby sa neodhalili citlivé informácie

## Spustenie aplikácie

### Krok 1: Prihlásenie a nastavenie endpointu

Autentifikácia je bezkľúčová (Microsoft Entra ID), takže API kľúč nie je potrebný. Prihláste sa a nastavte si Foundry endpoint:

**Windows (Príkazový riadok):**
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

**Prečo je to potrebné:**
- Azure AI Foundry používa Microsoft Entra ID na autentifikáciu inference požiadaviek
- Autentifikácia bez kľúča znamená, že v kóde ani prostredí nie sú uložené žiadne tajomstvá
- Váš účet potrebuje rolu **Cognitive Services OpenAI User** na príslušnom zdroji

### Krok 2: Vytvorenie a spustenie

Prejdite do priečinka projektu:
```bash
cd 04-PracticalSamples/petstory
```

Zostavte aplikáciu:
```bash
mvn clean compile
```

Spustite server:
```bash
mvn spring-boot:run
```

Aplikácia sa spustí na `http://localhost:8080`.

### Krok 3: Otestujte aplikáciu

1. **Otvorte** `http://localhost:8080` vo vašom prehliadači
2. **Opíšte** svoje zvieratko v textovom poli (napr. "Hravo zlatý retríver, ktorý rád nosí loptičku")
3. **Kliknite** na "Generate Story" pre získanie AI-generovaného príbehu
4. **Alebo** nahrajte obrázok zvieratka a automaticky vygenerujte jeho popis
5. **Prezrite si** kreatívny príbeh na základe opisu vášho zvieratka

## Ako to všetko spolu funguje

Tu je kompletný proces, keď generujete príbeh o zvieratku:

1. **Vstup používateľa:** Opisujete svoje zvieratko vo webovom formulári
2. **Odoslanie formulára:** Prehliadač posiela POST požiadavku na `/generate-story`
3. **Spracovanie kontrolérom:** `PetController` overuje a sanitizuje vstup
4. **Volanie AI služby:** `StoryService` posiela požiadavku modelu Azure AI Foundry
5. **Generovanie príbehu:** AI vytvorí kreatívny príbeh na základe opisu
6. **Spracovanie odpovede:** Kontrolér prijme príbeh a pridá ho do modelu
7. **Vykreslenie šablóny:** Thymeleaf vykreslí `result.html` so scénou príbehu
8. **Zobrazenie:** Používateľ vidí vygenerovaný príbeh vo svojom prehliadači

**Spracovanie chýb:**
Ak AI služba zlyhá:
1. Kontrolér zachytí výnimku
2. Vytvorí náhradný príbeh pomocou predpripravených šablón
3. Zobrazí náhradný príbeh s upozornením o nedostupnosti AI
4. Používateľ stále dostane príbeh, čím sa zabezpečí dobrý používateľský zážitok

## Pochopenie integrácie AI

### Azure AI Foundry (bez kľúča)
Aplikácia používa Azure AI Foundry s autentifikáciou bez kľúča (Microsoft Entra ID):

```java
// Autentifikácia bez kľúča - žiadny API kľúč
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Návrh promptov
Služba používa starostlivo vytvorené prompty, aby získala dobré výsledky:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Spracovanie odpovede
Odpoveď AI sa extrahuje a validuje:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Ďalšie kroky

Pre viac príkladov pozrite [Kapitolu 04: Praktické ukážky](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Vyhlásenie o zodpovednosti**:
Tento dokument bol preložený pomocou AI prekladateľskej služby [Co-op Translator](https://github.com/Azure/co-op-translator). Hoci sa snažíme o presnosť, vezmite prosím na vedomie, že automatické preklady môžu obsahovať chyby alebo nepresnosti. Pôvodný dokument v jeho natívnom jazyku by mal byť považovaný za autoritatívny zdroj. Pre kritické informácie sa odporúča profesionálny ľudský preklad. Nie sme zodpovední za žiadne nedorozumenia alebo nesprávne interpretácie vyplývajúce z použitia tohto prekladu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->