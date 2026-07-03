# מדריך מחשבון MCP למתחילים

## תוכן העניינים

- [מה תלמד](#מה-תלמד)
- [דרישות מוקדמות](#דרישות-מוקדמות)
- [הבנת מבנה הפרויקט](#הבנת-מבנה-הפרויקט)
- [הסבר רכיבים עיקריים](#הסבר-רכיבים-עיקריים)
  - [1. יישום ראשי](#1-יישום-ראשי)
  - [2. שירות המחשבון](#2-שירות-המחשבון)
  - [3. לקוח MCP ישיר](#3-לקוח-mcp-ישיר)
  - [4. לקוח מונע בינה מלאכותית](#4-לקוח-מונע-בינה-מלאכותית)
- [הרצת הדוגמאות](#הרצת-הדוגמאות)
- [איך הכל עובד יחד](#איך-הכל-עובד-יחד)
- [השלבים הבאים](#השלבים-הבאים)

## מה תלמד

מדריך זה מסביר כיצד לבנות שירות מחשבון באמצעות פרוטוקול הקשר מודל (MCP). תלמד:

- כיצד ליצור שירות שהבינה המלאכותית יכולה להשתמש בו ככלי
- כיצד להקים תקשורת ישירה עם שירותי MCP
- כיצד מודלים של בינה מלאכותית יכולים לבחור אוטומטית באילו כלים להשתמש
- ההבדל בין קריאות פרוטוקול ישירות לאינטראקציות המועזרות על ידי בינה מלאכותית

## דרישות מוקדמות

לפני שתתחיל, ודא שיש ברשותך:
- Java 21 ומעלה מותקן
- Maven לניהול תלותים
- פריסת מודל Azure AI Foundry (ספק אותה עם `azd up` — ראה [פרק 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), מחובר עם `az login` (אימות ללא מפתח)
- הבנה בסיסית ב-Java ו-Spring Boot

## הבנת מבנה הפרויקט

לפרויקט המחשבון קיימים מספר קבצים חשובים:

```
calculator/
├── src/main/java/com/microsoft/mcp/sample/server/
│   ├── McpServerApplication.java          # Main Spring Boot app
│   └── service/CalculatorService.java     # Calculator operations
└── src/test/java/com/microsoft/mcp/sample/client/
    ├── SDKClient.java                     # Direct MCP communication
    ├── LangChain4jClient.java            # AI-powered client
    └── Bot.java                          # Simple chat interface
```

## הסבר רכיבים עיקריים

### 1. יישום ראשי

**קובץ:** `McpServerApplication.java`

זהו נקודת הכניסה לשירות המחשבון שלנו. זה יישום Spring Boot סטנדרטי עם תוספת מיוחדת אחת:

```java
@SpringBootApplication
public class McpServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(McpServerApplication.class, args);
    }
    
    @Bean
    public ToolCallbackProvider calculatorTools(CalculatorService calculator) {
        return MethodToolCallbackProvider.builder().toolObjects(calculator).build();
    }
}
```

**מה שזה עושה:**
- מפעיל שרת רשת של Spring Boot על פורט 8080
- יוצר `ToolCallbackProvider` שמאפשר לשיטות המחשבון שלנו להיות זמינות ככלי MCP
- התווית `@Bean` מודיעה ל-Spring לנהל זאת כרכיב שחלקים אחרים יכולים להשתמש בו

### 2. שירות המחשבון

**קובץ:** `CalculatorService.java`

כאן מתבצעות כל החישובים. כל שיטה מתויגת עם `@Tool` כדי להפוך אותה לזמינה דרך MCP:

```java
@Service
public class CalculatorService {

    @Tool(description = "Add two numbers together")
    public String add(double a, double b) {
        double result = a + b;
        return formatResult(a, "+", b, result);
    }

    @Tool(description = "Subtract the second number from the first number")
    public String subtract(double a, double b) {
        double result = a - b;
        return formatResult(a, "-", b, result);
    }
    
    // עוד פעולות מחשבון...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**תכונות עיקריות:**

1. **תווית `@Tool`**: מודיעה ל-MCP שניתן לקרוא לשיטה זו מלקוחות חיצוניים
2. **תיאורים ברורים**: לכל כלי יש תיאור שמסייע למודלים של בינה מלאכותית להבין מתי להשתמש בו
3. **פורמט החזרת תוצאה קבוע**: כל הפעולות מחזירות מחרוזות קריאות לבני אדם כמו "5.00 + 3.00 = 8.00"
4. **טיפול בשגיאות**: חישוב חלוקה באפס וחישוב שורש ריבועי שלילי מחזירים הודעות שגיאה

**פעולות זמינות:**
- `add(a, b)` - מוסיף שני מספרים
- `subtract(a, b)` - מחסר את השני מהראשון
- `multiply(a, b)` - מכפיל שני מספרים
- `divide(a, b)` - מחלק את הראשון בשני (עם בדיקת אפס)
- `power(base, exponent)` - מחשב בחזקה
- `squareRoot(number)` - מחשב שורש ריבועי (עם בדיקת שלילי)
- `modulus(a, b)` - מחזיר את שארית החלוקה
- `absolute(number)` - מחזיר ערך מוחלט
- `help()` - מחזיר מידע על כל הפעולות

### 3. לקוח MCP ישיר

**קובץ:** `SDKClient.java`

לקוח זה מתקשר ישירות לשרת MCP בלי להשתמש בבינה מלאכותית. קורא ידנית לפונקציות ספציפיות של המחשבון:

```java
public class SDKClient {
    
    public static void main(String[] args) {
        McpClientTransport transport = WebFluxSseClientTransport.builder(
            WebClient.builder().baseUrl("http://localhost:8080")
        ).build();
        new SDKClient(transport).run();
    }
    
    public void run() {
        var client = McpClient.sync(this.transport).build();
        client.initialize();
        
        // רשימת הכלים הזמינים
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // קריאה לפונקציות מחשבון ספציפיות
        CallToolResult resultAdd = client.callTool(
            new CallToolRequest("add", Map.of("a", 5.0, "b", 3.0))
        );
        System.out.println("Add Result = " + resultAdd);
        
        CallToolResult resultSqrt = client.callTool(
            new CallToolRequest("squareRoot", Map.of("number", 16.0))
        );
        System.out.println("Square Root Result = " + resultSqrt);
        
        client.closeGracefully();
    }
}
```

**מה שזה עושה:**
1. **מתחבר** לשרת המחשבון בכתובת `http://localhost:8080` באמצעות תבנית בנאי
2. **מציג** רשימה של כל הכלים הזמינים (פונקציות המחשבון שלנו)
3. **קורא** לפונקציות ספציפיות עם פרמטרים מדויקים
4. **מציג** את התוצאות ישירות

**הערה:** דוגמה זו משתמשת בתלות Spring AI 1.1.0-SNAPSHOT, שהוסיפה תבנית בנאי ל-`WebFluxSseClientTransport`. אם אתם משתמשים בגרסה יציבה ישנה יותר, ייתכן שתצטרכו להשתמש בקונסטרקטור הישיר במקום.

**מתי להשתמש בזה:** כשאתה יודע בדיוק איזו חישוב ברצונך לבצע ורוצה לקרוא לה באופן תכנותי.

### 4. לקוח מונע בינה מלאכותית

**קובץ:** `LangChain4jClient.java`

לקוח זה משתמש במודל AI (GPT-4o-mini) שיכול לבחור אוטומטית באילו כלים של המחשבון להשתמש:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // הגדר את מודל ה-AI (Azure AI Foundry, אימות ללא מפתח באמצעות Microsoft Entra ID)
        String endpoint = System.getenv("AZURE_OPENAI_ENDPOINT");
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1";
        String token = new DefaultAzureCredentialBuilder().build()
                .getToken(new TokenRequestContext().addScopes("https://ai.azure.com/.default"))
                .block().getToken();
        ChatLanguageModel model = OpenAiOfficialChatModel.builder()
                .baseUrl(baseUrl)
                .apiKey(token)
                .modelName("gpt-4o-mini")
                .build();

        // התחבר לשרת המחשבון MCP שלנו
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // מציג את מה שה-AI עושה
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // תן ל-AI גישה לכלי המחשבון שלנו
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // צור בוט AI שיכול להשתמש במחשבון שלנו
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // עכשיו אנחנו יכולים לבקש מה-AI לבצע חישובים בשפה טבעית
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**מה שזה עושה:**
1. **יוצר** חיבור למודל AI עם אימות ללא מפתח (Microsoft Entra ID)
2. **מעלה** את חיבור ה-AI לשרת MCP של המחשבון שלנו
3. **נותן** ל-AI גישה לכל כלי המחשבון שלנו
4. **מאפשר** בקשות בשפה טבעית כמו "חשב את סכום 24.5 ו-17.3"

**ה-AI אוטומטית:**
- מבין שברצונך לחבר מספרים
- בוחר בכלי `add`
- קורא ל-`add(24.5, 17.3)`
- מחזיר את התוצאה בתגובה טבעית

## הרצת הדוגמאות

### שלב 1: הפעל את שרת המחשבון

קודם כל, היכנס והגדר את נקודת הקצה של Azure AI Foundry (נדרש ללקוח AI — אימות ללא מפתח, ללא מפתח API):

**Windows:**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**Linux/macOS:**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

הפעל את השרת:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

השרת יתחיל בכתובת `http://localhost:8080`. תראה:
```
Started McpServerApplication in X.XXX seconds
```

### שלב 2: בדוק עם לקוח ישיר

בטרמינל **חדש** כששרת עדיין רץ, הפעל את לקוח MCP הישיר:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

תראה פלט כזה:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### שלב 3: בדוק עם לקוח AI

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

תראה שה-AI משתמש בכלים באופן אוטומטי:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### שלב 4: סגור את שרת MCP

כשסיימת לבדוק, תוכל לעצור את לקוח ה-AI על ידי לחיצה על `Ctrl+C` בטרמינל שלו. שרת ה-MCP ימשיך לרוץ עד שתעצור אותו.
כדי לעצור את השרת, לחץ `Ctrl+C` בטרמינל שבו הוא רץ.

## איך הכל עובד יחד

הנה התהליך המלא כשאתה שואל את ה-AI "מה זה 5 + 3?":

1. **אתה** שואל את ה-AI בשפה טבעית
2. **ה-AI** מנתח את הבקשה ומבין שברצונך לחבר מספרים
3. **ה-AI** קורא לשרת MCP: `add(5.0, 3.0)`
4. **שירות המחשבון** מבצע: `5.0 + 3.0 = 8.0`
5. **שירות המחשבון** מחזיר: `"5.00 + 3.00 = 8.00"`
6. **ה-AI** מקבל את התוצאה ומנסח תגובה טבעית
7. **אתה** מקבל: "הסכום של 5 ו-3 הוא 8"

## השלבים הבאים

לעוד דוגמאות, ראה [פרק 04: דוגמאות מעשיות](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**כתב ויתור**:
מסמך זה תורגם באמצעות שירות תרגום אוטומטי [Co-op Translator](https://github.com/Azure/co-op-translator). למרות שאנו שואפים לדיוק, יש לקחת בחשבון שתרגומים אוטומטיים עלולים להכיל שגיאות או אי-דיוקים. יש להחשיב את המסמך המקורי בשפתו הטבעית כמקור הסמכות. למידע קריטי מומלץ להשתמש בתרגום מקצועי על ידי מתרגם אדם. אנו לא אחראים לכל אי-הבנה או פירוש שגוי הנובע מהשימוש בתרגום זה.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->