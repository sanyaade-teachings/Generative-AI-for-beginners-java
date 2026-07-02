# 生成式 AI 入门 - Java 版

## 你将学到什么

- **生成式 AI 基础知识**，包括大型语言模型（LLMs）、提示工程、Tokens、嵌入向量和向量数据库
- **比较 Java AI 开发工具**，包括 Azure OpenAI SDK、Spring AI 和 OpenAI Java SDK
- <strong>了解模型上下文协议（Model Context Protocol）</strong>及其在 AI 代理通信中的作用

## 目录

- [介绍](#介绍)
- [生成式 AI 概念快速回顾](#生成式-ai-概念快速回顾)
- [提示工程回顾](#提示工程回顾)
- [Tokens、嵌入向量和代理](#tokens嵌入向量和代理)
- [Java 的 AI 开发工具和库](#java-的-ai-开发工具和库)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [总结](#总结)
- [下一步](#下一步)

## 介绍

欢迎来到《生成式 AI 入门 - Java 版》的第一章！这一基础课程将向你介绍生成式 AI 的核心概念以及如何使用 Java 进行相关工作。你将学习 AI 应用的基本构建模块，包括大型语言模型（LLMs）、Tokens、嵌入向量和 AI 代理。我们还将探索你将在整个课程中使用的主要 Java 工具。

### 生成式 AI 概念快速回顾

生成式 AI 是一种能够基于从数据中学习到的模式和关系，创造新内容（如文本、图片或代码）的人工智能。生成式 AI 模型可以生成类似人类的回复，理解上下文，有时甚至能创造出看似人类创作的内容。

在开发 Java AI 应用时，你会使用 **生成式 AI 模型** 来创建内容。生成式 AI 模型的一些能力包括：

- <strong>文本生成</strong>：为聊天机器人、内容创作和文本补全生成逼真的文本。
- <strong>图像生成与分析</strong>：生成逼真图像、增强照片和检测物体。
- <strong>代码生成</strong>：编写代码片段或脚本。

有些模型针对不同任务进行了优化。例如，<strong>小型语言模型（SLMs）</strong>和<strong>大型语言模型（LLMs）</strong>都可以处理文本生成，且 LLMs 通常在复杂任务中性能更佳。对于图像相关任务，则会使用专门的视觉模型或多模态模型。

![图示：生成式 AI 模型类型及用例。](../../../translated_images/zh-CN/llms.225ca2b8a0d34473.webp)

当然，这些模型的回复并不总是完美的。你可能听说过模型“幻觉”或以权威的方式生成错误信息。但你可以通过提供明确的指令和上下文来引导模型生成更准确的回复，这就是 <strong>提示工程</strong> 的作用。

#### 提示工程回顾

提示工程是设计有效输入以引导 AI 模型产生期望输出的实践。它包括：

- <strong>清晰性</strong>：使指令明确无歧义。
- <strong>上下文</strong>：提供必要的背景信息。
- <strong>约束</strong>：指定任何限制或格式。

提示工程的一些最佳实践包括提示设计、清晰指令、任务拆解、单样本和少样本学习，以及提示微调。测试不同的提示对于找到最适合具体用例的方案至关重要。

开发应用时，你将使用不同类型的提示：
- <strong>系统提示</strong>：设定模型行为的基本规则和上下文
- <strong>用户提示</strong>：来自应用用户的输入数据
- <strong>助理提示</strong>：基于系统和用户提示模型生成的回复

> <strong>了解更多</strong>：更多关于提示工程的内容，请参阅[生成式 AI 入门课程的提示工程章节](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### Tokens、嵌入向量和代理

使用生成式 AI 模型时，你会遇到诸如 **tokens**、**嵌入向量（embeddings）**、**代理（agents）** 和 **模型上下文协议（MCP）** 等术语。以下是这些概念的详细介绍：

- **Tokens**：Tokens 是模型中最小的文本单位。它们可以是单词、字符或子词。Tokens 用于将文本数据表示成模型可以理解的格式。例如，句子“The quick brown fox jumped over the lazy dog”可以根据分词策略被拆分为["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"]或者["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"]。

![图示：生成式 AI tokens 示例，将单词拆分成 tokens](../../../translated_images/zh-CN/tokens.6283ed277a2ffff4.webp)

分词（Tokenization）是将文本拆分成这些更小单元的过程。这非常关键，因为模型处理的是 tokens 而非原始文本。提示中的 tokens 数量会影响模型的回复长度和质量，因为模型对上下文窗口有 tokens 数量限制（例如 GPT-4o 的上下文总限制为 128K tokens，包括输入和输出）。

  在 Java 中，可以使用诸如 OpenAI SDK 之类的库在发送请求给 AI 模型时自动处理分词。

- **嵌入向量（Embeddings）**：嵌入向量是表示 tokens 的向量化表达，能够捕捉语义含义。嵌入通常是数值表示（通常是浮点数数组），帮助模型理解单词之间的关系并生成语境相关的回复。语义相近的词汇其嵌入向量也相近，使模型能够理解同义词和语义关系。

![图示：嵌入向量](../../../translated_images/zh-CN/embedding.398e50802c0037f9.webp)

  在 Java 中，你可以使用 OpenAI SDK 或支持嵌入生成的其它库来生成嵌入向量。这些嵌入对语义搜索等任务尤为重要，因为你需要基于意义而非文本精确匹配来寻找相似内容。

- <strong>向量数据库</strong>：向量数据库是专门针对嵌入向量存储优化的系统。它们支持高效相似度搜索，是基于语义相似性而非精确匹配从大量数据中查找相关信息的检索增强生成（RAG）模式的关键基础。

![图示：向量数据库架构，展示嵌入向量如何被存储和用于相似度搜索。](../../../translated_images/zh-CN/vector.f12f114934e223df.webp)

> <strong>注意</strong>：本课程不会深入讲解向量数据库，但提及它们非常重要，因为它们在真实应用中极为常见。

- **代理（Agents）& MCP**：AI 组件能自主与模型、工具与外部系统交互。模型上下文协议（Model Context Protocol，MCP）提供了一种标准化方式，允许代理安全访问外部数据源和工具。详情请见我们的[MCP 入门课程](https://github.com/microsoft/mcp-for-beginners)。

在 Java AI 应用中，你将用 tokens 进行文本处理，用嵌入实现语义搜索和 RAG，使用向量数据库进行数据检索，利用带有 MCP 的代理构建智能、可使用工具的系统。

![图示：一个提示如何变成回复——tokens、向量、可选的 RAG 查找、LLM 思考，及 MCP 代理合成的快速流程。](../../../translated_images/zh-CN/flow.f4ef62c3052d12a8.webp)

### Java 的 AI 开发工具和库

Java 提供了优秀的 AI 开发工具。我们将探讨三个主要库——OpenAI Java SDK、Azure OpenAI SDK 和 Spring AI。

以下为各章节示例所用 SDK 的简表：

| 章节 | 示例 | SDK |
|---------|--------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**SDK 文档链接：**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

OpenAI SDK 是 OpenAI API 的官方 Java 库，提供了简洁且一致的接口来与 OpenAI 模型交互，使集成 AI 能力至 Java 应用变得简单。第 4 章中的宠物故事应用和 Foundry Local 示例展示了基于 Azure AI Foundry 的 OpenAI SDK 用法。

#### Spring AI

Spring AI 是一个全面框架，向 Spring 应用引入 AI 能力，提供统一的抽象层，支持多种 AI 提供商。它与 Spring 生态系统无缝集成，是企业级 Java 应用中实现 AI 功能的理想选择。

Spring AI 的优势在于其与 Spring 框架的紧密结合，使得使用诸如依赖注入、配置管理和测试框架等熟悉的 Spring 模式构建生产级 AI 应用变得轻松。在第 2 和第 4 章中，你将使用 Spring AI 构建结合 OpenAI 和模型上下文协议（MCP）的应用。

##### 模型上下文协议（MCP）

[模型上下文协议（MCP）](https://modelcontextprotocol.io/) 是一个新兴标准，支持 AI 应用安全地与外部数据源和工具交互。MCP 为 AI 模型访问上下文信息和执行操作提供了标准化方式。

第 4 章中，你将构建一个简单的 MCP 计算器服务，演示使用 Spring AI 进行模型上下文协议基本工具集成和服务架构的示例。

#### Azure OpenAI Java SDK

Azure OpenAI Java 客户端库是对 OpenAI REST API 的封装，提供了符合 Java 习惯的接口，并与 Azure SDK 生态系统集成。在第 3 章，你将使用 Azure OpenAI SDK 构建聊天、函数调用及基于 RAG 模式的应用。

> 注意：Azure OpenAI SDK 在功能上滞后于 OpenAI Java SDK，因此未来项目请考虑优先使用 OpenAI Java SDK。

## 总结

基础内容讲解完毕！你现在已经了解：

- 生成式 AI 的核心概念——从大型语言模型和提示工程，到 tokens、嵌入向量和向量数据库
- Java AI 开发的工具选择：Azure OpenAI SDK、Spring AI 及 OpenAI Java SDK
- 模型上下文协议的基本原理及其如何使 AI 代理能与外部工具协作

## 下一步

[第 2 章：搭建开发环境](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免责声明**：
本文件由 AI 翻译服务 [Co-op Translator](https://github.com/Azure/co-op-translator) 翻译完成。尽管我们力求准确，但请注意，自动翻译可能包含错误或不准确之处。原始语言版文件应视为权威来源。对于重要信息，建议使用专业人工翻译。我们对因使用本翻译而产生的任何误解或误释不承担责任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->