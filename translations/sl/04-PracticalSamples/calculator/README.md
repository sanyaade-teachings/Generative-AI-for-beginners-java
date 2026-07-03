# MCP kalkulator vodič za začetnike

## Kazalo

- [Kaj se boste naučili](#kaj-se-boste-naučili)
- [Predpogoji](#predpogoji)
- [Razumevanje strukture projekta](#razumevanje-strukture-projekta)
- [Razlaga glavnih komponent](#razlaga-glavnih-komponent)
  - [1. Glavna aplikacija](#1-glavna-aplikacija)
  - [2. Kalkulatorska storitev](#2-kalkulatorska-storitev)
  - [3. Neposredni MCP odjemalec](#3-neposredni-mcp-odjemalec)
  - [4. Odjemalec na osnovi AI](#4-odjemalec-na-osnovi-ai)
- [Zagon primerov](#zagon-primerov)
- [Kako vse deluje skupaj](#kako-vse-deluje-skupaj)
- [Naslednji koraki](#naslednji-koraki)

## Kaj se boste naučili

Ta vodič razloži, kako zgraditi kalkulatorsko storitev z uporabo protokola Model Context Protocol (MCP). Spoznali boste:

- Kako ustvariti storitev, ki jo AI lahko uporablja kot orodje
- Kako vzpostaviti neposredno komunikacijo z MCP storitvami
- Kako lahko AI modeli samodejno izbirajo orodja
- Razliko med neposrednimi protokolskimi klici in AI-podprtimi interakcijami

## Predpogoji

Pred začetkom poskrbite za:
- Java 21 ali novejšo nameščeno različico
- Maven za upravljanje odvisnosti
- Azure AI Foundry model deployment (zagotovite ga z `azd up` — glejte [Poglavje 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), prijavljeni z `az login` (avtentikacija brez ključa)
- Osnovno razumevanje Jave in Spring Boota

## Razumevanje strukture projekta

Kalkulator projekt vsebuje več pomembnih datotek:

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

## Razlaga glavnih komponent

### 1. Glavna aplikacija

**Datoteka:** `McpServerApplication.java`

To je vhodna točka naše kalkulatorske storitve. Gre za standardno Spring Boot aplikacijo z eno posebno dodatkom:

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

**Kaj to počne:**
- Zažene Spring Boot spletni strežnik na vratih 8080
- Ustvari `ToolCallbackProvider`, ki naredi naše kalkulatorske metode dostopne kot MCP orodja
- `@Bean` anotacija pove Springu, naj to upravlja kot komponento, ki jo lahko uporabljajo drugi deli

### 2. Kalkulatorska storitev

**Datoteka:** `CalculatorService.java`

Tukaj se dogajajo vsi matematični izračuni. Vsaka metoda je označena z `@Tool`, da je dostopna preko MCP:

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

1. **`@Tool` Anotacija**: Sporoča MCP, da lahko te metode kličejo zunanji odjemalci
2. **Jasni Opisi**: Vsako orodje ima opis, ki pomaga AI modelom razumeti, kdaj ga uporabiti
3. **Enoten format vračanja**: Vse operacije vračajo človeško berljive nize kot "5.00 + 3.00 = 8.00"
4. **Ravnanje z napakami**: Deljenje z nič in korenjenje negativnih števil vrneta sporočila o napaki

**Razpoložljive operacije:**
- `add(a, b)` - sešteje dve števili
- `subtract(a, b)` - odšteje drugo od prvega
- `multiply(a, b)` - pomnoži dve števili
- `divide(a, b)` - deli prvo z drugim (s preverjanjem za ničlo)
- `power(base, exponent)` - eksponentira osnovo na moč eksponenta
- `squareRoot(number)` - izračuna kvadratni koren (s preverjanjem negativnosti)
- `modulus(a, b)` - vrne ostanek pri deljenju
- `absolute(number)` - vrne absolutno vrednost
- `help()` - vrne informacije o vseh operacijah

### 3. Neposredni MCP odjemalec

**Datoteka:** `SDKClient.java`

Ta odjemalec komunicira neposredno z MCP strežnikom brez uporabe AI. Ročno kliče specifične kalkulatorske funkcije:

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

**Kaj to počne:**
1. **Poveže se** s kalkulatorskim strežnikom na `http://localhost:8080` z uporabo vzorca builder
2. **Našteje** vsa razpoložljiva orodja (naše kalkulatorske funkcije)
3. **Kliče** specifične funkcije z natančnimi parametri
4. **Izpiše** rezultate neposredno

**Opomba:** Ta primer uporablja Spring AI verzijo 1.1.0-SNAPSHOT, ki je uvedla builder vzorec za `WebFluxSseClientTransport`. Če uporabljate starejšo stabilno različico, boste morda morali uporabiti neposredni konstruktor.

**Kdaj to uporabiti:** Ko točno veste, kakšen izračun želite izvesti in ga želite poklicati programsko.

### 4. Odjemalec na osnovi AI

**Datoteka:** `LangChain4jClient.java`

Ta odjemalec uporablja AI model (GPT-4o-mini), ki samodejno izbere, katera kalkulatorska orodja uporabiti:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Nastavite AI model (Azure AI Foundry, avtentikacija brez ključa preko Microsoft Entra ID)
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

        // Povežite se z našim strežnikom kalkulatorja MCP
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

        // Zdaj lahko od AI zahtevamo izračune v naravnem jeziku
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Kaj to počne:**
1. **Ustvari** povezavo do AI modela z vašim GitHub žetonom
2. **Poveže** AI z našim kalkulatorskim MCP strežnikom
3. **Daje** AI dostop do vseh naših kalkulatorskih orodij
4. **Omogoča** naravne jezikovne zahteve, kot je "Izračunaj vsoto 24.5 in 17.3"

**AI samodejno:**
- Razume, da želite seštevati številke
- Izbere orodje `add`
- Kliče `add(24.5, 17.3)`
- Vrne rezultat v naravnem odgovoru

## Zagon primerov

### Korak 1: Zaženite kalkulatorski strežnik

Najprej se prijavite in nastavite vaš Azure AI Foundry endpoint (potrebno za AI odjemalca — avtentikacija brez ključa, brez API ključev):

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

Strežnik bo zagnan na `http://localhost:8080`. Videti bi morali:
```
Started McpServerApplication in X.XXX seconds
```

### Korak 2: Testirajte z neposrednim odjemalcem

V **NOVEM** terminalu, ko strežnik še teče, zaženite neposredni MCP odjemalec:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Videli boste izhod, kot je:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Korak 3: Testirajte z AI odjemalcem

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Videli boste, da AI samodejno uporablja orodja:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Korak 4: Zaprite MCP strežnik

Ko končate s testiranjem, ustavite AI odjemalca s pritiskom `Ctrl+C` v njegovem terminalu. MCP strežnik bo ostal zagnan, dokler ga ne ustavite sami.
Strežnik ustavite s pritiskom `Ctrl+C` v terminalu, kjer teče.

## Kako vse deluje skupaj

Tukaj je celoten potek, ko AI vprašate "Koliko je 5 + 3?":

1. **Vi** vprašate AI v naravnem jeziku
2. **AI** analizira vašo zahtevo in ugotovi, da želite seštevanje
3. **AI** kliče MCP strežnik: `add(5.0, 3.0)`
4. **Kalkulatorska storitev** izvede: `5.0 + 3.0 = 8.0`
5. **Kalkulatorska storitev** vrne: `"5.00 + 3.00 = 8.00"`
6. **AI** prejme rezultat in oblikuje naravni odgovor
7. **Vi** dobite: "Vsota 5 in 3 je 8"

## Naslednji koraki

Za več primerov glejte [Poglavje 04: Praktični primeri](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Omejitev odgovornosti**:
Ta dokument je bil preveden z uporabo AI prevajalske storitve [Co-op Translator](https://github.com/Azure/co-op-translator). Čeprav si prizadevamo za natančnost, vas prosimo, da upoštevate, da avtomatizirani prevodi lahko vsebujejo napake ali netočnosti. Izvirni dokument v njegovem izvirnem jeziku je treba obravnavati kot avtoritativni vir. Za kritične informacije je priporočljiv strokovni človeški prevod. Ne odgovarjamo za morebitna nesporazume ali napačne interpretacije, ki izhajajo iz uporabe tega prevoda.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->