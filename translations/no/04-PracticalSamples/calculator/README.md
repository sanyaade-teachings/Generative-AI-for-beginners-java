# MCP-kalkulatortutorial for nybegynnere

## Innholdsfortegnelse

- [Hva du vil lære](#hva-du-vil-lære)
- [Forutsetninger](#forutsetninger)
- [Forstå prosjektstrukturen](#forstå-prosjektstrukturen)
- [Kjernekomponenter forklart](#kjernekomponenter-forklart)
  - [1. Hovedapplikasjon](#1-hovedapplikasjon)
  - [2. Kalkulatortjeneste](#2-kalkulatortjeneste)
  - [3. Direkt MCP-klient](#3-direkt-mcp-klient)
  - [4. AI-drevet klient](#4-ai-drevet-klient)
- [Kjøre eksemplene](#kjøre-eksemplene)
- [Hvordan det hele fungerer sammen](#hvordan-det-hele-fungerer-sammen)
- [Neste steg](#neste-steg)

## Hva du vil lære

Denne tutorialen forklarer hvordan du bygger en kalkulatortjeneste med Model Context Protocol (MCP). Du vil forstå:

- Hvordan lage en tjeneste som AI kan bruke som et verktøy
- Hvordan sette opp direkte kommunikasjon med MCP-tjenester
- Hvordan AI-modeller kan automatisk velge hvilke verktøy som skal brukes
- Forskjellen mellom direkte protokollanrop og AI-assisterte interaksjoner

## Forutsetninger

Før du starter, sørg for at du har:
- Java 21 eller nyere installert
- Maven for avhengighetsadministrasjon
- En Azure AI Foundry-modellimplementering (provisjoner den med `azd up` — se [Kapittel 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), logget inn med `az login` (nøkkelfri autentisering)
- Grunnleggende kunnskap om Java og Spring Boot

## Forstå prosjektstrukturen

Kalkulatorprosjektet har flere viktige filer:

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

## Kjernekomponenter forklart

### 1. Hovedapplikasjon

**Fil:** `McpServerApplication.java`

Dette er inngangspunktet for vår kalkulatortjeneste. Det er en standard Spring Boot-applikasjon med en spesiell tillegg:

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

**Dette gjør den:**
- Starter en Spring Boot webserver på port 8080
- Oppretter en `ToolCallbackProvider` som gjør kalkulatormetodene våre tilgjengelige som MCP-verktøy
- `@Bean`-annotasjonen forteller Spring å håndtere dette som en komponent som andre deler kan bruke

### 2. Kalkulatortjeneste

**Fil:** `CalculatorService.java`

Her skjer all matematikken. Hver metode er merket med `@Tool` for å gjøre den tilgjengelig via MCP:

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
    
    // Flere kalkulatoroperasjoner...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Nøkkelfunksjoner:**

1. **`@Tool`-Annotasjon**: Dette forteller MCP at denne metoden kan kalles av eksterne klienter
2. **Klar beskrivelse**: Hvert verktøy har en beskrivelse som hjelper AI-modeller å forstå når det skal brukes
3. **Konsistent returformat**: Alle operasjoner returnerer menneskelesbare strenger som "5.00 + 3.00 = 8.00"
4. **Feilhåndtering**: Divisjon med null og negative kvadratrøtter returnerer feilmeldinger

**Tilgjengelige operasjoner:**
- `add(a, b)` - Legger sammen to tall
- `subtract(a, b)` - Trekker det andre fra det første
- `multiply(a, b)` - Multipliserer to tall
- `divide(a, b)` - Dividerer første tall med andre (med null-sjekk)
- `power(base, exponent)` - Opphøyer grunntall til eksponenten
- `squareRoot(number)` - Kalkulerer kvadratroten (med negativ sjekk)
- `modulus(a, b)` - Returnerer resten etter divisjon
- `absolute(number)` - Returnerer absoluttverdien
- `help()` - Returnerer informasjon om alle operasjoner

### 3. Direkt MCP-klient

**Fil:** `SDKClient.java`

Denne klienten kommuniserer direkte med MCP-serveren uten å bruke AI. Den kaller manuelt spesifikke kalkulatorfunksjoner:

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
        
        // List tilgjengelige verktøy
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Kall spesifikke kalkulatorfunksjoner
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

**Dette gjør den:**
1. **Kobler til** kalkulatorserveren på `http://localhost:8080` ved å bruke builder-mønsteret
2. **Lister opp** alle tilgjengelige verktøy (våre kalkulatorfunksjoner)
3. **Kaller** spesifikke funksjoner med nøyaktige parametere
4. **Skriver ut** resultatene direkte

**Merk:** Dette eksemplet bruker Spring AI 1.1.0-SNAPSHOT-avhengighet, som introduserte builder-mønsteret for `WebFluxSseClientTransport`. Hvis du bruker en eldre stabil versjon, må du kanskje bruke den direkte konstruktøren i stedet.

**Når du bør bruke denne:** Når du vet nøyaktig hvilken kalkulasjon du vil utføre og ønsker å kalle den programmatisk.

### 4. AI-drevet klient

**Fil:** `LangChain4jClient.java`

Denne klienten bruker en AI-modell (GPT-4o-mini) som automatisk kan avgjøre hvilke kalkulatorverktøy som skal brukes:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Sett opp AI-modellen (Azure AI Foundry, nøkkelfri autentisering via Microsoft Entra ID)
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

        // Koble til vår kalkulator MCP-server
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Viser hva AI-en gjør
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Gi AI-en tilgang til våre kalkulatorverktøy
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Lag en AI-bot som kan bruke kalkulatoren vår
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Nå kan vi be AI-en om å gjøre beregninger på naturlig språk
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Dette gjør den:**
1. **Oppretter** en AI-modelltilkobling med GitHub-tokenet ditt
2. **Kobler** AI til vår kalkulator MCP-server
3. **Gir** AI tilgang til alle kalkulatorverktøyene våre
4. **Tillater** naturlige språkforespørsler som "Beregne summen av 24,5 og 17,3"

**AI-en gjør automatisk:**
- Forstår at du ønsker å legge sammen tall
- Velger `add`-verktøyet
- Kaller `add(24.5, 17.3)`
- Returnerer resultatet i et naturlig svar

## Kjøre eksemplene

### Steg 1: Start kalkulatorserveren

Først, logg inn og sett din Azure AI Foundry-endepunkt (nødvendig for AI-klienten — nøkkelfri autentisering, ingen API-nøkkel):

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

Start serveren:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

Serveren vil starte på `http://localhost:8080`. Du bør se:
```
Started McpServerApplication in X.XXX seconds
```

### Steg 2: Test med direkte klient

I et **NYTT** terminalvindu med serveren fortsatt i gang, kjør den direkte MCP-klienten:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Du vil se utdata som:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Steg 3: Test med AI-klienten

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Du vil se AI-en automatisk bruke verktøyene:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Steg 4: Lukk MCP-serveren

Når du er ferdig med testing, kan du stoppe AI-klienten ved å trykke `Ctrl+C` i terminalen. MCP-serveren vil fortsette å kjøre til du stopper den.
For å stoppe serveren, trykk `Ctrl+C` i terminalen der den kjører.

## Hvordan det hele fungerer sammen

Her er hele flyten når du spør AI: "Hva er 5 + 3?":

1. **Du** spør AI på naturlig språk
2. **AI** analyserer forespørselen og skjønner at du vil legge sammen
3. **AI** kaller MCP-serveren: `add(5.0, 3.0)`
4. **Kalkulatortjenesten** utfører: `5.0 + 3.0 = 8.0`
5. **Kalkulatortjenesten** returnerer: `"5.00 + 3.00 = 8.00"`
6. **AI** mottar resultatet og formaterer et naturlig svar
7. **Du** får: "Summen av 5 og 3 er 8"

## Neste steg

For flere eksempler, se [Kapittel 04: Praktiske eksempler](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Ansvarsfraskrivelse**:
Dette dokumentet er oversatt ved hjelp av AI-oversettelsestjenesten [Co-op Translator](https://github.com/Azure/co-op-translator). Selv om vi streber etter nøyaktighet, vær oppmerksom på at automatiske oversettelser kan inneholde feil eller unøyaktigheter. Det opprinnelige dokumentet på originalspråket skal betraktes som den autoritative kilden. For kritisk informasjon anbefales profesjonell menneskelig oversettelse. Vi er ikke ansvarlige for eventuelle misforståelser eller feiltolkninger som oppstår ved bruk av denne oversettelsen.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->