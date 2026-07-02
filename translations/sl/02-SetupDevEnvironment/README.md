# Nastavitev razvojnega okolja za Generativno AI za Javo

> **Hiter začetek:** Priskrbite svoje AI modele na **Azure AI Foundry** kot kodo z Bicep + `azd` v nekaj minutah — glejte [Vodnik za nastavitev Azure AI Foundry](getting-started-azure-openai.md). Avtentikacija je **brezključna** (Microsoft Entra ID), tako da ni treba upravljati API ključev.

## Kaj se boste naučili

- Nastaviti razvojno okolje za Java AI aplikacije
- Izbrati in konfigurirati vaše priljubljeno razvojno okolje (prednostno v oblaku s Codespaces, lokalni razvojni kontejner ali popolna lokalna nastavitev)
- Preizkusiti vašo namestitev z povezavo na Azure AI Foundry model

## Kazalo vsebine

- [Kaj se boste naučili](#kaj-se-boste-naučili)
- [Uvod](#uvod)
- [1. korak: Nastavite razvojno okolje](#1-korak-nastavite-razvojno-okolje)
  - [Možnost A: GitHub Codespaces (priporočeno)](#možnost-a-github-codespaces-priporočeno)
  - [Možnost B: Lokalni razvojni kontejner](#možnost-b-lokalni-razvojni-kontejner)
  - [Možnost C: Uporabite svojo obstoječo lokalno namestitev](#možnost-c-uporabite-svojo-obstoječo-lokalno-namestitev)
- [2. korak: Priskrbite Azure AI Foundry](#2-korak-priskrbite-azure-ai-foundry)
- [3. korak: Preizkusite svojo namestitev](#3-korak-preizkusite-svojo-namestitev)
- [Razreševanje težav](#razreševanje-težav)
- [Povzetek](#povzetek)
- [Naslednji koraki](#naslednji-koraki)

## Uvod

To poglavje vas bo vodilo skozi postopek nastavitve razvojnega okolja. Za modele bomo ves čas tečaja uporabljali **Azure AI Foundry**. Modele priskrbite kot kodo z Bicep in Azure Developer CLI (`azd`), nato se povežite z **brezključnim preverjanjem pristnosti** (Microsoft Entra ID) — brez potrebe po kopiranju ali uhajanju API ključev.

**Lokalna namestitev ni potrebna!** Uporabite GitHub Codespaces, ki ponuja polno razvojno okolje v brskalniku, in od tam priskrbite Foundry.

Uporabljamo **Azure AI Foundry** za ta tečaj, ker:
- **Priskrbljeno kot koda** — en ukaz `azd up` namesti račun in modele
- **Brezključna avtentikacija** — preverjanje pristnosti preko Azure prijave ali upravljane identitete
- **Pripravljen za produkcijo** — ista koda deluje lokalno in v Azure
- **Fleksibilen** — zamenjate modele z menjavo imena namestitve, ne kode

> **Opomba**: Namestitve Azure AI Foundry se obračunavajo na osnovi tokenov (plačilo po porabi). Za podrobnosti o namestitvi, regijah in stroških glejte [vodnik za nastavitev Azure AI Foundry](getting-started-azure-openai.md).

## 1. korak: Nastavite razvojno okolje

<a name="quick-start-cloud"></a>

Ustvarili smo prednastavljen razvojni kontejner, da zmanjšamo čas za nastavitev in zagotovimo, da imate vse potrebne pripomočke za ta tečaj Generativne AI za Javo. Izberite želen način razvoja:

### Možnosti nastavitev okolja:

#### Možnost A: GitHub Codespaces (priporočeno)

**Začnite kodirati v 2 minutah - lokalna nastavitev ni potrebna!**

1. Razvejite ta repozitorij na vaš GitHub račun
   > **Opomba**: Če želite urediti osnovno konfiguracijo, poglejte [Konfiguracija razvojnega kontejnerja](../../../.devcontainer/devcontainer.json)
2. Kliknite **Code** → zavihek **Codespaces** → **...** → **New with options...**
3. Uporabite privzete nastavitve – izbrana bo **Konfiguracija razvojnega kontejnerja**: **Generative AI Java Development Environment**, prilagojeni devcontainer za ta tečaj
4. Kliknite **Create codespace**
5. Počakajte približno 2 minuti, da je okolje pripravljeno
6. Nadaljujte na [2. korak: Priskrbite Azure AI Foundry](#2-korak-priskrbite-azure-ai-foundry)

<img src="../../../translated_images/sl/codespaces.9945ded8ceb431a5.webp" alt="Posnetek zaslona: Codespaces podmeni" width="50%">

<img src="../../../translated_images/sl/image.833552b62eee7766.webp" alt="Posnetek zaslona: New with options" width="50%">

<img src="../../../translated_images/sl/codespaces-create.b44a36f728660ab7.webp" alt="Posnetek zaslona: Možnosti ustvarjanja codespace-a" width="50%">

> **Prednosti Codespaces:**
> - Ni potrebne lokalne namestitve
> - Deluje na katerikoli napravi z brskalnikom
> - Vnaprej konfigurirano z vsemi orodji in odvisnostmi
> - Brezplačnih 60 ur mesečno za osebne račune
> - Enotno okolje za vse udeležence

#### Možnost B: Lokalni razvojni kontejner

**Za razvijalce, ki imajo raje lokalni razvoj z Docker-jem**

1. Razvejite in sklonirajte ta repozitorij na vaš lokalni računalnik
   > **Opomba**: Če želite urediti osnovno konfiguracijo, poglejte [Konfiguracija razvojnega kontejnerja](../../../.devcontainer/devcontainer.json)
2. Namestite [Docker Desktop](https://www.docker.com/products/docker-desktop/) in [VS Code](https://code.visualstudio.com/)
3. Namestite [razširitev Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) v VS Code
4. Odprite mapo repozitorija v VS Code
5. Ko vas zahteva, kliknite **Reopen in Container** (ali uporabite `Ctrl+Shift+P` → "Dev Containers: Reopen in Container")
6. Počakajte, da se kontejner zgradi in zažene
7. Nadaljujte na [2. korak: Priskrbite Azure AI Foundry](#2-korak-priskrbite-azure-ai-foundry)

<img src="../../../translated_images/sl/devcontainer.21126c9d6de64494.webp" alt="Posnetek zaslona: Nastavitev razvojnega kontejnerja" width="50%">

<img src="../../../translated_images/sl/image-3.bf93d533bbc84268.webp" alt="Posnetek zaslona: Zaključek gradnje razvojnega kontejnerja" width="50%">

#### Možnost C: Uporabite svojo obstoječo lokalno namestitev

**Za razvijalce z obstoječim Java okoljem**

Pogoji:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html)
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) ali vaš priljubljeni IDE

Koraki:
1. Sklonirajte ta repozitorij na lokalni računalnik
2. Odprite projekt v vašem IDE
3. Nadaljujte na [2. korak: Priskrbite Azure AI Foundry](#2-korak-priskrbite-azure-ai-foundry)

> **Nasvet:** Če imate šibkejši računalnik, a želite lokalni VS Code, uporabite GitHub Codespaces! Vaš lokalni VS Code se lahko poveže z oblačnim Codespace-om za najboljše iz obeh svetov.

<img src="../../../translated_images/sl/image-2.fc0da29a6e4d2aff.webp" alt="Posnetek zaslona: Ustvarjena lokalna instanca devcontainer-ja" width="50%">

## 2. korak: Priskrbite Azure AI Foundry

Namestite AI modele tečaja na Azure AI Foundry kot kodo. Iz korena repozitorija:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```

`azd` bo zahteval ime okolja in regijo, priskrbel Azure AI Foundry račun z nameščanjem `gpt-4o-mini` in `text-embedding-3-small`, ter zapisal končno točko v primer `.env` — vse s **brezključnim** preverjanjem pristnosti (brez API ključev).

> **Celoten vodič:** Za predpogoje, ročno (portalno) alternativo, nasvete glede regije in opombe o stroških/čiščenju glejte [Vodnik za nastavitev Azure AI Foundry](getting-started-azure-openai.md).

## 3. korak: Preizkusite svojo namestitev

Ko so vaši Foundry modeli priskrbljeni, preizkusite povezavo z primer aplikacijo v [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure).

1. Odprite terminal v razvojnem okolju.
2. Pomaknite se do primerka:
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
3. Prepričajte se, da ste prijavljeni (brezključna avtentikacija potrebuje žeton):
   ```bash
   az login
   ```
   > Če ste zagnali `azd up`, je bil `.env` s končno točko že napisan za vas.
4. Zaženite aplikacijo:
   ```bash
   mvn clean spring-boot:run
   ```

Videli boste odgovor modela `gpt-4o-mini`.

### Razumevanje primer kode

Primer v `examples/basic-chat-azure` je Spring Boot aplikacija, ki uporablja **Spring AI** za povezavo z Azure AI Foundry z brezključnim preverjanjem pristnosti.

**Kaj ta koda počne:**
- **Poveže se** z Azure AI Foundry z vašo Azure prijavo (Microsoft Entra ID) — brez API ključa
- **Pošlje** poziv modelu `gpt-4o-mini`
- **Prejme** in prikaže odgovor AI
- **Preveri**, da vaša namestitev deluje pravilno

**Ključna odvisnost** (v `pom.xml`):
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

## Povzetek

Odlično! Sedaj imate vse nastavljeno:

- Priskrbljeni Azure AI Foundry modeli kot koda z Bicep + `azd`
- Vaše razvojno Java okolje deluje (bodisi Codespaces, razvojni kontejnerji ali lokalno)
- Povezali ste se z Azure AI Foundry z brezključnim preverjanjem pristnosti (Microsoft Entra ID) — brez API ključev
- Preizkusili ste, da vse deluje s preprostim primerom, ki komunicira z vašim modelom

## Naslednji koraki

[Poglavje 3: Osnovne tehnike generativne AI](../03-CoreGenerativeAITechniques/README.md)

## Razreševanje težav

Imate težave? Tukaj so pogoste težave in rešitve:

- **Avtentikacija ne uspeva (401/403)?** 
  - Zaženite `az login` — preverjanje pristnosti je brezključna, zato morate biti prijavljeni
  - Preverite, da ima vaš račun vlogo **Cognitive Services OpenAI User** na viru
  - Če ste pravkar priskrbeli, počakajte trenutek, da se dodelitev vlog razširi

- **Maven ni najden?** 
  - Če uporabljate razvojne kontejnere/Codespaces, je Maven že prednameščen
  - Za lokalno nastavitev zagotovite, da sta nameščena Java 21+ in Maven 3.9+
  - Poskusite `mvn --version`, da preverite namestitev

- **`azd` ni najden ali priskrbljanje ne uspe?** 
  - Namestite [Azure Developer CLI](https://aka.ms/azure-dev/install) in poženite `azd auth login`
  - Izberite regijo, kjer je na voljo `gpt-4o-mini` (npr. `eastus2`)
  - Glejte [vodnik za nastavitev Azure AI Foundry](getting-started-azure-openai.md) za podrobnosti

- **Razvojni kontejner se ne zažene?** 
  - Poskrbite, da je Docker Desktop zagnan (za lokalni razvoj)
  - Poskusite prebuildu kontejner s `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **Napake pri prevajanju aplikacije?**
  - Preverite, da ste v pravilni mapi: `02-SetupDevEnvironment/examples/basic-chat-azure`
  - Poskusite očistiti in ponovno sestaviti: `mvn clean compile`

> **Potrebujete pomoč?** Še vedno imate težave? Odprite vprašanje v repozitoriju in pomagali vam bomo.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Omejitev odgovornosti**:
Ta dokument je bil preveden z uporabo AI prevajalske storitve [Co-op Translator](https://github.com/Azure/co-op-translator). Čeprav si prizadevamo za natančnost, vas prosimo, da upoštevate, da avtomatizirani prevodi lahko vsebujejo napake ali netočnosti. Izvirni dokument v njegovem izvirnem jeziku je treba obravnavati kot avtoritativni vir. Za kritične informacije je priporočljiv strokovni človeški prevod. Ne odgovarjamo za morebitna nesporazume ali napačne interpretacije, ki izhajajo iz uporabe tega prevoda.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->