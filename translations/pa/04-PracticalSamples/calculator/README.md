# MCP ਕੈਲਕੁਲੇਟਰ ਟਿਊਟੋਰਿਯਲ ਨਵੇਂ ਸ਼ੁਰੂ ਕਰ ਰਹੇ ਲੋਕਾਂ ਲਈ

## ਸੂਚੀ

- [ਤੁਸੀਂ ਕੀ ਸਿੱਖੋਗੇ](#ਤੁਸੀਂ-ਕੀ-ਸਿੱਖੋਗੇ)
- [ਜਰੂਰੀਆਂ ਸ਼ਰਤਾਂ](#ਜਰੂਰੀਆਂ-ਸ਼ਰਤਾਂ)
- [ਪ੍ਰੋਜੈਕਟ ਦੀ ਬਣਤਰ ਸਮਝਣਾ](#ਪ੍ਰੋਜੈਕਟ-ਦੀ-ਬਣਤਰ-ਸਮਝਣਾ)
- [ਮੁੱਖ ਹਿੱਸਿਆਂ ਦੀ ਵਿਆਖਿਆ](#ਮੁੱਖ-ਹਿੱਸਿਆਂ-ਦੀ-ਵਿਆਖਿਆ)
  - [1. ਮੁੱਖ ਐਪਲੀਕੇਸ਼ਨ](#1-ਮੁੱਖ-ਐਪਲੀਕੇਸ਼ਨ)
  - [2. ਕੈਲਕੁਲੇਟਰ ਸਰਵਿਸ](#2-ਕੈਲਕੁਲੇਟਰ-ਸਰਵਿਸ)
  - [3. ਡਾਇਰੈਕਟ MCP ਕਲਾਇੰਟ](#3-ਡਾਇਰੈਕਟ-mcp-ਕਲਾਇੰਟ)
  - [4. AI-ਚਲਿਤ ਕਲਾਇੰਟ](#4-ai-ਚਲਿਤ-ਕਲਾਇੰਟ)
- [ਉਦਾਹਰਣਾਂ ਚਲਾਉਣਾ](#ਉਦਾਹਰਣ-ਚਲਾਉਣਾ)
- [ਸਾਰੇ ਕੰਮ ਇਕੱਠੇ ਕਿਵੇਂ ਕਰਦੇ ਹਨ](#ਸਾਰੇ-ਕੰਮ-ਇਕੱਠੇ-ਕਿਵੇਂ-ਕਰਦੇ-ਹਨ)
- [ਅਗਲੇ ਕਦਮ](#ਅਗਲੇ-ਕਦਮ)

## ਤੁਸੀਂ ਕੀ ਸਿੱਖੋਗੇ

ਇਹ ਟਿਊਟੋਰਿਯਲ ਦੱਸਦਾ ਹੈ ਕਿ ਮਾਡਲ ਕੰਟੈਕਸਟ ਪ੍ਰੋਟੋਕੋਲ (MCP) ਦੀ ਵਰਤੋਂ ਕਰਦਿਆਂ ਕੈਲਕੁਲੇਟਰ ਸਰਵਿਸ ਕਿਵੇਂ ਬਣਾਈ ਜਾਵੇ। ਤੁਸੀਂ ਸਮਝੋਗੇ:

- ਇੱਕ ਐਸੀ ਸਰਵਿਸ ਕਿਵੇਂ ਬਣਾਈਏ ਜੋ AI ਟੂਲ ਵਜੋਂ ਵਰਤ ਸਕੇ
- MCP ਸਰਵਿਸਾਂ ਨਾਲ ਸਿੱਧਾ ਸੰਚਾਰ ਕਿਵੇਂ ਸੈੱਟ ਕਰਨਾ ਹੈ
- AI ਮਾਡਲ ਖੁਦ ਬੁਖੂਦੀ ਕਿਸ ਟੂਲ ਦੀ ਵਰਤੋਂ ਕਰਨੀ ਹੈ ਇਹ ਚੁਣ ਸਕਦੇ ਹਨ
- ਡਾਇਰੈਕਟ ਪ੍ਰੋਟੋਕੋਲ ਕਾਲਾਂ ਅਤੇ AI-ਮਦਦਦਾਰ ਇਨਟਰੈਕਸ਼ਨਾਂ ਵਿੱਚ ਫਰਕ

## ਜਰੂਰੀਆਂ ਸ਼ਰਤਾਂ

ਸ਼ੁਰੂ ਕਰਨ ਤੋਂ ਪਹਿਲਾਂ, ਇਹ ਯਕੀਨੀ ਬਣਾਓ ਕਿ ਤੁਹਾਡੇ ਕੋਲ ਹੈ:
- ਜਾਵਾ 21 ਜਾਂ ਉਪਰ ਦਾ ਸੰਸਕਰਨ ਇੰਸਟਾਲ ਕੀਤਾ ਹੋਇਆ
- MAVEN ਡਿਪੈਂਡੇਸੀ ਮੈਨੇਜਮੈਂਟ ਲਈ
- ਇੱਕ ਅਜ਼ਯਰ AI ਫਾਉਂਡਰੀ ਮਾਡਲ ਡਿਪਲੋਇਮੈਂਟ (ਇਸ ਨੂੰ `azd up` ਨਾਲ ਪ੍ਰੋਵਾਈਜ਼ਨ ਕਰੋ — ਦੇਖੋ [ਅਧਿਆਇ 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [ਅਜ਼ਯਰ CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), `az login` (ਬਿਨਾਂ ਕੀਂ ਦੇ ਹਵਾਲੇ ਨਾਲ) ਸਾਈਨ ਇਨ ਕੀਤਾ ਹੋਇਆ
- ਜਾਵਾ ਅਤੇ ਸਪ੍ਰਿੰਗ ਬੂਟ ਦੀ ਬੁਨਿਆਦੀ ਸਮਝ

## ਪ੍ਰੋਜੈਕਟ ਦੀ ਬਣਤਰ ਸਮਝਣਾ

ਕੈਲਕੁਲੇਟਰ ਪ੍ਰੋਜੈਕਟ ਵਿੱਚ ਕਈ ਮਹੱਤਵਪੂਰਨ ਫਾਈਲਾਂ ਹਨ:

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

ਇਹ ਸਾਡੇ ਕੈਲਕੁਲੇਟਰ ਸਰਵਿਸ ਦਾ ਐਂਟਰੀ ਪੌਇੰਟ ਹੈ। ਇਹ ਇੱਕ ਸਧਾਰਣ ਸਪ੍ਰਿੰਗ ਬੂਟ ਐਪਲੀਕੇਸ਼ਨ ਹੈ ਜਿਸ ਵਿੱਚ ਇੱਕ ਵਿਸ਼ੇਸ਼ ਸ਼ਾਮਲ ਕੀਤਾ ਗਇਆ ਹੈ:

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

**ਇਹ ਕੀ ਕਰਦਾ ਹੈ:**
- ਪੋਰਟ 8080 'ਤੇ ਇੱਕ ਸਪ੍ਰਿੰਗ ਬੂਟ ਵੈੱਬ ਸਰਵਰ ਸ਼ੁਰੂ ਕਰਦਾ ਹੈ
- `ToolCallbackProvider` ਬਣਾਉਂਦਾ ਹੈ ਜੋ ਸਾਡੇ ਕੈਲਕੁਲੇਟਰ ਮੈਥਡਜ਼ ਨੂੰ MCP ਟੂਲ ਵਜੋਂ ਉਪਲਬਧ ਕਰਵਾਉਂਦਾ ਹੈ
- `@Bean` ਐਨੋਟੇਸ਼ਨ ਸਪ੍ਰਿੰਗ ਨੂੰ ਦੱਸਦਾ ਹੈ ਕਿ ਇਹ ਇਕ ਕੰਪੋਨੈਂਟ ਪ੍ਰਬੰਧਨ ਵਿੱਚ ਲਿਆ ਜਾਵੇ ਜੋ ਹੋਰ ਹਿੱਸੇ ਵਰਤ ਸਕਦੇ ਹਨ

### 2. ਕੈਲਕੁਲੇਟਰ ਸਰਵਿਸ

**ਫਾਈਲ:** `CalculatorService.java`

ਇਥੇ ਸਾਰੀ ਗਣਿਤੀ ਹੁੰਦੀ ਹੈ। ਹਰ ਮੈਥਡ `@Tool` ਨਾਲ ਨਿਸ਼ਾਨ ਲਗਿਆ ਹੁੰਦਾ ਹੈ ਤਾਂ ਜੋ ਉਹ MCP ਵਿੱਚ ਉਪਲਬਧ ਹੋਣ:

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
    
    // ਹੋਰ ਕੈਲਕੁਲੇਟਰ ਆਪਰੇਸ਼ਨ...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**ਮੁੱਖ ਖਾਸੀਅਤਾਂ:**

1. **`@Tool` ਐਨੋਟੇਸ਼ਨ**: ਇਹ MCP ਨੂੰ ਦੱਸਦਾ ਹੈ ਕਿ ਇਹ ਮੈਥਡ ਬਾਹਰੀ ਕਲਾਇੰਟਾਂ ਵੱਲੋਂ ਕਾਲ ਕੀਤਾ ਜਾ ਸਕਦਾ ਹੈ
2. **ਸਪਸ਼ਟ ਵੇਰਵਾ**: ਹਰ ਟੂਲ ਕੋਲ ਵਰਨਨ ਹੈ ਜੋ AI ਮਾਡਲ ਨੂੰ ਇਹ ਸਮਝਣ ਵਿੱਚ ਸਹਾਇਤਾ ਕਰਦਾ ਹੈ ਕਿ ਕਦੋਂ ਇਸਦੀ ਵਰਤੋਂ ਕਰਨੀ ਹੈ
3. **ਇਕਸਾਰ ਰਿਟਰਨ ਫਾਰਮੈਟ**: ਸਾਰੇ ਆਪਰੇਸ਼ਨਾਂ ਦੇ ਨਤੀਜੇ ਮਨੁੱਖੀ ਪੜ੍ਹਨ ਵਾਲੇ ਸਤਰਾਂ ਵਾਂਗ "5.00 + 3.00 = 8.00" ਵਾਪਸ ਦਿੰਦੇ ਹਨ
4. **ਗਲਤੀ ਸਾਂਭਣਾ**: ਸਿਫ਼ਰ ਨਾਲ ਭਾਗ ਅਤੇ ਨਕਾਰਾਤਮਕ ਵਰਗਮੂਲ ਲਈ ਗਲਤੀ ਸੰਦਰਭ ਵਾਪਸ ਦਿੰਦਾ ਹੈ

**ਉਪਲਬਧ ਆਪਰੇਸ਼ਨ:**
- `add(a, b)` - ਦੋ ਨੰਬਰ ਜੋੜਦਾ ਹੈ
- `subtract(a, b)` - ਦੂਜੇ ਨੂੰ ਪਹਿਲੇ ਵਿੱਚੋਂ ਘਟਾਉਂਦਾ ਹੈ
- `multiply(a, b)` - ਦੋ ਨੰਬਰਾਂ ਦਾ ਗੁਣਾ ਕਰਦਾ ਹੈ
- `divide(a, b)` - ਪਹਿਲਾ ਨੰਬਰ ਦੂਜੇ ਨਾਲ ਭਾਗ ਦਿੰਦਾ ਹੈ (ਸਿਫ਼ਰ ਜਾਂਚ ਨਾਲ)
- `power(base, exponent)` - ਬੇਸ ਨੂੰ ਐਕਸਪੋਨੇਨਟ ਦੀ ਤਾਕਤ 'ਤੇ ਚੜ੍ਹਾਉਂਦਾ ਹੈ
- `squareRoot(number)` - ਵਰਗਮੂਲ ਕੱਢਦਾ ਹੈ (ਨਕਾਰਾਤਮਕ ਚੈੱਕ ਨਾਲ)
- `modulus(a, b)` - ਭਾਗ ਦਾ ਬਾਕੀ ਵਾਪਸ ਕਰਦਾ ਹੈ
- `absolute(number)` - ਮੁਲ ਦਾ ਅਬਸੋਲੂਟ ਵੈਲਯੂ ਦਿੰਦਾ ਹੈ
- `help()` - ਸਾਰਿਆਂ ਆਪਰੇਸ਼ਨਾਂ ਬਾਰੇ ਜਾਣਕਾਰੀ ਵਾਪਸ ਕਰਦਾ ਹੈ

### 3. ਡਾਇਰੈਕਟ MCP ਕਲਾਇੰਟ

**ਫਾਈਲ:** `SDKClient.java`

ਇਹ ਕਲਾਇੰਟ AI ਦੀ ਵਰਤੋਂ ਬਿਨਾਂ ਸਿੱਧਾ MCP ਸਰਵਰ ਨਾਲ ਗੱਲ ਕਰਦਾ ਹੈ। ਇਹ ਖਾਸ ਕੈਲਕੁਲੇਟਰ ਫੰਕਸ਼ਨਜ਼ ਨੂੰ ਮੈਨੁਅਲ ਤੌਰ 'ਤੇ ਕਾਲ ਕਰਦਾ ਹੈ:

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
        
        // ਉਪਲਬਧ ਸੰਦ ਸੂਚੀਬੱਧ ਕਰੋ
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // ਖਾਸ ਕੈਲਕुलेਟਰ ਫੰਕਸ਼ਨਾਂ ਨੂੰ ਕਾਲ ਕਰੋ
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

**ਇਹ ਕੀ ਕਰਦਾ ਹੈ:**
1. ਬਿਲਡਰ ਪੈਟਰਨ ਦੀ ਵਰਤੋਂ ਕਰਦਿਆਂ `http://localhost:8080` ਤੇ ਕੈਲਕੁਲੇਟਰ ਸਰਵਰ ਨਾਲ ਜੁੜਦਾ ਹੈ
2. ਸਾਰੇ ਉਪਲਬਧ ਟੂਲ ਲਿਸਟ ਕਰਦਾ ਹੈ (ਸਾਡੇ ਕੈਲਕੁਲੇਟਰ ਫੰਕਸ਼ਨ)
3. ਵਿਸ਼ੇਸ਼ ਫੰਕਸ਼ਨਾਂ ਨੂੰ ਅਸਲੀ ਪੈਰਾਮੀਟਰ ਨਾਲ ਕਾਲ ਕਰਦਾ ਹੈ
4. ਨਤੀਜੇ ਸਿੱਧੇ ਪ੍ਰਿੰਟ ਕਰਦਾ ਹੈ

**ਨੋਟ:** ਇਸ ਉਦਾਹਰਨ ਵਿੱਚ ਸਪ੍ਰਿੰਗ AI 1.1.0-SNAPSHOT ਡਿਪੈਂਡੇਸੀ ਦੀ ਵਰਤੋਂ ਕੀਤੀ ਗਈ ਹੈ, ਜਿਸ ਨੇ `WebFluxSseClientTransport` ਲਈ ਬਿਲਡਰ ਪੈਟਰਨ ਸ਼ੁਰੂ ਕੀਤਾ। ਜੇ ਤੁਸੀਂ ਪੁਰਾਣੀ ਸਥਿਰ ਵਰਜਨ ਵਰਤ ਰਹੇ ਹੋ ਤਾਂ ਤੁਹਾਨੂੰ ਸਿੱਧਾ ਕੰਸਟ੍ਰੱਕਟਰ ਵਰਤਣਾ ਪੈ ਸਕਦਾ ਹੈ।

**ਕਦੋਂ ਵਰਤਣਾ ਹੈ:** ਜਦੋਂ ਤੁਹਾਨੂੰ ਪੂਰੀ ਤਰ੍ਹਾਂ ਪਤਾ ਹੋਵੇ ਕਿ ਤੁਸੀਂ ਕਿਹੜਾ ਹਿਸਾਬ ਕਰਨਾ ਹੈ ਅਤੇ ਇਸ ਨੂੰ ਪ੍ਰੋਗਰਾਮਮੈਟਿਕ ਤੌਰ 'ਤੇ ਕਾਲ ਕਰਨਾ ਚਾਹੁੰਦੇ ਹੋ।

### 4. AI-ਚਲਿਤ ਕਲਾਇੰਟ

**ਫਾਈਲ:** `LangChain4jClient.java`

ਇਹ ਕਲਾਇੰਟ ਇੱਕ AI ਮਾਡਲ (GPT-4o-mini) ਦੀ ਵਰਤੋਂ ਕਰਦਾ ਹੈ ਜੋ ਖੁਦ-ਬੁਖੂਦੀ ਇਨਸਪੈਕਟ ਕਰਦਾ ਹੈ ਕਿ ਕਿਹੜੇ ਕੈਲਕੁਲੇਟਰ ਟੂਲ ਦੀ ਵਰਤੋਂ ਕਰਨੀ ਹੈ:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // ਏਆਈ ਮਾਡਲ ਸੈਟਅਪ ਕਰੋ (Azure AI Foundry, Microsoft Entra ID ਰਾਹੀਂ ਬਿਨਾ ਕੁੰਜੀ ਦੀ ਪ੍ਰਮਾਣੀਕਰਨ)
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

        // ਏਆਈ ਨੂੰ ਸਾਡੇ ਕੈਲਕੂਲੇਟਰ ਟੂਲਾਂ ਦੀ ਪਹੁੰਚ ਦਿਓ
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // ਇੱਕ ਏਆਈ ਬੌਟ ਬਣਾਓ ਜੋ ਸਾਡੇ ਕੈਲਕੂਲੇਟਰ ਨੂੰ ਵਰਤ ਸਕੇ
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // ਹੁਣ ਅਸੀਂ ਏਆਈ ਨੂੰ ਕੁਦਰਤੀ ਭਾਸ਼ਾ ਵਿੱਚ ਹਿਸਾਬ ਕਰਨ ਲਈ ਕਹਿ ਸਕਦੇ ਹਾਂ
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**ਇਹ ਕੀ ਕਰਦਾ ਹੈ:**
1. ਤੁਹਾਡੇ ਗਿਟਹੱਬ ਟੋਕਨ ਦੀ ਵਰਤੋਂ ਨਾਲ AI ਮਾਡਲ ਕਨੈਕਸ਼ਨ ਬਣਾਉਂਦਾ ਹੈ
2. AI ਨੂੰ ਸਾਡੇ ਕੈਲਕੁਲੇਟਰ MCP ਸਰਵਰ ਨਾਲ ਜੋੜਦਾ ਹੈ
3. AI ਨੂੰ ਸਾਡੇ ਸਾਰੇ ਕੈਲਕੁਲੇਟਰ ਟੂਲ ਦੀ ਪਹੁੰਚ ਦਿੰਦਾ ਹੈ
4. ਕੁਦਰਤੀ ਭਾਸ਼ਾ ਵਾਲੀਆਂ ਬੇਨਤੀਆਂ ਵਰਗੇ "24.5 ਅਤੇ 17.3 ਦਾ ਜੋੜ ਕੱਡੋ" ਨੂੰ ਸਮਝਦਾ ਹੈ

**AI ਖੁਦ-ਬੁਖੂਦੀ:**
- ਸਮਝਦਾ ਹੈ ਕਿ ਤੁਸੀਂ ਗਿਣਤੀ ਜੋੜਨਾ ਚਾਹੁੰਦੇ ਹੋ
- `add` ਟੂਲ ਨੂੰ ਚੁਣਦਾ ਹੈ
- `add(24.5, 17.3)` ਕਾਲ ਕਰਦਾ ਹੈ
- ਨਤੀਜਾ ਕੁਦਰਤੀ ਜਵਾਬ ਵਿੱਚ ਵਾਪਸ ਕਰਦਾ ਹੈ

## ਉਦਾਹਰਣ ਚਲਾਉਣਾ

### ਕਦਮ 1: ਕੈਲਕੁਲੇਟਰ ਸਰਵਰ ਸ਼ੁਰੂ ਕਰੋ

ਪਹਿਲਾਂ, ਸਾਇਨ ਇਨ ਕਰੋ ਅਤੇ ਆਪਣਾ ਅਜ਼ਯਰ AI ਫਾਉਂਡਰੀ ਐਂਡਪੌਇੰਟ ਸੈੱਟ ਕਰੋ (AI ਕਲਾਇੰਟ ਲਈ ਜਰੂਰੀ — ਬਿਨਾਂ API ਕੀ):

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

ਸਰਵਰ ਸ਼ੁਰੂ ਕਰੋ:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

ਸਰਵਰ `http://localhost:8080` 'ਤੇ ਸ਼ੁਰੂ ਹੋਵੇਗਾ। ਤੁਹਾਨੂੰ ਇਹ ਦਿਖਾਈ ਦੇਣਾ ਚਾਹੀਦਾ ਹੈ:
```
Started McpServerApplication in X.XXX seconds
```

### ਕਦਮ 2: ਡਾਇਰੈਕਟ ਕਲਾਇੰਟ ਨਾਲ ਟੈਸਟ ਕਰੋ

ਇੱਕ **ਨਵੀਂ** ਟਰਮੀਨਲ ਵਿਚ ਜਿੱਥੇ ਸਰਵਰ ਚੱਲ ਰਿਹਾ ਹੈ, ਡਾਇਰੈਕਟ MCP ਕਲਾਇੰਟ ਚਲਾਓ:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

ਤੁਹਾਨੂੰ ਇਸ ਤਰ੍ਹਾਂ ਆਉਟਪੁੱਟ ਮਿਲੇਗੀ:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### ਕਦਮ 3: AI ਕਲਾਇੰਟ ਨਾਲ ਟੈਸਟ ਕਰੋ

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

ਤੁਸੀਂ AI ਨੂੰ ਖੁਦ-ਬੁਖੂਦੀ ਟੂਲ ਵਰਤਦੇ ਹੋਏ ਦੇਖੋਗੇ:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### ਕਦਮ 4: MCP ਸਰਵਰ ਬੰਦ ਕਰੋ

ਟੈਸਟ ਕਰਨਾ ਮੁਕੰਮਲ ਹੋਣ 'ਤੇ, AI ਕਲਾਇੰਟ ਨੂੰ ਉਸ ਦੀ ਟਰਮੀਨਲ ਵਿੱਚ `Ctrl+C` ਦਬਾ ਕੇ ਬੰਦ ਕੀਤਾ ਜਾ ਸਕਦਾ ਹੈ। MCP ਸਰਵਰ ਚੱਲਦਾ ਰਹਿੰਦਾ ਹੈ ਜਦ ਤੱਕ ਤੁਸੀਂ ਇਸਨੂੰ ਬੰਦ ਨਾ ਕਰੋ।
ਸਰਵਰ ਬੰਦ ਕਰਨ ਲਈ, ਜਿਸ ਟਰਮੀਨਲ ਵਿੱਚ ਕਦੇ ਚੱਲ ਰਹਾ ਹੈ ਉੱਥੇ `Ctrl+C` ਦਬਾਓ।

## ਸਾਰੇ ਕੰਮ ਇਕੱਠੇ ਕਿਵੇਂ ਕਰਦੇ ਹਨ

ਜਦੋਂ ਤੁਸੀਂ AI ਨੂੰ ਪੁੱਛਦੇ ਹੋ "5 + 3 ਕੀ ਹੈ?" ਤਾਂ ਪੂਰਾ ਪ੍ਰਕਿਰਿਆ ਇਹ ਹੈ:

1. **ਤੁਸੀਂ** ਕੁਦਰਤੀ ਭਾਸ਼ਾ ਵਿੱਚ AI ਨੂੰ ਪੁੱਛਦੇ ਹੋ
2. **AI** ਤੁਹਾਡੇ ਬੇਨਤੀ ਦੀ ਵਿਸ਼ਲੇਸ਼ਣਾ ਕਰਦਾ ਹੈ ਅਤੇ ਸਮਝਦਾ ਹੈ ਕਿ ਤੁਸੀਂ ਜੋੜ ਕਰਨਾ ਚਾਹੁੰਦੇ ਹੋ
3. **AI** MCP ਸਰਵਰ ਨੂੰ `add(5.0, 3.0)` ਕਾਲ ਕਰਦਾ ਹੈ
4. **ਕੈਲਕੁਲੇਟਰ ਸਰਵਿਸ** ਕਰਦਾ ਹੈ: `5.0 + 3.0 = 8.0`
5. **ਕੈਲਕੁਲੇਟਰ ਸਰਵਿਸ** ਵਾਪਸ ਕਰਦਾ ਹੈ: `"5.00 + 3.00 = 8.00"`
6. **AI** ਨਤੀਜਾ ਪ੍ਰਾਪਤ ਕਰਦਾ ਹੈ ਅਤੇ ਕੁਦਰਤੀ ਜਵਾਬ ਤਿਆਰ ਕਰਦਾ ਹੈ
7. **ਤੁਹਾਨੂੰ** ਮਿਲਦਾ ਹੈ: "5 ਅਤੇ 3 ਦਾ ਜੋੜ 8 ਹੈ"

## ਅਗਲੇ ਕਦਮ

ਹੋਰ ਉਦਾਹਰਣਾਂ ਲਈ, ਦੇਖੋ [ਅਧਿਆਇ 04: ਪ੍ਰਯੋਗਿਕ ਨਮੂਨੇ](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ਅਸਵੀਕਾਰੋਪਣ**:
ਇਸ ਦਸਤਾਵੇਜ਼ ਦਾ ਅਨੁਵਾਦ ਏਆਈ ਅਨੁਵਾਦ ਸੇਵਾ [Co-op Translator](https://github.com/Azure/co-op-translator) ਦੀ ਵਰਤੋਂ ਕਰਕੇ ਕੀਤਾ ਗਿਆ ਹੈ। ਜਦੋਂ ਕਿ ਅਸੀਂ ਸਹੀਤਾਵਾਂ ਲਈ ਯਤਨਸ਼ੀਲ ਹਾਂ, ਕਿਰਪਾ ਕਰਕੇ ਧਿਆਨ ਰੱਖੋ ਕਿ ਸਵੈਚਾਲਿਤ ਅਨੁਵਾਦਾਂ ਵਿੱਚ ਗਲਤੀਆਂ ਜਾਂ ਅਸਮੱਤਿਆਵਾਂ ਹੋ ਸਕਦੀਆਂ ਹਨ। ਮੂਲ ਦਸਤਾਵੇਜ਼ ਆਪਣੀ ਮੂਲ ਭਾਸ਼ਾ ਵਿੱਚ ਅਧਿਕਾਰਕ ਸਰੋਤ ਮੰਨਿਆ ਜਾਣਾ ਚਾਹੀਦਾ ਹੈ। ਜਰੂਰੀ ਜਾਣਕਾਰੀ ਲਈ, ਪੇਸ਼ੇਵਰ ਮਨੁੱਖੀ ਅਨੁਵਾਦ ਦੀ ਸਿਫ਼ਾਰਸ਼ ਕੀਤੀ ਜਾਂਦੀ ਹੈ। ਅਸੀਂ ਇਸ ਅਨੁਵਾਦ ਦੇ ਉਪਯੋਗ ਤੋਂ ਪੈਦਾ ਹੋਣ ਵਾਲੀਆਂ ਕਿਸੇ ਵੀ ਗਲਤਫਹਿਮੀਆਂ ਜਾਂ ਗਲਤ ਵਿਆਖਿਆਵਾਂ ਲਈ ਜਵਾਬਦੇਹ ਨਹੀਂ ਹਾਂ।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->