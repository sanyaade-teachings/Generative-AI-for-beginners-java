# MCP Calculator Tutorial fo Beginners

## Table of Contents

- [Wetn You Go Learn](#wetn-you-go-learn)
- [Wetin You Need First](#wetin-you-need-first)
- [How You Go Take Understand Di Project Structure](#how-you-go-take-understand-di-project-structure)
- [Core Components Wey Dem Explain](#core-components-wey-dem-explain)
  - [1. Main Application](#1-main-application)
  - [2. Calculator Service](#2-calculator-service)
  - [3. Direct MCP Client](#3-direct-mcp-client)
  - [4. AI-Powered Client](#4-ai-powered-client)
- [How To Run Di Examples](#how-to-run-di-examples)
- [How Everything Dey Work Together](#how-everything-dey-work-together)
- [Wet Next](#wet-next)

## Wetn You Go Learn

Dis tutorial go explain how to build calculator service using Model Context Protocol (MCP). You go sabi:

- How to create service wey AI fit use as tool
- How to set direct communication with MCP services
- How AI models fit automatically choose which tools dem go use
- The difference between direct protocol calls and AI-assisted interactions

## Wetin You Need First

Before you start, make sure sey you get:
- Java 21 or more installed
- Maven for dependency management
- Azure AI Foundry model deployment (deploy am with `azd up` — check [Chapter 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- The [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), sign in with `az login` (keyless auth)
- Basic understanding of Java and Spring Boot

## How You Go Take Understand Di Project Structure

The calculator project get some important files:

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

## Core Components Wey Dem Explain

### 1. Main Application

**File:** `McpServerApplication.java`

Na dis one be di entry point for our calculator service. Na normal Spring Boot app wey get one special addition:

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

**Wetin dis one de do:**
- E start Spring Boot web server for port 8080
- E create `ToolCallbackProvider` wey make our calculator methods dey available as MCP tools
- The `@Bean` annotation dey tell Spring make e manage am as one component wey other parts fit use

### 2. Calculator Service

**File:** `CalculatorService.java`

Na here all di math dem dey happen. Every method get `@Tool` make e dey available for MCP:

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

**Important tins:**

1. **`@Tool` Annotation**: E dey tell MCP say dis method fit be call by outside clients
2. **Clear Descriptions**: Every tool get description wey go help AI models understand when make dem use am
3. **Consistent Return Format**: All operation go return human-readable strings like "5.00 + 3.00 = 8.00"
4. **Error Handling**: Division by zero and negative square roots return error messages

**Operations We Dem Get:**
- `add(a, b)` - Add two numbers
- `subtract(a, b)` - Subtract second from first
- `multiply(a, b)` - Multiply two numbers
- `divide(a, b)` - Divide first by second (check if zero)
- `power(base, exponent)` - Raise base to power of exponent
- `squareRoot(number)` - Calculate square root (check if negative)
- `modulus(a, b)` - Return remainder of division
- `absolute(number)` - Return absolute value
- `help()` - Return information about all operations

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
        
        // List di tools wey dey available
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Call specific calculator functions
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

**Wetin dis one do:**
1. **Connect** to calculator server for `http://localhost:8080` using builder pattern
2. **List** all available tools (our calculator functions)
3. **Call** specific functions with exact parameters
4. **Print** the results direct

**Note:** Dis example dey use Spring AI 1.1.0-SNAPSHOT dependency wey bring builder pattern for `WebFluxSseClientTransport`. If you dey use older stable version, you fit need use direct constructor.

**When to use am:** When you sure exactly wetin calculation you want perform, and you want call am programmatically.

### 4. AI-Powered Client

**File:** `LangChain4jClient.java`

Dis client dey use AI model (GPT-4o-mini) wey fit automatically decide which calculator tools to use:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Setup di AI model (Azure AI Foundry, keyless auth through Microsoft Entra ID)
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

        // Give di AI access to our calculator tools
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Create AI bot we fit use our calculator
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Now we fit ask di AI to do calculations with normal talk
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Wetin e do:**
1. **Create** AI model connection using keyless authentication (Microsoft Entra ID)
2. **Connect** AI to our calculator MCP server
3. **Give** AI access to all our calculator tools
4. **Allow** natural language request like "Calculate the sum of 24.5 and 17.3"

**Di AI automatically:**
- Understand sey you want add numbers
- Choose `add` tool
- Call `add(24.5, 17.3)`
- Return di result inside natural response

## How To Run Di Examples

### Step 1: Start Di Calculator Server

First, sign in and set your Azure AI Foundry endpoint (needed for AI client — keyless auth, no API key):

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

Di server go start for `http://localhost:8080`. You go see:
```
Started McpServerApplication in X.XXX seconds
```

### Step 2: Test With Direct Client

For **NEW** terminal, while Server still dey run, run direct MCP client:
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

### Step 3: Test With AI Client

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

You go see AI dey automatically use tools:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Step 4: Stop Di MCP Server

When you don finish test, you fit stop AI client by press `Ctrl+C` for e terminal. MCP server go still dey run until you stop am.
To stop server, press `Ctrl+C` for the terminal wey e dey run.

## How Everything Dey Work Together

Na di full flow when you ask AI "Wetin be 5 + 3?":

1. **You** ask AI for natural language
2. **AI** analyze your request and realize say you want addition
3. **AI** call MCP server: `add(5.0, 3.0)`
4. **Calculator Service** do: `5.0 + 3.0 = 8.0`
5. **Calculator Service** return: `"5.00 + 3.00 = 8.00"`
6. **AI** receive di result and prepare natural response
7. **You** go get: "The sum of 5 and 3 is 8"

## Wet Next

For more examples, see [Chapter 04: Practical samples](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Disclaimer**:
Dis document don translate wit AI translation service [Co-op Translator](https://github.com/Azure/co-op-translator). Even tho we dey try make am correct, abeg make you know say automated translation fit get errors or mistakes. Di original document for dia own language na im be di correct source. For important info, make person wey sabi human translation do am. We no go responsible for any misunderstanding or wrong understanding wey fit happen because of dis translation.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->