# AGENTS.md

## Project Overview

Dis na educashonal repo wey you fit use learn Generative AI development with Java. E dey give una complete hands-on course wey cover Large Language Models (LLMs), prompt engineering, embeddings, RAG (Retrieval-Augmented Generation), plus di Model Context Protocol (MCP).

**Key Technologies:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI, and OpenAI SDKs

**Architecture:**
- Plenty standalone Spring Boot apps organized by chapters
- Sample projects wey show different AI patterns
- GitHub Codespaces-ready with pre-configured dev containers

## Setup Commands

### Prerequisites
- Java 21 or pass
- Maven 3.x
- Azure subscription wey get Azure AI Foundry model deployment (make you deploy with `azd up`)
- Azure CLI (`az`) and Azure Developer CLI (`azd`), you don sign for keyless auth

### Environment Setup

**Option 1: GitHub Codespaces (Recommended)**
```bash
# Fork di repository den create a codespace from GitHub UI
# Di dev container go automatically install all dependencies
# Wait about 2 minutes for environment setup
```

**Option 2: Local Dev Container**
```bash
# Clone di repository
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Open am for VS Code wit Dev Containers extension
# Reopen am for Container wen e ask you
```

**Option 3: Local Setup**
```bash
# Install all di tins wey e need
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Make sure say e don install well well
java -version
mvn -version
```

### Configuration

**Azure AI Foundry Setup (keyless, recommended):**
```bash
# Set up di Foundry account + model deployments as code
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd dey write examples/basic-chat-azure/.env wit your endpoint (no key)
```

**Manual endpoint config:**
```bash
# If you no use azd, set di endpoint yourself (auth go still keyless if you use az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Edit .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
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

**How to run Spring Boot app:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Building project:**
```bash
cd [project-directory]
mvn clean install
```

**Start MCP Calculator Server:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# Server dey run for http://localhost:8080
```

**Run Client Examples:**
```bash
# After you don start di server for anoda terminal
cd 04-PracticalSamples/calculator

# Direct MCP client
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# AI-powered client (you go need AZURE_OPENAI_ENDPOINT + az login)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Interactive bot
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Hot Reload
Spring Boot DevTools dey inside projects wey support hot reload:
```bash
# Changes wey you do for Java files go automatically reload wen you save am
mvn spring-boot:run
```

## Testing Instructions

### Running Tests

**Run all tests for project:**
```bash
cd [project-directory]
mvn test
```

**Run tests with detailed output:**
```bash
mvn test -X
```

**Run specific test class:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Test Structure
- Test files dey use JUnit 5 (Jupiter)
- Test classes dey `src/test/java/`
- Client examples for calculator project dey `src/test/java/com/microsoft/mcp/sample/client/`

### Manual Testing
Plenty examples na interactive apps wey need manual testing:

1. Start app with `mvn spring-boot:run`
2. Test endpoints or interact with CLI
3. Check say wetin happen follow wetin README.md talk for each project

### Testing with Azure AI Foundry
- Sign in with `az login` before you run examples (keyless auth)
- Make sure say your account get Cognitive Services OpenAI User role for the resource
- Test content filtering with responsible AI example for Chapter 5

## Code Style Guidelines

### Java Conventions
- **Java Version:** Java 21 with new features
- **Style:** Follow normal Java conventions
- **Naming:** 
  - Classes: PascalCase
  - Methods/variables: camelCase
  - Constants: UPPER_SNAKE_CASE
  - Package names: lowercase

### Spring Boot Patterns
- Use `@Service` for business logic
- Use `@RestController` for REST endpoints
- Configure by `application.yml` or `application.properties`
- Environment variables better pass hardcoded values
- Use `@Tool` annotation for MCP-exposed methods

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
- Manage with Maven `pom.xml`
- Spring AI BOM for version control
- LangChain4j for AI plugins
- Spring Boot starter parent for Spring dependencies

### Code Comments
- Add JavaDoc for public APIs
- Put comments wey explain complex AI work
- Document MCP tool descriptions well well

## Build and Deployment

### Building Projects

**Build without run test:**
```bash
mvn clean install -DskipTests
```

**Build with all checks:**
```bash
mvn clean install
```

**Package application:**
```bash
mvn package
# Dey create JAR inside target/ folder
```

### Output Directories
- Compiled classes: `target/classes/`
- Test classes: `target/test-classes/`
- JAR files: `target/*.jar`
- Maven artifacts: `target/`

### Environment-Specific Configuration

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

**Production:**
- Use managed identity instead of `az login` for keyless auth
- Point `AZURE_OPENAI_ENDPOINT` to your production Foundry resource
- Manage config with environment variables or Azure Key Vault

### Deployment Considerations
- Na educational repo with sample apps
- No design for production deployment as e be
- Samples dey show patterns to fit adapt for production
- Check each project README for deployment notes

## Additional Notes

### Azure AI Foundry
- **Keyless auth:** connect with Microsoft Entra ID — no API keys to manage
- **Provisioned as code:** Bicep + azd (`azd up`) create account and model deployments
- Same OpenAI-compatible code dey run locally (`az login`) and for Azure (managed identity)

### Working with Multiple Projects
Each sample project na standalone:
```bash
# Comot go one particular project
cd 04-PracticalSamples/[project-name]

# Each get im own pom.xml wey dem fit build by imsef
mvn clean install
```

### Common Issues

**Java Version Mismatch:**
```bash
# Make sure say Java 21 correct
java -version
# Change JAVA_HOME if e need am
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Dependency Download Problems:**
```bash
# Clear Maven cache and try again
rm -rf ~/.m2/repository
mvn clean install
```

**Endpoint or Sign-in Missing:**
```bash
# Set di endpoint for di current session and sign in (no need key)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Or use one .env file for di project directory
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Port Already Use:**
```bash
# Spring Boot dey use port 8080 by default
# Change for application.properties:
server.port=8081
```

### Multi-Language Support
- Documentation dey for 45+ languages through automated translation
- Translations dey `translations/` folder
- Translation controlled by GitHub Actions workflow

### Learning Path
1. Start with [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. Follow chapters one by one (01 → 05)
3. Complete hands-on examples inside each chapter
4. Explore sample projects for Chapter 4
5. Learn responsible AI behavior for Chapter 5

### Development Container
The `.devcontainer/devcontainer.json` dey setup:
- Java 21 development environment
- Maven pre-installed
- VS Code Java extensions
- Spring Boot tools
- GitHub Copilot integration
- Docker-in-Docker support
- Azure CLI

### Performance Considerations
- Azure AI Foundry deployments get per-minute token/request limits
- Use correct batch sizes for embeddings
- Think about caching for repeat API calls
- Monitor token use to save cost

### Security Notes
- No ever commit `.env` files (dem dey `.gitignore` already)
- Better use keyless auth (Microsoft Entra ID) pass API keys
- Use managed identities in Azure; `az login` for local development
- Follow responsible AI rules for Chapter 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Disclaimer**:
Dis document don translate wit AI translation service [Co-op Translator](https://github.com/Azure/co-op-translator). Even tho we dey try make am correct, abeg make you know say automated translation fit get errors or mistakes. Di original document for dia own language na im be di correct source. For important info, make person wey sabi human translation do am. We no go responsible for any misunderstanding or wrong understanding wey fit happen because of dis translation.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->