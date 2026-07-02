# MCP Calculator Guide för nybörjare

## Innehållsförteckning

- [Vad du kommer att lära dig](#vad-du-kommer-att-lära-dig)
- [Förkunskaper](#förkunskaper)
- [Förstå projektstrukturen](#förstå-projektstrukturen)
- [Kärnkomponenter förklarade](#kärnkomponenter-förklarade)
  - [1. Huvudapplikation](#1-huvudapplikation)
  - [2. Kalkylatortjänst](#2-kalkylatortjänst)
  - [3. Direkt MCP-klient](#3-direkt-mcp-klient)
  - [4. AI-driven klient](#4-ai-driven-klient)
- [Köra exemplen](#köra-exemplen)
- [Hur allt fungerar tillsammans](#hur-allt-fungerar-tillsammans)
- [Nästa steg](#nästa-steg)

## Vad du kommer att lära dig

Denna handledning förklarar hur man bygger en kalkylatortjänst med hjälp av Model Context Protocol (MCP). Du kommer att förstå:

- Hur man skapar en tjänst som AI kan använda som ett verktyg
- Hur man konfigurerar direkt kommunikation med MCP-tjänster
- Hur AI-modeller automatiskt kan välja vilka verktyg som ska användas
- Skillnaden mellan direkta protokollsamtal och AI-assisterade interaktioner

## Förkunskaper

Innan du börjar, se till att du har:
- Java 21 eller högre installerat
- Maven för beroendehantering
- En Azure AI Foundry-modellutplacering (provisionera den med `azd up` — se [Kapitel 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), inloggad med `az login` (nyckelfri autentisering)
- Grundläggande kunskaper i Java och Spring Boot

## Förstå projektstrukturen

Kalkylatorprojektet har flera viktiga filer:

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

## Kärnkomponenter förklarade

### 1. Huvudapplikation

**Fil:** `McpServerApplication.java`

Detta är startpunkten för vår kalkylatortjänst. Det är en standard Spring Boot-applikation med en speciell tillägg:

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

**Detta gör den:**
- Startar en Spring Boot-webbserver på port 8080
- Skapar en `ToolCallbackProvider` som gör våra kalkylatormetoder tillgängliga som MCP-verktyg
- `@Bean`-annoteringen instruerar Spring att hantera detta som en komponent som andra delar kan använda

### 2. Kalkylatortjänst

**Fil:** `CalculatorService.java`

Här sker all matematik. Varje metod är märkta med `@Tool` för att göra den tillgänglig via MCP:

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
    
    // Fler kalkylatoroperationer...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Huvudfunktioner:**

1. **`@Tool`-annotering**: Anger för MCP att denna metod kan anropas av externa klienter
2. **Tydliga beskrivningar**: Varje verktyg har en beskrivning som hjälper AI-modeller att förstå när det ska användas
3. **Enhetligt returformat**: Alla operationer returnerar mänskligt läsbara strängar som "5.00 + 3.00 = 8.00"
4. **Felfunktioner**: Division med noll och negativa kvadratrötter genererar felmeddelanden

**Tillgängliga operationer:**
- `add(a, b)` - Lägger till två tal
- `subtract(a, b)` - Subtraherar andra från första
- `multiply(a, b)` - Multiplicerar två tal
- `divide(a, b)` - Dividerar första med andra (med noll-kontroll)
- `power(base, exponent)` - Upphöjer basen till exponent
- `squareRoot(number)` - Beräknar kvadratroten (med negativ kontroll)
- `modulus(a, b)` - Returnerar resten vid division
- `absolute(number)` - Returnerar absolutvärdet
- `help()` - Returnerar information om alla operationer

### 3. Direkt MCP-klient

**Fil:** `SDKClient.java`

Denna klient kommunicerar direkt med MCP-servern utan AI. Den anropar manuellt specifika kalkylatorfunktioner:

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
        
        // Lista tillgängliga verktyg
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Anropa specifika kalkylatorfunktioner
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

**Detta gör den:**
1. **Ansluter** till kalkylatorservern på `http://localhost:8080` med builder-mönstret
2. **Lister** alla tillgängliga verktyg (våra kalkylatorfunktioner)
3. **Anropar** specifika funktioner med exakta parametrar
4. **Skriver ut** resultaten direkt

**Notera:** Detta exempel använder Spring AI 1.1.0-SNAPSHOT-beroendet, som introducerade builder-mönstret för `WebFluxSseClientTransport`. Om du använder en äldre stabil version kan du behöva använda direktkonstruktorn istället.

**När du ska använda detta:** När du vet exakt vilken beräkning du vill göra och vill anropa den programmässigt.

### 4. AI-driven klient

**Fil:** `LangChain4jClient.java`

Denna klient använder en AI-modell (GPT-4o-mini) som automatiskt kan avgöra vilka kalkylatorverktyg som ska användas:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Ställ in AI-modellen (Azure AI Foundry, nyckelfri autentisering via Microsoft Entra ID)
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

        // Anslut till vår kalkylatormcp-server
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Visar vad AI:n gör
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Ge AI:n tillgång till våra kalkylatorverktyg
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Skapa en AI-bot som kan använda vår kalkylator
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Nu kan vi be AI:n att göra beräkningar på naturligt språk
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Detta gör den:**
1. **Skapar** en AI-modulanslutning med din GitHub-token
2. **Ansluter** AI:n till vår kalkylator MCP-server
3. **Ger** AI:n åtkomst till alla våra kalkylatorverktyg
4. **Tillåter** naturliga språkförfrågningar som "Beräkna summan av 24.5 och 17.3"

**AI:n automatiskt:**
- Förstår att du vill addera tal
- Väljer verktyget `add`
- Anropar `add(24.5, 17.3)`
- Returnerar resultatet i ett naturligt svar

## Köra exemplen

### Steg 1: Starta kalkylatorservern

Logga först in och sätt din Azure AI Foundry-endpoint (nödvändigt för AI-klienten — nyckelfri autentisering, ingen API-nyckel):

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

Starta servern:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

Servern startar på `http://localhost:8080`. Du bör se:
```
Started McpServerApplication in X.XXX seconds
```

### Steg 2: Testa med Direkt Klient

I ett **NYTT** terminalfönster medan servern fortfarande körs, kör den direkta MCP-klienten:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Du kommer att se utdata som:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Steg 3: Testa med AI-klient

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Du kommer att se att AI:n automatiskt använder verktygen:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Steg 4: Stäng MCP-servern

När du är klar med testningen kan du stoppa AI-klienten genom att trycka `Ctrl+C` i dess terminal. MCP-servern fortsätter att köras tills du stoppar den.
För att stoppa servern, tryck `Ctrl+C` i terminalen där den körs.

## Hur allt fungerar tillsammans

Här är hela flödet när du frågar AI:n "Vad är 5 + 3?":

1. **Du** frågar AI:n på naturligt språk
2. **AI** analyserar din förfrågan och inser att du vill addera
3. **AI** anropar MCP-servern: `add(5.0, 3.0)`
4. **Kalkylatortjänsten** utför: `5.0 + 3.0 = 8.0`
5. **Kalkylatortjänsten** returnerar: `"5.00 + 3.00 = 8.00"`
6. **AI** tar emot resultatet och formulerar ett naturligt svar
7. **Du** får: "Summan av 5 och 3 är 8"

## Nästa steg

För fler exempel, se [Kapitel 04: Praktiska exempel](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Ansvarsfriskrivning**:
Detta dokument har översatts med hjälp av AI-översättningstjänsten [Co-op Translator](https://github.com/Azure/co-op-translator). Även om vi strävar efter noggrannhet, var vänlig notera att automatiska översättningar kan innehålla fel eller brister. Det ursprungliga dokumentet på dess modersmål bör betraktas som den auktoritativa källan. För kritisk information rekommenderas professionell mänsklig översättning. Vi ansvarar inte för några missförstånd eller feltolkningar som uppstår till följd av användningen av denna översättning.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->