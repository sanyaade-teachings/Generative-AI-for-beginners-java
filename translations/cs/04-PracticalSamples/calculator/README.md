# MCP Kalkulačka - tutoriál pro začátečníky

## Obsah

- [Co se naučíte](#co-se-naučíte)
- [Předpoklady](#předpoklady)
- [Pochopení struktury projektu](#pochopení-struktury-projektu)
- [Vysvětlení hlavních komponent](#vysvětlení-hlavních-komponent)
  - [1. Hlavní aplikace](#1-hlavní-aplikace)
  - [2. Kalkulační služba](#2-kalkulační-služba)
  - [3. Přímý MCP klient](#3-přímý-mcp-klient)
  - [4. Klient využívající AI](#4-klient-využívající-ai)
- [Spuštění příkladů](#spuštění-příkladů)
- [Jak to spolu funguje](#jak-to-spolu-funguje)
- [Další kroky](#další-kroky)

## Co se naučíte

V tomto tutoriálu se naučíte, jak vytvořit kalkulační službu pomocí Model Context Protocol (MCP). Pochopíte:

- Jak vytvořit službu, kterou může AI používat jako nástroj
- Jak nastavit přímou komunikaci se službami MCP
- Jak si AI modely automaticky vyberou, které nástroje použít
- Rozdíl mezi přímými voláními protokolu a interakcemi s asistencí AI

## Předpoklady

Než začnete, ujistěte se, že máte:
- Nainstalovanou Javu 21 nebo novější
- Maven pro správu závislostí
- Nasazený model Azure AI Foundry (zprovozněte příkazem `azd up` — viz [Kapitola 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), přihlášený příkazem `az login` (autentizace bez klíče)
- Základní znalost Java a Spring Boot

## Pochopení struktury projektu

Projekt kalkulačky obsahuje několik důležitých souborů:

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

## Vysvětlení hlavních komponent

### 1. Hlavní aplikace

**Soubor:** `McpServerApplication.java`

Toto je vstupní bod naší kalkulační služby. Je to standardní aplikace Spring Boot s jedním speciálním doplňkem:

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

**Co to dělá:**
- Spustí webový server Spring Boot na portu 8080
- Vytvoří `ToolCallbackProvider`, který zpřístupní metody kalkulačky jako nástroje MCP
- Anotace `@Bean` říká Springu, aby toto spravoval jako komponentu, kterou mohou využívat jiné části

### 2. Kalkulační služba

**Soubor:** `CalculatorService.java`

Tady se děje veškerá matematika. Každá metoda je označena `@Tool`, aby byla dostupná přes MCP:

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
    
    // Další operace kalkulačky...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Hlavní vlastnosti:**

1. **Anotace `@Tool`**: Říká MCP, že tuto metodu mohou volat externí klienti
2. **Srozumitelné popisy**: Každý nástroj má popis, který pomáhá AI modelům pochopit, kdy jej použít
3. **Konzistentní formát výsledku**: Všechny operace vrací lidsky čitelné řetězce jako "5.00 + 3.00 = 8.00"
4. **Zpracování chyb**: Dělení nulou a odmocnina ze záporného čísla vrací chybové hlášky

**Dostupné operace:**
- `add(a, b)` - Sčítá dvě čísla
- `subtract(a, b)` - Odečítá druhé číslo od prvního
- `multiply(a, b)` - Násobí dvě čísla
- `divide(a, b)` - Dělí první číslo druhým (s kontrolou nulou)
- `power(base, exponent)` - Umocňuje základ na exponent
- `squareRoot(number)` - Vypočítá druhou odmocninu (s kontrolou záporných vstupů)
- `modulus(a, b)` - Vrací zbytek po dělení
- `absolute(number)` - Vrací absolutní hodnotu čísla
- `help()` - Vrací informace o všech operacích

### 3. Přímý MCP klient

**Soubor:** `SDKClient.java`

Tento klient komunikuje přímo s MCP serverem bez použití AI. Ručně volá specifické kalkulační funkce:

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
        
        // Vypsat dostupné nástroje
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Zavolat konkrétní funkce kalkulačky
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

**Co to dělá:**
1. **Připojuje se** ke kalkulačnímu serveru na `http://localhost:8080` pomocí builder patternu
2. **Vylistuje** všechny dostupné nástroje (naše kalkulační funkce)
3. **Volá** konkrétní funkce s přesnými parametry
4. **Vypisuje** výsledky přímo

**Poznámka:** Tento příklad používá závislost Spring AI 1.1.0-SNAPSHOT, která zavedla builder pattern pro `WebFluxSseClientTransport`. Pokud používáte starší stabilní verzi, možná budete muset použít přímý konstruktor.

**Kdy to použít:** Pokud přesně víte, jaký výpočet chcete provést, a chcete jej volat programově.

### 4. Klient využívající AI

**Soubor:** `LangChain4jClient.java`

Tento klient využívá AI model (GPT-4o-mini), který automaticky rozhoduje, které kalkulační nástroje použít:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Nastavte AI model (Azure AI Foundry, autentizace bez klíče přes Microsoft Entra ID)
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

        // Připojte se k našemu serveru kalkulačky MCP
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Ukazuje, co AI právě dělá
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Dejte AI přístup k našim kalkulačním nástrojům
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Vytvořte AI bota, který může používat naši kalkulačku
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Nyní můžeme požádat AI, aby prováděla výpočty v přirozeném jazyce
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Co to dělá:**
1. **Vytvoří** propojení k AI modelu pomocí vašeho GitHub tokenu
2. **Připojí** AI ke kalkulačnímu MCP serveru
3. **Dá** AI přístup ke všem kalkulačním nástrojům
4. **Umožní** přirozené jazykové požadavky jako "Spočítej součet 24.5 a 17.3"

**AI automaticky:**
- Chápe, že chcete sčítat čísla
- Vybere nástroj `add`
- Zavolá `add(24.5, 17.3)`
- Vrátí výsledek v přirozené odpovědi

## Spuštění příkladů

### Krok 1: Spusťte kalkulační server

Nejprve se přihlaste a nastavte váš Azure AI Foundry endpoint (potřebné pro AI klienta — autentizace bez klíče, bez API klíče):

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

Spusťte server:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

Server poběží na `http://localhost:8080`. Měli byste vidět:
```
Started McpServerApplication in X.XXX seconds
```

### Krok 2: Otestujte s přímým klientem

V **NOVÉM** terminálu, zatímco server běží, spusťte přímého MCP klienta:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Uvidíte výstup jako:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Krok 3: Otestujte s AI klientem

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Uvidíte, jak AI automaticky používá nástroje:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Krok 4: Ukončete MCP server

Když skončíte s testováním, můžete ukončit AI klienta stiskem `Ctrl+C` v jeho terminálu. MCP server poběží dál, dokud jej nezastavíte.
Pro zastavení serveru stiskněte `Ctrl+C` v terminálu, kde běží.

## Jak to spolu funguje

Zde je kompletní postup, když se AI zeptáte „Kolik je 5 + 3?“:

1. **Vy** položíte AI otázku v přirozeném jazyce
2. **AI** analyzuje váš dotaz a pozná, že chcete sčítat
3. **AI** zavolá MCP server: `add(5.0, 3.0)`
4. **Kalkulační služba** provede výpočet: `5.0 + 3.0 = 8.0`
5. **Kalkulační služba** vrátí: `"5.00 + 3.00 = 8.00"`
6. **AI** přijme výsledek a vytvoří odpověď v přirozeném jazyce
7. **Vy** dostanete odpověď: "Součet 5 a 3 je 8"

## Další kroky

Pro více příkladů viz [Kapitola 04: Praktické ukázky](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Prohlášení o omezení odpovědnosti**:
Tento dokument byl přeložen pomocí AI překladatelské služby [Co-op Translator](https://github.com/Azure/co-op-translator). Přestože usilujeme o co největší přesnost, mějte prosím na paměti, že automatizované překlady mohou obsahovat chyby nebo nepřesnosti. Originální dokument v jeho mateřském jazyce by měl být považován za autoritativní zdroj. Pro kritické informace se doporučuje profesionální lidský překlad. Nejsme odpovědní za jakékoli nedorozumění nebo nesprávné interpretace vzniklé použitím tohoto překladu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->