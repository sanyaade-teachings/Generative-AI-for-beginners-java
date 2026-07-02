# Azure AI Foundry के साथ बेसिक चैट - एंड-टू-एंड उदाहरण

यह उदाहरण एक सरल Spring Boot एप्लिकेशन है जो **keyless authentication** (Microsoft Entra ID) का उपयोग करके **Azure AI Foundry** मॉडल से जुड़ता है और आपके सेटअप का परीक्षण करता है। यह Spring AI के `ChatClient` का उपयोग करता है।

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

इस उदाहरण को चलाने से पहले, सुनिश्चित करें कि आपके पास:

- एक Azure AI Foundry संसाधन है जिसमें `gpt-4o-mini` डिप्लॉयमेंट हो — इसे `azd up` के साथ या मैन्युअली [Azure AI Foundry setup guide](../../getting-started-azure-openai.md) के माध्यम से प्रोविजन करें
- उस संसाधन पर **Cognitive Services OpenAI User** भूमिका (यह रोल Bicep टेम्पलेट्स आपके लिए असाइन करते हैं)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), जिसमें आप `az login` के साथ साइन इन हैं
- Java 21+ और Maven 3.9+

> **कोई API कुंजी आवश्यक नहीं** — प्रमाणन Microsoft Entra ID के माध्यम से keyless है।

## Quick Start

```bash
# 1. प्रोजेक्ट पर जाएं
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. साइन इन करें ताकि कीलेस ऑथ को टोकन मिल सके
az login

# 3. एंडपॉइंट कॉन्फ़िगर करें
#    - यदि आपने `azd up` चलाया है, तो .env आपके लिए लिखा गया था (इसे छोड़ें)।
#    - अन्यथा टेम्पलेट कॉपी करें और AZURE_OPENAI_ENDPOINT सेट करें:
cp .env.example .env

# 4. एप्लिकेशन चलाएं
mvn spring-boot:run
```

## How Authentication Works

यह उदाहरण **Microsoft Entra ID** के साथ प्रमाणित होता है — कोई API कुंजी नहीं है।

जब केवल `spring.ai.azure.openai.endpoint` सेट होता है (और कोई api-key नहीं होता), तो Spring AI Azure OpenAI क्लाइंट को [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential) के साथ बनाता है। यह क्रेडेंशियल स्वचालित रूप से आपके स्थानीय `az login` सत्र से टोकन ढूंढता है, या Azure में चलने पर मैनेज्ड आइडेंटिटी से टोकन प्राप्त करता है — इसलिए एक ही कोड बिना किसी बदलाव के दोनों जगह काम करता है।

## Running the Application

### Using Maven

```bash
mvn spring-boot:run
```

### Using VS Code

1. प्रोजेक्ट को VS Code में खोलें
2. `F5` दबाएं या "Run and Debug" पैनल का उपयोग करें
3. "Spring Boot-BasicChatApplication" कॉन्फ़िगरेशन चुनें

> **नोट**: VS Code कॉन्फ़िगरेशन स्वचालित रूप से आपकी .env फ़ाइल को लोड करता है

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
| `AZURE_OPENAI_ENDPOINT` | Foundry (Azure OpenAI) endpoint URL | हाँ | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | चैट मॉडल डिप्लॉयमेंट नाम | नहीं | `gpt-4o-mini` (डिफ़ॉल्ट) |

> कोई **API कुंजी** वेरिएबल नहीं है — प्रमाणन keyless है (Microsoft Entra ID के माध्यम से `az login`)।

### Spring Configuration

`application.yml` फ़ाइल कॉन्फ़िगर करती है:
- **Endpoint**: `${AZURE_OPENAI_ENDPOINT}` - पर्यावरण वेरिएबल से
- **Deployment**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - पर्यावरण वेरिएबल से, फॉलबैक के साथ
- **Auth**: keyless — कोई `api-key` सेट नहीं है, इसलिए Spring AI `DefaultAzureCredential` का उपयोग करता है
- **Temperature**: `0.7` - रचनात्मकता नियंत्रित करता है (0.0 = निश्चित, 1.0 = रचनात्मक)
- **Max Tokens**: `500` - अधिकतम प्रतिक्रिया लंबाई

## Troubleshooting

### Common Issues

<details>
<summary><strong>त्रुटि: 401 / "PermissionDenied" / टोकन त्रुटियाँ</strong></summary>

- `az login` चलाएं — keyless प्रमाणन के लिए सक्रिय साइन-इन आवश्यक है टोकन प्राप्त करने के लिए
- सुनिश्चित करें कि आपके खाते को संसाधन पर **Cognitive Services OpenAI User** भूमिका मिली हुई है
- यदि आपने अभी रोल असाइन किया है, तो इसके लागू होने के लिए कुछ समय प्रतीक्षा करें
- पुष्टि करें कि आप सही टेनेंट / सब्सक्रिप्शन में हैं (`az account show`)
</details>

<details>
<summary><strong>त्रुटि: "The endpoint is not valid" / कनेक्शन त्रुटियाँ</strong></summary>

- सुनिश्चित करें कि `AZURE_OPENAI_ENDPOINT` पूर्ण बेस URL है (जैसे, `https://your-resource.openai.azure.com/`)
- ट्रेलिंग स्लैश की संगति जांचें
- पुष्टि करें कि endpoint आपके प्रोविजन किए गए संसाधन से मेल खाता है (`azd env get-values`)
</details>

<details>
<summary><strong>त्रुटि: "The deployment was not found"</strong></summary>

- जांचें कि `AZURE_OPENAI_DEPLOYMENT` Azure में किसी डिप्लॉयमेंट नाम से मेल खाता है
- जांचें कि मॉडल सफलतापूर्वक डिप्लॉय किया गया और सक्रिय है
- डिफ़ॉल्ट डिप्लॉयमेंट नाम `gpt-4o-mini` है
</details>

<details>
<summary><strong>VS Code: पर्यावरण वेरिएबल लोड नहीं हो रहे हैं</strong></summary>

- सुनिश्चित करें कि आपकी `.env` फ़ाइल प्रोजेक्ट रूट डायरेक्टरी में है (`pom.xml` के समान स्तर पर)
- VS Code के एकीकृत टर्मिनल में `mvn spring-boot:run` चलाने का प्रयास करें
- जांचें कि VS Code Java एक्सटेंशन सही ढंग से इंस्टॉल है
</details>

### Debug Mode

विस्तृत लॉगिंग सक्षम करने के लिए, `application.yml` में इन लाइनों को अनकमेंट करें:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Next Steps

**सेटअप पूरा!** अपनी सीख जारी रखें:

[Chapter 3: Core Generative AI Techniques](../../../03-CoreGenerativeAITechniques/README.md)

## Resources

- [Spring AI Azure OpenAI Documentation](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Microsoft Entra ID के साथ Keyless ऑथेंटिकेशन](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry पोर्टल](https://ai.azure.com/)
- [Azure AI Foundry डाक्युमेंटेशन](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**अस्वीकरण**:
इस दस्तावेज़ का अनुवाद AI अनुवाद सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) का उपयोग करके किया गया है। जबकि हम सटीकता के लिए प्रयास करते हैं, कृपया ध्यान दें कि स्वचालित अनुवादों में त्रुटियाँ या अशुद्धियाँ हो सकती हैं। मूल दस्तावेज़ अपनी मूल भाषा में ही प्रामाणिक स्रोत माना जाना चाहिए। महत्वपूर्ण जानकारी के लिए, पेशेवर मानव अनुवाद की सिफारिश की जाती है। इस अनुवाद के उपयोग से उत्पन्न किसी भी गलतफहमी या गलत व्याख्या के लिए हम उत्तरदायी नहीं हैं।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->