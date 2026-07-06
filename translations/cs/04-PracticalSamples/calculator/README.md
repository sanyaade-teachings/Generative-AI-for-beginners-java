# MCP Kalkulačka Tutorial pro Začátečníky

## Obsah

- [Co se Naučíte](#co-se-naučíte)
- [Požadavky](#požadavky)
- [Porozumění Struktuře Projektu](#porozumění-struktuře-projektu)
- [Vysvětlení Hlavních Komponent](#vysvětlení-hlavních-komponent)
  - [1. Hlavní Aplikace](#1-hlavní-aplikace)
  - [2. Kalkulační Služba](#2-kalkulační-služba)
  - [3. Přímý MCP Klient](#3-přímý-mcp-klient)
  - [4. Klient Pohlížený Umělou Inteligencí](#4-klient-pohlížený-umělou-inteligencí)
- [Spuštění Příkladů](#spuštění-příkladů)
- [Jak To Všechno Funguje Dohromady](#jak-to-všechno-funguje-dohromady)
- [Další Kroky](#další-kroky)

## Co se Naučíte

Tento tutoriál vysvětluje, jak vytvořit kalkulační službu pomocí Model Context Protocol (MCP). Pochopíte:

- Jak vytvořit službu, kterou může AI využít jako nástroj
- Jak nastavit přímou komunikaci se službami MCP
- Jak modely AI mohou automaticky vybírat, které nástroje použít
- Rozdíl mezi přímými voláními protokolu a interakcemi asistovanými AI

## Požadavky

Než začnete, ujistěte se, že máte:
- Nainstalovanou Javu 21 nebo vyšší
- Maven pro správu závislostí
- Nasazení modelu Azure AI Foundry (zprovozněte příkazem `azd up` — viz [Kapitola 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), přihlášený příkazem `az login` (autentizace bez klíče)
- Základní znalost Javy a Spring Boot

## Porozumění Struktuře Projektu

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

## Vysvětlení Hlavních Komponent

### 1. Hlavní Aplikace

**Soubor:** `McpServerApplication.java`

Toto je vstupní bod naší kalkulační služby. Je to standardní Spring Boot aplikace s jedním speciálním doplňkem:

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
- Spouští webový server Spring Boot na portu 8080
- Vytváří `ToolCallbackProvider`, který zpřístupňuje naše kalkulační metody jako MCP nástroje
- Anotace `@Bean` říká Springu, aby tuto komponentu spravoval a aby ji mohly používat ostatní části aplikace

### 2. Kalkulační Služba

**Soubor:** `CalculatorService.java`

Zde se děje veškerá matematika. Každá metoda je označena anotací `@Tool`, aby byla dostupná přes MCP:

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

1. **Anotace `@Tool`**: Umožňuje MCP, aby tuto metodu volali externí klienti
2. **Jasné popisy**: Každý nástroj má popis, který pomáhá AI modelům pochopit, kdy jej použít
3. **Konzistentní formát návratové hodnoty**: Všechny operace vracejí lidsky čitelné řetězce jako "5.00 + 3.00 = 8.00"
4. **Zpracování chyb**: Dělení nulou a odmocnina z negativního čísla vrací chybová hlášení

**Dostupné operace:**
- `add(a, b)` - Sčítá dvě čísla
- `subtract(a, b)` - Odečítá druhé číslo od prvního
- `multiply(a, b)` - Násobí dvě čísla
- `divide(a, b)` - Dělí první číslo druhým (kontrola nulou)
- `power(base, exponent)` - Umocňuje základ na exponent
- `squareRoot(number)` - Vypočítá druhou odmocninu (kontrola na záporné číslo)
- `modulus(a, b)` - Vrací zbytek po dělení
- `absolute(number)` - Vrací absolutní hodnotu
- `help()` - Vrací informace o všech operacích

### 3. Přímý MCP Klient

**Soubor:** `SDKClient.java`

Tento klient komunikuje přímo se serverem MCP bez použití AI. Ručně volá specifické kalkulační funkce:

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
1. **Připojuje se** k serveru kalkulačky na `http://localhost:8080` pomocí builder patternu
2. **Vypisuje** všechny dostupné nástroje (naše kalkulační funkce)
3. **Volá** konkrétní funkce s přesnými parametry
4. **Tiskne** výsledky přímo

**Poznámka:** Tento příklad používá závislost Spring AI 1.1.0-SNAPSHOT, která zavedla builder pattern pro `WebFluxSseClientTransport`. Pokud používáte starší stabilní verzi, možná budete muset použít přímý konstruktor.

**Kdy použít:** Když přesně víte, jaký výpočet chcete provést, a chcete ho volat programově.

### 4. Klient Pohlížený Umělou Inteligencí

**Soubor:** `LangChain4jClient.java`

Tento klient používá AI model (GPT-4o-mini), který může automaticky rozhodnout, které kalkulační nástroje použít:

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
                .logRequests(true)  // Zobrazuje, co AI právě dělá
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Dejte AI přístup k našim kalkulačkovým nástrojům
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
1. **Vytváří** připojení k AI modelu s autentizací bez klíče (Microsoft Entra ID)
2. **Připojuje** AI ke kalkulačnímu MCP serveru
3. **Dává** AI přístup ke všem kalkulačním nástrojům
4. **Umožňuje** přirozené jazykové požadavky jako „Vypočítej součet 24.5 a 17.3“

**AI automaticky:**
- Chápe, že chcete sčítat čísla
- Vybere nástroj `add`
- Zavolá `add(24.5, 17.3)`
- Vrátí výsledek v přirozené odpovědi

## Spuštění Příkladů

### Krok 1: Spusťte Server Kalkulačky

Nejdříve se přihlaste a nastavte svůj Azure AI Foundry endpoint (potřebné pro AI klienta — autentizace bez klíče, bez API klíče):

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

Server se spustí na `http://localhost:8080`. Měli byste vidět:
```
Started McpServerApplication in X.XXX seconds
```

### Krok 2: Otestujte s Přímým Klientem

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

### Krok 3: Otestujte s AI Klientem

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Uvidíte, jak AI automaticky používá nástroje:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Krok 4: Ukončete MCP Server

Až skončíte s testováním, AI klienta zastavíte stiskem `Ctrl+C` v jeho terminálu. MCP server poběží dál, dokud ho nezastavíte.
Pro zastavení serveru stiskněte `Ctrl+C` v terminálu, kde běží.

## Jak To Všechno Funguje Dohromady

Zde je kompletní průběh, když se AI zeptáte „Kolik je 5 + 3?“:

1. **Vy** se ptáte AI přirozeným jazykem
2. **AI** analyzuje požadavek a pochopí, že chcete sčítat
3. **AI** zavolá MCP server: `add(5.0, 3.0)`
4. **Kalkulační Služba** provede: `5.0 + 3.0 = 8.0`
5. **Kalkulační Služba** vrátí: `"5.00 + 3.00 = 8.00"`
6. **AI** přijme výsledek a vytvoří přirozenou odpověď
7. **Vy** dostanete: „Součet 5 a 3 je 8“

## Další Kroky

Pro více příkladů viz [Kapitola 04: Praktické ukázky](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Prohlášení o omezení odpovědnosti**:
Tento dokument byl přeložen pomocí AI překladatelské služby [Co-op Translator](https://github.com/Azure/co-op-translator). Přestože usilujeme o co největší přesnost, mějte prosím na paměti, že automatizované překlady mohou obsahovat chyby nebo nepřesnosti. Originální dokument v jeho mateřském jazyce by měl být považován za autoritativní zdroj. Pro kritické informace se doporučuje profesionální lidský překlad. Nejsme odpovědní za jakékoli nedorozumění nebo nesprávné interpretace vzniklé použitím tohoto překladu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->