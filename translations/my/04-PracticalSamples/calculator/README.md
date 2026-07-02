# MCP ကိန်းဂဏန်းတွက်ခွင့် တူတူလေ့လာမှုအတွက် မိတ်ဆက်လမ်းညွှန်

## အကြောင်းအရာစာရင်း

- [သင်တတ်နိုင်မည့်အရာများ](#သင်တတ်နိုင်မည့်အရာများ)
- [လိုအပ်ချက်များ](#လိုအပ်ချက်များ)
- [ပရောဂျက်ဖွဲ့စည်းမှု နားလည်ခြင်း](#ပရောဂျက်ဖွဲ့စည်းမှု-နားလည်ခြင်း)
- [အခြေခံအစိတ်အပိုင်းများ ရှင်းလင်းချက်](#အခြေခံအစိတ်အပိုင်းများ-ရှင်းလင်းချက်)
  - [၁။ အဓိကအက်ပ်လီကေးရှင်း](#၁-အဓိကအက်ပ်လီကေးရှင်း)
  - [၂။ ကိန်းဂဏန်းတွက်ဝန်ဆောင်မှု](#၂-ကိန်းဂဏန်းတွက်ဝန်ဆောင်မှု)
  - [၃။ တိုက်ရိုက် MCP ဖောက်သည်](#၃-တိုက်ရိုက်-mcp-ဖောက်သည်)
  - [၄။ AI ပါဝင် ထောက်ပံ့ထားသောဖောက်သည်](#၄-ai-ပါဝင်-ထောက်ပံ့ထားသောဖောက်သည်)
- [နမူနာများ ကို ပြေးဆွဲခြင်း](#နမူနာများ-ကို-ပြေးဆွဲခြင်း)
- [အားလုံး သွယ်ဝိုင်း အလုပ်လုပ်ပုံ](#အားလုံး-သွယ်ဝိုင်း-အလုပ်လုပ်ပုံ)
- [နောက်တစ်ဆင့်များ](#နောက်တစ်ဆင့်များ)

## သင်တတ်နိုင်မည့်အရာများ

ဤလမ်းညွှန်တွင် Model Context Protocol (MCP) ကို အသုံးပြု၍ ကိန်းဂဏန်းတွက် ဝန်ဆောင်မှုကို မည်ကဲ့သို့ တည်ဆောက်ရမည်ကို ရှင်းလင်းပြသထားပါတယ်။ သင်ကိန်းဂဏန်းတွက်မှုအတွက် AI အလုအယက်များအနေဖြင့် ကိရိယာတစ်ခုအနေနဲ့ ဝန်ဆောင်မှုတည်ဆောက်နည်း၊ MCP ဝန်ဆောင်မှုများနှင့် တိုက်ရိုက်ဆက်သွယ်နည်း၊ AI မော်ဒယ်များက အလိုအလျောက် ကိရိယာတွေ ရွေးချယ်သုံးနိုင်ခြင်း နဲ့ တိုက်ရိုက် protocol ခေါ်ဆိုမှုနဲ့ AI ကူညီပြီး ပြုလုပ်မည့် အခြားတခြား ကြားနားချက်တို့ကို နားလည်လိမ့်မည်။

## လိုအပ်ချက်များ

စတင်ရမည့်အချိန်မှာ အောက်ပါအချက်များရှိပြီး ဖြစ်ပါစေ-
- Java 21 သို့ အထက် ထည့်သွင်းပြီးဖြစ်ရမည်
- Maven ကို dependency စီမံခန့်ခွဲမှုအတွက် သုံးစွဲထားရှိရန်
- Azure AI Foundry model တစ်ခုကို ဖြန့်ချိပြီး (azd up ဖြင့် provision လုပ်ပါ — [အပိုင်း ၂](../../02-SetupDevEnvironment/getting-started-azure-openai.md) ကိုကြည့်ရှုပါ)
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) ကို`az login` ဖြင့် (keyless auth) လက်မှတ်ထိုးထားပါ
- Java နဲ့ Spring Boot အခြေခံ သိရှိမှုရှိရန်

## ပရောဂျက်ဖွဲ့စည်းမှု နားလည်ခြင်း

ကိန်းဂဏန်းတွက်ပရောဂျက်မှာ အောက်ပါ အရေးကြီးဖိုင်အချို့ ပါဝင်သည်-

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

## အခြေခံအစိတ်အပိုင်းများ ရှင်းလင်းချက်

### ၁. အဓိကအက်ပ်လီကေးရှင်း

**ဖိုင်**`McpServerApplication.java`

ဒီဟာက ကျွန်ုပ်တို့ရဲ့ ကိန်းဂဏန်းတွက် ဝန်ဆောင်မှု၏ ဝင်ကြောင်းဖြစ်ပါတယ်။ ပုံမှန် Spring Boot အက်ပ်လီကေးရှင်းတစ်ခုဖြစ်ပြီး အထူးတစ်ခုပါရှိသည်-

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

**လုပ်ဆောင်ချက်များဖြစ်ရပ်:**
- Port 8080 ပေါ်တွင် Spring Boot ဝက်ဘ်ဆာဗာကို စတင်ချက်
- ကိန်းဂဏန်းတွက်လုပ်ငန်းများကို MCP ကိရိယာအဖြစ် အသုံးပြုနိုင်စေမည့် `ToolCallbackProvider` ကို ဖန်တီးခြင်း
- `@Bean` အမှတ်အသားသည် သက်ဆိုင်သူများ အသုံးပြုနိုင်ရန် Spring မှ အစိတ်အပိုင်းအဖြစ် စီမံခန့်ခွဲခြင်းအား ဖော်ပြသည်

### ၂. ကိန်းဂဏန်းတွက်ဝန်ဆောင်မှု

**ဖိုင်** `CalculatorService.java`

ဒီနေရာမှာ ဆယ်လီနဲ့ သင်ခန်းစာတွေရဲ့ များစွာသော သင်္ချာဆိုင်ရာ တွက်ချက်မှုများ ဖြစ်ပေါ်သည်။ MCP မှတဆင့် ခေါ်နိုင်ရန် တစ်ခုချင်းစီကို `@Tool` ဖြင့် အမှတ်အသားပြုထားသည်-

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
    
    // ကိန်းဂဏန်းတွက်ချက်မှု ပိုများစွာ...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**အဓိက လက္ခဏာများ:**

1. **`@Tool` အမှတ်အသား** - MCP သို့ ပြင်ပဖောက်သည်များက ဤနည်းလမ်းကို ခေါ်ဆိုနိုင်ကြောင်း ပြထားသည်
2. **ရှင်းလင်းသော ဖော်ပြချက်များ** - AI မော်ဒယ်များ အဘယ်အခါ ဒီကိရိယာကို သုံးရမည်ကို ပိုမိုနားလည်စေရန် ဖော်ပြချက်များရှိသည်
3. **တစ်ညီတစ်ဝိုင်း ပြန်လာမည့် ပုံစံ** - လုပ်ဆောင်ချက်များမှ လူသားနားလည်နိုင်သည့် စာကြောင်းများ "5.00 + 3.00 = 8.00" အနေနဲ့ ပြန်ယူနိုင်သည်
4. **အမှားကိုင်တွယ်မှု** - သုညဖြင့် အပိုင်းခွဲခြင်းနှင့် အနုတ်ဂဏန်း မှတ်စု ပြဿနာကို အမှားချက်စာချုပ်များဖြင့် ပြန်ပေးသည်

**ရရှိနိုင်သော လုပ်ဆောင်ချက်များ:**
- `add(a, b)` - နံပါတ်နှစ်ခုကို စုစည်းတွက်ချက်သည်
- `subtract(a, b)` - ဒုတိယအရေအတွက်မှ ပထမအရေအတွက်ကို ဖယ်ရှားသည်
- `multiply(a, b)` - နံပါတ်နှစ်ခုကို ထပ်မံမြှောက်သည်
- `divide(a, b)` - ပထမအရေအတွက်ကို ဒုတိယဖြင့် တိုက်ရိုက်ခွဲသည် (သုညစစ်ဆေးမှု ပါဝင်)
- `power(base, exponent)` - မူလအခြေကို အမြောက်အမြားဖြင့်မြှောက်သည်
- `squareRoot(number)` - အချက်အလက်သက်တမ်းများကို တွက်ချက်ခြင်း (အနုတ်စစ်ဆေးမှုပါရှိ)
- `modulus(a, b)` - ခွဲခြားပြီး ကွဲကျန်ရလဒ်ကို ပြန်ပေးသည်
- `absolute(number)` - တန်ဖိုးအပြည့်အစုံကို ပြန်ပေးသည်
- `help()` - လုပ်ဆောင်ချက်အားလုံး အကြောင်းအရာကို ပြန်ပေးသည်

### ၃. တိုက်ရိုက် MCP ဖောက်သည်

**ဖိုင်** `SDKClient.java`

ဤဖောက်သည်သည် AI အသုံးပြုခြင်းမရှိဘဲ MCP ဆာဗာနှင့် တိုက်ရိုက် ဆက်သွယ်သည်။ အတိအကျ ကိန်းဂဏန်းတွက်နည်း လုပ်ဆောင်ချက်များကို လက်ဖြင့် ခေါ်ဆိုသည်-

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
        
        // အသုံးပြုနိုင်သောကိရိယာများကို စာရင်းပြုစုပါ
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // သတ်မှတ်ထားသောဂဏန်းတွက်ခြင်းဖန်နာလ်များကို ခေါ်ဆိုပါ
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

**လုပ်ဆောင်ချက်များ:**
1. Builder ပုံစံနဲ့ `http://localhost:8080` တွင် ကိန်းဂဏန်းဆာဗာကို ချိတ်ဆက်သည်
2. ရရှိနိုင်သော ကိရိယာများ အားလုံးကို စာရင်းပြုစုသည် (ကျွန်ုပ်တို့ရဲ့ ကိန်းဂဏန်းလုပ်ဆောင်ချက်များ)
3. တိကျသော ပါရာမီတာဖြင့် အတိအကျ function များ ခေါ်ဆိုသည်
4. ရလဒ်များကို တိုက်ရိုက် စာမူထုတ်ပြန်ပေးသည်

**မှတ်ချက်:** ဤနမူနာသည် Spring AI 1.1.0-SNAPSHOT dependency ကို သုံးထားပြီး WebFluxSseClientTransport ၏ builder ပုံစံကို စတင်ထည့်သွင်းထားသည်။ ပိုပြီးအဟောင်း stable version များ သုံးရမယ်ဆိုရင် တိုက်ရိုက် constructor အသုံးပြုရမည် ဖြစ်ပါတယ်။

**ဘယ်အချိန်အသုံးပြုရန်:** သင် တိတိကျကျ ထုတ်လွှင့်လိုသော တွက်ချက်ချက်ကို အစီအစဉ်နဲ့ ခေါ်ဆိုလိုသောအချိန်တွင် သုံးပါ။

### ၄. AI ပါဝင် ထောက်ပံ့ထားသော ဖောက်သည်

**ဖိုင်** `LangChain4jClient.java`

ဤဖောက်သည်သည် GPT-4o-mini AI မော်ဒယ်ကို အသုံးပြုကာ ကိန်းဂဏန်းတွက် ကိရိယာများကို အလိုအလျောက် ရွေးချယ်သုံးနိုင်စေပါသည်-

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // AI မော်ဒယ်အား စတင်တပ်ဆင်ပါ (Azure AI Foundry, Microsoft Entra ID ဖြင့် keyless ကိုးကားခြင်း)
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

        // ကျွန်ုပ်တို့၏ ကိန်းဂဏန်း MCP ဆာဗာနှင့် ချိတ်ဆက်ပါ
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // AI လုပ်ဆောင်နေသော အဖြစ်အပျက်ကို ပြရန်
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // ကျွန်ုပ်တို့၏ ကိန်းဂဏန်းကိရိယာများကို AI သို့ အသုံးပြုခွင့်ပေးပါ
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // ကျွန်ုပ်တို့၏ ကိန်းဂဏန်းကို အသုံးပြုနိုင်သော AI ဘော့တစ်ခု ဖန်တီးပါ
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // ယခု AI ကို သဘာဝဘာသာစကားဖြင့် ကိန်းဂဏန်းတွက်ချက်ရန် မေးနိုင်ပြီ ဖြစ်ပါသည်
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**လုပ်ဆောင်ချက်များ:**
1. GitHub token ကို အသုံးပြုကာ AI မော်ဒယ် ချိတ်ဆက်မှုတည်ဆောက်သည်
2. AI ကို ကျွန်တော်တို့ရဲ့ MCP ကိန်းဂဏန်းဆာဗာနှင့် ချိတ်ဆက်သည်
3. AI ကို ကိန်းဂဏန်းတွက် ကိရိယာများအားလုံးသို့ ဝင်ရောက်ခွင့်ပြုသည်
4. "24.5 နဲ့ 17.3 ကို စုပေါင်းတွက်ပါ" စသည့် သဘာဝဘာသာစကား အမိန့်များကို ခွင့်ပြုသည်

**AI အလိုအလျောက်ပြုလုပ်သည်များ-**
- သင်ယခု စုပေါင်းလိုသည့် အကြောင်းကို နားလည်သည်
- `add` ကိရိယာကို ရွေးချယ်သည်
- `add(24.5, 17.3)` ကို ခေါ်သုံးသည်
- ရလဒ်ကို သဘာဝတုနဲ့ ပြန်ပေးသည်

## နမူနာများ ကို ပြေးဆွဲခြင်း

### ခြေလှမ်း ၁: ကိန်းဂဏန်း ဆာဗာ စတင်ခြင်း

ပထမ တစ်ဆင့်အနေနဲ့ Azure AI Foundry endpoint ကို သင်အကောင့်ဝင်ပြီး ပေးပါ (AI client အတွက် လိုအပ်သည်- keyless auth၊ API key မလိုအပ်ပါ) -

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

ဆာဗာ စတင်ရန်:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

ဆာဗာသည် `http://localhost:8080` ပေါ်တွင် စတင်ပါမည်။ သင်အောက်ပါအတိုင်း မြင်ရမည်-
```
Started McpServerApplication in X.XXX seconds
```

### ခြေလှမ်း ၂: တိုက်ရိုက်ဖောက်သည်ဖြင့် စမ်းသပ်ခြင်း

ဆာဗာ ပုံမှန်သုံးဆဲနေစဉ် အသစ်သော တာမီနယ်တွင် တိုက်ရိုက် MCP ဖောက်သည် အစီအစဉ်ကို ပြေးပါ-
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

အထွက်အနေနဲ့ အောက်ပါအတိုင်း မြင်ရပါမည်-
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### ခြေလှမ်း ၃: AI ဖောက်သည်ဖြင့် စမ်းသပ်ခြင်း

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

AI သည် ကိရိယာများကို အလိုအလျောက် အသုံးပြုနေပါသည်-
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### ခြေလှမ်း ၄: MCP ဆာဗာ ပိတ်ရန်

စမ်းသပ်မှုပြီးသွားပါက AI ဖောက်သည်ကို Ctrl+C နှိပ်၍ ရပ်နားစေပါ။ MCP ဆာဗာကို သင်ပိတ်ရန် မရောက်ရှိသေးပါ။ ဆာဗာကို ပိတ်ရန် ဆာဗာပြေးနေသည့် တာမီနယ်တွင် Ctrl+C နှိပ်လိုက်ပါ။

## အားလုံး သွယ်ဝိုင်း အလုပ်လုပ်ပုံ

AI ကို "5 + 3 ဘာလဲ?" ဟု မေးသောအခါ အလုပ်လုပ်မှုလမ်းကြောင်း အပြည့်အစုံမှာ-

1. **သင်** သဘာဝဘာသာစကားဖြင့် AI ကို မေးသည်
2. **AI** သင်တောင်းဆိုချက် ကို လေ့လာ၍ စုစုပေါင်းတွက်ချင်သည်ကို နားလည်သည်
3. **AI** MCP ဆာဗာကို `add(5.0, 3.0)` လို ခေါ်ဆိုသည်
4. **ကိန်းဂဏန်းတွက်ဝန်ဆောင်မှု** `5.0 + 3.0 = 8.0` တွက်ချက်ပေးသည်
5. **ကိန်းဂဏန်းတွက်ဝန်ဆောင်မှု** `"5.00 + 3.00 = 8.00"` ဆို၍ ပြန်ပေးသည်
6. **AI** ရလဒ်ကို သိရှိကာ သဘာဝတုဖြင့် တုံ့ပြန်ချက်ပြုလုပ်သည်
7. **သင်** "5 နဲ့ 3 ပေါင်းရလဒ်က 8 ပါ" ဟူသော တုံ့ပြန်ချက် ရရှိသည်

## နောက်တစ်ဆင့်များ

နမူနာများ ပိုမိုကြည့်ရှုရန် [အပိုင်း 04: လက်တွေ့ နမူနာများ](../README.md) ကို ကြည့်ပါ။

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ပြောကြားချက်**
ဤစာတမ်းကို AI ဘာသာပြန်ဝန်ဆောင်မှု [Co-op Translator](https://github.com/Azure/co-op-translator) အသုံးပြု၍ ဘာသာပြန်ထားပါသည်။ ကျွန်ုပ်တို့သည် တိကျမှန်ကန်မှုအတွက် ကြိုးပမ်းနေသော်လည်း၊ စက်ကိရိယာဘာသာပြန်ခြင်းများတွင် အမှားများ သို့မဟုတ် မှားယွင်းချက်များ ပါဝင်နိုင်ကြောင်း သတိပြုပါရန် လိုအပ်ပါသည်။ မူလစာတမ်းကို မူရင်းဘာသာဖြင့်သာ ယုံကြည်စိတ်ချရသော အချက်အလက်အဖြစ် သတ်မှတ်သင့်သည်။ အရေးကြီးသည့် သတင်းအချက်အလက်များအတွက် ပရော်ဖက်ရှင်နယ် လူသားဘာသာပြန်သူဝန်ဆောင်မှုကို အကြံပြုပါသည်။ ဤဘာသာပြန်ချက်ကို အသုံးပြုခြင်းမှ ဖြစ်ပေါ်လာသော နားလည်မှုကွာခြားမှုများ သို့မဟုတ် မမှန်ကန်သော အသုံးပြုမှုများအတွက် ကျွန်ုပ်တို့ တာဝန်မခံပါ။
<!-- CO-OP TRANSLATOR DISCLAIMER END -->