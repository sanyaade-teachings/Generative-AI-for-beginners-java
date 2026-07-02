# Basic Chat with Azure AI Foundry - End-to-End Example

Ang halimbawa na ito ay isang simpleng Spring Boot application na kumokonekta sa isang **Azure AI Foundry** model gamit ang **keyless authentication** (Microsoft Entra ID) at sinusubukan ang iyong setup. Ginagamit nito ang Spring AI na `ChatClient`.

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

Bago patakbuhin ang halimbawang ito, siguraduhin na mayroon kang:

- Isang Azure AI Foundry resource na may `gpt-4o-mini` deployment — i-provision ito gamit ang `azd up` o manu-mano sa pamamagitan ng [Azure AI Foundry setup guide](../../getting-started-azure-openai.md)
- Ang **Cognitive Services OpenAI User** role para sa resource na iyon (ina-assign ito ng Bicep templates para sa iyo)
- Ang [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), naka-login gamit ang `az login`
- Java 21+ at Maven 3.9+

> **Hindi kailangan ng API key** — ang authentication ay keyless gamit ang Microsoft Entra ID.

## Quick Start

```bash
# 1. Pumunta sa proyekto
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Mag-sign in upang makakuha ng token ang keyless auth
az login

# 3. I-configure ang endpoint
#    - Kung pinatakbo mo ang `azd up`, naisulat na para sa iyo ang .env (laktawan ito).
#    - Kung hindi, kopyahin ang template at itakda ang AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. Patakbuhin ang aplikasyon
mvn spring-boot:run
```

## How Authentication Works

Ang halimbawa na ito ay nag-a-authenticate gamit ang **Microsoft Entra ID** — walang API key.

Kapag ang `spring.ai.azure.openai.endpoint` lamang ang na-set (at walang api-key), bumubuo ang Spring AI ng Azure OpenAI client gamit ang [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). Ang kredensyal na ito ay awtomatikong hahanap ng token mula sa iyong lokal na `az login` session, o mula sa managed identity kapag tumatakbo sa Azure — kaya ang parehong code ay gumagana sa dalawang lugar nang walang pagbabago.

## Running the Application

### Using Maven

```bash
mvn spring-boot:run
```

### Using VS Code

1. Buksan ang proyekto sa VS Code
2. Pindutin ang `F5` o gamitin ang "Run and Debug" panel
3. Piliin ang "Spring Boot-BasicChatApplication" configuration

> **Tandaan**: Ang VS Code configuration ay awtomatikong naglo-load ng iyong .env file

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

| Variable | Deskripsyon | Kailangan | Halimbawa |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | Foundry (Azure OpenAI) endpoint URL | Oo | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Pangalan ng chat model deployment | Hindi | `gpt-4o-mini` (default) |

> Walang **API key** variable — ang authentication ay keyless (Microsoft Entra ID gamit ang `az login`).

### Spring Configuration

Ang `application.yml` file ay nagko-configure ng:
- **Endpoint**: `${AZURE_OPENAI_ENDPOINT}` - Mula sa environment variable
- **Deployment**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - Mula sa environment variable na may fallback
- **Auth**: keyless — walang `api-key` na na-set, kaya ang Spring AI ay gumagamit ng `DefaultAzureCredential`
- **Temperature**: `0.7` - Kumokontrol sa pagiging malikhain (0.0 = deterministic, 1.0 = malikhain)
- **Max Tokens**: `500` - Pinakamahabang haba ng sagot

## Troubleshooting

### Common Issues

<details>
<summary><strong>Error: 401 / "PermissionDenied" / token errors</strong></summary>

- Patakbuhin ang `az login` — ang keyless auth ay nangangailangan ng aktibong sign-in para makakuha ng token
- Siguraduhing ang iyong account ay may **Cognitive Services OpenAI User** role sa resource
- Kapag bagong na-assign ang role, maghintay ng ilang minuto para ma-propagate ito
- Kumpirmahin na nasa tamang tenant/subscription ka (`az account show`)
</details>

<details>
<summary><strong>Error: "The endpoint is not valid" / connection errors</strong></summary>

- Siguraduhing ang `AZURE_OPENAI_ENDPOINT` ay ang buong base URL (hal. `https://your-resource.openai.azure.com/`)
- Tingnan ang consistency sa trailing slash
- Siguraduhing tugma ang endpoint sa iyong na-provision na resource (`azd env get-values`)
</details>

<details>
<summary><strong>Error: "The deployment was not found"</strong></summary>

- Siguraduhing ang `AZURE_OPENAI_DEPLOYMENT` ay tumutugma sa pangalan ng deployment sa Azure
- Tingnan kung matagumpay at aktibo ang pag-deploy ng modelo
- Ang default na pangalan ng deployment ay `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: Environment variables not loading</strong></summary>

- Siguraduhing ang iyong `.env` file ay nasa root directory ng proyekto (kateray sa `pom.xml`)
- Subukang patakbuhin ang `mvn spring-boot:run` sa VS Code's integrated terminal
- Tingnan kung tama ang pagkaka-install ng VS Code Java extension
</details>

### Debug Mode

Para paganahin ang detalyadong logging, alisin ang komentaryo sa mga linyang ito sa `application.yml`:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Next Steps

**Kumpleto na ang Setup!** Ipagpatuloy ang iyong pag-aaral:

[Chapter 3: Core Generative AI Techniques](../../../03-CoreGenerativeAITechniques/README.md)

## Resources

- [Spring AI Azure OpenAI Documentation](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Keyless authentication with Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry Portal](https://ai.azure.com/)
- [Azure AI Foundry Documentation](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Pagtatanggi**:
Ang dokumentong ito ay isinalin gamit ang serbisyo ng AI translation na [Co-op Translator](https://github.com/Azure/co-op-translator). Bagama't nagsusumikap kami para sa katumpakan, pakatandaan na ang awtomatikong pagsasalin ay maaaring maglaman ng mga pagkakamali o hindi pagkakatugma. Ang orihinal na dokumento sa orihinal nitong wika ang dapat ituring na pangunahing sanggunian. Para sa mahahalagang impormasyon, inirerekomenda ang propesyonal na pagsasalin ng tao. Hindi kami mananagot sa anumang maling pagkakaintindi o maling interpretasyon na nagmula sa paggamit ng pagsasaling ito.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->