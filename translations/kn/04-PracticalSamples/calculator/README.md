# ಆರಂಭಿಕರಿಗಾಗಿ MCP ಕ್ಯಾಲ್ಕ್ಯುಲೇಟರ್ ಟ್ಯೂಟೋರಿಯಲ್

## ವಿಷಯಸೂಚಿ

- [ನೀವು ಏನು ಕಲಿತீರಿ](#ನೀವು-ಏನು-ಕಲಿತೀರಿ)
- [ಮುನ್ನೋಟಗಳು](#ಮುನ್ನೋಟಗಳು)
- [ಪ್ರಾಜೆಕ್ಟ್ ರಚನೆ ಅರ್ಥಮಾಡಿಕೊಳ್ಳುವುದು](#ಪ್ರಾಜೆಕ್ಟ್-ರಚನೆ-ಅರ್ಥಮಾಡಿಕೊಳ್ಳುವುದು)
- [ಮೂಲಭೂತ घटಕಗಳು ವಿವರಿಸಲಾಗಿದೆ](#ಮೂಲಭೂತ-घटಕಗಳು-ವಿವರಿಸಲಾಗಿದೆ)
  - [1. ಮುಖ್ಯ ಅಪ್ಲಿಕೇಶನ್](#1-ಮುಖ್ಯ-ಅಪ್ಲಿಕೇಶನ್)
  - [2. ಕ್ಯಾಲ್ಕ್ಯುಲೇಟರ್ ಸೇವೆ](#2-ಕ್ಯಾಲ್ಕ್ಯುಲೇಟರ್-ಸೇವೆ)
  - [3. ನೇರ MCP ಕ್ಲೈಯಂಟ್](#3-ನೇರ-mcp-ಕ್ಲೈಯಂಟ್)
  - [4. AI-ಚಾಲಿತ ಕ್ಲೈಯಂಟ್](#4-ai-ಚಾಲಿತ-ಕ್ಲೈಯಂಟ್)
- [ಉದಾಹರಣೆಗಳನ್ನು چلಿಸುವುದು](#ಉದಾಹರಣೆಗಳನ್ನು-چلಿಸುವುದು)
- [ಇವು ಎಲ್ಲವು ಹೇಗೆ ಒಟ್ಟಾಗಿ ಕಾರ್ಯನಿರ್ವಹಿಸುತ್ತವೆ](#ಇವು-ಎಲ್ಲವು-ಹೇಗೆ-ಒಟ್ಟಾಗಿ-ಕಾರ್ಯನಿರ್ವಹಿಸುತ್ತವೆ)
- [ಮುಂದಿನ ಹೆಜ್ಜೆಗಳು](#ಮುಂದಿನ-ಹೆಜ್ಜೆಗಳು)

## ನೀವು ಏನು ಕಲಿತೀರಿ

ಈ ಟ್ಯೂಟೋರಿಯಲ್ ಮಾದರಿ ಸಂಗತಿಪ್ರೋಟೋಕಾಲ್ (MCP) ಬಳಸಿ ಕ್ಯಾಲ್ಕ್ಯುಲೇಟರ್ ಸೇವೆ ಹೇಗೆ ನಿರ್ಮಿಸುತಾರೆ ಎಂದು ವಿವರಿಸುತ್ತದೆ. ನೀವು ತಿಳಿದುಕೊಳ್ಳಲಿದ್ದೀರಿ:

- AI ಉಪಕರಣವಾಗಿ ಬಳಸಬಹುದಾದ ಸೇವೆಯನ್ನು ಹೇಗೆ ರಚಿಸುವುದು
- MCP ಸೇವೆಗಳೊಂದಿಗೆ ನೇರ ಸಂವಹನವನ್ನು ಹೇಗೆ ಸ್ಥಾಪಿಸುವುದು
- AI ಮಾದರಿಗಳು ಯಾವ ಟೂಲ್ಗಳನ್ನು ಬಳಸಬೇಕೆಂದು ಸ್ವಯಂಚಾಲಿತವಾಗಿ ಆರಿಸುವ ವಿಧಾನ
- ನೇರ ಪ್ರೋಟೋಕಾಲ್ ಕರೆಗೆ ಮತ್ತು AI-ಸಹಾಯಕ ಸಂವಹನಗಳ ನಡುವಿನ ವ್ಯತ್ಯಾಸ

## ಮುನ್ನೋಟಗಳು

ಪ್ರಾರಂಭಿಸುವ ಮೊದಲು, ಖಚಿತಪಡಿಸಿಕೊಳ್ಳಿ ನೀವು ಹೊಂದಿರುವುದು:
- Java 21 ಅಥವಾ ಅದಕ್ಕೂ ಮೇಲು ಸ್ಥಾಪನೆ
- ಅವಲಂಬನ ನಿರ್ವಹಣೆಗೆ Maven
- Azure AI Foundry ಮಾದರಿ ನಿಯೋಜನೆ (ಅಥವಾ `azd up` - ನೋಡಿ [ಅಧ್ಯಾಯ 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), `az login` ಮೂಲಕ ಸೈನ್ ಇನ್ (ಕೀಲೆಸ್ ಪ್ರಮಾನೀಕರಣ)
- Java ಮತ್ತು Spring Boot ನ ಮೂಲಭೂತ ಅರಿವು

## ಪ್ರಾಜೆಕ್ಟ್ ರಚನೆ ಅರ್ಥಮಾಡಿಕೊಳ್ಳುವುದು

ಕ್ಯಾಲ್ಕ್ಯುಲೇಟರ್ ಪ್ರಾಜೆಕ್ಟ್‌ಗೆ ಹಲವಾರು ಪ್ರಮುಖ ಫೈಲುಗಳು ಇವೆ:

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

## ಮೂಲಭೂತ घटಕಗಳು ವಿವರಿಸಲಾಗಿದೆ

### 1. ಮುಖ್ಯ ಅಪ್ಲಿಕೇಶನ್

**ಫೈಲ್:** `McpServerApplication.java`

ಇದು ನಮ್ಮ ಕ್ಯಾಲ್ಕ್ಯುಲೇಟರ್ ಸೇವೆಯ ಪ್ರವೇಶ ಬಿಂದುವಾಗಿದೆ. ಇದು ಒಂದು ಸಾಮಾನ್ಯ Spring Boot ಅಪ್ಲಿಕೇಶನ್ ಆಗಿದ್ದು ಒಂದು ವಿಶೇಷ ಸೇರಿಕೆ ಹೊಂದಿದೆ:

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

**ಇದರಿಂದ ಏನು ಆಗುತ್ತದೆ:**
- ಪೋರ್ಟ್ 8080 ನಲ್ಲಿ Spring Boot ವೆಬ್ ಸರ್ವರ್ ಪ್ರಾರಂಭಿಸುತ್ತದೆ
- ನಮ್ಮ ಕ್ಯಾಲ್ಕ್ಯುಲೇಟರ್ ವಿಧಾನಗಳನ್ನು MCP ಟೂಲ್ಗಳಾಗಿ ಲಭ್ಯವಾಗುವಂತೆ ಮಾಡುವ `ToolCallbackProvider` ರಚಿಸುತ್ತದೆ
- `@Bean` ಅರೇಖೆ Spring ನ ಈ ಘಟಕವನ್ನು ನಿರ್ವಹಿಸಲು ಮತ್ತಿತರ ಭಾಗಗಳು ಬಳಸಬಹುದು ಎಂದು ಸೂಚಿಸುತ್ತದೆ

### 2. ಕ್ಯಾಲ್ಕ್ಯುಲೇಟರ್ ಸೇವೆ

**ಫೈಲ್:** `CalculatorService.java`

ಇಲ್ಲಿ ಎಲ್ಲಾ ಗಣಿತ ಕಾರ್ಯಗಳು ನಡೆಯುತ್ತವೆ. ಪ್ರತಿ ವಿಧಾನದಲ್ಲಿ `@Tool` ಅಲೇಖನ ಹಾಕಲಾಗಿದೆ, ಇದರಿಂದ MCP ಮೂಲಕ ಲಭ್ಯವಾಗುತ್ತದೆ:

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
    
    // ಇನ್ನಷ್ಟು ಕ್ಯಾಲ್ಕ್ಯುಲೇಟರ್ ಕಾರ್ಯಾಚರಣೆಗಳು...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**ಮುಖ್ಯ ಲಕ್ಷಣಗಳು:**

1. **`@Tool` ಅಲೇಖನ**: ಈ ವಿಧಾನವನ್ನು ಬಾಹ್ಯ ಕ್ಲೈಯಂಟ್ ಇಳಿಸಲು MCP ಅಗೊಳಿಸುತ್ತದೆ
2. **ಸ್ಪಷ್ಟ ವಿವರಣೆಗಳು**: ಪ್ರತಿ ಟೂಲ್‌ಗೆ ವಿವರಣೆ ಇರುತ್ತದೆ, ಇದರಿಂದ AI ಮಾದರಿಗಳು ಯಾವಾಗ ಬಳಸಬೇಕೆಂದು ತಿಳಿದುಕೊಳ್ಳುತ್ತವೆ
3. **ಸತತವಾದ ಫಲಿತಾಂಶ ರೂಪ**: ಎಲ್ಲಾ ಕ್ರಿಯೆಗಳು "5.00 + 3.00 = 8.00" போன்ற ಮಾನವ ಓದಲು ಅನುಕೂಲವಾದ סטרಿಂಗ್ ಗಳನ್ನು ನೀಡುತ್ತವೆ
4. **ದೋಷ ನಿರ್ವಹಣೆ**: ಶೂನ್ಯದಿಂದ ಭಾಗಾಕಾರ ಮತ್ತು ನೆಗೆಟಿವ್ ಚದರ ಮೂಲಕ್ಕೆ ದೋಷ ಸಂದೇಶಗಳು ಸಿಗುತ್ತವೆ

**ಲಭ್ಯವಿರುವ ಕ್ರಿಯೆಗಳು:**
- `add(a, b)` - ಎರಡು ಸಂಖ್ಯೆಗಳ ಜೋಡಣೆ
- `subtract(a, b)` - ಎರಡನೇ ಸಂಖ್ಯೆಯನ್ನು ಮೊದಲನೇರಿಂದ ತೆಗೆದುಹಾಕುವುದು
- `multiply(a, b)` - ಎರಡು ಸಂಖ್ಯೆಗಳ ಗುಣಾಕಾರ
- `divide(a, b)` - ಮೊದಲನೆಯದನ್ನು ಎರಡನೇ ಸಂಖ್ಯೆಯಿಂದ ಭಾಗಿಸುವುದು (ಶೂನ್ಯ ಪರಿಶೀಲನೆ)
- `power(base, exponent)` - ಆಧಾರವನ್ನು ಘಾತಿಯ ಶಕ್ತಿಗೆ ಎತ್ತುವುದು
- `squareRoot(number)` - ಚದರ ಮೂಲ ಲೆಕ್ಕಿಸು (ನೆಗೆಟಿವ್ ಪರಿಶೀಲನೆ)
- `modulus(a, b)` - ಭಾಗಾಕಾರದ ಉಳಿಕೆ
- `absolute(number)` - ಪರಮ ಮೌಲ್ಯ
- `help()` - ಎಲ್ಲಾ ಕ್ರಿಯೆಗಳ ಬಗ್ಗೆ ಮಾಹಿತಿ ನೀಡುತ್ತದೆ

### 3. ನೇರ MCP ಕ್ಲೈಯಂಟ್

**ಫೈಲ್:** `SDKClient.java`

ಈ ಕ್ಲೈಯಂಟ್ ನೇರವಾಗಿ MCP ಸರ್ವರ್‌ನೊಂದಿಗೆ ಸಂವಹನ ಮಾಡುತ್ತದೆ, AI ಬಳಸುವುದಿಲ್ಲ. ಇದು ಅನೇಕ ನಿರ್ದಿಷ್ಟ ಕ್ಯಾಲ್ಕ್ಯುಲೇಟರ್ ಫังก್ಷನ್‌ಗಳನ್ನು ಮನ್ಯುಯಲಿ ಕರೆ ಮಾಡುತ್ತದೆ:

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
        
        // ಲಭ್ಯವಿರುವ ಸಾಧನಗಳನ್ನು ಪಟ್ಟಿಮಾಡಿ
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // ನಿರ್ದಿಷ್ಟ ಕ್ಯಾಲ್ಕುಲೇಟರ್ ಕಾರ್ಯಗಳನ್ನು ಕರೆಮಾಡಿ
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

**ಇದರಿಂದ ಏನು ಆಗುತ್ತದೆ:**
1. `http://localhost:8080` ನಲ್ಲಿ ಕ್ಯಾಲ್ಕ್ಯುಲೇಟರ್ ಸರ್ವರ್‌ ಜೊತೆ ನಿರ್ಮಾಣಕಾರರ ಧಾರಣೆ ಮೂಲಕ ಸಂಪರ್ಕ ಸೇರಿಸುವುದು
2. ಲಭ್ಯವಿರುವ ಎಲ್ಲಾ ಟೂಲ್ಗಳ ಪಟ್ಟಿಯನ್ನು ಪಡೆಯುವುದು (ನಮ್ಮ ಕ್ಯಾಲ್ಕ್ಯುಲೇಟರ್ ಕಾರ್ಯಗಳು)
3. ನಿಖರ ಪಾರಾಮೀಟರ್‌ಗಳೊಂದಿಗೆ ನಿರ್ದಿಷ್ಟ ಕಾರ್ಯಗಳನ್ನು ಕರೆಯುವುದು
4. ಫಲಿತಾಂಶಗಳನ್ನು ನೇರವಾಗಿ ಮುದ್ರಿಸುವುದು

**ಸೂಚನೆ:** ಈ ಉದಾಹರಣೆ Spring AI 1.1.0-SNAPSHOT ಅನುಬಂಧವನ್ನು ಬಳಸುತ್ತದೆ, ಇದು `WebFluxSseClientTransport` ಗೆ ನಿರ್ಮಾಣಕಾರರ ವಿಧಾನವನ್ನು ಪರಿಚಯಿಸಿದೆ. ನೀವು ಹಳೆಯ ಸ್ಥಿರ ಆವೃತ್ತಿ ಬಳಸುತ್ತಿದ್ದರೆ ನೇರ ಕಾಂಸ್ಟ್ರಕ್ಟರ್ ಬಳಕೆಯ ಅಗತ್ಯವಿರಬಹುದು.

**ಬಳಸುವಾಗ:** ನೀವು ಯಾವ ಲೆಕ್ಕировку ಮಾಡಬೇಕೆಂದು ನಿಖರವಾಗಿ ಗೊತ್ತಿದ್ದಾಗ ಮತ್ತು ಅದನ್ನು ಪ್ರೋಗ್ರಾಮ್ ಮೂಲಕ ಕರೆಮಾಡಬೇಕು ಎಂದಾಗ.

### 4. AI-ಚಾಲಿತ ಕ್ಲೈಯಂಟ್

**ಫೈಲ್:** `LangChain4jClient.java`

ಈ ಕ್ಲೈಯಂಟ್ AI ಮಾದರಿಯನ್ನು (GPT-4o-mini) ಬಳಸುತ್ತದೆ, ಇದು ಸ್ವಯಂಚಾಲಿತವಾಗಿ ಯಾವ ಕ್ಯಾಲ್ಕ್ಯುಲೇಟರ್ ಟೂಲ್ಗಳನ್ನು ಬಳಸಬೇಕೆಂದು ನಿರ್ಧರಿಸುತ್ತದೆ:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // AI ಮಾದರಿಯನ್ನು ಸೆಟ್ ಅಪ್ ಮಾಡಿರಿ (Azure AI Foundry, Microsoft Entra ID ಮೂಲಕ ಕೀಲೆस್ಷ್ ಅಕ್ರಹಿತ)
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

        // ನಮ್ಮ ಕ್ಯಾಲ್ಕುಲೇಟರ್ MCP ಸರ್ವರ್‌ಗೆ ಸಂಪರ್ಕಿಸಿರಿ
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // AI ಏನು ಮಾಡುತ್ತಿದೆ ಎಂದು ತೋರಿಸುತ್ತದೆ
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // ನಮ್ಮ ಕ್ಯಾಲ್ಕುಲೇಟರ್ ಉಪಕರಣಗಳಿಗೆ AI ಪ್ರವೇಶ ನೀಡಿರಿ
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // ನಮ್ಮ ಕ್ಯಾಲ್ಕುಲೇಟರ್ ಬಳಕೆ ಮಾಡಬಹುದಾದ AI ಬಾಟ್ ಸೃಷ್ಟಿಸಿರಿ
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // ಈಗ ನಾವು ನೈಸರ್ಗಿಕ ಭಾಷೆಯಲ್ಲಿ ಎಐನಿಂದ ಲೆಕ್ಕಾಚಾರಗಳನ್ನು ಕೇಳಬಹುದು
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**ಇದರಿಂದ ಏನು ಆಗುತ್ತದೆ:**
1. ನಿಮ್ಮ GitHub ಟೋಕನ್ ಬಳಸಿ AI ಮಾದರಿ ಸಂಪರ್ಕವನ್ನು ರಚಿಸುವುದು
2. AI ಅನ್ನು ನಮ್ಮ ಕ್ಯಾಲ್ಕ್ಯುಲೇಟರ್ MCP ಸರ್ವರ್‌ಗೆ ಕನೆಕ್ಟ್ ಮಾಡುವುದು
3. AI ಗೆ ನಮ್ಮ ಎಲ್ಲಾ ಕ್ಯಾಲ್ಕ್ಯುಲೇಟರ್ ಟೂಲ್‌ಗಳ ಪ್ರವೇಶ ನೀಡುವುದು
4. "24.5 ಮತ್ತು 17.3 ರ ಮೊತ್ತ ಲೆಕ್ಕಿಸು" ಎಂಬ ಹಾಗೆ ಸಹಜ ಭಾಷಾ ವಿನಂತಿಗಳನ್ನು ಸಾಧ್ಯವಾಗಿಸುವುದು

**AI ಸ್ವಯಂಚಾಲಿತವಾಗಿ:**
- ನೀವು ಸಂಖ್ಯೆಗಳನ್ನು ಸೇರಿಸಲು ಬಯಸಿದೀರಿ ಎಂದು ಅರ್ಥಮಾಡಿಕೊಳ್ಳುತ್ತದೆ
- `add` ಟೂಲ್ ಆಯ್ಕೆ ಮಾಡುತ್ತದೆ
- `add(24.5, 17.3)` ಅನ್ನು ಕರೆಮಾಡುತ್ತದೆ
- ಫಲಿತಾಂಶವನ್ನು ಸಹಜ ಪ್ರತಿಕ್ರಿಯೆಯಾಗಿ ನೀಡುತ್ತದೆ

## ಉದಾಹರಣೆಗಳನ್ನು چلಿಸುವುದು

### ಹಂತ 1: ಕ್ಯಾಲ್ಕ್ಯುಲೇಟರ್ ಸರ್ವರ್ ಪ್ರಾರಂಭಿಸಿ

ಮೊದಲು, ಸೈನ್ ಇನ್ ಮಾಡಿ ಮತ್ತು ನಿಮ್ಮ Azure AI Foundry ಎಂಡ್‌ಪಾಯಿಂಟ್ ಅನ್ನು ಸೆಟ್ ಮಾಡಿ (AI ಕ್ಲೈಯಂಟ್ ಗೆ ಅಗತ್ಯವಿದೆ — ಕೀಲೆಸ್ ಪ್ರಮಾನೀಕರಣ, API ಕೀ ಇಲ್ಲ):

**ವಿಂಡೋಸ್:**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**ಲಿನಕ್ಸ್/ಮ್ಯಾಕ್‌ಓಎಸ್:**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

ಸರ್ವರ್ ಪ್ರಾರಂಭಿಸಿ:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

ಸರ್ವರ್ `http://localhost:8080` ನಲ್ಲಿ ಪ್ರಾರಂಭವಾಗುತ್ತದೆ. ನಿಮಗೆ ಕೆಳಗಿನವು ಕಾಣಿಸುವುದಾಗಿರುತ್ತದೆ:
```
Started McpServerApplication in X.XXX seconds
```

### ಹಂತ 2: ನೇರ ಕ್ಲೈಯಂಟ್ ಸಹಿತ ಪರೀಕ್ಷೆ

ಸರ್ವರ್ ಇನ್ನೂ ಚಾಲನದಲ್ಲಿದ್ದಾಗ, **ಹೊಸ** ಟರ್ಮಿನಲ್‌ನಲ್ಲಿ ನೇರ MCP ಕ್ಲೈಯಂಟ್ ಚಲಾಯಿಸಿ:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

ನೀವು ಈ ರೀತಿ ಔಟ್‌ಪುಟ್ ಕಾಣಬಹುದು:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### ಹಂತ 3: AI ಕ್ಲೈಯಂಟ್ ಸಹಿತ ಪರೀಕ್ಷೆ

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

ನೀವು AI ಸ್ವಯಂಚಾಲಿತವಾಗಿ ಟೂಲ್ಗಳನ್ನು ಬಳಕೆ ಮಾಡುತ್ತಿರುವುದನ್ನು ಕಾಣುತ್ತೀರಿ:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### ಹಂತ 4: MCP ಸರ್ವರ್ ಅನ್ನು ಮುಚ್ಚಿ

ಪರೀಕ್ಷೆ ಮುಗಿಸಿದಾಗ, AI ಕ್ಲೈಯಂಟ್ ಅನ್ನು `Ctrl+C` ಒತ್ತಿ ತಡೆದಿರಿ. MCP ಸರ್ವರ್ ನೀವು ನಿಲ್ಲಿಸುವವರೆಗೆ ಚಾಲನದಲ್ಲಿರುತ್ತದೆ.
ಸರ್ವರ್ ನಿಲ್ಲಿಸಲು, ಅದನ್ನು ಚಾಲನೆ ಮಾಡುತ್ತಿರುವ ಟರ್ಮಿನಲ್‌ನಲ್ಲಿ `Ctrl+C` ಒತ್ತಿ.

## ಇವು ಎಲ್ಲವು ಹೇಗೆ ಒಟ್ಟಾಗಿ ಕಾರ್ಯನಿರ್ವಹಿಸುತ್ತವೆ

ನೀವು AI ಗೆ "5 + 3 ಎಷ್ಟು?" ಎಂದು ಕೇಳಿದಾಗ ಪೂರ್ಣ ವರ್ಕ್‌ಫ್ಲೋ ಇಂತಿರುತ್ತದೆ:

1. **ನೀವು** ಸಹಜ ಭಾಷೆಯಲ್ಲಿ AI ಗೆ ಕೇಳುತ್ತೀರಿ
2. **AI** ನಿಮ್ಮ ವಿನಂತಿಯನ್ನು ವಿಶ್ಲೇಷಿಸಿ, ನೀವು ಸೇರಿಸುವ ಕ್ರಿಯೆಯನ್ನು ಬಯಸುವಿರಿ ಎಂದರ್ಥಮಾಡಿಕೊಳ್ಳುತ್ತದೆ
3. **AI** MCP ಸರ್ವರ್‌ಗೆ `add(5.0, 3.0)` ಅನ್ನು ಕರೆಮಾಡುತ್ತದೆ
4. **ಕ್ಯಾಲ್ಕ್ಯುಲೇಟರ್ ಸೇವೆ** ಕಾರ್ಯನಿರ್ವಹಿಸಿ: `5.0 + 3.0 = 8.0`
5. **ಕ್ಯಾಲ್ಕ್ಯುಲೇಟರ್ ಸೇವೆ** ಫಲಿತಾಂಶ `"5.00 + 3.00 = 8.00"` ಅನ್ನು ನೀಡುತ್ತದೆ
6. **AI** ಫಲಿತಾಂಶವನ್ನು ಸ್ವಾಭಾವಿಕ ಪ್ರತಿಕ್ರಿಯೆಯಾಗಿ ರೂಪಿಸುತ್ತದೆ
7. **ನೀವು** ಪಡೆಯುತ್ತೀರಿ: "5 ಮತ್ತು 3 ಯ ಚಕ್ರದ ಮೊತ್ತ 8"

## ಮುಂದಿನ ಹೆಜ್ಜೆಗಳು

ಹೆಚ್ಚಿನ ಉದಾಹರಣೆಗಳಿಗೆ, ನೋಡಿ [ಅಧ್ಯಾಯ 04: ಪ್ರಾಯೋಗಿಕ ಮಾದರಿಗಳು](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ಅಸ್ವೀಕಾರ**:
ಈ ದಸ್ತಾವೇಜು AI ಅನುವಾದ ಸೇವೆ [Co-op Translator](https://github.com/Azure/co-op-translator) ಬಳಸಿ ಅನುವಾದಿಸಲಾಗಿದೆ. ನಾವು ನಿಖರತೆಯನ್ನು ಸಾಧಿಸಲು ಪ್ರಯತ್ನಿಸುತ್ತಿದ್ದರೂ, ದಯವಿಟ್ಟು ಗಮನಿಸಿ, ಸ್ವಯಂಚಾಲಿತ ಅನುವಾದಗಳಲ್ಲಿ ದೋಷಗಳು ಅಥವಾ ಅಸಡ್ಡೆಗಳು ಇರಬಹುದು. ಮೂಲ ಭಾಷೆಯಲ್ಲಿರುವ ಮೂಲ ದಸ್ತಾವೇಜು ಪ್ರಾಮಾಣಿಕ ಮೂಲವೆಂದು ಪರಿಗಣಿಸಬೇಕು. ಪ್ರಮುಖ ಮಾಹಿತಿಗಾಗಿ, ವೃತ್ತಿಪರ ಮಾನವ ಅನುವಾದವನ್ನು ಶಿಫಾರಸು ಮಾಡಲಾಗುತ್ತದೆ. ಈ ಅನುವಾದವನ್ನು ಬಳಸುವ ಮೂಲಕ ಉಂಟಾಗುವ ಯಾವುದೇ ತಪ್ಪು ಅರ್ಥಗಳ ಅಥವಾ ತಪ್ಪು ವ್ಯಾಖ್ಯಾನಗಳ ಬಗ್ಗೆ ನಾವು ಹೊಣೆಗಾರರಲ್ಲ.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->