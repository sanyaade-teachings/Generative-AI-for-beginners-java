# AGENTS.md

## Prosjektoversikt

Dette er et utdanningsrepo for å lære Generative AI-utvikling med Java. Det gir et omfattende praktisk kurs som dekker Store Språkmodeller (LLMs), prompt engineering, embeddings, RAG (Retrieval-Augmented Generation), og Model Context Protocol (MCP).

**Nøkkelteknologier:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI, og OpenAI SDK-er

**Arkitektur:**
- Flere frittstående Spring Boot-applikasjoner organisert etter kapitler
- Eksempelsprosjekter som demonstrerer ulike AI-mønstre
- GitHub Codespaces-klare med forhåndskonfigurerte utviklingscontainere

## Oppsettskommandoer

### Forutsetninger
- Java 21 eller nyere
- Maven 3.x
- Azure-abonnement med en Azure AI Foundry modell-deployering (provisjon med `azd up`)
- Azure CLI (`az`) og Azure Developer CLI (`azd`), pålogget for nøkkelfri autentisering

### Miljøoppsett

**Alternativ 1: GitHub Codespaces (anbefalt)**
```bash
# Forgrening av depotet og opprett en codespace fra GitHub-brukergrensesnittet
# Utviklingscontaineren vil automatisk installere alle avhengigheter
# Vent omtrent 2 minutter for oppsett av miljøet
```

**Alternativ 2: Lokal utviklingscontainer**
```bash
# Klon depotet
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Åpne i VS Code med Dev Containers-utvidelsen
# Åpne på nytt i beholder når du blir spurt
```

**Alternativ 3: Lokal oppsett**
```bash
# Installer avhengigheter
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Verifiser installasjonen
java -version
mvn -version
```

### Konfigurasjon

**Azure AI Foundry oppsett (nøkkelfri, anbefalt):**
```bash
# Tilrettelegg Foundry-kontoen + modellutrullinger som kode
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd skriver eksempler/basic-chat-azure/.env med sluttpunktet ditt (ingen nøkkel)
```

**Manuell endepunktkonfigurasjon:**
```bash
# Hvis du ikke brukte azd, sett endepunktet selv (autentisering forblir nøkkelfri via az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Rediger .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Utviklingsarbeidsflyt

### Prosjektstruktur
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

### Kjøre applikasjoner

**Kjøre en Spring Boot-applikasjon:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Bygge et prosjekt:**
```bash
cd [project-directory]
mvn clean install
```

**Starte MCP Calculator Server:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# Server kjører på http://localhost:8080
```

**Kjøre klienteksempler:**
```bash
# Etter å ha startet serveren i en annen terminal
cd 04-PracticalSamples/calculator

# Direkte MCP-klient
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# AI-drevet klient (krever AZURE_OPENAI_ENDPOINT + az login)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Interaktiv bot
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Hot Reload
Spring Boot DevTools er inkludert i prosjekter som støtter hot reload:
```bash
# Endringer i Java-filer lastes automatisk inn på nytt når de lagres
mvn spring-boot:run
```

## Testinstruksjoner

### Kjøre tester

**Kjør alle tester i et prosjekt:**
```bash
cd [project-directory]
mvn test
```

**Kjør tester med detaljert output:**
```bash
mvn test -X
```

**Kjør spesifikk testklasse:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Teststruktur
- Testfiler bruker JUnit 5 (Jupiter)
- Testklasser ligger i `src/test/java/`
- Klienteksempler i kalkulatorprosjektet ligger i `src/test/java/com/microsoft/mcp/sample/client/`

### Manuell testing
Mange eksempler er interaktive applikasjoner som krever manuell testing:

1. Start applikasjonen med `mvn spring-boot:run`
2. Test endepunkter eller interager med CLI
3. Verifiser at forventet atferd stemmer med dokumentasjonen i hvert prosjekts README.md

### Testing med Azure AI Foundry
- Logg inn med `az login` før du kjører eksempler (nøkkelfri autentisering)
- Sørg for at kontoen din har Cognitive Services OpenAI User-rollen på ressursen
- Test innholdsfiltrering med eksemplet for ansvarlig AI i Kapittel 5

## Kodeformatretningslinjer

### Java-konvensjoner
- **Java-versjon:** Java 21 med moderne funksjoner
- **Stil:** Følg standard Java-konvensjoner
- **Navngivning:** 
  - Klasser: PascalCase
  - Metoder/variabler: camelCase
  - Konstanter: UPPER_SNAKE_CASE
  - Pakkebenavn: små bokstaver

### Spring Boot-mønstre
- Bruk `@Service` for forretningslogikk
- Bruk `@RestController` for REST-endepunkter
- Konfigurasjon via `application.yml` eller `application.properties`
- Miljøvariabler foretrekkes fremfor hardkodede verdier
- Bruk `@Tool`-annotasjon for MCP-eksponerte metoder

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

### Avhengigheter
- Administrert via Maven `pom.xml`
- Spring AI BOM for versjonsstyring
- LangChain4j for AI-integrasjoner
- Spring Boot starter parent for Spring-avhengigheter

### Kodekommentarer
- Legg til JavaDoc for offentlige API-er
- Inkluder forklarende kommentarer for komplekse AI-interaksjoner
- Dokumenter MCP-verktøybeskrivelser tydelig

## Bygging og distribusjon

### Byggeprosjekter

**Bygg uten tester:**
```bash
mvn clean install -DskipTests
```

**Bygg med alle kontroller:**
```bash
mvn clean install
```

**Pakk applikasjonen:**
```bash
mvn package
# Oppretter JAR i target/-katalogen
```

### Utdata-mapper
- Kompilerte klasser: `target/classes/`
- Testklasser: `target/test-classes/`
- JAR-filer: `target/*.jar`
- Maven-artifakter: `target/`

### Miljøspesifikk konfigurasjon

**Utvikling:**
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

**Produksjon:**
- Bruk administrert identitet i stedet for `az login` for nøkkelfri autentisering
- Sett `AZURE_OPENAI_ENDPOINT` til produksjons Foundry-ressursen din
- Konfigurer via miljøvariabler eller Azure Key Vault

### Distribusjonshensyn
- Dette er et utdanningsrepo med eksempelsapplikasjoner
- Ikke ment for produksjonsdistribusjon som det er
- Eksempler demonstrerer mønstre for å tilpasses produksjonsbruk
- Se individuelle prosjekters README-filer for spesifikke distribusjonsnotater

## Tilleggsnotater

### Azure AI Foundry
- **Nøkkelfri autentisering:** koble til med Microsoft Entra ID — ingen API-nøkler å håndtere
- **Provisjonert som kode:** Bicep + azd (`azd up`) oppretter konto og modelldistribusjoner
- Den samme OpenAI-kompatible koden kjører lokalt (`az login`) og i Azure (administrert identitet)

### Arbeide med flere prosjekter
Hvert eksempelsprosjekt er frittstående:
```bash
# Naviger til spesifikt prosjekt
cd 04-PracticalSamples/[project-name]

# Hver har sin egen pom.xml og kan bygges uavhengig
mvn clean install
```

### Vanlige problemer

**Java-versjonskonflikt:**
```bash
# Verifiser Java 21
java -version
# Oppdater JAVA_HOME om nødvendig
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Problemer med nedlasting av avhengigheter:**
```bash
# Tøm Maven-cache og prøv igjen
rm -rf ~/.m2/repository
mvn clean install
```

**Endepunkt eller pålogging ikke funnet:**
```bash
# Sett endepunktet i gjeldende økt og logg inn (uten nøkkel)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Eller bruk en .env-fil i prosjektmappen
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Port allerede i bruk:**
```bash
# Spring Boot bruker port 8080 som standard
# Endring i application.properties:
server.port=8081
```

### Flerspråklig støtte
- Dokumentasjon tilgjengelig på 45+ språk via automatisert oversettelse
- Oversettelser i `translations/`-katalogen
- Oversettelse håndteres av GitHub Actions workflow

### Læringsløype
1. Start med [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. Følg kapitlene i rekkefølge (01 → 05)
3. Fullfør praktiske eksempler i hvert kapittel
4. Utforsk eksempelsprosjektene i Kapittel 4
5. Lær ansvarlig AI-praksis i Kapittel 5

### Utviklingscontainer
`.devcontainer/devcontainer.json` konfigurerer:
- Java 21 utviklingsmiljø
- Maven forhåndsinstallert
- VS Code Java-utvidelser
- Spring Boot-verktøy
- GitHub Copilot-integrasjon
- Docker-i-Docker støtte
- Azure CLI

### Ytelseshensyn
- Azure AI Foundry-distribusjoner har kvoter per minutt for tokens/forespørsler
- Bruk passende batch-størrelser for embeddings
- Vurder caching for gjentatte API-kall
- Overvåk tokenbruk for kostnadsoptimalisering

### Sikkerhetsnotater
- Aldri committ `.env`-filer (allerede i `.gitignore`)
- Foretrekk nøkkelfri autentisering (Microsoft Entra ID) fremfor API-nøkler
- Bruk administrerte identiteter i Azure; `az login` for lokal utvikling
- Følg retningslinjene for ansvarlig AI i Kapittel 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Ansvarsfraskrivelse**:
Dette dokumentet er oversatt ved hjelp av AI-oversettelsestjenesten [Co-op Translator](https://github.com/Azure/co-op-translator). Selv om vi streber etter nøyaktighet, vær oppmerksom på at automatiske oversettelser kan inneholde feil eller unøyaktigheter. Det opprinnelige dokumentet på originalspråket skal betraktes som den autoritative kilden. For kritisk informasjon anbefales profesjonell menneskelig oversettelse. Vi er ikke ansvarlige for eventuelle misforståelser eller feiltolkninger som oppstår ved bruk av denne oversettelsen.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->