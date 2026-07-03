# ジェネレーティブAI入門 - Javaエディション

## 学べること

- LLM、プロンプトエンジニアリング、トークン、埋め込み、ベクトルデータベースを含む<strong>ジェネレーティブAIの基礎</strong>
- Azure OpenAI SDK、Spring AI、OpenAI Java SDKを含む<strong>JavaのAI開発ツールの比較</strong>
- AIエージェント間の通信における<strong>Model Context Protocolの役割とその理解</strong>

## 目次

- [イントロダクション](#introduction)
- [ジェネレーティブAIの概念の簡単な復習](#ジェネレーティブaiの概念の簡単な復習)
- [プロンプトエンジニアリングの復習](#プロンプトエンジニアリングの復習)
- [トークン、埋め込み、エージェント](#トークン、埋め込み、エージェント)
- [JavaのAI開発ツールとライブラリ](#javaのai開発ツールとライブラリ)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [まとめ](#まとめ)
- [次のステップ](#次のステップ)

## Introduction

ジェネレーティブAI初心者向け - Javaエディションの最初の章へようこそ！この基礎レッスンではジェネレーティブAIのコア概念と、それをJavaで扱う方法を紹介します。Large Language Models（LLMs）、トークン、埋め込み、AIエージェントなどAIアプリケーションの基礎となる構成要素について学びます。また、本コースで使用する主要なJavaツールについても探ります。

### ジェネレーティブAIの概念の簡単な復習

ジェネレーティブAIは、データから学習したパターンや関係を基にテキスト、画像、コードなどの新しいコンテンツを生成する人工知能の一種です。ジェネレーティブAIモデルは人間のような応答を生成し、文脈を理解し、時には人間らしいコンテンツを作り出すことさえあります。

JavaによるAIアプリ開発では、<strong>ジェネレーティブAIモデル</strong>を使ってコンテンツを作成します。ジェネレーティブAIモデルが持つ主な能力には次のようなものがあります：

- <strong>テキスト生成</strong>：チャットボット、コンテンツ生成、テキスト補完のための人間らしいテキスト作成
- <strong>画像生成と解析</strong>：リアルな画像の生成、写真の強化、物体検出
- <strong>コード生成</strong>：コードスニペットやスクリプトの作成

タスクに合わせて最適化されたモデルのタイプがあります。例えば、テキスト生成には<strong>小規模言語モデル（SLM）</strong>と<strong>大規模言語モデル（LLM）</strong>の両方が使えますが、複雑なタスクではLLMの方が性能が良いことが多いです。画像関連のタスクでは、専門のビジョンモデルやマルチモーダルモデルを使います。

![Figure: Generative AI model types and use cases.](../../../translated_images/ja/llms.225ca2b8a0d34473.webp)

もちろん、モデルの応答は常に完璧というわけではありません。モデルが「幻覚」的に誤った情報を威厳を持って生成することもあります。しかし、明確な指示や文脈を提供することで、より良い応答を生成するようモデルを導くことができます。ここで登場するのが<strong>プロンプトエンジニアリング</strong>です。

#### プロンプトエンジニアリングの復習

プロンプトエンジニアリングは、AIモデルを望ましい出力に導く効果的な入力を設計する技術です。これには以下が含まれます：

- <strong>明確さ</strong>：指示を明確かつ曖昧さなくすること
- <strong>文脈</strong>：必要な背景情報を提供すること
- <strong>制約</strong>：任意の制限やフォーマットを指定すること

プロンプトエンジニアリングのベストプラクティスには、プロンプト設計、明確な指示、タスクの分解、ワンショット・フューショット学習、プロンプトチューニングがあります。異なるプロンプトをテストして、特定の用途に最適なものを見つけることが不可欠です。

アプリ開発時にはさまざまなタイプのプロンプトを扱います：
- <strong>システムプロンプト</strong>：モデルの振る舞いの基本ルールや文脈を設定
- <strong>ユーザープロンプト</strong>：アプリケーションユーザーからの入力データ
- <strong>アシスタントプロンプト</strong>：システムとユーザープロンプトに基づくモデルの応答

> <strong>詳細はこちら</strong>: [ジェネレーティブAI初心者向けコースのプロンプトエンジニアリング章](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### トークン、埋め込み、エージェント

ジェネレーティブAIモデルを扱うときに、<strong>トークン</strong>、<strong>埋め込み</strong>、<strong>エージェント</strong>、および<strong>Model Context Protocol (MCP)</strong> という用語に出会います。これらの概念を詳しく説明します。

- <strong>トークン</strong>：モデル内でテキストの最小単位です。単語、文字、またはサブワードが含まれます。トークンはモデルが理解できる形式でテキストデータを表現します。例えば、「The quick brown fox jumped over the lazy dog」という文は、トークン化戦略によって ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] や ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"] のようにトークンに分解されます。

![Figure: Generative AI tokens example of breaking words into tokens](../../../translated_images/ja/tokens.6283ed277a2ffff4.webp)

トークン化はテキストをこれらの小単位に分解する処理です。モデルは生テキストではなくトークンを操作するため、重要な処理となります。プロンプト内のトークン数は応答の長さや品質に影響を与えます。モデルには文脈ウィンドウのトークン制限があり（例：GPT-4oの合計文脈は128Kトークン、入力と出力を含む）、これを超えることはできません。

  Javaでは、OpenAI SDKのようなライブラリを使い、AIモデルへのリクエスト時にトークン化を自動的に扱えます。

- <strong>埋め込み</strong>：トークンの意味を捉えたベクトル表現です。通常は浮動小数点数の配列で表現され、単語間の意味的な関連性をモデルが理解できるようにします。類似した単語は類似した埋め込みを持ち、同義語や意味的関係を扱うことが可能です。

![Figure: Embeddings](../../../translated_images/ja/embedding.398e50802c0037f9.webp)

  JavaではOpenAI SDKや埋め込み生成をサポートする他のライブラリを使って埋め込みを生成できます。これらは意味的検索のように意味に基づき類似したコンテンツを見つけるタスクに不可欠です。

- <strong>ベクトルデータベース</strong>：埋め込みを効率的に保存・検索できる特殊なストレージシステムです。これにより厳密なテキストマッチングではなく意味的類似性に基づく大規模データセットからの関連情報検索が効率化されます。Retrieval-Augmented Generation (RAG) パターンで特に重要です。

![Figure: Vector database architecture showing how embeddings are stored and retrieved for similarity search.](../../../translated_images/ja/vector.f12f114934e223df.webp)

> <strong>注意</strong>：本コースではベクトルデータベース自体は扱いませんが、実際のアプリケーションで一般的に使われているため紹介しています。

- **エージェント & MCP**：モデルやツール、外部システムと自律的にやりとりするAIコンポーネントです。Model Context Protocol（MCP）は、エージェントが外部データソースやツールに安全にアクセスするための標準化された方法を提供します。詳細は [MCP for Beginners](https://github.com/microsoft/mcp-for-beginners) コースで学べます。

JavaのAIアプリケーションでは、テキスト処理にトークン、意味検索やRAGに埋め込み、データ検索にベクトルデータベース、知的なツール連携にエージェントとMCPを利用します。

![Figure: how a prompt becomes a reply—tokens, vectors, optional RAG lookup, LLM thinking, and an MCP agent all in one quick flow..](../../../translated_images/ja/flow.f4ef62c3052d12a8.webp)

### JavaのAI開発ツールとライブラリ

Javaは優れたAI開発用ツールを備えています。本コースで扱う主要な3つのライブラリは、OpenAI Java SDK、Azure OpenAI SDK、Spring AIです。

以下は各章の例で使用されるSDKのリファレンステーブルです：

| 章 | サンプル | SDK |
|---------|--------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**SDKドキュメントリンク：**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

OpenAI SDKはOpenAI APIの公式Javaライブラリで、OpenAIのモデルとの対話を簡単かつ一貫したインターフェースで提供し、JavaアプリケーションへAI機能を統合しやすくします。第4章のPet StoryアプリケーションおよびFoundry Local例ではOpenAI SDKを使ったAzure AI Foundryとの連携を実演しています。

#### Spring AI

Spring AIは、さまざまなAIプロバイダー間で一貫した抽象レイヤーを提供し、SpringアプリケーションにAI機能をもたらす包括的なフレームワークです。Springエコシステムとシームレスに連携し、企業向けJavaアプリに理想的な選択肢です。

Spring AIの強みは、依存性注入や設定管理、テストフレームワークなど馴染みのあるSpringパターンでプロダクションレベルAIアプリを容易に構築できる点にあります。第2章と第4章では、OpenAIとModel Context Protocol（MCP）を組み合わせたSpring AIライブラリを使用してアプリケーションを作成します。

##### Model Context Protocol (MCP)

[Model Context Protocol (MCP)](https://modelcontextprotocol.io/) は、AIアプリケーションが外部データソースやツールと安全にやりとりするための新たな標準です。MCPは、AIモデルが文脈情報に安全にアクセスし、アプリケーション内でアクションを実行するための標準化された方法を提供します。

第4章では、基本的なツール統合とサービスアーキテクチャを示す簡単なMCP電卓サービスをSpring AIで構築し、MCPの基礎を体験します。

#### Azure OpenAI Java SDK

Azure OpenAIのJavaクライアントライブラリは、OpenAIのREST APIをアダプトし、Azure SDKエコシステムに適したイディオマティックなインターフェースおよび統合を提供します。第3章ではAzure OpenAI SDKを使ったチャットアプリ、関数呼び出し、RAG（Retrieval-Augmented Generation）パターンのアプリを構築します。

> 注意：Azure OpenAI SDKは機能面でOpenAI Java SDKに遅れを取っているため、今後のプロジェクトではOpenAI Java SDKの使用を検討してください。

## まとめ

ここまでで基礎を学びました！理解した内容は：

- ジェネレーティブAIのコア概念—LLMやプロンプトエンジニアリングからトークン、埋め込み、ベクトルデータベースまで
- Java AI開発のツールキットの選択肢：Azure OpenAI SDK、Spring AI、OpenAI Java SDK
- Model Context Protocolの概要と、外部ツールと連携するAIエージェントの仕組み

## 次のステップ

[第2章：開発環境のセットアップ](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責事項**：
本書類は AI 翻訳サービス [Co-op Translator](https://github.com/Azure/co-op-translator) を使用して翻訳されています。正確性を期していますが、自動翻訳には誤りや不正確な部分が含まれる可能性があることをご承知おきください。原文の原語版が正式な情報源とみなされるべきです。重要な情報については、専門の人間による翻訳を推奨します。本翻訳の利用により生じたいかなる誤解や解釈違いについても、当方は責任を負いかねます。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->