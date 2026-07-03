# Aplicações Práticas & Projetos

## O Que Vai Aprender
Nesta secção iremos demonstrar três aplicações práticas que mostram padrões de desenvolvimento de IA generativa com Java:
- Criar um Gerador de Histórias de Animais de Estimação multimodal combinando IA do lado do cliente e do lado do servidor
- Implementar integração local de modelos de IA com a demo Foundry Local Spring Boot
- Desenvolver um serviço de Protocolo de Contexto de Modelo (MCP) com o exemplo da Calculadora

## Índice

- [Introdução](#introdução)
  - [Demo Foundry Local Spring Boot](#demo-foundry-local-spring-boot)
  - [Gerador de Histórias de Animais de Estimação](#gerador-de-histórias-de-animais-de-estimação)
  - [Serviço MCP Calculadora (Demo MCP Acessível a Iniciantes)](#serviço-mcp-calculadora-demo-mcp-acessível-a-iniciantes)
- [Progressão de Aprendizagem](#progressão-de-aprendizagem)
- [Resumo](#resumo)
- [Próximos Passos](#próximos-passos)

## Introdução

Este capítulo apresenta **projetos de exemplo** que demonstram padrões de desenvolvimento de IA generativa com Java. Cada projeto é totalmente funcional e demonstra tecnologias específicas de IA, padrões arquitetónicos e boas práticas que pode adaptar para as suas próprias aplicações.

### Demo Foundry Local Spring Boot

A **[Demo Foundry Local Spring Boot](foundrylocal/README.md)** demonstra como integrar com modelos de IA locais usando o **OpenAI Java SDK**. Mostra como conectar a modelos a correr no Foundry Local (ex: **Phi-4-mini**), com deteção automática de modelos, permitindo executar aplicações de IA sem depender de serviços na cloud.

### Gerador de Histórias de Animais de Estimação

O **[Gerador de Histórias de Animais de Estimação](petstory/README.md)** é uma aplicação web Spring Boot envolvente que demonstra **processamento de IA multimodal** para gerar histórias criativas sobre animais de estimação. Combina capacidades de IA do lado do cliente e do lado do servidor usando transformer.js para interações de IA no navegador e o OpenAI SDK para processamento do lado do servidor.

### Serviço MCP Calculadora (Demo MCP Acessível a Iniciantes)

O **[Serviço MCP Calculadora](calculator/README.md)** é uma demonstração simples do **Protocolo de Contexto de Modelo (MCP)** usando Spring AI. Fornece uma introdução acessível aos conceitos MCP, mostrando como criar um Servidor MCP básico que interage com clientes MCP.

## Progressão de Aprendizagem

Estes projetos estão desenhados para se basearem em conceitos dos capítulos anteriores:

1. **Começar Simples**: Inicie com a Demo Foundry Local Spring Boot para compreender a integração básica de IA com modelos locais
2. **Adicionar Interatividade**: Progrida para o Gerador de Histórias de Animais de Estimação para IA multimodal e interações baseadas na web
3. **Aprender os Fundamentos do MCP**: Experimente o Serviço MCP Calculadora para compreender os fundamentos do Protocolo de Contexto de Modelo

## Resumo

Bom trabalho! Agora explorou algumas aplicações reais:

- Experiências de IA multimodal que funcionam tanto no navegador como no servidor
- Integração local de modelos de IA usando frameworks e SDKs modernos Java
- O seu primeiro serviço de Protocolo de Contexto de Modelo para ver como as ferramentas se integram com IA

## Próximos Passos

[Capítulo 5: IA Generativa Responsável](../05-ResponsibleGenAI/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Aviso Legal**:
Este documento foi traduzido utilizando o serviço de tradução automática [Co-op Translator](https://github.com/Azure/co-op-translator). Embora nos esforcemos pela precisão, esteja ciente de que traduções automáticas podem conter erros ou imprecisões. O documento original na sua língua nativa deve ser considerado a fonte autorizada. Para informações críticas, recomenda-se tradução profissional humana. Não nos responsabilizamos por quaisquer mal-entendidos ou interpretações incorretas resultantes da utilização desta tradução.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->