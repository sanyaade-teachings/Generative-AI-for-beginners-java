# Tutorial Generator de Povestiri pentru Animale de Companie pentru Începutori

## Cuprins

- [Cerințe preliminare](#cerințe-preliminare)
- [Înțelegerea structurii proiectului](#înțelegerea-structurii-proiectului)
- [Componentele principale explicate](#componentele-principale-explicate)
  - [1. Aplicația principală](#1-aplicația-principală)
  - [2. Controller web](#2-controller-web)
  - [3. Serviciul de povestiri](#3-serviciul-de-povestiri)
  - [4. Șabloane web](#4-șabloane-web)
  - [5. Configurare](#5-configurare)
- [Rularea aplicației](#rularea-aplicației)
- [Cum funcționează totul împreună](#cum-funcționează-totul-împreună)
- [Înțelegerea integrării AI](#înțelegerea-integrării-ai)
- [Pașii următori](#pașii-următori)

## Cerințe preliminare

Înainte de a începe, asigurați-vă că aveți:
- Java 21 sau o versiune superioară instalată
- Maven pentru gestionarea dependențelor
- Un model Azure AI Foundry implementat (provisionați-l cu `azd up` — vedeți [Capitolul 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), autentificat cu `az login` (autentificare fără cheie)
- Cunoștințe de bază despre Java, Spring Boot și dezvoltare web

## Înțelegerea structurii proiectului

Proiectul de povestiri pentru animale de companie conține mai multe fișiere importante:

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

## Componentele principale explicate

### 1. Aplicația principală

**Fișier:** `PetStoryApplication.java`

Acesta este punctul de intrare pentru aplicația noastră Spring Boot:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Ce face acest fișier:**
- Anotarea `@SpringBootApplication` activează auto-configurarea și scanarea componentelor
- Pornește un server web încorporat (Tomcat) pe portul 8080
- Creează automat toate bean-urile și serviciile Spring necesare

### 2. Controller web

**Fișier:** `PetController.java`

Gestionează toate cererile web și interacțiunile utilizatorului:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Returnează șablonul index.html
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Validarea datelor de intrare
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Curățarea datelor de intrare pentru securitate
        String sanitizedDescription = sanitizeInput(description);
        
        // Generează povestea cu gestionarea erorilor
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Returnează șablonul result.html
            
        } catch (Exception e) {
            // Folosește povestea de rezervă dacă AI-ul eșuează
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Limitează lungimea
    }
}
```

**Caracteristici cheie:**

1. **Gestionarea rutelor**: `@GetMapping("/")` afișează formularul de upload, `@PostMapping("/generate-story")` procesează trimiterile
2. **Validarea inputului**: Verifică descrierile goale și limitele de lungime
3. **Securitate**: Curăță inputul utilizatorului pentru a preveni atacurile XSS
4. **Gestionarea erorilor**: Oferă povestiri alternative când serviciul AI eșuează
5. **Legarea modelului**: Transmite date către șabloanele HTML folosind `Model` din Spring

**Sistem de rezervă:**
Controller-ul include șabloane predefinite de povestiri care sunt folosite când serviciul AI nu este disponibil:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Folosește hash-ul descrierii pentru răspunsuri consistente
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Serviciul de povestiri

**Fișier:** `StoryService.java`

Acest serviciu comunică cu Azure AI Foundry pentru a genera povestiri folosind autentificare fără cheie:

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
        
        // Endpoint-ul compatibil cu OpenAI al Foundry se află sub /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Autentificare fără cheie cu Microsoft Entra ID (fără cheie API)
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
        
        // Configurează cererea AI
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Limitează lungimea răspunsului
                .temperature(0.8)          // Controlează creativitatea (0.0-1.0)
                .build();
        
        // Trimite cererea și primește răspunsul
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Componente cheie:**

1. **Client OpenAI**: Folosește SDK-ul oficial OpenAI pentru Java configurat pentru Azure AI Foundry (fără cheie)
2. **Prompt sistem**: Setează comportamentul AI pentru a scrie povestiri prietenoase pentru familie despre animale de companie
3. **Prompt utilizator**: Indică AI-ului exact ce poveste să scrie pe baza descrierii
4. **Parametri**: Controlează lungimea și nivelul de creativitate al poveștii
5. **Gestionarea erorilor**: Aruncă excepții pe care controller-ul le prinde și gestionează

### 4. Șabloane web

**Fișier:** `index.html` (formular de încărcare)

Pagina principală unde utilizatorii își descriu animalele de companie:

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

**Fișier:** `result.html` (afișare poveste)

Afișează povestea generată:

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

**Caracteristici ale șablonului:**

1. **Integrare Thymeleaf**: Folosește atribute `th:` pentru conținut dinamic
2. **Design responsiv**: Stiluri CSS pentru mobil și desktop
3. **Gestionarea erorilor**: Arată erori de validare utilizatorilor
4. **Procesare pe client**: JavaScript pentru analiză imagini (folosind Transformers.js)

### 5. Configurare

**Fișier:** `application.properties`

Setări de configurare pentru aplicație:

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

**Configurare explicată:**

1. **Încărcare fișiere**: Permite imagini de până la 10MB
2. **Logare**: Controlează ce informații sunt înregistrate în timpul execuției
3. **Azure AI Foundry**: Specifică endpoint-ul și modelul de implementat (autentificare fără cheie)
4. **Securitate**: Configurare pentru gestionarea erorilor, evitând expunerea unor informații sensibile

## Rularea aplicației

### Pasul 1: Autentificare și setarea endpoint-ului

Autentificarea este fără cheie (Microsoft Entra ID), deci nu există cheie API. Autentificați-vă și setați endpoint-ul Foundry:

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

**De ce este necesar acest pas:**
- Azure AI Foundry folosește Microsoft Entra ID pentru autentificarea cererilor de inferență
- Autentificarea fără cheie înseamnă că nu există secrete în codul sursă sau în mediul de rulare
- Contul dvs. trebuie să aibă rolul **Cognitive Services OpenAI User** pe resursă

### Pasul 2: Construirea și rularea

Navigați în directorul proiectului:
```bash
cd 04-PracticalSamples/petstory
```

Construiți aplicația:
```bash
mvn clean compile
```

Porniți serverul:
```bash
mvn spring-boot:run
```

Aplicația va porni la adresa `http://localhost:8080`.

### Pasul 3: Testați aplicația

1. **Deschideți** `http://localhost:8080` în browserul dvs.
2. **Descrieți** animalul dvs. de companie în zona de text (de ex., „Un golden retriever jucăuș care iubește să aducă mingea”)
3. **Apăsați** „Generate Story” pentru a primi o poveste generată de AI
4. **Alternativ**, încărcați o imagine a animalului pentru a genera automat o descriere
5. **Vizualizați** povestea creativă bazată pe descrierea oferită

## Cum funcționează totul împreună

Iată fluxul complet când generați o poveste despre animale de companie:

1. **Input utilizator**: Descrieți animalul dvs. pe formularul web
2. **Trimitere formular**: Browserul trimite o cerere POST la `/generate-story`
3. **Procesarea la controller**: `PetController` validează și curăță inputul
4. **Apel serviciu AI**: `StoryService` trimite cererea către modelul Azure AI Foundry
5. **Generare poveste**: AI generează o poveste creativă pe baza descrierii
6. **Gestionare răspuns**: Controller-ul primește povestea și o adaugă în model
7. **Randare șablon**: Thymeleaf generează `result.html` cu povestea
8. **Afișare**: Utilizatorul vede povestea generată în browser

**Flux de gestionare a erorilor:**
Dacă serviciul AI eșuează:
1. Controller-ul prinde excepția
2. Generează o poveste rezervă folosind șabloane predefinite
3. Afișează povestea de rezervă cu o mențiune despre indisponibilitatea AI-ului
4. Utilizatorul primește în continuare o poveste, asigurând o experiență bună

## Înțelegerea integrării AI

### Azure AI Foundry (fără cheie)
Aplicația folosește Azure AI Foundry cu autentificare fără cheie (Microsoft Entra ID):

```java
// Autentificare fără cheie - fără cheie API
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Ingineria promptului
Serviciul folosește prompturi atent construite pentru a obține rezultate bune:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Procesarea răspunsului
Răspunsul AI este extras și validat:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Pașii următori

Pentru mai multe exemple, vedeți [Capitolul 04: Exemple practice](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Declinare a responsabilității**:
Acest document a fost tradus folosind serviciul de traducere AI [Co-op Translator](https://github.com/Azure/co-op-translator). În timp ce ne străduim pentru acuratețe, vă rugăm să rețineți că traducerile automate pot conține erori sau inexactități. Documentul original în limba sa nativă trebuie considerat sursa autorizată. Pentru informații critice, se recomandă traducerea profesională realizată de un om. Nu ne asumăm responsabilitatea pentru eventualele neînțelegeri sau interpretări greșite care decurg din utilizarea acestei traduceri.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->