# Configurar o Ambiente de Desenvolvimento para Generative AI para Java

> **Início Rápido:** Provisione os seus modelos de IA na **Azure AI Foundry** como código com Bicep + `azd` em poucos minutos — consulte o [Guia de Configuração da Azure AI Foundry](getting-started-azure-openai.md). A autenticação é **sem chave** (Microsoft Entra ID), portanto não há chaves de API para gerir.

## O que Você Vai Aprender

- Configurar um ambiente de desenvolvimento Java para aplicações de IA
- Escolher e configurar o seu ambiente de desenvolvimento preferido (cloud-first com Codespaces, contentor de desenvolvimento local ou configuração local completa)
- Testar a sua configuração conectando-se a um modelo Azure AI Foundry

## Índice

- [O que Você Vai Aprender](#o-que-você-vai-aprender)
- [Introdução](#introdução)
- [Passo 1: Configurar o Seu Ambiente de Desenvolvimento](#passo-1-configurar-o-seu-ambiente-de-desenvolvimento)
  - [Opção A: GitHub Codespaces (Recomendado)](#opção-a-github-codespaces-recomendado)
  - [Opção B: Contentor de Desenvolvimento Local](#opção-b-contentor-de-desenvolvimento-local)
  - [Opção C: Use a Sua Instalação Local Existente](#opção-c-use-a-sua-instalação-local-existente)
- [Passo 2: Provisionar Azure AI Foundry](#passo-2-provisionar-azure-ai-foundry)
- [Passo 3: Testar a Sua Configuração](#passo-3-testar-a-sua-configuração)
- [Resolução de Problemas](#resolução-de-problemas)
- [Resumo](#resumo)
- [Próximos Passos](#próximos-passos)

## Introdução

Este capítulo irá guiá-lo na configuração de um ambiente de desenvolvimento. Usaremos a **Azure AI Foundry** para os modelos ao longo deste curso. Você provisiona os modelos como código com Bicep e a Azure Developer CLI (`azd`), depois conecta-se com **autenticação sem chave** (Microsoft Entra ID) — sem chaves de API para copiar ou vazar.

**Nenhuma configuração local requerida!** Pode usar o GitHub Codespaces, que oferece um ambiente completo de desenvolvimento no seu navegador, e provisionar o Foundry diretamente daí.

Usamos a **Azure AI Foundry** neste curso porque é:
- **Provisionada como código** — um `azd up` implanta a conta e os deployments dos modelos
- **Sem chaves** — autentique-se com o seu login Azure ou uma identidade gerida
- **Pronta para produção** — o mesmo código funciona localmente e na Azure
- **Flexível** — troque modelos mudando o nome do deployment, não o código

> **Nota**: Os deployments da Azure AI Foundry são faturados por token (pague conforme o uso). Veja o [guia de configuração da Azure AI Foundry](getting-started-azure-openai.md) para detalhes sobre provisão, região e custos.


## Passo 1: Configurar o Seu Ambiente de Desenvolvimento

<a name="quick-start-cloud"></a>

Criámos um contentor de desenvolvimento pré-configurado para minimizar o tempo de configuração e assegurar que tem todas as ferramentas necessárias para este curso de Generative AI para Java. Escolha a sua abordagem de desenvolvimento preferida:

### Opções para Configurar o Ambiente:

#### Opção A: GitHub Codespaces (Recomendado)

**Comece a programar em 2 minutos - sem necessidade de configuração local!**

1. Faça um fork deste repositório para a sua conta GitHub
   > **Nota**: Se quiser editar a configuração básica, consulte a [Configuração do Contentor de Desenvolvimento](../../../.devcontainer/devcontainer.json)
2. Clique em **Code** → separador **Codespaces** → **...** → **New with options...**
3. Use os padrões – isto selecionará a **Configuração do contentor de desenvolvimento**: **Ambiente de Desenvolvimento Generative AI Java** contentor personalizado criado para este curso
4. Clique em **Create codespace**
5. Aguarde cerca de 2 minutos até o ambiente estar pronto
6. Continue para [Passo 2: Provisionar Azure AI Foundry](#passo-2-provisionar-azure-ai-foundry)

<img src="../../../translated_images/pt-PT/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: submenu Codespaces" width="50%">

<img src="../../../translated_images/pt-PT/image.833552b62eee7766.webp" alt="Screenshot: Novo com opções" width="50%">

<img src="../../../translated_images/pt-PT/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: Opções de criar codespace" width="50%">


> **Vantagens do Codespaces**:
> - Nenhuma instalação local requerida
> - Funciona em qualquer dispositivo com browser
> - Pré-configurado com todas as ferramentas e dependências
> - 60 horas grátis por mês para contas pessoais
> - Ambiente consistente para todos os aprendentes

#### Opção B: Contentor de Desenvolvimento Local

**Para programadores que preferem desenvolvimento local com Docker**

1. Faça fork e clone este repositório para a sua máquina local
   > **Nota**: Se quiser editar a configuração básica, consulte a [Configuração do Contentor de Desenvolvimento](../../../.devcontainer/devcontainer.json)
2. Instale o [Docker Desktop](https://www.docker.com/products/docker-desktop/) e o [VS Code](https://code.visualstudio.com/)
3. Instale a [extensão Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) no VS Code
4. Abra a pasta do repositório no VS Code
5. Quando perguntado, clique em **Reopen in Container** (ou use `Ctrl+Shift+P` → "Dev Containers: Reopen in Container")
6. Aguarde o contentor ser construído e iniciado
7. Continue para [Passo 2: Provisionar Azure AI Foundry](#passo-2-provisionar-azure-ai-foundry)

<img src="../../../translated_images/pt-PT/devcontainer.21126c9d6de64494.webp" alt="Screenshot: Configuração do contentor de desenvolvimento" width="50%">

<img src="../../../translated_images/pt-PT/image-3.bf93d533bbc84268.webp" alt="Screenshot: Construção do contentor completa" width="50%">

#### Opção C: Use a Sua Instalação Local Existente

**Para programadores com ambientes Java existentes**

Pré-requisitos:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html) 
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) ou o seu IDE preferido

Passos:
1. Clone este repositório para a sua máquina local
2. Abra o projeto no seu IDE
3. Continue para [Passo 2: Provisionar Azure AI Foundry](#passo-2-provisionar-azure-ai-foundry)

> **Dica Profissional**: Se tem uma máquina de especificações baixas mas quer usar VS Code localmente, use o GitHub Codespaces! Pode conectar o seu VS Code local a um Codespace cloud para o melhor dos dois mundos.

<img src="../../../translated_images/pt-PT/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: instância do contentor de desenvolvimento local criada" width="50%">


## Passo 2: Provisionar Azure AI Foundry

Implemente os modelos de IA do curso na Azure AI Foundry como código. Na raiz do repositório:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```

`azd` pede um nome para o ambiente e a região, provisona uma conta Azure AI Foundry com os deployments `gpt-4o-mini` e `text-embedding-3-small`, e escreve o endpoint no `.env` do exemplo — tudo com autenticação **sem chave** (sem chaves de API).

> **Guia completo:** Veja o [Guia de Configuração da Azure AI Foundry](getting-started-azure-openai.md) para pré-requisitos, alternativa manual (portal), orientações sobre regiões e notas sobre custos/limpeza.

## Passo 3: Testar a Sua Configuração

Depois de provisionar os modelos Foundry, teste a ligação com a app de exemplo em [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure).

1. Abra o terminal no seu ambiente de desenvolvimento.
2. Navegue até ao exemplo:
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
3. Certifique-se que está autenticado (a autenticação sem chave precisa de um token):
   ```bash
   az login
   ```
   > Se executou `azd up`, o ficheiro `.env` com o seu endpoint já foi criado automaticamente.
4. Execute a aplicação:
   ```bash
   mvn clean spring-boot:run
   ```

Deverá ver uma resposta do modelo `gpt-4o-mini`.

### Compreender o Código de Exemplo

O exemplo em `examples/basic-chat-azure` é uma aplicação Spring Boot que usa **Spring AI** para ligar à Azure AI Foundry com autenticação sem chave.

**O que este código faz:**
- **Liga-se** à Azure AI Foundry usando o seu login Azure (Microsoft Entra ID) — sem chave API
- **Envia** um prompt para o modelo `gpt-4o-mini`
- **Recebe** e apresenta a resposta da IA
- **Valida** que a sua configuração está a funcionar corretamente

**Dependência chave** (no `pom.xml`):
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```

**Configuração** (`application.yml`):
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

## Resumo

Ótimo! Agora tem tudo configurado:

- Modelos Azure AI Foundry provisionados como código com Bicep + `azd`
- Ambiente de desenvolvimento Java a funcionar (seja Codespaces, contentores de desenvolvimento, ou local)
- Ligação à Azure AI Foundry com autenticação sem chave (Microsoft Entra ID) — sem chaves API
- Testou que tudo funciona com um exemplo simples que comunica com o seu modelo

## Próximos Passos

[Capítulo 3: Técnicas Básicas de Generative AI](../03-CoreGenerativeAITechniques/README.md)

## Resolução de Problemas

Está com problemas? Aqui estão problemas comuns e soluções:

- **Autenticação falha (401/403)?** 
  - Execute `az login` — a autenticação é sem chave, por isso tem de estar autenticado
  - Verifique se a sua conta tem o papel **Cognitive Services OpenAI User** no recurso
  - Se acabou de provisionar, aguarde um minuto para a atribuição do papel ser propagada

- **Maven não encontrado?** 
  - Se usar contentores dev/Codespaces, o Maven deve estar pré-instalado
  - Para configuração local, certifique-se que Java 21+ e Maven 3.9+ estão instalados
  - Experimente `mvn --version` para verificar a instalação

- **`azd` não encontrado ou provisão falha?** 
  - Instale a [Azure Developer CLI](https://aka.ms/azure-dev/install) e execute `azd auth login`
  - Escolha uma região onde o `gpt-4o-mini` esteja disponível (ex: `eastus2`)
  - Consulte o [guia de configuração da Azure AI Foundry](getting-started-azure-openai.md) para detalhes

- **Contentor dev não inicia?** 
  - Certifique-se que o Docker Desktop está a correr (para desenvolvimento local)
  - Tente reconstruir o contentor: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **Erros de compilação da aplicação?**
  - Certifique-se que está no diretório correto: `02-SetupDevEnvironment/examples/basic-chat-azure`
  - Tente limpar e compilar novamente: `mvn clean compile`

> **Precisa de ajuda?**: Continua a ter problemas? Abra um issue no repositório e ajudamos você.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Aviso Legal**:
Este documento foi traduzido utilizando o serviço de tradução automática [Co-op Translator](https://github.com/Azure/co-op-translator). Embora nos esforcemos pela precisão, esteja ciente de que traduções automáticas podem conter erros ou imprecisões. O documento original na sua língua nativa deve ser considerado a fonte autorizada. Para informações críticas, recomenda-se tradução profissional humana. Não nos responsabilizamos por quaisquer mal-entendidos ou interpretações incorretas resultantes da utilização desta tradução.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->