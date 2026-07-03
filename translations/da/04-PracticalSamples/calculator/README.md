# MCP Calculator Tutorial for Begyndere

## Indholdsfortegnelse

- [Hvad Du Vil Lære](#hvad-du-vil-lære)
- [Forudsætninger](#forudsætninger)
- [Forståelse af Projektstrukturen](#forståelse-af-projektstrukturen)
- [Kernekomponenter Forklaret](#kernekomponenter-forklaret)
  - [1. Hovedapplikation](#1-hovedapplikation)
  - [2. Calculator Service](#2-calculator-service)
  - [3. Direkte MCP Client](#3-direkte-mcp-client)
  - [4. AI-drevet Client](#4-ai-drevet-client)
- [Køre Eksemplerne](#køre-eksemplerne)
- [Hvordan Det Hele Fungerer Sammen](#hvordan-det-hele-fungerer-sammen)
- [Næste Skridt](#næste-skridt)

## Hvad Du Vil Lære

Denne tutorial forklarer, hvordan man bygger en calculator service ved brug af Model Context Protocol (MCP). Du vil forstå:

- Hvordan man opretter en service, som AI kan bruge som et værktøj
- Hvordan man opstiller direkte kommunikation med MCP services
- Hvordan AI-modeller automatisk kan vælge hvilke værktøjer der skal bruges
- Forskellen mellem direkte protokolkald og AI-assisterede interaktioner

## Forudsætninger

Før du starter, sørg for at have:
- Java 21 eller nyere installeret
- Maven til afhængighedsstyring
- En Azure AI Foundry model deployment (opret den med `azd up` — se [Kapitel 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), logget ind med `az login` (nøglefri autentificering)
- Grundlæggende forståelse af Java og Spring Boot

## Forståelse af Projektstrukturen

Calculator-projektet har flere vigtige filer:

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

## Kernekomponenter Forklaret

### 1. Hovedapplikation

**Fil:** `McpServerApplication.java`

Dette er indgangspunktet for vores calculator-service. Det er en standard Spring Boot-applikation med en særlig tilføjelse:

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

**Dette gør:**
- Starter en Spring Boot webserver på port 8080
- Opretter en `ToolCallbackProvider`, der gør vores calculator metoder tilgængelige som MCP-værktøjer
- `@Bean`-annotationen fortæller Spring at administrere dette som en komponent, som andre dele kan bruge

### 2. Calculator Service

**Fil:** `CalculatorService.java`

Her foregår al matematikken. Hver metode er markeret med `@Tool` for at gøre den tilgængelig gennem MCP:

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
    
    // Flere lommeregneroperationer...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Nøglefunktioner:**

1. **`@Tool` Annotation**: Dette fortæller MCP, at denne metode kan kaldes af eksterne klienter
2. **Klar Beskrivelse**: Hvert værktøj har en beskrivelse, der hjælper AI-modeller med at forstå, hvornår det skal bruges
3. **Konsistent Returformat**: Alle operationer returnerer menneskeligt læsbare strenge som "5.00 + 3.00 = 8.00"
4. **Fejlhåndtering**: Division med nul og negative kvadratrødder returnerer fejlbeskeder

**Tilgængelige Operationer:**
- `add(a, b)` - Lægger to tal sammen
- `subtract(a, b)` - Trækker det andet tal fra det første
- `multiply(a, b)` - Ganger to tal
- `divide(a, b)` - Dividerer det første tal med det andet (med nul-tjek)
- `power(base, exponent)` - Oprører basen til eksponenten
- `squareRoot(number)` - Beregner kvadratroden (med negativt tjek)
- `modulus(a, b)` - Returnerer resten ved division
- `absolute(number)` - Returnerer absolutværdien
- `help()` - Returnerer information om alle operationer

### 3. Direkte MCP Client

**Fil:** `SDKClient.java`

Denne klient kommunikerer direkte med MCP serveren uden at bruge AI. Den kalder manuelt specifikke calculator-funktioner:

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
        
        // List tilgængelige værktøjer
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Kald specifikke lommeregnerfunktioner
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

**Dette gør:**
1. **Forbinder** til calculator-serveren på `http://localhost:8080` ved brug af builder-pattern
2. **Lister** alle tilgængelige værktøjer (vores calculator-funktioner)
3. **Kalder** specifikke funktioner med præcise parametre
4. **Printer** resultaterne direkte

**Bemærk:** Dette eksempel bruger Spring AI 1.1.0-SNAPSHOT afhængigheden, som introducerede builder pattern for `WebFluxSseClientTransport`. Hvis du bruger en ældre stabil version, skal du muligvis bruge konstruktøren direkte i stedet.

**Hvornår du bruger dette:** Når du præcist ved, hvilken beregning du vil udføre, og du ønsker at kalde den programmatisk.

### 4. AI-drevet Client

**Fil:** `LangChain4jClient.java`

Denne klient bruger en AI-model (GPT-4o-mini), som automatisk kan beslutte, hvilke calculator-værktøjer der skal bruges:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Opsæt AI-modellen (Azure AI Foundry, nøglefri godkendelse via Microsoft Entra ID)
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

        // Forbind til vores calculator MCP-server
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Viser hvad AI'en laver
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Giv AI adgang til vores calculator-værktøjer
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Opret en AI-bot, der kan bruge vores calculator
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Nu kan vi bede AI'en om at udføre beregninger på naturligt sprog
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Dette gør:**
1. **Opretter** en AI-modelforbindelse med dit GitHub-token
2. **Forbinder** AI med vores calculator MCP-server
3. **Giver** AI adgang til alle vores calculator-værktøjer
4. **Tillader** naturlige sprog-anmodninger som "Beregn summen af 24.5 og 17.3"

**AI'en gør automatisk:**
- Forstår, at du ønsker at lægge tal sammen
- Vælger `add`-værktøjet
- Kalder `add(24.5, 17.3)`
- Returnerer resultatet i et naturligt svar

## Køre Eksemplerne

### Trin 1: Start Calculator Serveren

Først skal du logge ind og sætte din Azure AI Foundry endpoint (nødvendigt for AI-client — nøglefri autentificering, ingen API-nøgle):

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

Serveren starter på `http://localhost:8080`. Du bør se:
```
Started McpServerApplication in X.XXX seconds
```

### Trin 2: Test med Direkte Client

I et **NYT** terminalvindue med serveren stadig kørende, kør den direkte MCP client:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Du vil se output som:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Trin 3: Test med AI Client

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Du vil se AI'en automatisk bruge værktøjer:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Trin 4: Luk MCP Serveren

Når du er færdig med at teste, kan du stoppe AI clienten ved at trykke `Ctrl+C` i dens terminal. MCP serveren fortsætter med at køre, indtil du stopper den.
For at stoppe serveren, tryk `Ctrl+C` i terminalen, hvor den kører.

## Hvordan Det Hele Fungerer Sammen

Her er det komplette flow, når du spørger AI'en "Hvad er 5 + 3?":

1. **Du** stiller AI'en et spørgsmål på naturligt sprog
2. **AI** analyserer din anmodning og forstår, at du ønsker addition
3. **AI** kalder MCP-serveren: `add(5.0, 3.0)`
4. **Calculator Service** udfører: `5.0 + 3.0 = 8.0`
5. **Calculator Service** returnerer: `"5.00 + 3.00 = 8.00"`
6. **AI** modtager resultatet og formaterer et naturligt svar
7. **Du** får: "Summen af 5 og 3 er 8"

## Næste Skridt

For flere eksempler, se [Kapitel 04: Praktiske eksempler](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Ansvarsfraskrivelse**:
Dette dokument er blevet oversat ved hjælp af AI-oversættelsestjenesten [Co-op Translator](https://github.com/Azure/co-op-translator). Selvom vi bestræber os på nøjagtighed, skal du være opmærksom på, at automatiserede oversættelser kan indeholde fejl eller unøjagtigheder. Det originale dokument på dets oprindelige sprog bør betragtes som den autoritative kilde. For kritisk information anbefales professionel menneskelig oversættelse. Vi påtager os intet ansvar for misforståelser eller fejltolkninger, der opstår som følge af brugen af denne oversættelse.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->