# הקמת סביבת הפיתוח ל-AI גנרטיבי עבור Java

> **התחלה מהירה:** ספק את דגמי ה-AI שלך ב-**Azure AI Foundry** כקוד עם Bicep + `azd` בכמה דקות — ראה את [מדריך ההתקנה של Azure AI Foundry](getting-started-azure-openai.md). האימות הוא **ללא מפתח** (Microsoft Entra ID), כך שאין מפתחות API לנהל.

## מה תלמדו

- להקים סביבת פיתוח Java ליישומי AI
- לבחור ולהגדיר את סביבת הפיתוח המועדפת עליך (מובנה לענן עם Codespaces, מכולת פיתוח מקומית, או התקנה מקומית מלאה)
- לבדוק את ההתקנה על ידי חיבור לדגם של Azure AI Foundry

## תוכן העניינים

- [מה תלמדו](#מה-תלמדו)
- [מבוא](#מבוא)
- [שלב 1: הקמת סביבת הפיתוח](#שלב-1-הקמת-סביבת-הפיתוח)
  - [אפשרות A: GitHub Codespaces (מומלץ)](#אפשרות-a-github-codespaces-מומלץ)
  - [אפשרות B: מכולת פיתוח מקומית](#אפשרות-b-מכולת-פיתוח-מקומית)
  - [אפשרות C: שימוש בהתקנה מקומית קיימת](#אפשרות-c-שימוש-בהתקנה-מקומית-קיימת)
- [שלב 2: הקמת Azure AI Foundry](#שלב-2-הקמת-azure-ai-foundry)
- [שלב 3: בדיקת ההתקנה](#שלב-3-בדיקת-ההתקנה)
- [פתרון בעיות](#פתרון-בעיות)
- [סיכום](#סיכום)
- [שלבים הבאים](#שלבים-הבאים)

## מבוא

פרק זה ינחה אותך בהקמת סביבת פיתוח. נשתמש ב-**Azure AI Foundry** עבור הדגמים לאורך כל הקורס. אתה מספק את הדגמים כקוד באמצעות Bicep ו-Azure Developer CLI (`azd`), ואז מתחבר עם **אימות ללא מפתח** (Microsoft Entra ID) — אין צורך להעתיק או לשתף מפתחות API.

**אין צורך בהתקנה מקומית!** ניתן להשתמש ב-GitHub Codespaces, המספקת סביבת פיתוח מלאה בדפדפן שלך, ולספק את Foundry משם.

אנו משתמשים ב-**Azure AI Foundry** בקורס זה כי הוא:
- **מסופק כקוד** — ריצה אחת של `azd up` מפעילה את החשבון והפריסות של הדגמים
- **ללא מפתח** — התחבר עם חשבון Azure שלך או זהות מנוהלת
- **מוכן לייצור** — הקוד זהה פועל גם מקומית וגם ב-Azure
- **גמיש** — החלף דגמים על ידי שינוי שם פריסה, בלי לשנות את הקוד

> **הערה**: פריסות Azure AI Foundry מחויבות לפי טוקן (תשלום לפי שימוש). ראה את [מדריך ההתקנה של Azure AI Foundry](getting-started-azure-openai.md) לפרטים על הפריסה, האזור והעלויות.


## שלב 1: הקמת סביבת הפיתוח

<a name="quick-start-cloud"></a>

יצרנו מכולת פיתוח מוגדרת מראש בכדי למזער את זמן ההתקנה ולהבטיח שיש לך את כל הכלים הדרושים לקורס Generative AI ל-Java זה. בחר את גישת הפיתוח המועדפת עליך:

### אפשרויות הקמת סביבה:

#### אפשרות A: GitHub Codespaces (מומלץ)

**התחל לקודד תוך 2 דקות - אין צורך בהתקנה מקומית!**

1. בצע Fork למאגר זה לחשבון ה-GitHub שלך
   > **הערה**: אם ברצונך לערוך את הקונפיגורציה הבסיסית עיין ב-[Dev Container Configuration](../../../.devcontainer/devcontainer.json)
2. לחץ על **Code** → לשונית **Codespaces** → **...** → **New with options...**
3. השתמש בערכים המוגדרים כבר – זה יבחר את **Dev container configuration**: **סביבת פיתוח Java ל-AI גנרטיבי** - מכולת פיתוח מותאמת שנוצרה עבור הקורס
4. לחץ על **Create codespace**
5. המתן ~2 דקות עד לסיום הטעינה של הסביבה
6. המשך ל-[שלב 2: הקמת Azure AI Foundry](#שלב-2-הקמת-azure-ai-foundry)

<img src="../../../translated_images/he/codespaces.9945ded8ceb431a5.webp" alt="צילום מסך: תפריט משנה של Codespaces" width="50%">

<img src="../../../translated_images/he/image.833552b62eee7766.webp" alt="צילום מסך: New with options" width="50%">

<img src="../../../translated_images/he/codespaces-create.b44a36f728660ab7.webp" alt="צילום מסך: אפשרויות יצירת Codespace" width="50%">


> **היתרונות של Codespaces**:
> - אין צורך בהתקנה מקומית
> - עובד בכל מכשיר עם דפדפן
> - מוגדר מראש עם כל הכלים והתלויות
> - 60 שעות חינם לחודש לחשבונות אישיים
> - סביבת עבודה אחידה לכל הלומדים

#### אפשרות B: מכולת פיתוח מקומית

**למפתחים שמעדיפים פיתוח מקומי עם Docker**

1. בצע Fork ו-Clone של המאגר הזה למחשב המקומי שלך
   > **הערה**: אם ברצונך לערוך את הקונפיגורציה הבסיסית עיין ב-[Dev Container Configuration](../../../.devcontainer/devcontainer.json)
2. התקן את [Docker Desktop](https://www.docker.com/products/docker-desktop/) ואת [VS Code](https://code.visualstudio.com/)
3. התקן את [תוסף Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) ב-VS Code
4. פתח את תיקיית המאגר ב-VS Code
5. כאשר יופיע, לחץ על **Reopen in Container** (או השתמש ב-`Ctrl+Shift+P` → "Dev Containers: Reopen in Container")
6. המתן עד שהמכולה תיבנה ותפעל
7. המשך ל-[שלב 2: הקמת Azure AI Foundry](#שלב-2-הקמת-azure-ai-foundry)

<img src="../../../translated_images/he/devcontainer.21126c9d6de64494.webp" alt="צילום מסך: הגדרת מכולת פיתוח" width="50%">

<img src="../../../translated_images/he/image-3.bf93d533bbc84268.webp" alt="צילום מסך: בניית מכולת פיתוח הושלמה" width="50%">

#### אפשרות C: שימוש בהתקנה מקומית קיימת

**למפתחים עם סביבות Java קיימות**

דרישות מוקדמות:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html) 
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) או ה-IDE המועדף עליך

שלבים:
1. בצע Clone למאגר זה למחשב המקומי שלך
2. פתח את הפרויקט ב-IDE שלך
3. המשך ל-[שלב 2: הקמת Azure AI Foundry](#שלב-2-הקמת-azure-ai-foundry)

> **טיפ מקצועי**: אם יש לך מחשב עם מפרט נמוך אך תרצה VS Code מקומית, השתמש ב-GitHub Codespaces! תוכל לחבר את VS Code המקומי שלך ל-Codespace בענן ולקבל את הטוב משני העולמות.

<img src="../../../translated_images/he/image-2.fc0da29a6e4d2aff.webp" alt="צילום מסך: מופע מכולת פיתוח מקומית שנוצר" width="50%">


## שלב 2: הקמת Azure AI Foundry

פרוס את דגמי ה-AI של הקורס ל-Azure AI Foundry כקוד. משורש המאגר:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```
  
`azd` יבקש שם סביבה ואזור, יספק חשבון Azure AI Foundry עם פריסות `gpt-4o-mini` ו-`text-embedding-3-small`, ויכתוב את נקודת הקצה בקובץ `.env` של הדוגמה — הכל עם **אימות ללא מפתח** (אין מפתחות API).

> **מדריך מלא:** ראה את [מדריך ההקמה ל-Azure AI Foundry](getting-started-azure-openai.md) לתנאים מוקדמים, אלטרנטיבה ידנית (פורטאל), הנחיות לאיזור, והערות על עלויות וניקוי.

## שלב 3: בדיקת ההתקנה

לאחר שהדגמים ב-Foundry הוקמו, בדוק את החיבור עם אפליקציית הדוגמה ב-[`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure).

1. פתח את הטרמינל בסביבת הפיתוח שלך.  
2. עבור לדוגמה:  
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
  
3. ודא שאתה מחובר (אימות ללא מפתח זקוק לטוקן):  
   ```bash
   az login
   ```
   
> אם הפעלת `azd up`, קובץ `.env` עם נקודת הקצה כבר נכתב עבורך.  
4. הפעל את האפליקציה:  
   ```bash
   mvn clean spring-boot:run
   ```
  
עליך לראות תגובה מהדגם `gpt-4o-mini`.

### הבנת קוד הדוגמה

הדוגמה תחת `examples/basic-chat-azure` היא אפליקציית Spring Boot שמשתמשת ב-**Spring AI** להתחבר ל-Azure AI Foundry עם התחברות ללא מפתח.

**מה שהקוד עושה:**
- **מתחבר** אל Azure AI Foundry באמצעות ההתחברות שלך ל-Azure (Microsoft Entra ID) — אין מפתח API
- **שולח** פקודה לדגם `gpt-4o-mini`
- **מקבל** ומציג את תגובת ה-AI
- **מאמת** שההתקנה שלך עובדת כראוי

**תלות מפתח** (ב-`pom.xml`):  
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```
  
**קונפיגורציה** (`application.yml`):  
```yaml
spring:
  ai:
    azure:
      openai:
        # Endpoint only - no api-key. Spring AI uses DefaultAzureCredential (keyless).
        endpoint: ${AZURE_OPENAI_ENDPOINT}
        chat:
          options:
            deployment-name: ${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}
```
  
## סיכום

מעולה! עכשיו יש לך את כל מה שצריך:

- דגמי Azure AI Foundry מסופקים כקוד עם Bicep + `azd`
- סביבת הפיתוח Java שלך פועלת (בין אם Codespaces, מכולות פיתוח, או מקומית)
- התחברת ל-Azure AI Foundry עם אימות ללא מפתח (Microsoft Entra ID) — ללא מפתחות API
- בדקת שהכל עובד עם דוגמה פשוטה שמדברת עם הדגם שלך

## שלבים הבאים

[פרק 3: טכניקות מרכזיות ב-AI גנרטיבי](../03-CoreGenerativeAITechniques/README.md)

## פתרון בעיות

יש לך בעיות? הנה בעיות נפוצות ופתרונות:

- **האימות נכשל (401/403)?**  
  - הרץ `az login` — האימות הוא ללא מפתח, חייב להיות מחובר  
  - וודא שלחשבון שלך יש את התפקיד **Cognitive Services OpenAI User** במשאב  
  - אם הרגע סיפקת, המתן דקה להשלמת הקצאת התפקיד

- **Maven לא נמצא?**  
  - אם משתמש ב-dev containers/Codespaces, Maven מותקן מראש  
  - בהתקנה מקומית ודא ש-Java 21+ ו-Maven 3.9+ מותקנים  
  - נסה להריץ `mvn --version` לאימות ההתקנה

- **`azd` לא נמצא או ההקמה נכשלת?**  
  - התקן את [Azure Developer CLI](https://aka.ms/azure-dev/install) והריץ `azd auth login`  
  - בחר אזור שבו `gpt-4o-mini` זמין (למשל `eastus2`)  
  - ראה את [מדריך ההקמה של Azure AI Foundry](getting-started-azure-openai.md) לפרטים

- **מכולת הפיתוח לא מתחילה?**  
  - ודא ש-Docker Desktop רץ (להתקנה מקומית)  
  - נסה לבנות מחדש את המכולה: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **שגיאות קומפילציה של האפליקציה?**  
  - ודא שאתה בתיקייה הנכונה: `02-SetupDevEnvironment/examples/basic-chat-azure`  
  - נסה לנקות ולבנות מחדש: `mvn clean compile`

> **זקוק לעזרה?**: עדיין יש בעיות? פתח Issue במאגר ונעזור לך.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**כתב ויתור**:
מסמך זה תורגם באמצעות שירות תרגום אוטומטי [Co-op Translator](https://github.com/Azure/co-op-translator). למרות שאנו שואפים לדיוק, יש לקחת בחשבון שתרגומים אוטומטיים עלולים להכיל שגיאות או אי-דיוקים. יש להחשיב את המסמך המקורי בשפתו הטבעית כמקור הסמכות. למידע קריטי מומלץ להשתמש בתרגום מקצועי על ידי מתרגם אדם. אנו לא אחראים לכל אי-הבנה או פירוש שגוי הנובע מהשימוש בתרגום זה.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->