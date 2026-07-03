# מדריך ליצירת סיפורי חיות מחמד למתחילים

## תוכן העניינים

- [דרישות מוקדמות](#דרישות-מוקדמות)
- [הבנת מבנה הפרויקט](#הבנת-מבנה-הפרויקט)
- [הסבר על הרכיבים הבסיסיים](#הסבר-על-הרכיבים-הבסיסיים)
  - [1. היישום הראשי](#1-היישום-הראשי)
  - [2. בקר רשת](#2-בקר-רשת)
  - [3. שירות סיפורים](#3-שירות-סיפורים)
  - [4. תבניות רשת](#4-תבניות-רשת)
  - [5. הגדרות](#5-הגדרות)
- [הפעלת היישום](#הפעלת-היישום)
- [איך הכל עובד ביחד](#איך-הכל-עובד-ביחד)
- [הבנת האינטגרציה עם ה-AI](#הבנת-האינטגרציה-עם-ה-ai)
- [שלבים הבאים](#שלבים-הבאים)

## דרישות מוקדמות

לפני שמתחילים, וודאו שיש לכם:
- Java 21 או גרסה גבוהה יותר מותקנת
- Maven לניהול התלויות
- פריסת מודל Azure AI Foundry (הפעילו עם `azd up` — ראה [פרק 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), מחוברים עם `az login` (אימות ללא מפתח)
- הבנה בסיסית ב-Java, Spring Boot ופיתוח רשת

## הבנת מבנה הפרויקט

לפרויקט סיפורי חיות המחמד יש מספר קבצים חשובים:

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

## הסבר על הרכיבים הבסיסיים

### 1. היישום הראשי

**קובץ:** `PetStoryApplication.java`

זהו נקודת הכניסה ליישום ה-Spring Boot שלנו:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**מה זה עושה:**
- האנוטציה `@SpringBootApplication` מאפשרת קונפיגורציה אוטומטית וסריקת רכיבים
- מפעילה שרת אינטגרטיבי (Tomcat) בריבוט 8080
- יוצרת באופן אוטומטי את כל הבאנים והשירותים הדרושים של Spring

### 2. בקר רשת

**קובץ:** `PetController.java`

מתמודד עם כל בקשות הרשת ואינטראקציות המשתמש:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // מחזיר את תבנית index.html
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // אימות קלט
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // ניקוי הקלט לאבטחה
        String sanitizedDescription = sanitizeInput(description);
        
        // יצירת סיפור עם טיפול בשגיאות
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // מחזיר את תבנית result.html
            
        } catch (Exception e) {
            // השתמש בסיפור חלופי אם ה-AI נכשל
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // הגבלת אורך
    }
}
```

**תכונות עיקריות:**

1. **ניהול נתיבים**: `@GetMapping("/")` מציג את טופס ההעלאה, `@PostMapping("/generate-story")` מעבד הגשות
2. **אימות קלט**: בודק תיאורים ריקים והגבלת אורך
3. **אבטחה**: מנקה את קלט המשתמש כדי למנוע התקפות XSS
4. **ניהול שגיאות**: מספק סיפורים חלופיים כאשר שירות ה-AI נכשל
5. **קשירת מודל**: מעביר נתונים לתבניות HTML בעזרת `Model` של Spring

**מערכת גיבוי:**
הבקר כולל תבניות סיפורים מוקדמות שמשמשות כשהשירות AI אינו זמין:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // השתמש ב-hash של התיאור לתשובות עקביות
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. שירות סיפורים

**קובץ:** `StoryService.java`

שירות זה מתקשר עם Azure AI Foundry ליצירת סיפורים עם אימות ללא מפתח:

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
        
        // נקודת הקצה התואמת ל-OpenAI של Foundry נמצאת תחת /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // אימות ללא מפתח עם Microsoft Entra ID (ללא מפתח API)
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
        
        // הגדר את בקשת ה-AI
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // הגבל את אורך התגובה
                .temperature(0.8)          // שלוט ביצירתיות (0.0-1.0)
                .build();
        
        // שלח בקשה וקבל תגובה
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**רכיבים עיקריים:**

1. **לקוח OpenAI**: משתמש ב-SDK הרשמי של OpenAI ב-Java, מוגדר ל-Azure AI Foundry (ללא מפתח)
2. **הנחיית מערכת**: מגדיר את התנהגות ה-AI לכתיבת סיפורי חיות משפחתיים
3. **הנחיית משתמש**: מורה ל-AI מה לספר בדיוק לפי התיאור
4. **פרמטרים**: שולט על אורך הסיפור ורמת היצירתיות
5. **ניהול שגיאות**: זורק חריגות שהבקר תופס ומטפל בהן

### 4. תבניות רשת

**קובץ:** `index.html` (טופס העלאה)

העמוד הראשי שבו משתמשים מתארים את חיית המחמד שלהם:

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

**קובץ:** `result.html` (הצגת סיפור)

מציג את הסיפור שנוצר:

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

**תכונות התבנית:**

1. **אינטגרציה עם Thymeleaf**: משתמש בתכונות `th:` לתוכן דינמי
2. **עיצוב רספונסיבי**: סגנונות CSS למובייל ודסקטופ
3. **ניהול שגיאות**: מציג הודעות אימות למשתמשים
4. **עיבוד בצד הלקוח**: JavaScript לניתוח תמונה (באמצעות Transformers.js)

### 5. הגדרות

**קובץ:** `application.properties`

הגדרות התצורה של היישום:

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

**הסבר על ההגדרות:**

1. **העלאת קבצים**: מאפשר תמונות עד 10MB
2. **רישומונים (לוגים)**: שולט על אילו מידע נרשם בפעילות
3. **Azure AI Foundry**: מגדיר את נקודת הקצה ופריסת המודל לשימוש (אימות ללא מפתח)
4. **אבטחה**: הגדרת טיפול בשגיאות למניעת חשיפת מידע רגיש

## הפעלת היישום

### שלב 1: התחברו והגדירו את נקודת הקצה שלכם

האימות הוא ללא מפתח (Microsoft Entra ID), לכן אין מפתח API. התחברו והגדירו את נקודת הקצה של Foundry:

**Windows (שורת הפקודה):**
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

**למה זה צריך:**
- Azure AI Foundry משתמש ב-Microsoft Entra ID לאימות בקשות חישוב
- אימות ללא מפתח אומר שאין סודות בקוד המקור או בסביבה
- על החשבון שלכם להיות עם תפקיד **Cognitive Services OpenAI User** במשאב

### שלב 2: בניה והפעלה

נווטו לספריית הפרויקט:
```bash
cd 04-PracticalSamples/petstory
```

בנו את היישום:
```bash
mvn clean compile
```

התחילו את השרת:
```bash
mvn spring-boot:run
```

היישום יפעל בכתובת `http://localhost:8080`.

### שלב 3: בדקו את היישום

1. **פתחו** את `http://localhost:8080` בדפדפן שלכם
2. **תארו** את חיית המחמד שלכם באזור הטקסט (למשל, "רטריבר גולדן שובב שאוהב להביא כדור")
3. **לחצו** על "Generate Story" כדי לקבל סיפור שיצר ה-AI
4. **באופן חלופי**, העלו תמונת חיית מחמד ליצירת תיאור אוטומטית
5. **צפו** בסיפור היצירתי בהתאם לתיאור של חיית המחמד שלכם

## איך הכל עובד ביחד

הנה הזרימה המלאה כשאתם יוצרים סיפור חיית מחמד:

1. **קלט משתמש**: אתם מתארים את חיית המחמד בטופס האינטרנט
2. **שליחת טופס**: הדפדפן שולח בקשת POST ל-`/generate-story`
3. **עיבוד הבקר**: `PetController` מאמת ומנקה את הקלט
4. **קריאה לשירות AI**: `StoryService` שולח בקשה למודל ב-Azure AI Foundry
5. **יצירת סיפור**: ה-AI יוצר סיפור יצירתי לפי התיאור
6. **טיפול בתגובה**: הבקר מקבל את הסיפור ומוסיף אותו למודל
7. **הצגת תבנית**: Thymeleaf מרנדר את `result.html` עם הסיפור
8. **הצגה**: המשתמש רואה את הסיפור בדפדפן שלו

**זרימת טיפול בשגיאות:**
אם שירות ה-AI נכשל:
1. הבקר תופס את החריגה
2. יוצר סיפור חלופי מתבניות כתובות מראש
3. מציג את הסיפור החלופי עם הערה על אי זמינות ה-AI
4. המשתמש עדיין מקבל סיפור, לשמירת חוויית משתמש טובה

## הבנת האינטגרציה עם ה-AI

### Azure AI Foundry (ללא מפתח)
היישום משתמש ב-Azure AI Foundry עם אימות ללא מפתח (Microsoft Entra ID):

```java
// אימות ללא מפתח - ללא מפתח API
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### הנחיית פרומפט (Prompt Engineering)
השירות משתמש בפרומפטים מעוצבים בקפידה לקבלת תוצאות טובות:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### עיבוד תגובה
תגובה ה-AI מוצאת ומאומתת:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## שלבים הבאים

לדוגמאות נוספות, ראו [פרק 04: דוגמאות מעשיות](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**כתב ויתור**:
מסמך זה תורגם באמצעות שירות תרגום אוטומטי [Co-op Translator](https://github.com/Azure/co-op-translator). למרות שאנו שואפים לדיוק, יש לקחת בחשבון שתרגומים אוטומטיים עלולים להכיל שגיאות או אי-דיוקים. יש להחשיב את המסמך המקורי בשפתו הטבעית כמקור הסמכות. למידע קריטי מומלץ להשתמש בתרגום מקצועי על ידי מתרגם אדם. אנו לא אחראים לכל אי-הבנה או פירוש שגוי הנובע מהשימוש בתרגום זה.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->