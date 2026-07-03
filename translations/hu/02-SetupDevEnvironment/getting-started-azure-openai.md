# Azure AI Foundry fejlesztői környezet beállítása

> Ez az útmutató az ebben a kurzusban használt Java AI alkalmazásokhoz állítja be az **Azure AI Foundry** modelleket, **kulcs nélküli** hitelesítéssel (Microsoft Entra ID) — nincs API kulcs kezelés. Új vagy az eszközzel? Kezdd a [fejlesztői környezet útmutatónál](./README.md).

Ez az útmutató az ebben a kurzusban használt Java AI alkalmazásokhoz állítja be az **Azure AI Foundry** modelleket. Két lehetőséged van:

- **A lehetőség — `azd` + Bicep használata a kiépítéshez (ajánlott):** egy parancs telepíti a Foundry fiókot és a modelleket kódként. Nincs portálon kattintás.
- **B lehetőség — erőforrások kézi létrehozása** az Azure AI Foundry portálon.

Mindkét út **kulcs nélküli hitelesítést** használ (Microsoft Entra ID) — nincsenek API kulcsok, amelyeket másolni vagy szivárogtatni kéne.

## Tartalomjegyzék

- [Mi jön létre](#mi-jön-létre)
- [Előfeltételek](#előfeltételek)
- [A lehetőség: Kiépítés az azd + Bicep segítségével (ajánlott)](#option-a-provision-with-azd--bicep-recommended)
- [B lehetőség: Erőforrások kézi létrehozása](#b-lehetőség-erőforrások-kézi-létrehozása)
- [Környezet beállítása](#környezet-beállítása)
- [Teszteld a beállításodat](#teszteld-a-beállításodat)
- [Mi következik?](#mi-következik)
- [Erőforrások](#erőforrások)
- [További erőforrások](#további-erőforrások)

## Mi jön létre

Az [`infra/`](../../../02-SetupDevEnvironment/infra) mappában lévő Bicep sablonok az alábbiakat hozzák létre:

- Egy **Azure AI Foundry** fiókot (`Microsoft.CognitiveServices/accounts`, típus `AIServices`) egy projekttel
- Egy **beszélgetés** deploy-t — `gpt-4o-mini`
- Egy **embedding** deploy-t — `text-embedding-3-small` (a későbbi fejezetekben használva)
- Egy **kulcs nélküli szerepkör-hozzárendelést** (`Cognitive Services OpenAI User`), hogy az `az login` segítségével jelentkezz be kulcskezelés helyett

## Előfeltételek

- Egy [Azure előfizetés](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) és [Maven 3.9+](https://maven.apache.org/download.cgi)

## A lehetőség: Kiépítés az azd + Bicep segítségével (ajánlott)

A `02-SetupDevEnvironment` mappából:

```bash
cd 02-SetupDevEnvironment

# Bejelentkezés (mindkét eszköz)
azd auth login
az login

# A Foundry fiók és a modell telepítések előkészítése
azd up
```

Az `azd` kérni fog egy **környezeti nevet** (például `genai-java`) és egy **régiót**. Válassz olyan régiót, ahol elérhető a `gpt-4o-mini` és a `text-embedding-3-small` — például `eastus2` vagy `swedencentral`.

A kiépítés befejeztével az azd:

1. Telepíti az összeset, ami szerepel az [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep) fájlban.
2. Lefuttat egy postprovision hook-ot, amely létrehozza az [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) fájlt endpoint és deployment nevek tárolásával (nincs titok).

> **Tipp:** Bármikor futtasd újra az `azd up` parancsot a változtatások érvényesítéséhez. Az `azd down` megöli az összes erőforrást és megszünteti a költséget.

A generált beállítások megtekintéséhez:

```bash
azd env get-values
```

Most ugorj a [Teszteld a beállításodat](#teszteld-a-beállításodat) részhez.

## B lehetőség: Erőforrások kézi létrehozása

Inkább portált használnál? Kézzel hozd létre az erőforrásokat:

1. Lépj be az [Azure AI Foundry portálra](https://ai.azure.com/).
2. **Hozz létre egy projektet** (ez egy AI Foundry erőforrást is létrehoz). Adj neki nevet, például `GenAIJava`.
3. A projektben menj a **Modellek + végpontok** → **Modell telepítése** → **Alapmodell telepítése**.
4. Telepítsd a **gpt-4o-mini**-t (telepítés neve `gpt-4o-mini`). Ismételd meg a **text-embedding-3-small**-lal, ha az embedding példákat szeretnéd.
5. Az **Áttekintés** fülön másold ki a **végpontot** (például `https://<resource>.openai.azure.com/`).
6. Adj magadnak kulcs nélküli hozzáférést: a erőforrásnál nyisd meg a **Hozzáférés vezérlése (IAM)** → **Szerepkör hozzárendelés hozzáadása** → rendeld a **Cognitive Services OpenAI User** szerepkört a fiókodhoz.

> **Még gondjaid vannak?** Nézd meg az [Azure AI Foundry dokumentációját](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Környezet beállítása

**Ha az A lehetőséget használtad (`azd up`)**, a beállítási fájl már megvan — nincs teendő. Ugorj a [Teszteld a beállításodat](#teszteld-a-beállításodat) pontra.

**Ha a B lehetőséget (kézi) választottad**, magadnak kell létrehoznod az example `.env` fájlt:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

A `.env`-et szerkeszd a végpontoddal (kulcs nélkül — hitelesítés kulcs nélkül történik):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Biztonsági megjegyzés:** Nincs API kulcs, amit tárolni kéne. A Microsoft Entra ID-n keresztül hitelesítesz az `az login` parancssal (lokálisan) vagy felügyelt identitással (az Azure-ban). A `.env` fájl csak titoktalan beállításokat tartalmaz és a `.gitignore` már figyel rá.

## Teszteld a beállításodat

Győződj meg róla, hogy be vagy jelentkezve, hogy a kulcs nélküli hitelesítés tudjon tokent szerezni, majd futtasd a példát:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # ha még nem jelentkeztél be
mvn clean spring-boot:run
```

Látnod kell a `gpt-4o-mini` modell válaszát!

> **VS Code használóknak:** Nyomj `F5`-öt a futtatáshoz. Az alkalmazás automatikusan betölti a `.env`-ed.

> **Teljes példa:** Lásd a [Basic Chat with Azure AI Foundry példát](./examples/basic-chat-azure/README.md) részletes ismertetésért és hibakeresésért.

## Mi következik?

**A beállítás készen van!** Most már rendelkezel:
- Azure AI Foundry-val `gpt-4o-mini` és `text-embedding-3-small` telepítve
- Kulcs nélküli hitelesítéssel (Microsoft Entra ID) — nincs kulcskezelés
- Egy helyi `.env` fájl az endpointtal és a deployment nevekkel
- Egy működő Java fejlesztői környezet

**Folytasd a** [3. fejezettel: Alapvető generatív AI technikák](../03-CoreGenerativeAITechniques/README.md), hogy elkezdhesd AI alkalmazások építését!

## Erőforrások

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Kulcs nélküli hitelesítés Microsoft Entra ID-vel](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry Dokumentáció](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI Dokumentáció](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## További erőforrások

- [VS Code letöltése](https://code.visualstudio.com/Download)
- [Docker Desktop beszerzése](https://www.docker.com/products/docker-desktop)
- [Fejlesztői konténer konfigurációja](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Jogi nyilatkozat**:
Ez a dokumentum az AI fordítási szolgáltatás, a [Co-op Translator](https://github.com/Azure/co-op-translator) segítségével készült. Bár az pontosságra törekszünk, kérjük, vegye figyelembe, hogy az automatikus fordítások hibákat vagy pontatlanságokat tartalmazhatnak. Az eredeti dokumentum az anyanyelvén tekintendő hiteles forrásnak. Fontos információk esetén professzionális emberi fordítást javasolunk. Nem vállalunk felelősséget semmilyen félreértésért vagy téves értelmezésért, amely ebből a fordításból ered.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->