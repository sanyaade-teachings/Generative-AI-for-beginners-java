# AGENTS.md

## Projekt áttekintése

Ez egy oktató jellegű tároló a Generatív AI fejlesztés Java nyelven történő elsajátításához. Átfogó, gyakorlati tanfolyamot biztosít a Nagynyelvű modellek (LLM-ek), prompttervezés, beágyazások, RAG (Retrieval-Augmented Generation) és a Model Context Protocol (MCP) témakörében.

**Főbb technológiák:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI és OpenAI SDK-k

**Architektúra:**
- Több önálló Spring Boot alkalmazás fejezetenként szervezve
- Minta projektek, amelyek különböző AI mintákat mutatnak be
- GitHub Codespaces kompatibilis előre konfigurált fejlesztői konténerekkel

## Beállítási parancsok

### Előfeltételek
- Java 21 vagy újabb
- Maven 3.x
- Azure előfizetés Azure AI Foundry modell telepítéssel (az `azd up` paranccsal konfigurálható)
- Azure CLI (`az`) és Azure Developer CLI (`azd`), bejelentkezve kulcs nélküli hitelesítéshez

### Környezet beállítása

**1. lehetőség: GitHub Codespaces (ajánlott)**
```bash
# Ágazz el a tárházat, és hozz létre egy kódtérhelyet a GitHub felhasználói felületéről
# A fejlesztői tároló automatikusan telepíti az összes függőséget
# Várj kb. 2 percet a környezet beállítására
```

**2. lehetőség: helyi fejlesztői konténer**
```bash
# Tároló klónozása
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Megnyitás VS Code-ban a Dev Containers bővítménnyel
# Újra megnyitás konténerben, ha kéri
```

**3. lehetőség: helyi beállítás**
```bash
# Függőségek telepítése
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Telepítés ellenőrzése
java -version
mvn -version
```

### Konfiguráció

**Azure AI Foundry beállítása (kulcs nélküli, ajánlott):**
```bash
# A Foundry fiók és a modell telepítések kód általi előkészítése
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd létrehozza az examples/basic-chat-azure/.env fájlt a végpontoddal (kulcs nélkül)
```

**Kézi végpont konfiguráció:**
```bash
# Ha nem használtad az azd-t, állítsd be magad az végpontot (az autentikáció kulcs nélküli marad az az login keresztül)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Szerkeszd a .env-et: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Fejlesztési munkafolyamat

### Projekt felépítése
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

### Alkalmazások futtatása

**Spring Boot alkalmazás futtatása:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Projekt építése:**
```bash
cd [project-directory]
mvn clean install
```

**MCP kalkulátor szerver indítása:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# A szerver a http://localhost:8080 címen fut
```

**Ügyfél példák futtatása:**
```bash
# A szerver indítása után egy másik terminálban
cd 04-PracticalSamples/calculator

# Közvetlen MCP kliens
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# Mesterséges intelligenciával működő kliens (AZURE_OPENAI_ENDPOINT + az bejelentkezés szükséges)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Interaktív bot
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Hot Reload
A Spring Boot DevTools benne van azokban a projektekben, amelyek támogatják a hot reloadot:
```bash
# A Java fájlokban végzett módosítások mentéskor automatikusan újratöltődnek
mvn spring-boot:run
```

## Tesztelési utasítások

### Tesztek futtatása

**Minden teszt futtatása egy projektben:**
```bash
cd [project-directory]
mvn test
```

**Tesztek futtatása részletes kimenettel:**
```bash
mvn test -X
```

**Konkrét teszt osztály futtatása:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Teszt struktúra
- A tesztfájlok JUnit 5 (Jupiter) használatával készültek
- Teszt osztályok a `src/test/java/` könyvtárban találhatók
- A kalkulátor projekt kliens példái a `src/test/java/com/microsoft/mcp/sample/client/` mappában vannak

### Kézi tesztelés
Sok példa interaktív alkalmazás, amely kézi tesztelést igényel:

1. Indítsd el az alkalmazást a `mvn spring-boot:run` paranccsal
2. Teszteld az végpontokat vagy lépj interakcióba a CLI-vel
3. Ellenőrizd, hogy a várt viselkedés megfelel-e a dokumentációban szereplő leírásnak az egyes projektek README.md fájljaiban

### Tesztelés Azure AI Foundry-val
- Jelentkezz be az `az login` parancssal az példák futtatása előtt (kulcs nélküli hitelesítés)
- Győződj meg róla, hogy a fiókodnak Cognitive Services OpenAI User jogosultsága van az erőforráson
- Teszteld a tartalomszűrést a felelős AI példával az 5. fejezetben

## Kódstílus irányelvek

### Java konvenciók
- **Java verzió:** Java 21 modern funkciókkal
- **Stílus:** Alkalmazd a szabványos Java konvenciókat
- **Névhasználat:** 
  - Osztályok: PascalCase
  - Metódusok/változók: camelCase
  - Konstansok: UPPER_SNAKE_CASE
  - Csomagnevek: kisbetűs

### Spring Boot minták
- Használd az `@Service` annotációt az üzleti logikához
- Használd az `@RestController` annotációt REST végpontokhoz
- Konfiguráció `application.yml` vagy `application.properties` fájllal
- Környezeti változók előnyben a kódba ágyazott értékekkel szemben
- Használd az `@Tool` annotációt az MCP számára elérhető metódusokhoz

### Fájl szervezés
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

### Függőségek
- Maven `pom.xml` kezeli őket
- Spring AI BOM a verziókezeléshez
- LangChain4j az AI integrációkhoz
- Spring Boot starter parent a Spring függőségekhez

### Kódkommentek
- Írj JavaDoc kommenteket a publikus API-khoz
- Tartalmazz magyarázó kommenteket a komplex AI-interakcióknál
- Dokumentáld világosan az MCP eszközök leírásait

## Építés és telepítés

### Projektek építése

**Építés tesztek nélkül:**
```bash
mvn clean install -DskipTests
```

**Építés minden ellenőrzéssel:**
```bash
mvn clean install
```

**Alkalmazás csomagolása:**
```bash
mvn package
# Létrehozza a JAR-t a target/ könyvtárban
```

### Kimeneti könyvtárak
- Fordított osztályok: `target/classes/`
- Teszt osztályok: `target/test-classes/`
- JAR fájlok: `target/*.jar`
- Maven artefaktumok: `target/`

### Környezet-specifikus konfiguráció

**Fejlesztés:**
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

**Éles környezet:**
- Kezelt identity használata `az login` helyett kulcs nélküli hitelesítéshez
- Állítsd be az `AZURE_OPENAI_ENDPOINT` változót az éles Foundry erőforrásra
- Konfiguráció környezeti változókon vagy Azure Key Vault-on keresztül

### Telepítési megfontolások
- Ez egy oktató jellegű tároló mintaalkalmazásokkal
- Nem kifejezetten éles üzembehelyezésre kész
- A minták bemutatják azokat a mintákat, amelyeket éles környezethez alakíthatsz
- Egyedi projekt README-k tartalmazzák a specifikus telepítési megjegyzéseket

## További megjegyzések

### Azure AI Foundry
- **Kulcs nélküli hitelesítés:** Microsoft Entra ID csatlakozás — nincs API kulcs kezelés
- **Provisionolt kódként:** Bicep + azd (`azd up`) létrehozza a fiókot és a modell telepítéseket
- Ugyanaz az OpenAI-kompatibilis kód fut helyben (`az login`) és Azure-ban (kezelt identity)

### Több projekt kezelése
Minden minta projekt önálló:
```bash
# Navigáljon egy adott projekthez
cd 04-PracticalSamples/[project-name]

# Mindegyiknek megvan a saját pom.xml fájlja, és önállóan is felépíthető
mvn clean install
```

### Gyakori problémák

**Java verzió eltérés:**
```bash
# Ellenőrizze a Java 21-et
java -version
# Szükség esetén frissítse a JAVA_HOME változót
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Függőség letöltési problémák:**
```bash
# Töröld a Maven gyorsítótárát, és próbáld újra
rm -rf ~/.m2/repository
mvn clean install
```

**Végpont vagy bejelentkezés hiánya:**
```bash
# Állítsa be a végpontot az aktuális munkamenetben és jelentkezzen be (kulcs nélkül)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Vagy használjon .env fájlt a projekt könyvtárában
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Port már használatban:**
```bash
# A Spring Boot alapértelmezés szerint a 8080-as portot használja
# Változtatás az application.properties fájlban:
server.port=8081
```

### Többnyelvű támogatás
- Dokumentáció 45+ nyelven automatikus fordítással elérhető
- Fordítások a `translations/` mappában
- Fordítást GitHub Actions munkafolyamat kezeli

### Tanulási útvonal
1. Kezdd a [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md) fejezettel
2. Kövesd a fejezeteket sorrendben (01 → 05)
3. Végezd el a gyakorlati példákat minden fejezetben
4. Nézd meg a minta projekteket a 4. fejezetben
5. Tanuld meg a felelős AI gyakorlatokat az 5. fejezetben

### Fejlesztői konténer
A `.devcontainer/devcontainer.json` a következőket konfigurálja:
- Java 21 fejlesztői környezet
- Maven előtelepítve
- VS Code Java bővítmények
- Spring Boot eszközök
- GitHub Copilot integráció
- Docker-in-Docker támogatás
- Azure CLI

### Teljesítmény szempontok
- Az Azure AI Foundry telepítések percenkénti token/kérés kvótával rendelkeznek
- Használj megfelelő batch méreteket beágyazásokhoz
- Fontold meg a cache használatát ismételt API hívásokhoz
- Monitorozd a token használatot költség-optimalizálás céljából

### Biztonsági megjegyzések
- Soha ne vállalj be `.env` fájlokat (már .gitignore-ban vannak)
- Részesítsd előnyben a kulcs nélküli hitelesítést (Microsoft Entra ID) az API kulcsok helyett
- Használj kezelt identitást Azure-ban; helyi fejlesztéshez `az login`
- Kövesd a felelős AI irányelveket az 5. fejezetben

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Jogi nyilatkozat**:
Ez a dokumentum az AI fordítási szolgáltatás, a [Co-op Translator](https://github.com/Azure/co-op-translator) segítségével készült. Bár az pontosságra törekszünk, kérjük, vegye figyelembe, hogy az automatikus fordítások hibákat vagy pontatlanságokat tartalmazhatnak. Az eredeti dokumentum az anyanyelvén tekintendő hiteles forrásnak. Fontos információk esetén professzionális emberi fordítást javasolunk. Nem vállalunk felelősséget semmilyen félreértésért vagy téves értelmezésért, amely ebből a fordításból ered.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->