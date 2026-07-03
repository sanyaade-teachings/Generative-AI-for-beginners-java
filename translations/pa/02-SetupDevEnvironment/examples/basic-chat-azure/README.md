# Azure AI Foundry ਨਾਲ ਬੁਨਿਆਦੀ ਗੱਲਬਾਤ - ਅੰਤ ਤੱਕ ਉਦਾਹਰਨ

ਇਹ ਉਦਾਹਰਨ ਇੱਕ ਸਧਾਰਣ ਸਪ੍ਰਿੰਗ ਬੂਟ ਐਪਲੀਕੇਸ਼ਨ ਹੈ ਜੋ **Azure AI Foundry** ਮਾਡਲ ਨਾਲ **ਕੀਲੈੱਸ ਪ੍ਰਮਾਣਿਕਤਾ** (Microsoft Entra ID) ਦੀ ਵਰਤੋਂ ਕਰਕੇ ਜੁੜਦਾ ਹੈ ਅਤੇ ਤੁਹਾਡੇ ਸੈਟਅੱਪ ਦੀ ਜਾਂਚ ਕਰਦਾ ਹੈ। ਇਹ Spring AI ਦੇ `ChatClient` ਦੀ ਵਰਤੋਂ ਕਰਦਾ ਹੈ।

## ਮਿਸ਼ਰਣ ਸੂਚੀ

- [ਆਵਸ਼ਯਕਤਾਵਾਂ](#ਆਵਸ਼್ಯਕਤਾਵਾਂ)
- [ਤੁਰੰਤ ਸ਼ੁਰੂਆਤ](#ਤੁਰੰਤ-ਸ਼ੁਰੂਆਤ)
- [ਪ੍ਰਮਾਣਿਕਤਾ ਕਿਵੇਂ ਕੰਮ ਕਰਦੀ ਹੈ](#ਪ੍ਰਮਾਣਿਕਤਾ-ਕਿਵੇਂ-ਕੰਮ-ਕਰਦੀ-ਹੈ)
- [ਐਪਲੀਕੇਸ਼ਨ ਚਲਾਉਣਾ](#ਐਪਲੀਕੇਸ਼ਨ-ਚਲਾਉਣਾ)
  - [Maven ਦੀ ਵਰਤੋਂ](#maven-ਦੀ-ਵਰਤੋਂ)
  - [VS ਕੋਡ ਦੀ ਵਰਤੋਂ](#vs-ਕੋਡ-ਦੀ-ਵਰਤੋਂ)
  - [ਉਮੀਦਵਾਰ ਨਤੀਜਾ](#ਉਮੀਦਵਾਰ-ਨਤੀਜਾ)
- [ਸੈਟਿੰਗ ਦਾ ਹਵਾਲਾ](#ਸੈਟਿੰਗ-ਦਾ-ਹਵਾਲਾ)
  - [ਪ੍ਰਦੂਸ਼ਿਤ ਚਰ (Environment Variables)](#ਪ੍ਰਦੂਸ਼ਿਤ-ਚਰ-environment-variables)
  - [ਸਪ੍ਰਿੰਗ ਸੈਟਿੰਗ](#ਸਪ੍ਰਿੰਗ-ਸੈਟਿੰਗ)
- [ਸਮੱਸਿਆ ਹੱਲ](#ਸਮੱਸਿਆ-ਹੱਲ)
  - [ਆਮ ਸਮੱਸਿਆਵਾਂ](#ਆਮ-ਸਮੱਸਿਆਵਾਂ)
  - [ਡਿਬੱਗ ਮੋਡ](#ਡਿਬੱਗ-ਮੋਡ)
- [ਅਗਲੇ ਕਦਮ](#ਅਗਲੇ-ਕਦਮ)
- [ਸੰਸਾਧਨ](#ਸੰਸਾਧਨ)

## ਆਵਸ਼್ಯਕਤਾਵਾਂ

ਇਹ ਉਦਾਹਰਨ ਚਲਾਉਣ ਤੋਂ ਪਹਿਲਾਂ, ਇਹ ਯਕੀਨੀ ਬਣਾਓ ਕਿ ਤੁਹਾਡੇ ਕੋਲ ਹੈ:

- ਇੱਕ Azure AI Foundry ਸਾਧਨ ਜਿਸ ਵਿੱਚ `gpt-4o-mini` ਡਿਪਲੌਇਮੈਂਟ ਹੋਵੇ — ਇਸਨੂੰ `azd up` ਨਾਲ ਜਾਂ ਹੱਥੋਂਹੱਥ [Azure AI Foundry ਸੈਟਅਪ ਗਾਈਡ](../../getting-started-azure-openai.md) ਦੇ ਜ਼ਰੀਏ ਪ੍ਰੋਵਾਈਜਨ ਕਰੋ
- ਉਸ ਸਾਧਨ 'ਤੇ **Cognitive Services OpenAI User** ਰੋਲ (ਜੋ Bicep ਟੈਮਪਲੇਟ ਤੁਹਾਡੇ ਲਈ ਨਿਰਧਾਰਿਤ ਕਰਦੇ ਹਨ)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), ਜਿਸ ਵਿੱਚ `az login` ਨਾਲ ਸਾਈਨ ਇਨ ਕੀਤਾ ਹੋਵੇ
- Java 21+ ਅਤੇ Maven 3.9+

> **API ਕੁੰਜੀ ਦੀ ਲੋੜ ਨਹੀਂ** — ਪ੍ਰਮਾਣਿਕਤਾ Microsoft Entra ID ਰਾਹੀਂ ਕੀਲੈੱਸ ਹੈ।

## ਤੁਰੰਤ ਸ਼ੁਰੂਆਤ

```bash
# 1. ਪ੍ਰੋਜੈਕਟ ਤੇ ਜਾਓ
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. ਸਾਈਨ ਇਨ ਕਰੋ ਤਾਂ ਜੋ ਕੀਲੈੱਸ ਅਥਾਰਟੀਕੇਸ਼ਨ ਟੋਕਨ ਪ੍ਰਾਪਤ ਕਰ ਸਕੇ
az login

# 3. ਐਂਡਪੋਇੰਟ ਨੂੰ ਸੰਰਚਿਤ ਕਰੋ
#    - ਜੇ ਤੁਸੀਂ `azd up` ਚਲਾਇਆ ਸੀ, ਤਾਂ .env ਤੁਹਾਡੇ ਲਈ ਲਿਖ ਦਿੱਤਾ ਗਿਆ ਸੀ (ਇਸ ਨੂੰ ਛੱਡੋ).
#    - ਨਹੀਂ ਤਾਂ ਟੈਂਪਲੇਟ ਦੀ ਨਕਲ ਕਰੋ ਅਤੇ AZURE_OPENAI_ENDPOINT ਸੈਟ ਕਰੋ:
cp .env.example .env

# 4. ਐਪਲੀਕੇਸ਼ਨ ਚਲਾਓ
mvn spring-boot:run
```

## ਪ੍ਰਮਾਣਿਕਤਾ ਕਿਵੇਂ ਕੰਮ ਕਰਦੀ ਹੈ

ਇਹ ਉਦਾਹਰਨ **Microsoft Entra ID** ਦੇ ਨਾਲ ਪ੍ਰਮਾਣਿਕਤਾ ਕਰਦੀ ਹੈ — ਕੋਈ API ਕੁੰਜੀ ਨਹੀਂ ਹੈ।

ਜਦੋਂ ਸਿਰਫ਼ `spring.ai.azure.openai.endpoint` ਸੈੱਟ ਹੁੰਦਾ ਹੈ (ਅਤੇ ਕੋਈ api-key ਨਹੀਂ), Spring AI Azure OpenAI ਕਲਾਇੰਟ ਦਾ ਨਿਰਮਾਣ [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential) ਨਾਲ ਕਰਦਾ ਹੈ। ਇਹ ਕਰੈਡੇਂਸ਼ੀਅਲ ਆਪਣੇ ਆਪ ਤੁਹਾਡੇ `az login` ਸੈਸ਼ਨ ਤੋਂ ਟੋਕਨ ਲੱਭਦਾ ਹੈ, ਜਾਂ ਜਦੋਂ Azure ਵਿੱਚ ਚੱਲਾਇਆ ਜਾ ਰਿਹਾ ਹੋਵੇ ਤਾਂ ਮੈਨੇਜਡ ਆਈਡੈਂਟਿਟੀ ਤੋਂ — ਤਾਂ ਇਸੇ ਕੋਡ ਨਾਲ ਦੋਹਾਂ ਥਾਵਾਂ ਬਿਨਾਂ ਬਦਲਾਅ ਦੇ ਕੰਮ ਕਰਦਾ ਹੈ।

## ਐਪਲੀਕੇਸ਼ਨ ਚਲਾਉਣਾ

### Maven ਦੀ ਵਰਤੋਂ

```bash
mvn spring-boot:run
```

### VS ਕੋਡ ਦੀ ਵਰਤੋਂ

1. ਪ੍ਰੋਜੈਕਟ ਨੂੰ VS ਕੋਡ ਵਿੱਚ ਖੋਲ੍ਹੋ
2. `F5` ਦਬਾਓ ਜਾਂ "Run and Debug" ਪੈਨਲ ਦੀ ਵਰਤੋਂ ਕਰੋ
3. "Spring Boot-BasicChatApplication" ਕਨਫਿਗਰੇਸ਼ਨ ਚੁਣੋ

> **ਨੋਟ**: VS ਕੋਡ ਕਨਫਿਗਰੇਸ਼ਨ ਆਪਣੇ ਆਪ ਤੁਹਾਡੀ .env ਫ਼ਾਈਲ ਲੋਡ ਕਰਦਾ ਹੈ

### ਉਮੀਦਵਾਰ ਨਤੀਜਾ

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

## ਸੈਟਿੰਗ ਦਾ ਹਵਾਲਾ

### ਪ੍ਰਦੂਸ਼ਿਤ ਚਰ (Environment Variables)

| ਵੇਰੀਏਬਲ | ਵੇਰਵਾ | ਲੋੜੀਂਦਾ | ਉਦਾਹਰਨ |
|----------|--------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | Foundry (Azure OpenAI) ਐਂਡਪੌਇੰਟ URL | ਹਾਂ | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | ਗੱਲਬਾਤ ਮਾਡਲ ਡਿਪਲੌਇਮੈਂਟ ਨਾਮ | ਨਹੀਂ | `gpt-4o-mini` (ਮੂਲ) |

> ਕੋਈ API ਕੁੰਜੀ ਵੇਰੀਏਬਲ ਨਹੀਂ ਹੈ — ਪ੍ਰਮਾਣਿਕਤਾ ਕੀਲੈੱਸ ਹੈ (Microsoft Entra ID ਰਾਹੀਂ `az login`)।

### ਸਪ੍ਰਿੰਗ ਸੈਟਿੰਗ

`application.yml` ਫਾਈਲ ਕਨਫਿਗਰੇਸ਼ਨ:

- **ਐਂਡਪੌਇੰਟ**: `${AZURE_OPENAI_ENDPOINT}` - Environment variables ਤੋਂ
- **ਡਿਪਲੌਇਮੈਂਟ**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - Environment variables ਤੋਂ ਪੈਲਟ ਰਹਿਤ
- **ਪ੍ਰਮਾਣਿਕਤਾ**: ਕੀਲੈੱਸ — ਕੋਈ `api-key` ਨਹੀਂ ਸੈੱਟ, ਇਸ ਲਈ Spring AI `DefaultAzureCredential` ਵਰਤਦਾ ਹੈ
- **ਟੈਂਪਰੇਚਰ**: `0.7` - ਰਚਨਾਤਮਕਤਾ ਕੰਟਰੋਲ ਕਰਦਾ ਹੈ (0.0 = ਨਿਧਾਰਤ, 1.0 = ਰਚਨਾਤਮਕ)
- **ਅਧਿਕਤਮ ਟੋਕੇਨਸ**: `500` - ਜਵਾਬ ਦੀ ਵੱਧੋਤਮ ਲੰਬਾਈ

## ਸਮੱਸਿਆ ਹੱਲ

### ਆਮ ਸਮੱਸਿਆਵਾਂ

<details>
<summary><strong>ਤੁੱਟੀ: 401 / "PermissionDenied" / ਟੋਕਨ ਗਲਤੀਆਂ</strong></summary>

- `az login` ਦੌੜਾਓ — ਕੀਲੈੱਸ ਪ੍ਰਮਾਣਿਕਤਾ ਲਈ ਸਰਗਰਮ ਲਾਗ ਇਨ ਲੋੜੀਂਦਾ ਹੈ
- ਯਕੀਨੀ ਬਣਾਓ ਕਿ ਤੁਹਾਡੇ ਅਕਾਊਂਟ ਨੂੰ ਸਾਧਨ 'ਤੇ **Cognitive Services OpenAI User** ਰੋਲ ਮਿਲਿਆ ਹੋਇਆ ਹੈ
- ਜੇ ਰੋਲ ਹੁਣੇ ਅਸਾਈਨ ਕੀਤਾ ਹੈ, ਤਾਂ ਇਸ ਦੇ ਲਾਗੂ ਹੋਣ ਲਈ ਕੁਝ ਸਮਾਂ ਦਿਓ
- ਪੱਕਾ ਕਰੋ ਕਿ ਤੁਸੀਂ ਸਹੀ ਟੀਨੈਂਟ/ਸਬਸਕ੍ਰਿਪਸ਼ਨ ਵਿੱਚ ਹੋ (`az account show`)
</details>

<details>
<summary><strong>ਤੁੱਟੀ: "ਦਿ ਐਂਡਪੌਇੰਟ ਇਸਨੂੰ ਵੈਧ ਨਹੀਂ" / ਕਨੈਕਸ਼ਨ ਗਲਤੀਆਂ</strong></summary>

- ਯਕੀਨੀ ਬਣਾਓ ਕਿ `AZURE_OPENAI_ENDPOINT` ਪੂਰਾ ਬੇਸ URL ਹੈ (ਜਿਵੇਂ `https://your-resource.openai.azure.com/`)
- ਟ੍ਰੇਲਿੰਗ ਸਲੈਸ਼ ਦੀ ਸਥਿਰਤਾ ਜਾਂਚੋ
- ਇਹ ਪੱਕਾ ਕਰੋ ਕਿ ਐਂਡਪੌਇੰਟ ਤੁਹਾਡੇ ਪ੍ਰੋਵਾਈਜਨ ਕੀਤੇ ਸਾਧਨ ਨਾਲ ਮਿਲਦਾ ਹੈ (`azd env get-values`)
</details>

<details>
<summary><strong>ਤੁੱਟੀ: "ਡਿਪਲੌਇਮੈਂਟ ਨਹੀਂ ਮਿਲਿਆ"</strong></summary>

- ਯਕੀਨੀ ਬਣਾਓ ਕਿ `AZURE_OPENAI_DEPLOYMENT` Azure ਵਿੱਚ ਕਿਸੇ ਡਿਪਲੌਇਮੈਂਟ ਨਾਮ ਨਾਲ ਮਿਲਦਾ ਹੈ
- ਯਕੀਨੀ ਬਣਾਓ ਕਿ ਮਾਡਲ ਸਫਲਤਾਪੂਰਕ ਡਿਪਲੌਇ ਕੀਤਾ ਗਿਆ ਅਤੇ ਸਰਗਰਮ ਹੈ
- ਮੂਲ ਡਿਪਲੌਇਮੈਂਟ ਨਾਮ `gpt-4o-mini` ਹੈ
</details>

<details>
<summary><strong>VS ਕੋਡ: Environment variables ਲੋਡ ਨਹੀਂ ਹੋ ਰਹੀਆਂ</strong></summary>

- ਪੱਕਾ ਕਰੋ ਕਿ ਤੁਹਾਡੀ `.env` ਫਾਈਲ ਪ੍ਰੋਜੈਕਟ ਰੂਟ ਫੋਲਡਰ ਵਿੱਚ ਹੈ (`pom.xml` ਦੇ ਸਮਾਨ ਪੱਧਰ)
- ਕੋਸ਼ਿਸ਼ ਕਰੋ ਕਿ VS ਕੋਡ ਦੇ ਇੰਟੀਗਰੇਟਿਡ ਟਰਮੀਨਲ ਵਿੱਚ `mvn spring-boot:run` ਦੌੜਾਓ
- ਜਾਂਚੋ ਕਿ VS ਕੋਡ ਜਾਵਾ ਐਕਸਟੈਨਸ਼ਨ ਸਹੀ ਤਰ੍ਹਾਂ ਇੰਸਟਾਲ ਹੈ
</details>

### ਡਿਬੱਗ ਮੋਡ

ਵਿਸਥਾਰਿਤ ਲਾਗਿੰਗ ਲਈ, `application.yml` ਵਿੱਚ ਇਹ ਲਾਈਨਾਂ ਅਣਕੰਮੈਂਟ ਕਰੋ:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## ਅਗਲੇ ਕਦਮ

**ਸੈਟਅੱਪ ਮੁਕੰਮਲ!** ਆਪਣੀ ਸਿੱਖਿਆ ਯਾਤਰਾ ਜਾਰੀ ਰੱਖੋ:

[ਅਧਿਆਇ 3: ਕੋਰ ਜਨਰੈਟਿਵ AI ਤਕਨੀਕਾਂ](../../../03-CoreGenerativeAITechniques/README.md)

## ਸੰਸਾਧਨ

- [Spring AI Azure OpenAI ਦਸਤਾਵੇਜ਼](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Microsoft Entra ID ਨਾਲ ਕੀਲੈੱਸ ਪ੍ਰਮਾਣਿਕਤਾ](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry ਪੋਰਟਲ](https://ai.azure.com/)
- [Azure AI Foundry ਦਸਤਾਵੇਜ਼](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ਅਸਵੀਕਾਰੋਪਣ**:
ਇਸ ਦਸਤਾਵੇਜ਼ ਦਾ ਅਨੁਵਾਦ ਏਆਈ ਅਨੁਵਾਦ ਸੇਵਾ [Co-op Translator](https://github.com/Azure/co-op-translator) ਦੀ ਵਰਤੋਂ ਕਰਕੇ ਕੀਤਾ ਗਿਆ ਹੈ। ਜਦੋਂ ਕਿ ਅਸੀਂ ਸਹੀਤਾਵਾਂ ਲਈ ਯਤਨਸ਼ੀਲ ਹਾਂ, ਕਿਰਪਾ ਕਰਕੇ ਧਿਆਨ ਰੱਖੋ ਕਿ ਸਵੈਚਾਲਿਤ ਅਨੁਵਾਦਾਂ ਵਿੱਚ ਗਲਤੀਆਂ ਜਾਂ ਅਸਮੱਤਿਆਵਾਂ ਹੋ ਸਕਦੀਆਂ ਹਨ। ਮੂਲ ਦਸਤਾਵੇਜ਼ ਆਪਣੀ ਮੂਲ ਭਾਸ਼ਾ ਵਿੱਚ ਅਧਿਕਾਰਕ ਸਰੋਤ ਮੰਨਿਆ ਜਾਣਾ ਚਾਹੀਦਾ ਹੈ। ਜਰੂਰੀ ਜਾਣਕਾਰੀ ਲਈ, ਪੇਸ਼ੇਵਰ ਮਨੁੱਖੀ ਅਨੁਵਾਦ ਦੀ ਸਿਫ਼ਾਰਸ਼ ਕੀਤੀ ਜਾਂਦੀ ਹੈ। ਅਸੀਂ ਇਸ ਅਨੁਵਾਦ ਦੇ ਉਪਯੋਗ ਤੋਂ ਪੈਦਾ ਹੋਣ ਵਾਲੀਆਂ ਕਿਸੇ ਵੀ ਗਲਤਫਹਿਮੀਆਂ ਜਾਂ ਗਲਤ ਵਿਆਖਿਆਵਾਂ ਲਈ ਜਵਾਬਦੇਹ ਨਹੀਂ ਹਾਂ।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->