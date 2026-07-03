# 為 Java 的生成式 AI 建立開發環境

> **快速入門：** 使用 Bicep + `azd` 以程式碼方式在 **Azure AI Foundry** 上佈建您的 AI 模型，幾分鐘內完成 — 請參閱 [Azure AI Foundry 安裝指南](getting-started-azure-openai.md)。驗證是<strong>無需金鑰</strong>（Microsoft Entra ID），所以無需管理 API 金鑰。

## 您將學到什麼

- 設置 AI 應用程式的 Java 開發環境
- 選擇並配置您喜好的開發環境（以雲端為主的 Codespaces、本地開發容器或完整本地設置）
- 透過連接 Azure AI Foundry 模型測試您的設置

## 目錄

- [您將學到什麼](#您將學到什麼)
- [介紹](#介紹)
- [步驟 1：設定開發環境](#步驟-1設定開發環境)
  - [選項 A：GitHub Codespaces（推薦）](#選項-agithub-codespaces推薦)
  - [選項 B：本地開發容器](#選項-b本地開發容器)
  - [選項 C：使用您現有的本地安裝](#選項-c使用您現有的本地安裝)
- [步驟 2：佈建 Azure AI Foundry](#步驟-2佈建-azure-ai-foundry)
- [步驟 3：測試您的設置](#步驟-3測試您的設置)
- [故障排除](#故障排除)
- [摘要](#摘要)
- [下一步](#下一步)

## 介紹

本章將引導您完成開發環境的設置。我們整個課程都將使用 **Azure AI Foundry** 提供的模型。您可利用 Bicep 與 Azure Developer CLI (`azd`) 以程式碼方式佈建模型，再透過 <strong>無需金鑰的身份驗證</strong>（Microsoft Entra ID）連接 — 無需複製或外洩 API 金鑰。

**不需要本地設置！** 您可以使用 GitHub Codespaces，該服務在瀏覽器中提供完整的開發環境，並可從那裡直接佈建 Foundry。

本課程選用 **Azure AI Foundry** 的原因是：
- <strong>以程式碼佈建</strong> — 一個 `azd up` 指令可部署帳戶和模型部署
- <strong>無需金鑰</strong> — 使用您的 Azure 登入或托管身份認證
- <strong>生產級</strong> — 相同程式碼可在本地與 Azure 執行
- <strong>靈活</strong> — 只需更改部署名稱即可切換模型，不必更動程式碼

> <strong>注意</strong>：Azure AI Foundry 部署依照令牌計費（隨用隨付）。請參閱 [Azure AI Foundry 安裝指南](getting-started-azure-openai.md) 了解佈建、區域及費用明細。


## 步驟 1：設定開發環境

<a name="quick-start-cloud"></a>

我們已建立預先配置的開發容器，減少設定時間並確保您擁有本生成式 Java AI 課程所需的所有工具。請選擇您偏好的開發方式：

### 環境設置選項：

#### 選項 A：GitHub Codespaces（推薦）

**2 分鐘內開始編碼 — 無需本地安裝！**

1. 將此儲存庫轉存（Fork）到您的 GitHub 帳戶
   > <strong>注意</strong>：如果您想編輯基本設定，請參考 [Dev Container 配置](../../../.devcontainer/devcontainer.json)
2. 點選 **Code** → **Codespaces** 分頁 → **...** → **New with options...**
3. 使用預設值 — 這會選擇本課程專用的 **Generative AI Java Development Environment** 開發容器配置
4. 點選 **Create codespace**
5. 等待大約 2 分鐘，環境即建立完成
6. 繼續前往 [步驟 2：佈建 Azure AI Foundry](#步驟-2佈建-azure-ai-foundry)

<img src="../../../translated_images/zh-TW/codespaces.9945ded8ceb431a5.webp" alt="截圖: Codespaces 子選單" width="50%">

<img src="../../../translated_images/zh-TW/image.833552b62eee7766.webp" alt="截圖: New with options" width="50%">

<img src="../../../translated_images/zh-TW/codespaces-create.b44a36f728660ab7.webp" alt="截圖: 建立 codespace 選項" width="50%">


> **Codespaces 優勢：**
> - 無需本地安裝
> - 適用於任何有瀏覽器的裝置
> - 預配置所有工具與相依性
> - 個人帳戶每月免費 60 小時
> - 提供所有學員一致的環境

#### 選項 B：本地開發容器

**適合喜歡使用 Docker 進行本地開發的開發人員**

1. 將此儲存庫 Fork 並克隆到您的本地機器
   > <strong>注意</strong>：如果您想編輯基本設定，請參考 [Dev Container 配置](../../../.devcontainer/devcontainer.json)
2. 安裝 [Docker Desktop](https://www.docker.com/products/docker-desktop/) 與 [VS Code](https://code.visualstudio.com/)
3. 在 VS Code 中安裝 [Dev Containers 擴充套件](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
4. 在 VS Code 中打開儲存庫資料夾
5. 當收到提示時，點擊 **Reopen in Container**（或使用 `Ctrl+Shift+P` → "Dev Containers: Reopen in Container"）
6. 等待容器建置並啟動完成
7. 繼續前往 [步驟 2：佈建 Azure AI Foundry](#步驟-2佈建-azure-ai-foundry)

<img src="../../../translated_images/zh-TW/devcontainer.21126c9d6de64494.webp" alt="截圖: 開發容器設置" width="50%">

<img src="../../../translated_images/zh-TW/image-3.bf93d533bbc84268.webp" alt="截圖: 開發容器建置完成" width="50%">

#### 選項 C：使用您現有的本地安裝

**適用於已有 Java 環境的開發人員**

先決條件：
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html) 
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) 或您偏好的 IDE

步驟：
1. 將此儲存庫克隆到您的本地機器
2. 在您的 IDE 中開啟專案
3. 繼續前往 [步驟 2：佈建 Azure AI Foundry](#步驟-2佈建-azure-ai-foundry)

> <strong>小秘訣</strong>：如果您的機器規格不高但想用本地 VS Code，請使用 GitHub Codespaces！您可以將本地 VS Code 連接到雲端託管的 Codespace，享受兩者優勢。

<img src="../../../translated_images/zh-TW/image-2.fc0da29a6e4d2aff.webp" alt="截圖: 建立的本地開發容器實例" width="50%">


## 步驟 2：佈建 Azure AI Foundry

以程式碼方式將本課程的 AI 模型部署到 Azure AI Foundry。於儲存庫根目錄執行：

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```

`azd` 將提示輸入環境名稱與區域，佈建包含 `gpt-4o-mini` 和 `text-embedding-3-small` 部署的 Azure AI Foundry 帳戶，並將端點寫入範例的 `.env` 檔案 — 全程採用 <strong>無需金鑰</strong> 驗證（無 API 金鑰）。

> **完整教學：** 請參閱 [Azure AI Foundry 安裝指南](getting-started-azure-openai.md)，了解先決條件、手動（入口網站）佈建替代方案、區域建議，以及費用與清理注意事項。

## 步驟 3：測試您的設置

一旦您佈建好 Foundry 模型，使用 [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) 中的示例應用程式來測試連線。

1. 在您的開發環境中開啟終端機。
2. 切換到該範例目錄：
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```

3. 確認您已登入（無需金鑰的驗證需要令牌）：
   ```bash
   az login
   ```
   > 如果您執行過 `azd up`，則 `.env` 檔案已自動寫入端點資訊。
4. 執行應用程式：
   ```bash
   mvn clean spring-boot:run
   ```

您應能看到來自 `gpt-4o-mini` 模型的回應。

### 理解示例程式碼

`examples/basic-chat-azure` 下面的示例是一個使用 **Spring AI** 並以無需金鑰認證連接 Azure AI Foundry 的 Spring Boot 應用程式。

**此程式碼功能：**
- <strong>連接</strong>到 Azure AI Foundry，使用您的 Azure 登入（Microsoft Entra ID）— 無需 API 金鑰
- <strong>傳送</strong>提示到 `gpt-4o-mini` 模型
- <strong>接收</strong>並顯示 AI 回應
- <strong>驗證</strong>您的設置是否正確

<strong>關鍵相依套件</strong>（`pom.xml`）：
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```

<strong>設定</strong>（`application.yml`）：
```yaml
spring:
  ai:
    azure:
      openai:
        # Endpoint only - no api-key. Spring AI uses DefaultAzureCredential (keyless).
        endpoint: ${AZURE_OPENAI_ENDPOINT}
        chat:
          options:
            deployment-name: ${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}
```

## 摘要

太棒了！您已完成所有設置：

- 使用 Bicep + `azd` 以程式碼佈建 Azure AI Foundry 模型
- 不論是 Codespaces、開發容器或本地環境，都已建置好您的 Java 開發環境
- 使用無需金鑰的驗證（Microsoft Entra ID）連接 Azure AI Foundry — 無需 API 金鑰
- 透過與模型對話的簡易範例測試一切運作正常

## 下一步

[第 3 章：核心生成式 AI 技術](../03-CoreGenerativeAITechniques/README.md)

## 故障排除

遇到問題嗎？以下是常見問題與解決方案：

- **驗證失敗（401/403）？**
  - 執行 `az login` — 驗證是無需金鑰的，您必須先登入
  - 確認您的帳戶具有資源上的 **Cognitive Services OpenAI User** 角色
  - 若剛佈建，請稍待片刻，讓角色指派生效

- **找不到 Maven？**
  - 使用開發容器或 Codespaces 時，Maven 應已預裝
  - 本地環境請確認安裝 Java 21+ 與 Maven 3.9+
  - 可執行 `mvn --version` 以驗證安裝狀況

- **找不到 `azd` 或佈建失敗？**
  - 安裝 [Azure Developer CLI](https://aka.ms/azure-dev/install) 並執行 `azd auth login`
  - 選擇支援 `gpt-4o-mini` 的區域（如 `eastus2`）
  - 詳情請參閱 [Azure AI Foundry 安裝指南](getting-started-azure-openai.md)

- **開發容器無法啟動？**
  - 確認本機執行 Docker Desktop（用於本地開發）
  - 嘗試重建容器：`Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **應用程式編譯錯誤？**
  - 確認当前目录为：`02-SetupDevEnvironment/examples/basic-chat-azure`
  - 嘗試清理並重新編譯：`mvn clean compile`

> **需要幫助？**：仍有問題嗎？請在儲存庫開啟 issue，我們將協助您解決。

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
此文件已使用 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 進行翻譯。雖然我們努力追求準確性，但請注意自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應視為權威來源。對於關鍵資訊，建議採用專業人工翻譯。我們不對因使用此翻譯所產生的任何誤解或誤譯承擔責任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->