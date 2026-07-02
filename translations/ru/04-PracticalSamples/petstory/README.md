# Учебник по генератору историй о питомцах для начинающих

## Содержание

- [Требования](#требования)
- [Понимание структуры проекта](#понимание-структуры-проекта)
- [Объяснение основных компонентов](#объяснение-основных-компонентов)
  - [1. Основное приложение](#1-основное-приложение)
  - [2. Веб-контроллер](#2-веб-контроллер)
  - [3. Сервис историй](#3-сервис-историй)
  - [4. Веб-шаблоны](#4-веб-шаблоны)
  - [5. Конфигурация](#5-конфигурация)
- [Запуск приложения](#запуск-приложения)
- [Как все работает вместе](#как-все-работает-вместе)
- [Понимание интеграции ИИ](#понимание-интеграции-ии)
- [Следующие шаги](#следующие-шаги)

## Требования

Перед началом убедитесь, что у вас есть:
- Установлена Java 21 или выше
- Maven для управления зависимостями
- Развертывание модели Azure AI Foundry (настройте с помощью `azd up` — смотрите [Главу 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), вошли в систему через `az login` (аутентификация без ключа)
- Базовое понимание Java, Spring Boot и веб-разработки

## Понимание структуры проекта

Проект с историей о питомцах содержит несколько важных файлов:

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

## Объяснение основных компонентов

### 1. Основное приложение

**Файл:** `PetStoryApplication.java`

Это точка входа в наше приложение Spring Boot:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Что это делает:**
- Аннотация `@SpringBootApplication` включает автоконфигурацию и сканирование компонентов
- Запускает встроенный веб-сервер (Tomcat) на порту 8080
- Автоматически создает все необходимые бины и сервисы Spring

### 2. Веб-контроллер

**Файл:** `PetController.java`

Обрабатывает все веб-запросы и взаимодействия с пользователем:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Возвращает шаблон index.html
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Проверка входных данных
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Очистка входных данных для безопасности
        String sanitizedDescription = sanitizeInput(description);
        
        // Генерация истории с обработкой ошибок
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Возвращает шаблон result.html
            
        } catch (Exception e) {
            // Использовать запасную историю, если ИИ не справится
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Ограничить длину
    }
}
```

**Ключевые функции:**

1. **Обработка маршрутов**: `@GetMapping("/")` показывает форму загрузки, `@PostMapping("/generate-story")` обрабатывает отправку
2. **Проверка ввода**: Проверяет пустые описания и ограничения по длине
3. **Безопасность**: Очищает пользовательский ввод для предотвращения XSS-атак
4. **Обработка ошибок**: Предоставляет запасные истории при сбоях сервиса ИИ
5. **Связывание модели**: Передает данные в HTML-шаблоны с помощью модели Spring

**Система запасных вариантов:**
Контроллер включает заранее написанные шаблоны историй, которые используются, когда сервис ИИ недоступен:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Используйте хеш описания для согласованных ответов
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Сервис историй

**Файл:** `StoryService.java`

Этот сервис связывается с Azure AI Foundry для создания историй с использованием аутентификации без ключа:

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
        
        // Совместимая с OpenAI конечная точка Foundry находится по адресу /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Аутентификация без ключа с помощью Microsoft Entra ID (без API ключа)
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
        
        // Настроить запрос к ИИ
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Ограничить длину ответа
                .temperature(0.8)          // Контроль креативности (0.0-1.0)
                .build();
        
        // Отправить запрос и получить ответ
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Ключевые компоненты:**

1. **OpenAI клиент**: Использует официальный OpenAI Java SDK, настроенный для Azure AI Foundry (аутентификация без ключа)
2. **Промт системы**: Устанавливает поведение ИИ для написания семейных историй о питомцах
3. **Промт пользователя**: Инструктирует ИИ, какую именно историю писать на основе описания
4. **Параметры**: Управляют длиной истории и уровнем креативности
5. **Обработка ошибок**: Генерирует исключения, которые обрабатывает контроллер

### 4. Веб-шаблоны

**Файл:** `index.html` (форма загрузки)

Главная страница, где пользователи описывают своих питомцев:

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

**Файл:** `result.html` (показ истории)

Отображает сгенерированную историю:

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

**Особенности шаблона:**

1. **Интеграция с Thymeleaf**: Использует атрибуты `th:` для динамического контента
2. **Адаптивный дизайн**: Стили CSS для мобильных и настольных устройств
3. **Обработка ошибок**: Показывает пользователям ошибки валидации
4. **Обработка на стороне клиента**: JavaScript для анализа изображений (с использованием Transformers.js)

### 5. Конфигурация

**Файл:** `application.properties`

Настройки конфигурации приложения:

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

**Объяснение конфигурации:**

1. **Загрузка файлов**: Разрешает изображения размером до 10 МБ
2. **Логирование**: Управляет информацией, которая записывается во время работы
3. **Azure AI Foundry**: Указывает конечную точку и развернутую модель для использования (аутентификация без ключа)
4. **Безопасность**: Настройки обработки ошибок, чтобы избежать раскрытия конфиденциальной информации

## Запуск приложения

### Шаг 1: Вход и установка конечной точки

Аутентификация осуществляется без ключа (Microsoft Entra ID), поэтому API ключ отсутствует. Выполните вход и установите конечную точку Foundry:

**Windows (командная строка):**
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

**Почему это нужно:**
- Azure AI Foundry использует Microsoft Entra ID для аутентификации запросов на вывод
- Аутентификация без ключа означает отсутствие секретов в исходном коде или окружении
- Ваша учетная запись должна иметь роль **Cognitive Services OpenAI User** на ресурсе

### Шаг 2: Сборка и запуск

Перейдите в каталог проекта:
```bash
cd 04-PracticalSamples/petstory
```

Соберите приложение:
```bash
mvn clean compile
```

Запустите сервер:
```bash
mvn spring-boot:run
```

Приложение запустится по адресу `http://localhost:8080`.

### Шаг 3: Тестирование приложения

1. **Откройте** `http://localhost:8080` в браузере
2. **Опишите** вашего питомца в текстовом поле (например, "Игривый золотистый ретривер, который любит приносить палку")
3. **Нажмите** "Generate Story" для получения истории, созданной ИИ
4. **Либо** загрузите фотографию питомца для автоматического создания описания
5. **Просмотрите** творческую историю на основе описания вашего питомца

## Как все работает вместе

Вот полный процесс генерации истории о питомце:

1. **Ввод пользователя**: Вы описываете питомца в веб-форме
2. **Отправка формы**: Браузер отправляет POST-запрос на `/generate-story`
3. **Обработка контроллером**: `PetController` проверяет и очищает ввод
4. **Вызов сервиса ИИ**: `StoryService` отправляет запрос модели Azure AI Foundry
5. **Генерация истории**: ИИ создает креативную историю на основе описания
6. **Обработка ответа**: Контроллер получает историю и добавляет её в модель
7. **Отрисовка шаблона**: Thymeleaf формирует страницу `result.html` с историей
8. **Отображение**: Пользователь видит сгенерированную историю в браузере

**Обработка ошибок:**
Если сервис ИИ не работает:
1. Контроллер перехватывает исключение
2. Создает запасную историю на основе заранее написанных шаблонов
3. Показывает запасную историю с пометкой о недоступности ИИ
4. Пользователь всё равно получает историю, обеспечивая хороший пользовательский опыт

## Понимание интеграции ИИ

### Azure AI Foundry (аутентификация без ключа)
Приложение использует Azure AI Foundry с аутентификацией без ключа (Microsoft Entra ID):

```java
// Аутентификация без ключа - без API ключа
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Промт-инжиниринг
Сервис применяет тщательно подготовленные промты для получения качественного результата:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Обработка ответа
Извлечение и проверка ответа ИИ:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Следующие шаги

Для получения дополнительных примеров смотрите [Глава 04: Практические примеры](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Отказ от ответственности**:
Этот документ был переведен с использованием сервиса машинного перевода [Co-op Translator](https://github.com/Azure/co-op-translator). Несмотря на наши усилия по обеспечению точности, имейте в виду, что автоматический перевод может содержать ошибки или неточности. Оригинальный документ на его исходном языке следует считать авторитетным источником. Для получения критически важной информации рекомендуется обратиться к профессиональному человеческому переводу. Мы не несем ответственности за любые недоразумения или неправильные толкования, возникшие в результате использования этого перевода.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->