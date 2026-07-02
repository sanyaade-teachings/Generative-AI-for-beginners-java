# MCP Kalkulator Vodič za Početnike

## Sadržaj

- [Što ćete naučiti](#što-ćete-naučiti)
- [Preduvjeti](#preduvjeti)
- [Razumijevanje strukture projekta](#razumijevanje-strukture-projekta)
- [Objašnjenje glavnih komponenti](#objašnjenje-glavnih-komponenti)
  - [1. Glavna aplikacija](#1-glavna-aplikacija)
  - [2. Kalkulatorska usluga](#2-kalkulatorska-usluga)
  - [3. Direktni MCP klijent](#3-direktni-mcp-klijent)
  - [4. AI-pokretan klijent](#4-ai-pokretan-klijent)
- [Pokretanje primjera](#pokretanje-primjera)
- [Kako sve to zajedno funkcionira](#kako-sve-to-zajedno-funkcionira)
- [Sljedeći koraci](#sljedeći-koraci)

## Što ćete naučiti

Ovaj vodič objašnjava kako izgraditi kalkulatorsku uslugu koristeći Model Context Protocol (MCP). Naučit ćete:

- Kako stvoriti uslugu koju AI može koristiti kao alat
- Kako postaviti izravnu komunikaciju s MCP uslugama
- Kako AI modeli mogu automatski odabrati koje alate koristiti
- Razliku između izravnih protokolskih poziva i AI-pomoću interakcija

## Preduvjeti

Prije početka, provjerite imate li:
- Instaliran Java 21 ili noviju verziju
- Maven za upravljanje ovisnostima
- Azure AI Foundry model implementiran (provisionirajte ga s `azd up` — pogledajte [Poglavlje 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), prijavljen s `az login` (autentikacija bez ključa)
- Osnovno razumijevanje Java i Spring Boot

## Razumijevanje strukture projekta

Projekt kalkulatora ima nekoliko važnih datoteka:

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

## Objašnjenje glavnih komponenti

### 1. Glavna aplikacija

**Datoteka:** `McpServerApplication.java`

Ovo je ulazna točka naše kalkulatorske usluge. Standardna je Spring Boot aplikacija s jednim posebim dodatkom:

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

**Ovo radi:**
- Pokreće Spring Boot web poslužitelj na portu 8080
- Stvara `ToolCallbackProvider` koji čini naše metode kalkulatora dostupnima kao MCP alate
- `@Bean` anotacija govori Springu da ovo upravlja kao komponentom koju drugi dijelovi mogu koristiti

### 2. Kalkulatorska usluga

**Datoteka:** `CalculatorService.java`

Ovdje se odvijaju svi matematički izračuni. Svaka metoda označena je s `@Tool` kako bi bila dostupna putem MCP-a:

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
    
    // Još operacija kalkulatora...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Ključne značajke:**

1. **`@Tool` anotacija**: Govori MCP-u da ova metoda može biti pozvana od strane vanjskih klijenata
2. **Jasni opisi**: Svaki alat ima opis koji pomaže AI modelima da shvate kada ga koristiti
3. **Konzistentan format rezultata**: Sve operacije vraćaju razumljive stringove poput "5.00 + 3.00 = 8.00"
4. **Rukovanje pogreškama**: Dijeljenje s nulom i negativni korijen vraćaju poruke o pogrešci

**Dostupne operacije:**
- `add(a, b)` - Zbraja dva broja
- `subtract(a, b)` - Oduzima drugi od prvog
- `multiply(a, b)` - Množi dva broja
- `divide(a, b)` - Dijeli prvi s drugim (s provjerom na nulu)
- `power(base, exponent)` - Podigne bazu na potenciju
- `squareRoot(number)` - Računa kvadratni korijen (s provjerom za negativne vrijednosti)
- `modulus(a, b)` - Vraća ostatak dijeljenja
- `absolute(number)` - Vraća apsolutnu vrijednost
- `help()` - Vraća informacije o svim operacijama

### 3. Direktni MCP klijent

**Datoteka:** `SDKClient.java`

Ovaj klijent komunicira izravno s MCP poslužiteljem bez upotrebe AI. Ručno poziva određene funkcije kalkulatora:

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
        
        // Pozovi određene funkcije kalkulatora
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

**Ovo radi:**
1. **Povezuje se** s kalkulatorskim poslužiteljem na `http://localhost:8080` koristeći builder pattern
2. **Popisuje** sve dostupne alate (naše funkcije kalkulatora)
3. **Poziva** specifične funkcije s točnim parametrima
4. **Ispisuje** rezultate izravno

**Napomena:** Ovaj primjer koristi Spring AI 1.1.0-SNAPSHOT ovisnost koja je uvela builder pattern za `WebFluxSseClientTransport`. Ako koristite stariju stabilnu verziju, možda ćete trebati koristiti izravan konstruktor.

**Kada koristiti ovo:** Kada točno znate koju izračun želite i želite je pozvati programatski.

### 4. AI-pokretan klijent

**Datoteka:** `LangChain4jClient.java`

Ovaj klijent koristi AI model (GPT-4o-mini) koji može automatski odlučiti koje kalkulatorske alate koristiti:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Postavite AI model (Azure AI Foundry, autentifikacija bez ključa preko Microsoft Entra ID)
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

        // Dajte AI pristup našim kalkulator alatima
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Kreirajte AI bot koji može koristiti naš kalkulator
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Sada možemo tražiti od AI-a da obavlja izračune na prirodnom jeziku
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Ovo radi:**
1. **Stvara** AI model koristeći vaš GitHub token
2. **Povezuje** AI s našim kalkulatorskim MCP poslužiteljem
3. **Daje** AI pristup svim našim kalkulatorskim alatima
4. **Omogućava** prirodni jezik zahtjeva poput "Izračunaj zbroj 24.5 i 17.3"

**AI automatski:**
- Razumije da želite zbrojiti brojeve
- Odabire `add` alat
- Poziva `add(24.5, 17.3)`
- Vraća rezultat u prirodnom odgovoru

## Pokretanje primjera

### Korak 1: Pokrenite kalkulatorski poslužitelj

Prvo se prijavite i postavite svoj Azure AI Foundry endpoint (potrebno za AI klijenta — autentikacija bez ključa, bez API ključa):

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

Pokrenite poslužitelj:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

Poslužitelj će se pokrenuti na `http://localhost:8080`. Trebali biste vidjeti:
```
Started McpServerApplication in X.XXX seconds
```

### Korak 2: Test s direktnim klijentom

U **NOVOM** terminalu, dok je poslužitelj još pokrenut, pokrenite direktni MCP klijent:
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

### Korak 3: Test s AI klijentom

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Vidjet ćete da AI automatski koristi alate:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Korak 4: Zatvorite MCP poslužitelj

Kad završite s testiranjem, možete zaustaviti AI klijenta pritiskom `Ctrl+C` u njegovom terminalu. MCP poslužitelj će i dalje raditi dok ga ne zaustavite.
Za zaustavljanje poslužitelja pritisnite `Ctrl+C` u terminalu u kojem je pokrenut.

## Kako sve to zajedno funkcionira

Evo cijelog tijeka kad pitate AI "Koliko je 5 + 3?":

1. **Vi** postavite pitanje AI-ju na prirodnom jeziku
2. **AI** analizira vaš zahtjev i shvati da želite zbrajanje
3. **AI** poziva MCP poslužitelj: `add(5.0, 3.0)`
4. **Kalkulatorska usluga** izvršava: `5.0 + 3.0 = 8.0`
5. **Kalkulatorska usluga** vraća: `"5.00 + 3.00 = 8.00"`
6. **AI** prima rezultat i formatira prirodan odgovor
7. **Vi** dobivate: "Zbroj 5 i 3 je 8"

## Sljedeći koraci

Za više primjera, pogledajte [Poglavlje 04: Praktični primjeri](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Napomena**:
Ovaj dokument je preveden korištenjem AI prevoditeljskog servisa [Co-op Translator](https://github.com/Azure/co-op-translator). Iako težimo točnosti, imajte na umu da automatski prijevodi mogu sadržavati greške ili netočnosti. Izvorni dokument na izvornom jeziku treba smatrati autoritativnim izvorom. Za važne informacije preporuča se profesionalni ljudski prijevod. Nismo odgovorni za bilo kakva nesporazumevanja ili pogrešne interpretacije koje proizlaze iz korištenja ovog prijevoda.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->