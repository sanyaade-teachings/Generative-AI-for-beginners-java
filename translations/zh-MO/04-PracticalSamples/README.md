# 實際應用程式與專案

## 您將學到什麼
在本節中，我們將示範三個展示使用 Java 的生成式 AI 開發模式的實際應用程式：
- 創建一個結合客戶端與伺服器端 AI 的多模態寵物故事生成器
- 使用 Foundry Local Spring Boot 範例實作本地 AI 模型整合
- 使用計算器範例開發模型上下文協議 (MCP) 服務

## 內容目錄

- [介紹](#介紹)
  - [Foundry Local Spring Boot 範例](#foundry-local-spring-boot-範例)
  - [寵物故事生成器](#寵物故事生成器)
  - [MCP 計算器服務（適合初學者的 MCP 範例）](#mcp-計算器服務適合初學者的-mcp-範例)
- [學習進程](#學習進程)
- [總結](#總結)
- [下一步](#下一步)

## 介紹

本章展示示範使用 Java 生成式 AI 開發模式的<strong>範例專案</strong>。每個專案皆具備完整功能，示範特定的 AI 技術、架構模式與最佳實踐，您可將其調整應用於自己的應用程式中。

### Foundry Local Spring Boot 範例

**[Foundry Local Spring Boot 範例](foundrylocal/README.md)** 展示如何使用 **OpenAI Java SDK** 整合本地 AI 模型。該範例示範連接執行於 Foundry Local 上的模型（例如 **Phi-4-mini**），具備自動模型偵測功能，讓您在不依賴雲端服務的情況下執行 AI 應用。

### 寵物故事生成器

**[寵物故事生成器](petstory/README.md)** 是一個有趣的 Spring Boot 網頁應用程式，示範如何透過<strong>多模態 AI 處理</strong>來產生創意寵物故事。它結合客戶端與伺服器端的 AI 功能，使用 transformer.js 於瀏覽器端進行 AI 互動，並使用 OpenAI SDK 於伺服器端進行處理。

### MCP 計算器服務（適合初學者的 MCP 範例）

**[MCP 計算器服務](calculator/README.md)** 是一個利用 Spring AI 簡單示範<strong>模型上下文協議（MCP）</strong>的範例。它提供初學者友善的 MCP 介紹，展示如何建立基本的 MCP 伺服器，以與 MCP 客戶端互動。

## 學習進程

這些專案設計用來延續前面章節的概念：

1. <strong>從簡單開始</strong>：先從 Foundry Local Spring Boot 範例開始，理解使用本地模型的 AI 基本整合
2. <strong>增加互動性</strong>：進階到寵物故事生成器，體驗多模態 AI 與網頁互動
3. **學習 MCP 基礎**：嘗試 MCP 計算器服務，了解模型上下文協議的基本原理

## 總結

做得好！您目前已探索了一些實際應用：

- 同時能在瀏覽器與伺服器運作的多模態 AI 體驗
- 使用現代 Java 框架與 SDK 進行本地 AI 模型整合
- 您的第一個模型上下文協議服務，了解工具如何整合 AI

## 下一步

[第5章：負責任的生成式 AI](../05-ResponsibleGenAI/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
本文件使用 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 進行翻譯。雖然我們力求準確，但請注意，自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應被視為權威來源。對於重要資訊，建議尋求專業人工翻譯。我們不對因使用本翻譯而引起的任何誤解或曲解承擔責任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->