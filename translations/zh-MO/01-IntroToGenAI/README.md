# 介紹生成式人工智能 - Java 版本

## 你將學到什麼

- <strong>生成式人工智能基礎知識</strong>，包括大型語言模型（LLM）、提示工程、標記（tokens）、嵌入向量（embeddings）及向量數據庫
- **比較 Java AI 開發工具**，包括 Azure OpenAI SDK、Spring AI 和 OpenAI Java SDK
- <strong>認識模型上下文協議（Model Context Protocol）</strong>及其在 AI 代理通訊中的角色

## 目錄

- [介紹](#介紹)
- [生成式人工智能概念快速回顧](#生成式人工智能概念快速回顧)
- [提示工程回顧](#提示工程回顧)
- [標記、嵌入向量與代理](#標記、嵌入向量與代理)
- [Java 的 AI 開發工具與函式庫](#java-的-ai-開發工具與函式庫)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [總結](#總結)
- [接下來的步驟](#接下來的步驟)

## 介紹

歡迎來到《生成式人工智能初學者指南 - Java 版本》的第一章！這個基礎課程將帶你認識生成式人工智能的核心概念，以及如何使用 Java 來操作它們。你將學習 AI 應用程式的基本構件，包括大型語言模型（LLM）、標記（tokens）、嵌入向量（embeddings）與 AI 代理（agents）。我們還會介紹在整個課程中會用到的主要 Java 工具。

### 生成式人工智能概念快速回顧

生成式人工智能是一種根據數據中學到的模式和關係，創造新內容（例如文字、圖片或程式碼）的人工智能類型。生成式 AI 模型能夠產生類似人類的回應、理解上下文，甚至有時創造出看似人類創作的內容。

在開發你的 Java AI 應用時，你會使用 **生成式 AI 模型** 來創造內容。生成式 AI 模型的部分能力包括：

- <strong>文字生成</strong>：為聊天機器人、內容及文字補全創作類似人類的文字。
- <strong>影像生成與分析</strong>：製作寫實影像、提升照片品質和物體偵測。
- <strong>程式碼生成</strong>：撰寫程式碼片段或腳本。

不同的模型類型適用於不同任務。例如，<strong>小型語言模型（SLM）</strong>和<strong>大型語言模型（LLM）</strong>皆能處理文字生成，但 LLM 通常在較複雜任務中表現更優。影像相關任務則會使用專門的視覺模型或多模態模型。

![Figure: Generative AI model types and use cases.](../../../translated_images/zh-MO/llms.225ca2b8a0d34473.webp)

當然，這些模型的回應並非每次都完美。你可能聽過模型「幻覺」或以權威口吻產生錯誤資訊。但你可以透過提供明確的指示與上下文引導模型產生更佳的回應。這正是 <strong>提示工程</strong> 的用途。

#### 提示工程回顧

提示工程是設計有效輸入以引導 AI 模型產生期望輸出的做法。它包含：

- <strong>清晰度</strong>：使指令清楚且無歧義。
- <strong>上下文</strong>：提供必要的背景資訊。
- <strong>約束條件</strong>：指定限制或格式。

提示工程的一些最佳實踐包括提示設計、明確指示、任務拆解、單次學習與少量示例學習以及提示調整。測試不同提示以找出最適合特定使用情況的方法是必要的。

在開發應用時，你會使用不同類型的提示：
- <strong>系統提示</strong>：設定模型行為的基礎規則與上下文
- <strong>使用者提示</strong>：來自應用使用者的輸入資料
- <strong>助理提示</strong>：模型根據系統與使用者提示產生的回應

> <strong>深入了解</strong>：請參考[初學者生成式 AI 課程的提示工程章節](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### 標記、嵌入向量與代理

使用生成式 AI 模型時，會遇到<strong>標記（tokens）</strong>、**嵌入向量（embeddings）**、<strong>代理（agents）</strong>以及<strong>模型上下文協議（Model Context Protocol, MCP）</strong>等術語。以下詳細說明這些概念：

- <strong>標記</strong>：標記是模型處理的最小文字單位。可以是單詞、字元或子詞。標記用來將文字資料轉換成模型可理解的格式。例如句子"The quick brown fox jumped over the lazy dog"可能被標記化成 ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"]，或依照標記策略分成 ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"]。

![Figure: Generative AI tokens example of breaking words into tokens](../../../translated_images/zh-MO/tokens.6283ed277a2ffff4.webp)

標記化是將文字拆解為更小單位的過程。這很重要，因為模型是以標記運作，而非原始文字。提示中的標記數量會影響模型回應的長度與品質，因為模型對上下文視窗有標記數限制（例如 GPT-4o 的總上下文限制為 128K 標記，包含輸入與輸出）。

在 Java 中，你可以使用 OpenAI SDK 等函式庫自動處理標記化，發送請求給 AI 模型時會自動完成。

- <strong>嵌入向量</strong>：嵌入向量是標記的向量表示，用來捕捉語義意義。它們通常是浮點數陣列，讓模型能理解詞彙間的關係，並產生語義相關的回應。相似詞會有相似的嵌入，讓模型能理解同義詞及語義連結。

![Figure: Embeddings](../../../translated_images/zh-MO/embedding.398e50802c0037f9.webp)

在 Java 中，你可以透過 OpenAI SDK 或其他支持生成嵌入的函式庫來生成嵌入向量。這對於語義搜尋等任務非常重要，因為你想根據意義找到相似內容，而非純文字匹配。

- <strong>向量數據庫</strong>：向量數據庫是專門為嵌入向量儲存設計的資料庫系統，優化相似度搜尋。它們對於檢索增強生成（RAG）模式至關重要，能根據語義相似度從大型資料集中尋找相關資訊，而非精確匹配。

![Figure: Vector database architecture showing how embeddings are stored and retrieved for similarity search.](../../../translated_images/zh-MO/vector.f12f114934e223df.webp)

> <strong>注意</strong>：本課程不會深入探討向量數據庫，但值得提及，因為它們在實務應用中很普遍。

- **代理與 MCP**：代理是能自主與模型、工具及外部系統互動的 AI 元件。模型上下文協議（MCP）提供標準化方式，使代理能安全存取外部資料源與工具。詳情請參考我們的[MCP 初學者課程](https://github.com/microsoft/mcp-for-beginners)。

在 Java AI 應用中，你會運用標記處理文字、嵌入用於語義搜尋與 RAG、向量數據庫進行資料檢索，並利用帶有 MCP 的代理來構建智能、能使用工具的系統。

![Figure: how a prompt becomes a reply—tokens, vectors, optional RAG lookup, LLM thinking, and an MCP agent all in one quick flow..](../../../translated_images/zh-MO/flow.f4ef62c3052d12a8.webp)

### Java 的 AI 開發工具與函式庫

Java 提供了優秀的 AI 開發工具。我們將在課程中探討三個主要函式庫 — OpenAI Java SDK、Azure OpenAI SDK 和 Spring AI。

下面是一個快速參考表，顯示每章示例中使用的 SDK：

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

OpenAI SDK 是 OpenAI API 的官方 Java 函式庫。它提供簡單且一致的介面，讓你輕鬆將 AI 功能整合到 Java 應用。第四章的 Pet Story 應用程式與 Foundry Local 範例展示了利用 OpenAI SDK 及 Azure AI Foundry 的方法。

#### Spring AI

Spring AI 是專為 Spring 應用帶來 AI 能力的綜合框架，提供跨不同 AI 供應商的一致抽象層。它與 Spring 生態系統無縫整合，是企業級 Java 應用需要 AI 能力的理想選擇。

Spring AI 的優勢在於與 Spring 生態系統的完美整合，讓你能運用依賴注入、配置管理與測試框架等熟悉的 Spring 模式快速構建可投入生產的 AI 應用。你將在第二章與第四章使用 Spring AI，建構同時利用 OpenAI 與模型上下文協議（MCP）的應用。

##### 模型上下文協議（MCP）

[模型上下文協議（Model Context Protocol, MCP）](https://modelcontextprotocol.io/) 是一個新興標準，使 AI 應用能安全地與外部資料來源及工具互動。MCP 提供標準化方式，讓 AI 模型取得上下文資訊並在應用中執行操作。

第四章中，你將建構一個簡單的 MCP 計算機服務，示範如何使用 Spring AI 實作模型上下文協議基礎，並展示基本工具整合與服務架構。

#### Azure OpenAI Java SDK

Azure OpenAI Java 客戶端函式庫是 OpenAI REST API 的 Java 版本改編，提供符合習慣的介面並且與 Azure SDK 生態系統整合。在第三章，你將使用 Azure OpenAI SDK 建構應用，包括聊天應用、函數調用及 RAG（檢索增強生成）模式。

> 注意：Azure OpenAI SDK 在功能上落後於 OpenAI Java SDK，未來專案建議考慮使用 OpenAI Java SDK。

## 總結

基礎知識介紹完畢！你現在了解：

- 生成式 AI 的核心概念 — 從大型語言模型、提示工程到標記、嵌入與向量數據庫
- Java AI 開發的工具選擇：Azure OpenAI SDK、Spring AI 和 OpenAI Java SDK
- 模型上下文協議是什麼，以及它如何使 AI 代理能與外部工具協作

## 接下來的步驟

[第二章：設置開發環境](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
本文件使用 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 進行翻譯。雖然我們力求準確，但請注意，自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應被視為權威來源。對於重要資訊，建議尋求專業人工翻譯。我們不對因使用本翻譯而引起的任何誤解或曲解承擔責任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->