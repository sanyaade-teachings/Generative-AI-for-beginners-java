# MCP Calculator Tutorial kwa Waanzilishi

## Meza ya Yaliyomo

- [Unachojifunza](#unachojifunza)
- [Mahitaji ya Awali](#mahitaji-ya-awali)
- [Kuelewa Muundo wa Mradi](#kuelewa-muundo-wa-mradi)
- [Vipengele Muhimu Vilivyoelezewa](#vipengele-muhimu-vilivyoelezewa)
  - [1. Programu Kuu](#1-programu-kuu)
  - [2. Huduma ya Kihesabu](#2-huduma-ya-kihesabu)
  - [3. Mteja wa MCP Mtandaoni](#3-mteja-wa-mcp-mtandaoni)
  - [4. Mteja Aliyetumia AI](#4-mteja-aliyetumia-ai)
- [Kukimbia Mifano](#kukimbia-mifano)
- [Jinsi Yote Hufanya Kazi Pamoja](#jinsi-yote-hufanya-kazi-pamoja)
- [Hatua Zinazofuata](#hatua-zinazofuata)

## Unachojifunza

Mafunzo haya yanaeleza jinsi ya kujenga huduma ya kihesabu kwa kutumia Model Context Protocol (MCP). Utaunda uelewa wa:

- Jinsi ya kuunda huduma ambayo AI inaweza kuitumia kama chombo
- Jinsi ya kuanzisha mawasiliano ya moja kwa moja na huduma za MCP
- Jinsi mifano ya AI inaweza kuchagua moja kwa moja ni vifaa gani vitumike
- Tofauti kati ya simu za moja kwa moja za itifaki na mwingiliano wenye msaada wa AI

## Mahitaji ya Awali

Kabla ya kuanza, hakikisha una:
- Java 21 au zaidi imewekwa
- Maven kwa usimamizi wa utegemezi
- Uwekaji wa mfano wa Azure AI Foundry (uwekee kwa `azd up` — angalia [Sura ya 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), umeingia kwa `az login` (uthibitisho bila funguo)
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

## Vipengele Muhimu Vilivyoelezewa

### 1. Programu Kuu

**Faili:** `McpServerApplication.java`

Huu ni mlango wa kuingilia wa huduma yetu ya kihesabu. Ni programu ya kawaida ya Spring Boot yenye nyongeza moja maalum:

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
- Inaendesha seva ya wavuti ya Spring Boot kwenye bandari 8080
- Inaunda `ToolCallbackProvider` inayofanya mbinu zetu za kihesabu zipatikane kama zana za MCP
- Kielezi `@Bean` kinaambia Spring kusimamia kama sehemu ambayo sehemu zingine zinaweza kuitumia

### 2. Huduma ya Kihesabu

**Faili:** `CalculatorService.java`

Hapa ndipo hesabu zote hufanyika. Kila njia imeandikwa kwa `@Tool` ili iweze kupatikana kupitia MCP:

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

**Sifa kuu:**

1. **Kielezi cha `@Tool`**: Kinaonyesha MCP kwamba njia hii inaweza kuitwa na wateja wa nje
2. **Maelezo ya Wazi**: Kila chombo kinaelezo kinachosaidia mifano ya AI kuelewa lini kiitumie
3. **Muundo Thabiti wa Kurudisha**: Operesheni zote hurudisha mistari inayoweza kusomwa binadamu kama "5.00 + 3.00 = 8.00"
4. **Shughulikiaji la Makosa**: Kugawanya kwa sifuri na mizizi hasi hurudisha ujumbe wa kosa

**Operesheni Zinazopatikana:**
- `add(a, b)` - Huongeza nambari mbili
- `subtract(a, b)` - Hukata nambari ya pili kutoka ya kwanza
- `multiply(a, b)` - Huzidisha nambari mbili
- `divide(a, b)` - Hugaisha ya kwanza kwa ya pili (kwa ukaguzi wa sifuri)
- `power(base, exponent)` - Huinua msingi kwa nguvu ya exponent
- `squareRoot(number)` - Hukokotoa mzizi wa mraba (kwa ukaguzi hasi)
- `modulus(a, b)` - Hurudisha jinsi ya kuachwa kwa mgawanyo
- `absolute(number)` - Hurudisha thamani kamili
- `help()` - Hurudisha habari kuhusu operesheni zote

### 3. Mteja wa MCP Mtandaoni

**Faili:** `SDKClient.java`

Mteja huyu husalimiana moja kwa moja na seva ya MCP bila kutumia AI. Huitisha kazi maalum za kihesabu kwa mikono:

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
        
        // Orodhesha zana zinazopatikana
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Piga simu kwa kazi maalum za kifaa cha kuhesabu
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
1. **Inaunganisha** na seva ya kihesabu kwenye `http://localhost:8080` kwa kutumia muundo wa builder
2. **Inaorodhesha** zana zote zinazopatikana (kazi zetu za kihesabu)
3. **Huitisha** kazi maalum kwa vigezo halisi
4. **Inachapisha** matokeo moja kwa moja

**Kumbuka:** Mfano huu hutumia utegemezi wa Spring AI 1.1.0-SNAPSHOT, ulioweka muundo wa builder kwa `WebFluxSseClientTransport`. Ikiwa unatumia toleo la zamani linalo thibitishwa, unaweza kuhitaji kutumia muundaji wa moja kwa moja badala yake.

**Lini kutumia hii:** Unapojua hasa hesabu unayotaka kufanya na unataka kuitisha programmatically.

### 4. Mteja Aliyetumia AI

**Faili:** `LangChain4jClient.java`

Mteja huyu hutumia mfano wa AI (GPT-4o-mini) ambaye anaweza moja kwa moja kuamua ni zana gani za kihesabu zitumike:

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

        // Unganisha na seva yetu ya kipimo MCP
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Inaonyesha kile AI inachofanya
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Mpa AI ufikiaji wa zana zetu za kipimo
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Unda roboti ya AI inayoweza kutumia kipimo chetu
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Sasa tunaweza kumuomba AI kufanya mahesabu kwa lugha asilia
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Hii hufanya nini:**
1. **Hutangaza** muunganiko wa mfano wa AI kwa kutumia tokeni yako ya GitHub
2. **Inaunganisha** AI na seva yetu ya MCP ya kihesabu
3. **Inampa** AI ufikiaji wa zana zote za kihesabu zetu
4. **Inaruhusu** maombi kwa lugha ya asili kama "Hesabu jumla ya 24.5 na 17.3"

**AI hufanya moja kwa moja:**
- Huelewa unataka kuongeza nambari
- Huchagua chombo cha `add`
- Huitisha `add(24.5, 17.3)`
- Hurudisha matokeo kama jibu la asili

## Kukimbia Mifano

### Hatua ya 1: Anzisha Seva ya Kihesabu

Kwanza, ingia na weka anwani yako ya Azure AI Foundry (inahitajika kwa mteja wa AI — uthibitisho bila funguo, hakuna funguo za API):

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

### Hatua ya 2: Jaribu kwa Mteja wa Moja kwa Moja

Katika terminal **MPYA** na Seva bado ikiwa inakimbia, endesha mteja wa MCP wa moja kwa moja:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Utaona output kama:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Hatua ya 3: Jaribu kwa Mteja wa AI

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Utaona AI ikitumia zana moja kwa moja:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Hatua ya 4: Funga Seva ya MCP

Unapomaliza kujaribu, unaweza kusitisha mteja wa AI kwa kubonyeza `Ctrl+C` kwenye terminal yake. Seva ya MCP itaendelea kazi mpaka usitishe wewe.
Kusitisha seva, bonyeza `Ctrl+C` kwenye terminal inayoendesha.

## Jinsi Yote Hufanya Kazi Pamoja

Hii ni mfululizo kamili unapomuuliza AI "Ni 5 + 3 gani?":

1. **Wewe** unauliza AI kwa lugha ya asili
2. **AI** inachambua ombi lako na kugundua unataka kuongeza
3. **AI** huita seva ya MCP: `add(5.0, 3.0)`
4. **Huduma ya Kihesabu** hufanya: `5.0 + 3.0 = 8.0`
5. **Huduma ya Kihesabu** hurudisha: `"5.00 + 3.00 = 8.00"`
6. **AI** hupokea matokeo na kutengeneza jibu la kawaida
7. **Wewe** unapata: "Jumla ya 5 na 3 ni 8"

## Hatua Zinazofuata

Kwa mifano zaidi, angalia [Sura 04: Sampuli za Vitendo](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Kionyozo**:
Hati hii imetafsiriwa kwa kutumia huduma ya tafsiri ya AI [Co-op Translator](https://github.com/Azure/co-op-translator). Ingawa tunajitahidi kupata usahihi, tafadhali fahamu kwamba tafsiri za kiotomatiki zinaweza kuwa na makosa au upungufu wa usahihi. Hati ya asili katika lugha yake halisi inapaswa kuchukuliwa kama chanzo cha mamlaka. Kwa taarifa muhimu, tafsiri ya kitaalamu inayofanywa na binadamu inapendekezwa. Hatutojibu kwa kuelewa vibaya au tafsiri potofu zinazotokea kutokana na matumizi ya tafsiri hii.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->