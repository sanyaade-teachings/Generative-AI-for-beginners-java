# Configurando o Ambiente de Desenvolvimento para Azure AI Foundry

> Este guia configura os modelos do **Azure AI Foundry** para os aplicativos Java AI neste curso, usando autenticação **sem chave** (Microsoft Entra ID) — sem chaves de API para gerenciar. Novo nas ferramentas? Comece com o [guia do ambiente de desenvolvimento](./README.md).

Este guia configura os modelos do **Azure AI Foundry** para os aplicativos Java AI neste curso. Você tem dois caminhos:

- **Opção A — Provisionar com `azd` + Bicep (recomendado):** um comando implanta a conta Foundry e modelos como código. Sem cliques no portal.
- **Opção B — Criar recursos manualmente** no portal Azure AI Foundry.

Ambos os caminhos usam **autenticação sem chave** (Microsoft Entra ID) — não há chaves de API para copiar ou vazar.

## Sumário

- [O que é criado](#o-que-é-criado)
- [Pré-requisitos](#pré-requisitos)
- [Opção A: Provisionar com azd + Bicep (Recomendado)](#option-a-provision-with-azd--bicep-recommended)
- [Opção B: Criar Recursos Manualmente](#opção-b-criar-recursos-manualmente)
- [Configure seu Ambiente](#configure-seu-ambiente)
- [Teste sua Configuração](#teste-sua-configuração)
- [E agora?](#e-agora)
- [Recursos](#recursos)
- [Recursos Adicionais](#recursos-adicionais)

## O que é criado

Os templates Bicep em [`infra/`](../../../02-SetupDevEnvironment/infra) provisionam:

- Uma conta **Azure AI Foundry** (`Microsoft.CognitiveServices/accounts`, tipo `AIServices`) com um projeto
- Uma implantação de **chat** — `gpt-4o-mini`
- Uma implantação de **embedding** — `text-embedding-3-small` (usada em capítulos posteriores)
- Uma **atribuição de função sem chave** (`Cognitive Services OpenAI User`) para que você faça login com `az login` ao invés de gerenciar chaves

## Pré-requisitos

- Uma [assinatura Azure](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) e [Maven 3.9+](https://maven.apache.org/download.cgi)

## Opção A: Provisionar com azd + Bicep (Recomendado)

Na pasta `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# Entrar (ambas as ferramentas)
azd auth login
az login

# Provisionar a conta Foundry + implantações de modelos
azd up
```

O `azd` solicita um **nome do ambiente** (por exemplo `genai-java`) e uma **região**. Escolha uma região onde `gpt-4o-mini` e `text-embedding-3-small` estejam disponíveis — por exemplo `eastus2` ou `swedencentral`.

Quando o provisionamento terminar, o azd:

1. Implanta tudo definido em [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep).
2. Executa um hook pós-provisionamento que grava [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) com seu endpoint e nomes de implantação (sem segredos).

> **Dica:** Execute `azd up` novamente a qualquer momento para aplicar alterações. Execute `azd down` para excluir tudo e parar de gerar custos.

Para ver as configurações geradas:

```bash
azd env get-values
```

Agora pule para [Teste sua Configuração](#teste-sua-configuração).

## Opção B: Criar Recursos Manualmente

Prefere o portal? Crie os recursos manualmente:

1. Vá para o [portal do Azure AI Foundry](https://ai.azure.com/) e faça login.
2. **Crie um projeto** (isso também cria um recurso AI Foundry). Dê um nome como `GenAIJava`.
3. Em seu projeto, abra **Models + endpoints** → **Deploy model** → **Deploy base model**.
4. Faça a implantação do **gpt-4o-mini** (nome da implantação `gpt-4o-mini`). Repita para **text-embedding-3-small** se quiser os exemplos de embedding.
5. Em **Overview**, copie o **endpoint** (por exemplo `https://<resource>.openai.azure.com/`).
6. Conceda a si mesmo acesso sem chave: no recurso, abra **Controle de acesso (IAM)** → **Adicionar atribuição de função** → atribua **Cognitive Services OpenAI User** à sua conta.

> **Ainda com dificuldades?** Veja a [documentação do Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Configure seu Ambiente

**Se você usou a Opção A (`azd up`)**, seu arquivo de configurações já foi criado — não há nada para configurar. Pule para [Teste sua Configuração](#teste-sua-configuração).

**Se você usou a Opção B (manual)**, crie o arquivo `.env` do exemplo por conta própria:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

Edite `.env` com seu endpoint (sem chave — a autenticação é sem chave):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Nota de segurança:** Não há chave de API para armazenar. Você autentica com Microsoft Entra ID via `az login` (localmente) ou uma identidade gerenciada (no Azure). O arquivo `.env` contém apenas configurações não secretas e já está incluído no `.gitignore`.

## Teste sua Configuração

Certifique-se de estar logado para que a autenticação sem chave possa obter um token, então execute o exemplo:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # se você ainda não estiver conectado
mvn clean spring-boot:run
```

Você deve ver uma resposta do modelo `gpt-4o-mini`!

> **Usuários do VS Code:** Pressione `F5` para executar. O app carrega seu `.env` automaticamente.

> **Exemplo completo:** Veja o [Exemplo Básico de Chat com Azure AI Foundry](./examples/basic-chat-azure/README.md) para detalhes e solução de problemas.

## E agora?

**Configuração concluída!** Você agora tem:
- Azure AI Foundry com `gpt-4o-mini` e `text-embedding-3-small` implantados
- Autenticação sem chave (Microsoft Entra ID) — sem chaves para gerenciar
- Um `.env` local com seu endpoint e nomes das implantações
- Um ambiente de desenvolvimento Java pronto para uso

**Continue para** [Capítulo 3: Técnicas Core de IA Generativa](../03-CoreGenerativeAITechniques/README.md) para começar a construir aplicações de IA!

## Recursos

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Autenticação sem chave com Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Documentação Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)
- [Documentação Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [SDK Azure OpenAI Java](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Recursos Adicionais

- [Baixe o VS Code](https://code.visualstudio.com/Download)
- [Obtenha o Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Configuração do Dev Container](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Aviso Legal**:
Este documento foi traduzido usando o serviço de tradução por IA [Co-op Translator](https://github.com/Azure/co-op-translator). Embora nos esforcemos pela precisão, por favor, esteja ciente de que traduções automatizadas podem conter erros ou imprecisões. O documento original em seu idioma nativo deve ser considerado a fonte autorizada. Para informações críticas, recomenda-se tradução profissional humana. Não nos responsabilizamos por quaisquer mal-entendidos ou interpretações incorretas decorrentes do uso desta tradução.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->