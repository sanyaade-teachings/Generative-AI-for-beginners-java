# MCP ਕੈਲਕੂਲੇਟਰ ਟਿਊਟੋਰਿਅਲ ਸ਼ੁਰੂਆਤੀ ਲਈ

## ਟੇਬਲ ਆਫ਼ ਕਂਟੈਂਟਸ

- [ਤੁਸੀਂ ਕੀ ਸਿੱਖੋਗੇ](#ਤੁਸੀਂ-ਕੀ-ਸਿੱਖੋਗੇ)
- [ਆਗੇ ਦੀਆਂ ਲੋੜਾਂ](#ਆਗੇ-ਦੀਆਂ-ਲੋੜਾਂ)
- [ਪ੍ਰੋਜੈਕਟ ਦੀ ساخت ਨੂੰ ਸਮਝਣਾ](#ਪ੍ਰੋਜੈਕਟ-ਦੀ-ਸਟਰੱਕਚਰ-ਨੂੰ-ਸਮਝਣਾ)
- [ਮੁੱਖ ਹਿੱਸਿਆਂ ਦੀ ਵਿਆਖਿਆ](#ਮੁੱਖ-ਹਿੱਸਿਆਂ-ਦੀ-ਵਿਆਖਿਆ)
  - [1. ਮੁੱਖ ਐਪਲੀਕੇਸ਼ਨ](#1-ਮੁੱਖ-ਐਪਲੀਕੇਸ਼ਨ)
  - [2. ਕੈਲਕੂਲੇਟਰ ਸਰਵਿਸ](#2-ਕੈਲਕੂਲੇਟਰ-ਸਰਵਿਸ)
  - [3. ਡਾਇਰੈਕਟ MCP ਕਲੀਐਂਟ](#3-ਡਾਇਰੈਕਟ-mcp-ਕਲੀਐਂਟ)
  - [4. AI-ਚਲਿਤ ਕਲੀਐਂਟ](#4-ai-ਚਲਿਤ-ਕਲੀਐਂਟ)
- [ਉਦਾਹਰਣਾਂ ਚਲਾਉਣਾ](#ਉਦਾਹਰਣਾਂ-ਚਲਾਉਣਾ)
- [ਇਹ ਸਾਰਾ ਕਿਵੇਂ ਕੰਮ ਕਰਦਾ ਹੈ](#ਇਹ-ਸਾਰਾ-ਕਿਵੇਂ-ਕੰਮ-ਕਰਦਾ-ਹੈ)
- [ਅਗਲੇ ਕਦਮ](#ਅਗਲੇ-ਕਦਮ)

## ਤੁਸੀਂ ਕੀ ਸਿੱਖੋਗੇ

ਇਹ ਟਿਊਟੋਰਿਅਲ ਮਾਡਲ ਸੰਦਰਭ ਪ੍ਰੋਟੋਕੋਲ (MCP) ਦੀ ਵਰਤੋਂ ਨਾਲ ਕੈਲਕੂਲੇਟਰ ਸਰਵਿਸ ਬਣਾਉਣ ਦੀ ਵਿਵਰਣਾ ਦਿੰਦਾ ਹੈ। ਤੁਸੀਂ ਸਮਝੋਗੇ:

- ਕਿ ਕਿਵੇਂ ਇੱਕ ਐਸੀ ਸਰਵਿਸ ਬਣਾਈ ਜਾਵੇ ਜੋ AI ਨੂੰ ਟੂਲ ਵਜੋਂ ਵਰਤ ਸਕਦੀ ਹੋਵੇ
- MCP ਸਰਵਿਸਜ਼ ਨਾਲ ਸਿੱਧਾ ਸੰਚਾਰ ਕਿਵੇਂ ਸੈੱਟ ਕੀਤਾ ਜਾਵੇ
- ਕਿਵੇਂ AI ਮਾਡਲ ਆਪਣੇ ਆਪ ਸੋਚ ਕੇ ਟੂਲ ਚੁਣ ਸਕਦੇ ਹਨ
- ਸਿੱਧੇ ਪ੍ਰੋਟੋਕੋਲ ਕਾਲਾਂ ਅਤੇ AI-ਮਦਦਗਾਰ ਇੰਟਰੈਕਸ਼ਨਾਂ ਵਿਚ ਤਫ਼ਾਵਤ

## ਆਗੇ ਦੀਆਂ ਲੋੜਾਂ

ਸ਼ੁਰੂ ਕਰਨ ਤੋਂ ਪਹਿਲਾਂ, ਇਹ ਯਕੀਨੀ ਕਰੋ ਕਿ ਤੁਹਾਡੇ ਕੋਲ ਹੈ:
- ਜਾਵਾ 21 ਜਾਂ ਇਸ ਤੋਂ ਉੱਚਾ ਸਥਾਪਿਤ
- Maven ਦੀ ਡਿਪੇਂਡੇੰਸੀ ਮੈਨੇਜਮੈਂਟ ਲਈ
- Azure AI Foundry ਮਾਡਲ ਡਿਪਲੋਇਮੈਂਟ (ਇਸ ਨੂੰ `azd up` ਨਾਲ ਪ੍ਰੋਵਿਜ਼ਨ ਕਰੋ — ਵੇਖੋ [ਅਧਿਆਇ 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), `az login` ਨਾਲ ਸਾਈਨ ਇਨ (ਕੀਲੇਸ ਪ੍ਰਮਾਣਿਕਤਾ)
- ਜਾਵਾ ਅਤੇ ਸਪ੍ਰਿੰਗ ਬੂਟ ਬਾਰੇ ਬੁਨਿਆਦੀ ਸਮਝ

## ਪ੍ਰੋਜੈਕਟ ਦੀ ਸਟਰੱਕਚਰ ਨੂੰ ਸਮਝਣਾ

ਕੈਲਕੂਲੇਟਰ ਪ੍ਰੋਜੈਕਟ ਵਿੱਚ ਕਈ ਜਰੂਰੀ ਫਾਈਲਾਂ ਹਨ:

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

## ਮੁੱਖ ਹਿੱਸਿਆਂ ਦੀ ਵਿਆਖਿਆ

### 1. ਮੁੱਖ ਐਪਲੀਕੇਸ਼ਨ

**ਫਾਈਲ:** `McpServerApplication.java`

ਇਹ ਸਾਡੀ ਕੈਲਕੂਲੇਟਰ ਸਰਵਿਸ ਦਾ ਐਂਟਰੀ ਪੋਇੰਟ ਹੈ। ਇਹ ਇੱਕ ਸਟੈਂਡਰਡ ਸਪ੍ਰਿੰਗ ਬੂਟ ਐਪਲੀਕੇਸ਼ਨ ਹੈ ਜਿਸ ਵਿੱਚ ਇੱਕ ਖਾਸ ਸ਼ਾਮਲ ਹੈ:

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

**ਇਸਦਾ ਕੰਮ:**
- ਪੋਰਟ 8080 'ਤੇ ਸਪ੍ਰਿੰਗ ਬੂਟ ਵੈੱਬ ਸਰਵਰ ਸਟਾਰਟ ਕਰਦਾ ਹੈ
- `ToolCallbackProvider` ਬਣਾਉਂਦਾ ਹੈ ਜੋ ਸਾਡੇ ਕੈਲਕੂਲੇਟਰ ਦੇ ਤਰੀਕੇ MCP ਟੂਲਜ਼ ਵਜੋਂ ਉਪਲੱਬਧ ਕਰਵਾਉਂਦਾ ਹੈ
- `@Bean` ਐਨੋਟੇਸ਼ਨ ਸਪ੍ਰਿੰਗ ਨੂੰ ਦੱਸਦਾ ਹੈ ਕਿ ਇਸਨੂੰ ਇੱਕ ਘਟਕ ਵਜੋਂ ਸੰਜੋਇਆ ਜਾਏ ਜੋ ਹੋਰ ਹਿੱਸੇ ਵਰਤ ਸਕਣ

### 2. ਕੈਲਕੂਲੇਟਰ ਸਰਵਿਸ

**ਫਾਈਲ:** `CalculatorService.java`

ਇੱਥੇ ਸਾਰਾ ਗਣਿਤ ਹੁੰਦਾ ਹੈ। ਹਰ ਮੇਥਡ ਨੂੰ `@Tool` ਨਾਲ ਚਿੰਨ੍ਹਿਤ ਕੀਤਾ ਗਿਆ ਹੈ ਤਾਂ ਜੋ ਇਹ MCP ਕੇ ਜ਼ਰੀਏ ਉਪਲੱਬਧ ਹੋ ਸਕੇ:

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
    
    // ਹੋਰ ਕੈਲਕੂਲੇਟ ਕਰਮ...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**ਮੁੱਖ ਖਾਸੀਅਤਾਂ:**

1. **`@Tool` ਐਨੋਟੇਸ਼ਨ**: ਇਹ MCP ਨੂੰ ਦੱਸਦਾ ਹੈ ਕਿ ਇਹ ਮੇਥਡ ਬਾਹਰੀ ਕਲੀਐਂਟ ਵੱਲੋਂ ਕਾਲ ਕੀਤਾ ਜਾ ਸਕਦਾ ਹੈ
2. **ਸਪਸ਼ਟ ਵਰਣਨ**: ਹਰ ਟੂਲ ਦਾ ਵੇਰਵਾ ਦਿੰਦਾ ਹੈ ਜੋ AI ਮਾਡਲ ਨੂੰ ਸਮਝਾਉਂਦਾ ਹੈ ਕਿ ਕਦੋਂ ਇਸ ਦਾ ਇਸਤੇਮਾਲ ਕਰਨਾ ਹੈ
3. **ਇਕਸਾਰ ਵਾਪਸੀ ਫਾਰਮੈਟ**: ਸਭ ਓਪਰੇਸ਼ਨ ਮਨੁੱਖੀ-ਪਾਠ-ਯੋਗ ਸਤਰਾਂ "5.00 + 3.00 = 8.00" ਜਿਹੜੀਆਂ ਤਰ੍ਹਾਂ ਵਾਪਸ ਕਰਦੇ ਹਨ
4. **ਗਲਤੀ ਸੰਭਾਲਣਾ**: ਜ਼ੀਰੋ ਨਾਲ ਭਾਗ ਕਰਨ ਜਾਂ ਨਕਾਰਾਤਮਕ ਵਰਗਮੂਲ ਲਈ ਗਲਤੀ ਦੇ ਸੁਨੇਹੇ ਵਾਪਸ ਦਿੰਦਾ ਹੈ

**ਉਪਲਬਧ ਓਪਰੇਸ਼ਨ:**
- `add(a, b)` - ਦੋ ਨੰਬਰ ਜੋੜਦਾ ਹੈ
- `subtract(a, b)` - ਦੂਜਾ ਨੰਬਰ ਪਹਿਲੇ ਤੋਂ ਘਟਾਉਂਦਾ ਹੈ
- `multiply(a, b)` - ਦੋ ਨੰਬਰਾਂ ਦਾ ਗੁਣਾ ਕਰਦਾ ਹੈ
- `divide(a, b)` - ਪਹਿਲਾ ਨੰਬਰ ਦੂਜੇ ਨਾਲ ਭਾਜ਼ਦਾ ਹੈ (ਜ਼ੀਰੋ ਚੈੱਕ ਦੇ ਨਾਲ)
- `power(base, exponent)` - ਬੇਸ ਨੂੰ ਐਕਸਪੋਨੈਂਟ ਦੇ ਪਾਵਰ 'ਤੇ ਚੜ੍ਹਾਉਂਦਾ ਹੈ
- `squareRoot(number)` - ਵਰਗਮੂਲ ਕੱਢਦਾ ਹੈ (ਨਕਾਰਾਤਮਕ ਚੈੱਕ ਦੇ ਨਾਲ)
- `modulus(a, b)` - ਭਾਗ ਦੇ ਬਚਤ ਵੇਖਾਉਂਦਾ ਹੈ
- `absolute(number)` - ਪਰਮ ਮੁੱਲ ਦਿੰਦਾ ਹੈ
- `help()` - ਸਾਰੇ ਓਪਰੇਸ਼ੰਜ਼ ਬਾਰੇ ਜਾਣਕਾਰੀ ਦਿੰਦਾ ਹੈ

### 3. ਡਾਇਰੈਕਟ MCP ਕਲੀਐਂਟ

**ਫਾਈਲ:** `SDKClient.java`

ਇਹ ਕਲੀਐਂਟ AI ਦੀ ਵਰਤੋਂ ਨਾ ਕਰਦਿਆਂ ਸਿੱਧਾ MCP ਸਰਵਰ ਨਾਲ ਗੱਲ ਕਰਦਾ ਹੈ। ਇਹ ਹੱਥੋਂ-ਹੱਥ নির্দিষ্ট ਕੈਲਕੂਲੇਟਰ ਫੰਕਸ਼ਨਾਂ ਨੂੰ ਕਾਲ ਕਰਦਾ ਹੈ:

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
        
        // ਉਪਲਬਧ ਉਪਕਰਨਾਂ ਦੀ ਸੂਚੀ ਬਣਾਓ
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // ਖਾਸ ਕੈਲਕੂਲੇਟਰ ਫੰਕਸ਼ਨਾਂ ਨੂੰ ਕਾਲ ਕਰੋ
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

**ਇਸਦਾ ਕੰਮ:**
1. ਬਿਲਡਰ ਪੈਟਰਨ ਦੀ ਵਰਤੋਂ ਕਰਕੇ `http://localhost:8080` ਤੇ ਕੈਲਕੂਲੇਟਰ ਸਰਵਰ ਨਾਲ ਜੁੜਦਾ ਹੈ
2. ਸਾਰੇ ਉਪਲਬਧ ਟੂਲਜ਼ ਦੀ ਸੂਚੀ ਦਿਖਾਉਂਦਾ ਹੈ (ਸਾਡੇ ਕੈਲਕੂਲੇਟਰ ਫੰਕਸ਼ਨ)
3. ਸਹੀ ਪੈਰਾਮੀਟਰਾਂ ਨਾਲ ਖਾਸ ਫੰਕਸ਼ਨਾਂ ਨੂੰ ਕਾਲ ਕਰਦਾ ਹੈ
4. ਨਤੀਜੇ ਸਿੱਧਾ ਪ੍ਰਿੰਟ ਕਰਦਾ ਹੈ

**ਨੋਟ:** ਇਹ ਉਦਾਹਰਣ Spring AI 1.1.0-SNAPSHOT ਡਿਪੇਂਡੇੰਸੀ ਵਰਤਦਾ ਹੈ, ਜਿਸਨੇ `WebFluxSseClientTransport` ਲਈ ਬਿਲਡਰ ਪੈਟਰਨ ਸ਼ੁਰੂ ਕੀਤਾ। ਜੇ ਤੁਸੀਂ ਪੁਰਾਣਾ ਸਥਿਰ ਵਰਜਨ ਵਰਤ ਰਹੇ ਹੋ ਤਾਂ ਤੁਹਾਨੂੰ ਸਿੱਧਾ ਕਨਸਟਰਕਟਰ ਵਰਤਣਾ ਪੈ ਸਕਦਾ ਹੈ।

**ਕਦੋਂ ਵਰਤਣਾ:** ਜਦੋਂ ਤੁਹਾਨੂੰ ਪਤਾ ਹੋ ਕਿ ਕਿਸ ਗਣਨਾ ਨੂੰ ਕਿਵੇਂ ਕਰਨਾ ਹੈ ਅਤੇ ਤੁਸੀਂ ਇਸਨੂੰ ਪ੍ਰੋਗ੍ਰਾਮਾਂਗਤ ਤੌਰ 'ਤੇ ਕਾਲ ਕਰਨਾ ਚਾਹੁੰਦੇ ਹੋ।

### 4. AI-ਚਲਿਤ ਕਲੀਐਂਟ

**ਫਾਈਲ:** `LangChain4jClient.java`

ਇਹ ਕਲੀਐਂਟ AI ਮਾਡਲ (GPT-4o-mini) ਵਰਤਦਾ ਹੈ ਜੋ ਆਪਣੇ ਆਪ ਫੈਸਲਾ ਕਰਦਾ ਹੈ ਕਿ ਕਿਹੜੇ ਕੈਲਕੂਲੇਟਰ ਟੂਲ ਵਰਤਣੇ ਹਨ:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // ਏਆਈ ਮਾਡਲ ਸੈਟਅਪ ਕਰੋ (Azure AI Foundry, Microsoft Entra ID ਰਾਹੀਂ ਬਿਨਾਂ ਕੁੰਜੀ ਦੀ ਪ੍ਰਮਾਣਿਕਤਾ)
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

        // ਸਾਡੇ ਕੈਲਕੂਲੇਟਰ MCP ਸਰਵਰ ਨਾਲ ਜੁੜੋ
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // ਦਿਖਾਉਂਦਾ ਹੈ ਕਿ ਏਆਈ ਕੀ ਕਰ ਰਿਹਾ ਹੈ
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // ਏਆਈ ਨੂੰ ਸਾਡੇ ਕੈਲਕੂਲੇਟਰ ਟੂਲਾਂ ਤੱਕ ਪਹੁੰਚ ਦਿਓ
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // ਇੱਕ ਏਆਈ ਬੋਟ ਬਣਾਓ ਜੋ ਸਾਡੇ ਕੈਲਕੂਲੇਟਰ ਦਾ ਉਪਯੋਗ ਕਰ ਸਕਦਾ ਹੈ
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // ਹੁਣ ਅਸੀਂ ਏਆਈ ਨੂੰ ਕੁਦਰਤੀ ਭਾਸ਼ਾ ਵਿੱਚ ਗਣਨਾ ਕਰਨ ਲਈ ਕਹਿ ਸਕਦੇ ਹਾਂ
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**ਇਸਦਾ ਕੰਮ:**
1. Microsoft Entra ID ਦੀ ਕੀਲੇਸ ਪ੍ਰਮਾਣਿਕਤਾ ਨਾਲ AI ਮਾਡਲ ਕਨੈਕਸ਼ਨ ਬਣਾਉਂਦਾ ਹੈ
2. AI ਨੂੰ ਸਾਡੇ ਕੈਲਕੂਲੇਟਰ MCP ਸਰਵਰ ਨਾਲ ਜੋੜਦਾ ਹੈ
3. AI ਨੂੰ ਸਾਡੇ ਸਾਰੇ ਕੈਲਕੂਲੇਟਰ ਟੂਲਜ਼ ਦੀ ਪਹੁੰਚ ਦਿੰਦਾ ਹੈ
4. ਕਿਵੇਂ "24.5 ਅਤੇ 17.3 ਦਾ ਜੋੜ ਕੱਢੋ" ਵਰਗੀਆਂ ਕੁਦਰਤੀ ਭਾਸ਼ਾ ਦੀਆਂ ਬੇਨਤੀਆਂ ਸਵਾਗਤ ਕਰਦਾ ਹੈ

**AI ਸਵੈਚਾਲਿਤ ਤੌਰ 'ਤੇ:**
- ਸਮਝਦਾ ਹੈ ਕਿ ਤੁਸੀਂ ਨੰਬਰ ਜੋੜਨਾ ਚਾਹੁੰਦੇ ਹੋ
- `add` ਟੂਲ ਚੁਣਦਾ ਹੈ
- `add(24.5, 17.3)` ਕਾਲ ਕਰਦਾ ਹੈ
- ਕੁਦਰਤੀ ਉੱਤਰ ਵਿੱਚ ਨਤੀਜਾ ਵਾਪਸ ਦਿੰਦਾ ਹੈ

## ਉਦਾਹਰਣਾਂ ਚਲਾਉਣਾ

### ਕਦਮ 1: ਕੈਲਕੂਲੇਟਰ ਸਰਵਰ ਸ਼ੁਰੂ ਕਰੋ

ਸਭ ਤੋਂ ਪਹਿਲਾਂ, ਸਾਇਨ ਇਨ ਕਰੋ ਅਤੇ ਆਪਣਾ Azure AI Foundry ਏਂਡਪੋਇੰਟ ਸੈੱਟ ਕਰੋ (AI ਕਲੀਐਂਟ ਲਈ ਲੋੜੀਂਦਾ — ਕੀਲੇਸ ਪ੍ਰਮਾਣਿਕਤਾ, ਕੋਈ API ਕੁੰਜੀ ਨਹੀਂ):

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

ਸਰਵਰ ਸਟਾਰਟ ਕਰੋ:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

ਸਰਵਰ `http://localhost:8080` ਤੇ ਚੱਲੇਗਾ। ਤੁਸੀਂ ਇਹ ਵੇਖੋਗੇ:
```
Started McpServerApplication in X.XXX seconds
```

### ਕਦਮ 2: ਡਾਇਰੈਕਟ ਕਲੀਐਂਟ ਨਾਲ ਟੈਸਟ ਕਰੋ

ਸਰਵਰ ਤੋਂ ਇਲਾਵਾ ਇੱਕ **ਨਵੀਂ** ਟਰਮੀਨਲ ਵਿੱਚ, ਡਾਇਰੈਕਟ MCP ਕਲੀਐਂਟ ਚਲਾਓ:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

ਤੁਹਾਨੂੰ ਆਉਟਪੁੱਟ ਇਸ ਤਰ੍ਹਾਂ ਦੇਖਣ ਨੂੰ ਮਿਲੇਗਾ:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### ਕਦਮ 3: AI ਕਲੀਐਂਟ ਨਾਲ ਟੈਸਟ ਕਰੋ

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

ਤੁਸੀਂ ਦੇਖੋਗੇ ਕਿ AI ਸਵੈਚਾਲਿਤ ਤਰੀਕੇ ਨਾਲ ਟੂਲ ਵਰਤਦਾ ਹੈ:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### ਕਦਮ 4: MCP ਸਰਵਰ ਬੰਦ ਕਰੋ

ਜਦੋਂ ਤੁਸੀਂ ਟੈਸਟਿੰਗ ਖਤਮ ਕਰ ਲੋ, ਤਾਂ AI ਕਲੀਐਂਟ ਨੂੰ ਉਸ ਦੀ ਟਰਮੀਨਲ ਵਿੱਚ `Ctrl+C` ਦਬਾ ਕੇ ਬੰਦ ਕਰੋ। MCP ਸਰਵਰ ਚਲਦਾ ਰਹੇਗਾ ਜਦ ਤੱਕ ਤੁਸੀਂ ਇਸ ਨੂੰ ਖ਼ਤਮ ਨਾ ਕਰੋ।
ਸਰਵਰ ਬੰਦ ਕਰਨ ਲਈ, ਉਸ ਟਰਮੀਨਲ ਵਿੱਚ `Ctrl+C` ਦਬਾਓ ਜਿੱਥੇ ਇਹ ਚੱਲ ਰਿਹਾ ਹੈ।

## ਇਹ ਸਾਰਾ ਕਿਵੇਂ ਕੰਮ ਕਰਦਾ ਹੈ

ਜਦੋਂ ਤੁਸੀਂ AI ਨੂੰ ਪੁੱਛਦੇ ਹੋ "5 + 3 ਕੀ ਹੈ?" ਤਾਂ ਹੇਠਾਂ ਦਿੱਤਾ ਪੂਰਾ ਪ੍ਰਕਿਰਿਆ ਚੱਲਦੀ ਹੈ:

1. **ਤੁਸੀਂ** ਕੁਦਰਤੀ ਭਾਸ਼ਾ ਵਿੱਚ AI ਨੂੰ ਪੁੱਛਦੇ ਹੋ
2. **AI** ਤੁਹਾਡੀ ਬੇਨਤੀ ਦਾ ਵਿਸ਼ਲੇਸ਼ਣ ਕਰਦਾ ਹੈ ਅਤੇ ਸਮਝਦਾ ਹੈ ਕਿ ਤੁਸੀਂ ਜੋੜ ਚਾਹੁੰਦੇ ਹੋ
3. **AI** MCP ਸਰਵਰ ਨੂੰ ਕਾਲ ਕਰਦਾ ਹੈ: `add(5.0, 3.0)`
4. **ਕੈਲਕੂਲੇਟਰ ਸਰਵਿਸ** ਕਰਦਾ ਹੈ: `5.0 + 3.0 = 8.0`
5. **ਕੈਲਕੂਲੇਟਰ ਸਰਵਿਸ** ਵਾਪਸ ਕਰਦਾ ਹੈ: `"5.00 + 3.00 = 8.00"`
6. **AI** ਨਤੀਜੇ ਨੂੰ ਪ੍ਰਾਪਤ ਕਰਦਾ ਹੈ ਅਤੇ ਕੁਦਰਤੀ ਜਵਾਬ ਬਣਾਉਂਦਾ ਹੈ
7. **ਤੁਸੀਂ** ਪਾਉਂਦੇ ਹੋ: "5 ਅਤੇ 3 ਦਾ ਜੋੜ 8 ਹੈ"

## ਅਗਲੇ ਕਦਮ

ਹੋਰ ਉਦਾਹਰਣਾਂ ਲਈ ਵੇਖੋ [ਅਧਿਆਇ 04: ਅਮਲੀ ਨਮੂਨੇ](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ਅਸਵੀਕਾਰੋਪਣ**:
ਇਸ ਦਸਤਾਵੇਜ਼ ਦਾ ਅਨੁਵਾਦ ਏਆਈ ਅਨੁਵਾਦ ਸੇਵਾ [Co-op Translator](https://github.com/Azure/co-op-translator) ਦੀ ਵਰਤੋਂ ਕਰਕੇ ਕੀਤਾ ਗਿਆ ਹੈ। ਜਦੋਂ ਕਿ ਅਸੀਂ ਸਹੀਤਾਵਾਂ ਲਈ ਯਤਨਸ਼ੀਲ ਹਾਂ, ਕਿਰਪਾ ਕਰਕੇ ਧਿਆਨ ਰੱਖੋ ਕਿ ਸਵੈਚਾਲਿਤ ਅਨੁਵਾਦਾਂ ਵਿੱਚ ਗਲਤੀਆਂ ਜਾਂ ਅਸਮੱਤਿਆਵਾਂ ਹੋ ਸਕਦੀਆਂ ਹਨ। ਮੂਲ ਦਸਤਾਵੇਜ਼ ਆਪਣੀ ਮੂਲ ਭਾਸ਼ਾ ਵਿੱਚ ਅਧਿਕਾਰਕ ਸਰੋਤ ਮੰਨਿਆ ਜਾਣਾ ਚਾਹੀਦਾ ਹੈ। ਜਰੂਰੀ ਜਾਣਕਾਰੀ ਲਈ, ਪੇਸ਼ੇਵਰ ਮਨੁੱਖੀ ਅਨੁਵਾਦ ਦੀ ਸਿਫ਼ਾਰਸ਼ ਕੀਤੀ ਜਾਂਦੀ ਹੈ। ਅਸੀਂ ਇਸ ਅਨੁਵਾਦ ਦੇ ਉਪਯੋਗ ਤੋਂ ਪੈਦਾ ਹੋਣ ਵਾਲੀਆਂ ਕਿਸੇ ਵੀ ਗਲਤਫਹਿਮੀਆਂ ਜਾਂ ਗਲਤ ਵਿਆਖਿਆਵਾਂ ਲਈ ਜਵਾਬਦੇਹ ਨਹੀਂ ਹਾਂ।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->