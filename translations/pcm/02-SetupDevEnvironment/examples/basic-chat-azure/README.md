# Basic Chat wit Azure AI Foundry - End-to-End Example

Dis example na simple Spring Boot application wey connect to **Azure AI Foundry** model using **keyless authentication** (Microsoft Entra ID) and test your setup. E dey use Spring AI `ChatClient`.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [How Authentication Works](#how-authentication-works)
- [Running the Application](#running-the-application)
  - [Using Maven](#using-maven)
  - [Using VS Code](#using-vs-code)
  - [Expected Output](#expected-output)
- [Configuration Reference](#configuration-reference)
  - [Environment Variables](#environment-variables)
  - [Spring Configuration](#spring-configuration)
- [Troubleshooting](#troubleshooting)
  - [Common Issues](#common-issues)
  - [Debug Mode](#debug-mode)
- [Next Steps](#next-steps)
- [Resources](#resources)

## Prerequisites

Before you run dis example, make sure say you get:

- Azure AI Foundry resource wey get `gpt-4o-mini` deployment — you fit provision am with `azd up` or do am manually via the [Azure AI Foundry setup guide](../../getting-started-azure-openai.md)
- The **Cognitive Services OpenAI User** role for that resource (the Bicep templates dey assign dis for you)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), wey you don sign in with `az login`
- Java 21+ and Maven 3.9+

> **No API key required** — authentication na keyless via Microsoft Entra ID.

## Quick Start

```bash
# 1. Go enter di project
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Sign in make keyless auth fit get token
az login

# 3. Set di endpoint
#    - If you don run `azd up`, .env don write for you (skip dis one).
#    - If no, copy di template den set AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. Run di application
mvn spring-boot:run
```

## How Authentication Works

Dis example dey authenticate wit **Microsoft Entra ID** — no API key dey.

If only `spring.ai.azure.openai.endpoint` dey set (and no api-key), Spring AI go build Azure OpenAI client wit [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). Dat credential go automatically find token from your `az login` session for your local machine, or from managed identity when e dey run for Azure — so d same code fit run for both places without change.

## Running the Application

### Using Maven

```bash
mvn spring-boot:run
```

### Using VS Code

1. Open di project for VS Code
2. Press `F5` or use di "Run and Debug" panel
3. Select "Spring Boot-BasicChatApplication" configuration

> **Note**: The VS Code configuration go automatically load your .env file

### Expected Output

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

## Configuration Reference

### Environment Variables

| Variable | Description | Required | Example |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | Foundry (Azure OpenAI) endpoint URL | Yes | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Chat model deployment name | No | `gpt-4o-mini` (default) |

> No **API key** variable dey — authentication na keyless (Microsoft Entra ID via `az login`).

### Spring Configuration

The `application.yml` file dey configure:
- **Endpoint**: `${AZURE_OPENAI_ENDPOINT}` - From environment variable
- **Deployment**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - From environment variable with fallback
- **Auth**: keyless — no `api-key` dey set, so Spring AI dey use `DefaultAzureCredential`
- **Temperature**: `0.7` - Controls creativity (0.0 = deterministic, 1.0 = creative)
- **Max Tokens**: `500` - Maximum response length

## Troubleshooting

### Common Issues

<details>
<summary><strong>Error: 401 / "PermissionDenied" / token errors</strong></summary>

- Run `az login` — keyless auth need active sign-in to get token
- Confirm say your account get **Cognitive Services OpenAI User** role for the resource
- If you just assign the role, wait small time make e propagate
- Make sure say you dey the correct tenant/subscription (`az account show`)
</details>

<details>
<summary><strong>Error: "The endpoint is not valid" / connection errors</strong></summary>

- Make sure say `AZURE_OPENAI_ENDPOINT` na full base URL (like `https://your-resource.openai.azure.com/`)
- Check for trailing slash correctness
- Confirm endpoint dey match your provisioned resource (`azd env get-values`)
</details>

<details>
<summary><strong>Error: "The deployment was not found"</strong></summary>

- Verify `AZURE_OPENAI_DEPLOYMENT` dey match deployment name wey dey Azure
- Check say the model don deploy well and dey active
- Default deployment name na `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: Environment variables no dey load</strong></summary>

- Make sure your `.env` file dey project root directory (same level as `pom.xml`)
- Try run `mvn spring-boot:run` for VS Code integrated terminal
- Check say the VS Code Java extension don install well
</details>

### Debug Mode

To enable detailed logging, make uncomment dis lines for `application.yml`:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Next Steps

**Setup Done!** Continue your learning journey:

[Chapter 3: Core Generative AI Techniques](../../../03-CoreGenerativeAITechniques/README.md)

## Resources

- [Spring AI Azure OpenAI Documentation](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Keyless authentication wit Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry Portal](https://ai.azure.com/)
- [Azure AI Foundry Documentation](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Disclaimer**:
Dis document don translate wit AI translation service [Co-op Translator](https://github.com/Azure/co-op-translator). Even tho we dey try make am correct, abeg make you know say automated translation fit get errors or mistakes. Di original document for dia own language na im be di correct source. For important info, make person wey sabi human translation do am. We no go responsible for any misunderstanding or wrong understanding wey fit happen because of dis translation.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->