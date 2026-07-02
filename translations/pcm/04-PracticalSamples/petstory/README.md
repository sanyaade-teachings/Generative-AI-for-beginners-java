# Pet Story Generator Tutorial for Beginners

## Table of Contents

- [Prerequisites](#prerequisites)
- [Understanding the Project Structure](#understanding-the-project-structure)
- [Core Components Explained](#core-components-explained)
  - [1. Main Application](#1-main-application)
  - [2. Web Controller](#2-web-controller)
  - [3. Story Service](#3-story-service)
  - [4. Web Templates](#4-web-templates)
  - [5. Configuration](#5-configuration)
- [Running the Application](#running-the-application)
- [How It All Works Together](#how-it-all-works-together)
- [Understanding the AI Integration](#understanding-the-ai-integration)
- [Next Steps](#next-steps)

## Prerequisites

Before you start, make sure say you get:
- Java 21 or beta version wey pass dat
- Maven for how to manage the things wey your code need
- Azure AI Foundry model wey you deploy for ground (arrange am with `azd up` — see [Chapter 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), signin wit `az login` (no need API key)
- Small sabi for Java, Spring Boot, and how web development dey work

## Understanding the Project Structure

The pet story project get different important files:

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

## Core Components Explained

### 1. Main Application

**File:** `PetStoryApplication.java`

Na here we start our Spring Boot application:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Wetin this one dey do:**
- `@SpringBootApplication` annotation dey enable auto-configuration plus e dey check make e find components automatically
- E dey start embedded web server (Tomcat) for port 8080
- E go create all Spring beans and services wey e need automatically

### 2. Web Controller

**File:** `PetController.java`

Na im e dey handle all the web request and how user dem dey interact:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Returns index.html template
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Check di input
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Clean di input make e safe
        String sanitizedDescription = sanitizeInput(description);
        
        // Make story wit error handling
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Returns result.html template
            
        } catch (Exception e) {
            // Use fallback story if AI no work
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Limit di length
    }
}
```

**Main thing dem:**

1. **Route Handling**: `@GetMapping("/")` go show upload form, `@PostMapping("/generate-story")` go process wetin user send
2. **Input Validation**: E check say description no empty and e no pass size wey dem set
3. **Security**: E clean wetin user put make e no get XSS attack
4. **Error Handling**: E get backup stories if AI service no fit work
5. **Model Binding**: E pass data go HTML templates with Spring `Model`

**Fallback System:**
Controller get pre-made story templates wey e go use if AI service no dey:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Use description hash make responses dey consistent
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Story Service

**File:** `StoryService.java`

This service dey communicate with Azure AI Foundry to create stories, e dey use keyless authentication:

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
        
        // Foundry openai-compatible endpoint dey under /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Keyless authentication wit Microsoft Entra ID (no API key)
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
        
        // Configure di AI request
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Limit how long di response go be
                .temperature(0.8)          // Control how creative e go be (0.0-1.0)
                .build();
        
        // Send di request make you get di response
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Main parts:**

1. **OpenAI Client**: E dey use official OpenAI Java SDK wey setup for Azure AI Foundry (keyless)
2. **System Prompt**: E tell AI how e suppose take behave, to write stories wey make family dem happy about pets
3. **User Prompt**: E tell AI exact story wey e go write based on description
4. **Parameters**: E control story length and how creative e suppose be
5. **Error Handling**: E throw exceptions wey controller go catch and manage

### 4. Web Templates

**File:** `index.html` (Upload Form)

Na main page wey user go describe their pets:

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

**File:** `result.html` (Story Display)

Na here e go show the story wey e generate:

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

**Template features:**

1. **Thymeleaf Integration**: E use `th:` attributes for dynamic content
2. **Responsive Design**: CSS styling for mobile and desktop
3. **Error Handling**: E dey show users validation error messages
4. **Client-side Processing**: JavaScript dey do image analysis (using Transformers.js)

### 5. Configuration

**File:** `application.properties`

Na how the application settings be:

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

**Configuration explanation:**

1. **File Upload**: E allow images wey size reach 10MB
2. **Logging**: E control wetin e go log while e dey run
3. **Azure AI Foundry**: E specify endpoint and model deployment wey e go use (keyless auth)
4. **Security**: E configure error handling make e no show secret info

## Running the Application

### Step 1: Sign In and Set Your Endpoint

Authentication no need key (e use Microsoft Entra ID), so no API key necessary. Sign in and set your Foundry endpoint:

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

**Why you need am:**
- Azure AI Foundry dey use Microsoft Entra ID to authenticate inference requests
- Keyless authentication mean say no secrets for your code or environment
- Your account must get **Cognitive Services OpenAI User** role for the resource

### Step 2: Build and Run

Go the project folder:
```bash
cd 04-PracticalSamples/petstory
```

Build the application:
```bash
mvn clean compile
```

Start the server:
```bash
mvn spring-boot:run
```

The app go start for `http://localhost:8080`.

### Step 3: Test the Application

1. **Open** `http://localhost:8080` for your browser
2. **Describe** your pet inside the text area (for example, "Playful golden retriever wey sabi fetch well well")
3. **Click** "Generate Story" to get AI-generated story
4. **Or** upload pet picture to automatically make description
5. **View** the creative story wey relate to your pet description

## How It All Works Together

Here na how the whole flow go be when you generate pet story:

1. **User Input**: You go describe your pet for the web form
2. **Form Submission**: Your browser go send POST request to `/generate-story`
3. **Controller Processing**: `PetController` go validate plus clean the input
4. **AI Service Call**: `StoryService` go send request to Azure AI Foundry model
5. **Story Generation**: AI go generate creative story based on your description
6. **Response Handling**: Controller go receive story and add am to model
7. **Template Rendering**: Thymeleaf go render `result.html` with the story
8. **Display**: You go see the story for your browser

**Error Handling Flow:**
If AI service fail:
1. Controller go catch the exception
2. E go generate fallback story with pre-written templates
3. E go show fallback story plus tell say AI no ready
4. You still go get story so e go make sure users happy

## Understanding the AI Integration

### Azure AI Foundry (keyless)
This app dey use Azure AI Foundry with keyless authentication (Microsoft Entra ID):

```java
// Keyless authentication - no API key (no key dey)
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Prompt Engineering
This service dey use well thought-out prompts to get better results:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Response Processing
AI response go extract and validate:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Next Steps

For more examples, see [Chapter 04: Practical samples](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Disclaimer**:
Dis document don translate wit AI translation service [Co-op Translator](https://github.com/Azure/co-op-translator). Even tho we dey try make am correct, abeg make you know say automated translation fit get errors or mistakes. Di original document for dia own language na im be di correct source. For important info, make person wey sabi human translation do am. We no go responsible for any misunderstanding or wrong understanding wey fit happen because of dis translation.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->