# Навчальний посібник зі створення історій про тварин для початківців

## Зміст

- [Вимоги](#вимоги)
- [Розуміння структури проєкту](#розуміння-структури-проєкту)
- [Пояснення основних компонентів](#пояснення-основних-компонентів)
  - [1. Головний додаток](#1-головний-додаток)
  - [2. Веб-контролер](#2-веб-контролер)
  - [3. Сервіс історій](#3-сервіс-історій)
  - [4. Веб-шаблони](#4-веб-шаблони)
  - [5. Налаштування](#5-налаштування)
- [Запуск додатку](#запуск-додатку)
- [Як це все працює разом](#як-це-все-працює-разом)
- [Розуміння інтеграції зі ШІ](#розуміння-інтеграції-зі-ші)
- [Наступні кроки](#наступні-кроки)

## Вимоги

Перед початком переконайтеся, що у вас встановлено:
- Java 21 або вище
- Maven для керування залежностями
- Розгортання моделі Azure AI Foundry (підготуйте за допомогою `azd up` — див. [Розділ 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), увійшли у систему за допомогою `az login` (авторизація без ключа)
- Базове розуміння Java, Spring Boot та веб-розробки

## Розуміння структури проєкту

Проєкт для створення історій про тварин має кілька важливих файлів:

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

## Пояснення основних компонентів

### 1. Головний додаток

**Файл:** `PetStoryApplication.java`

Це точка входу для нашого додатку Spring Boot:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Що це робить:**
- Анотація `@SpringBootApplication` включає автоматичне налаштування та сканування компонентів
- Запускає вбудований веб-сервер (Tomcat) на порту 8080
- Автоматично створює всі необхідні Spring бени та сервіси

### 2. Веб-контролер

**Файл:** `PetController.java`

Він опрацьовує всі веб-запити та взаємодії користувача:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Повертає шаблон index.html
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Перевірка введених даних
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Очищення введених даних для безпеки
        String sanitizedDescription = sanitizeInput(description);
        
        // Генерація історії з обробкою помилок
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Повертає шаблон result.html
            
        } catch (Exception e) {
            // Використовувати запасну історію, якщо ШІ не вдається
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Обмежити довжину
    }
}
```

**Основні можливості:**

1. **Обробка маршрутів**: `@GetMapping("/")` показує форму завантаження, `@PostMapping("/generate-story")` опрацьовує відправлені дані
2. **Перевірка введення**: Перевіряє порожні описи і обмеження довжини
3. **Безпека**: Очищає введені дані для запобігання XSS-атакам
4. **Обробка помилок**: Надає запасні історії, якщо сервіс ШІ не працює
5. **Зв’язок з моделлю**: Передає дані у HTML-шаблони за допомогою Spring `Model`

**Система запасних варіантів:**
Контролер містить заздалегідь написані шаблони історій, які використовуються, коли сервіс ШІ недоступний:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Використовуйте хеш опису для послідовних відповідей
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Сервіс історій

**Файл:** `StoryService.java`

Цей сервіс спілкується з Azure AI Foundry для генерації історій з ключовою аутентифікацією без застосування API-ключа:

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
        
        // Сумісна з OpenAI кінцева точка Foundry розташована під /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Безключова автентифікація з Microsoft Entra ID (без ключа API)
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
        
        // Налаштувати запит до ШІ
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Обмежити довжину відповіді
                .temperature(0.8)          // Контролювати креативність (0.0-1.0)
                .build();
        
        // Надіслати запит і отримати відповідь
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Ключові компоненти:**

1. **OpenAI клієнт**: Використовує офіційний OpenAI Java SDK, налаштований для Azure AI Foundry (авторизація без ключа)
2. **Системний запит**: Встановлює поведінку ШІ — написання сімейних історій про тварин
3. **Запит користувача**: Чітко вказує ШІ, яку історію написати на основі опису
4. **Параметри**: Керує довжиною історії та рівнем креативності
5. **Обробка помилок**: Генерує виключення, які контролер ловить і обробляє

### 4. Веб-шаблони

**Файл:** `index.html` (Форма завантаження)

Головна сторінка, де користувачі описують своїх тварин:

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

**Файл:** `result.html` (Відображення історії)

Показує згенеровану історію:

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

**Особливості шаблону:**

1. **Інтеграція Thymeleaf**: Використання атрибутів `th:` для динамічного контенту
2. **Адаптивний дизайн**: CSS-стилі для мобільних та десктопних пристроїв
3. **Обробка помилок**: Відображає повідомлення про помилки для користувачів
4. **Обробка на клієнті**: JavaScript для аналізу зображень (з використанням Transformers.js)

### 5. Налаштування

**Файл:** `application.properties`

Налаштування додатку:

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

**Пояснення конфігурації:**

1. **Завантаження файлів**: Дозволено завантаження зображень до 10 МБ
2. **Логування**: Керує інформацією, яка логуватиметься під час виконання
3. **Azure AI Foundry**: Вказує точку доступу та розгортання моделі для використання (авторизація без ключа)
4. **Безпека**: Налаштування обробки помилок, щоб не розкривати конфіденційні дані

## Запуск додатку

### Крок 1: Увійдіть і встановіть свою точку доступу

Авторизація — без ключа (Microsoft Entra ID), тому немає API-ключа. Увійдіть у систему та встановіть свою точку доступу Foundry:

**Windows (Командний рядок):**
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

**Чому це потрібно:**
- Azure AI Foundry використовує Microsoft Entra ID для аутентифікації запитів на висновки
- Авторизація без ключа означає, що у вашому коді чи середовищі немає секретів
- Ваш обліковий запис повинен мати роль **Cognitive Services OpenAI User** на ресурсі

### Крок 2: Побудуйте та запустіть додаток

Перейдіть у каталог проєкту:
```bash
cd 04-PracticalSamples/petstory
```

Побудуйте додаток:
```bash
mvn clean compile
```

Запустіть сервер:
```bash
mvn spring-boot:run
```

Додаток буде запущений на `http://localhost:8080`.

### Крок 3: Тестування додатку

1. **Відкрийте** `http://localhost:8080` у браузері
2. **Опишіть** свого улюбленця в текстовому полі (наприклад, «Жвавий золотистий ретрівер, який любить приносити м’ячик»)
3. **Натисніть** "Generate Story" для отримання історії, створеної ШІ
4. **Або** завантажте зображення тварини для автоматичного створення опису
5. **Перегляньте** креативну історію, створену на основі опису вашого улюбленця

## Як це все працює разом

Ось повний процес створення історії про тварину:

1. **Введення користувачем**: Ви описуєте свого улюбленця у веб-формі
2. **Відправлення форми**: Браузер надсилає POST-запит на `/generate-story`
3. **Обробка контролером**: `PetController` перевіряє та очищує введені дані
4. **Виклик сервісу ШІ**: `StoryService` надсилає запит до моделі Azure AI Foundry
5. **Генерація історії**: ШІ створює креативну історію на основі опису
6. **Обробка відповіді**: Контролер отримує історію та додає її у модель
7. **Рендеринг шаблону**: Thymeleaf відображає `result.html` з історією
8. **Відображення**: Користувач бачить згенеровану історію у браузері

**Обробка помилок:**
Якщо сервіс ШІ не працює:
1. Контролер ловить виключення
2. Генерує запасну історію заздалегідь підготовленими шаблонами
3. Відображає запасну історію з повідомленням про недоступність ШІ
4. Користувач все одно отримує історію, що забезпечує хороший досвід

## Розуміння інтеграції зі ШІ

### Azure AI Foundry (авторизація без ключа)
Додаток використовує Azure AI Foundry з авторизацією без ключа (Microsoft Entra ID):

```java
// Аутентифікація без ключа - без API ключа
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Проектування запитів
Сервіс використовує ретельно створені запити для отримання хороших результатів:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Обробка відповіді
Відповідь ШІ витягується і перевіряється:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Наступні кроки

Для більшої кількості прикладів дивіться [Розділ 04: Практичні приклади](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Відмова від відповідальності**:
Цей документ було перекладено за допомогою сервісу штучного інтелекту для перекладу [Co-op Translator](https://github.com/Azure/co-op-translator). Хоча ми прагнемо до точності, будь ласка, майте на увазі, що автоматичні переклади можуть містити помилки або неточності. Оригінальний документ рідною мовою слід вважати авторитетним джерелом. Для критично важливої інформації рекомендується професійний людський переклад. Ми не несемо відповідальності за будь-які непорозуміння або неправильні тлумачення, що виникли внаслідок використання цього перекладу.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->