# 为 Java 构建生成式 AI 设置开发环境

> **快速开始：** 使用 Bicep + `azd` 在几分钟内将您的 AI 模型作为代码在 **Azure AI Foundry** 上配置——请参阅[Azure AI Foundry 设置指南](getting-started-azure-openai.md)。身份验证采用<strong>无密钥</strong>（Microsoft Entra ID），因此无需管理 API 密钥。

## 您将学到什么

- 设置用于 AI 应用的 Java 开发环境  
- 选择并配置您偏好的开发环境（云优先的 Codespaces、本地开发容器或完整本地设置）  
- 通过连接到 Azure AI Foundry 模型测试您的设置  

## 目录

- [您将学到什么](#您将学到什么)
- [介绍](#介绍)
- [步骤 1：设置开发环境](#步骤-1设置开发环境)
  - [选项 A：GitHub Codespaces（推荐）](#选项-agithub-codespaces推荐)
  - [选项 B：本地开发容器](#选项-b本地开发容器)
  - [选项 C：使用现有本地安装](#选项-c使用现有本地安装)
- [步骤 2：配置 Azure AI Foundry](#步骤-2配置-azure-ai-foundry)
- [步骤 3：测试您的设置](#步骤-3测试您的设置)
- [故障排除](#故障排除)
- [摘要](#摘要)
- [后续步骤](#后续步骤)

## 介绍

本章将引导您设置开发环境。在本课程中，我们将使用 **Azure AI Foundry** 进行模型管理。您可以通过 Bicep 和 Azure 开发者 CLI (`azd`) 将模型作为代码进行配置，然后采用 <strong>无密钥身份验证</strong>（Microsoft Entra ID）连接——无需复制或泄露 API 密钥。

**无需本地设置！** 您可以使用 GitHub Codespaces，它在浏览器中提供完整的开发环境，并可从那里配置 Foundry。

我们选用 **Azure AI Foundry** 是因为它：
- <strong>作为代码进行配置</strong> —— 一条 `azd up` 命令即可部署账户和模型  
- <strong>无密钥</strong> —— 使用 Azure 登录或托管身份进行身份验证  
- <strong>生产级别准备</strong> —— 同一份代码可以本地和 Azure 上运行  
- <strong>灵活性强</strong> —— 通过更改部署名称替换模型，无需修改代码  

> **注意：** Azure AI Foundry 的部署按令牌计费（按用量付费）。有关配置、区域和费用详情，请参阅[Azure AI Foundry 设置指南](getting-started-azure-openai.md)。

## 步骤 1：设置开发环境

<a name="quick-start-cloud"></a>

我们已创建了预配置的开发容器，以最大限度减少设置时间，确保您拥有本生成式 AI for Java 课程所需的所有工具。请选择您的开发方式：

### 环境设置选项：

#### 选项 A：GitHub Codespaces（推荐）

**2 分钟内开始编码——无需本地设置！**

1. 将此仓库 Fork 到您的 GitHub 账户  
   > **注意：** 如需编辑基础配置，请查看[开发容器配置](../../../.devcontainer/devcontainer.json)  
2. 点击 **Code** → **Codespaces** 标签 → **...** → **New with options...**  
3. 使用默认设置 —— 这将选择为本课程定制的 **生成式 AI Java 开发环境** 开发容器配置  
4. 点击 **Create codespace**  
5. 等待约 2 分钟，环境即准备好  
6. 继续进行[步骤 2：配置 Azure AI Foundry](#步骤-2配置-azure-ai-foundry)

<img src="../../../translated_images/zh-CN/codespaces.9945ded8ceb431a5.webp" alt="截图：Codespaces 子菜单" width="50%">

<img src="../../../translated_images/zh-CN/image.833552b62eee7766.webp" alt="截图：带选项的新建" width="50%">

<img src="../../../translated_images/zh-CN/codespaces-create.b44a36f728660ab7.webp" alt="截图：创建 codespace 选项" width="50%">

> **Codespaces 的优势：**  
> - 无需本地安装  
> - 在任何带浏览器的设备上运行  
> - 预配置所有工具和依赖  
> - 个人账户每月免费 60 小时  
> - 为所有学员提供一致环境

#### 选项 B：本地开发容器

**适合偏好本地使用 Docker 开发的开发者**

1. 将此仓库 Fork 并克隆到本地机器  
   > **注意：** 如需编辑基础配置，请查看[开发容器配置](../../../.devcontainer/devcontainer.json)  
2. 安装 [Docker Desktop](https://www.docker.com/products/docker-desktop/) 及 [VS Code](https://code.visualstudio.com/)  
3. 在 VS Code 中安装 [Dev Containers 扩展](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)  
4. 在 VS Code 中打开仓库文件夹  
5. 当提示时，点击 **Reopen in Container**（或使用 `Ctrl+Shift+P` → “Dev Containers: Reopen in Container”）  
6. 等待容器构建并启动完成  
7. 继续进行[步骤 2：配置 Azure AI Foundry](#步骤-2配置-azure-ai-foundry)

<img src="../../../translated_images/zh-CN/devcontainer.21126c9d6de64494.webp" alt="截图：开发容器设置" width="50%">

<img src="../../../translated_images/zh-CN/image-3.bf93d533bbc84268.webp" alt="截图：开发容器构建完成" width="50%">

#### 选项 C：使用现有本地安装

**适合已有 Java 环境的开发者**

前置要求：  
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html)  
- [Maven 3.9+](https://maven.apache.org/download.cgi)  
- [VS Code](https://code.visualstudio.com) 或您偏好的 IDE

步骤：  
1. 克隆此仓库到本地机器  
2. 在 IDE 中打开项目  
3. 继续进行[步骤 2：配置 Azure AI Foundry](#步骤-2配置-azure-ai-foundry)

> **专业提示：** 如果机器配置较低但想本地用 VS Code，强烈推荐使用 GitHub Codespaces！您可以将本地 VS Code 连接到云端的 Codespace，享受两者最佳体验。

<img src="../../../translated_images/zh-CN/image-2.fc0da29a6e4d2aff.webp" alt="截图：已创建的本地开发容器实例" width="50%">

## 步骤 2：配置 Azure AI Foundry

将课程 AI 模型作为代码部署到 Azure AI Foundry。请在仓库根目录执行：

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```
  
`azd` 会提示您输入环境名称和区域，配置包含 `gpt-4o-mini` 和 `text-embedding-3-small` 部署的 Azure AI Foundry 账户，并将端点写入示例的 `.env` 文件——均采用<strong>无密钥</strong>身份验证（无需 API 密钥）。

> **完整操作指导：** 请参阅[Azure AI Foundry 设置指南](getting-started-azure-openai.md)，了解先决条件、手动（门户）方式、区域建议以及费用/清理说明。

## 步骤 3：测试您的设置

Foundry 模型配置完成后，使用示例应用测试连接，路径为 [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure)。

1. 在开发环境中打开终端。  
2. 进入示例目录：  
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
  
3. 确保您已登录（无密钥身份验证需要令牌）：  
   ```bash
   az login
   ```
  
   > 如果您已运行 `azd up`，则 `.env` 文件和端点已自动写好。  
4. 运行应用：  
   ```bash
   mvn clean spring-boot:run
   ```
  
您应看到来自 `gpt-4o-mini` 模型的响应。

### 理解示例代码

`examples/basic-chat-azure` 下的示例是一个 Spring Boot 应用，使用 **Spring AI** 通过无密钥身份验证连接到 Azure AI Foundry。

**代码实现功能：**  
- <strong>连接</strong> Azure AI Foundry，使用您的 Azure 登录（Microsoft Entra ID）——无需 API 密钥  
- <strong>发送</strong> 提示给 `gpt-4o-mini` 模型  
- <strong>接收</strong> 并显示 AI 回复  
- <strong>验证</strong> 您的设置是否正确  

<strong>关键依赖</strong> （在 `pom.xml`）：  
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```
  
<strong>配置文件</strong> (`application.yml`)：  
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

太好了！您现在已完成所有配置：

- 使用 Bicep + `azd` 将 Azure AI Foundry 模型作为代码配置完毕  
- 启动了 Java 开发环境（无论是 Codespaces、开发容器还是本地环境）  
- 通过无密钥身份验证（Microsoft Entra ID）连接到 Azure AI Foundry——无须 API 密钥  
- 通过一个简单示例，成功测试了模型调用  

## 后续步骤

[第 3 章：核心生成式 AI 技术](../03-CoreGenerativeAITechniques/README.md)

## 故障排除

遇到问题？这里列出常见问题与解决方案：

- **认证失败（401/403）？**  
  - 运行 `az login` —— 身份验证为无密钥，必须先登录  
  - 确认您的账户在资源上具有 **认知服务 OpenAI 用户** 角色  
  - 若刚配置完成，请稍等一分钟让角色分配生效  

- **找不到 Maven？**  
  - 使用开发容器/Codespaces 时，Maven 应已预装  
  - 本地安装确保 Java 21+ 和 Maven 3.9+  
  - 运行 `mvn --version` 验证安装  

- **找不到 `azd` 或配置失败？**  
  - 安装 [Azure Developer CLI](https://aka.ms/azure-dev/install) 并执行 `azd auth login`  
  - 选择支持 `gpt-4o-mini` 的区域（例如 `eastus2`）  
  - 参阅[Azure AI Foundry 设置指南](getting-started-azure-openai.md)

- **开发容器无法启动？**  
  - 确认 Docker Desktop 已运行（针对本地开发）  
  - 尝试重建容器：`Ctrl+Shift+P` → “Dev Containers: Rebuild Container”  

- **应用编译错误？**  
  - 确保所在目录为：`02-SetupDevEnvironment/examples/basic-chat-azure`  
  - 尝试清理并重新编译：`mvn clean compile`  

> **需要帮助？** 遇到困难请在仓库提 Issue，我们会帮您解决。

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免责声明**：
本文件由 AI 翻译服务 [Co-op Translator](https://github.com/Azure/co-op-translator) 翻译完成。尽管我们力求准确，但请注意，自动翻译可能包含错误或不准确之处。原始语言版文件应视为权威来源。对于重要信息，建议使用专业人工翻译。我们对因使用本翻译而产生的任何误解或误释不承担责任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->