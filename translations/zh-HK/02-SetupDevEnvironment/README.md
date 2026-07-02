# 為 Java 設置生成式 AI 的開發環境

> **快速開始：** 使用 Bicep + `azd`，在幾分鐘內於 **Azure AI Foundry** 以代碼方式配置 AI 模型 — 請參見 [Azure AI Foundry 設置指南](getting-started-azure-openai.md)。身份驗證採用 **免 API key** 的方式（Microsoft Entra ID），無需管理 API 金鑰。

## 你將學到什麼

- 設置 AI 應用的 Java 開發環境
- 選擇並配置你偏好的開發環境（優先雲端的 Codespaces、本地開發容器，或完整本地設置）
- 通過連接 Azure AI Foundry 模型測試你的設置

## 目錄

- [你將學到什麼](#你將學到什麼)
- [介紹](#介紹)
- [步驟 1：設置你的開發環境](#步驟-1設置你的開發環境)
  - [選項 A：GitHub Codespaces（推薦）](#選項-agithub-codespaces推薦)
  - [選項 B：本地開發容器](#選項-b本地開發容器)
  - [選項 C：使用你現有的本地安裝](#選項-c使用你現有的本地安裝)
- [步驟 2：配置 Azure AI Foundry](#步驟-2配置-azure-ai-foundry)
- [步驟 3：測試你的設置](#步驟-3測試你的設置)
- [故障排除](#故障排除)
- [總結](#總結)
- [後續步驟](#後續步驟)

## 介紹

本章將引導你設置開發環境。在整個課程中，我們將使用 **Azure AI Foundry** 作為模型平台。你可以使用 Bicep 和 Azure 開發者 CLI (`azd`) 以代碼方式預配模型，然後使用 **免 API key** 的身份驗證（Microsoft Entra ID）連接 — 不需複製或洩露任何 API 金鑰。

**無需本地環境設置！** 你可以使用 GitHub Codespaces，在瀏覽器中獲得完整的開發環境並從那裡配置 Foundry。

我們選用 **Azure AI Foundry**，因為它具有：
- <strong>以代碼方式配置</strong> — 一次執行 `azd up` 即可完成帳戶和模型部署
- **免 API key** — 使用你的 Azure 登入或託管身份驗證
- <strong>生產等級</strong> — 同一組代碼可在本地或 Azure 執行
- <strong>靈活性</strong> — 只需更改部署名稱就能切換模型，無需更改代碼

> <strong>注意</strong>：Azure AI Foundry 部署按代幣計費（隨用隨付）。有關預配、地區和費用詳情，請參見 [Azure AI Foundry 設置指南](getting-started-azure-openai.md)。

## 步驟 1：設置你的開發環境

<a name="quick-start-cloud"></a>

我們創建了一個預配置的開發容器，以最小化設定時間並確保你擁有本生成式 Java AI 課程所需的所有工具。請選擇你偏好的開發方式：

### 環境設置選項：

#### 選項 A：GitHub Codespaces（推薦）

**2 分鐘即可開始編碼 — 無需本地安裝！**

1. 將本存儲庫 Fork 到你的 GitHub 帳戶  
   > <strong>注意</strong>：如果你想編輯基本配置，請查看 [開發容器配置](../../../.devcontainer/devcontainer.json)
2. 點擊 **Code** → **Codespaces** 標籤 → **...** → **New with options...**
3. 使用預設值 — 會選擇為此課程自定義的 **生成式 AI Java 開發環境 Dev 容器配置**
4. 點擊 **Create codespace**
5. 等待約 2 分鐘，環境即會準備好
6. 進入 [步驟 2：配置 Azure AI Foundry](#步驟-2配置-azure-ai-foundry)

<img src="../../../translated_images/zh-HK/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: Codespaces submenu" width="50%">

<img src="../../../translated_images/zh-HK/image.833552b62eee7766.webp" alt="Screenshot: New with options" width="50%">

<img src="../../../translated_images/zh-HK/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: Create codespace options" width="50%">


> **Codespaces 的好處**：
> - 無需本地安裝
> - 任何帶瀏覽器的設備均可使用
> - 預配置所有工具與依賴
> - 個人帳戶每月免費提供 60 小時使用
> - 為所有學習者提供一致的開發環境

#### 選項 B：本地開發容器

**適合偏好使用 Docker 本地開發的開發者**

1. 將本存儲庫 Fork 並克隆到本地機器  
   > <strong>注意</strong>：如果你想編輯基本配置，請查看 [開發容器配置](../../../.devcontainer/devcontainer.json)
2. 安裝 [Docker Desktop](https://www.docker.com/products/docker-desktop/) 與 [VS Code](https://code.visualstudio.com/)
3. 在 VS Code 中安裝 [Dev Containers 擴展](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
4. 使用 VS Code 開啟存儲庫資料夾
5. 出現提示時，點擊 **Reopen in Container**（或使用 `Ctrl+Shift+P` → 「Dev Containers: Reopen in Container」）
6. 等待容器構建及啟動
7. 進入 [步驟 2：配置 Azure AI Foundry](#步驟-2配置-azure-ai-foundry)

<img src="../../../translated_images/zh-HK/devcontainer.21126c9d6de64494.webp" alt="Screenshot: Dev container setup" width="50%">

<img src="../../../translated_images/zh-HK/image-3.bf93d533bbc84268.webp" alt="Screenshot: Dev container build complete" width="50%">

#### 選項 C：使用你現有的本地安裝

**適合已有 Java 環境的開發者**

前提條件：  
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html)  
- [Maven 3.9+](https://maven.apache.org/download.cgi)  
- [VS Code](https://code.visualstudio.com) 或你偏好的 IDE

步驟：  
1. 克隆本存儲庫到本地機器  
2. 在你的 IDE 中開啟此專案  
3. 進入 [步驟 2：配置 Azure AI Foundry](#步驟-2配置-azure-ai-foundry)

> <strong>小技巧</strong>：如果你的電腦硬體規格較低，但想要在本地使用 VS Code，可以使用 GitHub Codespaces！你可以將本地的 VS Code 連接到雲端託管的 Codespace，兩邊優點兼得。

<img src="../../../translated_images/zh-HK/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: created local devcontainer instance" width="50%">


## 步驟 2：配置 Azure AI Foundry

將本課程的 AI 模型以代碼形式部署到 Azure AI Foundry。於存儲庫根目錄執行：

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```
  
`azd` 會提示輸入環境名稱及地區，並預配一個 Azure AI Foundry 帳戶及 `gpt-4o-mini` 與 `text-embedding-3-small` 部署，並將端點寫入範例的 `.env` 檔 — 全程使用 **免 API key** 的身份驗證。

> **完整操作指南：** 請參閱 [Azure AI Foundry 設置指南](getting-started-azure-openai.md)，了解先決條件、手動(門戶)方式、地區建議及費用清理說明。

## 步驟 3：測試你的設置

當你的 Foundry 模型部署完成後，使用 [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) 中的範例應用測試連接。

1. 在你的開發環境中開啟終端機。  
2. 移動至範例資料夾：  
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
  
3. 確認你已登入（免 API key 認證需要令牌）：  
   ```bash
   az login
   ```
  
   > 若你已執行 `azd up`，範例的 `.env` 檔案會自動寫入你的端點。  
4. 執行應用：  
   ```bash
   mvn clean spring-boot:run
   ```
  
你應該會看到來自 `gpt-4o-mini` 模型的回應。

### 理解範例程式碼

`examples/basic-chat-azure` 下的範例是使用 **Spring Boot** 並利用 **Spring AI** 庫以免 API key 身份驗證連接 Azure AI Foundry 的應用。

**此程式碼實現：**  
- 使用 Azure 登入（Microsoft Entra ID）連接 Azure AI Foundry — 無需 API Key  
- 向 `gpt-4o-mini` 模型發送提示語  
- 接收並顯示 AI 回應  
- 驗證你的設置是否正確

<strong>關鍵依賴</strong>（`pom.xml` 中）：  
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```
  
<strong>配置</strong>（`application.yml`）：  
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

太好了！你現在已完成所有設置：

- 使用 Bicep + `azd` 以代碼方式配置 Azure AI Foundry 模型  
- 啟動你的 Java 開發環境（無論是 Codespaces、開發容器或本地）  
- 使用免 API key 身份驗證（Microsoft Entra ID）成功連接 Azure AI Foundry  
- 用簡單範例驗證模型連線成功並可正常使用

## 後續步驟

[第 3 章：核心生成式 AI 技術](../03-CoreGenerativeAITechniques/README.md)

## 故障排除

遇到問題？以下是常見問題與解決方法：

- **身份驗證失敗（401/403）？**  
  - 執行 `az login` — 身份驗證採免 API key，必須登入  
  - 確認你的帳戶擁有該資源的 **Cognitive Services OpenAI User** 角色  
  - 若剛配置完成，稍等片刻讓角色分配生效

- **找不到 Maven？**  
  - 使用開發容器或 Codespaces 時 Maven 應該已預安裝  
  - 本地設置需確保 Java 21+ 與 Maven 3.9+ 已安裝  
  - 嘗試執行 `mvn --version` 驗證是否安裝成功

- **找不到 `azd` 或預配失敗？**  
  - 安裝 [Azure Developer CLI](https://aka.ms/azure-dev/install)，並執行 `azd auth login`  
  - 選擇 `gpt-4o-mini` 可用的地區（例如 `eastus2`）  
  - 詳見 [Azure AI Foundry 設置指南](getting-started-azure-openai.md)

- **開發容器無法啟動？**  
  - 確保 Docker Desktop 正在運行（本地開發）  
  - 嘗試重建容器：使用 `Ctrl+Shift+P` → 「Dev Containers: Rebuild Container」

- **應用編譯錯誤？**  
  - 確認所在路徑正確：`02-SetupDevEnvironment/examples/basic-chat-azure`  
  - 試試執行 `mvn clean compile` 清理並重新編譯

> **需要幫助？** 若仍遇問題，請於本存儲庫提出 Issue，我們會協助你解決。

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
本文件由 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 翻譯而成。雖然我們致力於確保準確性，但請注意，機器自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應被視為權威來源。對於重要資訊，建議進行專業人工翻譯。我們不對因使用本翻譯而產生的任何誤解或誤釋承擔責任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->