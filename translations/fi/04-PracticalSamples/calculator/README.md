# MCP-laskinopas aloittelijoille

## Sisällys

- [Mitä opit](#mitä-opit)
- [Esivaatimukset](#esivaatimukset)
- [Projektirakenteen ymmärtäminen](#projektirakenteen-ymmäärtäminen)
- [Ydinosa-alueet selitettynä](#ydinosa-alueet-selitettynä)
  - [1. Pääsovellus](#1-pääsovellus)
  - [2. Laskinpalvelu](#2-laskinpalvelu)
  - [3. Suora MCP-asiakas](#3-suora-mcp-asiakas)
  - [4. AI-tehostettu asiakas](#4-ai-tehostettu-asiakas)
- [Esimerkkien suorittaminen](#esimerkkien-suorittaminen)
- [Kuinka kaikki toimii yhdessä](#kuinka-kaikki-toimii-yhdessä)
- [Seuraavat askeleet](#seuraavat-askellet)

## Mitä opit

Tässä oppaassa selitetään, kuinka rakennetaan laskinpalvelu käyttäen Model Context Protocol (MCP) -protokollaa. Ymmärrät:

- Kuinka luodaan palvelu, jota tekoäly voi käyttää työkaluna
- Kuinka määrittää suora yhteys MCP-palveluihin
- Kuinka tekoälymallit voivat automaattisesti valita käytettävät työkalut
- Ero suoran protokollakutsun ja tekoälyn avustamien vuorovaikutusten välillä

## Esivaatimukset

Ennen aloittamista varmista, että sinulla on:
- Java 21 tai uudempi asennettuna
- Maven riippuvuuksien hallintaan
- Azure AI Foundry -mallin käyttöönotto (otetaan käyttöön `azd up` -komennolla — katso [Luku 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) asennettuna ja kirjautuneena `az login` -komennolla (avaimeton tunnistus)
- Perustiedot Javasta ja Spring Bootista

## Projektirakenteen ymmärtäminen

Laskinprojektissa on useita tärkeitä tiedostoja:

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

## Ydinosa-alueet selitettynä

### 1. Pääsovellus

**Tiedosto:** `McpServerApplication.java`

Tämä on laskinpalvelumme käynnistyspiste. Se on tavallinen Spring Boot -sovellus yhdellä erityisellä lisällä:

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
- Käynnistää Spring Boot -verkkopalvelimen portissa 8080
- Luo `ToolCallbackProvider`-komponentin, joka tekee laskimen metodit saataville MCP-työkaluina
- `@Bean`-annotaatio kertoo Springille, että tämä hallitaan komponenttina muiden osien käyttöön

### 2. Laskinpalvelu

**Tiedosto:** `CalculatorService.java`

Täällä tapahtuu kaikki laskenta. Jokainen metodi on merkitty `@Tool`-annotaatiolla, jotta se on saatavilla MCP:n kautta:

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

**Keskeiset ominaisuudet:**

1. **`@Tool`-annotaatio**: Ilmoittaa MCP:lle, että tätä metodia voivat kutsua ulkoiset asiakkaat
2. **Selkeät kuvaukset**: Jokaisella työkalulla on kuvaus, joka auttaa tekoälymalleja ymmärtämään, milloin sitä käytetään
3. **Yhtenäinen palautusmuoto**: Kaikki toiminnot palauttavat ihmisen luettavia merkkijonoja kuten "5.00 + 3.00 = 8.00"
4. **Virheiden käsittely**: Nollalla jakaminen ja negatiiviset neliöjuuret palauttavat virheilmoituksia

**Saatavilla olevat operaatiot:**
- `add(a, b)` - Laskee kahden luvun summan
- `subtract(a, b)` - Vähentää toisen luvun ensimmäisestä
- `multiply(a, b)` - Kertoo kaksi lukua
- `divide(a, b)` - Jakaa ensimmäisen luvun toisella (sisältää nollantarkastuksen)
- `power(base, exponent)` - Korottaa kannan eksponenttiin
- `squareRoot(number)` - Laskee neliöjuuren (tarkistaa negatiivisuuden)
- `modulus(a, b)` - Palauttaa jakolaskun jäännöksen
- `absolute(number)` - Palauttaa itseisarvon
- `help()` - Palauttaa tietoja kaikista toiminnoista

### 3. Suora MCP-asiakas

**Tiedosto:** `SDKClient.java`

Tämä asiakas keskustelee suoraan MCP-palvelimen kanssa ilman tekoälyä. Se kutsuu käsin tiettyjä laskinfunktioita:

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
        
        // Listaa käytettävissä olevat työkalut
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
1. **Yhdistää** laskinpalvelimeen osoitteessa `http://localhost:8080` käyttäen builder-kuviota
2. **Listaa** kaikki saatavilla olevat työkalut (laskimen funktiot)
3. **Kutsuu** tarkkoja funktioita annetuilla parametreilla
4. **Tulostaa** tulokset suoraan

**Huom:** Tämä esimerkki käyttää Spring AI 1.1.0-SNAPSHOT -riippuvuutta, joka toi mukanaan builder-kuvion `WebFluxSseClientTransport`-komponentille. Jos käytät aiempaa vakaata versiota, saatat tarvita suoraa konstruktoria.

**Milloin käyttää:** Kun tiedät tarkasti, minkä laskutoiminnon haluat tehdä ja haluat kutsua sen ohjelmallisesti.

### 4. AI-tehostettu asiakas

**Tiedosto:** `LangChain4jClient.java`

Tämä asiakas käyttää AI-mallia (GPT-4o-mini), joka voi automaattisesti valita, mitä laskintyökaluja käyttää:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Määritä tekoälymalli (Azure AI Foundry, avaimeton todennus Microsoft Entra ID:n kautta)
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

        // Anna tekoälylle pääsy laskintyökaluihimme
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
1. **Luo** tekoälymalliyhteyden avaimettomalla tunnistuksella (Microsoft Entra ID)
2. **Yhdistää** tekoälyn laskin MCP -palvelimeemme
3. **Antaa** tekoälylle pääsyn kaikkiin laskintyökaluihimme
4. **Mahdollistaa** luonnolliskieliset pyynnöt kuten "Laske lukujen 24.5 ja 17.3 summa"

**Tekoäly automaattisesti:**
- Ymmärtää haluat lisätä luvut
- Valitsee `add`-työkalun
- Kutsuu `add(24.5, 17.3)`
- Palauttaa tuloksen luonnollisella vastauksella

## Esimerkkien suorittaminen

### Vaihe 1: Käynnistä laskinpalvelin

Kirjaudu ensin sisään ja aseta Azure AI Foundry -päätepiste (tarvitaan tekoälyasiakkaalle — avaimeton tunnistus, ei API-avainta):

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

Uudessa terminaalissa palvelimen vielä käydessä suorita suora MCP-asiakas:
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

### Vaihe 3: Testaa tekoälyasiakkaalla

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Näet tekoälyn käyttävän automaattisesti työkaluja:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Vaihe 4: Sulje MCP-palvelin

Kun olet valmis testaamaan, voit pysäyttää tekoälyasiakkaan painamalla `Ctrl+C` sen terminaalissa. MCP-palvelin pysyy käynnissä, kunnes pysäytät sen.
Palvelimen pysäyttämiseen paina `Ctrl+C` siinä terminaalissa, jossa se on käynnissä.

## Kuinka kaikki toimii yhdessä

Näin koko prosessi etenee, kun kysyt tekoälyltä "Paljonko on 5 + 3?":

1. **Sinä** kysyt tekoälyltä luonnollisella kielellä
2. **Tekoäly** analysoi pyyntösi ja tunnistaa halun laskea summa
3. **Tekoäly** kutsuu MCP-palvelinta: `add(5.0, 3.0)`
4. **Laskinpalvelu** suorittaa: `5.0 + 3.0 = 8.0`
5. **Laskinpalvelu** palauttaa: `"5.00 + 3.00 = 8.00"`
6. **Tekoäly** vastaanottaa tuloksen ja muotoilee luonnollisen vastauksen
7. **Sinä** saat vastauksen: "Lukujen 5 ja 3 summa on 8"

## Seuraavat askeleet

Lisää esimerkkejä löydät kohdasta [Luku 04: Käytännön esimerkit](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Vastuuvapauslauseke**:
Tämä asiakirja on käännetty käyttämällä tekoälypohjaista käännöspalvelua [Co-op Translator](https://github.com/Azure/co-op-translator). Vaikka pyrimme tarkkuuteen, otathan huomioon, että automaattiset käännökset saattavat sisältää virheitä tai epätarkkuuksia. Alkuperäinen asiakirja sen alkuperäiskielellä on virallinen lähde. Tärkeissä asioissa suositellaan ammattimaista ihmiskäännöstä. Emme ole vastuussa tämän käännöksen käytöstä aiheutuvista väärinymmärryksistä tai tulkinnoista.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->