# 使用 Azure AI Foundry 的基础聊天演示 - 端到端示例

此示例是一个简单的 Spring Boot 应用程序，使用<strong>无密钥身份验证</strong>（Microsoft Entra ID）连接到 **Azure AI Foundry** 模型并测试您的设置。它使用 Spring AI 的 `ChatClient`。

## 目录

- [先决条件](#先决条件)
- [快速开始](#快速开始)
- [身份验证工作原理](#身份验证工作原理)
- [运行应用程序](#运行应用程序)
  - [使用 Maven](#使用-maven)
  - [使用 VS Code](#使用-vs-code)
  - [预期输出](#预期输出)
- [配置参考](#配置参考)
  - [环境变量](#环境变量)
  - [Spring 配置](#spring-配置)
- [故障排除](#故障排除)
  - [常见问题](#常见问题)
  - [调试模式](#调试模式)
- [下一步](#下一步)
- [资源](#资源)

## 先决条件

运行此示例之前，请确保您具备：

- 一个带有 `gpt-4o-mini` 部署的 Azure AI Foundry 资源 —— 通过 `azd up` 或手动使用 [Azure AI Foundry 设置指南](../../getting-started-azure-openai.md) 进行配置
- 该资源上的 **认知服务 OpenAI 用户** 角色（Bicep 模板会为您分配）
- 已登录的 [Azure CLI（`az`）](https://learn.microsoft.com/cli/azure/install-azure-cli) ，使用 `az login`
- Java 21+ 和 Maven 3.9+

> **无需 API 密钥** — 身份验证通过 Microsoft Entra ID 无密钥完成。

## 快速开始

```bash
# 1. 导航到项目
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. 登录以便无密钥身份验证可以获取令牌
az login

# 3. 配置端点
#    - 如果你运行了 `azd up`，.env 文件已经为你写好（跳过此步骤）。
#    - 否则复制模板并设置 AZURE_OPENAI_ENDPOINT：
cp .env.example .env

# 4. 运行应用程序
mvn spring-boot:run
```

## 身份验证工作原理

此示例使用 **Microsoft Entra ID** 进行身份验证 —— 不需要 API 密钥。

仅设置了 `spring.ai.azure.openai.endpoint`（且无 api-key）时，Spring AI 会使用 [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential) 构建 Azure OpenAI 客户端。该凭据会自动从您的本地 `az login` 会话获取令牌，或在 Azure 中使用托管身份 —— 因此相同的代码无需更改即可在两者之间运行。

## 运行应用程序

### 使用 Maven

```bash
mvn spring-boot:run
```

### 使用 VS Code

1. 在 VS Code 中打开项目
2. 按 `F5` 或使用“运行和调试”面板
3. 选择 “Spring Boot-BasicChatApplication” 配置

> <strong>注意</strong>：VS Code 配置会自动加载您的 .env 文件

### 预期输出

```
Starting Basic Chat with Azure OpenAI...
Environment variables loaded successfully
Connecting to Azure OpenAI...
Sending prompt: What is AI in a short sentence? Max 100 words.

AI Response:
================
AI, or Artificial Intelligence, is the simulation of human intelligence in machines programmed to think and learn like humans.
================

Success! Azure OpenAI connection is working correctly.
```

## 配置参考

### 环境变量

| 变量 | 描述 | 是否必需 | 示例 |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | Foundry（Azure OpenAI）端点 URL | 是 | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | 聊天模型部署名称 | 否 | `gpt-4o-mini`（默认） |

> 没有 API 密钥变量 —— 身份验证是无密钥的（通过 `az login` 使用 Microsoft Entra ID）。

### Spring 配置

`application.yml` 文件配置：
- <strong>端点</strong>：`${AZURE_OPENAI_ENDPOINT}` - 来自环境变量
- <strong>部署</strong>：`${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - 来自环境变量，带回退值
- <strong>认证</strong>：无密钥 —— 未设置 `api-key`，Spring AI 使用 `DefaultAzureCredential`
- <strong>温度</strong>：`0.7` - 控制创造力（0.0=确定性，1.0=创造性）
- <strong>最大令牌数</strong>：`500` - 最大响应长度

## 故障排除

### 常见问题

<details>
<summary><strong>错误：401 / "PermissionDenied" / 令牌错误</strong></summary>

- 运行 `az login` — 无密钥身份验证需要有效登录以获取令牌
- 验证您的账户在资源上具备 **认知服务 OpenAI 用户** 角色
- 如果刚分配角色，等待一分钟以便生效
- 确认您处于正确的租户/订阅（`az account show`）
</details>

<details>
<summary><strong>错误："端点无效" / 连接错误</strong></summary>

- 确保 `AZURE_OPENAI_ENDPOINT` 是完整的基础 URL（例如 `https://your-resource.openai.azure.com/`）
- 检查尾部斜杠一致性
- 确认端点匹配您已配置的资源（`azd env get-values`）
</details>

<details>
<summary><strong>错误："找不到部署"</strong></summary>

- 验证 `AZURE_OPENAI_DEPLOYMENT` 是否与 Azure 中的部署名称一致
- 检查模型已成功部署并处于激活状态
- 默认部署名称是 `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code：环境变量未加载</strong></summary>

- 确保 `.env` 文件位于项目根目录（与 `pom.xml` 同级）
- 尝试在 VS Code 集成终端中运行 `mvn spring-boot:run`
- 确认已正确安装 VS Code 的 Java 扩展
</details>

### 调试模式

要启用详细日志，请取消注释 `application.yml` 中的以下内容：

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## 下一步

**设置完成！** 继续您的学习旅程：

[第3章：核心生成式 AI 技术](../../../03-CoreGenerativeAITechniques/README.md)

## 资源

- [Spring AI Azure OpenAI 文档](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [使用 Microsoft Entra ID 的无密钥身份验证](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry 门户](https://ai.azure.com/)
- [Azure AI Foundry 文档](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免责声明**：
本文件由 AI 翻译服务 [Co-op Translator](https://github.com/Azure/co-op-translator) 翻译完成。尽管我们力求准确，但请注意，自动翻译可能包含错误或不准确之处。原始语言版文件应视为权威来源。对于重要信息，建议使用专业人工翻译。我们对因使用本翻译而产生的任何误解或误释不承担责任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->