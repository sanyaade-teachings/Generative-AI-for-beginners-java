# AGENTS.md

## Projekti ülevaade

See on hariduslik hoidla generatiivse tehisintellekti arenduse õppimiseks Java abil. See pakub põhjalikku praktilist kursust, mis käsitleb suuri keelemudeleid (LLMs), promptide inseneriteadust, sisu embedimisi, RAG-i (Retrieval-Augmented Generation) ja mudelite kontekstiprotokolli (MCP).

**Põhitehnoloogiad:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI ja OpenAI SDK-d

**Arhitektuur:**
- Mitmed iseseisvad Spring Boot rakendused, mis on organiseeritud peatükkide järgi
- Näidistööprojektid, mis demonstreerivad erinevaid tehisintellekti mustreid
- GitHub Codespaces valmis koos eelkonfigureeritud arenduskonteineritega

## Paigalduskäsud

### Eeltingimused
- Java 21 või uuem
- Maven 3.x
- Azure tellimus koos Azure AI Foundry mudeli juurutusega (provisionida `azd up` abil)
- Azure CLI (`az`) ja Azure Developer CLI (`azd`), sisse logitud võtmeta autentimiseks

### Keskkonna seadistamine

**Valik 1: GitHub Codespaces (soovitatav)**
```bash
# Forkige hoidla ja looge GitHubi kasutajaliidesest koodiruum
# Arendus konteiner paigaldab automaatselt kõik sõltuvused
# Oodake keskkonna seadistamiseks umbes 2 minutit
```

**Valik 2: Kohalik arenduskonteiner**
```bash
# Klooni hoidla
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Ava VS Code'is Dev Containers laiendusega
# Ava uuesti konteineris, kui küsitakse
```

**Valik 3: Kohalik paigaldus**
```bash
# Paigalda sõltuvused
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Kontrolli paigaldust
java -version
mvn -version
```

### Konfiguratsioon

**Azure AI Foundry seadistus (võtmeta, soovitatav):**
```bash
# Käivita Foundry konto + mudelite juurutused koodina
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd kirjutab näite faili examples/basic-chat-azure/.env sinu otspunktiga (ilma võtmeta)
```

**Manuaalne lõpp-punkti konfiguratsioon:**
```bash
# Kui te ei kasutanud azd, seadistage lõpp-punkt ise (autentimine jääb az login abil võtmepõhiseks)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Muutke .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Arendusprotsess

### Projekti struktuur
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

### Rakenduste käivitamine

**Spring Boot rakenduse käivitamine:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Projekti koostamine:**
```bash
cd [project-directory]
mvn clean install
```

**MCP kalkulaatori serveri käivitamine:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# Server töötab aadressil http://localhost:8080
```

**Kliendi näidete käitamine:**
```bash
# Pärast serveri käivitamist teises terminalis
cd 04-PracticalSamples/calculator

# Otse MCP klient
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# Tehisintellekti abil juhitav klient (nõuab AZURE_OPENAI_ENDPOINT + az login)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Interaktiivne robot
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Kuumlaadimine
Spring Boot DevTools on lisatud neisse projektidesse, mis toetavad kuumlaadimist:
```bash
# Java failide muudatused laaditakse salvestamisel automaatselt uuesti
mvn spring-boot:run
```

## Testimise juhised

### Testide käivitamine

**Käivita kõik testid projektis:**
```bash
cd [project-directory]
mvn test
```

**Käivita testid põhjaliku väljundiga:**
```bash
mvn test -X
```

**Käivita konkreetne testiklass:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Testide struktuur
- Testifailid kasutavad JUnit 5 (Jupiter)
- Testiklassid asuvad `src/test/java/`
- Kalkulaatori kliendi näited on `src/test/java/com/microsoft/mcp/sample/client/`

### Käsitsi testimine
Paljud näited on interaktiivsed rakendused, mis nõuavad käsitsi testimist:

1. Käivita rakendus käsuga `mvn spring-boot:run`
2. Testi lõpp-punkte või suhtle käsurealiidesega
3. Kontrolli, et oodatav käitumine vastab iga projekti README.md dokumentatsioonile

### Testimine Azure AI Foundryga
- Logi sisse käsuga `az login` enne näidete käivitamist (võtmeta autentimine)
- Veendu, et sinu kontol on Cognitive Services OpenAI User roll ressursil
- Testi sisufiltrit vastutustundliku AI näitega peatükis 5

## Koodi stiilijuhised

### Java konventsioonid
- **Java versioon:** Java 21 kaasaegsete funktsioonidega
- **Stiil:** Järgi tavalisi Java stiilisoovitusi
- **Nimekujundus:** 
  - Klassid: PascalCase
  - Meetodid/muutujad: camelCase
  - Konstandid: UPPER_SNAKE_CASE
  - Paketid: väiketähed

### Spring Boot mustrid
- Kasuta `@Service` äriloogika jaoks
- Kasuta `@RestController` REST lõpp-punktide jaoks
- Konfiguratsioon failides `application.yml` või `application.properties`
- Eelistada keskkonnamuutujaid kõvade väärtuste asemel
- Kasuta `@Tool` annotatsiooni MCP kaudu eksponeeritud meetodite puhul

### Failide korraldus
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

### Sõltuvused
- Haldatud Maven `pom.xml` abil
- Spring AI BOM versioonihalduseks
- LangChain4j tehisintellekti integratsioonideks
- Spring Boot starter parent Spring sõltuvuste jaoks

### Koodi kommentaarid
- Lisa JavaDoc avalikele API-dele
- Kirjelda keerukaid AI interaktsioone selgitavate kommentaaridega
- Dokumenteeri MCP tööriistade kirjelduseid selgelt

## Koostamine ja juurutamine

### Projektide koostamine

**Koosta ilma testideta:**
```bash
mvn clean install -DskipTests
```

**Koosta koos kõigi kontrollidega:**
```bash
mvn clean install
```

**Paki rakendus:**
```bash
mvn package
# Loob JAR-faili kaustas target/
```

### Väljundkaustad
- Koostatud klassid: `target/classes/`
- Testiklassid: `target/test-classes/`
- JAR-failid: `target/*.jar`
- Maven artefaktid: `target/`

### Keskkonnasõltuv konfiguratsioon

**Arendus:**
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

**Tootmine:**
- Kasuta haldatud identiteeti `az login` asemel võtmeta autentimiseks
- Suuna `AZURE_OPENAI_ENDPOINT` tootmisressurssi Foundry teenuses
- Halda konfiguratsiooni keskkonnamuutujate või Azure Key Vaulti kaudu

### Juurutamise kaalutlused
- See on hariduslik hoidla koos näidisrakendustega
- Ei ole mõeldud tootmiskasutuseks otse
- Näited demonstreerivad mustreid, mida saab tootmises kasutada
- Vaata konkreetseid juurutusjuhiseid iga projekti README-st

## Täiendavad märkused

### Azure AI Foundry
- **Võtmeta autentimine:** ühendus Microsoft Entra ID-ga — pole vaja API võtmeid hallata
- **Provisioneeritud koodina:** Bicep + azd (`azd up`) loovad konto ja mudelid
- Sama OpenAI-sõbralik kood töötab lokaalselt (`az login`) ja Azure'is (haldatud identiteet)

### Mitme projekti kasutamine
Iga näidisprojekt on iseseisev:
```bash
# Liigu konkreetse projekti juurde
cd 04-PracticalSamples/[project-name]

# Igalühel on oma pom.xml ja neid saab eraldi ehitada
mvn clean install
```

### Levinud probleemid

**Java versiooni sobimatus:**
```bash
# Kontrolli Java 21 versiooni
java -version
# Uuenda vajadusel JAVA_HOME пути
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Sõltuvuste allalaadimise probleemid:**
```bash
# Tühjenda Maven'i vahemälu ja proovi uuesti
rm -rf ~/.m2/repository
mvn clean install
```

**Lõpp-punkti või sisselogimise leidmata:**
```bash
# Määra lõpp-punkt praeguses seansis ja logi sisse (ilma võtmeta)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Või kasuta projekti kaustas olevat .env faili
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Port juba kasutusel:**
```bash
# Spring Boot kasutab vaikimisi porti 8080
# Muuda application.properties failis:
server.port=8081
```

### Mitmekeelsuse tugi
- Dokumentatsioon saadaval üle 45 keeles automaatse tõlke kaudu
- Tõlked asuvad `translations/` kaustas
- Tõlget haldab GitHub Actions töövoog

### Õppimise rada
1. Alusta [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md) juhendist
2. Järgi peatükke järjest (01 → 05)
3. Läbi praktilised näited igas peatükis
4. Uuri näidisprojekte peatükis 4
5. Õpi vastutustundliku AI kasutamist peatükis 5

### Arenduskonteiner
Fail `.devcontainer/devcontainer.json` seadistab:
- Java 21 arenduskeskkonna
- Eelinstalleeritud Maven
- VS Code Java laiendused
- Spring Boot tööriistad
- GitHub Copilot integratsiooni
- Docker-in-Docker toe
- Azure CLI

### Tulemuslikkuse kaalutlused
- Azure AI Foundry juurutustel on minutipõhised päringute ja tokeni limiidid
- Kasuta sobivaid partiisuurusi embedimiste jaoks
- Mõtle vahemälude kasutamisele korduvate API päringute puhul
- Jälgi tokenite kasutust kulude optimeerimiseks

### Turbenõuanded
- Ära kunagi lisa `.env` faile versioonihaldusesse (on juba `.gitignore`s)
- Eelista võtmeta autentimist (Microsoft Entra ID) API võtmete asemel
- Kasuta Azure hallatavaid identiteete; `az login` kohalikuks arenduseks
- Järgi vastutustundliku AI juhiseid peatükis 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Lahtiütlus**:
See dokument on tõlgitud kasutades AI tõlketeenust [Co-op Translator](https://github.com/Azure/co-op-translator). Kuigi me püüdleme täpsuse poole, palun pange tähele, et automatiseeritud tõlgetes võib esineda vigu või ebatäpsusi. Originaaldokument selle emakeeles tuleks pidada autoriteetseks allikaks. Olulise teabe puhul soovitatakse kasutada professionaalset inimtõlget. Me ei vastuta selle tõlkega seotud eksimustest või valesti mõistmistest.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->