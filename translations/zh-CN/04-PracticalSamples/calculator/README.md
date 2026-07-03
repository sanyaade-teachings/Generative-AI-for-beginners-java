# MCP 计算器新手教程

## 目录

- [你将学习到什么](#你将学习到什么)
- [先决条件](#先决条件)
- [了解项目结构](#了解项目结构)
- [核心组件解析](#核心组件解析)
  - [1. 主应用](#1-主应用)
  - [2. 计算器服务](#2-计算器服务)
  - [3. 直接 MCP 客户端](#3-直接-mcp-客户端)
  - [4. AI 驱动客户端](#4-ai-驱动客户端)
- [运行示例](#运行示例)
- [整体工作流程](#整体工作流程)
- [下一步](#下一步)

## 你将学习到什么

本教程讲解如何使用模型上下文协议（MCP）构建一个计算器服务。你将了解：

- 如何创建一个 AI 可作为工具调用的服务
- 如何设置与 MCP 服务的直接通信
- AI 模型如何自动选择使用哪些工具
- 直接协议调用与 AI 辅助交互的区别

## 先决条件

开始前，请确保你具备：
- 已安装 Java 21 或更高版本
- 使用 Maven 进行依赖管理
- 部署了 Azure AI Foundry 模型（使用 `azd up` 进行部署 — 参见[第2章](../../02-SetupDevEnvironment/getting-started-azure-openai.md)）
- 安装并登录了 [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli)，通过 `az login` 进行无密钥认证
- 具备基本的 Java 和 Spring Boot 知识

## 了解项目结构

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

## 核心组件解析

### 1. 主应用

**文件：** `McpServerApplication.java`

这是计算器服务的入口点。它是一个标准的 Spring Boot 应用，带有一个特别的新增部分：

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
- 创建 `ToolCallbackProvider`，使计算器方法可作为 MCP 工具调用
- `@Bean` 注解告知 Spring 该组件由框架管理，供其他部分使用

### 2. 计算器服务

**文件：** `CalculatorService.java`

这里实现了所有数学运算。每个方法都用 `@Tool` 注解标记，使其可通过 MCP 调用：

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

**主要特点：**

1. **`@Tool` 注解**：告诉 MCP 该方法可被外部客户端调用
2. <strong>清晰说明</strong>：每个工具有描述，帮助 AI 模型理解何时使用
3. <strong>统一返回格式</strong>：所有运算返回易读字符串，如 "5.00 + 3.00 = 8.00"
4. <strong>错误处理</strong>：除零和负数开方返回错误信息

**支持的操作：**
- `add(a, b)` - 两数相加
- `subtract(a, b)` - 从第一个数中减去第二个数
- `multiply(a, b)` - 两数相乘
- `divide(a, b)` - 第一个数除以第二个数（带零检查）
- `power(base, exponent)` - 基数的指数次方
- `squareRoot(number)` - 计算平方根（带负数检查）
- `modulus(a, b)` - 取余数
- `absolute(number)` - 取绝对值
- `help()` - 返回所有操作说明

### 3. 直接 MCP 客户端

**文件：** `SDKClient.java`

该客户端直接与 MCP 服务器通信，不依赖 AI。手动调用具体计算器函数：

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
1. 使用构建器模式连接至 `http://localhost:8080` 的计算器服务器
2. 列出所有可用工具（计算器函数）
3. 使用精确参数调用指定函数
4. 直接打印结果

**注意：** 本示例依赖 Spring AI 1.1.0-SNAPSHOT，其中引入了 `WebFluxSseClientTransport` 的构建器模式。若使用较早的稳定版本，可能需要使用直接构造函数。

**适用场景：** 确知具体想执行的计算，并希望通过程序调用时使用。

### 4. AI 驱动客户端

**文件：** `LangChain4jClient.java`

该客户端使用 AI 模型（GPT-4o-mini），可自动决定使用哪些计算器工具：

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // 设置AI模型（Azure AI Foundry，通过Microsoft Entra ID无密钥认证）
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

        // 连接到我们的计算器MCP服务器
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // 显示AI正在做什么
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // 赋予AI访问我们的计算器工具的权限
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // 创建一个能够使用我们计算器的AI机器人
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // 现在我们可以用自然语言让AI进行计算了
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**功能说明：**
1. 使用你的 GitHub 令牌创建 AI 模型连接
2. 将 AI 连接到我们的 MCP 计算器服务器
3. 授予 AI 访问所有计算器工具的权限
4. 允许自然语言请求，如“计算 24.5 和 17.3 的和”

**AI 会自动：**
- 理解你想做加法
- 选择 `add` 工具
- 调用 `add(24.5, 17.3)`
- 以自然语言形式返回结果

## 运行示例

### 步骤 1：启动计算器服务器

首先登录并设置 Azure AI Foundry 端点（AI 客户端需要，无需 API 密钥的无密钥认证）：

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

启动服务：
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

服务器将运行在 `http://localhost:8080`，你会看到：
```
Started McpServerApplication in X.XXX seconds
```

### 步骤 2：使用直接客户端测试

在<strong>新</strong>终端中且服务器仍在运行，运行直接 MCP 客户端：
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

你将看到类似输出：
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### 步骤 3：使用 AI 客户端测试

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

你将看到 AI 自动调用工具：
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### 步骤 4：关闭 MCP 服务器

测试完成后，可以在 AI 客户端终端按 `Ctrl+C` 停止客户端。MCP 服务器会继续运行，直到你在运行服务器的终端按 `Ctrl+C` 停止。

## 整体工作流程

当你问 AI “5 + 3 等于多少？”时，完整流程如下：

1. <strong>你</strong>用自然语言向 AI 提问
2. <strong>AI</strong>分析请求，识别你需要做加法
3. <strong>AI</strong>调用 MCP 服务器方法：`add(5.0, 3.0)`
4. <strong>计算器服务</strong>执行计算：`5.0 + 3.0 = 8.0`
5. <strong>计算器服务</strong>返回字符串：`"5.00 + 3.00 = 8.00"`
6. <strong>AI</strong>获取结果，生成自然语言回答
7. <strong>你</strong>收到回答：“5 和 3 的和是 8”

## 下一步

更多示例，请参见[第04章：实用示例](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免责声明**：
本文件由 AI 翻译服务 [Co-op Translator](https://github.com/Azure/co-op-translator) 翻译完成。尽管我们力求准确，但请注意，自动翻译可能包含错误或不准确之处。原始语言版文件应视为权威来源。对于重要信息，建议使用专业人工翻译。我们对因使用本翻译而产生的任何误解或误释不承担责任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->