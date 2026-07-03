# AGENTS.md

## Project Overview

這是一個用於學習使用 Java 進行生成式 AI 開發的教育儲存庫。它提供一個涵蓋大型語言模型（LLMs）、提示工程、嵌入、RAG（檢索增強生成）以及模型上下文協議（MCP）的全面實作課程。

**關鍵技術：**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry、Azure OpenAI 與 OpenAI SDKs

**架構：**
- 多個獨立 Spring Boot 應用，依章節組織
- 展示不同 AI 模式的範例專案
- 支援 GitHub Codespaces，配備預設開發容器

## Setup Commands

### Prerequisites
- Java 21 或更高版本
- Maven 3.x
- 具備 Azure 訂閱且在 Azure AI Foundry 部署模型（以 `azd up` 進行佈署）
- Azure CLI (`az`) 與 Azure Developer CLI (`azd`)，且已登入以支援無需金鑰的認證

### Environment Setup

**Option 1: GitHub Codespaces (推薦)**
```bash
# 從 GitHub UI 派生此儲存庫並建立代碼空間
# 開發容器會自動安裝所有依賴項
# 等待約 2 分鐘以完成環境設置
```

**Option 2: Local Dev Container**
```bash
# 複製儲存庫
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# 使用 Dev Containers 擴充功能於 VS Code 中開啟
# 當提示時重新於容器中開啟
```

**Option 3: Local Setup**
```bash
# 安裝依賴
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# 驗證安裝
java -version
mvn -version
```

### Configuration

**Azure AI Foundry 設定（無需金鑰，推薦）:**
```bash
# 以代碼形式設定 Foundry 帳戶及模型部署
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd 會用你的端點（無鑰匙）寫入 examples/basic-chat-azure/.env
```

**手動端點設定：**
```bash
# 如果你沒有使用 azd，請自行設定端點（授權透過 az login 保持免鑰匙）
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# 編輯 .env：AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Development Workflow

### Project Structure
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

### Running Applications

**執行 Spring Boot 應用：**
```bash
cd [project-directory]
mvn spring-boot:run
```

**專案建置：**
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

**執行用戶端範例：**
```bash
# 在另一個終端機啟動伺服器後
cd 04-PracticalSamples/calculator

# 直接 MCP 客戶端
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# AI 驅動的客戶端（需要 AZURE_OPENAI_ENDPOINT + az 登入）
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# 互動式機械人
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Hot Reload
包含支援熱重載的 Spring Boot DevTools：
```bash
# 當保存時，對 Java 文件的更改將自動重新加載
mvn spring-boot:run
```

## Testing Instructions

### Running Tests

**執行專案中所有測試：**
```bash
cd [project-directory]
mvn test
```

**以詳細模式執行測試：**
```bash
mvn test -X
```

**執行特定測試類別：**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Test Structure
- 測試檔案使用 JUnit 5（Jupiter）
- 測試類別位於 `src/test/java/`
- 計算器專案中的用戶端範例在 `src/test/java/com/microsoft/mcp/sample/client/`

### Manual Testing
許多範例為互動式應用，需要手動測試：

1. 使用 `mvn spring-boot:run` 啟動應用
2. 測試端點或與 CLI 互動
3. 確認行為符合各專案 README.md 中的文件說明

### Testing with Azure AI Foundry
- 執行範例前使用 `az login` 登入（無需金鑰認證）
- 確保帳戶在資源上具有認知服務 OpenAI 使用者角色
- 使用第 5 章的負責 AI 範例測試內容過濾

## Code Style Guidelines

### Java Conventions
- **Java 版本：** Java 21，使用現代功能
- **程式風格：** 遵守標準 Java 約定
- **命名規則：** 
  - 類別：PascalCase
  - 方法/變數：camelCase
  - 常數：UPPER_SNAKE_CASE
  - 套件名稱：小寫

### Spring Boot Patterns
- 使用 `@Service` 實作商業邏輯
- 使用 `@RestController` 定義 REST 端點
- 配置使用 `application.yml` 或 `application.properties`
- 優先使用環境變數，避免硬編碼值
- 使用 `@Tool` 註解標記暴露於 MCP 的方法

### File Organization
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

### Dependencies
- 透過 Maven `pom.xml` 管理依賴
- 使用 Spring AI BOM 進行版本管理
- LangChain4j 用於 AI 整合
- Spring Boot starter parent 管理 Spring 依賴

### Code Comments
- 公共 API 添加 JavaDoc
- 複雜 AI 互動部分加註解說明
- 清楚記錄 MCP 工具說明

## Build and Deployment

### Building Projects

**不執行測試的建置：**
```bash
mvn clean install -DskipTests
```

**包含所有檢查的建置：**
```bash
mvn clean install
```

**封裝應用程式：**
```bash
mvn package
# 在 target/ 目錄中建立 JAR
```

### Output Directories
- 編譯後類別：`target/classes/`
- 測試類別：`target/test-classes/`
- JAR 檔案：`target/*.jar`
- Maven 工件：`target/`

### Environment-Specific Configuration

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
- 以管理身分取代 `az login` 實現無需金鑰認證
- `AZURE_OPENAI_ENDPOINT` 指向生產環境 Foundry 資源
- 透過環境變數或 Azure Key Vault 管理設定

### Deployment Considerations
- 此為教學儲存庫的範例應用
- 非設計為直接生產部署
- 範例示範可供生產環境調整的模式
- 參閱各專案 README 了解具體部署說明

## Additional Notes

### Azure AI Foundry
- **無需金鑰認證：** 使用 Microsoft Entra ID 連線，無需管理 API 金鑰
- **以基礎設施即代碼佈署：** 使用 Bicep + azd (`azd up`) 建立帳戶及模型部署
- 相同 OpenAI 相容程式碼，在本機（`az login`）與 Azure（管理身分）皆可運行

### Working with Multiple Projects
每個範例專案皆為獨立：
```bash
# 導航到特定項目
cd 04-PracticalSamples/[project-name]

# 每個都有自己的 pom.xml 並且可以獨立構建
mvn clean install
```

### Common Issues

**Java 版本不匹配：**
```bash
# 驗證 Java 21
java -version
# 如有需要，更新 JAVA_HOME
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**依賴下載問題：**
```bash
# 清除 Maven 緩存並重試
rm -rf ~/.m2/repository
mvn clean install
```

**找不到端點或登入錯誤：**
```bash
# 設定當前會話的端點並登入（無需金鑰）
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# 或使用專案目錄中的.env檔案
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**埠口已被使用：**
```bash
# Spring Boot 預設使用 8080 埠口
# 在 application.properties 中更改：
server.port=8081
```

### Multi-Language Support
- 文件以超過 45 種語言自動翻譯提供
- 翻譯檔位於 `translations/` 目錄
- 翻譯由 GitHub Actions 工作流程管理

### Learning Path
1. 從 [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md) 開始
2. 按照章節順序學習（01 → 05）
3. 完成每章的實作範例
4. 探索第 4 章的示例專案
5. 了解第 5 章的負責 AI 實踐

### Development Container
`.devcontainer/devcontainer.json` 配置內容：
- Java 21 開發環境
- 預裝 Maven
- VS Code Java 擴展
- Spring Boot 工具
- GitHub Copilot 整合
- Docker-in-Docker 支援
- Azure CLI

### Performance Considerations
- Azure AI Foundry 部署有每分鐘 token/請求配額限制
- 嵌入使用適當的批次大小
- 考慮快取以減少重複 API 呼叫
- 監控 token 使用量以優化成本

### Security Notes
- 切勿提交 `.env` 檔案（已加入 `.gitignore`）
- 優先使用無需金鑰的 Microsoft Entra ID 認證
- 在 Azure 使用管理身分，開發時使用 `az login`
- 遵循第 5 章負責 AI 指南

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
本文件使用 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 進行翻譯。雖然我們力求準確，但請注意，自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應被視為權威來源。對於重要資訊，建議尋求專業人工翻譯。我們不對因使用本翻譯而引起的任何誤解或曲解承擔責任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->