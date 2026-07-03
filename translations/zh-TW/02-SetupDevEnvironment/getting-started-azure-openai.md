# 設定 Azure AI Foundry 的開發環境

> 本指南協助您設定本課程中 Java AI 應用所使用的 **Azure AI Foundry** 模型，採用 <strong>無金鑰</strong> 驗證（Microsoft Entra ID）— 無需管理 API 金鑰。剛接觸這套工具？請先參閱[開發環境指南](./README.md)。

本指南協助您為本課程的 Java AI 應用設定 **Azure AI Foundry** 模型。您有兩種方式：

- **方案 A — 使用 `azd` + Bicep 佈建（推薦）：** 一條指令就能以程式碼部署 Foundry 帳戶與模型。無需點擊入口網站。
- **方案 B — 手動在 Azure AI Foundry 入口網站建立資源。**

兩種方式皆使用 <strong>無金鑰驗證</strong>（Microsoft Entra ID）— 您不需複製或洩漏任何 API 金鑰。

## 目錄

- [建立了什麼](#建立了什麼)
- [先決條件](#先決條件)
- [方案 A：使用 azd + Bicep 佈建（推薦）](#option-a-provision-with-azd--bicep-recommended)
- [方案 B：手動建立資源](#方案-b：手動建立資源)
- [設定您的環境](#設定您的環境)
- [測試您的設定](#測試您的設定)
- [接下來？](#接下來？)
- [資源](#資源)
- [額外資源](#額外資源)

## 建立了什麼

`infra/` 目錄下的 Bicep 模板會佈建：

- 一個 **Azure AI Foundry** 帳戶（`Microsoft.CognitiveServices/accounts`，種類為 `AIServices`）及一個專案
- 一個 <strong>聊天</strong> 部署 — `gpt-4o-mini`
- 一個 <strong>嵌入</strong> 部署 — `text-embedding-3-small`（後續章節會使用）
- 一個 <strong>無金鑰角色指派</strong>（`Cognitive Services OpenAI User`），讓您可使用 `az login` 登入，無需管理金鑰

## 先決條件

- 一個[Azure 訂閱](https://azure.microsoft.com/free/)
- [Azure 開發者 CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) 與 [Maven 3.9+](https://maven.apache.org/download.cgi)

## 方案 A：使用 azd + Bicep 佈建（推薦）

在 `02-SetupDevEnvironment` 資料夾中執行：

```bash
cd 02-SetupDevEnvironment

# 登入（兩個工具）
azd auth login
az login

# 配置 Foundry 帳戶和模型部署
azd up
```

`azd` 會要求您輸入<strong>環境名稱</strong>（例如 `genai-java`）與<strong>區域</strong>。請選擇 `gpt-4o-mini` 及 `text-embedding-3-small` 可用的區域—例如 `eastus2` 或 `swedencentral`。

佈建完成後，azd 會：

1. 部署 [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep) 中定義的所有資源。
2. 執行 postprovision 腳本，將您的端點與部署名稱寫入 [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure)（不含機密）。

> **提示：** 任何時候重新執行 `azd up` 以套用變更。執行 `azd down` 可刪除所有資源且停止產生費用。

要查看產生的設定檔：

```bash
azd env get-values
```

接著請跳至 [測試您的設定](#測試您的設定)。

## 方案 B：手動建立資源

偏好使用入口網站？請手動建立資源：

1. 前往 [Azure AI Foundry 入口網站](https://ai.azure.com/) 並登入。
2. <strong>建立專案</strong>（這也會建立 AI Foundry 資源）。命名例如 `GenAIJava`。
3. 在專案中開啟 **Models + endpoints** → **Deploy model** → **Deploy base model**。
4. 部署 **gpt-4o-mini**（部署名稱為 `gpt-4o-mini`）。如果想要使用嵌入範例，請重複部署 **text-embedding-3-small**。
5. 從 **Overview** 複製 <strong>端點</strong>（如 `https://<resource>.openai.azure.com/`）。
6. 賦予自己無金鑰存取權限：在該資源中，打開 **Access control (IAM)** → **Add role assignment** → 指派 **Cognitive Services OpenAI User** 給您的帳戶。

> **仍有問題嗎？** 請參閱 [Azure AI Foundry 文件](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects)。

## 設定您的環境

**如果您使用方案 A（`azd up`）**，設定檔已寫好 — 無需設定，直接跳至 [測試您的設定](#測試您的設定)。

**如果您使用方案 B（手動）**，請自行建立範例的 `.env` 檔案：

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

以您的端點編輯 `.env`（無需金鑰 — 驗證為無金鑰）：

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **安全性提醒：** 不需儲存 API 金鑰。您以 Microsoft Entra ID 透過 `az login`（在本機）或受管理身分（在 Azure）進行驗證。`.env` 僅存放非機密設定，且已列入 `.gitignore`。

## 測試您的設定

確定您已登入以便無金鑰驗證獲取令牌，然後執行範例：

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # 如果你還沒有登入
mvn clean spring-boot:run
```

您應該會看到來自 `gpt-4o-mini` 模型的回應！

> **VS Code 使用者：** 按 `F5` 執行。應用程式會自動載入您的 `.env`。

> **完整範例：** 請參閱 [Azure AI Foundry 的基本聊天範例](./examples/basic-chat-azure/README.md) 了解詳情與疑難排解。

## 接下來？

**設定完成！** 您現在擁有：
- 已佈建 `gpt-4o-mini` 與 `text-embedding-3-small` 的 Azure AI Foundry
- 無金鑰驗證（Microsoft Entra ID）— 無需管理金鑰
- 包含您端點與部署名稱的本機 `.env`
- 就緒的 Java 開發環境

<strong>繼續閱讀</strong> [第 3 章：核心生成式 AI 技術](../03-CoreGenerativeAITechniques/README.md) 開始建置 AI 應用程式！

## 資源

- [Azure 開發者 CLI (azd)](https://aka.ms/azure-dev/install)
- [使用 Microsoft Entra ID 的無金鑰驗證](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry 文件](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI 文件](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## 額外資源

- [下載 VS Code](https://code.visualstudio.com/Download)
- [取得 Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Dev Container 設定](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
此文件已使用 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 進行翻譯。雖然我們努力追求準確性，但請注意自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應視為權威來源。對於關鍵資訊，建議採用專業人工翻譯。我們不對因使用此翻譯所產生的任何誤解或誤譯承擔責任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->