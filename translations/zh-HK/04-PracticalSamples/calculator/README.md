# MCP 計算機教學初學者指南

## 目錄

- [你將學習什麼](#你將學習什麼)
- [前置條件](#前置條件)
- [了解專案結構](#了解專案結構)
- [核心組件說明](#核心組件說明)
  - [1. 主應用程式](#1-主應用程式)
  - [2. 計算機服務](#2-計算機服務)
  - [3. 直接 MCP 用戶端](#3-直接-mcp-用戶端)
  - [4. AI 驅動用戶端](#4-ai-驅動用戶端)
- [執行範例](#執行範例)
- [整體運作原理](#整體運作原理)
- [後續步驟](#後續步驟)

## 你將學習什麼

本教學會說明如何使用模型上下文協定 (Model Context Protocol, MCP) 建立一個計算機服務。你將了解：

- 如何建立可供 AI 作為工具使用的服務
- 如何設置與 MCP 服務的直接通訊
- AI 模型如何自動選擇要使用的工具
- 直接協定呼叫與 AI 輔助互動的差異

## 前置條件

開始之前，請確定你已經有：
- 安裝 Java 21 或以上版本
- 使用 Maven 來管理相依性
- 部署 Azure AI Foundry 模型（使用 `azd up` 佈署 — 詳見 [第 2 章](../../02-SetupDevEnvironment/getting-started-azure-openai.md)）
- 安裝 [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli)，並以 `az login` 登入（無需金鑰認證）
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

## 核心組件說明

### 1. 主應用程式

**檔案：** `McpServerApplication.java`

這是我們計算機服務的進入點。它是一個標準的 Spring Boot 應用程式，但有一個特別的新增：

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

**功能說明：**
- 啟動一個位於 8080 端口的 Spring Boot 網頁伺服器
- 建立一個 `ToolCallbackProvider`，使我們的計算機方法可作為 MCP 工具使用
- `@Bean` 註解讓 Spring 將此組件管理，供其他部分使用

### 2. 計算機服務

**檔案：** `CalculatorService.java`

所有數學計算在此進行。每個方法皆以 `@Tool` 註解標示，使其可以被 MCP 呼叫：

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
    
    // 更多計算機操作...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**主要特色：**

1. **`@Tool` 註解**：表示 MCP 可從外部呼叫該方法
2. <strong>明確說明</strong>：每個工具都有描述，有助 AI 模型理解何時使用
3. <strong>一致回傳格式</strong>：所有運算返回易懂的字串，如 "5.00 + 3.00 = 8.00"
4. <strong>錯誤處理</strong>：除以零及負數開根號會回傳錯誤訊息

**支援運算：**
- `add(a, b)` - 加法
- `subtract(a, b)` - 減法（第一數減第二數）
- `multiply(a, b)` - 乘法
- `divide(a, b)` - 除法（含零除錯誤檢查）
- `power(base, exponent)` - 次方運算
- `squareRoot(number)` - 平方根（含負數檢查）
- `modulus(a, b)` - 取餘數
- `absolute(number)` - 絕對值
- `help()` - 回傳所有運算說明

### 3. 直接 MCP 用戶端

**檔案：** `SDKClient.java`

此用戶端直接與 MCP 伺服器通訊，不透過 AI。它手動呼叫特定計算功能：

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

**功能說明：**
1. 以建造者模式連接位於 `http://localhost:8080` 的計算機伺服器
2. 列出所有可用工具（即計算機功能）
3. 以確切參數呼叫特定函式
4. 直接列印結果

<strong>注意：</strong>範例中使用 Spring AI 1.1.0-SNAPSHOT 版本，該版本新增了 `WebFluxSseClientTransport` 的建造者模式。如果你使用的是舊版穩定版，可能需要使用直接建構函式。

<strong>使用時機：</strong>當你清楚知道要執行的計算，且想程式化呼叫時。

### 4. AI 驅動用戶端

**檔案：** `LangChain4jClient.java`

這個用戶端使用 AI 模型（GPT-4o-mini），能自行決定使用哪些計算工具：

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // 設置 AI 模型（Azure AI Foundry，通過 Microsoft Entra ID 進行無密鑰驗證）
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

        // 給 AI 訪問我們的計算器工具的權限
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // 創建一個可以使用我們計算器的 AI 機械人
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // 現在我們可以用自然語言要求 AI 進行計算了
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**功能說明：**
1. 利用無金鑰認證（Microsoft Entra ID）建立 AI 模型連線
2. 將 AI 連接到我們的計算機 MCP 伺服器
3. 讓 AI 可存取所有計算機工具
4. 支援自然語言請求，如「計算 24.5 與 17.3 的和」

**AI 會自動：**
- 理解你想要進行加法
- 選擇 `add` 工具
- 呼叫 `add(24.5, 17.3)`
- 以自然語言格式回傳結果

## 執行範例

### 步驟 1：啟動計算機伺服器

首先，登入並設定你的 Azure AI Foundry 端點（AI 用戶端所需，鍵無需密鑰）：

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

伺服器將在 `http://localhost:8080` 上啟動。你應該看到：
```
Started McpServerApplication in X.XXX seconds
```

### 步驟 2：用直接用戶端測試

在伺服器仍運行的情況下，於新的終端機執行直接 MCP 用戶端：
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

### 步驟 3：用 AI 用戶端測試

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

你會看到 AI 自動使用工具的過程：
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### 步驟 4：關閉 MCP 伺服器

測試完成後，按 `Ctrl+C` 即可停止 AI 用戶端。MCP 伺服器會繼續執行，直到你在運行伺服器的終端機中按 `Ctrl+C` 停止它。

## 整體運作原理

以下是你問 AI 「5 + 3 等於多少？」時的完整流程：

1. <strong>你</strong> 用自然語言提出問題
2. **AI** 分析請求，判斷你要做加法
3. **AI** 呼叫 MCP 伺服器：`add(5.0, 3.0)`
4. <strong>計算機服務</strong> 執行：`5.0 + 3.0 = 8.0`
5. <strong>計算機服務</strong> 回傳：`"5.00 + 3.00 = 8.00"`
6. **AI** 收到結果並組成自然語言回答
7. <strong>你</strong> 得到： 「5 加 3 的和是 8」

## 後續步驟

更多範例，請參閱 [第 04 章：實務範例](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
本文件由 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 翻譯而成。雖然我們致力於確保準確性，但請注意，機器自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應被視為權威來源。對於重要資訊，建議進行專業人工翻譯。我們不對因使用本翻譯而產生的任何誤解或誤釋承擔責任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->