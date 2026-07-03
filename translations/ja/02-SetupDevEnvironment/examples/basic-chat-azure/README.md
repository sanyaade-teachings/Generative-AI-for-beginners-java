# Azure AI Foundryを使った基本的なチャット - エンドツーエンドの例

この例は、<strong>Azure AI Foundry</strong>モデルに<strong>キーなし認証</strong>（Microsoft Entra ID）を使って接続し、セットアップをテストするシンプルなSpring Bootアプリケーションです。Spring AIの`ChatClient`を使用しています。

## 目次

- [前提条件](#前提条件)
- [クイックスタート](#クイックスタート)
- [認証の仕組み](#認証の仕組み)
- [アプリケーションの実行](#アプリケーションの実行)
  - [Mavenの使用](#mavenの使用)
  - [VS Codeの使用](#vs-codeの使用)
  - [期待される出力](#期待される出力)
- [設定リファレンス](#設定リファレンス)
  - [環境変数](#環境変数)
  - [Springの設定](#springの設定)
- [トラブルシューティング](#トラブルシューティング)
  - [よくある問題](#よくある問題)
  - [デバッグモード](#デバッグモード)
- [次のステップ](#次のステップ)
- [リソース](#リソース)

## 前提条件

この例を実行する前に、以下を確認してください：

- `gpt-4o-mini`デプロイメントを持つAzure AI Foundryリソース — `azd up`でプロビジョニングするか、[Azure AI Foundryセットアップガイド](../../getting-started-azure-openai.md)で手動で作成
- そのリソースに対する<strong>Cognitive Services OpenAI User</strong>ロール（Bicepテンプレートは自動で割り当てます）
- サインイン済みの[Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli) (`az login`実行済み)
- Java 21以上とMaven 3.9以上

> **APIキー不要** — 認証はMicrosoft Entra ID経由のキーなしです。

## クイックスタート

```bash
# 1. プロジェクトに移動します
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. キーレス認証でトークンを取得できるようにサインインします
az login

# 3. エンドポイントを構成します
#    - `azd up` を実行した場合、.env が自動で作成されます（このステップはスキップしてください）。
#    - それ以外の場合はテンプレートをコピーし、AZURE_OPENAI_ENDPOINT を設定します。
cp .env.example .env

# 4. アプリケーションを実行します
mvn spring-boot:run
```

## 認証の仕組み

この例では<strong>Microsoft Entra ID</strong>で認証します — APIキーはありません。

`spring.ai.azure.openai.endpoint`のみが設定され（api-keyは設定なし）、Spring AIは [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential)でAzure OpenAIクライアントを構築します。この認証情報は、ローカルの`az login`セッションからトークンを自動取得するか、Azure実行時のマネージドIDから取得します。そのため同じコードがどちらの環境でも変更なしに動作します。

## アプリケーションの実行

### Mavenの使用

```bash
mvn spring-boot:run
```

### VS Codeの使用

1. プロジェクトをVS Codeで開く
2. `F5`キーを押すか、「Run and Debug」パネルを使う
3. 「Spring Boot-BasicChatApplication」構成を選択

> <strong>注意</strong>: VS Codeの設定は自動で `.env` ファイルを読み込みます

### 期待される出力

```
Starting Basic Chat with Azure OpenAI...
Environment variables loaded successfully
Connecting to Azure OpenAI...
Sending prompt: What is AI in a short sentence? Max 100 words.

AI Response:
================
AI, or Artificial Intelligence, is the simulation of human intelligence in machines programmed to think and learn like humans.
================

Success! Azure OpenAI connection is working correctly.
```

## 設定リファレンス

### 環境変数

| 変数名 | 説明 | 必須 | 例 |
|--------|---------|------|----|
| `AZURE_OPENAI_ENDPOINT` | Foundry（Azure OpenAI）のエンドポイントURL | 必須 | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | チャットモデルのデプロイメント名 | 任意 | `gpt-4o-mini`（デフォルト） |

> APIキー変数は<strong>ありません</strong> — 認証はキーなし（`az login`経由のMicrosoft Entra ID）です。

### Springの設定

`application.yml`ファイルの設定内容：
- <strong>エンドポイント</strong>：`${AZURE_OPENAI_ENDPOINT}` - 環境変数から取得
- <strong>デプロイメント</strong>：`${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - 環境変数から取得、デフォルトあり
- <strong>認証</strong>：キーなし — `api-key`は設定せず、Spring AIは`DefaultAzureCredential`を使用
- <strong>温度</strong>：`0.7` - 創造性を制御（0.0 ＝ 決定的、1.0 ＝ 創造的）
- <strong>最大トークン数</strong>：`500` - 最大応答長

## トラブルシューティング

### よくある問題

<details>
<summary><strong>エラー: 401 / "PermissionDenied" / トークン関連エラー</strong></summary>

- `az login`を実行 — キーなし認証にはトークン取得のための有効なサインインが必要
- アカウントにリソースの<strong>Cognitive Services OpenAI User</strong>ロールが付与されているか確認
- ロールを割り当てたばかりなら、反映まで数分待つ
- 正しいテナント／サブスクリプションであることを確認 (`az account show`)
</details>

<details>
<summary><strong>エラー: "The endpoint is not valid" / 接続エラー</strong></summary>

- `AZURE_OPENAI_ENDPOINT`が完全なベースURLか確認（例：`https://your-resource.openai.azure.com/`）
- 末尾スラッシュの有無を確認
- エンドポイントがプロビジョニング済みリソースと合っているか検証 (`azd env get-values`)
</details>

<details>
<summary><strong>エラー: "The deployment was not found"</strong></summary>

- `AZURE_OPENAI_DEPLOYMENT`がAzure上のデプロイメント名と一致しているか確認
- モデルが正常にデプロイされ、アクティブ状態か確認
- デフォルトのデプロイメント名は `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: 環境変数が読み込まれない</strong></summary>

- `.env`ファイルがプロジェクトルート（`pom.xml`と同階層）にあるか確認
- VS Codeの統合ターミナルで `mvn spring-boot:run` を試す
- VS CodeのJava拡張機能が正しくインストールされているかチェック
</details>

### デバッグモード

詳細なロギングを有効にするには、`application.yml`内の以下行のコメントを外します：

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## 次のステップ

**セットアップ完了！** 引き続き学習を進めましょう：

[第3章: コアな生成AI技術](../../../03-CoreGenerativeAITechniques/README.md)

## リソース

- [Spring AI Azure OpenAI ドキュメント](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Microsoft Entra IDによるキーなし認証](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundryポータル](https://ai.azure.com/)
- [Azure AI Foundry ドキュメント](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責事項**：
本書類は AI 翻訳サービス [Co-op Translator](https://github.com/Azure/co-op-translator) を使用して翻訳されています。正確性を期していますが、自動翻訳には誤りや不正確な部分が含まれる可能性があることをご承知おきください。原文の原語版が正式な情報源とみなされるべきです。重要な情報については、専門の人間による翻訳を推奨します。本翻訳の利用により生じたいかなる誤解や解釈違いについても、当方は責任を負いかねます。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->