# MCP 計算機教學入門

## 目錄

- [你將學到什麼](#你將學到什麼)
- [先決條件](#先決條件)
- [了解專案結構](#了解專案結構)
- [核心元件說明](#核心元件說明)
  - [1. 主要應用程式](#1-主要應用程式)
  - [2. 計算服務](#2-計算服務)
  - [3. 直接 MCP 用戶端](#3-直接-mcp-用戶端)
  - [4. AI 驅動用戶端](#4-ai-驅動用戶端)
- [執行範例](#執行範例)
- [整合運作機制](#整合運作機制)
- [下一步](#下一步)

## 你將學到什麼

本教學說明如何使用模型上下文協議（Model Context Protocol，MCP）構建計算服務。你將了解到：

- 如何創建 AI 可用作工具的服務
- 如何設定與 MCP 服務的直接通信
- AI 模型如何自動選擇要使用的工具
- 直接協議呼叫與 AI 輔助互動的差異

## 先決條件

開始前，請確保你已具備：
- 安裝 Java 21 或更高版本
- 使用 Maven 進行依賴管理
- 一個 Azure AI Foundry 模型部署（可透過 `azd up` 佈建 — 參考[第二章](../../02-SetupDevEnvironment/getting-started-azure-openai.md)）
- 安裝 [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli)，並使用 `az login` 登入（無需金鑰身份驗證）
- 基本 Java 與 Spring Boot 知識

## 了解專案結構

計算機專案包含多個重要檔案：

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

**檔案:** `McpServerApplication.java`

這是我們計算機服務的進入點。它是一個標準的 Spring Boot 應用程式，並附加一個特別的功能：

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

**此段功能說明：**
- 啟動在 8080 埠的 Spring Boot 網頁伺服器
- 建立一個 `ToolCallbackProvider`，使我們的計算方法可作為 MCP 工具提供
- `@Bean` 註解表示 Spring 將它管理為其他部分可使用的元件

### 2. 計算服務

**檔案:** `CalculatorService.java`

所有數學運算都在這裡完成。每個方法都標示 `@Tool`，使其透過 MCP 可用：

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

**主要特點：**

1. **`@Tool` 註解**：告訴 MCP 該方法可被外部用戶端呼叫
2. <strong>明確描述</strong>：每個工具都有描述，協助 AI 模型判斷何時使用
3. <strong>一致的回傳格式</strong>：所有操作均回傳如「5.00 + 3.00 = 8.00」的人類可讀字串
4. <strong>錯誤處理</strong>：除以零與平方根負數會回傳錯誤訊息

**可用操作：**
- `add(a, b)` - 加法
- `subtract(a, b)` - 減法
- `multiply(a, b)` - 乘法
- `divide(a, b)` - 除法（含零檢查）
- `power(base, exponent)` - 次方
- `squareRoot(number)` - 開根號（含負數檢查）
- `modulus(a, b)` - 取餘數
- `absolute(number)` - 絕對值
- `help()` - 回傳所有操作資訊

### 3. 直接 MCP 用戶端

**檔案:** `SDKClient.java`

此用戶端直接與 MCP 伺服器溝通，不使用 AI，自行呼叫特定計算功能：

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
        
        // 呼叫特定計算器函數
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

**此段功能說明：**
1. 使用建構者模式連接至 `http://localhost:8080` 的計算機伺服器
2. 列出所有可用工具（即計算函式）
3. 以明確參數呼叫指定函式
4. 直接印出結果

**注意：** 此範例使用 Spring AI 1.1.0-SNAPSHOT 版本，引入了 `WebFluxSseClientTransport` 的建構者模式。如果你使用較舊穩定版，可能需要改用直接建構函式。

**使用時機：** 當你確定要執行哪個計算，並以程式化方式呼叫時。

### 4. AI 驅動用戶端

**檔案:** `LangChain4jClient.java`

此用戶端使用 GPT-4o-mini AI 模型，自動判斷要使用哪些計算機工具：

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // 設定 AI 模型（Azure AI Foundry，透過 Microsoft Entra ID 進行無鑰匙驗證）
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
                .logRequests(true)  // 顯示 AI 正在做什麼
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // 給 AI 訪問我們的計算器工具
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // 創建一個可以使用我們計算器的 AI 機器人
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // 現在我們可以用自然語言請 AI 進行計算
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**此段功能說明：**
1. 建立使用無鍵驗證（Microsoft Entra ID）的 AI 模型連線
2. 連接 AI 至我們的 MCP 計算機伺服器
3. 賦予 AI 存取所有計算機工具的權限
4. 允許自然語言請求，如「計算 24.5 與 17.3 的總和」

**AI 自動流程：**
- 理解你想執行加法
- 選擇 `add` 工具
- 呼叫 `add(24.5, 17.3)`
- 以自然回應方式回傳結果

## 執行範例

### 步驟 1：啟動計算機伺服器

首先登入並設定 Azure AI Foundry 端點（AI 用戶端需求 — 無鍵驗證，無 API 金鑰）：

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

伺服器將啟動於 `http://localhost:8080`，你應該會看到：
```
Started McpServerApplication in X.XXX seconds
```

### 步驟 2：使用直接用戶端測試

在另一個<strong>新終端機</strong>，伺服器仍執行時，執行直接 MCP 用戶端：
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

### 步驟 3：使用 AI 用戶端測試

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

你會看到 AI 自動使用工具：
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### 步驟 4：關閉 MCP 伺服器

測試完畢後，可在 AI 用戶端的終端機按下 `Ctrl+C` 停止它。MCP 伺服器將持續運行直到你手動停止。
若要停止伺服器，在執行該伺服器的終端機中按下 `Ctrl+C`。

## 整合運作機制

當你問 AI「5 + 3 等於多少？」時，完整流程如下：

1. <strong>你</strong>以自然語言向 AI 發問
2. <strong>AI</strong>分析請求，判斷你想做加法
3. <strong>AI</strong>呼叫 MCP 伺服器：`add(5.0, 3.0)`
4. <strong>計算服務</strong>執行運算：`5.0 + 3.0 = 8.0`
5. <strong>計算服務</strong>回傳字串：`"5.00 + 3.00 = 8.00"`
6. <strong>AI</strong>接收結果並產生自然回應
7. <strong>你</strong>收到回答：「5 與 3 的總和是 8」

## 下一步

更多範例請參考 [第四章：實用範例](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
此文件已使用 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 進行翻譯。雖然我們努力追求準確性，但請注意自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應視為權威來源。對於關鍵資訊，建議採用專業人工翻譯。我們不對因使用此翻譯所產生的任何誤解或誤譯承擔責任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->