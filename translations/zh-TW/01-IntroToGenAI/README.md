# 生成式 AI 入門 - Java 版

## 您將學到的內容

- **生成式 AI 基礎**，包含大型語言模型 (LLM)、提示工程、分詞、嵌入向量和向量資料庫
- **比較 Java AI 開發工具**，包含 Azure OpenAI SDK、Spring AI 及 OpenAI Java SDK
- **探索模型上下文協議 (MCP)** 及其在 AI 代理通訊中的角色

## 目錄

- [簡介](#簡介)
- [生成式 AI 概念快速複習](#生成式-ai-概念快速複習)
- [提示工程複習](#提示工程複習)
- [分詞、嵌入與代理](#分詞、嵌入與代理)
- [Java 的 AI 開發工具與函式庫](#java-的-ai-開發工具與函式庫)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [總結](#總結)
- [後續步驟](#後續步驟)

## 簡介

歡迎來到《生成式 AI 初學者 - Java 版》的第一章！這個基礎課程將介紹生成式 AI 的核心概念，以及如何使用 Java 進行操作。您將學習 AI 應用的基本組件，包括大型語言模型 (LLM)、分詞、嵌入向量和 AI 代理。我們也將探索本課程中將使用的主要 Java 工具。

### 生成式 AI 概念快速複習

生成式 AI 是一種基於從資料中學習的模式與關聯，產生新內容（如文字、圖片或程式碼）的人工智慧。生成式 AI 模型能夠產生類似人類的回應、理解上下文，甚至有時能創作看起來像人類所寫的內容。

開發 Java AI 應用時，您會使用<strong>生成式 AI 模型</strong>來製作內容。生成式 AI 模型的一些功能包括：

- <strong>文字生成</strong>：撰寫聊天機器人對話、內容與文字補全。
- <strong>影像生成與分析</strong>：製作逼真影像、增強照片以及物件偵測。
- <strong>程式碼生成</strong>：撰寫程式碼片段或腳本。

有些模型類型會針對不同任務進行優化。例如，**小型語言模型 (SLM)** 和 **大型語言模型 (LLM)** 都能處理文字生成，但 LLM 通常在複雜任務上表現更佳。對於影像相關任務，您會選用專門的視覺模型或多模態模型。

![Figure: Generative AI model types and use cases.](../../../translated_images/zh-TW/llms.225ca2b8a0d34473.webp)

當然，這些模型的回應並非每次都完美無缺。您可能聽說過模型「幻覺」或以權威口吻生成錯誤資訊。但您可以透過提供清晰的指示和上下文，協助模型產生更佳回應。這就是<strong>提示工程</strong>的用武之地。

#### 提示工程複習

提示工程是設計有效輸入以引導 AI 模型產生期望輸出的做法。它包含：

- <strong>清晰度</strong>：使指令明確且無歧義。
- <strong>上下文</strong>：提供必要的背景資訊。
- <strong>限制條件</strong>：指定任何限制或格式。

提示工程的最佳實務包括提示設計、清楚指示、任務拆解、一次學習與少量示例學習 (one-shot & few-shot)、以及提示微調。測試不同提示對於找到最適合您案例的方案至關重要。

開發應用時，您會使用不同類型的提示：
- <strong>系統提示</strong>：設定模型行為的基本規則與上下文
- <strong>使用者提示</strong>：來自應用使用者的輸入資料
- <strong>助理提示</strong>：模型根據系統與使用者提示所產生的回應

> <strong>深入瞭解</strong>：請參考 [生成式 AI 初學者課程中的提示工程章節](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals) 以獲得更多提示工程資訊。

#### 分詞、嵌入與代理

使用生成式 AI 模型時，您會遇到「分詞 (tokens)」、「嵌入向量 (embeddings)」、「代理 (agents)」以及<strong>模型上下文協議 (Model Context Protocol, MCP)</strong> 等名詞。以下是這些概念的詳細介紹：

- <strong>分詞</strong>：分詞是模型中文本的最小單位，可為詞、字元或子詞。分詞用以將文字資料轉換成模型能理解的格式。例如句子「The quick brown fox jumped over the lazy dog」可依分詞策略拆成 ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] 或 ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"]。

![Figure: Generative AI tokens example of breaking words into tokens](../../../translated_images/zh-TW/tokens.6283ed277a2ffff4.webp)

分詞是把文字拆解成這些更小單位的過程。此過程很重要，因為模型是針對分詞運算，而非原始文字。提示中分詞數量會影響模型回應的長度與品質，因為模型對上下文視窗有分詞數限制（例如 GPT-4o 的總上下文包含輸入與輸出最多 128K 分詞）。

  在 Java 中，您可以使用像 OpenAI SDK 的函式庫，自動處理向 AI 模型發送請求時的分詞。

- <strong>嵌入向量</strong>：嵌入向量是分詞的向量表示，用於捕捉語意意義。它是數值型的表示（通常為浮點數陣列），可讓模型理解單詞間的關聯並產生有語境的回應。語意相近的詞擁有相似嵌入，使模型能了解同義詞與語意關係。

![Figure: Embeddings](../../../translated_images/zh-TW/embedding.398e50802c0037f9.webp)

  在 Java 中，您可以使用 OpenAI SDK 或其他支援嵌入向量生成的函式庫來產生嵌入。這些嵌入對於語意搜尋特別重要，例如根據意義找出相似內容，而非精確字詞比對。

- <strong>向量資料庫</strong>：向量資料庫是經過優化，用於儲存嵌入向量的專用儲存系統。它能高效執行相似度搜尋，對於「檢索增強生成」（Retrieval-Augmented Generation, RAG）模式十分關鍵，當您想根據語意相似度從大量資料中找到相關資訊時非常有用。

![Figure: Vector database architecture showing how embeddings are stored and retrieved for similarity search.](../../../translated_images/zh-TW/vector.f12f114934e223df.webp)

> <strong>注意</strong>：本課程不涵蓋向量資料庫，但值得一提的是，它們在實務應用中非常常見。

- **代理與 MCP**：代理是能自主與模型、工具及外部系統互動的 AI 元件。模型上下文協議 (MCP) 為代理提供標準化方式，以安全存取外部資料源及工具。詳情可參考我們的 [MCP 初學者課程](https://github.com/microsoft/mcp-for-beginners)。

在 Java AI 應用中，您會用分詞進行文字處理，嵌入用於語意搜尋與 RAG，向量資料庫用於資料檢索，代理搭配 MCP 則用於構建智慧型使用工具的系統。

![Figure: how a prompt becomes a reply—tokens, vectors, optional RAG lookup, LLM thinking, and an MCP agent all in one quick flow..](../../../translated_images/zh-TW/flow.f4ef62c3052d12a8.webp)

### Java 的 AI 開發工具與函式庫

Java 提供優秀的 AI 開發工具。本課程將探索三個主要函式庫：OpenAI Java SDK、Azure OpenAI SDK 和 Spring AI。

以下為各章節範例使用的 SDK 快速對照表：

| 章節 | 範例 | SDK |
|---------|--------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**SDK 文件連結：**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

OpenAI SDK 是官方提供的 OpenAI API Java 函式庫，提供簡單一致的介面與 OpenAI 模型互動，使得在 Java 應用中整合 AI 功能變得容易。第 4 章的寵物故事應用與 Foundry Local 範例演示了 OpenAI SDK 在 Azure AI Foundry 的使用方法。

#### Spring AI

Spring AI 是一套針對 Spring 應用整合 AI 能力的完整框架，提供多家 AI 服務提供者間的一致抽象層。它無縫整合於 Spring 生態系統，是企業 Java 應用需要 AI 功能的理想選擇。

Spring AI 的強項在於與 Spring 生態系統緊密結合，讓開發者能用熟悉的 Spring 模式（依賴注入、配置管理、測試框架）輕鬆打造生產級 AI 應用。您會在第 2 與第 4 章使用 Spring AI，結合 OpenAI 與模型上下文協議 (MCP) 的 Spring AI 函式庫。

##### 模型上下文協議 (MCP)

[模型上下文協議 (MCP)](https://modelcontextprotocol.io/) 是新興標準，使 AI 應用能安全地與外部資料來源及工具互動。MCP 提供 AI 模型存取上下文資訊及執行應用中動作的標準方式。

第 4 章中，您將構建一個簡單的 MCP 計算機服務，展現 MCP 與 Spring AI 的基礎，示範如何創建基本工具整合與服務架構。

#### Azure OpenAI Java SDK

Azure OpenAI Java 用戶端函式庫是 OpenAI REST API 的 Azure SDK 介面封裝，提供語法風格更自然且與 Azure SDK 生態系統整合的介面。第 3 章會使用 Azure OpenAI SDK 建構聊天應用、函式呼叫及 RAG（檢索增強生成）模式應用。

> 注意：Azure OpenAI SDK 功能落後於 OpenAI Java SDK，未來專案建議考慮使用 OpenAI Java SDK。

## 總結

基礎介紹至此結束！您目前已了解：

- 生成式 AI 的核心概念 — 從大型語言模型與提示工程，到分詞、嵌入與向量資料庫
- Java AI 開發工具選擇：Azure OpenAI SDK、Spring AI 及 OpenAI Java SDK
- 模型上下文協議的意義及其讓 AI 代理能與外部工具協作的方式

## 後續步驟

[第 2 章：建立開發環境](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
此文件已使用 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 進行翻譯。雖然我們努力追求準確性，但請注意自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應視為權威來源。對於關鍵資訊，建議採用專業人工翻譯。我們不對因使用此翻譯所產生的任何誤解或誤譯承擔責任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->