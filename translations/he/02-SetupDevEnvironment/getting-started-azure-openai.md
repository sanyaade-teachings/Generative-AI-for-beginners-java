# הגדרת סביבת הפיתוח עבור Azure AI Foundry

> מדריך זה מגדיר דגמי **Azure AI Foundry** עבור אפליקציות Java AI בקורס זה, באמצעות אימות **ללא מפתחות** (Microsoft Entra ID) — אין צורך לנהל מפתחות API. חדש בכלים? התחל עם [מדריך סביבת הפיתוח](./README.md).

מדריך זה מגדיר דגמי **Azure AI Foundry** עבור אפליקציות Java AI בקורס זה. יש לך שתי דרכים:

- **אפשרות א — פרוביזיה עם `azd` + Bicep (מומלץ):** פקודה אחת מפרסת את חשבון Foundry והדגמים כקוד. אין צורך ללחוץ בפורטל.
- **אפשרות ב — יצירת משאבים ידנית** בפורטל Azure AI Foundry.

שתי הדרכים משתמשות באימות **ללא מפתחות** (Microsoft Entra ID) — אין מפתחות API להעתקה או דליפה.

## תוכן העניינים

- [מה נוצר](#מה-נוצר)
- [דרישות מוקדמות](#דרישות-מוקדמות)
- [אפשרות א: פרוביזיה עם azd + Bicep (מומלץ)](#option-a-provision-with-azd--bicep-recommended)
- [אפשרות ב: יצירת משאבים ידנית](#אפשרות-ב-יצירת-משאבים-ידנית)
- [הגדר את הסביבה שלך](#הגדר-את-הסביבה-שלך)
- [בדוק את ההגדרה שלך](#בדוק-את-ההגדרה-שלך)
- [מה הלאה?](#מה-הלאה)
- [משאבים](#משאבים)
- [משאבים נוספים](#משאבים-נוספים)

## מה נוצר

תבניות Bicep בתיקיית [`infra/`](../../../02-SetupDevEnvironment/infra) יפרסו:

- חשבון **Azure AI Foundry** (`Microsoft.CognitiveServices/accounts`, סוג `AIServices`) עם פרויקט
- פריסת **צ'אט** — `gpt-4o-mini`
- פריסת **אמבדינג** — `text-embedding-3-small` (משמש בפרקים מאוחרים יותר)
- הקצאת תפקיד **ללא מפתחות** (`Cognitive Services OpenAI User`) כך שתוכל להיכנס עם `az login` במקום לנהל מפתחות

## דרישות מוקדמות

- [מנוי Azure](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) ו-[Maven 3.9+](https://maven.apache.org/download.cgi)

## אפשרות א: פרוביזיה עם azd + Bicep (מומלץ)

מתיקיית `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# היכנס (שני הכלים)
azd auth login
az login

# הפרש חשבון Foundry + פריסות מודל
azd up
```

`azd` מבקש **שם סביבה** (לדוגמה `genai-java`) ו-**אזור**. בחר אזור שבו `gpt-4o-mini` ו-`text-embedding-3-small` זמינים — לדוגמה `eastus2` או `swedencentral`.

כשהפרוביזיה מסתיימת, azd:

1. מפעיל פריסה של כל מה שהוגדר ב-[`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep).
2. מריץ תסריט לאחר הפרוביזיה שכותב את [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) עם נקודת הקצה ושמות הפריסה שלך (ללא סודות).

> **טיפ:** הרץ שוב `azd up` בכל עת כדי להחיל שינויים. הרץ `azd down` כדי למחוק הכל ולהפסיק לייצר עלויות.

כדי לראות את ההגדרות שנוצרו:

```bash
azd env get-values
```

דלג עכשיו אל [בדוק את ההגדרה שלך](#בדוק-את-ההגדרה-שלך).

## אפשרות ב: יצירת משאבים ידנית

מעדיף את הפורטל? צור את המשאבים ידנית:

1. עבור אל [פורטל Azure AI Foundry](https://ai.azure.com/) והיכנס.
2. **צור פרויקט** (זה גם יוצר משאב AI Foundry). תן לו שם כמו `GenAIJava`.
3. בפרויקט שלך, פתח **Models + endpoints** → **Deploy model** → **Deploy base model**.
4. פרוס את **gpt-4o-mini** (שם הפריסה `gpt-4o-mini`). חזור על כך עבור **text-embedding-3-small** אם ברצונך בדוגמאות האמבדינג.
5. מתוך **Overview**, העתק את ה-**endpoint** (לדוגמה `https://<resource>.openai.azure.com/`).
6. תן לעצמך גישה ללא מפתחות: על המשאב, פתח **Access control (IAM)** → **Add role assignment** → הקצה את התפקיד **Cognitive Services OpenAI User** לחשבונך.

> **עדיין נתקל בקושי?** ראה את [תיעוד Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## הגדר את הסביבה שלך

**אם השתמשת באפשרות א (`azd up`)**, קובץ ההגדרות שלך כבר נכתב — אין צורך בהגדרה. דלג אל [בדוק את ההגדרה שלך](#בדוק-את-ההגדרה-שלך).

**אם השתמשת באפשרות ב (ידני)**, צור בעצמך את קובץ `.env` של הדוגמה:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

ערוך את `.env` עם נקודת הקצה שלך (ללא מפתח — האימות הוא ללא מפתחות):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **הערת אבטחה:** אין מפתח API לשמירה. אתה מאמת עם Microsoft Entra ID דרך `az login` (מקומי) או בזהות מנוהלת (ב-Azure). קובץ `.env` מכיל רק הגדרות לא סודיות וכבר כלול ב-`.gitignore`.

## בדוק את ההגדרה שלך

ודא שאתה מחובר כך שאימות ללא מפתחות יכול לקבל אסימון, ואז הרץ את הדוגמה:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # אם אתה עדיין לא מחובר
mvn clean spring-boot:run
```

עליך לראות תגובה מהדגם `gpt-4o-mini`!

> **משתמשי VS Code:** לחץ `F5` כדי להריץ. האפליקציה טוענת את קובץ `.env` שלך אוטומטית.

> **דוגמה מלאה:** עיין ב-[הדוגמה הבסיסית לצ'אט עם Azure AI Foundry](./examples/basic-chat-azure/README.md) לפרטים ופתרון בעיות.

## מה הלאה?

**ההגדרה הושלמה!** עכשיו יש לך:
- Azure AI Foundry עם `gpt-4o-mini` ו-`text-embedding-3-small` בפרוסה
- אימות ללא מפתחות (Microsoft Entra ID) — אין מפתחות לנהל
- קובץ `.env` מקומי עם נקודת הקצה ושמות הפריסות שלך
- סביבת פיתוח Java מוכנה לשימוש

**המשך אל** [פרק 3: טכניקות AI גנרטיבי בסיסיות](../03-CoreGenerativeAITechniques/README.md) כדי להתחיל בבניית אפליקציות AI!

## משאבים

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [אימות ללא מפתחות עם Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [תיעוד Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)
- [תיעוד Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## משאבים נוספים

- [הורד VS Code](https://code.visualstudio.com/Download)
- [קבל את Docker Desktop](https://www.docker.com/products/docker-desktop)
- [הגדרות מיכל פיתוח](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**כתב ויתור**:
מסמך זה תורגם באמצעות שירות תרגום אוטומטי [Co-op Translator](https://github.com/Azure/co-op-translator). למרות שאנו שואפים לדיוק, יש לקחת בחשבון שתרגומים אוטומטיים עלולים להכיל שגיאות או אי-דיוקים. יש להחשיב את המסמך המקורי בשפתו הטבעית כמקור הסמכות. למידע קריטי מומלץ להשתמש בתרגום מקצועי על ידי מתרגם אדם. אנו לא אחראים לכל אי-הבנה או פירוש שגוי הנובע מהשימוש בתרגום זה.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->