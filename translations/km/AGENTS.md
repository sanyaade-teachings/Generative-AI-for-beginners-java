# AGENTS.md

## គំរោងទូទៅ

នេះគឺជាឃ្លាំងរៀនសម្រាប់ការសិក្សាការអភិវឌ្ឍន៍ Generative AI ជាមួយ Java។ វាបំពេញដោយវគ្គសិក្សាដែលផ្តល់បទពិសោធន៍ផ្ទាល់ដូចជា Large Language Models (LLMs), ការបង្កើត prompt, embeddings, RAG (Retrieval-Augmented Generation), និង Model Context Protocol (MCP)។

**បច្ចេកវិទ្យាសំខាន់ៗ៖**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI, និង SDKs របស់ OpenAI

**ស្ថាបัตยការ៖**
- មានកម្មវិធី Spring Boot ឯករាជ្យច្រើនដែលរៀបចំដោយជំពូក
- គំរូគំរោងដែលបង្ហាញពីលំនាំ AI ផ្សេងៗគ្នា
- ប្រមូលផ្ដុំរួចជាស្រេចសម្រាប់ GitHub Codespaces ជាមួយ dev containers ដែលបានកំណត់រួច

## ពាក្យបញ្ជាការតំឡើង

### តម្រូវការជាមុន
- Java 21 ឬខ្ពស់ជាងនេះ
- Maven 3.x
- មានជាវ Azure ដែលមាន model deployment នៅលើ Azure AI Foundry (បង្កើតជាមួយ `azd up`)
- Azure CLI (`az`) និង Azure Developer CLI (`azd`), ចូលឈ្មោះហើយសម្រាប់ការផ្ទៀងផ្ទាត់ keyless

### ការតំឡើងបរិស្ថាន

**ជម្រើស 1: GitHub Codespaces (ណែនាំ)**
```bash
# បំបែកហាងដើមហើយបង្កើត codespace មួយពី UI GitHub
# កុងតឺន័រ dev នឹងដំឡើងអាសយដ្ឋានទាំងអស់ដោយស្វ័យប្រវត្តិ
# សូមរង់ចាំប្រហែល ~2 នាទីសម្រាប់ការតំឡើងបរិយាកាស
```

**ជម្រើស 2: Dev Container នៅក្នុងតំបន់ដំណើរការ**
```bash
# ចម្លងឃ្លាំង
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# បើកក្នុង VS Code ជាមួយផ្នែកបន្ថែម Dev Containers
# បើកម្ដងទៀតក្នុង Container ពេលដែលត្រូវបានស្នើ
```

**ជម្រើស 3: តំឡើងនៅលើម៉ាស៊ីនផ្ទាល់**
```bash
# ដំឡើងការពឹងផ្អែក
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# បញ្ជាក់ទម្រង់ការដំឡើង
java -version
mvn -version
```

### ការបង្កើតរចនាសម្ព័ន្ធ

**ការតំឡើង Azure AI Foundry (keyless, ណែនាំ):**
```bash
# បង្កើតគណនី Foundry + ការដាក់ម៉ូដែលជាកូដ
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd សរសេរ examples/basic-chat-azure/.env ជាមួយចំណុចចូលរបស់អ្នក (គ្មានកូនសោ)
```

**កំណត់ endpoint ដោយដៃ:**
```bash
# ប្រសិនបើអ្នកមិនបានប្រើ azd សូមកំណត់ចំណុចផ្ដាច់មុខដោយខ្លួនឯង (ការផ្ទៀងផ្ទាត់មិនបាច់ប្រើកូនសោជាមួយ az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# កែប្រែ .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## សន្ទស្សន៍ការអភិវឌ្ឍន៍

### រចនាសម្ព័ន្ធគំរោង
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

### បានដំណើរការ​កម្មវិធី

**ការរត់កម្មវិធី Spring Boot:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**ការសង់គំរោង:**
```bash
cd [project-directory]
mvn clean install
```

**ចាប់ផ្តើម MCP Calculator Server:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# ម៉ាស៊ីនមេដំណើរការនៅលើ http://localhost:8080
```

**រត់ Client Examples:**
```bash
# បន្ទាប់ពីបើកម៉ាស៊ីនមេនៅក្នុងបន្ទាត់មួយផ្សេងទៀត
cd 04-PracticalSamples/calculator

# អតិថិជន MCP ដ៏ផ្ទាល់
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# អតិថិជនដែលដំណើរការដោយ AI (តម្រូវឱ្យមាន AZURE_OPENAI_ENDPOINT + az login)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# បុតួយឆ្លើយតបអន្តរកម្ម
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### ការផ្ទុកឡើងវិញយ៉ាន់យ៉ង
Spring Boot DevTools ត្រូវបានដាក់បញ្ចូលក្នុងគំរោងដែលគាំទ្រការផ្ទុកឡើងវិញយ៉ាន់យ៉ង៖
```bash
# ការផ្លាស់ប្តូរទៅកាន់ឯកសារ Java នឹងធ្វើការឡើងវិញដោយស្វ័យប្រវត្តិពេលបានរក្សាទុក
mvn spring-boot:run
```

## សេចក្តីណែនាំសម្រាប់ការតេស្ត

### រត់ការតេស្ត

**រត់ការតេស្តទាំងអស់នៅក្នុងគំរោង:**
```bash
cd [project-directory]
mvn test
```

**រត់តេស្តជាមួយលទ្ធផលលម្អិត:**
```bash
mvn test -X
```

**រត់តេស្តថ្នាក់ជាក់លាក់:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### រចនាសម្ព័ន្ធការតេស្ត
- ឯកសារតេស្តប្រើ JUnit 5 (Jupiter)
- ថ្នាក់តេស្តនៅក្នុង `src/test/java/`
- Client examples ក្នុងគំរោង calculator មាននៅ `src/test/java/com/microsoft/mcp/sample/client/`

### ការតេស្តដោយដៃ
គំរូជាច្រើនគឺកម្មវិធីអន្តរចលដែលត្រូវការការតេស្តដោយដៃ៖

1. ចាប់ផ្តើមកម្មវិធីជាមួយ `mvn spring-boot:run`
2. តេស្ត endpoints ឬអន្តរប្រតិបត្តិការជាមួយ CLI
3. ពិនិត្យមើលអាកប្បកិរិយាប្រៀបធៀបទៅនឹងឯកសារព័ត៌មានក្នុង README.md នៃគំរោងនីមួយៗ

### ការតេស្តជាមួយ Azure AI Foundry
- ចូលដោយ `az login` មុនពេលដំណើរការ គំរូ (keyless auth)
- ពិនិត្យមើលថា គណនីរបស់អ្នកមានតួនាទី Cognitive Services OpenAI User លើធនធាន
- តេស្តការត្រួតពិនិត្យមាតិកាមួយជាមួយតម្លៃ AI ទទួលខុសត្រូវក្នុងជំពូក 5

## មគ្គុទេសក៍រចនាទម្រង់កូដ

### ព័ត៌មាន Java
- **កំណែ Java:** Java 21 ជាមួយលក្ខណៈទំនើប
- **ទម្រង់៖** តាមរចនាបថ Java ស្តង់ដារ
- **ការកំណត់ឈ្មោះ:** 
  - ថ្នាក់: PascalCase
  - មេធតូត/អថេរ: camelCase
  - តម្លៃថេរ: UPPER_SNAKE_CASE
  - ឈ្មោះកញ្ចប់: តូចទាំងអស់

### លំនាំ Spring Boot
- ប្រើ `@Service` សម្រាប់លូក់លែងអាជីវកម្ម
- ប្រើ `@RestController` សម្រាប់ REST endpoints
- ការកំណត់តាម `application.yml` ឬ `application.properties`
- រំពឹងអថេរបរិស្ថានជាងតម្លៃដែលបានកូដរឹត
- ប្រើស្លាក `@Tool` សម្រាប់មេធតូតដែល MCP បង្ហាញ

### ការរៀបចំឯកសារ
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

### ដោយផ្អែកលើ Dependencies
- គ្រប់គ្រងជា Maven `pom.xml`
- Spring AI BOM សម្រាប់គ្រប់គ្រងកំណែ
- LangChain4j សម្រាប់ការតភ្ជាប់ AI
- Spring Boot starter parent សម្រាប់ Dependencies របស់ Spring

### ការពិពណ៌នាកូដ
- បន្ថែម JavaDoc សម្រាប់ API សាធារណៈ
- បញ្ចូលការពិពណ៌នាសម្រាប់ប្រតិបត្តិការលំបាកជាមួយ AI
- ចុះឈ្មោះឧបករណ៍ MCP ឲ្យច្បាស់លាស់

## ការសង់និងដំណើរការ

### ការសង់គំរោង

**សង់ដោយគ្មានការតេស្ត:**
```bash
mvn clean install -DskipTests
```

**សង់ជាមួយការត្រួតពិនិត្យទាំងអស់:**
```bash
mvn clean install
```

**ការកញ្ចប់កម្មវិធី:**
```bash
mvn package
# បង្កើត JAR នៅក្នុងថត target/
```

### កន្លែងលទ្ធផល
- ថ្នាក់បានបង្កប់៖ `target/classes/`
- ថ្នាក់តេស្ត៖ `target/test-classes/`
- ឯកសារ JAR: `target/*.jar`
- របស់ Maven: `target/`

### ការកំណត់រចនាសម្ព័ន្ធសម្រាប់បរិស្ថាន

**បរិស្ថានអភិវឌ្ឍន៍:**
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

**ផលិតកម្ម:**
- ប្រើ managed identity ជំនួស `az login` សម្រាប់ keyless auth
- បញ្ជូន `AZURE_OPENAI_ENDPOINT` ទៅធនធាន Foundry ផលិតកម្មរបស់អ្នក
- គ្រប់គ្រងការបង្កើតរចនាសម្ព័ន្ធតាមអថេរបរិស្ថាន ឬ Azure Key Vault

### ការពិចារណាសម្រាប់ការដាក់ឲ្យដំណើរការ
- នេះគឺជាឃ្លាំងសិក្សាជាមួយកម្មវិធីគំរូ
- មិនរចនាសម្រាប់ដំណើរការផលិតកម្មជាក់ស្តែងទេ
- គំរូបង្ហាញលំនាំដែលអាចដល់ការប្រើប្រាស់ផលិតកម្ម
- មើល README.md សម្រាប់កំណត់សម្គាល់ដាក់បញ្ចូល

## កំណត់សំគាល់បន្ថែម

### Azure AI Foundry
- **Keyless auth:** តភ្ជាប់ជាមួយ Microsoft Entra ID — មិនមាន key API ដើម្បីគ្រប់គ្រង
- **Provisioned ជាឃូដ:** Bicep + azd (`azd up`) បង្កើតគណនី និងការប្រើម៉ូឌែល
- កូដ OpenAI-compatible ដដែលដំណើរការ នៅក្នុងម៉ាស៊ីនផ្ទាល់ (`az login`) និងនៅក្នុង Azure (managed identity)

### ការងារជាមួយគំរោងជាច្រើន
គំរូគំរោងនីមួយៗជាកម្មវិធីឯករាជ្យ៖
```bash
# ទៅកាន់គម្រោងជាក់លាក់
cd 04-PracticalSamples/[project-name]

# ពីរមាន pom.xml ផ្ទាល់ខ្លួន ហើយអាចសង់ឡើងដោយឯករាជ្យបាន
mvn clean install
```

### បញ្ហាទូទៅ

**កំណែ Java មិនផ្គូផ្គង:**
```bash
# ពិនិត្យ Java ២០
java -version
# បន្ទាន់សម័យ JAVA_HOME ប្រសិនបើត្រូវការ
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**បញ្ហាការទាញយក Dependencies:**
```bash
# សម្អាតខ្សែផ្ទុក Maven ហើយព្យាយាមម្តងទៀត
rm -rf ~/.m2/repository
mvn clean install
```

**មិនមាន Endpoint ឬការចូល:**
```bash
# កំណត់ចំណុចបញ្ចប់នៅក្នុងស ម័យបច្ចុប្បន្ន និងចូលបញ្ជូល (គ្មានកូនសោ)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# ឬប្រើឯកសារ .env នៅក្នុងថតគម្រោង
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**ច្រកចូលត្រូវបានប្រើហើយ:**
```bash
# Spring Boot ប្រើច្រក ៨០៨០ ជាធម្មតា
# ផ្លាស់ប្ដូរនៅក្នុង application.properties:
server.port=8081
```

### ការគាំទ្រភាសាច្រើន
- ឯកសារបានផ្តល់ជាភាសា ៤៥+ តាមការបញ្ចួនបំលែងស្វ័យប្រវត្តិ
- មើលក្នុងថត `translations/`
- ការបកប្រែគ្រប់គ្រងដោយ GitHub Actions workflow

### ផ្លូវអប់រំ
1. ចាប់ផ្តើមជាមួយ [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. ដើរតាមជំពូកតាមលំដាប់ (01 → 05)
3. បំពេញគំរូដៃគូរាដ៏ពេញលេញក្នុងនីមួយៗ
4. រៀនពីគំរូគំរោងនៅក្នុងជំពូក 4
5. រៀនពីការអនុវត្ត AI ទទួលខុសត្រូវ ក្នុងជំពូក 5

### Development Container
ឯកសារ `.devcontainer/devcontainer.json` កំណត់៖
- បរិយាកាសអភិវឌ្ឍ Java 21
- Maven ត្រូវបានតំឡើងរួច
- VS Code Java បន្ថែម
- ឧបករណ៍ Spring Boot
- GitHub Copilot បញ្ចូល
- Docker-in-Docker គាំទ្រ
- Azure CLI

### ការពិចារណាអំពីកំណត់ប្រសិទ្ធភាព
- Azure AI Foundry មានកំណត់ Token/សំណើក្នុងមួយនាទី
- ប្រើ batch ទំហំសមរម្យសម្រាប់ embeddings
- គិតពីការផ្ទុកទិន្នន័យសម្រាប់ការហៅ API ជាប្រចាំ
- តាមដានការប្រើប្រាស់ Token សម្រាប់សន្សំចំណាយ

### កំណត់សម្គាល់សុវត្ថិភាព
- កុំចូល commit `.env` (បានដាក់ក្នុង `.gitignore` រួច)
- ត្រូវនិយម keyless auth (Microsoft Entra ID) ជាង key API
- ប្រើ managed identities នៅក្នុង Azure; `az login` សម្រាប់អភិវឌ្ឍនៅក្នុងតំបន់ដំណើរការ
- តាមដានមគ្គុទេសក៍ AI ទទួលខុសត្រូវ ក្នុងជំពូក 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ការបដិសេធ**:
ឯកសារនេះត្រូវបានបម្លែងភាសា ដោយប្រើសេវាបម្លែងភាសា AI [Co-op Translator](https://github.com/Azure/co-op-translator)។ ទោះយើងខ្ញុំមានក្តីប្រាថ្នាឱ្យបានច្បាស់លាស់ តែសូមយល់ដឹងថាការបម្លែងដោយស្វ័យប្រវត្តិក៏អាចមានកំហុសឬភាពមិនត្រឹមត្រូវ។ ឯកសារដើមជាភាសាទីតាំងគួរត្រូវបានគេប្រើជាប្រភពច្បាស់លាស់។ សម្រាប់ព័ត៌មានសំខាន់ៗ សូមណែនាំឱ្យប្រើប្រាស់ការប្រែដោយមនុស្សជំនាញ។ យើងខ្ញុំមិនទទួលខុសត្រូវចំពោះការយល់ច្រឡំ ឬការបកស្រាយខុសបន្ទាប់ពីការប្រើប្រាស់ការបម្លែងនេះនោះទេ។
<!-- CO-OP TRANSLATOR DISCLAIMER END -->