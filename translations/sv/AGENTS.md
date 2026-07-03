# AGENTS.md

## Projektöversikt

Detta är ett utbildningsförråd för att lära sig utveckling av Generative AI med Java. Det tillhandahåller en omfattande praktisk kurs som täcker stora språkmodeller (LLMs), prompt engineering, embeddings, RAG (Retrieval-Augmented Generation) och Model Context Protocol (MCP).

**Viktiga teknologier:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI och OpenAI SDKs

**Arkitektur:**
- Flera fristående Spring Boot-applikationer organiserade efter kapitel
- Exempelprojekt som demonstrerar olika AI-mönster
- GitHub Codespaces-redo med förkonfigurerade utvecklingscontainer

## Setup-kommandon

### Förutsättningar
- Java 21 eller högre
- Maven 3.x
- Azure-prenumeration med en Azure AI Foundry-modellutplacering (provisionera med `azd up`)
- Azure CLI (`az`) och Azure Developer CLI (`azd`), inloggad för keyless-auth

### Miljöinställning

**Alternativ 1: GitHub Codespaces (Rekommenderat)**
```bash
# Gaffla lagringsplatsen och skapa en codespace från GitHub UI
# Dev-containern installerar automatiskt alla beroenden
# Vänta cirka 2 minuter för miljöinställning
```

**Alternativ 2: Lokal Dev Container**
```bash
# Klona repository
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Öppna i VS Code med Dev Containers-tillägget
# Öppna igen i Container när du uppmanas
```

**Alternativ 3: Lokal Setup**
```bash
# Installera beroenden
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Verifiera installation
java -version
mvn -version
```

### Konfiguration

**Azure AI Foundry Setup (keyless, rekommenderat):**
```bash
# Tillhandahåll Foundry-kontot + modellutplaceringar som kod
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd skriver examples/basic-chat-azure/.env med din slutpunkt (inget nyckel)
```

**Manuell endpoint-konfig:**
```bash
# Om du inte använde azd, ställ in slutpunkten själv (autentisering förblir nyckellös via az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Redigera .env: AZURE_OPENAI_ENDPOINT=https://<resurs>.openai.azure.com/
```

## Utvecklingsarbetsflöde

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

### Köra applikationer

**Köra en Spring Boot-applikation:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Bygga ett projekt:**
```bash
cd [project-directory]
mvn clean install
```

**Starta MCP Calculator Server:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# Servern körs på http://localhost:8080
```

**Köra klientexempel:**
```bash
# Efter att ha startat servern i en annan terminal
cd 04-PracticalSamples/calculator

# Direkt MCP-klient
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# AI-driven klient (kräver AZURE_OPENAI_ENDPOINT + az login)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Interaktiv bot
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Hot Reload
Spring Boot DevTools ingår i projekt som stödjer hot reload:
```bash
# Ändringar i Java-filer kommer automatiskt att laddas om när de sparas
mvn spring-boot:run
```

## Testinstruktioner

### Köra tester

**Kör alla tester i ett projekt:**
```bash
cd [project-directory]
mvn test
```

**Kör tester med detaljerad utskrift:**
```bash
mvn test -X
```

**Kör en specifik testklass:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Teststruktur
- Testfiler använder JUnit 5 (Jupiter)
- Testklasser finns i `src/test/java/`
- Klientexempel i calculator-projektet finns i `src/test/java/com/microsoft/mcp/sample/client/`

### Manuell testning
Många exempel är interaktiva applikationer som kräver manuell testning:

1. Starta applikationen med `mvn spring-boot:run`
2. Testa endpoints eller interagera via CLI
3. Verifiera att förväntat beteende stämmer överens med dokumentationen i varje projekts README.md

### Testning med Azure AI Foundry
- Logga in med `az login` innan du kör exempel (keyless auth)
- Säkerställ att ditt konto har rollen Cognitive Services OpenAI User på resursen
- Testa innehållsfiltrering med ansvarsfull AI-exemplet i Kapitel 5

## Kodstilriktlinjer

### Java-konventioner
- **Java-version:** Java 21 med moderna funktioner
- **Stil:** Följ standard Java-konventioner
- **Namn:**
  - Klasser: PascalCase
  - Metoder/variabler: camelCase
  - Konstanter: UPPER_SNAKE_CASE
  - Paketnamn: gemener

### Spring Boot-mönster
- Använd `@Service` för affärslogik
- Använd `@RestController` för REST-endpoints
- Konfiguration via `application.yml` eller `application.properties`
- Miljövariabler föredras framför hårdkodade värden
- Använd `@Tool`-annotering för MCP-exponerade metoder

### Filorganisation
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

### Beroenden
- Hanteras via Maven `pom.xml`
- Spring AI BOM för versionshantering
- LangChain4j för AI-integrationer
- Spring Boot starter parent för Spring-beroenden

### Kode kommentarer
- Lägg till JavaDoc för publika API:er
- Inkludera förklarande kommentarer för komplexa AI-interaktioner
- Dokumentera MCP-verktygsbeskrivningar tydligt

## Bygg och distribution

### Bygga projekt

**Bygg utan tester:**
```bash
mvn clean install -DskipTests
```

**Bygg med alla kontroller:**
```bash
mvn clean install
```

**Paketera applikation:**
```bash
mvn package
# Skapar JAR i target/ katalogen
```

### Utdatakataloger
- Kompilerade klasser: `target/classes/`
- Testklasser: `target/test-classes/`
- JAR-filer: `target/*.jar`
- Maven-artifakter: `target/`

### Miljöspecifik konfiguration

**Utveckling:**
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
- Använd managed identity istället för `az login` för keyless auth
- Sätt `AZURE_OPENAI_ENDPOINT` till din produktions-Foundry-resurs
- Hantera konfiguration via miljövariabler eller Azure Key Vault

### Distributionsöverväganden
- Detta är ett utbildningsförråd med exempelapplikationer
- Inte designat för produktion så som det är
- Exempel visar mönster att anpassa för produktion
- Se respektive projekts README för specifika distributionsanteckningar

## Ytterligare anteckningar

### Azure AI Foundry
- **Keyless auth:** anslut med Microsoft Entra ID — inga API-nycklar att hantera
- **Provisions-as-code:** Bicep + azd (`azd up`) skapar konto och modellutplaceringar
- Samma OpenAI-kompatibla kod körs lokalt (`az login`) och i Azure (managed identity)

### Arbeta med flera projekt
Varje exempelprojekt är fristående:
```bash
# Navigera till specifikt projekt
cd 04-PracticalSamples/[project-name]

# Var och en har sin egen pom.xml och kan byggas oberoende
mvn clean install
```

### Vanliga problem

**Java-version som inte matchar:**
```bash
# Verifiera Java 21
java -version
# Uppdatera JAVA_HOME vid behov
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Problem med beroendenedladdning:**
```bash
# Rensa Maven-cache och försök igen
rm -rf ~/.m2/repository
mvn clean install
```

**Endpoint eller inloggning saknas:**
```bash
# Sätt slutpunkten i den aktuella sessionen och logga in (nyckellöst)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Eller använd en .env-fil i projektkatalogen
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Port redan upptagen:**
```bash
# Spring Boot använder port 8080 som standard
# Ändra i application.properties:
server.port=8081
```

### Flerspråkigt stöd
- Dokumentation finns på 45+ språk via automatisk översättning
- Översättningar i `translations/`-katalogen
- Översättningshantering via GitHub Actions-workflow

### Lärandestig
1. Börja med [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. Följ kapitel i ordning (01 → 05)
3. Slutför praktiska exempel i varje kapitel
4. Utforska exempelprojekt i Kapitel 4
5. Lär dig ansvarsfull AI i Kapitel 5

### Utvecklingscontainer
`.devcontainer/devcontainer.json` konfigurerar:
- Java 21 utvecklingsmiljö
- Maven förinstallerat
- VS Code Java-tillägg
- Spring Boot-verktyg
- GitHub Copilot-integration
- Docker-in-Docker-stöd
- Azure CLI

### Prestandaöverväganden
- Azure AI Foundry-utplaceringar har per-minut token/förfrågningskvoter
- Använd lämpliga batchstorlekar för embeddings
- Överväg caching vid upprepade API-anrop
- Övervaka tokenanvändning för kostnadsoptimering

### Säkerhetsanteckningar
- Checka aldrig in `.env`-filer (ingår redan i `.gitignore`)
- Föredra keyless auth (Microsoft Entra ID) framför API-nycklar
- Använd managed identities i Azure; `az login` för lokal utveckling
- Följ riktlinjer för ansvarsfull AI i Kapitel 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Ansvarsfriskrivning**:
Detta dokument har översatts med hjälp av AI-översättningstjänsten [Co-op Translator](https://github.com/Azure/co-op-translator). Även om vi strävar efter noggrannhet, var vänlig notera att automatiska översättningar kan innehålla fel eller brister. Det ursprungliga dokumentet på dess modersmål bör betraktas som den auktoritativa källan. För kritisk information rekommenderas professionell mänsklig översättning. Vi ansvarar inte för några missförstånd eller feltolkningar som uppstår till följd av användningen av denna översättning.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->