# MCP kalkulaatori juhend algajatele

## Sisukord

- [Mida sa õpid](#mida-sa-õpid)
- [Eeldused](#eeldused)
- [Projekti ülesehituse mõistmine](#projekti-ülesehituse-mõistmine)
- [Põhikompontentide selgitus](#põhikompontentide-selgitus)
  - [1. Peamine rakendus](#1-peamine-rakendus)
  - [2. Kalkulaatori teenus](#2-kalkulaatori-teenus)
  - [3. Otsene MCP klient](#3-otsene-mcp-klient)
  - [4. AI-jõul töötav klient](#4-ai-jõul-töötav-klient)
- [Näidete käivitamine](#näidete-käivitamine)
- [Kuidas kõik koos töötab](#kuidas-koik-koos-toob)
- [Järgmised sammud](#järgmised-sammud)

## Mida sa õpid

See juhend seletab, kuidas luua kalkulaatori teenus Model Context Protocol'i (MCP) abil. Sa saad aru:

- Kuidas luua teenust, mida AI saab tööriistana kasutada
- Kuidas seadistada otsene suhtlus MCP teenustega
- Kuidas AI mudelid saavad automaatselt valida, milliseid tööriistu kasutada
- Erinevus otseste protokolli kõnede ja AI-toega interaktsioonide vahel

## Eeldused

Enne alustamist veendu, et sul on olemas:
- Java 21 või uuem versioon
- Maven sõltuvuste halduseks
- Azure AI Foundry mudeli juurutus (provisioni see käsuga `azd up` — vt [2. peatükk](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), sisse logitud käsuga `az login` (võtmeta autentimine)
- Põhilised teadmised Javast ja Spring Bootist

## Projekti ülesehituse mõistmine

Kalkulaatori projektis on mitu olulist faili:

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

## Põhikompontentide selgitus

### 1. Peamine rakendus

**Fail:** `McpServerApplication.java`

See on meie kalkulaatori teenuse stardipunkt. Tegemist on tavalise Spring Boot rakendusega, millel on üks eriline täiendus:

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

**Mida see teeb:**
- Käivitab Spring Boot veebi serveri pordil 8080
- Loob `ToolCallbackProvider`-i, mis teeb meie kalkulaatori meetodid kättesaadavaks MCP tööriistadena
- Märgend `@Bean` ütleb Springile, et see on komponent, mida teised osad saavad kasutada

### 2. Kalkulaatori teenus

**Fail:** `CalculatorService.java`

Siin toimub kogu matemaatika. Iga meetod on märgistatud `@Tool`-iga, et see oleks MCP kaudu kättesaadav:

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
    
    // Rohkem kalkulaatori toiminguid...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Peamised omadused:**

1. **`@Tool` märgend**: See ütleb MCP-le, et seda meetodit saavad välised kliendid kutsuda
2. **Selged kirjeldused**: Igal tööriistal on kirjeldus, mis aitab AI mudelitel aru saada, millal seda kasutada
3. **Järjekindel vastuse formaat**: Kõik tehteid tagastavad inimese jaoks loetavat teksti, nt "5.00 + 3.00 = 8.00"
4. **Vigade käsitlemine**: Nulliga jagamine ja negatiivse ruutjuure puhul tagastatakse veateated

**Saadaval toimingud:**
- `add(a, b)` - Liidab kaks arvu
- `subtract(a, b)` - Lahutab teisest esimesest
- `multiply(a, b)` - Korrutab kaks arvu
- `divide(a, b)` - Jagab esimest teisega (nulli kontrolliga)
- `power(base, exponent)` - Tõstab aluse astmesse
- `squareRoot(number)` - Arvutab ruutjuure (negatiivsuse kontrolliga)
- `modulus(a, b)` - Tagastab jagamisjäägi
- `absolute(number)` - Tagastab absoluutväärtuse
- `help()` - Tagastab info kõigi toimingute kohta

### 3. Otsene MCP klient

**Fail:** `SDKClient.java`

See klient suhtleb MCP serveriga otse ilma AI-ta. See kutsub käsitsi spetsiifilisi kalkulaatori funktsioone:

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
        
        // Loetle saadaolevad tööriistad
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Kutsu spetsiifilisi kalkulaatori funktsioone
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

**Mida see teeb:**
1. **Ühendub** kalkulaatori serveriga aadressil `http://localhost:8080` kasutades builder musterit
2. **Nimekirjab** kõik saadaval olevad tööriistad (meie kalkulaatorifunktsioonid)
3. **Kutsutakse** spetsiifilisi funktsioone täpsete parameetritega
4. **Trükib** tulemused otse välja

**Märkus:** See näide kasutab Spring AI 1.1.0-SNAPSHOT sõltuvust, mis tõi sisse builder mustri `WebFluxSseClientTransport` jaoks. Kui kasutad vanemat stabiilset versiooni, võid vajada hoopis otseteed konstruktorit.

**Millal seda kasutada:** Kui tead täpselt, millist arvutust soovid teha, ja tahad seda programmiliselt kutsuda.

### 4. AI-jõul töötav klient

**Fail:** `LangChain4jClient.java`

See klient kasutab AI mudelit (GPT-4o-mini), mis suudab automaatselt otsustada, milliseid kalkulaatori tööriistu kasutada:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Seadista tehisintellekti mudel (Azure AI Foundry, võtmeta autentimine Microsoft Entra ID kaudu)
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

        // Ühenda meie kalkulaatori MCP serveriga
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Kuvab, mida tehisintellekt teeb
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Anna tehisintellektile ligipääs meie kalkulaatori tööriistadele
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Loo tehisintellekti robot, kes saab kasutada meie kalkulaatorit
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Nüüd saame paluda tehisintellektil teha arvutusi loomulikus keeles
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Mida see teeb:**
1. **Loob** AI mudeliga ühenduse kasutades sinu GitHub tokenit
2. **Ühendab** AI meie kalkulaatori MCP serveriga
3. **Annavad** AI-le ligipääsu kõikidele meie kalkulaatori tööriistadele
4. **Lubab** teha loomuliku keelega päringuid nagu "Arvuta 24.5 ja 17.3 summa"

**AI teeb automaatselt:**
- Mõistab, et sa tahad numbreid liita
- Valib tööriista `add`
- Kutsub `add(24.5, 17.3)`
- Tagastab tulemuse loomulikus vastuses

## Näidete käivitamine

### Samm 1: Alusta kalkulaatori serverit

Esmalt logi sisse ja määra oma Azure AI Foundry lõpp-punkt (vajalik AI kliendile — võtmeta autentimine, ilma API võtmeta):

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

Käivita server:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

Server käivitub aadressil `http://localhost:8080`. Sa peaksid nägema:
```
Started McpServerApplication in X.XXX seconds
```

### Samm 2: Testi otsese kliendiga

Uues terminaliaknas, kus server veel töötab, käivita otsene MCP klient:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Sa näed sellist väljundit:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Samm 3: Testi AI kliendiga

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Sa näed, kuidas AI automaatselt tööriistu kasutab:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Samm 4: Sule MCP server

Kui oled testi lõpetanud, saad AI kliendi peatada vajutades `Ctrl+C` selles terminalis. MCP server jätkab tööd seni, kuni sa selle ise peatud.
Serveri peatamiseks vajuta `Ctrl+C` terminalis, kus ta töötab.

## Kuidas kõik koos töötab

Siin on kogu töövoog, kui küsid AI-lt "Mis on 5 + 3?":

1. **Sina** esitad AI-le loomulikus keeles küsimuse
2. **AI** analüüsib päringut ja mõistab, et soovid liitmist
3. **AI** kutsub MCP serverit: `add(5.0, 3.0)`
4. **Kalkulaatori teenus** teostab: `5.0 + 3.0 = 8.0`
5. **Kalkulaatori teenus** tagastab: `"5.00 + 3.00 = 8.00"`
6. **AI** saab tulemuse ja vormindab loomuliku vastuse
7. **Sina** saad: "Arvude 5 ja 3 summa on 8"

## Järgmised sammud

Rohkem näidete jaoks vaata [Peatükk 04: Praktilised näited](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Lahtiütlus**:
See dokument on tõlgitud kasutades AI tõlketeenust [Co-op Translator](https://github.com/Azure/co-op-translator). Kuigi me püüdleme täpsuse poole, palun pange tähele, et automatiseeritud tõlgetes võib esineda vigu või ebatäpsusi. Originaaldokument selle emakeeles tuleks pidada autoriteetseks allikaks. Olulise teabe puhul soovitatakse kasutada professionaalset inimtõlget. Me ei vastuta selle tõlkega seotud eksimustest või valesti mõistmistest.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->