# AGENTS.md

## परियोजना अवलोकन

यह जावा के साथ जनरेटिव एआई विकास सीखने के लिए एक शैक्षिक रिपॉजिटरी है। यह बड़े भाषा मॉडल (LLMs), प्रॉम्प्ट इंजीनियरिंग, एम्बेडिंग, RAG (रिकवरी-अगमेंटेड जनरेशन), और मॉडल कॉन्टेक्स्ट प्रोटोकॉल (MCP) को कवर करते हुए एक व्यापक हाथों-हाथ कोर्स प्रदान करता है।

**मुख्य तकनीकें:**
- जावा 21
- स्प्रिंग बूट 3.5.x
- स्प्रिंग AI 1.1.x
- मेवन
- LangChain4j
- Azure AI Foundry, Azure OpenAI, और OpenAI SDKs

**आर्किटेक्चर:**
- अध्यायों द्वारा व्यवस्थित कई स्वतंत्र स्प्रिंग बूट एप्लिकेशन
- विभिन्न AI पैटर्न दर्शाने वाले उदाहरण परियोजनाएं
- प्री-कॉन्फ़िगरड डेव कंटेनरों के साथ GitHub Codespaces-तैयार

## सेटअप कमांड्स

### पूर्वापेक्षाएँ
- जावा 21 या उससे उच्चतर
- मेवन 3.x
- Azure सदस्यता जिसमें Azure AI Foundry मॉडल डिप्लॉयमेंट हो (प्रावधान के लिए `azd up` का उपयोग करें)
- Azure CLI (`az`) और Azure Developer CLI (`azd`), बिना कुंजी प्रमाणीकरण के लिए साइन इन किया हुआ

### पर्यावरण सेटअप

**विकल्प 1: GitHub Codespaces (सिफारिश की गई)**
```bash
# रिपॉजिटरी को फोर्क करें और GitHub UI से एक कोडस्पेस बनाएं
# डेवलपमेंट कंटेनर सभी निर्भरताएँ स्वचालित रूप से इंस्टॉल करेगा
# पर्यावरण सेटअप के लिए लगभग 2 मिनट तक प्रतीक्षा करें
```

**विकल्प 2: स्थानीय डेव कंटेनर**
```bash
# रिपॉजिटरी क्लोन करें
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Dev Containers एक्सटेंशन के साथ VS कोड में खोलें
# जब संकेत दिया जाए तो कंटेनर में फिर से खोलें
```

**विकल्प 3: स्थानीय सेटअप**
```bash
# निर्भरता स्थापित करें
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# स्थापना सत्यापित करें
java -version
mvn -version
```

### कॉन्फ़िगरेशन

**Azure AI Foundry सेटअप (कीलेस, सिफारिश):**
```bash
# फाउंड्री खाता + मॉडल डिप्लॉयमेंट्स कोड के रूप में प्रावधानित करें
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd आपके एन्डपॉइंट (कोई कुंजी नहीं) के साथ examples/basic-chat-azure/.env लिखता है
```

**मैनुअल एंडपॉइंट कॉन्फ़िग:**
```bash
# यदि आपने azd का उपयोग नहीं किया है, तो स्वयं endpoint सेट करें (auth az login के माध्यम से keyless रहता है)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# .env संपादित करें: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## विकास कार्यप्रवाह

### परियोजना संरचना
```
/
├── 01-IntroToGenAI/              # Chapter 1: Introduction
├── 02-SetupDevEnvironment/       # Chapter 2: Environment setup
│   └── examples/                 # Working examples
├── 03-CoreGenerativeAITechniques/ # Chapter 3: Core techniques
├── 04-PracticalSamples/          # Chapter 4: Sample projects
│   ├── calculator/               # MCP service example
│   ├── foundrylocal/            # Local model integration
│   └── petstory/                # Multi-modal app
├── 05-ResponsibleGenAI/         # Chapter 5: Responsible AI
└── translations/                # Multi-language support
```

### एप्लिकेशन चलाना

**स्प्रिंग बूट एप्लिकेशन चलाना:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**परियोजना बनाना:**
```bash
cd [project-directory]
mvn clean install
```

**MCP कैलकुलेटर सर्वर शुरू करना:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# सर्वर http://localhost:8080 पर चलता है
```

**क्लाइंट उदाहरण चलाना:**
```bash
# दूसरे टर्मिनल में सर्वर शुरू करने के बाद
cd 04-PracticalSamples/calculator

# सीधे MCP क्लाइंट
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# AI-समर्थित क्लाइंट (AZURE_OPENAI_ENDPOINT + az लॉगिन आवश्यक)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# इंटरैक्टिव बॉट
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### हॉट रीलोड
स्प्रिंग बूट DevTools उन परियोजनाओं में शामिल हैं जो हॉट रीलोड का समर्थन करती हैं:
```bash
# जब Java फ़ाइलों को सहेजा जाएगा तो वे स्वचालित रूप से पुनः लोड हो जाएंगी
mvn spring-boot:run
```

## परीक्षण निर्देश

### परीक्षण चलाना

**परियोजना में सभी परीक्षण चलाएं:**
```bash
cd [project-directory]
mvn test
```

**विस्तृत आउटपुट के साथ परीक्षण चलाएं:**
```bash
mvn test -X
```

**विशिष्ट परीक्षण क्लास चलाएं:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### परीक्षण संरचना
- परीक्षण फाइलें JUnit 5 (Jupiter) का उपयोग करती हैं
- परीक्षण क्लासेस `src/test/java/` में होती हैं
- कैलकुलेटर परियोजना के क्लाइंट उदाहरण `src/test/java/com/microsoft/mcp/sample/client/` में हैं

### मैनुअल परीक्षण
कई उदाहरण इंटरैक्टिव एप्लिकेशन हैं जिन्हें मैनुअल परीक्षण की आवश्यकता होती है:

1. `mvn spring-boot:run` से एप्लिकेशन शुरू करें
2. एंडपॉइंट्स का परीक्षण करें या CLI के साथ संवाद करें
3. प्रत्येक परियोजना के README.md में उल्लेखित अपेक्षित व्यवहार की पुष्टि करें

### Azure AI Foundry के साथ परीक्षण
- उदाहरण चलाने से पहले `az login` से साइन इन करें (कीलेस प्रमाणीकरण)
- सुनिश्चित करें कि आपके खाते को संसाधन पर Cognitive Services OpenAI User भूमिका प्राप्त है
- चैप्टर 5 में जिम्मेदार AI उदाहरण के साथ सामग्री फ़िल्टरिंग का परीक्षण करें

## कोड स्टाइल दिशानिर्देश

### जावा सम्मेलन
- **जावा संस्करण:** आधुनिक फीचर्स के साथ जावा 21
- **स्टाइल:** मानक जावा सम्मेलन का पालन करें
- **नामकरण:** 
  - क्लासेस: PascalCase
  - मेथड/वेरिएबल्स: camelCase
  - कॉन्सटेंट्स: UPPER_SNAKE_CASE
  - पैकेज नाम: lowercase

### स्प्रिंग बूट पैटर्न
- बिजनेस लॉजिक के लिए `@Service` का उपयोग करें
- REST एंडपॉइंट्स के लिए `@RestController` का उपयोग करें
- कॉन्फ़िगरेशन `application.yml` या `application.properties` के माध्यम से
- हार्ड-कोडेड वैल्यूज की बजाय परिवेश चर बेहतर हैं
- MCP-एक्सपोज़ किए गए मेथड के लिए `@Tool` एनोटेशन का उपयोग करें

### फाइल संगठन
```
src/
├── main/
│   ├── java/
│   │   └── com/microsoft/[component]/
│   │       ├── [Component]Application.java
│   │       ├── config/
│   │       ├── controller/
│   │       ├── service/
│   │       └── exception/
│   └── resources/
│       ├── application.yml
│       └── static/
└── test/
    └── java/
        └── com/microsoft/[component]/
```

### निर्भरताएं
- मेवन `pom.xml` के माध्यम से प्रबंधित
- संस्करण प्रबंधन के लिए स्प्रिंग AI BOM
- AI इंटीग्रेशन के लिए LangChain4j
- स्प्रिंग निर्भरताओं के लिए स्प्रिंग बूट स्टार्टर पैरेंट

### कोड टिप्पणियां
- सार्वजनिक APIs के लिए JavaDoc जोड़ें
- जटिल AI इंटरैक्शन के लिए व्याख्यात्मक टिप्पणियां शामिल करें
- MCP टूल विवरण स्पष्ट रूप से दस्तावेजित करें

## निर्माण और डिप्लॉयमेंट

### परियोजनाएं बनाना

**परीक्षण के बिना निर्माण करें:**
```bash
mvn clean install -DskipTests
```

**सभी चेक के साथ निर्माण करें:**
```bash
mvn clean install
```

**एप्लिकेशन पैकेज करें:**
```bash
mvn package
# target/ निर्देशिका में JAR बनाता है
```

### आउटपुट निर्देशिकाएँ
- संकलित क्लासेस: `target/classes/`
- परीक्षण क्लासेस: `target/test-classes/`
- JAR फाइलें: `target/*.jar`
- मेवन आर्टिफैक्ट्स: `target/`

### पर्यावरण-विशिष्ट कॉन्फ़िगरेशन

**विकास:**
```yaml
# application.yml (keyless - no api-key; auth via DefaultAzureCredential)
spring:
  ai:
    azure:
      openai:
        endpoint: ${AZURE_OPENAI_ENDPOINT}
        chat:
          options:
            deployment-name: ${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}
```

**प्रोडक्शन:**
- कीलेस प्रमाणीकरण के लिए `az login` के बजाय प्रबंधित पहचान का उपयोग करें
- `AZURE_OPENAI_ENDPOINT` को अपनी प्रोडक्शन Foundry संसाधन की ओर इंगित करें
- कॉन्फ़िगरेशन को पर्यावरण चर या Azure Key Vault के माध्यम से प्रबंधित करें

### डिप्लॉयमेंट विचार
- यह एक शैक्षिक रिपॉजिटरी है जिसमें उदाहरण एप्लिकेशन हैं
- इसे वैसे का वैसे प्रोडक्शन डिप्लॉयमेंट के लिए डिज़ाइन नहीं किया गया है
- उदाहरण प्रोडक्शन उपयोग के लिए पैटर्न को अनुकूलित करने के लिए दिखाते हैं
- विशिष्ट डिप्लॉयमेंट नोट्स के लिए व्यक्तिगत परियोजना README देखें

## अतिरिक्त नोट्स

### Azure AI Foundry
- **कीलेस प्रमाणीकरण:** Microsoft Entra ID के साथ कनेक्ट करें — कोई API कुंजी प्रबंधित नहीं करनी होती
- **कोड के रूप में प्रावधान:** Bicep + azd (`azd up`) खाते और मॉडल डिप्लॉयमेंट बनाते हैं
- वही OpenAI-संगत कोड स्थानीय (`az login`) और Azure (प्रबंधित पहचान) दोनों जगह चलता है

### कई परियोजनाओं के साथ काम करना
प्रत्येक उदाहरण परियोजना स्वतंत्र है:
```bash
# विशिष्ट परियोजना पर जाएं
cd 04-PracticalSamples/[project-name]

# प्रत्येक का अपना pom.xml होता है और इसे स्वतंत्र रूप से बनाया जा सकता है
mvn clean install
```

### सामान्य समस्याएं

**जावा संस्करण मेल नहीं खाता:**
```bash
# जावा 21 सत्यापित करें
java -version
# यदि आवश्यक हो तो JAVA_HOME अपडेट करें
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**निर्भरता डाउनलोड समस्या:**
```bash
# Maven कैश साफ़ करें और पुनः प्रयास करें
rm -rf ~/.m2/repository
mvn clean install
```

**एंडपॉइंट या साइन-इन नहीं मिला:**
```bash
# वर्तमान सत्र में एंडपॉइंट सेट करें और साइन इन करें (कीलेस)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# या प्रोजेक्ट डायरेक्टरी में .env फ़ाइल का उपयोग करें
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**पोर्ट पहले से उपयोग में है:**
```bash
# Spring Boot डिफ़ॉल्ट रूप से पोर्ट 8080 का उपयोग करता है
# application.properties में परिवर्तन:
server.port=8081
```

### बहुभाषी समर्थन
- 45+ भाषाओं में स्वचालित अनुवाद के जरिए दस्तावेज़ उपलब्ध
- अनुवाद `translations/` निर्देशिका में
- अनुवाद प्रबंधन GitHub Actions वर्कफ़्लो द्वारा

### सीखने का मार्ग
1. [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md) से शुरू करें
2. अध्यायों को क्रम से पालन करें (01 → 05)
3. प्रत्येक अध्याय में हैंड्स-ऑन उदाहरण पूर्ण करें
4. अध्याय 4 में उदाहरण परियोजनाओं का अन्वेषण करें
5. चैप्टर 5 में जिम्मेदार AI प्रथाएं सीखें

### विकास कंटेनर
`.devcontainer/devcontainer.json` कॉन्फ़िगर करता है:
- जावा 21 विकास वातावरण
- मेवन पूर्व-इंस्टॉल्ड
- VS कोड जावा एक्सटेंशन
- स्प्रिंग बूट टूल्स
- GitHub Copilot एकीकरण
- Docker-in-Docker समर्थन
- Azure CLI

### प्रदर्शन विचार
- Azure AI Foundry डिप्लॉयमेंट्स में प्रति मिनट टोकन/अनुरोध कोटा है
- एम्बेडिंग के लिए उपयुक्त बैच साइज का उपयोग करें
- पुनः API कॉल्स के लिए कैशिंग पर विचार करें
- लागत अनुकूलन के लिए टोकन उपयोग मॉनिटर करें

### सुरक्षा नोट्स
- कभी भी `.env` फाइलें कमिट न करें (पहले से .gitignore में हैं)
- API कुंजी की बजाय कीलेस ऑथेंटिकेशन (Microsoft Entra ID) प्राथमिकता दें
- एज़ूर में प्रबंधित पहचान का उपयोग करें; स्थानीय विकास के लिए `az login`
- चैप्टर 5 में जिम्मेदार AI दिशानिर्देशों का पालन करें

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**अस्वीकरण**:
इस दस्तावेज़ का अनुवाद AI अनुवाद सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) का उपयोग करके किया गया है। जबकि हम सटीकता के लिए प्रयास करते हैं, कृपया ध्यान दें कि स्वचालित अनुवादों में त्रुटियाँ या अशुद्धियाँ हो सकती हैं। मूल दस्तावेज़ अपनी मूल भाषा में ही प्रामाणिक स्रोत माना जाना चाहिए। महत्वपूर्ण जानकारी के लिए, पेशेवर मानव अनुवाद की सिफारिश की जाती है। इस अनुवाद के उपयोग से उत्पन्न किसी भी गलतफहमी या गलत व्याख्या के लिए हम उत्तरदायी नहीं हैं।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->