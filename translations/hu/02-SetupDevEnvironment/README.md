# Generatív AI Java fejlesztői környezet beállítása

> **Gyors indulás:** Szerezz be AI modelljeidet az **Azure AI Foundry**-ban kódként Bicep + `azd` segítségével pár perc alatt — lásd a [Azure AI Foundry beállítási útmutatót](getting-started-azure-openai.md). Hitelesítés **kulcs nélküli** (Microsoft Entra ID), így nincs API kulcs kezelése.

## Amit megtanulsz

- Java fejlesztői környezet beállítása AI alkalmazásokhoz  
- Válaszd ki és konfiguráld a preferált fejlesztői környezeted (cloud-first Codespaces-szal, helyi fejlesztői konténer vagy teljes helyi beállítás)  
- Teszteld a beállításod, hogy csatlakozol egy Azure AI Foundry modellhez  

## Tartalomjegyzék

- [Amit megtanulsz](#amit-megtanulsz)
- [Bevezetés](#bevezetés)
- [1. lépés: Fejlesztői környezet beállítása](#1-lépés-fejlesztői-környezet-beállítása)
  - [A lehetőség: GitHub Codespaces (ajánlott)](#a-lehetőség-github-codespaces-ajánlott)
  - [B lehetőség: Helyi fejlesztői konténer](#b-lehetőség-helyi-fejlesztői-konténer)
  - [C lehetőség: Használd meglévő helyi telepítésed](#c-lehetőség-használd-meglévő-helyi-telepítésed)
- [2. lépés: Azure AI Foundry biztosítása](#2-lépés-azure-ai-foundry-biztosítása)
- [3. lépés: Beállítás tesztelése](#3-lépés-beállítás-tesztelése)
- [Hibaelhárítás](#hibaelhárítás)
- [Összefoglaló](#összefoglaló)
- [Következő lépések](#következő-lépések)

## Bevezetés

Ez a fejezet végigvezet a fejlesztői környezet beállításán. A kurzus során az **Azure AI Foundry**-t használjuk modellekhez. A modelleket kódként biztosítod Bicep és az Azure Developer CLI (`azd`) segítségével, majd kulcs nélküli hitelesítéssel (Microsoft Entra ID) csatlakozol — nincs API kulcsmásolás vagy -szivárgás.

**Nincs szükség helyi beállításra!** Használhatod a GitHub Codespaces-t, ami teljes fejlesztői környezetet biztosít a böngésződben, és onnan biztosíthatod a Foundryt.

Azért használjuk az **Azure AI Foundry**-t, mert:  
- **Kódként biztosítható** — egy `azd up` telepíti a fiókot és a modell telepítéseket  
- **Kulcs nélküli** — az Azure bejelentkezéseddel vagy egy kezelt identitással hitelesít  
- **Éles használatra alkalmas** — ugyanaz a kód fut lokálisan és Azure-ban is  
- **Rugalmas** — modellek cseréje telepítésnév megváltoztatásával, nem a kód módosításával  

> **Megjegyzés**: Az Azure AI Foundry telepítések tokenenként (fizess amennyit használsz) kerülnek kiszámlázásra. A részletekért lásd az [Azure AI Foundry beállítási útmutatót](getting-started-azure-openai.md).

  
## 1. lépés: Fejlesztői környezet beállítása

<a name="quick-start-cloud"></a>

Egy előre konfigurált fejlesztői konténert hoztunk létre, hogy minimalizáljuk a beállítás idejét, és biztosítsuk az összes szükséges eszközt ehhez a Generatív AI Java kurzushoz. Válaszd ki a neked megfelelő fejlesztési megközelítést:

### Fejlesztői környezet beállítási lehetőségek:

#### A lehetőség: GitHub Codespaces (ajánlott)

**Kezdj el kódolni 2 perc alatt - nincs helyi beállítás szükséges!**

1. Forkold ezt a repót a GitHub fiókodba  
   > **Megjegyzés**: Ha szeretnéd szerkeszteni az alap konfigurációt, nézd meg a [Dev Container konfigurációt](../../../.devcontainer/devcontainer.json)  
2. Kattints a **Code** → **Codespaces** fülre → **...** → **New with options...**  
3. Használd az alapbeállításokat – ez kiválasztja a **Fejlesztői konténer konfigurációt**: a kurzusra szabott **Generatív AI Java fejlesztői környezet** devcontainer-t  
4. Kattints a **Create codespace** gombra  
5. Várj kb. 2 percet, amíg a környezet elkészül  
6. Folytasd a [2. lépéssel: Azure AI Foundry biztosítása](#2-lépés-azure-ai-foundry-biztosítása)

<img src="../../../translated_images/hu/codespaces.9945ded8ceb431a5.webp" alt="Pillanatkép: Codespaces almenü" width="50%">

<img src="../../../translated_images/hu/image.833552b62eee7766.webp" alt="Pillanatkép: Új opciókkal" width="50%">

<img src="../../../translated_images/hu/codespaces-create.b44a36f728660ab7.webp" alt="Pillanatkép: Codespace létrehozásának opciói" width="50%">


> **Codespaces előnyei**:  
> - Nincs szükség helyi telepítésre  
> - Bármilyen böngészőt futtató eszközön működik  
> - Előre konfigurált az összes eszközzel és függőséggel  
> - Személyes fiókoknak havi 60 óráig ingyenes  
> - Minden tanulónak egységes fejlesztői környezetet biztosít  

#### B lehetőség: Helyi fejlesztői konténer

**Fejlesztőknek, akik helyi Dockert preferálnak**

1. Forkold és klónozd a repót a gépedre  
   > **Megjegyzés**: Ha szeretnéd szerkeszteni az alap konfigurációt, nézd meg a [Dev Container konfigurációt](../../../.devcontainer/devcontainer.json)  
2. Telepítsd a [Docker Desktop](https://www.docker.com/products/docker-desktop/) és [VS Code](https://code.visualstudio.com/) programokat  
3. Telepítsd a [Dev Containers kiterjesztést](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) a VS Code-ban  
4. Nyisd meg a repó mappáját VS Code-ban  
5. Amikor megjelenik, kattints a **Reopen in Container** gombra (vagy `Ctrl+Shift+P` → „Dev Containers: Reopen in Container”)  
6. Várj, amíg a konténer elkészül és elindul  
7. Folytasd a [2. lépéssel: Azure AI Foundry biztosítása](#2-lépés-azure-ai-foundry-biztosítása)

<img src="../../../translated_images/hu/devcontainer.21126c9d6de64494.webp" alt="Pillanatkép: Fejlesztői konténer beállítása" width="50%">

<img src="../../../translated_images/hu/image-3.bf93d533bbc84268.webp" alt="Pillanatkép: Fejlesztői konténer felépült" width="50%">

#### C lehetőség: Használd meglévő helyi telepítésed

**Fejlesztőknek meglévő Java környezettel**

Előfeltételek:  
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html)  
- [Maven 3.9+](https://maven.apache.org/download.cgi)  
- [VS Code](https://code.visualstudio.com) vagy az általad preferált IDE  

Lépések:  
1. Klónozd a repót a gépedre  
2. Nyisd meg a projektet az IDE-dben  
3. Folytasd a [2. lépéssel: Azure AI Foundry biztosítása](#2-lépés-azure-ai-foundry-biztosítása)

> **Pro Tipp**: Ha gyenge géped van, de helyi VS Code-ot szeretnél, használd a GitHub Codespaces-t! Csatlakoztasd a helyi VS Code-odat egy cloud-hostolt Codespace-hez a legjobb élményért.

<img src="../../../translated_images/hu/image-2.fc0da29a6e4d2aff.webp" alt="Pillanatkép: létrehozott helyi devcontainer példány" width="50%">


## 2. lépés: Azure AI Foundry biztosítása

Telepítsd a kurzus AI modelljeit az Azure AI Foundry-ba kódként. A repó gyökérkönyvtárából:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```
  
Az `azd` megkérdezi a környezet nevét és régióját, biztosít egy Azure AI Foundry fiókot a `gpt-4o-mini` és `text-embedding-3-small` telepítésekkel, és az endpointot beírja a példa `.env` fájljába — mindezt **kulcs nélküli** hitelesítéssel (nincs API kulcs).

> **Teljes útmutató:** Lásd az [Azure AI Foundry beállítási útmutatót](getting-started-azure-openai.md) a feltételekhez, kézi (portálos) alternatívához, régió kiválasztáshoz és költség/takarítási megjegyzésekhez.

## 3. lépés: Beállítás tesztelése

Miután a Foundry modelljeid működnek, teszteld a kapcsolatot a példaprogrammal a [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) mappában.

1. Nyisd meg a terminált a fejlesztői környezetedben.  
2. Navigálj a példába:  
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
  
3. Győződj meg róla, hogy be vagy jelentkezve (a kulcs nélküli hitelesítés token szükséges):  
   ```bash
   az login
   ```
  
   > Ha `azd up`-ot futtattál, a `.env` fájl már tartalmazza az endpointot számodra.  
4. Futtasd az alkalmazást:  
   ```bash
   mvn clean spring-boot:run
   ```
  
Látnod kell választ a `gpt-4o-mini` modelltől.

### A példa kód megértése

A `examples/basic-chat-azure` mappában lévő példa egy Spring Boot alkalmazás, amely a **Spring AI**-t használja az Azure AI Foundry kulcs nélküli hitelesítésű elérésére.

**Mit csinál a kód:**  
- **Csatlakozik** az Azure AI Foundry-hoz az Azure bejelentkezéseddel (Microsoft Entra ID) — nincs API kulcs  
- **Küld** egy promptot a `gpt-4o-mini` modellnek  
- **Fogadja** és megjeleníti a mesterséges intelligencia válaszát  
- **Ellenőrzi**, hogy a beállítás megfelelően működik  

**Fő függőség** (`pom.xml`-ben):  
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```
  
**Konfiguráció** (`application.yml`):  
```yaml
spring:
  ai:
    azure:
      openai:
        # Endpoint only - no api-key. Spring AI uses DefaultAzureCredential (keyless).
        endpoint: ${AZURE_OPENAI_ENDPOINT}
        chat:
          options:
            deployment-name: ${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}
```
  
## Összefoglaló

Szuper! Most minden be van állítva:

- Bicep + `azd` segítségével biztosított Azure AI Foundry modellek kódként  
- Elindult a Java fejlesztői környezeted (akár Codespaces, fejlesztői konténer, vagy helyi)  
- Kapcsolódtál az Azure AI Foundry-hoz kulcs nélküli hitelesítéssel (Microsoft Entra ID) — nincs API kulcs  
- Letesztelted egy egyszerű példával, amely kommunikál a modelleddel  

## Következő lépések

[3. fejezet: Alapvető generatív AI technikák](../03-CoreGenerativeAITechniques/README.md)

## Hibaelhárítás

Problémák adódtak? Itt a leggyakoribb hibák és megoldások:

- **Hitelesítés sikertelen (401/403)?**  
  - Futtasd az `az login` parancsot — a hitelesítés kulcs nélküli, tehát be kell jelentkezned  
  - Ellenőrizd, hogy a fiókod rendelkezik-e a **Cognitive Services OpenAI User** szerepkörrel az erőforráson  
  - Ha most biztosítottad, várj egy percet, hogy a szerepkör hozzárendelés érvénybe lépjen  

- **Nem található Maven?**  
  - Dev konténerekben/Codespaces-ben előre telepítve van a Maven  
  - Helyi beállításnál biztosítsd, hogy Java 21+ és Maven 3.9+ telepítve legyen  
  - Próbáld ki a `mvn --version` parancsot a telepítés ellenőrzéséhez  

- **`azd` nem található vagy a biztosítás sikertelen?**  
  - Telepítsd az [Azure Developer CLI-t](https://aka.ms/azure-dev/install), majd futtasd az `azd auth login` parancsot  
  - Válassz régiót, ahol elérhető a `gpt-4o-mini` (pl. `eastus2`)  
  - Lásd az [Azure AI Foundry beállítási útmutatót](getting-started-azure-openai.md) további részletekért  

- **Dev konténer nem indul?**  
  - Győződj meg róla, hogy a Docker Desktop fut (helyi fejlesztéshez)  
  - Próbáld újraépíteni a konténert: `Ctrl+Shift+P` → „Dev Containers: Rebuild Container”  

- **Alkalmazás fordítási hibák?**  
  - Legyél a helyes könyvtárban: `02-SetupDevEnvironment/examples/basic-chat-azure`  
  - Próbáld meg tisztítani és újra fordítani: `mvn clean compile`  

> **Segítségre van szükséged?**: Ha még mindig gond van, nyiss egy issue-t a repóban, és segítünk!

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Jogi nyilatkozat**:
Ez a dokumentum az AI fordítási szolgáltatás, a [Co-op Translator](https://github.com/Azure/co-op-translator) segítségével készült. Bár az pontosságra törekszünk, kérjük, vegye figyelembe, hogy az automatikus fordítások hibákat vagy pontatlanságokat tartalmazhatnak. Az eredeti dokumentum az anyanyelvén tekintendő hiteles forrásnak. Fontos információk esetén professzionális emberi fordítást javasolunk. Nem vállalunk felelősséget semmilyen félreértésért vagy téves értelmezésért, amely ebből a fordításból ered.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->