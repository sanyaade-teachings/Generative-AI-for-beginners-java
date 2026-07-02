# AGENTS.md

## Muhtasari wa Mradi

Hii ni hazina ya elimu kwa kujifunza maendeleo ya AI Inayozalisha kwa kutumia Java. Inatoa kozi kamili ya vitendo inayojumuisha Modeli Kubwa za Lugha (LLMs), uhandisi wa prompti, uingizaji data, RAG (Retrieval-Augmented Generation), na Itifaki ya Muktadha wa Modeli (MCP).

**Teknolojia Muhimu:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI, na OpenAI SDKs

**Mimknimu:**
- Programu nyingi za Spring Boot za kujitegemea zilizopangwa kwa sura
- Miradi ya sampuli inaonyesha mifumo tofauti ya AI
- Tayari kwa GitHub Codespaces na kontena za maendeleo zilizopangwa awali

## Amri za Usanidi

### Mahitaji Kabla
- Java 21 au juu
- Maven 3.x
- Uanachama wa Azure na uanzishaji wa mfano wa Azure AI Foundry (patanisha na `azd up`)
- Azure CLI (`az`) na Azure Developer CLI (`azd`), umeingia kwa uthibitisho usio na funguo

### Usanidi wa Mazingira

**Chaguo 1: GitHub Codespaces (Inayopendekezwa)**
```bash
# Tenganisha hazina na unda nafasi ya msimbo kutoka UI ya GitHub
# Kontena la maendeleo litaweka kiotomatiki utegemezi wote
# Subiri takriban dakika 2 kwa ajili ya usanidi wa mazingira
```

**Chaguo 2: Kontena ya Maendeleo ya Mbali**
```bash
# Nakili hazina
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Fungua katika VS Code kwa nyongeza ya Dev Containers
# Fungua tena ndani ya Kontena wakati unapoulizwa
```

**Chaguo 3: Usanidi wa Mbali**
```bash
# Sakinisha utegemezi
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Thibitisha ufungaji
java -version
mvn -version
```

### Usanidi

**Usanidi wa Azure AI Foundry (bila funguo, unaopendekezwa):**
```bash
# Toa akaunti ya Foundry + usambazaji wa modeli kama msimbo
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd inaandika examples/basic-chat-azure/.env na kiunganishi chako (bila funguo)
```

**Usanidi wa mwisho kwa mwisho kwa mkono:**
```bash
# Ikiwa hukutumia azd, weka sehemu ya mwisho mwenyewe (uthibitishaji unabaki bila kufungua funguo kupitia az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Hariri .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Mtiririko wa Maendeleo

### Muundo wa Mradi
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

### Kuendesha Programu

**Kuendesha programu ya Spring Boot:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Kujenga mradi:**
```bash
cd [project-directory]
mvn clean install
```

**Kuanza MCP Calculator Server:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# Seva inaendesha kwenye http://localhost:8080
```

**Kuendesha Mifano ya Mteja:**
```bash
# Baada ya kuanzisha seva kwenye terminali nyingine
cd 04-PracticalSamples/calculator

# Mteja wa MCP wa moja kwa moja
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# Mteja aliyewezeshwa na AI (inahitaji AZURE_OPENAI_ENDPOINT + az login)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Roboti ya ushirikiano
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Ufrishaji wa Haraka (Hot Reload)
Spring Boot DevTools imejumuishwa katika miradi inayounga mkono ufrishaji wa haraka:
```bash
# Mabadiliko kwenye faili za Java yatajirudisha moja kwa moja mara tu yanapohifadhiwa
mvn spring-boot:run
```

## Maelekezo ya Upimaji

### Kuendesha Jaribio

**Endesha majaribio yote katika mradi:**
```bash
cd [project-directory]
mvn test
```

**Endesha majaribio na maelezo ya kina:**
```bash
mvn test -X
```

**Endesha darasa la jaribio mahususi:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Muundo wa Jaribio
- Faili za majaribio zinatumia JUnit 5 (Jupiter)
- Madarasa ya majaribio yanafanyika katika `src/test/java/`
- Mifano ya wateja katika mradi wa kalkuleta iko katika `src/test/java/com/microsoft/mcp/sample/client/`

### Upimaji wa Mkono
Mifano mingi ni programu za kushirikiana zinazohitaji upimaji wa mkono:

1. Anzisha programu kwa `mvn spring-boot:run`
2. Jaribu vituo vya mwisho au shirikiana na CLI
3. Thibitisha tabia inayotarajiwa inalingana na nyaraka katika README.md ya kila mradi

### Upimaji kwa Azure AI Foundry
- Ingia kwa `az login` kabla ya kuendesha mifano (uthibitisho usio na funguo)
- Hakikisha akaunti yako ina sehemu ya Cognitive Services OpenAI User kwenye rasilimali
- Jaribu uchujaji wa maudhui kwa mfano wa AI unaowajibika katika Sura ya 5

## Mwongozo wa Mtindo wa Msimbo

### Miongozo ya Java
- **Toleo la Java:** Java 21 na sifa za kisasa
- **Mtindo:** Fuata kanuni za kawaida za Java
- **Majina:** 
  - Madarasa: PascalCase
  - Mbinu/mbadiliko: camelCase
  - Miskani: UPPER_SNAKE_CASE
  - Majina ya paket: herufi ndogo

### Mifumo ya Spring Boot
- Tumia `@Service` kwa mantiki ya biashara
- Tumia `@RestController` kwa vituo vya REST
- Usanidi kupitia `application.yml` au `application.properties`
- Vigezo vya mazingira vinapendekezwa kwa juu ya thamani zilizowekwa
- Tumia alama ya `@Tool` kwa njia zilizofunguliwa na MCP

### Mpangilio wa Faili
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

### Mahitaji ya Kutegemea
- Imesimamiwa kupitia Maven `pom.xml`
- Spring AI BOM kwa usimamizi wa toleo
- LangChain4j kwa ushirikiano wa AI
- Spring Boot starter parent kwa kutegemea za Spring

### Maelezo ya Msimbo
- Ongeza JavaDoc kwa API za umma
- Jumuisha maelezo ya kuelezea kwa mwingiliano tata wa AI
- Andika maelezo ya zana za MCP kwa uwazi

## Ujenzi na Upandikaji

### Kujenga Miradi

**Jenga bila majaribio:**
```bash
mvn clean install -DskipTests
```

**Jenga na ukaguzi wote:**
```bash
mvn clean install
```

**Pakua programu:**
```bash
mvn package
# Inaunda JAR katika saraka ya target/
```

### Folda za Matokeo
- Madarasa yaliyokusanywa: `target/classes/`
- Madarasa ya majaribio: `target/test-classes/`
- Faili za JAR: `target/*.jar`
- Vifaa vya Maven: `target/`

### Usanidi wa Mazingira Maalum

**Maendeleo:**
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

**Uzalishaji:**
- Tumia utambulisho ulio kusimamiwa badala ya `az login` kwa uthibitisho usio na funguo
- Elekeza `AZURE_OPENAI_ENDPOINT` kwa rasilimali yako ya Foundry ya uzalishaji
- Simamia usanidi kupitia vigezo vya mazingira au Azure Key Vault

### Mambo ya Kuzingatia kwa Uwekaji

- Hii ni hazina ya elimu yenye programu za sampuli
- Hairuhusiwi kwa usambazaji moja kwa moja kama ilivyo
- Sampuli zinaonyesha mifumo ya kurekebisha kwa matumizi ya uzalishaji
- Angalia README za miradi binafsi kwa maelezo maalum ya usambazaji

## Vidokezo Zaidi

### Azure AI Foundry
- **Uthibitishaji bila funguo:** ungana na Microsoft Entra ID — hakuna funguo za API za kusimamia
- **Iliundwa kama msimbo:** Bicep + azd (`azd up`) hutoa akaunti na uanzishaji wa mifano
- Msimbo uleule unaoendana na OpenAI unafanya kazi eneo la mtaa (`az login`) na Azure (utambulisho ulio simamiwa)

### Kazi na Miradi Mingi
Kila mradi wa mfano ni wa kujitegemea:
```bash
# Elekea kwenye mradi maalum
cd 04-PracticalSamples/[project-name]

# Kila mmoja ana pom.xml yake na unaweza kujengwa kwa kujitegemea
mvn clean install
```

### Masuala ya Kawaida

**Kutofautiana kwa Toleo la Java:**
```bash
# Thibitisha Java 21
java -version
# Sasisha JAVA_HOME ikiwa inahitajika
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Matatizo ya Upakuaji wa Tegemezi:**
```bash
# Futa cache ya Maven na jaribu tena
rm -rf ~/.m2/repository
mvn clean install
```

**Vituo vya Mwisho au Kuingia Havipatikani:**
```bash
# Weka mwisho wa huduma katika kikao cha sasa na ingia (bila funguo)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Au tumia faili la .env katika saraka ya mradi
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Bandari Imekuwa Inatumiwa:**
```bash
# Spring Boot hutumia bandari 8080 kama chaguo-msingi
# Badilisha katika application.properties:
server.port=8081
```

### Msaada wa Lugha Mbalimbali
- Nyaraka zinapatikana katika lugha 45+ kwa tafsiri ya kiotomatiki
- Tafsiri ziko katika saraka ya `translations/`
- Tafsiri inasimamiwa na mtiririko wa kazi wa GitHub Actions

### Njia ya Kujifunza
1. Anza na [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. Fuata sura kwa mpangilio (01 → 05)
3. Kamili mifano ya vitendo katika kila sura
4. Chunguza miradi ya sampuli katika Sura ya 4
5. Jifunze mbinu za AI inayowajibika katika Sura ya 5

### Kontena la Maendeleo
Faili `.devcontainer/devcontainer.json` hupanua:
- Mazingira ya maendeleo ya Java 21
- Maven imewekwa kabla
- Virutubishi vya VS Code Java
- Zana za Spring Boot
- Uchanganuzi wa GitHub Copilot
- Msaada wa Docker-in-Docker
- Azure CLI

### Mambo ya Utendaji
- Uanzishaji wa Azure AI Foundry una vikwazo vya tokeni/maombi kwa dakika
- Tumia ukubwa unaofaa wa kundi kwa uingizaji data
- Fikiria kuhifadhi cache kwa wito wa API unaojirudia
- Angalia matumizi ya tokeni kwa kuboresha gharama

### Vidokezo vya Usalama
- Kamwe usiweke faili za `.env` (zipo tayari `.gitignore`)
- Pendekeza uthibitisho usio na funguo (Microsoft Entra ID) badala ya funguo za API
- Tumia utambuzi ulio simamiwa katika Azure; `az login` kwa maendeleo ya ndani
- Fuata miongozo ya AI inayowajibika katika Sura ya 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Kionyozo**:
Hati hii imetafsiriwa kwa kutumia huduma ya tafsiri ya AI [Co-op Translator](https://github.com/Azure/co-op-translator). Ingawa tunajitahidi kupata usahihi, tafadhali fahamu kwamba tafsiri za kiotomatiki zinaweza kuwa na makosa au upungufu wa usahihi. Hati ya asili katika lugha yake halisi inapaswa kuchukuliwa kama chanzo cha mamlaka. Kwa taarifa muhimu, tafsiri ya kitaalamu inayofanywa na binadamu inapendekezwa. Hatutojibu kwa kuelewa vibaya au tafsiri potofu zinazotokea kutokana na matumizi ya tafsiri hii.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->