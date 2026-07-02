# מדריך מחשבון MCP למתחילים

## תוכן העניינים

- [מה תלמדו](#מה-תלמדו)
- [דרישות מוקדמות](#דרישות-מוקדמות)
- [הבנת מבנה הפרויקט](#הבנת-מבנה-הפרויקט)
- [הסבר על הרכיבים המרכזיים](#הסבר-על-הרכיבים-המרכזיים)
  - [1. האפליקציה הראשית](#1-האפליקציה-הראשית)
  - [2. שירות המחשבון](#2-שירות-המחשבון)
  - [3. לקוח MCP ישיר](#3-לקוח-mcp-ישיר)
  - [4. לקוח מבוסס בינה מלאכותית](#4-לקוח-מבוסס-בינה-מלאכותית)
- [הרצת הדוגמאות](#הרצת-הדוגמאות)
- [כיצד הכל עובד יחד](#כיצד-הכל-עובד-יחד)
- [השלבים הבאים](#השלבים-הבאים)

## מה תלמדו

מדריך זה מסביר כיצד לבנות שירות מחשבון באמצעות پروטוקול הקשר מודל (MCP). תלמדו:

- כיצד ליצור שירות שהבינה המלאכותית יכולה להשתמש בו ככלי
- כיצד להגדיר תקשורת ישירה עם שירותי MCP
- כיצד דגמי AI יכולים לבחור באופן אוטומטי אילו כלים להשתמש
- ההבדל בין קריאות פרוטוקול ישירות לבין אינטראקציות בסיוע AI

## דרישות מוקדמות

לפני שמתחילים, ודאו שיש לכם:
- Java 21 או גרסה גבוהה יותר מותקנת
- Maven לניהול תלותים
- פריסת דגם Azure AI Foundry (הפעילו אותו עם `azd up` — ראו [פרק 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) מחובר עם `az login` (אימות ללא מפתח)
- הבנה בסיסית ב-Java ו-Spring Boot

## הבנת מבנה הפרויקט

לפרויקט המחשבון יש כמה קבצים חשובים:

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

## הסבר על הרכיבים המרכזיים

### 1. האפליקציה הראשית

**קובץ:** `McpServerApplication.java`

זוהי נקודת הכניסה של שירות המחשבון שלנו. זוהי אפליקציית Spring Boot סטנדרטית עם תוספת מיוחדת אחת:

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

**מה זה עושה:**
- מפעיל שרת ווב Spring Boot ב-port 8080
- יוצר `ToolCallbackProvider` שהופך את שיטות המחשבון שלנו לזמינות ככלי MCP
- התווית `@Bean` אומרת ל-Spring לנהל את זה כרכיב שיכולות להשתמש בו חלקים אחרים

### 2. שירות המחשבון

**קובץ:** `CalculatorService.java`

כאן מתרחשות כל הפעולות המתמטיות. כל שיטה מסומנת עם `@Tool` כדי להופיע דרך MCP:

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
    
    // פעולות מחשבון נוספות...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**מאפיינים מרכזיים:**

1. **התווית `@Tool`**: מודיעה ל-MCP כי שיטה זו ניתנת לקריאה על ידי לקוחות חיצוניים
2. **תיאורים ברורים**: לכל כלי יש תיאור שעוזר לדגמי AI להבין מתי להשתמש בו
3. **פורמט החזרה עקבי**: כל הפעולות מחזירות מחרוזות קריאות לבני אדם כמו "5.00 + 3.00 = 8.00"
4. **טיפול בשגיאות**: חילוק באפס ושורש ריבועי שלילי מחזירים הודעות שגיאה

**פעולות זמינות:**
- `add(a, b)` - מחבר שני מספרים
- `subtract(a, b)` - מחסיר את השני מהראשון
- `multiply(a, b)` - מכפיל שני מספרים
- `divide(a, b)` - מחלק את הראשון בשני (עם בדיקת אפס)
- `power(base, exponent)` - מעלים בחזקה
- `squareRoot(number)` - מחשב שורש ריבועי (עם בדיקה שלילי)
- `modulus(a, b)` - מחזיר שארית חלוקה
- `absolute(number)` - מחזיר ערך מוחלט
- `help()` - מחזיר מידע על כל הפעולות

### 3. לקוח MCP ישיר

**קובץ:** `SDKClient.java`

לקוח זה מתקשר ישירות עם שרת MCP ללא שימוש ב-AI. הוא קורא ידנית לפונקציות ספציפיות של המחשבון:

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
        
        // רשימת כלים זמינים
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

**מה זה עושה:**
1. **מתחבר** לשרת המחשבון בכתובת `http://localhost:8080` באמצעות תבנית בנאי (builder)
2. **מציג** רשימה של כל הכלים הזמינים (הפונקציות של המחשבון)
3. **קורא** לפונקציות מסוימות עם פרמטרים מדויקים
4. **מציג** את התוצאות ישירות

**הערה:** דוגמה זו משתמשת בתלות Spring AI 1.1.0-SNAPSHOT, שהוסיפה תבנית בנאי עבור `WebFluxSseClientTransport`. אם אתם משתמשים בגרסה יציבה ישנה יותר, ייתכן ותצטרכו להשתמש בקונסטרקטור הישיר במקום.

**מתי להשתמש בזה:** כאשר אתם יודעים בדיוק איזה חישוב לבצע ורוצים לקרוא לו מתוכנית.

### 4. לקוח מבוסס בינה מלאכותית

**קובץ:** `LangChain4jClient.java`

לקוח זה משתמש בדגם AI (GPT-4o-mini) שיכול לבחור באופן אוטומטי אילו כלים של המחשבון להשתמש:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // הגדר את מודל ה-AI (Azure AI Foundry, אימות ללא מפתח דרך Microsoft Entra ID)
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
                .logRequests(true)  // מציג מה ה-AI עושה
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

**מה זה עושה:**
1. **יוצר** חיבור לדגם AI באמצעות טוקן GitHub שלכם
2. **מחבר** את ה-AI לשרת MCP של המחשבון שלנו
3. **נותן** ל-AI גישה לכל כלי המחשבון שלנו
4. **מאפשר** בקשות בשפה טבעית כמו "חשב את הסכום של 24.5 ו-17.3"

**ה-AI עושה באופן אוטומטי:**
- מבין שאתם רוצים לחבר מספרים
- בוחר את הכלי `add`
- קורא ל-`add(24.5, 17.3)`
- מחזיר את התוצאה בתגובה טבעית

## הרצת הדוגמאות

### שלב 1: הפעלת שרת המחשבון

ראשית, היכנסו והגדירו את נקודת הסיום של Azure AI Foundry (נדרש ללקוח AI — אימות ללא מפתח):

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

השרת יפעל בכתובת `http://localhost:8080`. אמור להיראות כך:
```
Started McpServerApplication in X.XXX seconds
```

### שלב 2: בדיקה עם הלקוח הישיר

בטרמינל **חדש** כשהשרת עדיין רץ, הריצו את הלקוח הישיר MCP:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

תראו פלט כמו:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### שלב 3: בדיקה עם הלקוח AI

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

תראו שה-AI משתמש בכלים אוטומטית:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### שלב 4: סגירת שרת MCP

כשתסיימו לבדוק, תוכלו לעצור את לקוח ה-AI על ידי לחיצה על `Ctrl+C` בטרמינל שלו. שרת MCP ימשיך לרוץ עד שתעצורוהו.
כדי לעצור את השרת, לחצו על `Ctrl+C` בטרמינל שבו הוא רץ.

## כיצד הכל עובד יחד

זה הזרימה המלאה כאשר אתם שואלים את ה-AI "כמה זה 5 + 3?":

1. **אתם** שואלים את ה-AI בשפה טבעית
2. **ה-AI** מנתח את הבקשה ומזהה שאתם רוצים חיבור
3. **ה-AI** קורא לשרת MCP: `add(5.0, 3.0)`
4. **שירות המחשבון** מבצע: `5.0 + 3.0 = 8.0`
5. **שירות המחשבון** מחזיר: `"5.00 + 3.00 = 8.00"`
6. **ה-AI** מקבל את התוצאה ומנסח תגובה טבעית
7. **אתם** מקבלים: "סכום 5 ו-3 הוא 8"

## השלבים הבאים

למידע נוסף, ראו [פרק 04: דוגמאות מעשיות](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**כתב ויתור**:
מסמך זה תורגם באמצעות שירות תרגום אוטומטי [Co-op Translator](https://github.com/Azure/co-op-translator). למרות שאנו שואפים לדיוק, יש לקחת בחשבון שתרגומים אוטומטיים עלולים להכיל שגיאות או אי-דיוקים. יש להחשיב את המסמך המקורי בשפתו הטבעית כמקור הסמכות. למידע קריטי מומלץ להשתמש בתרגום מקצועי על ידי מתרגם אדם. אנו לא אחראים לכל אי-הבנה או פירוש שגוי הנובע מהשימוש בתרגום זה.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->