# MCP Calculator Handleiding voor Beginners

## Inhoudsopgave

- [Wat je zult leren](#wat-je-zult-leren)
- [Vereisten](#vereisten)
- [Projectstructuur begrijpen](#projectstructuur-begrijpen)
- [Uitleg van kerncomponenten](#uitleg-van-kerncomponenten)
  - [1. Hoofdtoepassing](#1-hoofdtoepassing)
  - [2. Calculator Service](#2-calculator-service)
  - [3. Directe MCP Client](#3-directe-mcp-client)
  - [4. AI-aangedreven Client](#4-ai-aangedreven-client)
- [De voorbeelden uitvoeren](#de-voorbeelden-uitvoeren)
- [Hoe het geheel samenwerkt](#hoe-het-geheel-samenwerkt)
- [Volgende stappen](#volgende-stappen)

## Wat je zult leren

Deze handleiding legt uit hoe je een calculatorservice bouwt met het Model Context Protocol (MCP). Je zult begrijpen:

- Hoe je een service maakt die AI kan gebruiken als een hulpmiddel
- Hoe je directe communicatie met MCP-services opzet
- Hoe AI-modellen automatisch kunnen kiezen welke hulpmiddelen te gebruiken
- Het verschil tussen directe protocoloproepen en AI-ondersteunde interacties

## Vereisten

Voordat je begint, zorg dat je het volgende hebt:
- Java 21 of hoger geïnstalleerd
- Maven voor afhankelijkheidsbeheer
- Een Azure AI Foundry model deployment (voorzie deze met `azd up` — zie [Hoofdstuk 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- De [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), ingelogd met `az login` (keyless auth)
- Basiskennis van Java en Spring Boot

## Projectstructuur begrijpen

Het calculatorproject bevat enkele belangrijke bestanden:

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

## Uitleg van kerncomponenten

### 1. Hoofdtoepassing

**Bestand:** `McpServerApplication.java`

Dit is het toegangspunt van onze calculatorservice. Het is een standaard Spring Boot-applicatie met één speciale toevoeging:

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

**Dit doet het:**
- Start een Spring Boot webserver op poort 8080
- Maakt een `ToolCallbackProvider` die onze calculatormethoden beschikbaar maakt als MCP-tools
- De `@Bean`-annotatie vertelt Spring dit te beheren als een component die door andere delen gebruikt kan worden

### 2. Calculator Service

**Bestand:** `CalculatorService.java`

Hier vindt alle wiskunde plaats. Elke methode is gemarkeerd met `@Tool` om deze beschikbaar te maken via MCP:

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
    
    // Meer rekenmachinebewerkingen...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Belangrijkste kenmerken:**

1. **`@Tool` Annotatie**: Dit vertelt MCP dat deze methode door externe clients kan worden aangeroepen
2. **Duidelijke omschrijvingen**: Elke tool heeft een beschrijving die AI-modellen helpt te begrijpen wanneer het te gebruiken
3. **Consistent terugkeerformaat**: Alle bewerkingen geven menselijk leesbare strings terug zoals "5.00 + 3.00 = 8.00"
4. **Foutafhandeling**: Delen door nul en negatieve kwadratenwortels geven foutmeldingen terug

**Beschikbare bewerkingen:**
- `add(a, b)` - Telt twee getallen op
- `subtract(a, b)` - Trekt tweede van eerste af
- `multiply(a, b)` - Vermenigvuldigt twee getallen
- `divide(a, b)` - Deelt eerste door tweede (controle op nul)
- `power(base, exponent)` - Verheft basis tot macht exponent
- `squareRoot(number)` - Berekent vierkantswortel (controle op negatief)
- `modulus(a, b)` - Geeft rest van deling terug
- `absolute(number)` - Geeft absolute waarde terug
- `help()` - Geeft informatie over alle bewerkingen

### 3. Directe MCP Client

**Bestand:** `SDKClient.java`

Deze client communiceert rechtstreeks met de MCP-server zonder AI te gebruiken. Het roept handmatig specifieke calculatorfuncties aan:

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
        
        // Beschikbare tools opsommen
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Specifieke rekenmachinefuncties aanroepen
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

**Dit doet het:**
1. **Verbindt** met de calculatorserver op `http://localhost:8080` via de builder pattern
2. **Toont een lijst** van alle beschikbare tools (onze calculatorfuncties)
3. **Roept** specifieke functies aan met exacte parameters
4. **Print** de resultaten onmiddellijk

**Opmerking:** Dit voorbeeld gebruikt de Spring AI 1.1.0-SNAPSHOT dependency, die een builder pattern introduceerde voor `WebFluxSseClientTransport`. Als je een oudere stabiele versie gebruikt, moet je mogelijk de directe constructor gebruiken.

**Wanneer te gebruiken:** Als je precies weet welke berekening je wilt uitvoeren en deze programmatisch wilt aanroepen.

### 4. AI-aangedreven Client

**Bestand:** `LangChain4jClient.java`

Deze client gebruikt een AI-model (GPT-4o-mini) dat automatisch kan beslissen welke calculatorhulpmiddelen te gebruiken:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Stel het AI-model in (Azure AI Foundry, keyless authenticatie via Microsoft Entra ID)
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

        // Verbind met onze rekenmachine MCP-server
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Toont wat de AI aan het doen is
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Geef de AI toegang tot onze rekenmachine-tools
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Maak een AI-bot die onze rekenmachine kan gebruiken
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Nu kunnen we de AI vragen om berekeningen in natuurlijke taal te doen
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Dit doet het:**
1. **Maakt** een AI-modelverbinding met je GitHub-token
2. **Verbindt** de AI met onze calculator MCP-server
3. **Geeft** de AI toegang tot al onze calculatorhulpmiddelen
4. **Maakt** natuurlijke taalverzoeken mogelijk zoals "Bereken de som van 24.5 en 17.3"

**De AI doet automatisch:**
- Begrijpt dat je getallen wilt optellen
- Kiest de `add` tool
- Roept `add(24.5, 17.3)` aan
- Geeft het resultaat terug in een natuurlijke respons

## De voorbeelden uitvoeren

### Stap 1: Start de Calculator Server

Log eerst in en stel je Azure AI Foundry endpoint in (nodig voor de AI-client — keyless auth, geen API-sleutel):

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

Start de server:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

De server start op `http://localhost:8080`. Je zou moeten zien:
```
Started McpServerApplication in X.XXX seconds
```

### Stap 2: Test met Directe Client

Open een **NIEUWE** terminal terwijl de Server nog draait, en voer de directe MCP-client uit:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Je ziet output zoals:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Stap 3: Test met AI Client

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Je ziet dat de AI automatisch tools gebruikt:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Stap 4: Sluit de MCP Server af

Als je klaar bent met testen, kun je de AI-client stoppen door `Ctrl+C` te drukken in de terminal. De MCP-server blijft draaien totdat je deze stopt.
Om de server te stoppen, druk je op `Ctrl+C` in de terminal waar deze draait.

## Hoe het geheel samenwerkt

Hier is de volledige stroom wanneer je de AI vraagt "Wat is 5 + 3?":

1. **Jij** vraagt de AI in natuurlijke taal
2. **AI** analyseert je verzoek en ziet dat je optelling wilt
3. **AI** roept de MCP-server aan: `add(5.0, 3.0)`
4. **Calculator Service** voert uit: `5.0 + 3.0 = 8.0`
5. **Calculator Service** retourneert: `"5.00 + 3.00 = 8.00"`
6. **AI** ontvangt het resultaat en formuleert een natuurlijke respons
7. **Jij** krijgt: "De som van 5 en 3 is 8"

## Volgende stappen

Voor meer voorbeelden, zie [Hoofdstuk 04: Praktische voorbeelden](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Disclaimer**:
Dit document is vertaald met behulp van de AI vertaaldienst [Co-op Translator](https://github.com/Azure/co-op-translator). Hoewel we streven naar nauwkeurigheid, dient u er rekening mee te houden dat geautomatiseerde vertalingen fouten of onnauwkeurigheden kunnen bevatten. Het originele document in de oorspronkelijke taal moet worden beschouwd als de gezaghebbende bron. Voor kritieke informatie wordt professionele menselijke vertaling aanbevolen. Wij zijn niet aansprakelijk voor eventuele misverstanden of verkeerde interpretaties die voortvloeien uit het gebruik van deze vertaling.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->