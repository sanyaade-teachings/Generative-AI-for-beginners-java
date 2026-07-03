# MCP Calculator Tutorial za Početnike

## Sadržaj

- [Što ćete naučiti](#što-ćete-naučiti)
- [Preduvjeti](#preduvjeti)
- [Razumijevanje strukture projekta](#razumijevanje-strukture-projekta)
- [Objašnjenje osnovnih komponenti](#objašnjenje-osnovnih-komponenti)
  - [1. Glavna aplikacija](#1-glavna-aplikacija)
  - [2. Kalkulatorska usluga](#2-kalkulatorska-usluga)
  - [3. Direktni MCP klijent](#3-direktni-mcp-klijent)
  - [4. Klijent koji koristi AI](#4-klijent-koji-koristi-ai)
- [Pokretanje primjera](#pokretanje-primjera)
- [Kako sve zajedno funkcionira](#kako-sve-zajedno-funkcionira)
- [Sljedeći koraci](#sljedeći-koraci)

## Što ćete naučiti

Ovaj vodič objašnjava kako izgraditi kalkulatorsku uslugu koristeći Model Context Protocol (MCP). Naučit ćete:

- Kako kreirati uslugu koju AI može koristiti kao alat
- Kako uspostaviti direktnu komunikaciju s MCP uslugama
- Kako AI modeli mogu automatski birati koje alate koristiti
- Razliku između direktnih protokol poziva i interakcija uz pomoć AI

## Preduvjeti

Prije početka, osigurajte da imate:
- Instaliranu Javu 21 ili noviju verziju
- Maven za upravljanje ovisnostima
- Deployment modela Azure AI Foundry (dostavljen pomoću `azd up` — vidi [Poglavlje 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), prijavljeni s `az login` (autentifikacija bez ključa)
- Osnovno razumijevanje Jave i Spring Boota

## Razumijevanje strukture projekta

Projekt kalkulatora sadrži nekoliko važnih datoteka:

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

## Objašnjenje osnovnih komponenti

### 1. Glavna aplikacija

**Datoteka:** `McpServerApplication.java`

Ovo je ulazna točka naše kalkulatorske usluge. To je standardna Spring Boot aplikacija s jednom posebnom dodatkom:

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

**Što ovo radi:**
- Pokreće Spring Boot web server na portu 8080
- Stvara `ToolCallbackProvider` koji omogućuje naše kalkulatorske metode kao MCP alate
- Oznaka `@Bean` govori Springu da upravlja ovim komponentama koje drugi dijelovi mogu koristiti

### 2. Kalkulatorska usluga

**Datoteka:** `CalculatorService.java`

Ovdje se odvijaju sve matematičke radnje. Svaka metoda je označena s `@Tool` kako bi bila dostupna putem MCP-a:

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
    
    // Više operacija kalkulatora...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Ključne značajke:**

1. **`@Tool` oznaka**: Ovo govori MCP-u da ovu metodu mogu pozivati vanjski klijenti
2. **Jasni opisi**: Svaki alat ima opis koji pomaže AI modelima razumjeti kada ga koristiti
3. **Konzistentan format povratka**: Sve operacije vraćaju ljudski čitljive stringove poput "5.00 + 3.00 = 8.00"
4. **Rukovanje greškama**: Dijeljenje s nulom i korijen ne-negativnog broja vraćaju poruke o grešci

**Dostupne operacije:**
- `add(a, b)` - Zbraja dva broja
- `subtract(a, b)` - Oduzima drugi od prvog
- `multiply(a, b)` - Množi dva broja
- `divide(a, b)` - Dijeli prvi s drugim (s provjerom na nulu)
- `power(base, exponent)` - Potencira bazu na eksponent
- `squareRoot(number)` - Izračunava korijen (s provjerom na negativan broj)
- `modulus(a, b)` - Vraća ostatak dijeljenja
- `absolute(number)` - Vraća apsolutnu vrijednost
- `help()` - Vraća informacije o svim operacijama

### 3. Direktni MCP klijent

**Datoteka:** `SDKClient.java`

Ovaj klijent izravno komunicira s MCP serverom bez korištenja AI-ja. Ručno poziva specifične kalkulatorske funkcije:

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
        
        // Nabroj dostupne alate
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Pozovi specifične funkcije kalkulatora
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

**Što ovo radi:**
1. **Povezuje se** na kalkulatorski server na `http://localhost:8080` koristeći builder pattern
2. **Nabavlja popis** svih dostupnih alata (naših kalkulatorskih funkcija)
3. **Poziva** specifične funkcije s točnim parametrima
4. **Ispisuje** rezultate direktno

**Napomena:** Ovaj primjer koristi Spring AI verziju 1.1.0-SNAPSHOT, koja je uvela builder pattern za `WebFluxSseClientTransport`. Ako koristite stariju stabilnu verziju, možda ćete trebati koristiti direktni konstruktor.

**Kada koristiti:** Kad točno znate koju kalkulaciju želite napraviti i želite je pozvati programatski.

### 4. Klijent koji koristi AI

**Datoteka:** `LangChain4jClient.java`

Ovaj klijent koristi AI model (GPT-4o-mini) koji može automatski odlučiti koje kalkulatorske alate koristiti:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Postavite AI model (Azure AI Foundry, autentifikacija bez ključa putem Microsoft Entra ID)
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

        // Povežite se na naš MCP poslužitelj kalkulatora
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Prikazuje što AI radi
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Dajte AI pristup našim alatima kalkulatora
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Kreirajte AI bota koji može koristiti naš kalkulator
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Sada možemo zatražiti od AI-a da izvrši izračune na prirodnom jeziku
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Što ovo radi:**
1. **Stvara** AI model vezu koristeći autentifikaciju bez ključa (Microsoft Entra ID)
2. **Povezuje** AI s našim MCP kalkulatorskim serverom
3. **Daje** AI pristup svim našim kalkulatorskim alatima
4. **Omogućava** zahtjeve na prirodnom jeziku poput "Izračunaj zbroj od 24.5 i 17.3"

**AI automatski:**
- Razumije da želite zbrojiti brojeve
- Odabire alat `add`
- Poziva `add(24.5, 17.3)`
- Vraća rezultat u prirodnom odgovoru

## Pokretanje primjera

### Korak 1: Pokrenite kalkulatorski server

Prvo se prijavite i postavite svoj Azure AI Foundry endpoint (potrebno za AI klijent — autentifikacija bez ključa, bez API ključa):

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

Pokrenite server:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

Server će se pokrenuti na `http://localhost:8080`. Trebali biste vidjeti:
```
Started McpServerApplication in X.XXX seconds
```

### Korak 2: Testirajte s direktnim klijentom

U **NOVOM** terminalu dok je server još pokrenut, pokrenite direktni MCP klijent:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Vidjet ćete izlaz poput:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Korak 3: Testirajte s AI klijentom

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Vidjet ćete da AI automatski koristi alate:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Korak 4: Zatvorite MCP server

Kada završite s testiranjem, zaustavite AI klijent pritiskom na `Ctrl+C` u njegovom terminalu. MCP server će nastaviti raditi dok ga ne zaustavite.
Za zaustavljanje servera pritisnite `Ctrl+C` u terminalu gdje je pokrenut.

## Kako sve zajedno funkcionira

Evo potpuni tijek kada pitate AI "Koliko je 5 + 3?":

1. **Vi** pitate AI na prirodnom jeziku
2. **AI** analizira vaš zahtjev i shvati da želite zbrajanje
3. **AI** poziva MCP server: `add(5.0, 3.0)`
4. **Kalkulatorska usluga** izvršava: `5.0 + 3.0 = 8.0`
5. **Kalkulatorska usluga** vraća: `"5.00 + 3.00 = 8.00"`
6. **AI** prima rezultat i formulira prirodan odgovor
7. **Vi** dobivate: "Zbroj od 5 i 3 je 8"

## Sljedeći koraci

Za više primjera, vidi [Poglavlje 04: Praktični primjeri](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Napomena**:
Ovaj dokument je preveden korištenjem AI prevoditeljskog servisa [Co-op Translator](https://github.com/Azure/co-op-translator). Iako težimo točnosti, imajte na umu da automatski prijevodi mogu sadržavati greške ili netočnosti. Izvorni dokument na izvornom jeziku treba smatrati autoritativnim izvorom. Za važne informacije preporuča se profesionalni ljudski prijevod. Nismo odgovorni za bilo kakva nesporazumevanja ili pogrešne interpretacije koje proizlaze iz korištenja ovog prijevoda.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->