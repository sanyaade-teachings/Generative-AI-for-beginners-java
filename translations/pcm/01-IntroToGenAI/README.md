# Introduction to Generative AI - Java Edition

## Wetin You Go Learn

- **Generative AI fundamentals** wey include LLMs, prompt engineering, tokens, embeddings, and vector databases
- **Compare Java AI development tools** wey include Azure OpenAI SDK, Spring AI, and OpenAI Java SDK
- **Discover the Model Context Protocol** and how e dey work for AI agent communication

## Table of Contents

- [Introduction](#introduction)
- [A quick refresh on Generative AI concepts](#a-quick-refresh-on-generative-ai-concepts)
- [Prompt engineering review](#prompt-engineering-review)
- [Tokens, embeddings, and agents](#tokens-embeddings-and-agents)
- [AI Development Tools and Libraries for Java](#ai-development-tools-and-libraries-for-java)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [Summary](#summary)
- [Next Steps](#next-steps)

## Introduction

Welcome to di first chapter of Generative AI for Beginners - Java Edition! Dis foundational lesson go introduce you to di main concepts of generative AI and how you fit take work wit dem using Java. You go learn about di essential building blocks of AI applications, wey include Large Language Models (LLMs), tokens, embeddings, and AI agents. We go also explore di main Java tooling wey you go use throughout dis course.

### A quick refresh on Generative AI concepts

Generative AI na one kain artificial intelligence wey dey create new content, like text, images, or code, based on patterns and relationships wey e don learn from data. Generative AI models fit generate human-like responses, understand context, and sometimes even create content wey dey look like human be dat.

As you dey develop your Java AI applications, you go dey work wit **generative AI models** to create content. Some things wey generative AI models fit do include:

- **Text Generation**: To create human-like text for chatbots, content, and text completion.
- **Image Generation and Analysis**: To produce realistic images, improve photos, and detect objects.
- **Code Generation**: To write code snippets or scripts.

Some specific types of models dey wey dem optimize for different tasks. For example, both **Small Language Models (SLMs)** and **Large Language Models (LLMs)** fit do text generation, but LLMs dey usually give better performance for complex tasks. For image-related tasks, you go use special vision models or multi-modal models.

![Figure: Generative AI model types and use cases.](../../../translated_images/pcm/llms.225ca2b8a0d34473.webp)

Of course, di responses from these models no dey perfect all di time. You don probably hear about models "hallucinating" or generating wrong information in one way wey e sure. But you fit help guide di model to generate better responses by giving dem clear instructions and context. Na here **prompt engineering** come in.

#### Prompt engineering review

Prompt engineering na di practice of designing correct inputs wey go guide AI models to give the output wey you want. E involve:

- **Clarity**: Make instructions clear and no get chance for confusion.
- **Context**: Give di necessary background information.
- **Constraints**: Talk any limitations or formats as e suppose be.

Some better ways for prompt engineering na prompt design, clear instructions, dividing di task, one-shot and few-shot learning, and prompt tuning. To try different prompts dey important to know wetin work well for your specific case.

When you dey build applications, you go handle different prompt types:
- **System prompts**: Set di base rules and context for how di model go behave
- **User prompts**: Di input data wey your application users go give
- **Assistant prompts**: Di model responses based on system and user prompts

> **Learn more**: Learn more about prompt engineering for [Prompt Engineering chapter of GenAI for Beginners course](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### Tokens, embeddings, and agents

When you dey work wit generative AI models, you go see terms like **tokens**, **embeddings**, **agents**, and **Model Context Protocol (MCP)**. Here na detailed overview of these concepts:

- **Tokens**: Tokens na di smallest tin for text inside model. Dem fit be words, characters, or subwords. Tokens dey used to represent text data in one format wey model fit understand. For example, di sentence "The quick brown fox jumped over the lazy dog" fit tokenize as ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] or ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"] depending on di tokenization way you use.

![Figure: Generative AI tokens example of breaking words into tokens](../../../translated_images/pcm/tokens.6283ed277a2ffff4.webp)

Tokenization na di process of breaking text down into dem small units like dis. E important because models dey operate on tokens, no be raw text. Di number of tokens for prompt fit affect di length and quality of di model response, as models get token limits for their context window (for example, 128K tokens for GPT-4o total context, including both input and output).

  For Java, you fit use libraries like di OpenAI SDK to handle tokenization automatically when you dey send requests to AI models.

- **Embeddings**: Embeddings na vector representations of tokens wey capture di meaning inside. Dem na numerical representations (usually arrays of floating-point numbers) wey allow models to understand relationships between words and generate responses wey correct according to context. Words wey similar get similar embeddings, this one help di model understand concepts like synonyms and semantic connections.

![Figure: Embeddings](../../../translated_images/pcm/embedding.398e50802c0037f9.webp)

  For Java, you fit generate embeddings using di OpenAI SDK or other libraries wey support embedding generation. These embeddings dey important for tasks like semantic search, where you want to find content wey similar based on meaning, no be exact text match.

- **Vector databases**: Vector databases na special storage systems wey dem optimize for embeddings. Dem allow fast similarity search and e important for Retrieval-Augmented Generation (RAG) patterns where you want find correct information from big data sets based on semantic similarity, no be exact matches.

![Figure: Vector database architecture showing how embeddings are stored and retrieved for similarity search.](../../../translated_images/pcm/vector.f12f114934e223df.webp)

> **Note**: For dis course, we no go cover Vector databases but we think say e good make we mention am as dem dey commonly use for real-world applications.

- **Agents & MCP**: AI parts wey dey work by themselves with models, tools, and external systems. Di Model Context Protocol (MCP) dey give standard way for agents to safely access external data and tools. Learn more for our [MCP for Beginners](https://github.com/microsoft/mcp-for-beginners) course.

For Java AI applications, you go use tokens for text processing, embeddings for semantic search and RAG, vector databases for data retrieval, and agents with MCP to build smart systems wey fit use tools. 

![Figure: how a prompt becomes a reply—tokens, vectors, optional RAG lookup, LLM thinking, and an MCP agent all in one quick flow..](../../../translated_images/pcm/flow.f4ef62c3052d12a8.webp)

### AI Development Tools and Libraries for Java

Java get better tooling for AI development. We get three main libraries wey we go explore throughout dis course - OpenAI Java SDK, Azure OpenAI SDK, and Spring AI.

Here na one quick reference table wey show which SDK dey use for each chapter example:

| Chapter | Sample | SDK |
|---------|--------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**SDK Documentation Links:**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

Di OpenAI SDK na di official Java library for di OpenAI API. E provide a simple and consistent interface to interact with OpenAI's models, making am easy to add AI capabilities inside Java applications. Chapter 4 Pet Story application and Foundry Local example show how OpenAI SDK work with Azure AI Foundry.

#### Spring AI

Spring AI na one big framework wey dey bring AI power to Spring applications, e provide one consistent abstraction layer for different AI providers. E join smoothly wit di Spring ecosystem, making am di correct choice for enterprise Java apps wey need AI.

Spring AI strong pass for how e join well with di Spring ecosystem, e make am easy to build production-ready AI apps wey dey use familiar Spring ways like dependency injection, configuration management, and testing frameworks. You go use Spring AI for Chapter 2 and 4 to build apps wey dey use both OpenAI and Model Context Protocol (MCP) Spring AI libraries.

##### Model Context Protocol (MCP)

Di [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) na one new standardwey allow AI applications to dey interact safely with external data sources and tools. MCP dey give standard way for AI models to access contextual info and do actions for your applications.

For Chapter 4, you go build one simple MCP calculator service wey show main principles of Model Context Protocol with Spring AI, show how to do basic tool integrations and service architectures.

#### Azure OpenAI Java SDK

Di Azure OpenAI client library for Java na adaptation of OpenAI's REST APIs wey provide idiomatic interface and integrate with di whole Azure SDK ecosystem. For Chapter 3, you go build apps wey dey use Azure OpenAI SDK, including chat apps, function calling, and RAG (Retrieval-Augmented Generation) patterns.

> Note: Azure OpenAI SDK never reach OpenAI Java SDK for features, so for future projects, try use OpenAI Java SDK.

## Summary

That na di end of di foundations! You don sabi:

- Di core concepts behind generative AI - from LLMs and prompt engineering to tokens, embeddings, and vector databases
- Your toolkit options for Java AI development: Azure OpenAI SDK, Spring AI, and OpenAI Java SDK
- Wetin di Model Context Protocol be and how e dey make AI agents fit work with external tools

## Next Steps

[Chapter 2: Setting Up the Development Environment](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Disclaimer**:
Dis document don translate wit AI translation service [Co-op Translator](https://github.com/Azure/co-op-translator). Even tho we dey try make am correct, abeg make you know say automated translation fit get errors or mistakes. Di original document for dia own language na im be di correct source. For important info, make person wey sabi human translation do am. We no go responsible for any misunderstanding or wrong understanding wey fit happen because of dis translation.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->