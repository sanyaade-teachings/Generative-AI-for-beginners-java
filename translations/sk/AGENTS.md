# AGENTS.md

## Prehľad projektu

Toto je vzdelávacie úložisko na učenie vývoja Generatívnej AI s Java. Poskytuje komplexný praktický kurz pokrývajúci Veľké jazykové modely (LLMs), prompt engineering, embeddingy, RAG (Retrieval-Augmented Generation) a Model Context Protocol (MCP).

**Kľúčové technológie:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI a OpenAI SDK

**Architektúra:**
- Viacero samostatných aplikácií Spring Boot organizovaných podľa kapitol
- Ukážkové projekty demonštrujúce rôzne AI vzory
- Pripravené na GitHub Codespaces s predkonfigurovanými vývojárskymi kontajnermi

## Príkazy na nastavenie

### Požiadavky
- Java 21 alebo novšia
- Maven 3.x
- Predplatné Azure s nasadením modelu Azure AI Foundry (provízia pomocou `azd up`)
- Azure CLI (`az`) a Azure Developer CLI (`azd`), prihlásené pre bezkľúčovú autentifikáciu

### Nastavenie prostredia

**Možnosť 1: GitHub Codespaces (odporúčané)**
```bash
# Vytvorte fork úložiska a vytvorte codespace z používateľského rozhrania GitHub
# Vývojové kontajner automaticky nainštaluje všetky závislosti
# Počkajte približne 2 minúty na nastavenie prostredia
```

**Možnosť 2: Lokálny vývojársky kontajner**
```bash
# Klonovať úložisko
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Otvorte vo VS Code s rozšírením Dev Containers
# Znova otvoriť v kontajneri, keď budete vyzvaní
```

**Možnosť 3: Lokálne nastavenie**
```bash
# Nainštalujte závislosti
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Overte inštaláciu
java -version
mvn -version
```

### Konfigurácia

**Nastavenie Azure AI Foundry (bezklúčové, odporúčané):**
```bash
# Poskytnite účet Foundry + nasadenia modelu ako kód
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd zapisuje examples/basic-chat-azure/.env s vaším koncovým bodom (bez kľúča)
```

**Manuálna konfigurácia endpointu:**
```bash
# Ak ste nepoužili azd, nastavte si koncový bod sami (autentifikácia zostáva bez kľúča cez az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Upravte .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Vývojový pracovný postup

### Štruktúra projektu
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

### Spúšťanie aplikácií

**Spustenie Spring Boot aplikácie:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Skompilovanie projektu:**
```bash
cd [project-directory]
mvn clean install
```

**Spustenie MCP Calculator Servera:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# Server beží na http://localhost:8080
```

**Spustenie ukážok klienta:**
```bash
# Po spustení servera v inom termináli
cd 04-PracticalSamples/calculator

# Priamy MCP klient
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# Klient poháňaný AI (vyžaduje AZURE_OPENAI_ENDPOINT + az prihlásenie)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Interaktívny bot
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Hot Reload
Spring Boot DevTools je zahrnutý v projektoch, ktoré podporujú hot reload:
```bash
# Zmeny v súboroch Java sa automaticky znovu načítajú po uložení
mvn spring-boot:run
```

## Pokyny na testovanie

### Spustenie testov

**Spustiť všetky testy v projekte:**
```bash
cd [project-directory]
mvn test
```

**Spustiť testy s podrobným výstupom:**
```bash
mvn test -X
```

**Spustiť konkrétnu testovaciu triedu:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Štruktúra testov
- Testovacie súbory používajú JUnit 5 (Jupiter)
- Testovacie triedy sú umiestnené v `src/test/java/`
- Ukážky klienta v projekte kalkulačky sú v `src/test/java/com/microsoft/mcp/sample/client/`

### Manuálne testovanie
Mnohé príklady sú interaktívne aplikácie vyžadujúce manuálne testovanie:

1. Spustite aplikáciu pomocou `mvn spring-boot:run`
2. Testujte endpointy alebo interagujte s CLI
3. Overte, či sa očakávané správanie zhoduje s dokumentáciou v README.md každého projektu

### Testovanie s Azure AI Foundry
- Prihláste sa pomocou `az login` pred spustením príkladov (bezklúčová autentifikácia)
- Uistite sa, že váš účet má rolu Cognitive Services OpenAI User na zdroji
- Otestujte filtrovanie obsahu pomocou príkladu zodpovednej AI v kapitole 5

## Pokyny ku kódovaciemu štýlu

### Java konvencie
- **Verzia Java:** Java 21 s modernými funkciami
- **Štýl:** Dodržiavajte štandardné Java konvencie
- **Názvy:** 
  - Triedy: PascalCase
  - Metódy/premenné: camelCase
  - Konštanty: UPPER_SNAKE_CASE
  - Názvy balíkov: malé písmená

### Spring Boot vzory
- Používajte `@Service` pre biznis logiku
- Používajte `@RestController` pre REST endpointy
- Konfigurácia cez `application.yml` alebo `application.properties`
- Preferujte environmentálne premenné pred hardcoded hodnotami
- Používajte anotáciu `@Tool` pre metódy sprístupnené MCP

### Organizácia súborov
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
- Spravované cez Maven `pom.xml`
- Spring AI BOM pre správu verzií
- LangChain4j pre AI integrácie
- Spring Boot starter parent pre Spring závislosti

### Komentáre v kóde
- Pridávajte JavaDoc pre verejné API
- Zahrňte vysvetľujúce komentáre pre zložité AI interakcie
- Dokumentujte jasne popisy MCP nástrojov

## Kompilácia a nasadenie

### Kompilácia projektov

**Kompilovať bez testov:**
```bash
mvn clean install -DskipTests
```

**Kompilovať so všetkými kontrolami:**
```bash
mvn clean install
```

**Zabaliť aplikáciu:**
```bash
mvn package
# Vytvára JAR v priečinku target/
```

### Výstupné adresáre
- Skopilované triedy: `target/classes/`
- Testovacie triedy: `target/test-classes/`
- JAR súbory: `target/*.jar`
- Maven artefakty: `target/`

### Konfigurácia podľa prostredia

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

**Produkcia:**
- Použite spravovanú identitu namiesto `az login` pre bezkľúčovú autentifikáciu
- Nasmerujte `AZURE_OPENAI_ENDPOINT` na váš produkčný Foundry zdroj
- Spravujte konfiguráciu cez environmentálne premenne alebo Azure Key Vault

### Poznámky k nasadeniu
- Toto je vzdelávacie úložisko s ukážkovými aplikáciami
- Nie je určené na priame nasadenie do produkcie
- Ukážky demonštrujú vzory, ktoré je potrebné prispôsobiť pre produkčné použitie
- Pozrite README každého projektu pre konkrétne poznámky k nasadeniu

## Ďalšie poznámky

### Azure AI Foundry
- **Bezkľúčová autentifikácia:** pripojenie cez Microsoft Entra ID — žiadne API kľúče na spravovanie
- **Provízia ako kód:** Bicep + azd (`azd up`) vytvoria účet a nasadenia modelov
- Rovnaký kód kompatibilný s OpenAI beží lokálne (`az login`) aj v Azure (spravovaná identita)

### Práca s viacerými projektmi
Každý ukážkový projekt je samostatný:
```bash
# Prejdite na konkrétny projekt
cd 04-PracticalSamples/[project-name]

# Každý má svoj vlastný pom.xml a môže byť zostavený nezávisle
mvn clean install
```

### Bežné problémy

**Nesúlad verzie Java:**
```bash
# Overiť Java 21
java -version
# Aktualizovať JAVA_HOME, ak je potrebné
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Problémy so sťahovaním závislostí:**
```bash
# Vymazať Maven cache a skúsiť znova
rm -rf ~/.m2/repository
mvn clean install
```

**Endpoint alebo prihlásenie nenájdené:**
```bash
# Nastavte koncový bod v aktuálnej relácii a prihláste sa (bez kľúča)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Alebo použite súbor .env v adresári projektu
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Port už používaný:**
```bash
# Spring Boot štandardne používa port 8080
# Zmena v application.properties:
server.port=8081
```

### Podpora viacerých jazykov
- Dokumentácia dostupná vo viac než 45 jazykoch cez automatizovaný preklad
- Preklady v adresári `translations/`
- Preklady riadené pomocou Github Actions workflow

### Vzdelávacia cesta
1. Začnite s [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. Nasledujte kapitoly v poradí (01 → 05)
3. Dokončite praktické príklady v každej kapitole
4. Preskúmajte ukážkové projekty v kapitole 4
5. Naučte sa zodpovedné AI praktiky v kapitole 5

### Vývojársky kontajner
`.devcontainer/devcontainer.json` konfiguruje:
- Vývojové prostredie Java 21
- Predinštalovaný Maven
- VS Code Java rozšírenia
- Nástroje Spring Boot
- Integráciu GitHub Copilot
- Podporu Docker-in-Docker
- Azure CLI

### Výkonové úvahy
- Nasadenia Azure AI Foundry majú kvóty tokenov/požiadaviek za minútu
- Používajte vhodné veľkosti dávok pre embeddingy
- Zvážte cache pri opakovaných API volaniach
- Sledujte využitie tokenov pre optimalizáciu nákladov

### Bezpečnostné poznámky
- Nikdy neukladajte `.env` súbory do repozitára (sú v `.gitignore`)
- Preferujte bezkľúčovú autentifikáciu (Microsoft Entra ID) namiesto API kľúčov
- Používajte spravované identity v Azure; `az login` pre lokálny vývoj
- Dodržiavajte zásady zodpovednej AI v kapitole 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Vyhlásenie o zodpovednosti**:
Tento dokument bol preložený pomocou AI prekladateľskej služby [Co-op Translator](https://github.com/Azure/co-op-translator). Hoci sa snažíme o presnosť, vezmite prosím na vedomie, že automatické preklady môžu obsahovať chyby alebo nepresnosti. Pôvodný dokument v jeho natívnom jazyku by mal byť považovaný za autoritatívny zdroj. Pre kritické informácie sa odporúča profesionálny ľudský preklad. Nie sme zodpovední za žiadne nedorozumenia alebo nesprávne interpretácie vyplývajúce z použitia tohto prekladu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->