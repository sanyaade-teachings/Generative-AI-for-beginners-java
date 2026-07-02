# 生成式人工智能入門 - Java 版

## 你將學到什麼

- <strong>生成式人工智能基礎知識</strong>，包括大型語言模型(LLM)、提示工程、詞元、嵌入及向量資料庫
- **比較 Java AI 開發工具**，包含 Azure OpenAI SDK、Spring AI 和 OpenAI Java SDK
- <strong>探索模型上下文協議</strong>及其在 AI 代理通信中的角色

## 目錄

- [介紹](#介紹)
- [生成式 AI 概念快速回顧](#生成式-ai-概念快速回顧)
- [提示工程回顧](#提示工程回顧)
- [詞元、嵌入與代理人](#詞元、嵌入與代理人)
- [Java 的 AI 開發工具與函式庫](#java-的-ai-開發工具與函式庫)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [總結](#總結)
- [後續步驟](#後續步驟)

## 介紹

歡迎來到生成式人工智能初學者 - Java 版的第一章！這個基礎課程將帶你認識生成式 AI 的核心概念及如何用 Java 操作它們。你將了解 AI 應用程序的基本構建模組，包括大型語言模型(LLM)、詞元、嵌入和 AI 代理人。我們同時也會探討你在整個課程中會用到的主要 Java 工具。

### 生成式 AI 概念快速回顧

生成式人工智能是一種基於從資料中學習的模式與關係，來創造新內容的人工智能，內容可能是文字、圖像或程式碼。生成式 AI 模型可以產生類似人類的回覆、理解上下文，有時甚至能創造出看似人類創作的內容。

在開發 Java AI 應用程式時，你會使用<strong>生成式 AI 模型</strong>來製作內容。生成式 AI 模型的一些能力包括：

- <strong>文字生成</strong>：為聊天機器人、內容創作與文字補完撰寫類人類文字。
- <strong>圖像生成與分析</strong>：製造真實感圖片、強化照片、偵測物體。
- <strong>程式碼生成</strong>：撰寫程式碼片段或腳本。

有專門為不同任務最佳化的模型，例如<strong>小型語言模型(SLM)</strong>與<strong>大型語言模型(LLM)</strong>都能處理文本生成，但 LLM 通常在複雜任務上表現更佳。針對影像相關的任務，會使用專門的視覺模型或多模態模型。

![Figure: Generative AI model types and use cases.](../../../translated_images/zh-HK/llms.225ca2b8a0d34473.webp)

當然，這些模型的回答並不總是完美無缺。你可能聽過模型「產生幻覺」或以權威方式輸出錯誤資訊。但你可以透過提供明確指示與上下文，協助模型產生更佳回答。這就是<strong>提示工程</strong>的用武之地。

#### 提示工程回顧

提示工程是設計有效輸入以引導 AI 模型達到期望輸出的實務。它包含：

- <strong>清晰度</strong>：指令要清楚且無歧義。
- <strong>上下文</strong>：提供必要背景資訊。
- <strong>限制條件</strong>：指定任何限制或格式要求。

提示工程的最佳實踐包括提示設計、清晰指示、任務拆解、一例學習與少量學習、提示調整。測試不同的提示對找出適合你特定用例的方式非常重要。

開發應用時，你會接觸到不同類型的提示：
- <strong>系統提示</strong>：設置模型行為的基本規則與上下文
- <strong>使用者提示</strong>：來自應用使用者的輸入資料
- <strong>助理提示</strong>：模型基於系統及使用者提示所作的回應

> <strong>深入了解</strong>：深入學習提示工程，請參閱[生成式 AI 初學者課程提示工程章節](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### 詞元、嵌入與代理人

使用生成式 AI 模型時，你會遇到<strong>詞元</strong>、<strong>嵌入</strong>、<strong>代理人</strong>與<strong>模型上下文協議(Model Context Protocol, MCP)</strong>等詞彙。以下是這些概念的詳細說明：

- <strong>詞元</strong>：詞元是模型中最小的文字單位，可以是字詞、字元或子詞。詞元用來以模型能理解的格式表示文字資料。例如句子 "The quick brown fox jumped over the lazy dog" 可能被切分成 ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] 或依切詞策略不同成為 ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"]。

![Figure: Generative AI tokens example of breaking words into tokens](../../../translated_images/zh-HK/tokens.6283ed277a2ffff4.webp)

詞元化(tokenization)是將文字拆解成這些更小單元的過程。這很重要，因為模型是基於詞元運作，而非原始文字。提示中的詞元數量會影響模型回應的長度與品質，因為模型對上下文視窗詞元有上限（例：GPT-4o 總上下文包含輸入及輸出詞元上限為128K）。

  在 Java 中，你可以使用像 OpenAI SDK 這樣的函式庫在向 AI 模型發送請求時自動處理詞元化。

- <strong>嵌入</strong>：嵌入是詞元的向量表示，能捕捉語意意義。它們是數值表示（通常是浮點數數組），幫助模型理解字詞間關聯並生成上下文相關回應。相似的字詞擁有相似嵌入，使模型能理解同義詞及語義關係。

![Figure: Embeddings](../../../translated_images/zh-HK/embedding.398e50802c0037f9.webp)

  在 Java 中，你可以透過 OpenAI SDK 或其他支援生成嵌入的函式庫製作嵌入。嵌入對於如語意搜尋等任務非常重要，當你想基於意義尋找相似內容而非準確文字匹配時尤其有用。

- <strong>向量資料庫</strong>：向量資料庫是優化儲存嵌入的專門系統。它們支援高效相似度搜尋，對於採用檢索增強生成(Retrieval-Augmented Generation, RAG)模式的場景非常關鍵，在此模式中你需要基於語義相似度從大量資料中找到相關資訊，而非簡單文字匹配。

![Figure: Vector database architecture showing how embeddings are stored and retrieved for similarity search.](../../../translated_images/zh-HK/vector.f12f114934e223df.webp)

> <strong>注意</strong>：本課程不會深入探討向量資料庫，但由於其在實務應用中相當普遍，因此值得一提。

- **代理人與 MCP**：代理人是能自主與模型、工具及外部系統互動的 AI 元件。模型上下文協議(Model Context Protocol，MCP)則提供標準化方式，讓代理人能安全地存取外部資料來源及工具。詳情請參考我們的[MCP 初學者課程](https://github.com/microsoft/mcp-for-beginners)。

在 Java AI 應用中，你會用詞元進行文字處理，使用嵌入完成語意搜尋及 RAG，利用向量資料庫做資料取得，並透過 MCP 代理人打造具有智能、可使用工具的系統。

![Figure: how a prompt becomes a reply—tokens, vectors, optional RAG lookup, LLM thinking, and an MCP agent all in one quick flow..](../../../translated_images/zh-HK/flow.f4ef62c3052d12a8.webp)

### Java 的 AI 開發工具與函式庫

Java 提供傑出的 AI 開發工具。整個課程將探討三大函式庫——OpenAI Java SDK、Azure OpenAI SDK 與 Spring AI。

以下是每章範例使用的 SDK 快速參考表：

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

OpenAI SDK 是 OpenAI API 的官方 Java 函式庫。它提供簡單且一致的介面與 OpenAI 模型互動，使得將 AI 能力整合到 Java 應用變得輕鬆。第 4 章的寵物故事應用和 Foundry Local 範例展示了 OpenAI SDK 與 Azure AI Foundry 的結合使用。

#### Spring AI

Spring AI 是一個提供 AI 能力給 Spring 應用的完整框架，在不同 AI 供應商間提供一致的抽象層。它與 Spring 生態無縫整合，是需要 AI 功能的企業級 Java 應用的理想選擇。

Spring AI 的強項在於與 Spring 生態系統緊密結合，讓你能用熟悉的 Spring 模式（如依賴注入、配置管理、測試框架）輕鬆構建可投入生產的 AI 應用。你會在第 2 章和第 4 章使用 Spring AI，打造同時可利用 OpenAI 與模型上下文協議(MCP)的 Spring AI 函式庫應用。

##### 模型上下文協議 (MCP)

[模型上下文協議 (MCP)](https://modelcontextprotocol.io/) 是一項新興標準，使 AI 應用能安全地與外部資料來源和工具互動。MCP 提供標準化方式讓 AI 模型取得上下文資訊並執行應用中動作。

第 4 章中，你將打造簡單的 MCP 計算器服務，示範使用 Spring AI 實現模型上下文協議的基礎，展示基本工具整合與服務架構的建立。

#### Azure OpenAI Java SDK

Azure OpenAI Java 用戶端函式庫是 OpenAI REST API 的 Java 封裝，提供 Azure SDK 生態系統風格的介面與整合。第 3 章你會使用 Azure OpenAI SDK 建構應用，其中包含聊天應用、函數呼叫及 RAG（檢索增強生成）模式。

> 注意：Azure OpenAI SDK 在功能上落後於 OpenAI Java SDK，因此未來專案建議考慮使用 OpenAI Java SDK。

## 總結

基礎部分到此結束！你現在已了解：

- 生成式 AI 的核心概念——從大型語言模型與提示工程，到詞元、嵌入與向量資料庫
- Java AI 開發的工具選項：Azure OpenAI SDK、Spring AI 和 OpenAI Java SDK
- 模型上下文協議的意義，以及它如何讓 AI 代理人與外部工具協作

## 後續步驟

[第 2 章：設定開發環境](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
本文件由 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 翻譯而成。雖然我們致力於確保準確性，但請注意，機器自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應被視為權威來源。對於重要資訊，建議進行專業人工翻譯。我們不對因使用本翻譯而產生的任何誤解或誤釋承擔責任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->