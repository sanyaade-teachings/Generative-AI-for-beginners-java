# 实际应用与项目

## 你将学习的内容
在本节中，我们将演示三个展示使用Java进行生成式AI开发模式的实际应用：
- 创建一个结合客户端和服务器端AI的多模态宠物故事生成器
- 使用Foundry Local Spring Boot演示实现本地AI模型集成
- 使用计算器示例开发模型上下文协议（MCP）服务

## 目录

- [介绍](#介绍)
  - [Foundry Local Spring Boot 演示](#foundry-local-spring-boot-演示)
  - [宠物故事生成器](#宠物故事生成器)
  - [MCP 计算器服务（适合初学者的MCP演示）](#mcp-计算器服务（适合初学者的mcp演示）)
- [学习进度](#学习进度)
- [总结](#总结)
- [后续步骤](#后续步骤)

## 介绍

本章展示<strong>示例项目</strong>，演示使用Java进行生成式AI开发的模式。每个项目均为完整功能，展示了可供你自己的应用采用的特定AI技术、架构模式和最佳实践。

### Foundry Local Spring Boot 演示

**[Foundry Local Spring Boot 演示](foundrylocal/README.md)** 演示如何使用<strong>OpenAI Java SDK</strong>集成本地AI模型。它展示了如何连接运行在Foundry Local上的模型（例如<strong>Phi-4-mini</strong>），具备自动模型检测功能，使你可以无需依赖云服务即可运行AI应用。

### 宠物故事生成器

**[宠物故事生成器](petstory/README.md)** 是一个有趣的Spring Boot网络应用，演示了<strong>多模态AI处理</strong>以生成创意宠物故事。它结合了客户端和服务器端的AI功能，使用transformer.js实现浏览器端AI交互，使用OpenAI SDK实现服务器端处理。

### MCP 计算器服务（适合初学者的MCP演示）

**[MCP 计算器服务](calculator/README.md)** 是一个使用Spring AI演示<strong>模型上下文协议（MCP）</strong>的简单示例。它为初学者介绍了MCP的概念，展示了如何创建一个与MCP客户端交互的基础MCP服务器。

## 学习进度

这些项目设计为建立在之前章节概念的基础上：

1. <strong>从简单开始</strong>：先从Foundry Local Spring Boot演示开始，理解与本地模型的基础AI集成
2. <strong>增加交互性</strong>：进阶到宠物故事生成器，体验多模态AI和基于web的交互
3. **学习MCP基础**：尝试MCP计算器服务，理解模型上下文协议基础

## 总结

干得好！你已经探索了一些真实应用：

- 同时在浏览器和服务器端工作的多模态AI体验
- 使用现代Java框架和SDK实现的本地AI模型集成
- 你的第一个模型上下文协议服务，了解工具如何与AI集成

## 后续步骤

[第5章：负责任的生成式AI](../05-ResponsibleGenAI/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免责声明**：
本文件由 AI 翻译服务 [Co-op Translator](https://github.com/Azure/co-op-translator) 翻译完成。尽管我们力求准确，但请注意，自动翻译可能包含错误或不准确之处。原始语言版文件应视为权威来源。对于重要信息，建议使用专业人工翻译。我们对因使用本翻译而产生的任何误解或误释不承担责任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->