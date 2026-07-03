# MCP 計算機入門教學

## 目錄

- [你將學到什麼](#你將學到什麼)
- [先決條件](#先決條件)
- [了解專案結構](#了解專案結構)
- [核心元件說明](#核心元件說明)
  - [1. 主要應用程式](#1-主要應用程式)
  - [2. 計算機服務](#2-計算機服務)
  - [3. 直接 MCP 客戶端](#3-直接-mcp-客戶端)
  - [4. AI 驅動的客戶端](#4-ai-驅動的客戶端)
- [執行範例](#執行範例)
- [整體運作流程](#整體運作流程)
- [接下來的步驟](#接下來的步驟)

## 你將學到什麼

本教學將解釋如何使用模型上下文協定（Model Context Protocol, MCP）建立一個計算機服務。你將了解：

- 如何建立 AI 可用作工具的服務
- 如何設定與 MCP 服務的直接通訊
- AI 模型如何自動選擇使用哪個工具
- 直接協定呼叫與 AI 輔助互動的差異

## 先決條件

開始之前，請確保你已具備：
- 安裝 Java 21 或以上版本
- 使用 Maven 進行相依管理
- 一個 Azure AI Foundry 模型部署（可用 `azd up` 佈署 — 參見 [第2章](../../02-SetupDevEnvironment/getting-started-azure-openai.md)）
- 已安裝並登入 [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) (`az login`，無需 API 金鑰)
- 具備 Java 與 Spring Boot 的基本知識

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

這是我們計算機服務的進入點。它是一個標準的 Spring Boot 應用程式，並有一個特別的設定：

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

**此程式做了什麼：**
- 啟動一個運行在 8080 端口的 Spring Boot 網路伺服器
- 建立一個 `ToolCallbackProvider`，使我們的計算機方法能作為 MCP 工具被使用
- `@Bean` 註解告訴 Spring 將它管理為其他元件可以使用的組件

### 2. 計算機服務

**檔案：** `CalculatorService.java`

這裡是所有計算發生的地方。每個方法用 `@Tool` 標記，可透過 MCP 被調用：

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

1. **`@Tool` 註解**：告訴 MCP 此方法可被外部客戶端呼叫
2. <strong>清晰描述</strong>：每個工具都有描述，幫助 AI 模型判斷何時使用
3. <strong>統一回傳格式</strong>：所有操作均回傳人類易讀的字串，例如 "5.00 + 3.00 = 8.00"
4. <strong>錯誤處理</strong>：除以零及負數開根號會回傳錯誤訊息

**可用運算：**
- `add(a, b)` - 加法
- `subtract(a, b)` - 減法（前減後）
- `multiply(a, b)` - 乘法
- `divide(a, b)` - 除法（含零除檢查）
- `power(base, exponent)` - 次方
- `squareRoot(number)` - 開平方（含負數檢查）
- `modulus(a, b)` - 取餘數
- `absolute(number)` - 絕對值
- `help()` - 回傳所有運算說明

### 3. 直接 MCP 客戶端

**檔案：** `SDKClient.java`

此客戶端直接與 MCP 伺服器通訊，不使用 AI，手動呼叫特定計算功能：

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

**此程式做了什麼：**
1. 使用建造者模式連接到 `http://localhost:8080` 的計算機伺服器
2. 列出所有可用工具（即計算機功能）
3. 使用精確參數呼叫特定功能
4. 直接列印結果

**注意：** 本範例使用 Spring AI 1.1.0-SNAPSHOT 版本，引入了 `WebFluxSseClientTransport` 的建造者模式。若你使用較舊穩定版本，可能需使用直接建構子。

**使用時機：** 當你已清楚知道要執行哪個計算，且想以程式化方式呼叫時。

### 4. AI 驅動的客戶端

**檔案：** `LangChain4jClient.java`

此客戶端使用 AI 模型（GPT-4o-mini）能自動決定使用哪些計算工具：

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // 設定 AI 模型（Azure AI Foundry，通過 Microsoft Entra ID 進行無密鑰驗證）
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

        // 讓 AI 能夠存取我們的計算器工具
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // 建立一個可以使用我們計算器的 AI 機械人
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // 現在我們可以用自然語言要求 AI 進行計算
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**此程式做了什麼：**
1. 利用你的 GitHub Token 建立 AI 模型連結
2. 將 AI 連接到我們的計算機 MCP 伺服器
3. 給 AI 存取全部計算機工具的權限
4. 允許自然語言請求，如「計算 24.5 與 17.3 的總和」

**AI 自動：**
- 理解你想要執行加法
- 選擇 `add` 工具
- 呼叫 `add(24.5, 17.3)`
- 以自然語言回應結果

## 執行範例

### 步驟 1：啟動計算機伺服器

首先，登入並設定你的 Azure AI Foundry 端點（AI 客戶端必須，無 API 金鑰）：

**Windows：**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**Linux/macOS：**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

啟動伺服器：
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

伺服器會運作於 `http://localhost:8080`，你應該會看到：
```
Started McpServerApplication in X.XXX seconds
```

### 步驟 2：使用直接客戶端測試

在伺服器運行的同時，開一個<strong>新</strong>終端機，執行直接 MCP 客戶端：
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

### 步驟 3：使用 AI 客戶端測試

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

你會看到 AI 自動使用工具：
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### 步驟 4：關閉 MCP 伺服器

測試完畢，於 AI 客戶端終端按 `Ctrl+C` 停止，MCP 伺服器會繼續執行直到你手動停止。
停止伺服器，在啟動它的終端機按 `Ctrl+C`。

## 整體運作流程

以下為當你問 AI「5 + 3 等於多少？」的完整流程：

1. <strong>你</strong> 用自然語言向 AI 提問
2. **AI** 分析請求，判斷你想要加法
3. **AI** 呼叫 MCP 伺服器：`add(5.0, 3.0)`
4. <strong>計算機服務</strong> 執行計算：`5.0 + 3.0 = 8.0`
5. <strong>計算機服務</strong> 回傳結果：`"5.00 + 3.00 = 8.00"`
6. **AI** 接收結果並組成自然語言回應
7. <strong>你</strong> 收到：「5 和 3 的總和是 8」

## 接下來的步驟

更多範例請參考 [第 04 章：實際範例](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
本文件由 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 翻譯而成。雖然我們致力於確保準確性，但請注意，機器自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應被視為權威來源。對於重要資訊，建議進行專業人工翻譯。我們不對因使用本翻譯而產生的任何誤解或誤釋承擔責任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->