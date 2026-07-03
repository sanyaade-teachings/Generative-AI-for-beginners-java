# MCP കല്കുലേറ്റർ ട്യൂട്ടോറിയൽ ബിഗിനേഴ്സ്‌ക്കു വേണ്ടി

## ഉള്ളടക്ക പട്ടിക

- [നിങ്ങള്‍ എന്ത് പഠിക്കും](#നിങ്ങള്‍-എന്തു-പഠിക്കും)
- [അവശ്യമായ മുന്‍കൂട്ടി അറിയലുകള്‍](#ആവശ്യമായ-മുന്‍കൂട്ടി-അറിയലുകള്‍)
- [പ്രോജക്ട് ഘടന മനസിലാക്കല്‍](#പ്രോജക്ട്-ഘടന-മനസിലാക്കല്‍)
- [പ്രധാന ഘടകങ്ങള്‍ വിശദീകരിച്ചത്](#പ്രധാന-ഘടകങ്ങള്‍-വിശദീകരിച്ചത്)
  - [1. പ്രധാന അപ്ലിക്കേഷൻ](#1-പ്രധാന-അപ്ലിക്കേഷൻ)
  - [2. കല്കുലേറ്റർ സര്‍വീസ്](#2-കല്കുലേറ്റർ-സർവീസ്)
  - [3. നേരിട്ട് MCP ക്ലയന്റ്](#3-നേരിട്ട്-mcp-ക്ലയന്റ്)
  - [4. AI-പവേർഡ് ക്ലയന്റ്](#4-ai-പവേർഡ്-ക്ലയന്റ്)
- [ഉദാഹരണങ്ങള്‍ ഓടിക്കുന്നു](#ഉദാഹരണങ്ങള്‍-ഓടിക്കുന്നു)
- [എങ്ങിനെ എല്ലാം ചേർന്ന് പ്രവർത്തിക്കുന്നു](#എങ്ങിനെ-എല്ലാം-ചേർന്ന്-പ്രവർത്തിക്കുന്നു)
- [അടുത്ത ഘട്ടങ്ങൾ](#അടുത്ത-ഘട്ടങ്ങൾ)

## നിങ്ങള്‍ എന്തു പഠിക്കും

ഈ ട്യൂട്ടോറിയല്‍ എങ്ങനെ Model Context Protocol (MCP) ഉപയോഗിച്ച് ഒരു കല്കുലേറ്റർ സർവീസ് നിർമ്മിക്കാനാണ് വിശദീകരിക്കുന്നത്. നിങ്ങൾക്ക് മനസ്സിലാകും:

- എങ്ങനെ AI ടൂളായി ഉപയോഗിക്കാനുള്ള സർവീസ് നിർമ്മിക്കാം
- MCP സർവീസുകളുമായി നേരിട്ട് സംവദിക്കൽ എങ്ങനെ ക്രമീകരിക്കാം
- എങ്ങനെ AI മോഡലുകൾ സ്വയം ഏത് ടൂൾസ് ഉപയോഗിക്കാമെന്ന് തിരഞ്ഞെടുക്കുന്നു
- നേരിട്ട് പ്രോട്ടോകോള്‍ കോളുകളും AI-സഹായമുള്ള സംവരണങ്ങളും തമ്മിലുള്ള വ്യത്യാസം

## ആവശ്യമായ മുന്‍കൂട്ടി അറിയലുകള്‍

തുടങ്ങുന്നതിന് മുമ്പ് ഉറപ്പാക്കുക:
- Java 21 അല്ലെങ്കിൽ അതിന് മുകളിലുള്ള പതിപ്പ് ഇൻസ്റ്റാൾ ചെയ്തു
- Maven ഡിപെൻഡൻസി മാനേജ്മെന്റിനായി
- Azure AI Foundry മോഡൽ ഡിപ്ലോയ്മെന്റ് ( `azd up` ഉപയോഗിച്ച് പ്രൊവിഷൻ ചെയ്യുക — [Chapter 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md) കാണുക)
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), `az login` ഉപയോഗിച്ച് സൈൻ ഇൻ ചെയ്തിട്ടുള്ളത് (കീലെസ് ഓതെൻറിക്കേഷൻ)
- Java-നും Spring Boot-നും അടിസ്ഥാന അറിവ്

## പ്രോജക്ട് ഘടന മനസിലാക്കല്‍

കല്കുലേറ്റർ പ്രോജക്ടിന് താഴെപ്പറയുന്ന പ്രധാന ഫയലുകളുണ്ട്:

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
  
## പ്രധാന ഘടകങ്ങള്‍ വിശദീകരിച്ചത്

### 1. പ്രധാന അപ്ലിക്കേഷൻ

**ഫയൽ:** `McpServerApplication.java`

ഇതാണ് നമ്മുടെ കല്കുലേറ്റർ സർവീസിന്റെ എൻട്രി പോയിന്റ്. ഇത് ഒരു സ്റ്റാന്റേഡ് Spring Boot അപ്ലിക്കേഷനാണ്, പ്രത്യേകൊരു അധിക സഹായത്തോടെ:

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
- പോർട്ട് 8080 ല്‍ Spring Boot വെബ് സെർവർ ആരംഭിക്കുന്നു  
- మా കല്കുലേറ്റർ മെതഡുകൾ MCP ടൂളുകൾ ആയി ലഭ്യമാക്കുന്ന `ToolCallbackProvider` സൃഷ്ടിക്കുന്നു  
- `@Bean` അനോട്ടേഷൻ ഇത് മറ്റുഭാഗങ്ങൾ ഉപയോഗിക്കാവുന്ന ഒരു കമ്പോണന്റ് ആയി Spring ക്ക് മാനേജ് ചെയ്യാൻ പറയുന്നു  

### 2. കല്കുലേറ്റർ സർവീസ്

**ഫയൽ:** `CalculatorService.java`

ഇവിടെ എല്ലാ ഗണിത നടപടി ക്രിയകളും നടക്കുന്നു. ഓരോ മെതഡ്‌ക്കും `@Tool` അനോട്ടേഷൻ കൊടുത്തിരിക്കുന്നു MCP വഴി ലഭ്യമാക്കാൻ:

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
    
    // കൂടുതൽ കാൽകുലേറ്റർ പ്രവർത്തനങ്ങൾ...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```
  
**പ്രധാന ഘടകങ്ങൾ:**

1. **`@Tool` അനോട്ടേഷൻ**: ഈ മെതഡ് പുറത്തുള്ള ക്ലയന്റുകൾക്ക് വിളിക്കാമെന്ന് MCP-ക്ക് അറിയിച്ചു  
2. **സ്പഷ്ടമായ വിവരണങ്ങൾ**: ഓരോ ടൂളിനും AI മോഡലുകൾക്ക് എപ്പോള്‍ ব্যবহারിക്കാം എന്ന് മനസ്സിലാക്കിയെടുക്കാന്‍ സഹായിക്കുന്ന വിവരണം  
3. **സ്ഥിരമായ റിട്ടണ്‍ ഫോർമാറ്റ്**: എല്ലാ പ്രവർത്തനങ്ങളും മനുഷ്യന് വായിക്കാൻ എളുപ്പമുള്ള സ്ട്രിംഗുകളായ "5.00 + 3.00 = 8.00" ഫോർമാറ്റിൽ ഫലം നൽകുന്നു  
4. **പിശക് കൈകാര്യം ചെയ്യല്‍**: സൂന്യത്തോടെ വിഭജിക്കൽക്കും നെഗറ്റീവ് സ്ക്വയർ റൂട്ടിനും പിശക് സന്ദേശങ്ങൾ നൽകുന്നു  

**ലഭ്യമായ പ്രവർത്തനങ്ങൾ:**  
- `add(a, b)` - രണ്ട് സംഖ്യകൾ ചേർത്തു നൽകുന്നു  
- `subtract(a, b)` - രണ്ടാമത്തെ സംഖ്യ ആദ്യം നിന്നു വെട്ടിച്ചു നീക്കുന്നു  
- `multiply(a, b)` - രണ്ട് സംഖ്യകൾ ഗുണിക്കുന്നു  
- `divide(a, b)` - ഒന്നാം സംഖ്യ രണ്ടാമത്തേത് കൊണ്ട് വിഭജിക്കുന്നു (സൂന്യ പരിശോധനയോടെ)  
- `power(base, exponent)` - ആധാരത്തെ ഒരു ഘാതത്തിലേക്ക് ഉയർത്തുന്നു  
- `squareRoot(number)` - സ്ക്വയർ റूटു കണക്കാക്കുന്നു (നെഗറ്റീവ് പരിശോധനയോടെ)  
- `modulus(a, b)` - വിഭജന ശേഷി നൽകുന്നു  
- `absolute(number)` - നീഗറ്റീവ് മൂല്യം ഒഴിവാക്കി തീരുന്നു  
- `help()` - എല്ലാ പ്രവർത്തനങ്ങളെക്കുറിച്ചും വിവരങ്ങൾ നൽകുന്നു  

### 3. നേരിട്ട് MCP ക്ലയന്റ്

**ഫയൽ:** `SDKClient.java`

ഈ ക്ലയന്റ് AI ഉപയോഗിക്കാതെ നേരിട്ട് MCP സെർവറുമായി സംസാരിക്കുന്നു. കൃത്യമായ കല്കുലേറ്റർ ഫങ്ഷനുകൾ കൈമാറുന്നു:

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
1. നിർമ്മാതൃ മാതൃക ഉപയോഗിച്ച് `http://localhost:8080` എന്ന സെർവറിലേക്ക് കണക്ട് ചെയ്യും  
2. ലഭ്യമായ എല്ലാ ടൂളുകളും ലിസ്റ്റ് ചെയ്യുന്നു (അഥവാ കല്കുലേറ്റർ ഫങ്ഷനുകൾ)  
3. കൃത്യമായ പാരാമീറ്ററുകളുമായി പ്രത്യേക ഫങ്ഷനുകൾ വിളിക്കുന്നു  
4. ഫലങ്ങൾ നേരിട്ട് പ്രിന്റ് ചെയ്യുന്നു  

**കുറിപ്പ്:** ഈ ഉദാഹരണം Spring AI 1.1.0-SNAPSHOT ഡിപെൻഡൻസി ഉപയോഗിക്കുന്നു, ഇത് `WebFluxSseClientTransport`-ന് ബിൽഡർ പാറ്റേൺ പരിചയപ്പെടുത്തി. പഴയ സ്റ്റേബിൾ പതിപ്പുകൾ ഉപയോഗിക്കുന്നുവെങ്കിൽ, നേരിട്ടുള്ള കൺസ്ട്രക്ടർ ഉപയോഗിക്കേണ്ടതുണ്ടാകും.

**മൊത്തം എപ്പോൾ ഉപയോഗിക്കണം:** നിങ്ങൾക്ക് ഏതൊരു കണക്കാക്കൽ നിർബന്ധമായും ചെയ്യണം എങ്കിൽ പ്രോഗ്രാമാറ്റിക്കായി വിളിക്കാനാഗ്രഹിക്കുമ്പോൾ.

### 4. AI-പവേർഡ് ക്ലയന്റ്

**ഫയൽ:** `LangChain4jClient.java`

ഈ ക്ലയന്റ് GPT-4o-mini എന്ന AI മോഡൽ ഉപയോഗിച്ച് ഓട്ടോമാറ്റിക് ആയി എവിടെ ഏത് കല്കുലേറ്റർ ടൂളുകൾ ഉപയോഗിക്കണമെന്ന് തീരുമാനിക്കുന്നു:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // AI മോഡൽ സജ്ജമാക്കുക (Azure AI Foundry, Microsoft Entra ID വഴി കീരഹിതമായ പ്രാമാണീകരണം)
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

        // ഞങ്ങളുടെ കാൽക്കുലേറ്റർ MCP സെർവറുമായി കണക്ട് ചെയ്യുക
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // AI എന്ത് ചെയ്യും എന്ന് കാണിക്കുന്നു
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // AIന് ഞങ്ങളുടെ കാൽക്കുലേറ്റർ ഉപകരണങ്ങൾ ഉപയോഗിക്കുന്നതിന് പ്രവേശനം നൽകുക
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // ഞങ്ങളുടെ കാൽക്കുലേറ്റർ ഉപയോഗിക്കാൻ കഴിയുന്ന AI ബോട്ട് സൃഷ്ടിക്കുക
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // ഇന്ന് നാം AI-ന് സ്വാഭാവിക ഭാഷയിൽ കണക്കുകൾ ചെയ്യാൻ അഭ്യർത്ഥിക്കാം
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```
  
**ഇത് ചെയ്യുന്നത്:**  
1. കീലെസ് ഓതെൻറിക്കേഷൻ (Microsoft Entra ID) ഉപയോഗിച്ച് AI മോഡൽ കണക്ഷൻ സൃഷ്ടിക്കുന്നു  
2. AI-നെ കല്കുലേറ്റർ MCP സർവറുമായി ബന്ധിപ്പിക്കുന്നു  
3. AI-ന് എല്ലാ കല്കുലേറ്റർ ടൂളുകളിലേക്കും ആക്‌സസ് നൽകുന്നു  
4. "24.5നും 17.3നും സംയോജനം കണക്കാക്കുക" പോലുള്ള സ്വാഭാവികഭാഷ അഭ്യർത്ഥനകൾ അനുവദിക്കുന്നു  

**AI സ്വയം ചെയ്യുന്നത്:**  
- നിങ്ങൾക്ക് സംഖ്യകൾ കൂട്ടണമെന്ന് മനസ്സിലാക്കുന്നു  
- `add` ടൂൾ തിരഞ്ഞെടുക്കുന്നു  
- `add(24.5, 17.3)` വിളിക്കുന്നു  
- പ്രകൃതിദത്ത മറുപടി ഫോർമാറ്റിൽ ഫലം നൽകുന്നു  

## ഉദാഹരണങ്ങള്‍ ഓടിക്കുന്നു

### ഘട്ടം 1: കല്കുലേറ്റർ സേർവർ തുടങ്ങുക

ആദ്യമേ സൈൻ ഇൻ ചെയ്ത് Azure AI Foundry എന്റ്പോയിന്റ് ക്രമീകരിക്കുക (AI ക്ലയന്റിനായി — കീലെസ് ഓഥേന്റ്, എങ്കിലും API കീ ആവശ്യമില്ല):

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
  
സേർവർ തുടങ്ങുക:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```
  
സേർവർ `http://localhost:8080`-ൽ ആരംഭിക്കും. നിങ്ങൾക്ക് ഇങ്ങനെ കാണാം:
```
Started McpServerApplication in X.XXX seconds
```
  
### ഘട്ടം 2: നേരിട്ട് ക്ലയന്റ് ഉപയോഗിച്ച് പരീക്ഷിക്കുക

**പുതിയ** ടേർമിനൽ തുറന്ന്, സേർവർ ഓടുമ്പോൾ നേരിട്ട് MCP ക്ലയന്റ് ഓടിക്കുക:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```
  
ഇവിടെ ഇങ്ങനെ ഔട്ട്പുട്ട് കിട്ടും:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```
  
### ഘട്ടം 3: AI ക്ലയന്റ് ഉപയോഗിച്ച് പരീക്ഷിക്കുക

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```
  
AI സ്വതഃസിദ്ധമായി ടൂളുകൾ ഉപയോഗിക്കുന്നത് കാണാൻ കഴിയും:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```
  
### ഘട്ടം 4: MCP സേർവർ അടയ്ക്കുക

പരിശോധനകൾ കഴിഞ്ഞാൽ, AI ക്ലയന്റ് പ്രവർത്തനം നിർത്താൻ അതിന്റെ ടേർമിനലിൽ `Ctrl+C` അമർത്തുക. MCP സർവർ പ്രവർത്തനം നിർത്തതുവരെ ഓടും.  
സേർവറിനെ നിർത്താൻ, അതു ഓടുന്ന ടേർമിനലിൽ `Ctrl+C` അമർത്തുക.

## എങ്ങിനെ എല്ലാം ചേർന്ന് പ്രവർത്തിക്കുന്നു

AI-നോട് "5 + 3 എന്താണ്?" എന്ന് ചോദിക്കുമ്പോഴുള്ള പൂര്‍ണ പ്രവാഹം ഇവിടെയാണ്:

1. **നിങ്ങൾ** സ്വാഭാവിക ഭാഷയില്‍ AI-നോട് ചോദിക്കുന്നു  
2. **AI** നിങ്ങളുടെ അഭ്യര്‍ഥന വിശകലനം ചെയ്ത് കൂട്ടല്‍ വേണമെന്നും തിരിച്ചറിഞ്ഞു  
3. **AI** MCP സർവറിലെ `add(5.0, 3.0)` വിളിക്കുന്നു  
4. **കല്കുലേറ്റർ സർവീസ്** `5.0 + 3.0 = 8.0` കണക്കാക്കുന്നു  
5. **കള്കുലേറ്റർ സർവീസ്** `"5.00 + 3.00 = 8.00"` എന്ന ഫലം തിരികെ നൽകുന്നു  
6. **AI** ഫലം സ്വീകരിച്ച് പ്രകൃതിദത്ത മറുപടി തയ്യാറാക്കുന്നു  
7. **നിങ്ങൾക്ക്** ലഭിക്കുന്നത്: "5നും 3നും കൂട്ടിയുനർവിൽ 8 ആണ്"  

## അടുത്ത ഘട്ടങ്ങൾ

കൂടുതല്‍ ഉദാഹരണങ്ങള്‍ക്കായി, [Chapter 04: Practical samples](../README.md) കാണുക

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**അറിയിപ്പ്**:
ഈ രേഖ AI പരിഭാഷാ സേവനം [Co-op Translator](https://github.com/Azure/co-op-translator) ഉപയോഗിച്ച് പരിഭാഷപ്പെടുത്തിയതാണ്. ഞങ്ങൾ കൃത്യതയ്ക്കായി ശ്രമിക്കുന്നുവെങ്കിലും, ഓട്ടോമേറ്റഡ് പരിഭാഷകളിൽ പിഴവുകൾ അല്ലെങ്കിൽ തെറ്റായ വിവരങ്ങൾ ഉണ്ടാകാൻ സാധ്യതയുണ്ട്. അതിന്റെ സ്വാഭാവിക ഭാഷയിലുള്ള അസൽ രേഖയാണ് പ്രാമാണികമായ ഉറവിടമായി പരിഗണിക്കേണ്ടത്. നിർണായകമായ വിവരങ്ങൾക്ക്, പ്രൊഫഷണൽ മനുഷ്യ പരിഭാഷ ശുപാർശ ചെയ്യുന്നു. ഈ പരിഭാഷ ഉപയോഗിച്ച് ഉണ്ടാകുന്ന തെറ്റിദ്ധാരണകൾ അല്ലെങ്കിൽ തെറ്റായ വ്യാഖ്യാനങ്ങൾക്കായി ഞങ്ങൾ ഉത്തരവാദികളല്ല.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->