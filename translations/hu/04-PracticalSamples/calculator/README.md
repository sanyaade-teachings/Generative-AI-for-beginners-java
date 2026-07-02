# MCP Számológép Oktatóanyag Kezdőknek

## Tartalomjegyzék

- [Mit fogsz megtanulni](#mit-fogsz-megtanulni)
- [Előfeltételek](#előfeltételek)
- [A projekt struktúrájának megértése](#a-projekt-struktúrájának-megértése)
- [A fő komponensek magyarázata](#a-fő-komponensek-magyarázata)
  - [1. Fő alkalmazás](#1-fő-alkalmazás)
  - [2. Számológép szolgáltatás](#2-számológép-szolgáltatás)
  - [3. Közvetlen MCP kliens](#3-közvetlen-mcp-kliens)
  - [4. Mesterséges intelligencia által vezérelt kliens](#4-mesterséges-intelligencia-által-vezérelt-kliens)
- [A példák futtatása](#a-példák-futtatása)
- [Hogyan működik az egész együtt](#hogyan-működik-az-egész-együtt)
- [Következő lépések](#következő-lépések)

## Mit fogsz megtanulni

Ez az oktatóanyag elmagyarázza, hogyan építsünk egy számológép szolgáltatást a Model Context Protocol (MCP) használatával. Meg fogod érteni:

- Hogyan készíts szolgáltatást, amelyet a MI eszközként használhat
- Hogyan állíts be közvetlen kommunikációt az MCP szolgáltatásokkal
- Hogyan választhatnak automatikusan a MI modellek eszközöket
- A közvetlen protokollhívások és a MI által segített interakciók közötti különbséget

## Előfeltételek

Mielőtt elkezdenéd, győződj meg róla, hogy:
- Telepítve van Java 21 vagy újabb verzió
- Maven a függőségkezeléshez
- Egy Azure AI Foundry modell telepítése (állítsd be az `azd up` paranccsal — lásd [2. fejezet](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- Az [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) telepítve, bejelentkezve `az login` parancssal (kulcs nélküli hitelesítés)
- Alapismeretek Java és Spring Boot témakörben

## A projekt struktúrájának megértése

A számológép projekt több fontos fájlt tartalmaz:

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

## A fő komponensek magyarázata

### 1. Fő alkalmazás

**Fájl:** `McpServerApplication.java`

Ez a belépési pontja a számológép szolgáltatásunknak. Egy szokásos Spring Boot alkalmazás egy különleges kiegészítéssel:

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

**Mit csinál ez:**
- Egy Spring Boot web szervert indít a 8080-as porton
- Létrehoz egy `ToolCallbackProvider`-t, amely elérhetővé teszi számológép metódusainkat MCP eszközként
- Az `@Bean` annotáció arra utasítja a Springet, hogy ezt egy olyan komponensként kezelje, amelyet más részek is használhatnak

### 2. Számológép szolgáltatás

**Fájl:** `CalculatorService.java`

Itt történik az összes matematikai művelet. Minden metódus `@Tool`-lal van megjelölve, hogy elérhető legyen MCP-n keresztül:

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
    
    // További számológép műveletek...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Fő jellemzők:**

1. **`@Tool` annotáció**: Ez jelzi az MCP-nek, hogy ezt a metódust külső kliensek hívhatják meg
2. **Világos leírások**: Minden eszköz tartalmaz leírást, ami segíti a MI modelleket annak megértésében, mikor használják azt
3. **Egységes visszatérési formátum**: Minden művelet ember által olvasható stringet ad vissza, például "5.00 + 3.00 = 8.00"
4. **Hibakezelés**: Nullával való osztás és negatív gyök szintén hibajelzést ad vissza

**Elérhető műveletek:**
- `add(a, b)` - Két szám összeadása
- `subtract(a, b)` - Kivonja a másodikat az elsőből
- `multiply(a, b)` - Két szám összeszorzása
- `divide(a, b)` - Osztja az elsőt a másodikkal (nulla ellenőrzéssel)
- `power(base, exponent)` - A bázist hatványra emeli
- `squareRoot(number)` - Négyzetgyök számítása (negatív ellenőrzéssel)
- `modulus(a, b)` - Maradékos osztás eredménye
- `absolute(number)` - Abszolút érték visszaadása
- `help()` - Információk az összes műveletről

### 3. Közvetlen MCP kliens

**Fájl:** `SDKClient.java`

Ez a kliens közvetlenül az MCP szerverrel kommunikál AI nélkül. Kézzel hív meg konkrét számológép funkciókat:

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
        
        // Elérhető eszközök listázása
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Meghatározott számológép funkciók hívása
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

**Mit csinál ez:**
1. **Csatlakozik** a számológép szerverhez a `http://localhost:8080` címen a builder mintával
2. **Felsorolja** az összes elérhető eszközt (számológép funkcióinkat)
3. **Hív** konkrét funkciókat pontos paraméterekkel
4. **Kiírja** az eredményeket közvetlenül

**Megjegyzés:** Ez a példa a Spring AI 1.1.0-SNAPSHOT függőséget használja, amely bevezette a builder mintát a `WebFluxSseClientTransport`-hoz. Ha régebbi stabil verziót használsz, lehet, hogy a közvetlen konstruktort kell használnod.

**Mikor használd:** Amikor pontosan tudod, milyen számítást akarsz végezni, és programozottan akarod hívni.

### 4. Mesterséges intelligencia által vezérelt kliens

**Fájl:** `LangChain4jClient.java`

Ez a kliens egy MI modellt használ (GPT-4o-mini), amely automatikusan eldönti, melyik számológép eszközöket használja:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Állítsa be az MI modellt (Azure AI Foundry, kulcs nélküli hitelesítés a Microsoft Entra ID-n keresztül)
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

        // Csatlakozás a számológép MCP szerverünkhöz
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Megmutatja, mit csinál az MI
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Adjon hozzáférést az MI-nek a számológép eszközeinkhez
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Hozzon létre egy MI botot, amely használhatja a számológépünket
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Most már megkérhetjük az MI-t, hogy természetes nyelven végezzen számításokat
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Mit csinál ez:**
1. **Létrehoz** egy MI modell kapcsolatot a GitHub tokeneddel
2. **Csatlakoztatja** az MI-t a számológép MCP szerverhez
3. **Hozzáférést ad** az MI-nek az összes számológép eszközünkhöz
4. **Engedélyezi** a természetes nyelvű kéréseket, például "Számold ki 24,5 és 17,3 összegét"

**Az MI automatikusan:**
- Megérti, hogy összeadást szeretnél
- Kiválasztja az `add` eszközt
- Meghívja az `add(24.5, 17.3)`-at
- Visszaadja az eredményt természetes válasz formátumban

## A példák futtatása

### 1. lépés: Indítsd el a számológép szervert

Először jelentkezz be és állítsd be az Azure AI Foundry végpontodat (szükséges az MI klienshez — kulcs nélküli hitelesítés, nincs API kulcs):

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

Indítsd el a szervert:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

A szerver a `http://localhost:8080` címen fog futni. Ezt kell látnod:
```
Started McpServerApplication in X.XXX seconds
```

### 2. lépés: Teszteld a közvetlen klienst

Egy **ÚJ** terminálablakban, miközben a szerver fut, indítsd el a közvetlen MCP klienst:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Ilyen kimenetet fogsz látni:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### 3. lépés: Teszteld az MI klienst

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Látni fogod, hogy az MI automatikusan használ eszközöket:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### 4. lépés: Állítsd le az MCP szervert

Ha végeztél a teszteléssel, leállíthatod az MI klienst a `Ctrl+C` billentyűkombinációval a terminálban. Az MCP szerver futni fog, amíg te le nem állítod.
A szerver leállításához nyomd meg a `Ctrl+C`-t abban a terminálablakban, ahol fut.

## Hogyan működik az egész együtt

Íme a teljes folyamat, amikor megkérdezed az MI-től: „Mi az 5 + 3?”:

1. **Te** természetes nyelven kérdezel az MI-től
2. **Az MI** elemzi a kérésed és felismeri, hogy összeadást akarsz
3. **Az MI** meghívja az MCP szervert: `add(5.0, 3.0)`
4. **A számológép szolgáltatás** végrehajtja: `5.0 + 3.0 = 8.0`
5. **A számológép szolgáltatás** visszaadja: `"5.00 + 3.00 = 8.00"`
6. **Az MI** megkapja az eredményt és természetes válaszzá alakítja
7. **Te** ezt kapod: "Az 5 és 3 összege 8"

## Következő lépések

További példákért lásd a [4. fejezet: Gyakorlati példák](../README.md)-t

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Jogi nyilatkozat**:
Ez a dokumentum az AI fordítási szolgáltatás, a [Co-op Translator](https://github.com/Azure/co-op-translator) segítségével készült. Bár az pontosságra törekszünk, kérjük, vegye figyelembe, hogy az automatikus fordítások hibákat vagy pontatlanságokat tartalmazhatnak. Az eredeti dokumentum az anyanyelvén tekintendő hiteles forrásnak. Fontos információk esetén professzionális emberi fordítást javasolunk. Nem vállalunk felelősséget semmilyen félreértésért vagy téves értelmezésért, amely ebből a fordításból ered.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->