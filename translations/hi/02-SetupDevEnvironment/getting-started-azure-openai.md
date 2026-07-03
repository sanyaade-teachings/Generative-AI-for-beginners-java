# Azure AI Foundry के लिए विकास पर्यावरण सेटअप करना

> यह गाइड इस कोर्स में Java AI ऐप्स के लिए **Azure AI Foundry** मॉडल सेटअप करता है, **keyless** प्रमाणीकरण (Microsoft Entra ID) का उपयोग करते हुए — कोई API कुंजी प्रबंधित करने की जरूरत नहीं। टूलिंग में नया हैं? [डेवलपमेंट एनवायरमेंट गाइड](./README.md) से शुरू करें।

यह गाइड इस कोर्स में Java AI ऐप्स के लिए **Azure AI Foundry** मॉडल सेटअप करता है। आपके पास दो विकल्प हैं:

- **विकल्प A — `azd` + Bicep के साथ प्रोविजन करें (अनुशंसित):** एक कमांड के साथ Foundry खाता और मॉडल कोड के रूप में तैनात करता है। कोई पोर्टल पर क्लिक करने की जरूरत नहीं।
- **विकल्प B — Azure AI Foundry पोर्टल में मैन्युअल रूप से संसाधन बनाएं।**

दोनों विकल्प **keyless प्रमाणीकरण** (Microsoft Entra ID) का उपयोग करते हैं — कोई API कुंजी कॉपी या लीक नहीं होगी।

## विषय सूची

- [क्या बनता है](#क्या-बनता-है)
- [पूर्व आवश्यकताएं](#पूर्व-आवश्यकताएं)
- [विकल्प A: azd + Bicep के साथ प्रोविजन करें (अनुशंसित)](#option-a-provision-with-azd--bicep-recommended)
- [विकल्प B: मैन्युअल रूप से संसाधन बनाएं](#विकल्प-b-मैन्युअल-रूप-से-संसाधन-बनाएं)
- [अपने पर्यावरण को कॉन्फ़िगर करें](#अपने-पर्यावरण-को-कॉन्फ़िगर-करें)
- [अपने सेटअप का परीक्षण करें](#अपने-सेटअप-का-परीक्षण-करें)
- [अगला क्या है?](#अगला-क्या-है)
- [संसाधन](#संसाधन)
- [अतिरिक्त संसाधन](#अतिरिक्त-संसाधन)

## क्या बनता है

[`infra/`](../../../02-SetupDevEnvironment/infra) में Bicep टेम्प्लेट प्रोविजन करते हैं:

- एक **Azure AI Foundry** खाता (`Microsoft.CognitiveServices/accounts`, प्रकार `AIServices`) एक प्रोजेक्ट के साथ
- एक **चैट** तैनाती — `gpt-4o-mini`
- एक **एम्बेडिंग** तैनाती — `text-embedding-3-small` (बाद के अध्यायों में उपयोग होती है)
- एक **keyless भूमिका असाइनमेंट** (`Cognitive Services OpenAI User`) ताकि आप कुंजी प्रबंधित करने के बजाय `az login` के साथ साइन इन कर सकें

## पूर्व आवश्यकताएं

- एक [Azure सदस्यता](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) और [Maven 3.9+](https://maven.apache.org/download.cgi)

## विकल्प A: azd + Bicep के साथ प्रोविजन करें (अनुशंसित)

`02-SetupDevEnvironment` फ़ोल्डर से:

```bash
cd 02-SetupDevEnvironment

# साइन इन करें (दोनों उपकरण)
azd auth login
az login

# Foundry खाता + मॉडल तैनाती की व्यवस्था करें
azd up
```
  
`azd` आपसे एक **पर्यावरण नाम** (उदाहरण के लिए `genai-java`) और एक **क्षेत्र** पूछेगा। ऐसा क्षेत्र चुनें जहां `gpt-4o-mini` और `text-embedding-3-small` उपलब्ध हों — उदाहरण के लिए `eastus2` या `swedencentral`।

जब प्रोविजनिंग पूरी हो जाती है, तो azd:

1. [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep) में परिभाषित सब कुछ तैनात करता है।
2. एक पोस्टप्रोविजन हुक चलाता है जो [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) में आपके एन्डपॉइंट और तैनाती नाम लिखता है (कोई गुप्त जानकारी नहीं)।

> **टिप:** जब भी परिवर्तनों को लागू करना हो `azd up` पुनः चलाएं। सब कुछ हटाने और लागत रोकने के लिए `azd down` चलाएं।

बनाई गई सेटिंग्स देखने के लिए:

```bash
azd env get-values
```
  
अब [अपने सेटअप का परीक्षण करें](#अपने-सेटअप-का-परीक्षण-करें) पर जाएं।

## विकल्प B: मैन्युअल रूप से संसाधन बनाएं

पोर्टल पसंद है? संसाधन मैन्युअल रूप से बनाएं:

1. [Azure AI Foundry पोर्टल](https://ai.azure.com/) पर जाएं और साइन इन करें।
2. **एक प्रोजेक्ट बनाएं** (यह भी एक AI Foundry संसाधन बनाता है)। इसे `GenAIJava` जैसा नाम दें।
3. अपने प्रोजेक्ट में, **Models + endpoints** खोलें → **Deploy model** → **Deploy base model**।
4. **gpt-4o-mini** तैनात करें (तैनाती नाम `gpt-4o-mini`)। यदि एम्बेडिंग उदाहरण चाहते हैं तो **text-embedding-3-small** भी दोहराएं।
5. **Overview** से, **एन्डपॉइंट** कॉपी करें (उदा. `https://<resource>.openai.azure.com/`)।
6. अपने आप को keyless एक्सेस दें: संसाधन पर, **Access control (IAM)** खोलें → **Add role assignment** → अपनी खाते को **Cognitive Services OpenAI User** असाइन करें।

> **अभी भी समस्या हो रही है?** [Azure AI Foundry डोक्यूमेंटेशन](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects) देखें।

## अपने पर्यावरण को कॉन्फ़िगर करें

**यदि आपने विकल्प A (`azd up`) का उपयोग किया है**, तो आपकी सेटिंग्स फ़ाइल पहले से ही लिखी जा चुकी है — कॉन्फ़िगर करने की जरूरत नहीं। सीधे [अपने सेटअप का परीक्षण करें](#अपने-सेटअप-का-परीक्षण-करें) पर जाएं।

**यदि आपने विकल्प B (मैन्युअल) का उपयोग किया है**, तो उदाहरण का `.env` फ़ाइल स्वयं बनाएं:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```
  
`.env` को अपने एन्डपॉइंट के साथ संपादित करें (कोई कुंजी नहीं — प्रमाणीकरण keyless है):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```
  
> **सुरक्षा नोट:** संग्रहीत करने के लिए कोई API कुंजी नहीं है। आप Microsoft Entra ID के माध्यम से `az login` (स्थानीय) या Azure में प्रबंधित पहचान के साथ प्रमाणीकरण करते हैं। `.env` फ़ाइल केवल गैर-गुप्त सेटिंग्स रखती है और `.gitignore` द्वारा पहले से ही कवर है।

## अपने सेटअप का परीक्षण करें

सुनिश्चित करें कि आप साइन इन हैं ताकि keyless प्रमाणीकरण के लिए टोकन मिल सके, फिर उदाहरण चलाएं:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # यदि आप पहले से साइन इन नहीं हैं
mvn clean spring-boot:run
```
  
आपको `gpt-4o-mini` मॉडल से प्रतिक्रिया दिखनी चाहिए!

> **VS कोड उपयोगकर्ता:** रन करने के लिए `F5` दबाएं। ऐप अपने आप आपकी `.env` लोड करता है।

> **पूर्ण उदाहरण:** विवरण और समस्या निवारण के लिए [Azure AI Foundry के साथ बेसिक चैट उदाहरण](./examples/basic-chat-azure/README.md) देखें।

## अगला क्या है?

**सेटअप पूरा!** अब आपके पास है:  
- Azure AI Foundry जिस पर `gpt-4o-mini` और `text-embedding-3-small` तैनात हैं  
- Keyless प्रमाणीकरण (Microsoft Entra ID) — प्रबंधित करने के लिए कोई कुंजी नहीं  
- स्थानीय `.env` फ़ाइल जिसमें आपका एन्डपॉइंट और तैनाती के नाम हैं  
- Java विकास पर्यावरण उपयोग के लिए तैयार

**आगे बढ़ें** [अध्याय 3: Core Generative AI Techniques](../03-CoreGenerativeAITechniques/README.md) पर जाकर AI ऐप बनाना शुरू करें!

## संसाधन

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Microsoft Entra ID के साथ Keyless प्रमाणीकरण](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry डोक्यूमेंटेशन](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI डोक्यूमेंटेशन](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## अतिरिक्त संसाधन

- [VS कोड डाउनलोड करें](https://code.visualstudio.com/Download)
- [Docker Desktop प्राप्त करें](https://www.docker.com/products/docker-desktop)
- [Dev Container कॉन्फ़िगरेशन](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**अस्वीकरण**:
इस दस्तावेज़ का अनुवाद AI अनुवाद सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) का उपयोग करके किया गया है। जबकि हम सटीकता के लिए प्रयास करते हैं, कृपया ध्यान दें कि स्वचालित अनुवादों में त्रुटियाँ या अशुद्धियाँ हो सकती हैं। मूल दस्तावेज़ अपनी मूल भाषा में ही प्रामाणिक स्रोत माना जाना चाहिए। महत्वपूर्ण जानकारी के लिए, पेशेवर मानव अनुवाद की सिफारिश की जाती है। इस अनुवाद के उपयोग से उत्पन्न किसी भी गलतफहमी या गलत व्याख्या के लिए हम उत्तरदायी नहीं हैं।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->