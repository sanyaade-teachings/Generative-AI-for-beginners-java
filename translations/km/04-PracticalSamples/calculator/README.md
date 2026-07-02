# មេរៀនគណនី MCP សម្រាប់អ្នកចាប់ផ្តើម

## តារាងមាតិកា

- [អ្វីដែលអ្នកនឹងរៀន](#អ្វីដែលអ្នកនឹងរៀន)
- [លក្ខខណ្ឌមុនបើកផ្លូវ](#លក្ខខណ្ឌមុនបើកផ្លូវ)
- [ការយល់ដឹងពីរចនាសម្ព័ន្ធគម្រោង](#ការយល់ដឹងពីរចនាសម្ព័ន្ធគម្រោង)
- [ធាតុសំខាន់ៗជាអត្ថាធិប្បាយ](#ធាតុសំខាន់ៗជាអត្ថាធិប្បាយ)
  - [1. កម្មវិធីចម្បង](#1-កម្មវិធីចម្បង)
  - [2. សេវាកម្មគណនាកម្ម](#2-សេវាកម្មគណនាកម្ម)
  - [3. អតិថិជន MCP ផ្ទាល់](#3-អតិថិជន-mcp-ផ្ទាល់)
  - [4. អតិថិជនបិទថាមពលដោយ AI](#4-អតិថិជនបិទថាមពលដោយ-ai)
- [ការរត់ឧទាហរណ៍](#ការរត់ឧទាហរណ៍)
- [របៀបដែលវាដំណើរការជារួម](#របៀបដែលវាដំណើរការជារួម)
- [ជំហានបន្ទាប់](#ជំហានបន្ទាប់)

## អ្វីដែលអ្នកនឹងរៀន

មេរៀននេះពន្យល់ពីរបៀបបង្កើតសេវាកម្មគណនាកម្មដោយប្រើពិធីការបរិបទម៉ូដែល (Model Context Protocol - MCP)។ អ្នកនឹងយល់៖

- របៀបបង្កើតសេវាកម្មដែល AI អាចប្រើជាឧបករណ៍
- របៀបដំឡើងការទំនាក់ទំនងផ្ទាល់ខ្លួនជាមួយសេវាកម្ម MCP
- របៀបដែលម៉ូដែល AI អាចជ្រើសរើសឧបករណ៍ដោយស្វ័យប្រវត្តិ
-	javaដម្លែងរវាងការហៅពិធីការផ្ទាល់ និងការប្រាស្រ័យទាក់ទងជាមួយ AI

## លក្ខខណ្ឌមុនបើកផ្លូវ

មុនចាប់ផ្តើម សូមប្រាកដថាអ្នកមាន៖
- វ័រលេខ Java 21 ឬខ្ពស់ជាងនេះបានដំឡើង
- Maven សម្រាប់គ្រប់គ្រងអាស្រ័យភាព
- ការដាក់ម៉ូដែល Azure AI Foundry (Provision ជាមួយ `azd up` — មើល [ជំពូក 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) ដែលបានចុះឈ្មោះដោយ `az login` (auth keyless)
- ការយល់ដឹងមូលដ្ឋានពី Java និង Spring Boot

## ការយល់ដឹងពីរចនាសម្ព័ន្ធគម្រោង

គម្រោងគណនាកម្មមានឯកសារសំខាន់ៗមួយចំនួន៖

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

## ធាតុសំខាន់ៗជាអត្ថាធិប្បាយ

### 1. កម្មវិធីចម្បង

**ឯកសារ៖** `McpServerApplication.java`

នេះគឺជាចំណុចចាប់ផ្តើមសម្រាប់សេវាកម្មគណនាកម្មរបស់យើង។ វាជាកម្មវិធី Spring Boot ធម្មតាមួយដែលមានការផ្តល់បន្ថែមពិសេសមួយ៖

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

**អ្វីដែលវាធ្វើ៖**
- ចាប់ផ្តើមម៉ាស៊ីនមេតំបន់បណ្ដាញ Spring Boot លើកំពង់ផែ 8080
- បង្កើត `ToolCallbackProvider` ដែលធ្វើឲ្យវិធីសាស្រ្តគណនារបស់យើងអាចប្រើបានជា​ឧបករណ៍ MCP
- ស្លាក `@Bean` ប្រាប់ Spring ឲ្យគ្រប់គ្រងវាជាផ្នែកដែលផ្នែកផ្សេងទៀតអាចប្រើបាន

### 2. សេវាកម្មគណនាកម្ម

**ឯកសារ៖** `CalculatorService.java`

នេះជាកន្លែងកើតមានគណនា។ វិធីសាស្រ្តនីមួយៗត្រូវបានសម្គាល់ដោយ `@Tool` ដើម្បីអោយវាអាចប្រើប្រាស់តាមរយៈ MCP ៖

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
    
    // ប្រតិបត្តិការឈ្មួញបន្ថែម...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**លក្ខណៈសំខាន់ៗ៖**

1. **ស្លាក `@Tool`**៖ ប្រាប់ MCP ថាវិធីសាស្រ្តនេះអាចហៅបានពីអតិថិជនខាងក្រៅ
2. **ការពិពណ៌នាច្បាស់លាស់**៖ ឧបករណ៍នីមួយៗមានការពិពណ៌នាដើម្បីជួយម៉ូដែល AI យល់ពេលណាដែលត្រូវប្រើវា
3. **ទ្រង់ទ្រាយត្រឡប់មកវិញជាប់គ្នា**៖ ប្រតិបត្តិការទាំងអស់ត្រឡប់មកជាខ្សែអក្សរដែលមនុស្សអាចអានបាន ដូចជា "5.00 + 3.00 = 8.00"
4. **ការដោះស្រាយកំហុស**៖ ការចែកដោយសូន្យ និងការរុំគោលខាងអវិជ្ជមានត្រឡប់មកជាសារ​កំហុស

**ប្រតិបត្តិការដែលអាចប្រើបាន៖**
- `add(a, b)` - បូកលេខពីរ
- `subtract(a, b)` - ដកលេខទីពីរពីលេខទីមួយ
- `multiply(a, b)` - គុណលេខពីរ
- `divide(a, b)` - ចែកលេខទីមួយដោយលេខទីពីរ (ពិនិត្យសូន្យ)
- `power(base, exponent)` - លើកមូលដ្ឋានទៅកម្រិតអគ្គីសនី
- `squareRoot(number)` - គណនគន្លងឫសការេ (ពិនិត្យអវិជ្ជមាន)
- `modulus(a, b)` - ត្រឡប់សរីរាង្គសន្លឹកចុងក្រោយ
- `absolute(number)` - ត្រឡប់តម្លៃល្អិតជាសំណង់
- `help()` - ត្រឡប់ពត៌មានអំពីប្រតិបត្តិការ​ទាំងអស់

### 3. អតិថិជន MCP ផ្ទាល់

**ឯកសារ៖** `SDKClient.java`

អតិថិជននេះនិយាយផ្ទាល់ទៅម៉ាស៊ីនមេ MCP ដោយគ្មានការប្រើ AI។ វាហៅមុខងារគណនាពិសេសដោយដៃ៖

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
        
        // បញ្ជីឧបករណ៍ដែលមាន
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // ហៅមុខងារគណនាផ្សេងៗ
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

**អ្វីដែលវាធ្វើ៖**
1. **ភ្ជាប់** ទៅម៉ាស៊ីនមេគណនាកម្មនៅ `http://localhost:8080` ដោយប្រើរបៀបរចនាស្ថាបត្យកម្ម builder
2. **បង្ហាញ** ឧបករណ៍ទាំងអស់ដែលមាន (មុខងារគណនារបស់យើង)
3. **ហៅ** មុខងារពិសេសជាមួយប៉ារ៉ាម៉ែត្រ​ត្រឹមត្រូវ
4. **បោះពុម្ភ** លទ្ធផលដោយផ្ទាល់

**ចំណាំ៖** ឧទាហរណ៍នេះប្រើ Maven dependency Spring AI 1.1.0-SNAPSHOT ដែលណែនាំរបៀបរចនាស្ថាបត្យកម្ម builder សម្រាប់ `WebFluxSseClientTransport`។ ប្រសិនបើអ្នកប្រើកំណែចាស់ជាងនេះ អ្នកអាចត្រូវប្រើ constructor ផ្ទាល់។

**ពេលណាដែលត្រូវប្រើ៖** ពេលអ្នកដឹងច្បាស់ថាប្រាក់គណនាអ្វី ហើយចង់ហៅដោយកម្មវិធី។

### 4. អតិថិជនបិទថាមពលដោយ AI

**ឯកសារ៖** `LangChain4jClient.java`

អតិថិជននេះប្រើម៉ូដែល AI (GPT-4o-mini) ដែលអាចសម្រេចចិត្តដោយស្វ័យប្រវត្តិថាតើត្រូវប្រើឧបករណ៍គណនាណាដើម្បីប្រើ៖

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // រៀបចំម៉ូដែល AI (Azure AI Foundry, អត្ថានុញ្ញាតដោយគ្មានកូនសោរដោយ Microsoft Entra ID)
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

        // ខ្សែករចូលទៅម៉ាស៊ីនគណនាតុល្យភាព MCP របស់យើង
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // បង្ហាញអ្វីដែល AI កំពុងធ្វើ
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // ផ្ដល់ការចូលដំណើរការដល់ឧបករណ៍គណនាតុល្យភាពរបស់យើងឲ្យ AI
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // បង្កើតបុត AI ដែលអាចប្រើឧបករណ៍គណនាតុល្យភាពរបស់យើង
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // ឥឡូវនេះយើងអាចសុំអោយ AI ធ្វើការគណនាបានជាភាសាធម្មជាតិ
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**អ្វីដែលវាធ្វើ៖**
1. **បង្កើត** ការតភ្ជាប់ម៉ូដែល AI ដោយប្រើ token GitHub របស់អ្នក
2. **ភ្ជាប់** AI ទៅម៉ាស៊ីនមេ MCP គណនាកម្មរបស់យើង
3. **ផ្ដល់អោយ** AI ចូលដំណើរការទៅឧបករណ៍គណនារបស់យើងទាំងមូល
4. **អនុញ្ញាត** អោយមានសំណើជាភាសាធម្មជាតិ ដូចជា "គណនាចំនួនបូករវាង 24.5 និង 17.3"

**AI ដំណើរការតាមស្វ័យប្រវត្តិ៖**
- យល់ថាអ្នកចង់បូកលេខ
- ជ្រើសឧបករណ៍ `add`
- ហៅ `add(24.5, 17.3)`
- ត្រឡប់លទ្ធផលជាការឆ្លើយឆ្លងភាសាធម្មជាតិ

## ការរត់ឧទាហរណ៍

### ជំហាន 1៖ ចាប់ផ្តើមម៉ាស៊ីនមេគណនាកម្ម

ដំបូង ចុះឈ្មោះ និងកំណត់ចំណុចចេញ Azure AI Foundry របស់អ្នក (ចាំបាច់សម្រាប់អតិថិជន AI — auth keyless គ្មាន API key):

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

ចាប់ផ្តើមម៉ាស៊ីនមេ៖
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

ម៉ាស៊ីនមេនឹងចាប់ផ្តើមនៅ `http://localhost:8080`។ អ្នកគួរតែឃើញ៖
```
Started McpServerApplication in X.XXX seconds
```

### ជំហាន 2៖ សាកល្បងជាមួយអតិថិជនផ្ទាល់

នៅក្នុងចំណុចបញ្ជារថ្មីមួយ ដែលម៉ាស៊ីនមេស្ថិតនៅតែដំណើរការ ចាប់ផ្តើមអតិថិជន MCP ផ្ទាល់៖
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

អ្នកនឹងឃើញលទ្ធផលដូចជា៖
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### ជំហាន 3៖ សាកល្បងជាមួយអតិថិជន AI

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

អ្នកនឹងឃើញ AI ប្រើឧបករណ៍ដោយស្វ័យប្រវត្តិ៖
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### ជំហាន 4៖ បិទម៉ាស៊ីនមេ MCP

ពេលអ្នកបញ្ចប់ការសាកល្បង អាចបិទអតិថិជន AI ដោយចុច `Ctrl+C` នៅក្នុងចំណុចបញ្ជាររបស់វា។ ម៉ាស៊ីនមេ MCP នឹងបន្តដំណើរការរហូតដល់អ្នកបិទវា។
ដើម្បីបិទម៉ាស៊ីនមេ សូមចុច `Ctrl+C` នៅក្នុងចំណុចបញ្ជារដែលវាកំពុងដំណើរការ។

## របៀបដែលវាដំណើរការជារួម

នេះជាដំណើរការពេញលេញនៅពេលអ្នកសួរប្រសាសត AI "5 + 3 ទៅម៉េច?":

1. **អ្នក** សួរប្រសាសជា​ភាសាធម្មជាតិ
2. **AI** វិភាគសំណើរបស់អ្នក ហើយដឹងថាអ្នកចង់បូក
3. **AI** ហៅម៉ាស៊ីនមេ MCP: `add(5.0, 3.0)`
4. **សេវាកម្មគណនាកម្ម** ប្រតិបត្តិ: `5.0 + 3.0 = 8.0`
5. **សេវាកម្មគណនាកម្ម** ត្រឡប់: `"5.00 + 3.00 = 8.00"`
6. **AI** ទទួលលទ្ធផល ហើយបង្កើតការឆ្លើយតបជាភាសាធម្មជាតិ
7. **អ្នក** ទទួលបាន៖ "ចំនួនបូករបស់ 5 និង 3 គឺ 8"

## ជំហានបន្ទាប់

សម្រាប់ឧទាហរណ៍បន្ថែម សូមមើល [ជំពូក 04៖ ឧទាហរណ៍ជាក់ស្តែង](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ការបដិសេធ**:
ឯកសារនេះត្រូវបានបម្លែងភាសា ដោយប្រើសេវាបម្លែងភាសា AI [Co-op Translator](https://github.com/Azure/co-op-translator)។ ទោះយើងខ្ញុំមានក្តីប្រាថ្នាឱ្យបានច្បាស់លាស់ តែសូមយល់ដឹងថាការបម្លែងដោយស្វ័យប្រវត្តិក៏អាចមានកំហុសឬភាពមិនត្រឹមត្រូវ។ ឯកសារដើមជាភាសាទីតាំងគួរត្រូវបានគេប្រើជាប្រភពច្បាស់លាស់។ សម្រាប់ព័ត៌មានសំខាន់ៗ សូមណែនាំឱ្យប្រើប្រាស់ការប្រែដោយមនុស្សជំនាញ។ យើងខ្ញុំមិនទទួលខុសត្រូវចំពោះការយល់ច្រឡំ ឬការបកស្រាយខុសបន្ទាប់ពីការប្រើប្រាស់ការបម្លែងនេះនោះទេ។
<!-- CO-OP TRANSLATOR DISCLAIMER END -->