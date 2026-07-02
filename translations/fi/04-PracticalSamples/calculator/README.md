# MCP-laskinopas aloittelijoille

## Sisällysluettelo

- [Mitä opit](#mitä-opit)
- [Esivaatimukset](#esivaatimukset)
- [Projektin rakenteen ymmärtäminen](#projektin-rakenteen-ymmärrys)
- [Keskeiset osat selitettynä](#keskeiset-osat-selitettynä)
  - [1. Pääsovellus](#1-pääsovellus)
  - [2. Laskinpalvelu](#2-laskinpalvelu)
  - [3. Suora MCP-asiakas](#3-suora-mcp-asiakas)
  - [4. AI-voimainen asiakas](#4-ai-voimainen-asiakas)
- [Esimerkkien ajaminen](#esimerkkien-ajaminen)
- [Miten kaikki toimii yhdessä](#miten-kaikki-toimii-yhdessä)
- [Seuraavat askeleet](#seuraavat-askelmat)

## Mitä opit

Tässä oppaassa selitetään, miten rakennetaan laskinpalvelu käyttäen Model Context Protocolia (MCP). Ymmärrät:

- Miten luodaan palvelu, jota AI voi käyttää työkaluna
- Miten asetetaan suora yhteys MCP-palveluihin
- Miten AI-mallit voivat automaattisesti valita käytettävät työkalut
- Ero suoran protokollakutsun ja AI-avusteisen vuorovaikutuksen välillä

## Esivaatimukset

Ennen aloittamista varmista, että sinulla on:
- Asennettuna Java 21 tai uudempi
- Maven riippuvuuksien hallintaan
- Azure AI Foundry -mallivienti (perusta `azd up` -komennolla — katso [Luku 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) asennettuna, ja kirjauduttu `az login` (avainvapaa autentikointi)
- Perustiedot Javasta ja Spring Bootista

## Projektin rakenteen ymmärrys

Laskinprojekti sisältää useita tärkeitä tiedostoja:

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

## Keskeiset osat selitettynä

### 1. Pääsovellus

**Tiedosto:** `McpServerApplication.java`

Tämä on laskinpalvelumme käynnistyspiste. Kyseessä on tavallinen Spring Boot -sovellus yhdellä erityisellä lisällä:

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

**Mitä tämä tekee:**
- Käynnistää Spring Boot -web-palvelimen porttiin 8080
- Luo `ToolCallbackProvider`-komponentin, joka tekee laskimen metodit saataville MCP-työkaluina
- `@Bean`-annotaatio kertoo Springille, että tämä hallitaan komponenttina, jota muut osat voivat käyttää

### 2. Laskinpalvelu

**Tiedosto:** `CalculatorService.java`

Täällä tapahtuvat kaikki laskutoimitukset. Jokainen metodi on merkitty `@Tool`-annotaatiolla, jotta ne ovat käytettävissä MCP:n kautta:

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
    
    // Lisää laskimen toimintoja...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Tärkeimmät ominaisuudet:**

1. **`@Tool`-annotaatio**: Ilmoittaa MCP:lle, että tätä metodia voivat kutsua ulkoiset asiakkaat
2. **Selkeät kuvaukset**: Jokaisella työkalulla on kuvaus, joka auttaa AI-malleja ymmärtämään, milloin sitä käytetään
3. **Yhtenäinen palautusmuoto**: Kaikki operaatiot palauttavat ihmisenlukuisia merkkijonoja, kuten "5.00 + 3.00 = 8.00"
4. **Virheenkäsittely**: Nollalla jakaminen ja negatiiviset neliöjuuret palauttavat virheilmoituksia

**Saatavilla olevat toiminnot:**
- `add(a, b)` - Laskee kahden luvun summan
- `subtract(a, b)` - Vähentää toisen luvun ensimmäisestä
- `multiply(a, b)` - Kertoo kaksi lukua
- `divide(a, b)` - Jakaa ensimmäisen luvun toisella (tarkistaa nollalla jakamisen)
- `power(base, exponent)` - Korottaa kantaluvun eksponentin potenssiin
- `squareRoot(number)` - Laskee neliöjuuren (tarkistaa negatiivisuuden)
- `modulus(a, b)` - Palauttaa jakamisen jäännöksen
- `absolute(number)` - Palauttaa luvun itseisarvon
- `help()` - Palauttaa tietoa kaikista toiminnoista

### 3. Suora MCP-asiakas

**Tiedosto:** `SDKClient.java`

Tämä asiakas keskustelee suoraan MCP-palvelimen kanssa ilman AI:tä. Se kutsuu manuaalisesti tiettyjä laskimen funktioita:

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
        
        // Listaa saatavilla olevat työkalut
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Kutsu tiettyjä laskimen toimintoja
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

**Mitä tämä tekee:**
1. **Yhdistää** laskinpalvelimeen osoitteessa `http://localhost:8080` käyttäen rakennusmallia
2. **Listaa** kaikki saatavilla olevat työkalut (laskimen funktiot)
3. **Kutsuu** tiettyjä funktioita tarkkojen parametrien kanssa
4. **Tulostaa** tulokset suoraan

**Huom:** Tämä esimerkki käyttää Spring AI 1.1.0-SNAPSHOT -riippuvuutta, joka toi rakennusmallin `WebFluxSseClientTransport`-luokkaan. Jos käytät vanhempaa vakaata versiota, saatat joutua käyttämään suoraa konstruktoria.

**Mihin käyttää:** Kun tiedät täsmälleen, mitä laskutoimitusta haluat tehdä ja haluat kutsua sen ohjelmallisesti.

### 4. AI-voimainen asiakas

**Tiedosto:** `LangChain4jClient.java`

Tämä asiakas käyttää AI-mallia (GPT-4o-mini), joka osaa automaattisesti päättää, mitä laskintyökaluja käyttää:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Määritä tekoälymalli (Azure AI Foundry, avaimeton tunnistus Microsoft Entra ID:n kautta)
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

        // Yhdistä laskin MCP -palvelimeemme
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Näyttää, mitä tekoäly tekee
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Anna tekoälylle pääsy laskentatyökaluihimme
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Luo tekoälybotti, joka voi käyttää laskintamme
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Nyt voimme pyytää tekoälyä tekemään laskutoimituksia luonnollisella kielellä
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Mitä tämä tekee:**
1. **Luo** AI-mallin yhteyden GitHub-tunnisteesi avulla
2. **Yhdistää** AI:n laskin-MCP-palvelimeemme
3. **Antaa** AI:lle pääsyn kaikkiin laskintyökaluihimme
4. **Mahdollistaa** luonnollisen kielen pyynnöt, kuten "Laske lukujen 24,5 ja 17,3 summa"

**AI tekee automaattisesti:**
- Ymmärtää, että haluat laskea summan
- Valitsee `add`-työkalun
- Kutsuu `add(24.5, 17.3)`
- Palauttaa tuloksen luonnollisessa vastauksessa

## Esimerkkien ajaminen

### Vaihe 1: Käynnistä laskinpalvelin

Kirjaudu ensin sisään ja määritä Azure AI Foundry -päätepiste (tarvitaan AI-asiakkaalle — avainvapaa autentikointi):

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

Käynnistä palvelin:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

Palvelin käynnistyy osoitteeseen `http://localhost:8080`. Näet:
```
Started McpServerApplication in X.XXX seconds
```

### Vaihe 2: Testaa suoralla asiakkaalla

Uudessa terminaalissa palvelimen ollessa käynnissä, suorita suora MCP-asiakas:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Näet tulosteen kuten:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Vaihe 3: Testaa AI-asiakkaalla

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Näet AI:n automaattisesti käyttävän työkaluja:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Vaihe 4: Sulje MCP-palvelin

Kun olet valmis testaamaan, voit pysäyttää AI-asiakkaan painamalla `Ctrl+C` sen terminaalissa. MCP-palvelin jatkaa käynnissä, kunnes pysäytät sen.
Pysäytä palvelin painamalla `Ctrl+C` siinä terminaalissa, jossa se on käynnistetty.

## Miten kaikki toimii yhdessä

Tässä on kokonaisprosessi, kun kysyt AI:lta "Paljonko on 5 + 3?":

1. **Sinä** kysyt AI:ta luonnollisella kielellä
2. **AI** analysoi pyyntösi ja tajuaa, että haluat laskea yhteen
3. **AI** kutsuu MCP-palvelinta: `add(5.0, 3.0)`
4. **Laskinpalvelu** suorittaa laskun: `5.0 + 3.0 = 8.0`
5. **Laskinpalvelu** palauttaa: `"5.00 + 3.00 = 8.00"`
6. **AI** vastaanottaa tuloksen ja muodostaa luonnollisen vastauksen
7. **Sinä** saat vastauksen: "Lukujen 5 ja 3 summa on 8"

## Seuraavat askeleet

Lisää esimerkkejä löydät [Luvusta 04: Käytännön esimerkit](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Vastuuvapauslauseke**:
Tämä asiakirja on käännetty käyttämällä tekoälypohjaista käännöspalvelua [Co-op Translator](https://github.com/Azure/co-op-translator). Vaikka pyrimme tarkkuuteen, otathan huomioon, että automaattiset käännökset saattavat sisältää virheitä tai epätarkkuuksia. Alkuperäinen asiakirja sen alkuperäiskielellä on virallinen lähde. Tärkeissä asioissa suositellaan ammattimaista ihmiskäännöstä. Emme ole vastuussa tämän käännöksen käytöstä aiheutuvista väärinymmärryksistä tai tulkinnoista.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->