# AGENTS.md

## 專案概覽

這是一個用於學習使用 Java 進行生成式 AI 開發的教學儲存庫。它提供了涵蓋大型語言模型（LLMs）、提示工程、嵌入技術、RAG（檢索增強生成）和模型上下文協議（MCP）的全面實作課程。

**主要技術：**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry、Azure OpenAI 與 OpenAI SDKs

**架構：**
- 按章節組織的多個獨立 Spring Boot 應用程式
- 展示不同 AI 模式的範例專案
- GitHub Codespaces 已準備好，預先配置開發容器

## 安裝指令

### 前置條件
- Java 21 或更高版本
- Maven 3.x
- 具有 Azure AI Foundry 模型部署的 Azure 訂閱（使用 `azd up` 進行佈建）
- 已登入的 Azure CLI（`az`）和 Azure 開發者 CLI（`azd`），以支援無金鑰認證

### 環境設定

**選項 1：GitHub Codespaces（建議）**
```bash
# 從 GitHub 用戶界面分叉倉庫並創建代碼空間
# 開發容器會自動安裝所有依賴項
# 等待約 2 分鐘完成環境設置
```

**選項 2：本地開發容器**
```bash
# 複製倉庫
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# 使用 Dev Containers 擴充功能於 VS Code 中打開
# 當提示時重新在容器中打開
```

**選項 3：本地環境安裝**
```bash
# 安裝依賴項
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# 驗證安裝
java -version
mvn -version
```

### 配置

**Azure AI Foundry 設定（無金鑰，建議）：**
```bash
# 以代碼方式配置 Foundry 帳戶及模型部署
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd 用您的端點 (無金鑰) 寫入 examples/basic-chat-azure/.env
```

**手動端點配置：**
```bash
# 如果你沒有使用 azd，請自行設定端點（通過 az login 認證保持無密鑰）
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# 編輯 .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
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

### 運行應用程式

**運行 Spring Boot 應用程式：**
```bash
cd [project-directory]
mvn spring-boot:run
```

**建置專案：**
```bash
cd [project-directory]
mvn clean install
```

**啟動 MCP 計算器伺服器：**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# 伺服器運行於 http://localhost:8080
```

**運行客戶端範例：**
```bash
# 在另一個終端啟動服務器後
cd 04-PracticalSamples/calculator

# 直接MCP客戶端
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# AI驅動客戶端（需要AZURE_OPENAI_ENDPOINT + az登錄）
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# 互動機械人
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### 熱重載
Spring Boot DevTools 已包含於支援熱重載的專案中：
```bash
# 儲存時對 Java 檔案的更改會自動重新載入
mvn spring-boot:run
```

## 測試說明

### 執行測試

**執行專案中所有測試：**
```bash
cd [project-directory]
mvn test
```

**以詳細輸出執行測試：**
```bash
mvn test -X
```

**執行特定測試類別：**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### 測試架構
- 測試使用 JUnit 5（Jupiter）
- 測試類別位於 `src/test/java/`
- 計算器專案的客戶端範例位於 `src/test/java/com/microsoft/mcp/sample/client/`

### 手動測試
許多範例為互動式應用程式，需手動測試：

1. 使用 `mvn spring-boot:run` 啟動應用程式
2. 測試端點或與 CLI 互動
3. 驗證行為是否符合各專案 README.md 中的文件說明

### 使用 Azure AI Foundry 進行測試
- 執行範例前請先使用 `az login` 登入（無金鑰認證）
- 確保您的帳戶在資源上具有認知服務 OpenAI 使用者角色
- 在第 5 章的負責任 AI 範例中測試內容過濾功能

## 代碼風格指南

### Java 規範
- **Java 版本：** Java 21，使用現代特性
- **風格：** 遵循標準 Java 規範
- **命名：** 
  - 類別：PascalCase
  - 方法/變數：camelCase
  - 常數：UPPER_SNAKE_CASE
  - 套件名稱：小寫

### Spring Boot 模式
- 使用 `@Service` 處理業務邏輯
- 使用 `@RestController` 建立 REST 端點
- 配置使用 `application.yml` 或 `application.properties`
- 優先使用環境變數，避免硬編碼
- MCP 對外方法標註 `@Tool` 註解

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
- 使用 Maven `pom.xml` 管理
- Spring AI BOM 版本管理
- 使用 LangChain4j 進行 AI 整合
- Spring Boot starter parent 管理 Spring 依賴

### 代碼註解
- 為公用 API 撰寫 JavaDoc
- 複雜 AI 交互邏輯需附有說明註解
- 清楚紀錄 MCP 工具描述

## 建置與部署

### 專案建置

**建置跳過測試：**
```bash
mvn clean install -DskipTests
```

**執行所有檢查建置：**
```bash
mvn clean install
```

**打包應用程式：**
```bash
mvn package
# 於 target/ 目錄中建立 JAR
```

### 輸出目錄
- 編譯類別：`target/classes/`
- 測試類別：`target/test-classes/`
- JAR 檔：`target/*.jar`
- Maven 產物：`target/`

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
- 使用管理型身分代替 `az login` 以實現無金鑰認證
- 將 `AZURE_OPENAI_ENDPOINT` 指向您的生產 Foundry 資源
- 配置透過環境變數或 Azure Key Vault 管理

### 部署注意事項
- 本儲存庫為教學範例應用程式
- 非設計為直接投入生產環境
- 範例演示可供生產應用改編的設計模式
- 具體部署細節請參考各專案 README

## 補充說明

### Azure AI Foundry
- **無金鑰認證：** 使用 Microsoft Entra ID，無需管理 API 金鑰
- **以程式碼部署：** 使用 Bicep 與 azd (`azd up`) 創建帳戶與模型部署
- 相同的 OpenAI 兼容程式碼可以在本地 (`az login`) 與 Azure (管理型身分) 執行

### 多專案操作
每個範例專案皆為獨立：
```bash
# 導航至指定項目
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

**依賴下載失敗：**
```bash
# 清除 Maven 快取並重試
rm -rf ~/.m2/repository
mvn clean install
```

**找不到端點或登入失敗：**
```bash
# 在當前會話中設置端點並登入（無需金鑰）
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# 或在項目目錄中使用 .env 文件
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**埠口已被佔用：**
```bash
# Spring Boot 預設使用 8080 埠口
# 在 application.properties 更改：
server.port=8081
```

### 多語言支援
- 文件透過自動翻譯支援 45+ 語言
- 翻譯檔案存放於 `translations/` 目錄
- 由 GitHub Actions 工作流程管理翻譯

### 學習路徑
1. 從 [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md) 開始
2. 依章節順序學習（01 → 05）
3. 完成每章的實作範例
4. 探索第 4 章的範例專案
5. 學習第 5 章中的負責任 AI 實踐

### 開發容器
`.devcontainer/devcontainer.json` 配置：
- Java 21 開發環境
- 預裝 Maven
- VS Code Java 延伸功能
- Spring Boot 工具
- GitHub Copilot 整合
- Docker-in-Docker 支援
- Azure CLI

### 性能考量
- Azure AI Foundry 部署有每分鐘令牌/請求配額限制
- 對嵌入請求使用適當批次大小
- 可考慮快取重複的 API 呼叫
- 監控令牌使用以優化成本

### 安全注意事項
- 永遠不要提交 `.env` 檔案（已加入 `.gitignore`）
- 優先使用無金鑰認證（Microsoft Entra ID），避免 API 金鑰
- 在 Azure 中使用管理型身分，本地則使用 `az login`
- 遵守第 5 章中的負責任 AI 指南

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
本文件由 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 翻譯而成。雖然我們致力於確保準確性，但請注意，機器自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應被視為權威來源。對於重要資訊，建議進行專業人工翻譯。我們不對因使用本翻譯而產生的任何誤解或誤釋承擔責任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->