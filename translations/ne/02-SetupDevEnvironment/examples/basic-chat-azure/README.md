# Azure AI Foundry सँग आधारभूत कुराकानी - अन्तदेखि-अन्त उदाहरण

यस उदाहरणले एउटा साधारण Spring Boot अनुप्रयोग हो जुन **Azure AI Foundry** मोडेलसँग **keyless authentication** (Microsoft Entra ID) प्रयोग गरेर जडान हुन्छ र तपाईंको सेटअप जाँच गर्दछ। यसले Spring AI को `ChatClient` प्रयोग गर्दछ।

## सामग्री तालिका

- [पूर्व आवश्यकताहरू](#पूर्व-आवश्यकताहरू)
- [छिटो सुरुवात](#छिटो-सुरुवात)
- [प्रमाणीकरण कसरी काम गर्छ](#प्रमाणीकरण-कसरी-काम-गर्छ)
- [अनुप्रयोग चलाउने तरिका](#अनुप्रयोग-चलाउने-तरिका)
  - [Maven प्रयोग गर्दै](#maven-प्रयोग-गर्दै)
  - [VS Code प्रयोग गर्दै](#vs-code-प्रयोग-गर्दै)
  - [अपेक्षित नतिजा](#अपेक्षित-नतिजा)
- [कन्फिगरेसन सन्दर्भ](#कन्फिगरेसन-सन्दर्भ)
  - [पर्यावरण चरहरू](#वातावरण-चरहरू)
  - [Spring कन्फिगरेसन](#spring-कन्फिगरेसन)
- [समस्या समाधान](#समस्या-समाधान)
  - [सार्वजनिक समस्याहरू](#सार्वजनिक-समस्याहरू)
  - [डिबग मोड](#डिबग-मोड)
- [अर्को चरणहरू](#अर्को-चरणहरू)
- [संसाधनहरू](#स्रोतहरू)

## पूर्व आवश्यकताहरू

यस उदाहरण चलाउनुअघि सुनिश्चित गर्नुहोस् कि तपाईंसँग:

- एउटा Azure AI Foundry स्रोत जसमा `gpt-4o-mini` डिप्लोयमेन्ट छ — यसलाई `azd up` मार्फत वा म्यानुअली [Azure AI Foundry सेटअप गाइड](../../getting-started-azure-openai.md) बाट प्रावधान गर्नुहोस्
- उक्त स्रोतमा **Cognitive Services OpenAI User** भूमिका (Bicep टेम्प्लेटहरूले यो तपाईंका लागि सेट गर्छन्)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), जुन `az login` सँग साइन इन गरिएको छ
- Java 21+ र Maven 3.9+

> **कुनै API कुञ्जी आवश्यक छैन** — प्रमाणीकरण Microsoft Entra ID मार्फत keyless हुन्छ।

## छिटो सुरुवात

```bash
# 1. परियोजनामा जानुहोस्
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. साइन इन गर्नुहोस् ताकि keyless auth ले टोकन प्राप्त गर्न सकोस्
az login

# 3. एन्डपोइन्ट कन्फिगर गर्नुहोस्
#    - यदि तपाईंले `azd up` चलाउनुभएको छ भने, .env तपाईंको लागि लेखिएको थियो (यो भाग छोड्नुहोस्)।
#    - अन्यथा टेम्पलेट कपी गरेर AZURE_OPENAI_ENDPOINT सेट गर्नुहोस्:
cp .env.example .env

# 4. अनुप्रयोग चलाउनुहोस्
mvn spring-boot:run
```

## प्रमाणीकरण कसरी काम गर्छ

यस उदाहरणले **Microsoft Entra ID** सँग प्रमाणीकरण गर्छ — कुनै API कुञ्जी छैन।

जब केवल `spring.ai.azure.openai.endpoint` सेट हुन्छ (र api-key हुँदैन), Spring AI ले Azure OpenAI क्लाइन्ट [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential) सँग बनाउँछ। त्यो प्रमाणीकरणले तपाईंको स्थानीय `az login` सेसनबाट वा Azure मा चलाउँदा व्यवस्थापित पहिचानबाट टोकन पत्ता लगाउँछ — त्यसैले एउटै कोड दुवै ठाउँमा बिना परिवर्तन काम गर्छ।

## अनुप्रयोग चलाउने तरिका

### Maven प्रयोग गर्दै

```bash
mvn spring-boot:run
```

### VS Code प्रयोग गर्दै

1. VS Code मा परियोजना खोल्नुहोस्
2. `F5` थिच्नुहोस् वा "Run and Debug" प्यानल प्रयोग गर्नुहोस्
3. "Spring Boot-BasicChatApplication" कन्फिगरेसन चयन गर्नुहोस्

> **सूचना**: VS Code कन्फिगरेसनले तपाईंको .env फाइल स्वतः लोड गर्दछ

### अपेक्षित नतिजा

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

## कन्फिगरेसन सन्दर्भ

### वातावरण चरहरू

| चर | वर्णन | आवश्यक | उदाहरण |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | Foundry (Azure OpenAI) अन्त बिन्दु URL | हो | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | च्याट मोडेल डिप्लोयमेन्ट नाम | होइन | `gpt-4o-mini` (पूर्वनिर्धारित) |

> **कुनै** API कुञ्जी चर छैन — प्रमाणीकरण keyless (Microsoft Entra ID मार्फत `az login`) हो।

### Spring कन्फिगरेसन

`application.yml` फाइलले निम्न कन्फिगर गर्छ:
- **अन्त बिन्दु**: `${AZURE_OPENAI_ENDPOINT}` - वातावरण चरबाट
- **डिप्लोयमेन्ट**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - वातावरण चरबाट र फallback सहित
- **प्रमाणीकरण**: keyless — कुनै `api-key` सेट गरिएको छैन, त्यसैले Spring AI ले `DefaultAzureCredential` प्रयोग गर्छ
- **टेम्परेचर**: `0.7` - सिर्जनात्मकता नियन्त्रित गर्छ (0.0 = निश्चित, 1.0 = सिर्जनात्मक)
- **अधिकतम टोकन**: `500` - अधिकतम प्रतिक्रिया लम्बाइ

## समस्या समाधान

### सार्वजनिक समस्याहरू

<details>
<summary><strong>त्रुटि: 401 / "PermissionDenied" / टोकन त्रुटिहरू</strong></summary>

- `az login` चलाउनुहोस् — keyless प्रमाणीकरणले सक्रिय साइन-इन आवश्यक छ टोकन प्राप्त गर्न
- तपाईंको खातामा स्रोतमा **Cognitive Services OpenAI User** भूमिका हो कि छैन जाँच गर्नुहोस्
- यदि तपाईंले भर्खर भूमिका दिनुभयो भने केहि समय कुर्नुहोस् त्यसको प्रसारणको लागि
- सुनिश्चित गर्नुहोस् कि तपाईं सहि टेनेन्ट/सब्सक्रिप्शनमा हुनुहुन्छ (`az account show`)
</details>

<details>
<summary><strong>त्रुटि: "अन्त बिन्दु वैध छैन" / जडान त्रुटिहरू</strong></summary>

- सुनिश्चित गर्नुहोस् `AZURE_OPENAI_ENDPOINT` पूर्ण आधार URL हो (जस्तै, `https://your-resource.openai.azure.com/`)
- समाप्ति स्ल्यासको स्थिरता जाँच गर्नुहोस्
- अन्त बिन्दु तपाईंको प्रावधान गरिएको स्रोतसँग मेल खान्छ कि छैन जाँच गर्नुहोस् (`azd env get-values`)
</details>

<details>
<summary><strong>त्रुटि: "डिप्लोयमेन्ट फेला परेन"</strong></summary>

- सुनिश्चित गर्नुहोस् `AZURE_OPENAI_DEPLOYMENT` Azure मा कुनै डिप्लोयमेन्ट नामसँग मेल खान्छ
- मोडेल सफलतापूर्वक डिप्लोय गरिएको र सक्रिय छ कि छैन जाँच गर्नुहोस्
- पूर्वनिर्धारित डिप्लोयमेन्ट नाम `gpt-4o-mini` हो
</details>

<details>
<summary><strong>VS Code: वातावरण चरहरू लोड नहुने समस्या</strong></summary>

- सुनिश्चित गर्नुहोस् `.env` फाइल परियोजनाको मुख्य निर्देशिकामा छ (pom.xml सँग एउटै तहमा)
- VS Code को एकीकृत टर्मिनलमा `mvn spring-boot:run` चलाएर प्रयास गर्नुहोस्
- VS Code Java एक्सटेन्सन ठीकसँग इन्स्टल छ कि छैन जाँच गर्नुहोस्
</details>

### डिबग मोड

विस्तृत लगिङ्ग सक्षम गर्न, `application.yml` मा यी लाइनहरू अनकमेण्ट गर्नुहोस्:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## अर्को चरणहरू

**सेटअप पूरा भयो!** तपाईंको सिकाइ यात्रालाई जारी राख्नुहोस्:

[अध्याय ३: मुख्य सृजनात्मक AI प्रविधिहरू](../../../03-CoreGenerativeAITechniques/README.md)

## स्रोतहरू

- [Spring AI Azure OpenAI डकुमेन्टेसन](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Microsoft Entra ID सँग keyless प्रमाणीकरण](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry पोर्टल](https://ai.azure.com/)
- [Azure AI Foundry डकुमेन्टेसन](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**अस्वीकरण**:
यो दस्तावेज़ AI अनुवाद सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) प्रयोग गरेर अनुवाद गरिएको हो। हामी सही हुन प्रयास गर्छौं, तर कृपया जानकार हुनुस् कि स्वचालित अनुवादमा त्रुटिहरू वा अशुद्धताहरू हुन सक्छन्। मूल दस्तावेज़ यसको मूल भाषामा आधिकारिक स्रोत मानिनुपर्छ। महत्वपूर्ण जानकारीका लागि व्यावसायिक मानव अनुवाद सिफारिस गरिन्छ। यस अनुवादको प्रयोगबाट उत्पन्न कुनै पनि गलत बुझाइ वा त्रुटिको लागि हामी जिम्मेवार छैनौं।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->