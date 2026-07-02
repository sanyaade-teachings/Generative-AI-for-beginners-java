# AGENTS.md

## 專案概覽

這是一個用於學習使用 Java 進行生成式 AI 開發的教育性資源庫。它提供了一個涵蓋大型語言模型（LLMs）、提示工程、嵌入向量、RAG（檢索增強生成）以及模型上下文協議（MCP）的完整實作課程。

**關鍵技術：**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry、Azure OpenAI 與 OpenAI SDKs

**架構：**
- 按章節組織的多個獨立 Spring Boot 應用程式
- 展示不同 AI 模式的範例專案
- 具備 GitHub Codespaces 支援，預先配置好開發容器

## 安裝指令

### 先決條件
- Java 21 或更高版本
- Maven 3.x
- 具有 Azure AI Foundry 模型部署的 Azure 訂閱（使用 `azd up` 進行佈署）
- Azure CLI (`az`) 及 Azure Developer CLI (`azd`)，已登入以進行無密鑰身份驗證

### 環境設定

**方案 1：GitHub Codespaces（推薦）**
```bash
# 從 GitHub UI 分叉此存儲庫並建立一個代碼空間
# 開發容器會自動安裝所有依賴項
# 等待約 2 分鐘以完成環境設置
```

**方案 2：本地開發容器**
```bash
# 克隆儲存庫
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# 使用 Dev Containers 擴充功能在 VS Code 中開啟
# 出現提示時重新在容器中開啟
```

**方案 3：本地設定**
```bash
# 安裝依賴項
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# 驗證安裝
java -version
mvn -version
```

### 配置

**Azure AI Foundry 設定（無密鑰，推薦）：**
```bash
# 將 Foundry 帳戶和模型部署以程式碼方式配置
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd 使用您的端點（無金鑰）寫入 examples/basic-chat-azure/.env
```

**手動端點配置：**
```bash
# 如果您沒有使用 azd，請自行設定端點（透過 az login 認證保持無金鑰狀態）
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# 編輯 .env：AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## 開發工作流程

### 專案結構
```
/
├── 01-IntroToGenAI/              # Chapter 1: Introduction
├── 02-SetupDevEnvironment/       # Chapter 2: Environment setup
│   └── examples/                 # Working examples
├── 03-CoreGenerativeAITechniques/ # Chapter 3: Core techniques
├── 04-PracticalSamples/          # Chapter 4: Sample projects
│   ├── calculator/               # MCP service example
│   ├── foundrylocal/            # Local model integration
│   └── petstory/                # Multi-modal app
├── 05-ResponsibleGenAI/         # Chapter 5: Responsible AI
└── translations/                # Multi-language support
```

### 執行應用程式

**執行 Spring Boot 應用程式：**
```bash
cd [project-directory]
mvn spring-boot:run
```

**建置專案：**
```bash
cd [project-directory]
mvn clean install
```

**啟動 MCP 計算伺服器：**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# 伺服器運行於 http://localhost:8080
```

**執行客戶端範例：**
```bash
# 在另一個終端啟動伺服器後
cd 04-PracticalSamples/calculator

# 直接 MCP 用戶端
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# AI 驅動的用戶端（需要 AZURE_OPENAI_ENDPOINT + az 登入）
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# 互動式機器人
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### 熱重載
支持熱重載的專案已包含 Spring Boot DevTools：
```bash
# 對 Java 檔案的更改在儲存時會自動重新加載
mvn spring-boot:run
```

## 測試說明

### 執行測試

**執行專案中所有測試：**
```bash
cd [project-directory]
mvn test
```

**以詳細模式執行測試：**
```bash
mvn test -X
```

**執行指定的測試類別：**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### 測試結構
- 測試檔案使用 JUnit 5（Jupiter）
- 測試類別位於 `src/test/java/`
- 計算器專案中的客戶端示例位於 `src/test/java/com/microsoft/mcp/sample/client/`

### 手動測試
許多範例為互動式應用程式，需手動測試：

1. 使用 `mvn spring-boot:run` 啟動應用程式
2. 測試端點或與 CLI 互動
3. 確認預期行為與各專案 README.md 文件描述相符

### 使用 Azure AI Foundry 進行測試
- 執行範例前請先執行 `az login` 登入（無密鑰驗證）
- 確保帳號具備 Cognitive Services OpenAI User 角色
- 使用第 5 章中的負責任 AI 範例進行內容篩選測試

## 程式碼風格指南

### Java 規範
- **Java 版本：** Java 21，並採用現代語法功能
- **風格：** 遵循標準 Java 規範
- **命名：** 
  - 類別：PascalCase
  - 方法/變數：camelCase
  - 常數：UPPER_SNAKE_CASE
  - 套件名稱：小寫

### Spring Boot 模式
- 使用 `@Service` 實作商業邏輯
- 使用 `@RestController` 建置 REST 端點
- 配置透過 `application.yml` 或 `application.properties`
- 優先使用環境變數，代替硬編碼數值
- 對外暴露 MCP 方法使用 `@Tool` 註解

### 檔案組織
```
src/
├── main/
│   ├── java/
│   │   └── com/microsoft/[component]/
│   │       ├── [Component]Application.java
│   │       ├── config/
│   │       ├── controller/
│   │       ├── service/
│   │       └── exception/
│   └── resources/
│       ├── application.yml
│       └── static/
└── test/
    └── java/
        └── com/microsoft/[component]/
```

### 依賴管理
- 透過 Maven `pom.xml` 管理依賴
- 使用 Spring AI BOM 進行版本統一
- LangChain4j 用於 AI 整合
- 使用 Spring Boot starter parent 管理 Spring 相關依賴

### 程式碼註解
- 對公開 API 添加 JavaDoc
- 複雜 AI 互動加入說明註解
- 清晰記錄 MCP 工具說明

## 建置與部署

### 專案建置

**跳過測試進行建置：**
```bash
mvn clean install -DskipTests
```

**執行所有檢查進行建置：**
```bash
mvn clean install
```

**打包應用程式：**
```bash
mvn package
# 在 target/ 目錄中建立 JAR
```

### 輸出目錄
- 編譯後類別：`target/classes/`
- 測試類別：`target/test-classes/`
- JAR 檔案：`target/*.jar`
- Maven 果件：`target/`

### 環境特定配置

**開發環境：**
```yaml
# application.yml (keyless - no api-key; auth via DefaultAzureCredential)
spring:
  ai:
    azure:
      openai:
        endpoint: ${AZURE_OPENAI_ENDPOINT}
        chat:
          options:
            deployment-name: ${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}
```

**生產環境：**
- 使用託管身份取代 `az login` 進行無密鑰驗證
- `AZURE_OPENAI_ENDPOINT` 指向生產環境 Foundry 資源
- 配置通過環境變數或 Azure Key Vault 管理

### 部署考量
- 本資源庫為教育用途，含範例應用程式
- 不建議直接用於生產部署
- 範例展示生產使用時可調整的模式
- 詳見各專案 README.md 獲取部署細節

## 補充說明

### Azure AI Foundry
- **無密鑰驗證：** 透過 Microsoft Entra ID 連線，無需管理 API 金鑰
- **以程式碼部署：** Bicep + azd (`azd up`) 建立帳戶與模型部署
- 相同步驟可同時於本機（`az login`）及 Azure（託管身份）環境執行 OpenAI 相容程式碼

### 多專案協作
每個範例專案為獨立實體：
```bash
# 導航到特定專案
cd 04-PracticalSamples/[project-name]

# 每個都有自己的 pom.xml，並且可以獨立構建
mvn clean install
```

### 常見問題

**Java 版本不符：**
```bash
# 驗證 Java 21
java -version
# 如有需要，更新 JAVA_HOME
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**依賴下載問題：**
```bash
# 清除 Maven 快取後重試
rm -rf ~/.m2/repository
mvn clean install
```

**找不到端點或未登入狀況：**
```bash
# 在當前會話中設定端點並登入（無需金鑰）
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# 或使用專案目錄中的 .env 檔案
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**埠口已被佔用：**
```bash
# Spring Boot 預設使用 8080 埠口
# 在 application.properties 中更改：
server.port=8081
```

### 多語言支援
- 文件透過自動翻譯提供超過 45 種語言版本
- 翻譯檔在 `translations/` 目錄
- 翻譯流程由 GitHub Actions 工作流程管理

### 學習路徑
1. 從 [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md) 開始
2. 按章節順序學習（01 → 05）
3. 完成每章節中的實作示例
4. 探索第 4 章的範例專案
5. 學習第 5 章的負責任 AI 實踐

### 開發容器
`.devcontainer/devcontainer.json` 配置了：
- Java 21 開發環境
- 已預裝 Maven
- VS Code Java 擴充套件
- Spring Boot 工具
- GitHub Copilot 整合
- Docker-in-Docker 支援
- Azure CLI

### 效能考量
- Azure AI Foundry 佈署有每分鐘 token/請求配額
- 適當調整嵌入維度批次大小
- 考慮快取避免重複 API 呼叫
- 監控 token 用量以優化成本

### 安全說明
- 切勿提交 `.env` 檔案（已加入 `.gitignore`）
- 優先使用無密鑰驗證（Microsoft Entra ID）代替 API 金鑰
- 在 Azure 中使用託管身份；本地開發使用 `az login`
- 遵守第 5 章的負責任 AI 指南

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
此文件已使用 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 進行翻譯。雖然我們努力追求準確性，但請注意自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應視為權威來源。對於關鍵資訊，建議採用專業人工翻譯。我們不對因使用此翻譯所產生的任何誤解或誤譯承擔責任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->