# जनरेटिभ एआईका लागि जाभा विकास वातावरण सेटअप गर्ने

> **छिटो सुरु गर्नुहोस्:** केही मिनेटभित्र Bicep + `azd` सँग कोड भएर आफ्नो एआई मोडेलहरू **Azure AI Foundry** मा प्रोभिजन गर्नुहोस् — हेर्नुहोस् [Azure AI Foundry Setup Guide](getting-started-azure-openai.md)। प्रमाणीकरण **keyless** छ (Microsoft Entra ID), त्यसैले कुनै API कुञ्जीहरूको व्यवस्थापन छैन।

## के तपाईंले सिक्नुहुनेछ

- एआई अनुप्रयोगहरूको लागि जाभा विकास वातावरण सेटअप गर्ने
- आफ्नो मनपर्ने विकास वातावरण (Codespaces संग क्लाउड-प्रथम, स्थानीय Dev Container, वा पूर्ण स्थानीय सेटअप) छनौट र कन्फिगर गर्ने
- Azure AI Foundry मोडेलसँग जडान गरेर आफ्नो सेटअप परीक्षण गर्ने

## सामग्री तालिका

- [के तपाईंले सिक्नुहुनेछ](#के-तपाईंले-सिक्नुहुनेछ)
- [परिचय](#परिचय)
- [चरण १: विकास वातावरण सेटअप गर्ने](#चरण-१-विकास-वातावरण-सेटअप-गर्ने)
  - [विकल्प A: GitHub Codespaces (सिफारिस)](#विकल्प-a-github-codespaces-सिफारिस)
  - [विकल्प B: स्थानीय Dev Container](#विकल्प-b-स्थानीय-dev-container)
  - [विकल्प C: आफ्नो अवस्थित स्थानीय स्थापना प्रयोग गर्ने](#विकल्प-c-आफ्नो-अवस्थित-स्थानीय-स्थापना-प्रयोग-गर्ने)
- [चरण २: Azure AI Foundry प्रोभिजन गर्ने](#चरण-२-azure-ai-foundry-प्रोभिजन-गर्ने)
- [चरण ३: आफ्नो सेटअप परिक्षण गर्ने](#चरण-३-आफ्नो-सेटअप-परिक्षण-गर्ने)
- [समस्या समाधान](#समस्या-समाधान)
- [सारांश](#सारांश)
- [अर्को कदमहरू](#अर्को-कदमहरू)

## परिचय

यस अध्यायले तपाइँलाई विकास वातावरण सेटअप गर्न मार्गदर्शन गर्नेछ। हामी यो कोर्सभरि मोडेलहरूको लागि **Azure AI Foundry** प्रयोग गर्नेछौं। तपाईंले मोडेलहरूलाई Bicep र Azure Developer CLI (`azd`) संग कोडको रूपमा प्रोभिजन गर्नुहुन्छ, र त्यसपछि **keyless प्रमाणीकरण** (Microsoft Entra ID) मार्फत जडान गर्नुहोस् — कुनै API कुञ्जीहरू कपी वा चुहावट हुँदैन।

**कुनै स्थानीय सेटअप आवश्यक छैन!** तपाईं GitHub Codespaces प्रयोग गर्न सक्नुहुन्छ, जसले तपाईंको ब्राउजरमा पूर्ण विकास वातावरण प्रदान गर्छ, र त्यहाँबाट Foundry लाई प्रोभिजन गर्नुहोस्।

हामी यो कोर्सका लागि **Azure AI Foundry** प्रयोग गर्छौं किनभने:
- **कोडको रूपमा प्रोभिजन गरिएको** — एउटा `azd up` ले खाता र मोडेल डिप्लोयमेन्टहरू तैनाथ गर्छ
- **Keyless** — तपाईंको Azure साइन-इन वा प्रबन्धित पहिचानसँग प्रमाणीकरण गर्नुहोस्
- **उत्पादन-तयार** — सोही कोड स्थानीय र Azure दुवैमा चल्छ
- **लचिलो** — डिप्लोयमेन्ट नाम परिवर्तन गरेर मोडेलहरू स्वैप गर्न सकिन्छ, तपाईँको कोड होइन

> **ध्यान दिनुहोस्**: Azure AI Foundry डिप्लोयमेन्टहरू टोकन अनुसार बिल गरिन्छ (pay-as-you-go)। provisioning, क्षेत्र, र लागत विवरणहरूका लागि [Azure AI Foundry setup guide](getting-started-azure-openai.md) हेर्नुहोस्।

## चरण १: विकास वातावरण सेटअप गर्ने

<a name="quick-start-cloud"></a>

हामीले Generative AI for Java कोर्सका लागि आवश्यक सबै उपकरणहरू सुनिश्चित गर्दै सेटअप समय कम गर्न पूर्व कन्फिगर गरिएको विकास कन्टेनर सिर्जना गरेका छौं। तपाईं आफ्नो मनपर्ने विकास विधि छनौट गर्नुहोस्:

### वातावरण सेटअप विकल्पहरू:

#### विकल्प A: GitHub Codespaces (सिफारिस)

**२ मिनेटमा कोडिङ सुरु गर्नुहोस् - कुनै स्थानीय सेटअप आवश्यक छैन!**

1. यो रिपोजिटोरीलाई आफ्नो GitHub खातामा Fork गर्नुहोस्
   > **ध्यान दिनुहोस्**: यदि तपाईँले आधारभूत कन्फिग सम्पादन गर्न चाहनुहुन्छ भने कृपया [Dev Container Configuration](../../../.devcontainer/devcontainer.json) हेर्नुहोस्
2. **Code** → **Codespaces** ट्याब → **...** → **New with options...** क्लिक गर्नुहोस्
3. पूर्वनिर्धारित सेटिङहरू प्रयोग गर्नुहोस् – यसले यो कोर्सका लागि तयार गरिएको **Generative AI Java Development Environment** कस्टम devcontainer कन्फिग चयन गर्नेछ
4. **Create codespace** मा क्लिक गर्नुहोस्
5. वातावरण तयार हुन करीव २ मिनेट पर्खनुहोस्
6. [चरण २: Azure AI Foundry प्रोभिजन गर्ने](#चरण-२-azure-ai-foundry-प्रोभिजन-गर्ने) मा अगाडि बढ्नुहोस्

<img src="../../../translated_images/ne/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: Codespaces submenu" width="50%">

<img src="../../../translated_images/ne/image.833552b62eee7766.webp" alt="Screenshot: New with options" width="50%">

<img src="../../../translated_images/ne/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: Create codespace options" width="50%">

> **Codespaces का फाइदाहरू**:
> - कुनै स्थानीय स्थापना आवश्यक छैन
> - कुनै पनि ब्राउजर भएको उपकरणमा काम गर्छ
> - सबै उपकरण र निर्भरता पूर्व-व्यवस्थित
> - व्यक्तिगत खाताहरूका लागि महिनामा ६० घण्टा निःशुल्क
> - सबै विद्यार्थीहरूको लागि सुसंगत वातावरण

#### विकल्प B: स्थानीय Dev Container

**डोकर सहित स्थानीय विकास मन पराउने विकासकर्ताहरूका लागि**

1. यो रिपोजिटोरीलाई तपाईंको स्थानीय मेसिनमा Fork र Clone गर्नुहोस्
   > **ध्यान दिनुहोस्**: यदि तपाईँले आधारभूत कन्फिग सम्पादन गर्न चाहनुहुन्छ भने कृपया [Dev Container Configuration](../../../.devcontainer/devcontainer.json) हेर्नुहोस्
2. [Docker Desktop](https://www.docker.com/products/docker-desktop/) र [VS Code](https://code.visualstudio.com/) स्थापना गर्नुहोस्
3. VS Code मा [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) इन्स्टल गर्नुहोस्
4. VS Code मा रिपोजिटोरी फोल्डर खोल्नुहोस्
5. जब संकेत दिइन्छ, **Reopen in Container** मा क्लिक गर्नुहोस् (वा `Ctrl+Shift+P` → "Dev Containers: Reopen in Container" प्रयोग गर्नुहोस्)
6. कन्टेनर बिल्ड र सुरु हुन पर्खनुहोस्
7. [चरण २: Azure AI Foundry प्रोभिजन गर्ने](#चरण-२-azure-ai-foundry-प्रोभिजन-गर्ने) मा अगाडि बढ्नुहोस्

<img src="../../../translated_images/ne/devcontainer.21126c9d6de64494.webp" alt="Screenshot: Dev container setup" width="50%">

<img src="../../../translated_images/ne/image-3.bf93d533bbc84268.webp" alt="Screenshot: Dev container build complete" width="50%">

#### विकल्प C: आफ्नो अवस्थित स्थानीय स्थापना प्रयोग गर्ने

**पहिले देखि नै जाभा वातावरण भएका विकासकर्ताहरूका लागि**

पूर्वआवश्यकताहरू:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html) 
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) वा आफ्नो मनपर्ने IDE

कदमहरू:
1. यो रिपोजिटोरीलाई आफ्नो स्थानीय मेसिनमा क्लोन गर्नुहोस्
2. आफ्नो IDE मा प्रोजेक्ट खोल्नुहोस्
3. [चरण २: Azure AI Foundry प्रोभिजन गर्ने](#चरण-२-azure-ai-foundry-प्रोभिजन-गर्ने) मा अगाडि बढ्नुहोस्

> **प्रो टिप:** यदि तपाईंसँग कम स्पेकको मेसिन छ तर VS Code स्थानीय प्रयोग गर्न चाहनुहुन्छ भने GitHub Codespaces प्रयोग गर्नुहोस्! तपाईंले आफ्नो स्थानीय VS Code लाई क्लाउड होस्ट गरिएको Codespace सँग जडान गर्न सक्नुहुन्छ।

<img src="../../../translated_images/ne/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: created local devcontainer instance" width="50%">

## चरण २: Azure AI Foundry प्रोभिजन गर्ने

कोर्सको एआई मोडेलहरू Azure AI Foundry मा कोडको रूपमा डिप्लोय गर्नुहोस्। रिपोजिटोरी रुटबाट:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```
  
`azd` ले वातावरण नाम र क्षेत्रको लागि सोध्छ, `gpt-4o-mini` र `text-embedding-3-small` डिप्लोयमेन्टहरूसहित Azure AI Foundry खाता प्रोभिजन गर्छ, र उदाहरणको `.env` फाइलमा अन्तबिन्दु लेख्छ — सबै **keyless** प्रमाणीकरणसहित (कुनै API कुञ्जीहरू छैनन्)।

> **पूरा प्रक्रिया:** आवश्यकताहरू, म्यानुअल (पोर्टल) विकल्प, क्षेत्र निर्देशन, र लागत/सफाई नोटहरूका लागि [Azure AI Foundry Setup Guide](getting-started-azure-openai.md) हेर्नुहोस्।

## चरण ३: आफ्नो सेटअप परिक्षण गर्ने

Foundry मोडेलहरू प्रोभिजन भएपछि, [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) मा रहेको उदाहरण अनुप्रयोगसँग जडान परीक्षण गर्नुहोस्।

1. आफ्नो विकास वातावरणमा टर्मिनल खोल्नुहोस्।
2. उदाहरण फोल्डरमा जानुहोस्:
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
  
3. सुनिश्चित गर्नुहोस् कि तपाईं साइन इन हुनु भएको छ (keyless auth मा टोकन आवश्यक छ):
   ```bash
   az login
   ```
  
   > यदि तपाईंले `azd up` चलाउनुभयो भने, `.env` फाइल तपाईँको अन्तबिन्दुसहित पहिले नै लेखिएको हुन्छ।
4. अनुप्रयोग चलाउनुहोस्:
   ```bash
   mvn clean spring-boot:run
   ```
  
तपाईंले `gpt-4o-mini` मोडेलबाट प्रतिक्रिया देख्न सक्नुहुन्छ।

### उदाहरण कोड बुझ्नुहोस्

`examples/basic-chat-azure` अन्तर्गत रहेको उदाहरण Spring Boot एप हो जसले **Spring AI** प्रयोग गरी Azure AI Foundry सँग keyless प्रमाणीकरणका साथ जडान गर्दछ।

**यो कोडले के गर्छ:**
- तपाईंको Azure साइन-इन (Microsoft Entra ID) प्रयोग गरी Azure AI Foundry सँग जडान गर्छ — कुनै API कुञ्जी छैन
- `gpt-4o-mini` मोडेललाई प्रम्प्ट पठाउँछ
- AI को प्रतिक्रिया प्राप्त गरेर प्रदर्शन गर्छ
- तपाईंको सेटअप ठीकसँग काम गरिरहेको छ कि छैन जाँच गर्छ

**प्रमुख निर्भरता** (`pom.xml` मा):
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```
  
**कन्फिगरेसन** (`application.yml` मा):
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


## सारांश

शानदार! अब तपाईंले सबै कुरा सेटअप गरिसक्नुभएको छ:

- Bicep + `azd` सँग Azure AI Foundry मोडेलहरू कोडको रूपमा प्रोभिजन गरियो
- तपाईंको जाभा विकास वातावरण चलिरहेको छ (Codespaces, dev container वा स्थानीय कुनै पनि)
- Azure AI Foundry सँग keyless प्रमाणीकरण (Microsoft Entra ID) द्वारा जडान भयो — कुनै API कुञ्जी छैन
- तपाईंको मोडेलसँग कुरा गर्ने सरल उदाहरणबाट सबै ठीक काम गर्छ भनेर जाँच गरियो

## अर्को कदमहरू

[अध्याय ३: कोर जनरेटिभ एआई प्रविधिहरू](../03-CoreGenerativeAITechniques/README.md)

## समस्या समाधान

समस्या छ? यहाँ सामान्य समस्याहरू र समाधानहरू छन्:

- **प्रमाणीकरण असफल (401/403)?**  
  - `az login` चलाउनुहोस् — प्रमाणीकरण keyless छ, त्यसैले तपाईंले साइन इन गर्नुपर्छ  
  - तपाईंको खातामा स्रोतमा **Cognitive Services OpenAI User** भूमिका छ कि छैन जाँच गर्नुहोस्  
  - यदि तपाईंले अहिल्यै प्रोभिजन गर्नुभयो भने, भूमिका असाइनमेंट फैलिन एक मिनेट पर्खनुहोस्  

- **Maven फेला परेन?**  
  - dev containers/Codespaces प्रयोग गर्दा Maven पूर्व-स्थापित हुन्छ  
  - स्थानीय सेटअपका लागि Java 21+ र Maven 3.9+ स्थापना छ कि छैन सुनिश्चित गर्नुहोस्  
  - स्थापना जाँच्न `mvn --version` प्रयोग गर्नुहोस्  

- **`azd` फेला परेन वा प्रोभिजन असफल?**  
  - [Azure Developer CLI](https://aka.ms/azure-dev/install) स्थापना गरी `azd auth login` चलाउनुहोस्  
  - `gpt-4o-mini` उपलब्ध क्षेत्र चुन्नुहोस् (उदाहरण: `eastus2`)  
  - विवरणका लागि [Azure AI Foundry setup guide](getting-started-azure-openai.md) हेर्नुहोस्  

- **Dev container सुरु हुँदैन?**  
  - Docker Desktop चलिरहेको छ कि छैन सुनिश्चित गर्नुहोस् (स्थानीय विकासको लागि)  
  - कन्टेनर पुनःनिर्माण प्रयास गर्नुहोस्: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"  

- **अनुप्रयोग संकलन त्रुटिहरू?**  
  - सुनिश्चित गर्नुहोस् कि तपाईँ सही निर्देशिकामा हुनुहुन्छ: `02-SetupDevEnvironment/examples/basic-chat-azure`  
  - सफा गरेर पुन: संकलन प्रयास गर्नुहोस्: `mvn clean compile`  

> **मदत चाहिन्छ?**: अझै समस्या छ? रिपोजिटोरीमा समस्या खोल्नुहोस्, हामी तपाईंलाई सहयोग गर्नेछौं।

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**अस्वीकरण**:
यो दस्तावेज़ AI अनुवाद सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) प्रयोग गरेर अनुवाद गरिएको हो। हामी सही हुन प्रयास गर्छौं, तर कृपया जानकार हुनुस् कि स्वचालित अनुवादमा त्रुटिहरू वा अशुद्धताहरू हुन सक्छन्। मूल दस्तावेज़ यसको मूल भाषामा आधिकारिक स्रोत मानिनुपर्छ। महत्वपूर्ण जानकारीका लागि व्यावसायिक मानव अनुवाद सिफारिस गरिन्छ। यस अनुवादको प्रयोगबाट उत्पन्न कुनै पनि गलत बुझाइ वा त्रुटिको लागि हामी जिम्मेवार छैनौं।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->