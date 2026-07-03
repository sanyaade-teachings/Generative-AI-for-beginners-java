# MCP കാൽക്കുലേറ്റർ ട്യൂട്ടോറിയൽ ബിഗിനേഴ്സ്‌ക്ക്

## ഉള്ളടക്ക പട്ടിക

- [നിങ്ങൾ പഠിക്കുയത്രുള്ളത്](#നിങ്ങൾ-പഠിക്കുയത്രുള്ളത്)
- [ആവശ്യമായ മുൻകരുതലുകൾ](#ആവശ്യമുളള-മുൻകരുതലുകൾ)
- [പ്രോജക്ട് സ്ട്രക്ചർ മനസ്സിലാക്കൽ](#പ്രോജക്ട്-സ്ട്രക്ചർ-മനസ്സിലാക്കൽ)
- [പ്രധാന ഘടകങ്ങൾ വിശദീകരിച്ചു](#പ്രധാന-ഘടകങ്ങൾ-വിശദീകരിച്ചു)
  - [1. മെയിൻ ആപ്ലിക്കേഷൻ](#1-മെയിൻ-ആപ്ലിക്കേഷൻ)
  - [2. കാൽക്കുലേറ്റർ സേർവീസ്](#2-കാൽക്കുലേറ്റർ-സേവീസ്)
  - [3. ഡയരക്ട് MCP ക്ലയന്റ്](#3-direct-mcp-client)
  - [4. AI-പവേർഡ് ക്ലയന്റ്](#4-ai-പവേർഡ്-ക്ലയന്റ്)
- [ഉദാഹരണങ്ങൾ ഓടിക്കൽ](#ഉദാഹരണങ്ങൾ-ഓടിക്കൽ)
- [എന്ത് ബഹുമുഖം പ്രവർത്തിക്കുന്നു](#എന്ത്-ബഹുമുഖം-പ്രവർത്തിക്കുന്നു)
- [അടുത്ത ചുവടുകൾ](#അടുത്ത-ചുവടുകൾ)

## നിങ്ങൾ പഠിക്കുയത്രുള്ളത്

ഈ ട്യൂട്ടോറിയൽ മോഡൽ കോൺടെക്സ് പ്രോട്ടോക്കോൾ (MCP) ഉപയോഗിച്ച് കാൽക്കുലേറ്റർ സേവനം എങ്ങനെ നിർമ്മിക്കാമെന്ന് വിശദീകരിക്കുന്നു. നിങ്ങൾക്ക് മനസ്സിലാകും:

- എങ്ങനെ AI-ക്ക് ടൂൾ ആയി ഉപയോഗിക്കാവുന്ന ഒരു സേവനം സൃഷ്ടിക്കാമെന്ന്
- MCP സേവനങ്ങളോടുള്ള നേരിട്ടുള്ള സംവാദം എങ്ങനെ ക്രമീകരിക്കാമെന്ന്
- എഐ മോഡലുകൾ സ്വയം ഏത് ടൂളുകൾ ഉപയോഗിക്കാമെന്ന് തിരഞ്ഞെടുക്കുന്നത് എങ്ങനെ നടക്കുന്നത്
- നേരിട്ടുള്ള പ്രോട്ടോക്കോൾ കോൾസും AI-സഹായിത ഇടപെടലുകളും തമ്മിലുള്ള വ്യത്യാസം

## ആവശ്യമുളള മുൻകരുതലുകൾ

ആരംഭിക്കുന്നതിന് മുമ്പ്, ദയവായി ഉറപ്പാക്കുക:
- Java 21 അല്ലെങ്കിൽ അതിനുമുകളിൽ ഇൻസ്റ്റാൾ ചെയ്തിട്ടുണ്ട്
- ഡിപ്പെൻഡൻസി മാനേജ്‌മെന്റിനായി Maven
- Azure AI Foundry മോഡൽ ഡിപ്ലോയ്മെന്റ് ( `azd up` ഉപയോഗിച്ച് പ്രൊവിഷൻ ചെയ്യുക — [Chapter 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md) കാണുക)
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), `az login` ഉപയോഗിച്ച് സൈൻ ഇൻ ചെയ്തിട്ടുണ്ട് (കീലെസ്സ് ഓഥ്)
- Java, Spring Boot എന്നിവയുടെ അടിസ്ഥാന അറിവ്

## പ്രോജക്ട് സ്ട്രക്ചർ മനസ്സിലാക്കൽ

കാൽക്കുലേറ്റർ പ്രോജക്ടിൽ ചില പ്രധാന ഫയലുകൾ ഉണ്ട്:

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


## പ്രധാന ഘടകങ്ങൾ വിശദീകരിച്ചു

### 1. മെയിൻ ആപ്ലിക്കേഷൻ

**ഫയൽ:** `McpServerApplication.java`

ഇതാണ് നമ്മുടെ കാൽക്കുലേറ്റർ സേവനത്തിന്‍റെ എൻട്രി പോയിൻറ്. ഇത് ഒരു സ്റ്റാൻഡേർഡ് Spring Boot ആപ്ലിക്കേഷനാണ്, ഒരു പ്രത്യേക അധിക സവിശേഷതയോടുകൂടി:

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


**ഇത് ചെയ്യുന്നത്:**
- 8080 പോർട്ടിൽ Spring Boot വെബ് സെർവർ ആരംഭിക്കുന്നു
- കാൽക്കുലേറ്റർ മേധാവിത്വം MCP ടൂളുകളായി ലഭ്യമാക്കുന്ന `ToolCallbackProvider` സൃഷ്ടിക്കുന്നു
- `@Bean` ഒറ്റക്കുറിപ്പ് Spring-ന് ഇത് ഒരു ഘടകമായി മാനേജു ചെയ്യാനായി അറിയിക്കുന്നു, മറ്റുഭാഗങ്ങൾ ഉപയോഗിക്കാം

### 2. കാൽക്കുലേറ്റർ സേവീസ്

**ഫയൽ:** `CalculatorService.java`

അല്ലം കല്കുലേഷനുകൾ ഇവിടെ നടക്കുന്നു. ഓരോ մեթոդും MCP വഴി ലഭ്യമായിരിക്കാൻ `@Tool` ഉപയോഗിച്ച് അടയാളപ്പെടുത്തിയിരിക്കുന്നു:

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
    
    // കൂടി കാൽക്കുലേറ്റർ പ്രവർത്തനങ്ങൾ...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```


**പ്രധാന സവിശേഷതകൾ:**

1. **`@Tool` അനേകാൽ**: ഈ մեթഡ്സ് MCP യുടെ പുറമേത്തുള്ള ക്ലയന്റുകൾക്ക് വിളിക്കാവുന്നതായി അറിയിക്കുന്നു
2. **സ്വച്ഛമായ വിവരണങ്ങൾ**: ഓരോ ടൂള്ക്കും എപ്പോഴ kullanപ്പെടുത്തണമെന്ന് AI മോഡൽ‌ക്ക് അവബോധം നൽകുന്ന വിവരണം ഉണ്ട്
3. **മാനവൻ വായിക്കാൻ കഴിയുന്ന ഫോർമാറ്റിൽ തീയതി**: എല്ലാവിധ ഓപ്പറേഷനുകളും "5.00 + 3.00 = 8.00" തരം വായിക്കാൻ എളുപ്പമാക്കിയ ഫലങ്ങൾ നൽകുന്നു
4. **എരോർ കൈകാര്യം**: പൂജ്യം വ്യത്യാസത്തിലോ നെഗറ്റീവ് വേരൂട്ട് എന്നിവയുള്ളത് പിഴവുകൾ നൽകുന്നു

**ലഭ്യമായ പ്രവര്‍ത്തനങ്ങൾ:**
- `add(a, b)` - രണ്ട് സംഖ്യകൾ ചേർക്കുന്നു
- `subtract(a, b)` - രണ്ടാമത്തെത്തിന്നും ആദ്യത്തേത് ന്ദേഹം ഘടിപ്പിക്കുന്നു
- `multiply(a, b)` - രണ്ട് സംഖ്യകളുടെ ഗുണനം
- `divide(a, b)` - ആദ്യ സംഖ്യ രണ്ടാമത്താൻ മുഖേന വിഭജിക്കുന്നു (പൂജ്യം പരിശോധിച്ച്)
- `power(base, exponent)` - ബേസിന്.എസ് പവർ ഉയർത്തുന്നു
- `squareRoot(number)` - വേരൂട്ട് കണക്കാക്കുന്നു (നെഗറ്റിവ് പരിശോധിച്ച്)
- `modulus(a, b)` - വിഭജിച്ച് ശേഷിക്കുന്ന ഭാഗം നൽകുന്നു
- `absolute(number)` - മൂല്യം ആണെങ്കിൽ നമ്മുടെ
- `help()` - എല്ലാ പ്രവർത്തനങ്ങളേക്കുറിച്ചുള്ള വിവരം തിരികെ നൽകുന്നു

### 3. Direct MCP Client

**ഫയൽ:** `SDKClient.java`

ഈ ക്ലയന്റ് AI ഉപയോഗിക്കാതെ നേരിട്ട് MCP സെർവറുമായി സംസാരിക്കുന്നു. ഇത് വ്യക്തിഗത കാൽക്കുലേറ്റർ ഫംഗ്ഷനുകൾ കൈകാര്യം ചെയ്യുന്നു:

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
        
        // ലഭ്യമായ ഉപകരണങ്ങളുടെ പട്ടിക
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // പ്രത്യേക കാൽക്കുലേറ്റർ ഫംഗ്ഷനുകൾ വിളിക്കുക
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


**ഇത് ചെയ്യുന്നത്:**
1. `http://localhost:8080` -ലെ കാൽക്കുലേറ്റർ സെർവറുമായി ബില്ഡർ പാറ്റേൺ ഉപയോഗിച്ച് കണക്ട് ചെയ്യുന്നു
2. ലഭ്യമായ എല്ലാ ടൂളുകളും ലിസ്റ്റ് ചെയ്യുന്നു (നമ്മുടെ കാൽക്കുലേറ്റർ ഫംഗ്ഷനുകൾ)
3. കൃത്യമായ പാരാമീറ്ററുകൾ ഉപയോഗിച്ച് പ്രത്യേക ഫംഗ്ഷനുകൾ വിളിക്കുന്നു
4. ഫലങ്ങൾ നേരിട്ട് പ്രിന്റ് ചെയ്യുന്നു

**നോട്ടീസ്:** ഈ ഉദാഹരണം Spring AI 1.1.0-SNAPSHOT ഡിപ്പെൻഡൻസി ഉപയോഗിക്കുന്നു, ഇത് `WebFluxSseClientTransport` നു വേണ്ടി ബില്ഡർ പാറ്റേൺ അവതരിപ്പിച്ചു. പഴയ സ്റ്റേബിൾ പതിപ്പുകൾ ഉപയോഗിക്കുകയാണെങ്കിൽ, നേരിട്ടുള്ള കൺസ്ട്രക്ടർ ഉപയോഗിക്കേണ്ടിവരും.

**എപ്പോൾ ഉപയോഗിക്കാം:** നിങ്ങളുടെ കണക്കു കൃത്യമായി തിരഞ്ഞെടുക്കുകയും പ്രോഗ്രാമാറ്റിക്കായി വിളിക്കാനുളള ആഗ്രഹം ഉണ്ടെങ്കിലേ.

### 4. AI-പവേർഡ് ക്ലയന്റ്

**ഫയൽ:** `LangChain4jClient.java`

ഈ ക്ലയന്റ് AI മോഡൽ (GPT-4o-mini) ഉപയോഗിക്കുന്നു, അത് സ്വയം തെരഞ്ഞെടുക്കാം ഏത് കാൽക്കുലേറ്റർ ടൂൾ ഉപയോഗിക്കണമെന്നും:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // AI മോഡലിനെ സജ്ജമാക്കുക (Azure AI Foundry, Microsoft Entra ID മുഖേന കീലെസ് ഓത്തൻ)
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

        // നമ്മുടെ കാൽക്കുലേറ്റർ MCP സർവറുമായി കണക്ട് ചെയ്യുക
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // AI എന്ത് ചെയ്യുകയാണെന്ന് കാണിക്കുന്നു
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // AIക്ക് നമ്മുടെ കാൽക്കുലേറ്റർ ഉപകരണങ്ങളിൽ ആക്‌സസ്സ് നൽകുക
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // നമ്മുടെ കാൽക്കുലേറ്റർ ഉപയോഗിക്കാൻ കഴിവുള്ള AI ബോട്ട് സൃഷ്‌ടിക്കുക
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // ഇപ്പഴിതന്നെ പക്ഷേ, നിസ്സാര ഭാഷയിൽ കണക്ക് ചെയ്യാൻ AI നോട് അഭ്യർത്ഥന കാണിക്കാം
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```


**ഇത് ചെയ്യുന്നത്:**
1. ձեր GitHub ടോക്കൺ ഉപയോഗിച്ച് AI മോഡൽ കണക്ഷൻ സൃഷ്ടിക്കുന്നു
2. AI-യെ നമ്മുടെ ക്യൾക്കുലേറ്റർ MCP സെർവറുമായി കണക്ട് ചെയ്യുന്നു
3. AI-ന് എല്ലാ കാൽക്കുലേറ്റർ ടൂളുകളെയും ആക്‌സസ് നൽകുന്നു
4. "24.5നും 17.3നും കൂട്ടിച്ചേർക്കുക" പോലെയുള്ള സ്വാഭാവിക ഭാഷാ അഭ്യർത്ഥനകൾ അനുവദിക്കുന്നു

**AI സ്വയം:**
- നിങ്ങൾക്ക് സംഖ്യകൾ കൂട്ടിയാൽ എന്നത് മനസ്സിലാക്കുന്നു
- `add` ടൂൾ തിരഞ്ഞെടുക്കുന്നു
- `add(24.5, 17.3)` വിളിക്കുന്നു
- ഫലത്തെ സ്വാഭാവിക മറുപടിയാക്കി തിരികെ നൽകുന്നു

## ഉദാഹരണങ്ങൾ ഓടിക്കൽ

### പടി 1: കാൽക്കുലേറ്റർ സെർവർ തുടങ്ങുക

ആദ്യം, സൈൻ ഇൻ ചെയ്ത് Azure AI Foundry എന്റ്പോയിന്റ് ക്രമീകരിക്കുക (AI ക്ലയന്റിന് ആവശ്യമാണ് — കീലെസ്സ് ഓഥ്, API കീ ഇല്ലാതെ):

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


സെർവർ ആരംഭിക്കുക:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```


സെർവർ `http://localhost:8080`-ൽ ആരംഭിക്കും. നിങ്ങൾ കാണുക:
```
Started McpServerApplication in X.XXX seconds
```


### പടി 2: ഡയരക്ട് ക്ലയന്റ് ഉപയോഗിച്ച് പരീക്ഷിക്കുക

**പുതിയ** ടെർമിനലിൽ, സെർവർ ഓടിക്കുമ്പോൾ, ഡയരക്ട് MCP ക്ലയന്റ് റൺ ചെയ്യുക:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```


അത്തരത്തിലുള്ള ഔട്ട്പുട്ട് കാണും:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```


### പടി 3: AI ക്ലയന്റ് ഉപയോഗിച്ച് പരീക്ഷിക്കുക

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```


AI സ്വയം ടൂളുകൾ ഉപയോഗിക്കുന്നതിൻറെ ദൃശ്യങ്ങൾ കാണും:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```


### പടി 4: MCP സെർവർ বন্ধാക്കുക

പരീക്ഷണം കഴിഞ്ഞാൽ, AI ക്ലയന്റ് ടെർമിനലിൽ `Ctrl+C` അമർത്തി നിർത്താം. MCP സെർവർ നിങ്ങൾ നിർത്താത്തതു വരെ ഓടലാണ്.
സെർവർ നിർത്താൻ அதன் ടെർമിനലിൽ `Ctrl+C` അമർത്തുക.

## എന്ത് ബഹുമുഖം പ്രവർത്തിക്കുന്നു

നിങ്ങൾ AI-ന് "5 + 3 എന്താണ്?" എന്ന് ചോദിക്കുമ്പോൾ പൂർണ്ണ പ്രവർത്തന പ്രവാഹം ഇതാണ്:

1. **നിങ്ങൾ** AI-ന് സ്വാഭാവിക ഭാഷയിൽ ചോദിക്കുന്നു
2. **AI** അഭ്യർത്ഥന വിശകലനം ചെയ്ത് കൂട്ടിച്ചേർക്കൽ ആഗ്രഹം തിരിച്ചറിയുന്നു
3. **AI** MCP സെർവറെ വിളിക്കുന്നു: `add(5.0, 3.0)`
4. **കാൽക്കുലേറ്റർ സേവീസ്** പ്രവർത്തിക്കുന്നു: `5.0 + 3.0 = 8.0`
5. **കാൽക്കുലേറ്റർ സേവീസ്** തിരികെ നൽകുന്നു: `"5.00 + 3.00 = 8.00"`
6. **AI** ഫലം സ്വീകരിച്ചു സ്വാഭാവിക മറുപടി രൂപപ്പെടുത്തുന്നു
7. **നിങ്ങൾ** ലഭിക്കുന്നു: "5 ഉം 3 ഉം ചേർക്കുന്നതിന്റെ ഫലം 8 ആണ്"

## അടുത്ത ചുവടുകൾ

കൂടുതൽ ഉദാഹരണങ്ങൾക്ക് [Chapter 04: Practical samples](../README.md) കാണുക

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**അറിയിപ്പ്**:
ഈ രേഖ AI പരിഭാഷാ സേവനം [Co-op Translator](https://github.com/Azure/co-op-translator) ഉപയോഗിച്ച് പരിഭാഷപ്പെടുത്തിയതാണ്. ഞങ്ങൾ കൃത്യതയ്ക്കായി ശ്രമിക്കുന്നുവെങ്കിലും, ഓട്ടോമേറ്റഡ് പരിഭാഷകളിൽ പിഴവുകൾ അല്ലെങ്കിൽ തെറ്റായ വിവരങ്ങൾ ഉണ്ടാകാൻ സാധ്യതയുണ്ട്. അതിന്റെ സ്വാഭാവിക ഭാഷയിലുള്ള അസൽ രേഖയാണ് പ്രാമാണികമായ ഉറവിടമായി പരിഗണിക്കേണ്ടത്. നിർണായകമായ വിവരങ്ങൾക്ക്, പ്രൊഫഷണൽ മനുഷ്യ പരിഭാഷ ശുപാർശ ചെയ്യുന്നു. ഈ പരിഭാഷ ഉപയോഗിച്ച് ഉണ്ടാകുന്ന തെറ്റിദ്ധാരണകൾ അല്ലെങ്കിൽ തെറ്റായ വ്യാഖ്യാനങ്ങൾക്കായി ഞങ്ങൾ ഉത്തരവാദികളല്ല.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->