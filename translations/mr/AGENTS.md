# AGENTS.md

## प्रकल्पाचा आढावा

हा जावा वापरून जनरेटिव्ह AI विकास शिकण्यासाठी शैक्षणिक रिपॉझिटरी आहे. यात मोठ्या भाषा मॉडेल्स (LLMs), प्रॉम्प्ट इंजिनिअरिंग, एम्बेडिंग्ज, RAG (रिट्रीव्हल-अग्मेंटेड जनरेशन), आणि मॉडेल कंटेक्स्ट प्रोटोकॉल (MCP) यांचा समावेश असलेला सर्वसमावेशक हॅण्ड्स-ऑन कोर्स दिला आहे.

**महत्त्वाच्या तंत्रज्ञान:**  
- जावा 21  
- स्प्रिंग बूट 3.5.x  
- स्प्रिंग AI 1.1.x  
- मेव्हन  
- LangChain4j  
- Azure AI Foundry, Azure OpenAI, आणि OpenAI SDKs

**आर्किटेक्चर:**  
- प्रकरणांनुसार आयोजित केलेली अनेक स्वतंत्र स्प्रिंग बूट अॅप्लिकेशन्स  
- वेगवेगळ्या AI पॅटर्न्सचे उदाहरण दाखवणारे सॅम्पल प्रोजेक्ट्स  
- प्री-कॉन्फिगर केलेल्या डेव्ह कंटेनर्ससह GitHub Codespaces-रेडी

## सेटअप कमांड्स

### मागणी  
- जावा 21 किंवा त्यापेक्षा वरचा  
- मेव्हन 3.x  
- Azure AI Foundry मॉडेल तैनातीसह Azure सदस्यता (प्रोव्हिजनसाठी `azd up`)  
- Azure CLI (`az`) आणि Azure Developer CLI (`azd`), कीलेस प्रमाणीकरणासाठी साइन इन केलेले

### वातावरण सेटअप

**पर्याय 1: GitHub Codespaces (शिफारसीय)**  
```bash
# रिपॉझिटरी फोर्क करा आणि GitHub UI कडून कोडस्पेस तयार करा
# देव कंटेनर स्वयंचलितपणे सर्व अवलंबित्वे इंस्टॉल करेल
# पर्यावरण सेटअपसाठी सुमारे २ मिनिटे प्रतीक्षा करा
```
  
**पर्याय 2: स्थानिक डेव्ह कंटेनर**  
```bash
# रेपॉजिटरी क्लोन करा
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Dev Containers विस्तारासह VS Code मध्ये उघडा
# विचारल्यावर कंटेनरमध्ये पुन्हा उघडा
```
  
**पर्याय 3: स्थानिक सेटअप**  
```bash
# अवलंबित्वे स्थापित करा
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# स्थापना सत्यापित करा
java -version
mvn -version
```
  
### कॉन्फिगरेशन

**Azure AI Foundry सेटअप (कीलेस, शिफारसीय):**  
```bash
# फाउंड्री खाते + मॉडेल तैनाती कोड म्हणून पुरवठा करा
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd आपल्या एंडपॉइंटसह (की नाही) examples/basic-chat-azure/.env लिहिते
```
  
**मॅन्युअल एन्डपॉइंट कॉन्फिग:**  
```bash
# जर आपण azd वापरला नसेल, तर एन्डपॉइंट स्वतः सेट करा (auth az login द्वारे कीलेस राहते)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# .env संपादित करा: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```
  
## विकास कार्यप्रवाह

### प्रकल्प रचना  
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
  
### अॅप्लिकेशन्स चालविणे

**स्प्रिंग बूट अॅप्लिकेशन चालविणे:**  
```bash
cd [project-directory]
mvn spring-boot:run
```
  
**प्रकल्प बिल्ड करणे:**  
```bash
cd [project-directory]
mvn clean install
```
  
**MCP कॅल्क्युलेटर सर्व्हर सुरू करणे:**  
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# सर्व्हर http://localhost:8080 वर चालतो
```
  
**क्लायंट उदाहरणे चालविणे:**  
```bash
# दुसऱ्या टर्मिनलमध्ये सर्व्हर सुरू केल्यानंतर
cd 04-PracticalSamples/calculator

# थेट MCP क्लायंट
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# AI-चालित क्लायंट (AZURE_OPENAI_ENDPOINT + az login आवश्यक)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# संवादात्मक बॉट
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```
  
### हट रीलोड  
जे प्रकल्प हट रीलोड सपोर्ट करतात त्यात Spring Boot DevTools समाविष्ट आहे:  
```bash
# जावा फाईल्समध्ये बदल जतन करताच आपोआप पुन्हा लोड होतील
mvn spring-boot:run
```
  
## चाचणी सूचना

### चाचण्या चालविणे

**प्रकल्पातील सर्व चाचण्या चालवा:**  
```bash
cd [project-directory]
mvn test
```
  
**विडंबनशील आउटपुटसह चाचण्या चालवा:**  
```bash
mvn test -X
```
  
**विशिष्ट चाचणी वर्ग चालवा:**  
```bash
mvn test -Dtest=CalculatorServiceTest
```
  
### चाचणी रचना  
- चाचणी फाइल्स JUnit 5 (Jupiter) वापरतात  
- चाचणी वर्ग `src/test/java/` मध्ये असतात  
- कॅल्क्युलेटर प्रकल्पातील क्लायंट उदाहरणे `src/test/java/com/microsoft/mcp/sample/client/` मध्ये असतात  

### मॅन्युअल चाचणी  
अनेक उदाहरणे इंटरऐक्टिव्ह अॅप्लिकेशन्स आहेत ज्यांना मॅन्युअल चाचणीची गरज असते:  

1. `mvn spring-boot:run` या कमांडने अॅप्लिकेशन सुरू करा  
2. एन्डपॉइंट्सची चाचणी किंवा CLI शी संवाद करा  
3. प्रत्येक प्रकल्पाच्या README.md मध्ये असलेल्या दस्तऐवजांशी अपेक्षित वर्तन जुळवून पाहा  

### Azure AI Foundry सह चाचणी  
- उदाहरणे चालविण्यापूर्वी `az login` ने साइन इन करा (कीलेस प्रमाणीकरण)  
- खात्याकडे Cognitive Services OpenAI User भूमिका त्या रिसोर्सवर आहे याची खात्री करा  
- कलम 5 मधील जबाबदार AI उदाहरणाने सामग्री फिल्टरिंगची चाचणी करा  

## कोड शैली मार्गदर्शक

### जावा परंपरा  
- **जावा आवृत्ती:** आधुनिक वैशिष्ट्यांसह जावा 21  
- **शैली:** मानक जावा परंपरांचे पालन करा  
- **नावकरण:**  
  - वर्ग: PascalCase  
  - मेथड/व्हेरिएबल्स: camelCase  
  - स्थिरांक: UPPER_SNAKE_CASE  
  - पॅकेज नावे: लहान अक्षरे  

### स्प्रिंग बूट पॅटर्न्स  
- व्यावसायिक लॉजिकसाठी `@Service` वापरा  
- REST एन्डपॉइंटसाठी `@RestController` वापरा  
- `application.yml` किंवा `application.properties` मधून कॉन्फिगरेशन  
- हार्ड-कोडेड मूल्यांऐवजी पर्यावरण चल वापरा  
- MCP एक्सपोज्ड मेथडसाठी `@Tool` अॅनोटेशन वापरा  

### फाईल संघटन  
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
  
### अवलंबनीयता  
- मेव्हन `pom.xml` द्वारे व्यवस्थापित  
- वर्शन व्यवस्थापनासाठी स्प्रिंग AI BOM  
- AI एकत्रिकरणासाठी LangChain4j  
- स्प्रिंग अवलंबनांसाठी स्प्रिंग बूट स्टार्टर पॅरेन्ट  

### कोड टिप्पणी  
- सार्वजनिक API साठी JavaDoc जोडा  
- क्लिष्ट AI संवादासाठी स्पष्टीकरणात्मक टिप्पण्या जोडा  
- MCP टूल वर्णने स्पष्टपणे दस्तऐवज करा  

## बिल्ड आणि तैनाती

### प्रकल्प बिल्ड करणे

**चाचण्या न करता बिल्ड करा:**  
```bash
mvn clean install -DskipTests
```
  
**सर्व तपासण्या सह बिल्ड करा:**  
```bash
mvn clean install
```
  
**अॅप्लिकेशन पॅकेजिंग:**  
```bash
mvn package
# target/ निर्देशिकेत JAR तयार करतो
```
  
### आउटपुट निर्देशिका  
- संकलित वर्ग: `target/classes/`  
- चाचणी वर्ग: `target/test-classes/`  
- JAR फायली: `target/*.jar`  
- मेव्हन आर्टिफॅक्ट्स: `target/`  

### वातावरण-विशिष्ट कॉन्फिगरेशन  

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
- कीलेस प्रमाणीकरणासाठी `az login` ऐवजी व्यवस्थापित ओळखीचा वापर करा  
- उत्पादन Foundry रिसोर्सवर `AZURE_OPENAI_ENDPOINT` ने निर्देश करा  
- पर्यावरण चल किंवा Azure Key Vault द्वारे कॉन्फिगरेशन व्यवस्थापित करा  

### तैनाती विचार  
- हे शैक्षणिक रिपॉझिटरी असून सॅम्पल अॅप्लिकेशन्स आहेत  
- थेट उत्पादनासाठी तैनात करण्यासाठी डिझाइन केलेले नाही  
- सॅम्पल प्रोडक्शन वापरासाठी पॅटर्न दाखवतात  
- विशिष्ट तैनाती नोट्ससाठी प्रकल्पांच्या README पहा  

## अतिरिक्त टिपा

### Azure AI Foundry  
- **कीलेस प्रमाणीकरण:** Microsoft Entra ID शी कनेक्ट करा — API की व्यवस्थापित करण्याची गरज नाही  
- **कोड म्हणून प्रोव्हिजन:** Bicep + azd (`azd up`) द्वारे अकाउंट आणि मॉडेल तैनाती तयार होते  
- स्थानिक (`az login`) आणि Azure (व्यवस्थापित ओळख) दोन्ही ठिकाणी समान OpenAI-सुसंगत कोड चालतो  

### अनेक प्रकल्पांवर काम करणे  
प्रत्येक सॅम्पल प्रकल्प स्वतंत्र आहे:  
```bash
# विशिष्ट प्रकल्पाकडे नेव्हिगेट करा
cd 04-PracticalSamples/[project-name]

# प्रत्येकाचा स्वतःचा pom.xml असतो आणि स्वतंत्रपणे तयार केला जाऊ शकतो
mvn clean install
```
  
### सामान्य समस्या

**जावा आवृत्ती जुळत नाही:**  
```bash
# Java 21 सत्यापित करा
java -version
# आवश्यकता असल्यास JAVA_HOME अद्यतनित करा
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```
  
**डिपेंडेंसी डाउनलोड समस्या:**  
```bash
# Maven कॅशे साफ करा आणि पुन्हा प्रयत्न करा
rm -rf ~/.m2/repository
mvn clean install
```
  
**एन्डपॉइंट किंवा साइन-इन सापडले नाही:**  
```bash
# चालू सत्रात एंडपॉइंट सेट करा आणि साइन इन करा (कीलेस)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# किंवा प्रोजेक्ट निर्देशिकेत .env फाइल वापरा
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```
  
**पोर्ट आधीच वापरात आहे:**  
```bash
# स्प्रिंग बूट डीफॉल्टने पोर्ट ८०८० वापरतो
# application.properties मध्ये बदल:
server.port=8081
```
  
### बहुभाषिक समर्थन  
- ऑटोमेटेड अनुवादाद्वारे 45+ भाषांमध्ये दस्तऐवज उपलब्ध  
- अनुवाद `translations/` फोल्डरमध्ये साठवलेले  
- GitHub Actions वर्कफ्लोने अनुवाद व्यवस्थापित  

### शिकण्याचा मार्ग  
1. [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md) पासून सुरुवात करा  
2. प्रकरणे क्रमाने फॉलो करा (01 → 05)  
3. प्रत्येक प्रकरणातील हॅण्ड्स-ऑन उदाहरणे पूर्ण करा  
4. प्रकरण 4 मधील सॅम्पल प्रकल्प एक्सप्लोर करा  
5. प्रकरण 5 मध्ये जबाबदार AI पद्धती शिका  

### विकास कंटेनर  
`.devcontainer/devcontainer.json` मध्ये खालील कॉन्फिगरेशन आहे:  
- जावा 21 विकास वातावरण  
- मेव्हन प्री-इंस्टॉल्ड  
- VS कोड जावा एक्सटेन्शन्स  
- स्प्रिंग बूट टूल्स  
- GitHub Copilot इंटिग्रेशन  
- Docker-in-Docker सपोर्ट  
- Azure CLI  

### परफॉर्मन्स विचार  
- Azure AI Foundry तैनातींना मिनिटाला टोकन/विनंती मर्यादा असते  
- एम्बेडिंगसाठी योग्य बॅच साइज वापरा  
- पुनरावृत्ती API कॉलसाठी कॅशिंगचा विचार करा  
- खर्च ऑप्टिमायझेशनसाठी टोकन वापर मॉनिटर करा  

### सुरक्षा टिपा  
- `.env` फायली कधीही कमिट करू नका (हे `.gitignore` मध्ये आहे)  
- API कींपेक्षा कीलेस प्रमाणीकरण (Microsoft Entra ID) प्राधान्य द्या  
- Azure मध्ये व्यवस्थापित ओळखी वापरा; स्थानिक विकासासाठी `az login`  
- प्रकरण 5 मधील जबाबदार AI मार्गदर्शकाचा वापर करा

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**अस्वीकरण**:
हा दस्तऐवज AI भाषांतर सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) चा वापर करून अनुवादित केला आहे. जरी आम्ही अचूकतेसाठी प्रयत्न करतो, तरी कृपया लक्षात घ्या की स्वयंचलित भाषांतरांमध्ये त्रुटी किंवा अचूकतेची कमतरता असू शकते. मूळ दस्तऐवज त्याच्या मूळ भाषेत अधिकृत स्रोत मानला पाहिजे. महत्त्वाची माहिती असल्यास, व्यावसायिक मानवी भाषांतराची शिफारस केली जाते. या भाषांतराच्या वापरामुळे उद्भवणाऱ्या कोणत्याही गैरसमज किंवा चुकीच्या अर्थलावणीसाठी आम्ही जबाबदार नाही.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->