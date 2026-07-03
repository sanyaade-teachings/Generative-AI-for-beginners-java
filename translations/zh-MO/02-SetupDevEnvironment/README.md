# 為 Java 設定生成式 AI 的開發環境

> **快速入門：** 用 Bicep + `azd` 在幾分鐘內以程式碼方式在 **Azure AI Foundry** 上佈建您的 AI 模型 — 請參閱 [Azure AI Foundry 設定指南](getting-started-azure-openai.md)。身份驗證採用 <strong>無金鑰</strong> (Microsoft Entra ID)，無需管理 API 金鑰。

## 您將學習到的內容

- 建立 AI 應用程式的 Java 開發環境
- 選擇並設定您偏好的開發環境（以 Codespaces 雲端開發為主、在本地使用開發容器或完整本地安裝）
- 透過連接到 Azure AI Foundry 模型測試您的設定

## 目錄

- [您將學習到的內容](#您將學習到的內容)
- [介紹](#介紹)
- [步驟 1：設定您的開發環境](#步驟-1設定您的開發環境)
  - [選項 A：GitHub Codespaces（推薦）](#選項-agithub-codespaces推薦)
  - [選項 B：本地開發容器](#選項-b本地開發容器)
  - [選項 C：使用您現有的本地安裝](#選項-c使用您現有的本地安裝)
- [步驟 2：佈建 Azure AI Foundry](#步驟-2佈建-azure-ai-foundry)
- [步驟 3：測試您的設定](#步驟-3測試您的設定)
- [故障排除](#故障排除)
- [總結](#總結)
- [下一步](#下一步)

## 介紹

本章將引導您設定開發環境。在整個課程中，我們將使用 **Azure AI Foundry** 作為模型。您可以使用 Bicep 和 Azure Developer CLI (`azd`) 以程式碼方式佈建模型，然後以 <strong>無金鑰身份驗證</strong>（Microsoft Entra ID）連接 — 無需複製或洩露 API 金鑰。

**無需本地設定！** 您可以使用 GitHub Codespaces，該服務能在瀏覽器中提供完整的開發環境，並直接從那裡佈建 Foundry。

我們使用 **Azure AI Foundry** 的原因是：
- <strong>以程式碼方式佈建</strong> — 一個 `azd up` 就能部署帳戶和模型部署
- <strong>無金鑰</strong> — 使用 Azure 登入或受控身份驗證
- <strong>生產就緒</strong> — 同一套程式碼可在本地及 Azure 運行
- <strong>靈活性高</strong> — 只需改變部署名稱即可切換模型，無需修改程式碼

> <strong>注意</strong>：Azure AI Foundry 部署以 token 計費（按使用付費）。佈建、區域及成本詳情請參閱 [Azure AI Foundry 設定指南](getting-started-azure-openai.md)。

## 步驟 1：設定您的開發環境

<a name="quick-start-cloud"></a>

我們已建立預先配置的開發容器，以減少設定時間並確保您擁有本生成式 AI for Java 課程所需的所有工具。請選擇您偏好的開發方式：

### 環境設定選項：

#### 選項 A：GitHub Codespaces（推薦）

**兩分鐘開始編碼 — 無需本地設定！**

1. 將此存放庫 fork 至您的 GitHub 帳戶  
   > <strong>注意</strong>：如果您想編輯基本設定，請參閱 [開發容器設定](../../../.devcontainer/devcontainer.json)  
2. 點擊 **Code** → **Codespaces** 分頁 → **...** → **使用選項新建...**  
3. 使用預設選項 — 這會選擇為本課程建立的 **生成式 AI Java 開發環境** 自訂 devcontainer  
4. 點擊 **建立 Codespace**  
5. 等待約 2 分鐘，環境即會準備好  
6. 繼續進行 [步驟 2：佈建 Azure AI Foundry](#步驟-2-佈建-azure-ai-foundry)

<img src="../../../translated_images/zh-MO/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: Codespaces submenu" width="50%">

<img src="../../../translated_images/zh-MO/image.833552b62eee7766.webp" alt="Screenshot: New with options" width="50%">

<img src="../../../translated_images/zh-MO/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: Create codespace options" width="50%">

> **Codespaces 的優點**：  
> - 無需本地安裝  
> - 任何具有瀏覽器的裝置皆可使用  
> - 預配置所有工具和相依  
> - 個人帳戶免費 60 小時/月  
> - 為所有學員提供一致的環境

#### 選項 B：本地開發容器

**適合偏好本地使用 Docker 開發的開發者**

1. 將此存放庫 fork 並克隆至本地機器  
   > <strong>注意</strong>：如果您想編輯基本設定，請參閱 [開發容器設定](../../../.devcontainer/devcontainer.json)  
2. 安裝 [Docker Desktop](https://www.docker.com/products/docker-desktop/) 及 [VS Code](https://code.visualstudio.com/)  
3. 在 VS Code 中安裝 [Dev Containers 擴充功能](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)  
4. 在 VS Code 開啟存放庫資料夾  
5. 當出現提示時，點擊 <strong>在容器中重新開啟</strong>（或使用 `Ctrl+Shift+P` →「Dev Containers: Reopen in Container」）  
6. 等待容器建置並啟動完成  
7. 繼續進行 [步驟 2：佈建 Azure AI Foundry](#步驟-2-佈建-azure-ai-foundry)

<img src="../../../translated_images/zh-MO/devcontainer.21126c9d6de64494.webp" alt="Screenshot: Dev container setup" width="50%">

<img src="../../../translated_images/zh-MO/image-3.bf93d533bbc84268.webp" alt="Screenshot: Dev container build complete" width="50%">

#### 選項 C：使用您現有的本地安裝

**適合已擁有 Java 環境的開發者**

先決條件：  
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html)  
- [Maven 3.9+](https://maven.apache.org/download.cgi)  
- [VS Code](https://code.visualstudio.com) 或您偏好的 IDE

步驟：  
1. 將此存放庫複製至本地機器  
2. 在您的 IDE 中開啟專案  
3. 繼續進行 [步驟 2：佈建 Azure AI Foundry](#步驟-2-佈建-azure-ai-foundry)

> <strong>專業提示</strong>：若機器規格低，但想在本地使用 VS Code，可使用 GitHub Codespaces！您可讓本地 VS Code 連接至雲端 Codespace，兩者兼得。

<img src="../../../translated_images/zh-MO/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: created local devcontainer instance" width="50%">

## 步驟 2：佈建 Azure AI Foundry

將本課程所用的 AI 模型以程式碼方式部署到 Azure AI Foundry。於存放庫根目錄執行：

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```
  
`azd` 會提示輸入環境名稱和區域，並佈建帶有 `gpt-4o-mini` 和 `text-embedding-3-small` 部署的 Azure AI Foundry 帳戶，並將終端點寫入範例的 `.env` 檔案中 — 全程採用 <strong>無金鑰</strong> 認證 (無需 API 金鑰)。

> **完整操作說明：** 請參閱 [Azure AI Foundry 設定指南](getting-started-azure-openai.md) 了解先決條件、手動（入口網站）替代方案、區域指南與成本/清理說明。

## 步驟 3：測試您的設定

模型佈建完成後，使用 [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) 範例應用程式測試連線。

1. 在開發環境開啟終端機。  
2. 進入範例資料夾：  
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
3. 確認您已登入（無金鑰認證需有權杖）：  
   ```bash
   az login
   ```
   > 若您執行過 `azd up`，已自動為您寫入含終端點的 `.env` 檔案。  
4. 執行應用程式：  
   ```bash
   mvn clean spring-boot:run
   ```
  
您應該會看到來自 `gpt-4o-mini` 模型的回應。

### 瞭解範例程式碼

`examples/basic-chat-azure` 下的範例是一個 Spring Boot 應用程式，使用 **Spring AI** 透過無金鑰認證連接到 Azure AI Foundry。

**此程式碼的功能：**  
- 使用您的 Azure 登入（Microsoft Entra ID）連線到 Azure AI Foundry — 無需 API 金鑰  
- 對 `gpt-4o-mini` 模型發送提示  
- 接收並顯示 AI 回應  
- 驗證您的設定是否正確運作

<strong>關鍵相依性</strong>（位於 `pom.xml`）：  
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```
  
<strong>設定</strong>（位於 `application.yml`）：  
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


## 總結

太好了！您現在已完成所有設定：

- 使用 Bicep + `azd` 以程式碼方式佈建 Azure AI Foundry 模型  
- 啟動了 Java 開發環境（無論是 Codespaces、開發容器或本地）  
- 使用無金鑰認證（Microsoft Entra ID）連接 Azure AI Foundry — 無需 API 金鑰  
- 以簡單範例測試成功連接並與模型互動

## 下一步

[第 3 章：核心生成式 AI 技巧](../03-CoreGenerativeAITechniques/README.md)

## 故障排除

遇到問題？以下是常見問題及解決方案：

- **身份驗證失敗（401/403）？**  
  - 執行 `az login` — 身份驗證為無金鑰，必須登入  
  - 確認帳戶在資源上具有 **Cognitive Services OpenAI User** 角色  
  - 如果剛佈建，請等候一分鐘讓角色分配生效

- **找不到 Maven？**  
  - 若使用開發容器/Codespaces，Maven 應已預先安裝  
  - 本地設定須安裝 Java 21+ 和 Maven 3.9+  
  - 嘗試 `mvn --version` 以確認安裝

- **找不到 `azd` 或佈建失敗？**  
  - 安裝 [Azure Developer CLI](https://aka.ms/azure-dev/install) 並執行 `azd auth login`  
  - 選擇可用 `gpt-4o-mini` 的區域（例如 `eastus2`）  
  - 詳情請參閱 [Azure AI Foundry 設定指南](getting-started-azure-openai.md)

- **開發容器無法啟動？**  
  - 確認 Docker Desktop 正在執行（針對本地開發）  
  - 嘗試重新建置容器：`Ctrl+Shift+P` →「Dev Containers: Rebuild Container」

- **應用程式編譯錯誤？**  
  - 確認目前目錄正確：`02-SetupDevEnvironment/examples/basic-chat-azure`  
  - 嘗試清理並編譯：`mvn clean compile`

> **需要協助？**：問題仍無法解決？在存放庫開啟 issue，我們會協助您。

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
本文件使用 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 進行翻譯。雖然我們力求準確，但請注意，自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應被視為權威來源。對於重要資訊，建議尋求專業人工翻譯。我們不對因使用本翻譯而引起的任何誤解或曲解承擔責任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->