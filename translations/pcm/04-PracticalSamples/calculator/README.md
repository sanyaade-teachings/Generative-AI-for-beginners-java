# MCP Calculator Tutorial for Beginners

## Table of Contents

- [Wetin You Go Learn](#wetin-you-go-learn)
- [Wetin You Must Get Before](#wetin-you-must-get-before)
- [How To Understand Di Project Structure](#how-to-understand-di-project-structure)
- [Core Components We Explain](#core-components-we-explain)
  - [1. Main Application](#1-main-application)
  - [2. Calculator Service](#2-calculator-service)
  - [3. Direct MCP Client](#3-direct-mcp-client)
  - [4. AI-Powered Client](#4-ai-powered-client)
- [How To Run Di Examples](#how-to-run-di-examples)
- [How Everything Dey Work Together](#how-everything-dey-work-together)
- [Next Steps](#next-steps)

## Wetin You Go Learn

Dis tutorial go explain how to build calculator service wit Model Context Protocol (MCP). You go sabi:

- How to create service wey AI fit use as tool
- How to setup direct talk with MCP services
- How AI models fit automatically select which tools to use
- The difference between direct protocol calls and AI-assisted interaction dem

## Wetin You Must Get Before

Before you start, make sure say you get:
- Java 21 or any version wey dey higher
- Maven for managing dependencies
- Azure AI Foundry model deployment (make you deploy am wit `azd up` — check [Chapter 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- The [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), sign in wit `az login` (no API key needed)
- Basic knowledge of Java and Spring Boot

## How To Understand Di Project Structure

The calculator project get plenty important files:

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

## Core Components We Explain

### 1. Main Application

**File:** `McpServerApplication.java`

Na here calculator service start. E be typical Spring Boot application but e get one special thing:

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

**Wetin dis one dey do:**
- E go start Spring Boot web server for port 8080
- E go create `ToolCallbackProvider` wey go make our calculator methods dey available as MCP tools
- The `@Bean` annotation mean say Spring go manage am as component wey other parts fit use

### 2. Calculator Service

**File:** `CalculatorService.java`

Na here be where all math work dey happen. Each method get `@Tool` wey mean say MCP fit call am:

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
    
    // More calculator wahala dem...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Main features:**

1. **`@Tool` Annotation**: E tell MCP say dis method fit dey called by external clients
2. **Clear Descriptions**: Each tool get description wey go help AI models sabi wen to use am
3. **Consistent Return Format**: All operations dey return human-readable strings like "5.00 + 3.00 = 8.00"
4. **Error Handling**: Division by zero and negative square roots dey return error messages

**Operations wey dey available:**
- `add(a, b)` - Add two numbers
- `subtract(a, b)` - Subtract second number from first
- `multiply(a, b)` - Multiply two numbers
- `divide(a, b)` - Divide first by second (check for zero)
- `power(base, exponent)` - Raise base to power of exponent
- `squareRoot(number)` - Calculate square root (check for negative)
- `modulus(a, b)` - Return remainder of division
- `absolute(number)` - Return absolute value
- `help()` - Return info about all operations

### 3. Direct MCP Client

**File:** `SDKClient.java`

Dis client dey talk directly to MCP server without AI. E dey manually call specific calculator functions:

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
        
        // List all di tools wey dey available
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Call di specific calculator functions
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

**Wetin dis one dey do:**
1. **Connect** to calculator server at `http://localhost:8080` using builder pattern
2. **List** all available tools (our calculator functions)
3. **Call** specific functions wit exact parameters
4. **Print** results directly

**Note:** Dis example dey use Spring AI 1.1.0-SNAPSHOT dependency, wey introduce builder pattern for `WebFluxSseClientTransport`. If you dey use older stable version, you fit need use direct constructor.

**When to use dis:** When you sabi exactly which calculation you want perform and you want call am programmatically.

### 4. AI-Powered Client

**File:** `LangChain4jClient.java`

Dis client dey use AI model (GPT-4o-mini) wey fit automatically decide which calculator tools to take:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Arrange di AI model (Azure AI Foundry, no need key auth through Microsoft Entra ID)
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

        // Connect to our calculator MCP server
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Show wetin di AI dey do
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Allow di AI make e fit use our calculator tools
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Make one AI bot wey fit use our calculator
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Now we fit ask di AI to do calculations for natural language
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Wetin dis one dey do:**
1. **Create** AI model connection using your GitHub token
2. **Connect** AI to our calculator MCP server
3. **Give** AI access to all our calculator tools
4. **Allow** natural language commands like "Calculate the sum of 24.5 and 17.3"

**The AI automatically:**
- Understand say you wan add numbers
- Choose the `add` tool
- Call `add(24.5, 17.3)`
- Return result in normal way

## How To Run Di Examples

### Step 1: Start The Calculator Server

First, sign in and set your Azure AI Foundry endpoint (needed for AI client — no API key):

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

Start the server:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

Server go start for `http://localhost:8080`. You go see:
```
Started McpServerApplication in X.XXX seconds
```

### Step 2: Test wit Direct Client

For **NEW** terminal wey server still dey run, run direct MCP client:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

You go see output like:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Step 3: Test wit AI Client

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

You go see AI dey use tools automatically:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Step 4: Close The MCP Server

When finish your test, you fit stop AI client by pressing `Ctrl+C` for e terminal. MCP server go still dey run until you stop am.
To stop server, press `Ctrl+C` for terminal wey e dey run.

## How Everything Dey Work Together

This na how e go flow when you ask AI "Wetin be 5 + 3?":

1. **You** ask AI for natural language
2. **AI** go analyze your request, come sabi say you want add
3. **AI** go call MCP server: `add(5.0, 3.0)`
4. **Calculator Service** go perform: `5.0 + 3.0 = 8.0`
5. **Calculator Service** go return: `"5.00 + 3.00 = 8.00"`
6. **AI** go receive result, come format natural response
7. **You** go see: "The sum of 5 and 3 is 8"

## Next Steps

For more examples, check [Chapter 04: Practical samples](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Disclaimer**:
Dis document don translate wit AI translation service [Co-op Translator](https://github.com/Azure/co-op-translator). Even tho we dey try make am correct, abeg make you know say automated translation fit get errors or mistakes. Di original document for dia own language na im be di correct source. For important info, make person wey sabi human translation do am. We no go responsible for any misunderstanding or wrong understanding wey fit happen because of dis translation.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->