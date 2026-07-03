# Azure AI Foundry の開発環境の設定

> このガイドでは、本コースの Java AI アプリ用に **Azure AI Foundry** モデルを、<strong>キー管理不要</strong> (Microsoft Entra ID) の認証でセットアップします。ツールに不慣れな場合は、[開発環境ガイド](./README.md)から始めてください。

このガイドでは、本コースの Java AI アプリのために **Azure AI Foundry** モデルをセットアップします。方法は二つあります:

- **オプション A — `azd` + Bicep でプロビジョニング（推奨）:** ワンコマンドで Foundry アカウントとモデルをコードとしてデプロイ。ポータルでの操作不要。
- **オプション B — Azure AI Foundry ポータルで手動でリソース作成。**

どちらの方法も <strong>キー管理不要の認証</strong>（Microsoft Entra ID）を使用するため、API キーをコピーしたり漏えいの心配はありません。

## 目次

- [作成されるもの](#作成されるもの)
- [前提条件](#前提条件)
- [オプション A：azd + Bicep でプロビジョニング（推奨）](#option-a-provision-with-azd--bicep-recommended)
- [オプション B：手動でリソース作成](#オプション-b：手動でリソース作成)
- [環境の構成](#環境の構成)
- [セットアップのテスト](#セットアップのテスト)
- [次は？](#次は？)
- [リソース](#リソース)
- [追加リソース](#追加リソース)

## 作成されるもの

[`infra/`](../../../02-SetupDevEnvironment/infra) の Bicep テンプレートで以下をプロビジョニングします:

- **Azure AI Foundry** アカウント (`Microsoft.CognitiveServices/accounts`、種別は `AIServices`) とプロジェクト
- <strong>チャット</strong> 展開 — `gpt-4o-mini`
- <strong>埋め込み</strong> 展開 — `text-embedding-3-small`（後の章で使用）
- <strong>キー管理不要のロール割り当て</strong>（`Cognitive Services OpenAI User`）、`az login` でサインイン可能に

## 前提条件

- [Azure サブスクリプション](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) と [Maven 3.9+](https://maven.apache.org/download.cgi)

## オプション A：azd + Bicep でプロビジョニング（推奨）

`02-SetupDevEnvironment` フォルダーから:

```bash
cd 02-SetupDevEnvironment

# サインイン（両方のツール）
azd auth login
az login

# Foundryアカウントとモデルデプロイのプロビジョニング
azd up
```
  
`azd` は <strong>環境名</strong>（例: `genai-java`）と <strong>リージョン</strong>を入力するよう求めます。`gpt-4o-mini` と `text-embedding-3-small` が利用可能なリージョンを選択してください（例: `eastus2` または `swedencentral`）。

プロビジョニング終了後、azd は次の処理を行います:

1. [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep) に定義されたすべてをデプロイします。
2. エンドポイントと展開名を含む [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) を生成する postprovision フックを実行します（シークレットは含まれません）。

> **ヒント:** 変更を反映したい場合はいつでも `azd up` を再実行。すべて削除して料金発生を止めたい場合は `azd down` を使います。

生成された設定を確認するには:

```bash
azd env get-values
```
  
次に、[セットアップのテスト](#セットアップのテスト)に進んでください。

## オプション B：手動でリソース作成

ポータル操作を好む場合は、以下の手順でリソースを作成してください:

1. [Azure AI Foundry ポータル](https://ai.azure.com/)にアクセスし、サインイン。
2. <strong>プロジェクトを作成</strong>（同時に AI Foundry リソースも作成されます）。名前は例として `GenAIJava` にしてください。
3. プロジェクト内の **Models + endpoints** → **Deploy model** → **Deploy base model** を開く。
4. <strong>gpt-4o-mini</strong>をデプロイ（展開名は `gpt-4o-mini`）。埋め込み例を使いたい場合は **text-embedding-3-small** も同様にデプロイ。
5. **Overview** から <strong>エンドポイント</strong>をコピー（例: `https://<resource>.openai.azure.com/`）。
6. キー管理不要アクセスを付与: リソース左メニューの **Access control (IAM)** → **Add role assignment** → 自分のアカウントに **Cognitive Services OpenAI User** を割り当て。

> **まだ問題がありますか？** [Azure AI Foundry ドキュメント](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects)を参照してください。

## 環境の構成

**オプション A (`azd up`) を使った場合**、設定ファイルは自動で作成済みなので、構成は不要です。[セットアップのテスト](#セットアップのテスト)へ進んでください。

**オプション B（手動）の場合**、サンプルの `.env` ファイルを自分で作成してください:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```
  
`.env` を編集し、エンドポイントを指定します（キーは不要、認証はキー管理不要）:

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```
  
> **セキュリティ注意:** API キーは保存しません。認証は `az login`（ローカル）または Azure のマネージド ID で Microsoft Entra ID を使います。`.env` ファイルはシークレットを含まない設定のみで、すでに `.gitignore` 管理下にあります。

## セットアップのテスト

キー管理不要認証用のトークン取得のためにサインインしたことを確認し、例を実行してください:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # まだサインインしていない場合
mvn clean spring-boot:run
```
  
`gpt-4o-mini` モデルからの応答が表示されるはずです！

> **VS Code ユーザー:** `F5` キーで実行可能。アプリが `.env` を自動読み込みします。

> **完全な例:** 詳細やトラブルシューティングは [Azure AI Foundry を使った Basic Chat 例](./examples/basic-chat-azure/README.md)をご覧ください。

## 次は？

**セットアップ完了！**  
以下の環境が整いました:
- `gpt-4o-mini` と `text-embedding-3-small` をデプロイした Azure AI Foundry
- キー管理不要認証（Microsoft Entra ID）
- エンドポイントと展開名を含むローカル `.env`
- Java 開発環境の準備完了

<strong>続けて</strong> [第3章: コア生成AI技術](../03-CoreGenerativeAITechniques/README.md) にて AI アプリ構築を始めましょう！

## リソース

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Microsoft Entra ID を使ったキー管理不要認証](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry ドキュメント](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI ドキュメント](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## 追加リソース

- [VS Code をダウンロード](https://code.visualstudio.com/Download)
- [Docker Desktop を入手](https://www.docker.com/products/docker-desktop)
- [Dev Container 構成](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責事項**：
本書類は AI 翻訳サービス [Co-op Translator](https://github.com/Azure/co-op-translator) を使用して翻訳されています。正確性を期していますが、自動翻訳には誤りや不正確な部分が含まれる可能性があることをご承知おきください。原文の原語版が正式な情報源とみなされるべきです。重要な情報については、専門の人間による翻訳を推奨します。本翻訳の利用により生じたいかなる誤解や解釈違いについても、当方は責任を負いかねます。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->