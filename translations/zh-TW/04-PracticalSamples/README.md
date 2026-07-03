# 實作應用與專案

## 你將學到什麼
在本節中，我們將示範三個展示使用 Java 的生成式 AI 開發模式的實作應用：
- 建立結合客戶端和伺服器端 AI 的多模態寵物故事產生器
- 使用 Foundry Local Spring Boot 範例實作本地 AI 模型整合
- 透過計算機範例開發 Model Context Protocol (MCP) 服務

## 目錄

- [介紹](#介紹)
  - [Foundry Local Spring Boot 範例](#foundry-local-spring-boot-範例)
  - [寵物故事產生器](#寵物故事產生器)
  - [MCP 計算機服務（適合初學者的 MCP 範例）](#mcp-計算機服務適合初學者的-mcp-範例)
- [學習進度](#學習進度)
- [總結](#總結)
- [下一步](#下一步)

## 介紹

本章展示了展示使用 Java 的生成式 AI 開發模式的<strong>範例專案</strong>。每個專案皆為全功能，展示特定 AI 技術、架構模式和最佳實務，你可以將其用於自己的應用程式。

### Foundry Local Spring Boot 範例

**[Foundry Local Spring Boot 範例](foundrylocal/README.md)** 演示如何使用 **OpenAI Java SDK** 整合本地 AI 模型。展示連接在 Foundry Local 運行的模型（例如 **Phi-4-mini**），並具備自動模型偵測功能，讓你無須依賴雲端服務即可執行 AI 應用。

### 寵物故事產生器

**[寵物故事產生器](petstory/README.md)** 是個有趣的 Spring Boot 網頁應用，展示了用於生成創意寵物故事的<strong>多模態 AI 處理</strong>。它結合客戶端與伺服器端 AI 功能，使用 transformer.js 實現瀏覽器端的 AI 互動，並用 OpenAI SDK 處理伺服器端作業。

### MCP 計算機服務（適合初學者的 MCP 範例）

**[MCP 計算機服務](calculator/README.md)** 是使用 Spring AI 演示 **Model Context Protocol (MCP)** 的簡單範例。提供適合初學者的 MCP 概念入門，展示如何建立基本 MCP 伺服器與 MCP 用戶端進行互動。

## 學習進度

這些專案設計為建立在先前章節的概念之上：

1. <strong>由簡入深</strong>：從 Foundry Local Spring Boot 範例開始理解本地模型的基本 AI 整合
2. <strong>增加互動性</strong>：進階到寵物故事產生器，體驗多模態 AI 與基於網頁的互動
3. **學習 MCP 基礎**：嘗試 MCP 計算機服務以理解 Model Context Protocol 的基本原理

## 總結

做得好！你現在已經探索了一些實際應用：

- 在瀏覽器與伺服器雙重運作的多模態 AI 體驗
- 使用現代 Java 框架與 SDK 進行本地 AI 模型整合
- 第一個 Model Context Protocol 服務，了解工具如何整合 AI

## 下一步

[第 5 章：負責任的生成式 AI](../05-ResponsibleGenAI/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
此文件已使用 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 進行翻譯。雖然我們努力追求準確性，但請注意自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應視為權威來源。對於關鍵資訊，建議採用專業人工翻譯。我們不對因使用此翻譯所產生的任何誤解或誤譯承擔責任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->