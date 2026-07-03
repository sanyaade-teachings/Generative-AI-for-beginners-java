# MCP számológép oktatóanyag kezdőknek

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
- [Hogyan működik együtt az egész](#hogyan-működik-együtt-az-egész)
- [Következő lépések](#következő-lépések)

## Mit fogsz megtanulni

Ez az oktatóanyag azt magyarázza el, hogyan építsünk fel egy számológép szolgáltatást a Model Context Protocol (MCP) segítségével. Meg fogod érteni:

- Hogyan hozz létre egy szolgáltatást, amelyet a mesterséges intelligencia eszközként használhat
- Hogyan állíts be közvetlen kommunikációt az MCP szolgáltatásokkal
- Hogyan választhatnak az MI modellek automatikusan eszközöket
- A közvetlen protokollhívások és az MI által segített interakciók közötti különbséget

## Előfeltételek

A kezdés előtt győződj meg róla, hogy rendelkezel:
- Java 21 vagy újabb verzióval telepítve
- Mavennel a függőségkezeléshez
- Egy Azure AI Foundry modell telepítésével (telepítsd `azd up` parancsal — lásd a [2. fejezetet](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- Az [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) telepítve, bejelentkezve `az login`-nal (kulcs nélküli hitelesítés)
- Alapvető Java és Spring Boot ismeretekkel

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

Ez a számológép szolgáltatásunk belépési pontja. Ez egy szabványos Spring Boot alkalmazás egy különleges kiegészítéssel:

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
- Elindít egy Spring Boot web szervert a 8080-as porton
- Létrehoz egy `ToolCallbackProvider`-t, ami elérhetővé teszi számológép metódusainkat MCP eszközként
- Az `@Bean` annotációval a Spring kezeli ezt komponensként, amit más részek használhatnak

### 2. Számológép szolgáltatás

**Fájl:** `CalculatorService.java`

Itt történnek a számítások. Minden metódus `@Tool` annotációval van jelölve, hogy MCP-n keresztül elérhető legyen:

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

1. **`@Tool` annotáció**: Jelzi az MCP számára, hogy ezt a metódust külső kliensek hívhatják
2. **Egyértelmű leírások**: Minden eszköz rendelkezik leírással, ami segít az MI modelleknek megérteni mikor használják
3. **Konzisztens visszatérési formátum**: Minden művelet ember által olvasható sztringet ad vissza, pl. "5.00 + 3.00 = 8.00"
4. **Hibakezelés**: A nullával való osztás és a negatív négyzetgyök esetén hibaüzeneteket ad vissza

**Elérhető műveletek:**
- `add(a, b)` - Két szám összeadása
- `subtract(a, b)` - Az elsőből kivonja a másodikat
- `multiply(a, b)` - Két szám szorzása
- `divide(a, b)` - Az első osztása a másodikkal (nulla ellenőrzéssel)
- `power(base, exponent)` - Az alap hatványozása
- `squareRoot(number)` - Négyzetgyök számítás (negatív ellenőrzéssel)
- `modulus(a, b)` - Maradékos osztás eredménye
- `absolute(number)` - Abszolút érték
- `help()` - Információk visszaadása az összes műveletről

### 3. Közvetlen MCP kliens

**Fájl:** `SDKClient.java`

Ez a kliens közvetlenül az MCP szerverrel kommunikál AI nélkül. Kézzel hívja meg a kijelölt számológép funkciókat:

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
        
        // Az elérhető eszközök listázása
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Meghívni egyéni számológép funkciókat
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

**Mit csinál:**
1. **Kapcsolódik** a számológép szerverhez `http://localhost:8080` címen a builder mintával
2. **Felsorolja** az elérhető eszközöket (számológép funkcióinkat)
3. **Meghív** konkrét funkciókat pontos paraméterekkel
4. **Kiírja** az eredményeket közvetlenül

**Megjegyzés:** Ez a példa a Spring AI 1.1.0-SNAPSHOT függőséget használja, amely bevezetett egy builder mintát a `WebFluxSseClientTransport`-hoz. Ha régebbi stabil verziót használsz, lehet, hogy az egyszerű konstruktort kell alkalmazni.

**Mikor használd ezeket:** Amikor pontosan tudod, melyik számítást akarod elvégezni, és programból meghívnád.

### 4. Mesterséges intelligencia által vezérelt kliens

**Fájl:** `LangChain4jClient.java`

Ez a kliens egy MI modellt (GPT-4o-mini) használ, amely automatikusan dönti el, hogy melyik számológép eszközt használja:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Állítsa be az AI modellt (Azure AI Foundry, kulcs nélküli hitelesítés Microsoft Entra ID-n keresztül)
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

        // Csatlakozás a kalkulátor MCP szerverünkhöz
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Megmutatja, mit csinál az AI
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Adja meg az AI-nak a hozzáférést a kalkulátor eszközeinkhez
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Hozzon létre egy AI botot, amely képes használni a kalkulátorunkat
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Most már természetes nyelven kérhetjük az AI-t számításokra
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Mit csinál:**
1. **Létrehoz** egy MI modell kapcsolatot kulcs nélküli hitelesítéssel (Microsoft Entra ID)
2. **Csatlakoztatja** az MI-t számológép MCP szerverünkhöz
3. **Hozzáférést ad** az MI-nek az összes számológép eszközünkhöz
4. **Lehetővé teszi** természetes nyelvű kérések kezelését, pl. "Számold ki 24.5 és 17.3 összegét"

**Az MI automatikusan:**
- Megérti, hogy összeadásra van szükség
- Kiválasztja az `add` eszközt
- Meghívja az `add(24.5, 17.3)`-at
- Természetes formátumban adja vissza az eredményt

## A példák futtatása

### 1. lépés: Indítsd el a számológép szervert

Először jelentkezz be és állítsd be az Azure AI Foundry végpontot (szükséges az AI klienshez — kulcs nélküli hitelesítés, nincs API kulcs):

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

A szerver elindul `http://localhost:8080` címen. Ezt fogod látni:
```
Started McpServerApplication in X.XXX seconds
```

### 2. lépés: Teszteld a közvetlen klienst

Egy **ÚJ** terminálban, miközben a szerver fut, futtasd a közvetlen MCP klienst:
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

### 3. lépés: Teszteld az AI klienst

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Látni fogod, ahogy az MI automatikusan használja az eszközöket:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### 4. lépés: Állítsd le az MCP szervert

Amikor végeztél a tesztekkel, az AI kliens termináljában nyomd meg a `Ctrl+C`-t a leállításhoz. Az MCP szerver fut tovább, amíg le nem állítod.
A szerver leállításához nyomd meg a `Ctrl+C`-t abban a terminálban, ahol fut.

## Hogyan működik együtt az egész

Így néz ki a teljes folyamat, amikor az AI-nak azt kérdezed: "Mennyi 5 + 3?":

1. **Te** természetes nyelven teszed fel a kérdést az MI-nek
2. **Az MI** elemzi a kérést, és felismeri, hogy összeadásra van szükség
3. **Az MI** meghívja az MCP szervert: `add(5.0, 3.0)`
4. **A számológép szolgáltatás** elvégzi: `5.0 + 3.0 = 8.0`
5. **A szolgáltatás** visszaadja: `"5.00 + 3.00 = 8.00"`
6. **Az MI** megkapja az eredményt és természetes választ formáz belőle
7. **Te** megkapod: "5 és 3 összege 8"

## Következő lépések

További példákért lásd a [04. fejezetet: Gyakorlati példák](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Jogi nyilatkozat**:
Ez a dokumentum az AI fordítási szolgáltatás, a [Co-op Translator](https://github.com/Azure/co-op-translator) segítségével készült. Bár az pontosságra törekszünk, kérjük, vegye figyelembe, hogy az automatikus fordítások hibákat vagy pontatlanságokat tartalmazhatnak. Az eredeti dokumentum az anyanyelvén tekintendő hiteles forrásnak. Fontos információk esetén professzionális emberi fordítást javasolunk. Nem vállalunk felelősséget semmilyen félreértésért vagy téves értelmezésért, amely ebből a fordításból ered.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->