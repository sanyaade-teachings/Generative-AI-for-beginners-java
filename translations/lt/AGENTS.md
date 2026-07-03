# AGENTS.md

## Projekto apžvalga

Tai edukacinis repozitorijus, skirtas mokytis Generatyviosios AI kūrimo su Java. Jame pateikiamas išsamus praktinis kursas, apimantis Didelius kalbos modelius (LLM), užklausų inžineriją, įterpimus, RAG (Retrieval-Augmented Generation) ir Modelio konteksto protokolą (MCP).

**Pagrindinės technologijos:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI ir OpenAI SDK

**Architektūra:**
- Keli atskiri Spring Boot aplikacijų projektai, sugrupuoti pagal skyrius
- Pavyzdiniai projektai, demonstruojantys įvairius AI modelius
- Paruošta naudoti GitHub Codespaces su iš anksto sukonfigūruotomis kūrimo aplinkomis

## Diegimo komandos

### Išankstiniai reikalavimai
- Java 21 arba naujesnė
- Maven 3.x
- Azure prenumerata su Azure AI Foundry modelio diegimu (sukuriama su `azd up`)
- Azure CLI (`az`) ir Azure Developer CLI (`azd`), prisijungę naudotojai be raktų (keyless auth)

### Aplinkos konfigūracija

**Pasirinkimas 1: GitHub Codespaces (rekomenduojama)**
```bash
# Sukurkite šaką (fork) saugykloje ir sukurkite kodų erdvę (codespace) iš GitHub sąsajos
# Kūrimo konteineris automatiškai įdiegs visas priklausomybes
# Palaukite apie 2 minutes, kol bus paruošta aplinka
```

**Pasirinkimas 2: Vietinis kūrimo konteineris**
```bash
# Klonuoti repozitoriją
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Atidaryti VS Code su Dev Containers plėtiniu
# Pakartotinai atidaryti konteineryje, kai bus paprašyta
```

**Pasirinkimas 3: Vietinis diegimas**
```bash
# Įdiekite priklausomybes
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Patikrinkite diegimą
java -version
mvn -version
```

### Konfigūracija

**Azure AI Foundry diegimas (be raktų, rekomenduojama):**
```bash
# Sukurkite Foundry paskyrą + modelių diegimus kaip kodą
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd įrašo examples/basic-chat-azure/.env su jūsų galiniu tašku (be rakto)
```

**Rankinė galinio taško konfigūracija:**
```bash
# Jei nenaudojote azd, patys nustatykite galinį tašką (autentifikacija lieka be rakto per az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Redaguokite .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Kūrimo darbo eiga

### Projekto struktūra
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

### Aplikacijų paleidimas

**Spring Boot aplikacijos paleidimas:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Projekto kompiliavimas:**
```bash
cd [project-directory]
mvn clean install
```

**MCP skaičiuoklio serverio paleidimas:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# Serveris veikia adresu http://localhost:8080
```

**Klientų pavyzdžių paleidimas:**
```bash
# Po serverio paleidimo kitame terminale
cd 04-PracticalSamples/calculator

# Tiesioginis MCP klientas
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# AI varomas klientas (reikia AZURE_OPENAI_ENDPOINT + az prisijungimo)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Interaktyvus botas
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Karštasis persikrovimas
Spring Boot DevTools įtrauktos į projektus, kurie palaiko karštąjį persikrovimą:
```bash
# Pakeitimai Java failuose bus automatiškai perkraunami juos išsaugojus
mvn spring-boot:run
```

## Testavimo instrukcijos

### Testų vykdymas

**Visų projekto testų paleidimas:**
```bash
cd [project-directory]
mvn test
```

**Testų paleidimas su išsamia išvestimi:**
```bash
mvn test -X
```

**Konkrečios testų klasės paleidimas:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Testų struktūra
- Testų failai naudoja JUnit 5 (Jupiter)
- Testų klasės yra `src/test/java/` kataloge
- Klientų pavyzdžiai skaičiuoklio projekte yra `src/test/java/com/microsoft/mcp/sample/client/`

### Rankinis testavimas
Daugelis pavyzdžių yra interaktyvios aplikacijos, kurios reikalauja rankinio testavimo:

1. Paleiskite aplikaciją komanda `mvn spring-boot:run`
2. Testuokite galinius taškus arba bendraukite per komandų eilutę
3. Patikrinkite, ar elgsena atitinka kiekvieno projekto README.md dokumentacijoje aprašytus lūkesčius

### Testavimas su Azure AI Foundry
- Prisijunkite su `az login` prieš paleisdami pavyzdžius (be raktų autentifikavimas)
- Užtikrinkite, kad jūsų paskyra turi Cognitive Services OpenAI User vaidmenį resurse
- Testuokite turinio filtravimą su atsakingos AI pavyzdžiu 5 skyriuje

## Kodo stiliaus gairės

### Java konvencijos
- **Java versija:** Java 21 su moderniomis funkcijomis
- **Stilius:** Laikytis standartinių Java konvencijų
- **Pavadinimai:** 
  - Klasės: PascalCase
  - Metodai/kintamieji: camelCase
  - Konstantos: UPPER_SNAKE_CASE
  - Paketo pavadinimai: mažosiomis raidėmis

### Spring Boot modeliai
- Naudoti `@Service` verslo logikai
- Naudoti `@RestController` REST galiniams taškams
- Konfigūracija per `application.yml` arba `application.properties`
- Aplinkos kintamieji preferuojami prieš kietai užkoduotas reikšmes
- Naudoti `@Tool` anotaciją MCP eksponuotoms metodams

### Failų organizavimas
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

### Priklausomybės
- Valdomos per Maven `pom.xml`
- Naudojamas Spring AI BOM versijų valdymui
- LangChain4j AI integracijoms
- Spring Boot starter parent Spring priklausomybėms

### Kodo komentarai
- Rašyti JavaDoc viešajam API
- Įtraukti paaiškinamuosius komentarus sudėtingiems AI sąveikoms
- Aiškiai dokumentuoti MCP įrankių aprašymus

## Kūrimas ir diegimas

### Projektų kūrimas

**Kūrimas be testų:**
```bash
mvn clean install -DskipTests
```

**Kūrimas su visais patikrinimais:**
```bash
mvn clean install
```

**Aplikacijos paketavimas:**
```bash
mvn package
# Sukuria JAR failą target/ kataloge
```

### Išvesties katalogai
- Surinktos klasės: `target/classes/`
- Testų klasės: `target/test-classes/`
- JAR failai: `target/*.jar`
- Maven artefaktai: `target/`

### Aplinkos specifinė konfiguracija

**Vystymui:**
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

**Gamybai:**
- Naudoti valdomą tapatybę vietoje `az login` keyless autentifikavimui
- Nustatyti `AZURE_OPENAI_ENDPOINT` į savo gamybinį Foundry resursą
- Valdyti konfigūraciją per aplinkos kintamuosius arba Azure Key Vault

### Diegimo pastabos
- Tai edukacinis repozitorijus su pavyzdinėmis aplikacijomis
- Nėra skirtas gamybiniam diegimui be papildomų adaptacijų
- Pavyzdžiai demonstruoja modelius, kuriuos galima pritaikyti gamybinėms reikmėms
- Peržiūrėkite individualių projektų README.md dėl konkrečių diegimo pastabų

## Papildomos pastabos

### Azure AI Foundry
- **Keyless autentifikacija:** jungtis per Microsoft Entra ID — nereikia valdyti API raktų
- **Pankodinis diegimas:** Bicep + azd (`azd up`) sukuria paskyrą ir modelių diegimus
- Toks pats OpenAI suderinamas kodas veikia vietoje (`az login`) ir Azure aplinkoje (valdomos tapatybės)

### Darbas su keliais projektais
Kiekvienas pavyzdinis projektas yra atskiras:
```bash
# Pereikite į konkretų projektą
cd 04-PracticalSamples/[project-name]

# Kiekvienas turi savo pom.xml ir gali būti statomas nepriklausomai
mvn clean install
```

### Dažnos problemos

**Java versijų neatitikimas:**
```bash
# Patikrinti Java 21
java -version
# Atnaujinti JAVA_HOME, jei reikia
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Priklausomybių atsisiuntimo problemos:**
```bash
# Išvalykite Maven talpyklą ir bandykite dar kartą
rm -rf ~/.m2/repository
mvn clean install
```

**Galinio taško arba prisijungimo nepavykimas:**
```bash
# Nustatykite galutinį tašką dabartinėje sesijoje ir prisijunkite (be rakto)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Arba naudokite .env failą projekto kataloge
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Prievadas jau užimtas:**
```bash
# Spring Boot pagal numatytuosius nustatymus naudoja 8080 prievadą
# Keisti application.properties faile:
server.port=8081
```

### Daugiakalbystės palaikymas
- Dokumentacija prieinama daugiau nei 45 kalbomis per automatinį vertimą
- Vertimai saugomi `translations/` kataloge
- Vertimo valdymas atliekamas per GitHub Actions darbo eigą

### Mokymosi kelias
1. Pradėkite nuo [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. Sekite skyrius pagal eilę (01 → 05)
3. Atlikite praktinius pavyzdžius kiekviename skyriuje
4. Tyrinėkite pavyzdinius projektus 4 skyriuje
5. Mokykitės atsakingos AI praktikos 5 skyriuje

### Kūrimo konteineris
Failas `.devcontainer/devcontainer.json` konfigūruoja:
- Java 21 kūrimo aplinką
- Iš anksto įdiegtą Maven
- VS Code Java plėtinius
- Spring Boot įrankius
- GitHub Copilot integraciją
- Docker-in-Docker palaikymą
- Azure CLI

### Veikimo našumo aspektai
- Azure AI Foundry diegimuose yra minučių tokenų/užklausų kvotos
- Naudokite tinkamus paketus įterpimams
- Apsvarstykite kešavimą pakartotinėms API užklausoms
- Stebėkite tokenų naudojimą kaštų optimizavimui

### Saugumo pastabos
- Niekada nekelkite `.env` failų (jie jau įtraukti į `.gitignore`)
- Prioritetinis keyless autentifikavimas (Microsoft Entra ID) prieš API raktus
- Naudokite valdomas tapatybes Azure; `az login` vietiniam vystymui
- Vadovaukitės atsakingos AI gairėmis 5 skyriuje

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Atsakomybės apribojimas**:
Šis dokumentas buvo išverstas naudojant dirbtinio intelekto vertimo paslaugą [Co-op Translator](https://github.com/Azure/co-op-translator). Nors siekiame tikslumo, prašome atkreipti dėmesį, kad automatiniai vertimai gali turėti klaidų ar netikslumų. Originalus dokumentas jo gimtąja kalba laikomas autoritetingu šaltiniu. Svarbiai informacijai rekomenduojama naudoti profesionalų žmogiškąjį vertimą. Mes neatsakome už jokius nesusipratimus ar neteisingą interpretaciją, kilusią naudojantis šiuo vertimu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->