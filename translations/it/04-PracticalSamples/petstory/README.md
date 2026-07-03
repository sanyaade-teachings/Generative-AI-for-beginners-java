# Guida al Generatore di Storie di Animali Domestici per Principianti

## Indice

- [Prerequisiti](#prerequisiti)
- [Comprendere la Struttura del Progetto](#comprendere-la-struttura-del-progetto)
- [Componenti Principali Spiegati](#componenti-principali-spiegati)
  - [1. Applicazione Principale](#1-applicazione-principale)
  - [2. Controller Web](#2-controller-web)
  - [3. Servizio Storie](#3-servizio-storie)
  - [4. Template Web](#4-template-web)
  - [5. Configurazione](#5-configurazione)
- [Esecuzione dell'Applicazione](#esecuzione-dellapplicazione)
- [Come Funziona Tutto Insieme](#come-funziona-tutto-insieme)
- [Comprendere l'Integrazione AI](#comprendere-lintegrazione-ai)
- [Passi Successivi](#passi-successivi)

## Prerequisiti

Prima di iniziare, assicurati di avere:
- Java 21 o versioni superiori installato
- Maven per la gestione delle dipendenze
- Un modello Azure AI Foundry distribuito (configuralo con `azd up` — vedi [Capitolo 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), autenticato con `az login` (autenticazione senza chiave)
- Comprensione base di Java, Spring Boot e sviluppo web

## Comprendere la Struttura del Progetto

Il progetto per la storia sugli animali domestici ha diversi file importanti:

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

## Componenti Principali Spiegati

### 1. Applicazione Principale

**File:** `PetStoryApplication.java`

Questo è il punto di ingresso della nostra applicazione Spring Boot:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Cosa fa:**
- L'annotazione `@SpringBootApplication` abilita l'auto-configurazione e la scansione dei componenti
- Avvia un server web embedded (Tomcat) sulla porta 8080
- Crea automaticamente tutti i bean e servizi necessari di Spring

### 2. Controller Web

**File:** `PetController.java`

Gestisce tutte le richieste web e le interazioni utente:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Restituisce il template index.html
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Validazione dell'input
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Sanitizza l'input per la sicurezza
        String sanitizedDescription = sanitizeInput(description);
        
        // Genera la storia con gestione degli errori
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Restituisce il template result.html
            
        } catch (Exception e) {
            // Usa una storia di riserva se l'IA fallisce
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Limita la lunghezza
    }
}
```

**Caratteristiche principali:**

1. **Gestione delle rotte**: `@GetMapping("/")` mostra il modulo di upload, `@PostMapping("/generate-story")` elabora le sottomissioni
2. **Validazione Input**: Controlla descrizioni vuote e limiti di lunghezza
3. **Sicurezza**: Sanifica l'input utente per prevenire attacchi XSS
4. **Gestione Errori**: Fornisce storie alternative quando il servizio AI fallisce
5. **Binding Modello**: Passa i dati ai template HTML usando il `Model` di Spring

**Sistema di fallback:**
Il controller include template di storie pre-scritte che vengono usate quando il servizio AI non è disponibile:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Usa l'hash della descrizione per risposte coerenti
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Servizio Storie

**File:** `StoryService.java`

Questo servizio comunica con Azure AI Foundry per generare storie usando l'autenticazione senza chiave:

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
        
        // L'endpoint compatibile con OpenAI di Foundry si trova sotto /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Autenticazione senza chiave con Microsoft Entra ID (nessuna chiave API)
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
        
        // Configura la richiesta AI
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Limita la lunghezza della risposta
                .temperature(0.8)          // Controlla la creatività (0.0-1.0)
                .build();
        
        // Invia la richiesta e ottieni la risposta
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Componenti chiave:**

1. **Client OpenAI**: Usa l'SDK ufficiale OpenAI Java configurato per Azure AI Foundry (keyless)
2. **Prompt di sistema**: Imposta il comportamento dell'AI per scrivere storie di animali domestici adatte alle famiglie
3. **Prompt utente**: Indica all'AI esattamente quale storia scrivere basandosi sulla descrizione
4. **Parametri**: Controlla la lunghezza della storia e il livello di creatività
5. **Gestione errori**: Lancia eccezioni che il controller cattura e gestisce

### 4. Template Web

**File:** `index.html` (Modulo di caricamento)

La pagina principale dove gli utenti descrivono i loro animali:

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

**File:** `result.html` (Visualizzazione della storia)

Mostra la storia generata:

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

**Caratteristiche del template:**

1. **Integrazione Thymeleaf**: Usa attributi `th:` per contenuto dinamico
2. **Design responsive**: Stile CSS per dispositivi mobili e desktop
3. **Gestione errori**: Mostra agli utenti errori di validazione
4. **Elaborazione client-side**: JavaScript per analisi immagini (con Transformers.js)

### 5. Configurazione

**File:** `application.properties`

Impostazioni di configurazione per l'applicazione:

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

**Spiegazione della configurazione:**

1. **Upload file**: Consente immagini fino a 10MB
2. **Logging**: Controlla quali informazioni vengono loggate durante l'esecuzione
3. **Azure AI Foundry**: Specifica endpoint e modello da utilizzare (autenticazione senza chiave)
4. **Sicurezza**: Configurazione per la gestione degli errori in modo da non esporre informazioni sensibili

## Esecuzione dell'Applicazione

### Passo 1: Accedi e Imposta il Tuo Endpoint

L'autenticazione è senza chiave (Microsoft Entra ID), quindi non c'è nessuna chiave API. Accedi e imposta il tuo endpoint Foundry:

**Windows (Prompt dei comandi):**
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

**Perché è necessario:**
- Azure AI Foundry usa Microsoft Entra ID per autenticare le richieste di inferenza
- L'autenticazione keyless significa niente segreti nel codice sorgente o nell'ambiente
- Il tuo account deve avere il ruolo **Cognitive Services OpenAI User** sulla risorsa

### Passo 2: Compila ed Esegui

Spostati nella directory del progetto:
```bash
cd 04-PracticalSamples/petstory
```

Compila l'applicazione:
```bash
mvn clean compile
```

Avvia il server:
```bash
mvn spring-boot:run
```

L'applicazione partirà su `http://localhost:8080`.

### Passo 3: Testa l'Applicazione

1. **Apri** `http://localhost:8080` nel browser
2. **Descrivi** il tuo animale domestico nell'area di testo (es. "Un golden retriever giocoso che ama riportare la palla")
3. **Clicca** su "Generate Story" per ottenere una storia generata dall'AI
4. **In alternativa**, carica un'immagine del tuo animale per generare automaticamente una descrizione
5. **Visualizza** la storia creativa generata basata sulla descrizione del tuo animale

## Come Funziona Tutto Insieme

Ecco il flusso completo quando generi una storia di animali:

1. **Input Utente**: Descrivi il tuo animale nel modulo web
2. **Invio Modulo**: Il browser invia una richiesta POST a `/generate-story`
3. **Elaborazione dal Controller**: `PetController` valida e sanifica l'input
4. **Chiamata al Servizio AI**: `StoryService` invia la richiesta al modello Azure AI Foundry
5. **Generazione Storia**: L'AI genera una storia creativa basata sulla descrizione
6. **Gestione Risposta**: Il controller riceve la storia e la aggiunge al modello
7. **Rendering Template**: Thymeleaf rende `result.html` con la storia
8. **Visualizzazione**: L'utente vede la storia generata nel browser

**Flusso di Gestione Errori:**
Se il servizio AI fallisce:
1. Il controller cattura l'eccezione
2. Genera una storia di fallback usando template pre-scritti
3. Mostra la storia di fallback con una nota sull'indisponibilità dell'AI
4. L'utente riceve comunque una storia, garantendo una buona esperienza

## Comprendere l'Integrazione AI

### Azure AI Foundry (senza chiave)
L'applicazione usa Azure AI Foundry con autenticazione senza chiave (Microsoft Entra ID):

```java
// Autenticazione senza chiave - nessuna chiave API
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Ingegneria del Prompt
Il servizio usa prompt creati con cura per ottenere buoni risultati:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Elaborazione della Risposta
La risposta AI è estratta e validata:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Passi Successivi

Per ulteriori esempi, vedi [Capitolo 04: Esempi pratici](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Disclaimer**:
Questo documento è stato tradotto utilizzando il servizio di traduzione AI [Co-op Translator](https://github.com/Azure/co-op-translator). Sebbene ci impegniamo per garantire la precisione, si prega di notare che le traduzioni automatizzate possono contenere errori o imprecisioni. Il documento originale nella sua lingua nativa deve essere considerato la fonte autorevole. Per informazioni critiche, si raccomanda una traduzione professionale effettuata da un essere umano. Non siamo responsabili per eventuali malintesi o interpretazioni errate derivanti dall’uso di questa traduzione.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->