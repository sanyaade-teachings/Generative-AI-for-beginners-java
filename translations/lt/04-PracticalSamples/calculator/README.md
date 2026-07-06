# MCP skaičiuoklės pamoka pradedantiesiems

## Turinys

- [Ko išmoksite](#ko-išmoksite)
- [Reikalavimai](#reikalavimai)
- [Projekto struktūros supratimas](#projekto-struktūros-supratimas)
- [Pagrindiniai komponentai paaiškinti](#pagrindiniai-komponentai-paaiškinti)
  - [1. Pagrindinė programa](#1-pagrindinė-programa)
  - [2. Skaičiuoklės paslauga](#2-skaičiuoklės-paslauga)
  - [3. Tiesioginis MCP klientas](#3-tiesioginis-mcp-klientas)
  - [4. Dirbtiniu intelektu pagrįstas klientas](#4-dirbtiniu-intelektu-pagrįstas-klientas)
- [Pavyzdžių paleidimas](#pavyzdžių-paleidimas)
- [Kaip visa tai veikia kartu](#kaip-visa-tai-veikia-kartu)
- [Kiti žingsniai](#kiti-žingsniai)

## Ko išmoksite

Šioje pamokoje paaiškinama, kaip sukurti skaičiuoklės paslaugą naudojant Model Context Protocol (MCP). Sužinosite:

- Kaip sukurti paslaugą, kurią DI gali naudoti kaip įrankį
- Kaip nustatyti tiesioginę komunikaciją su MCP paslaugomis
- Kaip DI modeliai gali automatiškai pasirinkti, kuriuos įrankius naudoti
- Skirtumus tarp tiesioginių protokolo kvietimų ir DI pagalbos interakcijų

## Reikalavimai

Prieš pradedant, įsitikinkite, kad turite:
- Įdiegtą Java 21 ar naujesnę versiją
- Maven priklausomybių valdymui
- Azure AI Foundry modelio diegimą (įdiekite su `azd up` — žr. [2 skyrių](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), prisijungus su `az login` (autentifikacija be rakto)
- Pagrindines Java ir Spring Boot žinias

## Projekto struktūros supratimas

Skaičiuoklės projekte yra keletas svarbių failų:

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

## Pagrindiniai komponentai paaiškinti

### 1. Pagrindinė programa

**Failas:** `McpServerApplication.java`

Tai mūsų skaičiuoklės paslaugos įėjimo taškas. Tai standartinė Spring Boot programa su viena ypatinga pridėtine dalimi:

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

**Tai daro:**
- Paleidžia Spring Boot žiniatinklio serverį 8080 prievade
- Sukuria `ToolCallbackProvider`, kuris padaro mūsų skaičiuoklės metodus pasiekiamus kaip MCP įrankius
- `@Bean` anotacija nurodo Spring valdyti šį komponentą, kad kitos dalys galėtų jį naudoti

### 2. Skaičiuoklės paslauga

**Failas:** `CalculatorService.java`

Čia vyksta visi matematiniai skaičiavimai. Kiekvienas metodas pažymėtas `@Tool`, kad būtų prieinamas per MCP:

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
    
    // Daugiau skaičiuotuvo operacijų...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Pagrindinės savybės:**

1. **`@Tool` anotacija**: Nurodo MCP, kad šį metodą galima kviesti išoriniams klientams
2. **Aiškūs aprašymai**: Kiekvienas įrankis turi aprašymą, kuris padeda DI modeliams suprasti, kada jį naudoti
3. **Nuoseklus atsakymų formatas**: Visos operacijos grąžina žmonėms suprantamus tekstus, pvz., "5.00 + 3.00 = 8.00"
4. **Klaidų tvarkymas**: Dalyba iš nulio ir neigiamų skaičių šaknies radimas grąžina klaidų pranešimus

**Galimos operacijos:**
- `add(a, b)` - Sudeda du skaičius
- `subtract(a, b)` - Atima antrą skaičių iš pirmo
- `multiply(a, b)` - Dauginą du skaičius
- `divide(a, b)` - Dalina pirmą iš antro (tikrina nulį)
- `power(base, exponent)` - Kelia pagrindą laipsniu
- `squareRoot(number)` - Apskaičiuoja kvadratinę šaknį (tikrina neigiamus)
- `modulus(a, b)` - Grąžina likutį po dalybos
- `absolute(number)` - Grąžina absoliučią reikšmę
- `help()` - Grąžina informaciją apie visas operacijas

### 3. Tiesioginis MCP klientas

**Failas:** `SDKClient.java`

Šis klientas tiesiogiai bendrauja su MCP serveriu nenaudodamas DI. Rankiniu būdu kviečia konkrečias skaičiuoklės funkcijas:

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
        
        // Išvardinti galimus įrankius
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Iškviesti konkrečias skaičiuotuvo funkcijas
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

**Tai daro:**
1. **Prisijungia** prie skaičiuoklės serverio adresu `http://localhost:8080` naudodamas kūrėjo modelį
2. **Išvardina** visus prieinamus įrankius (mūsų skaičiuoklės funkcijas)
3. **Kviečia** konkrečias funkcijas su tikslias parametrais
4. **Išspausdina** rezultatus tiesiogiai

**Pastaba:** Šiame pavyzdyje naudojama Spring AI 1.1.0-SNAPSHOT priklausomybė, kuri pristatė kūrėjo modelį `WebFluxSseClientTransport`. Jei naudojate senesnę stabilią versiją, gali reikėti naudoti tiesioginį konstruktorių.

**Kada naudoti:** Kai žinote, kokį tikslų skaičiavimą norite atlikti ir norite jį iškviesti programiškai.

### 4. Dirbtiniu intelektu pagrįstas klientas

**Failas:** `LangChain4jClient.java`

Šis klientas naudoja DI modelį (GPT-4o-mini), kuris automatiškai nusprendžia, kuriuos skaičiuoklės įrankius naudoti:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Nustatykite DI modelį (Azure AI Foundry, be raktinio autentifikavimo per Microsoft Entra ID)
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

        // Prisijunkite prie mūsų MCP skaičiuoklio serverio
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Rodo, ką DI šiuo metu daro
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Suteikite DI prieigą prie mūsų skaičiuoklio įrankių
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Sukurkite DI botą, kuris gali naudoti mūsų skaičiuoklį
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Dabar galime prašyti DI atlikti skaičiavimus natūralia kalba
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Tai daro:**
1. **Sukuria** DI modelio ryšį be rakto (Microsoft Entra ID autentifikacija)
2. **Jungia** DI prie mūsų skaičiuoklės MCP serverio
3. **Suteikia** DI prieigą prie visų mūsų skaičiuoklės įrankių
4. **Leidžia** natūralios kalbos užklausas, pvz., "Apskaičiuok 24.5 ir 17.3 sumą"

**DI automatiškai:**
- Supranta, kad norite sudėti skaičius
- Pasirenka įrankį `add`
- Kvies `add(24.5, 17.3)`
- Grąžina rezultatą natūralioje atsakyme

## Pavyzdžių paleidimas

### 1 žingsnis: Paleiskite skaičiuoklės serverį

Pirmiausia prisijunkite ir nustatykite savo Azure AI Foundry galinį tašką (reikalinga DI klientui — autentifikacija be rakto, be API rakto):

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

Serveris pradės veikti adresu `http://localhost:8080`. Turėtumėte pamatyti:
```
Started McpServerApplication in X.XXX seconds
```

### 2 žingsnis: Išbandykite tiesioginį klientą

Naujoje terminalo lange, kai serveris dar veikia, paleiskite tiesioginį MCP klientą:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Pamatysite tokį išvestį:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### 3 žingsnis: Išbandykite DI klientą

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Pamatysite, kaip DI automatiškai naudoja įrankius:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### 4 žingsnis: Užbaikite MCP serverio darbą

Kai baigsite testavimą, galite sustabdyti DI klientą paspausdami `Ctrl+C` jo terminale. MCP serveris toliau veiks, kol jį sustabdysite.
Serverį sustabdyti galite paspaudę `Ctrl+C` terminale, kuriame jis veikia.

## Kaip visa tai veikia kartu

Štai visas procesas, kai paklausiate DI „Kiek yra 5 + 3?“:

1. **Jūs** kreipiatės į DI natūralia kalba
2. **DI** analizuoja užklausą ir supranta, kad norite sudėti
3. **DI** kviečia MCP serverį: `add(5.0, 3.0)`
4. **Skaičiuoklės paslauga** atlieka: `5.0 + 3.0 = 8.0`
5. **Skaičiuoklės paslauga** grąžina: `"5.00 + 3.00 = 8.00"`
6. **DI** gauna rezultatą ir pateikia natūralų atsakymą
7. **Jūs** išgirstate: "5 ir 3 suma yra 8"

## Kiti žingsniai

Daugiau pavyzdžių rasite [4 skyriuje: Praktiniai pavyzdžiai](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Atsakomybės apribojimas**:
Šis dokumentas buvo išverstas naudojant dirbtinio intelekto vertimo paslaugą [Co-op Translator](https://github.com/Azure/co-op-translator). Nors siekiame tikslumo, prašome atkreipti dėmesį, kad automatiniai vertimai gali turėti klaidų ar netikslumų. Originalus dokumentas jo gimtąja kalba laikomas autoritetingu šaltiniu. Svarbiai informacijai rekomenduojama naudoti profesionalų žmogiškąjį vertimą. Mes neatsakome už jokius nesusipratimus ar neteisingą interpretaciją, kilusią naudojantis šiuo vertimu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->