# Aplicações Práticas & Projetos

## O Que Você Vai Aprender
Nesta seção, mostraremos três aplicações práticas que demonstram padrões de desenvolvimento de IA generativa com Java:
- Criar um Gerador de Histórias de Animais multi-modal combinando IA client-side e server-side
- Implementar integração de modelo de IA local com a demo Foundry Local Spring Boot
- Desenvolver um serviço Model Context Protocol (MCP) com o exemplo da Calculadora

## Tabela de Conteúdos

- [Introdução](#introdução)
  - [Demo Foundry Local Spring Boot](#demo-foundry-local-spring-boot)
  - [Gerador de Histórias de Animais](#gerador-de-histórias-de-animais)
  - [Serviço MCP Calculadora (Demo MCP Amigável para Iniciantes)](#serviço-mcp-calculadora-demo-mcp-amigável-para-iniciantes)
- [Progressão de Aprendizado](#progressão-de-aprendizado)
- [Resumo](#resumo)
- [Próximos Passos](#próximos-passos)

## Introdução

Este capítulo apresenta **projetos de exemplo** que demonstram padrões de desenvolvimento de IA generativa com Java. Cada projeto é totalmente funcional e demonstra tecnologias específicas de IA, padrões arquiteturais e melhores práticas que você pode adaptar para suas próprias aplicações.

### Demo Foundry Local Spring Boot

A **[Demo Foundry Local Spring Boot](foundrylocal/README.md)** demonstra como integrar com modelos de IA locais usando o **OpenAI Java SDK**. Ela mostra a conexão com modelos rodando no Foundry Local (por exemplo, **Phi-4-mini**), com detecção automática de modelo, permitindo que você execute aplicações de IA sem depender de serviços na nuvem.

### Gerador de Histórias de Animais

O **[Gerador de Histórias de Animais](petstory/README.md)** é uma aplicação web Spring Boot envolvente que demonstra **processamento de IA multi-modal** para gerar histórias criativas sobre animais de estimação. Combina capacidades de IA client-side e server-side usando transformer.js para interações de IA no navegador e o OpenAI SDK para processamento no servidor.

### Serviço MCP Calculadora (Demo MCP Amigável para Iniciantes)

O **[Serviço MCP Calculadora](calculator/README.md)** é uma demonstração simples do **Model Context Protocol (MCP)** utilizando Spring AI. Ele oferece uma introdução amigável aos conceitos de MCP, mostrando como criar um MCP Server básico que interage com clientes MCP.

## Progressão de Aprendizado

Estes projetos foram criados para construir sobre os conceitos dos capítulos anteriores:

1. **Comece Simples**: Inicie com a Demo Foundry Local Spring Boot para entender a integração básica de IA com modelos locais
2. **Adicione Interatividade**: Progrida para o Gerador de Histórias de Animais para IA multi-modal e interações baseadas na web
3. **Aprenda o Básico do MCP**: Experimente o Serviço MCP Calculadora para compreender os fundamentos do Model Context Protocol

## Resumo

Bom trabalho! Agora você explorou algumas aplicações reais:

- Experiências de IA multi-modal que funcionam tanto no navegador quanto no servidor
- Integração de modelos de IA locais usando frameworks Java modernos e SDKs
- Seu primeiro serviço Model Context Protocol para ver como ferramentas se integram com IA

## Próximos Passos

[Capítulo 5: IA Generativa Responsável](../05-ResponsibleGenAI/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Aviso Legal**:
Este documento foi traduzido usando o serviço de tradução por IA [Co-op Translator](https://github.com/Azure/co-op-translator). Embora nos esforcemos pela precisão, por favor, esteja ciente de que traduções automatizadas podem conter erros ou imprecisões. O documento original em seu idioma nativo deve ser considerado a fonte autorizada. Para informações críticas, recomenda-se tradução profissional humana. Não nos responsabilizamos por quaisquer mal-entendidos ou interpretações incorretas decorrentes do uso desta tradução.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->