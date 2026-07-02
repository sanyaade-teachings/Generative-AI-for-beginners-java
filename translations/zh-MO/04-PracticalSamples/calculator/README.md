# MCP 計算機教程入門

## 目錄

- [你將學到什麼](#你將學到什麼)
- [先決條件](#先決條件)
- [了解專案結構](#了解專案結構)
- [核心元件說明](#核心元件說明)
  - [1. 主要應用程式](#1-主要應用程式)
  - [2. 計算機服務](#2-計算機服務)
  - [3. 直接 MCP 用戶端](#3-直接-mcp-用戶端)
  - [4. AI 驅動用戶端](#4-ai-驅動用戶端)
- [執行範例](#執行範例)
- [整體運作方式](#整體運作方式)
- [後續步驟](#後續步驟)

## 你將學到什麼

本教程說明如何使用模型上下文協議（Model Context Protocol，MCP）建構計算機服務。你將會理解：

- 如何建立 AI 能使用作為工具的服務
- 如何設定與 MCP 服務的直接通訊
- AI 模型如何自動選擇使用哪些工具
- 直接協議呼叫與 AI 協助互動的差異

## 先決條件

開始前，請確定你已具備：
- 安裝 Java 21 或更高版本
- 使用 Maven 進行依賴管理
- 已部署 Azure AI Foundry 模型（透過 `azd up` 佈署 — 詳見[第二章](../../02-SetupDevEnvironment/getting-started-azure-openai.md)）
- 安裝[Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli)，並以 `az login` 登入（無需 API 金鑰驗證）
- 基本的 Java 與 Spring Boot 知識

## 了解專案結構

計算機專案包含幾個重要檔案：

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

## 核心元件說明

### 1. 主要應用程式

**檔案：** `McpServerApplication.java`

這是我們計算機服務的進入點。它是一個標準的 Spring Boot 應用程式，帶有一個特別附加：

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

**作用：**
- 在 8080 埠啟動 Spring Boot Web 伺服器
- 建立一個 `ToolCallbackProvider`，使計算機方法可作為 MCP 工具被使用
- `@Bean` 註解讓 Spring 將其管理為可供其它元件使用的組件

### 2. 計算機服務

**檔案：** `CalculatorService.java`

所有計算邏輯都在這裡。每個方法都以 `@Tool` 註解標記，讓 MCP 可調用：

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
    
    // 更多計算器操作...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**主要特色：**

1. **`@Tool` 註解**：告訴 MCP 此方法可被外部用戶端呼叫
2. <strong>清楚的描述</strong>：每個工具有描述，幫助 AI 模型理解使用時機
3. <strong>一致的回傳格式</strong>：所有運算皆傳回如 "5.00 + 3.00 = 8.00" 的人類可讀字串
4. <strong>錯誤處理</strong>：除以零和負數平方根會回傳錯誤訊息

**可用運算：**
- `add(a, b)` - 兩數相加
- `subtract(a, b)` - 從第一數減去第二數
- `multiply(a, b)` - 兩數相乘
- `divide(a, b)` - 第一數除以第二數（含除零檢查）
- `power(base, exponent)` - 計算次方
- `squareRoot(number)` - 計算平方根（含負數檢查）
- `modulus(a, b)` - 回傳除法餘數
- `absolute(number)` - 回傳絕對值
- `help()` - 回傳所有運算說明

### 3. 直接 MCP 用戶端

**檔案：** `SDKClient.java`

此用戶端不透過 AI，直接呼叫 MCP 伺服器。手動呼叫特定的計算機函數：

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
        
        // 呼叫特定計算器功能
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

**作用：**
1. 使用建造者模式連線至 `http://localhost:8080` 計算機伺服器
2. 列出所有可用工具（我們的計算機函數）
3. 使用精確參數呼叫指定函數
4. 直接印出結果

<strong>注意：</strong>範例使用 Spring AI 1.1.0-SNAPSHOT 版本，該版本新導入了 `WebFluxSseClientTransport` 的建造者模式。若使用較舊穩定版，可能需改用直接建構子。

<strong>適用時機：</strong>當你明確知道要計算什麼，並想以程式化方式呼叫時。

### 4. AI 驅動用戶端

**檔案：** `LangChain4jClient.java`

此用戶端使用 AI 模型（GPT-4o-mini），可自動判斷需使用哪些計算工具：

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // 設定 AI 模型（Azure AI Foundry，透過 Microsoft Entra ID 進行無金鑰認證）
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

        // 連接到我們的計算器 MCP 伺服器
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // 顯示 AI 正在執行的操作
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // 給予 AI 使用我們計算器工具的權限
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // 創建一個能使用我們計算器的 AI 機械人
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // 現在我們可以讓 AI 使用自然語言進行計算
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**作用：**
1. 使用你的 GitHub 令牌建立 AI 模型連線
2. 連接 AI 至我們的計算機 MCP 伺服器
3. 提供 AI 存取所有計算機工具
4. 支援自然語言請求，如「計算 24.5 和 17.3 的總和」

**AI 自動：**
- 了解你想做加法
- 選擇 `add` 工具
- 呼叫 `add(24.5, 17.3)`
- 以自然回應格式回傳結果

## 執行範例

### 第 1 步：啟動計算機伺服器

首先登入並設定你的 Azure AI Foundry 端點（AI 用戶端需，無需 API 金鑰驗證）：

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

啟動伺服器：
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

伺服器將在 `http://localhost:8080` 運行。你應會看到：
```
Started McpServerApplication in X.XXX seconds
```

### 第 2 步：使用直接用戶端測試

在一個<strong>全新</strong>終端機，且伺服器仍在運行，執行直接 MCP 用戶端：
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

你會看到類似輸出：
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### 第 3 步：使用 AI 用戶端測試

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

你會看到 AI 自動使用工具：
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### 第 4 步：關閉 MCP 伺服器

測試完成後，可在 AI 用戶端終端機按下 `Ctrl+C` 停止。MCP 伺服器會繼續運行，直到你在伺服器終端機按 `Ctrl+C` 停止。

## 整體運作方式

下面是你問 AI「5 + 3 是多少？」時的完整流程：

1. <strong>你</strong>以自然語言問 AI
2. <strong>AI</strong>分析請求，判斷你想做加法
3. <strong>AI</strong>呼叫 MCP 伺服器：`add(5.0, 3.0)`
4. <strong>計算機服務</strong>執行：`5.0 + 3.0 = 8.0`
5. <strong>計算機服務</strong>回傳結果：`"5.00 + 3.00 = 8.00"`
6. <strong>AI</strong>接收結果並組成自然回應
7. <strong>你</strong>得到答案：「5 加 3 的和是 8」

## 後續步驟

更多範例，請參見[第四章：實用範例](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
本文件使用 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 進行翻譯。雖然我們力求準確，但請注意，自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應被視為權威來源。對於重要資訊，建議尋求專業人工翻譯。我們不對因使用本翻譯而引起的任何誤解或曲解承擔責任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->