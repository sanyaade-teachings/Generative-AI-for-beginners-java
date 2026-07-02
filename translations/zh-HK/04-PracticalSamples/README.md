# 實用應用與專案

## 你將學到什麼
在本節中，我們將展示三個展示 Java 生成式 AI 開發模式的實用應用：
- 創建一個結合客戶端與伺服器端 AI 的多模態寵物故事生成器
- 使用 Foundry Local Spring Boot 範例實現本地 AI 模型整合
- 開發一個帶有計算器範例的模型上下文協議（MCP）服務

## 目錄

- [介紹](#介紹)
  - [Foundry Local Spring Boot 範例](#foundry-local-spring-boot-範例)
  - [寵物故事生成器](#寵物故事生成器)
  - [MCP 計算器服務（適合初學者的 MCP 範例）](#mcp-計算器服務（適合初學者的-mcp-範例）)
- [學習進度](#學習進度)
- [總結](#總結)
- [後續步驟](#後續步驟)

## 介紹

本章展示了展示 Java 生成式 AI 開發模式的<strong>範例專案</strong>。每個專案皆為完整功能，展示特定 AI 技術、架構模式及最佳實踐，您可以將其調整並應用於自己的應用程式。

### Foundry Local Spring Boot 範例

**[Foundry Local Spring Boot 範例](foundrylocal/README.md)** 展示如何使用 **OpenAI Java SDK** 整合本地 AI 模型。範例示範連接執行於 Foundry Local (例如 **Phi-4-mini**) 的模型，並具備自動模型偵測功能，讓您無需依賴雲端服務即可執行 AI 應用程式。

### 寵物故事生成器

**[寵物故事生成器](petstory/README.md)** 是一個有趣的 Spring Boot 網頁應用，演示了 **多模態 AI 處理** 以生成富創意的寵物故事。該專案結合了客戶端與伺服器端 AI 能力，使用 transformer.js 實現瀏覽器端 AI 互動，並利用 OpenAI SDK 進行伺服器端處理。

### MCP 計算器服務（適合初學者的 MCP 範例）

**[MCP 計算器服務](calculator/README.md)** 是一個簡單的 **模型上下文協議（MCP）** 示範，採用 Spring AI 開發。它提供一個易於入門的 MCP 概念介紹，展示如何建立與 MCP 用戶端互動的基本 MCP 伺服器。

## 學習進度

這些專案設計以建立在前幾章的概念之上：

1. <strong>從簡單開始</strong>：從 Foundry Local Spring Boot 範例開始，了解本地模型的 AI 基本整合
2. <strong>增加互動性</strong>：進階到寵物故事生成器，體驗多模態 AI 和基於網頁的互動
3. **學習 MCP 基礎**：嘗試 MCP 計算器服務，理解模型上下文協議的基本原理

## 總結

做得好！您現在已探索了一些真實應用：

- 在瀏覽器與伺服器端皆可運作的多模態 AI 體驗
- 使用現代 Java 框架與 SDK 的本地 AI 模型整合
- 第一個模型上下文協議服務，了解工具如何與 AI 結合

## 後續步驟

[Chapter 5: Responsible Generative AI](../05-ResponsibleGenAI/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
本文件由 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 翻譯而成。雖然我們致力於確保準確性，但請注意，機器自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應被視為權威來源。對於重要資訊，建議進行專業人工翻譯。我們不對因使用本翻譯而產生的任何誤解或誤釋承擔責任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->