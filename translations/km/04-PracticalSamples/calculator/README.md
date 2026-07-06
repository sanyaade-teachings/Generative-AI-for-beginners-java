# មេរៀនគណនា MCP សម្រាប់អ្នកចាប់ផ្ដើម

## តារាងមាតិកា

- [អ្វីដែលអ្នកនឹងរៀន](#អ្វីដែលអ្នកនឹងរៀន)
- [លក្ខខណ្ឌមុនពេលចាប់ផ្ដើម](#លក្ខខណ្ឌមុនពេលចាប់ផ្ដើម)
- [ការយល់ដឹងអំពីរចនាសម្ព័ន្ធគម្រោង](#ការយល់ដឹងអំពីរចនាសម្ព័ន្ធគម្រោង)
- [ការពន្យល់អំពីរបង្រួមសំខាន់ៗ](#core-components-explained)
  - [1. កម្មវិធីសំខាន់](#1-កម្មវិធីសំខាន់)
  - [2. សេវាកម្មគណនា](#2-សេវាកម្មគណនា)
  - [3. អតិថិជន MCP ផ្ទាល់](#3-អតិថិជន-mcp-ផ្ទាល់)
  - [4. អតិថិជនដែលមានថាមពល AI](#4-អតិថិជនដែលមានថាមពល-ai)
- [ការប្រតិបត្តិនូវឧទាហរណ៍](#ការប្រតិបត្តិនូវឧទាហរណ៍)
- [របៀបធ្វើការជាមួយគ្នាទាំងអស់](#របៀបធ្វើការជាមួយគ្នាទាំងអស់)
- [ជំហានបន្ទាប់](#ជំហានបន្ទាប់)

## អ្វីដែលអ្នកនឹងរៀន

មេរៀននេះពន្យល់ពីរបៀបបង្កើតសេវាកម្មគណនា ដោយប្រើ Model Context Protocol (MCP)។ អ្នកនឹងយល់ដឹងពី៖

- របៀបបង្កើតសេវាកម្មដែល AI អាចប្រើជាឧបករណ៍
- របៀបកំណត់ទំនាក់ទំនងផ្ទាល់ជាមួយសេវាកម្ម MCP
- របៀបដែលម៉ូដែល AI អាចជ្រើសរើសឧបករណ៍ដែលត្រូវប្រើដោយស្វ័យប្រវត្តិ
- ភាពខុសគ្នារវាងការហៅពProtocol MCP ផ្ទាល់ និងការបន្តភ្ជាប់ជាង AI

## លក្ខខណ្ឌមុនពេលចាប់ផ្ដើម

មុនចាប់ផ្ដើម សូមប្រាកដថាអ្នកមាន៖
- Java 21 ឬខ្ពស់ជាងនេះបានដំឡើងរួច
- Maven សម្រាប់គ្រប់គ្រងការពឹងផ្អែក
- ការបញ្ជូនម៉ូដែល Azure AI Foundry (សូម provision ជាមួយ `azd up` — មើល [ជំពូកទី 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) ដែលបានចុះហត្ថលេខា `az login` (ប្រើ keyless auth)
- ការយល់ដឹងមូលដ្ឋានអំពី Java និង Spring Boot

## ការយល់ដឹងអំពីរចនាសម្ព័ន្ធគម្រោង

គម្រោងគណនាមានឯកសារសំខាន់ៗដូចខាងក្រោម៖

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

## Core Components Explained

### 1. កម្មវិធីសំខាន់

**File:** `McpServerApplication.java`

នេះជាច្រកចូលសម្រាប់សេវាកម្មគណនានេះ។ វាជាកម្មវិធី Spring Boot ស្តង់ដារមានការបន្ថែមពិសេសមួយ៖

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
- ចាប់ផ្ដើមប្រព័ន្ធ Spring Boot វែបសើវើលើព្រិវ 8080
- បង្កើត `ToolCallbackProvider` ដែលធ្វើឲ្យមុខងារគណនារបស់យើងអាចប្រើបានជាឧបករណ៍ MCP
- @Bean សញ្ញាបង្ហាញថា Spring នឹងគ្រប់គ្រងវាជាគ្រឿងផ្សំនៃកម្មវិធីដែលផ្នែកផ្សេងអាចប្រើបាន

### 2. សេវាកម្មគណនា

**File:** `CalculatorService.java`

នេះជាគន្លងដែលគណនាគ្រប់យ៉ាងកើតឡើង។ មុខងារនីមួយៗត្រូវបានសម្គាល់ជាមួយ `@Tool` ដើម្បីធ្វើឲ្យវាអាចប្រើបានតាមរយៈ MCP៖

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
    
    // ប្រតិបត្តិការគណនា​បន្ថែម​ទៀត...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**លក្ខណៈសំខាន់៖**

1. **`@Tool` Annotation**៖ បង្ហាញថាមុខងារនេះអាចត្រូវបានហៅដោយអតិថិជនខាងក្រៅ
2. **ការពិពណ៌នាច្បាស់លាស់**៖ ឧបករណ៍នីមួយៗមានការពិពណ៌នាដើម្បីជួយម៉ូដែល AI យល់ពីពេលដែលគួរប្រើវា
3. **ទ្រង់ទ្រាយត្រឡប់លទ្ធផលឆ្លាតវៃ**៖ ប្រតិបត្តិការទាំងអស់ត្រឡប់មកជាខ្សែអក្សរដែលមនុស្សអាចអានបាន ដូចជា "5.00 + 3.00 = 8.00"
4. **ការគ្រប់គ្រងកំហុស**៖ ការចែកដោយសូន្យ និងដើមការ​យក​សម្ងាត់អវិជ្ជមាន បង្ហាញសារ​កំហុស

**ប្រតិបត្តិការដែលមាន៖**
- `add(a, b)` - បូកលេខពីរផ្ទាល់
- `subtract(a, b)` - លែងលេខទីពីរពីលេខទីមួយ
- `multiply(a, b)` - គុណលេខពីរ
- `divide(a, b)` - ចែកលេខទីមួយដោយលេខទីពីរ (ពិនិត្យសូន្យ)
- `power(base, exponent)` - លើកកម្រិតមូលដ្ឋានទៅកម្រិតអគ្គិសនី
- `squareRoot(number)` - គណនារូបមន្តឫសការេ (ពិនិត្យអវិជ្ជមាន)
- `modulus(a, b)` - ត្រឡប់កំណត់ត្រារបស់ការចែក
- `absolute(number)` - ត្រឡប់តម្លៃដាច់ខាត
- `help()` - ត្រឡប់ព័ត៌មានអំពីប្រតិបត្តិការទាំងអស់

### 3. អតិថិជន MCP ផ្ទាល់

**File:** `SDKClient.java`

អតិថិជននេះនិយាយទៅម៉ាស៊ីនមេ MCP ត្រង់ ដោយមិនប្រើ AI។ វាហៅមុខងារគណនាពិសេសជាដៃគូ៖

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
        
        // រាយបញ្ជីឧបករណ៍ដែលមាន
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // ហៅមុខងារគណនា​ជាក់លាក់
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
1. **ភ្ជាប់** ទៅម៉ាស៊ីនមេគណនា `http://localhost:8080` ដោយប្រើ builder pattern
2. **បញ្ជី** ឧបករណ៍ទាំងអស់ដែលមាន (មុខងារគណនារបស់យើង)
3. **ហៅ** មុខងារពិសេសជាមួយប៉ារ៉ាម៉ែត្រត្រឹមត្រូវ
4. **បោះពុម្ព** លទ្ធផលដោយផ្ទាល់

**ចំណាំ៖** ឧទាហរណ៍នេះប្រើ Spring AI 1.1.0-SNAPSHOT ដែលបានណែនាំ builder pattern សម្រាប់ `WebFluxSseClientTransport`។ ប្រសិនបើអ្នកប្រើកំណែចាស់រឹងមាំ អ្នកប្រហែលជាត្រូវប្រើ constructor ដោយផ្ទាល់ជំនួស។

**ពេលដែលគួរប្រើ៖** ពេលដែលអ្នកដឹងច្បាស់ថាត្រូវគណនាអ្វីហើយចង់ហៅវាដោយកម្មវិធី។

### 4. អតិថិជនដែលមានថាមពល AI

**File:** `LangChain4jClient.java`

អតិថិជននេះប្រើម៉ូដែល AI (GPT-4o-mini) ដែលអាចសម្រេចចិត្តដោយស្វ័យប្រវត្តិថាត្រូវប្រើឧបករណ៍គណនាណា៖

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // រៀបចំម៉ូឌែល AI (Azure AI Foundry, ការផ្ទៀងផ្ទាត់អត្តសញ្ញាណគ្មានកូនសោជាមួយ Microsoft Entra ID)
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

        // ភ្ជាប់ទៅម៉ាស៊ីនបម្រើ MCP គណនាគ្រាន់របស់យើង
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // បង្ហាញអ្វីដែល AI កំពុងធ្វើ
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // ផ្ដល់ឱ្យ AI នូវការចូលប្រើឧបករណ៍គណនាគ្រាន់របស់យើង
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // បង្កើតប្រដាប់បង្ហាញ AI ដែលអាចប្រើគណនាគ្រាន់របស់យើង
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // ឥឡូវនេះយើងអាចស្នើសុំ AI ធ្វើការគណនានៅក្នុងភាសាធម្មជាតិបានហើយ
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**អ្វីដែលវាធ្វើ៖**
1. **បង្កើត** ការតភ្ជាប់ម៉ូដែល AI ដោយប្រើ keyless authentication (Microsoft Entra ID)
2. **ភ្ជាប់** AI ទៅម៉ាស៊ីនមេ MCP គណនារបស់យើង
3. **ផ្តល់** អោយ AI អាចប្រើឧបករណ៍គណនាទាំងអស់
4. **អនុញ្ញាត** សំណើជាភាសាបច្ចុប្បន្នដូចជា "គណនាផលបូកនៃ 24.5 និង 17.3"

**AI បើកប្រាស់ដោយស្វ័យប្រវត្តិ៖**
- យល់ថាអ្នកចង់បូកលេខ
- ជ្រើសរើសឧបករណ៍ `add`
- ហៅ `add(24.5, 17.3)`
- ត្រឡប់លទ្ធផលជាចម្លើយធម្មតា

## ការប្រតិបត្តិនូវឧទាហរណ៍

### ជំហាន 1៖ ចាប់ផ្ដើមម៉ាស៊ីនមេគណនា

ដំបូង ចុះហត្ថលេខា និងកំណត់ចំណុចផ្តើម Azure AI Foundry (ចាំបាច់សម្រាប់អតិថិជន AI — keyless auth, គ្មាន API key):

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

ចាប់ផ្ដើមម៉ាស៊ីនមេ៖
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

ម៉ាស៊ីនមេនឹងចាប់ផ្ដើមនៅ `http://localhost:8080`។ អ្នកនឹងមើលឃើញ៖
```
Started McpServerApplication in X.XXX seconds
```

### ជំហាន 2៖ សាកល្បងជាមួយអតិថិជនផ្ទាល់

នៅក្នុង terminal ថ្មីមួយដែលម៉ាស៊ីនមេកំពុងដំណើរការ រត់អតិថិជន MCP ផ្ទាល់៖
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

ពេលអ្នកបានធ្វើការបញ្ចប់សាកល្បង អ្នកអាចបញ្ឈប់អតិថិជន AI ដោយចុច `Ctrl+C` នៅ terminal របស់វា។ ម៉ាស៊ីនមេ MCP នឹងនៅតែដំណើរការរហូតដល់អ្នកបញ្ឈប់វា។
ដើម្បីបិទម៉ាស៊ីនមេ ចុច `Ctrl+C` នៅ terminal ដែលកំពុងដំណើរការ។

## របៀបធ្វើការជាមួយគ្នាទាំងអស់

នេះជាច្រកដំណើរការពេញលេញពេលអ្នកសួរ AI ថា "5 + 3 ជា​ប៉ុន្មាន?":

1. **អ្នក** សួរ AI ជាភាសាធម្មតា
2. **AI** វិភាគសំណើររបស់អ្នក និងដឹងថាអ្នកចង់បូកលេខ
3. **AI** ហៅ MCP ម៉ាស៊ីនមេ៖ `add(5.0, 3.0)`
4. **សេវាកម្មគណនា** ប្រតិបត្តិ៖ `5.0 + 3.0 = 8.0`
5. **សេវាកម្មគណនា** ត្រឡប់៖ `"5.00 + 3.00 = 8.00"`
6. **AI** ទទួលលទ្ធផល និងរៀបចំចម្លើយជាភាសាធម្មតា
7. **អ្នក** ទទួលបាន៖ "ផលបូករបស់ 5 និង 3 គឺ 8"

## ជំហានបន្ទាប់

សម្រាប់ឧទាហរណ៍បន្ថែម សូមមើល [ជំពូក 04៖ ឧទាហរណ៍ប្រើប្រាស់](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ការបដិសេធ**:
ឯកសារនេះត្រូវបានបម្លែងភាសា ដោយប្រើសេវាបម្លែងភាសា AI [Co-op Translator](https://github.com/Azure/co-op-translator)។ ទោះយើងខ្ញុំមានក្តីប្រាថ្នាឱ្យបានច្បាស់លាស់ តែសូមយល់ដឹងថាការបម្លែងដោយស្វ័យប្រវត្តិក៏អាចមានកំហុសឬភាពមិនត្រឹមត្រូវ។ ឯកសារដើមជាភាសាទីតាំងគួរត្រូវបានគេប្រើជាប្រភពច្បាស់លាស់។ សម្រាប់ព័ត៌មានសំខាន់ៗ សូមណែនាំឱ្យប្រើប្រាស់ការប្រែដោយមនុស្សជំនាញ។ យើងខ្ញុំមិនទទួលខុសត្រូវចំពោះការយល់ច្រឡំ ឬការបកស្រាយខុសបន្ទាប់ពីការប្រើប្រាស់ការបម្លែងនេះនោះទេ។
<!-- CO-OP TRANSLATOR DISCLAIMER END -->