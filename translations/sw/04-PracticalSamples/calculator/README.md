# Mafunzo ya Kihesabu ya MCP kwa Waanzilishi

## Jedwali la Yaliyomo

- [Utakachojifunza](#utakachojifunza)
- [Mahitaji ya Awali](#mahitaji-ya-awali)
- [Kuelewa Muundo wa Mradi](#kuelewa-muundo-wa-mradi)
- [Vipengele Muhimu Vilivyoelezwa](#vipengele-muhimu-vilivyoelezwa)
  - [1. Programu Kuu](#1-programu-kuu)
  - [2. Huduma ya Kihesabu](#2-huduma-ya-kihesabu)
  - [3. Mteja wa MCP Moja kwa Moja](#3-mteja-wa-mcp-moja-kwa-moja)
  - [4. Mteja Anayezingatia AI](#4-mteja-anayezingatia-ai)
- [Kukimbiza Mifano](#kukimbiza-mifano)
- [Jinsi Yote Hufanya Kazi Pamoja](#jinsi-yote-hufanya-kazi-pamoja)
- [Hatua Zinazofuata](#hatua-zinazofuata)

## Utakachojifunza

Mafunzo haya yanaeleza jinsi ya kujenga huduma ya kihesabu kwa kutumia Itifaki ya Muktadha wa Mfano (MCP). Utaelewa:

- Jinsi ya kuunda huduma ambayo AI inaweza kuitumia kama chombo
- Jinsi ya kuanzisha mawasiliano ya moja kwa moja na huduma za MCP
- Jinsi mifano ya AI inaweza kuchagua moja kwa moja ni zana gani za kutumia
- Tofauti kati ya miito ya itifaki ya moja kwa moja na mwingiliano unaosaidiwa na AI

## Mahitaji ya Awali

Kabla ya kuanza, hakikisha una:
- Java 21 au zaidi imewekwa
- Maven kwa usimamizi wa utegemezi
- Uwekaji wa mfano wa Azure AI Foundry (utengeneze kwa `azd up` — angalia [Sura ya 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), umeingia kwa kutumia `az login` (uthibitishaji bila funguo)
- Uelewa wa msingi wa Java na Spring Boot

## Kuelewa Muundo wa Mradi

Mradi wa kihesabu una faili kadhaa muhimu:

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

## Vipengele Muhimu Vilivyoelezwa

### 1. Programu Kuu

**Faili:** `McpServerApplication.java`

Hii ni sehemu ya kuingia ya huduma yetu ya kihesabu. Ni programu ya kawaida ya Spring Boot na ongezeko moja maalum:

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

**Hii hufanya nini:**
- Huanza seva ya mtandao ya Spring Boot kwenye bandari 8080
- Inaunda `ToolCallbackProvider` inayofanya njia za kihesabu zetu zipatikane kama zana za MCP
- Alama `@Bean` inamwambia Spring kusimamia hii kama sehemu inayoweza kutumika na sehemu zingine

### 2. Huduma ya Kihesabu

**Faili:** `CalculatorService.java`

Hapa ndipo hesabu zote zinafanyika. Kila njia imesanifiwa na `@Tool` ili ifikike kupitia MCP:

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
    
    // Operesheni zaidi za kalkuleta...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Sifa Muhimu:**

1. **Alama ya `@Tool`**: Hii inamwambia MCP kwamba njia hii inaweza kuitwa na wateja wa nje
2. **Maelezo Wazi**: Kila zana ina maelezo yanayosaidia mifano ya AI kuelewa lini ya kuitumia
3. **Mwonekano Thabiti wa Matokeo**: Matendo yote hurudisha herufi zinazoweza kusomeka na binadamu kama "5.00 + 3.00 = 8.00"
4. **Ushughulikiaji wa Makosa**: Mgawanyiko kwa sifuri na mizizi hasi hurudisha ujumbe wa makosa

**Matendo Yanayopatikana:**
- `add(a, b)` - Kuongeza nambari mbili
- `subtract(a, b)` - Kutoa ya pili kutoka ya kwanza
- `multiply(a, b)` - Kuzidisha nambari mbili
- `divide(a, b)` - Kugawanya ya kwanza kwa ya pili (kwa ukaguzi wa sifuri)
- `power(base, exponent)` - Kuweka msingi katika nguvu ya exponent
- `squareRoot(number)` - Kukokotoa mzizi wa mraba (kwa ukaguzi wa hasi)
- `modulus(a, b)` - Kurudisha mabaki ya mgawanyiko
- `absolute(number)` - Kurudisha thamani isiyo na alama
- `help()` - Kutoa taarifa kuhusu matendo yote

### 3. Mteja wa MCP Moja kwa Moja

**Faili:** `SDKClient.java`

Mteja huyu husema moja kwa moja na seva ya MCP bila kutumia AI. Anaita kwa mikono kazi maalum za kihesabu:

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
        
        // Orodha ya zana zilizopatikana
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Piga simu kwa kazi maalum za kalkuleta
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

**Hii hufanya nini:**
1. **Inaunganisha** na seva ya kihesabu kwenye `http://localhost:8080` kwa kutumia muundo wa muundaji
2. **Inaorodhesha** zana zote zinazopatikana (kazi zetu za kihesabu)
3. **Huita** kazi maalum kwa vigezo kamili
4. **Inachapisha** matokeo moja kwa moja

**Kumbuka:** Mfano huu unatumia utegemezi wa Spring AI 1.1.0-SNAPSHOT, ulioleta muundo wa muundaji kwa `WebFluxSseClientTransport`. Ikiwa unatumia toleo la zamani thabiti, huenda ukahitaji kutumia mjenzi wa moja kwa moja badala yake.

**Unapotumia:** Unapojua haswa hesabu gani unayotaka kufanya na unataka kuiita kwa njia ya programu.

### 4. Mteja Anayezingatia AI

**Faili:** `LangChain4jClient.java`

Mteja huyu hutumia mfano wa AI (GPT-4o-mini) ambaye anaweza kuchagua moja kwa moja ni zana gani ya kihesabu kutumia:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Weka mfano wa AI (Azure AI Foundry, uthibitishaji bila funguo kupitia Microsoft Entra ID)
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

        // Unganisha na seva yetu ya kisanduku MCP
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Inaonyesha kile AI inachofanya
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Toa AI ufikiaji kwa zana zetu za kisanduku
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Unda roboti ya AI inayoweza kutumia kisanduku chetu
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Sasa tunaweza kumwomba AI kufanya mahesabu kwa lugha ya asili
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Hii hufanya nini:**
1. **Inaunda** muunganisho wa mfano wa AI kwa kutumia uthibitishaji bila funguo (Microsoft Entra ID)
2. **Inaunganisha** AI na seva yetu ya MCP ya kihesabu
3. **Inampa** AI upatikanaji wa zana zote za kihesabu zetu
4. **Inaruhusu** maombi ya lugha asilia kama "Hesabu jumla ya 24.5 na 17.3"

**AI moja kwa moja:**
- Huelewa unataka kuongeza nambari
- Huchagua zana ya `add`
- Huita `add(24.5, 17.3)`
- Hurudisha matokeo katika jibu la asili

## Kukimbiza Mifano

### Hatua ya 1: Anzisha Seva ya Kihesabu

Kwanza, ingia na weka anwani yako ya Azure AI Foundry (inahitajika kwa mteja wa AI — uthibitishaji bila funguo, bila ufunguo wa API):

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

Anzisha seva:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

Seva itaanza kwenye `http://localhost:8080`. Utauona:
```
Started McpServerApplication in X.XXX seconds
```

### Hatua ya 2: Jaribu na Mteja wa Moja kwa Moja

Katika terminal **MPYA** huku Seva ikiantia kazi, endesha mteja wa MCP wa moja kwa moja:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Utaona matokeo kama:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Hatua ya 3: Jaribu na Mteja wa AI

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Utaona AI ikitumia zana moja kwa moja:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Hatua ya 4: Funga Seva ya MCP

Ukimaliza kupima, unaweza kusitisha mteja wa AI kwa kubonyeza `Ctrl+C` kwenye terminal yake. Seva ya MCP itaendelea kuendesha hadi utakapotaka kuiacha.
Kusitisha seva, bonyeza `Ctrl+C` kwenye terminal inayoendesha.

## Jinsi Yote Hufanya Kazi Pamoja

Hivi ndivyo mtiririko kamili unavyokuwa unapoambia AI "Ni 5 + 3?":

1. **Wewe** unauliza AI kwa lugha ya asili
2. **AI** inachambua ombi lako na kutambua unataka kuongeza
3. **AI** inaita seva ya MCP: `add(5.0, 3.0)`
4. **Huduma ya Kihesabu** inafanya: `5.0 + 3.0 = 8.0`
5. **Huduma ya Kihesabu** hurudisha: `"5.00 + 3.00 = 8.00"`
6. **AI** inapokea matokeo na kuandaa jibu la asili
7. **Wewe** unapata: "Jumla ya 5 na 3 ni 8"

## Hatua Zinazofuata

Kwa mifano zaidi, angalia [Sura ya 04: Sampuli za Vitendo](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Kionyozo**:
Hati hii imetafsiriwa kwa kutumia huduma ya tafsiri ya AI [Co-op Translator](https://github.com/Azure/co-op-translator). Ingawa tunajitahidi kupata usahihi, tafadhali fahamu kwamba tafsiri za kiotomatiki zinaweza kuwa na makosa au upungufu wa usahihi. Hati ya asili katika lugha yake halisi inapaswa kuchukuliwa kama chanzo cha mamlaka. Kwa taarifa muhimu, tafsiri ya kitaalamu inayofanywa na binadamu inapendekezwa. Hatutojibu kwa kuelewa vibaya au tafsiri potofu zinazotokea kutokana na matumizi ya tafsiri hii.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->