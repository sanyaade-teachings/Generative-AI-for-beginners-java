# AGENTS.md

## Project Overview

இது Java உடன் Generative AI மேம்பாட்டை கற்றுக்கொள்ளும் கல்வி களஞ்சியமாகும். இது பெரிய மொழி மாதிரிகள் (LLMs), ப்ராம்ட் பொறியியல், எம்பெட்டிங்ஸ், RAG (Retrieval-Augmented Generation), மற்றும் Model Context Protocol (MCP) ஆகியவற்றை உள்ளடக்கிய முழுமையான நடைமுறை பாடத்திட்டத்தைக் வழங்குகிறது.

**முக்கிய தொழில்நுட்பங்கள்:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI, மற்றும் OpenAI SDKகள்

**கட்டமைப்பு:**
- அத்தியாயங்களின்படி ஒழுங்குபடுத்தப்பட்ட பல தனித்து இயங்கும் Spring Boot பயன்பாடுகள்
- வெவ்வேறு AI வடிவங்களை காட்டு மாதிரிகள் உள்ள படைப்புகள்
- முன்கூட்டியே கான்ஃபிகர் செய்யப்பட்ட dev கன்டெய்நர்களுடன் GitHub Codespaces தயார்

## Setup Commands

### Prerequisites
- Java 21 அல்லது அதற்கு மேல்
- Maven 3.x
- Azure AI Foundry மாதிரி பிணைப்பு கொண்ட Azure சந்தா ( `azd up` மூலம் ஏற்படுத்தவும் )
- Azure CLI (`az`) மற்றும் Azure Developer CLI (`azd`), விசையற்ற அங்கீகாரம் மூலம் உள்நுழைந்துள்ளீர்கள்

### Environment Setup

**Option 1: GitHub Codespaces (பரிந்துரைக்கப்படுகிறது)**
```bash
# தொகுப்பகத்தை கிளைபடுத்து மற்றும் GitHub UI இல் இருந்து ஒரு கோட்பகுதி உருவாக்கு
# டெவ் கட்டகம் தானாகவே அனைத்து சார்புகளையும் நிறுவும்
# சூழலை அமைப்பதற்கான ~2 நொடிகள் காத்திரு
```

**Option 2: Local Dev Container**
```bash
# களஞ்சியத்தை நகல் எடுத்தல்
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Dev Containers நீட்சியுடன் VS Code இல் திறக்கவும்
# கேட்டபோது கன்டெயினரில் மீண்டும் திறக்கவும்
```

**Option 3: Local Setup**
```bash
# சார்புகள் நிறுவுக
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# நிறுவலை சரிபார்க்கவும்
java -version
mvn -version
```

### Configuration

**Azure AI Foundry அமைப்பு (விசையற்ற, பரிந்துரைக்கப்பட்டது):**
```bash
# கோடு ஆக Foundry கணக்கு + மாதிரி பிரசாரங்களை ஏற்பாடு செய்க
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd உங்கள் எண்ட்பாயிண்ட் (எந்த விசையும் இல்லாமல்) உடன் examples/basic-chat-azure/.env ஐ எழுதுகிறது
```

**கையால் endpoint அமைப்பு:**
```bash
# நீங்கள் azd பயன்படுத்தவில்லை என்றால், தானாகவே முனையத்தை அமைக்கவும் (அங்கீகாரம் az login மூலம் கீ இல்லாமல் தொடரும்)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# .env ஐ திருத்தவும்: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
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

**ஒரு Spring Boot பயன்பாட்டை இயக்குவதற்கான வழி:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**ஒரு திட்டத்தை கட்டிப்பிடிப்பது:**
```bash
cd [project-directory]
mvn clean install
```

**MCP கால்குலேட்டர் சேவையை துவக்குவது:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# சர்வர் http://localhost:8080ல் இயங்குகிறது
```

**கிளைண்ட் உதாரணங்களை இயக்குதல்:**
```bash
# மற்றொரு டெர்மினலில் சர்வரைத் துவக்கிய பிறகு
cd 04-PracticalSamples/calculator

# நேரடி MCP கிளையன்ட்
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# ஏ.ஐ. சக்தியாளர் கிளையன்ட் (AZURE_OPENAI_ENDPOINT + az login தேவை)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# இடையாட்சியான பாட்டி
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Hot Reload
Spring Boot DevTools உடன் சூடு மீள் ஏற்றும் ஆதரவு உள்ள திட்டங்கள்:
```bash
# Java கோப்புகளில் செய்யப்பட்ட மாற்றங்கள் சேமிக்கப்படும்போது தானாக மீண்டும் ஏற்றப்படும்
mvn spring-boot:run
```

## Testing Instructions

### Running Tests

**ஒரு திட்டத்தில் அனைத்து பரிசோதனைகளையும் இயக்க:**
```bash
cd [project-directory]
mvn test
```

**விரிவான வெளியீட்டுடன் பரிசோதனைகள் இயக்க:**
```bash
mvn test -X
```

**குறிப்பிட்ட பரிசோதனை வகுப்பை இயக்க:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Test Structure
- பரிசோதனை கோப்புகள் JUnit 5 (Jupiter) பயன்படுத்துகின்றன
- பரிசோதனை வகுப்புகள் `src/test/java/` ல் உள்ளன
- கால்குலேட்டர் திட்டத்தின் கிளையண்ட் உதாரணங்கள் `src/test/java/com/microsoft/mcp/sample/client/` ல் உள்ளன

### Manual Testing
பல உதாரணங்கள் கைமுறையான பரிசோதனையை தேவைப்படும் இடைமுக பயன்பாடுகள் ஆகும்:

1. `mvn spring-boot:run` கொண்டு பயன்பாட்டை தொடங்கு
2. செருகுச்சொற்களை பரிசோதிக்க அல்லது CLI உடன் தொடர்புகொள்
3. ஒவ்வொரு திட்டத்தின் README.md இல் உள்ள ஆவணங்களுடன் நடத்தை பொருந்துகிறதா என்று உறுதி செய்

### Testing with Azure AI Foundry
- உதாரணங்களை இயக்குவதற்கு முன் `az login` கொண்டு உள்நுழையவும் (விசையற்ற அங்கீகாரம்)
- உங்கள் கணக்கில் Cognitive Services OpenAI User உரிமை இருப்பதை உறுதி செய்க
- அத்தியாயம் 5 இல் உள்ள பொறுப்புள்ள AI உதாரணத்துடன் உள்ளடக்க வடிகட்டலை பரிசோதிதல்

## Code Style Guidelines

### Java Conventions
- **Java பதிப்பு:** Java 21 மற்றும் நவீன அம்சங்கள்
- **ஈழை:** தரமான Java முறைகளை பின்பற்று
- **பெயரிடல்:** 
  - வகுப்புகள்: PascalCase
  - முறை/மாறிலிகள்: camelCase
  - மாறாதவை: UPPER_SNAKE_CASE
  - தொகுப்பு பெயர்கள்: சிறிய எழுத்துக்கள்

### Spring Boot Patterns
- வணிக தர்க்கத்துக்கு `@Service` பயன்படுத்தவும்
- REST செருகுச்சொற்களுக்கு `@RestController` பயன்படுத்தவும்
- `application.yml` அல்லது `application.properties` மூலம் கட்டமைப்பு
- கடினமாக்கப்பட்ட மதிப்புகளுக்கு மேலாக சுற்றுச்சூழல் மாறிலிகள் பயன்படுத்தப்பட வேண்டும்
- MCP வெளியிடப்படும் முறைமைகளுக்கு `@Tool` குறிச்சொல் பயன்படுத்தவும்

### File Organization
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

### Dependencies
- Maven `pom.xml` மூலம் நிர்வகிக்கப்படுகிறது
- பதிப்பு மேலாண்மைக்கு Spring AI BOM
- AI இணைப்புகளுக்கு LangChain4j
- Spring சார்ந்த உதவிக்கூறுகளுக்கான Spring Boot starter parent

### Code Comments
- பொதுப் APIs க்கு JavaDoc சேர்க்கவும்
- சிக்கலான AI தொடர்புகளுக்கு விளக்கக் குறிப்புகளை சேர்க்கவும்
- MCP கருவிகள் விளக்கங்களையும் ஆவணப்படுத்தவும்

## Build and Deployment

### Building Projects

**பரிசோதனைகள் இல்லாமல் கட்டமைக்க:**
```bash
mvn clean install -DskipTests
```

**அனைத்து சரிபார்ப்புகளுடன் கட்டமைக்க:**
```bash
mvn clean install
```

**பயன்பாட்டை தொகுக்க:**
```bash
mvn package
# target/ கோப்புறையில் JAR உருவாக்கும்
```

### Output Directories
- கட்டமைக்கப்பட்ட வகுப்புகள்: `target/classes/`
- பரிசோதனை வகுப்புகள்: `target/test-classes/`
- JAR கோப்புகள்: `target/*.jar`
- Maven தயாரிப்புகள்: `target/`

### Environment-Specific Configuration

**மேம்பாட்டு:**
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

**தயாரிப்பு:**
- விசையற்ற அங்கீகாரத்திற்கு `az login` இடத்துக்கு பதிலாக மேலாண்மை அடையாளத்தைப் பயன்படுத்தவும்
- உங்களுடைய தயாரிப்பு Foundry வளத்திற்கு `AZURE_OPENAI_ENDPOINT` ஐ நோக்கவும்
- சுற்றுச்சூழல் மாறிலிகள் அல்லது Azure Key Vault மூலம் கட்டமைப்பை நிர்வகிக்கவும்

### Deployment Considerations
- இது எடுத்துக்காட்டு பயன்பாடுகள் கொண்ட கல்வி களஞ்சியம்
- தயாரிப்பு சுற்றுச்சூழலுக்கு நேரடியாக பயன்படுத்தும் விதமாக இல்லை
- தயாரிப்பு பயன்பாட்டிற்கு இவற்றை சரிசெய்யவும்
- தனி திட்ட READMEகளை விரிவாக பார்க்கவும்

## Additional Notes

### Azure AI Foundry
- **விசையற்ற அங்கீகாரம்:** Microsoft Entra ID மூலம் இணைவு — API விசைகள் தேவையில்லை
- **குறிமுறையாக உருவாக்கப்பட்டது:** Bicep + azd (`azd up`) மூலம் கணக்கு மற்றும் மாதிரி பிணைப்புகள் உருவாக்கப்படுகின்றன
- அதே OpenAI பொருத்தமான குறியீடு உள்ளூர் (`az login`) மற்றும் Azure (மேலாண்மை அடையாளம்) இரண்டிலும் இயங்கும்

### Working with Multiple Projects
ஒவ்வொரு எடுத்துக்காட்டு திட்டமும் தனித்துவமாக உள்ளது:
```bash
# குறிப்பிட்ட திட்டத்திற்கு வழி செல்லவும்
cd 04-PracticalSamples/[project-name]

# ஒவ்வொன்றுக்கும் தனித்தனி pom.xml உள்ளன மற்றும் தனித்தனியாக கட்டிக்கொள்ளக்கூடியவை
mvn clean install
```

### Common Issues

**Java பதிப்பு பொருந்தாமை:**
```bash
# ஜாவா 21 ஐ சரிபார்க்கவும்
java -version
# தேவையானால் JAVA_HOMEஐ புதுப்பிக்கவும்
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**இணைப்புக் குறியீட்டு பதிவிறக்கம் சிக்கல்கள்:**
```bash
# மெவன் கேஷை அழித்து மீண்டும் முயற்சி செய்
rm -rf ~/.m2/repository
mvn clean install
```

**செறுகுத்தளம் அல்லது உள்நுழைவு காணாதது:**
```bash
# தற்போதைய அமர்வில் முகவரி அமைக்கவும் மற்றும் உள்நுழையவும் (கீர் இல்லாமல்)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# அல்லது திட்ட அடைவில் உள்ள .env கோப்பை பயன்படுத்தவும்
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**ஓர்கொண்ட போர்ட் ஏற்கனவே பயன்படுத்தப்பட்டிருப்பது:**
```bash
# Spring Boot இயல்பாக 8080 போர்ட்டைப் பயன்படுத்துகிறது
# application.properties இல் மாற்றம்:
server.port=8081
```

### Multi-Language Support
- 45+ மொழிகளில் தானாக மொழிபெயர்ப்பு உடன் ஆவணங்கள் கிடைக்கும்
- மொழிபெயர்ப்புகள் `translations/` கோப்பகத்தில் உள்ளன
- மொழிபெயர்ப்பு GitHub Actions வேலைப்பாட்டால் நிர்வகிக்கப்படுகிறது

### Learning Path
1. [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md) ஆக தொடங்கவும்
2. அத்தியாயங்களை வரிசைப்படி பின்பற்றவும் (01 → 05)
3. ஒவ்வொரு அத்தியாயத்திலும் நடைமுறை உதாரணங்களை முடிக்கவும்
4. அத்தியாயம் 4 இல் எடுக்கும் திட்டங்களை ஆராயவும்
5. அத்தியாயம் 5 இல் பொறுப்புள்ள AI முறைமைகளை கற்றுக் கொள்ளவும்

### Development Container
`.devcontainer/devcontainer.json` இல் உள்ள அமைப்புகள்:
- Java 21 மேம்பாட்டு சுற்றுச்சூழல்
- முன்கூட்டியே நிறுவப்பட்ட Maven
- VS Code Java நீட்சிகள்
- Spring Boot கருவிகள்
- GitHub Copilot ஒருங்கிணைப்பு
- Docker-in-Docker ஆதரவு
- Azure CLI

### Performance Considerations
- Azure AI Foundry பிணைப்புகளில் நிமிடக் குறைந்த அளவில் டோக்கன்/வினா வரம்புகள் உள்ளன
- எம்பெட்டிங்ஸ் க்கான பொருத்தமான தொகுப்புகளுக்கு பயன்படுத்தவும்
- மீண்டும் மீண்டும் API அழைப்புகளை காச்சே செய்யலாம்
- செயற்கை செலவு குறைக்க டோக்கன் பயன்பாட்டைக் கண்காணிக்கவும்

### Security Notes
- `.env` கோப்புகளை ஒருபோதும் குறியீட்டுக் கட்டுப்பாட்டில் சேர்க்காதீர்கள் (`.gitignore` இல் உள்ளது)
- விசையற்ற அங்கீகாரம் (Microsoft Entra ID) API விசைகளுக்கு மேல் விருப்பத்தேர்வாகவும்
- Azure இல் மேலாண்மையைக் கொண்ட அடையாளங்களைப் பயன்படுத்தவும்; உள்ளூரில் மேம்பாட்டிற்காக `az login` ஐப் பயன்படுத்தவும்
- அத்தியாயம் 5 இல் உள்ள பொறுப்புள்ள AI வழிகாட்டுதல்களை பின்பற்றவும்

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**மறுப்பு**:
இந்த ஆவணம் AI மொழிபெயர்ப்பு சேவை [Co-op Translator](https://github.com/Azure/co-op-translator) பயன்படுத்தி மொழிபெயர்க்கப்பட்டுள்ளது. நாங்கள் துல்லியத்திற்காக முயற்சி செய்துள்ளோம், ஆனால் தானாக செய்யப்படும் மொழிபெயர்ப்புகளில் பிழைகள் அல்லது தவறுகள் இருக்கலாம் என்பதை கவனத்தில் கொள்ளவும். அசல் ஆவணம் அதன் தாய்மொழியில் அதிகாரப்பூர்வ ஆதாரமாக கருதப்பட வேண்டும். முக்கியமான தகவல்களுக்கு, தொழில்நுட்பமான மனித மொழிபெயர்ப்பு பரிந்துரைக்கப்படுகிறது. இந்த மொழிபெயர்ப்பைப் பயன்படுத்துவதால் ஏற்படும் எந்த தவறான புரிதல்கள் அல்லது தவறான விளக்கத்திற்கும் நாங்கள் பொறுப்பில்வில்லை.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->