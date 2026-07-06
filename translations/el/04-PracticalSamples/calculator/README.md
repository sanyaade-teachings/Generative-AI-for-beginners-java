# Οδηγός Υπολογιστή MCP για Αρχάριους

## Περιεχόμενα

- [Τι Θα Μάθετε](#τι-θα-μάθετε)
- [Απαιτούμενα](#απαιτούμενα)
- [Κατανόηση της Δομής του Έργου](#κατανόηση-της-δομής-του-έργου)
- [Επεξήγηση Βασικών Συστατικών](#επεξήγηση-βασικών-συστατικών)
  - [1. Κύρια Εφαρμογή](#1-κύρια-εφαρμογή)
  - [2. Υπηρεσία Υπολογιστή](#2-υπηρεσία-υπολογιστή)
  - [3. Άμεσος Πελάτης MCP](#3-άμεσος-πελάτης-mcp)
  - [4. Πελάτης με Τεχνητή Νοημοσύνη](#4-πελάτης-με-τεχνητή-νοημοσύνη)
- [Εκτέλεση των Παραδειγμάτων](#εκτέλεση-των-παραδειγμάτων)
- [Πώς Λειτουργεί Όλο Μαζί](#πώς-λειτουργεί-όλο-μαζί)
- [Επόμενα Βήματα](#επόμενα-βήματα)

## Τι Θα Μάθετε

Αυτός ο οδηγός εξηγεί πώς να φτιάξετε μια υπηρεσία υπολογιστή χρησιμοποιώντας το Πρωτόκολλο Πλαισίου Μοντέλου (Model Context Protocol - MCP). Θα κατανοήσετε:

- Πώς να δημιουργήσετε μια υπηρεσία που η Τεχνητή Νοημοσύνη μπορεί να χρησιμοποιήσει ως εργαλείο
- Πώς να ρυθμίσετε άμεση επικοινωνία με υπηρεσίες MCP
- Πώς τα μοντέλα AI μπορούν αυτόματα να επιλέγουν ποια εργαλεία να χρησιμοποιήσουν
- Τη διαφορά μεταξύ άμεσων κλήσεων πρωτοκόλλου και αλληλεπιδράσεων με βοήθεια AI

## Απαιτούμενα

Πριν ξεκινήσετε, βεβαιωθείτε ότι έχετε:
- Εγκατεστημένη Java 21 ή νεότερη έκδοση
- Maven για τη διαχείριση εξαρτήσεων
- Μια ανάπτυξη μοντέλου Azure AI Foundry (κάντε provisioning με `azd up` — δείτε [Κεφάλαιο 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- Το [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), με σύνδεση μέσω `az login` (αυθεντικοποίηση χωρίς κλειδί)
- Βασική κατανόηση Java και Spring Boot

## Κατανόηση της Δομής του Έργου

Το έργο του υπολογιστή έχει αρκετά σημαντικά αρχεία:

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

## Επεξήγηση Βασικών Συστατικών

### 1. Κύρια Εφαρμογή

**Αρχείο:** `McpServerApplication.java`

Αυτή είναι η είσοδος της υπηρεσίας υπολογιστή μας. Πρόκειται για μια τυπική εφαρμογή Spring Boot με μια ειδική προσθήκη:

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

**Τι κάνει αυτό:**
- Εκκινεί έναν web server Spring Boot στην πόρτα 8080
- Δημιουργεί έναν `ToolCallbackProvider` που καθιστά τις μεθόδους του υπολογιστή μας διαθέσιμες ως εργαλεία MCP
- Η ετικέτα `@Bean` λέει στο Spring να το διαχειρίζεται ως συστατικό το οποίο μπορούν να χρησιμοποιούν άλλες μονάδες

### 2. Υπηρεσία Υπολογιστή

**Αρχείο:** `CalculatorService.java`

Εδώ γίνονται όλοι οι υπολογισμοί. Κάθε μέθοδος σηματοδοτείται με `@Tool` για να είναι διαθέσιμη μέσω MCP:

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
    
    // Περισσότερες λειτουργίες αριθμομηχανής...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Κύρια χαρακτηριστικά:**

1. **Σήμανση `@Tool`**: Δηλώνει στο MCP ότι αυτή η μέθοδος μπορεί να κληθεί από εξωτερικούς πελάτες
2. **Καθαρές Περιγραφές**: Κάθε εργαλείο έχει μια περιγραφή που βοηθά τα μοντέλα AI να καταλάβουν πότε να το χρησιμοποιήσουν
3. **Συνεπής Μορφή Επιστροφής**: Όλες οι εντολές επιστρέφουν ανθρώπινα αναγνώσιμες συμβολοσειρές όπως "5.00 + 3.00 = 8.00"
4. **Διαχείριση Σφαλμάτων**: Η διαίρεση με το μηδέν και οι αρνητικές τετραγωνικές ρίζες επιστρέφουν μηνύματα σφάλματος

**Διαθέσιμες Λειτουργίες:**
- `add(a, b)` - Προσθέτει δύο αριθμούς
- `subtract(a, b)` - Αφαιρεί τον δεύτερο από τον πρώτο
- `multiply(a, b)` - Πολλαπλασιάζει δύο αριθμούς
- `divide(a, b)` - Διαιρεί τον πρώτο με τον δεύτερο (με έλεγχο μηδενικού)
- `power(base, exponent)` - Υψώνει τη βάση στη δύναμη του εκθέτη
- `squareRoot(number)` - Υπολογίζει τετραγωνική ρίζα (με έλεγχο για αρνητικό)
- `modulus(a, b)` - Επιστρέφει το υπόλοιπο της διαίρεσης
- `absolute(number)` - Επιστρέφει την απόλυτη τιμή
- `help()` - Επιστρέφει πληροφορίες για όλες τις λειτουργίες

### 3. Άμεσος Πελάτης MCP

**Αρχείο:** `SDKClient.java`

Αυτός ο πελάτης επικοινωνεί απευθείας με τον MCP server χωρίς να χρησιμοποιεί AI. Καλεί χειροκίνητα συγκεκριμένες λειτουργίες υπολογιστή:

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
        
        // Καταλόγισε τα διαθέσιμα εργαλεία
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Κλήση συγκεκριμένων συναρτήσεων αριθμομηχανής
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

**Τι κάνει αυτό:**
1. **Συνδέεται** στον server υπολογιστή στο `http://localhost:8080` χρησιμοποιώντας το πρότυπο builder
2. **Λίστα** όλων των διαθέσιμων εργαλείων (των λειτουργιών υπολογιστή μας)
3. **Καλεί** συγκεκριμένες λειτουργίες με ακριβείς παραμέτρους
4. **Εκτυπώνει** απευθείας τα αποτελέσματα

**Σημείωση:** Αυτό το παράδειγμα χρησιμοποιεί την εξάρτηση Spring AI 1.1.0-SNAPSHOT, που εισήγαγε το πρότυπο builder για το `WebFluxSseClientTransport`. Αν χρησιμοποιείτε παλιότερη σταθερή έκδοση, μπορεί να χρειαστεί να χρησιμοποιήσετε απευθείας τον κατασκευαστή.

**Πότε να το χρησιμοποιήσετε:** Όταν ξέρετε ακριβώς ποιον υπολογισμό θέλετε να κάνετε και θέλετε να τον καλέσετε προγραμματιστικά.

### 4. Πελάτης με Τεχνητή Νοημοσύνη

**Αρχείο:** `LangChain4jClient.java`

Αυτός ο πελάτης χρησιμοποιεί μοντέλο AI (GPT-4o-mini) που μπορεί αυτόματα να αποφασίσει ποια εργαλεία υπολογιστή να χρησιμοποιήσει:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Ορίστε το μοντέλο AI (Azure AI Foundry, αυθεντικοποίηση χωρίς κλειδί μέσω Microsoft Entra ID)
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

        // Συνδεθείτε στον διακομιστή MCP του υπολογιστή μας
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Δείχνει τι κάνει το AI
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Δώστε στο AI πρόσβαση στα εργαλεία του υπολογιστή μας
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Δημιουργήστε ένα bot AI που μπορεί να χρησιμοποιεί τον υπολογιστή μας
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Τώρα μπορούμε να ζητήσουμε από το AI να κάνει υπολογισμούς σε φυσική γλώσσα
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Τι κάνει αυτό:**
1. **Δημιουργεί** σύνδεση με το μοντέλο AI χρησιμοποιώντας αυθεντικοποίηση χωρίς κλειδί (Microsoft Entra ID)
2. **Συνδέει** το AI με τον MCP server του υπολογιστή μας
3. **Παρέχει** στο AI πρόσβαση σε όλα τα εργαλεία του υπολογιστή μας
4. **Επιτρέπει** αιτήματα σε φυσική γλώσσα όπως "Υπολόγισε το άθροισμα του 24.5 και 17.3"

**Το AI αυτόματα:**
- Καταλαβαίνει ότι θέλετε να προσθέσετε αριθμούς
- Επιλέγει το εργαλείο `add`
- Καλεί `add(24.5, 17.3)`
- Επιστρέφει το αποτέλεσμα σε φυσική απάντηση

## Εκτέλεση των Παραδειγμάτων

### Βήμα 1: Εκκίνηση του Server Υπολογιστή

Πρώτα, συνδεθείτε και ορίστε το endpoint του Azure AI Foundry (απαραίτητο για τον πελάτη AI — αυθεντικοποίηση χωρίς κλειδί, χωρίς API key):

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

Εκκινήστε τον server:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

Ο server θα ξεκινήσει στο `http://localhost:8080`. Θα δείτε:
```
Started McpServerApplication in X.XXX seconds
```

### Βήμα 2: Δοκιμή με Άμεσο Πελάτη

Σε ένα **ΝΕΟ** τερματικό ενώ ο Server τρέχει, εκτελέστε τον άμεσο MCP πελάτη:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Θα δείτε έξοδο όπως:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Βήμα 3: Δοκιμή με Πελάτη AI

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Θα δείτε το AI να χρησιμοποιεί αυτόματα εργαλεία:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Βήμα 4: Κλείσιμο του MCP Server

Όταν τελειώσετε με τις δοκιμές, μπορείτε να σταματήσετε τον πελάτη AI πατώντας `Ctrl+C` στο τερματικό του. Ο MCP server θα συνεχίσει να τρέχει μέχρι να τον σταματήσετε.
Για να σταματήσετε τον server, πατήστε `Ctrl+C` στο τερματικό όπου τρέχει.

## Πώς Λειτουργεί Όλο Μαζί

Ακολουθεί η πλήρης διαδικασία όταν ρωτάτε το AI «Πόσο κάνει 5 + 3;»:

1. **Εσείς** ρωτάτε το AI σε φυσική γλώσσα
2. **Το AI** αναλύει το αίτημα και καταλαβαίνει ότι θέλετε πρόσθεση
3. **Το AI** καλεί τον MCP server: `add(5.0, 3.0)`
4. **Η Υπηρεσία Υπολογιστή** εκτελεί: `5.0 + 3.0 = 8.0`
5. **Η Υπηρεσία Υπολογιστή** επιστρέφει: `"5.00 + 3.00 = 8.00"`
6. **Το AI** λαμβάνει το αποτέλεσμα και διαμορφώνει φυσική απάντηση
7. **Εσείς** λαμβάνετε: "Το άθροισμα του 5 και του 3 είναι 8"

## Επόμενα Βήματα

Για περισσότερα παραδείγματα, δείτε [Κεφάλαιο 04: Πρακτικά παραδείγματα](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Αποποίηση ευθυνών**:
Αυτό το έγγραφο έχει μεταφραστεί χρησιμοποιώντας την υπηρεσία μετάφρασης με τεχνητή νοημοσύνη [Co-op Translator](https://github.com/Azure/co-op-translator). Ενώ επιδιώκουμε την ακρίβεια, παρακαλούμε να έχετε υπόψη ότι οι αυτοματοποιημένες μεταφράσεις ενδέχεται να περιέχουν λάθη ή ανακρίβειες. Το πρωτότυπο έγγραφο στη μητρική του γλώσσα πρέπει να θεωρείται η αυθεντική πηγή. Για κρίσιμες πληροφορίες, συνιστάται επαγγελματική ανθρώπινη μετάφραση. Δεν φέρουμε ευθύνη για τυχόν παρεξηγήσεις ή λανθασμένες ερμηνείες που προκύπτουν από τη χρήση αυτής της μετάφρασης.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->