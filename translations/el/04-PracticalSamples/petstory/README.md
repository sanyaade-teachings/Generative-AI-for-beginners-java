# Οδηγός Δημιουργίας Ιστορίας Κατοικιδίων για Αρχάριους

## Πίνακας Περιεχομένων

- [Προαπαιτούμενα](#προαπαιτούμενα)
- [Κατανόηση της Δομής του Έργου](#κατανόηση-της-δομής-του-έργου)
- [Επεξήγηση Βασικών Στοιχείων](#επεξήγηση-βασικών-στοιχείων)
  - [1. Κύρια Εφαρμογή](#1-κύρια-εφαρμογή)
  - [2. Ελεγκτής Ιστού](#2-ελεγκτής-ιστού)
  - [3. Υπηρεσία Ιστορίας](#3-υπηρεσία-ιστορίας)
  - [4. Πρότυπα Ιστού](#4-πρότυπα-ιστού)
  - [5. Διαμόρφωση](#5-διαμόρφωση)
- [Εκτέλεση της Εφαρμογής](#εκτέλεση-της-εφαρμογής)
- [Πώς Λειτουργούν Όλα Μαζί](#πώς-λειτουργούν-όλα-μαζί)
- [Κατανόηση της Ενσωμάτωσης Τεχνητής Νοημοσύνης](#κατανόηση-της-ενσωμάτωσης-τεχνητής-νοημοσύνης)
- [Επόμενα Βήματα](#επόμενα-βήματα)

## Προαπαιτούμενα

Πριν ξεκινήσετε, βεβαιωθείτε ότι έχετε:
- Εγκατεστημένη την Java 21 ή νεότερη
- Το Maven για διαχείριση εξαρτήσεων
- Αναπτύξει μοντέλο Azure AI Foundry (προμηθευτείτε το με `azd up` — δείτε το [Κεφάλαιο 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), συνδεδεμένοι με `az login` (αυθεντικοποίηση χωρίς κλειδί)
- Βασική κατανόηση της Java, Spring Boot και ανάπτυξης ιστοσελίδων

## Κατανόηση της Δομής του Έργου

Το έργο ιστορίας κατοικιδίων περιέχει αρκετά σημαντικά αρχεία:

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

## Επεξήγηση Βασικών Στοιχείων

### 1. Κύρια Εφαρμογή

**Αρχείο:** `PetStoryApplication.java`

Αυτή είναι η είσοδος για την εφαρμογή Spring Boot μας:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Τι κάνει:**
- Η ανάλυση `@SpringBootApplication` ενεργοποιεί αυτόματη διαμόρφωση και σάρωση συστατικών
- Εκκινεί έναν ενσωματωμένο διακομιστή ιστού (Tomcat) στην θύρα 8080
- Δημιουργεί αυτόματα όλα τα απαραίτητα Spring beans και υπηρεσίες

### 2. Ελεγκτής Ιστού

**Αρχείο:** `PetController.java`

Αυτός διαχειρίζεται όλα τα αιτήματα ιστού και τις αλληλεπιδράσεις των χρηστών:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Επιστρέφει το πρότυπο index.html
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Επαλήθευση εισόδου
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Καθαρισμός εισόδου για ασφάλεια
        String sanitizedDescription = sanitizeInput(description);
        
        // Δημιουργία ιστορίας με διαχείριση σφαλμάτων
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Επιστρέφει το πρότυπο result.html
            
        } catch (Exception e) {
            // Χρήση εναλλακτικής ιστορίας αν αποτύχει η AI
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Περιορισμός μήκους
    }
}
```

**Κύρια χαρακτηριστικά:**

1. **Διαχείριση Διαδρομών**: Το `@GetMapping("/")` εμφανίζει τη φόρμα αποστολής, το `@PostMapping("/generate-story")` επεξεργάζεται υποβολές
2. **Επικύρωση Εισόδου**: Ελέγχει για κενές περιγραφές και όρια μήκους
3. **Ασφάλεια**: Καθαρίζει την είσοδο χρήστη για αποφυγή επιθέσεων XSS
4. **Διαχείριση Σφαλμάτων**: Παρέχει εναλλακτικές ιστορίες όταν η υπηρεσία AI αποτυγχάνει
5. **Δέσμευση Μοντέλου**: Μεταφέρει δεδομένα στα HTML πρότυπα χρησιμοποιώντας το Spring `Model`

**Σύστημα Εναλλακτικής Επιλογής:**
Ο ελεγκτής περιλαμβάνει προεγγραμμένα πρότυπα ιστοριών που χρησιμοποιούνται όταν η υπηρεσία AI δεν είναι διαθέσιμη:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Χρησιμοποιήστε το hash περιγραφής για συνεπείς απαντήσεις
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Υπηρεσία Ιστορίας

**Αρχείο:** `StoryService.java`

Αυτή η υπηρεσία επικοινωνεί με το Azure AI Foundry για να δημιουργεί ιστορίες χρησιμοποιώντας αυθεντικοποίηση χωρίς κλειδί:

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
        
        // Το συμβατό με OpenAI τελικό σημείο του Foundry βρίσκεται κάτω από /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Αυθεντικοποίηση χωρίς κλειδί με Microsoft Entra ID (χωρίς κλειδί API)
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
        
        // Διαμορφώστε το αίτημα AI
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Περιορίστε το μήκος της απόκρισης
                .temperature(0.8)          // Ελέγξτε τη δημιουργικότητα (0.0-1.0)
                .build();
        
        // Αποστολή αιτήματος και λήψη απόκρισης
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Κύρια στοιχεία:**

1. **OpenAI Client**: Χρησιμοποιεί το επίσημο OpenAI Java SDK ρυθμισμένο για Azure AI Foundry (χωρίς κλειδί)
2. **Σύστημα Οδηγίας**: Καθορίζει τη συμπεριφορά του AI για να γράφει οικογενειακές ιστορίες κατοικιδίων
3. **Οδηγία Χρήστη**: Παροτρύνει το AI τι ακριβώς ιστορία να γράψει βάσει της περιγραφής
4. **Παράμετροι**: Ελέγχουν το μήκος και τη δημιουργικότητα της ιστορίας
5. **Διαχείριση Σφαλμάτων**: Πετάει εξαιρέσεις που το controller αναλαμβάνει και διαχειρίζεται

### 4. Πρότυπα Ιστού

**Αρχείο:** `index.html` (Φόρμα Αποστολής)

Η κύρια σελίδα όπου οι χρήστες περιγράφουν τα κατοικίδιά τους:

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

**Αρχείο:** `result.html` (Εμφάνιση Ιστορίας)

Εμφανίζει την παραγόμενη ιστορία:

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

**Χαρακτηριστικά προτύπου:**

1. **Ενσωμάτωση Thymeleaf**: Χρησιμοποιεί χαρακτηριστικά `th:` για δυναμικό περιεχόμενο
2. **Ανταποκρινόμενος Σχεδιασμός**: Στυλιζάρισμα CSS για κινητές και επιτραπέζιες συσκευές
3. **Διαχείριση Σφαλμάτων**: Εμφανίζει μηνύματα επικύρωσης στους χρήστες
4. **Επεξεργασία στην πλευρά του πελάτη**: JavaScript για ανάλυση εικόνας (χρησιμοποιώντας Transformers.js)

### 5. Διαμόρφωση

**Αρχείο:** `application.properties`

Ρυθμίσεις διαμόρφωσης για την εφαρμογή:

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

**Επεξήγηση Διαμόρφωσης:**

1. **Αποστολή Αρχείων**: Επιτρέπει εικόνες έως 10MB
2. **Καταγραφή**: Ελέγχει τις πληροφορίες που καταγράφονται κατά την εκτέλεση
3. **Azure AI Foundry**: Προσδιορίζει το endpoint και την ανάπτυξη μοντέλου που θα χρησιμοποιηθεί (χωρίς κλειδί)
4. **Ασφάλεια**: Διαμόρφωση διαχείρισης σφαλμάτων για αποφυγή αποκάλυψης ευαίσθητων πληροφοριών

## Εκτέλεση της Εφαρμογής

### Βήμα 1: Σύνδεση και Ρύθμιση του Endpoint

Η αυθεντικοποίηση είναι χωρίς κλειδί (Microsoft Entra ID), οπότε δεν απαιτείται API key. Συνδεθείτε και ορίστε το endpoint του Foundry:

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

**Γιατί είναι απαραίτητο:**
- Το Azure AI Foundry χρησιμοποιεί το Microsoft Entra ID για την αυθεντικοποίηση αιτημάτων inference
- Η αυθεντικοποίηση χωρίς κλειδί σημαίνει ότι δεν υπάρχουν μυστικά στον κώδικα ή στο περιβάλλον σας
- Ο λογαριασμός σας πρέπει να έχει το ρόλο **Cognitive Services OpenAI User** στον πόρο

### Βήμα 2: Δημιουργία και Εκτέλεση

Πλοηγηθείτε στον φάκελο του έργου:
```bash
cd 04-PracticalSamples/petstory
```

Δημιουργήστε την εφαρμογή:
```bash
mvn clean compile
```

Εκκινήστε τον διακομιστή:
```bash
mvn spring-boot:run
```

Η εφαρμογή θα ξεκινήσει στη διεύθυνση `http://localhost:8080`.

### Βήμα 3: Δοκιμή της Εφαρμογής

1. **Ανοίξτε** `http://localhost:8080` στον περιηγητή σας
2. **Περιγράψτε** το κατοικίδιό σας στο πεδίο κειμένου (π.χ. "Ένας παιχνιδιάρης χρυσός ριτρίβερ που αγαπά να φέρνει αντικείμενα")
3. **Κάντε κλικ** στο "Generate Story" για να λάβετε μια ιστορία που δημιούργησε το AI
4. **Εναλλακτικά**, ανεβάστε μια εικόνα κατοικιδίου για αυτόματη δημιουργία περιγραφής
5. **Δείτε** την δημιουργική ιστορία με βάση την περιγραφή του κατοικιδίου σας

## Πώς Λειτουργούν Όλα Μαζί

Ακολουθεί η πλήρης ροή όταν δημιουργείτε μια ιστορία κατοικιδίου:

1. **Είσοδος Χρήστη**: Περιγράφετε το κατοικίδιό σας στη φόρμα ιστού
2. **Υποβολή Φόρμας**: Ο περιηγητής στέλνει POST αίτημα στο `/generate-story`
3. **Επεξεργασία από τον Ελεγκτή**: Ο `PetController` επικυρώνει και καθαρίζει την είσοδο
4. **Κλήση Υπηρεσίας AI**: Ο `StoryService` στέλνει το αίτημα στο μοντέλο Azure AI Foundry
5. **Δημιουργία Ιστορίας**: Το AI παράγει μια δημιουργική ιστορία βάσει της περιγραφής
6. **Διαχείριση Απάντησης**: Ο ελεγκτής λαμβάνει την ιστορία και τη προσθέτει στο μοντέλο
7. **Απόδοση Προτύπου**: Ο Thymeleaf αποδίδει το `result.html` με την ιστορία
8. **Εμφάνιση**: Ο χρήστης βλέπει την ιστορία που δημιουργήθηκε στον περιηγητή του

**Ροή Διαχείρισης Σφαλμάτων:**
Αν η υπηρεσία AI αποτύχει:
1. Ο ελεγκτής πιάνει την εξαίρεση
2. Δημιουργεί μια εναλλακτική ιστορία χρησιμοποιώντας προεγγραμμένα πρότυπα
3. Εμφανίζει την εναλλακτική ιστορία με μια σημείωση για μη διαθεσιμότητα του AI
4. Ο χρήστης λαμβάνει ακόμα μια ιστορία, εξασφαλίζοντας καλή εμπειρία χρήστη

## Κατανόηση της Ενσωμάτωσης Τεχνητής Νοημοσύνης

### Azure AI Foundry (χωρίς κλειδί)
Η εφαρμογή χρησιμοποιεί το Azure AI Foundry με αυθεντικοποίηση χωρίς κλειδί (Microsoft Entra ID):

```java
// Αυθεντικοποίηση χωρίς κλειδί - χωρίς κλειδί API
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Μηχανική Οδηγιών (Prompt Engineering)
Η υπηρεσία χρησιμοποιεί προσεκτικά κατασκευασμένες οδηγίες για καλύτερα αποτελέσματα:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Επεξεργασία Απάντησης
Η απάντηση του AI εξάγεται και επικυρώνεται:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Επόμενα Βήματα

Για περισσότερα παραδείγματα, δείτε το [Κεφάλαιο 04: Πρακτικά παραδείγματα](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Αποποίηση ευθυνών**:
Αυτό το έγγραφο έχει μεταφραστεί χρησιμοποιώντας την υπηρεσία μετάφρασης με τεχνητή νοημοσύνη [Co-op Translator](https://github.com/Azure/co-op-translator). Ενώ επιδιώκουμε την ακρίβεια, παρακαλούμε να έχετε υπόψη ότι οι αυτοματοποιημένες μεταφράσεις ενδέχεται να περιέχουν λάθη ή ανακρίβειες. Το πρωτότυπο έγγραφο στη μητρική του γλώσσα πρέπει να θεωρείται η αυθεντική πηγή. Για κρίσιμες πληροφορίες, συνιστάται επαγγελματική ανθρώπινη μετάφραση. Δεν φέρουμε ευθύνη για τυχόν παρεξηγήσεις ή λανθασμένες ερμηνείες που προκύπτουν από τη χρήση αυτής της μετάφρασης.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->