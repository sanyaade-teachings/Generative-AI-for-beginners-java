# Azure AI Foundry ਲਈ ਵਿਕਾਸ ਪਰਿਵੇਸ਼ ਸੈੱਟ ਕਰਨਾ

> ਇਹ ਗਾਈਡ ਇਸ ਕੋਰਸ ਵਿੱਚ ਜਾਵਾ ਏਆਈ ਐਪਸ ਲਈ **Azure AI Foundry** ਮਾਡਲਸ ਨੂੰ **keyless** ਪ੍ਰਮਾਣੀਕਰਣ (Microsoft Entra ID) ਦੀ ਵਰਤੋਂ ਨਾਲ ਸੈੱਟ ਕਰਦਾ ਹੈ — ਕੋਈ API ਕੁੰਜੀਆਂ ਸੰਭਾਲਣ ਦੀ ਜਰੂਰਤ ਨਹੀਂ। ਟੂਲਿੰਗ ਵਿੱਚ ਨਵਾਂ ਹੋ? [ਵਿਕਾਸ ਪਰਿਵੇਸ਼ ਗਾਈਡ](./README.md) ਨਾਲ ਸ਼ੁਰੂ ਕਰੋ।

ਇਹ ਗਾਈਡ ਇਸ ਕੋਰਸ ਵਿੱਚ ਜਾਵਾ ਏਆਈ ਐਪਸ ਲਈ **Azure AI Foundry** ਮਾਡਲਸ ਸੈੱਟ ਕਰਦਾ ਹੈ। ਤੁਹਾਡੇ ਕੋਲ ਦੋ ਰਸਤੇ ਹਨ:

- **ਚੋਣ A — `azd` + Bicep ਨਾਲ ਪ੍ਰੌਵੀਜ਼ਨ ਕਰੋ (ਸਿਫਾਰਸ਼ੀ):** ਇੱਕ ਕਮਾਂਡ ਨਾਲ Foundry ਅਕਾਊਂਟ ਅਤੇ ਮਾਡਲਸ ਕੋਡ ਦੇ ਤੌਰ 'ਤੇ ਤਫਤੀਸ਼ ਕਰਦਾ ਹੈ। ਕੋਈ ਪੋਰਟਲ 'ਤੇ ਕਲਿੱਕ ਕਰਨ ਦੀ ਲੋੜ ਨਹੀਂ।
- **ਚੋਣ B — Resources ਹੱਥੋਂ ਬਣਾੳੋ** Azure AI Foundry ਪੋਰਟਲ ਵਿੱਚ।

ਦੋਵਾਂ ਰਸਤੇ **keyless authentication** (Microsoft Entra ID) ਦੀ ਵਰਤੋਂ ਕਰਦੇ ਹਨ — ਕੋਈ API ਕੁੰਜੀਆਂ ਕਾਪੀ ਜਾਂ ਲੀਕ ਕਰਨ ਦੀ ਜਰੂਰਤ ਨਹੀਂ।

## ਸੂਚੀ

- [ਕੀ ਬਣਾਇਆ ਜਾਂਦਾ ਹੈ](#ਕੀ-ਬਣਾਇਆ-ਜਾਂਦਾ-ਹੈ)
- [ਪੂਰਵ-ਆਵਸ਼ਯਕਤਾਵਾਂ](#ਪੂਰਵ-ਆਵਸ਼ਯਕਤਾ)
- [ਚੋਣ A: azd + Bicep ਨਾਲ ਪ੍ਰੌਵੀਜ਼ਨ (ਸਿਫਾਰਸ਼ੀ)](#option-a-provision-with-azd--bicep-recommended)
- [ਚੋਣ B: Resources ਹੱਥੋਂ ਬਣਾਓ](#ਚੋਣ-b-resources-ਹੱਥੋਂ-ਬਣਾਓ)
- [ਆਪਣਾ ਪਰਿਵੇਸ਼ ਸੰਰਚਿਤ ਕਰੋ](#ਆਪਣਾ-ਪਰਿਵੇਸ਼-ਸੰਰਚਿਤ-ਕਰੋ)
- [ਆਪਣੇ ਸੈੱਟਅੱਪ ਦੀ ਜਾਂਚ ਕਰੋ](#ਆਪਣੇ-ਸੈੱਟਅੱਪ-ਦੀ-ਜਾਂਚ-ਕਰੋ)
- [ਅਗਲੇ ਕਦਮ ਕੀ ਹਨ?](#ਅਗਲੇ-ਕਦਮ-ਕੀ-ਹਨ)
- [ਸਾਧਨ](#ਸਾਧਨ)
- [ਅਤਿਰੀਕਤ ਸਾਧਨ](#ਅਤਿਰੀਕਤ-ਸਾਧਨ)

## ਕੀ ਬਣਾਇਆ ਜਾਂਦਾ ਹੈ

[`infra/`](../../../02-SetupDevEnvironment/infra) ਵਿੱਚ Bicep ਟੈਮਪਲੇਟ ਇਹ ਬਣਾਉਂਦੇ ਹਨ:

- ਇੱਕ **Azure AI Foundry** ਖਾਤਾ (`Microsoft.CognitiveServices/accounts`, ਕਿਸਮ `AIServices`) ਅਤੇ ਇੱਕ ਪ੍ਰੋਜੈਕਟ
- ਇੱਕ **chat** ਡਿਪਲੋਇਮੈਂਟ — `gpt-4o-mini`
- ਇੱਕ **embedding** ਡਿਪਲੋਇਮੈਂਟ — `text-embedding-3-small` (ਬਾਅਦਲੇ ਅਧਿਆਇਆਂ ਵਿੱਚ ਵਰਤਿਆ)
- ਇੱਕ **keyless Role Assignment** (`Cognitive Services OpenAI User`) ਤਾਂ ਜੋ ਤੁਹਾਡੇ ਕੋਲ `az login` ਨਾਲ ਸਾਈਨ ਇਨ ਹੋਵਣ ਦਾ ਢੰਗ ਹੋਵੇ, ਕੁੰਜੀਆਂ ਸੰਭਾਲਣ ਦੀ ਜਰੂਰਤ ਨਹੀਂ

## ਪੂਰਵ-ਆਵਸ਼ਯਕਤਾ

- ਇੱਕ [Azure ਸਬਸਕ੍ਰਿਪਸ਼ਨ](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) ਅਤੇ [Maven 3.9+](https://maven.apache.org/download.cgi)

## ਚੋਣ A: azd + Bicep ਨਾਲ ਪ੍ਰੌਵੀਜ਼ਨ (ਸਿਫਾਰਸ਼ੀ)

`02-SetupDevEnvironment` ਫੋਲਡਰ ਤੋਂ:

```bash
cd 02-SetupDevEnvironment

# ਸਾਈਨ ਇਨ ਕਰੋ (ਦੋਹਾਂ ਟੂਲਾਂ ਲਈ)
azd auth login
az login

# ਫਾਊਂਡਰੀ ਖਾਤਾ + ਮਾਡਲ ਡਿਪਲੌਇਮੈਂਟ ਪ੍ਰੋਵਿਜਨ ਕਰੋ
azd up
```

`azd` ਤੁਹਾਡੇ ਤੋਂ ਇੱਕ **environment name** (ਉਦਾਹਰਨ ਲਈ `genai-java`) ਅਤੇ ਇੱਕ **region** ਪੁੱਛਦਾ ਹੈ। ਇੱਕ ਇਲਾਕਾ ਚੁਣੋ ਜਿੱਥੇ `gpt-4o-mini` ਅਤੇ `text-embedding-3-small` ਉਪਲਬਧ ਹਨ — ਉਦਾਹਰਨ ਲਈ `eastus2` ਜਾਂ `swedencentral`।

ਜਦੋਂ ਪ੍ਰੌਵੀਜ਼ਨਿੰਗ ਮੁੱਕ ਜਾਂਦੀ ਹੈ, ਤਦ azd:

1. ਸਾਰਾ ਕੁਝ ਜੋ [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep) ਵਿੱਚ ਦਰਸਾਇਆ ਗਿਆ ਹੈ ਤਫਤੀਸ਼ ਕਰਦਾ ਹੈ।
2. ਇੱਕ post-provision ਹੂਕ ਚਲਾਉਂਦਾ ਹੈ ਜੋ ਤੁਹਾਡੇ ਐਂਡਪੁਆਇੰਟ ਅਤੇ ਡਿਪਲੋਇਮੈਂਟ ਦੇ ਨਾਮਾਂ ਦੇ ਨਾਲ [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) ਲਿਖਦਾ ਹੈ (ਕੋਈ ਸੁਰੰਗ ਨਹੀਂ)।

> **ਸੁਝਾਅ:** ਕੋਈ ਵੀ ਸਮੇਂ `azd up` ਚਲਾਓ ਤਾ ਕਿ ਬਦਲਾਅ ਲਾਗੂ ਹੋ ਜਾਵੇ। ਸਾਰਾ ਕੁਝ ਹਟਾਉਣ ਲਈ `azd down` ਚਲਾਓ।

ਤਿਆਰ ਹੋਏ ਸੈਟਿੰਗ ਵੇਖਣ ਲਈ:

```bash
azd env get-values
```

ਹੁਣ [ਆਪਣੇ ਸੈੱਟਅੱਪ ਦੀ ਜਾਂਚ ਕਰੋ](#ਆਪਣੇ-ਸੈੱਟਅੱਪ-ਦੀ-ਜਾਂਚ-ਕਰੋ) 'ਤੇ ਜਾਓ।

## ਚੋਣ B: Resources ਹੱਥੋਂ ਬਣਾਓ

ਪੋਰਟਲ ਵਰਤਣਾ ਪਸੰਦ ਹੈ? ਹੱਥੋਂ Resources ਬਣਾਓ:

1. [Azure AI Foundry ਪੋਰਟਲ](https://ai.azure.com/) 'ਤੇ ਜਾਓ ਅਤੇ ਸਾਈਨ ਇਨ ਕਰੋ।
2. **ਇੱਕ ਪ੍ਰੋਜੈਕਟ ਬਣਾਓ** (ਜੋ ਕਿ ਇੱਕ AI Foundry ਰਿਸੋਰਸ ਵੀ ਬਣਾਉਂਦਾ ਹੈ)। ਇਸਨੂੰ `GenAIJava` ਵਰਗਾ ਨਾਮ ਦਿਓ।
3. ਆਪਣੇ ਪ੍ਰੋਜੈਕਟ ਵਿੱਚ, **Models + endpoints** → **Deploy model** → **Deploy base model** ਖੋਲ੍ਹੋ।
4. **gpt-4o-mini** ਡਿਪਲੋਇ ਕਰੋ (ਡਿਪਲੋਇਮੈਂਟ ਨਾਮ `gpt-4o-mini`)। ਜੇ ਐਂਬੈੱਡਿੰਗ ਉਦਾਹਰਣਾਂ ਚਾਹੀਦੀਆਂ ਹਨ ਤਾਂ **text-embedding-3-small** ਵੀ ਡਿਪਲੋਇ ਕਰੋ।
5. **Overview** ਤੋਂ, **endpoint** ਕਾਪੀ ਕਰੋ (ਉਦਾਹਰਨ ਲਈ `https://<resource>.openai.azure.com/`)।
6. ਆਪਣੇ ਆਪ ਨੂੰ keyless ਪਹਿਲੀ ਅਗਵਾਈ ਦਿਓ: resource 'ਤੇ ਜਾ ਕੇ **Access control (IAM)** → **Add role assignment** → ਆਪਣੀ ਅਕਾਊਂਟ ਲਈ **Cognitive Services OpenAI User** ਅਸਾਈਨ ਕਰੋ।

> **ਅਜੇ ਵੀ ਮੁਸ਼ਕਿਲਾਂ ਆ ਰਹੀਆਂ ਹਨ?** [Azure AI Foundry ਡਾਕਯੂਮੈਂਟੇਸ਼ਨ](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects) ਵੇਖੋ।

## ਆਪਣਾ ਪਰਿਵੇਸ਼ ਸੰਰਚਿਤ ਕਰੋ

**ਜੇ ਤੁਸੀਂ ਚੋਣ A (`azd up`) ਵਰਤੀ**, ਤੁਹਾਡੀ ਸੈਟਿੰਗ ਫਾਈਲ ਪਹਿਲਾਂ ਹੀ ਲਿਖੀ ਗਈ ਹੈ — ਕੋਈ ਵਧੀਆ ਸੰਰਚਨਾ ਕਰਨ ਦੀ ਲੋੜ ਨਹੀਂ। [ਆਪਣੇ ਸੈੱਟਅੱਪ ਦੀ ਜਾਂਚ ਕਰੋ](#ਆਪਣੇ-ਸੈੱਟਅੱਪ-ਦੀ-ਜਾਂਚ-ਕਰੋ) 'ਤੇ ਜਾਓ।

**ਜੇ ਤੁਸੀਂ ਚੋਣ B (ਮੈਨੁਅਲ) ਵਰਤੀ**, ਤਾਂ ਉਦਾਹਰਣ ਵਾਲੀ `.env` ਫਾਈਲ ਖੁਦ ਬਣਾਉ:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

ਆਪਣੇ ਐਂਡਪੁਆਇੰਟ ਨਾਲ `.env` ਸੰਪਾਦਿਤ ਕਰੋ (ਕੋਈ ਕੁੰਜੀ ਨਹੀਂ — auth keyless ਹੈ):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **ਸੁਰੱਖਿਆ ਨੋਟ:** ਕੋਈ API ਕੁੰਜੀ ਸੰਗ੍ਰਹਿਤ ਕਰਨ ਦੀ ਲੋੜ ਨਹੀਂ। ਤੁਸੀਂ Microsoft Entra ID ਰਾਹੀਂ `az login` (ਸਥਾਨਕ ਤੌਰ 'ਤੇ) ਜਾਂ ਮੈਨੇਜਡ ਇਡੈਂਟਿਟੀ (Azure ਵਿੱਚ) ਨਾਲ ਪ੍ਰਮਾਣਿਕ ਹੁੰਦੇ ਹੋ। `.env` ਫਾਈਲ ਵਿੱਚ ਸਿਰਫ਼ ਗੈਰ-ਸਿਕਰਟ ਸੈਟਿੰਗਾਂ ਹਨ ਅਤੇ ਇਹ `.gitignore` ਨਾਲ ਪਹਿਲਾਂ ਹੀ ਕਵਰ ਕੀਤੀ ਗਈ ਹੈ।

## ਆਪਣੇ ਸੈੱਟਅੱਪ ਦੀ ਜਾਂਚ ਕਰੋ

ਪੱਕਾ ਕਰੋ ਕਿ ਤੁਸੀਂ ਸਾਈਨ ਇਨ ਹੋ ਚੁੱਕੇ ਹੋ ਤਾਂ ਜੋ keyless auth ਟੋਕਨ ਪ੍ਰਾਪਤ ਕਰ ਸਕੇ, ਫਿਰ ਉਦਾਹਰਣ ਚਲਾੳੋ:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # ਜੇ ਤੁਸੀਂ ਪਹਿਲਾਂ ਹੀ ਸਾਈਨ ਇਨ ਨਹੀਂ ਹੋਏ ਹੋ
mvn clean spring-boot:run
```

ਤੁਹਾਨੂੰ `gpt-4o-mini` ਮਾਡਲ ਤੋਂ ਜਵਾਬ ਮਿਲੇਗਾ!

> **VS ਕੋਡ ਵਰਤੋਂਕਾਰ:** `F5` ਦਬਾਓ ਚਲਾਉਣ ਲਈ। ਐਪ ਆਪਣੇ `.env` ਨੂੰ ਆਟੋਮੈਟਿਕ ਲੋਡ ਕਰਦਾ ਹੈ।

> **ਪੂਰਾ ਉਦਾਹਰਣ:** [Basic Chat with Azure AI Foundry example](./examples/basic-chat-azure/README.md) ਵੇਖੋ ਵੇਰਵਿਆਂ ਅਤੇ ਸਮੱਸਿਆ-ਨਿਵਾਰੇ ਲਈ।

## ਅਗਲੇ ਕਦਮ ਕੀ ਹਨ?

**ਸੈੱਟਅੱਪ ਮੁਕੰਮਲ!** ਤੁਹਾਡੇ ਕੋਲ ਹੁਣ ਹਨ:
- Azure AI Foundry ਜਿਸ ਵਿੱਚ `gpt-4o-mini` ਅਤੇ `text-embedding-3-small` ਤਫਤੀਸ਼ ਕੀਤੇ ਗਏ ਹਨ
- Keyless authentication (Microsoft Entra ID) — ਕੋਈ ਕੁੰਜੀਆਂ ਨਹੀਂ ਸੰਭਾਲਣ ਲਈ
- ਇੱਕ ਸਥਾਨਕ `.env` ਜਿਸ ਵਿੱਚ ਤੁਹਾਡਾ ਐਂਡਪੁਆਇੰਟ ਅਤੇ ਡਿਪਲੋਇਮੈਂਟ ਦੇ ਨਾਮ ਹਨ
- ਜਾਵਾ ਵਿਕਾਸ ਪਰਿਵੇਸ਼ ਤਿਆਰ

**ਆਗੇ ਵਧੋ** [ਅਧਿਆਇ 3: ਕੋਰ ਜਨਰੇਟਿਵ ਏਆਈ ਤਕਨੀਕਾਂ](../03-CoreGenerativeAITechniques/README.md) 'ਤੇ ਤਾ ਕਿ ਏਆਈ ਐਪਲੀਕੇਸ਼ਨਾਂ ਦਾ ਨਿਰਮਾਣ ਸ਼ੁਰੂ ਕਰ ਸਕੋ!

## ਸਾਧਨ

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Microsoft Entra ID ਨਾਲ keyless authentication](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry ਡਾਕਯੂਮੈਂਟੇਸ਼ਨ](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI ਡਾਕ्यूਮੈਂਟੇਸ਼ਨ](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI ਜਾਵਾ SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## ਅਤਿਰੀਕਤ ਸਾਧਨ

- [VS ਕੋਡ ਡਾਊਨਲੋਡ ਕਰੋ](https://code.visualstudio.com/Download)
- [ਡੋਕਰ ਡੈਸਕਟਾਪ ਪ੍ਰਾਪਤ ਕਰੋ](https://www.docker.com/products/docker-desktop)
- [Dev Container ਸੰਰਚਨਾ](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ਅਸਵੀਕਾਰੋਪਣ**:
ਇਸ ਦਸਤਾਵੇਜ਼ ਦਾ ਅਨੁਵਾਦ ਏਆਈ ਅਨੁਵਾਦ ਸੇਵਾ [Co-op Translator](https://github.com/Azure/co-op-translator) ਦੀ ਵਰਤੋਂ ਕਰਕੇ ਕੀਤਾ ਗਿਆ ਹੈ। ਜਦੋਂ ਕਿ ਅਸੀਂ ਸਹੀਤਾਵਾਂ ਲਈ ਯਤਨਸ਼ੀਲ ਹਾਂ, ਕਿਰਪਾ ਕਰਕੇ ਧਿਆਨ ਰੱਖੋ ਕਿ ਸਵੈਚਾਲਿਤ ਅਨੁਵਾਦਾਂ ਵਿੱਚ ਗਲਤੀਆਂ ਜਾਂ ਅਸਮੱਤਿਆਵਾਂ ਹੋ ਸਕਦੀਆਂ ਹਨ। ਮੂਲ ਦਸਤਾਵੇਜ਼ ਆਪਣੀ ਮੂਲ ਭਾਸ਼ਾ ਵਿੱਚ ਅਧਿਕਾਰਕ ਸਰੋਤ ਮੰਨਿਆ ਜਾਣਾ ਚਾਹੀਦਾ ਹੈ। ਜਰੂਰੀ ਜਾਣਕਾਰੀ ਲਈ, ਪੇਸ਼ੇਵਰ ਮਨੁੱਖੀ ਅਨੁਵਾਦ ਦੀ ਸਿਫ਼ਾਰਸ਼ ਕੀਤੀ ਜਾਂਦੀ ਹੈ। ਅਸੀਂ ਇਸ ਅਨੁਵਾਦ ਦੇ ਉਪਯੋਗ ਤੋਂ ਪੈਦਾ ਹੋਣ ਵਾਲੀਆਂ ਕਿਸੇ ਵੀ ਗਲਤਫਹਿਮੀਆਂ ਜਾਂ ਗਲਤ ਵਿਆਖਿਆਵਾਂ ਲਈ ਜਵਾਬਦੇਹ ਨਹੀਂ ਹਾਂ।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->