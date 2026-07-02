# Chat Básico com Azure AI Foundry - Exemplo Completo

Este exemplo é uma aplicação simples Spring Boot que se conecta a um modelo **Azure AI Foundry** usando **autenticação sem chave** (Microsoft Entra ID) e testa sua configuração. Ele usa o `ChatClient` do Spring AI.

## Índice

- [Pré-requisitos](#prerequisitos)
- [Início Rápido](#início-rápido)
- [Como a Autenticação Funciona](#como-a-autenticação-funciona)
- [Executando a Aplicação](#executando-a-aplicação)
  - [Usando Maven](#usando-maven)
  - [Usando VS Code](#usando-vs-code)
  - [Saída Esperada](#saída-esperada)
- [Referência de Configuração](#referência-de-configuração)
  - [Variáveis de Ambiente](#variáveis-de-ambiente)
  - [Configuração do Spring](#configuração-do-spring)
- [Solução de Problemas](#solução-de-problemas)
  - [Problemas Comuns](#problemas-comuns)
  - [Modo de Depuração](#modo-de-depuração)
- [Próximos Passos](#próximos-passos)
- [Recursos](#recursos)

## Pré-requisitos

Antes de executar este exemplo, certifique-se de que você tem:

- Um recurso Azure AI Foundry com um deployment `gpt-4o-mini` — provisionado com `azd up` ou manualmente via o [guia de configuração do Azure AI Foundry](../../getting-started-azure-openai.md)
- O papel **Cognitive Services OpenAI User** nesse recurso (os templates Bicep atribuem isso para você)
- O [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), logado com `az login`
- Java 21+ e Maven 3.9+

> **Nenhuma chave de API é necessária** — a autenticação é sem chave via Microsoft Entra ID.

## Início Rápido

```bash
# 1. Navegue até o projeto
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Faça login para que a autenticação sem chave possa obter um token
az login

# 3. Configure o endpoint
#    - Se você executou `azd up`, .env foi criado para você (pule esta etapa).
#    - Caso contrário, copie o modelo e defina AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. Execute a aplicação
mvn spring-boot:run
```

## Como a Autenticação Funciona

Este exemplo autentica com **Microsoft Entra ID** — não há chave de API.

Quando apenas `spring.ai.azure.openai.endpoint` está configurado (e nenhuma api-key), o Spring AI cria o cliente Azure OpenAI com [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). Essa credencial automaticamente encontra um token da sua sessão `az login` localmente, ou de uma identidade gerenciada quando executando no Azure — assim o mesmo código funciona nos dois ambientes sem alterações.

## Executando a Aplicação

### Usando Maven

```bash
mvn spring-boot:run
```

### Usando VS Code

1. Abra o projeto no VS Code
2. Pressione `F5` ou use o painel "Run and Debug"
3. Selecione a configuração "Spring Boot-BasicChatApplication"

> **Nota**: A configuração do VS Code carrega automaticamente seu arquivo .env

### Saída Esperada

```
Starting Basic Chat with Azure OpenAI...
Environment variables loaded successfully
Connecting to Azure OpenAI...
Sending prompt: What is AI in a short sentence? Max 100 words.

AI Response:
================
AI, or Artificial Intelligence, is the simulation of human intelligence in machines programmed to think and learn like humans.
================

Success! Azure OpenAI connection is working correctly.
```

## Referência de Configuração

### Variáveis de Ambiente

| Variável | Descrição | Obrigatório | Exemplo |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | URL do endpoint Foundry (Azure OpenAI) | Sim | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Nome do deployment do modelo de chat | Não | `gpt-4o-mini` (padrão) |

> Não existe variável de chave de API — a autenticação é sem chave (Microsoft Entra ID via `az login`).

### Configuração do Spring

O arquivo `application.yml` configura:
- **Endpoint**: `${AZURE_OPENAI_ENDPOINT}` - Pela variável de ambiente
- **Deployment**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - Pela variável de ambiente com fallback
- **Autenticação**: sem chave — nenhuma `api-key` é configurada, então o Spring AI usa `DefaultAzureCredential`
- **Temperatura**: `0.7` - Controla criatividade (0.0 = determinístico, 1.0 = criativo)
- **Max Tokens**: `500` - Tamanho máximo da resposta

## Solução de Problemas

### Problemas Comuns

<details>
<summary><strong>Erro: 401 / "PermissionDenied" / erros de token</strong></summary>

- Execute `az login` — autenticação sem chave precisa de login ativo para obter token
- Verifique se sua conta tem o papel **Cognitive Services OpenAI User** no recurso
- Se acabou de atribuir o papel, espere um minuto para a propagação
- Confirme que está no tenant/assinatura correta (`az account show`)
</details>

<details>
<summary><strong>Erro: "The endpoint is not valid" / erros de conexão</strong></summary>

- Verifique se `AZURE_OPENAI_ENDPOINT` é a URL base completa (ex: `https://your-resource.openai.azure.com/`)
- Confira a consistência da barra no final
- Confirme que o endpoint corresponde ao recurso provisionado (`azd env get-values`)
</details>

<details>
<summary><strong>Erro: "The deployment was not found"</strong></summary>

- Verifique se `AZURE_OPENAI_DEPLOYMENT` corresponde a um nome de deployment no Azure
- Confirme que o modelo está implantado e ativo com sucesso
- O nome padrão do deployment é `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: Variáveis de ambiente não carregando</strong></summary>

- Certifique-se de que seu arquivo `.env` está na raíz do projeto (mesmo nível do `pom.xml`)
- Tente executar `mvn spring-boot:run` no terminal integrado do VS Code
- Verifique se a extensão Java do VS Code está corretamente instalada
</details>

### Modo de Depuração

Para habilitar logs detalhados, descomente estas linhas em `application.yml`:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Próximos Passos

**Configuração Completa!** Continue sua jornada de aprendizado:

[Capítulo 3: Técnicas Centrais de IA Generativa](../../../03-CoreGenerativeAITechniques/README.md)

## Recursos

- [Documentação Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Autenticação sem chave com Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Portal Azure AI Foundry](https://ai.azure.com/)
- [Documentação Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Aviso Legal**:
Este documento foi traduzido usando o serviço de tradução por IA [Co-op Translator](https://github.com/Azure/co-op-translator). Embora nos esforcemos pela precisão, por favor, esteja ciente de que traduções automatizadas podem conter erros ou imprecisões. O documento original em seu idioma nativo deve ser considerado a fonte autorizada. Para informações críticas, recomenda-se tradução profissional humana. Não nos responsabilizamos por quaisquer mal-entendidos ou interpretações incorretas decorrentes do uso desta tradução.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->