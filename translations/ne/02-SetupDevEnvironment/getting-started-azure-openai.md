# Azure AI Foundry को लागि विकास वातावरण सेटअप गर्दै

> यस गाइडले यस पाठ्यक्रमका लागि Java AI एपहरूमा प्रयोग हुने **Azure AI Foundry** मोडेलहरू **keyless** प्रमाणीकरण (Microsoft Entra ID) प्रयोग गरी सेटअप गर्दछ — कुनै API कुञ्जीहरू व्यवस्थापन गर्न आवश्यक छैन। नयाँ हुनुहुन्छ? सुरु गर्नुहोस् [विकास वातावरण गाइड](./README.md) बाट।

यस गाइडले यस पाठ्यक्रमका लागि Java AI एपहरूमा प्रयोग हुने **Azure AI Foundry** मोडेलहरू सेटअप गर्दछ। तपाईंसँग दुई विकल्पहरू छन्:

- **विकल्प A — `azd` + Bicep प्रयोग गरी प्रोभिजन गर्नुहोस् (सिफारिस गरिएको):** एक कमाण्डले Foundry खाता र मोडेलहरू कोडको रूपमा तैनाथ गर्छ। कुनै पोर्टल क्लिक गर्न आवश्यक छैन।
- **विकल्प B — Azure AI Foundry पोर्टलमा स्रोतहरू म्यानुअली सिर्जना गर्नुहोस्।**

दुवै विकल्पहरू **keyless प्रमाणीकरण** (Microsoft Entra ID) प्रयोग गर्छन् — कुनै API कुञ्जीहरू कपी वा लिक हुँदैनन्।

## सामग्री तालिका

- [के के सिर्जना हुन्छ](#के-के-सिर्जना-हुन्छ)
- [पूर्वशर्तहरू](#पूर्वशर्तहरू)
- [विकल्प A: azd + Bicep प्रयोग गरी प्रोभिजन (सिफारिस गरिएको)](#option-a-provision-with-azd--bicep-recommended)
- [विकल्प B: स्रोतहरू म्यानुअली सिर्जना गर्नुहोस्](#विकल्प-b-स्रोतहरू-म्यानुअली-सिर्जना-गर्नुहोस्)
- [आफ्नो वातावरण कन्फिगर गर्नुहोस्](#आफ्नो-वातावरण-कन्फिगर-गर्नुहोस्)
- [आफ्नो सेटअप परीक्षण गर्नुहोस्](#आफ्नो-सेटअप-परीक्षण-गर्नुहोस्)
- [अर्को के हो?](#अर्को-के-हो)
- [स्रोतहरू](#स्रोतहरू)
- [थप स्रोतहरू](#थप-स्रोतहरू)

## के के सिर्जना हुन्छ

[`infra/`](../../../02-SetupDevEnvironment/infra) भित्रको Bicep टेम्पलेटहरूले:

- एउटा **Azure AI Foundry** खाता (`Microsoft.CognitiveServices/accounts`, प्रकार `AIServices`) एउटा प्रोजेक्टसँग
- एउटा **च्याट** डिप्लोइमेन्ट — `gpt-4o-mini`
- एउटा **एम्बेडिंग** डिप्लोइमेन्ट — `text-embedding-3-small` (पछि अध्यायमा प्रयोग गरिने)
- एउटा **keyless भूमिका असाइनमेन्ट** (`Cognitive Services OpenAI User`), जसले तपाईंलाई कुञ्जी व्यवस्थापन नगरी `az login` मार्फत साइन इन गर्न अनुमति दिन्छ

## पूर्वशर्तहरू

- एउटा [Azure सदस्यता](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) र [Maven 3.9+](https://maven.apache.org/download.cgi)

## विकल्प A: azd + Bicep प्रयोग गरी प्रोभिजन (सिफारिस गरिएको)

`02-SetupDevEnvironment` फोल्डरबाट:

```bash
cd 02-SetupDevEnvironment

# लग इन गर्नुहोस् (दुवै उपकरणहरू)
azd auth login
az login

# Foundry खाता + मोडेल तैनाथीहरू प्रावधान गर्नुहोस्
azd up
```
  
`azd` ले तपाईंलाई एउटा **वातावरणको नाम** (जस्तै `genai-java`) र एउटा **क्षेत्र** सोध्छ। त्यस्तो क्षेत्र छान्नुहोस् जहाँ `gpt-4o-mini` र `text-embedding-3-small` उपलब्ध छन् — उदाहरणका लागि `eastus2` वा `swedencentral`।

प्रोभिजन सकिएपछि, azd:

1. [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep) मा परिभाषित सबै कुरा डिप्लोइ गर्छ।
2. एउटा पोस्ट-प्रोभिजन हुक चलाउँछ जुन [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) फाईलमा तपाईको अन्तिम बिन्दु र डिप्लोइमेन्ट नामहरू लेख्छ (कुनै गोप्य कुरा बिना)।

> **सुझाव:** परिवर्तन लागू गर्न जुनसुकै बेला `azd up` पुन: चलाउनुहोस्। सबै मेटाउन र खर्च रोक्न `azd down` चलाउनुहोस्।

निर्मित सेटिङहरू हेर्न:

```bash
azd env get-values
```
  
अब [आफ्नो सेटअप परीक्षण गर्नुहोस्](#आफ्नो-सेटअप-परीक्षण-गर्नुहोस्) मा जानुहोस्।

## विकल्प B: स्रोतहरू म्यानुअली सिर्जना गर्नुहोस्

पोर्टल मन पर्छ? स्रोतहरू म्यानुअली सिर्जना गर्नुहोस्:

1. [Azure AI Foundry पोर्टल](https://ai.azure.com/) मा जानुहोस् र साइन इन गर्नुहोस्।
2. एउटा **प्रोजेक्ट सिर्जना गर्नुहोस्** (यसले AI Foundry स्रोत पनि सिर्जना गर्छ)। यसलाई `GenAIJava` जस्तो नाम दिनुहोस्।
3. आफ्नो प्रोजेक्टमा, **Models + endpoints** → **Deploy model** → **Deploy base model** खोल्नुहोस्।
4. **gpt-4o-mini** तैनाथ गर्नुहोस् (डिप्लोइमेन्ट नाम `gpt-4o-mini`)। यदि तपाईंले एम्बेडिंग उदाहरणहरू चाहनुहुन्छ भने **text-embedding-3-small** पनि दोहोर्याउनुहोस्।
5. **Overview** बाट **endpoint** कपी गर्नुहोस् (जस्तै `https://<resource>.openai.azure.com/`)।
6. आफ्नो खातालाई keyless पहुँच दिनुहोस्: स्रोतमा **Access control (IAM)** → **Add role assignment** → तपाईंको खातामा **Cognitive Services OpenAI User** भूमिका दिनुहोस्।

> **अझै समस्या?** हेर्नुहोस् [Azure AI Foundry कागजातहरू](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects)।

## आफ्नो वातावरण कन्फिगर गर्नुहोस्

**यदि तपाईंले विकल्प A (`azd up`) प्रयोग गर्नुभयो भने**, तपाईंको सेटिङ फाईल पहिले नै लेखिएको छ — कन्फिगर गर्न केही छैन। [आफ्नो सेटअप परीक्षण गर्नुहोस्](#आफ्नो-सेटअप-परीक्षण-गर्नुहोस्) मा जानुहोस्।

**यदि तपाईंले विकल्प B (म्यानुअल) प्रयोग गर्नुभयो भने**, उदाहरणको `.env` फाईल आफैं सिर्जना गर्नुहोस्:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```
  
एन्डपोइन्ट सहित `.env` सम्पादन गर्नुहोस् (कुञ्जी चाहिँदैन — प्रमाणीकरण keyless छ):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```
  
> **सुरक्षा सूचना:** कुनै API कुञ्जी भण्डारण गर्न पर्दैन। तपाईं Microsoft Entra ID मार्फत `az login` (स्थानीय रूपमा) वा मेनेज्ड आइडेन्टिटी (Azure मा) द्वारा प्रमाणीकरण गर्नुहुन्छ। `.env` फाईलमा केवल गैर-गोप्य सेटिङहरू मात्र हुन्छन् र `.gitignore` मा राखिन्छ।

## आफ्नो सेटअप परीक्षण गर्नुहोस्

keyless प्रमाणीकरणले टोकन प्राप्त गर्न सकोस् भनेर साइन इन हुनुहोस्, त्यसपछि उदाहरण चलाउनुहोस्:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # यदि तपाईं पहिले नै साइन इन गर्नुभएको छैन भने
mvn clean spring-boot:run
```
  
तपाईंले `gpt-4o-mini` मोडेलबाट प्रतिक्रिया देख्नु पर्नेछ!

> **VS Code प्रयोगकर्ताहरू:** चलाउन `F5` थिच्नुहोस्। एपले तपाईंको `.env` अटोम्याटिक लोड गर्छ।

> **पूर्ण उदाहरण:** विवरण र समस्या समाधानका लागि [Basic Chat with Azure AI Foundry उदाहरण](./examples/basic-chat-azure/README.md) हेर्नुहोस्।

## अर्को के हो?

**सेटअप पूरा भयो!** तपाईंसँग अब छ:  
- Azure AI Foundry `gpt-4o-mini` र `text-embedding-3-small` सहित तैनाथ  
- Keyless प्रमाणीकरण (Microsoft Entra ID) — कुनै कुञ्जी व्यवस्थापन छैन  
- अन्तिम बिन्दु र डिप्लोइमेन्ट नामहरूसहितको स्थानीय `.env`  
- Java विकास वातावरण तयार छ

**आफ्नो AI एप्लिकेसन बनाउन सुरु गर्नका लागि** [अध्याय ३: कोर जनरेटिभ AI प्रविधिहरू](../03-CoreGenerativeAITechniques/README.md) मा जानुहोस्!

## स्रोतहरू

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Microsoft Entra ID सँग keyless प्रमाणीकरण](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry कागजातहरू](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI कागजातहरू](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## थप स्रोतहरू

- [VS Code डाउनलोड गर्नुहोस्](https://code.visualstudio.com/Download)
- [Docker Desktop प्राप्त गर्नुहोस्](https://www.docker.com/products/docker-desktop)
- [डेभ कन्टेनर कन्फिगरेसन](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**अस्वीकरण**:
यो दस्तावेज़ AI अनुवाद सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) प्रयोग गरेर अनुवाद गरिएको हो। हामी सही हुन प्रयास गर्छौं, तर कृपया जानकार हुनुस् कि स्वचालित अनुवादमा त्रुटिहरू वा अशुद्धताहरू हुन सक्छन्। मूल दस्तावेज़ यसको मूल भाषामा आधिकारिक स्रोत मानिनुपर्छ। महत्वपूर्ण जानकारीका लागि व्यावसायिक मानव अनुवाद सिफारिस गरिन्छ। यस अनुवादको प्रयोगबाट उत्पन्न कुनै पनि गलत बुझाइ वा त्रुटिको लागि हामी जिम्मेवार छैनौं।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->