# Samouczek Generatora Opowieści o Zwierzakach dla Początkujących

## Spis Treści

- [Wymagania Wstępne](#wymagania-wstępne)
- [Zrozumienie Struktury Projektu](#zrozumienie-struktury-projektu)
- [Omówienie Głównych Komponentów](#omówienie-głównych-komponentów)
  - [1. Główna Aplikacja](#1-główna-aplikacja)
  - [2. Kontroler WWW](#2-kontroler-www)
  - [3. Serwis Opowieści](#3-serwis-opowieści)
  - [4. Szablony WWW](#4-szablony-www)
  - [5. Konfiguracja](#5-konfiguracja)
- [Uruchamianie Aplikacji](#uruchamianie-aplikacji)
- [Jak To Wszystko Działa Razem](#jak-to-wszystko-działa-razem)
- [Zrozumienie Integracji AI](#zrozumienie-integracji-ai)
- [Kolejne Kroki](#kolejne-kroki)

## Wymagania Wstępne

Przed rozpoczęciem upewnij się, że masz:
- Zainstalowaną Javę 21 lub nowszą
- Maven do zarządzania zależnościami
- Wdrożenie modelu Azure AI Foundry (uruchom `azd up` — zobacz [Rozdział 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), zalogowany za pomocą `az login` (uwierzytelnianie bezkluczowe)
- Podstawową znajomość Java, Spring Boot i tworzenia aplikacji webowych

## Zrozumienie Struktury Projektu

Projekt opowieści o zwierzakach zawiera kilka ważnych plików:

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

## Omówienie Głównych Komponentów

### 1. Główna Aplikacja

**Plik:** `PetStoryApplication.java`

To punkt startowy naszej aplikacji Spring Boot:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Co to robi:**
- Adnotacja `@SpringBootApplication` włącza auto-konfigurację i skanowanie komponentów
- Uruchamia wbudowany serwer webowy (Tomcat) na porcie 8080
- Automatycznie tworzy wszystkie potrzebne beany i serwisy Springa

### 2. Kontroler WWW

**Plik:** `PetController.java`

Obsługuje wszystkie żądania webowe i interakcje użytkownika:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Zwraca szablon index.html
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Walidacja danych wejściowych
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Oczyszczanie danych wejściowych dla bezpieczeństwa
        String sanitizedDescription = sanitizeInput(description);
        
        // Generowanie opowieści z obsługą błędów
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Zwraca szablon result.html
            
        } catch (Exception e) {
            // Użyj zapasowej opowieści, jeśli AI zawiedzie
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Ogranicz długość
    }
}
```

**Kluczowe cechy:**

1. **Obsługa Ścieżek:** `@GetMapping("/")` wyświetla formularz przesyłania, `@PostMapping("/generate-story")` przetwarza zgłoszenia
2. **Walidacja Wprowadzanych Danych:** Sprawdza, czy opis nie jest pusty i czy nie przekracza limitu długości
3. **Bezpieczeństwo:** Oczyszcza dane wejściowe, aby zapobiec atakom XSS
4. **Obsługa Błędów:** Dostarcza zapasowe historie, gdy usługa AI jest niedostępna
5. **Mapowanie Modelu:** Przekazuje dane do szablonów HTML przy pomocy Springowego `Model`

**System Zapasowy:**
Kontroler zawiera gotowe szablony opowieści, które są wykorzystywane, gdy usługa AI jest niedostępna:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Użyj skrótu opisu dla spójnych odpowiedzi
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Serwis Opowieści

**Plik:** `StoryService.java`

Ten serwis komunikuje się z Azure AI Foundry, generując opowieści przy uwierzytelnianiu bezkluczowym:

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
        
        // Punkt końcowy kompatybilny z OpenAI Foundry znajduje się pod /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Uwierzytelnianie bez klucza z Microsoft Entra ID (bez klucza API)
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
        
        // Skonfiguruj zapytanie AI
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Ogranicz długość odpowiedzi
                .temperature(0.8)          // Kontroluj kreatywność (0.0-1.0)
                .build();
        
        // Wyślij zapytanie i otrzymaj odpowiedź
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Kluczowe składniki:**

1. **Klient OpenAI:** Korzysta z oficjalnego SDK OpenAI dla Javy skonfigurowanego pod Azure AI Foundry (bezkluczowe uwierzytelnianie)
2. **Systemowy Prompt:** Ustawia zachowanie AI, aby pisało przyjazne rodzinie opowieści o zwierzakach
3. **Prompt Użytkownika:** Precyzuje AI, jaką historię ma napisać na podstawie opisu
4. **Parametry:** Kontrolują długość opowieści i poziom kreatywności
5. **Obsługa Błędów:** Rzuca wyjątki, które są przechwytywane i obsługiwane przez kontroler

### 4. Szablony WWW

**Plik:** `index.html` (Formularz Przesyłania)

Główna strona, gdzie użytkownicy opisują swoje zwierzaki:

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

**Plik:** `result.html` (Wyświetlanie Historii)

Pokazuje wygenerowaną opowieść:

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

**Cechy szablonu:**

1. **Integracja Thymeleaf:** Używa atrybutów `th:` dla dynamicznej zawartości
2. **Responsywny Design:** Stylizacja CSS dla urządzeń mobilnych i desktopów
3. **Obsługa Błędów:** Wyświetla użytkownikom informacje o błędach walidacji
4. **Przetwarzanie Po Stronie Klienta:** JavaScript do analizy obrazów (z użyciem Transformers.js)

### 5. Konfiguracja

**Plik:** `application.properties`

Ustawienia konfiguracyjne aplikacji:

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

**Wyjaśnienie konfiguracji:**

1. **Przesyłanie Plików:** Pozwala na przesyłanie obrazów do 10 MB
2. **Logowanie:** Kontroluje jakie informacje są logowane podczas działania
3. **Azure AI Foundry:** Określa punkt końcowy i używane wdrożenie modelu (uwierzytelnianie bezkluczowe)
4. **Bezpieczeństwo:** Konfiguracja obsługi błędów, by nie ujawniać wrażliwych informacji

## Uruchamianie Aplikacji

### Krok 1: Zaloguj Się i Ustaw Swój Punkt Końcowy

Uwierzytelnianie jest bezkluczowe (Microsoft Entra ID), więc nie ma klucza API. Zaloguj się i ustaw punkt końcowy Foundry:

**Windows (Wiersz Poleceń):**
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

**Dlaczego to jest potrzebne:**
- Azure AI Foundry używa Microsoft Entra ID do uwierzytelniania żądań inferencji
- Uwierzytelnianie bezkluczowe oznacza brak sekretów w kodzie źródłowym lub środowisku
- Twoje konto musi mieć rolę **Cognitive Services OpenAI User** na zasobie

### Krok 2: Buduj i Uruchom

Przejdź do katalogu projektu:
```bash
cd 04-PracticalSamples/petstory
```

Zbuduj aplikację:
```bash
mvn clean compile
```

Uruchom serwer:
```bash
mvn spring-boot:run
```

Aplikacja wystartuje pod adresem `http://localhost:8080`.

### Krok 3: Testuj Aplikację

1. **Otwórz** `http://localhost:8080` w przeglądarce
2. **Opisz** swojego zwierzaka w polu tekstowym (np. „Zabawny golden retriever, który uwielbia aportować”)
3. **Kliknij** „Generate Story”, aby otrzymać opowieść generowaną przez AI
4. **Alternatywnie**, prześlij zdjęcie zwierzaka, aby automatycznie wygenerować opis
5. **Zobacz** kreatywną historię na podstawie opisu Twojego zwierzaka

## Jak To Wszystko Działa Razem

Oto pełny przebieg, gdy generujesz opowieść o zwierzaku:

1. **Dane od Użytkownika:** Opisujesz zwierzaka w formularzu webowym
2. **Wysłanie Formularza:** Przeglądarka wysyła żądanie POST do `/generate-story`
3. **Przetwarzanie przez Kontroler:** `PetController` waliduje i oczyszcza dane wejściowe
4. **Wywołanie Usługi AI:** `StoryService` wysyła żądanie do modelu Azure AI Foundry
5. **Generowanie Opowieści:** AI tworzy kreatywną historię na podstawie opisu
6. **Obsługa Odpowiedzi:** Kontroler odbiera historię i dodaje ją do modelu
7. **Renderowanie Szablonu:** Thymeleaf generuje stronę `result.html` z opowieścią
8. **Wyświetlenie:** Użytkownik widzi wygenerowaną historię w przeglądarce

**Przebieg Obsługi Błędów:**
Jeśli usługa AI zawiedzie:
1. Kontroler przechwytuje wyjątek
2. Generuje zapasową opowieść z gotowych szablonów
3. Wyświetla zapasową historię wraz z informacją o niedostępności AI
4. Użytkownik nadal otrzymuje opowieść, co zapewnia dobrą jakość doświadczenia

## Zrozumienie Integracji AI

### Azure AI Foundry (bezkluczowe)
Aplikacja korzysta z Azure AI Foundry z uwierzytelnianiem bezkluczowym (Microsoft Entra ID):

```java
// Uwierzytelnianie bez klucza - brak klucza API
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Tworzenie Promptów
Serwis używa starannie opracowanych promptów, aby uzyskać dobre wyniki:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Przetwarzanie Odpowiedzi
Odpowiedź AI jest wyciągana i walidowana:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Kolejne Kroki

Po więcej przykładów zobacz [Rozdział 04: Praktyczne przykłady](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Zastrzeżenie**:
Niniejszy dokument został przetłumaczony za pomocą usługi tłumaczenia AI [Co-op Translator](https://github.com/Azure/co-op-translator). Choć dążymy do dokładności, prosimy pamiętać, że automatyczne tłumaczenia mogą zawierać błędy lub niedokładności. Oryginalny dokument w jego języku źródłowym należy uznawać za autorytatywne źródło. W przypadku informacji krytycznych zalecane jest skorzystanie z profesjonalnego tłumaczenia wykonanego przez człowieka. Nie ponosimy odpowiedzialności za jakiekolwiek nieporozumienia lub błędne interpretacje wynikające z użycia tego tłumaczenia.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->