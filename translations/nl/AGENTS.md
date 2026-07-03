# AGENTS.md

## Projectoverzicht

Dit is een educatieve repository om Generative AI-ontwikkeling met Java te leren. Het biedt een uitgebreide hands-on cursus die Large Language Models (LLM’s), prompt engineering, embeddings, RAG (Retrieval-Augmented Generation) en het Model Context Protocol (MCP) behandelt.

**Belangrijke Technologieën:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI en OpenAI SDK’s

**Architectuur:**
- Meerdere zelfstandige Spring Boot applicaties georganiseerd per hoofdstuk
- Voorbeeldprojecten die verschillende AI-patronen demonstreren
- GitHub Codespaces-ready met vooraf geconfigureerde ontwikkelcontainers

## Installatiecommando’s

### Vereisten
- Java 21 of hoger
- Maven 3.x
- Azure-abonnement met een Azure AI Foundry model-implementatie (voorzie met `azd up`)
- Azure CLI (`az`) en Azure Developer CLI (`azd`), ingelogd voor keyless authenticatie

### Omgevingsinstelling

**Optie 1: GitHub Codespaces (Aanbevolen)**
```bash
# Fork de repository en maak een codespace aan vanuit de GitHub UI
# De dev container zal automatisch alle afhankelijkheden installeren
# Wacht ongeveer 2 minuten voor de omgeving is opgezet
```

**Optie 2: Lokale ontwikkelcontainer**
```bash
# Kloneer opslagplaats
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Open in VS Code met Dev Containers-extensie
# Opnieuw openen in container wanneer gevraagd
```

**Optie 3: Lokale installatie**
```bash
# Installeer afhankelijkheden
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Verifieer installatie
java -version
mvn -version
```

### Configuratie

**Azure AI Foundry Setup (keyless, aanbevolen):**
```bash
# Zorg voor de Foundry-account + modelimplementaties als code
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd schrijft examples/basic-chat-azure/.env met jouw endpoint (geen sleutel)
```

**Handmatige endpointconfiguratie:**
```bash
# Als je azd niet gebruikte, stel dan zelf de endpoint in (auth blijft sleutelvrij via az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Bewerk .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Ontwikkelworkflow

### Projectstructuur
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

### Applicaties draaien

**Een Spring Boot applicatie draaien:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Een project bouwen:**
```bash
cd [project-directory]
mvn clean install
```

**De MCP Calculator Server starten:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# Server draait op http://localhost:8080
```

**Clientvoorbeelden uitvoeren:**
```bash
# Nadat de server is gestart in een ander terminalvenster
cd 04-PracticalSamples/calculator

# Directe MCP-client
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# AI-gestuurde client (vereist AZURE_OPENAI_ENDPOINT + az login)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Interactieve bot
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Hot Reload
Spring Boot DevTools is inbegrepen in projecten die hot reload ondersteunen:
```bash
# Wijzigingen in Java-bestanden worden automatisch opnieuw geladen bij het opslaan
mvn spring-boot:run
```

## Testinstructies

### Tests uitvoeren

**Alle tests in een project uitvoeren:**
```bash
cd [project-directory]
mvn test
```

**Tests met uitgebreide output uitvoeren:**
```bash
mvn test -X
```

**Specifieke testklasse uitvoeren:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Teststructuur
- Testbestanden gebruiken JUnit 5 (Jupiter)
- Testklassen bevinden zich in `src/test/java/`
- Clientvoorbeelden in het calculator-project zijn in `src/test/java/com/microsoft/mcp/sample/client/`

### Handmatig testen
Veel voorbeelden zijn interactieve applicaties die handmatig getest moeten worden:

1. Start de applicatie met `mvn spring-boot:run`
2. Test endpoints of interacteer met de CLI
3. Controleer of het verwachte gedrag overeenkomt met de documentatie in de README.md van elk project

### Testen met Azure AI Foundry
- Log in met `az login` vóór het uitvoeren van voorbeelden (keyless authenticatie)
- Zorg dat je account de rol Cognitive Services OpenAI User heeft op de resource
- Test content filtering met het verantwoordelijke AI voorbeeld in Hoofdstuk 5

## Code Stijl Richtlijnen

### Java Conventies
- **Java Versie:** Java 21 met moderne features
- **Stijl:** Volg standaard Java conventies
- **Naamgeving:**
  - Klassen: PascalCase
  - Methoden/variabelen: camelCase
  - Constants: UPPER_SNAKE_CASE
  - Pakketten: kleine letters

### Spring Boot Patronen
- Gebruik `@Service` voor businesslogica
- Gebruik `@RestController` voor REST endpoints
- Configuratie via `application.yml` of `application.properties`
- Omgevingsvariabelen geven de voorkeur boven hardcoded waarden
- Gebruik `@Tool` annotatie voor MCP-blootgestelde methoden

### Bestandsorganisatie
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

### Afhankelijkheden
- Beheerd via Maven `pom.xml`
- Spring AI BOM voor versiebeheer
- LangChain4j voor AI-integraties
- Spring Boot starter parent voor Spring-afhankelijkheden

### Codecommentaar
- Voeg JavaDoc toe voor publieke API’s
- Voeg verklarende commentaren toe voor complexe AI-interacties
- Documenteer MCP tool omschrijvingen duidelijk

## Bouwen en Implementatie

### Projecten bouwen

**Bouwen zonder tests:**
```bash
mvn clean install -DskipTests
```

**Bouwen met alle controles:**
```bash
mvn clean install
```

**Applicatie packagen:**
```bash
mvn package
# Maakt JAR aan in de target/ map
```

### Outputmappen
- Gecompileerde classes: `target/classes/`
- Testclasses: `target/test-classes/`
- JAR-bestanden: `target/*.jar`
- Maven-artifacts: `target/`

### Omgevingsspecifieke configuratie

**Ontwikkeling:**
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

**Productie:**
- Gebruik een managed identity in plaats van `az login` voor keyless authenticatie
- Verwijs de `AZURE_OPENAI_ENDPOINT` naar uw productie Foundry resource
- Beheer configuratie via omgevingsvariabelen of Azure Key Vault

### Overwegingen bij implementatie
- Dit is een educatieve repository met voorbeeldapplicaties
- Niet ontworpen voor productie-implementatie zoals het is
- Voorbeelden tonen patronen die u kunt aanpassen voor productiegebruik
- Zie de individuele project-README’s voor specifieke implementatie-aantekeningen

## Extra Notities

### Azure AI Foundry
- **Keyless authenticatie:** verbinden met Microsoft Entra ID — geen API-sleutels om te beheren
- **Voorzien als code:** Bicep + azd (`azd up`) maken het account en modelimplementaties aan
- Dezelfde OpenAI-compatibele code draait lokaal (`az login`) en in Azure (managed identity)

### Werken met Meerdere Projecten
Elk voorbeeldproject staat op zichzelf:
```bash
# Navigeer naar een specifiek project
cd 04-PracticalSamples/[project-name]

# Elk heeft zijn eigen pom.xml en kan onafhankelijk worden gebouwd
mvn clean install
```

### Veelvoorkomende Problemen

**Java Versie Onverenigbaar:**
```bash
# Verifieer Java 21
java -version
# Werk JAVA_HOME bij indien nodig
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Problemen met Dependency Download:**
```bash
# Maak de Maven-cache leeg en probeer opnieuw
rm -rf ~/.m2/repository
mvn clean install
```

**Endpoint of Inloggen Niet Gevonden:**
```bash
# Stel het eindpunt in de huidige sessie in en meld aan (zonder sleutel)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Of gebruik een .env-bestand in de projectmap
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Poort Al in Gebruik:**
```bash
# Spring Boot gebruikt standaard poort 8080
# Wijzig in application.properties:
server.port=8081
```

### Meertalige Ondersteuning
- Documentatie beschikbaar in 45+ talen via automatische vertaling
- Vertalingen in de map `translations/`
- Vertaling wordt beheerd door een GitHub Actions workflow

### Leerpad
1. Begin met [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. Volg de hoofdstukken op volgorde (01 → 05)
3. Voltooi hands-on voorbeelden in elk hoofdstuk
4. Verken voorbeeldprojecten in Hoofdstuk 4
5. Leer verantwoordelijke AI-praktijken in Hoofdstuk 5

### Ontwikkelcontainer
De `.devcontainer/devcontainer.json` configureert:
- Java 21 ontwikkelomgeving
- Maven vooraf geïnstalleerd
- VS Code Java extensies
- Spring Boot tools
- GitHub Copilot integratie
- Docker-in-Docker ondersteuning
- Azure CLI

### Prestatie-overwegingen
- Azure AI Foundry-implementaties hebben quota per minuut voor tokens/verzoeken
- Gebruik geschikte batchgroottes voor embeddings
- Overweeg caching voor herhaalde API-aanroepen
- Monitor tokenverbruik voor kostenoptimalisatie

### Beveiligingsnotities
- Commit nooit `.env` bestanden (staan al in `.gitignore`)
- Geef de voorkeur aan keyless authenticatie (Microsoft Entra ID) boven API-sleutels
- Gebruik managed identities in Azure; `az login` voor lokale ontwikkeling
- Volg verantwoordelijke AI-richtlijnen in Hoofdstuk 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Disclaimer**:
Dit document is vertaald met behulp van de AI vertaaldienst [Co-op Translator](https://github.com/Azure/co-op-translator). Hoewel we streven naar nauwkeurigheid, dient u er rekening mee te houden dat geautomatiseerde vertalingen fouten of onnauwkeurigheden kunnen bevatten. Het originele document in de oorspronkelijke taal moet worden beschouwd als de gezaghebbende bron. Voor kritieke informatie wordt professionele menselijke vertaling aanbevolen. Wij zijn niet aansprakelijk voor eventuele misverstanden of verkeerde interpretaties die voortvloeien uit het gebruik van deze vertaling.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->