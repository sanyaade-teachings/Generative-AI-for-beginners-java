# Java के लिए जनरेटिव AI के विकास पर्यावरण की स्थापना

> **त्वरित शुरूआत:** अपने AI मॉडल को कुछ ही मिनटों में Bicep + `azd` के साथ **Azure AI Foundry** पर कोड के रूप में प्रदत्त करें — देखें [Azure AI Foundry सेटअप गाइड](getting-started-azure-openai.md)। प्रमाणीकरण **कीलेस** (Microsoft Entra ID) है, इसलिए कोई API कुंजी प्रबंधित करने की आवश्यकता नहीं है।

## आप क्या सीखेंगे

- AI अनुप्रयोगों के लिए Java विकास पर्यावरण स्थापित करना
- अपनी पसंदीदा विकास पर्यावरण का चयन और कॉन्फ़िगर करना (Codespaces के साथ क्लाउड-प्रथम, स्थानीय डिव कंटेनर, या पूर्ण स्थानीय सेटअप)
- Azure AI Foundry मॉडल से कनेक्ट करके अपने सेटअप का परीक्षण करना

## सामग्री सूची

- [आप क्या सीखेंगे](#आप-क्या-सीखेंगे)
- [परिचय](#परिचय)
- [चरण 1: अपना विकास पर्यावरण सेट करें](#चरण-1-अपना-विकास-पर्यावरण-सेट-करें)
  - [विकल्प A: GitHub Codespaces (अनुशंसित)](#विकल्प-a-github-codespaces-अनुशंसित)
  - [विकल्प B: स्थानीय डिव कंटेनर](#विकल्प-b-स्थानीय-डिव-कंटेनर)
  - [विकल्प C: अपने मौजूदा स्थानीय इंस्टॉलेशन का उपयोग करें](#विकल्प-c-अपने-मौजूदा-स्थानीय-इंस्टॉलेशन-का-उपयोग-करें)
- [चरण 2: Azure AI Foundry की व्यवस्था करें](#चरण-2-azure-ai-foundry-की-व्यवस्था-करें)
- [चरण 3: अपने सेटअप का परीक्षण करें](#चरण-3-अपने-सेटअप-का-परीक्षण-करें)
- [समस्या निवारण](#समस्या-निवारण)
- [सारांश](#सारांश)
- [अगले कदम](#अगले-कदम)

## परिचय

यह अध्याय आपको एक विकास पर्यावरण स्थापित करने के लिए मार्गदर्शन करेगा। इस पाठ्यक्रम में हम मॉडल के लिए **Azure AI Foundry** का उपयोग करेंगे। आप Bicep और Azure Developer CLI (`azd`) के साथ मॉडल को कोड के रूप में स्थापित करते हैं, फिर **कीलेस प्रमाणीकरण** (Microsoft Entra ID) के साथ कनेक्ट करते हैं — कोई API कुंजियाँ कॉपी या लीक करने की जरूरत नहीं।

**कोई स्थानीय सेटअप आवश्यक नहीं!** आप GitHub Codespaces का उपयोग कर सकते हैं, जो आपके ब्राउज़र में पूर्ण विकास पर्यावरण प्रदान करता है, और वहीं से Foundry को प्रदान करें।

हम इस कोर्स के लिए **Azure AI Foundry** का उपयोग करते हैं क्योंकि यह:
- **कोड के रूप में प्रदान किया गया है** — एक `azd up` खाते और मॉडल तैनातियों को तैनात करता है
- **कीलेस** — अपने Azure साइन-इन या प्रबंधित पहचान से प्रमाणीकरण करें
- **उत्पादन-तैयार** — वही कोड स्थानीय और Azure में चलता है
- **लचीला** — तैनाती नाम बदलकर मॉडल बदलें, आपके कोड को नहीं

> **ध्यान दें**: Azure AI Foundry तैनातियां टोकन के आधार पर बिल की जाती हैं (pay-as-you-go)। प्रावधान, क्षेत्र, और लागत विवरण के लिए [Azure AI Foundry सेटअप गाइड](getting-started-azure-openai.md) देखें।


## चरण 1: अपना विकास पर्यावरण सेट करें

<a name="quick-start-cloud"></a>

हमने इस जनरेटिव AI for Java कोर्स के लिए सभी आवश्यक उपकरण आपको उपलब्ध कराने और सेटअप समय कम करने के लिए एक पूर्व-कॉन्फ़िगर किया हुआ विकास कंटेनर बनाया है। अपनी पसंदीदा विकास विधि चुनें:

### पर्यावरण सेटअप विकल्प:

#### विकल्प A: GitHub Codespaces (अनुशंसित)

**2 मिनट में कोडिंग शुरू करें - कोई स्थानीय सेटअप आवश्यक नहीं!**

1. इस रिपॉजिटरी को अपने GitHub खाते में फोर्क करें  
   > **ध्यान दें**: यदि आप बुनियादी कॉन्फिग संपादित करना चाहते हैं तो कृपया [Dev Container Configuration](../../../.devcontainer/devcontainer.json) देखें
2. क्लिक करें **Code** → **Codespaces** टैब → **...** → **New with options...**
3. डिफ़ॉल्ट का उपयोग करें – यह चुनता है **Dev container configuration**: इस कोर्स के लिए बनाया गया **Generative AI Java Development Environment** कस्टम devcontainer
4. क्लिक करें **Create codespace**
5. पर्यावरण तैयार होने तक लगभग 2 मिनट प्रतीक्षा करें
6. आगे बढ़ें [चरण 2: Azure AI Foundry की व्यवस्था करें](#चरण-2-azure-ai-foundry-की-व्यवस्था-करें)

<img src="../../../translated_images/hi/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: Codespaces submenu" width="50%">

<img src="../../../translated_images/hi/image.833552b62eee7766.webp" alt="Screenshot: New with options" width="50%">

<img src="../../../translated_images/hi/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: Create codespace options" width="50%">


> **Codespaces के लाभ**:
> - कोई स्थानीय इंस्टॉलेशन आवश्यक नहीं
> - किसी भी ब्राउज़र वाले डिवाइस पर काम करता है
> - सभी टूल और डिपेंडेंसी के साथ पूर्व-कॉन्फ़िगर किया हुआ
> - व्यक्तिगत खातों के लिए प्रति माह 60 घंटे मुफ्त
> - सभी सीखने वालों के लिए सुसंगत पर्यावरण

#### विकल्प B: स्थानीय डिव कंटेनर

**उन डेवलपर्स के लिए जो Docker के साथ स्थानीय विकास पसंद करते हैं**

1. इस रिपॉजिटरी को फोर्क और क्लोन करें अपने स्थानीय मशीन में  
   > **ध्यान दें**: यदि आप बुनियादी कॉन्फिग संपादित करना चाहते हैं तो कृपया [Dev Container Configuration](../../../.devcontainer/devcontainer.json) देखें
2. [Docker Desktop](https://www.docker.com/products/docker-desktop/) और [VS Code](https://code.visualstudio.com/) इंस्टॉल करें
3. VS Code में [Dev Containers एक्सटेंशन](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) इंस्टॉल करें
4. VS Code में रिपॉजिटरी फ़ोल्डर खोलें
5. जब पूछा जाए, क्लिक करें **Reopen in Container** (या `Ctrl+Shift+P` → "Dev Containers: Reopen in Container" का उपयोग करें)
6. कंटेनर के बनने और शुरू होने का इंतजार करें
7. आगे बढ़ें [चरण 2: Azure AI Foundry की व्यवस्था करें](#चरण-2-azure-ai-foundry-की-व्यवस्था-करें)

<img src="../../../translated_images/hi/devcontainer.21126c9d6de64494.webp" alt="Screenshot: Dev container setup" width="50%">

<img src="../../../translated_images/hi/image-3.bf93d533bbc84268.webp" alt="Screenshot: Dev container build complete" width="50%">

#### विकल्प C: अपने मौजूदा स्थानीय इंस्टॉलेशन का उपयोग करें

**उन डेवलपर्स के लिए जिनके पास मौजूदा Java पर्यावरण है**

पूर्वापेक्षाएँ:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html) 
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) या आपकी पसंदीदा IDE

चरण:
1. इस रिपॉजिटरी को अपने स्थानीय मशीन में क्लोन करें
2. प्रोजेक्ट अपनी IDE में खोलें
3. आगे बढ़ें [चरण 2: Azure AI Foundry की व्यवस्था करें](#चरण-2-azure-ai-foundry-की-व्यवस्था-करें)

> **प्रो टिप**: यदि आपकी मशीन कम स्पेस वाली है लेकिन आप स्थानीय रूप से VS Code चाहते हैं, तो GitHub Codespaces का उपयोग करें! आप अपने स्थानीय VS Code को क्लाउड-होस्टेड Codespace से कनेक्ट कर सकते हैं, दोनों का सर्वश्रेष्ठ लाभ पाने के लिए।

<img src="../../../translated_images/hi/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: created local devcontainer instance" width="50%">


## चरण 2: Azure AI Foundry की व्यवस्था करें

पाठ्यक्रम के AI मॉडल को Azure AI Foundry में कोड के रूप में तैनात करें। रिपॉजिटरी रूट से:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```
  
`azd` आपसे एक पर्यावरण नाम और क्षेत्र के लिए पूछता है, `gpt-4o-mini` और `text-embedding-3-small` तैनतियों के साथ Azure AI Foundry खाता प्रदान करता है, और उदाहरण के `.env` में एंडपॉइंट लिखता है — सभी **कीलेस** प्रमाणीकरण (कोई API कुंजी नहीं) के साथ।

> **पूर्ण मार्गदर्शन:** पूर्वापेक्षाओं, मैनुअल (पोर्टल) विकल्प, क्षेत्र मार्गदर्शन और लागत/सफाई नोट के लिए [Azure AI Foundry सेटअप गाइड](getting-started-azure-openai.md) देखें।

## चरण 3: अपने सेटअप का परीक्षण करें

एक बार आपके Foundry मॉडल तैनात हो जाने के बाद, उदाहरण ऐप के साथ कनेक्शन का परीक्षण करें [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) में।

1. अपने विकास पर्यावरण में टर्मिनल खोलें।
2. उदाहरण तक नेविगेट करें:  
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
  
3. सुनिश्चित करें कि आप साइन इन हैं (कीलेस प्रमाणीकरण के लिए टोकन चाहिए):  
   ```bash
   az login
   ```
  
   > यदि आपने `azd up` चलाया है, तो आपका एंडपॉइंट `.env` फ़ाइल में पहले से लिखा गया है।  
4. एप्लिकेशन चलाएं:  
   ```bash
   mvn clean spring-boot:run
   ```
  
आपको `gpt-4o-mini` मॉडल से एक प्रतिक्रिया दिखनी चाहिए।

### उदाहरण कोड को समझना

`examples/basic-chat-azure` के तहत उदाहरण एक Spring Boot ऐप है जो **Spring AI** का उपयोग करता है Azure AI Foundry से कीलेस प्रमाणीकरण के साथ कनेक्ट करने के लिए।

**यह कोड क्या करता है:**
- आपके Azure साइन-इन (Microsoft Entra ID) का उपयोग करके Azure AI Foundry से **कनेक्ट** होता है — कोई API कुंजी नहीं
- `gpt-4o-mini` मॉडल को एक प्रॉम्प्ट **भेजता है**
- AI की प्रतिक्रिया **प्राप्त** करता है और दिखाता है
- आपके सेटअप के सही काम करने को **सत्यापित** करता है

**मुख्य डिपेंडेंसी** (`pom.xml` में):  
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```
  
**कॉन्फ़िगरेशन** (`application.yml`):  
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

शानदार! अब आपके पास सब कुछ सेट है:

- Bicep + `azd` के साथ Azure AI Foundry मॉडल कोड के रूप में प्रदान किया गया
- अपना Java विकास पर्यावरण चल रहा है (चाहे वह Codespaces हो, dev कंटेनर हो, या स्थानीय)
- कीलेस प्रमाणीकरण (Microsoft Entra ID) के साथ Azure AI Foundry से कनेक्ट किया — कोई API कुंजी नहीं
- एक सरल उदाहरण के साथ परीक्षण किया जो आपके मॉडल से बात करता है

## अगले कदम

[अध्याय 3: कोर जनरेटिव AI तकनीकें](../03-CoreGenerativeAITechniques/README.md)

## समस्या निवारण

कोई समस्या आ रही है? यहाँ सामान्य समस्याएं और समाधान हैं:

- **प्रमाणीकरण विफल (401/403)?**  
  - `az login` चलाएं — प्रमाणीकरण कीलेस है, इसलिए आपको साइन इन होना चाहिए  
  - सुनिश्चित करें कि आपके खाते के पास संसाधन पर **Cognitive Services OpenAI User** भूमिका है  
  - यदि आपने अभी अभी प्रावधान किया है, तो भूमिका असाइनमेंट के फैलाव के लिए एक मिनट प्रतीक्षा करें  

- **Maven नहीं मिला?**  
  - यदि आप dev कंटेनर/Codespaces का उपयोग कर रहे हैं, तो Maven पहले से इंस्टॉल्ड होना चाहिए  
  - स्थानीय सेटअप के लिए, सुनिश्चित करें कि Java 21+ और Maven 3.9+ इंस्टॉल हैं  
  - सत्यापन के लिए `mvn --version` चलाएं  

- **`azd` नहीं मिला या प्रावधान विफल?**  
  - [Azure Developer CLI](https://aka.ms/azure-dev/install) इंस्टॉल करें और `azd auth login` चलाएं  
  - वह क्षेत्र चुनें जहाँ `gpt-4o-mini` उपलब्ध है (जैसे `eastus2`)  
  - विवरण के लिए [Azure AI Foundry सेटअप गाइड](getting-started-azure-openai.md) देखें  

- **Dev कंटेनर शुरू नहीं हो रहा?**  
  - सुनिश्चित करें कि Docker Desktop चल रहा है (स्थानीय विकास के लिए)  
  - कंटेनर दोबारा बनाएं: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"  

- **एप्लिकेशन संकलन त्रुटियाँ?**  
  - सुनिश्चित करें कि आप सही डायरेक्टरी में हैं: `02-SetupDevEnvironment/examples/basic-chat-azure`  
  - क्लीन और रीबिल्ड करें: `mvn clean compile`  

> **मदद चाहिए?**: अभी भी समस्या आ रही है? रिपॉजिटरी में एक इश्यू खोलें, हम आपकी मदद करेंगे।

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**अस्वीकरण**:
इस दस्तावेज़ का अनुवाद AI अनुवाद सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) का उपयोग करके किया गया है। जबकि हम सटीकता के लिए प्रयास करते हैं, कृपया ध्यान दें कि स्वचालित अनुवादों में त्रुटियाँ या अशुद्धियाँ हो सकती हैं। मूल दस्तावेज़ अपनी मूल भाषा में ही प्रामाणिक स्रोत माना जाना चाहिए। महत्वपूर्ण जानकारी के लिए, पेशेवर मानव अनुवाद की सिफारिश की जाती है। इस अनुवाद के उपयोग से उत्पन्न किसी भी गलतफहमी या गलत व्याख्या के लिए हम उत्तरदायी नहीं हैं।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->