# AGENTS.md

## Projektoversigt

Dette er et uddannelseslager til læring af Generative AI-udvikling med Java. Det tilbyder et omfattende praktisk kursus, der dækker Large Language Models (LLMs), prompt engineering, embeddings, RAG (Retrieval-Augmented Generation) og Model Context Protocol (MCP).

**Nøgle teknologier:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI og OpenAI SDK’er

**Arkitektur:**
- Flere standalone Spring Boot-applikationer organiseret efter kapitler
- Eksempler på projekter, der demonstrerer forskellige AI-mønstre
- GitHub Codespaces-klar med forudkonfigurerede udviklingscontainere

## Opsætningskommandoer

### Forudsætninger
- Java 21 eller nyere
- Maven 3.x
- Azure-abonnement med en Azure AI Foundry-modeludrulning (opret med `azd up`)
- Azure CLI (`az`) og Azure Developer CLI (`azd`), logget ind til keyless auth

### Miljøopsætning

**Mulighed 1: GitHub Codespaces (Anbefalet)**
```bash
# Forgren repository'et og opret et codespace fra GitHub UI
# Dev-containeren installerer automatisk alle afhængigheder
# Vent ca. 2 minutter på opsætning af miljøet
```

**Mulighed 2: Lokal Dev Container**
```bash
# Klon repository
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Åbn i VS Code med Dev Containers-udvidelse
# Genåbn i container, når du bliver bedt om det
```

**Mulighed 3: Lokal opsætning**
```bash
# Installer afhængigheder
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Bekræft installation
java -version
mvn -version
```

### Konfiguration

**Azure AI Foundry opsætning (keyless, anbefalet):**
```bash
# Provisioner Foundry-kontoen + modelimplementeringer som kode
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd skriver examples/basic-chat-azure/.env med dit endepunkt (ingen nøgle)
```

**Manuel endpoint-konfiguration:**
```bash
# Hvis du ikke brugte azd, skal du selv indstille endepunktet (godkendelse forbliver nøgleløs via az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Rediger .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Udviklingsworkflow

### Projektstruktur
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

### Kørsel af applikationer

**Kør en Spring Boot-applikation:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Byg et projekt:**
```bash
cd [project-directory]
mvn clean install
```

**Start MCP Calculator Server:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# Server kører på http://localhost:8080
```

**Kør klienteksempler:**
```bash
# Efter at have startet serveren i et andet terminalvindue
cd 04-PracticalSamples/calculator

# Direkte MCP-klient
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# AI-drevet klient (kræver AZURE_OPENAI_ENDPOINT + az login)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Interaktiv bot
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Hot Reload
Spring Boot DevTools er inkluderet i projekter, der understøtter hot reload:
```bash
# Ændringer i Java-filer vil automatisk genindlæses, når de gemmes
mvn spring-boot:run
```

## Testinstruktioner

### Kørsel af tests

**Kør alle tests i et projekt:**
```bash
cd [project-directory]
mvn test
```

**Kør tests med detaljeret output:**
```bash
mvn test -X
```

**Kør specifik testklasse:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Teststruktur
- Testfiler bruger JUnit 5 (Jupiter)
- Testklasser ligger i `src/test/java/`
- Klienteksempler i calculator-projektet findes i `src/test/java/com/microsoft/mcp/sample/client/`

### Manuelle tests
Mange eksempler er interaktive applikationer, der kræver manuel test:

1. Start applikationen med `mvn spring-boot:run`
2. Test endpoints eller interagér via CLI
3. Verificér forventet adfærd svarer til dokumentationen i hvert projekts README.md

### Test med Azure AI Foundry
- Log ind med `az login` før kørsel af eksempler (keyless auth)
- Sørg for, at din konto har rollen Cognitive Services OpenAI User på ressourcen
- Test indholdsfiltrering med eksemplet om ansvarlig AI i Kapitel 5

## Kodestil retningslinjer

### Java konventioner
- **Java version:** Java 21 med moderne funktioner
- **Stil:** Følg standard Java konventioner
- **Navngivning:** 
  - Klasser: PascalCase
  - Metoder/variabler: camelCase
  - Konstanter: UPPER_SNAKE_CASE
  - Pakke-navne: små bogstaver

### Spring Boot mønstre
- Brug `@Service` til forretningslogik
- Brug `@RestController` til REST endpoints
- Konfiguration via `application.yml` eller `application.properties`
- Miljøvariabler foretrækkes frem for hardcodede værdier
- Brug `@Tool` annotation for MCP-eksponerede metoder

### Filorganisering
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

### Afhængigheder
- Styres via Maven `pom.xml`
- Spring AI BOM til versionsstyring
- LangChain4j til AI-integrationer
- Spring Boot starter parent til Spring-afhængigheder

### Kodekommentarer
- Tilføj JavaDoc for offentlige APIs
- Inkluder forklarende kommentarer for komplekse AI-interaktioner
- Dokumentér MCP tool-beskrivelser klart

## Byg og Udrulning

### Byg projekter

**Byg uden tests:**
```bash
mvn clean install -DskipTests
```

**Byg med alle checks:**
```bash
mvn clean install
```

**Pak applikation:**
```bash
mvn package
# Opretter JAR i target/-mappen
```

### Output-mapper
- Kompilerede klasser: `target/classes/`
- Testklasser: `target/test-classes/`
- JAR-filer: `target/*.jar`
- Maven-artifakter: `target/`

### Miljøspecifik konfiguration

**Udvikling:**
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

**Produktion:**
- Brug managed identity i stedet for `az login` til keyless auth
- Peg `AZURE_OPENAI_ENDPOINT` på din produktions Foundry-ressource
- Styr konfiguration via miljøvariabler eller Azure Key Vault

### Overvejelser ved udrulning
- Dette er et undervisningslager med eksempelsapplikationer
- Ikke designet til direkte produktion
- Eksempler demonstrerer mønstre, der kan tilpasses produktion
- Se individuelle projekters README’er for specifikke udrulningsnoter

## Yderligere bemærkninger

### Azure AI Foundry
- **Keyless auth:** forbind med Microsoft Entra ID — ingen API-nøgler at administrere
- **Provisoneret som kode:** Bicep + azd (`azd up`) opretter konto og modeludrulninger
- Samme OpenAI-kompatible kode kører lokalt (`az login`) og i Azure (managed identity)

### Arbejde med flere projekter
Hvert eksempelprojekt er standalone:
```bash
# Naviger til specifikt projekt
cd 04-PracticalSamples/[project-name]

# Hver har sin egen pom.xml og kan bygges uafhængigt
mvn clean install
```

### Almindelige problemer

**Java versions mismatch:**
```bash
# Bekræft Java 21
java -version
# Opdater JAVA_HOME, hvis nødvendigt
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Problemer med afhængighedsdownload:**
```bash
# Ryd Maven-cache og prøv igen
rm -rf ~/.m2/repository
mvn clean install
```

**Endpoint eller login ikke fundet:**
```bash
# Indstil slutpunktet i den aktuelle session og log ind (uden nøgle)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Eller brug en .env-fil i projektmappen
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Port allerede i brug:**
```bash
# Spring Boot bruger som standard port 8080
# Ændring i application.properties:
server.port=8081
```

### Understøttelse af flere sprog
- Dokumentation tilgængelig på 45+ sprog via automatisk oversættelse
- Oversættelser i `translations/` mappe
- Oversættelse styres af GitHub Actions workflow

### Læringsvej
1. Start med [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. Følg kapitlerne i rækkefølge (01 → 05)
3. Udfør praktiske eksempler i hvert kapitel
4. Udforsk eksempelsprojekter i Kapitel 4
5. Lær ansvarlig AI-praksis i Kapitel 5

### Udviklingscontainer
`.devcontainer/devcontainer.json` konfigurerer:
- Java 21 udviklingsmiljø
- Maven forudinstalleret
- VS Code Java-udvidelser
- Spring Boot værktøjer
- GitHub Copilot-integration
- Docker-i-Docker understøttelse
- Azure CLI

### Ydelsesovervejelser
- Azure AI Foundry-udrulninger har per-minut token/forespørgselskvoter
- Brug passende batch-størrelser til embeddings
- Overvej caching ved gentagne API-kald
- Overvåg tokenforbrug for omkostningsoptimering

### Sikkerhedsbemærkninger
- Overfør aldrig `.env` filer (allerede i `.gitignore`)
- Foretræk keyless auth (Microsoft Entra ID) fremfor API-nøgler
- Brug managed identities i Azure; `az login` til lokal udvikling
- Følg ansvarlige AI-retningslinjer i Kapitel 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Ansvarsfraskrivelse**:
Dette dokument er blevet oversat ved hjælp af AI-oversættelsestjenesten [Co-op Translator](https://github.com/Azure/co-op-translator). Selvom vi bestræber os på nøjagtighed, skal du være opmærksom på, at automatiserede oversættelser kan indeholde fejl eller unøjagtigheder. Det originale dokument på dets oprindelige sprog bør betragtes som den autoritative kilde. For kritisk information anbefales professionel menneskelig oversættelse. Vi påtager os intet ansvar for misforståelser eller fejltolkninger, der opstår som følge af brugen af denne oversættelse.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->