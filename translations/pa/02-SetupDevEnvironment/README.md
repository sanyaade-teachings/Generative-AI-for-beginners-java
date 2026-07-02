# ਜੇਨੇਰੇਟਿਵ ਏਆਈ ਫੋਰ ਜਾਵਾ ਲਈ ਵਿਕਾਸ ਦਾ ਮਾਹੌਲ ਸੈਟਅਪ ਕਰਨਾ

> **ਫਟਾਫਟ ਸ਼ੁਰੂਆਤ:** ਆਪਣੇ ਏਆਈ ਮਾਡਲਾਂ ਨੂੰ ਕੁਝ ਮਿੰਟਾਂ ਵਿੱਚ ਕੋਡ ਦੇ ਤੌਰ ਤੇ **Azure AI Foundry** 'ਤੇ ਬਾਇਸਪ + `azd` ਨਾਲ ਪ੍ਰੋਵਿਜ਼ਨ ਕਰੋ — ਵੇਖੋ [Azure AI Foundry ਸੈਟਅਪ ਗਾਈਡ](getting-started-azure-openai.md)। ਪ੍ਰਮਾਣਿਕਤਾ **ਕੀਲੈੱਸ** ਹੈ (Microsoft Entra ID), ਇਸ ਲਈ ਕੋਈ API ਕੁੰਜੀਆਂ ਪ੍ਰਬੰਧਿਤ ਕਰਨ ਦੀ ਲੋੜ ਨਹੀਂ ਹੈ।

## ਤੁਸੀਂ ਕੀ ਸਿੱਖੋਗੇ

- ਏਆਈ ਐਪਲੀਕੇਸ਼ਨਾਂ ਲਈ ਜਾਵਾ ਵਿਕਾਸ ਦਾ ਮਾਹੌਲ ਸੈਟਅਪ ਕਰਨਾ  
- ਆਪਣੀ ਪਸੰਦ ਦੀ ਵਿਕਾਸ ਮਾਹੌਲ ਚੁਣੋ ਅਤੇ ਕਨਫਿਗਰ ਕਰੋ (ਕਲਾਉਡ-ਪਹਿਲਾ ਕੋਡਸਪੇਸਸ, ਲੋਕਲ ਡੈਵ ਕੰਟੇਨਰ, ਜਾਂ ਪੂਰਾ ਲੋਕਲ ਸੈਟਅਪ)  
- Azure AI Foundry ਮਾਡਲ ਨਾਲ ਜੁੜ ਕੇ ਆਪਣਾ ਸੈਟਅਪ ਟੈਸਟ ਕਰੋ  

## ਸੂਚੀ

- [ਤੁਸੀਂ ਕੀ ਸਿੱਖੋਗੇ](#ਤੁਸੀਂ-ਕੀ-ਸਿੱਖੋਗੇ)  
- [ਪ੍ਰਸਤਾਵਨਾ](#ਪ੍ਰਸਤਾਵਨਾ)  
- [ਕਦਮ 1: ਆਪਣੇ ਵਿਕਾਸ ਮਾਹੌਲ ਸੈੱਟ ਕਰੋ](#ਕਦਮ-1-ਆਪਣੇ-ਵਿਕਾਸ-ਮਾਹੌਲ-ਸੈੱਟ-ਕਰੋ)  
  - [ਵਿਕਲਪ A: GitHub Codespaces (ਸਿਫਾਰਸੀ)](#ਵਿਕਲਪ-a-github-codespaces-ਸਿਫਾਰਸੀ)  
  - [ਵਿਕਲਪ B: ਲੋਕਲ ਡੈਵ ਕੰਟੇਨਰ](#ਵਿਕਲਪ-b-ਲੋਕਲ-ਡੈਵ-ਕੰਟੇਨਰ)  
  - [ਵਿਕਲਪ C: ਆਪਣਾ ਮੌਜੂਦਾ ਲੋਕਲ ਇੰਸਟਾਲੇਸ਼ਨ ਵਰਤੋ](#ਵਿਕਲਪ-c-ਆਪਣਾ-ਮੌਜੂਦਾ-ਲੋਕਲ-ਇੰਸਟਾਲੇਸ਼ਨ-ਵਰਤੋ)  
- [ਕਦਮ 2: Azure AI Foundry ਪ੍ਰੋਵਿਜ਼ਨ ਕਰੋ](#ਕਦਮ-2-azure-ai-foundry-ਪ੍ਰੋਵਿਜ਼ਨ-ਕਰੋ)  
- [ਕਦਮ 3: ਆਪਣਾ ਸੈਟਅਪ ਟੈਸਟ ਕਰੋ](#ਕਦਮ-3-ਆਪਣਾ-ਸੈਟਅਪ-ਟੈਸਟ-ਕਰੋ)  
- [ਟ੍ਰਬਲਸ਼ੂਟਿੰਗ](#ਟ੍ਰਬਲਸ਼ੂਟਿੰਗ)  
- [ਸਾਰ](#ਸਾਰ)  
- [ਅਗਲੇ ਕਦਮ](#ਅਗਲੇ-ਕਦਮ)  

## ਪ੍ਰਸਤਾਵਨਾ

ਇਸ ਅਧਿਆਇ ਵਿੱਚ ਅਸੀਂ ਤੁਹਾਨੂੰ ਵਿਕਾਸ ਦਾ ਮਾਹੌਲ ਸੈੱਟਅਪ ਕਰਨ ਦਾ ਮਾਰਗਦਰਸ਼ਨ ਕਰਾਂਗੇ। ਅਸੀਂ ਸਾਰੇ ਕੋਰਸ ਲਈ ਮਾਡਲਾਂ ਲਈ **Azure AI Foundry** ਵਰਤਾਂਗੇ। ਤੁਸੀਂ ਮਾਡਲਾਂ ਨੂੰ ਬਾਇਸਪ ਅਤੇ Azure Developer CLI (`azd`) ਨਾਲ ਕੋਡ ਦੇ ਤੌਰ 'ਤੇ ਪ੍ਰੋਵਿਜਨ ਕਰਦੇ ਹੋ, ਫਿਰ **ਕੀਲੈੱਸ ਪ੍ਰਮਾਣਿਕਤਾ** (Microsoft Entra ID) ਨਾਲ ਜੁੜਦੇ ਹੋ — ਕੋਈ API ਕੁੰਜੀਆਂ ਕਾਪੀ ਜਾਂ ਲੀਕ ਕਰਨ ਦੀ ਲੋੜ ਨਹੀਂ।

**ਕੋئی ਲੋਕਲ ਸੈਟਅਪ ਲੋੜੀਂਦਾ ਨਹੀਂ!** ਤੁਸੀਂ GitHub Codespaces ਵਰਤ ਸਕਦੇ ਹੋ, ਜੋ ਤੁਹਾਡੇ ਬਰਾਊਜ਼ਰ ਵਿੱਚ ਪੂਰਾ ਵਿਕਾਸ ਮਾਹੌਲ ਮੁਹੱਈਆ ਕਰਦਾ ਹੈ, ਅਤੇ ਉਥੋਂ Foundry ਪ੍ਰੋਵਿਜਨ ਕਰ ਸਕਦੇ ਹੋ।

ਅਸੀਂ ਇਸ ਕੋਰਸ ਲਈ **Azure AI Foundry** ਇਸ ਲਈ ਵਰਤਦੇ ਹਾਂ ਕਿਉਂਕਿ ਇਹ ਹੈ:  
- **ਕੋਡ ਦੇ ਤੌਰ 'ਤੇ ਪ੍ਰੋਵਿਜਨ ਕੀਤਾ ਗਿਆ** — ਇੱਕ `azd up` ਖਾਤਾ ਅਤੇ ਮਾਡਲ ਡਿਪਲੌਇਮੈਂਟ ਤਿਆਰ ਕਰਦਾ ਹੈ  
- **ਕੀਲੈੱਸ** — Azure ਸਾਇਨ-ਇਨ ਜਾਂ ਮੈਨੇਜ ਕੀਤੀ ਪਹਚਾਣ ਨਾਲ ਪ੍ਰਮਾਣਿਤ  
- **ਪ੍ਰੋਡਕਸ਼ਨ-ਰੇਡੀ** — ਇੱਕੋ ਕੋਡ ਲੋਕਲ ਅਤੇ Azure ਦੋਹਾਂ 'ਤੇ ਚੱਲਦਾ ਹੈ  
- **ਲਚਕੀਲਾ** — ਡਿਪਲੌਇਮੈਂਟ ਦੇ ਨਾਮ ਬਦਲ ਕੇ ਮਾਡਲ ਬਦਲੋ, ਨਾ ਕਿ ਆਪਣੇ ਕੋਡ ਨੂੰ  

> **ਟਿੱਪਣੀ**: Azure AI Foundry ਡਿਪਲੌਇਮੈਂਟ ਟੋਕਨ ਮੁਤਾਬਕ ਬਿਲ ਕੀਤੇ ਜਾਂਦੇ ਹਨ (ਪੇਅ-ਏਜ਼-ਯੂ-ਗੋ)। ਵੇਰਵੇ ਲਈ [Azure AI Foundry ਸੈਟਅਪ ਗਾਈਡ](getting-started-azure-openai.md) ਵੇਖੋ।  

## ਕਦਮ 1: ਆਪਣੇ ਵਿਕਾਸ ਮਾਹੌਲ ਸੈੱਟ ਕਰੋ

<a name="quick-start-cloud"></a>

ਅਸੀਂ ਵਧੇਰੇ ਸੈਟਅਪ ਸਮਾਂ ਘਟਾਉਣ ਅਤੇ Generative AI for Java ਕੋਰਸ ਲਈ ਲੋੜੀਂਦੇ ਸਾਰੇ ਟੂਲਾਂ ਨੂੰ ਸੁਨਿਸ਼ਚਿਤ ਕਰਨ ਵਾਸਤੇ ਇਕ ਪ੍ਰੀਕਨਫਿਗਰਡ ਡੈਵਲਪਮੈਂਟ ਕੰਟੇਨਰ ਬਣਾਇਆ ਹੈ। ਆਪਣੇ ਫਸੰਦ ਦਾ ਵਿਕਾਸ ਵਿਧੀ ਚੁਣੋ:

### ਮਾਹੌਲ ਸੈਟਅਪ ਵਿਕਲਪ:

#### ਵਿਕਲਪ A: GitHub Codespaces (ਸਿਫਾਰਸੀ)

**2 ਮਿੰਟਾਂ ਵਿੱਚ ਕੋਡਿੰਗ ਸ਼ੁਰੂ ਕਰੋ - ਕੋਈ ਲੋਕਲ ਸੈਟਅਪ ਲੋੜੀਂਦਾ ਨਹੀਂ!**

1. ਇਸ ਰੀਪੋਜ਼ਿਟਰੀ ਨੂੰ ਆਪਣੇ GitHub ਖਾਤੇ ਵਿੱਚ ਫੋਰਕ ਕਰੋ  
   > **ਨੋਟ**: ਜੇ ਤੁਸੀਂ ਬੁਨਿਆਦੀ ਸੰਰਚਨਾ ਸੋਧਣਾ ਚਾਹੁੰਦੇ ਹੋ ਤਾਂ [Dev Container Configuration](../../../.devcontainer/devcontainer.json) ਨੂੰ ਵੇਖੋ  
2. **Code** → **Codespaces** ਟੈਬ → **...** → **New with options...** 'ਤੇ ਕਲਿੱਕ ਕਰੋ  
3. ਡਿਫੌਲਟ ਵਰਤੋ – ਇਹ **Dev container configuration** ਚੁਣੇਗਾ: **Generative AI Java Development Environment** ਜੋ ਇਸ ਕੋਰਸ ਲਈ ਬਣਾਇਆ ਗਿਆ ਹੈ  
4. **Create codespace** 'ਤੇ ਕਲਿੱਕ ਕਰੋ  
5. ਮਾਹੌਲ ਨੂੰ ਤਿਆਰ ਹੋਣ ਲਈ ~2 ਮਿੰਟ ਦੀ ਉਡੀਕ ਕਰੋ  
6. ਜਾਵੋ [ਕਦਮ 2: Azure AI Foundry ਪ੍ਰੋਵਿਜ਼ਨ ਕਰੋ](#ਕਦਮ-2-ਪ੍ਰੋਵਿਜ਼ਨ-azure-ai-foundry)  

<img src="../../../translated_images/pa/codespaces.9945ded8ceb431a5.webp" alt="স্ক্রীਨশট: Codespaces submenu" width="50%">

<img src="../../../translated_images/pa/image.833552b62eee7766.webp" alt="스크린샷 : New with options" width="50%">

<img src="../../../translated_images/pa/codespaces-create.b44a36f728660ab7.webp" alt="스크린샷: Create codespace options" width="50%">

> **Codespaces ਦੇ ਫਾਇਦੇ**:  
> - ਕੋਈ ਲੋਕਲ ਇੰਸਟਾਲੇਸ਼ਨ ਲੋੜੀਂਦੀ ਨਹੀਂ  
> - ਕਿਸੇ ਵੀ ਡਿਵਾਈਸ ਤੇ ਬਰਾਊਜ਼ਰ ਨਾਲ ਕੰਮ ਕਰਦਾ ਹੈ  
> - ਸਾਰੇ ਟੂਲਾਂ ਅਤੇ ਡਿਪੈਂਡੇਂਸੀਜ਼ ਨਾਲ ਪਹਿਲਾਂ ਤੋਂ ਸੰਰਚਿਤ  
> - ਪ੍ਰਤੀ ਮਹੀਨਾ ਨਿੱਜੀ ਖਾਤਿਆਂ ਲਈ 60 ਘੰਟੇ ਮੁਫ਼ਤ  
> - ਸਾਰੇ ਵਿਦਿਆਰਥੀਆਂ ਲਈ ਇਕਸਾਰ ਮਾਹੌਲ  

#### ਵਿਕਲਪ B: ਲੋਕਲ ਡੈਵ ਕੰਟੇਨਰ

**ਜਿਹੜੇ ਵਿਕਾਸਕਾਰ ਲੋਕਲ Docker ਨਾਲ ਵਿਕਾਸ ਚਾਹੁੰਦੇ ਹਨ**

1. ਇਸ ਰੀਪੋਜ਼ਿਟਰੀ ਨੂੰ ਆਪਣੇ ਲੋਕਲ ਮਸ਼ੀਨ 'ਤੇ ਫੋਰਕ ਅਤੇ ਕਲੋਨ ਕਰੋ  
   > **ਨੋਟ**: ਕਿਸੇ ਵੀ ਬੁਨਿਆਦੀ ਸੰਰਚਨਾ ਦੀ ਸੋਧ ਲਈ ਵੇਖੋ [Dev Container Configuration](../../../.devcontainer/devcontainer.json)  
2. [Docker Desktop](https://www.docker.com/products/docker-desktop/) ਅਤੇ [VS Code](https://code.visualstudio.com/) ਇੰਸਟਾਲ ਕਰੋ  
3. VS Code ਵਿੱਚ [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) ਇੰਸਟਾਲ ਕਰੋ  
4. ਰੀਪੋਜ਼ਿਟਰੀ ਫੋਲਡਰ VS Code ਵਿੱਚ ਖੋਲ੍ਹੋ  
5. ਜਦੋਂ ਪੁੱਛਿਆ ਜਾਵੇ, **Reopen in Container** 'ਤੇ ਕਲਿੱਕ ਕਰੋ (ਜਾਂ `Ctrl+Shift+P` → "Dev Containers: Reopen in Container" ਵਰਤੋ)  
6. ਕੰਟੇਨਰ ਨੂੰ ਬਣਨ ਅਤੇ ਸ਼ੁਰੂ ਹੋਣ ਦੀ ਉਡੀਕ ਕਰੋ  
7. ਜਾਵੋ [ਕਦਮ 2: Azure AI Foundry ਪ੍ਰੋਵਿਜ਼ਨ ਕਰੋ](#ਕਦਮ-2-ਪ੍ਰੋਵਿਜ਼ਨ-azure-ai-foundry)  

<img src="../../../translated_images/pa/devcontainer.21126c9d6de64494.webp" alt="스크린샷 : Dev container setup" width="50%">

<img src="../../../translated_images/pa/image-3.bf93d533bbc84268.webp" alt="스크린샷 : Dev container build complete" width="50%">

#### ਵਿਕਲਪ C: ਆਪਣਾ ਮੌਜੂਦਾ ਲੋਕਲ ਇੰਸਟਾਲੇਸ਼ਨ ਵਰਤੋ

**ਉਹ ਵਿਕਾਸਕਾਰ ਜਿਨ੍ਹਾਂ ਕੋਲ ਪਹਿਲਾਂ ਤੋਂ ਜਾਵਾ ਮਾਹੌਲ ਹੈ**

ਜ਼ਰੂਰੀ ਚੀਜ਼ਾਂ:  
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html)  
- [Maven 3.9+](https://maven.apache.org/download.cgi)  
- [VS Code](https://code.visualstudio.com) ਜਾਂ ਆਪਣਾ ਮਨਪਸੰਦ IDE  

ਕਦਮ:  
1. ਇਸ ਰੀਪੋਜ਼ਿਟਰੀ ਨੂੰ ਆਪਣੇ ਲੋਕਲ ਮਸ਼ੀਨ 'ਤੇ ਕਲੋਨ ਕਰੋ  
2. ਆਪਣੀ IDE ਵਿੱਚ ਪ੍ਰੋਜੈਕਟ ਖੋਲ੍ਹੋ  
3. ਜਾਵੋ [ਕਦਮ 2: Azure AI Foundry ਪ੍ਰੋਵਿਜ਼ਨ ਕਰੋ](#ਕਦਮ-2-ਪ੍ਰੋਵਿਜ਼ਨ-azure-ai-foundry)  

> **ਮਾਹਿਰ ਸਲਾਹ**: ਜੇ ਤੁਹਾਡੇ ਕੋਲ ਘੱਟ ਸਪੀਕਡ ਮਸ਼ੀਨ ਹੈ ਪਰ ਤੁਸੀਂ ਲੋਕਲ ਤੇ VS Code ਚਾਹੁੰਦੇ ਹੋ, ਤਾਂ GitHub Codespaces ਵਰਤੋ! ਤੁਸੀਂ ਆਪਣੀ ਲੋਕਲ VS Code ਨੂੰ ਕਲਾਉਡ-ਹੋਸਟ ਕੀਤੇ Codespace ਨਾਲ ਜੋੜ ਕੇ ਸਭ ਤੋਂ ਵਧੀਆ ਦੁਨੀਆ ਦੇ ਫਾਇਦੇ ਲੈ ਸਕਦੇ ਹੋ।  

<img src="../../../translated_images/pa/image-2.fc0da29a6e4d2aff.webp" alt="스크린샷 : created local devcontainer instance" width="50%">

## ਕਦਮ 2: Azure AI Foundry ਪ੍ਰੋਵਿਜ਼ਨ ਕਰੋ

ਕੋਰਸ ਦੇ ਏਆਈ ਮਾਡਲਾਂ ਨੂੰ Azure AI Foundry 'ਤੇ ਕੋਡ ਦੇ ਤੌਰ 'ਤੇ ਡਿਪਲੌਇ ਕਰੋ। ਰੀਪੋਜ਼ਿਟਰੀ ਦੇ ਰੂਟ ਤੋਂ:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```
  
`azd` ਵਾਤਾਵਰਣ ਦਾ ਨਾਮ ਤੇ ਇਲਾਕਾ ਪੁੱਛਦਾ ਹੈ, `gpt-4o-mini` ਤੇ `text-embedding-3-small` ਡਿਪਲੌਇਮੈਂਟ ਦੇ ਨਾਲ ਇੱਕ Azure AI Foundry ਖਾਤਾ ਪ੍ਰੋਵਿਜ਼ਨ ਕਰਦਾ ਹੈ, ਅਤੇ ਉਦਾਹਰਨ ਦੇ `.env` ਵਿੱਚ ਐਂਡਪੌਇੰਟ ਲਿਖਦਾ ਹੈ — ਸਾਰੇ **ਕੀਲੈੱਸ** ਪ੍ਰਮਾਣਿਕਤਾ (ਕੋਈ API ਕੁੰਜੀਆਂ ਨਹੀਂ) ਨਾਲ।

> **ਪੂਰਾ ਮਾਰਗਦਰਸ਼ਨ:** ਪੂਰਵ ਸ਼ਰਤਾਂ, ਮੈਨੂਅਲ (ਪੋਰਟਲ) ਵਿਕਲਪ, ਇਲਾਕਾ ਮਦਦ ਅਤੇ ਲਾਗਤ/ਸਾਫ਼-ਸੁਥਰਾ ਨੋਟਾਂ ਲਈ [Azure AI Foundry Setup Guide](getting-started-azure-openai.md) ਵੇਖੋ।  

## ਕਦਮ 3: ਆਪਣਾ ਸੈਟਅਪ ਟੈਸਟ ਕਰੋ

ਜਦੋਂ ਤੁਹਾਡੇ Foundry ਮਾਡਲ ਪ੍ਰੋਵਿਜ਼ਨ ਹੋ ਜਾਣ, ਉਦਾਹਰਨ ਐਪ ਨਾਲ ਸੰਪਰਕ ਟੈਸਟ ਕਰੋ ਜੋ [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) ਵਿੱਚ ਹੈ।

1. ਆਪਣੀ ਵਿਕਾਸ ਮਾਹੌਲ ਵਿੱਚ ਟਰਮੀਨਲ ਖੋਲ੍ਹੋ।  
2. ਉਦਾਹਰਨ ਫੋਲਡਰ ਵਿੱਚ ਜਾਓ:  
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
  
3. ਯਕੀਨੀ ਬਣਾਓ ਕਿ ਤੁਸੀਂ ਸਾਈਨ-ਇਨ ਹੋ (ਕੀਲੈੱਸ ਪ੍ਰਮਾਣਿਕਤਾ ਲਈ ਟੋਕਨ ਚਾਹੀਦਾ ਹੈ):  
   ```bash
   az login
   ```
  
   > ਜੇ ਤੁਸੀਂ `azd up` ਚਲਾਇਆ ਸੀ, ਤਾਂ `.env` ਫਾਈਲ ਵਿੱਚ ਤੁਹਾਡਾ ਐਂਡਪੌਇੰਟ ਪਹਿਲਾਂ ਹੀ ਲਿਖਿਆ ਹੋਇਆ ਸੀ।  
4. ਐਪਲੀਕੇਸ਼ਨ ਚਲਾੋ:  
   ```bash
   mvn clean spring-boot:run
   ```
  
ਤੁਹਾਨੂੰ `gpt-4o-mini` ਮਾਡਲ ਤੋਂ ਇੱਕ ਜਵਾਬ ਦੇਖਣਾ ਚਾਹੀਦਾ ਹੈ।

### ਉਦਾਹਰਨ ਕੋਡ ਸਮਝਣਾ

`examples/basic-chat-azure` ਹੇਠਾਂ ਦੀ ਉਦਾਹਰਨ ਇੱਕ Spring Boot ਐਪ ਹੈ ਜੋ **Spring AI** ਵਰਤ ਕੇ Azure AI Foundry ਨਾਲ ਕੀਲੈੱਸ ਪ੍ਰਮਾਣਿਕਤਾ ਨਾਲ ਜੁਰਦੀ ਹੈ।

**ਇਹ ਕੋਡ ਕੀ ਕਰਦਾ ਹੈ:**  
- Azure AI Foundry ਨਾਲ ਤੁਹਾਡੇ Azure ਸਾਈਨ-ਇਨ (Microsoft Entra ID) ਤੋਂ ਬਿਨਾਂ API ਕੀ ਦੇ ਸਾਥ ਜੋੜਦਾ ਹੈ  
- `gpt-4o-mini` ਮਾਡਲ ਨੂੰ ਪ੍ਰਾਂਪਟ ਭੇਜਦਾ ਹੈ  
- ਏਆਈ ਦਾ ਜਵਾਬ ਪ੍ਰਾਪਤ ਕਰਦਾ ਹੈ ਅਤੇ ਦਿਖਾਉਂਦਾ ਹੈ  
- ਤੁਹਾਡੇ ਸੈਟਅਪ ਦੀ ਸਹੀ ਕਾਰਗੁਜ਼ਾਰੀ ਨੁਹ ਹੋਣੀ ਜਾਂਚਦਾ ਹੈ  

**ਮੁੱਖ ਡਿਪੈਂਡੈਂਸੀ** (`pom.xml` ਵਿੱਚ):  
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```
  
**ਕੰਫਿਗਰੇਸ਼ਨ** (`application.yml`):  
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
  

## ਸਾਰ

ਵਧੀਆ! ਹੁਣ ਤੁਹਾਡੇ ਕੋਲ ਸਭ ਕੁਝ ਸੈਟ ਹੈ:

- Azure AI Foundry ਮਾਡਲਾਂ ਨੂੰ ਬਾਇਸਪ + `azd` ਨਾਲ ਕੋਡ ਦੇ ਤੌਰ 'ਤੇ ਪ੍ਰੋਵਿਜ਼ਨ ਕੀਤਾ  
- ਆਪਣੇ ਜਾਵਾ ਵਿਕਾਸ ਮਾਹੌਲ ਨੂੰ ਚੱਲਾਉਣਾ ਸਿੱਖਿਆ (ਚਾਹੇ Codespaces, ਡੈਵ ਕੰਟੇਨਰ ਜਾਂ ਲੋਕਲ ਹੋਵੇ)  
- Azure AI Foundry ਨਾਲ ਕੀਲੈੱਸ ਪ੍ਰਮਾਣਿਕਤਾ (Microsoft Entra ID) ਨਾਲ ਜੁੜਿਆ — ਕੋਈ API ਕੀ ਨਹੀਂ  
- ਸਧਾਰਣ ਉਦਾਹਰਨ ਨਾਲ ਸਭ ਕੁਝ ਟੈਸਟ ਕੀਤਾ ਜੋ ਤੁਹਾਡੇ ਮਾਡਲ ਨਾਲ ਗੱਲ ਕਰਦਾ ਹੈ  

## ਅਗਲੇ ਕਦਮ

[ਅਧਿਆਇ 3: ਕੋਰ ਜੇਨੇਰੇਟਿਵ ਏਆਈ ਤਕਨੀਕਾਂ](../03-CoreGenerativeAITechniques/README.md)  

## ਟ੍ਰਬਲਸ਼ੂਟਿੰਗ

ਕੋਈ ਸਮੱਸਿਆ ਆ ਰਹੀ ਹੈ? ਇੱਥੇ ਆਮ ਸਮੱਸਿਆਵਾਂ ਅਤੇ ਉਨ੍ਹਾਂ ਦੇ ਹੱਲ ਹਨ:

- **ਪ੍ਰਮਾਣਿਕਤਾ ਫੇਲ ਹੋ ਰਹੀ ਹੈ (401/403)?**  
  - `az login` ਚਲਾਓ — ਪ੍ਰਮਾਣਿਕਤਾ ਕੀਲੈੱਸ ਹੈ, ਇਸ ਲਈ ਤੁਹਾਨੂੰ ਸਾਇਨ-ਇਨ ਕਰਨਾ ਲਾਜ਼ਮੀ ਹੈ  
  - ਯਕੀਨੀ ਬਣਾਓ ਤੁਹਾਡੇ ਖਾਤੇ ਕੋਲ ਸਰੋਤ `Cognitive Services OpenAI User` ਦੀ ਭੂਮਿਕਾ ਹੈ  
  - ਜੇ ਤੁਸੀਂ ਹਾਲ ਹੀ ਵਿੱਚ ਪ੍ਰੋਵਿਜ਼ਨ ਕੀਤਾ ਹੈ, ਤਾਂ ਭੂਮਿਕਾ ਅਸਾਈਨਮੈਂਟ ਲਾਗੂ ਹੋਣ ਲਈ ਕੁਝ ਸਮਾਂ ਉਡੀਕੋ  

- **Maven ਨਹੀਂ ਲੱਭ ਰਿਹਾ?**  
  - ਜੇ ਤੁਸੀਂ ਡੈਵ ਕੰਟੇਨਰਸ/ਕੋਡਸਪੇਸਸ ਵਰਤ ਰਹੇ ਹੋ, ਤਾਂ Maven ਪਹਿਲਾਂ ਤੋਂ ਇੰਸਟਾਲ ਹੋਣਾ ਚਾਹੀਦਾ ਹੈ  
  - ਲੋਕਲ ਸੈਟਅਪ ਲਈ Java 21+ ਅਤੇ Maven 3.9+ ਇੰਸਟਾਲ ਹੋਣ ਚਾਹੀਦੇ ਹਨ  
  - ਇੰਸਟਾਲੇਸ਼ਨ ਦੀ ਪੁਸ਼ਟੀ ਲਈ `mvn --version` ਚਲਾਓ  

- **`azd` ਨਹੀਂ ਮਿਲ ਰਿਹਾ ਜਾਂ ਪ੍ਰੋਵਿਜ਼ਨ ਫੇਲ?**  
  - [Azure Developer CLI](https://aka.ms/azure-dev/install) ਇੰਸਟਾਲ ਕਰਕੇ `azd auth login` ਚਲਾਓ  
  - ਹੇਠਾਂ ਜਿਹੜੇ ਇਲਾਕੇ ਵਿੱਚ `gpt-4o-mini` ਹੈ ਉਹਨਾਂ ਚੋਂ ਕੋਈ ਇਲਾਕਾ ਚੁਣੋ (ਜਿਵੇਂ `eastus2`)  
  - ਵੇਰਵਿਆਂ ਲਈ [Azure AI Foundry setup guide](getting-started-azure-openai.md) ਵੇਖੋ  

- **ਡੈਵ ਕੰਟੇਨਰ ਸ਼ੁਰੂ ਨਹੀਂ ਹੋ ਰਿਹਾ?**  
  - ਯਕੀਨੀ ਬਣਾਓ Docker Desktop ਚੱਲ ਰਿਹਾ ਹੈ (ਲੋਕਲ ਵਿਕਾਸ ਲਈ)  
  - ਕੰਟੇਨਰ ਨੂੰ ਦੁਬਾਰਾ ਬਣਾਉਣ ਦੀ ਕੋਸ਼ਿਸ਼ ਕਰੋ: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"  

- **ਐਪਲੀਕੇਸ਼ਨ ਕੰਪਾਈਲ ਕਰੋ ਸਮੇਂ ਏਰਰ ਆ ਰਹੇ ਹਨ?**  
  - ਯਕੀਨੀ ਬਣਾਓ ਕਿ ਤੁਸੀਂ ਸਹੀ ਡਾਇਰੈਕਟਰੀ `02-SetupDevEnvironment/examples/basic-chat-azure` ਵਿੱਚ ਹੋ  
  - ਸਾਫ ਸਫਾਈ ਅਤੇ ਦੁਬਾਰਾ ਬਣਾਉਣ ਦੀ ਕੋਸ਼ਿਸ਼ ਕਰੋ: `mvn clean compile`  

> **ਮਦਦ ਚਾਹੀਦੀ ਹੈ?**: ਅਜੇ ਵੀ ਸਮੱਸਿਆਵਾਂ? ਰੀਪੋਜ਼ਿਟਰੀ ਵਿੱਚ ਇੱਕ ਇਸ਼ੂ ਖੋਲ੍ਹੋ ਅਸੀਂ ਤੁਹਾਡੀ ਮਦਦ ਕਰਾਂਗੇ।

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ਅਸਵੀਕਾਰੋਪਣ**:
ਇਸ ਦਸਤਾਵੇਜ਼ ਦਾ ਅਨੁਵਾਦ ਏਆਈ ਅਨੁਵਾਦ ਸੇਵਾ [Co-op Translator](https://github.com/Azure/co-op-translator) ਦੀ ਵਰਤੋਂ ਕਰਕੇ ਕੀਤਾ ਗਿਆ ਹੈ। ਜਦੋਂ ਕਿ ਅਸੀਂ ਸਹੀਤਾਵਾਂ ਲਈ ਯਤਨਸ਼ੀਲ ਹਾਂ, ਕਿਰਪਾ ਕਰਕੇ ਧਿਆਨ ਰੱਖੋ ਕਿ ਸਵੈਚਾਲਿਤ ਅਨੁਵਾਦਾਂ ਵਿੱਚ ਗਲਤੀਆਂ ਜਾਂ ਅਸਮੱਤਿਆਵਾਂ ਹੋ ਸਕਦੀਆਂ ਹਨ। ਮੂਲ ਦਸਤਾਵੇਜ਼ ਆਪਣੀ ਮੂਲ ਭਾਸ਼ਾ ਵਿੱਚ ਅਧਿਕਾਰਕ ਸਰੋਤ ਮੰਨਿਆ ਜਾਣਾ ਚਾਹੀਦਾ ਹੈ। ਜਰੂਰੀ ਜਾਣਕਾਰੀ ਲਈ, ਪੇਸ਼ੇਵਰ ਮਨੁੱਖੀ ਅਨੁਵਾਦ ਦੀ ਸਿਫ਼ਾਰਸ਼ ਕੀਤੀ ਜਾਂਦੀ ਹੈ। ਅਸੀਂ ਇਸ ਅਨੁਵਾਦ ਦੇ ਉਪਯੋਗ ਤੋਂ ਪੈਦਾ ਹੋਣ ਵਾਲੀਆਂ ਕਿਸੇ ਵੀ ਗਲਤਫਹਿਮੀਆਂ ਜਾਂ ਗਲਤ ਵਿਆਖਿਆਵਾਂ ਲਈ ਜਵਾਬਦੇਹ ਨਹੀਂ ਹਾਂ।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->