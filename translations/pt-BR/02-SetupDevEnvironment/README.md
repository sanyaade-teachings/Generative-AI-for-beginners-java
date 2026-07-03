# Configurando o Ambiente de Desenvolvimento para IA Generativa para Java

> **Início Rápido:** Provisione seus modelos de IA no **Azure AI Foundry** como código com Bicep + `azd` em poucos minutos — veja o [Guia de Configuração do Azure AI Foundry](getting-started-azure-openai.md). A autenticação é **sem chave** (Microsoft Entra ID), então não há chaves de API para gerenciar.

## O Que Você Vai Aprender

- Configurar um ambiente de desenvolvimento Java para aplicações de IA
- Escolher e configurar seu ambiente de desenvolvimento preferido (priorizando cloud com Codespaces, container de desenvolvimento local ou configuração totalmente local)
- Testar sua configuração conectando-se a um modelo Azure AI Foundry

## Índice

- [O Que Você Vai Aprender](#o-que-você-vai-aprender)
- [Introdução](#introdução)
- [Passo 1: Configure Seu Ambiente de Desenvolvimento](#passo-1-configure-seu-ambiente-de-desenvolvimento)
  - [Opção A: GitHub Codespaces (Recomendado)](#opção-a-github-codespaces-recomendado)
  - [Opção B: Container de Desenvolvimento Local](#opção-b-container-de-desenvolvimento-local)
  - [Opção C: Use Sua Instalação Local Existente](#opção-c-use-sua-instalação-local-existente)
- [Passo 2: Provisione o Azure AI Foundry](#passo-2-provisione-o-azure-ai-foundry)
- [Passo 3: Teste Sua Configuração](#passo-3-teste-sua-configuração)
- [Resolução de Problemas](#resolução-de-problemas)
- [Resumo](#resumo)
- [Próximos Passos](#próximos-passos)

## Introdução

Este capítulo vai guiá-lo na configuração de um ambiente de desenvolvimento. Usaremos o **Azure AI Foundry** para os modelos ao longo deste curso. Você provisiona os modelos como código com Bicep e a CLI do Azure Developer (`azd`), depois se conecta com **autenticação sem chave** (Microsoft Entra ID) — sem chaves de API para copiar ou vazar.

**Nenhuma configuração local necessária!** Você pode usar o GitHub Codespaces, que fornece um ambiente completo de desenvolvimento no seu navegador, e provisionar o Foundry a partir daí.

Usamos o **Azure AI Foundry** para este curso porque ele é:
- **Provisionado como código** — um único `azd up` implanta a conta e os deployments dos modelos
- **Sem chave** — autentique-se com seu login do Azure ou uma identidade gerenciada
- **Pronto para produção** — o mesmo código roda localmente e no Azure
- **Flexível** — troque modelos mudando o nome do deployment, não seu código

> **Nota**: Os deployments do Azure AI Foundry são cobrados por token (pague conforme o uso). Veja o [guia de configuração do Azure AI Foundry](getting-started-azure-openai.md) para detalhes de provisão, região e custos.


## Passo 1: Configure Seu Ambiente de Desenvolvimento

<a name="quick-start-cloud"></a>

Criamos um container de desenvolvimento pré-configurado para minimizar o tempo de configuração e garantir que você tenha todas as ferramentas necessárias para este curso de IA Generativa para Java. Escolha a abordagem de desenvolvimento que preferir:

### Opções de Configuração do Ambiente:

#### Opção A: GitHub Codespaces (Recomendado)

**Comece a programar em 2 minutos - nenhuma configuração local necessária!**

1. Faça um fork deste repositório para sua conta no GitHub  
   > **Nota**: Se quiser editar a configuração básica, consulte a [Configuração do Container de Desenvolvimento](../../../.devcontainer/devcontainer.json)
2. Clique em **Code** → aba **Codespaces** → **...** → **Novo com opções...**
3. Use os padrões – isso selecionará a **configuração do container de desenvolvimento**: **Ambiente de Desenvolvimento IA Generativa Java** container dev personalizado criado para este curso
4. Clique em **Criar codespace**
5. Aguarde cerca de 2 minutos para o ambiente estar pronto
6. Prossiga para o [Passo 2: Provisione o Azure AI Foundry](#passo-2-provisione-o-azure-ai-foundry)

<img src="../../../translated_images/pt-BR/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: Codespaces submenu" width="50%">

<img src="../../../translated_images/pt-BR/image.833552b62eee7766.webp" alt="Screenshot: New with options" width="50%">

<img src="../../../translated_images/pt-BR/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: Create codespace options" width="50%">


> **Benefícios do Codespaces**:
> - Nenhuma instalação local necessária
> - Funciona em qualquer dispositivo com navegador
> - Pré-configurado com todas as ferramentas e dependências
> - 60 horas gratuitas por mês para contas pessoais
> - Ambiente consistente para todos os alunos

#### Opção B: Container de Desenvolvimento Local

**Para desenvolvedores que preferem desenvolvimento local com Docker**

1. Faça fork e clone este repositório na sua máquina local  
   > **Nota**: Se quiser editar a configuração básica, veja a [Configuração do Container de Desenvolvimento](../../../.devcontainer/devcontainer.json)
2. Instale o [Docker Desktop](https://www.docker.com/products/docker-desktop/) e o [VS Code](https://code.visualstudio.com/)
3. Instale a [extensão Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) no VS Code
4. Abra a pasta do repositório no VS Code
5. Quando solicitado, clique em **Reabrir no Container** (ou use `Ctrl+Shift+P` → "Dev Containers: Reabrir no Container")
6. Aguarde o container construir e iniciar
7. Prossiga para o [Passo 2: Provisione o Azure AI Foundry](#passo-2-provisione-o-azure-ai-foundry)

<img src="../../../translated_images/pt-BR/devcontainer.21126c9d6de64494.webp" alt="Screenshot: Dev container setup" width="50%">

<img src="../../../translated_images/pt-BR/image-3.bf93d533bbc84268.webp" alt="Screenshot: Dev container build complete" width="50%">

#### Opção C: Use Sua Instalação Local Existente

**Para desenvolvedores com ambientes Java existentes**

Pré-requisitos:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html) 
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) ou sua IDE preferida

Passos:
1. Clone este repositório na sua máquina local
2. Abra o projeto na sua IDE
3. Prossiga para o [Passo 2: Provisione o Azure AI Foundry](#passo-2-provisione-o-azure-ai-foundry)

> **Dica Profissional**: Se você tem uma máquina com especificações baixas mas quer usar o VS Code localmente, use o GitHub Codespaces! Você pode conectar seu VS Code local a um Codespace hospedado na nuvem para o melhor dos dois mundos.

<img src="../../../translated_images/pt-BR/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: created local devcontainer instance" width="50%">


## Passo 2: Provisione o Azure AI Foundry

Implante os modelos de IA do curso no Azure AI Foundry como código. A partir da raiz do repositório:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```

O `azd` solicita um nome de ambiente e região, provisiona uma conta Azure AI Foundry com deployments `gpt-4o-mini` e `text-embedding-3-small`, e grava o endpoint no `.env` do exemplo — tudo com autenticação **sem chave** (sem chaves de API).

> **Guia completo:** Veja o [Guia de Configuração do Azure AI Foundry](getting-started-azure-openai.md) para pré-requisitos, alternativa manual (portal), orientação de região e notas de custo/limpeza.

## Passo 3: Teste Sua Configuração

Depois que seus modelos Foundry estiverem provisionados, teste a conexão com o app exemplo em [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure).

1. Abra o terminal no seu ambiente de desenvolvimento.
2. Navegue até o exemplo:  
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
3. Certifique-se que você está logado (a autenticação sem chave precisa de token):  
   ```bash
   az login
   ```
   > Se você executou `azd up`, o arquivo `.env` com seu endpoint já foi criado para você.  
4. Execute a aplicação:  
   ```bash
   mvn clean spring-boot:run
   ```

Você deverá ver uma resposta do modelo `gpt-4o-mini`.

### Entendendo o Código do Exemplo

O exemplo em `examples/basic-chat-azure` é um app Spring Boot que usa o **Spring AI** para conectar ao Azure AI Foundry com autenticação sem chave.

**O que este código faz:**
- **Conecta** ao Azure AI Foundry usando seu login Azure (Microsoft Entra ID) — sem chave API
- **Envia** um prompt para o modelo `gpt-4o-mini`
- **Recebe** e exibe a resposta da IA
- **Valida** que sua configuração está funcionando corretamente

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

Ótimo! Agora você tem tudo configurado:

- Modelos Azure AI Foundry provisionados como código com Bicep + `azd`
- Ambiente de desenvolvimento Java rodando (seja Codespaces, dev containers ou local)
- Conectado ao Azure AI Foundry com autenticação sem chave (Microsoft Entra ID) — sem chaves de API
- Testado tudo funcionando com um exemplo simples que conversa com seu modelo

## Próximos Passos

[Capítulo 3: Técnicas Core de IA Generativa](../03-CoreGenerativeAITechniques/README.md)

## Resolução de Problemas

Está com problemas? Aqui estão problemas comuns e soluções:

- **Falha na autenticação (401/403)?**  
  - Execute `az login` — a autenticação é sem chave, então você precisa estar logado  
  - Verifique se sua conta tem o papel **Cognitive Services OpenAI User** no recurso  
  - Se você acabou de provisionar, espere um minuto para a atribuição do papel propagar

- **Maven não encontrado?**  
  - Se usar containers de desenvolvimento/Codespaces, o Maven já deve estar pré-instalado  
  - Para configuração local, certifique-se que Java 21+ e Maven 3.9+ estão instalados  
  - Teste `mvn --version` para verificar a instalação

- **`azd` não encontrado ou provisionamento falha?**  
  - Instale o [Azure Developer CLI](https://aka.ms/azure-dev/install) e execute `azd auth login`  
  - Escolha uma região onde `gpt-4o-mini` está disponível (ex: `eastus2`)  
  - Veja o [guia de configuração do Azure AI Foundry](getting-started-azure-openai.md) para detalhes

- **Container de desenvolvimento não inicia?**  
  - Certifique-se de que o Docker Desktop está rodando (para desenvolvimento local)  
  - Tente reconstruir o container: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **Erros de compilação da aplicação?**  
  - Verifique se está no diretório correto: `02-SetupDevEnvironment/examples/basic-chat-azure`  
  - Tente limpar e recompilar: `mvn clean compile`

> **Precisa de ajuda?**: Ainda com problemas? Abra uma issue no repositório e vamos ajudá-lo.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Aviso Legal**:
Este documento foi traduzido usando o serviço de tradução por IA [Co-op Translator](https://github.com/Azure/co-op-translator). Embora nos esforcemos pela precisão, por favor, esteja ciente de que traduções automatizadas podem conter erros ou imprecisões. O documento original em seu idioma nativo deve ser considerado a fonte autorizada. Para informações críticas, recomenda-se tradução profissional humana. Não nos responsabilizamos por quaisquer mal-entendidos ou interpretações incorretas decorrentes do uso desta tradução.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->