# មេរៀនបង្កើតរឿងសត្វចិញ្ចឹមសម្រាប់​អ្នក​ចាប់​ផ្តើម

## តារាងមាតិកា

- [លក្ខខណ្ឌ​ធម្មតា](#លក្ខខណ្ឌ​ធម្មតា)
- [ការយល់ដឹងស្ដីពីរចនាសម្ព័ន្ធគំរោង](#ការយល់ដឹងស្ដីពីរចនាសម្ព័ន្ធគំរោង)
- [ការពន្យល់ពីគ្រឿងចក្រស្នូល](#ការពន្យល់ពីគ្រឿងចក្រស្នូល)
  - [1. កម្មវិធីសំខាន់](#1-កម្មវិធីសំខាន់)
  - [2. ឧបករណ៍គ្រប់គ្រងវេបសាយ](#2-ឧបករណ៍គ្រប់គ្រងវេបសាយ)
  - [3. សេវាកម្មរឿង](#3-សេវាកម្មរឿង)
  - [4. ទំព័រគំរូវេបសាយ](#4-ទំព័រគំរូវេបសាយ)
  - [5. ការកំណត់រចនាសម្ព័ន្ធ](#5-ការកំណត់រចនាសម្ព័ន្ធ)
- [ការបើកដំណើរកម្មកម្មវិធី](#ការបើកដំណើរកម្មកម្មវិធី)
- [វិធីសាស្រ្តដើម្បីឲ្យវាដំណើរការជាក្រុម](#វិធីសាស្រ្តដើម្បីឲ្យវាដំណើរការជាក្រុម)
- [ការយល់ដឹងអំពីការបញ្ចូល AI](#ការយល់ដឹងអំពីការបញ្ចូល-ai)
- [ជំហានបន្ទាប់](#ជំហានបន្ទាប់)

## លក្ខខណ្ឌ​ធម្មតា

មុនចាប់ផ្តើម, សូមប្រាកដថាអ្នកមានៈ
- Java 21 ឬកម្រិតខ្ពស់ជាងនេះបានដំឡើង
- Maven សម្រាប់គ្រប់គ្រងផ្នត់ផ្គត់ផ្គង់
- ការផ្តល់សេវា Azure AI Foundry សម្រាប់ម៉ូដែល (ជាមួយ `azd up` — មើល [ជំពូក 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), បានចុះឈ្មោះជាមួយ `az login` (គ្មានកូនសោ)
- ការយល់ដឹងមូលដ្ឋានពី Java, Spring Boot និងការអភិវឌ្ឍន៍វេបសាយ

## ការយល់ដឹងស្ដីពីរចនាសម្ព័ន្ធគំរោង

គំរោងរឿងសត្វចិញ្ចឹមមានឯកសារសំខាន់ជាច្រើន៖

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

## ការពន្យល់ពីគ្រឿងចក្រស្នូល

### 1. កម្មវិធីសំខាន់

**ឯកសារ៖** `PetStoryApplication.java`

នេះគឺជាច្រកចូលសម្រាប់កម្មវិធី Spring Boot របស់យើង៖

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**អ្វីដែលវាធ្វើ:**
- ស្លាក `@SpringBootApplication` ផ្តល់សមត្ថភាពកំណត់រចនាសម្ព័ន្ធដោយស្វ័យប្រវត្តិ និងស្កេនគ្រឿងចក្រជា​រ
- ចាប់ផ្តើមម៉ាស៊ីនបម្រើវេបសាយបញ្ចូល (Tomcat) នៅច្រក 8080
- បង្កើត Spring beans និងសេវាកម្មទាំងអស់ដោយស្វ័យប្រវត្តិ

### 2. ឧបករណ៍គ្រប់គ្រងវេបសាយ

**ឯកសារ៖** `PetController.java`

នេះគឺជាឧបករណ៍ដែលគ្រប់គ្រងសំណើវេប និងអន្តរកម្មអ្នកប្រើ៖

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // បញ្ជូនតំបន់ព្រឹត្តិប័ត្រ index.html
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // ត្រួតពិនិត្យការបញ្ចូល
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // លាងសម្អាតការបញ្ចូលសម្រាប់សុវត្ថិភាព
        String sanitizedDescription = sanitizeInput(description);
        
        // បង្កើតរឿងជាមួយការដោះស្រាយកំហុស
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // បញ្ជូនតំបន់ព្រឹត្តិប័ត្រ result.html
            
        } catch (Exception e) {
            // ប្រើរឿងជំនួស ប្រសិនបើ AI ខ្វះខាត
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // កំណត់ប្រវែង
    }
}
```

**មុខងារសំខាន់ៗ៖**

1. **ការគ្រប់គ្រងផ្លូវការផ្សេងៗ**: `@GetMapping("/")` បង្ហាញបែបបទបញ្ជូនទិន្នន័យ, `@PostMapping("/generate-story")` ដំណើរការការដាក់ស្នើ
2. **ការត្រួតពិនិត្យបញ្ចូល**: ពិនិត្យសម្រាប់ការពណ៌នាទទេ និងកំណត់ប្រវែងអត្ថបទ
3. **សុវត្ថិភាព**: សម្រេចអក្សរបញ្ចូលដើម្បីការពារការវាយប្រហារបែប XSS
4. **ការដោះស្រាយកំហុស**: ផ្តល់រឿងជំនួសនៅពេលសេវាកម្ម AI បរាជ័យ
5. **ការចងភ្ជាប់ម៉ូដែល**: ផ្ញើទិន្នន័យទៅទំព័រ HTML តាមរយៈ Model របស់ Spring

**ប្រព័ន្ធជំនួស:**
ឧបករណ៍គ្រប់គ្រងមានគំរូរឿងដែលបានចងក្រងរួចហើយ ដែលប្រើនៅពេលសេវា AI មិនអាចប្រើបាន៖

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // ប្រើ hash ពិពណ៌នាសម្រាប់ចម្លើយមានភាពរឹងមាំ
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. សេវាកម្មរឿង

**ឯកសារ៖** `StoryService.java`

សេវាកម្មនេះទំនាក់ទំនងជាមួយ Azure AI Foundry ដើម្បីបង្កើតរឿងដោយប្រើ authentication គ្មាន key៖

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
        
        // ចំណុចចេញផ្សាយដែលឆបគ្នានឹង OpenAI របស់ Foundry មាននៅក្រោម /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // ការផ្ទៀងផ្ទាត់គ្មានកូនសោជាមួយ Microsoft Entra ID (គ្មានចុះ API)
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
        
        // កំណត់ការស្នើសុំ AI
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // កំណត់ដែនកំណត់ប្រវែងចម្លើយ
                .temperature(0.8)          // គ្រប់គ្រងភាពច្នៃប្រឌិត (0.0-1.0)
                .build();
        
        // ផ្ញើសំណើហើយទទួលចម្លើយ
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**គ្រឿងចក្រ​សំខាន់ៗ៖**

1. **OpenAI Client**: ប្រើ OpenAI Java SDK ផ្លូវការដែលបានកំណត់សម្រាប់ Azure AI Foundry (គ្មាន key)
2. **ប្រអប់ប្រព័ន្ធ**: កំណត់ប្រតិកម្ម AI ដើម្បីសរសេររឿងសត្វចិញ្ចឹមសម្រាប់គ្រួសារជាការសម្របសម្រួល
3. **ប្រអប់អ្នកប្រើ**: បញ្ជាក់អ្វីដែល AI ត្រូវសរសេរតាមការពណ៌នា
4. **ប៉ារ៉ាម៉ែត្រ**: គ្រប់គ្រងប្រវែងរឿង និងកម្រិតភាពច្នៃប្រឌិត
5. **ការដោះស្រាយកំហុស**: បោះបង់ករណីកំហុស ដែលឧបករណ៍គ្រប់គ្រងនឹងចាប់ និងដោះស្រាយ

### 4. ទំព័រគំរូវេបសាយ

**ឯកសារ៖** `index.html` (បែបបទបញ្ជូន)

ទំព័រដើមដែលអ្នកប្រើពណ៌នាស្តីពីសត្វចិញ្ចឹមរបស់ពួកគេ៖

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

**ឯកសារ៖** `result.html` (បង្ហាញរឿង)

បង្ហាញរឿងដែលបានបង្កើត៖

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

**លក្ខណៈគំរូ៖**

1. **ការបញ្ចូល Thymeleaf**: ប្រើលក្ខណៈ `th:` សម្រាប់មាតិកាដែលចល័ត
2. **រចនាប្រកបដោយឆ្លើយតប**: ការតាំងស្ទាយ CSS សម្រាប់ទូរស័ព្ទចល័ត និងកុំព្យូទ័រដេសក្ទុប
3. **ការដោះស្រាយកំហុស**: បង្ហាញកំហុសផ្ទៀងផ្ទាត់ទៅអ្នកប្រើ
4. **ដំណើរការផ្នែកអ្នកដំណើរការ**: JavaScript សម្រាប់វិភាគរូបភាព (ប្រើ Transformers.js)

### 5. ការកំណត់រចនាសម្ព័ន្ធ

**ឯកសារ៖** `application.properties`

ការកំណត់សម្រាប់កម្មវិធី៖

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

**ការពន្យល់ការកំណត់៖**

1. **ការផ្ទុកឯកសារ**: អនុញ្ញាតរូបភាពមានទំហំឡើងដល់ 10MB
2. **ការចុះកំណត់ហេតុ**: គ្រប់គ្រងពត៌មានដែលត្រូវចុះកំណត់ហេតុពេលដំណើរការ
3. **Azure AI Foundry**: បញ្ជាក់ចំណុចចេញ និងការផ្តល់ម៉ូដែលតាម authentication គ្មាន key
4. **សុវត្ថិភាព**: ការកំណត់សម្រាប់ដោះស្រាយកំហុស ដើម្បីជៀសវាងការបង្ហាញព័ត៌មានចាក់ទុក

## ការបើកដំណើរកម្មកម្មវិធី

### ជំហានទី ១៖ ចុះឈ្មោះ និងកំណត់ចំណុចចេញរបស់អ្នក

ការផ្ទៀងផ្ទាត់គឺគ្មាន key (Microsoft Entra ID) ដូច្នេះមិនមានកូនសោ API ទេ។ ចុះឈ្មោះហើយកំណត់ចំណុចចេញ Foundry របស់អ្នក៖

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

**ហេតុអ្វីវាត្រូវការ:**
- Azure AI Foundry ប្រើ Microsoft Entra ID ក្នុងការផ្ទៀងផ្ទាត់សំណើ inference
- ការផ្ទៀងផ្ទាត់គ្មាន key មានន័យថាគ្មានអ្វីលេចធ្លោនៅក្នុងកូដប្រភព ឬបរិស្ថានរបស់អ្នក
- គណនីរបស់អ្នកត្រូវការរ៉ូល **Cognitive Services OpenAI User** លើធនធាន

### ជំហានទី ២៖ សាងសង់ និងដំណើរការ

ចូលទៅថតគំរោង៖
```bash
cd 04-PracticalSamples/petstory
```

សាងសង់កម្មវិធី៖
```bash
mvn clean compile
```

ចាប់ផ្តើមម៉ាស៊ីនបម្រើ៖
```bash
mvn spring-boot:run
```

កម្មវិធីនឹងចាប់ផ្តើមលើ `http://localhost:8080`។

### ជំហានទី ៣៖ សាកល្បងកម្មវិធី

1. **បើក** `http://localhost:8080` ក្នុងកម្មវិធីរកមើលវេបរបស់អ្នក
2. **ពណ៌នា** សត្វចិញ្ចឹមរបស់អ្នកនៅក្នុងទីតាំងអក្សរចូល (ឧ. "សត្វសេះប្រភេទ golden retriever ដែលចូលចិត្តយកវត្ថុមក")
3. **ចុច** "Generate Story" ដើម្បីទទួលរឿងដែល AI បង្កើត
4. **ជាជម្រើសទៀត**, បញ្ចូលរូបភាពសត្វចិញ្ចឹមដើម្បីបង្កើតការពណ៌នាពីរូបភាពស្វ័យប្រវត្តិ
5. **មើល** រឿងច្នៃប្រឌិតផ្អែកលើការពណ៌នារបស់សត្វចិញ្ចឹមរបស់អ្នក

## វិធីសាស្រ្តដើម្បីឲ្យវាដំណើរការជាក្រុម

នេះជាប្រតិបត្តិការប្រុងប្រយ័ត្នពេញលេញពេលអ្នកបង្កើតរឿងពិភពចិញ្ចឹម៖

1. **បញ្ចូលរបស់អ្នកប្រើ**: អ្នកពណ៌នាសត្វចិញ្ចឹមរបស់អ្នកលើបែបបទវេប
2. **ការដាក់ស្នើបែបបទ**: កម្មវិធីរកមើលផ្ញើសំណើ POST ទៅ `/generate-story`
3. **ដំណើរការឧបករណ៍គ្រប់គ្រង**: `PetController` ពិនិត្យត្រឹមត្រូវ និងសម្រេចអក្សរបញ្ចូល
4. **សេវាកម្ម AI**: `StoryService` ផ្ញើសំណើទៅម៉ូដែល Azure AI Foundry
5. **បង្កើតរឿង**: AI បង្កើតរឿងច្នៃប្រឌិតផ្អែកលើការពណ៌នា
6. **ដោះស្រាយការឆ្លើយតប**: ឧបករណ៍គ្រប់គ្រងទទួលរឿងហើយបន្ថែមទៅម៉ូដែល
7. **បង្ហាញលទ្ធផលទំព័រ**: Thymeleaf រៀបចំ `result.html` ជាមួយរឿង
8. **បង្ហាញ**: អ្នកប្រើឃើញរឿងដែលបានបង្កើតនៅក្នុងកម្មវិធីរកមើលរបស់ពួកគេ

**ដំណើរការដោះស្រាយកំហុស:**
ប្រសិនបើសេវាកម្ម AI បរាជ័យ:
1. អ្នកគ្រប់គ្រងចាប់ករណីកំហុស
2. បង្កើតរឿងជំនួសដោយប្រើគំរូរួចហើយ
3. បង្ហាញរឿងជំនួសជាមួយកំណត់សម្គាល់អំពីមិនអាចប្រើ AI
4. អ្នកប្រើនៅតែទទួលបានរឿង ដើម្បីធានាបទពិសោធន៍ល្អ

## ការយល់ដឹងអំពីការបញ្ចូល AI

### Azure AI Foundry (គ្មាន key)
កម្មវិធីប្រើ Azure AI Foundry ជាមួយ authentication គ្មាន key (Microsoft Entra ID)៖

```java
// ការផ្ទៀងផ្ទាត់គ្មានកន្លែងគន្លឹះ - គ្មានកូនសោ API
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### បច្ចេកទេសការបញ្ចូល Prompt
សេវាកម្មប្រើ prompt បានរៀបចំយ៉ាងហ្មត់ចត់ដើម្បីទទួលបានលទ្ធផលល្អ៖

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### ដំណើរការឆ្លើយតប
ចម្លើយ AI ត្រូវបានដកស្រង់និងផ្ទៀងផ្ទាត់៖

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## ជំហានបន្ទាប់

សម្រាប់ឧទាហរណ៍បន្ថែម, សូមមើល [ជំពូក 04: ឧទាហរណ៍អនុវត្តន៍](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ការបដិសេធ**:
ឯកសារនេះត្រូវបានបម្លែងភាសា ដោយប្រើសេវាបម្លែងភាសា AI [Co-op Translator](https://github.com/Azure/co-op-translator)។ ទោះយើងខ្ញុំមានក្តីប្រាថ្នាឱ្យបានច្បាស់លាស់ តែសូមយល់ដឹងថាការបម្លែងដោយស្វ័យប្រវត្តិក៏អាចមានកំហុសឬភាពមិនត្រឹមត្រូវ។ ឯកសារដើមជាភាសាទីតាំងគួរត្រូវបានគេប្រើជាប្រភពច្បាស់លាស់។ សម្រាប់ព័ត៌មានសំខាន់ៗ សូមណែនាំឱ្យប្រើប្រាស់ការប្រែដោយមនុស្សជំនាញ។ យើងខ្ញុំមិនទទួលខុសត្រូវចំពោះការយល់ច្រឡំ ឬការបកស្រាយខុសបន្ទាប់ពីការប្រើប្រាស់ការបម្លែងនេះនោះទេ។
<!-- CO-OP TRANSLATOR DISCLAIMER END -->