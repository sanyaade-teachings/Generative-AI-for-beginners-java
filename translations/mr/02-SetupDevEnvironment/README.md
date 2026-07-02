# Java साठी जनरेटिव्ह AI साठी विकासाचे पर्यावरण सेट करणे

> **तत्काळ प्रारंभ:** बायसेप + `azd` सह कोड स्वरूपात तुमची AI मॉडेल्स **Azure AI Foundry** वर काही मिनिटांत प्राव्हिजन करा — पहा [Azure AI Foundry सेटअप मार्गदर्शक](getting-started-azure-openai.md). प्रमाणीकरण **कीलेस** आहे (Microsoft Entra ID), त्यामुळे API कीज व्यवस्थापित करण्याची गरज नाही.

## तुम्हाला काय शिकायला मिळेल

- AI अनुप्रयोगांसाठी Java विकास पर्यावरण सेट करणे
- तुमची प्राधान्यक्रम विकास वातावरण निवडणे आणि कॉन्फिगर करणे (Codespaces सह क्लाउड-फर्स्ट, लोकल डेव्ह कंटेनर, किंवा पूर्ण लोकल सेटअप)
- Azure AI Foundry मॉडेलशी कनेक्ट करून तुमच्या सेटअपने चाचणी करणे

## विषय सूची

- [तुम्हाला काय शिकायला मिळेल](#तुम्हाला-काय-शिकायला-मिळेल)
- [परिचय](#परिचय)
- [पायरी 1: तुमचे विकास पर्यावरण सेट करा](#पायरी-1-तुमचे-विकास-पर्यावरण-सेट-करा)
  - [पर्याय A: GitHub Codespaces (शिफारस)](#पर्याय-a-github-codespaces-शिफारस)
  - [पर्याय B: लोकल डेव्ह कंटेनर](#पर्याय-b-लोकल-डेव्ह-कंटेनर)
  - [पर्याय C: तुमचा विद्यमान लोकल इंस्टॉलेशन वापरा](#पर्याय-c-तुमचा-विद्यमान-लोकल-इंस्टॉलेशन-वापरा)
- [पायरी 2: Azure AI Foundry प्राव्हिजन करा](#पायरी-2-azure-ai-foundry-प्राव्हिजन-करा)
- [पायरी 3: तुमचे सेटअप चाचणी करा](#पायरी-3-तुमचे-सेटअप-चाचणी-करा)
- [समस्यांचे निवारण](#समस्यांचे-निवारण)
- [सारांश](#सारांश)
- [पुढील पावले](#पुढील-पावले)

## परिचय

हा अध्याय तुम्हाला विकासाचे पर्यावरण सेट करण्यासाठी मार्गदर्शन करेल. संपूर्ण कोर्स दरम्यान आपण **Azure AI Foundry** मॉडेल्ससाठी वापर करू. तुम्ही मॉडेल्स बायसेप आणि Azure Developer CLI (`azd`) सह कोड स्वरूपात प्राव्हिजन कराल, नंतर **कीलेस प्रमाणीकरण** (Microsoft Entra ID) सह कनेक्ट कराल — कोणतीही API की कॉपी किंवा लीक करण्याची गरज नाही.

**कोणताही लोकल सेटअप आवश्यक नाही!** तुम्ही GitHub Codespaces वापरू शकता, जे तुम्हाला तुमच्या ब्राउझरमध्ये पूर्ण विकास पर्यावरण प्रदान करते, आणि तेथून Foundry प्राव्हिजन करा.

आपण हा कोर्ससाठी **Azure AI Foundry** वापरतो कारण:
- **कोड स्वरूपात प्राव्हिजन केलेले** — एक `azd up` खाते आणि मॉडेल डिप्लॉयमेंट्स तैनात करते
- **कीलेस** — Azure साईन-इन किंवा व्यवस्थापित ओळखीने प्रमाणीकरण
- **प्रॉडक्शन-रेडी** — तोच कोड लोकल आणि Azure मध्ये चालतो
- **लवचिक** — तुझ्या कोडचा बदल न करता, फक्त डिप्लॉयमेंटचे नाव बदलून मॉडेल बदलू शकतो

> **टीप:** Azure AI Foundry डिप्लॉयमेंट्स टोकनप्रमाणे शुल्क आकारले जाते (पे-एज-यू-गो). प्राव्हिजनिंग, प्रदेश आणि खर्च तपशीलांसाठी [Azure AI Foundry सेटअप मार्गदर्शक](getting-started-azure-openai.md) पाहा.

## पायरी 1: तुमचे विकास पर्यावरण सेट करा

<a name="quick-start-cloud"></a>

या जनरेटिव्ह AI Java कोर्ससाठी आवश्यक असलेले सर्व टूल्स ताणायला आणि सेटअप वेळ कमी व्हावा म्हणून आम्ही एक पूर्व-कॉन्फिगर केलेला विकास कंटेनर तयार केला आहे. तुमच्या प्राधान्यक्रम विकास पद्धती निवडा:

### पर्यावरण सेटअप पर्याय:

#### पर्याय A: GitHub Codespaces (शिफारस)

**2 मिनिटांत कोडिंग सुरू करा - कोणतेही लोकल सेटअप आवश्यक नाही!**

1. या रिपॉझिटरीचा फोर्क तुमच्या GitHub खात्यावर करा  
   > **टीप:** जर तुम्हाला मूळ कॉन्फिग संपादित करायचा असेल तर कृपया [Dev Container Configuration](../../../.devcontainer/devcontainer.json) पहा  
2. **Code** → **Codespaces** टॅब → **...** → **New with options...** क्लिक करा  
3. डिफॉल्ट्स वापरा – यामुळे **Dev container configuration** निवडले जाईल: या कोर्ससाठी तयार केलेले **Generative AI Java Development Environment** कस्टम डेव्हकंटेनर  
4. **Create codespace** क्लिक करा  
5. पर्यावरण तयार होईपर्यंत सुमारे 2 मिनिटे थांबा  
6. पुढे जा [पायरी 2: Azure AI Foundry प्राव्हिजन करा](#पायरी-2-azure-ai-foundry-प्राव्हिजन-करा)

<img src="../../../translated_images/mr/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: Codespaces submenu" width="50%">

<img src="../../../translated_images/mr/image.833552b62eee7766.webp" alt="Screenshot: New with options" width="50%">

<img src="../../../translated_images/mr/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: Create codespace options" width="50%">

> **Codespaces चे फायदे**:  
> - कोणताही लोकल इंस्टॉलेशन आवश्यक नाही  
> - कोणत्याही डिव्हाइसवर ब्राउझरसह वापरता येते  
> - सर्व टूल्स आणि अवलंबनांसह पूर्व-कॉन्फिगर केलेले  
> - वैयक्तिक खात्यांसाठी प्रति महिना 60 तास फ्री  
> - सर्व शिकणाऱ्यांसाठी एकसंध पर्यावरण

#### पर्याय B: लोकल डेव्ह कंटेनर

**डॉकऱ सह लोकल विकास पसंत करणाऱ्या विकासकांसाठी**

1. या रिपॉझिटरीचा फोर्क करून तुमच्या लोकल मशीनवर क्लोन करा  
   > **टीप:** मूळ कॉन्फिग संपादित करायचा असल्यास [Dev Container Configuration](../../../.devcontainer/devcontainer.json) पहा  
2. [Docker Desktop](https://www.docker.com/products/docker-desktop/) आणि [VS Code](https://code.visualstudio.com/) इंस्टॉल करा  
3. VS Code मध्ये [Dev Containers एक्सटेंशन](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) इंस्टॉल करा  
4. रिपॉझिटरी फोल्डर VS Code मध्ये उघडा  
5. जेव्हा विनंती होईल, **Reopen in Container** क्लिक करा (किंवा `Ctrl+Shift+P` → "Dev Containers: Reopen in Container" वापरा)  
6. कंटेनर बिल्ड होईपर्यंत आणि चालू होईपर्यंत थांबा  
7. पुढे जा [पायरी 2: Azure AI Foundry प्राव्हिजन करा](#पायरी-2-azure-ai-foundry-प्राव्हिजन-करा)

<img src="../../../translated_images/mr/devcontainer.21126c9d6de64494.webp" alt="Screenshot: Dev container setup" width="50%">

<img src="../../../translated_images/mr/image-3.bf93d533bbc84268.webp" alt="Screenshot: Dev container build complete" width="50%">

#### पर्याय C: तुमचा विद्यमान लोकल इंस्टॉलेशन वापरा

**विद्यमान Java पर्यावरण असलेल्या विकासकांसाठी**

पूर्वअट:  
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html)  
- [Maven 3.9+](https://maven.apache.org/download.cgi)  
- [VS Code](https://code.visualstudio.com) किंवा तुमचा प्राधान्यक्रम IDE

पावले:  
1. या रिपॉझिटरीला तुमच्या लोकल मशीनवर क्लोन करा  
2. तुमच्या IDE मध्ये प्रोजेक्ट उघडा  
3. पुढे जा [पायरी 2: Azure AI Foundry प्राव्हिजन करा](#पायरी-2-azure-ai-foundry-प्राव्हिजन-करा)

> **सुप्रतिपादक टीप:** जर तुमच्याकडे कमी किमतीचे मशीन असेल पण लोकल VS Code वापरायचा असेल तर GitHub Codespaces वापरा! तुम्ही तुमच्या लोकल VS Code ला क्लाउड-होस्टेड Codespace शी कनेक्ट करू शकता, ज्याने दोन्ही जगांचं फायदे मिळतील.

<img src="../../../translated_images/mr/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: created local devcontainer instance" width="50%">

## पायरी 2: Azure AI Foundry प्राव्हिजन करा

कोर्सचे AI मॉडेल Azure AI Foundry वर कोड स्वरूपात डिप्लॉय करा. रिपॉझिटरीच्या रूटमधून:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```
  
`azd` पर्यावरण नाव आणि प्रदेश विचारतो, `gpt-4o-mini` आणि `text-embedding-3-small` डिप्लॉयमेंटसह Azure AI Foundry खाते प्राव्हिजन करतो, आणि उदाहरणाच्या `.env` मध्ये एंडपॉइंट लिहितो — हे सगळं **कीलेस** प्रमाणीकरण (कोणतीही API की नाही) सह.

> **पूर्ण मार्गदर्शन:** पूर्वअटीसाठी, मॅन्युअल (पोर्टल) पर्यायासाठी, प्रदेश मार्गदर्शनासाठी आणि खर्च/स्वच्छता नोट्ससाठी [Azure AI Foundry सेटअप मार्गदर्शक](getting-started-azure-openai.md) पहा.

## पायरी 3: तुमचे सेटअप चाचणी करा

तुमचे Foundry मॉडेल प्राव्हिजन झाल्यानंतर, उदाहरण अॅप [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) मध्ये कनेक्शन चाचणी करा.

1. तुमच्या विकास पर्यावरणात टर्मिनल उघडा.  
2. उदाहरण फोल्डरमध्ये जा:  
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
  
3. खात्री करा की तुम्ही साइन इन केला आहात (कीलेस ऑथसाठी टोकन आवश्यक आहे):  
   ```bash
   az login
   ```
  
   > जर तुम्ही `azd up` चालवले असेल तर `.env` फाईल तुमच्यासाठी एंडपॉइंटसह आधीच लिहिली गेली आहे.  
4. अॅप्लिकेशन चालवा:  
   ```bash
   mvn clean spring-boot:run
   ```
  
तुम्हाला `gpt-4o-mini` मॉडेलकडून प्रतिसाद दिसेल.

### उदाहरण कोड समजून घेणे

`examples/basic-chat-azure` अंतर्गत उदाहरण एक Spring Boot अॅप आहे ज्यात **Spring AI** वापरून Azure AI Foundry शी कीलेस प्रमाणीकरणाद्वारे कनेक्ट केले जाते.

**हा कोड काय करतो:**  
- Azure AI Foundry शी तुमच्या Azure साइन-इन (Microsoft Entra ID) वापरून कनेक्ट होतो — कोणतीही API की नाही  
- `gpt-4o-mini` मॉडेलला प्रम्प्ट पाठवतो  
- AI चा प्रतिसाद प्राप्त करतो आणि दर्शवतो  
- तुमच्या सेटअपची योग्य कार्यक्षमता पडताळतो

**महत्वाची अवलंबन (in `pom.xml`):**  
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```
  
**कॉन्फिगरेशन** (`application.yml`):  
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

छान! आता तुम्ही सगळं सेट केलं आहे:

- Bicep + `azd` सह कोड स्वरूपात Azure AI Foundry मॉडेल्स प्राव्हिजन केले  
- तुमचं Java विकास पर्यावरण चालू आहे (Codespaces, डेव्ह कंटेनर, किंवा लोकल काहीही असो)  
- कीलेस प्रमाणीकरण (Microsoft Entra ID) सह Azure AI Foundry शी कनेक्ट केले — कोणतीही API की नाही  
- तुमच्या मॉडेलशी संवाद साधणाऱ्या सोप्या उदाहरणाने सगळं व्यवस्थित काम करतंय याची चाचणी केली

## पुढील पावले

[अध्याय 3: कोअर जनरेटिव्ह AI तंत्रे](../03-CoreGenerativeAITechniques/README.md)

## समस्यांचे निवारण

काही समस्या येत आहेत? येथे सामान्य समस्या आणि उपाय आहेत:

- **प्रमाणीकरण अयशस्वी (401/403)?**  
  - `az login` चालवा — प्रमाणीकरण कीलेस असल्यामुळे तुम्हाला साइन इन असावं लागतं  
  - खात्यावर **Cognitive Services OpenAI User** भूमिका आहे का तपासा  
  - जर तुम्ही नुकताच प्राव्हिजन केले असेल, भूमिका लागू होण्यासाठी थोडा वेळ थांबा

- **Maven आढळत नाही?**  
  - डेव्ह कंटेनर/Codespaces वापरत असल्यास Maven पूर्व-इंस्टॉल्ड असणे अपेक्षित आहे  
  - लोकल सेटअपसाठी, Java 21+ आणि Maven 3.9+ इंस्टॉल केलेले आहेत की नाही ते तपासा  
  - `mvn --version` चालवून इंस्टॉलेशन तपासा

- **`azd` आढळत नाही किंवा प्राव्हिजनिंग अयशस्वी?**  
  - [Azure Developer CLI](https://aka.ms/azure-dev/install) इंस्टॉल करा आणि `azd auth login` चालवा  
  - `gpt-4o-mini` उपलब्ध असलेला प्रदेश निवडा (उदा. `eastus2`)  
  - तपशीलांसाठी [Azure AI Foundry सेटअप मार्गदर्शक](getting-started-azure-openai.md) पहा

- **डेव्ह कंटेनर सुरू होत नाही?**  
  - Docker Desktop चालू आहे का ते पाहा (लोकल विकासासाठी)  
  - कंटेनर पुन्हा तयार करण्याचा प्रयत्न करा: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **अॅप्लिकेशन संकलनातील त्रुटी?**  
  - योग्य फोल्डर मध्ये आहात का याची खात्री करा: `02-SetupDevEnvironment/examples/basic-chat-azure`  
  - क्लीन आणि पुन्हा संकलन करण्याचा प्रयत्न करा: `mvn clean compile`

> **मदतीची गरज आहे?**: अजूनही समस्या असल्यास रिपॉझिटरीमध्ये इश्यू उघडा, आम्ही मदत करू.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**अस्वीकरण**:
हा दस्तऐवज AI भाषांतर सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) चा वापर करून अनुवादित केला आहे. जरी आम्ही अचूकतेसाठी प्रयत्न करतो, तरी कृपया लक्षात घ्या की स्वयंचलित भाषांतरांमध्ये त्रुटी किंवा अचूकतेची कमतरता असू शकते. मूळ दस्तऐवज त्याच्या मूळ भाषेत अधिकृत स्रोत मानला पाहिजे. महत्त्वाची माहिती असल्यास, व्यावसायिक मानवी भाषांतराची शिफारस केली जाते. या भाषांतराच्या वापरामुळे उद्भवणाऱ्या कोणत्याही गैरसमज किंवा चुकीच्या अर्थलावणीसाठी आम्ही जबाबदार नाही.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->