# Configuração do Ambiente de Desenvolvimento para o Azure AI Foundry

> Este guia configura os modelos do **Azure AI Foundry** para as aplicações Java de IA deste curso, usando autenticação **sem chave** (Microsoft Entra ID) — sem chaves de API para gerir. Novo nas ferramentas? Comece pelo [guia do ambiente de desenvolvimento](./README.md).

Este guia configura modelos do **Azure AI Foundry** para as aplicações Java de IA deste curso. Tem dois caminhos:

- **Opção A — Provisionar com `azd` + Bicep (recomendado):** um comando executa o deploy da conta Foundry e dos modelos como código. Sem depender do portal.
- **Opção B — Criar recursos manualmente** no portal Azure AI Foundry.

Ambos os caminhos usam **autenticação sem chave** (Microsoft Entra ID) — não existem chaves de API para copiar ou expor.

## Índice

- [O que será criado](#o-que-será-criado)
- [Pré-requisitos](#pré-requisitos)
- [Opção A: Provisionar com azd + Bicep (Recomendado)](#option-a-provision-with-azd--bicep-recommended)
- [Opção B: Criar Recursos Manualmente](#opção-b-criar-recursos-manualmente)
- [Configurar o Ambiente](#configurar-o-ambiente)
- [Testar a Configuração](#testar-a-configuração)
- [Próximos Passos](#próximos-passos)
- [Recursos](#recursos)
- [Recursos Adicionais](#recursos-adicionais)

## O que será criado

Os templates Bicep em [`infra/`](../../../02-SetupDevEnvironment/infra) criam:

- Uma conta **Azure AI Foundry** (`Microsoft.CognitiveServices/accounts`, tipo `AIServices`) com um projeto
- Um deployment de **chat** — `gpt-4o-mini`
- Um deployment de **embedding** — `text-embedding-3-small` (usado em capítulos posteriores)
- Uma **atribuição de função sem chave** (`Cognitive Services OpenAI User`) para que se possa autenticar com `az login` em vez de gerir chaves

## Pré-requisitos

- Uma [subscrição Azure](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) e [Maven 3.9+](https://maven.apache.org/download.cgi)

## Opção A: Provisionar com azd + Bicep (Recomendado)

A partir da pasta `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# Iniciar sessão (ambas as ferramentas)
azd auth login
az login

# Providenciar a conta Foundry + implementações de modelos
azd up
```

O `azd` pede um **nome de ambiente** (por exemplo `genai-java`) e uma **região**. Escolha uma região onde `gpt-4o-mini` e `text-embedding-3-small` estejam disponíveis — por exemplo `eastus2` ou `swedencentral`.

Quando a configuração terminar, o azd:

1. Desdobra tudo o que está definido em [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep).
2. Executa um hook pós-provisionamento que escreve [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) com o seu endpoint e nomes de deployment (sem segredos).

> **Dica:** Execute `azd up` sempre que quiser aplicar alterações. Execute `azd down` para apagar tudo e parar de incorrer em custos.

Para ver as definições geradas:

```bash
azd env get-values
```

Agora vá para [Testar a Configuração](#testar-a-configuração).

## Opção B: Criar Recursos Manualmente

Prefere o portal? Crie os recursos manualmente:

1. Aceda ao [portal Azure AI Foundry](https://ai.azure.com/) e inicie sessão.
2. **Crie um projeto** (isto também cria um recurso AI Foundry). Dê-lhe um nome como `GenAIJava`.
3. No seu projeto, abra **Modelos + endpoints** → **Deploy model** → **Deploy base model**.
4. Faça o deploy do **gpt-4o-mini** (nome do deployment `gpt-4o-mini`). Repita para o **text-embedding-3-small** se quiser exemplos com embeddings.
5. Na secção **Overview**, copie o **endpoint** (por exemplo, `https://<resource>.openai.azure.com/`).
6. Conceda a si mesmo acesso sem chave: no recurso, abra **Access control (IAM)** → **Add role assignment** → atribua **Cognitive Services OpenAI User** à sua conta.

> **Ainda com dificuldades?** Consulte a [documentação do Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Configurar o Ambiente

**Se usou a Opção A (`azd up`)**, o ficheiro de definições já está criado — não precisa de configurar nada. Vá para [Testar a Configuração](#testar-a-configuração).

**Se usou a Opção B (manual)**, crie o ficheiro `.env` do exemplo manualmente:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

Edite `.env` com o seu endpoint (sem chave — a autenticação é sem chave):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Nota de segurança:** Não há chave de API para guardar. Autentica com Microsoft Entra ID via `az login` (localmente) ou uma identidade gerida (no Azure). O ficheiro `.env` contém apenas definições não secretas e já está incluído no `.gitignore`.

## Testar a Configuração

Certifique-se de que está autenticado para que a autenticação sem chave possa obter um token, depois execute o exemplo:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # se ainda não tiver sessão iniciada
mvn clean spring-boot:run
```

Deverá ver uma resposta do modelo `gpt-4o-mini`!

> **Utilizadores do VS Code:** prima `F5` para executar. A aplicação carrega o seu `.env` automaticamente.

> **Exemplo completo:** Consulte o [exemplo Basic Chat with Azure AI Foundry](./examples/basic-chat-azure/README.md) para detalhes e resolução de problemas.

## Próximos Passos

**Configuração concluída!** Agora tem:
- Azure AI Foundry com `gpt-4o-mini` e `text-embedding-3-small` implementados
- Autenticação sem chave (Microsoft Entra ID) — sem chaves para gerir
- Um `.env` local com o seu endpoint e nomes dos deployments
- Um ambiente de desenvolvimento Java pronto a usar

**Continue para** [Capítulo 3: Técnicas fundamentais de IA generativa](../03-CoreGenerativeAITechniques/README.md) para começar a construir aplicações de IA!

## Recursos

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Autenticação sem chave com Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Documentação Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)
- [Documentação Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Recursos Adicionais

- [Descarregar VS Code](https://code.visualstudio.com/Download)
- [Obter Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Configuração do Dev Container](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Aviso Legal**:
Este documento foi traduzido utilizando o serviço de tradução automática [Co-op Translator](https://github.com/Azure/co-op-translator). Embora nos esforcemos pela precisão, esteja ciente de que traduções automáticas podem conter erros ou imprecisões. O documento original na sua língua nativa deve ser considerado a fonte autorizada. Para informações críticas, recomenda-se tradução profissional humana. Não nos responsabilizamos por quaisquer mal-entendidos ou interpretações incorretas resultantes da utilização desta tradução.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->