# AGENTS.md

## Přehled projektu

Toto je vzdělávací repozitář pro učení vývoje Generativní AI v Javě. Poskytuje komplexní praktický kurz pokrývající Velké jazykové modely (LLM), návrh promptů, embeddings, RAG (Retrieval-Augmented Generation) a Model Context Protocol (MCP).

**Klíčové technologie:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI a OpenAI SDK

**Architektura:**
- Více samostatných Spring Boot aplikací uspořádaných podle kapitol
- Ukázkové projekty demonstrující různé AI vzory
- Připraveno pro GitHub Codespaces s předkonfigurovanými vývojovými kontejnery

## Příkazy pro nastavení

### Požadavky
- Java 21 nebo vyšší
- Maven 3.x
- Azure předplatné s nasazením modelu Azure AI Foundry (provision přes `azd up`)
- Azure CLI (`az`) a Azure Developer CLI (`azd`), přihlášení pro keyless autentizaci

### Nastavení prostředí

**Možnost 1: GitHub Codespaces (doporučeno)**
```bash
# Vytvořte fork repozitáře a vytvořte codespace přes GitHub UI
# Vývojové kontejner automaticky nainstaluje všechny závislosti
# Počkejte přibližně 2 minuty na nastavení prostředí
```

**Možnost 2: Lokální vývojový kontejner**
```bash
# Klonovat repozitář
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Otevřít ve VS Code s rozšířením Dev Containers
# Při výzvě znovu otevřít v kontejneru
```

**Možnost 3: Lokální nastavení**
```bash
# Nainstalujte závislosti
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Ověřte instalaci
java -version
mvn -version
```

### Konfigurace

**Nastavení Azure AI Foundry (bez klíče, doporučeno):**
```bash
# Zajistěte účet Foundry + nasazení modelů jako kód
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd zapíše examples/basic-chat-azure/.env s vaším koncovým bodem (bez klíče)
```

**Manuální konfigurace endpointu:**
```bash
# Pokud jste nepoužili azd, nastavte si koncový bod sami (autentizace zůstává bezklíčová přes az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Upravit .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Vývojový workflow

### Struktura projektu
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

### Spuštění aplikací

**Spuštění Spring Boot aplikace:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Sestavení projektu:**
```bash
cd [project-directory]
mvn clean install
```

**Zahájení MCP Calculator Serveru:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# Server běží na http://localhost:8080
```

**Spuštění klientských příkladů:**
```bash
# Po spuštění serveru v jiném terminálu
cd 04-PracticalSamples/calculator

# Přímý MCP klient
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# Klient poháněný AI (vyžaduje AZURE_OPENAI_ENDPOINT + az login)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Interaktivní bot
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Hot Reload
Spring Boot DevTools je zahrnuto v projektech podporujících hot reload:
```bash
# Změny v Java souborech se automaticky znovu načtou při uložení
mvn spring-boot:run
```

## Instrukce pro testování

### Spuštění testů

**Spustit všechny testy v projektu:**
```bash
cd [project-directory]
mvn test
```

**Spustit testy s podrobným výstupem:**
```bash
mvn test -X
```

**Spustit konkrétní testovací třídu:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Struktura testů
- Testovací soubory používají JUnit 5 (Jupiter)
- Testovací třídy jsou umístěny v `src/test/java/`
- Klientské příklady v calculator projektu jsou v `src/test/java/com/microsoft/mcp/sample/client/`

### Manuální testování
Mnoho příkladů jsou interaktivní aplikace, které vyžadují manuální testování:

1. Spusťte aplikaci příkazem `mvn spring-boot:run`
2. Testujte endpointy nebo pracujte s CLI
3. Ověřte, zda chování odpovídá dokumentaci v README.md každého projektu

### Testování s Azure AI Foundry
- Přihlaste se pomocí `az login` před spuštěním příkladů (keyless autentizace)
- Ujistěte se, že vaše konto má roli Cognitive Services OpenAI User na zdroji
- Testujte filtrování obsahu pomocí responsible AI příkladu v kapitole 5

## Pravidla stylu kódu

### Java konvence
- **Verze Javy:** Java 21 s moderními funkcemi
- **Styl:** Dodržujte standardní Java konvence
- **Pojmenování:** 
  - Třídy: PascalCase
  - Metody/proměnné: camelCase
  - Konstanty: UPPER_SNAKE_CASE
  - Balíčky: malá písmena

### Spring Boot vzory
- Používejte `@Service` pro business logiku
- Používejte `@RestController` pro REST endpointy
- Konfigurace přes `application.yml` nebo `application.properties`
- Preferujte proměnné prostředí před pevně zakódovanými hodnotami
- Používejte anotaci `@Tool` pro metody vystavené MCP

### Organizace souborů
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

### Závislosti
- Spravováno přes Maven `pom.xml`
- Spring AI BOM pro řízení verzí
- LangChain4j pro AI integrace
- Spring Boot starter parent pro Spring závislosti

### Komentáře v kódu
- Přidávejte JavaDoc pro veřejné API
- Vkládejte vysvětlující komentáře pro složité AI interakce
- Jasně dokumentujte popisy nástrojů MCP

## Sestavení a nasazení

### Sestavení projektů

**Sestavit bez testů:**
```bash
mvn clean install -DskipTests
```

**Sestavit se všemi kontrolami:**
```bash
mvn clean install
```

**Zabalit aplikaci:**
```bash
mvn package
# Vytváří JAR v adresáři target/
```

### Výstupní adresáře
- Přeložené třídy: `target/classes/`
- Testovací třídy: `target/test-classes/`
- JAR soubory: `target/*.jar`
- Maven artefakty: `target/`

### Konfigurace specifická pro prostředí

**Vývoj:**
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

**Produkce:**
- Používejte spravovanou identitu místo `az login` pro keyless autentizaci
- Nastavte `AZURE_OPENAI_ENDPOINT` na váš produkční Foundry zdroj
- Spravujte konfiguraci přes proměnné prostředí nebo Azure Key Vault

### Poznámky k nasazení
- Tento repozitář je vzdělávací s ukázkovými aplikacemi
- Není určen pro přímé produkční nasazení
- Ukázky ilustrují vzory pro produkční použití
- Pro konkrétní poznámky k nasazení viz README jednotlivých projektů

## Další poznámky

### Azure AI Foundry
- **Keyless autentizace:** připojení s Microsoft Entra ID — nejsou potřeba API klíče
- **Provisioning jako kód:** Bicep + azd (`azd up`) vytvoří účet a nasazení modelů
- Stejný kód kompatibilní s OpenAI běží lokálně (`az login`) i v Azure (spravovaná identita)

### Práce s více projekty
Každý ukázkový projekt je samostatný:
```bash
# Přejít na konkrétní projekt
cd 04-PracticalSamples/[project-name]

# Každý má svůj vlastní pom.xml a může být sestaven samostatně
mvn clean install
```

### Běžné problémy

**Neshoda verze Javy:**
```bash
# Ověřit Java 21
java -version
# Aktualizovat JAVA_HOME, pokud je to potřeba
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Problémy se stahováním závislostí:**
```bash
# Vyčistěte mezipaměť Maven a zkuste to znovu
rm -rf ~/.m2/repository
mvn clean install
```

**Nenašel se endpoint nebo přihlášení:**
```bash
# Nastavte koncový bod v aktuální relaci a přihlaste se (bez klíče)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Nebo použijte soubor .env v adresáři projektu
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Port již používán:**
```bash
# Spring Boot ve výchozím nastavení používá port 8080
# Změna v application.properties:
server.port=8081
```

### Podpora vícejazyčnosti
- Dokumentace dostupná ve více než 45 jazycích přes automatický překlad
- Překlady v adresáři `translations/`
- Překlad spravován workflow GitHub Actions

### Učební cesta
1. Začněte s [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. Postupujte kapitolami v pořadí (01 → 05)
3. Splňte praktické příklady v každé kapitole
4. Prozkoumejte ukázkové projekty v kapitole 4
5. Naučte se zásady responsible AI v kapitole 5

### Vývojový kontejner
Soubor `.devcontainer/devcontainer.json` konfiguruje:
- Vývojové prostředí Java 21
- Předinstalovaný Maven
- VS Code Java rozšíření
- Nástroje Spring Boot
- Integraci GitHub Copilot
- Docker-in-Docker podporu
- Azure CLI

### Výkonnostní poznámky
- Nasazení Azure AI Foundry mají limity tokenů/požadavků za minutu
- Používejte vhodné velikosti batchů pro embeddings
- Zvažte cachování pro opakované API volání
- Sledujte využití tokenů pro optimalizaci nákladů

### Bezpečnostní poznámky
- Nikdy necommitujte soubory `.env` (jsou v `.gitignore`)
- Preferujte keyless autentizaci (Microsoft Entra ID) před API klíči
- Používejte spravované identity v Azure; `az login` pro lokální vývoj
- Dodržujte zásady responsible AI z kapitoly 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Prohlášení o omezení odpovědnosti**:
Tento dokument byl přeložen pomocí AI překladatelské služby [Co-op Translator](https://github.com/Azure/co-op-translator). Přestože usilujeme o co největší přesnost, mějte prosím na paměti, že automatizované překlady mohou obsahovat chyby nebo nepřesnosti. Originální dokument v jeho mateřském jazyce by měl být považován za autoritativní zdroj. Pro kritické informace se doporučuje profesionální lidský překlad. Nejsme odpovědní za jakékoli nedorozumění nebo nesprávné interpretace vzniklé použitím tohoto překladu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->