# AGENTS.md

## परियोजना अवलोकन

यो Java सँग जनरेटिभ AI विकास सिक्नका लागि शैक्षिक भण्डारणशाला हो। यसले ठूलो भाषा मोडेलहरू (LLMs), प्रॉम्प्ट इन्जिनियरिङ, एम्बेडिङ्स, RAG (रिट्रिवल-अग्मेन्टेड जेनेरेसन), र मोडेल कन्टेक्स्ट प्रोटोकल (MCP) को समग्र व्यावहारिक पाठक्रम प्रदान गर्दछ।

**प्रमुख प्रविधिहरू:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI, र OpenAI SDKs

**वास्‍तुशिल्प:**
- अध्यायहरूको अनुसार व्यवस्थित धेरै स्वतन्त्र Spring Boot अनुप्रयोगहरू
- विभिन्न AI पैटर्नहरू देखाउने नमूना परियोजनाहरू
- पूर्व-कन्फिगर गरिएको विकास कन्टेनरका साथ GitHub Codespaces- तयार

## सेटअप आदेशहरू

### पूर्वआवश्यकताहरू
- Java 21 वा माथि
- Maven 3.x
- Azure AI Foundry मोडेल डिप्लोयमेन्ट भएको Azure सदस्यता (प्रदान गर्न `azd up` प्रयोग गर्नुहोस्)
- Azure CLI (`az`) र Azure Developer CLI (`azd`), कुंजी बिना प्रमाणीकरणका लागि साइन इन गरिएको

### वातावरण सेटअप

**विकल्प 1: GitHub Codespaces (सिफारिस गरिएको)**
```bash
# रिपोजिटरी फोर्क गर्नुहोस् र GitHub UI बाट कोडस्पेस सिर्जना गर्नुहोस्
# डेव कन्टेनरले स्वचालित रूपमा सबै निर्भरताहरू स्थापना गर्नेछ
# वातावरण सेटअपको लागि करिब २ मिनेट पर्खनुहोस्
```

**विकल्प 2: स्थानीय विकास कन्टेनर**
```bash
# रिपोजिटरी क्लोन गर्नुहोस्
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Dev Containers एक्सटेन्सनसहित VS Code मा खोल्नुहोस्
# सोधिएपछि कन्टेनरमा पुन: खोल्नुहोस्
```

**विकल्प 3: स्थानीय सेटअप**
```bash
# निर्भरता स्थापना गर्नुहोस्
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# स्थापना पुष्टि गर्नुहोस्
java -version
mvn -version
```

### कन्फिगरेसन

**Azure AI Foundry सेटअप (कुंजी बिना, सिफारिस गरिएको):**
```bash
# Foundry खातालाई + मोडेल परिनियोजनहरूलाई कोडको रूपमा प्रावधान गर्नुहोस्
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd ले तपाईंको अन्तबिन्दुको साथ examples/basic-chat-azure/.env लेख्छ (कुनै कुञ्जी छैन)
```

**म्यानुअल अन्तिम बिन्दु कन्फिग:**
```bash
# यदि तपाईंले azd प्रयोग गर्नुभएको छैन भने, अन्तिम बिन्दु आफैं सेट गर्नुहोस् (प्रमाणीकरण az login मार्फत कुञ्जी रहित रहन्छ)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# .env सम्पादन गर्नुहोस्: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
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

### अनुप्रयोगहरू चलाइरहनुहोस्

**Spring Boot अनुप्रयोग चलाउने:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**परियोजना निर्माण गर्ने:**
```bash
cd [project-directory]
mvn clean install
```

**MCP क्यालकुलेटर सर्भर सुरु गर्ने:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# सर्भर http://localhost:8080 मा चलिरहेको छ
```

**क्लाइन्ट उदाहरणहरू चलाउने:**
```bash
# अर्को टर्मिनलमा सर्भर सुरु गरेपछि
cd 04-PracticalSamples/calculator

# सोझो MCP क्लाइन्ट
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# AI-सञ्चालित क्लाइन्ट (AZURE_OPENAI_ENDPOINT + az login आवश्यक)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# अन्तरक्रियात्मक बोट
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### हट रीलोड
Spring Boot DevTools ती परियोजनाहरूमा समावेश गरिएको छ जुन हट रीलोडलाई समर्थन गर्छन्:
```bash
# जाभा फाइलहरूमा परिवर्तनहरू सेभ गर्दा स्वचालित रूपमा पुन: लोड हुनेछन्
mvn spring-boot:run
```

## परीक्षण निर्देशनहरू

### परीक्षणहरू चलाउने

**परियोजनामा सबै परीक्षणहरू चलाउनुहोस्:**
```bash
cd [project-directory]
mvn test
```

**विश्लेषणात्मक आउटपुटसहित परीक्षण चलाउनुहोस्:**
```bash
mvn test -X
```

**विशिष्ट परीक्षण कक्षा चलाउनुहोस्:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### परीक्षण संरचना
- परीक्षण फाइलहरू JUnit 5 (Jupiter) प्रयोग गर्छन्
- परीक्षण कक्षा `src/test/java/` मा छन्
- क्यालकुलेटर परियोजनाको क्लाइन्ट उदाहरणहरू `src/test/java/com/microsoft/mcp/sample/client/` मा छन्

### म्यानुअल परीक्षण
धेरै उदाहरणहरू अन्तरक्रियात्मक अनुप्रयोगहरू हुन् जसलाई म्यानुअल परीक्षण आवश्यक छ:

1. अनुप्रयोग `mvn spring-boot:run` सँग सुरु गर्नुहोस्
2. अन्तिम बिन्दुहरू परीक्षण गर्नुहोस् वा CLI सँग अन्तरक्रिया गर्नुहोस्
3. अपेक्षित व्यवहार हरेक परियोजनाको README.md मा रहेको कागजातसँग मेल खान्छ कि छैन जाँच गर्नुहोस्

### Azure AI Foundry सँग परीक्षण
- उदाहरणहरू चलाउनु अघि `az login` सँग साइन इन गर्नुहोस् (कुंजी बिना प्रमाणीकरण)
- सुनिश्चित गर्नुहोस् तपाईंको खातामा स्रोतमा Cognitive Services OpenAI प्रयोगकर्ता भूमिका छ
- अध्याय ५ मा जिम्मेवार AI उदाहरणसँग सामग्री फिल्टरिङ परीक्षण गर्नुहोस्

## कोड शैली निर्देशनहरू

### Java सम्मेलनहरू
- **Java संस्करण:** आधुनिक सुविधाहरू सहित Java 21
- **शैली:** मानक Java सम्मेलनहरू अनुसरण गर्नुहोस्
- **नामकरण:**
  - कक्षाहरू: PascalCase
  - विधि/चरहरू: camelCase
  - स्थायीहरू: UPPER_SNAKE_CASE
  - प्याकेज नामहरू: साना अक्षरहरू

### Spring Boot ढाँचा
- व्यावसायिक तर्कका लागि `@Service` प्रयोग गर्नुहोस्
- REST अन्तिम बिन्दुहरूका लागि `@RestController` प्रयोग गर्नुहोस्
- `application.yml` वा `application.properties` मार्फत कन्फिगरेसन
- हार्ड-कोडेड मानहरू भन्दा वातावरण चरहरू प्राथमिकता दिनुहोस्
- MCP-प्रदर्शित विधिहरूका लागि `@Tool` एनोटेसन प्रयोग गर्नुहोस्

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

### निर्भरता
- Maven `pom.xml` मार्फत व्यवस्थापन
- संस्करण व्यवस्थापनका लागि Spring AI BOM
- AI एकीकरणका लागि LangChain4j
- Spring निर्भरता लागि Spring Boot स्टार्टर पलेन्ट

### कोड टिप्पणीहरू
- सार्वजनिक API हरूको लागि JavaDoc थप्नुहोस्
- जटिल AI अन्तरक्रियाहरूका लागि व्याख्यात्मक टिप्पणीहरू समावेश गर्नुहोस्
- MCP उपकरण विवरणहरू स्पष्ट रूपमा कागजातिकरण गर्नुहोस्

## निर्माण र तैनाती

### परियोजनाहरू निर्माण

**परीक्षण बिना निर्माण:**
```bash
mvn clean install -DskipTests
```

**सबै जाँचसहित निर्माण:**
```bash
mvn clean install
```

**अनुप्रयोग प्याकेजिङ:**
```bash
mvn package
# target/ फोल्डरमा JAR बनाउँछ
```

### आउटपुट निर्देशिका
- कम्पाइल गरिएको कक्षाहरू: `target/classes/`
- परीक्षण कक्षाहरू: `target/test-classes/`
- JAR फाइलहरू: `target/*.jar`
- Maven वस्तुहरू: `target/`

### वातावरण-विशिष्ट कन्फिगरेसन

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

**उत्पादन:**
- कुंजी बिना प्रमाणीकरणको लागि `az login` को सट्टा व्यवस्थापित पहिचान प्रयोग गर्नुहोस्
- आफ्नो उत्पादन Foundry स्रोतमा `AZURE_OPENAI_ENDPOINT` निर्दिष्ट गर्नुहोस्
- कन्फिगरेसन वातावरण चरहरू वा Azure Key Vault मार्फत व्यवस्थापन गर्नुहोस्

### तैनाती विचारहरू
- यो नमूना अनुप्रयोगहरूसहित शैक्षिक भण्डारणशाला हो
- सिधै उत्पादनमा तैनाथिका लागि डिजाइन गरिएको छैन
- नमूनाहरू उत्पादनमा अनुकूलनका लागि विभिन्न ढाँचाहरू देखाउँछन्
- विशिष्ट तैनाती नोटहरू हरेक परियोजनाको README मा हेर्नुहोस्

## अतिरिक्त नोटहरू

### Azure AI Foundry
- **कुंजी बिना प्रमाणीकरण:** Microsoft Entra ID मार्फत जडान — API कुञ्जी प्रबन्धन आवश्यक छैन
- **कोडको रूपमा प्रावधान:** Bicep + azd (`azd up`) ले खाता र मोडेल डिप्लोयमेन्टहरू सिर्जना गर्दछ
- एउटै OpenAI-संगत कोड स्थानीय (`az login`) र Azure (व्यवस्थित पहिचान) मा चल्छ

### धेरै परियोजनाहरूको काम
प्रत्येक नमूना परियोजना स्वतन्त्र छ:
```bash
# विशेष परियोजनामा नेभिगेट गर्नुहोस्
cd 04-PracticalSamples/[project-name]

# प्रत्येकसँग आफ्नै pom.xml हुन्छ र स्वतन्त्र रूपमा निर्माण गर्न सकिन्छ
mvn clean install
```

### सामान्य समस्या

**Java संस्करण असङ्गति:**
```bash
# Java 21 प्रमाणित गर्नुहोस्
java -version
# आवश्यक भए JAVA_HOME अपडेट गर्नुहोस्
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**निर्भरता डाउनलोड समस्या:**
```bash
# Maven क्यास सफा गर्नुहोस् र पुन: प्रयास गर्नुहोस्
rm -rf ~/.m2/repository
mvn clean install
```

**अन्तिम बिन्दु वा साइन-इन फेला परेन:**
```bash
# हालको सत्रमा अन्तिम बिन्दु सेट गर्नुहोस् र साइन इन गर्नुहोस् (किप्राप्त नभएको)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# वा परियोजना निर्देशिकामा .env फाइल प्रयोग गर्नुहोस्
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**पोर्ट पहिले नै प्रयोगमा छ:**
```bash
# Spring Boot ले डिफल्ट रूपमा पोर्ट ८०८० प्रयोग गर्दछ
# application.properties मा परिवर्तन:
server.port=8081
```

### बहुभाषी समर्थन
- ४५+ भाषामा स्वचालित अनुवादमार्फत कागजात उपलब्ध
- `translations/` निर्देशिकामा अनुवादहरू
- GitHub Actions workflow द्वारा अनुवाद व्यवस्थापन

### सिकाइ मार्ग
1. [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md) बाट सुरु गर्नुहोस्
2. अध्यायहरू क्रमशः अनुसरण गर्नुहोस् (01 → 05)
3. हरेक अध्यायमा व्यावहारिक उदाहरणहरू पूरा गर्नुहोस्
4. अध्याय ४ मा नमूना परियोजनाहरू अन्वेषण गर्नुहोस्
5. अध्याय ५ मा जिम्मेवार AI अभ्यासहरू सिक्नुहोस्

### विकास कन्टेनर
`.devcontainer/devcontainer.json` ले कन्फिगर गर्छ:
- Java 21 विकास वातावरण
- प्रि-इन्स्टल गरिएको Maven
- VS Code Java एक्सटेन्सनहरू
- Spring Boot उपकरणहरू
- GitHub Copilot एकीकरण
- Docker-in-Docker समर्थन
- Azure CLI

### प्रदर्शन विचारहरू
- Azure AI Foundry डिप्लोयमेन्टहरूमा प्रति मिनेट टोकन/अनुरोध सीमा हुन्छ
- एम्बेडिङका लागि उपयुक्त ब्याच साइजहरू प्रयोग गर्नुहोस्
- दोहोरिएका API कलहरूका लागि क्यासिङ विचार गर्नुहोस्
- लागत अनुकूलनका लागि टोकन प्रयोग निगरानी गर्नुहोस्

### सुरक्षा नोटहरू
- `.env` फाइलहरू कहिल्यै कमिट नगर्नुहोस् (पहिले नै `.gitignore` मा छन्)
- API कुञ्जीको सट्टा कुंजी बिना प्रमाणीकरण (Microsoft Entra ID) प्राथमिकता दिनुहोस्
- Azure मा व्यवस्थापित पहिचानहरू प्रयोग गर्नुहोस्; स्थानीय विकासका लागि `az login` प्रयोग
- अध्याय ५ मा जिम्मेवार AI निर्देशनहरू पालना गर्नुहोस्

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**अस्वीकरण**:
यो दस्तावेज़ AI अनुवाद सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) प्रयोग गरेर अनुवाद गरिएको हो। हामी सही हुन प्रयास गर्छौं, तर कृपया जानकार हुनुस् कि स्वचालित अनुवादमा त्रुटिहरू वा अशुद्धताहरू हुन सक्छन्। मूल दस्तावेज़ यसको मूल भाषामा आधिकारिक स्रोत मानिनुपर्छ। महत्वपूर्ण जानकारीका लागि व्यावसायिक मानव अनुवाद सिफारिस गरिन्छ। यस अनुवादको प्रयोगबाट उत्पन्न कुनै पनि गलत बुझाइ वा त्रुटिको लागि हामी जिम्मेवार छैनौं।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->