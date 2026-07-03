# AGENTS.md

## Project Overview

ఇది Java తో Generative AI అభివృద్ధిని నేర్చుకునేందుకు విద్యా గోపురం. ఇది Large Language Models (LLMs), prompt engineering, embeddings, RAG (Retrieval-Augmented Generation), మరియు Model Context Protocol (MCP) లను kapsule చేసుకునే సమగ్ర hands-on కోర్సును అందిస్తుంది.

**ప్రధాన సాంకేతికతలు:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI, మరియు OpenAI SDKs

**విన್ಯಾಸం:**
- అദ്ധాయాల వారీగా క్రమబద్ధీకరించిన బహుళ స్వతంత్ర Spring Boot అప్లికేషన్లు
- వివిధ AI నమూనాలను ప్రదర్శించే నమూనా ప్రాజెక్టులు
- ముందుగా కాన్ఫిగర్ చేసిన dev కంటైనర్లతో GitHub Codespaces-సిద్దంగా

## Setup Commands

### Prerequisites
- Java 21 లేదా అంతకన్నా ఎక్కువ
- Maven 3.x
- Azure AI Foundry మోడల్ నియామకంతో Azure సబ్‌స్క్రిప్షన్ (`azd up` తో ప్రావిజన్ చేయండి)
- Azure CLI (`az`) మరియు Azure Developer CLI (`azd`), కీలెస్ ప్రామాణీకరణ కోసం సైన్ ఇన్

### Environment Setup

**ఎంపిక 1: GitHub Codespaces (సిఫార్సు):**
```bash
# రిపోజిటరీని ఫోర్క్ చేసి GitHub UI నుండి కోడ్స్పేస్ సృష్టించండి
# డెవలప్మెంట్ కంటైనర్ ఆటోమేటిగ్గా అన్ని డిపెండెన్సీలను ఇన్‌స్టాల్ చేస్తుంది
# వాతావరణ సెటప్ కోసం సుమారు ~2 నిమిషాలు కండి
```

**ఎంపిక 2: లోకల్ డెవ్ కంటైనర్**
```bash
# రిపోజిటరీని క్లోన్ చేయండి
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# డెవ్ కంటైనర్లు ఎక్స్టెన్షన్ తో VS కోడ్ లో తెరవండి
# సూచించినప్పుడు కంటైనర్‌లో మళ్లీ తెరవండి
```

**ఎంపిక 3: లోకల్ సెటప్**
```bash
# ఆధారాలు ఇన్స్టాల్ చేయండి
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# ఇన్స్టాలేషన్‌ను ధృవీకరించండి
java -version
mvn -version
```

### Configuration

**Azure AI Foundry సెటప్ (కీలెస్, సిఫార్సు):**
```bash
# ఫౌండ్రీ ఖాతాను + మోడల్ మోపోతలను కోడ్‌గా సమకూర్చండి
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd आपके ఎండ్‌పాయింట్ (కీ లేకుండా) తో examples/basic-chat-azure/.env రాస్తుంది
```

**మానువల్ ఎండ్పాయింట్ కాన్ఫిగరేషన్:**
```bash
# మీరు azd వాడకపోతే, ఎండ్పాయింట్ ను మీరు మీరే సెట్ చేయండి (auth az login ద్వారా keylessగా ఉంటుంది)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# .env ని సవరించండి: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Development Workflow

### Project Structure
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

### Running Applications

**Spring Boot అప్లికేషన్ నడపడం:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**ప్రాజెక్ట్ నిర్మాణం:**
```bash
cd [project-directory]
mvn clean install
```

**MCP కాలిక్యులేటర్ సర్వర్ ప్రారంభించడం:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# సర్వర్ http://localhost:8080 పై నడుస్తోంది
```

**క్లయింట్ ఉదాహరణలు నడపడం:**
```bash
# మరో టెర్మినల్‌లో సర్వర్ ప్రారంభించిన తర్వాత
cd 04-PracticalSamples/calculator

# నేరుగా MCP క్లయింట్
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# AI-శక్తితో కూడిన క్లయింట్ (AZURE_OPENAI_ENDPOINT + az లాగిన్ అవసరం)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# పరస్పర కార్యకలాప బాట్
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### హాట్ రీలోడ్
హాట్ రీలోడ్ కి మద్దతు ఉన్న ప్రాజెక్టుల్లో Spring Boot DevTools చేర్చబడింది:
```bash
# జావా ఫైళ్లలో చేసిన మార్పులు సేవ్ చేసిన వెంటనే ఆటోమేటిక్‌గా రీలోడ్ అవుతాయి
mvn spring-boot:run
```

## Testing Instructions

### పరీక్షలు జరపడం

**ప్రాజెక్ట్ లో అన్ని పరీక్షలను నడపండి:**
```bash
cd [project-directory]
mvn test
```

**విస్తృత అవుట్‌పుట్‌తో పరీక్షలు నడపండి:**
```bash
mvn test -X
```

**నిర్దిష్ట పరీక్ష తరగతి నడపండి:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### పరీక్ష నిర్మాణం
- పరీక్ష ఫైళ్ళు JUnit 5 (Jupiter) ఉపయోగిస్తాయి
- పరీక్ష తరగతులు `src/test/java/` లో ఉంటాయి
- కాలిక్యులేటర్ ప్రాజెక్ట్‌లో క్లయింట్ ఉదాహరణలు `src/test/java/com/microsoft/mcp/sample/client/` లో ఉన్నాయి

### మానువల్ పరీక్ష
చాలా ఉదాహరణలు ఇంటరాక్టివ్ అప్లికేషన్లు కావడంతో మానువల్ పరీక్ష అవసరం:

1. `mvn spring-boot:run` తో అప్లికేషన్ ప్రారంభించండి
2. ఎండ్పాయింట్లను పరీక్షించండి లేదా CLI తో ఇంటరాక్ట్ చేయండి
3. ప్రతి ప్రాజెక్ట్ README.md లో ఉన్న డాక్యుమెంటేషన్ అనుసరించి ఆశించిన ప్రవర్తనను ధృవీకరించండి

### Azure AI Foundry తో పరీక్షించడం
- ఉదాహరణలు నడపటం ముందు `az login` తో సైన్ ఇన్ అవ్వండి (కీళెస్ ప్రామాణీకరణ)
- మీ ఖాతాకు Cognitive Services OpenAI User పాత్ర ఉన్నదని నిర్ధారించుకోండి
- చాప్టర్ 5 లోని బాధ్యత కలిగిన AI ఉదాహరణతో కంటెంట్ ఫిల్టర్ టెస్ట్ చేయండి

## Code Style Guidelines

### Java Conventions
- **Java వెర్షన్:** Java 21 ఆధునిక లక్షణాలతో
- **శైలి:** సాధారణ Java నియమాలను అనుసరించండి
- **పేరు付్లు:** 
  - తరగతులు: PascalCase
  - మెథడ్లు/వేరియబుల్స్: camelCase
  - స్థిరాంకాలు: UPPER_SNAKE_CASE
  - ప్యాకేజ్ పేర్లు: చిన్న అక్షరాలు

### Spring Boot నమూనాలు
- వ్యాపార లాజిక్ కోసం `@Service` వాడండి
- REST ఎండ్పాయింట్ల కోసం `@RestController` వాడండి
- కాన్ఫిగరేషన్ `application.yml` లేదా `application.properties` ద్వారా
- హార్డ్-కోడ్ చేసిన విలువల కంటే ఎన్విరాన్మెంట్ వేరియబుల్స్ ప్రాధాన్యం
- MCP ద్వారా ప్రదర్శించబడిన మెథడ్లకు `@Tool` అనోటేషన్ వాడండి

### ఫైల్ ఏర్పాటు
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

### ఆధారాలు
- Maven `pom.xml` ద్వారా నిర్వహించబడుతుంది
- వెర్షన్ నిర్వహణకు Spring AI BOM
- AI సమ్మిళితాల కోసం LangChain4j
- Spring ఆధారాల కోసం Spring Boot స్టార్టర్ ప్యారెంట్

### కోడ్ వ్యాఖ్యలు
- పబ్లిక్ APIలకు JavaDoc జోడించండి
- క్లిష్టమైన AI ఇంటరాక్షన్లకు వివరణాత్మక వ్యాఖ్యలు చేర్చండి
- MCP టూల్ వివరణలను స్పష్టంగా డాక్యుమెంట్ చేయండి

## Build and Deployment

### ప్రాజెక్టులు నిర్మించడం

**పరీక్షలు లేకుండా నిర్మించండి:**
```bash
mvn clean install -DskipTests
```

**అన్ని తనిఖీలు సహా నిర్మించండి:**
```bash
mvn clean install
```

**అప్లికేషన్ ప్యాకేజీ చేయడం:**
```bash
mvn package
# target/ డైరెక్టరీలో JAR సృష్టిస్తుంది
```

### ఔట్‌పుట్ డైరెక్టరీలు
- కంపైల్ చేసిన తరగతులు: `target/classes/`
- పరీక్ష తరగతులు: `target/test-classes/`
- JAR ఫైళ్లు: `target/*.jar`
- Maven ఆర్టిఫాక్టులు: `target/`

### పర్యావరణ-విశేష కాన్ఫిగరేషన్

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
- కీళెస్ ప్రామాణీకరణ కోసం `az login` స్థానంలో మేనేజ్‌డ్ ఐడెంటిటీ వాడండి
- `AZURE_OPENAI_ENDPOINT` ని మీ ప్రొడక్షన్ Foundry రిసోర్స్‌కు సెట్ చేయండి
- కాన్ఫిగరేషన్‌ని ఎన్విరాన్మెంట్ వేరియబుల్స్ లేదా Azure Key Vault ద్వారా నిర్వహించండి

### డిప్లాయ్‌మెంట్ పరిగణనలు
- ఇది విద్యా గోపురం స్వరూపంలో నమూనా అప్లికేషన్లు కలిగిన రిపోజిటరీ
- దీన్ని ప్రత్యక్ష ప్రొడషన్ డిప్లాయ్‌మెంట్ కోసం డిజైన్ చేయలేదు
- నమూనాలు ప్రొడక్షన్ ఉపయోగానికి అనుగుణంగా అంగీకరించడానికి నమూనాలు చూపిస్తాయి
- ప్రత్యేక ప్రాజెక్ట్ README లలో డిప్లాయ్‌మెంట్ గమనికలు చూడండి

## Additional Notes

### Azure AI Foundry
- **కీళెస్ ప్రామాణీకరణ:** Microsoft Entra ID తో కనెక్ట్ అవ్వండి — API కీలు అవసరం లేదు
- **కోడ్ రూపంలో ప్రావిజనింగ్:** Bicep + azd (`azd up`) తో ఖాతా మరియు మోడల్ నియామకాలు సృష్టించండి
- అదే OpenAI అనుకూల కోడ్ స్థానికంగా (`az login`) మరియు Azure (మేనేజ్‌డ్ ఐడెంటిటీ) లో నడుస్తుంది

### బహుళ ప్రాజెక్టులతో పని
ప్రతి నమూనా ప్రాజెక్ట్ స్వతంత్రంగా ఉంటుంది:
```bash
# నిర్దిష్ట ప్రాజెక్ట్‌కు నావిగేట్ చేయండి
cd 04-PracticalSamples/[project-name]

# ప్రతి ఒక్కదానికి తన pom.xml ఉంటుంది మరియు స్వతంత్రంగా నిర్మించవచ్చు
mvn clean install
```

### సాధారణ సమస్యలు

**Java వెర్షన్ మిళితం:**
```bash
# జావా 21ని నిర్ధారించండి
java -version
# అవసరమైతే JAVA_HOME‌ను నవీకరించండి
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**ఆధారం డౌన్‌లోడ్ సమస్యలు:**
```bash
# మావెన్ క్యాషే క్లియర్ చేసి మళ్ళీ ప్రయత్నించండి
rm -rf ~/.m2/repository
mvn clean install
```

**ఎండ్పాయింట్ లేదా సైన్-ఇన్ కనపడడం లేదు:**
```bash
# ప్రస్తుత సెషన్లో ఎండ్‌పాయింట్‌ని సెట్ చేసి సైన్ ఇన్ చేయండి (పాస్‌వర్డ్ అవసరం లేదు)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# లేదా ప్రాజెక్ట్ డైరెక్టరీలో ఉన్న .env ఫైల్‌ను ఉపయోగించండి
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**పోర్ట్ ఇప్పటికే ఉపయోగంలో ఉంది:**
```bash
# Spring Boot డిఫాల్టుగా పోర్ట్ 8080 ను ఉపయోగిస్తుంది
# application.properties లో మార్పు:
server.port=8081
```

### బహుభాషా మద్దతు
- 45+ భాషలలో ఆటోమేటెడ్ అనువాదం ద్వారా డాక్యుమెంటేషన్ అందుబాటులో ఉంది
- అనువాదాలు `translations/` డైరెక్టరీలో గలవి
- అనువాదాన్ని GitHub Actions వర్క్‌ఫ్లో నిర్వహిస్తుంది

### విద్యాపథం
1. [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md) తో ప్రారంభించండి
2. అధ్యాయాలను క్రమం తప్పకుండా అనుసరించండి (01 → 05)
3. ప్రతి అధ్యాయంలో హ్యాండ్స్ ఆన్లు పూర్తి చేయండి
4. ఛాప్టర్ 4 లోని నమూనా ప్రాజెక్టులు పరిశీలించండి
5. ఛాప్టర్ 5 లో బాధ్యత కలిగిన AI సాధనాలు నేర్చుకోండి

### డెవలప్‌మెంట్ కంటైనర్
`.devcontainer/devcontainer.json` కింది అంశాలను సెట్ చేస్తుంది:
- Java 21 అభివృద్ధి వాతావరణం
- Maven ముందుగానే ఇన్స్టాల్
- VS Code Java ఎక్స్‌టెన్షన్లు
- Spring Boot టూల్స్
- GitHub Copilot ఇంటిగ్రేషన్
- Docker-in-Docker మద్దతు
- Azure CLI

### ప్రదర్శన పరిగణనలు
- Azure AI Foundry నియామకాలకు ప్రతిమినిట్ టోకెన్/అనుకుని పరిమితులు ఉన్నాయి
- embeddings కి సరైన బ్యాచ్ సైజులను వాడండి
- పునఃరావృత API కాల్స్ కోసం క్యాషింగ్ పరిగణించండి
- ఖర్చు ఆప్టిమైజేషన్ కోసం టోకెన్ వినియోగాన్ని సక్రమంగా పర్యవేక్షించండి

### భద్రతా గమనికలు
- `.env` ఫైళ్లను ఎప్పుడూ కమిట్ చేయవద్దు (`.gitignore` లో ఇప్పటికే ఉంది)
- API కీ కంటే కీళెస్ ప్రామాణీకరణ (Microsoft Entra ID) ను ప్రాధాన్యం ఇవ్వండి
- Azure లో మేనేజ్‌డ్ ఐడెంటిటీలను వాడండి; లోకల్ డెవలప్‌మెంట్ కి `az login` ఉపయోగించండి
- ఛాప్టర్ 5 లో ఉన్న బాధ్యత కలిగిన AI మార్గదర్శకాలు అనుసరించండి

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**అస్వీకరణ**:
ఈ పత్రం AI అనువాద సేవ [Co-op Translator](https://github.com/Azure/co-op-translator) ఉపయోగించి అనువదించబడింది. మేము ఖచ్చితత్వానికి ప్రయత్నిస్తున్నప్పటికీ, ఆటోమేటెడ్ అనువాదాలు తప్పులు లేదా అసమగ్రతలను కలిగి ఉండవచ్చు. దాని స్వదేశ భాషలో ఉన్న అసలు పత్రాన్ని అధికారం కలిగిన మూలంగా పరిగణించాలి. కీలకమైన సమాచారం కోసం, ప్రొఫెషనల్ మానవ అనువాదాన్ని సిఫారసు చేస్తాము. ఈ అనువాదం ఉపయోగం వల్ల కలిగే ఏవైనా అపార్థాలు లేదా తప్పుదారులు కోసం మేము బాధ్యత వహించము.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->