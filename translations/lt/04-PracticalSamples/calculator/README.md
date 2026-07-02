# MCP skaičiuotuvo mokymo programa pradedantiesiems

## Turinys

- [Ko Išmoksite](#ko-išmoksite)
- [Prieš Reikalavimus](#prieš-reikalavimus)
- [Projekto Struktūros Supratimas](#projekto-struktūros-supratimas)
- [Pagrindinės Komponentų Paaiškinimas](#pagrindinės-komponentų-paaiškinimas)
  - [1. Pagrindinė Programa](#1-pagrindinė-programa)
  - [2. Skaičiuotuvo Servisas](#2-skaičiuotuvo-servisas)
  - [3. Tiesioginis MCP Klientas](#3-tiesioginis-mcp-klientas)
  - [4. AI Valdomas Klientas](#4-ai-valdomas-klientas)
- [Pavyzdžių Vykdymas](#pavyzdžių-vykdymas)
- [Kaip Viskas Veikia Kartu](#kaip-viskas-veikia-kartu)
- [Kiti Žingsniai](#kiti-žingsniai)

## Ko Išmoksite

Ši mokymo programa paaiškina, kaip sukurti skaičiuotuvo servisą naudojant Modelio Konteksto Protokolą (MCP). Suprasite:

- Kaip sukurti servisą, kurį AI gali naudoti kaip įrankį
- Kaip nustatyti tiesioginę komunikaciją su MCP servisais
- Kaip AI modeliai automatiškai pasirenka, kuriuos įrankius naudoti
- Skirtumą tarp tiesioginių protokolo skambučių ir AI pagalba vykdomų sąveikų

## Prieš Reikalavimus

Prieš pradėdami, įsitikinkite, kad turite:
- Įdiegta Java 21 arba naujesnę versiją
- Maven priklausomybių valdymui
- Azure AI Foundry modelio diegimą (įdiegti su `azd up` — žr. [2 skyrių](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), prisijungusi su `az login` (autentifikacija be rakto)
- Pagrindines žinias apie Java ir Spring Boot

## Projekto Struktūros Supratimas

Skaičiuotuvo projekte yra keletas svarbių failų:

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

## Pagrindinės Komponentų Paaiškinimas

### 1. Pagrindinė Programa

**Failas:** `McpServerApplication.java`

Tai mūsų skaičiuotuvo serviso įėjimo taškas. Tai standartinė Spring Boot programa su viena ypatinga dalimi:

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

**Ką tai daro:**
- Paleidžia Spring Boot interneto serverį 8080 prievade
- Sukuria `ToolCallbackProvider`, kuris leidžia mūsų skaičiuotuvo metodus naudoti kaip MCP įrankius
- `@Bean` anotacija nurodo Spring valdyti šį komponentą, kad kitos dalys galėtų jį naudoti

### 2. Skaičiuotuvo Servisas

**Failas:** `CalculatorService.java`

Čia vyksta visa matematika. Kiekvienas metodas pažymėtas `@Tool`, kad būtų prieinamas per MCP:

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
    
    // Daugiau kalkuliatoriaus operacijų...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Svarbiausios savybės:**

1. **`@Tool` anotacija**: Nurodo MCP, kad šį metodą gali kviesti išoriniai klientai
2. **Aiškūs aprašymai**: Kiekvienas įrankis turi aprašymą, kuris padeda AI modeliams suprasti, kada jį naudoti
3. **Nuosekli atsako formatas**: Visi veiksmai grąžina suprantamus tekstinius rezultatus, pvz., "5.00 + 3.00 = 8.00"
4. **Klaidų tvarkymas**: Dalykimas iš nulio ir neigiamas kvadratinės šaknies skaičiavimas grąžina klaidų pranešimus

**Galimi veiksmai:**
- `add(a, b)` - Sudeda du skaičius
- `subtract(a, b)` - Atima antrą skaičių iš pirmo
- `multiply(a, b)` - Dauginimas
- `divide(a, b)` - Dalyba (tikrina nulius)
- `power(base, exponent)` - Laipsnio kėlimas
- `squareRoot(number)` - Kvadratinė šaknis (tikrina neigiamas reikšmes)
- `modulus(a, b)` - Liko radimas po dalybos
- `absolute(number)` - Absoliuti reikšmė
- `help()` - Grąžina informaciją apie visus veiksmus

### 3. Tiesioginis MCP Klientas

**Failas:** `SDKClient.java`

Šis klientas tiesiogiai bendrauja su MCP serveriu be AI. Rankiniu būdu kviečia specifines skaičiuotuvo funkcijas:

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
        
        // Išvardinkite prieinamus įrankius
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Kvieskite konkrečias skaičiuotuvo funkcijas
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

**Ką tai daro:**
1. **Prisijungia** prie skaičiuotuvo serverio adresu `http://localhost:8080` naudodamas builder šabloną
2. **Išveda** visų įrankių sąrašą (mūsų skaičiuotuvo funkcijos)
3. **Kviečia** konkrečias funkcijas su tiksliomis reikšmėmis
4. **Atspausdina** rezultatus tiesiogiai

**Pastaba:** Šiame pavyzdyje naudojama Spring AI 1.1.0-SNAPSHOT priklausomybė, kuri pristatė builder šabloną `WebFluxSseClientTransport`. Jei naudojate senesnę stabilią versiją, gali reikėti naudoti tiesioginį konstruktorių.

**Naudojimo atvejis:** Kai žinote tiksliai, kokį skaičiavimą norite atlikti ir norite jį iškvieti programiškai.

### 4. AI Valdomas Klientas

**Failas:** `LangChain4jClient.java`

Šis klientas naudoja AI modelį (GPT-4o-mini), kuris automatiškai sprendžia, kuriuos skaičiuotuvo įrankius naudoti:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Nustatykite AI modelį (Azure AI Foundry, autentifikacija be rakto per Microsoft Entra ID)
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

        // Prisijunkite prie mūsų skaičiuotuvo MCP serverio
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Rodo, ką AI daro
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Suteikite AI prieigą prie mūsų skaičiuotuvo įrankių
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Sukurkite AI robotą, kuris gali naudoti mūsų skaičiuotuvą
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Dabar galime prašyti AI atlikti skaičiavimus natūralia kalba
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Ką tai daro:**
1. **Sukuria** AI modelio ryšį naudojant jūsų GitHub tokeną
2. **Prijungia** AI prie mūsų MCP skaičiuotuvo serverio
3. **Suteikia** AI prieigą prie visų skaičiuotuvo įrankių
4. **Leidžia** natūralios kalbos užklausas, pvz., "Apskaičiuokite 24.5 ir 17.3 sumą"

**AI automatiškai:**
- Supranta, kad norite sudėti skaičius
- Pasirenka `add` įrankį
- Kvies `add(24.5, 17.3)`
- Grąžina rezultatą natūralios kalbos atsakyme

## Pavyzdžių Vykdymas

### 1 žingsnis: Paleiskite Skaičiuotuvo Serverį

Pirmiausia prisijunkite ir nustatykite Azure AI Foundry endpointą (būtina AI klientui – autentifikacija be rakto, API rakto nereikia):

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

Paleiskite serverį:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

Serveris bus pasiekiamas adresu `http://localhost:8080`. Turėtumėte matyti:
```
Started McpServerApplication in X.XXX seconds
```

### 2 žingsnis: Išbandykite Tiesioginį Klientą

Naujoje terminalo lange, kol serveris paleistas, vykdykite tiesioginį MCP klientą:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Pamatysite tokį išvedimą:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### 3 žingsnis: Išbandykite AI Klientą

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Pamatysite, kaip AI automatiškai naudoja įrankius:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### 4 žingsnis: Užverkite MCP Serverį

Kai baigsite testuoti, AI klientą užverkite paspausdami `Ctrl+C` jo terminale. MCP serveris veiks tol, kol jį sustabdysite.
Norėdami sustabdyti serverį, paspauskite `Ctrl+C` terminale, kuriame jis veikia.

## Kaip Viskas Veikia Kartu

Štai visas procesas, kai jūs klausiat AI „Kiek yra 5 + 3?“:

1. **Jūs** užduodate klausimą natūralia kalba
2. **AI** analizuoja Jūsų užklausą ir supranta, kad norite sudėtis
3. **AI** iškviečia MCP serverį: `add(5.0, 3.0)`
4. **Skaičiuotuvo Servisas** atlieka veiksmą: `5.0 + 3.0 = 8.0`
5. **Skaičiuotuvo Servisas** grąžina: `"5.00 + 3.00 = 8.00"`
6. **AI** gauna rezultatą ir suformuoja natūralų atsakymą
7. **Jūs** gaunate: „Skaičių 5 ir 3 suma yra 8“

## Kiti Žingsniai

Daugiau pavyzdžių rasite [4 skyriuje: Praktiniai pavyzdžiai](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Atsakomybės apribojimas**:
Šis dokumentas buvo išverstas naudojant dirbtinio intelekto vertimo paslaugą [Co-op Translator](https://github.com/Azure/co-op-translator). Nors siekiame tikslumo, prašome atkreipti dėmesį, kad automatiniai vertimai gali turėti klaidų ar netikslumų. Originalus dokumentas jo gimtąja kalba laikomas autoritetingu šaltiniu. Svarbiai informacijai rekomenduojama naudoti profesionalų žmogiškąjį vertimą. Mes neatsakome už jokius nesusipratimus ar neteisingą interpretaciją, kilusią naudojantis šiuo vertimu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->