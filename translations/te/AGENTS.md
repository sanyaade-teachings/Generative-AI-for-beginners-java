# AGENTS.md

## ప్రాజెక్ట్ అవలోకనం

Java తో Generative AI అభివృద్ధిని నేర్చుకునేందుకు ఇది ఒక విద్యా గోదామా. ఇది Large Language Models (LLMs), prompt engineering, embeddings, RAG (Retrieval-Augmented Generation), మరియు Model Context Protocol (MCP) కవర్ చేసే సుదీర్ఘ హ్యాండ్స్-ఆన్ కోర్సును అందిస్తుంది.

**ప్రధాన సాంకేతికతలు:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI, మరియు OpenAI SDKs

**ఆర్కిటెక్చర్:**
- అధ్యాయాల వారీగా సమకూర్చబడిన బహువిధ సొంత Spring Boot అప్లికేషన్లు
- వివిధ AI నమూనాలను డెమోonstrేషన్ చేసే నమూనా ప్రాజెక్టులు
- GitHub Codespaces-కి సిద్ధంగా, ప్రీ-కన్ఫిగర్ చేసిన డెవ్ కంటెయినర్లు

## సెటప్ కమాండ్లు

### ప్రీరిక్విజిట్స్
- Java 21 లేదా అంతకంటే పై версия
- Maven 3.x
- Azure AI Foundry మోడల్ డిప్లాయ్‌మెంట్ ఉన్న Azure సబ్‌స్క్రిప్షన్ (ప్రొవిజన్ చేయడానికి `azd up`)
- Azure CLI (`az`) మరియు Azure Developer CLI (`azd`), కీ లేని ఆథెంటికేషన్ సిగ్నిన్

### ఎన్విరాన్‌మెంట్ సెటప్

**ఎంపిక 1: GitHub Codespaces (సిఫార్సు చేయబడింది)**
```bash
# రిపాజిటరీని ఫోర్క్ చేసి GitHub UI నుండి కోడ్స్‌పేస్‌ను సృష్టించండి
# డెవలప్మెంట్ కంటెయినర్ అన్ని ఆధారాలను స్వయంచాలకంగా ఇన్‌స్టాల్ చేస్తుంది
# పర్యావరణం సెట్‌అప్ కోసం సగటున సుమారు 2 నిమిషాలు వేచి ఉండండి
```

**ఎంపిక 2: లోకల్ డెవ్ కంటెయినర్**
```bash
# రిపాజిటరీని క్లోన్ చేయండి
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Dev Containers విస్తరణతో VS కోడ్‌లో తెరవండి
# సూచించినప్పుడు కంటైనర్‌లో మళ్లీ తెరవండి
```

**ఎంపిక 3: లోకల్ సెటప్**
```bash
# డిపెండెన్సీలను ఇన్‌స్టాల్ చేయండి
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# ఇన్‌స్టలేషన్‌ను నిర్ధారించండి
java -version
mvn -version
```


### కాన్ఫిగరేషన్

**Azure AI Foundry సెటప్ (కీలెస్, సిఫార్సు):**
```bash
# Foundry ఖాతా + మోడల్ డిప్లాయ్‌మెంట్‌లను కోడ్‌గా అందించండి
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd మీ ఎండ్‌పాయింట్ (కీ లేకుండా)తో examples/basic-chat-azure/.env రాసుతుంది
```

**మాన్యువల్ ఎండ్‌పాయింట్ కాన్ఫిగ్:**
```bash
# మీరు azd ఉపయోగించకపోతే, ఎండ్పాయింట్‌ను మీరు స్వయంగా సెట్ చేయండి (auth az login ద్వారా keylessగా ఉంటుంది)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# .env ను సవరించండి: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```


## అభివృద్ధి వర్క్‌ఫ్లో

### ప్రాజెక్ట్ నిర్మాణం
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


### అప్లికేషన్లు నడిపించడం

**Spring Boot అప్లికేషన్ నడపడం:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**ప్రాజెక్ట్ నిర్మించడం:**
```bash
cd [project-directory]
mvn clean install
```

**MCP కాలిక్యులేటర్ సర్వర్ ప్రారంభించడం:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# సర్వర్ http://localhost:8080 వద్ద চলుతుంది
```

**క్లయింట్ ఉదాహరణలు నడిపించడం:**
```bash
# ఇంకొక టెర్మినల్‌లో సర్వర్ ప్రారంభించిన తర్వాత
cd 04-PracticalSamples/calculator

# డైరెక్ట్ MCP క్లయింట్
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# AI ఆధారిత క్లయింట్ (AZURE_OPENAI_ENDPOINT + az లాగిన్ అవసరం)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# ఇంటరాక్టివ్ బాట్
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```


### హాట్ రీలోడ్
హాట్ రీలోడ్ మద్దతు ఉన్న ప్రాజెక్టుల్లో Spring Boot DevTools ని చేర్చారు:
```bash
# Java ఫైళ్లలో చేసిన మార్పులు దాచినప్పుడు ఆటోమేటిక్‌గా రీలోడ్ చేయబడతాయి
mvn spring-boot:run
```


## పరీక్ష సూచనలు

### పరీక్షలు నడిపించడం

**ప్రాజెక్ట్లో అన్ని పరీక్షలు నడపండి:**
```bash
cd [project-directory]
mvn test
```

**వెర్బోస్ అవుట్పుట్‌తో పరీక్షలు నడిపించండి:**
```bash
mvn test -X
```

**కారణిక పరీక్ష తరగతిని నడిపించండి:**
```bash
mvn test -Dtest=CalculatorServiceTest
```


### పరీక్ష నిర్మాణం
- పరీక్ష ఫైల్స్ JUnit 5 (Jupiter) ఉపయోగిస్తాయి
- పరీక్ష తరగతులు `src/test/java/` లో ఉంటాయి
- కాలిక్యులేటర్ ప్రాజెక్ట్‌లో క్లయింట్ ఉదాహరణలు `src/test/java/com/microsoft/mcp/sample/client/` లో ఉంటాయి

### మాన్యువల్ పరీక్ష
చాలా ఉదాహరణలు మాన్యువల్ పరీక్ష అవసరమయ్యే ఇంటరాక్టివ్ అప్లికేషన్లుగా ఉంటాయి:

1. `mvn spring-boot:run` తో అప్లికేషన్ ప్రారంభించండి
2. ఎండ్‌పాయింట్లను పరీక్షించండి లేదా CLI తో ఇంటరాక్ట్ అవ్వండి
3. ప్రస్థావనలోని README.md లోని డాక్యుమెంటేషన్ తో సరిపోయే విధంగా ప్రవర్తనను ధృవీకరించండి

### Azure AI Foundryతో పరీక్ష
- ఉదాహరణలు నడుపుటకు ముందు `az login` చేయండి (కీలెస్ ఆథెంట్)
- మీ అకౌంట్‌కు Cognitive Services OpenAI User పాత్ర ఉండాలి
- 5వ అధ్యాయంలో ఉన్న రిస్పాన్సిబుల్ AI ఉదాహరణతో కంటెంట్ ఫిల్టర్ టెస్టింగ్ చేయండి

## కోడ్ శైలి మార్గదర్శకాలు

### Java నిబంధనలు
- **Java వెర్షన్:** ఆధునిక ఫీచర్లతో Java 21
- **వైపు:** సాంప్రదాయ Java మార్గదర్శకాలు అనుసరించండి
- **పేరు పద్ధతి:** 
  - తరగతులు: PascalCase
  - పద్ధతులు/చరాలు: camelCase
  - కాన్స్టెంట్లు: UPPER_SNAKE_CASE
  - ప్యాకేజ్ పేర్లు: చిన్న అక్షరాలు

### Spring Boot నమూనాలు
- వ్యాపార లాజిక్ కోసం `@Service` ఉపయోగించండి
- REST ఎండ్‌పాయింట్లకు `@RestController` ఉపయోగించండి
- కన్ఫిగరేషన్ కోసం `application.yml` లేదా `application.properties`
- హార్డ్ కోడ్ కాకుండా ఎన్విరాన్‌మెంట్ వేరియబుల్స్ ప్రాధాన్యం
- MCP మెథడ్లకు `@Tool` అనోటేషన్ ఉపయోగించండి

### ఫైల్ స organização
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


### డిపెండెన్సీలు
- Maven `pom.xml` ద్వారా నిర్వహించబడుతుంది
- సాంప్రదాయ ప్రబంధం కోసం Spring AI BOM
- AI ఇంటిగ్రేషన్లకు LangChain4j
- Spring డిపెండెన్సీలకు Spring Boot స్టార్టర్ పెరెంటు

### కోడ్ వ్యాఖ్యలు
- పబ్లిక్ APIలకు JavaDoc జత చేయండి
- సంక్లిష్ట AI ఇంటరాక్షన్లకు వివరణాత్మక వ్యాఖ్యలు జత చేయండి
- MCP టూల్ వివరణలను స్పష్టంగా డాక్యుమెంటేషన్ చేయండి

## బిల్డ్ మరియు డిప్లాయ్‌మెంట్

### ప్రోజెక్ట్లను బిల్డ్ చేయడం

**పరీక్షలు లేకుండా బిల్డ్ చేయండి:**
```bash
mvn clean install -DskipTests
```

**అన్ని చెక్లతో బిల్డ్ చేయండి:**
```bash
mvn clean install
```

**ఆప్లికేషన్ ప్యాకేజింగ్:**
```bash
mvn package
# target/ డైరెక్టరీలో JAR ని సృష్టిస్తుంది
```


### అవుట్పుట్ డైరెక్టరీలు
- కంపైల్ చేసిన తరగతులు: `target/classes/`
- టెస్ట్ తరగతులు: `target/test-classes/`
- జార్ ఫైల్స్: `target/*.jar`
- Maven ఆర్టిఫాక్ట్స్: `target/`

### ప్రత్యేక ఎన్విరాన్‌మెంటు కాన్ఫిగరేషన్

**డెవలప్‌మెంట్:**
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

**ప్రొడక్షన్:**
- కీ లేని ఆన్లైన్ ఆథెంటికేషన్ కోసం `az login` కాకుండా Managed Identity ఉపయోగించండి
- `AZURE_OPENAI_ENDPOINT` ను ప్రొడక్షన్ Foundry రిసోర్స్‌కు సూచించండి
- కాన్ఫిగరేషన్‌ను ఎన్విరాన్‌మెంట్ వేరియబుల్స్ లేదా Azure Key Vault ద్వారా నిర్వహించండి

### డిప్లాయ్‌మెంట్ పరిగణనలు
- ఇది ఒక విద్యా గోదామా, నమూనా అప్లికేషన్లతో కూడి ఉంటుంది
- నేరుగా ప్రొడక్షన్‌కు ఉపయోగించవలసినది కాదు
- నమూనాలు ప్రొడక్షన్ వినియోగానికి తగిన నమూనాలను ప్రదర్శిస్తాయి
- ప్రత్యేక డిప్లాయ్‌మెంట్ గమనికలకు ప్రాజెక్ట్ README చూడండి

## అదనపు గమనికలు

### Azure AI Foundry
- **కీలెస్ ఆథెంటికేషన్:** Microsoft Entra IDతో కనెక్ట్ అవ్వండి — API కీలు అవసరం లేదు
- **కోడ్‌గా ప్రొవిజనింగ్:** Bicep + azd (`azd up`) ఖాతా మరియు మోడల్ డిప్లాయ్‌మెంట్‌లను సృష్టిస్తుంది
- అది OpenAI అనుకూల కోడ్ స్థానికంగా (`az login`) మరియు Azureలో (Managed Identity) నడుస్తుంది

### బహుళ ప్రాజెక్ట్‌లతో పని
ప్రతి నమూనా ప్రాజెక్ట్ స్వతంత్రంగా ఉంటుంది:
```bash
# ప్రత్యేక ప్రాజెక్ట్ వైపు నావిగేట్ చేయండి
cd 04-PracticalSamples/[project-name]

# ప్రతి ఒక్కదానికి దాని స్వంత pom.xml ఉంటుంది మరియు స్వతంత్రంగా నిర్మించవచ్చు
mvn clean install
```


### సాధారణ సమస్యలు

**Java వెర్షన్ సరిపోకపోవడం:**
```bash
# జావా 21ను సరిచూడండి
java -version
# అవసరమైతే JAVA_HOMEని నవీకరించండి
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**డిపెండెన్సీ డౌన్లోడ్ సమస్యలు:**
```bash
# మావెన్ కేచేను సుపష్టం చేసి మళ్ళీ ప్రయత్నించండి
rm -rf ~/.m2/repository
mvn clean install
```

**ఎండ్‌పాయింట్ లేదా సైన్-ఇన్ దొరకడం లేదు:**
```bash
# ప్రస్తుత సెషన్‌లో ఎండ్‌పాయింట్‌ను సెట్ చేసి సైన్ ఇన్ అవ్వండి (కీ లెస్)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# లేదా ప్రాజెక్ట్ డైరెక్టరీలో .env ఫైల్ ఉపయోగించండి
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**పోర్ట్ ఇప్పటికే ఉపయోగంలో ఉంది:**
```bash
# Spring Boot డిఫాల్ట్‌గా పోర్ట్ 8080 ను ఉపయోగిస్తుంది
# application.properties లో మార్పు:
server.port=8081
```


### బహుభాషా మద్దతు
- 45కి పైగా భాషలలో ఆటోమేటెడ్ అనువాదాలతో డాక్యుమెంటేషన్ అందుబాటులో ఉంది
- అనువాదాలు `translations/` డైరెక్టరీలో ఉంటాయి
- అనువాదం GitHub Actions వర్క్‌ఫ్లో ద్వారా నిర్వహించబడుతుంది

### అభ్యాస మార్గం
1. [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md) నుండి ప్రారంభించండి
2. అధ్యాయాలను వరుసగా అనుసరించండి (01 → 05)
3. ప్రతి అధ్యాయంలోని హ్యాండ్స్-ఆన్ ఉదాహరణలను పూర్తి చేసుకోండి
4. 4వ అధ్యాయంలో నమూనా ప్రాజెక్టులను అన్వేషించండి
5. 5వ అధ్యాయంలో రిస్పాన్సిబుల్ AI ప్రాక్టీస్‌లను తెలుసుకోండి

### అభివృద్ధి కంటెయినర్
`.devcontainer/devcontainer.json` ఈ క్రింది వాటిని కంఫిగర్ చేస్తుంది:
- Java 21 అభివృద్ధి ఎన్విరాన్‌మెంట్
- Maven ప్రీ-ఇన్స్టాల్‌డ్
- VS Code Java ఎక్స్‌టెన్షన్స్
- Spring Boot టూల్స్
- GitHub Copilot సమీకరణ
- Docker-in-Docker మద్దతు
- Azure CLI

### పనితీరు పరిగణనలు
- Azure AI Foundry డిప్లాయ్‌మెంట్‌లకు నిమిషానికి టోకెన్/రివక్వెస్ట్ పరిమితులు ఉంటాయి
- embeddings కోసం సరైన బ్యాచ్ పరిమాణాలు ఉపయోగించండి
- పునరావృత API కాల్స్ కోసం క్యాచింగ్ పరిగణించండి
- ఖర్చు సవరణ కోసం టోకెన్ వినియోగాన్ని ట్రాక్ చేయండి

### భద్రత గమనికలు
- ఎప్పుడూ `.env` ఫైల్స్ కమిట్ చేయకండి (ఇవి ఇప్పటికే `.gitignore` లో ఉన్నాయి)
- API కీలు కంటే కీలెస్ ఆథెన్ (Microsoft Entra ID) ప్రాధాన్యం ఇవ్వండి
- Azureలో Managed Identities ఉపయోగించండి; లోకల్ అభివృద్ధికి `az login`
- 5వ అధ్యాయంలో రిస్పాన్సిబుల్ AI మార్గదర్శకాలను అనుసరించండి

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**అస్వీకరణ**:
ఈ పత్రం AI అనువాద సేవ [Co-op Translator](https://github.com/Azure/co-op-translator) ఉపయోగించి అనువదించబడింది. మేము ఖచ్చితత్వానికి ప్రయత్నిస్తున్నప్పటికీ, ఆటోమేటెడ్ అనువాదాలు తప్పులు లేదా అసమగ్రతలను కలిగి ఉండవచ్చు. దాని స్వదేశ భాషలో ఉన్న అసలు పత్రాన్ని అధికారం కలిగిన మూలంగా పరిగణించాలి. కీలకమైన సమాచారం కోసం, ప్రొఫెషనల్ మానవ అనువాదాన్ని సిఫారసు చేస్తాము. ఈ అనువాదం ఉపయోగం వల్ల కలిగే ఏవైనా అపార్థాలు లేదా తప్పుదారులు కోసం మేము బాధ్యత వహించము.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->