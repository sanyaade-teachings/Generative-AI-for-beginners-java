# MCP Kalkulačka Návod pre Začiatočníkov

## Obsah

- [Čo sa Naučíte](#čo-sa-naučíte)
- [Predpoklady](#predpoklady)
- [Pochopenie Štruktúry Projektu](#pochopenie-štruktúry-projektu)
- [Vysvetlenie Hlavných Komponentov](#vysvetlenie-hlavných-komponentov)
  - [1. Hlavná Aplikácia](#1-hlavná-aplikácia)
  - [2. Kalkulačná Služba](#2-kalkulačná-služba)
  - [3. Priamy MCP Klient](#3-priamy-mcp-klient)
  - [4. Klient Poháňaný AI](#4-klient-poháňaný-ai)
- [Spustenie Príkladov](#spustenie-príkladov)
- [Ako to Všetko Spolu Funguje](#ako-to-všetko-spolu-funguje)
- [Ďalšie Kroky](#ďalšie-kroky)

## Čo sa Naučíte

Tento návod vysvetľuje, ako vytvoriť kalkulačnú službu pomocou Model Context Protocol (MCP). Pochopíte:

- Ako vytvoriť službu, ktorú AI môže použiť ako nástroj
- Ako nastaviť priamu komunikáciu so službami MCP
- Ako môžu AI modely automaticky vybrať, ktoré nástroje použiť
- Rozdiel medzi priamymi protokolovými volaniami a interakciami asistovanými AI

## Predpoklady

Pred začatím sa uistite, že máte:
- Nainštalovanú Javu 21 alebo novšiu
- Maven pre správu závislostí
- Nasadený model Azure AI Foundry (zabezpečte ho pomocou `azd up` — pozri [Kapitolu 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) prihlásený cez `az login` (autentifikácia bez kľúča)
- Základné znalosti Javy a Spring Boot

## Pochopenie Štruktúry Projektu

Projekt kalkulačky obsahuje niekoľko dôležitých súborov:

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

## Vysvetlenie Hlavných Komponentov

### 1. Hlavná Aplikácia

**Súbor:** `McpServerApplication.java`

Toto je vstupný bod našej kalkulačnej služby. Je to štandardná Spring Boot aplikácia s jedným špeciálnym doplnkom:

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

**Čo to robí:**
- Spúšťa Spring Boot webový server na porte 8080
- Vytvára `ToolCallbackProvider`, ktorý sprístupňuje naše kalkulačné metódy ako MCP nástroje
- Anotácia `@Bean` hovorí Springu, aby to spravoval ako komponent, ktorý môžu používať iné časti

### 2. Kalkulačná Služba

**Súbor:** `CalculatorService.java`

Tu sa deje všetka matematika. Každá metóda je označená `@Tool`, aby bola dostupná cez MCP:

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
    
    // Viac operácií kalkulačky...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Kľúčové vlastnosti:**

1. **Anotácia `@Tool`**: Hovorí MCP, že túto metódu môže volať externý klient
2. **Jasné Popisy**: Každý nástroj má popis, ktorý pomáha AI modelom pochopiť, kedy ho použiť
3. **Konzistentný Formát Výstupu**: Všetky operácie vracajú ľahko čitateľné reťazce, napr. "5.00 + 3.00 = 8.00"
4. **Riešenie Chýb**: Delenie nulou a odmocnina zo záporného čísla vracajú chybové hlásenia

**Dostupné Operácie:**
- `add(a, b)` - Sčítava dve čísla
- `subtract(a, b)` - Odčíta druhé číslo od prvého
- `multiply(a, b)` - Násobí dve čísla
- `divide(a, b)` - Delí prvé číslom druhé (kontrola delenia nulou)
- `power(base, exponent)` - Mocní základ na exponent
- `squareRoot(number)` - Vypočíta druhú odmocninu (kontrola záporného čísla)
- `modulus(a, b)` - Vracia zvyšok po delení
- `absolute(number)` - Vracia absolútnu hodnotu
- `help()` - Vracia informácie o všetkých operáciách

### 3. Priamy MCP Klient

**Súbor:** `SDKClient.java`

Tento klient komunikuje priamo so serverom MCP bez použitia AI. Manuálne volá konkrétne kalkulačné funkcie:

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
        
        // Zoznam dostupných nástrojov
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Zavolať konkrétne funkcie kalkulačky
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

**Čo to robí:**
1. **Pripojí sa** k serveru kalkulačky na `http://localhost:8080` pomocou builder vzoru
2. **Zobrazí** zoznam všetkých dostupných nástrojov (naše kalkulačné funkcie)
3. **Volá** konkrétne funkcie s presnými parametrami
4. **Vypíše** výsledky priamo

**Poznámka:** Tento príklad používa Spring AI verziu 1.1.0-SNAPSHOT, ktorá zaviedla builder vzor pre `WebFluxSseClientTransport`. Ak používate staršiu stabilnú verziu, možno budete musieť použiť priamy konštruktor.

**Kedy použiť:** Keď presne viete, aký výpočet chcete vykonať a chcete ho volať programovo.

### 4. Klient Poháňaný AI

**Súbor:** `LangChain4jClient.java`

Tento klient používa AI model (GPT-4o-mini), ktorý dokáže automaticky rozhodnúť, ktoré kalkulačné nástroje použiť:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Nastavte AI model (Azure AI Foundry, autentifikácia bez kľúča cez Microsoft Entra ID)
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

        // Pripojte sa k nášmu serveru kalkulačky MCP
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Zobrazuje, čo AI práve robí
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Udelte AI prístup k našim kalkulačkovým nástrojom
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Vytvorte AI bota, ktorý dokáže používať našu kalkulačku
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Teraz môžeme požiadať AI o vykonanie výpočtov v prirodzenom jazyku
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Čo to robí:**
1. **Vytvorí** pripojenie k AI modelu pomocou vášho GitHub tokenu
2. **Pripojí** AI ku nášmu kalkulačnému MCP serveru
3. **Umožní** AI prístup ku všetkým našim kalkulačným nástrojom
4. **Povoľuje** prirodzené jazykové požiadavky, ako napríklad "Vypočítaj súčet 24.5 a 17.3"

**AI automaticky:**
- Pochopí, že chcete sčítať čísla
- Vyberie nástroj `add`
- Zavolá `add(24.5, 17.3)`
- Vráti výsledok v prirodzenej odpovedi

## Spustenie Príkladov

### Krok 1: Spustite Kalkulačný Server

Najskôr sa prihláste a nastavte svoj Azure AI Foundry endpoint (potrebné pre AI klienta — autentifikácia bez kľúča, bez API kľúča):

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

Spustite server:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

Server sa spustí na `http://localhost:8080`. Mali by ste vidieť:
```
Started McpServerApplication in X.XXX seconds
```

### Krok 2: Otestujte Priamy Klient

V **NOVOM** termináli so spusteným serverom spustite priamy MCP klient:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Uvidíte výstup ako:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Krok 3: Otestujte AI Klienta

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Uvidíte, ako AI automaticky používa nástroje:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Krok 4: Zatvorte MCP Server

Keď skončíte s testovaním, môžete AI klienta zastaviť stlačením `Ctrl+C` v jeho termináli. MCP server bude bežať ďalej, kým ho nezastavíte.
Pre zastavenie servera stlačte `Ctrl+C` v termináli, kde beží.

## Ako to Všetko Spolu Funguje

Tu je kompletný tok, keď sa AI opýtate "Koľko je 5 + 3?":

1. **Vy** položíte otázku AI v prirodzenom jazyku
2. **AI** analyzuje požiadavku a zistí, že chcete sčítanie
3. **AI** zavolá MCP server: `add(5.0, 3.0)`
4. **Kalkulačná Služba** vykoná: `5.0 + 3.0 = 8.0`
5. **Kalkulačná Služba** vráti: `"5.00 + 3.00 = 8.00"`
6. **AI** prijme výsledok a vytvorí prirodzenú odpoveď
7. **Vy** dostanete: "Súčet 5 a 3 je 8"

## Ďalšie Kroky

Pre viac príkladov pozrite [Kapitolu 04: Praktické ukážky](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Vyhlásenie o zodpovednosti**:
Tento dokument bol preložený pomocou AI prekladateľskej služby [Co-op Translator](https://github.com/Azure/co-op-translator). Hoci sa snažíme o presnosť, vezmite prosím na vedomie, že automatické preklady môžu obsahovať chyby alebo nepresnosti. Pôvodný dokument v jeho natívnom jazyku by mal byť považovaný za autoritatívny zdroj. Pre kritické informácie sa odporúča profesionálny ľudský preklad. Nie sme zodpovední za žiadne nedorozumenia alebo nesprávne interpretácie vyplývajúce z použitia tohto prekladu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->