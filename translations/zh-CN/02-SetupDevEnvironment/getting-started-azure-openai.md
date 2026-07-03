# 设置 Azure AI Foundry 的开发环境

> 本指南使用 <strong>无密钥</strong> 认证（Microsoft Entra ID）为本课程中的 Java AI 应用设置 **Azure AI Foundry** 模型 — 无需管理 API 密钥。工具使用新手？请先阅读[开发环境指南](./README.md)。

本指南为本课程的 Java AI 应用设置 **Azure AI Foundry** 模型。您有两种途径：

- **方案 A — 使用 `azd` + Bicep 自动配置（推荐）：** 一条命令通过代码部署 Foundry 账户和模型，无需门户操作。
- **方案 B — 在 Azure AI Foundry 门户手动创建资源。**

两种方案均使用 <strong>无密钥认证</strong>（Microsoft Entra ID） — 无需复制或泄露 API 密钥。

## 目录

- [将创建的内容](#将创建的内容)
- [先决条件](#先决条件)
- [方案 A：使用 azd + Bicep 自动配置（推荐）](#option-a-provision-with-azd--bicep-recommended)
- [方案 B：手动创建资源](#方案-b：手动创建资源)
- [配置您的环境](#配置您的环境)
- [测试您的设置](#测试您的设置)
- [接下来做什么？](#接下来做什么？)
- [资源](#资源)
- [更多资源](#更多资源)

## 将创建的内容

[`infra/`](../../../02-SetupDevEnvironment/infra) 中的 Bicep 模板将创建：

- 一个 **Azure AI Foundry** 账户（资源类型 `Microsoft.CognitiveServices/accounts`，种类 `AIServices`），以及一个项目
- 一个 <strong>聊天</strong> 部署 — `gpt-4o-mini`
- 一个 <strong>向量嵌入</strong> 部署 — `text-embedding-3-small`（后续章节使用）
- 一个 <strong>无密钥角色分配</strong>（`Cognitive Services OpenAI User`），可通过 `az login` 签入，无需管理密钥

## 先决条件

- 一个 [Azure 订阅](https://azure.microsoft.com/free/)
- [Azure 开发者 CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) 和 [Maven 3.9+](https://maven.apache.org/download.cgi)

## 方案 A：使用 azd + Bicep 自动配置（推荐）

在 `02-SetupDevEnvironment` 文件夹中执行：

```bash
cd 02-SetupDevEnvironment

# 登录（两个工具）
azd auth login
az login

# 配置Foundry账户和模型部署
azd up
```

`azd` 会提示您输入 <strong>环境名称</strong>（比如 `genai-java`）和 <strong>区域</strong>。选择支持 `gpt-4o-mini` 和 `text-embedding-3-small` 的区域，例如 `eastus2` 或 `swedencentral`。

配置完成后，azd 会：

1. 部署 [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep) 中定义的所有资源。
2. 运行一个后置配置钩子，写入 [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) 文件，包含您的终结点和部署名称（无秘密信息）。

> **提示：** 随时重新运行 `azd up` 应用更改。运行 `azd down` 删除所有资源并停止产生费用。

查看生成的设置：

```bash
azd env get-values
```

然后跳转到 [测试您的设置](#测试您的设置)。

## 方案 B：手动创建资源

偏好使用门户？请手动创建资源：

1. 访问 [Azure AI Foundry 门户](https://ai.azure.com/) 并登录。
2. <strong>创建一个项目</strong>（这也会创建一个 AI Foundry 资源）。命名例如 `GenAIJava`。
3. 在您的项目中，打开 <strong>模型与终结点</strong> → <strong>部署模型</strong> → <strong>部署基础模型</strong>。
4. 部署 **gpt-4o-mini**（部署名称为 `gpt-4o-mini`）。如果需要嵌入示例，也部署 **text-embedding-3-small**。
5. 在 <strong>概览</strong> 菜单中，复制 <strong>终结点</strong>（例如 `https://<resource>.openai.azure.com/`）。
6. 授予自己无密钥访问权限：在资源上打开 **访问控制 (IAM)** → <strong>添加角色分配</strong> → 将 **Cognitive Services OpenAI User** 角色分配给您的账户。

> **仍有困难？** 参见 [Azure AI Foundry 文档](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects)。

## 配置您的环境

**如果您使用了方案 A（`azd up`），** 您的设置文件已经写好，无需配置。直接跳转到 [测试您的设置](#测试您的设置)。

**如果您使用了方案 B（手动），** 请自行创建示例的 `.env` 文件：

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

用您的终结点编辑 `.env` 文件（无需密钥 — 认证为无密钥）：

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **安全提示：** 无需保存任何 API 密钥。认证通过 `az login`（本地）或托管身份（Azure 内）使用 Microsoft Entra ID 完成。`.env` 文件只包含非机密设置，且已被 `.gitignore` 保护。

## 测试您的设置

确保您已登录以便无密钥认证获得令牌，然后运行示例：

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # 如果您尚未登录
mvn clean spring-boot:run
```

您应该看到来自 `gpt-4o-mini` 模型的响应！

> **VS Code 用户：** 按 `F5` 运行。应用会自动加载您的 `.env` 文件。

> **完整示例：** 详情和故障排查见 [Azure AI Foundry 基础聊天示例](./examples/basic-chat-azure/README.md)。

## 接下来做什么？

**设置完成！** 您现在拥有：
- 已部署 `gpt-4o-mini` 和 `text-embedding-3-small` 的 Azure AI Foundry
- 无密钥认证（Microsoft Entra ID）— 无需管理密钥
- 包含终结点和部署名称的本地 `.env` 文件
- 准备好的 Java 开发环境

<strong>继续阅读</strong> [第三章：核心生成式 AI 技术](../03-CoreGenerativeAITechniques/README.md) 开始构建 AI 应用吧！

## 资源

- [Azure 开发者 CLI (azd)](https://aka.ms/azure-dev/install)
- [使用 Microsoft Entra ID 的无密钥认证](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry 文档](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI 文档](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## 更多资源

- [下载 VS Code](https://code.visualstudio.com/Download)
- [获取 Docker Desktop](https://www.docker.com/products/docker-desktop)
- [开发容器配置](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免责声明**：
本文件由 AI 翻译服务 [Co-op Translator](https://github.com/Azure/co-op-translator) 翻译完成。尽管我们力求准确，但请注意，自动翻译可能包含错误或不准确之处。原始语言版文件应视为权威来源。对于重要信息，建议使用专业人工翻译。我们对因使用本翻译而产生的任何误解或误释不承担责任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->