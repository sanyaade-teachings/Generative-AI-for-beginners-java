# AGENTS.md

## סקירת הפרויקט

זהו מאגר לימודי ללמידת פיתוח AI גנרטיבי עם Java. הוא מספק קורס מעשי מקיף המכסה מודלים לשוניים גדולים (LLMs), הנדסת פרומפטים, אמבדינגס, RAG (גירנציה משופרת בשליפה), ופרוטוקול הקשר המודל (MCP).

**טכנולוגיות מרכזיות:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI, ו-SDKs של OpenAI

**ארכיטקטורה:**
- מספר אפליקציות Spring Boot עצמאיות הממוינות לפי פרקים
- פרויקטים לדוגמה המדגימים דפוסי AI שונים
- מוכנות ל-GitHub Codespaces עם מיכלי פיתוח מוגדרים מראש

## פקודות התקנה

### דרישות מוקדמות
- Java 21 או גרסה גבוהה יותר
- Maven 3.x
- מנוי Azure עם פריסה של מודל Azure AI Foundry (הגדרה עם `azd up`)
- Azure CLI (`az`) ו-Azure Developer CLI (`azd`), מחוברים לאישור keyless

### הגדרת סביבה

**אפשרות 1: GitHub Codespaces (מומלץ)**
```bash
# פצל את המאגר וצור סביבת קוד מ- GitHub UI
# מכולת הפיתוח תתקין אוטומטית את כל התלויות
# המתן כ-2 דקות להקמת הסביבה
```

**אפשרות 2: מיכל פיתוח מקומי**
```bash
# שכפל מאגר
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# פתח ב- VS Code עם תוסף Dev Containers
# פתח מחדש במכולה כשהתבקש
```

**אפשרות 3: הגדרה מקומית**
```bash
# התקן תלותים
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# אמת התקנה
java -version
mvn -version
```

### קונפיגורציה

**הגדרת Azure AI Foundry (keyless, מומלץ):**
```bash
# פרוס את חשבון Foundry + פריסות מודל כקוד
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd כותב examples/basic-chat-azure/.env עם נקודת הקצה שלך (ללא מפתח)
```

**קונפיגורציית נקודת קצה ידנית:**
```bash
# אם לא השתמשת ב-azd, הגדיר את נקודת הקצה בעצמך (ההזדהות נשארת ללא מפתח דרך az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# ערוך את .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## זרימת עבודה בפיתוח

### מבנה הפרויקט
```
/
├── 01-IntroToGenAI/              # Chapter 1: Introduction
├── 02-SetupDevEnvironment/       # Chapter 2: Environment setup
│   └── examples/                 # Working examples
├── 03-CoreGenerativeAITechniques/ # Chapter 3: Core techniques
├── 04-PracticalSamples/          # Chapter 4: Sample projects
│   ├── calculator/               # MCP service example
│   ├── foundrylocal/            # Local model integration
│   └── petstory/                # Multi-modal app
├── 05-ResponsibleGenAI/         # Chapter 5: Responsible AI
└── translations/                # Multi-language support
```

### הפעלת אפליקציות

**הפעלת אפליקציית Spring Boot:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**בנייה של פרויקט:**
```bash
cd [project-directory]
mvn clean install
```

**הפעלת שרת מחשבון MCP:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# השרת רץ על http://localhost:8080
```

**הפעלת דוגמאות לקוח:**
```bash
# לאחר הפעלת השרת בטרמינל אחר
cd 04-PracticalSamples/calculator

# לקוח MCP ישיר
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# לקוח המופעל בעזרת בינה מלאכותית (דורש AZURE_OPENAI_ENDPOINT + az login)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# בוט אינטראקטיבי
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### טעינה חמה (Hot Reload)
Spring Boot DevTools כלול בפרויקטים שתומכים בטעינה חמה:
```bash
# שינויים בקבצי Java ייטענו מחדש אוטומטית בעת שמירה
mvn spring-boot:run
```

## הוראות בדיקה

### הרצת בדיקות

**הרץ את כל הבדיקות בפרויקט:**
```bash
cd [project-directory]
mvn test
```

**הרץ בדיקות עם פלט מפורט:**
```bash
mvn test -X
```

**הרץ מחלקת בדיקה ספציפית:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### מבנה מבחנים
- קבצי מבחן משתמשים ב-JUnit 5 (Jupiter)
- מחלקות הבדיקה נמצאות ב- `src/test/java/`
- דוגמאות לקוח בפרויקט המחשבון ב- `src/test/java/com/microsoft/mcp/sample/client/`

### בדיקות ידניות
הרבה דוגמאות הן אפליקציות אינטראקטיביות שדורשות בדיקה ידנית:

1. הפעל את האפליקציה עם `mvn spring-boot:run`
2. בדוק נקודות קצה או אינטראקציה עם CLI
3. וודא שההתנהגות הצפויה תואמת את התיעוד ב- README.md של כל פרויקט

### בדיקה עם Azure AI Foundry
- התחבר עם `az login` לפני הרצת הדוגמאות (אישור keyless)
- ודא שלחשבון שלך יש תפקיד Cognitive Services OpenAI User במשאב
- בדוק סינון תוכן עם דוגמת AI אחראי בפרק 5

## קווי הנחיה לסגנון קוד

### קונבנציות Java
- **גרסת Java:** Java 21 עם תכונות מודרניות
- **סטייל:** עקוב אחרי הקונבנציות הסטנדרטיות של Java
- **שמות:**
  - מחלקות: PascalCase
  - שיטות/משתנים: camelCase
  - קבועים: UPPER_SNAKE_CASE
  - שמות חבילות: אותיות קטנות

### דפוסי Spring Boot
- השתמש ב-`@Service` ללוגיקת עסקים
- השתמש ב-`@RestController` לנקודות קצה REST
- קונפיגורציה דרך `application.yml` או `application.properties`
- העדף משתני סביבה על פני ערכים קבועים בקוד
- השתמש ב-`@Tool` עבור שיטות המוצגות דרך MCP

### ארגון קבצים
```
src/
├── main/
│   ├── java/
│   │   └── com/microsoft/[component]/
│   │       ├── [Component]Application.java
│   │       ├── config/
│   │       ├── controller/
│   │       ├── service/
│   │       └── exception/
│   └── resources/
│       ├── application.yml
│       └── static/
└── test/
    └── java/
        └── com/microsoft/[component]/
```

### תלותים
- מנוהלים דרך Maven `pom.xml`
- Spring AI BOM לניהול גרסאות
- LangChain4j לאינטגרציות AI
- Spring Boot starter parent לתלותות Spring

### הערות בקוד
- הוסף JavaDoc ל-APIs ציבוריים
- כלול הערות להסבר אינטראקציות AI מורכבות
- תעד תיאורי כלים של MCP בצורה ברורה

## בנייה ופריסה

### בניית פרויקטים

**בנה ללא בדיקות:**
```bash
mvn clean install -DskipTests
```

**בנה עם כל הבדיקות:**
```bash
mvn clean install
```

**חבילה של אפליקציה:**
```bash
mvn package
# יוצר קובץ JAR בתיקיית target/
```

### תיקיות פלט
- מחלקות מקומפלות: `target/classes/`
- מחלקות בדיקה: `target/test-classes/`
- קבצי JAR: `target/*.jar`
- ארטיפקטים של Maven: `target/`

### קונפיגורציה סביבה-ספציפית

**פיתוח:**
```yaml
# application.yml (keyless - no api-key; auth via DefaultAzureCredential)
spring:
  ai:
    azure:
      openai:
        endpoint: ${AZURE_OPENAI_ENDPOINT}
        chat:
          options:
            deployment-name: ${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}
```

**ייצור:**
- השתמש בזהות מנוהלת במקום `az login` לאישור keyless
- הצב את `AZURE_OPENAI_ENDPOINT` למשאב Foundry בייצור שלך
- נהל קונפיגורציה דרך משתני סביבה או Azure Key Vault

### שיקולי פריסה
- מאגר זה הוא למידה עם דוגמאות אפליקציות
- לא מיועד לפריסה בייצור כמו שהוא
- הדוגמאות מדגימות דפוסים להתאמה לשימוש בייצור
- ראה README של כל פרויקט לפרטים ספציפיים לפריסה

## הערות נוספות

### Azure AI Foundry
- **אישור keyless:** התחבר עם Microsoft Entra ID — אין צורך בניהול מפתחות API
- **הגדרה כקוד:** Bicep + azd (`azd up`) יוצרים את החשבון ופריסות המודלים
- קוד תואם OpenAI זהה פועל מקומית (`az login`) וב-Azure (זהות מנוהלת)

### עבודה עם מספר פרויקטים
כל פרויקט דוגמה הוא עצמאי:
```bash
# לנווט לפרויקט ספציפי
cd 04-PracticalSamples/[project-name]

# לכל אחד יש pom.xml משלו וניתן לבנות אותו באופן עצמאי
mvn clean install
```

### בעיות נפוצות

**אי התאמת גרסת Java:**
```bash
# אמת Java 21
java -version
# עדכן את JAVA_HOME במידת הצורך
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**בעיות בהורדת תלותים:**
```bash
# נקה את מטמון Maven ונסה מחדש
rm -rf ~/.m2/repository
mvn clean install
```

**נקודת קצה או התחברות לא נמצאו:**
```bash
# הגדר את נקודת הקצה במפגש הנוכחי והתחבר (ללא מפתח)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# או השתמש בקובץ .env בתיקיית הפרויקט
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**הפורט כבר בשימוש:**
```bash
# ספרינג בוט משתמש ביציאה 8080 כברירת מחדל
# שינוי בקובץ application.properties:
server.port=8081
```

### תמיכה רב-לשונית
- תיעוד זמין בלמעלה מ-45 שפות בתרגום אוטומטי
- תרגומים בתיקיית `translations/`
- ניהול תרגום על ידי GitHub Actions workflow

### מסלול למידה
1. התחל עם [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. עקוב אחר הפרקים בסדר (01 → 05)
3. השלם דוגמאות מעשיות בכל פרק
4. חקור פרויקטים לדוגמה בפרק 4
5. למד פרקטיקות AI אחראי בפרק 5

### מיכל פיתוח (Development Container)
הקובץ `.devcontainer/devcontainer.json` מגדיר:
- סביבת פיתוח Java 21
- Maven מותקן מראש
- הרחבות Java ל-VS Code
- כלי Spring Boot
- אינטגרציית GitHub Copilot
- תמיכה ב-Docker-in-Docker
- Azure CLI

### שיקולי ביצועים
- בפריסות Azure AI Foundry יש מגבלות טוקנים/בקשות לדקה
- השתמש בגודל קבוצות מתאים לאמבדינגס
- שקול מטמון לקריאות API חוזרות
- נטר שימוש בטוקנים לאופטימיזציית עלויות

### הערות אבטחה
- לעולם אל תcommיט קבצי `.env` (כבר מרוחקים ב- `.gitignore`)
- העדף אישור keyless (Microsoft Entra ID) על פני מפתחות API
- השתמש בזהויות מנוהלות ב-Azure; `az login` לפיתוח מקומי
- עקוב אחרי הנחיות AI אחראי בפרק 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**כתב ויתור**:
מסמך זה תורגם באמצעות שירות תרגום אוטומטי [Co-op Translator](https://github.com/Azure/co-op-translator). למרות שאנו שואפים לדיוק, יש לקחת בחשבון שתרגומים אוטומטיים עלולים להכיל שגיאות או אי-דיוקים. יש להחשיב את המסמך המקורי בשפתו הטבעית כמקור הסמכות. למידע קריטי מומלץ להשתמש בתרגום מקצועי על ידי מתרגם אדם. אנו לא אחראים לכל אי-הבנה או פירוש שגוי הנובע מהשימוש בתרגום זה.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->