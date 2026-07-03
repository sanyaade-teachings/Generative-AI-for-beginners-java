# Postavljanje razvojnog okruženja za Generativnu AI za Javu

> **Brzi početak:** Nabavite svoje AI modele na **Azure AI Foundry** kao kod s Bicepom + `azd` za nekoliko minuta — pogledajte [Vodič za postavljanje Azure AI Foundry](getting-started-azure-openai.md). Autentifikacija je **bez ključa** (Microsoft Entra ID), pa nema API ključeva za upravljanje.

## Što ćete naučiti

- Postaviti razvojno okruženje za AI aplikacije u Javi
- Odabrati i konfigurirati željeno razvojno okruženje (cloud-first s Codespaces, lokalni razvojni kontejner ili potpuna lokalna instalacija)
- Testirati postavke povezivanjem s modelom Azure AI Foundry

## Sadržaj

- [Što ćete naučiti](#što-ćete-naučiti)
- [Uvod](#uvod)
- [Korak 1: Postavite razvojno okruženje](#korak-1-postavite-razvojno-okruženje)
  - [Opcija A: GitHub Codespaces (Preporučeno)](#opcija-a-github-codespaces-preporučeno)
  - [Opcija B: Lokalni razvojni kontejner](#opcija-b-lokalni-razvojni-kontejner)
  - [Opcija C: Koristite postojeću lokalnu instalaciju](#opcija-c-koristite-postojeću-lokalnu-instalaciju)
- [Korak 2: Nabavite Azure AI Foundry](#korak-2-nabavite-azure-ai-foundry)
- [Korak 3: Testirajte postavke](#korak-3-testirajte-postavke)
- [Rješavanje problema](#rješavanje-problema)
- [Sažetak](#sažetak)
- [Naredni koraci](#naredni-koraci)

## Uvod

Ovo poglavlje vodi vas kroz postavljanje razvojnog okruženja. Tijekom ovog tečaja koristit ćemo **Azure AI Foundry** za modele. Modele nabavljate kao kod koristeći Bicep i Azure Developer CLI (`azd`), a zatim se povezujete s **autentifikacijom bez ključa** (Microsoft Entra ID) — bez potrebe za kopiranjem ili upravljanjem API ključevima.

**Nije potrebna lokalna instalacija!** Možete koristiti GitHub Codespaces koji pruža puno razvojno okruženje u vašem pregledniku i odatle nabaviti Foundry.

Koristimo **Azure AI Foundry** za ovaj tečaj jer:
- **Dobavlja se kao kod** — jedan `azd up` implementira račun i modele
- **Bez ključeva** — autentificirajte se pomoću Azure prijave ili upravljanog identiteta
- **Spreman za produkciju** — isti kod radi lokalno i na Azureu
- **Fleksibilan** — mijenjajte modele promjenom imena implementacije, ne koda

> **Napomena**: Azure AI Foundry implementacije se naplaćuju po tokenu (plaćanje prema potrošnji). Pogledajte [vodič za postavljanje Azure AI Foundry](getting-started-azure-openai.md) za informacije o nabavi, regiji i troškovima.


## Korak 1: Postavite razvojno okruženje

<a name="quick-start-cloud"></a>

Napravili smo predkonfigurirani razvojni kontejner da bismo smanjili vrijeme postavljanja i osigurali da imate sve potrebne alate za ovaj tečaj Generativne AI za Javu. Odaberite željeni način razvoja:

### Opcije za postavljanje okruženja:

#### Opcija A: GitHub Codespaces (Preporučeno)

**Započnite razvoj za 2 minute - nije potrebna lokalna instalacija!**

1. Forkajte ovaj repozitorij na svoj GitHub račun
   > **Napomena**: Ako želite urediti osnovnu konfiguraciju, pogledajte [Dev Container konfiguraciju](../../../.devcontainer/devcontainer.json)
2. Kliknite **Code** → kartica **Codespaces** → **...** → **New with options...**
3. Koristite zadane postavke – odabrat će se **Dev container konfiguracija**: **Generative AI Java Development Environment** prilagođeni devcontainer za ovaj tečaj
4. Kliknite **Create codespace**
5. Pričekajte ~2 minute da se okruženje pripremi
6. Nastavite na [Korak 2: Nabavite Azure AI Foundry](#korak-2-nabavite-azure-ai-foundry)

<img src="../../../translated_images/hr/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: Codespaces submenu" width="50%">

<img src="../../../translated_images/hr/image.833552b62eee7766.webp" alt="Screenshot: New with options" width="50%">

<img src="../../../translated_images/hr/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: Create codespace options" width="50%">


> **Prednosti Codespaces**:
> - Nije potrebna lokalna instalacija
> - Radi na bilo kojem uređaju s preglednikom
> - Predkonfiguriran sa svim alatima i ovisnostima
> - Besplatnih 60 sati mjesečno za osobne račune
> - Konzistentno okruženje za sve polaznike

#### Opcija B: Lokalni razvojni kontejner

**Za developere koji preferiraju lokalni razvoj s Dockerom**

1. Forkajte i klonirajte ovaj repozitorij na svoje lokalno računalo
   > **Napomena**: Ako želite urediti osnovnu konfiguraciju, pogledajte [Dev Container konfiguraciju](../../../.devcontainer/devcontainer.json)
2. Instalirajte [Docker Desktop](https://www.docker.com/products/docker-desktop/) i [VS Code](https://code.visualstudio.com/)
3. Instalirajte [Dev Containers ekstenziju](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) u VS Code-u
4. Otvorite mapu repozitorija u VS Code-u
5. Kada se pojavi upit, kliknite **Reopen in Container** (ili koristite `Ctrl+Shift+P` → "Dev Containers: Reopen in Container")
6. Pričekajte da se kontejner izgradi i pokrene
7. Nastavite na [Korak 2: Nabavite Azure AI Foundry](#korak-2-nabavite-azure-ai-foundry)

<img src="../../../translated_images/hr/devcontainer.21126c9d6de64494.webp" alt="Screenshot: Dev container setup" width="50%">

<img src="../../../translated_images/hr/image-3.bf93d533bbc84268.webp" alt="Screenshot: Dev container build complete" width="50%">

#### Opcija C: Koristite postojeću lokalnu instalaciju

**Za developere s postojećim Java okruženjima**

Preduvjeti:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html) 
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) ili vaš omiljeni IDE

Koraci:
1. Klonirajte ovaj repozitorij na svoje lokalno računalo
2. Otvorite projekt u svom IDE-u
3. Nastavite na [Korak 2: Nabavite Azure AI Foundry](#korak-2-nabavite-azure-ai-foundry)

> **Savjet**: Ako imate malo snažniji stroj ali želite koristiti VS Code lokalno, upotrijebite GitHub Codespaces! Svoj lokalni VS Code možete povezati s Codespaceom u oblaku za najbolje od oba svijeta.

<img src="../../../translated_images/hr/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: created local devcontainer instance" width="50%">


## Korak 2: Nabavite Azure AI Foundry

Implementirajte AI modele za tečaj u Azure AI Foundry kao kod. Iz korijena repozitorija:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```

`azd` traži ime okruženja i regiju, nabavlja Azure AI Foundry račun s implementacijama `gpt-4o-mini` i `text-embedding-3-small`, i zapisuje endpoint u `.env` datoteku primjera — sve s **autentifikacijom bez ključa** (bez API ključeva).

> **Potpuni vodič:** Pogledajte [Vodič za postavljanje Azure AI Foundry](getting-started-azure-openai.md) za preduvjete, alternativu ručnog unosa (portal), preporuke o regijama i napomene o troškovima i čišćenju.

## Korak 3: Testirajte postavke

Kad su vaši Foundry modeli nabavljeni, testirajte vezu s primjerom aplikacije u [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure).

1. Otvorite terminal u svom razvojnom okruženju.
2. Idite do primjera:
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
3. Provjerite jeste li prijavljeni (autentifikacija bez ključa treba token):
   ```bash
   az login
   ```
   > Ako ste pokrenuli `azd up`, `.env` datoteka s vašim endpointom već je zapisane za vas.
4. Pokrenite aplikaciju:
   ```bash
   mvn clean spring-boot:run
   ```

Trebali biste vidjeti odgovor iz modela `gpt-4o-mini`.

### Razumijevanje primjera koda

Primjer pod `examples/basic-chat-azure` je Spring Boot aplikacija koja koristi **Spring AI** za povezivanje s Azure AI Foundry uz autentifikaciju bez ključa.

**Što ovaj kod radi:**
- **Povezuje se** s Azure AI Foundry koristeći vašu Azure prijavu (Microsoft Entra ID) — bez API ključa
- **Šalje** upit modelu `gpt-4o-mini`
- **Prima** i prikazuje odgovor AI-ja
- **Provjerava** da li je postavka ispravna

**Ključna ovisnost** (u `pom.xml`):
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```

**Konfiguracija** (`application.yml`):
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

## Sažetak

Odlično! Sada imate sve postavljeno:

- Nabavili Azure AI Foundry modele kao kod s Bicep + `azd`
- Pokrenuli razvojno okruženje za Javu (bilo da je to Codespaces, dev kontejneri ili lokalno)
- Povezali se na Azure AI Foundry s autentifikacijom bez ključa (Microsoft Entra ID) — bez API ključeva
- Testirali da sve radi s jednostavnim primjerom koji komunicira s vašim modelom

## Naredni koraci

[Poglavlje 3: Osnovne tehnike Generativne AI](../03-CoreGenerativeAITechniques/README.md)

## Rješavanje problema

Imate problema? Evo uobičajenih problema i rješenja:

- **Autentifikacija ne uspijeva (401/403)?** 
  - Pokrenite `az login` — autentifikacija je bez ključa, stoga morate biti prijavljeni
  - Provjerite ima li vaš račun ulogu **Cognitive Services OpenAI User** na resursu
  - Ako ste tek nabavili, pričekajte minutu da se dodjela uloge propagira

- **Maven nije pronađen?** 
  - Ako koristite dev kontejnere/Codespaces, Maven bi trebao biti unaprijed instaliran
  - Za lokalnu instalaciju, osigurajte Java 21+ i Maven 3.9+
  - Probajte `mvn --version` za provjeru instalacije

- **`azd` nije pronađen ili nabava ne uspijeva?** 
  - Instalirajte [Azure Developer CLI](https://aka.ms/azure-dev/install) i pokrenite `azd auth login`
  - Odaberite regiju u kojoj je `gpt-4o-mini` dostupan (npr. `eastus2`)
  - Pogledajte [vodič za Azure AI Foundry](getting-started-azure-openai.md) za detalje

- **Dev kontejner se ne pokreće?** 
  - Provjerite je li Docker Desktop pokrenut (za lokalni razvoj)
  - Pokušajte ponovno izgraditi kontejner: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **Greške pri kompajliranju aplikacije?**
  - Provjerite jeste li u ispravnoj datoteci: `02-SetupDevEnvironment/examples/basic-chat-azure`
  - Pokušajte očistiti i ponovno kompajlirati: `mvn clean compile`

> **Trebate pomoć?** Još uvijek imate problema? Otvorite issue u repozitoriju i pomoći ćemo vam.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Napomena**:
Ovaj dokument je preveden korištenjem AI prevoditeljskog servisa [Co-op Translator](https://github.com/Azure/co-op-translator). Iako težimo točnosti, imajte na umu da automatski prijevodi mogu sadržavati greške ili netočnosti. Izvorni dokument na izvornom jeziku treba smatrati autoritativnim izvorom. Za važne informacije preporuča se profesionalni ljudski prijevod. Nismo odgovorni za bilo kakva nesporazumevanja ili pogrešne interpretacije koje proizlaze iz korištenja ovog prijevoda.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->