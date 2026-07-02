# AGENTS.md

## Project Overview

Ito ay isang pang-edukasyon na repositoryo para matutunan ang Generative AI development gamit ang Java. Nagbibigay ito ng komprehensibong hands-on na kurso na sumasaklaw sa Large Language Models (LLMs), prompt engineering, embeddings, RAG (Retrieval-Augmented Generation), at ang Model Context Protocol (MCP).

**Pangunahing Teknolohiya:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI, at OpenAI SDKs

**Arkitektura:**
- Maramihang standalone na Spring Boot applications na inayos ayon sa mga kabanata
- Mga sample na proyekto na nagpapakita ng iba't ibang AI na mga pattern
- GitHub Codespaces-ready na may pre-configured na dev containers

## Setup Commands

### Mga Kinakailangan
- Java 21 o mas mataas
- Maven 3.x
- Azure subscription na may Azure AI Foundry model deployment (proseso gamit ang `azd up`)
- Azure CLI (`az`) at Azure Developer CLI (`azd`), naka sign-in para sa keyless auth

### Pagsasaayos ng Kapaligiran

**Opsyon 1: GitHub Codespaces (Inirerekomenda)**
```bash
# I-fork ang repository at gumawa ng codespace mula sa GitHub UI
# Awtomatikong i-install ng dev container ang lahat ng dependencies
# Maghintay ng mga ~2 minuto para sa pag-setup ng kapaligiran
```

**Opsyon 2: Lokal na Dev Container**
```bash
# Kopyahin ang repositoryo
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Buksan sa VS Code gamit ang Dev Containers extension
# Muling buksan sa Container kapag na-prompt
```

**Opsyon 3: Lokal na Setup**
```bash
# I-install ang mga kinakailangang dependencies
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Suriin ang pag-install
java -version
mvn -version
```

### Konpigurasyon

**Azure AI Foundry Setup (keyless, inirerekomenda):**
```bash
# Maghanda ng Foundry account + mga deployment ng modelo bilang code
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# Isinusulat ng azd ang examples/basic-chat-azure/.env kasama ang iyong endpoint (walang key)
```

**Manwal na pagkokonekta ng endpoint:**
```bash
# Kung hindi mo ginamit ang azd, itakda mo ang endpoint nang sarili (mananatiling keyless ang auth sa pamamagitan ng az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# I-edit ang .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Development Workflow

### Estruktura ng Proyekto
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

### Pagpapatakbo ng mga Application

**Pagpapatakbo ng Spring Boot application:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Pagbuo ng proyekto:**
```bash
cd [project-directory]
mvn clean install
```

**Pagsimula ng MCP Calculator Server:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# Tumakbo ang server sa http://localhost:8080
```

**Pagpapatakbo ng Mga Halimbawang Kliyente:**
```bash
# Pagkatapos simulan ang server sa ibang terminal
cd 04-PracticalSamples/calculator

# Direktang MCP client
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# Client na pinatatakbo ng AI (kailangan ang AZURE_OPENAI_ENDPOINT + az login)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Interaktibong bot
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Hot Reload
Kasama ang Spring Boot DevTools sa mga proyekto na sumusuporta sa hot reload:
```bash
# Ang mga pagbabago sa mga Java file ay awtomatikong magre-reload kapag na-save
mvn spring-boot:run
```

## Mga Panuto para sa Pagsusuri

### Pagpapatakbo ng mga Pagsubok

**Patakbuhin lahat ng pagsubok sa isang proyekto:**
```bash
cd [project-directory]
mvn test
```

**Patakbuhin ang mga pagsubok na may verbose output:**
```bash
mvn test -X
```

**Patakbuhin ang tiyak na test class:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Estruktura ng Pagsubok
- Ginagamit ang mga test file ng JUnit 5 (Jupiter)
- Ang mga test class ay matatagpuan sa `src/test/java/`
- Ang mga halimbawang kliyente sa calculator project ay nasa `src/test/java/com/microsoft/mcp/sample/client/`

### Manwal na Pagsusuri
Maraming halimbawa ay mga interactive na aplikasyon na nangangailangan ng manwal na pagsusuri:

1. Simulan ang aplikasyon gamit ang `mvn spring-boot:run`
2. Suriin ang mga endpoint o makipag-interact sa CLI
3. Tiyakin na ang inaasahang gawi ay tumutugma sa dokumentasyon sa README.md ng bawat proyekto

### Pagsusuri gamit ang Azure AI Foundry
- Mag sign-in gamit ang `az login` bago patakbuhin ang mga halimbawa (keyless auth)
- Siguraduhing mayroong Cognitive Services OpenAI User role ang iyong account sa resource
- Subukan ang content filtering gamit ang responsible AI example sa Kabanata 5

## Mga Alituntunin sa Estilo ng Code

### Mga Konbensyon sa Java
- **Bersyon ng Java:** Java 21 na may modernong mga katangian
- **Estilo:** Sundin ang karaniwang mga konbensyon sa Java
- **Pagpapangalan:** 
  - Mga Klase: PascalCase
  - Mga Metodo/variable: camelCase
  - Mga Constant: UPPER_SNAKE_CASE
  - Mga pangalan ng package: maliit ang titik

### Mga Pattern ng Spring Boot
- Gamitin ang `@Service` para sa business logic
- Gamitin ang `@RestController` para sa mga REST endpoint
- Konpigurasyon gamit ang `application.yml` o `application.properties`
- Mas gusto ang environment variables kaysa sa hard-coded na mga halaga
- Gamitin ang `@Tool` annotation para sa mga MCP-exposed na mga metodo

### Organisasyon ng File
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

### Mga Dependencies
- Pinamamahalaan gamit ang Maven `pom.xml`
- Spring AI BOM para sa pamamahala ng bersyon
- LangChain4j para sa AI integrations
- Spring Boot starter parent para sa Spring dependencies

### Mga Komento sa Code
- Magdagdag ng JavaDoc para sa mga public API
- Isama ang mga paliwanag na komento para sa komplikadong AI interactions
- I-dokumento nang malinaw ang mga paglalarawan ng MCP tool

## Pagbuo at Deployment

### Pagbuo ng mga Proyekto

**Pagbuo nang walang pagsusuri:**
```bash
mvn clean install -DskipTests
```

**Pagbuo na may lahat ng tseke:**
```bash
mvn clean install
```

**Pag-package ng aplikasyon:**
```bash
mvn package
# Lumilikha ng JAR sa direktoryo ng target/
```

### Mga Output na Direktoryo
- Mga na-compile na klase: `target/classes/`
- Mga test class: `target/test-classes/`
- Mga JAR file: `target/*.jar`
- Mga Maven artifact: `target/`

### Kapaligirang Espesipikong Konpigurasyon

**Development:**
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

**Produksyon:**
- Gumamit ng managed identity sa halip na `az login` para sa keyless auth
- Itutok ang `AZURE_OPENAI_ENDPOINT` sa iyong production Foundry resource
- Pamahalaan ang konpigurasyon sa pamamagitan ng environment variables o Azure Key Vault

### Mga Pagsasaalang-alang para sa Deployment
- Ito ay isang pang-edukasyon na repositoryo na may mga sample na aplikasyon
- Hindi ito idinisenyo para sa production deployment nang direkta
- Ipinapakita ng mga sample ang mga pattern na maaari mong baguhin para sa production use
- Tingnan ang mga README ng bawat proyekto para sa mga tukoy na tala sa deployment

## Karagdagang Tala

### Azure AI Foundry
- **Keyless auth:** kumonekta gamit ang Microsoft Entra ID — walang kailangan na pamahalaang API keys
- **Provisioned as code:** Bicep + azd (`azd up`) ang gumagawa ng account at model deployments
- Ang parehong OpenAI-compatible code ay tumatakbo nang lokal (`az login`) at sa Azure (managed identity)

### Paggamit ng Maramihang Proyekto
Ang bawat sample na proyekto ay standalone:
```bash
# Mag-navigate sa partikular na proyekto
cd 04-PracticalSamples/[project-name]

# Bawat isa ay may sariling pom.xml at maaaring itayo nang magkahiwalay
mvn clean install
```

### Karaniwang Mga Isyu

**Maling Bersyon ng Java:**
```bash
# Suriin ang Java 21
java -version
# I-update ang JAVA_HOME kung kinakailangan
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Mga Isyu sa Pag-download ng Dependency:**
```bash
# I-clear ang Maven cache at subukang muli
rm -rf ~/.m2/repository
mvn clean install
```

**Hindi Makakita ng Endpoint o Sign-in:**
```bash
# Itakda ang endpoint sa kasalukuyang session at mag-sign in (walang susi)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# O gumamit ng .env file sa direktoryo ng proyekto
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Port Na Ginagamit Na:**
```bash
# Ang Spring Boot ay gumagamit ng port 8080 bilang default
# Baguhin sa application.properties:
server.port=8081
```

### Suporta sa Maramihang Wika
- Dokumentasyon ay makukuha sa 45+ na mga wika sa pamamagitan ng automated na pagsasalin
- Mga pagsasalin nasa direktoryong `translations/`
- Pamamahala ng pagsasalin gamit ang GitHub Actions workflow

### Landas ng Pag-aaral
1. Magsimula sa [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. Sundan ang mga kabanata nang sunud-sunod (01 → 05)
3. Kumpletuhin ang mga hands-on na halimbawa sa bawat kabanata
4. Tuklasin ang mga sample na proyekto sa Kabanata 4
5. Matutunan ang mga responsableng AI na gawi sa Kabanata 5

### Development Container
Inaayos ng `.devcontainer/devcontainer.json` ang:
- Java 21 na development environment
- Maven na naka-install na
- Mga VS Code Java extension
- Mga Spring Boot na tool
- Integrasyon ng GitHub Copilot
- Suporta sa Docker-in-Docker
- Azure CLI

### Mga Pagsasaalang-alang sa Performance
- Ang mga deployment ng Azure AI Foundry ay may mga quota ng token/request kada minuto
- Gamitin ang wastong batch sizes para sa embeddings
- Isaalang-alang ang caching para sa mga paulit-ulit na API calls
- I-monitor ang paggamit ng token para sa cost optimization

### Mga Tala sa Seguridad
- Huwag kailanman i-commit ang `.env` files (nasa `.gitignore` na ito)
- Mas gusto ang keyless auth (Microsoft Entra ID) kaysa sa API keys
- Gumamit ng managed identities sa Azure; `az login` para sa lokal na development
- Sundin ang mga gabay sa responsableng AI sa Kabanata 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Pagtatanggi**:
Ang dokumentong ito ay isinalin gamit ang serbisyo ng AI translation na [Co-op Translator](https://github.com/Azure/co-op-translator). Bagama't nagsusumikap kami para sa katumpakan, pakatandaan na ang awtomatikong pagsasalin ay maaaring maglaman ng mga pagkakamali o hindi pagkakatugma. Ang orihinal na dokumento sa orihinal nitong wika ang dapat ituring na pangunahing sanggunian. Para sa mahahalagang impormasyon, inirerekomenda ang propesyonal na pagsasalin ng tao. Hindi kami mananagot sa anumang maling pagkakaintindi o maling interpretasyon na nagmula sa paggamit ng pagsasaling ito.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->