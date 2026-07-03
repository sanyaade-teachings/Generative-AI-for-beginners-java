# AGENTS.md

## 项目概览

这是一个用于学习基于 Java 的生成式 AI 开发的教育仓库。它提供了一个涵盖大型语言模型（LLM）、提示工程、嵌入、RAG（检索增强生成）和模型上下文协议（MCP）的综合实操课程。

**关键技术：**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry，Azure OpenAI 和 OpenAI SDK

**架构：**
- 按章节组织的多个独立 Spring Boot 应用
- 演示不同 AI 模式的示例项目
- 支持 GitHub Codespaces，预配置开发容器

## 设置命令

### 前提条件
- Java 21 或更高版本
- Maven 3.x
- Azure 订阅并部署 Azure AI Foundry 模型（通过 `azd up` 进行配置）
- Azure CLI (`az`) 和 Azure 开发者 CLI (`azd`)，已登录以实现无密钥认证

### 环境设置

**选项 1：GitHub Codespaces（推荐）**
```bash
# 从 GitHub UI 分叉仓库并创建一个代码空间
# 开发容器会自动安装所有依赖项
# 等待大约 2 分钟完成环境设置
```

**选项 2：本地开发容器**
```bash
# 克隆仓库
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# 使用 Dev Containers 扩展在 VS Code 中打开
# 提示时重新在容器中打开
```

**选项 3：本地设置**
```bash
# 安装依赖
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# 验证安装
java -version
mvn -version
```

### 配置

**Azure AI Foundry 设置（无密钥，推荐）：**
```bash
# 将Foundry账户和模型部署作为代码进行配置
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd使用你的端点（无密钥）写入examples/basic-chat-azure/.env文件
```

**手动端点配置：**
```bash
# 如果你没有使用 azd，请自行设置端点（通过 az login 认证保持无密钥状态）
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# 编辑 .env 文件：AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## 开发工作流程

### 项目结构
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

### 运行应用

**运行 Spring Boot 应用：**
```bash
cd [project-directory]
mvn spring-boot:run
```

**构建项目：**
```bash
cd [project-directory]
mvn clean install
```

**启动 MCP 计算器服务器：**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# 服务器运行在 http://localhost:8080
```

**运行客户端示例：**
```bash
# 在另一个终端启动服务器后
cd 04-PracticalSamples/calculator

# 直接MCP客户端
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# AI驱动的客户端（需要AZURE_OPENAI_ENDPOINT + az登录）
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# 交互式机器人
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### 热重载
支持热重载的项目包含 Spring Boot DevTools：
```bash
# 保存时对Java文件的更改将自动重新加载
mvn spring-boot:run
```

## 测试说明

### 运行测试

**运行项目中的所有测试：**
```bash
cd [project-directory]
mvn test
```

**运行带详细输出的测试：**
```bash
mvn test -X
```

**运行特定测试类：**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### 测试结构
- 测试文件使用 JUnit 5 (Jupiter)
- 测试类位于 `src/test/java/`
- 计算器项目中的客户端示例位于 `src/test/java/com/microsoft/mcp/sample/client/`

### 手动测试
许多示例是交互式应用，需手动测试：

1. 使用 `mvn spring-boot:run` 启动应用
2. 测试端点或与命令行交互
3. 确认预期行为与各项目 README.md 中的文档匹配

### 使用 Azure AI Foundry 测试
- 运行示例前使用 `az login` 登录（无密钥认证）
- 确保账户在资源上拥有认知服务 OpenAI 用户角色
- 使用第五章的责任 AI 示例测试内容过滤机制

## 代码风格指南

### Java 约定
- **Java 版本：** Java 21，使用现代特性
- **风格：** 遵循标准 Java 约定
- **命名：** 
  - 类名：PascalCase
  - 方法/变量名：camelCase
  - 常量：UPPER_SNAKE_CASE
  - 包名：小写

### Spring Boot 模式
- 使用 `@Service` 标注业务逻辑
- 使用 `@RestController` 标注 REST 端点
- 通过 `application.yml` 或 `application.properties` 配置
- 优先使用环境变量替代硬编码值
- 使用 `@Tool` 注解标注 MCP 暴露的方法

### 文件组织
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

### 依赖管理
- 通过 Maven `pom.xml` 管理
- 使用 Spring AI BOM 统一版本控制
- 使用 LangChain4j 进行 AI 集成
- Spring Boot 起步父级管理 Spring 依赖

### 代码注释
- 为公共 API 添加 JavaDoc
- 对复杂的 AI 交互添加解释注释
- 清晰记录 MCP 工具描述

## 构建与部署

### 构建项目

**不带测试构建：**
```bash
mvn clean install -DskipTests
```

**带全部检查构建：**
```bash
mvn clean install
```

**打包应用：**
```bash
mvn package
# 在 target/ 目录中创建 JAR
```

### 输出目录
- 编译类文件：`target/classes/`
- 测试类文件：`target/test-classes/`
- JAR 文件：`target/*.jar`
- Maven 工件：`target/`

### 环境特定配置

**开发环境：**
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

**生产环境：**
- 推荐使用托管身份代替 `az login` 实现无密钥认证
- 将 `AZURE_OPENAI_ENDPOINT` 指向生产 Foundry 资源
- 通过环境变量或 Azure Key Vault 管理配置

### 部署注意事项
- 本仓库为教育用途示例应用
- 并非为生产部署设计
- 示例项目演示生产环境适用的模式
- 具体部署细节请参阅各项目 README

## 其他说明

### Azure AI Foundry
- **无密钥认证：** 通过 Microsoft Entra ID 连接，无需管理 API 密钥
- **代码方式预配置：** 通过 Bicep + azd (`azd up`) 创建账户和模型部署
- 同一份兼容 OpenAI 的代码本地（`az login`）和 Azure（托管身份）均可运行

### 多项目协作
每个示例项目独立运行：
```bash
# 导航到特定项目
cd 04-PracticalSamples/[project-name]

# 每个都有自己的 pom.xml，可以独立构建
mvn clean install
```

### 常见问题

**Java 版本不匹配：**
```bash
# 验证 Java 21
java -version
# 如有必要，更新 JAVA_HOME
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**依赖下载问题：**
```bash
# 清除 Maven 缓存并重试
rm -rf ~/.m2/repository
mvn clean install
```

**找不到端点或登录失败：**
```bash
# 在当前会话中设置端点并登录（无密钥）
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# 或在项目目录中使用 .env 文件
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**端口已被占用：**
```bash
# Spring Boot 默认使用端口 8080
# 在 application.properties 中更改：
server.port=8081
```

### 多语言支持
- 文档通过自动翻译支持 45+ 种语言
- 翻译文件位于 `translations/` 目录
- 翻译过程由 GitHub Actions 工作流管理

### 学习路径
1. 从 [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md) 开始
2. 按章节顺序学习（01 → 05）
3. 完成各章节的实操示例
4. 探索第四章示例项目
5. 学习第五章的责任 AI 实践

### 开发容器
`.devcontainer/devcontainer.json` 配置了：
- Java 21 开发环境
- 预装 Maven
- VS Code Java 扩展
- Spring Boot 工具
- GitHub Copilot 集成
- 支持 Docker-in-Docker
- Azure CLI

### 性能考虑
- Azure AI Foundry 部署有每分钟令牌/请求配额限制
- 嵌入操作请使用合适批量大小
- 重复 API 调用建议使用缓存
- 监控令牌使用以优化成本

### 安全注意事项
- 切勿提交 `.env` 文件（已加入 `.gitignore`）
- 优先使用无密钥认证（Microsoft Entra ID）替代 API 密钥
- Azure 中使用托管身份；本地开发使用 `az login`
- 遵循第五章中的责任 AI 指南

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免责声明**：
本文件由 AI 翻译服务 [Co-op Translator](https://github.com/Azure/co-op-translator) 翻译完成。尽管我们力求准确，但请注意，自动翻译可能包含错误或不准确之处。原始语言版文件应视为权威来源。对于重要信息，建议使用专业人工翻译。我们对因使用本翻译而产生的任何误解或误释不承担责任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->