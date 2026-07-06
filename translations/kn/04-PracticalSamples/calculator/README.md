# MCP ಕ್ಯಾಲ್ಕುಲೇಟರ್ ಟ್ಯುಟೋರಿಯಲ್ ಆರಂಭಿಕರಿಗಾಗಿ

## ವಿಷಯಗಳ ಪಟ್ಟಿವು

- [ನೀವು ಏನು ಕಲಿಯಲಿದ್ದೀರಿ](#ನೀವು-ಏನು-ಕಲಿಯಲಿದ್ದೀರಿ)
- [ಮುಂಚಿತ ಅಗತ್ಯಗಳು](#ಮುಂಚಿತ-ಅಗತ್ಯಗಳು)
- [ಪ್ರಾಜೆಕ್ಟ್ ರಚನೆಯನ್ನು ಅರ್ಥಮಾಡಿಕೊಳ್ಳುವುದು](#ಪ್ರಾಜೆಕ್ಟ್-ರಚನೆಯನ್ನು-ಅರ್ಥಮಾಡಿಕೊಳ್ಳುವುದು)
- [ಕೋರ್ ಕಂಪೋನಂಟ್‌ಗಳನ್ನು ವಿವರಿಸಲಾಗಿದೆ](#ಕೋರ್-ಕಂಪೋನಂಟ್‌ಗಳನ್ನು-ವಿವರಿಸಲಾಗಿದೆ)
  - [1. ಮುಖ್ಯ ಅಪ್ಲಿಕೇಶನ್](#1-ಮುಖ್ಯ-ಅಪ್ಲಿಕೇಶನ್)
  - [2. ಕ್ಯಾಲ್ಕುಲೇಟರ್ ಸರ್ವಿಸ್](#2-ಕ್ಯಾಲ್ಕುಲೇಟರ್-ಸರ್ವಿಸ್)
  - [3. ನೇರ MCP ಕ್ಲೈಂಟ್](#3-ನೇರ-mcp-ಕ್ಲೈಂಟ್)
  - [4. AI-ಚಾಲಿತ ಕ್ಲೈಂಟ್](#4-ai-ಚಾಲಿತ-ಕ್ಲೈಂಟ್)
- [ಉದಾಹರಣೆಗಳನ್ನು ಚಾಲನೆ ಮಾಡುವುದು](#ಉದಾಹರಣೆಗಳನ್ನು-ಚಾಲನೆ-ಮಾಡುವುದು)
- [ಎಲ್ಲದರ ಒಟ್ಟಿಗೆ ಕೆಲಸ ಮಾಡುವ ವಿಧಾನ](#ಎಲ್ಲದರ-ಒಟ್ಟಿಗೆ-ಕೆಲಸ-ಮಾಡುವ-ವಿಧಾನ)
- [ಮುಂದಿನ ಹಂತಗಳು](#ಮುಂದಿನ-ಹಂತಗಳು)

## ನೀವು ಏನು ಕಲಿಯಲಿದ್ದೀರಿ

ಈ ಟ್ಯುಟೋರಿಯಲ್‌ನಲ್ಲಿ Model Context Protocol (MCP) ಬಳಸಿಕೊಂಡು ಕ್ಯಾಲ್ಕುಲೇಟರ್ ಸರ್ವಿಸ್ನನ್ನು ಹೇಗೆ ನಿರ್ಮಿಸುವುದು ಎಂಬುದನ್ನು ವಿವರಿಸಲಾಗುತ್ತದೆ. ನೀವು ತಿಳಿದುಕೊಳ್ಳುವುದು:

- AI ಉಪಕರಣವಾಗಿ ಬಳಸಬಹುದಾದ ಸರ್ವಿಸ్న್ನ ನಿರ್ಮಾಣ ಹೇಗೆ ಮಾಡುವುದು
- MCP ಸರ್ವಿಸುಗಳೊಂದಿಗೆ ನೇರ ಸಂವಹನವನ್ನು ಸ್ಥಾಪಿಸುವುದು ಹೇಗೆ
- AI ಮಾದರಿಗಳು ಸ್ವಯಂಚಾಲಿತವಾಗಿ ಯಾವ ಉಪಕರಣಗಳನ್ನು ಬಳಸಬೇಕೆಂದು ಆಯ್ಕೆಮಾಡುವುದು ಹೇಗೆ
- ನೇರ ಪ್ರೋಟೋಕಾಲ್ ಕರೆ ಮತ್ತು AI ಸಹಾಯಿತ ಸಂವಹನದ ನಡುವಿನ ವ್ಯತ್ಯಾಸ

## ಮುಂಚಿತ ಅಗತ್ಯಗಳು

ಆರಂಭಿಸುವ ಮುಂಚೆ, ಕೆಳಗಿನವುಗಳಿದ್ದಂತೆ ಖಚಿತಪಡಿಸಿಕೊಳ್ಳಿ:
- Java 21 ಅಥವಾ ಹೆಚ್ಚಿನ ಆವೃತ್ತಿ ಇನ್‌ಸ್ಟಾಲ್ ಮಾಡಿರುವುದು
- ನಿರ್ಭರತೆಯ ನಿರ್ವಹಣೆಗೆ Maven
- Azure AI Foundry ಮಾದರಿ ನಿಯೋಜನೆ (ಮಾತ್ರ `azd up` ಮೂಲಕ ವ್ಯವಸ್ಥಾಪಿಸಿ — [ಅಧ್ಯಾಯ 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md) ನೋಡಿ)
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), `az login` ಮೂಲಕ ಸೈನ್ ಇನ್ (ಕೀ ರಹಿತ ಪ್ರಮಾಣಿકરણ)
- Java ಮತ್ತು Spring Boot ಸ್ಥೂಲ ಜ್ಞಾನ

## ಪ್ರಾಜೆಕ್ಟ್ ರಚನೆಯನ್ನು ಅರ್ಥಮಾಡಿಕೊಳ್ಳುವುದು

ಕ್ಯಾಲ್ಕುಲೇಟರ್ ಪ್ರಾಜೆಕ್ಟ್ನಲ್ಲಿ ಕೆಲವು ಪ್ರಮುಖ ಫೈಲುಗಳು ಇವೆ:

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

## ಕೋರ್ ಕಂಪೋನಂಟ್‌ಗಳನ್ನು ವಿವರಿಸಲಾಗಿದೆ

### 1. ಮುಖ್ಯ ಅಪ್ಲಿಕೇಶನ್

**ಫೈಲು:** `McpServerApplication.java`

ಇದು ನಮ್ಮ ಕ್ಯಾಲ್ಕುಲೇಟರ್ ಸರ್ವಿಸ್ನ ಪ್ರವೆಶ ಬಿಂದುವಾಗಿದೆ. ಇದು ಸಾಮಾನ್ಯ Spring Boot ಅಪ್ಲಿಕೇಶನ್ ಆಗಿದ್ದು, ಒಂದೇ ವಿಶೇಷ ಹೆಚ್ಚುವರಿ ಹೊಂದಿದೆ:

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

**ಇದು ಮಾಡುವದು:**
- 8080 ಪೋರ್ಟ್‌ನಲ್ಲಿ Spring Boot ವೆಬ್ ಸರ್ವರ್ ಪ್ರಾರಂಭಿಸುತ್ತದೆ
- ನಮ್ಮ ಕ್ಯಾಲ್ಕುಲೇಟರ್ ವಿಧಾನಗಳನ್ನು MCP ಉಪಕರಣಗಳಾಗಿ ಲಭ್ಯವಾಗಿಸುವ `ToolCallbackProvider` ಸೃಷ್ಟಿಸುತ್ತದೆ
- `@Bean` ಅಂಶ Spring ಅನ್ನು ಈ ಭಾಗವನ್ನು ಇತರವರು ಬಳಸುವ ಕಂಪೋನಂಟ್ ಆಗಿ ನಿರ್ವಹಿಸಲು ಸೂಚಿಸುತ್ತದೆ

### 2. ಕ್ಯಾಲ್ಕುಲೇಟರ್ ಸರ್ವಿಸ್

**ಫೈಲು:** `CalculatorService.java`

ಇಲ್ಲಿ ಎಲ್ಲ ಗಣಿತ ಕಾರ್ಯಗಳು ನಡೆಯುತ್ತವೆ. ಪ್ರತಿ ವಿಧಾನವನ್ನು `@Tool` ಅನ್ನು ಉಪಯೋಗಿಸಿಕೊಂಡು MCP ಮೂಲಕ ಲಭ್ಯವಾಗಿಸುವಂತೆ ಮಾಡಲಾಗಿದೆ:

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
    
    // ಇನ್ನಷ್ಟು ಕ್ಯಾಲ್ಕುಲೇಟರ್ ಕಾರ್ಯಾಚರಣೆಗಳು...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**ಪ್ರಮುಖ ವೈಶಿಷ್ಟ್ಯಗಳು:**

1. **`@Tool` ಅಂಕಣ:** ಇದರಿಂದ MCPಗೆ ತಿಳಿಯುತ್ತದೆ ಈ ವಿಧಾನವನ್ನು ಹೊರಗಿನ ಕ್ಲೈಂಟ್‌ಗಳು ಕರೆ ಮಾಡಬಹುದು
2. **ಸ್ಪಷ್ಟ ವಿವರಣೆಗಳು:** ಪ್ರತಿ ಉಪಕರಣವು ಯಾವಾಗ ಬಳಸಬೇಕೆಂದು AI ಮಾದರಿಗಳಿಗೆ ಸಹಾಯ ಮಾಡುವ ವಿವರಣೆ ಪಡೆದಿದೆ
3. **ಸ್ಥಿರವಾದ ಪ್ರತಿಕ್ರಿಯಾ ಸ್ವರೂಪ:** ಎಲ್ಲಾ ಕಾರ್ಯಾಚರಣೆಗಳು "5.00 + 3.00 = 8.00" ಇವರಂತೆ ಮಾನವ ಓದಬಹುದಾದ ಸ್ಟ್ರಿಂಗ್ ಅನ್ನು ನೀಡುತ್ತವೆ
4. **ದೋಷ ನಿರ್ವಹಣೆ:** ಶೂನ್ಯದಿಂದ ವಿಭಾಗಿಸುವುದು ಮತ್ತು ಋಣಾತ್ಮಕ ವರ್ಗಮೂಲಗಳಿಗான ದೋಷ ಸಂದೇಶಗಳನ್ನು ನೀಡುತ್ತದೆ

**ಲಭ್ಯವಿರುವ ಕಾರ್ಯಾಚರಣೆಗಳು:**
- `add(a, b)` - ಎರಡು ಸಂಖ್ಯೆಗಳ ಮೊತ್ತ ಸೇರಿಸುವುದು
- `subtract(a, b)` - ಎರಡನೇ ಸಂಖ್ಯೆಯಿಂದ ಮೊದಲನ್ನು ಕಡಿಮೆ ಮಾಡುವುದು
- `multiply(a, b)` - ಎರಡು ಸಂಖ್ಯೆಗಳ ಗುಣಾಕಾರ
- `divide(a, b)` - ಮೊದಲ ಸಂಖ್ಯೆಯನ್ನು ಎರಡನೇಹಾಗೆ ಭಾಗಿಸುವುದು (ಶೂನ್ಯ ಪರಿಶೀಲನೆ ಸಹಿತ)
- `power(base, exponent)` - ಮೂಲ ಸಂಖ್ಯೆExponentಶಕ್ತಿ ಮೇಲೆ ಏರಿಸುವುದು
- `squareRoot(number)` - ವರ್ಗಮೂಲ ಲೆಕ್ಕಿಸುವುದು (ಋಣಾತ್ಮಕ ಪರೀಕ್ಷೆ ಕೂಡ)
- `modulus(a, b)` - ವಿಭಾಗದ ಅವಶೇಷವನ್ನು ನೀಡುವುದು
- `absolute(number)` - ಪರಮ ಮೌಲ್ಯವನ್ನು ನೀಡುವುದು
- `help()` - ಎಲ್ಲ ಕಾರ್ಯಾಚರಣೆಗಳ ಬಗ್ಗೆ ಮಾಹಿತಿ ನೀಡುತ್ತದೆ

### 3. ನೇರ MCP ಕ್ಲೈಂಟ್

**ಫೈಲು:** `SDKClient.java`

ಈ ಕ್ಲೈಂಟ್ AI ಬಳಸದೆ ನೇರವಾಗಿ MCP ಸರ್ವರ್‌ಗೆ ಮಾತಾಡುತ್ತದೆ. ಇದು ನಿರ್ದಿಷ್ಟ ಕ್ಯಾಲ್ಕುಲೇಟರ್ ಫಂಕ್ಷನ್‌ಗಳನ್ನು ಕೈಯಿಂದ ಕರೆ ಮಾಡುತ್ತದೆ:

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
        
        // ಲಭ್ಯವಿರುವ ಸಾಧನಗಳನ್ನು ಪಟ್ಟಿ ಮಾಡಿ
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // ನಿರ್ದಿಷ್ಟ ಕ್ಯಾಲ್ಕುಲೇಟರ್ ಫಂಕ್ಷನ್‌ಗಳನ್ನು ಕರೆಮಾಡಿ
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

**ಇದು ಮಾಡುವದು:**
1. `http://localhost:8080` ನಲ್ಲಿ ಕ್ಯಾಲ್ಕುಲೇಟರ್ ಸರ್ವರ್‌ಗೆ ಬಿಲ್ಡರ್ ಮಾದರಿಯನ್ನು ಬಳಸಿ ಸಂಪರ್ಕಿಸುತ್ತದೆ
2. ಲಭ್ಯವಿರುವ ಎಲ್ಲಾ ಉಪಕರಣಗಳ ಪಟ್ಟಿ ತೋರಿಸುತ್ತದೆ (ನಮ್ಮ ಕ್ಯಾಲ್ಕುಲೇಟರ್ ಕಾರ್ಯಗಳು)
3. ನಿಖರವಾದ ಪ್ಯಾರಾಮೀಟರ್ ಗಳೊಂದಿಗೆ ನಿರ್ದಿಷ್ಟ ಕಾರ್ಯಗಳನ್ನು ಕರೆ ಮಾಡುತ್ತದೆ
4. ಫಲಿತಾಂಶಗಳನ್ನು ನೇರವಾಗಿ ಮುದ್ರಿಸುತ್ತದೆ

**ಗಮನಿಸಿರಿ:** ಈ ಉದಾಹರಣೆಯಲ್ಲಿ Spring AI 1.1.0-SNAPSHOT ಅನುಪಾತ ಸಹಿತ `WebFluxSseClientTransport` ಗೆ ಬಿಲ್ಡರ್ ಮಾದರಿಯನ್ನು ಪರಿಚಯಿಸಲಾಗಿದ್ದು, ಹಳೆಯ ಸ್ಥಿರ ಆವೃತ್ತಿಯಲ್ಲಿ ನೇರ ಕಾನ್ಸ್ಟ್ರಕ್ಟರ್ ಬಳಕೆ అవసರವಾಗಬಹುದು.

**ಯಾವಾಗ ಬಳಸಬೇಕು:** ನೀವು ನಿರ್ದಿಷ್ಟವಾಗಿ ಯಾವ ಲೆಕ್ಕಾಚಾರ ಮಾಡಬೇಕೆಂದು ತಿಳಿದಿದ್ದಾಗ ಮತ್ತು ಪ್ರೋಗ್ರಾಮ್ಯುಕ್ತವಾಗಿ ಕರೆಯಬೇಕಾದಾಗ.

### 4. AI-ಚಾಲಿತ ಕ್ಲೈಂಟ್

**ಫೈಲು:** `LangChain4jClient.java`

ಈ ಕ್ಲೈಂಟ್ ಒಂದು AI ಮಾದರಿಯನ್ನು (GPT-4o-mini) ಬಳಸುತ್ತದೆ, ಅದು ಸ್ವಯಂಚಾಲಿತವಾಗಿ ಯಾವ ಕ್ಯಾಲ್ಕುಲೇಟರ್ ಉಪಕರಣಗಳನ್ನು ಬಳಸಬೇಕು ಎಂದು ನಿರ್ಧರಿಸುತ್ತದೆ:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // ಮೈಕ್ರೋಸಾಫ್ಟ್ ಎಂಟ್ರಾ ಐಡಿ ಮೂಲಕ ಕೀಲಿಯಿಲ್ಲದ ಪ್ರಾಮಾಣೀಕರಣದೊಂದಿಗೆ AI ಮಾದರಿಯನ್ನು ಸೆಟ್ ಅಪ್ ಮಾಡಿ (Azure AI Foundry)
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

        // ನಮ್ಮ ಕ್ಯಾಲ್ಕ್ಯುಲೇಟರ್ MCP ಸರ್ವರ್‌ಗೆ ಸಂಪರ್ಕಿಸಿ
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // AI ಇದು ಏನು ಮಾಡುತ್ತಿದೆ ಎಂದು ತೋರಿಸುತ್ತದೆ
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // AI ಗೆ ನಮ್ಮ ಕ್ಯಾಲ್ಕ್ಯುಲೇಟರ್ ಉಪಕರಣಗಳಿಗೆ ಪ್ರವೇಶ ನೀಡండి
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // ನಮ್ಮ ಕ್ಯಾಲ್ಕ್ಯುಲೇಟರ್ ಬಳಸಬಹುದಾದ AI ಬಾಟ್ ರಚಿಸಿ
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // ಈಗ ನಾವು ಪ್ರಕೃತಭಾಷೆಯಲ್ಲಿ ಲೆಕ್ಕಾಚಾರಗಳನ್ನು ಮಾಡಲು AIಗೆ ಕೇಳಬಹುದು
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**ಇದು ಮಾಡುವದು:**
1. ಕೀ ರಹಿತ ಪ್ರಮಾಣಿಕರಣ (Microsoft Entra ID) ಬಳಸಿ AI ಮಾದರಿ ಸಂಪರ್ಕವನ್ನು ರಚಿಸುತ್ತದೆ
2. AI ಯನ್ನು ನಮ್ಮ MCP ಕ್ಯಾಲ್ಕುಲೇಟರ್ ಸರ್ವರ್‌ಗೆ ಸಂಪರ್ಕ ಮಾಡುತ್ತದೆ
3. AIಗೆ ನಮ್ಮ ಎಲ್ಲಾ ಕ್ಯಾಲ್ಕುಲೇಟರ್ ಉಪಕರಣಗಳಿಗೆ ಪ್ರವೇಶವನ್ನು ನೀಡುತ್ತದೆ
4. "24.5 ಮತ್ತು 17.3 ರ ಮೊತ್ತ ಲೆಕ್ಕಿಸಿ" ಹೀಗಾದ ಸಹಜ ಭಾಷೆ ವಿನಂತಿಗಳನ್ನು ಅನುಮತಿಸುತ್ತದೆ

**AI ಸ್ವಯಂಚಾಲಿತವಾಗಿ:**
- ನೀವು ಸಂಖ್ಯೆಗಳನ್ನು ಸೇರಿಸಲು ಇಚ್ಛಿಸುವಿರಿ ಎಂದು ಅರ್ಥಮಾಡಿಕೊಳ್ಳುತ್ತದೆ
- `add` ಉಪಕರಣವನ್ನು ಆಯ್ಕೆಮಾಡುತ್ತದೆ
- `add(24.5, 17.3)` ಅನ್ನು ಕರೆ ಮಾಡುತ್ತದೆ
- ಫಲಿತಾಂಶವನ್ನು ಸಹಜ ಉತ್ತರದ ರೂಪದಲ್ಲಿ ನೀಡುತ್ತದೆ

## ಉದಾಹರಣೆಗಳನ್ನು ಚಾಲನೆ ಮಾಡುವುದು

### ಹಂತ 1: ಕ್ಯಾಲ್ಕುಲೇಟರ್ ಸರ್ವರ್ ಪ್ರಾರಂಭಿಸಿ

ಮೊದಲು, ಸೈನ್ ಇನ್ ಆಗಿ ಮತ್ತು ನಿಮ್ಮ Azure AI Foundry ಎಂಡ್‌ಪಾಯಿಂಟ್ ಅನ್ನು ಸೆಟ್ ಮಾಡಿ (AI ಕ್ಲೈಂಟ್‌ಗಾಗಿ ಅಗತ್ಯವಾಗಿದೆ — ಕೀ ರಹಿತ ಪ್ರಮಾಣಿ ಕರ್ನಾಟಕ, API ಕೀ ಇಲ್ಲ):

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

ಸರ್ವರ್ ಪ್ರಾರಂಭಿಸಿ:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

ಸರ್ವರ್ `http://localhost:8080` ನಲ್ಲಿ ಪ್ರಾರಂಭವಾಗುತ್ತದೆ. ನೀವು ನೋಡಬಲ್ಲಿರಿ:
```
Started McpServerApplication in X.XXX seconds
```

### ಹಂತ 2: ನೇರ ಕ್ಲೈಂಟ್ ಜೊತೆಗೆ ಪರೀಕ್ಷೆ ಮಾಡಿ

ಸರ್ವರ್ ಇನ್ನೂ ಚಾಲನೆಯಲ್ಲಿ ಇరಲಾಗಿ **ಹೊಸ** ಟರ್ಮಿನಲ್‌ನಲ್ಲಿ ನೇರ MCP ಕ್ಲೈಂಟ್ ಅನ್ನು ಚಾಲನೆ ಮಾಡಿ:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

ನೀವು ಈ ರೀತಿ ಔಟ್‌ಪುಟ್ ನೋಡಬಹುದು:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### ಹಂತ 3: AI ಕ್ಲೈಂಟ್ ಜೊತೆಗೆ ಪರೀಕ್ಷೆ ಮಾಡಿ

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

ನೀವು AI ಸ್ವಯಂಚಾಲಿತವಾಗಿ ಉಪಕರಣಗಳನ್ನು ಬಳಸುತ್ತಿರುವುದನ್ನು ನೋಡಬಹುದು:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### ಹಂತ 4: MCP ಸರ್ವರ್ ಅನ್ನು ಮುಚ್ಚಿ

ಪರೀಕ್ಷೆ ಮಾಡಿದ್ದನ್ನು ಕೊನೆಗೊಳಿಸಿದ ಬಳಿಕ, AI ಕ್ಲೈಂಟ್ ಟರ್ಮಿನಲ್‌ನಲ್ಲಿ `Ctrl+C` ಒತ್ತಿ ಆತನನ್ನು ನಿಂತಿಸಬಹುದು. MCP ಸರ್ವರ್ ನೀವು ನಿಂತಿಸದವರೆಗೂ ಚಾಲನೆಯಲ್ಲಿ ಇರುತ್ತದೆ. ಸರ್ವರ್ ನಿಲ್ಲಿಸಲು, ಅದು ಓಡುತ್ತಿರುವ ಟರ್ಮಿನಲ್‌ನಲ್ಲಿ `Ctrl+C` ಒತ್ತಿ.

## ಎಲ್ಲದರ ಒಟ್ಟಿಗೆ ಕೆಲಸ ಮಾಡುವ ವಿಧಾನ

ನೀವು AI ಗೆ "5 + 3 ಎಷ್ಟು?" ಎಂದು ಕೇಳಿದಾಗ ಸಂಪೂರ್ಣ ಪ್ರಕ್ರಿಯೆ ಹೀಗೆ:

1. **ನೀವು** ಸಹಜ ಭಾಷೆಯಲ್ಲಿ AIಗೆ ಕೇಳುತ್ತೀರಿ
2. **AI** ನಿಮ್ಮ ವಿನಂತಿಯನ್ನು ವಿಶ್ಲೇಷಿಸಿ ಇನ್‌ಹೆಂಟ್ಸ್ ಹೊಂದಿದೆ ನೀವು ಸೇರಿಸಬೇಕೆಂದು ತಿಳಿದುಕೊಳ್ಳುತ್ತದೆ
3. **AI** MCP ಸರ್ವರ್ ಅನ್ನು ಕರೆ ಮಾಡುತ್ತದೆ: `add(5.0, 3.0)`
4. **ಕ್ಯಾಲ್ಕುಲೇಟರ್ ಸರ್ವಿಸ್:** `5.0 + 3.0 = 8.0` ಎಂಬಂತೆ ಲೆಕ್ಕಿಂಸುತ್ತದೆ
5. **ಕ್ಯಾಲ್ಕುಲೇಟರ್ ಸರ್ವಿಸ್:** `"5.00 + 3.00 = 8.00"` ಅಭಿವ್ಯಕ್ತಿಯನ್ನು ನೀಡುತ್ತದೆ
6. **AI:** ಫಲಿತಾಂಶವನ್ನು ಸ್ವಾಭಾವಿಕ ಉತ್ತರ ರೂಪದಲ್ಲಿ ಮರುಭಾವನೆ ಮಾಡುತ್ತದೆ
7. **ನೀವು** ಪಡೆಯುವಿರಿ: "5 ಮತ್ತು 3 ರ ಮೊತ್ತ 8"

## ಮುಂದಿನ ಹಂತಗಳು

ಹೆಚ್ಚಿನ ಉದಾಹರಣೆಗಳಿಗೆ, [ಅಧ್ಯಾಯ 04: ಪ್ರವೃತ್ತಿ ಉದಾಹರಣೆಗಳು](../README.md) ನೋಡಿ

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ಅಸ್ವೀಕಾರ**:
ಈ ದಸ್ತಾವೇಜು AI ಅನುವಾದ ಸೇವೆ [Co-op Translator](https://github.com/Azure/co-op-translator) ಬಳಸಿ ಅನುವಾದಿಸಲಾಗಿದೆ. ನಾವು ನಿಖರತೆಯನ್ನು ಸಾಧಿಸಲು ಪ್ರಯತ್ನಿಸುತ್ತಿದ್ದರೂ, ದಯವಿಟ್ಟು ಗಮನಿಸಿ, ಸ್ವಯಂಚಾಲಿತ ಅನುವಾದಗಳಲ್ಲಿ ದೋಷಗಳು ಅಥವಾ ಅಸಡ್ಡೆಗಳು ಇರಬಹುದು. ಮೂಲ ಭಾಷೆಯಲ್ಲಿರುವ ಮೂಲ ದಸ್ತಾವೇಜು ಪ್ರಾಮಾಣಿಕ ಮೂಲವೆಂದು ಪರಿಗಣಿಸಬೇಕು. ಪ್ರಮುಖ ಮಾಹಿತಿಗಾಗಿ, ವೃತ್ತಿಪರ ಮಾನವ ಅನುವಾದವನ್ನು ಶಿಫಾರಸು ಮಾಡಲಾಗುತ್ತದೆ. ಈ ಅನುವಾದವನ್ನು ಬಳಸುವ ಮೂಲಕ ಉಂಟಾಗುವ ಯಾವುದೇ ತಪ್ಪು ಅರ್ಥಗಳ ಅಥವಾ ತಪ್ಪು ವ್ಯಾಖ್ಯಾನಗಳ ಬಗ್ಗೆ ನಾವು ಹೊಣೆಗಾರರಲ್ಲ.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->