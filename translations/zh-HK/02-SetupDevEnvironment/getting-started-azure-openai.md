# 設置 Azure AI Foundry 開發環境

> 本指南設置本課程中的 Java AI 應用程式所使用的 **Azure AI Foundry** 模型，採用 <strong>無金鑰</strong> 驗證（Microsoft Entra ID）— 無須管理 API 金鑰。對工具不熟悉？請從[開發環境指南](./README.md)開始。

本指南設置本課程中的 **Azure AI Foundry** 模型，用於 Java AI 應用程式。你有兩個選擇：

- **選項 A — 使用 `azd` + Bicep 進行部署（推薦）：** 一條命令以代碼方式部署 Foundry 帳戶和模型。無需點擊入口網站。
- **選項 B — 在 Azure AI Foundry 入口網站手動建立資源。**

兩條路徑都使用 <strong>無金鑰驗證</strong>（Microsoft Entra ID） — 不需複製或洩漏 API 金鑰。

## 目錄

- [建立了什麼](#建立了什麼)
- [前置條件](#前置條件)
- [選項 A：使用 azd + Bicep 部署（推薦）](#option-a-provision-with-azd--bicep-recommended)
- [選項 B：手動建立資源](#選項-b：手動建立資源)
- [配置你的環境](#配置你的環境)
- [測試你的設置](#測試你的設置)
- [下一步是什麼？](#下一步是什麼？)
- [資源](#資源)
- [額外資源](#額外資源)

## 建立了什麼

[`infra/`](../../../02-SetupDevEnvironment/infra) 中的 Bicep 範本會部署：

- 一個 **Azure AI Foundry** 帳戶（`Microsoft.CognitiveServices/accounts`，類型 `AIServices`）及一個專案
- 一個 <strong>聊天</strong> 部署 — `gpt-4o-mini`
- 一個 <strong>嵌入式</strong> 部署 — `text-embedding-3-small`（用於後面章節）
- 一個 <strong>無金鑰角色指派</strong>（`Cognitive Services OpenAI User`），讓你用 `az login` 登入而非管理金鑰

## 前置條件

- 一個 [Azure 訂閱](https://azure.microsoft.com/free/)
- [Azure 開發者 CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) 和 [Maven 3.9+](https://maven.apache.org/download.cgi)

## 選項 A：使用 azd + Bicep 部署（推薦）

在 `02-SetupDevEnvironment` 資料夾中：

```bash
cd 02-SetupDevEnvironment

# 登入（兩個工具）
azd auth login
az login

# 配置 Foundry 帳戶 + 模型部署
azd up
```

`azd` 會提示輸入 <strong>環境名稱</strong>（如 `genai-java`）和 <strong>區域</strong>。請選擇支援 `gpt-4o-mini` 和 `text-embedding-3-small` 的區域 — 例如 `eastus2` 或 `swedencentral`。

部署完成後，azd：

1. 部署 [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep) 中定義的所有資源。
2. 執行部署後掛勾，將你的端點與部署名稱寫入 [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure)（不含任何秘密資訊）。

> **小提示：** 之後任何時候重新執行 `azd up` 以套用變更。執行 `azd down` 可刪除所有資源並停止產生費用。

查看生成的設定：

```bash
azd env get-values
```

現在跳至 [測試你的設置](#測試你的設置)。

## 選項 B：手動建立資源

偏好入口網站操作？請手動建立資源：

1. 前往 [Azure AI Foundry 入口網站](https://ai.azure.com/) 並登入。
2. <strong>建立新專案</strong>（同時建立 AI Foundry 資源）。命名為 `GenAIJava` 等。
3. 在專案中，開啟 **模型 + 端點** → <strong>部署模型</strong> → <strong>部署基礎模型</strong>。
4. 部署 **gpt-4o-mini**（部署名稱為 `gpt-4o-mini`）。如需使用嵌入示例，也部署 **text-embedding-3-small**。
5. 從 <strong>概覽</strong> 複製 <strong>端點</strong>（例如 `https://<resource>.openai.azure.com/`）。
6. 賦予自己無金鑰存取權限：在資源中，開啟 **存取控制 (IAM)** → <strong>新增角色指派</strong> → 指派 **Cognitive Services OpenAI User** 給你的帳戶。

> **仍有問題？** 請參閱 [Azure AI Foundry 文件](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects)。

## 配置你的環境

**如果使用選項 A（`azd up`）**，你的設定檔已完成寫入 — 無須額外配置。直接跳至 [測試你的設置](#測試你的設置)。

**如果使用選項 B（手動）**，請自行建立範例的 `.env` 檔案：

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

編輯 `.env` 並填入你的端點（無需金鑰 — 授權為無金鑰）：

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **安全提示：** 沒有 API 金鑰需要儲存。你透過 `az login`（本地端）或 Azure 的管理身分驗證（managed identity）使用 Microsoft Entra ID 進行驗證。`.env` 檔案只含非秘密設定，且已被 `.gitignore` 排除。

## 測試你的設置

請確認你已登入以便無金鑰授權能取得存取權杖，然後執行範例：

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # 如果你尚未登入
mvn clean spring-boot:run
```

你應該會看到來自 `gpt-4o-mini` 模型的回應！

> **VS Code 使用者：** 按 `F5` 執行。應用程式會自動載入你的 `.env`。

> **完整範例：** 詳情及故障排除請參見 [使用 Azure AI Foundry 的基本聊天範例](./examples/basic-chat-azure/README.md)。

## 下一步是什麼？

**設定完成！** 你現在擁有：
- 部署有 `gpt-4o-mini` 和 `text-embedding-3-small` 的 Azure AI Foundry
- 無金鑰授權（Microsoft Entra ID）— 無須管理金鑰
- 含有端點和部署名稱的本地 `.env`
- 準備就緒的 Java 開發環境

<strong>繼續前往</strong> [第三章：核心生成式 AI 技術](../03-CoreGenerativeAITechniques/README.md) 開始建構 AI 應用！

## 資源

- [Azure 開發者 CLI (azd)](https://aka.ms/azure-dev/install)
- [透過 Microsoft Entra ID 進行無金鑰驗證](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry 文件](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI 文件](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## 額外資源

- [下載 VS Code](https://code.visualstudio.com/Download)
- [取得 Docker Desktop](https://www.docker.com/products/docker-desktop)
- [開發容器配置](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
本文件由 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 翻譯而成。雖然我們致力於確保準確性，但請注意，機器自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應被視為權威來源。對於重要資訊，建議進行專業人工翻譯。我們不對因使用本翻譯而產生的任何誤解或誤釋承擔責任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->