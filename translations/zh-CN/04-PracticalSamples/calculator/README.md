# MCP 计算器初学者教程

## 目录

- [你将学到什么](#你将学到什么)
- [先决条件](#先决条件)
- [理解项目结构](#理解项目结构)
- [核心组件讲解](#核心组件讲解)
  - [1. 主应用程序](#1-主应用程序)
  - [2. 计算器服务](#2-计算器服务)
  - [3. 直接 MCP 客户端](#3-直接-mcp-客户端)
  - [4. AI 驱动客户端](#4-ai-驱动客户端)
- [运行示例](#运行示例)
- [它们如何协同工作](#它们如何协同工作)
- [下一步](#下一步)

## 你将学到什么

本教程解释如何使用模型上下文协议（MCP）构建一个计算器服务。你将了解：

- 如何创建 AI 可用作工具的服务
- 如何设置与 MCP 服务的直接通信
- AI 模型如何自动选择要使用的工具
- 直接协议调用与 AI 辅助交互的区别

## 先决条件

在开始之前，请确保你已经具备：
- 安装了 Java 21 或更高版本
- 用于依赖管理的 Maven
- Azure AI Foundry 模型部署（使用 `azd up` 进行预配 — 参见[第2章](../../02-SetupDevEnvironment/getting-started-azure-openai.md)）
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli)，已使用 `az login` 登录（无密钥认证）
- 基本的 Java 和 Spring Boot 知识

## 理解项目结构

计算器项目包含几个重要文件：

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

## 核心组件讲解

### 1. 主应用程序

**文件：** `McpServerApplication.java`

这是我们计算器服务的入口点。它是一个标准的 Spring Boot 应用程序，但有一个特殊的附加项：

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

**功能说明：**
- 在端口 8080 启动 Spring Boot Web 服务器
- 创建一个 `ToolCallbackProvider` ，将我们的计算器方法作为 MCP 工具暴露
- `@Bean` 注解告诉 Spring 把它当作组件管理，供其他部分使用

### 2. 计算器服务

**文件：** `CalculatorService.java`

所有数学计算都在这里完成。每个方法都用 `@Tool` 标记，让 MCP 可以调用：

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
    
    // 更多计算器操作...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**主要特性：**

1. **`@Tool` 注解**：告诉 MCP 该方法可被外部客户端调用
2. <strong>清晰描述</strong>：每个工具都有描述，帮助 AI 模型理解何时使用它
3. <strong>统一返回格式</strong>：所有操作返回人类可读的字符串，如 "5.00 + 3.00 = 8.00"
4. <strong>错误处理</strong>：除零和负数开方都会返回错误消息

**可用操作：**
- `add(a, b)` - 两数相加
- `subtract(a, b)` - 第二个数从第一个数减去
- `multiply(a, b)` - 两数相乘
- `divide(a, b)` - 第一个数除以第二个数（含零检测）
- `power(base, exponent)` - 求 base 的 exponent 次幂
- `squareRoot(number)` - 计算平方根（含负数检测）
- `modulus(a, b)` - 求余数
- `absolute(number)` - 绝对值
- `help()` - 返回所有操作的信息

### 3. 直接 MCP 客户端

**文件：** `SDKClient.java`

该客户端直接与 MCP 服务器通信，不使用 AI。它手动调用具体的计算器函数：

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
        
        // 列出可用工具
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // 调用特定的计算器功能
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

**功能说明：**
1. <strong>连接</strong> 到位于 `http://localhost:8080` 的计算器服务器，使用构建者模式
2. <strong>列出</strong> 所有可用工具（计算器函数）
3. <strong>调用</strong> 带有准确参数的具体函数
4. <strong>打印</strong> 结果

**注意：** 示例使用 Spring AI 1.1.0-SNAPSHOT 依赖，新增了 `WebFluxSseClientTransport` 的构建者模式。如果你使用的是旧版本稳定版，可能需要直接使用构造器。

**适用场景：** 当你明确知道要执行哪个计算，并想以编程方式调用时使用。

### 4. AI 驱动客户端

**文件：** `LangChain4jClient.java`

该客户端使用 AI 模型（GPT-4o-mini）自动决定使用哪些计算器工具：

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // 设置 AI 模型（Azure AI Foundry，通过 Microsoft Entra ID 无密钥认证）
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

        // 连接到我们的计算器 MCP 服务器
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // 显示 AI 正在做什么
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // 给予 AI 访问我们的计算器工具的权限
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // 创建一个可以使用我们计算器的 AI 机器人
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // 现在我们可以用自然语言让 AI 进行计算
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**功能说明：**
1. <strong>创建</strong> AI 模型连接，使用无密钥认证（Microsoft Entra ID）
2. <strong>将</strong> AI 连接至我们的 MCP 计算器服务器
3. <strong>赋予</strong> AI 访问我们所有计算器工具的权限
4. <strong>支持</strong> 自然语言请求，例如“计算 24.5 和 17.3 的和”

**AI 自动执行：**
- 识别你想要加法
- 选择 `add` 工具
- 调用 `add(24.5, 17.3)`
- 返回自然语言格式的结果

## 运行示例

### 第1步：启动计算器服务器

首先，登录并设置你的 Azure AI Foundry 终结点（AI 客户端需要，无密钥认证，无需 API 密钥）：

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

启动服务器：
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

服务器将在 `http://localhost:8080` 启动。你应看到：
```
Started McpServerApplication in X.XXX seconds
```

### 第2步：使用直接客户端测试

在服务器运行的同时，打开一个<strong>新</strong>终端，运行直接 MCP 客户端：
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

你会看到类似输出：
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### 第3步：使用 AI 客户端测试

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

你会看到 AI 自动使用工具：
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### 第4步：关闭 MCP 服务器

测试完成后，可以在 AI 客户端终端按 `Ctrl+C` 停止它。MCP 服务器会继续运行，直到你手动停止。
停止服务器时，在其终端按 `Ctrl+C`。

## 它们如何协同工作

当你问 AI “5 + 3 等于多少？”的完整流程如下：

1. <strong>你</strong> 用自然语言向 AI 提问
2. **AI** 分析请求，判断你要做加法
3. **AI** 调用 MCP 服务器：`add(5.0, 3.0)`
4. <strong>计算器服务</strong> 执行：`5.0 + 3.0 = 8.0`
5. <strong>计算器服务</strong> 返回：`"5.00 + 3.00 = 8.00"`
6. **AI** 接收结果并格式化为自然语言回答
7. <strong>你</strong> 得到回应：“5 和 3 的和是 8”

## 下一步

更多示例，请参见 [第 04 章：实用示例](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免责声明**：
本文件由 AI 翻译服务 [Co-op Translator](https://github.com/Azure/co-op-translator) 翻译完成。尽管我们力求准确，但请注意，自动翻译可能包含错误或不准确之处。原始语言版文件应视为权威来源。对于重要信息，建议使用专业人工翻译。我们对因使用本翻译而产生的任何误解或误释不承担责任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->