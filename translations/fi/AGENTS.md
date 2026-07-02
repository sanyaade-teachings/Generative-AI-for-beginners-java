# AGENTS.md

## Projektin yleiskatsaus

Tämä on opetusrekisteri Generatiivisen tekoälyn kehittämisen oppimiseen Java-kielellä. Se tarjoaa kattavan käytännön kurssin, joka kattaa suuret kielimallit (LLM), kehotteiden suunnittelun, upotukset, RAG:n (Retrieval-Augmented Generation) ja Malli-kontekstiprotokollan (MCP).

**Keskeiset teknologiat:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI ja OpenAI SDK:t

**Arkkitehtuuri:**
- Useita itsenäisiä Spring Boot -sovelluksia luvuittain järjestettynä
- Esimerkkiprojekteja, jotka demonstroivat erilaisia tekoälykuvioita
- GitHub Codespaces -valmiit esikonfiguroiduilla kehityssäiliöillä

## Asennuskäskyt

### Esivaatimukset
- Java 21 tai uudempi
- Maven 3.x
- Azure-tilaus, jossa on Azure AI Foundry -malliasennus (provisioi `azd up` komennolla)
- Azure CLI (`az`) ja Azure Developer CLI (`azd`), kirjauduttu sisään avaimettomaan tunnistautumiseen

### Ympäristön asennus

**Vaihtoehto 1: GitHub Codespaces (suositus)**
```bash
# Haarauta repositorio ja luo codespace GitHub-käyttöliittymästä
# Kehityssäiliö asentaa automaattisesti kaikki riippuvuudet
# Odota noin 2 minuuttia ympäristön asennusta varten
```

**Vaihtoehto 2: Paikallinen kehityssäiliö**
```bash
# Kopioi varasto
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Avaa VS Codessa Dev Containers -laajennuksella
# Avaa uudelleen säiliössä, kun sinua kehotetaan
```

**Vaihtoehto 3: Paikallinen asennus**
```bash
# Asenna riippuvuudet
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Vahvista asennus
java -version
mvn -version
```

### Konfigurointi

**Azure AI Foundryn asennus (avaimeton, suositus):**
```bash
# Määritä Foundry-tili + mallin käyttöönotot koodina
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd kirjoittaa examples/basic-chat-azure/.env tiedoston päätepisteelläsi (ei avainta)
```

**Manuaalinen päätepisteen konfigurointi:**
```bash
# Jos et käyttänyt azd:tä, aseta päätepiste itse (todentaminen pysyy avaimettomana az loginin kautta)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Muokkaa .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Kehitystyönkulku

### Projektin rakenne
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

### Sovellusten käynnistäminen

**Spring Boot -sovelluksen käynnistäminen:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Projektin rakentaminen:**
```bash
cd [project-directory]
mvn clean install
```

**MCP-laskimen palvelimen käynnistäminen:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# Palvelin toimii osoitteessa http://localhost:8080
```

**Asiakasesimerkkien suorittaminen:**
```bash
# Käynnistämisen jälkeen palvelin toisessa päätelaitteessa
cd 04-PracticalSamples/calculator

# Suora MCP-asiakas
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# Tekoälypohjainen asiakas (vaatii AZURE_OPENAI_ENDPOINT + az login)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Vuorovaikutteinen botti
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Kuuma uudelleenkäynnistys
Spring Boot DevTools sisältyy projekteihin, jotka tukevat kuumaa uudelleenkäynnistystä:
```bash
# Java-tiedostojen muutokset ladataan automaattisesti uudelleen tallennettaessa
mvn spring-boot:run
```

## Testausohjeet

### Testien suorittaminen

**Suorita kaikki testit projektissa:**
```bash
cd [project-directory]
mvn test
```

**Suorita testit yksityiskohtaisilla tulosteilla:**
```bash
mvn test -X
```

**Suorita tietty testiluokka:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Testauksen rakenne
- Testitiedostot käyttävät JUnit 5:tä (Jupiter)
- Testiluokat sijaitsevat kansiossa `src/test/java/`
- Asiakasesimerkit laskuprojektissa ovat kansiossa `src/test/java/com/microsoft/mcp/sample/client/`

### Manuaalinen testaus
Monet esimerkit ovat interaktiivisia sovelluksia, jotka vaativat manuaalista testausta:

1. Käynnistä sovellus komennolla `mvn spring-boot:run`
2. Testaa päätepisteitä tai toimi komentoriviltä
3. Varmista, että odotettu käyttäytyminen vastaa kunkin projektin README.md-tiedostossa olevaa dokumentaatiota

### Testaus Azure AI Foundryn kanssa
- Kirjaudu sisään `az login` -komennolla ennen esimerkkien suorittamista (avaimeton tunnistus)
- Varmista, että tililläsi on Cognitive Services OpenAI User -rooli resurssissa
- Testaa sisällön suodatus vastuullisen AI-esimerkillä luvussa 5

## Koodityyliohjeet

### Java-konventiot
- **Java-versio:** Java 21 modernien ominaisuuksien kanssa
- **Tyyli:** Noudata vakiintuneita Java-käytäntöjä
- **Nimeäminen:** 
  - Luokat: PascalCase
  - Metodit/muuttujat: camelCase
  - Konstanteille: UPPER_SNAKE_CASE
  - Pakettinimet: pienaakkoset

### Spring Boot -mallit
- Käytä `@Service` liiketoimintalogiikkaan
- Käytä `@RestController` REST-päätepisteisiin
- Konfiguraatio `application.yml` tai `application.properties` -tiedostojen kautta
- Ympäristömuuttujat mieluummin kovakoodattujen arvojen sijaan
- Käytä `@Tool`-annotaatiota MCP:n julkaisemiin metodeihin

### Tiedostojen organisointi
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

### Riippuvuudet
- Hallitaan Mavenin `pom.xml`:n kautta
- Spring AI BOM versionhallintaan
- LangChain4j tekoälyintegraatioita varten
- Spring Boot starter parent Spring-riippuvuuksille

### Koodikommentit
- Lisää JavaDoc julkisiin rajapintoihin
- Sisällytä selittäviä kommentteja monimutkaisista tekoälytoiminnoista
- Dokumentoi MCP-työkalujen kuvaukset selkeästi

## Käännäminen ja käyttöönotto

### Projektien kääntäminen

**Käännä ilman testejä:**
```bash
mvn clean install -DskipTests
```

**Käännä kaikilla tarkistuksilla:**
```bash
mvn clean install
```

**Pakkaa sovellus:**
```bash
mvn package
# Luo JAR-tiedoston target/-hakemistoon
```

### Tulostuskansiot
- Käännetyt luokat: `target/classes/`
- Testiluokat: `target/test-classes/`
- JAR-tiedostot: `target/*.jar`
- Maven-artifaktit: `target/`

### Ympäristökohtainen konfiguraatio

**Kehitys:**
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

**Tuotanto:**
- Käytä hallittua identiteettiä `az login` sijaan avaimettomaan tunnistautumiseen
- Aseta `AZURE_OPENAI_ENDPOINT` tuotantoympäristön Foundryn resurssiin
- Hallitse asetukset ympäristömuuttujilla tai Azure Key Vaultilla

### Käyttöönottoon liittyvät huomioinnit
- Tämä on opetusrekisteri esimerkkisovelluksilla
- Ei suunniteltu suoraan tuotantokäyttöön
- Esimerkit demonstroivat malleja, joita voi soveltaa tuotantoon
- Katso kunkin projektin README:t tarkemmat käyttöönottotiedot

## Lisätietoja

### Azure AI Foundry
- **Avaimeton tunnistus:** yhdistä Microsoft Entra ID:llä — API-avaimia ei tarvitse hallita
- **Provisionointi koodina:** Bicep + azd (`azd up`) luo tilin ja malliasennukset
- Sama OpenAI-yhteensopiva koodi toimii paikallisesti (`az login`) ja Azuren hallitulla identiteetillä

### Usean projektin työskentely
Jokainen esimerkkiprojekti on itsenäinen:
```bash
# Siirry tiettyyn projektiin
cd 04-PracticalSamples/[project-name]

# Jokaisella on oma pom.xml-tiedosto ja ne voidaan kääntää itsenäisesti
mvn clean install
```

### Yleisimmät ongelmat

**Java-version ristiriita:**
```bash
# Vahvista Java 21
java -version
# Päivitä JAVA_HOME tarvittaessa
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Riippuvuuksien latausongelmat:**
```bash
# Tyhjennä Mavenin välimuisti ja yritä uudelleen
rm -rf ~/.m2/repository
mvn clean install
```

**Päätepistettä tai kirjautumista ei löydy:**
```bash
# Aseta palvelupiste nykyiseen istuntoon ja kirjaudu sisään (avaimeton)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Tai käytä .env-tiedostoa projektihakemistossa
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Portti on jo käytössä:**
```bash
# Spring Boot käyttää oletuksena porttia 8080
# Muutos tiedostossa application.properties:
server.port=8081
```

### Monikielinen tuki
- Dokumentaatio saatavilla yli 45 kielellä automatisoidun kääntämisen avulla
- Käännökset kansiossa `translations/`
- Käännöksiä hallitaan GitHub Actions -työnkululla

### Oppimispolku
1. Aloita [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md) -dokumentilla
2. Seuraa lukuja järjestyksessä (01 → 05)
3. Suorita käytännön esimerkit jokaisessa luvussa
4. Tutustu esimerkkiprojekteihin luvussa 4
5. Opi vastuullisen tekoälyn käytännöt luvussa 5

### Kehityssäiliö
Tiedosto `.devcontainer/devcontainer.json` konfiguroi:
- Java 21 kehitysympäristön
- Mavenin valmiiksi asennettuna
- VS Code Java -laajennukset
- Spring Boot -työkalut
- GitHub Copilot -integraation
- Docker-in-Docker -tuen
- Azure CLI:n

### Suorituskykyhuomiot
- Azure AI Foundryn asennuksilla on token-/pyyntörajoituksia minuutissa
- Käytä sopivan kokoisia eräpaketteja upotuksiin
- Harkitse välimuistia toistuvissa API-kutsuissa
- Seuraa tokenin käyttöä kustannusten optimointia varten

### Turvallisuusmuistutukset
- Älä koskaan tallenna `.env`-tiedostoja versiohallintaan (on jo `.gitignore`-tiedostossa)
- Suosi avaimetonta tunnistautumista (Microsoft Entra ID) API-avainten sijaan
- Käytä Azure-ympäristössä hallittuja identiteettejä; paikallisessa kehityksessä `az login` -kirjautumista
- Noudata vastuullisen tekoälyn ohjeita luvussa 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Vastuuvapauslauseke**:
Tämä asiakirja on käännetty käyttämällä tekoälypohjaista käännöspalvelua [Co-op Translator](https://github.com/Azure/co-op-translator). Vaikka pyrimme tarkkuuteen, otathan huomioon, että automaattiset käännökset saattavat sisältää virheitä tai epätarkkuuksia. Alkuperäinen asiakirja sen alkuperäiskielellä on virallinen lähde. Tärkeissä asioissa suositellaan ammattimaista ihmiskäännöstä. Emme ole vastuussa tämän käännöksen käytöstä aiheutuvista väärinymmärryksistä tai tulkinnoista.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->