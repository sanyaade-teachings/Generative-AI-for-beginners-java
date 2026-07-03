# AGENTS.md

## プロジェクト概要

これはJavaでの生成AI開発を学ぶための教育用リポジトリです。大規模言語モデル（LLM）、プロンプトエンジニアリング、埋め込み、RAG（Retrieval-Augmented Generation）、およびモデルコンテキストプロトコル（MCP）をカバーする包括的な実践コースを提供します。

**主要技術:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry、Azure OpenAI、およびOpenAI SDK

**アーキテクチャ:**
- 章ごとに整理された複数の独立したSpring Bootアプリケーション
- 様々なAIパターンを示すサンプルプロジェクト
- 事前構成済みの開発コンテナを備えたGitHub Codespaces対応

## セットアップコマンド

### 前提条件
- Java 21以上
- Maven 3.x
- Azure AI FoundryモデルのデプロイがあるAzureサブスクリプション（`azd up`でプロビジョニング）
- サインイン済みのAzure CLI（`az`）とAzure Developer CLI（`azd`）、キー不要認証用

### 環境セットアップ

**オプション1: GitHub Codespaces（推奨）**
```bash
# リポジトリをフォークしてGitHub UIからコードスペースを作成します
# 開発コンテナがすべての依存関係を自動的にインストールします
# 環境設定に約2分待ちます
```

**オプション2: ローカル開発コンテナ**
```bash
# リポジトリをクローンする
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Dev Containers拡張機能を使用してVS Codeで開く
# プロンプトが表示されたらコンテナ内で再度開く
```

**オプション3: ローカルセットアップ**
```bash
# 依存関係をインストールする
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# インストールを確認する
java -version
mvn -version
```

### 設定

**Azure AI Foundryセットアップ（キー不要、推奨）：**
```bash
# Foundryアカウントとモデルデプロイメントをコードとしてプロビジョニングする
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azdはエンドポイント（キーなし）を使ってexamples/basic-chat-azure/.envを書き込みます
```

**手動エンドポイント設定：**
```bash
# azd を使用しなかった場合は、エンドポイントを自分で設定してください（認証は az login を介してキー不要のままです）
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# .env を編集してください: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## 開発ワークフロー

### プロジェクト構成
```
/
├── 01-IntroToGenAI/              # Chapter 1: Introduction
├── 02-SetupDevEnvironment/       # Chapter 2: Environment setup
│   └── examples/                 # Working examples
├── 03-CoreGenerativeAITechniques/ # Chapter 3: Core techniques
├── 04-PracticalSamples/          # Chapter 4: Sample projects
│   ├── calculator/               # MCP service example
│   ├── foundrylocal/            # Local model integration
│   └── petstory/                # Multi-modal app
├── 05-ResponsibleGenAI/         # Chapter 5: Responsible AI
└── translations/                # Multi-language support
```

### アプリケーションの実行

**Spring Bootアプリケーションの実行:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**プロジェクトのビルド:**
```bash
cd [project-directory]
mvn clean install
```

**MCP計算サーバーの起動:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# サーバーは http://localhost:8080 で実行されます
```

**クライアント例の実行:**
```bash
# 別のターミナルでサーバーを起動した後
cd 04-PracticalSamples/calculator

# 直接MCPクライアント
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# AI搭載クライアント（AZURE_OPENAI_ENDPOINT + az loginが必要）
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# インタラクティブなボット
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### ホットリロード
Spring Boot DevToolsはホットリロード対応プロジェクトに含まれています:
```bash
# Javaファイルへの変更は保存時に自動的にリロードされます
mvn spring-boot:run
```

## テスト手順

### テストの実行

**プロジェクト内のすべてのテストを実行:**
```bash
cd [project-directory]
mvn test
```

**詳細出力でテストを実行:**
```bash
mvn test -X
```

**特定のテストクラスを実行:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### テスト構成
- テストファイルはJUnit 5（Jupiter）を使用
- テストクラスは `src/test/java/` に配置
- 計算機プロジェクトのクライアント例は `src/test/java/com/microsoft/mcp/sample/client/` に配置

### 手動テスト
多くの例は対話型アプリケーションで手動テストが必要です:

1. `mvn spring-boot:run` でアプリケーションを起動
2. エンドポイントのテストやCLIとの対話
3. 各プロジェクトのREADME.mdのドキュメントに記載された期待動作を確認

### Azure AI Foundryを使ったテスト
- 例を実行する前に `az login` でサインイン（キー不要認証用）
- アカウントにリソースのCognitive Services OpenAIユーザー役割があることを確認
- 第5章の責任あるAI例でコンテンツフィルタリングをテスト

## コードスタイルガイドライン

### Javaの規約
- **Javaバージョン:** Java 21の最新機能を利用
- **スタイル:** 標準のJava規約に従う
- **命名規則:** 
  - クラス: PascalCase
  - メソッド/変数: camelCase
  - 定数: UPPER_SNAKE_CASE
  - パッケージ名: 小文字

### Spring Bootパターン
- ビジネスロジックには`@Service`を使用
- RESTエンドポイントには`@RestController`を使用
- 設定は`application.yml`または`application.properties`で管理
- ハードコーディングより環境変数を推奨
- MCPで公開するメソッドには`@Tool`アノテーションを使用

### ファイル構成
```
src/
├── main/
│   ├── java/
│   │   └── com/microsoft/[component]/
│   │       ├── [Component]Application.java
│   │       ├── config/
│   │       ├── controller/
│   │       ├── service/
│   │       └── exception/
│   └── resources/
│       ├── application.yml
│       └── static/
└── test/
    └── java/
        └── com/microsoft/[component]/
```

### 依存関係
- Mavenの`pom.xml`で管理
- バージョン管理にSpring AI BOMを使用
- AI統合にLangChain4jを使用
- Spring依存にSpring Boot starter parentを利用

### コードコメント
- 公開APIにはJavaDocを追加
- 複雑なAI連携には説明コメントを記載
- MCPツールの説明は明確にドキュメント化

## ビルドとデプロイ

### プロジェクトのビルド

**テストなしでビルド:**
```bash
mvn clean install -DskipTests
```

**すべてのチェックを含めてビルド:**
```bash
mvn clean install
```

**アプリケーションのパッケージング:**
```bash
mvn package
# target/ ディレクトリにJARを作成します
```

### 出力ディレクトリ
- コンパイル済みクラス: `target/classes/`
- テストクラス: `target/test-classes/`
- JARファイル: `target/*.jar`
- Mavenアーティファクト: `target/`

### 環境別設定

**開発環境:**
```yaml
# application.yml (keyless - no api-key; auth via DefaultAzureCredential)
spring:
  ai:
    azure:
      openai:
        endpoint: ${AZURE_OPENAI_ENDPOINT}
        chat:
          options:
            deployment-name: ${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}
```

**本番環境:**
- キー不要認証に`az login`ではなくマネージドIDを使用
- `AZURE_OPENAI_ENDPOINT`は本番のFoundryリソースを指す
- 環境変数やAzure Key Vaultで設定を管理

### デプロイの注意点
- これは教育用リポジトリでサンプルアプリケーション
- そのまま本番用に使うことは想定されていない
- サンプルは本番利用に適用可能なパターンを示す
- 各プロジェクトREADMEに特定のデプロイ注記あり

## 追加説明

### Azure AI Foundry
- **キー不要認証:** Microsoft Entra IDで接続、APIキー管理不要
- **コードでのプロビジョニング:** Bicep + azd（`azd up`）でアカウントとモデルデプロイを作成
- 同じOpenAI互換コードがローカル（`az login`）とAzure（マネージドID）で動作

### 複数プロジェクトの扱い
各サンプルプロジェクトは独立しています:
```bash
# 特定のプロジェクトに移動する
cd 04-PracticalSamples/[project-name]

# それぞれ独自のpom.xmlを持ち、独立してビルドできます
mvn clean install
```

### よくある問題

**Javaバージョンの不一致:**
```bash
# Java 21を確認してください
java -version
# 必要に応じてJAVA_HOMEを更新してください
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**依存関係ダウンロードの問題:**
```bash
# Mavenのキャッシュをクリアして再試行してください
rm -rf ~/.m2/repository
mvn clean install
```

**エンドポイントやサインインが見つからない:**
```bash
# 現在のセッションでエンドポイントを設定し、サインインします（キー不要）
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# またはプロジェクトディレクトリ内の.envファイルを使用してください
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**ポートがすでに使用中:**
```bash
# Spring Bootはデフォルトでポート8080を使用します
# application.propertiesの変更:
server.port=8081
```

### 多言語対応
- 45以上の言語で自動翻訳されたドキュメントあり
- 翻訳は`translations/`ディレクトリに格納
- GitHub Actionsワークフローで翻訳を管理

### 学習の流れ
1. [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md) から開始
2. 章を順に進める（01 → 05）
3. 各章のハンズオンを完了
4. 第4章のサンプルプロジェクトを探検
5. 第5章で責任あるAIを学習

### 開発コンテナ
`.devcontainer/devcontainer.json`では以下を設定:
- Java 21開発環境
- Maven事前インストール
- VS Code Java拡張機能
- Spring Bootツール
- GitHub Copilot統合
- Docker-in-Docker対応
- Azure CLI

### パフォーマンス考慮
- Azure AI Foundryのデプロイには分単位のトークン/リクエスト制限あり
- 埋め込みは適切なバッチサイズで使用
- 繰り返しAPI呼び出しのキャッシュを検討
- コスト最適化のためにトークン使用状況を監視

### セキュリティ注意点
- `.env`ファイルは決してコミットしない（すでに`.gitignore`に登録済み）
- APIキーよりもキー不要認証（Microsoft Entra ID）を推奨
- AzureではマネージドIDを使用、ローカルは`az login`で開発
- 第5章の責任あるAIガイドラインに従うこと

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責事項**：
本書類は AI 翻訳サービス [Co-op Translator](https://github.com/Azure/co-op-translator) を使用して翻訳されています。正確性を期していますが、自動翻訳には誤りや不正確な部分が含まれる可能性があることをご承知おきください。原文の原語版が正式な情報源とみなされるべきです。重要な情報については、専門の人間による翻訳を推奨します。本翻訳の利用により生じたいかなる誤解や解釈違いについても、当方は責任を負いかねます。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->