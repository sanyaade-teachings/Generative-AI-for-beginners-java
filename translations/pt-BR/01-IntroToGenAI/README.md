# Introdução à IA Generativa - Edição Java

## O que você vai aprender

- **Fundamentos da IA Generativa** incluindo LLMs, engenharia de prompts, tokens, embeddings e bancos de dados vetoriais
- **Comparar ferramentas de desenvolvimento de IA em Java** incluindo Azure OpenAI SDK, Spring AI e OpenAI Java SDK
- **Descobrir o Protocolo de Contexto do Modelo** e seu papel na comunicação entre agentes de IA

## Índice

- [Introdução](#introdução)
- [Uma rápida revisão dos conceitos de IA Generativa](#uma-rápida-revisão-dos-conceitos-de-ia-generativa)
- [Revisão da engenharia de prompts](#revisão-da-engenharia-de-prompts)
- [Tokens, embeddings e agentes](#tokens-embeddings-e-agentes)
- [Ferramentas e bibliotecas de desenvolvimento de IA para Java](#ferramentas-e-bibliotecas-de-desenvolvimento-de-ia-para-java)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [Resumo](#resumo)
- [Próximos Passos](#próximos-passos)

## Introdução

Bem-vindo ao primeiro capítulo de IA Generativa para Iniciantes - Edição Java! Esta lição fundamental apresenta os conceitos centrais da IA generativa e como trabalhar com eles usando Java. Você aprenderá sobre os blocos essenciais de construção de aplicações de IA, incluindo Grandes Modelos de Linguagem (LLMs), tokens, embeddings e agentes de IA. Também exploraremos as principais ferramentas Java que você usará ao longo deste curso.

### Uma rápida revisão dos conceitos de IA Generativa

IA Generativa é um tipo de inteligência artificial que cria novo conteúdo, como texto, imagens ou código, com base em padrões e relacionamentos aprendidos a partir de dados. Modelos de IA generativa podem gerar respostas semelhantes às humanas, entender contexto e, às vezes, até criar conteúdo que parece humano.

Ao desenvolver suas aplicações de IA em Java, você trabalhará com **modelos de IA generativa** para criar conteúdo. Algumas capacidades dos modelos de IA generativa incluem:

- **Geração de Texto**: Criar texto semelhante ao humano para chatbots, conteúdo e finalização de texto.
- **Geração e Análise de Imagens**: Produzir imagens realistas, melhorar fotos e detectar objetos.
- **Geração de Código**: Escrever trechos de código ou scripts.

Existem tipos específicos de modelos otimizados para diferentes tarefas. Por exemplo, tanto **Modelos de Linguagem Pequenos (SLMs)** quanto **Grandes Modelos de Linguagem (LLMs)** podem lidar com geração de texto, sendo que LLMs geralmente oferecem melhor desempenho para tarefas complexas. Para tarefas relacionadas a imagens, você usaria modelos de visão especializados ou modelos multimodais.

![Figura: Tipos e casos de uso de modelos de IA generativa.](../../../translated_images/pt-BR/llms.225ca2b8a0d34473.webp)

Claro que as respostas desses modelos não são perfeitas o tempo todo. Você provavelmente já ouviu falar sobre modelos "alucinandos" ou gerando informações incorretas de uma forma autoritária. Mas você pode ajudar a guiar o modelo para gerar respostas melhores fornecendo a ele instruções claras e contexto. É aí que entra a **engenharia de prompts**.

#### Revisão da engenharia de prompts

Engenharia de prompts é a prática de projetar entradas eficazes para guiar modelos de IA a saídas desejadas. Isso envolve:

- **Clareza**: Tornar as instruções claras e inequívocas.
- **Contexto**: Fornecer informações de fundo necessárias.
- **Restrições**: Especificar quaisquer limitações ou formatos.

Algumas boas práticas para engenharia de prompts incluem design de prompt, instruções claras, divisão de tarefas, aprendizado one-shot e few-shot, e ajuste de prompt. Testar diferentes prompts é essencial para encontrar o que funciona melhor para seu caso de uso específico.

Ao desenvolver aplicações, você trabalhará com diferentes tipos de prompts:
- **Prompts do sistema**: Definem as regras básicas e o contexto para o comportamento do modelo
- **Prompts do usuário**: Os dados de entrada dos usuários da sua aplicação
- **Prompts do assistente**: As respostas do modelo baseadas nos prompts do sistema e do usuário

> **Saiba mais**: Saiba mais sobre engenharia de prompts no [capítulo de Engenharia de Prompts do curso GenAI para Iniciantes](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### Tokens, embeddings e agentes

Ao trabalhar com modelos de IA generativa, você encontrará termos como **tokens**, **embeddings**, **agentes** e **Protocolo de Contexto do Modelo (MCP)**. Aqui está uma visão detalhada desses conceitos:

- **Tokens**: Tokens são a menor unidade de texto em um modelo. Podem ser palavras, caracteres ou subpalavras. Tokens são usados para representar dados de texto em um formato que o modelo pode entender. Por exemplo, a frase "The quick brown fox jumped over the lazy dog" pode ser tokenizada como ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] ou ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"] dependendo da estratégia de tokenização.

![Figura: Exemplo de tokens em IA generativa quebrando palavras em tokens](../../../translated_images/pt-BR/tokens.6283ed277a2ffff4.webp)

Tokenização é o processo de dividir o texto nessas unidades menores. Isso é crucial porque modelos operam sobre tokens e não sobre texto bruto. A quantidade de tokens em um prompt afeta o comprimento e a qualidade da resposta do modelo, pois modelos têm limites de tokens para sua janela de contexto (exemplo: 128K tokens para contexto total do GPT-4o, incluindo entrada e saída).

  Em Java, você pode usar bibliotecas como o OpenAI SDK para tratar tokenização automaticamente ao enviar requisições para modelos de IA.

- **Embeddings**: Embeddings são representações vetoriais de tokens que capturam significado semântico. São representações numéricas (tipicamente arrays de números em ponto flutuante) que permitem aos modelos entender relacionamentos entre palavras e gerar respostas contextualmente relevantes. Palavras semelhantes têm embeddings semelhantes, permitindo ao modelo compreender conceitos como sinônimos e relações semânticas.

![Figura: Embeddings](../../../translated_images/pt-BR/embedding.398e50802c0037f9.webp)

  Em Java, você pode gerar embeddings usando o OpenAI SDK ou outras bibliotecas que suportem geração de embeddings. Esses embeddings são essenciais para tarefas como busca semântica, onde se deseja encontrar conteúdo semelhante baseado em significado em vez de correspondência textual exata.

- **Bancos de dados vetoriais**: Bancos de dados vetoriais são sistemas de armazenamento especializados otimizados para embeddings. Eles possibilitam buscas de similaridade eficientes e são cruciais para padrões de Geração Recuperada (RAG - Retrieval-Augmented Generation), onde você precisa encontrar informação relevante em grandes conjuntos de dados baseando-se em similaridade semântica, não em correspondência exata.

![Figura: Arquitetura de banco de dados vetorial mostrando como embeddings são armazenados e recuperados para busca por similaridade.](../../../translated_images/pt-BR/vector.f12f114934e223df.webp)

> **Nota**: Neste curso, não cobriremos bancos de dados vetoriais, mas achamos importante mencioná-los pois são comumente usados em aplicações reais.

- **Agentes & MCP**: Componentes de IA que interagem autonomamente com modelos, ferramentas e sistemas externos. O Protocolo de Contexto do Modelo (MCP) oferece uma forma padronizada para agentes acessarem de modo seguro fontes de dados externas e ferramentas. Saiba mais em nosso curso [MCP para Iniciantes](https://github.com/microsoft/mcp-for-beginners).

Em aplicações de IA em Java, você usará tokens para processamento de texto, embeddings para busca semântica e RAG, bancos de dados vetoriais para recuperação de dados e agentes com MCP para construir sistemas inteligentes que usam ferramentas.

![Figura: Como um prompt se torna uma resposta — tokens, vetores, busca RAG opcional, pensamento do LLM e um agente MCP em um fluxo rápido.](../../../translated_images/pt-BR/flow.f4ef62c3052d12a8.webp)

### Ferramentas e bibliotecas de desenvolvimento de IA para Java

Java oferece excelentes ferramentas para desenvolvimento de IA. Há três bibliotecas principais que exploraremos ao longo do curso - OpenAI Java SDK, Azure OpenAI SDK e Spring AI.

Aqui está uma tabela de referência rápida mostrando qual SDK é usado nos exemplos de cada capítulo:

| Capítulo | Exemplo | SDK |
|---------|--------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**Links para documentação dos SDKs:**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

O OpenAI SDK é a biblioteca oficial em Java para a API OpenAI. Ele fornece uma interface simples e consistente para interação com os modelos da OpenAI, facilitando a integração de capacidades de IA em aplicações Java. A aplicação Pet Story e o exemplo Foundry Local do Capítulo 4 demonstram a abordagem do OpenAI SDK com Azure AI Foundry.

#### Spring AI

Spring AI é um framework abrangente que traz capacidades de IA para aplicações Spring, oferecendo uma camada de abstração consistente entre diferentes provedores de IA. Ele se integra perfeitamente ao ecossistema Spring, tornando-o a escolha ideal para aplicações empresariais Java que necessitam de funcionalidades de IA.

A força do Spring AI está em sua integração perfeita com o ecossistema Spring, facilitando a construção de aplicações de IA prontas para produção com padrões familiares do Spring como injeção de dependência, gerenciamento de configuração e frameworks de teste. Você usará Spring AI nos Capítulos 2 e 4 para construir aplicações que aproveitam tanto o OpenAI quanto o Protocolo de Contexto do Modelo (MCP) das bibliotecas Spring AI.

##### Protocolo de Contexto do Modelo (MCP)

O [Protocolo de Contexto do Modelo (MCP)](https://modelcontextprotocol.io/) é um padrão emergente que permite que aplicações de IA interajam de forma segura com fontes externas de dados e ferramentas. O MCP oferece uma forma padronizada para modelos de IA acessarem informações contextuais e executarem ações em suas aplicações.

No Capítulo 4, você construirá um serviço simples de calculadora MCP que demonstra os fundamentos do Protocolo de Contexto do Modelo com Spring AI, mostrando como criar integrações básicas de ferramentas e arquiteturas de serviços.

#### Azure OpenAI Java SDK

A biblioteca cliente Azure OpenAI para Java é uma adaptação das APIs REST da OpenAI que fornece uma interface idiomática e integração com o restante do ecossistema do Azure SDK. No Capítulo 3, você construirá aplicações usando o Azure OpenAI SDK, incluindo aplicações de chat, chamadas de função e padrões RAG (Geração Recuperada).

> Nota: O Azure OpenAI SDK está atrás do OpenAI Java SDK em termos de funcionalidades, por isso para projetos futuros considere usar o OpenAI Java SDK.

## Resumo

Isso conclui os fundamentos! Agora você entende:

- Os conceitos centrais por trás da IA generativa - desde LLMs e engenharia de prompts até tokens, embeddings e bancos de dados vetoriais
- Suas opções de ferramentas para desenvolvimento de IA em Java: Azure OpenAI SDK, Spring AI e OpenAI Java SDK
- O que é o Protocolo de Contexto do Modelo e como ele permite que agentes de IA trabalhem com ferramentas externas

## Próximos Passos

[Capítulo 2: Configurando o Ambiente de Desenvolvimento](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Aviso Legal**:
Este documento foi traduzido usando o serviço de tradução por IA [Co-op Translator](https://github.com/Azure/co-op-translator). Embora nos esforcemos pela precisão, por favor, esteja ciente de que traduções automatizadas podem conter erros ou imprecisões. O documento original em seu idioma nativo deve ser considerado a fonte autorizada. Para informações críticas, recomenda-se tradução profissional humana. Não nos responsabilizamos por quaisquer mal-entendidos ou interpretações incorretas decorrentes do uso desta tradução.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->