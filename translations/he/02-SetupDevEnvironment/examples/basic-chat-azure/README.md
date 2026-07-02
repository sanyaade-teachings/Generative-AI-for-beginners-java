# שיחה בסיסית עם Azure AI Foundry - דוגמה מקצה לקצה

דוגמה זו היא יישום Spring Boot פשוט שמתחבר למודל **Azure AI Foundry** באמצעות **אימות ללא מפתח** (Microsoft Entra ID) ובודק את ההתקנה שלך. הוא משתמש ב-`ChatClient` של Spring AI.

## תוכן העניינים

- [דרישות מוקדמות](#דרישות-מוקדמות)
- [התחלה מהירה](#התחלה-מהירה)
- [כיצד האימות עובד](#כיצד-האימות-עובד)
- [הרצת היישום](#הרצת-היישום)
  - [שימוש ב-Maven](#שימוש-ב-maven)
  - [שימוש ב-VS Code](#שימוש-ב-vs-code)
  - [פלט צפוי](#פלט-צפוי)
- [הפניה לקונפיגורציה](#הפניה-לקונפיגורציה)
  - [משתני סביבה](#משתני-סביבה)
  - [קונפיגורציית Spring](#קונפיגורציית-spring)
- [פתרון בעיות](#פתרון-בעיות)
  - [בעיות נפוצות](#בעיות-נפוצות)
  - [מוד Debug](#מוד-debug)
- [השלבים הבאים](#השלבים-הבאים)
- [משאבים](#משאבים)

## דרישות מוקדמות

לפני הרצת דוגמה זו, ודא שיש לך:

- משאב Azure AI Foundry עם פריסת `gpt-4o-mini` — ספק אותו עם `azd up` או ידנית דרך [מדריך ההתקנה של Azure AI Foundry](../../getting-started-azure-openai.md)
- התפקיד **Cognitive Services OpenAI User** על המשאב הזה (תבניות Bicep ממלאות זאת עבורך)
- את [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), מחובר עם `az login`
- Java 21+ ו-Maven 3.9+

> **אין צורך במפתח API** — האימות הוא ללא מפתח דרך Microsoft Entra ID.

## התחלה מהירה

```bash
# 1. נווט לפרויקט
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. התחבר כדי שאימות ללא מפתח יוכל לקבל אסימון
az login

# 3. הגדר את נקודת הקצה
#    - אם הרצת `azd up`, הקובץ .env נכתב עבורך (דלג על זה).
#    - אחרת העתק את התבנית והגדר את AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. הפעל את היישום
mvn spring-boot:run
```

## כיצד האימות עובד

דוגמה זו מאמתת עם **Microsoft Entra ID** — אין מפתח API.

כאשר רק `spring.ai.azure.openai.endpoint` מוגדר (ולא api-key), Spring AI בונה את לקוח Azure OpenAI עם [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). קרדנציאל זה מוצא אוטומטית אסימון מה-sesion של `az login` מקומית, או מתוך זהות מנוהלת כשמריצים ב-Azure — כך שהקוד זהה עובד בשני המקומות ללא שינויים.

## הרצת היישום

### שימוש ב-Maven

```bash
mvn spring-boot:run
```

### שימוש ב-VS Code

1. פתח את הפרויקט ב-VS Code  
2. לחץ על `F5` או השתמש בפאנל "הרצה ודיבאג"  
3. בחר את הקונפיגורציה "Spring Boot-BasicChatApplication"

> **הערה**: קונפיגורציית VS Code טוענת באופן אוטומטי את קובץ .env שלך

### פלט צפוי

```
Starting Basic Chat with Azure OpenAI...
Environment variables loaded successfully
Connecting to Azure OpenAI...
Sending prompt: What is AI in a short sentence? Max 100 words.

AI Response:
================
AI, or Artificial Intelligence, is the simulation of human intelligence in machines programmed to think and learn like humans.
================

Success! Azure OpenAI connection is working correctly.
```

## הפניה לקונפיגורציה

### משתני סביבה

| משתנה | תיאור | חובה | דוגמה |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | כתובת קצה Foundry (Azure OpenAI) | כן | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | שם פריסת מודל שיחה | לא | `gpt-4o-mini` (ברירת מחדל) |

> אין משתנה מפתח API — האימות הוא ללא מפתח (Microsoft Entra ID דרך `az login`).

### קונפיגורציית Spring

קובץ `application.yml` מגדיר:  
- **נקודת קצה**: `${AZURE_OPENAI_ENDPOINT}` - ממשתנה סביבה  
- **פריסה**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - ממשתנה סביבה עם ברירת מחדל  
- **אימות**: ללא מפתח — לא מוגדר `api-key`, לכן Spring AI משתמש ב-`DefaultAzureCredential`  
- **טמפרטורה**: `0.7` - שולט ביצירתיות (0.0 = דטרמיניסטי, 1.0 = יצירתי)  
- **מקסימום טוקנים**: `500` - אורך מרבי של התגובה

## פתרון בעיות

### בעיות נפוצות

<details>
<summary><strong>שגיאה: 401 / "PermissionDenied" / שגיאות אסימון</strong></summary>

- הפעל `az login` — האימות ללא מפתח דורש כניסה פעילה לקבלת אסימון  
- ודא שיש לך את התפקיד **Cognitive Services OpenAI User** על המשאב  
- אם זה הרגע שהקצית את התפקיד, המתן דקה להפצה  
- ודא שאתה בטננט/מנוי הנכון (`az account show`)  
</details>

<details>
<summary><strong>שגיאה: "The endpoint is not valid" / שגיאות חיבור</strong></summary>

- ודא ש-`AZURE_OPENAI_ENDPOINT` הוא כתובת בסיס מלאה (למשל, `https://your-resource.openai.azure.com/`)  
- בדוק עקביות בסלאש הסופי  
- ודא שהנקודה תואמת למשאב שסיפקת (`azd env get-values`)  
</details>

<details>
<summary><strong>שגיאה: "The deployment was not found"</strong></summary>

- ודא ש-`AZURE_OPENAI_DEPLOYMENT` תואם לשם פריסה ב-Azure  
- ודא שהמודל פרוס ומופעל בהצלחה  
- שם הפריסה ברירת המחדל הוא `gpt-4o-mini`  
</details>

<details>
<summary><strong>VS Code: משתני סביבה לא נטענים</strong></summary>

- ודא שקובץ `.env` נמצא בתיקיית השורש של הפרויקט (ברמת `pom.xml`)  
- נסה להריץ `mvn spring-boot:run` בטרמינל המשולב של VS Code  
- בדוק שהרחבת Java של VS Code מותקנת כראוי  
</details>

### מוד Debug

כדי לאפשר רישום מפורט, הסר את ההערה מהשורות האלו ב-`application.yml`:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## השלבים הבאים

**ההתקנה הושלמה!** המשך במסע הלמידה שלך:

[פרק 3: טכניקות AI יצירתי מרכזיות](../../../03-CoreGenerativeAITechniques/README.md)

## משאבים

- [תיעוד Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)  
- [אימות ללא מפתח עם Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)  
- [פורטל Azure AI Foundry](https://ai.azure.com/)  
- [תיעוד Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**כתב ויתור**:
מסמך זה תורגם באמצעות שירות תרגום אוטומטי [Co-op Translator](https://github.com/Azure/co-op-translator). למרות שאנו שואפים לדיוק, יש לקחת בחשבון שתרגומים אוטומטיים עלולים להכיל שגיאות או אי-דיוקים. יש להחשיב את המסמך המקורי בשפתו הטבעית כמקור הסמכות. למידע קריטי מומלץ להשתמש בתרגום מקצועי על ידי מתרגם אדם. אנו לא אחראים לכל אי-הבנה או פירוש שגוי הנובע מהשימוש בתרגום זה.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->