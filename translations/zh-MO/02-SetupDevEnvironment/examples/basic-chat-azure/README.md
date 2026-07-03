# 使用 Azure AI Foundry 的基本聊天 - 端到端範例

此範例是一個簡單的 Spring Boot 應用程式，連接到使用 **keyless 驗證**（Microsoft Entra ID）的 **Azure AI Foundry** 模型，並測試您的設定。它使用 Spring AI 的 `ChatClient`。

## 目錄

- [先決條件](#先決條件)
- [快速開始](#快速開始)
- [驗證機制說明](#驗證機制說明)
- [執行應用程式](#執行應用程式)
  - [使用 Maven](#使用-maven)
  - [使用 VS Code](#使用-vs-code)
  - [預期輸出](#預期輸出)
- [設定參考](#設定參考)
  - [環境變數](#環境變數)
  - [Spring 配置](#spring-配置)
- [疑難排解](#疑難排解)
  - [常見問題](#常見問題)
  - [偵錯模式](#偵錯模式)
- [後續步驟](#後續步驟)
- [資源](#資源)

## 先決條件

在執行此範例之前，請確保您已具備：

- 具有 `gpt-4o-mini` 部署的 Azure AI Foundry 資源 — 可透過 `azd up` 或手動方式，依照 [Azure AI Foundry 設定指南](../../getting-started-azure-openai.md) 進行佈建
- 該資源上的 **Cognitive Services OpenAI User** 角色（Bicep 模板會自動分配）
- 已登入的 [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)，使用 `az login`
- Java 21+ 和 Maven 3.9+

> **免 API 金鑰** — 驗證是透過 Microsoft Entra ID 的無鑰方式進行。

## 快速開始

```bash
# 1. 前往項目
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. 登入以便無鑰匙驗證可以獲取憑證
az login

# 3. 配置端點
#    - 如果你執行過 `azd up`，已經為你寫入了.env（跳過此步）。
#    - 否則複製範本並設定 AZURE_OPENAI_ENDPOINT：
cp .env.example .env

# 4. 運行應用程式
mvn spring-boot:run
```

## 驗證機制說明

此範例透過 **Microsoft Entra ID** 進行驗證 — 無需 API 金鑰。

當只設定 `spring.ai.azure.openai.endpoint`（且無 api-key）時，Spring AI 會使用 [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential) 建構 Azure OpenAI 用戶端。該憑證會自動從您本地的 `az login` 工作階段找到權杖，或在 Azure 中執行時使用管理員身分認證取得權杖 — 因此同一段程式碼可無須變更，即可在不同環境下執行。

## 執行應用程式

### 使用 Maven

```bash
mvn spring-boot:run
```

### 使用 VS Code

1. 在 VS Code 開啟專案
2. 按下 `F5` 或使用「執行與除錯」面板
3. 選擇「Spring Boot-BasicChatApplication」組態

> <strong>注意</strong>：VS Code 組態會自動載入您的 .env 檔案

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
|------|-------|------|-------|
| `AZURE_OPENAI_ENDPOINT` | Foundry（Azure OpenAI）端點 URL | 是 | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | 聊天模型部署名稱 | 否 | `gpt-4o-mini`（預設） |

> 無 API 金鑰變數 — 授權採用無鑰方式（透過 `az login` 的 Microsoft Entra ID）。

### Spring 配置

`application.yml` 檔案配置：
- <strong>端點</strong>：`${AZURE_OPENAI_ENDPOINT}` - 從環境變數取得
- <strong>部署</strong>：`${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - 從環境變數取得，並有後備預設
- <strong>驗證</strong>：無鑰 — 未設定 `api-key`，Spring AI 使用 `DefaultAzureCredential`
- <strong>溫度</strong>：`0.7` - 創意控制（0.0 = 決定性，1.0 = 創意）
- **最大 Token 數**：`500` - 最大回應長度

## 疑難排解

### 常見問題

<details>
<summary><strong>錯誤：401 / "PermissionDenied" / 權杖錯誤</strong></summary>

- 執行 `az login` — 無鑰驗證需有效登入以取得權杖
- 確認您的帳戶在資源上擁有 **Cognitive Services OpenAI User** 角色
- 若剛分配角色，請稍待一分鐘以完成角色生效
- 確認您使用的是正確租戶/訂閱（使用 `az account show`）
</details>

<details>
<summary><strong>錯誤："The endpoint is not valid" / 連線錯誤</strong></summary>

- 確認 `AZURE_OPENAI_ENDPOINT` 為完整的基本 URL（例如 `https://your-resource.openai.azure.com/`）
- 檢查結尾斜線是否一致
- 確認端點與您佈建的資源吻合（使用 `azd env get-values`）
</details>

<details>
<summary><strong>錯誤："The deployment was not found"</strong></summary>

- 確認 `AZURE_OPENAI_DEPLOYMENT` 與 Azure 中部署名稱一致
- 確認模型已成功部署且為啟用狀態
- 預設部署名稱為 `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code：環境變數未載入</strong></summary>

- 確認 `.env` 檔案位於專案根目錄（與 `pom.xml` 同目錄）
- 嘗試在 VS Code 的整合終端機執行 `mvn spring-boot:run`
- 確認 VS Code Java 擴充套件已正確安裝
</details>

### 偵錯模式

欲啟用詳細日誌，請在 `application.yml` 中取消註解以下行：

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
- [使用 Microsoft Entra ID 的無鑰驗證](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry 入口網站](https://ai.azure.com/)
- [Azure AI Foundry 文件](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
本文件使用 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 進行翻譯。雖然我們力求準確，但請注意，自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應被視為權威來源。對於重要資訊，建議尋求專業人工翻譯。我們不對因使用本翻譯而引起的任何誤解或曲解承擔責任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->