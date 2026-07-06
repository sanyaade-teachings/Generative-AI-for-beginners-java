# 初心者向けMCP計算機チュートリアル

## 目次

- [学べること](#学べること)
- [前提条件](#前提条件)
- [プロジェクト構成の理解](#プロジェクト構成の理解)
- [コアコンポーネントの説明](#コアコンポーネントの説明)
  - [1. メインアプリケーション](#1-メインアプリケーション)
  - [2. 計算サービス](#2-計算サービス)
  - [3. 直接MCPクライアント](#3-直接mcpクライアント)
  - [4. AI搭載クライアント](#4-ai搭載クライアント)
- [例の実行](#例の実行)
- [全体の動作概要](#全体の動作概要)
- [次のステップ](#次のステップ)

## 学べること

このチュートリアルでは、Model Context Protocol (MCP) を使って計算サービスを構築する方法を説明します。以下のことが理解できます：

- AIがツールとして使えるサービスの作り方
- MCPサービスとの直接通信の設定方法
- AIモデルが自動的にツールを選択する方法
- 直接プロトコル呼び出しとAI支援型インタラクションの違い

## 前提条件

始める前に、以下を用意してください：
- Java 21以上がインストールされていること
- 依存関係管理のためのMaven
- Azure AI Foundryモデルのデプロイ（`azd up`でプロビジョニング—詳しくは[第2章](../../02-SetupDevEnvironment/getting-started-azure-openai.md)参照）
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli)をインストールし、`az login`でサインイン済み（キー不要の認証）
- JavaとSpring Bootの基本知識

## プロジェクト構成の理解

計算機プロジェクトには、いくつか重要なファイルがあります：

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

## コアコンポーネントの説明

### 1. メインアプリケーション

**ファイル:** `McpServerApplication.java`

これは計算機サービスのエントリーポイントです。標準的なSpring Bootアプリケーションに特別な追加があります：

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

**これがすること：**
- ポート8080でSpring BootのWebサーバーを起動
- 計算機のメソッドをMCPツールとして使えるようにする`ToolCallbackProvider`を作成
- `@Bean`注釈で、Springが他の部分で利用できるコンポーネントとして管理

### 2. 計算サービス

**ファイル:** `CalculatorService.java`

ここで全ての計算が行われます。各メソッドには`@Tool`が付けられており、MCP経由で呼び出せます：

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
    
    // もっと計算機の操作...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**主な特徴：**

1. **`@Tool`注釈**：このメソッドが外部クライアントから呼び出せることをMCPに伝える
2. <strong>わかりやすい説明</strong>：AIモデルがいつ使うか理解しやすい説明付き
3. <strong>一貫した戻り値形式</strong>：全て「5.00 + 3.00 = 8.00」のような人間が読める文字列を返す
4. <strong>エラーハンドリング</strong>：ゼロ除算や負の平方根はエラーメッセージを返す

**利用可能な操作一覧：**
- `add(a, b)` - 2つの数字を加算
- `subtract(a, b)` - 2番目を1番目から減算
- `multiply(a, b)` - 2つの数字を乗算
- `divide(a, b)` - 1番目を2番目で除算（ゼロチェックあり）
- `power(base, exponent)` - baseのexponent乗を計算
- `squareRoot(number)` - 平方根を計算（負数チェックあり）
- `modulus(a, b)` - 剰余を返す
- `absolute(number)` - 絶対値を返す
- `help()` - 全操作の情報を返す

### 3. 直接MCPクライアント

**ファイル:** `SDKClient.java`

このクライアントはAIを使わず、MCPサーバーに直接通信して計算機機能を手動で呼び出します：

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
        
        // 利用可能なツールをリストする
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // 特定の計算機能を呼び出す
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

**これがすること：**
1. `http://localhost:8080`にビルダーパターンで計算機サーバーに接続
2. 利用可能なツール（計算機関数）の一覧を取得
3. 正確なパラメーターで特定の関数を呼び出し
4. 直接結果を出力

**注記：** この例はSpring AI 1.1.0-SNAPSHOT依存関係を使っており、`WebFluxSseClientTransport`にビルダーパターンを導入しています。安定版の古いバージョンを使っている場合は直接コンストラクターを利用する必要があります。

**使うべき場面：** 実行したい計算が明確で、プログラム的に呼び出したい場合。

### 4. AI搭載クライアント

**ファイル:** `LangChain4jClient.java`

このクライアントはAIモデル（GPT-4o-mini）を使い、どの計算ツールを使うか自動で判断します：

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // AIモデルを設定します（Azure AI Foundry、Microsoft Entra IDを使用したキー不要認証）
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

        // 私たちの計算機MCPサーバーに接続します
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // AIが何をしているかを表示します
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // AIに私たちの計算ツールへのアクセスを許可します
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // 私たちの計算機を使えるAIボットを作成します
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // これでAIに自然言語で計算を依頼できます
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**これがすること：**
1. 鍵不要認証（Microsoft Entra ID）でAIモデル接続を作成
2. AIと計算機MCPサーバーを接続
3. AIに全ての計算機ツールへのアクセス権を与える
4. 「24.5と17.3の合計を計算して」といった自然言語リクエストを可能にする

**AIは自動的に：**
- 数字の合計を求めたいことを理解
- `add`ツールを選択
- `add(24.5, 17.3)`を呼び出し
- 自然な応答で結果を返す

## 例の実行

### ステップ1：計算機サーバー起動

まずサインインし、Azure AI Foundryエンドポイントを設定します（AIクライアント用・キー不要APIキーなし）：

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

サーバーを起動：
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

サーバーは `http://localhost:8080` で起動します。以下が表示されるはずです：
```
Started McpServerApplication in X.XXX seconds
```

### ステップ2：直接クライアントでテスト

サーバーを起動したまま<strong>新しい</strong>ターミナルで直接MCPクライアントを実行：
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

以下のような出力が表示されます：
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### ステップ3：AIクライアントでテスト

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

AIが自動でツールを使う様子が見られます：
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### ステップ4：MCPサーバーを終了

テストが終わったら、AIクライアントのターミナルで`Ctrl+C`を押して終了できます。MCPサーバーは停止するまで動作し続けます。
サーバーを停止するには、サーバーを起動したターミナルで`Ctrl+C`を押します。

## 全体の動作概要

AIに「5 + 3は？」と尋ねた時の全体の流れは：

1. <strong>あなた</strong>が自然言語でAIに質問
2. <strong>AI</strong>がリクエストを理解し、加算が必要と判断
3. <strong>AI</strong>がMCPサーバーへ `add(5.0, 3.0)` と呼び出し
4. <strong>計算サービス</strong>が計算を実行： `5.0 + 3.0 = 8.0`
5. <strong>計算サービス</strong>が `"5.00 + 3.00 = 8.00"` を返す
6. <strong>AI</strong>が結果を受け取り自然な応答を生成
7. <strong>あなた</strong>に「5と3の合計は8です」と返答

## 次のステップ

さらなる例については [第04章：実用サンプル](../README.md) をご覧ください。

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責事項**：
本書類は AI 翻訳サービス [Co-op Translator](https://github.com/Azure/co-op-translator) を使用して翻訳されています。正確性を期していますが、自動翻訳には誤りや不正確な部分が含まれる可能性があることをご承知おきください。原文の原語版が正式な情報源とみなされるべきです。重要な情報については、専門の人間による翻訳を推奨します。本翻訳の利用により生じたいかなる誤解や解釈違いについても、当方は責任を負いかねます。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->