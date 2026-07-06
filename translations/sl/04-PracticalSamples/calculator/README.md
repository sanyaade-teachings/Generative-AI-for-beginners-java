# Navodila za uporabo MCP kalkulatorja za začetnike

## Kazalo vsebine

- [Kaj se boste naučili](#kaj-se-boste-naučili)
- [Predpogoji](#predpogoji)
- [Razumevanje strukture projekta](#razumevanje-strukture-projekta)
- [Razlaga ključnih komponent](#razlaga-ključnih-komponent)
  - [1. Glavna aplikacija](#1-glavna-aplikacija)
  - [2. Kalkulator storitev](#2-kalkulator-storitev)
  - [3. Neposredni MCP klient](#3-neposredni-mcp-klient)
  - [4. AI-podprt klient](#4-ai-podprt-klient)
- [Zagon primerov](#zagon-primerov)
- [Kako vse skupaj deluje](#kako-vse-skupaj-deluje)
- [Naslednji koraki](#naslednji-koraki)

## Kaj se boste naučili

Ta vodič pojasnjuje, kako zgraditi kalkulator storitev z uporabo Model Context Protocol-a (MCP). Spoznali boste:

- Kako ustvariti storitev, ki jo AI lahko uporablja kot orodje
- Kako vzpostaviti neposredno komunikacijo z MCP storitvami
- Kako AI modeli lahko samodejno izberejo, katera orodja uporabiti
- Razliko med neposrednimi protokolarnimi klici in interakcijami, ki jih podpira AI

## Predpogoji

Pred začetkom poskrbite, da imate:
- nameščen Java 21 ali novejšo različico
- Maven za upravljanje odvisnosti
- Azure AI Foundry model nameščen (vzpostavite ga z `azd up` — glejte [Poglavje 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), prijavljen z `az login` (avtentikacija brez ključa)
- Osnovno razumevanje Jave in Spring Boot

## Razumevanje strukture projekta

Projekt kalkulator vsebuje več pomembnih datotek:

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

## Razlaga ključnih komponent

### 1. Glavna aplikacija

**Datoteka:** `McpServerApplication.java`

To je vhodna točka naše kalkulator storitve. Gre za standardno Spring Boot aplikacijo z eno posebno dodatno funkcijo:

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

**Kaj ta izvaja:**
- Zažene Spring Boot spletni strežnik na vratih 8080
- Ustvari `ToolCallbackProvider`, ki bo naredil metode kalkulatorja dostopne kot MCP orodja
- Oznaka `@Bean` pove Springu, naj to upravlja kot komponento, ki jo lahko uporabljajo druge dele aplikacije

### 2. Kalkulator storitev

**Datoteka:** `CalculatorService.java`

Tu se izvajajo vse matematične operacije. Vsaka metoda je označena z `@Tool`, da je dostopna preko MCP:

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
    
    // Več operacij kalkulatorja...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Ključne lastnosti:**

1. **Oznaka `@Tool`**: sporoča MCP, da se lahko ta metoda kliče iz zunanjih klientov
2. **Jasni opisi**: vsako orodje ima opis, ki AI modelom pomaga razumeti, kdaj ga uporabiti
3. **Konsistenten format rezultata**: vse operacije vračajo človeško berljive nize, kot na primer "5.00 + 3.00 = 8.00"
4. **Ravnanje z napakami**: deljenje z nič ter negativni koreni vrnejo sporočila o napaki

**Razpoložljive operacije:**
- `add(a, b)` - sešteje dve števili
- `subtract(a, b)` - odšteje drugo število od prvega
- `multiply(a, b)` - zmnoži dve števili
- `divide(a, b)` - deli prvo s drugim (s preverjanjem na ničlo)
- `power(base, exponent)` - potenča osnovo na eksponent
- `squareRoot(number)` - izračuna kvadratni koren (s preverjanjem na negativno vrednost)
- `modulus(a, b)` - vrne ostanek pri deljenju
- `absolute(number)` - vrne absolutno vrednost
- `help()` - vrne informacije o vseh operacijah

### 3. Neposredni MCP klient

**Datoteka:** `SDKClient.java`

Ta klient neposredno komunicira z MCP strežnikom brez uporabe AI. Ročno kliče specifične funkcije kalkulatorja:

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
        
        // Naštej razpoložljiva orodja
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Pokliči določene funkcije kalkulatorja
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

**Kaj ta dela:**
1. **Poveže se** s kalkulator strežnikom na `http://localhost:8080` z uporabo vzorca graditelja
2. **Izpiše** seznam vseh razpoložljivih orodij (naših kalkulator funkcij)
3. **Kliče** določene funkcije s točno določenimi parametri
4. **Izpiše** rezultate neposredno

**Opomba:** Ta primer uporablja Spring AI 1.1.0-SNAPSHOT odvisnost, ki je uvedla vzorec graditelja za `WebFluxSseClientTransport`. Če uporabljate starejšo stabilno različico, boste morda morali uporabiti neposredni konstruktor.

**Kdaj uporabiti to:** Ko točno veste, katero računanje želite izvesti in ga želite poklicati programsko.

### 4. AI-podprt klient

**Datoteka:** `LangChain4jClient.java`

Ta klient uporablja AI model (GPT-4o-mini), ki lahko samodejno odloči, katera orodja kalkulatorja uporabiti:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Nastavite AI model (Azure AI Foundry, brezključna avtentikacija preko Microsoft Entra ID)
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

        // Povežite se z našim MCP strežnikom kalkulatorja
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Prikaže, kaj AI počne
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Dajte AI dostop do naših orodij kalkulatorja
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Ustvarite AI bota, ki lahko uporablja naš kalkulator
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Zdaj lahko AI zaprosimo za izračune v naravnem jeziku
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Kaj ta dela:**
1. **Vzpostavi** povezavo z AI modelom z uporabo avtentikacije brez ključa (Microsoft Entra ID)
2. **Poveže** AI s kalkulator MCP strežnikom
3. **Omogoči** AI dostop do vseh naših kalkulator orodij
4. **Dovoli** naravne jezikovne zahteve, kot so "Izračunaj vsoto 24,5 in 17,3"

**AI samodejno:**
- Razume, da želite seštevanje
- Izbere orodje `add`
- Pokliče `add(24.5, 17.3)`
- Vrne rezultat v naravnem odgovoru

## Zagon primerov

### Korak 1: Zaženite strežnik kalkulatorja

Najprej se prijavite in nastavite svojo Azure AI Foundry končno točko (potrebna za AI klienta — avtentikacija brez ključa, brez API ključa):

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

Zaženite strežnik:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

Strežnik bo dostopen na `http://localhost:8080`. Videli boste:
```
Started McpServerApplication in X.XXX seconds
```

### Korak 2: Preizkusite z neposrednim klientom

V **NOVEM** terminalu, medtem ko strežnik še teče, zaženite neposrednega MCP klienta:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Videli boste izpis, kot je:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Korak 3: Preizkusite z AI klientom

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Videli boste, da AI samodejno uporablja orodja:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Korak 4: Zaprite MCP strežnik

Ko končate s testiranjem, lahko AI klienta ustavite s pritiskom `Ctrl+C` v njegovem terminalu. MCP strežnik bo tekel, dokler ga sami ne ustavite.
Za zaustavitev strežnika pritisnite `Ctrl+C` v terminalu, kjer teče.

## Kako vse skupaj deluje

Tukaj je celoten potek, ko AI vprašate "Kaj je 5 + 3?":

1. **Vi** postavite vprašanje AI v naravnem jeziku
2. **AI** analizira zahtevo in ugotovi, da želite seštevanje
3. **AI** pokliče MCP strežnik: `add(5.0, 3.0)`
4. **Kalkulator storitev** izvede: `5.0 + 3.0 = 8.0`
5. **Kalkulator storitev** vrne: `"5.00 + 3.00 = 8.00"`
6. **AI** prejme rezultat in oblikuje naraven odgovor
7. **Vi** prejmete: "Vsota 5 in 3 je 8"

## Naslednji koraki

Za več primerov glejte [Poglavje 04: Praktični primeri](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Omejitev odgovornosti**:
Ta dokument je bil preveden z uporabo AI prevajalske storitve [Co-op Translator](https://github.com/Azure/co-op-translator). Čeprav si prizadevamo za natančnost, vas prosimo, da upoštevate, da avtomatizirani prevodi lahko vsebujejo napake ali netočnosti. Izvirni dokument v njegovem izvirnem jeziku je treba obravnavati kot avtoritativni vir. Za kritične informacije je priporočljiv strokovni človeški prevod. Ne odgovarjamo za morebitna nesporazume ali napačne interpretacije, ki izhajajo iz uporabe tega prevoda.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->