# 使用 Azure AI Foundry 進行基本聊天 - 端到端範例

此範例為一個簡單的 Spring Boot 應用程式，透過 <strong>無金鑰驗證</strong>（Microsoft Entra ID）連線至 **Azure AI Foundry** 模型，並測試您的設定。它使用 Spring AI 的 `ChatClient`。

## 目錄

- [先決條件](#先決條件)
- [快速開始](#快速開始)
- [驗證原理](#驗證原理)
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

## 先決條件

在執行此範例之前，請確保您已具備：

- 一個擁有 `gpt-4o-mini` 部署的 Azure AI Foundry 資源 — 可使用 `azd up` 或手動透過[Azure AI Foundry 設定指南](../../getting-started-azure-openai.md)佈署
- 該資源的 **Cognitive Services OpenAI User** 角色（Bicep 範本會自動為您指派）
- 已登入的 [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)，使用 `az login`
- Java 21 以上版本與 Maven 3.9 以上

> **不需要 API 金鑰** — 授權採用 Microsoft Entra ID 的無金鑰方式。

## 快速開始

```bash
# 1. 導航至專案
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. 登入以便無密碼身份驗證可以取得令牌
az login

# 3. 配置端點
#    - 如果你執行了 `azd up`，已為你寫入 .env（跳過此步）。
#    - 否則複製範本並設定 AZURE_OPENAI_ENDPOINT：
cp .env.example .env

# 4. 執行應用程式
mvn spring-boot:run
```

## 驗證原理

此範例使用 **Microsoft Entra ID** 驗證 — 無需 API 金鑰。

當只設定 `spring.ai.azure.openai.endpoint`（且未設置 api-key）時，Spring AI 會用 [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential) 建立 Azure OpenAI 客戶端。該認證會自動從您本地的 `az login` 會話取得權杖，或在 Azure 上執行時從管理身份取得，因此相同程式碼可在兩者間無需更改地運作。

## 執行應用程式

### 使用 Maven

```bash
mvn spring-boot:run
```

### 使用 VS Code

1. 在 VS Code 開啟專案
2. 按下 `F5` 或使用「執行與除錯」面板
3. 選擇 "Spring Boot-BasicChatApplication" 設定

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

| 變數 | 說明 | 是否必須 | 範例 |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | Foundry（Azure OpenAI）終端網址 | 是 | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | 聊天模型的部署名稱 | 否 | `gpt-4o-mini`（預設） |

> 不存在 API 金鑰變數 — 授權採無金鑰模式（Microsoft Entra ID，透過 `az login`）。

### Spring 設定

`application.yml` 檔案設定：
- **Endpoint**：`${AZURE_OPENAI_ENDPOINT}` - 來自環境變數
- **Deployment**：`${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - 來自環境變數，並有預設值
- <strong>驗證</strong>：無金鑰 — 未設定 `api-key`，Spring AI 會使用 `DefaultAzureCredential`
- <strong>溫度參數</strong>：`0.7` - 控制創意度（0.0 = 確定性，1.0 = 創意）
- <strong>最大標記數</strong>：`500` - 最大回應長度

## 故障排除

### 常見問題

<details>
<summary><strong>錯誤：401 / "PermissionDenied" / 權杖錯誤</strong></summary>

- 執行 `az login` — 無金鑰認證需要有效登入才能取得權杖
- 確認您的帳戶擁有該資源的 **Cognitive Services OpenAI User** 角色
- 若剛指派角色，請等待約一分鐘讓權限生效
- 確認您所在的租戶/訂閱正確（使用 `az account show`）
</details>

<details>
<summary><strong>錯誤："The endpoint is not valid" / 連線錯誤</strong></summary>

- 確保 `AZURE_OPENAI_ENDPOINT` 是完整的基底網址（如 `https://your-resource.openai.azure.com/`）
- 檢查末端斜線是否一致
- 驗證終端是否與您的佈署資源相符（使用 `azd env get-values`）
</details>

<details>
<summary><strong>錯誤："The deployment was not found"</strong></summary>

- 確認 `AZURE_OPENAI_DEPLOYMENT` 與 Azure 中的部署名稱吻合
- 檢查模型是否成功佈署且處於啟用狀態
- 預設部署名稱為 `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code：環境變數未載入</strong></summary>

- 確認您的 `.env` 檔案在專案根目錄（與 `pom.xml` 同層）
- 嘗試在 VS Code 內建終端執行 `mvn spring-boot:run`
- 確認已正確安裝 VS Code 的 Java 擴充套件
</details>

### 除錯模式

若要開啟詳細日誌，在 `application.yml` 中取消以下行的註解：

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## 後續步驟

**設定完成！** 繼續您的學習旅程：

[第三章：核心生成式 AI 技術](../../../03-CoreGenerativeAITechniques/README.md)

## 資源

- [Spring AI Azure OpenAI 文件](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [使用 Microsoft Entra ID 的無金鑰認證](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry 入口網站](https://ai.azure.com/)
- [Azure AI Foundry 文件](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
本文件由 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 翻譯而成。雖然我們致力於確保準確性，但請注意，機器自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應被視為權威來源。對於重要資訊，建議進行專業人工翻譯。我們不對因使用本翻譯而產生的任何誤解或誤釋承擔責任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->