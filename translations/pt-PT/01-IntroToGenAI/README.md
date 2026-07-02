# Introdução à IA Generativa - Edição Java

## O Que Vai Aprender

- **Fundamentos da IA Generativa** incluindo LLMs, engenharia de prompts, tokens, embeddings e bases de dados vetoriais
- **Comparar ferramentas de desenvolvimento de IA em Java** incluindo Azure OpenAI SDK, Spring AI e OpenAI Java SDK
- **Descobrir o Protocolo de Contexto de Modelo** e o seu papel na comunicação de agentes de IA

## Índice

- [Introdução](#introdução)
- [Uma rápida atualização sobre conceitos de IA Generativa](#uma-rápida-atualização-sobre-conceitos-de-ia-generativa)
- [Revisão da engenharia de prompts](#revisão-da-engenharia-de-prompts)
- [Tokens, embeddings e agentes](#tokens-embeddings-e-agentes)
- [Ferramentas e Bibliotecas de Desenvolvimento de IA para Java](#ferramentas-e-bibliotecas-de-desenvolvimento-de-ia-para-java)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [Resumo](#resumo)
- [Próximos Passos](#próximos-passos)

## Introdução

Bem-vindo ao primeiro capítulo de IA Generativa para Iniciantes - Edição Java! Esta lição fundamental apresenta os conceitos principais da IA generativa e como trabalhar com eles utilizando Java. Vai aprender sobre os blocos essenciais da construção de aplicações de IA, incluindo Grandes Modelos de Linguagem (LLMs), tokens, embeddings e agentes de IA. Também exploraremos as principais ferramentas Java que irá usar ao longo deste curso.

### Uma rápida atualização sobre conceitos de IA Generativa

A IA generativa é um tipo de inteligência artificial que cria novo conteúdo, como texto, imagens ou código, baseado em padrões e relações aprendidas a partir dos dados. Os modelos de IA generativa podem gerar respostas semelhantes às humanas, compreender contexto e, por vezes, até criar conteúdo que parece feito por humanos.

Ao desenvolver as suas aplicações de IA em Java, trabalhará com **modelos de IA generativa** para criar conteúdo. Algumas capacidades dos modelos de IA generativa incluem:

- **Geração de Texto**: Criação de texto humano para chatbots, conteúdos e completamento de texto.
- **Geração e Análise de Imagens**: Produção de imagens realistas, melhoria de fotografias e deteção de objetos.
- **Geração de Código**: Escrita de fragmentos de código ou scripts.

Existem tipos específicos de modelos otimizados para tarefas diferentes. Por exemplo, tanto **Modelos de Linguagem Pequenos (SLMs)** como **Grandes Modelos de Linguagem (LLMs)** conseguem lidar com a geração de texto, sendo que os LLMs geralmente apresentam melhor desempenho para tarefas mais complexas. Para tarefas relacionadas com imagem, utilizará modelos de visão especializados ou modelos multimodais.

![Figure: Generative AI model types and use cases.](../../../translated_images/pt-PT/llms.225ca2b8a0d34473.webp)

Claro que as respostas destes modelos nem sempre são perfeitas. Provavelmente já ouviu falar de modelos a "alucinar" ou a gerar informações incorretas de forma autoritária. Mas pode ajudar a guiar o modelo a gerar melhores respostas fornecendo-lhe instruções e contexto claros. É aqui que entra a **engenharia de prompts**.

#### Revisão da engenharia de prompts

A engenharia de prompts é a prática de conceber entradas eficazes para orientar os modelos de IA para os resultados desejados. Envolve:

- **Clareza**: Tornar as instruções claras e inequívocas.
- **Contexto**: Fornecer informação de fundo necessária.
- **Restrições**: Especificar eventuais limitações ou formatos.

Algumas boas práticas na engenharia de prompts incluem design do prompt, instruções claras, decomposição da tarefa, aprendizagem one-shot e few-shot, e ajuste do prompt. Testar diferentes prompts é essencial para encontrar o que funciona melhor para o seu caso de uso específico.

Ao desenvolver aplicações, trabalhará com diferentes tipos de prompts:
- **Prompts do sistema**: Definem as regras base e o contexto do comportamento do modelo
- **Prompts do utilizador**: Os dados de entrada dos seus utilizadores
- **Prompts do assistente**: As respostas do modelo baseadas nos prompts do sistema e do utilizador

> **Saiba mais**: Saiba mais sobre engenharia de prompts no [capítulo de Engenharia de Prompts do curso GenAI para Iniciantes](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### Tokens, embeddings e agentes

Ao trabalhar com modelos de IA generativa, vai encontrar termos como **tokens**, **embeddings**, **agentes** e **Protocolo de Contexto de Modelo (MCP)**. Aqui está uma visão detalhada destes conceitos:

- **Tokens**: Tokens são a menor unidade de texto num modelo. Podem ser palavras, caracteres ou subpalavras. Os tokens são usados para representar dados textuais num formato que o modelo compreende. Por exemplo, a frase "The quick brown fox jumped over the lazy dog" pode ser tokenizada como ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] ou ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"] dependendo da estratégia de tokenização.

![Figure: Generative AI tokens example of breaking words into tokens](../../../translated_images/pt-PT/tokens.6283ed277a2ffff4.webp)

A tokenização é o processo de dividir o texto nestas unidades menores. Isto é crucial porque os modelos operam com tokens em vez do texto bruto. O número de tokens num prompt afeta o tamanho e qualidade da resposta do modelo, pois os modelos têm limites de tokens para a sua janela de contexto (exemplo, 128K tokens para o contexto total do GPT-4o, incluindo input e output).

  Em Java, pode usar bibliotecas como o OpenAI SDK para tratar automaticamente a tokenização ao enviar pedidos a modelos de IA.

- **Embeddings**: Embeddings são representações vetoriais de tokens que capturam o significado semântico. São representações numéricas (tipicamente arrays de números de ponto flutuante) que permitem aos modelos compreender relações entre palavras e gerar respostas contextualmente relevantes. Palavras semelhantes têm embeddings semelhantes, permitindo ao modelo entender conceitos como sinónimos e relações semânticas.

![Figure: Embeddings](../../../translated_images/pt-PT/embedding.398e50802c0037f9.webp)

  Em Java, pode gerar embeddings usando o OpenAI SDK ou outras bibliotecas que suportem a geração de embeddings. Estes embeddings são essenciais para tarefas como pesquisa semântica, onde pretende encontrar conteúdo semelhante baseado no significado em vez de correspondências exatas no texto.

- **Bases de dados vetoriais**: Bases de dados vetoriais são sistemas de armazenamento especializados otimizados para embeddings. Permitem pesquisas eficientes por similaridade e são cruciais para padrões Retrieval-Augmented Generation (RAG), onde precisa de encontrar informação relevante em grandes conjuntos de dados com base na semelhança semântica e não em correspondências exatas.

![Figure: Vector database architecture showing how embeddings are stored and retrieved for similarity search.](../../../translated_images/pt-PT/vector.f12f114934e223df.webp)

> **Nota**: Neste curso, não vamos abordar bases de dados vetoriais, mas achamos importante mencioná-las pois são frequentemente usadas em aplicações do mundo real.

- **Agentes & MCP**: Componentes de IA que interagem autonomamente com modelos, ferramentas e sistemas externos. O Protocolo de Contexto de Modelo (MCP) fornece uma forma padronizada para agentes acederem de forma segura a fontes de dados e ferramentas externas. Saiba mais no nosso curso [MCP para Iniciantes](https://github.com/microsoft/mcp-for-beginners).

Nas aplicações Java de IA, vai usar tokens para processamento de texto, embeddings para pesquisa semântica e RAG, bases de dados vetoriais para recuperação de dados, e agentes com MCP para construir sistemas inteligentes que utilizam ferramentas. 

![Figure: how a prompt becomes a reply—tokens, vectors, optional RAG lookup, LLM thinking, and an MCP agent all in one quick flow..](../../../translated_images/pt-PT/flow.f4ef62c3052d12a8.webp)

### Ferramentas e Bibliotecas de Desenvolvimento de IA para Java

Java oferece um excelente conjunto de ferramentas para desenvolvimento de IA. Existem três bibliotecas principais que iremos explorar ao longo deste curso – OpenAI Java SDK, Azure OpenAI SDK e Spring AI.

Aqui está uma tabela de referência rápida que mostra qual SDK é usado nos exemplos de cada capítulo:

| Capítulo | Exemplo | SDK |
|---------|--------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**Links para Documentação dos SDKs:**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

O OpenAI SDK é a biblioteca oficial Java para a API OpenAI. Fornece uma interface simples e consistente para interagir com os modelos da OpenAI, facilitando a integração de capacidades de IA em aplicações Java. A aplicação Pet Story e o exemplo Foundry Local do Capítulo 4 demonstram a abordagem do OpenAI SDK com Azure AI Foundry.

#### Spring AI

Spring AI é um framework abrangente que traz capacidades de IA para aplicações Spring, proporcionando uma camada de abstração consistente através de diferentes provedores de IA. Integra-se de forma fluida com o ecossistema Spring, sendo a escolha ideal para aplicações Java empresariais que precisam de capacidades de IA.

A força do Spring AI reside na sua integração perfeita com o ecossistema Spring, facilitando a construção de aplicações de IA prontas para produção com padrões Spring familiares como injeção de dependência, gestão de configuração e frameworks de teste. Vai usar Spring AI nos Capítulos 2 e 4 para construir aplicações que aproveitam tanto a OpenAI como o Protocolo de Contexto de Modelo (MCP) com as bibliotecas Spring AI.

##### Protocolo de Contexto de Modelo (MCP)

O [Protocolo de Contexto de Modelo (MCP)](https://modelcontextprotocol.io/) é um standard emergente que permite que aplicações de IA interajam de forma segura com fontes de dados e ferramentas externas. O MCP fornece uma forma padronizada para os modelos de IA acederem a informação contextual e executarem ações nas suas aplicações.

No Capítulo 4, vai construir um serviço simples de calculadora MCP que demonstra os fundamentos do Protocolo de Contexto de Modelo com Spring AI, mostrando como criar integrações básicas de ferramentas e arquiteturas de serviços.

#### Azure OpenAI Java SDK

A biblioteca cliente Azure OpenAI para Java é uma adaptação das APIs REST da OpenAI que fornece uma interface idiomática e integração com o resto do ecossistema Azure SDK. No Capítulo 3, vai construir aplicações usando o Azure OpenAI SDK, incluindo aplicações de chat, chamada de funções e padrões RAG (Retrieval-Augmented Generation).

> Nota: O Azure OpenAI SDK fica atrás do OpenAI Java SDK em termos de funcionalidades, pelo que para projetos futuros, considere usar o OpenAI Java SDK.

## Resumo

E assim terminam os fundamentos! Agora compreende:

- Os conceitos essenciais por detrás da IA generativa – desde LLMs e engenharia de prompts a tokens, embeddings e bases de dados vetoriais
- As suas opções de toolkit para desenvolvimento de IA em Java: Azure OpenAI SDK, Spring AI e OpenAI Java SDK
- O que é o Protocolo de Contexto de Modelo e como permite que agentes de IA trabalhem com ferramentas externas

## Próximos Passos

[Capítulo 2: Configurar o Ambiente de Desenvolvimento](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Aviso Legal**:
Este documento foi traduzido utilizando o serviço de tradução automática [Co-op Translator](https://github.com/Azure/co-op-translator). Embora nos esforcemos pela precisão, esteja ciente de que traduções automáticas podem conter erros ou imprecisões. O documento original na sua língua nativa deve ser considerado a fonte autorizada. Para informações críticas, recomenda-se tradução profissional humana. Não nos responsabilizamos por quaisquer mal-entendidos ou interpretações incorretas resultantes da utilização desta tradução.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->