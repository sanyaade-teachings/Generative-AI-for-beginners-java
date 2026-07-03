# စောင့်ရှောက်ရန်တိရစ္ဆာန် कहानी ဂျင်နရေးတာ သင်ခန်းစာ တ初心者向け

## အကြောင်းအရာ စာရင်း

- [လိုအပ်ချက်များ](#လိုအပ်ချက်များ)
- [ပရောဂျက် ဖွဲ့စည်းမှု ကို နားလည်ခြင်း](#ပရောဂျက်-ဖွဲ့စည်းမှု-ကို-နားလည်ခြင်း)
- [အဓိက အစိတ်အပိုင်းများ ရှင်းလင်းချက်](#အဓိက-အစိတ်အပိုင်းများ-ရှင်းလင်းချက်)
  - [1. မိမိ အသုံးပြုနေသော အက်ပ်ပလီကေးရှင်း](#၁။-မိမိ-အသုံးပြုနေသော-အက်ပ်ပလီကေးရှင်း)
  - [2. ဝက်ဘ် ထိန်းချုပ်ရေးသူ](#၂။-ဝက်ဘ်-ထိန်းချုပ်ရေးသူ)
  - [3. ပုံပြင်ဝန်ဆောင်မှု](#၃။-ပုံပြင်-ဝန်ဆောင်မှု)
  - [4. ဝက်ဘ် ပုံစံများ](#၄။-ဝက်ဘ်-ပုံစံများ)
  - [5. ဖွဲ့စည်းတပ်ဆင်မှု](#၅။-ဖွဲ့စည်းတပ်ဆင်မှု)
- [အက်ပ်ပလီကေးရှင်း စတင်လည်ပတ်ခြင်း](#အက်ပ်ပလီကေးရှင်း-စတင်လည်ပတ်ခြင်း)
- [ဒါအားလုံး ဘယ်လိုပေါင်းစည်း လည်ပတ်သလဲ](#ဒါအားလုံး-ဘယ်လိုပေါင်းစည်း-လည်ပတ်သလဲ)
- [AI ပေါင်းစည်းမှု ကို နားလည်မြင်သာခြင်း](#ai-ပေါင်းစည်းမှု-ကို-နားလည်မြင်သာခြင်း)
- [နောက်တစ်ခုပြုလုပ်ရန် အဆင့်များ](#နောက်တစ်ခုပြုလုပ်ရန်-အဆင့်များ)

## လိုအပ်ချက်များ

စတင်ရန်အရင်မှာ သေချာစွာ သင်မှာရှိထားရမည့် အရာများ -
- Java 21 သို့မဟုတ် ထို့အထက် ထည့်သွင်းထား
- လိုအပ်သော dependencies များကို စီမံရန် Maven
- Azure AI Foundry မော်ဒယ် ဖြန့်ကျက်မှု ( `azd up` ဖြင့် provision လုပ်ပါ — ကြည့်ရန် [အခန်း 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))၊ `az login` ဖြင့် သွင်းပေးထားပြီး (keyless authentication)
- Java, Spring Boot နှင့် ဝက်ဘ် ဖွံ့ဖြိုးရေး၏ မူလ အခြေခံ နားလည်မှု

## ပရောဂျက် ဖွဲ့စည်းမှု ကို နားလည်ခြင်း

တိရစ္ဆာန် ပုံပြင် ပရောဂျက်တွင် အရေးပါသော ဖိုင် အချို့ပါရှိသည် -

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


## အဓိက အစိတ်အပိုင်းများ ရှင်းလင်းချက်

### ၁။ မိမိ အသုံးပြုနေသော အက်ပ်ပလီကေးရှင်း

**ဖိုင်:** `PetStoryApplication.java`

ဤသည်သည် ကျွန်ုပ်တို့၏ Spring Boot အက်ပ်ပလီကေးရှင်း၏ ဝင်ခွင့် နေရာဖြစ်သည် -

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**ဤအရာများ လုပ်ဆောင်ပုံ:**  
- `@SpringBootApplication` annotation က auto-configuration နှင့် component scanning ကို ဖွင့်လှစ်ပေးသည်  
- 8080 ပေါ့(Port) တွင် embedded web server (Tomcat) စတင်လည်ပတ်သည်  
- အလိုအလျောက် လိုအပ်သော Spring beans နှင့် ဝန်ဆောင်မှုများကို ဖန်တီးပေးသည်  

### ၂။ ဝက်ဘ် ထိန်းချုပ်ရေးသူ

**ဖိုင်:** `PetController.java`

တစ်ခုချင်းလုပ်ဆောင်ချက်များနှင့် အသုံးပြုသူ သက်ဆိုင်ရာ မေးခွန်းများကို အုပ်ချုပ်သည် -

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // index.html စာမျက်နှာပုံစံ ပြန်လည်ပေးပို့သည်
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // အချက်အလက် ထည့်သွင်းမှု စစ်ဆေးခြင်း
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // လုံခြုံမှုအတွက် အချက်အလက် သန့်စင်ခြင်း
        String sanitizedDescription = sanitizeInput(description);
        
        // အမှားကို ကိုင်တွယ်၍ ပုံပြင် ဖန်တီးခြင်း
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // result.html စာမျက်နှာပုံစံ ပြန်လည်ပေးပို့သည်
            
        } catch (Exception e) {
            // AI မအောင်မြင်လျှင် အစားထိုး ပုံပြင် အသုံးပြုပါ
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // အရှည်ကို ကန့်သတ်ပါ
    }
}
```

**အဓိက လက္ခဏာများ:**

1. **လမ်းကြောင်း ကိုင်တွယ်မှု**: `@GetMapping("/")` မှာ upload form ကိုပြသသည်၊ `@PostMapping("/generate-story")` မှာ ပုံပြင်ထုတ်လွှင့်ရန် စာတင်ပြုလုပ်ခြင်း  
2. **ထည့်သွင်းချက် စစ်ဆေးမှု**: ဖော်ပြချက်များ မရှိခြင်း၊ အရှည် ကန့်သတ်မှု စစ်ဆေး  
3. **လုံခြုံရေး**: အသုံးပြုသူ ထည့်သွင်းချက်များကို XSS ကျိုးပဲ့မှုမှ ကာကွယ်ရန် စနစ်တကျ သန့်ရှင်း  
4. **အမှား ကင်းရှင်းမှု**: AI ဝန်ဆောင်မှု မအောင်မြင်သောအခါ ရှေ့ပြေး ပုံပြင်များ ပေးသွင်း  
5. **မော်ဒယ် ချိတ်ဆက်မှု**: Spring ၏ `Model` ကို အသုံးပြု၍ ဒေတာများကို HTML ပုံစံများတွင် ပေးပို့

**ရှေ့ပြေး စနစ်:**
controller တွင် AI ဝန်ဆောင်မှု မရရှိနိုင်သောအခါ အသုံးပြုသည့် ရေးသားပြီးသား ပုံပြင်ပုံစံများ ပါဝင်သည် -

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // တူညီသောတုံ့ပြန်မှုများအတွက် ဖော်ပြချက်ဟက်ကို အသုံးပြုပါ
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```


### ၃။ ပုံပြင် ဝန်ဆောင်မှု

**ဖိုင်:** `StoryService.java`

ဤဝန်ဆောင်မှုသည် keyless authentication ဖြင့် Azure AI Foundry နှင့် ဆက်သွယ်ကာ ပုံပြင်များ ဖန်တီးပေးသည် -

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
        
        // Foundry ၏ OpenAI ကိုသင့်တော်သော endpoint သည် /openai/v1/ အောက်တွင် တည်ရှိသည်
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Microsoft Entra ID ဖြင့် Key ဂုဏ်ပြုခြင်း (API key မလိုအပ်ပါ)
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
        
        // AI အတွက် တောင်းဆိုမှုကို ဖွဲ့စည်းစီစဉ်ပါ
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // တုံ့ပြန်မှု အရှည်ကို ကန့်သတ်ပါ
                .temperature(0.8)          // ဖန်တီးမှု ထိန်းချုပ်မှု (0.0-1.0)
                .build();
        
        // တောင်းဆိုမှု ပေးပို့ပြီး တုံ့ပြန်မှု ရယူပါ
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**အဓိက အစိတ်အပိုင်းများ:**

1. **OpenAI Client**: Azure AI Foundry အတွက် official OpenAI Java SDK ကို (keyless) ကိုသုံး  
2. **System Prompt**: AI ကို မိသားစုအတွက် သင့်တော်သော တိရစ္ဆာန် ပုံပြင်များ ရေးရန် အပြုသဘောသတ်မှတ်  
3. **User Prompt**: ဖော်ပြချက်အတိုင်း AI ကိုပုံပြင် ရေးရန် တိတိကျကျ ပြောဆို  
4. **ပမာဏများ**: ပုံပြင် အရှည်နှင့် ဖန်တီးမှု အဆင့်ကို ထိန်းချုပ်  
5. **အမှား ကင်းရှင်းမှု**: controller ဖြင့် ဖမ်းဆီးစစ်ဆေးသည့် exceptions များကို ပေါက်ခတ်ပေး

### ၄။ ဝက်ဘ် ပုံစံများ

**ဖိုင်:** `index.html` (Upload Form)

အသုံးပြုသူများ ၌ သူတို့၏ တိရစ္ဆာန်ကို ဖော်ပြနိုင်သော မူလစာမျက်နှာ -

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

**ဖိုင်:** `result.html` (ပုံပြင် ပြသမှု)

ဖန်တီးပြီး ပုံပြင်ကိုပြသသည် -

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

**ပုံစံ လက္ခဏာများ:**

1. **Thymeleaf ပေါင်းစည်းမှု**: dynamic content အတွက် `th:` attribute များကို အသုံးပြု  
2. **တုံ့ပြန်ခြင်း ဒီဇိုင်း**: မိုဘိုင်း နှင့် ဒီစကတော့ နှစ်မျိုးစလုံးအတွက် CSS style  
3. **အမှား ကင်းရှင်းမှု**: လက်ခံရိုက်ထည့်မှု အမှားများကို အသုံးပြုသူထံ ပြသ  
4. **Client-side လုပ်ဆောင်မှု**: ဓာတ်ပုံ ခွဲခြမ်းစိတ်ဖြတ်မှုအတွက် JavaScript (Transformers.js သုံး)

### ၅။ ဖွဲ့စည်းတပ်ဆင်မှု

**ဖိုင်:** `application.properties`

အက်ပ်ပလီကေးရှင်း ဆက်တင်များ -

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

**ဖွဲ့စည်းတပ်ဆင်မှု ရှင်းလင်းချက်:**

1. **ဖိုင် တင်ခြင်း**: ၁၀MB အထိ ဓာတ်ပုံများ ခွင့်ပြုမှု  
2. **မှတ်တမ်းတင်ခြင်း (Logging)**: လည်ပတ်မှုအတွင်း မှတ်တမ်းတင်ချက် ကို ထိန်းချုပ်  
3. **Azure AI Foundry**: အသုံးပြုမည့် endpoint နှင့် မော်ဒယ် ဖြန့်ချိမှုကို ဖော်ပြသည် (keyless auth)  
4. **လုံခြုံရေး**: အမှား ကင်းရှင်းမှု ဆက်တင်များကို ဝင်ရောက်ရရှိနိုင်သော သတင်းအချက်အလက် မဖော်ပြရန် ထိန်းချုပ်

## အက်ပ်ပလီကေးရှင်း စတင်လည်ပတ်ခြင်း

### အဆင့် ၁: လက်မှတ်ထိုး လက်ရေးထိုးပြီး သင့် endpoint ကို သတ်မှတ်ပါ

Authentication က keyless (Microsoft Entra ID) ဖြစ်သောကြောင့် API key မလိုပါ။ လက်မှတ်ထိုးပြီး သင့် Foundry endpoint ကို သတ်မှတ်ပါ -

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

**ဘာကြောင့် လိုအပ်သလဲ:**
- Azure AI Foundry သည် Microsoft Entra ID ဖြင့် inference တောင်းဆိုမှုများကို အတည်ပြုသည့် API  
- Keyless auth ဟူသည် သင့် စကားဝှက် မရှိဘဲ ပြန်လည် အသုံးပြုနိုင်မှု ဖြစ်သည်  
- သင့် အကောင့်မှာ **Cognitive Services OpenAI User** အခန်းကဏ္ဍ ရရှိထား ရမည်

### အဆင့် ၂: တည်ဆောက်ပြီး လည်ပတ်ပါ

ပရောဂျက် ဖိုဒားထဲ သို့ သွားပါ -
```bash
cd 04-PracticalSamples/petstory
```

အက်ပ် တည်ဆောက်ပါ -
```bash
mvn clean compile
```

ဆာဗာ စတင်ပါ -
```bash
mvn spring-boot:run
```

အက်ပ်ပလီကေးရှင်းသည် `http://localhost:8080` တွင် စတင် လည်ပတ်မည်။

### အဆင့် ၃: အက်ပ်ကို စမ်းသပ်ကြည့်ပါ

1. **ဖွင့်ပါ** `http://localhost:8080` ကို browser တွင်  
2. **သင့်တိရစ္ဆာန်ကို ဖော်ပြပါ** (ဥပမာ - "အရူးအမူးကစားရတာကြိုက်တဲ့ ရွှေရောင် ရထားရီဗာ")  
3. **နှိပ်ပါ** "Generate Story" ခလုတ်ကို AI ဖန်တီးပုံပြင် ရရှိရန်  
4. **အခြားနည်း** အဖြစ် တိရစ္ဆာန် ဓာတ်ပုံ တင်ပြီး အလိုအလျောက် ဖော်ပြချက် ရရှိစေပါ  
5. **သင့်တိရစ္ဆာန် ဖော်ပြချက်အရ ဖန်တီးထားသော ကောင်းကင်ပုံပြင်ကို ကြည့်ရန်**

## ဒါအားလုံး ဘယ်လိုပေါင်းစည်း လည်ပတ်သလဲ

သင်တိရစ္ဆာန် ပုံပြင် တစ်ပုဒ် ဖန်တီးသော အပြည့်အစုံ စဉ်ဆက်ဖြစ်ပုံ -

1. **အသုံးပြုသူ ထည့်သွင်းချက်**: အသုံးပြုသူ မှ ဝက်ဘ်ပုံစံတွင် သူ့တိရစ္ဆာန်ကို ဖော်ပြသည်  
2. **ပုံစံ တင်သွင်းခြင်း**: Browser မှ POST request ကို `/generate-story` သို့ ပေးပို့  
3. **ထိန်းချုပ်ရေးသူ လုပ်ငန်းစဉ်**: `PetController` က ဖော်ပြချက်ကို စစ်ဆေး သန့်စင်  
4. **AI ဝန်ဆောင်မှု ခေါ်ယူခြင်း**: `StoryService` မှ Azure AI Foundry မော်ဒယ် သို့ တောင်းဆို  
5. **ပုံပြင် ဖန်တီးခြင်း**: AI မှ ဖန်တီးထားသော ဖန်တီးမှုအများဆုံးပုံပြင်  
6. **တုံ့ပြန်မှု ကိုင်တွယ်မှု**: Controller က ပုံပြင်ကို လက်ခံပြီး Model ထဲ ထည့်  
7. **ပုံစံ ဖော်ပြချက်**: Thymeleaf သည် `result.html` ကို ပုံပြင်နှင့် rendering ပြုလုပ်  
8. **ပြသခြင်း**: မကြာခင် အဆိုပါ ဖန်တီးထားသော ပုံပြင်ကို အသုံးပြုသူ browser တွင် ပြသ   

**အမှား ကိုင်တွယ်မှု လမ်းစဉ်:**
AI ဝန်ဆောင်မှု မအောင်မြင်ပါက -
1. Controller က exception ကို ဖမ်းဆီး  
2. ရေးသားပြီးသား ရှေ့ပြေးပုံပြင်များဖြင့် fallback ပုံပြင် ဖန်တီး  
3. AI ဝန်ဆောင်မှု မရရှိနိုင်ခြင်း နှင့် ပတ်သက်၍ အကြောင်းကြားချက် ထည့်သွင်း ပြသ  
4. အသုံးပြုသူသည် ပုံပြင် တစ်ပုဒ် ရရှိပြီး ကောင်းမွန်သော အသုံးပြုသူ အတွေ့အကြုံ ရရှိ

## AI ပေါင်းစည်းမှု ကို နားလည်မြင်သာခြင်း

### Azure AI Foundry (keyless)
အက်ပ်သည် Azure AI Foundry ကို keyless authentication (Microsoft Entra ID) နည်းဖြင့် အသုံးပြု -

```java
// သော့မပါသောအတည်ပြုမှု - API သော့မရှိပါ။
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Prompt Engineering
ဝန်ဆောင်မှုသည် ကောင်းမွန်သော တုံ့ပြန်ချက် ရရှိရန် အထူးပြု ပြင်ဆင်ထားသည့် prompts များ သုံး -

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### တုံ့ပြန်ချက် ကိုင်တွယ်ခြင်း
AI အဖြေကို ထုတ်ယူစစ်ဆေး -

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```


## နောက်တစ်ခုပြုလုပ်ရန် အဆင့်များ

နမူနာများ ပိုမိုကြည့်ရှုရန်၊ [အခန်း 04: အသုံးဝင် နမူနာများ](../README.md) သို့ သွားကြည့်ပါ။

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ပြောကြားချက်**
ဤစာတမ်းကို AI ဘာသာပြန်ဝန်ဆောင်မှု [Co-op Translator](https://github.com/Azure/co-op-translator) အသုံးပြု၍ ဘာသာပြန်ထားပါသည်။ ကျွန်ုပ်တို့သည် တိကျမှန်ကန်မှုအတွက် ကြိုးပမ်းနေသော်လည်း၊ စက်ကိရိယာဘာသာပြန်ခြင်းများတွင် အမှားများ သို့မဟုတ် မှားယွင်းချက်များ ပါဝင်နိုင်ကြောင်း သတိပြုပါရန် လိုအပ်ပါသည်။ မူလစာတမ်းကို မူရင်းဘာသာဖြင့်သာ ယုံကြည်စိတ်ချရသော အချက်အလက်အဖြစ် သတ်မှတ်သင့်သည်။ အရေးကြီးသည့် သတင်းအချက်အလက်များအတွက် ပရော်ဖက်ရှင်နယ် လူသားဘာသာပြန်သူဝန်ဆောင်မှုကို အကြံပြုပါသည်။ ဤဘာသာပြန်ချက်ကို အသုံးပြုခြင်းမှ ဖြစ်ပေါ်လာသော နားလည်မှုကွာခြားမှုများ သို့မဟုတ် မမှန်ကန်သော အသုံးပြုမှုများအတွက် ကျွန်ုပ်တို့ တာဝန်မခံပါ။
<!-- CO-OP TRANSLATOR DISCLAIMER END -->