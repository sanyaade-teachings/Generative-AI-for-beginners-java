# MCP 計算機初學者教學

## 目錄

- [您將學到什麼](#您將學到什麼)
- [先決條件](#先決條件)
- [了解專案結構](#了解專案結構)
- [核心元件說明](#核心元件說明)
  - [1. 主應用程式](#1-主應用程式)
  - [2. 計算機服務](#2-計算機服務)
  - [3. 直接 MCP 用戶端](#3-直接-mcp-用戶端)
  - [4. AI 驅動用戶端](#4-ai-驅動用戶端)
- [執行範例](#執行範例)
- [整體運作原理](#整體運作原理)
- [後續步驟](#後續步驟)

## 您將學到什麼

本教學說明如何使用 Model Context Protocol (MCP) 建立一個計算機服務。您將了解：

- 如何建立 AI 可用作工具的服務
- 如何設定與 MCP 服務的直接通訊
- AI 模型如何自動選擇使用哪些工具
- 直接協議調用與 AI 協助互動的差異

## 先決條件

開始前請確保您擁有：
- 已安裝 Java 21 或更高版本
- Maven 用於依賴管理
- 一個 Azure AI Foundry 模型部署（透過 `azd up` 設定 — 請參閱 [第 2 章](../../02-SetupDevEnvironment/getting-started-azure-openai.md)）
- 已登入的 [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli)，使用 `az login`（無需 API 金鑰）
- 基本的 Java 與 Spring Boot 知識

## 了解專案結構

計算機專案包含數個重要檔案：

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

### 1. 主應用程式

**檔案：** `McpServerApplication.java`

這是我們計算機服務的入口點。它是一個標準的 Spring Boot 應用程式，並有一個特別的新增：

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

**此程式做的事：**
- 啟動一個埠號為 8080 的 Spring Boot 網頁伺服器
- 建立一個 `ToolCallbackProvider`，使我們的計算方法能作為 MCP 工具被外部調用
- `@Bean` 註解讓 Spring 將它管理成一個其他元件可使用的組件

### 2. 計算機服務

**檔案：** `CalculatorService.java`

這裡是所有數學運算的地方。每個方法都用 `@Tool` 標記，使其可透過 MCP 被調用：

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

**關鍵特色：**

1. **`@Tool` 註解**：告訴 MCP 這些方法可被外部客戶端呼叫
2. <strong>清楚的說明</strong>：每個工具都有說明，幫助 AI 模型判斷何時使用
3. <strong>一致的回傳格式</strong>：所有運算回傳易讀的字串，例如 "5.00 + 3.00 = 8.00"
4. <strong>錯誤處理</strong>：除以零和負數開根號會回傳錯誤訊息

**可用的運算：**
- `add(a, b)` - 加法運算
- `subtract(a, b)` - 減法運算
- `multiply(a, b)` - 乘法運算
- `divide(a, b)` - 除法運算（含零除檢查）
- `power(base, exponent)` - 次方計算
- `squareRoot(number)` - 開平方（含負數檢查）
- `modulus(a, b)` - 取餘數
- `absolute(number)` - 取絕對值
- `help()` - 回傳所有運算說明

### 3. 直接 MCP 用戶端

**檔案：** `SDKClient.java`

此用戶端直接與 MCP 伺服器通訊，不透過 AI。它手動呼叫特定的計算功能：

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
        
        // 調用特定計算器功能
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

**此程式做的事：**
1. 使用建構者模式連接到 `http://localhost:8080` 的計算機伺服器
2. 列出所有可用工具（我們的計算函式）
3. 以精確參數呼叫特定函式
4. 直接印出結果

**注意事項：** 此範例使用 Spring AI 1.1.0-SNAPSHOT 版本，該版本引入了 `WebFluxSseClientTransport` 的建構者模式。如使用較舊穩定版，可能需要使用傳統建構子。

**使用時機：** 適合當你明確知道要執行哪項計算，並以程式碼呼叫時。

### 4. AI 驅動用戶端

**檔案：** `LangChain4jClient.java`

此用戶端使用 AI 模型（GPT-4o-mini）能自動決定使用哪些計算工具：

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // 設置 AI 模型（Azure AI Foundry，透過 Microsoft Entra ID 進行無金鑰驗證）
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
                .logRequests(true)  // 顯示 AI 的動作
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // 讓 AI 存取我們的計算器工具
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // 建立一個能使用我們計算器的 AI 機器人
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

**此程式做的事：**
1. 用您的 GitHub 令牌建立 AI 模型連線
2. 將 AI 連接到我們的計算機 MCP 伺服器
3. 讓 AI 存取所有計算機工具
4. 允許自然語言請求，例如「計算 24.5 和 17.3 的總和」

**AI 自動：**
- 理解您要執行加法
- 選擇 `add` 工具
- 呼叫 `add(24.5, 17.3)`
- 以自然語言回應結果

## 執行範例

### 第 1 步：啟動計算機伺服器

先登入並設定您的 Azure AI Foundry 端點（AI 用戶端所需 — 無需 API 金鑰）：

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

伺服器將在 `http://localhost:8080` 啟動。您應該會看到：
```
Started McpServerApplication in X.XXX seconds
```

### 第 2 步：使用直接用戶端測試

在保留伺服器執行的<strong>新</strong>終端機中，執行直接 MCP 用戶端：
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

您將看到類似輸出：
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### 第 3 步：使用 AI 用戶端測試

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

您將看到 AI 自動使用工具：
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### 第 4 步：關閉 MCP 伺服器

測試完成後，可在 AI 用戶端終端按 `Ctrl+C` 停止它。MCP 伺服器會持續運行，直到您手動停止。
若要停止伺服器，請在執行它的終端按 `Ctrl+C`。

## 整體運作原理

當您問 AI「5 + 3 是多少？」時，完整流程如下：

1. <strong>您</strong> 用自然語言詢問 AI
2. **AI** 解析請求，判斷您要加法
3. **AI** 呼叫 MCP 伺服器：`add(5.0, 3.0)`
4. <strong>計算機服務</strong> 執行：`5.0 + 3.0 = 8.0`
5. <strong>計算機服務</strong> 回傳：`"5.00 + 3.00 = 8.00"`
6. **AI** 收到結果，生成自然語言回應
7. <strong>您</strong> 收到：「5 與 3 的和是 8」

## 後續步驟

更多範例，請見 [第 04 章：實作範例](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
此文件已使用 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 進行翻譯。雖然我們努力追求準確性，但請注意自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應視為權威來源。對於關鍵資訊，建議採用專業人工翻譯。我們不對因使用此翻譯所產生的任何誤解或誤譯承擔責任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->