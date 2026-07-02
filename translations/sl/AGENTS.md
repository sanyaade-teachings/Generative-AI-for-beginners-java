# AGENTS.md

## Pregled projekta

To je izobraževalno repozitorij za učenje razvoja Generativne umetne inteligence z Javo. Ponuja obsežen praktični tečaj, ki pokriva velike jezikovne modele (LLM), inženiring pozivov, vdelave, RAG (Generiranje z nadgradnjo z iskanjem) in protokol Model Context (MCP).

**Ključne tehnologije:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI in OpenAI SDK-ji

**Arhitektura:**
- Več samostojnih aplikacij Spring Boot organiziranih po poglavjih
- Primeri projektov, ki prikazujejo različne vzorce AI
- Pripravljeno za GitHub Codespaces z vnaprej konfiguriranimi razvojnimi okolji

## Ukazi za nastavitev

### Predpogoji
- Java 21 ali novejša
- Maven 3.x
- Azure naročnina z nameščenim modelom v Azure AI Foundry (zagotovi z `azd up`)
- Azure CLI (`az`) in Azure Developer CLI (`azd`), prijavljeni za prijavo brez ključev

### Nastavitev okolja

**Možnost 1: GitHub Codespaces (priporočeno)**
```bash
# Razvezi repozitorij in ustvari codespace iz GitHub vmesnika
# Razvojno okolje bo samodejno namestilo vse odvisnosti
# Počakaj približno 2 minuti za nastavitev okolja
```

**Možnost 2: Lokalno razvojno okolje v kontejnerju**
```bash
# Kloniraj repozitorij
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Odpri v VS Code z razširitvijo Dev Containers
# Ponovno odpri v kontejnerju, ko te pozove
```

**Možnost 3: Lokalna nastavitev**
```bash
# Namestite odvisnosti
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Preverite namestitev
java -version
mvn -version
```

### Konfiguracija

**Nastavitev Azure AI Foundry (brez ključev, priporočeno):**
```bash
# Zagotovite račun Foundry + uvajanja modelov kot kodo
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd zapiše examples/basic-chat-azure/.env z vašo končno točko (brez ključa)
```

**Ročna konfiguracija končne točke:**
```bash
# Če niste uporabili azd, nastavite končno točko sami (avtorizacija ostane brez ključa preko az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Uredite .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Razvojni potek

### Struktura projekta
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

### Zagon aplikacij

**Zagon aplikacije Spring Boot:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Izgradnja projekta:**
```bash
cd [project-directory]
mvn clean install
```

**Zagon MCP kalkulator strežnika:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# Strežnik teče na http://localhost:8080
```

**Zagon primerov odjemalcev:**
```bash
# Po zagonu strežnika v drugem terminalu
cd 04-PracticalSamples/calculator

# Neposredni MCP odjemalec
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# AI-podprt odjemalec (zahteva AZURE_OPENAI_ENDPOINT + az prijava)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Interaktivni bot
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Hot Reload
Spring Boot DevTools je vključen v projektih, ki podpirajo hot reload:
```bash
# Spremembe v Java datotekah se bodo samodejno ponovno naložile ob shranjevanju
mvn spring-boot:run
```

## Navodila za testiranje

### Zagon testov

**Zaženi vse teste v projektu:**
```bash
cd [project-directory]
mvn test
```

**Zaženi teste z obsežnim izpisom:**
```bash
mvn test -X
```

**Zaženi določen test razred:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Struktura testov
- Testne datoteke uporabljajo JUnit 5 (Jupiter)
- Testni razredi so v `src/test/java/`
- Primeri odjemalcev v kalkulator projektu so v `src/test/java/com/microsoft/mcp/sample/client/`

### Ročno testiranje
Veliko primerov so interaktivne aplikacije, ki zahtevajo ročno testiranje:

1. Zaženi aplikacijo z `mvn spring-boot:run`
2. Testiraj končne točke ali interakcijo v CLI
3. Preveri, da se pričakovano vedenje ujema z dokumentacijo v README.md vsakega projekta

### Testiranje z Azure AI Foundry
- Prijavi se z `az login` pred zagonom primerov (avtorizacija brez ključev)
- Prepričaj se, da ima tvoj račun vlogo Cognitive Services OpenAI User na viru
- Testiraj filtriranje vsebin z odgovornim AI primerom v poglavju 5

## Smernice za slog kode

### Java konvencije
- **Java verzija:** Java 21 z modernimi funkcijami
- **Slog:** Sledi standardnim Java konvencijam
- **Poimenovanje:** 
  - Razredi: PascalCase
  - Metode/spremenljivke: camelCase
  - Konstantne vrednosti: VELIKE_SNAKE_CASE
  - Imena paketov: male črke

### Vzorce Spring Boot
- Uporabi `@Service` za poslovno logiko
- Uporabi `@RestController` za REST končne točke
- Konfiguracija preko `application.yml` ali `application.properties`
- Prometna spremenljivka okolja predvidena pred trdo kodiranimi vrednostmi
- Uporabi oznako `@Tool` za metode izpostavljene v MCP

### Organizacija datotek
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

### Odvisnosti
- Upravljanje z Maven `pom.xml`
- Spring AI BOM za upravljanje verzij
- LangChain4j za AI integracije
- Spring Boot starter parent za Spring odvisnosti

### Komentarji v kodi
- Dodaj JavaDoc za javne API-je
- Vključi pojasnilne komentarje za kompleksne AI interakcije
- Jasno dokumentiraj opise orodij MCP

## Izgradnja in uvajanje

### Izgradnja projektov

**Izgradi brez testov:**
```bash
mvn clean install -DskipTests
```

**Izgradi z vsemi preverjanji:**
```bash
mvn clean install
```

**Pakiranje aplikacije:**
```bash
mvn package
# Ustvari JAR v imeniku target/
```

### Izhodne mape
- Prevedeni razredi: `target/classes/`
- Testni razredi: `target/test-classes/`
- JAR datoteke: `target/*.jar`
- Maven artefakti: `target/`

### Konfiguracija specifična za okolje

**Razvoj:**
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

**Produkcija:**
- Uporabi upravljano identiteto namesto `az login` za prijavo brez ključev
- Nastavi `AZURE_OPENAI_ENDPOINT` na tvoj produkcijski Foundry vir
- Upravljaj konfiguracijo preko okoljskih spremenljivk ali Azure Key Vault

### Premisleki o uvajanju
- To je izobraževalni repozitorij s primeri aplikacij
- Ni načrtovan za direktno uporabo v produkciji
- Primeri prikazujejo vzorce, prilagojene za produkcijsko uporabo
- Oglej si README-je posameznih projektov za specifične opombe o uvajanju

## Dodatne opombe

### Azure AI Foundry
- **Prijava brez ključev:** poveži se z Microsoft Entra ID - ni potrebnih API ključev
- **Nastavljeno kot koda:** Bicep + azd (`azd up`) ustvarita račun in namestitev modelov
- Isto združljivo OpenAI kodo lahko uporabljaš lokalno (`az login`) in v Azure (upravljana identiteta)

### Delo z več projekti
Vsak vzorčni projekt je samostojen:
```bash
# Pomaknite se do specifičnega projekta
cd 04-PracticalSamples/[project-name]

# Vsak ima svojo datoteko pom.xml in se lahko gradi neodvisno
mvn clean install
```

### Pogoste težave

**Neujemanje verzije Jave:**
```bash
# Preverite Java 21
java -version
# Posodobite JAVA_HOME, če je potrebno
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Težave z nalaganjem odvisnosti:**
```bash
# Počisti Maven predpomnilnik in poskusi znova
rm -rf ~/.m2/repository
mvn clean install
```

**Končna točka ali prijava ni najdena:**
```bash
# Nastavite končno točko v trenutni seji in se vpišite (brez ključa)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Ali uporabite datoteko .env v imeniku projekta
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Vrat že v uporabi:**
```bash
# Spring Boot privzeto uporablja vrata 8080
# Sprememba v application.properties:
server.port=8081
```

### Podpora več jezikom
- Dokumentacija na voljo v več kot 45 jezikih preko avtomatskega prevajanja
- Prevodi v mapi `translations/`
- Prevajanje upravlja GitHub Actions potek dela

### Pot učenja
1. Začni z [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. Sledi poglavjem v vrstnem redu (01 → 05)
3. Dokončaj praktične primere v vsakem poglavju
4. Raziskuj vzorčne projekte v poglavju 4
5. Nauči se praks odgovorne AI v poglavju 5

### Razvojno okolje v kontejnerju
`.devcontainer/devcontainer.json` konfigurira:
- Razvojno okolje Java 21
- Vnaprej nameščen Maven
- Razširitve VS Code za Javo
- Orodja Spring Boot
- Integracijo GitHub Copilot
- Podporo Docker-in-Docker
- Azure CLI

### Premisleki glede zmogljivosti
- Azure AI Foundry nameščanja imajo kvote tokenov/poizvedb na minuto
- Uporabljaj primerne velikosti paketov za vdelave
- Razmisli o predpomnjenju za ponavljajoče API klice
- Spremljaj uporabo tokenov za optimizacijo stroškov

### Varnostne opombe
- Nikoli ne komitiraj `.env` datotek (že v `.gitignore`)
- Prednostno uporabljaj avtentikacijo brez ključev (Microsoft Entra ID) namesto API ključev
- Uporabljaj upravljane identitete v Azure; lokalno za razvoj je `az login`
- Sledi smernicam odgovorne AI v poglavju 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Omejitev odgovornosti**:
Ta dokument je bil preveden z uporabo AI prevajalske storitve [Co-op Translator](https://github.com/Azure/co-op-translator). Čeprav si prizadevamo za natančnost, vas prosimo, da upoštevate, da avtomatizirani prevodi lahko vsebujejo napake ali netočnosti. Izvirni dokument v njegovem izvirnem jeziku je treba obravnavati kot avtoritativni vir. Za kritične informacije je priporočljiv strokovni človeški prevod. Ne odgovarjamo za morebitna nesporazume ali napačne interpretacije, ki izhajajo iz uporabe tega prevoda.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->