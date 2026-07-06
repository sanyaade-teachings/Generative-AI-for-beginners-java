# Návod na kalkulačku MCP pre začiatočníkov

## Obsah

- [Čo sa naučíte](#čo-sa-naučíte)
- [Predpoklady](#predpoklady)
- [Pochopenie štruktúry projektu](#pochopenie-štruktúry-projektu)
- [Vysvetlenie hlavných komponentov](#vysvetlenie-hlavných-komponentov)
  - [1. Hlavná aplikácia](#1-hlavná-aplikácia)
  - [2. Kalkulačná služba](#2-kalkulačná-služba)
  - [3. Priamy MCP klient](#3-priamy-mcp-klient)
  - [4. Klient s podporou AI](#4-klient-s-podporou-ai)
- [Spustenie príkladov](#spustenie-príkladov)
- [Ako to všetko spolu funguje](#ako-to-všetko-spolu-funguje)
- [Nasledujúce kroky](#nasledujúce-kroky)

## Čo sa naučíte

Tento návod vysvetľuje, ako vytvoriť kalkulačnú službu pomocou Model Context Protocol (MCP). Pochopíte:

- Ako vytvoriť službu, ktorú môže AI používať ako nástroj
- Ako nastaviť priamu komunikáciu so službami MCP
- Ako AI modely môžu automaticky vybrať, ktoré nástroje použiť
- Rozdiel medzi priamymi volaniami protokolu a interakciami s pomocou AI

## Predpoklady

Pred začatím sa uistite, že máte:
- Nainštalovanú Javu 21 alebo novšiu
- Maven na správu závislostí
- Nasadený model Azure AI Foundry (nasadte ho pomocou `azd up` — pozri [Kapitolu 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), prihlásený cez `az login` (overenie bez kľúča)
- Základné znalosti Javy a Spring Boot

## Pochopenie štruktúry projektu

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

## Vysvetlenie hlavných komponentov

### 1. Hlavná aplikácia

**Súbor:** `McpServerApplication.java`

Toto je vstupný bod našej kalkulačnej služby. Ide o štandardnú Spring Boot aplikáciu s jedným špeciálnym doplnkom:

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
- Spúšťa Spring Boot web server na porte 8080
- Vytvára `ToolCallbackProvider`, ktorý sprístupňuje naše kalkulačné metódy ako MCP nástroje
- Anotácia `@Bean` hovorí Springu, aby tento komponent spravoval a ostatné časti ho mohli používať

### 2. Kalkulačná služba

**Súbor:** `CalculatorService.java`

Tu prebieha všetka matematika. Každá metóda je označená `@Tool`, aby bola dostupná cez MCP:

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
    
    // Viac kalkulačných operácií...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Kľúčové vlastnosti:**

1. **Anotácia `@Tool`**: Označuje, že metóda môže byť volaná externými klientmi
2. **Jasné popisy**: Každý nástroj má popis, ktorý pomáha AI modelom pochopiť, kedy ho použiť
3. **Konzistentný formát návratu**: Všetky operácie vracajú ľahko čitateľné reťazce ako "5.00 + 3.00 = 8.00"
4. **Riešenie chýb**: Delenie nulou a záporné odmocniny vracajú chybové správy

**Dostupné operácie:**
- `add(a, b)` - Sčíta dve čísla
- `subtract(a, b)` - Odčíta druhé číslo od prvého
- `multiply(a, b)` - Vynásobí dve čísla
- `divide(a, b)` - Delí prvé číslom druhého (kontrola na nulu)
- `power(base, exponent)` - Zdvihne základ na exponent
- `squareRoot(number)` - Vypočíta druhú odmocninu (kontrola na záporné číslo)
- `modulus(a, b)` - Vráti zvyšok po delení
- `absolute(number)` - Vráti absolútnu hodnotu
- `help()` - Poskytne informácie o všetkých operáciách

### 3. Priamy MCP klient

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
1. **Pripája sa** k kalkulačnému serveru na `http://localhost:8080` pomocou builder patternu
2. **Vypíše** všetky dostupné nástroje (funkcie kalkulačky)
3. **Volá** konkrétne funkcie s presnými parametrami
4. **Vypisuje** výsledky priamo

**Poznámka:** Tento príklad používa Spring AI 1.1.0-SNAPSHOT závislosť, ktorá zaviedla builder pattern pre `WebFluxSseClientTransport`. Ak používate staršiu stabilnú verziu, možno budete musieť použiť priamy konštruktor.

**Kedy použiť:** Ak presne viete, aký výpočet chcete spraviť a chcete ho volať programovo.

### 4. Klient s podporou AI

**Súbor:** `LangChain4jClient.java`

Tento klient používa AI model (GPT-4o-mini), ktorý automaticky rozhoduje, ktoré kalkulačné nástroje použiť:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Nastavte AI model (Azure AI Foundry, bezkľúčová autentifikácia cez Microsoft Entra ID)
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
                .logRequests(true)  // Zobrazuje, čo AI robí
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Umožnite AI prístup k našim nástrojom kalkulačky
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Vytvorte AI bota, ktorý môže používať našu kalkulačku
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Teraz môžeme požiadať AI, aby vykonala výpočty v prirodzenom jazyku
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Čo to robí:**
1. **Vytvára** pripojenie k AI modelu pomocou overenia bez kľúča (Microsoft Entra ID)
2. **Prepája** AI s našim MCP kalkulačným serverom
3. **Umožňuje** AI pristupovať ku všetkým našim kalkulačným nástrojom
4. **Povoľuje** prirodzené jazykové požiadavky ako "Vypočítaj súčet 24.5 a 17.3"

**AI automaticky:**
- Chápe, že chcete sčítať čísla
- Vyberie nástroj `add`
- Zavolá `add(24.5, 17.3)`
- Vráti výsledok prirodzenou odpoveďou

## Spustenie príkladov

### Krok 1: Spustite kalkulačný server

Najskôr sa prihláste a nastavte svoj Azure AI Foundry endpoint (potrebné pre AI klienta – overenie bez kľúča, bez API kľúča):

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

### Krok 2: Otestujte s priamym klientom

V **NOVOM** termináli, pričom server beží, spustite priamy MCP klient:
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

### Krok 3: Otestujte s AI klientom

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

AI automaticky použije nástroje:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Krok 4: Ukončite MCP server

Keď skončíte s testovaním, môžete zastaviť AI klienta stlačením `Ctrl+C` v jeho termináli. MCP server pobehne ďalej, až kým ho nezastavíte.
Ak chcete zastaviť server, stlačte `Ctrl+C` v termináli, v ktorom beží.

## Ako to všetko spolu funguje

Tu je kompletný tok, keď sa AI spýtate "Čo je 5 + 3?":

1. **Vy** položíte otázku AI prirodzeným jazykom
2. **AI** analyzuje požiadavku a zistí, že chcete sčítať
3. **AI** zavolá MCP server: `add(5.0, 3.0)`
4. **Kalkulačná služba** vypočíta: `5.0 + 3.0 = 8.0`
5. **Kalkulačná služba** vráti: `"5.00 + 3.00 = 8.00"`
6. **AI** obdrží výsledok a formátuje prirodzenú odpoveď
7. **Vy** dostanete: "Súčet 5 a 3 je 8"

## Nasledujúce kroky

Pre viac príkladov si pozrite [Kapitolu 04: Praktické ukážky](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Vyhlásenie o zodpovednosti**:
Tento dokument bol preložený pomocou AI prekladateľskej služby [Co-op Translator](https://github.com/Azure/co-op-translator). Hoci sa snažíme o presnosť, vezmite prosím na vedomie, že automatické preklady môžu obsahovať chyby alebo nepresnosti. Pôvodný dokument v jeho natívnom jazyku by mal byť považovaný za autoritatívny zdroj. Pre kritické informácie sa odporúča profesionálny ľudský preklad. Nie sme zodpovední za žiadne nedorozumenia alebo nesprávne interpretácie vyplývajúce z použitia tohto prekladu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->