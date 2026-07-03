# စတင်လေ့လာသူများအတွက် MCP တွက်ချက်စက် ပညာပေးလမ်းညွှန်

## အကြောင်းအရာဇယား

- [အလိုက်သင်ယူမည့်အချက်များ](#အလိုက်သင်ယူမည့်အချက်များ)
- [မတိုင်မီရှိရန်လိုအပ်ချက်များ](#မတိုင်မီရှိရန်လိုအပ်ချက်များ)
- [ပရောဂျက်ဖွဲ့စည်းမှုကိုနားလည်ခြင်း](#ပရောဂျက်-ဖွဲ့စည်းမှုကို-နားလည်ခြင်း)
- [အဓိကအစိတ်အပိုင်းများ ရှင်းလင်းချက်](#အဓိကအစိတ်အပိုင်းများ-ရှင်းလင်းချက်)
  - [1. မူလအက်ပလီကေးရှင်း](#1-မူလအက်ပလီကေးရှင်း)
  - [2. တွက်ချက်စက် ဝန်ဆောင်မှု](#2-တွက်ချက်စက်-ဝန်ဆောင်မှု)
  - [3. တိုက်ရိုက် MCP Client](#3-တိုက်ရိုက်-mcp-client)
  - [4. AI ဖြင့်စွမ်းအားမြှင့် Client](#4-ai-ဖြင့်စွမ်းအားမြှင့်-client)
- [နမူနာများကို လုပ်ဆောင်ခြင်း](#နမူနာများကို-လုပ်ဆောင်ခြင်း)
- [အားလုံး ဘယ်လိုအတူတူ လုပ်ဆောင်ကြပုံ](#အားလုံး-ဘယ်လိုအတူတူ-လုပ်ဆောင်ကြပုံ)
- [နောက်တစ်ဆင့်များ](#နောက်တစ်ဆင့်များ)

## အလိုက်သင်ယူမည့်အချက်များ

ဤသင်ခန်းစာသည် Model Context Protocol (MCP) ကို အသုံးပြုသော တွက်ချက်စက်ဝန်ဆောင်မှု တည်ဆောက်နည်းကို ရှင်းပြသည်။ သင်သိရှိမည်မှာ -

- AI ကိုကိရိယာအဖြစ် အသုံးပြုနိုင်သော ဝန်ဆောင်မှုတစ်ခု တည်ဆောက်နည်း
- MCP ဝန်ဆောင်မှုများနှင့် တိုက်ရိုက်ဆက်သွယ်မှု ဆောက်လုပ်နည်း
- AI မော်ဒယ်များနှင့် ကိရိယာများကို မည်သို့အလိုအလျောက် ရွေးချယ်သုံးစွဲနိုင်သည်
- တိုက်ရိုက် ပရိုတိုကောခေါ်ဆိုမှုများနှင့် AI အကူအညီဖြင့် ဆက်သွယ်ပုံများ၏ ကွာခြားချက်

## မတိုင်မီရှိရန်လိုအပ်ချက်များ

စတင်မလုပ်ဆောင်မီ ကောက်ချုပ်ထားရန် -

- Java 21 သို့မဟုတ် အထက်တွင် ထည့်သွင်းထားရန်
- Maven ကို ပါဝင်မှုစီမံခန့်ခွဲမှုအတွက်
- Azure AI Foundry မော်ဒယ်တင်သွင်းမှု (azd up ဖြင့် စတင်ပါ — [အခန်း ၂](../../02-SetupDevEnvironment/getting-started-azure-openai.md) တွင် ကြည့်ရှုနိုင်သည်)
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) ကို az login ဖြင့် လက်မှတ်ထိုးထားပြီး (keyless အတည်ပြုမှု)
- Java နှင့် Spring Boot အကြောင်းအရင်းအမြစ်အနေဖြင့် ရှင်းလင်းတတ်မှု

## ပရောဂျက် ဖွဲ့စည်းမှုကို နားလည်ခြင်း

တွက်ချက်စက် ပရောဂျက်တွင် အရေးကြီးဖိုင်များ အများကြီး ပါဝင်သည် -

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

## အဓိကအစိတ်အပိုင်းများ ရှင်းလင်းချက်

### 1. မူလအက်ပလီကေးရှင်း

**ဖိုင်:** `McpServerApplication.java`

ဤသည်မှာ ကျွန်တော်တို့၏ တွက်ချက်စက် ဝန်ဆောင်မှု၏ စတင်ဝင်ရောက်ရာနေရာဖြစ်သည်။ ယင်းမှာ ပုံမှန် Spring Boot အက်ပလီကေးရှင်းတစ်ခုဖြစ်ပြီး အထူးအစိတ်အပိုင်းတစ်ခု ပါဝင်သည် -

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

**လုပ်ဆောင်ပုံ**

- 8080 ပို့ အပေါ် Spring Boot web server ကို စတင်
- ကျွန်တော်တို့၏ တွက်ချက်စက် နည်းလမ်းများကို MCP ကိရိယာများအဖြစ် အသုံးပြုနိုင်ရန် ToolCallbackProvider တည်ဆောက်
- `@Bean` ဆိုသည်မှာ Spring သည် အစိတ်အပိုင်းအဖြစ် ထိန်းချုပ်ရန် ထားရှိသည်ကို ပြောဆိုသည်

### 2. တွက်ချက်စက် ဝန်ဆောင်မှု

**ဖိုင်:** `CalculatorService.java`

ဒီမှာ သင်္ချာဆိုင်ရာ လုပ်ဆောင်ပုံများ အားလုံး ဖြစ်ပေါ်နေပါသည်။ နည်းလမ်းတိုင်းမှာ `@Tool` ဖြင့် အမှတ်အသားပြုထားပြီး MCP မှတဆင့် ခေါ်ယူနိုင်သည်။

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
    
    // ကိန်းဂဏန်းရေးစက်၏ လုပ်ဆောင်ချက်များ ပိုမိုများပြားသည်...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**အဓိက ထူးခြားချက်များ**

1. **`@Tool` အမှတ်အသား** - MCP မှ ပြင်ပ Client များမှ ခေါ်နိုင်ကြောင်း ဖော်ပြသည်
2. **ရှင်းလင်းသော ဖော်ပြချက်များ** - AI မော်ဒယ်များ အတွက် အသုံးပြုရန် အခါကို နားလည်စေရန် ဖော်ပြချက်များပါရှိသည်
3. **တစ်နေရာတည်း ပြန်လည်ဖြေဆိုမှု ပုံစံ** - "5.00 + 3.00 = 8.00" ကဲ့သို့ လူသိရှင်းသော စာသားတစ်ခု ဖြင့် ပြန်တမ်း ပေးသည်
4. **အမှားစွဲချက်များ ကိုင်တွယ်မှု** - သုညဖြင့်ပိုင်းခြားခြင်းနှင့် မကျေမှုရှိသော square root တွင် အမှား သတိပေးစာများ ပြန်လည်ပေးသည်

**လက်ရှိ လုပ်ဆောင်မှုများ**
- `add(a, b)` - နံပါတ်နှစ်ခုတိုးခြင်း
- `subtract(a, b)` - ပထမနံပါတ်မှ ဒုတိယနံပါတ်ဖြုတ်ခြင်း
- `multiply(a, b)` - နံပါတ်နှစ်ခုမြှောက်တိုးခြင်း
- `divide(a, b)` - ပထမနံပါတ်ကို ဒုတိယနံပါတ်ဖြင့် ခွဲခြမ်းခြင်း (သုည စစ်ဆေးမှုပါ)
- `power(base, exponent)` - အခြေသို့ အမြတ် အရာ မြှောက်ခြင်း
- `squareRoot(number)` - စတုရန်း ရှုပ်ထွေးချက်တွက်ချက်မှု (မကျေမှု စစ်ဆေးမှုပါ)
- `modulus(a, b)` - ခွဲခြမ်းပြီး ကျန်ကြွင်းကို ပေးခြင်း
- `absolute(number)` - တန်ဖိုး၏ သန့်ရှင်းမှု
- `help()` - လုပ်ဆောင်မှုပေါင်းစုံအကြောင်း ရှင်းလင်းချက် ထုတ်ပြန်ခြင်း

### 3. တိုက်ရိုက် MCP Client

**ဖိုင်:** `SDKClient.java`

ဤ Client သည် AI မသုံးဘဲ MCP Server သို့ တိုက်ရိုက် ဆက်သွယ်ပြီး တွက်ချက်စက် လုပ်ဆောင်ချက်အတိအကျ ခေါ်ဆိုပေးသည်။

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
        
        // ရနိုင်သောကိရိယာများစာရင်းပြပါ
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // သီးသန့်တွက်ချက်စက်လုပ်ဆောင်ချက်များခေါ်ဆိုပါ
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

**လုပ်ဆောင်ချက်များ**

1. Builder ပုံစံဖြင့် `http://localhost:8080` တွင်ရှိသော တွက်ချက်စက် ဆာဗာနှင့် ဆက်သွယ်ခြင်း
2. ရရှိနိုင်သည့် ကိရိယာများအားလုံးကို စာရင်းပြုစုခြင်း
3. တိကျသည့် ပရာမီတာများဖြင့် လုပ်ဆောင်ချက်များ ခေါ်ယူခြင်း
4. ရလဒ်များကို တိုက်ရိုက် ပုံနှိပ်ပြသခြင်း

**မှတ်ချက်** - ဤနမူနာ သည် Spring AI 1.1.0-SNAPSHOT အား အသုံးပြုထားပြီး `WebFluxSseClientTransport` အတွက် builder ပုံစံအတွက် အသစ်ထည့်သွင်းထားသည်။ ရှေးအင်္ဂါရပ်မှာ တိုက်ရိုက် constructor သုံးရပါမည်။

**ဘယ်အခါအသုံးပြုမလဲ** - ခန့်မှန်းချက်တိကျသော တွက်ချက်မှုကို အလိုအလျောက် မသုံးသလို ကိုယ်တိုင် အစီအစဉ်ထားခေါ်ယူချင်သောအခါ။

### 4. AI ဖြင့်စွမ်းအားမြှင့် Client

**ဖိုင်:** `LangChain4jClient.java`

ဤ Client သည် GPT-4o-mini ဆိုသော AI မော်ဒယ်ကို အသုံးပြုပြီး ဤကိရိယာများကို အလိုအလျောက် ရွေးချယ်အသုံးပြုနိုင်သည်။

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // AI မော်ဒယ်ကို စီမံထားခြင်း (Azure AI Foundry, Microsoft Entra ID မှ keyless authentication ဖြင့်)
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

        // ကျွန်တော်တို့ရဲ့ calculator MCP ဆာဗာကို ချိတ်ဆက်မည်
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // AI က ဘာလုပ်နေသည်ကို ပြသသည်
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // AI ကို ကျွန်တော်တို့ calculator ကိရိယာများသို့ ဝင်ရောက်ခွင့်ပြုရန်
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // ကျွန်တော်တို့ calculator ကို အသုံးပြုနိုင်သော AI bot တစ်ခု ဖန်တီးခြင်း
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // ယခု AI ကို သဘာဝဘာသာဖြင့် တွက်ချက်စေလိုနိုင်ပြီဖြစ်သည်
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**လုပ်ဆောင်ချက်များ**

1. Keyless Authentication (Microsoft Entra ID) အသုံးပြုပြီး AI မော်ဒယ် ဆက်သွယ်မှု တည်ဆောက်ခြင်း
2. AI ကို ကျွန်တော်တို့၏ MCP တွက်ချက်စက် ဆာဗာနှင့် ချိတ်ဆက်ခြင်း
3. ကျွန်တော်တို့၏ ကိရိယာများအားလုံးကို AI သို့ လက်လှမ်းမီအောင် ခွင့်ပြုခြင်း
4. "24.5 နှင့် 17.3 ကို ပေါင်းတွက်ပါ" ကဲ့သို့ သဘာဝဘာသာစကား မေးခွန်းများကို ခွင့်ပြုခြင်း

**AI အလိုအလျောက် လုပ်ဆောင်ပုံ**

- သင်သည် နံပါတ်များကို ပေါင်းချင်ကြောင်း နားလည်သည်
- `add` ကိရိယာကို ရွေးချယ်သည်
- `add(24.5, 17.3)` ကို ခေါ်ဆိုသည်
- ရလဒ်ကို သဘာဝ အဖြေ တစ်ခုအဖြစ် ပြန်ပေးသည်

## နမူနာများကို လုပ်ဆောင်ခြင်း

### အဆင့် ၁: တွက်ချက်စက် ဆာဗာ စတင်

ပထမဦးစွာ သင့် Azure AI Foundry endpoint ကို စာရင်းသွင်းပြီး သတ်မှတ်ပါ (AI client အသုံးပြုရန်လိုအပ်သည် - keyless auth, API key မလို)

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
  
ဆာဗာကို စတင်ပါ -  
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```
  
ဆာဗာသည် `http://localhost:8080` ပေါ်တွင် စတင်ပါလိမ့်မည်။ အောက်ပါအတိုင်း တွေ့မြင်ရမည် -  
```
Started McpServerApplication in X.XXX seconds
```
  
### အဆင့် ၂: တိုက်ရိုက် Client ဖြင့် စမ်းသပ်ခြင်း

Server စက်ကို ဖွင့်ထားသော terminal အသစ်တွင် တိုက်ရိုက် MCP client ကို လည်ပတ်ပါ -  
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```
  
ထွက်ရှိမည့် အချက်အလက် -  
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```
  
### အဆင့် ၃: AI Client ဖြင့် စမ်းသပ်ခြင်း

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```
  
AI ကိရိယာများကို အလိုအလျောက် အသုံးပြုနေသည်ကို တွေ့မြင်ရမည် -  
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```
  
### အဆင့် ၄: MCP ဆာဗာ ပိတ်သိမ်းခြင်း

စမ်းသပ်မှု ပြီးဆုံးလျှင် AI client ကို terminal တွင် `Ctrl+C` နှိပ်၍ ရပ်နားနိုင်သည်။ MCP ဆာဗာသိုလည်း သင် ပိတ်ရန် မတိုင်မီ ဆက်လက်အလုပ်လုပ်နေပါလိမ့်မည်။ ဆာဗာကို ရပ်တန့်ရန် သော့ `Ctrl+C` ကို ဆာဗာ ရပ်နေသော terminal တွင် နှိပ်ပါ။

## အားလုံး ဘယ်လိုအတူတူ လုပ်ဆောင်ကြပုံ

AI ကို "5 + 3 ဆိုတာ ဘာလဲ?" ဟုပေးမေးသောအခါ ပြည့်စုံသောလည်ပတ်ပုံ -

1. **သင်** သဘာဝဘာသာစကားဖြင့် AI ကို မေးမြန်းသည်
2. **AI** သင့်မေးခွန်းကို ခွဲခြားလိုက်ပြီး ပေါင်းအစီအစဉ်ကို သဘောပေါက်သည်
3. **AI** MCP ဆာဗာတွင် `add(5.0, 3.0)` ကို ခေါ်ဆိုသည်
4. **တွက်ချက်စက် ဝန်ဆောင်မှု** သည် `5.0 + 3.0 = 8.0` ကို တည်ဆောက်သည်
5. **တွက်ချက်စက် ဝန်ဆောင်မှု** သည် `"5.00 + 3.00 = 8.00"` အတိုင်း ပြန်လည်ပေးပို့သည်
6. **AI** ရလဒ်ကို လက်ခံပြီး သဘာဝတုန့်ပြန်မှု ဖော်ဆောင်သည်
7. **သင်** "5 နှင့် 3 ၏ တွက်ချက်သည် 8 ဖြစ်သည်" ဆိုပြီး ရရှိသည်

## နောက်တစ်ဆင့်များ

နမူနာများပိုမိုကြည့်ရန် [အခန်း 04: လက်တွေ့နမူနာများ](../README.md) ကိုကြည့်ပါ။

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ပြောကြားချက်**
ဤစာတမ်းကို AI ဘာသာပြန်ဝန်ဆောင်မှု [Co-op Translator](https://github.com/Azure/co-op-translator) အသုံးပြု၍ ဘာသာပြန်ထားပါသည်။ ကျွန်ုပ်တို့သည် တိကျမှန်ကန်မှုအတွက် ကြိုးပမ်းနေသော်လည်း၊ စက်ကိရိယာဘာသာပြန်ခြင်းများတွင် အမှားများ သို့မဟုတ် မှားယွင်းချက်များ ပါဝင်နိုင်ကြောင်း သတိပြုပါရန် လိုအပ်ပါသည်။ မူလစာတမ်းကို မူရင်းဘာသာဖြင့်သာ ယုံကြည်စိတ်ချရသော အချက်အလက်အဖြစ် သတ်မှတ်သင့်သည်။ အရေးကြီးသည့် သတင်းအချက်အလက်များအတွက် ပရော်ဖက်ရှင်နယ် လူသားဘာသာပြန်သူဝန်ဆောင်မှုကို အကြံပြုပါသည်။ ဤဘာသာပြန်ချက်ကို အသုံးပြုခြင်းမှ ဖြစ်ပေါ်လာသော နားလည်မှုကွာခြားမှုများ သို့မဟုတ် မမှန်ကန်သော အသုံးပြုမှုများအတွက် ကျွန်ုပ်တို့ တာဝန်မခံပါ။
<!-- CO-OP TRANSLATOR DISCLAIMER END -->