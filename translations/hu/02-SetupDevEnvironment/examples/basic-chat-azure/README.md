# Alap csevegés Azure AI Foundry-val - Végponttól végpontig példa

Ez a példa egy egyszerű Spring Boot alkalmazás, amely az **Azure AI Foundry** modellhez kapcsolódik **kulcs nélküli hitelesítéssel** (Microsoft Entra ID), és teszteli a beállításokat. A Spring AI `ChatClient`-jét használja.

## Tartalomjegyzék

- [Előfeltételek](#előfeltételek)
- [Gyors kezdés](#gyors-kezdés)
- [Hogyan működik a hitelesítés](#hogyan-működik-a-hitelesítés)
- [Az alkalmazás futtatása](#az-alkalmazás-futtatása)
  - [Maven használata](#maven-használata)
  - [VS Code használata](#vs-code-használata)
  - [Várt kimenet](#várt-kimenet)
- [Konfigurációs hivatkozás](#konfigurációs-hivatkozás)
  - [Környezeti változók](#környezeti-változók)
  - [Spring konfiguráció](#spring-konfiguráció)
- [Hibaelhárítás](#hibaelhárítás)
  - [Gyakori hibák](#gyakori-hibák)
  - [Hibakeresési mód](#hibakeresési-mód)
- [Következő lépések](#következő-lépések)
- [Források](#források)

## Előfeltételek

A példa futtatása előtt győződj meg róla, hogy rendelkezel:

- Egy Azure AI Foundry erőforrással, amelyen `gpt-4o-mini` telepítés fut — állítsd elő az `azd up` parancsal vagy kézzel a [Azure AI Foundry beállítási útmutató](../../getting-started-azure-openai.md) alapján
- A **Cognitive Services OpenAI User** szerepkörrel az erőforráson (ezt a Bicep sablonok automatikusan hozzárendelik)
- A [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli) telepítve és bejelentkezve az `az login` parancs segítségével
- Java 21+ és Maven 3.9+ verzióval

> **API-kulcs nem szükséges** — a hitelesítés kulcs nélküli, Microsoft Entra ID-n keresztül történik.

## Gyors kezdés

```bash
# 1. Navigálj a projekthez
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Jelentkezz be, hogy a kulcs nélküli hitelesítés tokenhez jusson
az login

# 3. Állítsd be a végpontot
#    - Ha futtattad az `azd up` parancsot, a .env fájlt neked írták (ezt kihagyhatod).
#    - Ellenkező esetben másold a sablont és állítsd be az AZURE_OPENAI_ENDPOINT értékét:
cp .env.example .env

# 4. Futtasd az alkalmazást
mvn spring-boot:run
```

## Hogyan működik a hitelesítés

Ez a példa a **Microsoft Entra ID** segítségével hitelesít — nincs API-kulcs.

Ha csak a `spring.ai.azure.openai.endpoint` van megadva (és nincs api-kulcs), a Spring AI az Azure OpenAI kliensét a [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential) használatával építi fel. Ez a hitelesítő automatikusan megtalálja a tokenedet a helyi `az login` munkamenetből, vagy egy kezelt identitásból, ha Azure-ban futsz — így ugyanaz a kód mindkét helyen működik módosítás nélkül.

## Az alkalmazás futtatása

### Maven használata

```bash
mvn spring-boot:run
```

### VS Code használata

1. Nyisd meg a projektet VS Code-ban
2. Nyomd meg az `F5`-öt vagy használd a "Run and Debug" panelt
3. Válaszd a "Spring Boot-BasicChatApplication" konfigurációt

> **Megjegyzés**: A VS Code konfiguráció automatikusan betölti a `.env` fájlodat

### Várt kimenet

```
Starting Basic Chat with Azure OpenAI...
Environment variables loaded successfully
Connecting to Azure OpenAI...
Sending prompt: What is AI in a short sentence? Max 100 words.

AI Response:
================
AI, or Artificial Intelligence, is the simulation of human intelligence in machines programmed to think and learn like humans.
================

Success! Azure OpenAI connection is working correctly.
```

## Konfigurációs hivatkozás

### Környezeti változók

| Változó | Leírás | Kötelező | Példa |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | Foundry (Azure OpenAI) végpont URL | Igen | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Csevegő modell telepítésének neve | Nem | `gpt-4o-mini` (alapértelmezett) |

> Nincs **API-kulcs** változó — a hitelesítés kulcs nélküli (Microsoft Entra ID az `az login`-en keresztül).

### Spring konfiguráció

Az `application.yml` fájl a következőket állítja be:
- **Végpont**: `${AZURE_OPENAI_ENDPOINT}` - Környezeti változóból
- **Telepítés**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - Környezeti változóból, alapértelmezéssel
- **Hitelesítés**: kulcs nélküli — nincs `api-key` megadva, ezért a Spring AI a `DefaultAzureCredential`-t használja
- **Hőmérséklet**: `0.7` - Kreativitás szabályozása (0.0 = determinisztikus, 1.0 = kreatív)
- **Maximális tokenek**: `500` - A válasz maximális hossza

## Hibaelhárítás

### Gyakori hibák

<details>
<summary><strong>Hiba: 401 / "PermissionDenied" / token hibák</strong></summary>

- Futtasd az `az login` parancsot — kulcs nélküli hitelesítéshez aktív bejelentkezés szükséges a token megszerzéséhez
- Ellenőrizd, hogy a fiókod rendelkezik a **Cognitive Services OpenAI User** szerepkörrel az erőforráson
- Ha most rendelkezted hozzá a szerepkört, várj egy percet a terjedéséhez
- Győződj meg arról, hogy a megfelelő bérlőnél/fióknál vagy (`az account show`)
</details>

<details>
<summary><strong>Hiba: "A végpont nem érvényes" / kapcsolat hibák</strong></summary>

- Győződj meg róla, hogy az `AZURE_OPENAI_ENDPOINT` a teljes alap URL (pl. `https://your-resource.openai.azure.com/`)
- Ellenőrizd a perjeleket a végén megfelelően
- Ellenőrizd, hogy a végpont egyezik a kiépített erőforrással (`azd env get-values`)
</details>

<details>
<summary><strong>Hiba: "Nem található a telepítés"</strong></summary>

- Ellenőrizd, hogy az `AZURE_OPENAI_DEPLOYMENT` megegyezik egy telepítés nevével az Azure-ban
- Győződj meg róla, hogy a modell sikeresen telepítve és aktív
- Az alapértelmezett telepítés neve `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: Nem töltődnek be a környezeti változók</strong></summary>

- Győződj meg róla, hogy a `.env` fájl a projekt gyökérkönyvtárában van (ugyanolyan szinten, mint a `pom.xml`)
- Próbáld meg futtatni a `mvn spring-boot:run` parancsot a VS Code integrált termináljában
- Ellenőrizd, hogy a VS Code Java bővítmény megfelelően telepítve van
</details>

### Hibakeresési mód

A részletes naplózás engedélyezéséhez távolítsd el ezeknek a soroknak a kommentelését az `application.yml` fájlban:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Következő lépések

**Beállítás kész!** Folytasd a tanulási utadat:

[3. fejezet: Alapvető generatív AI technikák](../../../03-CoreGenerativeAITechniques/README.md)

## Források

- [Spring AI Azure OpenAI dokumentáció](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Kulcs nélküli hitelesítés Microsoft Entra ID-val](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry portál](https://ai.azure.com/)
- [Azure AI Foundry dokumentáció](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Jogi nyilatkozat**:
Ez a dokumentum az AI fordítási szolgáltatás, a [Co-op Translator](https://github.com/Azure/co-op-translator) segítségével készült. Bár az pontosságra törekszünk, kérjük, vegye figyelembe, hogy az automatikus fordítások hibákat vagy pontatlanságokat tartalmazhatnak. Az eredeti dokumentum az anyanyelvén tekintendő hiteles forrásnak. Fontos információk esetén professzionális emberi fordítást javasolunk. Nem vállalunk felelősséget semmilyen félreértésért vagy téves értelmezésért, amely ebből a fordításból ered.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->