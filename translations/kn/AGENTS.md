# AGENTS.md

## Project Overview

ಇದು Java ಮೂಲಕ ಶ್ರುತಿ ಹರಿತ ಎಐ ಅಭಿವೃದ್ಧಿಯನ್ನು ಕಲಿಕೆಯೇ ಗುರಿಯಾದ ಒಂದು ಶೈಕ್ಷಣಿಕ ಸಂಗ್ರಹಸ್ಥಳವಾಗಿದೆ. ಇದು ದೊಡ್ಡ ಭಾಷಾ ಮಾದರಿಗಳು (LLMs), ಪ್ರಾಂಪ್ಟ್ ಎಂಜಿನಿಯರಿಂಗ್, ಅಥವಾ ಸಂಯೋಜನೆಗಳು, RAG (ರಿಟ್ರಿವಲ್-ಆಗುಮೆಂಟೆಡ್ ಜನರೇಶನ್), ಮತ್ತು ಮಾದರಿ ಕಾನ್ಟೆಕ್ಸ್ಟ್ ಪ್ರೋಟೋಕಾಲ್ (MCP) ಗಳನ್ನು ಒಳಗೊಂಡ ಸಮಗ್ರ ಪ್ರಾಯೋಗಿಕ ಕೋರ್ಸ್ ಅನ್ನು ಒದಗಿಸುತ್ತದೆ.

**ಪ್ರಮುಖ ತಂತ್ರಜ್ಞಾನಗಳು:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI, ಮತ್ತು OpenAI SDK ಗಳು

**ವಾಸ್ತುಶಿಲ್ಪ:**
- ಅಧ್ಯಾಯಗಳ ಪ್ರಕಾರ ಆಯೋಜಿಸಲಾದ ಬಹು ಏಕಾಂಗಿ Spring Boot ಅಪ್ಲಿಕೇಶನ್ಗಳು
- ವಿಭಿನ್ನ AI ಮಾದರಿಗಳನ್ನು ಪ್ರದರ್ಶಿಸುವ ಮಾದರಿ ಪ್ರಾಜೆಕ್ಟ್ಗಳು
- ಪೂರ್ವ-ಕಾನ್ಫಿಗರ್ ಮಾಡಿದ ಡೆವ್ ಕಂಟೈನರ್ಗಳ ಜೊತೆಗೆ GitHub Codespaces-ಗೆ ಸಿದ್ಧ

## Setup Commands

### Prerequisites
- Java 21 ಅಥವಾ ಅಧಿಕ
- Maven 3.x
- Azure ಸಬ್ಸ್ಕ್ರಿಪ್ಶನ್ ಮತ್ತು Azure AI Foundry ಮಾದರಿ ನಿಯೋಜನೆ (ಪ್ರವೇಶಕ್ಕೆ `azd up` ಬಳಸಿ)
- Azure CLI (`az`) ಮತ್ತು Azure ಡೆವಲಪರ್ CLI (`azd`), ಕೀಲೇಸ್ ಪ್ರామಾಣೀಕರಣಕ್ಕೆ ಸೈನ್ ಇನ್ ಆಗಿರಬೇಕು

### Environment Setup

**ಆಯ್ಕೆ 1: GitHub Codespaces (ಶಿಫಾರಸು ಮಾಡಿ)**
```bash
# ರೆಪೊಸಿಟರಿಯನ್ನು ಫೋರ್ಕ್ ಮಾಡಿ ಮತ್ತು GitHub UI ನಿಂದ ಕೋಡ್ಸ್‌ಪೇಸ್ ಅನ್ನು ರಚಿಸಿ
# ಡೆವ್ ಕಂಟೈನರ್ ಸ್ವಯಂಚಾಲಿತವಾಗಿ ಎಲ್ಲಾ ಅವಲಂಬನೆಗಳನ್ನು ಸ್ಥಾಪಿಸಿ
# ಪರಿಸರ ವ್ಯವಸ್ಥೆಗೆ ~2 ನಿಮಿಷಗಳು ಕಾಯಿರಿ
```

**ಆಯ್ಕೆ 2: ಸ್ಥಳೀಯ ಡೆವ್ ಕಂಟೈನರ್**
```bash
# ರೆಪೊಸಿಟರಿಯನ್ನು ಕ್ಲೋನ್ ಮಾಡಿ
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# ಡೆವ್ ಕಂಟೈನರ್ ವಿಸ್ತರಣೆೊಂದಿಗೆ VS ಕೋಡ್ ನಲ್ಲಿ ತೆರೆಯಿರಿ
# ಕೇಳಿಸಿದಾಗ ಕಂಟೈನರ್ ನಲ್ಲಿ ಪುನಃ ತೆರೆಯಿರಿ
```

**ಆಯ್ಕೆ 3: ಸ್ಥಳೀಯ ಸೆಟಪ್**
```bash
# ಅವಲಂಬನೆಗಳನ್ನು ಸ್ಥಾಪಿಸಿ
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# ಸ್ಥಾಪನೆ ಪರಿಶೀಲಿಸಿ
java -version
mvn -version
```

### Configuration

**Azure AI Foundry ಸೆಟಪ್ (ಕೀಲೇಸ್, ಶಿಫಾರಸು):**
```bash
# ಫೌಂಡ್ರಿ ಖಾತೆ + ಮಾದರಿ ನಿಯೋಜನೆಗಳನ್ನು ಕೋಡ್ ಆಗಿ ಒದಗಿಸಿ
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd ನಿಮ್ಮ ಎಂಡ್ಪಾಯಿಂಟ್ (ಕೀ ಇಲ್ಲ) ಸಹಿತ examples/basic-chat-azure/.env ಅನ್ನು ಬರೆಯುತ್ತದೆ
```

**ಮ್ಯಾನ್‌ಯುಯಲ್ ಎಂಡ್‌ಪಾಯಿಂಟ್ ನಿರ್ಮಾಣ:**
```bash
# ನೀವು azd ಬಳಸದಿದ್ದರೆ, ಸ್ವತಃ ಎಂಡ್ ಪಾಯಿಂಟ್ ಅನ್ನು ಸೆಟ್ ಮಾಡಿ (ಪ್ರಾಮಾಣಿಕತೆ az login ಮೂಲಕ ಕೀಲಿಕೋಲಿಕೆಯಿಲ್ಲದೇ ಇರುತ್ತದೆ)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# .env ಸಂಪಾದಿಸಿ: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
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

**Spring Boot ಅಪ್ಲಿಕೇಶನ್ ಚಲಾಯಿಸುವುದು:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**ಪ್ರಾಜೆಕ್ಟ್ ಬಿಲ್ಡ್ ಮಾಡುವುದು:**
```bash
cd [project-directory]
mvn clean install
```

**MCP ಕ್ಯಾಲ್ಕುಲೇಟರ್ ಸರ್ವರ್ ಪ್ರಾರಂಭಿಸುವುದು:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# ಸರ್ವರ್ http://localhost:8080 ನ್ನು ಆವರಿಸುತ್ತದೆ
```

**ಕ್ಲಯಿಂಟ್ ಉದಾಹರಣೆಗಳನ್ನು ಚಲಾಯಿಸುವುದು:**
```bash
# ಇನ್ನೊಂದು ಟರ್ಮಿನಲ್‌ನಲ್ಲಿ ಸರ್ವರ್ ಪ್ರಾರಂಭಿಸಿದ ನಂತರ
cd 04-PracticalSamples/calculator

# ನೇರ MCP ಕ್ಲೈಂಟ್
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# AI-ಚಾಲಿತ ಕ್ಲೈಂಟ್ (AZURE_OPENAI_ENDPOINT + az ಲಾಗಿನ್ ಅಗತ್ಯವಿದೆ)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# ಅಂತರಾಷ್ಟ್ರೀಯ ಬಾಟ್
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### ಹಾಟ್ ರೀಲೋಡ್
Spring Boot DevTools ಅನ್ನು ಹಾಟ್ ರೀಲೋಡ್ಗೆ ಬೆಂಬಲಿಸುವ ಪ್ರಾಜೆಕ್ಟ್ಗಳು ಸೇರಿಸಿಕೊಂಡಿವೆ:
```bash
# ಜಾವಾ ಕಡತಗಳಲ್ಲಿ ಮಾಡಲಾದ ಬದಲಾವಣೆಗಳು ಉಳಿಸಿದ 순간 ಸ್ವಯಂಚಾಲಿತವಾಗಿ ಮರುಲೋಡ್ ಆಗುತ್ತವೆ
mvn spring-boot:run
```

## Testing Instructions

### ಚಾಲನೆ ಪರೀಕ್ಷೆಗಳು

**ಒಂದಕ್ಕೆ ಎಲ್ಲಾ ಪರೀಕ್ಷೆಗಳನ್ನು ನಡೆಸುವುದು:**
```bash
cd [project-directory]
mvn test
```

**ವಿವರಣಾತ್ಮಕ ಔಟ್‌ಪುಟ್ ಜೊತೆ ಪರೀಕ್ಷೆಗಳನ್ನು ನಡೆಸುವುದು:**
```bash
mvn test -X
```

**ನಿರ್ದಿಷ್ಟ ಪರೀಕ್ಷಾ ವರ್ಗವನ್ನು ಚಲಾಯಿಸುವುದು:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### ಪರೀಕ್ಷಾ ಸಂರಚನೆ
- ಪರೀಕ್ಷಾ ಫೈಲ್‌ಗಳು JUnit 5 (ಜುಪಿಟರ್) ಬಳಸುತ್ತವೆ
- ಪರೀಕ್ಷಾ ವರ್ಗಗಳು `src/test/java/` ನಲ್ಲಿವೆ
- ಕ್ಯಾಲ್ಕುಲೇಟರ್ ಪ್ರಾಜೆಕ್ಟಿನ ಕ್ಲಯಿಂಟ್ ಉದಾಹರಣೆಗಳು `src/test/java/com/microsoft/mcp/sample/client/` ನಲ್ಲಿವೆ

### ಕೈಯಿಂದ ಪರೀಕ್ಷೆ
ಹೆಚ್ಛಿನ ಉದಾಹರಣೆಗಳು ಕೈಯಿಂದ ಪರೀಕ್ಷೆ ಅಗತ್ಯವಿರುವ ಅಂತರ್‌ಕ್ರಿಯಾತ್ಮಕ ಅಪ್ಲಿಕೇಶನ್ಗಳು:

1. `mvn spring-boot:run` ಮೂಲಕ ಅಪ್ಲಿಕೇಶನ್ ಪ್ರಾರಂಭಿಸಿ
2. ಎಂಡ್ಪಾಯಿಂಟ್‌ಗಳು ಅಥವಾ CLI ಜೊತೆ ಪರೀಕ್ಷೆ ನಡೆಸಿ
3. ಪ್ರತಿ ಪ್ರಾಜೆಕ್ಟಿನ README.md ನಲ್ಲಿ ದಾಖಲೆ ಇದ್ದಂತೆ ನಿರೀಕ್ಷಿತ ವರ್ತನೆ ಪರಿಶೀಲಿಸಿ

### Azure AI Foundry ಜೊತೆಗೆ ಪರೀಕ್ಷೆ
- ಉದಾಹರಣೆಗಳನ್ನು ಚಾಲನೆಮಾಡೋ ಮೊದಲು `az login` ಮೂಲಕ ಸೈನ್ ಇನ್ ಆಗಿ (ಕೀಲೇಸ್ ಪ್ರಾಮಾಣೀಕರಣ)
- ನಿಮ್ಮ ಖಾತೆಗೆ ಕಾಗ್ನಿಟಿವ್ ಸರ್ವಿಸಸ್ OpenAI ಯೂಸರ್ ಪಾತ್ರ ಹೊಂದಿರುವುದನ್ನು ಖಚಿತಪಡಿಸಿಕೊಳ್ಳಿ
- ಅಧ್ಯಾಯ 5 ರಲ್ಲಿ ಜವಾಬ್ದಾರಿ ಏಐ ಉದಾಹರಣೆಯಲ್ಲಿ ವಿಷಯ ಫಿಲ್ಟರಿಂಗ್ ಪರೀಕ್ಷಿಸಿ

## Code Style Guidelines

### Java Conventions
- **Java ಆವೃತ್ತಿ:** ನವೀನ ವೈಶಿಷ್ಟ್ಯಗಳೊಂದಿಗೆ Java 21
- **ಶೈಲಿ:** ಸ್ಟಾಂಡರ್ಡ್ ಜಾವಾ ಸಮರ್ಪಕರು ಅನುಸರಿಸಿ
- **ಹೆಸರುಕರಣ:** 
  - ವರ್ಗಗಳು: PascalCase
  - ವಿಧಾನಗಳು/ಚರಗಳು: camelCase
  - ಸ್ಥಿರಾಂಕಗಳು: UPPER_SNAKE_CASE
  - ಪ್ಯಾಕೇಜ್ ಹೆಸರುಗಳು: ಸಣ್ಣಕ್ಷರ

### Spring Boot ಮಾದರಿಗಳು
- ವ್ಯಾಪಾರ ಲಾಜಿಕ್ಗಾಗಿ `@Service` ಬಳಸುವುದು
- REST ಎಂಡ್ಪಾಯಿಂಟ್‌ಗಳಿಗೆ `@RestController` ಬಳಸುವುದು
- ಸಂರಚನೆಗೆ `application.yml` ಅಥವಾ `application.properties` ಬಳಸಿ
- ಹಾರ್ಡ್-ಕೋಡ್ ಮೌಲ್ಯಗಳಿಗಿಂತ ಪರಿಸರ চলಕಗಳು ಪ್ರಾಧಾನ್ಯ
- MCP-ಪ್ರಕಟಿಸಲ್ಪಟ್ಟ ವಿಧಾನಗಳಿಗೆ `@Tool` ಅನೋಟೇಶನ್ ಬಳಸಿ

### ಫೈಲ್ ಸಂರಚನೆ
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

### ಅವಲಂಬನೆಗಳು
- Maven `pom.xml` ಮೂಲಕ ನಿರ್ವಹಣೆ
- Spring AI BOM ಆವೃತ್ತಿ ನಿರ್ವಹಣೆಗೆ
- AI ಸಂಯೋಜನೆಗಳಿಗೆ LangChain4j
- Spring ಅವಲಂಬನೆಗಳಿಗೆ Spring Boot ಸ್ಟಾರ್ಟರ್ ಪ್ಯಾರೆಂಟ್

### ಕೋಡ್ ಟಿಪ್ಪಣಿಗಳು
- ಸಾರ್ವಜನಿಕ API ಗಳಿಗಾಗೀ JavaDoc ಸೇರಿಸಿ
- ಜಟಿಲ AI ಸಂವಹನಗಳಿಗೆ ವಿವರಾತ್ಮಕ ಟಿಪ್ಪಣಿಗಳು ಸೇರಿಸಿ
- MCP ಉಪಕರಣ ವಿವರಣೆಗಳನ್ನು ಸ್ಪಷ್ಟವಾಗಿ ದಾಖಲಿಸಿ

## Build and Deployment

### ಪ್ರಾಜೆಕ್ಟ್ ಕಟ್ಟುವಿಕೆ

**ಪರೀಕ್ಷೆಗಳು ಇಲ್ಲದೆ ನಿರ್ಮಿಸುವುದು:**
```bash
mvn clean install -DskipTests
```

**ಎಲ್ಲಾ ತಪಾಸಣೆಯೊಂದಿಗೆ ನಿರ್ಮಿಸುವುದು:**
```bash
mvn clean install
```

**ಅಪ್ಲಿಕೇಶನ್ ಪ್ಯಾಕೇಜ್ ಮಾಡುವುದು:**
```bash
mvn package
# target/ ಡೈರೆಕ್ಟರಿಯಲ್ಲಿ JAR ರಚಿಸುತ್ತದೆ
```

### ಔಟ್‌ಪುಟ್ ಡೈರೆಕ್ಟರಿಗಳು
- ಸಂಕಲಿತ ವರ್ಗಗಳು: `target/classes/`
- ಪರೀಕ್ಷಾ ವರ್ಗಗಳು: `target/test-classes/`
- JAR ಫೈಲ್‌ಗಳು: `target/*.jar`
- Maven ಉತ್ಪನ್ನಗಳು: `target/`

### ಪರಿಸರ-ವಿಶೇಷ ಸಂರಚನೆ

**ಅಭಿವೃದ್ಧಿ:**
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

**ಉತ್ಪಾದನೆ:**
- ಕೀಲೇಸ್ ಪ್ರಾಮಾಣೀಕರಣಕ್ಕೆ `az login` ಬದಲಾಗಿ ಮ್ಯಾನೇಜ್ ಮಾಡಿದ ಐಡೆಂಟಿಟಿ ಬಳಸಿ
- ನಿಮ್ಮ ಉತ್ಪಾದನಾ ಫೌಂಡ್ರಿ ಸಂಪನ್ಮೂಲವನ್ನು `AZURE_OPENAI_ENDPOINT` ನೊಂದಿಗೆ ಸೂಚಿಸಿ
- ಸಂರಚನೆಯನ್ನು ಪರಿಸರ ಚರಗಳು ಅಥವಾ Azure ಕೀ ವಾಲ್ಟ್ ಮೂಲಕ ನಿರ್ವಹಿಸಿ

### ನಿಯೋಜನೆ ಪರಿಗಣನೆಗಳು
- ಇದು ಮಾದರಿ ಅಪ್ಲಿಕೇಶನ್‌ಗಳೊಂದಿಗೆ ಶೈಕ್ಷಣಿಕ ಸಂಗ್ರಹಸ್ಥಳವಾಗಿದೆ
- ತಕ್ಷಣದ ಉತ್ಪಾದನಾ ನಿಯೋಜನೆಗಾಗಿ ವಿನ್ಯಾಸಗೊಳಿಸಿ ಇಲ್ಲ
- ಮಾದರಿ ವಿನ್ಯಾಸಗಳು ಉತ್ಪಾದನೆಗಾಗಿ ಹೊಂದಿಕೊಡುವುದಕ್ಕೆ ಕಾಣಿಸುತ್ತದೆ
- ವೈಯಕ್ತಿಕ ಪ್ರಾಜೆಕ್ಟ್ README ಗಳಲ್ಲಿ ನಿಯೋಜನೆ ಸೂಚನೆಗಳನ್ನು ನೋಡಿ

## Additional Notes

### Azure AI Foundry
- **ಕೀಲೇಸ್ ಪ್ರಾಮಾಣೀಕರಣ:**Microsoft Entra ID ಮೂಲಕ ಸಂಪರ್ಕ, ಯಾವುದೇ API ಕೀ ನಿರ್ವಹಣೆ ಅಗತ್ಯವಿಲ್ಲ
- **ಕೋಡ್ ಆಗ provision:** ಬೈಸಪ್ + azd (`azd up`) ಖಾತೆ ಮತ್ತು ಮಾದರಿ ನಿಯೋಜನೆಗಳನ್ನು ರಚಿಸುತ್ತದೆ
- ಒಂದೇ OpenAI-ಅನುರೂಪ ಕೋಡ್ ಸ್ಥಳೀಯವಾಗಿ (`az login`) ಮತ್ತು Azure (ಮ್ಯಾನೇಜ್ಡ್ ಐಡೆಂಟಿಟಿ) ನಲ್ಲಿ ಕಾರ್ಯನಿರ್ವಹಿಸುತ್ತದೆ

### ಬಹು ಪ್ರಾಜೆಕ್ಟ್ ಜೊತೆಗೆ ಕೆಲಸ ಮಾಡುವುದು
ಪ್ರತಿ ಮಾದರಿ ಪ್ರಾಜೆಕ್ಟು ಸ್ವತಂತ್ರವಾಗಿದೆ:
```bash
# ವಿಶೇಷ ಯೋಜನೆಗೆ ಹಾದು ಮುನ್ನುಗ್ಗಿ
cd 04-PracticalSamples/[project-name]

# ಪ್ರತಿಯೊಬ್ಬರಿಗೂ ತಮ್ಮದೇ ಪಮ್.xml ಇರುತ್ತದೆ ಮತ್ತು ಸ್ವತಂತ್ರವಾಗಿ ನಿರ್ಮಿಸಬಹುದು
mvn clean install
```

### ಸಾಮಾನ್ಯ ಸಮಸ್ಯೆಗಳು

**Java ಆವೃತ್ತಿ ಅಸಮ್ಮತತೆ:**
```bash
# ಜಾವಾ 21 ಪರಿಶೀಲಿಸಿ
java -version
# ಅಗತ್ಯವಿದ್ದರೆ JAVA_HOME ನವೀಕರಿಸಿ
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**ಅವಲಂಬನೆ ಡೌನ್‌ಲೋಡ್ ಸಮಸ್ಯೆಗಳು:**
```bash
# Maven ಕ್ಯಾಶೆ ಸ್ಪಷ್ಟಗೊಳಿಸಿ ಮತ್ತು ಮರುಪ್ರಯತ್ನಿಸಿ
rm -rf ~/.m2/repository
mvn clean install
```

**ಎಂಡ್‌ಪಾಯಿಂಟ್ ಅಥವಾ ಸೈನ್-ಇನ್ ಸಿಗದು:**
```bash
# ಪ್ರಸ್ತುತ ಸೆಷನ್‌ನಲ್ಲಿ ಎಂಡ್‌ಪಾಯಿಂಟ್ ಅನ್ನು ಸೆಟ್ ಮಾಡಿ ಮತ್ತು ಸೈನ್ ಇನ್ ಮಾಡಿ (ಕೀಲೆಸ್)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# ಅಥವಾ ಪ್ರಾಜೆಕ್ಟ್ ಡೈರೆಕ್ಟರಿಯಲ್ಲಿನ .env ಫೈಲ್ ಅನ್ನು ಬಳಸಿ
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**ಪೋರ್ಟ್ ಈಗಾಗಲೇ ಬಳಕೆಯಲ್ಲಿದೆ:**
```bash
# Spring Boot ಡೀಫಾಲ್ಟ್‌ಾಗಿ ಪೋರ್ಟ್ 8080 ಅನ್ನು ಬಳಸುತ್ತದೆ
# application.properties ನಲ್ಲಿ ಬದಲಾವಣೆ:
server.port=8081
```

### ಬಹು ಭಾಷಾ ಬೆಂಬಲ
- 45+ ಭಾಷೆಗಳಲ್ಲಿ ಸ್ವಯಂಚಾಲಿತ ಭಾಷಾಂತರದಿಂದ ಲಭ್ಯವಿದೆ
- `translations/` ಡೈರೆಕ್ಟರಿಯಲ್ಲಿ ಭಾಷಾಂತರಗಳು
- ಭಾಷಾಂತರವನ್ನು GitHub Actions ವರ್ಕ್‌ಫ್ಲೋ ಮೂಲಕ ನಿರ್ವಹಿಸಲಾಗಿದೆ

### ಕಲಿಕೆ ಮಾರ್ಗ
1. [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)ೊಂದಿಗೆ ಪ್ರಾರಂಭಿಸಿ
2. ಅಧ್ಯಾಯಗಳನ್ನು ಕ್ರಮಬದ್ಧವಾಗಿ ಅನುಸರಿಸಿ (01 → 05)
3. ಪ್ರತಿ ಅಧ್ಯಾಯದಲ್ಲಿ ಪ್ರಾಯೋಗಿಕ ಉದಾಹರಣೆಗಳನ್ನು ಪೂರ್ಣಗೊಳಿಸಿ
4. ಅಧ್ಯಾಯ 4 ರಲ್ಲಿ ಮಾದರಿ ಪ್ರಾಜೆಕ್ಟ್ಗಳನ್ನು ಅನ್ವೇಷಿಸಿ
5. ಅಧ್ಯಾಯ 5 ರಲ್ಲಿ ಜವಾಬ್ದಾರಿಯುತ AI ನಿಟ್ಟಿನಲ್ಲಿ ಕಲಿಕೆ

### ಡೆವಲಪ್ಮೆಂಟ್ ಕಂಟೈನರ್
`.devcontainer/devcontainer.json` ಈ ಕೆಳಗಿನವುಗಳನ್ನು ಸಂರಚಿಸುತ್ತದೆ:
- Java 21 ಅಭಿವೃದ್ಧಿ ಪರಿಸರ
- ಪೂರ್ವ-ಸ್ಥಾಪಿತ Maven
- VS Code Java ವಿಸ್ತರಣೆಗಳು
- Spring Boot ಉಪಕರಣಗಳು
- GitHub Copilot ಸಂಯೋಜನೆ
- Docker-in-Docker ಬೆಂಬಲ
- Azure CLI

### ಪ್ರದರ್ಶನ ಪರಿಗಣನೆಗಳು
- Azure AI Foundry ನಿಯೋಜನೆಗಳು ನಿಮಿಷಕ್ಕೊಂದು ಟೋಕನ್/ವಿನಂತಿ ಮಿತಿಯನ್ನು ಹೊಂದಿವೆ
- ಸಂಯೋಜನೆಗಳಿಗೆ ಸೂಕ್ತ ಬ್ಯಾಚ್ ಗಾತ್ರಗಳನ್ನು ಉಪಯೋಗಿಸಿ
- ಪುನರಾವರ್ತಿತ API ಕರೆಗಳಿಗಾಗಿ ಕ್ಯಾಶಿಂಗ್ ಪರಿಗಣಿಸಿ
- ವೆಚ್ಚದಿಗಾಗಿ ಟೋಕನ್ ಬಳಕೆಯನ್ನು ನಿಗಾ ವಹಿಸಿ

### ಭದ್ರತಾ ಟಿಪ್ಪಣಿಗಳು
- `.env` ಫೈಲ್‌ಗಳನ್ನು ಎಂದಿಗೂ ಕಮಿಟ್ ಮಾಡಬೇಡಿ (`.gitignore` ನಲ್ಲಿ ಸೇರಿವೆ)
- ಕೀಗಳಿಗಿಂತ ಕೀಲೇಸ್ ಪ್ರಾಮಾಣೀಕರಣ (Microsoft Entra ID) ಅನ್ನು ಆದ್ಯತೆ ನೀಡಿ
- ಸ್ಥಳೀಯ ಅಭಿವೃದ್ಧಿಗಾಗಿ Azure ನಲ್ಲಿ ಮ್ಯಾನೇಜ್ ಮಾಡಲಾಗುವ ಐಡೆಂಟಿಟಿಗಳೊಂದಿಗೆ `az login` ಬಳಸಿ
- ಅಧ್ಯಾಯ 5 ರಲ್ಲಿ ಜವಾಬ್ದಾರಿಯುತ AI ಮಾರ್ಗಸೂಚಿಗಳನ್ನು ಅನುಸರಿಸಿ

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ಅಸ್ವೀಕಾರ**:
ಈ ದಸ್ತಾವೇಜು AI ಅನುವಾದ ಸೇವೆ [Co-op Translator](https://github.com/Azure/co-op-translator) ಬಳಸಿ ಅನುವಾದಿಸಲಾಗಿದೆ. ನಾವು ನಿಖರತೆಯನ್ನು ಸಾಧಿಸಲು ಪ್ರಯತ್ನಿಸುತ್ತಿದ್ದರೂ, ದಯವಿಟ್ಟು ಗಮನಿಸಿ, ಸ್ವಯಂಚಾಲಿತ ಅನುವಾದಗಳಲ್ಲಿ ದೋಷಗಳು ಅಥವಾ ಅಸಡ್ಡೆಗಳು ಇರಬಹುದು. ಮೂಲ ಭಾಷೆಯಲ್ಲಿರುವ ಮೂಲ ದಸ್ತಾವೇಜು ಪ್ರಾಮಾಣಿಕ ಮೂಲವೆಂದು ಪರಿಗಣಿಸಬೇಕು. ಪ್ರಮುಖ ಮಾಹಿತಿಗಾಗಿ, ವೃತ್ತಿಪರ ಮಾನವ ಅನುವಾದವನ್ನು ಶಿಫಾರಸು ಮಾಡಲಾಗುತ್ತದೆ. ಈ ಅನುವಾದವನ್ನು ಬಳಸುವ ಮೂಲಕ ಉಂಟಾಗುವ ಯಾವುದೇ ತಪ್ಪು ಅರ್ಥಗಳ ಅಥವಾ ತಪ್ಪು ವ್ಯಾಖ್ಯಾನಗಳ ಬಗ್ಗೆ ನಾವು ಹೊಣೆಗಾರರಲ್ಲ.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->