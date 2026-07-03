# Java向け生成AIの開発環境の設定

> **クイックスタート:** Bicep + `azd` を使用して数分でコードとして **Azure AI Foundry** にAIモデルをプロビジョニング — 詳しくは [Azure AI Foundry セットアップガイド](getting-started-azure-openai.md) を参照してください。認証は<strong>キー不要</strong>（Microsoft Entra ID）で、APIキーの管理は不要です。

## 学べること

- AIアプリケーション向けのJava開発環境のセットアップ
- 使いたい開発環境の選択と設定（Codespacesを使ったクラウド優先、ローカルの開発コンテナ、または完全なローカルセットアップ）
- Azure AI Foundryモデルへの接続によるセットアップのテスト

## 目次

- [学べること](#学べること)
- [はじめに](#はじめに)
- [ステップ1: 開発環境のセットアップ](#ステップ1-開発環境のセットアップ)
  - [オプションA: GitHub Codespaces（推奨）](#オプションa-github-codespaces推奨)
  - [オプションB: ローカル開発コンテナ](#オプションb-ローカル開発コンテナ)
  - [オプションC: 既存のローカルインストールを使用](#オプションc-既存のローカルインストールを使用)
- [ステップ2: Azure AI Foundryのプロビジョニング](#ステップ2-azure-ai-foundryのプロビジョニング)
- [ステップ3: セットアップのテスト](#ステップ3-セットアップのテスト)
- [トラブルシューティング](#トラブルシューティング)
- [まとめ](#まとめ)
- [次のステップ](#次のステップ)

## はじめに

この章では開発環境の設定方法を説明します。本コースではモデルに **Azure AI Foundry** を使用します。Bicep と Azure Developer CLI（`azd`）でコードとしてモデルをプロビジョニングし、<strong>キー不要認証</strong>（Microsoft Entra ID）で接続します。APIキーをコピーしたり管理したりする必要はありません。

**ローカルセットアップは不要です！** GitHub Codespacesを利用すればブラウザ上で完全な開発環境が利用可能で、そこからFoundryのプロビジョニングが行えます。

本コースで **Azure AI Foundry** を使う理由は：
- <strong>コードとしてプロビジョニング可能</strong> — 1回の `azd up` でアカウントとモデルの展開をデプロイ
- <strong>キー不要</strong> — AzureサインインやマネージドIDで認証
- <strong>本番対応済み</strong> — 同じコードがローカルとAzureの両方で実行可能
- <strong>柔軟性</strong> — コードを変えずに展開名を変えるだけでモデルの切り替えが可能

> <strong>注意</strong>: Azure AI Foundryの展開はトークン単位の従量課金（ペイ・アズ・ユー・ゴー）です。プロビジョニング、リージョン、コスト詳細については [Azure AI Foundryセットアップガイド](getting-started-azure-openai.md) を参照してください。


## ステップ1: 開発環境のセットアップ

<a name="quick-start-cloud"></a>

本コース向けに事前に設定済みの開発コンテナを用意しています。セットアップ時間を短縮し必要なツールを確実に揃えられます。お好みの開発方法を選択してください：

### 開発環境セットアップの選択肢：

#### オプションA: GitHub Codespaces（推奨）

**ローカルセットアップ不要、2分でコーディング開始可能！**

1. このリポジトリをGitHubのアカウントにフォークする  
   > <strong>注意</strong>: 基本設定を編集したい場合は [Dev Container Configuration](../../../.devcontainer/devcontainer.json) をご確認ください
2. **Code** ボタン → **Codespaces** タブ → **...** → **New with options...** をクリック
3. デフォルトのままでOK — 本コース用に作成されたカスタムdevcontainer「Generative AI Java Development Environment」が選択されます
4. **Create codespace** をクリック
5. 約2分待って環境が準備されるのを待つ
6. [ステップ2: Azure AI Foundryのプロビジョニング](#ステップ2-azure-ai-foundryのプロビジョニング) へ進む

<img src="../../../translated_images/ja/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: Codespaces submenu" width="50%">

<img src="../../../translated_images/ja/image.833552b62eee7766.webp" alt="Screenshot: New with options" width="50%">

<img src="../../../translated_images/ja/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: Create codespace options" width="50%">


> **Codespacesの利点**:
> - ローカルインストール不要
> - ブラウザがあればどのデバイスでも動作
> - 全ツール・依存関係が事前構成済み
> - 個人アカウントは月60時間無料
> - 受講者全員に一貫した環境を提供

#### オプションB: ローカル開発コンテナ

**Dockerを用いたローカル開発を好む開発者向け**

1. このリポジトリをローカルにフォーク＆クローンする  
   > <strong>注意</strong>: 基本設定を編集したい場合は [Dev Container Configuration](../../../.devcontainer/devcontainer.json) をご確認ください
2. [Docker Desktop](https://www.docker.com/products/docker-desktop/) と [VS Code](https://code.visualstudio.com/) をインストール
3. VS Codeに [Dev Containers拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) をインストール
4. VS Codeでリポジトリフォルダーを開く
5. プロンプトが表示されたら **Reopen in Container** をクリック（または `Ctrl+Shift+P` → 「Dev Containers: Reopen in Container」）
6. コンテナのビルドと起動を待つ
7. [ステップ2: Azure AI Foundryのプロビジョニング](#ステップ2-azure-ai-foundryのプロビジョニング) へ進む

<img src="../../../translated_images/ja/devcontainer.21126c9d6de64494.webp" alt="Screenshot: Dev container setup" width="50%">

<img src="../../../translated_images/ja/image-3.bf93d533bbc84268.webp" alt="Screenshot: Dev container build complete" width="50%">

#### オプションC: 既存のローカルインストールを使用

**すでにJava環境を持つ開発者向け**

前提条件：
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html)
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) またはお好みのIDE

手順：
1. このリポジトリをローカルにクローン
2. IDEでプロジェクトを開く
3. [ステップ2: Azure AI Foundryのプロビジョニング](#ステップ2-azure-ai-foundryのプロビジョニング) へ進む

> <strong>プロのヒント</strong>: 低スペックマシンでローカルにVS Codeを使いたい場合、GitHub Codespacesがおすすめ！ローカルのVS Codeからクラウド上のCodespaceに接続して両方の利点を活用できます。

<img src="../../../translated_images/ja/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: created local devcontainer instance" width="50%">


## ステップ2: Azure AI Foundryのプロビジョニング

本コースのAIモデルをコードとしてAzure AI Foundryに展開します。リポジトリルートから：

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```
  
`azd` は環境名とリージョンを尋ね、`gpt-4o-mini` と `text-embedding-3-small` の展開を含むAzure AI Foundryアカウントをプロビジョニングし、例の `.env` にエンドポイントを書き込みます — すべて<strong>キー不要</strong>認証（APIキー不要）で行います。

> **完全な手順:** 前提条件、手動（ポータル）による代替、リージョンの案内、コストやクリーンアップの注意点については [Azure AI Foundryセットアップガイド](getting-started-azure-openai.md) を参照してください。

## ステップ3: セットアップのテスト

Foundryモデルのプロビジョニングが完了したら、[`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) のサンプルアプリで接続をテストします。

1. 開発環境内のターミナルを開く。
2. サンプルへ移動：  
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
3. サインインしているか確認（キー不要認証にはトークンが必要）：  
   ```bash
   az login
   ```
   > もし `azd up` を実行済みなら、エンドポイントが書き込まれた `.env` ファイルが既にあります。  
4. アプリケーションを実行：  
   ```bash
   mvn clean spring-boot:run
   ```
  
`gpt-4o-mini` モデルからの応答が見られるはずです。

### サンプルコードの理解

`examples/basic-chat-azure` にあるサンプルはSpring Bootアプリで、**Spring AI** を使いキー不要認証でAzure AI Foundryに接続します。

**このコードの内容：**  
- Azureサインイン（Microsoft Entra ID）でAzure AI Foundryに<strong>接続</strong>（APIキー不要）  
- `gpt-4o-mini` モデルにプロンプトを<strong>送信</strong>  
- AIの応答を<strong>受信</strong>し表示  
- セットアップが正しく動作するか<strong>検証</strong>

<strong>重要な依存関係</strong>（`pom.xml`内）：  
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```
  
<strong>設定ファイル</strong>（`application.yml`）：  
```yaml
spring:
  ai:
    azure:
      openai:
        # Endpoint only - no api-key. Spring AI uses DefaultAzureCredential (keyless).
        endpoint: ${AZURE_OPENAI_ENDPOINT}
        chat:
          options:
            deployment-name: ${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}
```


## まとめ

素晴らしい！これで以下がすべて整いました：

- Bicep + `azd` でコードとしてAzure AI Foundryモデルをプロビジョニング済み
- Javaの開発環境が動作中（Codespacesでもdev containerでもローカルでも）
- キー不要認証（Microsoft Entra ID）でAzure AI Foundryに接続済み — APIキー不要
- モデルとやり取りする簡単な例で動作確認済み

## 次のステップ

[第3章: コア生成AI技術](../03-CoreGenerativeAITechniques/README.md)

## トラブルシューティング

問題が発生していますか？よくある問題と解決法：

- **認証が失敗する（401/403）？**  
  - `az login` を実行 — 認証はキー不要なのでサインインが必要  
  - アカウントにリソースの **Cognitive Services OpenAI User** ロールがあるか確認  
  - プロビジョニング直後はロール割当の反映に数分かかることがあります

- **Mavenが見つからない？**  
  - dev container/Codespaces利用ならMavenは事前インストール済み  
  - ローカルセットアップの場合はJava 21+ と Maven 3.9+ がインストールされているか確認  
  - `mvn --version` でインストール状況を確認

- **`azd`が見つからない、またはプロビジョニングに失敗？**  
  - [Azure Developer CLI](https://aka.ms/azure-dev/install) をインストールし `azd auth login` を実行  
  - `gpt-4o-mini` が利用可能なリージョンを選択（例：`eastus2`）  
  - 詳細は [Azure AI Foundryセットアップガイド](getting-started-azure-openai.md) を参照

- **Dev containerが起動しない？**  
  - Docker Desktopが起動しているか確認（ローカル開発時）  
  - コンテナの再ビルドを試す：`Ctrl+Shift+P` → 「Dev Containers: Rebuild Container」

- **アプリケーションのコンパイルエラー？**  
  - 正しいディレクトリにいるか確認：`02-SetupDevEnvironment/examples/basic-chat-azure`  
  - `mvn clean compile` でクリーンビルドを試す

> **サポートが必要ですか？** 問題が解決しない場合は、リポジトリでIssueを開いてください。お手伝いします。

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責事項**：
本書類は AI 翻訳サービス [Co-op Translator](https://github.com/Azure/co-op-translator) を使用して翻訳されています。正確性を期していますが、自動翻訳には誤りや不正確な部分が含まれる可能性があることをご承知おきください。原文の原語版が正式な情報源とみなされるべきです。重要な情報については、専門の人間による翻訳を推奨します。本翻訳の利用により生じたいかなる誤解や解釈違いについても、当方は責任を負いかねます。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->