# Pet Story Generator Tutorial für Anfänger

## Inhaltsverzeichnis

- [Voraussetzungen](#voraussetzungen)
- [Verstehen der Projektstruktur](#verstehen-der-projektstruktur)
- [Erklärung der Kernkomponenten](#erklärung-der-kernkomponenten)
  - [1. Hauptanwendung](#1-hauptanwendung)
  - [2. Web-Controller](#2-web-controller)
  - [3. Story-Service](#3-story-service)
  - [4. Web-Vorlagen](#4-web-vorlagen)
  - [5. Konfiguration](#5-konfiguration)
- [Ausführen der Anwendung](#ausführen-der-anwendung)
- [Wie alles zusammen funktioniert](#wie-alles-zusammen-funktioniert)
- [Verstehen der KI-Integration](#verstehen-der-ki-integration)
- [Nächste Schritte](#nächste-schritte)

## Voraussetzungen

Bevor Sie beginnen, stellen Sie sicher, dass Sie Folgendes haben:
- Java 21 oder höher installiert
- Maven für das Abhängigkeitsmanagement
- Eine Azure AI Foundry Modellbereitstellung (stellen Sie sie mit `azd up` bereit — siehe [Kapitel 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), angemeldet mit `az login` (schlüsselose Authentifizierung)
- Grundkenntnisse in Java, Spring Boot und Webentwicklung

## Verstehen der Projektstruktur

Das Haustiergeschichten-Projekt enthält mehrere wichtige Dateien:

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

## Erklärung der Kernkomponenten

### 1. Hauptanwendung

**Datei:** `PetStoryApplication.java`

Dies ist der Einstiegspunkt für unsere Spring Boot Anwendung:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Was dies bewirkt:**
- Die Annotation `@SpringBootApplication` aktiviert Auto-Konfiguration und Komponentenscanning
- Startet einen eingebetteten Webserver (Tomcat) auf Port 8080
- Erstellt automatisch alle notwendigen Spring Beans und Services

### 2. Web-Controller

**Datei:** `PetController.java`

Dieser verarbeitet alle Webanfragen und Benutzerinteraktionen:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Gibt die index.html Vorlage zurück
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Eingabvalidierung
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Eingabe für Sicherheit bereinigen
        String sanitizedDescription = sanitizeInput(description);
        
        // Geschichte mit Fehlerbehandlung generieren
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Gibt die result.html Vorlage zurück
            
        } catch (Exception e) {
            // Verwende Ersatzgeschichte, falls KI fehlschlägt
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Länge begrenzen
    }
}
```

**Wichtige Funktionen:**

1. **Routenverwaltung**: `@GetMapping("/")` zeigt das Upload-Formular, `@PostMapping("/generate-story")` verarbeitet Einsendungen
2. **Eingabevalidierung**: Prüft auf leere Beschreibungen und Längenbeschränkungen
3. **Sicherheit**: Reinigt Benutzereingaben zur Vermeidung von XSS-Angriffen
4. **Fehlerbehandlung**: Liefert Ersatzgeschichten, wenn der KI-Dienst ausfällt
5. **Modellbindung**: Übergibt Daten mithilfe von Springs `Model` an HTML-Vorlagen

**Fallback-System:**
Der Controller enthält vorgefertigte Geschichtenvorlagen, die verwendet werden, wenn der KI-Dienst nicht verfügbar ist:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Verwenden Sie den Beschreibungshash für konsistente Antworten
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Story-Service

**Datei:** `StoryService.java`

Dieser Service kommuniziert mit Azure AI Foundry, um Geschichten mit schlüsselloser Authentifizierung zu generieren:

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
        
        // Der OpenAI-kompatible Endpunkt von Foundry befindet sich unter /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Schlüsselose Authentifizierung mit Microsoft Entra ID (kein API-Schlüssel)
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
        
        // Die KI-Anfrage konfigurieren
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Antwortlänge begrenzen
                .temperature(0.8)          // Kreativität steuern (0.0-1.0)
                .build();
        
        // Anfrage senden und Antwort erhalten
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Hauptbestandteile:**

1. **OpenAI Client**: Verwendet das offizielle OpenAI Java SDK, konfiguriert für Azure AI Foundry (schlüssellos)
2. **System-Prompt**: Bestimmt das Verhalten der KI, um familienfreundliche Haustiergeschichten zu schreiben
3. **User-Prompt**: Sagt der KI genau, welche Geschichte basierend auf der Beschreibung zu schreiben ist
4. **Parameter**: Kontrolliert die Länge der Geschichte und den Kreativitätsgrad
5. **Fehlerbehandlung**: Löst Ausnahmen aus, die der Controller erfasst und behandelt

### 4. Web-Vorlagen

**Datei:** `index.html` (Upload-Formular)

Die Hauptseite, auf der Benutzer ihre Haustiere beschreiben:

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

**Datei:** `result.html` (Geschichtendarstellung)

Zeigt die generierte Geschichte an:

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

**Vorlagenfunktionen:**

1. **Thymeleaf-Integration**: Verwendet `th:`-Attribute für dynamische Inhalte
2. **Responsives Design**: CSS-Styles für Mobilgeräte und Desktop
3. **Fehlerbehandlung**: Zeigt Validierungsfehler für Benutzer an
4. **Clientseitige Verarbeitung**: JavaScript für Bildanalyse (mit Transformers.js)

### 5. Konfiguration

**Datei:** `application.properties`

Konfigurationseinstellungen für die Anwendung:

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

**Erklärung der Konfiguration:**

1. **Datei-Upload**: Erlaubt Bilder bis zu 10MB
2. **Logging**: Steuert welche Informationen während der Ausführung geloggt werden
3. **Azure AI Foundry**: Gibt den Endpunkt und die Modellbereitstellung an (schlüssellose Authentifizierung)
4. **Sicherheit**: Fehlerbehandlungskonfiguration, um das Offenlegen sensibler Informationen zu vermeiden

## Ausführen der Anwendung

### Schritt 1: Anmelden und Endpunkt setzen

Die Authentifizierung ist schlüssellos (Microsoft Entra ID), daher gibt es keinen API-Schlüssel. Melden Sie sich an und setzen Sie Ihren Foundry-Endpunkt:

**Windows (Eingabeaufforderung):**
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

**Warum das notwendig ist:**
- Azure AI Foundry verwendet Microsoft Entra ID zur Authentifizierung von Inferenzanfragen
- Schlüssellose Authentifizierung bedeutet keine Geheimnisse im Quellcode oder in der Umgebung
- Ihr Konto benötigt die **Cognitive Services OpenAI User** Rolle für die Ressource

### Schritt 2: Bauen und Starten

Navigieren Sie in das Projektverzeichnis:
```bash
cd 04-PracticalSamples/petstory
```

Bauen Sie die Anwendung:
```bash
mvn clean compile
```

Starten Sie den Server:
```bash
mvn spring-boot:run
```

Die Anwendung startet unter `http://localhost:8080`.

### Schritt 3: Anwendung testen

1. **Öffnen** Sie `http://localhost:8080` in Ihrem Browser
2. **Beschreiben** Sie Ihr Haustier im Textfeld (z. B. "Ein verspielter Golden Retriever, der gerne apportiert")
3. **Klicken** Sie auf "Geschichte generieren", um eine KI-generierte Geschichte zu erhalten
4. **Alternativ** laden Sie ein Bild Ihres Haustiers hoch, um automatisch eine Beschreibung zu erstellen
5. **Betrachten** Sie die kreative Geschichte basierend auf der Beschreibung Ihres Haustiers

## Wie alles zusammen funktioniert

Hier ist der komplette Ablauf, wenn Sie eine Haustiergeschichte generieren:

1. **Benutzereingabe**: Sie beschreiben Ihr Haustier im Webformular
2. **Formularabsendung**: Der Browser sendet eine POST-Anfrage an `/generate-story`
3. **Controller-Verarbeitung**: `PetController` validiert und reinigt die Eingabe
4. **KI-Dienst-Aufruf**: `StoryService` sendet eine Anfrage an das Azure AI Foundry Modell
5. **Geschichtenerstellung**: Die KI erstellt eine kreative Geschichte basierend auf der Beschreibung
6. **Antwortverarbeitung**: Der Controller erhält die Geschichte und fügt sie dem Modell hinzu
7. **Vorlagendarstellung**: Thymeleaf rendert `result.html` mit der Geschichte
8. **Anzeige**: Der Benutzer sieht die generierte Geschichte im Browser

**Fehlerbehandlungsablauf:**
Falls der KI-Dienst ausfällt:
1. Fängt der Controller die Ausnahme auf
2. Erzeugt eine Ersatzgeschichte mit vorgefertigten Vorlagen
3. Zeigt die Ersatzgeschichte mit einem Hinweis zur Nichtverfügbarkeit der KI an
4. Der Benutzer erhält trotzdem eine Geschichte, was die Benutzererfahrung verbessert

## Verstehen der KI-Integration

### Azure AI Foundry (schlüssellos)
Die Anwendung verwendet Azure AI Foundry mit schlüsselloser Authentifizierung (Microsoft Entra ID):

```java
// Authentifizierung ohne Schlüssel - kein API-Schlüssel
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Prompt Engineering
Der Service nutzt sorgfältig gestaltete Prompts, um gute Ergebnisse zu erzielen:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Antwortverarbeitung
Die KI-Antwort wird extrahiert und validiert:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Nächste Schritte

Für weitere Beispiele siehe [Kapitel 04: Praktische Beispiele](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Haftungsausschluss**:
Dieses Dokument wurde mit dem KI-Übersetzungsdienst [Co-op Translator](https://github.com/Azure/co-op-translator) übersetzt. Obwohl wir uns um Genauigkeit bemühen, beachten Sie bitte, dass automatisierte Übersetzungen Fehler oder Ungenauigkeiten enthalten können. Das Originaldokument in seiner Ursprungssprache gilt als maßgebliche Quelle. Bei kritischen Informationen wird eine professionelle menschliche Übersetzung empfohlen. Wir übernehmen keine Haftung für Missverständnisse oder Fehlinterpretationen, die aus der Verwendung dieser Übersetzung entstehen.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->