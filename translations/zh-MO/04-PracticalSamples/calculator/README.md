# MCP 計算機入門教學

## 目錄

- [你將學到甚麼](#你將學到甚麼)
- [先決條件](#先決條件)
- [了解專案結構](#了解專案結構)
- [核心元件說明](#核心元件說明)
  - [1. 主應用程式](#1-主應用程式)
  - [2. 計算服務](#2-計算服務)
  - [3. 直接 MCP 客戶端](#3-直接-mcp-客戶端)
  - [4. AI 驅動客戶端](#4-ai-驅動客戶端)
- [執行範例](#執行範例)
- [整合運作原理](#整合運作原理)
- [下一步驟](#下一步驟)

## 你將學到甚麼

本教學介紹如何使用模型上下文協議（MCP）建立計算服務。你將了解：

- 如何建立 AI 可當作工具使用的服務
- 如何設定與 MCP 服務的直接通訊
- AI 模型如何自動選擇使用的工具
- 直接協議呼叫與 AI 輔助互動的差異

## 先決條件

開始之前，請確保你已具備：
- 安裝 Java 21 或更新版本
- 使用 Maven 管理相依性
- 已在 Azure AI Foundry 部署模型（使用 `azd up` 指令部署 — 詳見[第 2 章](../../02-SetupDevEnvironment/getting-started-azure-openai.md)）
- 已安裝 [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli)，並透過 `az login` 登入（無需 API 金鑰）
- 基本 Java 和 Spring Boot 知識

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

### 1. 主應用程式

**檔案：** `McpServerApplication.java`

這是我們計算服務的入口。為標準 Spring Boot 應用，但有一個特殊額外功能：

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

**其功能為：**
- 啟動一個在 8080 埠的 Spring Boot 網頁伺服器
- 建立一個 `ToolCallbackProvider`，使我們的計算機方法能作為 MCP 工具被使用
- `@Bean` 註解讓 Spring 將此管理為可由其他元件使用的組件

### 2. 計算服務

**檔案：** `CalculatorService.java`

數學運算都在這裡執行。每個方法均用 `@Tool` 標記，讓 MCP 可以使用：

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

**主要特點：**

1. **`@Tool` 註解**：告訴 MCP 此方法可被外部客戶端呼叫
2. <strong>明確描述</strong>：每個工具有描述，幫助 AI 模型判斷使用時機
3. <strong>一致的回傳格式</strong>：所有運算皆回傳如 "5.00 + 3.00 = 8.00" 的易讀字串
4. <strong>錯誤處理</strong>：除以零和負數開根號會回傳錯誤資訊

**可用操作：**
- `add(a, b)` - 加法
- `subtract(a, b)` - 減法
- `multiply(a, b)` - 乘法
- `divide(a, b)` - 除法（含零除檢查）
- `power(base, exponent)` - 次方
- `squareRoot(number)` - 開根號（含負數檢查）
- `modulus(a, b)` - 取餘數
- `absolute(number)` - 絕對值
- `help()` - 回傳所有操作的說明

### 3. 直接 MCP 客戶端

**檔案：** `SDKClient.java`

此客戶端直接與 MCP 伺服器通訊，不使用 AI，手動呼叫特定計算函數：

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

**其功能為：**
1. 使用建構者模式連接至 `http://localhost:8080` 的計算伺服器
2. 列出所有可用工具（計算機功能）
3. 以確切參數呼叫指定函數
4. 直接列印結果

**注意：** 此範例使用 Spring AI 1.1.0-SNAPSHOT 版本，該版本引入了 `WebFluxSseClientTransport` 的建構者模式。如果你使用較舊的穩定版本，可能需要改用直接建構子。

**適用情境：** 當你確定要執行的計算並想程式化調用時。

### 4. AI 驅動客戶端

**檔案：** `LangChain4jClient.java`

此客戶端使用一個 AI 模型（GPT-4o-mini）可自動決定使用哪些計算工具：

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // 設定人工智能模型（Azure AI Foundry，透過 Microsoft Entra ID 無鑰匙驗證）
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

        // 連接到我們的計算機 MCP 伺服器
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // 顯示人工智能正在做什麼
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // 給人工智能存取我們的計算機工具
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // 建立一個能使用我們計算機的人工智能機械人
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // 現在我們可以用自然語言要求人工智能做計算
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**其功能為：**
1. 使用無須 API 金鑰的 Microsoft Entra ID 認證建立 AI 模型連線
2. 連接 AI 至我們的計算機 MCP 伺服器
3. 讓 AI 可訪問所有計算工具
4. 支援自然語言請求，如「計算 24.5 與 17.3 的和」

**AI 自動：**
- 理解你想加總兩個數字
- 選擇 `add` 工具
- 呼叫 `add(24.5, 17.3)`
- 回傳自然語言形式的結果

## 執行範例

### 步驟 1：啟動計算機伺服器

先登入並設置你的 Azure AI Foundry 端點（AI 客戶端需要，無 API 金鑰驗證）：

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

伺服器會在 `http://localhost:8080` 啟動，畫面應顯示：
```
Started McpServerApplication in X.XXX seconds
```

### 步驟 2：用直接客戶端測試

在伺服器持續運行的<strong>新視窗</strong>終端機中，執行直接 MCP 客戶端：
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

### 步驟 3：用 AI 客戶端測試

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

你會看到 AI 自動使用工具：
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### 步驟 4：關閉 MCP 伺服器

測試完成後，可以按客戶端視窗中的 `Ctrl+C` 停止 AI 客戶端。MCP 伺服器會持續運行直到你在伺服器視窗按下 `Ctrl+C` 才停止。

## 整合運作原理

當你向 AI 詢問「5 + 3 是多少？」時，完整流程如下：

1. <strong>你</strong> 使用自然語言請求 AI
2. **AI** 解析並判斷你要求執行加法
3. **AI** 呼叫 MCP 伺服器執行：`add(5.0, 3.0)`
4. <strong>計算服務</strong> 執行運算：`5.0 + 3.0 = 8.0`
5. <strong>計算服務</strong> 回傳結果字串：`"5.00 + 3.00 = 8.00"`
6. **AI** 接收結果並組裝自然語言回應
7. <strong>你</strong> 得到回覆：「5 與 3 的總和是 8」

## 下一步驟

如需更多範例，請參閱[第 04 章：實務範例](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
本文件使用 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 進行翻譯。雖然我們力求準確，但請注意，自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應被視為權威來源。對於重要資訊，建議尋求專業人工翻譯。我們不對因使用本翻譯而引起的任何誤解或曲解承擔責任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->