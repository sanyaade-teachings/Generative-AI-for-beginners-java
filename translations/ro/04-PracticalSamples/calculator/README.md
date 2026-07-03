# Tutorial calculator MCP pentru începători

## Cuprins

- [Ce vei învăța](#ce-vei-învața)
- [Precondiții](#precondiții)
- [Înțelegerea structurii proiectului](#înțelegerea-structurii-proiectului)
- [Componentele principale explicate](#componentele-principale-explicate)
  - [1. Aplicația principală](#1-aplicația-principală)
  - [2. Serviciul calculator](#2-serviciul-calculator)
  - [3. Client MCP direct](#3-client-mcp-direct)
  - [4. Client asistat de AI](#4-client-asistat-de-ai)
- [Rularea exemplelor](#rularea-exemplelor)
- [Cum funcționează totul împreună](#cum-funcționează-totul-împreună)
- [Pașii următori](#pașii-următori)

## Ce vei învăța

Acest tutorial explică cum să construiești un serviciu de calculator folosind Model Context Protocol (MCP). Vei înțelege:

- Cum să creezi un serviciu pe care AI îl poate folosi ca instrument
- Cum să stabilești o comunicare directă cu serviciile MCP
- Cum modelele AI pot alege automat ce instrumente să utilizeze
- Diferența dintre apeluri directe prin protocol și interacțiuni asistenționate de AI

## Precondiții

Înainte de a începe, asigură-te că ai:
- Java 21 sau o versiune superioară instalată
- Maven pentru gestionarea dependențelor
- O implementare model Azure AI Foundry (provisionată cu `azd up` — vezi [Capitolul 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), autentificat cu `az login` (autentificare fără cheie)
- Cunoștințe de bază despre Java și Spring Boot

## Înțelegerea structurii proiectului

Proiectul calculator are mai multe fișiere importante:

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

## Componentele principale explicate

### 1. Aplicația principală

**Fișier:** `McpServerApplication.java`

Acesta este punctul de intrare al serviciului nostru de calculator. Este o aplicație standard Spring Boot cu o singură adiție specială:

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

**Ce face acest cod:**
- Pornește un server web Spring Boot pe portul 8080
- Creează un `ToolCallbackProvider` care face metodele noastre de calculator disponibile ca unelte MCP
- Anotarea `@Bean` indică Spring să gestioneze această componentă pentru ca alte părți să o poată folosi

### 2. Serviciul calculator

**Fișier:** `CalculatorService.java`

Aici se face toată matematica. Fiecare metodă este marcată cu `@Tool` pentru a fi disponibilă prin MCP:

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
    
    // Mai multe operații ale calculatorului...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Caracteristici cheie:**

1. **Anotarea `@Tool`**: Indică MCP că această metodă poate fi apelată de clienți externi
2. **Descrieri clare**: Fiecare unealtă are o descriere care ajută modelele AI să înțeleagă când să o folosească
3. **Format consistent de retur**: Toate operațiile returnează șiruri ușor de citit, de exemplu "5.00 + 3.00 = 8.00"
4. **Gestionarea erorilor**: Împărțirea la zero și rădăcina pătrată a unui număr negativ returnează mesaje de eroare

**Operații disponibile:**
- `add(a, b)` - Adună două numere
- `subtract(a, b)` - Scade al doilea număr din primul
- `multiply(a, b)` - Înmulțește două numere
- `divide(a, b)` - Împarte primul la al doilea (cu verificare zero)
- `power(base, exponent)` - Ridică baza la puterea exponentului
- `squareRoot(number)` - Calculează rădăcina pătrată (cu verificare negativă)
- `modulus(a, b)` - Returnează restul împărțirii
- `absolute(number)` - Returnează valoarea absolută
- `help()` - Returnează informații despre toate operațiile

### 3. Client MCP direct

**Fișier:** `SDKClient.java`

Acest client comunică direct cu serverul MCP fără a folosi AI. Apelează manual funcții specifice ale calculatorului:

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
        
        // Listează uneltele disponibile
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Apelează funcții specifice ale calculatorului
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

**Ce face acest cod:**
1. **Se conectează** la serverul calculator la `http://localhost:8080` folosind pattern-ul builder
2. **Listează** toate uneltele disponibile (funcțiile calculatorului nostru)
3. **Apelează** funcții specifice cu parametri exacți
4. **Afișează** rezultatele direct

**Notă:** Acest exemplu folosește dependența Spring AI 1.1.0-SNAPSHOT, care a introdus pattern-ul builder pentru `WebFluxSseClientTransport`. Dacă folosești o versiune stabilă mai veche, poate fi necesar să folosești constructorul direct.

**Când să folosești asta:** Când știi exact ce calcul vrei să faci și dorești să îl apelezi programatic.

### 4. Client asistat de AI

**Fișier:** `LangChain4jClient.java`

Acest client folosește un model AI (GPT-4o-mini) care poate decide automat ce unelte calculator să utilizeze:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Configurați modelul AI (Azure AI Foundry, autentificare fără cheie prin Microsoft Entra ID)
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

        // Conectați-vă la serverul nostru de calculator MCP
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Arată ce face AI-ul
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Oferiți AI-ului acces la uneltele noastre de calculator
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Creați un bot AI care poate folosi calculatorul nostru
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Acum putem cere AI-ului să efectueze calcule în limbaj natural
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Ce face acest cod:**
1. **Creează** o conexiune la modelul AI folosind token-ul tău GitHub
2. **Conectează** AI-ul la serverul nostru calculator MCP
3. **Oferă** AI-ului acces la toate uneltele calculatorului nostru
4. **Permite** cereri în limbaj natural ca "Calculează suma dintre 24.5 și 17.3"

**AI-ul automat:**
- Înțelege că vrei să aduni numere
- Alege unealta `add`
- Apelează `add(24.5, 17.3)`
- Returnează rezultatul într-un răspuns natural

## Rularea exemplelor

### Pasul 1: Pornește serverul calculator

Mai întâi, autentifică-te și setează punctul tău final Azure AI Foundry (necesar pentru clientul AI — autentificare fără cheie, fără cheie API):

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

Pornește serverul:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

Serverul va porni la `http://localhost:8080`. Ar trebui să vezi:
```
Started McpServerApplication in X.XXX seconds
```

### Pasul 2: Testează cu clientul direct

Într-un terminal **NOU**, cu serverul încă pornit, rulează clientul MCP direct:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Vei vedea o ieșire de genul:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Pasul 3: Testează cu clientul AI

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Vei vedea AI-ul folosind uneltele automat:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Pasul 4: Închide serverul MCP

Când termini testarea, poți opri clientul AI apăsând `Ctrl+C` în terminalul lui. Serverul MCP va continua să funcționeze până îl oprești.
Pentru a opri serverul, apasă `Ctrl+C` în terminalul în care rulează.

## Cum funcționează totul împreună

Iată fluxul complet când întrebi AI-ul „Cât face 5 + 3?”:

1. **Tu** întrebi AI-ul în limbaj natural
2. **AI-ul** analizează cererea și realizează că dorești adunare
3. **AI-ul** apelează serverul MCP: `add(5.0, 3.0)`
4. **Serviciul calculator** execută: `5.0 + 3.0 = 8.0`
5. **Serviciul calculator** returnează: `"5.00 + 3.00 = 8.00"`
6. **AI-ul** primește rezultatul și formulează un răspuns natural
7. **Tu** primești: "Suma lui 5 și 3 este 8"

## Pașii următori

Pentru mai multe exemple, vezi [Capitolul 04: Exemple practice](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Declinare a responsabilității**:
Acest document a fost tradus folosind serviciul de traducere AI [Co-op Translator](https://github.com/Azure/co-op-translator). În timp ce ne străduim pentru acuratețe, vă rugăm să rețineți că traducerile automate pot conține erori sau inexactități. Documentul original în limba sa nativă trebuie considerat sursa autorizată. Pentru informații critice, se recomandă traducerea profesională realizată de un om. Nu ne asumăm responsabilitatea pentru eventualele neînțelegeri sau interpretări greșite care decurg din utilizarea acestei traduceri.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->