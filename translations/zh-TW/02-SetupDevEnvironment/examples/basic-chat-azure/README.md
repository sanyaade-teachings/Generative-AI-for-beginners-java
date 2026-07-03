# 基本聊天與 Azure AI Foundry - 端到端範例

此範例是一個簡單的 Spring Boot 應用程式，使用 **keyless 認證**（Microsoft Entra ID）連接到 **Azure AI Foundry** 模型並測試您的設定。它使用 Spring AI 的 `ChatClient`。

## 目錄

- [前置條件](#前置條件)
- [快速開始](#快速開始)
- [認證如何運作](#認證如何運作)
- [執行應用程式](#執行應用程式)
  - [使用 Maven](#使用-maven)
  - [使用 VS Code](#使用-vs-code)
  - [預期輸出](#預期輸出)
- [設定參考](#設定參考)
  - [環境變數](#環境變數)
  - [Spring 設定](#spring-設定)
- [故障排除](#故障排除)
  - [常見問題](#常見問題)
  - [除錯模式](#除錯模式)
- [後續步驟](#後續步驟)
- [資源](#資源)

## 前置條件

在執行此範例之前，請確保您已擁有：

- 具備 `gpt-4o-mini` 部署的 Azure AI Foundry 資源 — 使用 `azd up` 或手動透過 [Azure AI Foundry 安裝指南](../../getting-started-azure-openai.md) 建立
- 該資源的 **認知服務 OpenAI 使用者** 角色（Bicep 範本會自動分配）
- 已使用 `az login` 登入的 [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- Java 21+ 與 Maven 3.9+

> **不需 API 金鑰** — 認證透過 Microsoft Entra ID 以無金鑰方式進行。

## 快速開始

```bash
# 1. 導航至專案
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. 登入以便無金鑰認證能取得令牌
az login

# 3. 配置端點
#    - 如果你執行過 `azd up`，.env 檔案已為你寫好（可跳過此步驟）。
#    - 否則請複製範本並設定 AZURE_OPENAI_ENDPOINT：
cp .env.example .env

# 4. 執行應用程式
mvn spring-boot:run
```

## 認證如何運作

此範例使用 **Microsoft Entra ID** 進行認證 — 無需 API 金鑰。

當僅設定 `spring.ai.azure.openai.endpoint`（且未指定 api-key）時，Spring AI 使用 [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential) 建立 Azure OpenAI 用戶端。該憑證會自動從您本地的 `az login` 工作階段取得存取權杖，或在 Azure 執行時從管理身分識別 (Managed Identity) 取得，因此同一份程式碼可在兩者間無需變更地運作。

## 執行應用程式

### 使用 Maven

```bash
mvn spring-boot:run
```

### 使用 VS Code

1. 在 VS Code 開啟專案
2. 按下 `F5` 或使用「執行與除錯」面板
3. 選擇「Spring Boot-BasicChatApplication」設定

> <strong>注意</strong>：VS Code 設定會自動載入您的 .env 檔案

### 預期輸出

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

## 設定參考

### 環境變數

| 變數 | 說明 | 必填 | 範例 |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | Foundry（Azure OpenAI）端點 URL | 是 | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | 聊天模型部署名稱 | 否 | `gpt-4o-mini`（預設） |

> 無 API 金鑰變數 — 認證採用無金鑰方式（透過 `az login` 使用 Microsoft Entra ID）。

### Spring 設定

`application.yml` 檔案配置：
- <strong>端點</strong>：`${AZURE_OPENAI_ENDPOINT}` - 取自環境變數
- <strong>部署</strong>：`${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - 取自環境變數並有預設值
- <strong>認證</strong>：無金鑰 — 未設定 `api-key`，Spring AI 使用 `DefaultAzureCredential`
- <strong>溫度</strong>：`0.7` - 控制創意度（0.0 = 確定性，1.0 = 創意）
- <strong>最大令牌數</strong>：`500` - 最大回應長度

## 故障排除

### 常見問題

<details>
<summary><strong>錯誤：401 / "PermissionDenied" / 權杖錯誤</strong></summary>

- 執行 `az login` — 無金鑰認證須有有效登入以取得權杖
- 確認您的帳戶在資源有 **認知服務 OpenAI 使用者** 角色
- 角色剛指派後，請等待一分鐘讓權限生效
- 確認您處於正確的租戶/訂閱 (`az account show`)
</details>

<details>
<summary><strong>錯誤："The endpoint is not valid" / 連線錯誤</strong></summary>

- 確認 `AZURE_OPENAI_ENDPOINT` 為完整基本 URL（例如 `https://your-resource.openai.azure.com/`）
- 檢查尾端斜線的一致性
- 驗證端點與您已有的資源相符 (`azd env get-values`)
</details>

<details>
<summary><strong>錯誤："The deployment was not found"</strong></summary>

- 確認 `AZURE_OPENAI_DEPLOYMENT` 與 Azure 部署名稱一致
- 檢查模型已成功部署且處於啟用狀態
- 預設部署名稱為 `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code：環境變數未載入</strong></summary>

- 確認您的 `.env` 檔置於專案根目錄（與 `pom.xml` 同層級）
- 嘗試在 VS Code 內建終端機執行 `mvn spring-boot:run`
- 確認 VS Code Java 延伸套件已安裝完畢
</details>

### 除錯模式

欲啟用詳細日誌，請解除註解 `application.yml` 中這些行：

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## 後續步驟

**設定完成！** 繼續您的學習旅程：

[第 3 章：核心生成式 AI 技術](../../../03-CoreGenerativeAITechniques/README.md)

## 資源

- [Spring AI Azure OpenAI 文件](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [使用 Microsoft Entra ID 進行無金鑰認證](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry 入口網站](https://ai.azure.com/)
- [Azure AI Foundry 文件](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
此文件已使用 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 進行翻譯。雖然我們努力追求準確性，但請注意自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應視為權威來源。對於關鍵資訊，建議採用專業人工翻譯。我們不對因使用此翻譯所產生的任何誤解或誤譯承擔責任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->