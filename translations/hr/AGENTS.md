# AGENTS.md

## Pregled Projekta

Ovo je edukativni repozitorij za učenje razvoja Generative AI uz Java. Pruža sveobuhvatan praktični tečaj koji pokriva Velike Jezične Modele (LLM-ove), inženjering prompta, ugradnje (embeddings), RAG (Retrieval-Augmented Generation) i Protokol za Kontekst Modela (MCP).

**Ključne Tehnologije:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI i OpenAI SDK-ovi

**Arhitektura:**
- Više samostalnih Spring Boot aplikacija organiziranih po poglavljima
- Primjeri projekata koji demonstriraju različite AI obrasce
- GitHub Codespaces spreman s unaprijed konfiguriranim razvojnim kontejnerima

## Komande za Postavljanje

### Preduvjeti
- Java 21 ili noviji
- Maven 3.x
- Azure pretplata s Azure AI Foundry model deployment-om (osigurajte s `azd up`)
- Azure CLI (`az`) i Azure Developer CLI (`azd`), prijavljeni za autentifikaciju bez ključeva

### Postavljanje Okruženja

**Opcija 1: GitHub Codespaces (Preporučeno)**
```bash
# Napravi fork repozitorija i kreiraj codespace iz GitHub sučelja
# Razvojno okruženje će automatski instalirati sve ovisnosti
# Pričekaj ~2 minute za postavljanje okruženja
```

**Opcija 2: Lokalni Razvojni Kontejner**
```bash
# Klonirajte repozitorij
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Otvorite u VS Code s Dev Containers ekstenzijom
# Ponovno otvorite u kontejneru kad se to zatraži
```

**Opcija 3: Lokalno Postavljanje**
```bash
# Instaliraj ovisnosti
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Provjeri instalaciju
java -version
mvn -version
```

### Konfiguracija

**Azure AI Foundry Postavljanje (bez ključeva, preporučeno):**
```bash
# Postavite Foundry račun + implementacije modela kao kod
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd zapisuje examples/basic-chat-azure/.env s vašom krajnjom točkom (bez ključa)
```

**Ručno konfiguriranje krajnje točke:**
```bash
# Ako niste koristili azd, sami postavite krajnju točku (autentifikacija ostaje bez ključa putem az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Uredite .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Razvojni Tijek

### Struktura Projekta
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

### Pokretanje Aplikacija

**Pokretanje Spring Boot aplikacije:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Izgradnja projekta:**
```bash
cd [project-directory]
mvn clean install
```

**Pokretanje MCP Calculator Servera:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# Poslužitelj radi na http://localhost:8080
```

**Pokretanje Client Primjera:**
```bash
# Nakon pokretanja poslužitelja u drugom terminalu
cd 04-PracticalSamples/calculator

# Izravni MCP klijent
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# Klijent s umjetnom inteligencijom (zahtijeva AZURE_OPENAI_ENDPOINT + az prijavu)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Interaktivni bot
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Hot Reload
Spring Boot DevTools je uključen u projekte koji podržavaju hot reload:
```bash
# Promjene na Java datotekama automatski će se ponovno učitati kada se spremi
mvn spring-boot:run
```

## Upute za Testiranje

### Pokretanje Testova

**Pokreni sve testove u projektu:**
```bash
cd [project-directory]
mvn test
```

**Pokreni testove s detaljnim ispisom:**
```bash
mvn test -X
```

**Pokreni određenu test klasu:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Struktura Testova
- Test datoteke koriste JUnit 5 (Jupiter)
- Test klase su smještene u `src/test/java/`
- Klijentski primjeri u calculator projektu nalaze se u `src/test/java/com/microsoft/mcp/sample/client/`

### Ručno Testiranje
Mnogi primjeri su interaktivne aplikacije koje zahtijevaju ručno testiranje:

1. Pokrenite aplikaciju s `mvn spring-boot:run`
2. Testirajte krajnje točke ili komunicirajte s CLI-em
3. Provjerite da očekivano ponašanje odgovara dokumentaciji u README.md svakog projekta

### Testiranje s Azure AI Foundry
- Prijavite se s `az login` prije pokretanja primjera (autentifikacija bez ključeva)
- Osigurajte da vaš račun ima ulogu Cognitive Services OpenAI User na resursu
- Testirajte filtriranje sadržaja s primjerom odgovornog AI u Poglavlju 5

## Smjernice za Stil Koda

### Java Konvencije
- **Verzija Jave:** Java 21 s modernim značajkama
- **Stil:** Slijedite standardne Java konvencije
- **Nazivi:** 
  - Klase: PascalCase
  - Metode/varijable: camelCase
  - Konstante: UPPER_SNAKE_CASE
  - Imena paketa: mala slova

### Spring Boot Obrasci
- Koristite `@Service` za poslovnu logiku
- Koristite `@RestController` za REST krajnje točke
- Konfiguracija preko `application.yml` ili `application.properties`
- Varijable okruženja su preferirane umjesto hardkodiranih vrijednosti
- Koristite `@Tool` oznaku za metode izložene putem MCP-a

### Organizacija Datoteka
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

### Ovisnosti
- Upravljano putem Maven `pom.xml`
- Spring AI BOM za upravljanje verzijama
- LangChain4j za AI integracije
- Spring Boot starter parent za Spring ovisnosti

### Komentari u Kod
- Dodajte JavaDoc za javne API-je
- Uključite objašnjavajuće komentare za složene AI interakcije
- Jasno dokumentirajte opise MCP alata

## Izgradnja i Deploy

### Izgradnja Projekata

**Izgradi bez testova:**
```bash
mvn clean install -DskipTests
```

**Izgradi sa svim provjerama:**
```bash
mvn clean install
```

**Paketiraj aplikaciju:**
```bash
mvn package
# Stvara JAR u direktoriju target/
```

### Izlazni Direktoriji
- Kompilirane klase: `target/classes/`
- Test klase: `target/test-classes/`
- JAR datoteke: `target/*.jar`
- Maven artefakti: `target/`

### Konfiguracija Specifična za Okruženje

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
- Koristite upravljani identitet umjesto `az login` za autentifikaciju bez ključeva
- Postavite `AZURE_OPENAI_ENDPOINT` na vaš produkcijski Foundry resurs
- Upravljanje konfiguracijom preko varijabli okruženja ili Azure Key Vault-a

### Razmatranja za Deploy
- Ovo je edukativni repozitorij s primjerima aplikacija
- Nije dizajniran za produkcijski deploy takav kakav jest
- Primjeri demonstriraju obrasce za prilagodbu produkcijskoj upotrebi
- Pogledajte README-ove pojedinačnih projekata za specifične napomene o deployu

## Dodatne Napomene

### Azure AI Foundry
- **Autentifikacija bez ključeva:** povezuje se s Microsoft Entra ID — nema potrebe za upravljanjem API ključevima
- **Provisioniranje kao kod:** Bicep + azd (`azd up`) kreiraju račun i model deployment-e
- Isti OpenAI-kompatibilni kod radi lokalno (`az login`) i u Azureu (upravljani identitet)

### Rad s Više Projekata
Svaki primjer projekta je samostalan:
```bash
# Navigiraj do određenog projekta
cd 04-PracticalSamples/[project-name]

# Svaki ima svoj pom.xml i može se graditi neovisno
mvn clean install
```

### Česti Problemi

**Neusklađenost verzije Jave:**
```bash
# Provjerite Java 21
java -version
# Ažurirajte JAVA_HOME ako je potrebno
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Problemi s preuzimanjem ovisnosti:**
```bash
# Očisti Maven predmemoriju i pokušaj ponovno
rm -rf ~/.m2/repository
mvn clean install
```

**Krajnja točka ili prijava nije pronađena:**
```bash
# Postavite endpoint u trenutnoj sesiji i prijavite se (bez ključa)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Ili koristite .env datoteku u direktoriju projekta
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Port je već zauzet:**
```bash
# Spring Boot prema zadanim postavkama koristi port 8080
# Promjena u application.properties:
server.port=8081
```

### Podrška za Više Jezika
- Dokumentacija dostupna na 45+ jezika putem automatskog prevođenja
- Prijevodi u direktoriju `translations/`
- Prijevod upravljan GitHub Actions workflow-om

### Put Učenja
1. Počnite s [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. Slijedite poglavlja redom (01 → 05)
3. Završite praktične primjere u svakom poglavlju
4. Istražite primjere projekata u Poglavlju 4
5. Učite prakse odgovornog AI u Poglavlju 5

### Razvojni Kontejner
`.devcontainer/devcontainer.json` konfigurira:
- Java 21 razvojno okruženje
- Maven unaprijed instaliran
- VS Code Java ekstenzije
- Spring Boot alati
- GitHub Copilot integracija
- Docker unutar Dockera podrška
- Azure CLI

### Razmatranja Performansi
- Azure AI Foundry deploymenti imaju kvote tokena/zahtjeva po minuti
- Koristite odgovarajuće veličine batch-a za ugradnje
- Razmotrite caching za ponovljene API pozive
- Pratite korištenje tokena radi optimizacije troškova

### Napomene o Sigurnosti
- Nikada ne commitajte `.env` datoteke (već su u `.gitignore`)
- Preferirajte autentifikaciju bez ključeva (Microsoft Entra ID) umjesto API ključeva
- Koristite upravljane identitete u Azureu; `az login` za lokalni razvoj
- Slijedite smjernice odgovornog AI-ja u Poglavlju 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Napomena**:
Ovaj dokument je preveden korištenjem AI prevoditeljskog servisa [Co-op Translator](https://github.com/Azure/co-op-translator). Iako težimo točnosti, imajte na umu da automatski prijevodi mogu sadržavati greške ili netočnosti. Izvorni dokument na izvornom jeziku treba smatrati autoritativnim izvorom. Za važne informacije preporuča se profesionalni ljudski prijevod. Nismo odgovorni za bilo kakva nesporazumevanja ili pogrešne interpretacije koje proizlaze iz korištenja ovog prijevoda.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->