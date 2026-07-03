# Туторијал за генератор прича о кућним љубимцима за почетнике

## Садржај

- [Претпоставке](#претпоставке)
- [Разумевање структуре пројекта](#разумевање-структуре-пројекта)
- [Објашњење основних компоненти](#објашњење-основних-компоненти)
  - [1. Главна апликација](#1-главна-апликација)
  - [2. Веб контролер](#2-веб-контролер)
  - [3. Сервис за приче](#3-сервис-за-приче)
  - [4. Веб шаблони](#4-веб-шаблони)
  - [5. Конфигурација](#5-конфигурација)
- [Покретање апликације](#покретање-апликације)
- [Како све функционише заједно](#како-све-функционише-заједно)
- [Разумевање интеграције вештачке интелигенције](#разумевање-интеграције-вештачке-интелигенције)
- [Следећи кораци](#следећи-кораци)

## Претпоставке

Пре почетка, проверите да ли имате:
- Јава 21 или новију инсталацију
- Maven за управљање зависностима
- Деплојмент модела Azure AI Foundry (поставите га помоћу `azd up` — видите [Поглавље 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), пријављени помоћу `az login` (аутентификација без кључа)
- Основно разумевање Јаве, Spring Boot-а и веб развоја

## Разумевање структуре пројекта

Пројекат за приче о кућним љубимцима садржи неколико важних фајлова:

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

## Објашњење основних компоненти

### 1. Главна апликација

**Фајл:** `PetStoryApplication.java`

Ово је улазна тачка наше Spring Boot апликације:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Шта ово ради:**
- Анотација `@SpringBootApplication` омогућава аутоматску конфигурацију и скенирање компоненти
- Покреће уграђени веб сервер (Tomcat) на порту 8080
- Аутоматски креира све потребне Spring bean-ове и сервисе

### 2. Веб контролер

**Фајл:** `PetController.java`

Ово рутира све веб захтеве и интеракције са корисником:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Враћа index.html шаблон
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Валидација уноса
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Очисти унос ради безбедности
        String sanitizedDescription = sanitizeInput(description);
        
        // Генериши причу са обрадом грешака
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Враћа result.html шаблон
            
        } catch (Exception e) {
            // Користи резервну причу ако вештачка интелигенција не успе
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Ограничење дужине
    }
}
```

**Кључне карактеристике:**

1. **Роутинг:** `@GetMapping("/")` приказује форму за отпремање, `@PostMapping("/generate-story")` обрађује слање
2. **Валидација уноса:** Провера празних описа и граница дужине
3. **Безбедност:** Санитизација корисничког уноса ради спречавања XSS напада
4. **Обрада грешака:** Обезбеђује резервне приче када AI сервис не ради
5. **Биндинг модела:** Прослеђује податке HTML шаблонима користећи Spring `Model`

**Резервни систем:**
Контролер садржи унапред написане шаблоне прича који се користе када AI сервис није доступан:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Користите хеш описa за доследне одговоре
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Сервис за приче

**Фајл:** `StoryService.java`

Овај сервис комуницира са Azure AI Foundry-јем да генерише приче користећи аутентификацију без кључа:

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
        
        // OpenAI-компатибилна тачка приступа Foundry-ја се налази под /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Аутентификација без кључа помоћу Microsoft Entra ID (без API кључа)
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
        
        // Конфигуришите захтев вештачке интелигенције
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Ограничите дужину одговора
                .temperature(0.8)          // Контролишите креативност (0.0-1.0)
                .build();
        
        // Пошаљите захтев и добијте одговор
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Кључне компоненте:**

1. **OpenAI клијент:** Користи званични OpenAI Java SDK конфигурисан за Azure AI Foundry (без кључа)
2. **Системски промпт:** Поставља понашање AI-ја да пише породичне и пријатељске приче о љубимцима
3. **Кориснички промпт:** Тиме AI-ју прецизно описује коју причу треба да напише на основу описа
4. **Параметри:** Контролишу дужину приче и ниво креативности
5. **Обрада грешака:** Баца изузетке које контролер хвата и обрађује

### 4. Веб шаблони

**Фајл:** `index.html` (форма за отпремање)

Главна страница где корисници описују своје љубимце:

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

**Фајл:** `result.html` (приказ приче)

Приказује генерисану причу:

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

**Особине шаблона:**

1. **Thymeleaf интеграција:** Користи `th:` атрибуте за динамички садржај
2. **Респонзиван дизајн:** CSS стилови за мобилне и десктоп уређаје
3. **Обрада грешака:** Приказује валидирајуће грешке корисницима
4. **Обрада са стране клијента:** JavaScript за анализу слика (користећи Transformers.js)

### 5. Конфигурација

**Фајл:** `application.properties`

Подешавања конфигурације апликације:

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

**Објашњење конфигурације:**

1. **Отпремање фајлова:** Дозвољава слике до 10MB
2. **Логовање:** Контролише које информације се бележе током извршавања
3. **Azure AI Foundry:** Одређује endpoint и модел за коришћење (аутентификација без кључа)
4. **Безбедност:** Конфигурише обраду грешака како не би откривала поверљиве информације

## Покретање апликације

### Корак 1: Пријавите се и подесите ваш Endpoint

Аутентификација је без кључа (Microsoft Entra ID), па нема потребе за API кључем. Пријавите се и подесите ваш Foundry endpoint:

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

**Зашто је ово потребно:**
- Azure AI Foundry користи Microsoft Entra ID за аутентификацију захтева за инференцу
- Аутентификација без кључа значи да нема поверљивих података у вашем изворном коду или окружењу
- Ваш налог мора имати улогу **Cognitive Services OpenAI User** на ресурсу

### Корак 2: Саставите и покрените

Идите у директоријум пројекта:
```bash
cd 04-PracticalSamples/petstory
```

Саставите апликацију:
```bash
mvn clean compile
```

Покрените сервер:
```bash
mvn spring-boot:run
```

Апликација ће бити доступна на `http://localhost:8080`.

### Корак 3: Тестирајте апликацију

1. **Отворите** `http://localhost:8080` у вашем прегледачу
2. **Опишите** вашег љубимца у текст пољу (нпр. „Весео златни ретривер који воли да доноси лопту“)
3. **Кликните** на „Генериши причу“ да добијете AI генерисану причу
4. **Алтернативно**, отпремите слику љубимца да би аутоматски генерисали опис
5. **Погледајте** креативну причу базирану на опису вашег љубимца

## Како све функционише заједно

Ево комплетног тока када генеришете причу о љубимцу:

1. **Кориснички унос**: описујете вашег љубимца на веб форми
2. **Слање форме**: прегледач шаље POST захтев на `/generate-story`
3. **Обрада контролера**: `PetController` валида и санитизује унос
4. **Позив AI сервиса**: `StoryService` шаље захтев Azure AI Foundry моделу
5. **Генерисање приче**: AI генерише креативну причу на основу описа
6. **Обрада одговора**: контролер прими причу и додаје је у модел
7. **Рендеровање шаблона**: Thymeleaf приказује `result.html` са причом
8. **Приказ**: корисник види генерисану причу у свом прегледачу

**Ток обраде грешака:**
Ако AI сервис не ради:
1. Контролер хвата изузетак
2. Генерише резервну причу користећи унапред написане шаблоне
3. Приказује резервну причу са напоменом о недоступности AI-а
4. Корисник ипак добија причу, обезбеђујући добар кориснички доживљај

## Разумевање интеграције вештачке интелигенције

### Azure AI Foundry (без кључа)
Апликација користи Azure AI Foundry са аутентификацијом без кључа (Microsoft Entra ID):

```java
// Аутентификација без кључа - нема API кључа
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Prompt inženjering
Сервис користи пажљиво сачињене промптове да добије добре резултате:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Обрада одговора
AI одговор се издваја и верификује:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Следећи кораци

За више примера погледајте [Поглавље 04: Практични примери](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Изјава о одрицању одговорности**:
Овај документ је преведен коришћењем услуге за аутоматски превод [Co-op Translator](https://github.com/Azure/co-op-translator). Иако тежимо тачности, имајте у виду да аутоматски преводи могу садржати грешке или нетачности. Оригинални документ на његовом изворном језику треба сматрати ауторитативним извором. За критичне информације препоручује се професионални људски превод. Нисмо одговорни за било каква неспоразума или погрешна тумачења која произилазе из коришћења овог превода.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->