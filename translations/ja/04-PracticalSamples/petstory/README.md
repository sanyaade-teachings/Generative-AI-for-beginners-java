# Pet Story Generator チュートリアル（初心者向け）

## 目次

- [前提条件](#前提条件)
- [プロジェクト構造の理解](#プロジェクト構造の理解)
- [コアコンポーネントの説明](#コアコンポーネントの説明)
  - [1. メインアプリケーション](#1-メインアプリケーション)
  - [2. Webコントローラ](#2-webコントローラ)
  - [3. ストーリーサービス](#3-ストーリーサービス)
  - [4. Webテンプレート](#4-webテンプレート)
  - [5. 設定](#5-設定)
- [アプリケーションの実行](#アプリケーションの実行)
- [すべてが連携する仕組み](#すべてが連携する仕組み)
- [AI連携の理解](#ai連携の理解)
- [次のステップ](#次のステップ)

## 前提条件

開始する前に、以下を確認してください：
- Java 21 以上がインストールされていること
- 依存関係管理のための Maven
- Azure AI Foundry モデルのデプロイ（`azd up` でプロビジョニング — [第2章](../../02-SetupDevEnvironment/getting-started-azure-openai.md)参照）、`az login` でサインイン済み（キー不要の認証）
- Java、Spring Boot、およびウェブ開発の基本知識

## プロジェクト構造の理解

ペットストーリープロジェクトにはいくつかの重要なファイルがあります：

```
petstory/
├── src/main/java/com/example/petstory/
│   ├── PetStoryApplication.java       # Main Spring Boot application
│   ├── PetController.java             # Web request handler
│   ├── StoryService.java              # AI story generation service
│   └── SecurityConfig.java            # Security configuration
├── src/main/resources/
│   ├── application.properties         # App configuration
│   └── templates/
│       ├── index.html                 # Upload form page
│       └── result.html               # Story display page
└── pom.xml                           # Maven dependencies
```

## コアコンポーネントの説明

### 1. メインアプリケーション

**ファイル:** `PetStoryApplication.java`

これは Spring Boot アプリケーションのエントリポイントです：

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**概要:**
- `@SpringBootApplication` アノテーションは自動構成とコンポーネントスキャンを有効にします
- ポート8080で組み込みのウェブサーバ（Tomcat）を起動します
- 必要なSpringのBeanとサービスを自動で作成します

### 2. Webコントローラ

**ファイル:** `PetController.java`

これはすべてのウェブリクエストとユーザーインタラクションを処理します：

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // index.htmlテンプレートを返す
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // 入力の検証
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // セキュリティのために入力をサニタイズする
        String sanitizedDescription = sanitizeInput(description);
        
        // エラーハンドリング付きでストーリーを生成する
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // result.htmlテンプレートを返す
            
        } catch (Exception e) {
            // AIが失敗した場合にフォールバックのストーリーを使用する
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // 長さを制限する
    }
}
```

**主な機能：**

1. <strong>ルート処理</strong>: `@GetMapping("/")` はアップロードフォームを表示し、`@PostMapping("/generate-story")` は送信を処理します
2. <strong>入力検証</strong>: 空の説明や文字数制限をチェックします
3. <strong>セキュリティ</strong>: ユーザー入力をサニタイズしてXSS攻撃を防ぎます
4. <strong>エラーハンドリング</strong>: AIサービスが失敗した場合は代替ストーリーを提供します
5. <strong>モデルバインディング</strong>: Springの `Model` を使ってデータをHTMLテンプレートに渡します

**フォールバックシステム:**
コントローラにはAIサービスが利用できない場合に使用する事前作成済みのストーリーテンプレートがあります：

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // 一貫した応答のために説明ハッシュを使用する
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. ストーリーサービス

**ファイル:** `StoryService.java`

このサービスは Azure AI Foundry と通信し、キー不要の認証を使ってストーリーを生成します：

```java
@Service
public class StoryService {
    
    private final OpenAIClient openAIClient;
    private final String modelName;
    
    public StoryService(@Value("${azure.openai.endpoint:}") String endpoint,
                       @Value("${azure.openai.deployment:gpt-4o-mini}") String modelName) {
        this.modelName = modelName;
        if (endpoint == null || endpoint.isBlank()) {
            endpoint = System.getenv("AZURE_OPENAI_ENDPOINT");
        }
        
        // FoundryのOpenAI互換エンドポイントは/openai/v1/にあります
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Microsoft Entra IDによるキーレス認証（APIキー不要）
        DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
        this.openAIClient = OpenAIOkHttpClient.builder()
                .baseUrl(baseUrl)
                .credential(BearerTokenCredential.create(
                        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
                .build();
    }
    
    public String generateStory(String description) {
        String systemPrompt = "You are a creative storyteller who writes fun, " +
                             "family-friendly short stories about pets. " +
                             "Keep stories under 500 words and appropriate for all ages.";
        
        String userPrompt = "Write a fun short story about a pet described as: " + description;
        
        // AIリクエストを設定する
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // 応答の長さを制限する
                .temperature(0.8)          // 創造性を制御する（0.0〜1.0）
                .build();
        
        // リクエストを送信して応答を取得する
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**主な構成要素:**

1. **OpenAI クライアント**: Azure AI Foundry 用に設定された公式OpenAI Java SDK（キー不要）
2. <strong>システムプロンプト</strong>: AIに家族向けのペットストーリーを書くよう指示
3. <strong>ユーザープロンプト</strong>: 説明に基づきAIにどのようなストーリーを書くか正確に指示
4. <strong>パラメータ</strong>: ストーリーの長さと創造性のレベルを制御
5. <strong>エラーハンドリング</strong>: コントローラが捕捉可能な例外を投げます

### 4. Webテンプレート

**ファイル:** `index.html`（アップロードフォーム）

ユーザーがペットを説明するメインページです：

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Pet Story Generator</title>
    <!-- CSS styling -->
</head>
<body>
    <div class="container">
        <h1>Pet Story Generator</h1>
        <p>Describe your pet and we'll create a fun story about them!</p>
        
        <!-- Error message display -->
        <div th:if="${error}" class="error" th:text="${error}"></div>
        
        <!-- Story generation form -->
        <form action="/generate-story" method="post">
            <div class="form-group">
                <label for="description">Describe your pet:</label>
                <textarea id="description" name="description" 
                         placeholder="Tell us about your pet - what they look like, their personality, favorite activities..."
                         maxlength="1000" required></textarea>
            </div>
            <button type="submit" class="btn btn-primary">Generate Story</button>
        </form>
        
        <!-- Image upload section with client-side processing -->
        <div class="upload-section">
            <h2>Or Upload a Photo</h2>
            <input type="file" id="imageInput" accept="image/*" />
            <button onclick="analyzeImage()" class="upload-btn">Analyze Image</button>
        </div>
        
        <script>
            // Client-side image analysis using Transformers.js
            async function analyzeImage() {
                // Image processing code here
                // Generates description automatically from uploaded image
            }
        </script>
    </div>
</body>
</html>
```

**ファイル:** `result.html`（ストーリー表示）

生成されたストーリーを表示します：

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Pet Story Result</title>
</head>
<body>
    <div class="container">
        <h1>Your Pet's Story</h1>
        
        <div class="result-section">
            <div class="result-label">Pet Description:</div>
            <div class="result-content" th:text="${caption}"></div>
        </div>
        
        <div class="result-section">
            <div class="result-label">Generated Story:</div>
            <div class="result-content" th:text="${story}"></div>
        </div>
        
        <div class="result-section" th:if="${analysisType}">
            <div class="result-label">Analysis Type:</div>
            <div class="result-content" th:text="${analysisType}"></div>
        </div>
        
        <a href="/" class="back-link">Generate Another Story</a>
    </div>
</body>
</html>
```

**テンプレートの特徴：**

1. **Thymeleaf連携**: 動的コンテンツ用の `th:` 属性を使用
2. <strong>レスポンシブデザイン</strong>: モバイルとデスクトップ向けのCSSスタイリング
3. <strong>エラーハンドリング</strong>: 検証エラーをユーザーに表示
4. <strong>クライアントサイド処理</strong>: 画像解析用JavaScript（Transformers.jsを使用）

### 5. 設定

**ファイル:** `application.properties`

アプリケーションの設定内容：

```properties
spring.application.name=pet-story-app

# File upload limits
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=10MB

# Logging configuration
logging.level.com.example.petstory=INFO

# Azure AI Foundry (keyless) configuration
azure.openai.endpoint=${AZURE_OPENAI_ENDPOINT:}
azure.openai.deployment=${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}
```

**設定の説明：**

1. <strong>ファイルアップロード</strong>: 最大10MBの画像を許可
2. <strong>ログ出力</strong>: 実行中のログ情報を制御
3. **Azure AI Foundry**: 使用するエンドポイントとモデルデプロイを指定（キー不要認証）
4. <strong>セキュリティ</strong>: 敏感情報を公開しないエラーハンドリング設定

## アプリケーションの実行

### 手順1: サインインしてエンドポイントを設定

認証はキー不要のMicrosoft Entra IDを使いますので、APIキー不要です。サインインしてFoundryエンドポイントを設定します：

**Windows（コマンドプロンプト）：**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**Windows（PowerShell）：**
```powershell
az login
$env:AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
```

**Linux/macOS：**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**必要な理由：**
- Azure AI Foundryは推論リクエスト認証にMicrosoft Entra IDを使用します
- キー不要認証により、ソースコードや環境変数に秘密情報を含める必要がありません
- アカウントは対象リソースに対して **Cognitive Services OpenAI User** ロールが必要です

### 手順2: ビルドと実行

プロジェクトディレクトリに移動：
```bash
cd 04-PracticalSamples/petstory
```

アプリケーションをビルド：
```bash
mvn clean compile
```

サーバーを起動：
```bash
mvn spring-boot:run
```

アプリケーションは `http://localhost:8080` で起動します。

### 手順3: アプリケーションをテスト

1. ブラウザで `http://localhost:8080` を開く
2. テキストエリアにペットを説明（例：「遊び好きのゴールデンレトリバーでボール遊びが大好き」）
3. 「ストーリーを生成」ボタンをクリックしてAI生成のストーリーを取得
4. あるいはペットの画像をアップロードして自動で説明を生成
5. ペットの説明に基づくクリエイティブなストーリーを閲覧

## すべてが連携する仕組み

ペットストーリー生成の全体の流れは以下の通りです：

1. <strong>ユーザー入力</strong>：ウェブフォームでペットの説明を入力
2. <strong>フォーム送信</strong>：ブラウザが `/generate-story` にPOSTリクエスト送信
3. <strong>コントローラ処理</strong>：`PetController` が入力を検証しサニタイズ
4. **AIサービス呼び出し**：`StoryService` がAzure AI Foundryモデルにリクエスト送信
5. <strong>ストーリー生成</strong>：AIが説明に基づいてクリエイティブなストーリーを生成
6. <strong>レスポンス処理</strong>：コントローラがストーリーを受け取りモデルに追加
7. <strong>テンプレート描画</strong>：Thymeleafが `result.html` をストーリー付きでレンダリング
8. <strong>表示</strong>：ユーザーのブラウザに生成ストーリーを表示

**エラーハンドリングの流れ：**  
AIサービスが失敗した場合：
1. コントローラが例外を捕捉
2. 事前用意したテンプレートから代替ストーリーを生成
3. AI非利用状態の説明付きで代替ストーリーを表示
4. ユーザーには常にストーリーが提供され、良いユーザー体験を確保

## AI連携の理解

### Azure AI Foundry（キー不要認証）  
アプリは Azure AI Foundry をキー不要認証（Microsoft Entra ID）で使用しています：

```java
// キーレス認証 - APIキー不要
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### プロンプトエンジニアリング  
優れた結果を得るために慎重に設計されたプロンプトを使用：

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### 応答処理  
AIの応答を抽出し検証：

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## 次のステップ

さらなる例については [第04章: 実践サンプル](../README.md) をご覧ください。

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責事項**：
本書類は AI 翻訳サービス [Co-op Translator](https://github.com/Azure/co-op-translator) を使用して翻訳されています。正確性を期していますが、自動翻訳には誤りや不正確な部分が含まれる可能性があることをご承知おきください。原文の原語版が正式な情報源とみなされるべきです。重要な情報については、専門の人間による翻訳を推奨します。本翻訳の利用により生じたいかなる誤解や解釈違いについても、当方は責任を負いかねます。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->