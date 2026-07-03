# Урок за Генератор на Истории с Домашни Любимци за Начинаещи

## Съдържание

- [Предварителни изисквания](#предварителни-изисквания)
- [Разбиране на структурата на проекта](#разбиране-на-структурата-на-проекта)
- [Обяснение на основните компоненти](#обяснение-на-основните-компоненти)
  - [1. Основно приложение](#1-основно-приложение)
  - [2. Уеб контролер](#2-уеб-контролер)
  - [3. Story Service](#3-story-service)
  - [4. Уеб шаблони](#4-уеб-шаблони)
  - [5. Конфигурация](#5-конфигурация)
- [Стартиране на приложението](#стартиране-на-приложението)
- [Как всичко работи заедно](#как-всичко-работи-заедно)
- [Разбиране на AI интеграцията](#разбиране-на-ai-интеграцията)
- [Следващи стъпки](#следващи-стъпки)

## Предварителни изисквания

Преди да започнете, уверете се, че имате:
- Инсталиран Java 21 или по-нова версия
- Maven за управление на зависимости
- Разгърнат модел на Azure AI Foundry (осигурете го с `azd up` — вижте [Глава 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), влезли сте с `az login` (авторизация без ключ)
- Основни познания по Java, Spring Boot и уеб разработка

## Разбиране на структурата на проекта

Проектът за истории с домашни любимци има няколко важни файла:

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

## Обяснение на основните компоненти

### 1. Основно приложение

**Файл:** `PetStoryApplication.java`

Това е входната точка за нашето Spring Boot приложение:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Какво прави:**
- Анотацията `@SpringBootApplication` активира автоматично конфигуриране и сканиране на компоненти
- Стартира вграден уеб сървър (Tomcat) на порт 8080
- Автоматично създава всички необходими Spring бинове и услуги

### 2. Уеб контролер

**Файл:** `PetController.java`

Той обработва всички уеб заявки и взаимодействия с потребителите:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Връща шаблона index.html
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Валидация на входните данни
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Почиства входните данни за сигурност
        String sanitizedDescription = sanitizeInput(description);
        
        // Генерира история с обработка на грешки
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Връща шаблона result.html
            
        } catch (Exception e) {
            // Използва резервна история, ако AI се провали
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Ограничаване на дължината
    }
}
```

**Основни функции:**

1. **Обработка на маршрути:** `@GetMapping("/")` показва формата за качване, `@PostMapping("/generate-story")` обработва изпращанията
2. **Валидиране на входни данни:** Проверява за празни описания и ограничения на дължина
3. **Сигурност:** Санитизира потребителския вход за предотвратяване на XSS атаки
4. **Обработка на грешки:** Осигурява резервни истории, когато AI услугата не работи
5. **Привързване на модел:** Предава данни към HTML шаблоните чрез Spring `Model`

**Система за резервни варианти:**
Контролерът включва предварително написани шаблони за истории, които се използват, когато AI услугата не е налична:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Използвайте хеш на описанието за последователни отговори
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Story Service

**Файл:** `StoryService.java`

Тази услуга комуникира с Azure AI Foundry за генериране на истории с използване на авторизация без ключ:

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
        
        // OpenAI-съвместимата крайна точка на Foundry се намира под /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Удостоверяване без ключ с Microsoft Entra ID (без API ключ)
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
        
        // Конфигурирайте AI заявката
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Ограничете дължината на отговора
                .temperature(0.8)          // Контролирайте креативността (0.0-1.0)
                .build();
        
        // Изпратете заявката и вземете отговора
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Ключови компоненти:**

1. **OpenAI клиент:** Използва официалния OpenAI Java SDK, конфигуриран за Azure AI Foundry (без ключ)
2. **Системен подсказ:** Задава поведението на AI да пише истории за домашни любимци, подходящи за семейна аудитория
3. **Потребителски подсказ:** Казва на AI точно коя история да напише въз основа на описанието
4. **Параметри:** Контролира дължината и степента на креативност на историята
5. **Обработка на грешки:** Хвърля изключения, които контролерът прихваща и обработва

### 4. Уеб шаблони

**Файл:** `index.html` (форма за качване)

Основната страница, на която потребителите описват своите домашни любимци:

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

**Файл:** `result.html` (показване на история)

Показва генерираната история:

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

**Функционалности на шаблона:**

1. **Интеграция с Thymeleaf:** Използва атрибути `th:` за динамично съдържание
2. **Отзивчив дизайн:** CSS стилове за мобилни устройства и настолни компютри
3. **Обработка на грешки:** Показва грешки по време на валидиране на потребителите
4. **Обработка на клиентската страна:** JavaScript за анализ на изображения (използвайки Transformers.js)

### 5. Конфигурация

**Файл:** `application.properties`

Конфигурационни настройки за приложението:

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

**Обяснение на конфигурацията:**

1. **Качване на файлове:** Позволява изображения до 10MB
2. **Логване:** Контролира каква информация се логва по време на изпълнение
3. **Azure AI Foundry:** Посочва крайна точка и разгънат модел за използване (без ключ)
4. **Сигурност:** Конфигурация за обработка на грешки, за да не се излагат чувствителни данни

## Стартиране на приложението

### Стъпка 1: Вход и настройка на крайна точка

Авторизацията е без ключ (Microsoft Entra ID), затова няма API ключ. Влезте и задайте крайна точка за Foundry:

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

**Защо е нужно това:**
- Azure AI Foundry използва Microsoft Entra ID за удостоверяване на заявки за инференс
- Безключовата авторизация означава, че няма тайни в изходния код или средата
- Вашият акаунт трябва да има роля **Cognitive Services OpenAI User** за ресурса

### Стъпка 2: Компилиране и стартиране

Навигирайте до директорията на проекта:
```bash
cd 04-PracticalSamples/petstory
```

Компилирайте приложението:
```bash
mvn clean compile
```

Стартирайте сървъра:
```bash
mvn spring-boot:run
```

Приложението ще стартира на `http://localhost:8080`.

### Стъпка 3: Тествайте приложението

1. **Отворете** `http://localhost:8080` в браузъра
2. **Опишете** вашия домашен любимец в текстовото поле (например "Игрив златист ретрийвър, който обича да донася топки")
3. **Натиснете** "Генерирай история", за да получите AI-генерирана история
4. **Алтернативно**, качете снимка на домашния любимец, за да се генерира автоматично описание
5. **Прегледайте** креативната история, базирана на описанието на вашия любимец

## Как всичко работи заедно

Ето пълния процес при генериране на история с домашен любимец:

1. **Потребителски вход:** Вие описвате домашния си любимец във формата в уеб страницата
2. **Изпращане на формата:** Браузърът изпраща POST заявка към `/generate-story`
3. **Обработка в контролера:** `PetController` валидира и санитизира входа
4. **Извикване на AI услугата:** `StoryService` изпраща заявка към модела на Azure AI Foundry
5. **Генериране на история:** AI създава креативна история въз основа на описанието
6. **Обработка на отговора:** Контролерът получава историята и я добавя в модела
7. **Изобразяване на шаблона:** Thymeleaf рендерира `result.html` с историята
8. **Покажи:** Потребителят вижда генерираната история в браузъра си

**Процес при грешка:**
Ако AI услугата се провали:
1. Контролерът прихваща изключението
2. Генерира резервна история с използване на предварително написани шаблони
3. Показва резервната история с бележка, че AI не е наличен
4. Потребителят все пак получава история, осигурявайки добро потребителско преживяване

## Разбиране на AI интеграцията

### Azure AI Foundry (без ключ)
Приложението използва Azure AI Foundry с безключова авторизация (Microsoft Entra ID):

```java
// Безключова автентикация - без API ключ
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Създаване на подскази
Услугата използва внимателно създадени подскази за добри резултати:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Обработка на отговора
Отговорът от AI се извлича и валидира:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Следващи стъпки

За още примери вижте [Глава 04: Практически примери](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Отказ от отговорност**:
Този документ е преведен с помощта на AI преводачески услуга [Co-op Translator](https://github.com/Azure/co-op-translator). Въпреки че се стремим към точност, моля имайте предвид, че автоматизираните преводи могат да съдържат грешки или неточности. Оригиналният документ на неговия роден език трябва да се счита за авторитетен източник. За критична информация се препоръчва професионален човешки превод. Ние не носим отговорност за каквито и да е недоразумения или неправилни тълкувания, произтичащи от използването на този превод.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->