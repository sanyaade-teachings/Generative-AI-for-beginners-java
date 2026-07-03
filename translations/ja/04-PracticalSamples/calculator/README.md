# 初心者向けMCP電卓チュートリアル

## 目次

- [学べること](#学べること)
- [前提条件](#前提条件)
- [プロジェクト構成の理解](#プロジェクト構成の理解)
- [主要コンポーネントの解説](#主要コンポーネントの解説)
  - [1. メインアプリケーション](#1-メインアプリケーション)
  - [2. 電卓サービス](#2-電卓サービス)
  - [3. 直接MCPクライアント](#3-直接mcpクライアント)
  - [4. AI搭載クライアント](#4-ai搭載クライアント)
- [例の実行方法](#例の実行方法)
- [全体の連携動作](#全体の連携動作)
- [次のステップ](#次のステップ)

## 学べること

このチュートリアルでは、Model Context Protocol (MCP) を使って電卓サービスを構築する方法を説明します。以下のことが理解できます：

- AIがツールとして使用できるサービスの作り方
- MCPサービスと直接通信を設定する方法
- AIモデルが自動的に使うツールを選ぶ方法
- 直接プロトコル呼び出しとAI支援インタラクションの違い

## 前提条件

始める前に以下を用意してください：
- Java 21以上のインストール
- 依存関係管理用のMaven
- Azure AI Foundryモデルのデプロイ（`azd up`コマンドでプロビジョニングしてください — [第2章](../../02-SetupDevEnvironment/getting-started-azure-openai.md)参照）
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) をサインイン済み（`az login`、キー不要認証）
- Java と Spring Boot の基本知識

## プロジェクト構成の理解

電卓プロジェクトにはいくつか重要なファイルがあります：

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

## 主要コンポーネントの解説

### 1. メインアプリケーション

**ファイル:** `McpServerApplication.java`

これは電卓サービスのエントリーポイントです。標準のSpring Bootアプリケーションで、ただし一つ特別な追加があります：

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

**これがすること:**
- ポート8080でSpring BootのWebサーバーを起動する
- `ToolCallbackProvider`を作成して電卓のメソッドをMCPツールとして利用可能にする
- `@Bean`アノテーションでSpringがこれを管理コンポーネントとして扱い他部分で利用できるようにする

### 2. 電卓サービス

**ファイル:** `CalculatorService.java`

ここで全ての計算処理が行われています。メソッドはすべて`@Tool`でマークされ、MCP経由で使えるようになっています：

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
    
    // さらに計算機の操作...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**主な特徴:**

1. **`@Tool` アノテーション**: これによってMCPが外部クライアントから呼び出せることを認識
2. <strong>分かりやすい説明文</strong>: 各ツールにはAIモデルがいつ使うか理解しやすい説明が付いている
3. <strong>一貫した返却フォーマット</strong>: すべての操作は「5.00 + 3.00 = 8.00」のような人が読める文字列で返す
4. <strong>エラーハンドリング</strong>: ゼロ除算や負の平方根はエラーメッセージを返す

**利用可能な操作:**
- `add(a, b)` - 2つの数値を加算
- `subtract(a, b)` - 2つの数値を減算
- `multiply(a, b)` - 2つの数値を乗算
- `divide(a, b)` - 1つ目を2つ目で除算（ゼロチェック有）
- `power(base, exponent)` - べき乗計算
- `squareRoot(number)` - 平方根計算（負数チェック有）
- `modulus(a, b)` - 剰余算
- `absolute(number)` - 絶対値
- `help()` - 操作一覧の説明を返す

### 3. 直接MCPクライアント

**ファイル:** `SDKClient.java`

このクライアントはAIを使わずにMCPサーバーに直接アクセスします。手動で特定の電卓関数を呼び出します：

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
        
        // 利用可能なツールを一覧表示する
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

**これがすること:**
1. ビルダーパターンで`http://localhost:8080`の電卓サーバーに接続する
2. 利用可能なすべてのツール（電卓機能）を一覧表示
3. 特定の関数を正確なパラメータで呼び出す
4. 結果を直接出力する

**注意:** この例はSpring AI 1.1.0-SNAPSHOTの依存関係を使っており、`WebFluxSseClientTransport`にビルダーパターンが導入されています。古い安定版を使う場合は直接コンストラクタ呼び出しが必要な場合があります。

**使うタイミング:** 実行したい計算が明確に決まっている時にプログラムから呼び出したい場合。

### 4. AI搭載クライアント

**ファイル:** `LangChain4jClient.java`

このクライアントはAIモデル（GPT-4o-mini）を使って、どの電卓ツールを使うかを自動的に判断します：

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // AIモデルを設定する（Azure AI Foundry、Microsoft Entra IDによるキー不要認証）
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

        // 当社の計算機MCPサーバーに接続する
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // AIが何をしているかを表示する
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // AIに当社の計算機ツールへのアクセス権を与える
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // 当社の計算機を使えるAIボットを作成する
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // これでAIに自然言語で計算を依頼できるようになる
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**これがすること:**
1. GitHubトークンでAIモデル接続を作成
2. AIを電卓MCPサーバーに接続
3. AIにすべての電卓ツールへのアクセス権を付与
4. 「24.5と17.3の合計を計算して」のような自然言語リクエストに対応可能にする

**AIは自動で:**
- 数値の加算を理解
- `add` ツールを選択
- `add(24.5, 17.3)` を呼び出し
- 結果を自然な返答として返す

## 例の実行方法

### ステップ 1: 電卓サーバーを起動する

まずサインインし、Azure AI Foundryのエンドポイントを設定します（AIクライアント用 - キー不要認証）：

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

サーバー起動コマンド：
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

サーバーは `http://localhost:8080` で起動します。以下のような表示が出ます：
```
Started McpServerApplication in X.XXX seconds
```

### ステップ 2: 直接クライアントでテスト

サーバーを起動したまま<strong>新しい</strong>ターミナルで直接MCPクライアントを実行：
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

次のような出力が表示されます：
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### ステップ 3: AIクライアントでテスト

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

AIが自動的にツールを使う様子が見られます：
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### ステップ 4: MCPサーバーを閉じる

テストが終わったら、AIクライアントのターミナルで `Ctrl+C` を押して終了できます。MCPサーバーは停止するまで動作を続けます。
サーバーを停止する場合は、それが起動しているターミナルで `Ctrl+C` を押してください。

## 全体の連携動作

AIに「5 + 3は？」と聞いた時の全体の流れは以下の通りです：

1. <strong>あなた</strong>が自然言語でAIに質問
2. <strong>AI</strong>が解析し、加算をしたいと判断
3. <strong>AI</strong>がMCPサーバーに呼び出しを実行：`add(5.0, 3.0)`
4. <strong>電卓サービス</strong>が計算を実行：`5.0 + 3.0 = 8.0`
5. <strong>電卓サービス</strong>が `"5.00 + 3.00 = 8.00"` を返す
6. <strong>AI</strong>が結果を受け取り自然な応答文を生成
7. <strong>あなた</strong>に「5と3の合計は8です」と返答

## 次のステップ

さらに多くの例を見たい方は、[第04章：実践サンプル](../README.md) をご覧ください。

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責事項**：
本書類は AI 翻訳サービス [Co-op Translator](https://github.com/Azure/co-op-translator) を使用して翻訳されています。正確性を期していますが、自動翻訳には誤りや不正確な部分が含まれる可能性があることをご承知おきください。原文の原語版が正式な情報源とみなされるべきです。重要な情報については、専門の人間による翻訳を推奨します。本翻訳の利用により生じたいかなる誤解や解釈違いについても、当方は責任を負いかねます。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->