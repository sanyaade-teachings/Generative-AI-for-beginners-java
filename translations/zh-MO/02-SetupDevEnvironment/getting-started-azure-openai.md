# 設定 Azure AI Foundry 的開發環境

> 本指南使用 **keyless** 認證（Microsoft Entra ID）為本課程中的 Java AI 應用程式設置 **Azure AI Foundry** 模型 — 無需管理 API 金鑰。對此工具不熟悉？請從[開發環境指南](./README.md)開始。

本指南為本課程中的 Java AI 應用程式設置 **Azure AI Foundry** 模型。你有兩條途徑：

- **方案 A — 使用 `azd` + Bicep 進行佈建（推薦）：** 一條命令以程式碼形式部署 Foundry 帳戶和模型。無需點擊入口網站。
- **方案 B — 在 Azure AI Foundry 入口網站手動建立資源**。

兩條途徑都使用 **keyless 認證**（Microsoft Entra ID）— 無需複製或洩漏 API 金鑰。

## 目錄

- [建立了哪些資源](#建立了哪些資源)
- [先決條件](#先決條件)
- [方案 A：使用 azd + Bicep 佈建（推薦）](#option-a-provision-with-azd--bicep-recommended)
- [方案 B：手動建立資源](#方案-b：手動建立資源)
- [配置你的環境](#配置你的環境)
- [測試你的設置](#測試你的設置)
- [接下來呢？](#接下來呢？)
- [資源](#資源)
- [更多資源](#更多資源)

## 建立了哪些資源

[`infra/`](../../../02-SetupDevEnvironment/infra) 中的 Bicep 範本會佈建：

- 一個 **Azure AI Foundry** 帳戶（`Microsoft.CognitiveServices/accounts`，種類為 `AIServices`）及一個專案
- 一個 <strong>聊天</strong> 部署 — `gpt-4o-mini`
- 一個 <strong>嵌入</strong> 部署 — `text-embedding-3-small`（後續章節會用到）
- 一個 **keyless 角色指派**（`Cognitive Services OpenAI User`），讓你用 `az login` 登入，而非管理金鑰

## 先決條件

- 一個 [Azure 訂閱](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) 與 [Maven 3.9+](https://maven.apache.org/download.cgi)

## 方案 A：使用 azd + Bicep 佈建（推薦）

在 `02-SetupDevEnvironment` 資料夾中：

```bash
cd 02-SetupDevEnvironment

# 登入（兩種工具）
azd auth login
az login

# 設置 Foundry 帳戶及模型部署
azd up
```

`azd` 會提示輸入 <strong>環境名稱</strong>（例如 `genai-java`）和 <strong>區域</strong>。請選擇支援 `gpt-4o-mini` 與 `text-embedding-3-small` 的區域，例如 `eastus2` 或 `swedencentral`。

完成佈建後，azd：

1. 部署 [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep) 中定義的所有資源。
2. 執行後置佈建掛鉤，將你的端點和部署名稱寫入 [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure)（無秘密資訊）。

> **提示：** 任何時候可重新執行 `azd up` 以應用變更。執行 `azd down` 刪除所有並停用計費。

查看產生的設定：

```bash
azd env get-values
```

現在跳至[測試你的設置](#測試你的設置)。

## 方案 B：手動建立資源

偏好使用入口網站？請手動建立資源：

1. 前往 [Azure AI Foundry 入口網站](https://ai.azure.com/) 並登入。
2. <strong>建立專案</strong>（同時會建立 AI Foundry 資源）。為專案命名，例如 `GenAIJava`。
3. 在專案中，打開 **Models + endpoints** → **Deploy model** → **Deploy base model**。
4. 部署 **gpt-4o-mini**（部署名稱為 `gpt-4o-mini`）。如需嵌入範例，請重複部署 **text-embedding-3-small**。
5. 在 **Overview** 中複製 **endpoint**（例如 `https://<resource>.openai.azure.com/`）。
6. 授予自己 keyless 存取權：在資源上打開 **Access control (IAM)** → **Add role assignment** → 指派 **Cognitive Services OpenAI User** 權限給你的帳戶。

> **仍有問題？** 請參考 [Azure AI Foundry 文件](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects)。

## 配置你的環境

**如果你使用方案 A（`azd up`）**，你的設定檔已自動完成，無需額外設定。請直接跳到[測試你的設置](#測試你的設置)。

**如果你使用方案 B（手動）**，請自行建立範例的 `.env` 檔案：

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

以你的端點編輯 `.env`（無需金鑰，因為認證是 keyless）：

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **安全提示：** 無 API 金鑰需儲存。你透過 `az login`（本機）或 Azure 中的管理身分驗證進行 Microsoft Entra ID 認證。`.env` 檔案僅包含非祕密設定，且已在 `.gitignore` 中保護。

## 測試你的設置

請確保已登入，讓 keyless 認證能取得 token，然後執行範例：

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # 如果你尚未登入
mvn clean spring-boot:run
```

你應該會看到來自 `gpt-4o-mini` 模型的回應！

> **VS Code 使用者：** 按 `F5` 運行。應用程式會自動載入你的 `.env`。

> **完整範例：** 詳情及疑難排解請參閱 [與 Azure AI Foundry 的 Basic Chat 範例](./examples/basic-chat-azure/README.md)。

## 接下來呢？

**設置完成！** 你現在擁有：
- 部署好的 Azure AI Foundry，包含 `gpt-4o-mini` 與 `text-embedding-3-small`
- Keyless 認證（Microsoft Entra ID）— 無需管理金鑰
- 包含你的端點及部署名稱的本地 `.env`
- 準備好的 Java 開發環境

<strong>繼續前往</strong> [第3章：核心生成式 AI 技術](../03-CoreGenerativeAITechniques/README.md) 開始建立 AI 應用程式！

## 資源

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [使用 Microsoft Entra ID 的 keyless 認證](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry 文件](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI 文件](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## 更多資源

- [下載 VS Code](https://code.visualstudio.com/Download)
- [下載 Docker Desktop](https://www.docker.com/products/docker-desktop)
- [開發容器設定](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
本文件使用 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 進行翻譯。雖然我們力求準確，但請注意，自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應被視為權威來源。對於重要資訊，建議尋求專業人工翻譯。我們不對因使用本翻譯而引起的任何誤解或曲解承擔責任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->