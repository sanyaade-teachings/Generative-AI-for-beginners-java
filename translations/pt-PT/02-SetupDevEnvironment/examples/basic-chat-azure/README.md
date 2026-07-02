# Chat Básico com Azure AI Foundry - Exemplo Completo

Este exemplo é uma aplicação simples Spring Boot que se liga a um modelo **Azure AI Foundry** usando **autenticação sem chave** (Microsoft Entra ID) e testa a sua configuração. Utiliza o `ChatClient` do Spring AI.

## Índice

- [Pré-requisitos](#pré-requisitos)
- [Início Rápido](#início-rápido)
- [Como Funciona a Autenticação](#como-funciona-a-autenticação)
- [Executar a Aplicação](#executar-a-aplicação)
  - [Usando Maven](#usando-maven)
  - [Usando VS Code](#usando-vs-code)
  - [Saída Esperada](#saída-esperada)
- [Referência de Configuração](#referência-de-configuração)
  - [Variáveis de Ambiente](#variáveis-de-ambiente)
  - [Configuração do Spring](#configuração-do-spring)
- [Resolução de Problemas](#resolução-de-problemas)
  - [Problemas Comuns](#problemas-comuns)
  - [Modo de Depuração](#modo-de-depuração)
- [Próximos Passos](#próximos-passos)
- [Recursos](#recursos)

## Pré-requisitos

Antes de executar este exemplo, certifique-se de ter:

- Um recurso Azure AI Foundry com um deployment `gpt-4o-mini` — crie-o com `azd up` ou manualmente através do [guia de configuração Azure AI Foundry](../../getting-started-azure-openai.md)
- A função **Cognitive Services OpenAI User** atribuída nesse recurso (os templates Bicep atribuem-na automaticamente)
- O [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), autenticado com `az login`
- Java 21+ e Maven 3.9+

> **Nenhuma chave API necessária** — a autenticação é sem chave via Microsoft Entra ID.

## Início Rápido

```bash
# 1. Navegar para o projeto
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Iniciar sessão para que a autenticação sem chave possa obter um token
az login

# 3. Configurar o endpoint
#    - Se executou `azd up`, o .env foi criado para si (pule este passo).
#    - Caso contrário, copie o template e defina AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. Execute a aplicação
mvn spring-boot:run
```

## Como Funciona a Autenticação

Este exemplo autentica com **Microsoft Entra ID** — não há chave API.

Quando apenas `spring.ai.azure.openai.endpoint` está definido (e sem api-key), o Spring AI cria o cliente Azure OpenAI com [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). Essa credencial encontra automaticamente um token da sua sessão local `az login` ou de uma identidade gerida quando executada no Azure — portanto, o mesmo código funciona em ambos os ambientes sem alterações.

## Executar a Aplicação

### Usando Maven

```bash
mvn spring-boot:run
```

### Usando VS Code

1. Abra o projeto no VS Code
2. Pressione `F5` ou use o painel "Run and Debug"
3. Selecione a configuração "Spring Boot-BasicChatApplication"

> **Nota**: A configuração do VS Code carrega automaticamente o seu ficheiro .env

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

| Variável | Descrição | Obrigatória | Exemplo |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | URL do endpoint Foundry (Azure OpenAI) | Sim | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Nome do deployment do modelo de chat | Não | `gpt-4o-mini` (padrão) |

> Não existe variável de chave API — a autenticação é sem chave (Microsoft Entra ID via `az login`).

### Configuração do Spring

O ficheiro `application.yml` configura:
- **Endpoint**: `${AZURE_OPENAI_ENDPOINT}` - A partir da variável de ambiente
- **Deployment**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - A partir da variável de ambiente com valor predefinido
- **Auth**: sem chave — não é definida `api-key`, pelo que o Spring AI usa `DefaultAzureCredential`
- **Temperature**: `0.7` - Controla a criatividade (0.0 = determinístico, 1.0 = criativo)
- **Max Tokens**: `500` - Comprimento máximo da resposta

## Resolução de Problemas

### Problemas Comuns

<details>
<summary><strong>Erro: 401 / "PermissionDenied" / erros de token</strong></summary>

- Execute `az login` — autenticação sem chave requer sessão ativa para obter token
- Verifique se a sua conta tem a função **Cognitive Services OpenAI User** atribuída no recurso
- Se acabou de atribuir a função, aguarde um minuto para a propagação
- Confirme que está no tenant/subscrição correta (`az account show`)
</details>

<details>
<summary><strong>Erro: "The endpoint is not valid" / erros de conexão</strong></summary>

- Certifique-se de que `AZURE_OPENAI_ENDPOINT` é a URL base completa (ex., `https://your-resource.openai.azure.com/`)
- Verifique a consistência da barra no final
- Confirme que o endpoint corresponde ao recurso provisionado (`azd env get-values`)
</details>

<details>
<summary><strong>Erro: "The deployment was not found"</strong></summary>

- Verifique se `AZURE_OPENAI_DEPLOYMENT` corresponde a um nome de deployment no Azure
- Confirme que o modelo está implantado e ativo com sucesso
- O nome de deployment padrão é `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: Variáveis de ambiente não carregam</strong></summary>

- Assegure-se que o ficheiro `.env` está na raiz do projeto (ao mesmo nível do `pom.xml`)
- Tente executar `mvn spring-boot:run` no terminal integrado do VS Code
- Verifique se a extensão Java do VS Code está instalada corretamente
</details>

### Modo de Depuração

Para ativar logging detalhado, descomente estas linhas no `application.yml`:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Próximos Passos

**Configuração Completa!** Continue a sua jornada de aprendizagem:

[Capítulo 3: Técnicas Core de IA Generativa](../../../03-CoreGenerativeAITechniques/README.md)

## Recursos

- [Documentação Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Autenticação sem chave com Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Portal Azure AI Foundry](https://ai.azure.com/)
- [Documentação Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Aviso Legal**:
Este documento foi traduzido utilizando o serviço de tradução automática [Co-op Translator](https://github.com/Azure/co-op-translator). Embora nos esforcemos pela precisão, esteja ciente de que traduções automáticas podem conter erros ou imprecisões. O documento original na sua língua nativa deve ser considerado a fonte autorizada. Para informações críticas, recomenda-se tradução profissional humana. Não nos responsabilizamos por quaisquer mal-entendidos ou interpretações incorretas resultantes da utilização desta tradução.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->