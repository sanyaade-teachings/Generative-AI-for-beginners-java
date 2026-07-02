# Nastavenie vývojového prostredia pre Generatívnu AI pre Java

> **Rýchly štart:** Províziu vašich AI modelov na **Azure AI Foundry** ako kódu pomocou Bicep + `azd` za pár minút — pozrite si [Azure AI Foundry Sprievodcu nastavením](getting-started-azure-openai.md). Overovanie je **bez kľúčov** (Microsoft Entra ID), takže nie je potrebné spravovať API kľúče.

## Čo sa naučíte

- Nastaviť vývojové prostredie pre AI aplikácie v Jave
- Vybrať a nakonfigurovať preferované vývojové prostredie (cloud-first s Codespaces, lokálny vývojový kontajner alebo plná lokálna inštalácia)
- Otestovať nastavenie prepojením s modelom Azure AI Foundry

## Obsah

- [Čo sa naučíte](#čo-sa-naučíte)
- [Úvod](#úvod)
- [Krok 1: Nastavte svoje vývojové prostredie](#krok-1-nastavte-svoje-vývojové-prostredie)
  - [Možnosť A: GitHub Codespaces (odporúčané)](#možnosť-a-github-codespaces-odporúčané)
  - [Možnosť B: Lokálny Dev Container](#možnosť-b-lokálny-dev-container)
  - [Možnosť C: Použite existujúcu lokálnu inštaláciu](#možnosť-c-použite-existujúcu-lokálnu-inštaláciu)
- [Krok 2: Provízia Azure AI Foundry](#krok-2-provízia-azure-ai-foundry)
- [Krok 3: Otestujte svoje nastavenie](#krok-3-otestujte-svoje-nastavenie)
- [Riešenie problémov](#riešenie-problémov)
- [Zhrnutie](#zhrnutie)
- [Ďalšie kroky](#ďalšie-kroky)

## Úvod

Táto kapitola vás prevedie nastavením vývojového prostredia. Počas tohto kurzu budeme používať **Azure AI Foundry** pre modely. Modele provisionujete ako kód pomocou Bicep a Azure Developer CLI (`azd`), potom sa pripájate pomocou **overovania bez kľúčov** (Microsoft Entra ID) — bez kopírovania alebo úniku API kľúčov.

**Nie je potrebné žiadne lokálne nastavenie!** Môžete použiť GitHub Codespaces, ktoré poskytuje plné vývojové prostredie priamo vo vašom prehliadači a z neho provisionovať Foundry.

Používame **Azure AI Foundry**, pretože je:
- **Provisionované ako kód** — jedno `azd up` nasadí účet a nasadenia modelov
- **Bez kľúčov** — autentifikácia cez váš Azure prihlásenie alebo spravovanú identitu
- **Pripravené na produkciu** — rovnaký kód beží lokálne aj v Azure
- **Flexibilné** — výmenu modelov spravíte zmenou názvu nasadenia, nie kódu

> **Poznámka**: Nasadenia Azure AI Foundry sa účtujú podľa počtu tokenov (platíte podľa spotreby). Viac informácií o provisioning, regiónoch a cenách nájdete v [Azure AI Foundry sprievodcovi nastavením](getting-started-azure-openai.md).


## Krok 1: Nastavte svoje vývojové prostredie

<a name="quick-start-cloud"></a>

Vytvorili sme predkonfigurovaný vývojový kontajner na minimalizáciu času nastavenia a zabezpečenie všetkých potrebných nástrojov pre tento kurz Generatívnej AI pre Java. Vyberte si preferovaný spôsob vývoja:

### Možnosti nastavenia prostredia:

#### Možnosť A: GitHub Codespaces (odporúčané)

**Začnite kódovať za 2 minúty - žiadne lokálne nastavenie potrebné!**

1. Forknite si tento repozitár do svojho GitHub účtu  
   > **Poznámka**: Ak chcete upraviť základnú konfiguráciu, pozrite si [Dev Container Configuration](../../../.devcontainer/devcontainer.json)
2. Kliknite na **Code** → záložka **Codespaces** → **...** → **New with options...**
3. Použite prednastavené hodnoty – vyberie sa **Dev container configuration**: **Generative AI Java Development Environment** špeciálny devcontainer vytvorený pre tento kurz
4. Kliknite na **Create codespace**
5. Počkajte približne 2 minúty, kým bude prostredie pripravené
6. Pokračujte na [Krok 2: Provízia Azure AI Foundry](#krok-2-provízia-azure-ai-foundry)

<img src="../../../translated_images/sk/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: Codespaces submenu" width="50%">

<img src="../../../translated_images/sk/image.833552b62eee7766.webp" alt="Screenshot: New with options" width="50%">

<img src="../../../translated_images/sk/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: Create codespace options" width="50%">


> **Výhody Codespaces**:  
> - Nie je potrebná lokálna inštalácia  
> - Funguje na akomkoľvek zariadení s prehliadačom  
> - Predkonfigurované so všetkými nástrojmi a závislosťami  
> - Zadarmo 60 hodín mesačne pre osobné účty  
> - Konzistentné prostredie pre všetkých študentov  

#### Možnosť B: Lokálny Dev Container

**Pre vývojárov, ktorí preferujú lokálny vývoj s Dockerom**

1. Forknite a naklonujte tento repozitár na svoj lokálny počítač  
   > **Poznámka**: Ak chcete upraviť základnú konfiguráciu, pozrite si [Dev Container Configuration](../../../.devcontainer/devcontainer.json)
2. Nainštalujte [Docker Desktop](https://www.docker.com/products/docker-desktop/) a [VS Code](https://code.visualstudio.com/)
3. Nainštalujte rozšírenie [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) vo VS Code
4. Otvorte priečinok repozitára vo VS Code
5. Keď sa zobrazí výzva, kliknite na **Reopen in Container** (alebo použite `Ctrl+Shift+P` → "Dev Containers: Reopen in Container")
6. Počkajte, kým sa kontajner zostaví a spustí
7. Pokračujte na [Krok 2: Provízia Azure AI Foundry](#krok-2-provízia-azure-ai-foundry)

<img src="../../../translated_images/sk/devcontainer.21126c9d6de64494.webp" alt="Screenshot: Dev container setup" width="50%">

<img src="../../../translated_images/sk/image-3.bf93d533bbc84268.webp" alt="Screenshot: Dev container build complete" width="50%">

#### Možnosť C: Použite existujúcu lokálnu inštaláciu

**Pre vývojárov s existujúcim Java prostredím**

Požiadavky:  
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html)  
- [Maven 3.9+](https://maven.apache.org/download.cgi)  
- [VS Code](https://code.visualstudio.com) alebo preferované IDE

Kroky:  
1. Naklonujte si tento repozitár do svojho lokálneho počítača  
2. Otvorte projekt vo svojom IDE  
3. Pokračujte na [Krok 2: Provízia Azure AI Foundry](#krok-2-provízia-azure-ai-foundry)

> **Tip**: Ak máte slabší počítač, ale chcete používať VS Code lokálne, využite GitHub Codespaces! Môžete prepojiť lokálny VS Code s cloud-hosted Codespace pre to najlepšie z oboch svetov.

<img src="../../../translated_images/sk/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: created local devcontainer instance" width="50%">


## Krok 2: Provízia Azure AI Foundry

Nasadte AI modely kurzu do Azure AI Foundry ako kód. V koreňovom priečinku repozitára spustite:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```
  
`azd` vás vyzve na zadanie názvu prostredia a regiónu, provisionuje Azure AI Foundry účet s nasadeniami `gpt-4o-mini` a `text-embedding-3-small` a zapíše endpoint do súboru `.env` v príklade — všetko s **overovaním bez kľúčov** (žiadne API kľúče).

> **Plný postup:** Pozrite si [Azure AI Foundry Sprievodcu nastavením](getting-started-azure-openai.md) pre požiadavky, manuálnu (portal) alternatívu, odporúčania regiónov a informácie o cenách a odstránení.

## Krok 3: Otestujte svoje nastavenie

Po provisionovaní Foundry modelov otestujte pripojenie príkladovou aplikáciou v [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure).

1. Otvorte terminál vo vašom vývojovom prostredí.  
2. Prejdite do príkladu:  
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
  
3. Uistite sa, že ste prihlásený (overovanie bez kľúčov vyžaduje token):  
   ```bash
   az login
   ```
  
   > Ak ste spustili `azd up`, `.env` súbor s endpointom už bol vytvorený za vás.  
4. Spustite aplikáciu:  
   ```bash
   mvn clean spring-boot:run
   ```
  
Mali by ste vidieť odpoveď od modelu `gpt-4o-mini`.

### Porozumenie príkladovému kódu

Príklad v `examples/basic-chat-azure` je Spring Boot aplikácia, ktorá používa **Spring AI** pre pripojenie k Azure AI Foundry s overením bez kľúčov.

**Čo tento kód robí:**  
- **Pripája sa** k Azure AI Foundry pomocou vášho Azure prihlásenia (Microsoft Entra ID) — bez API kľúča  
- **Posiela** prompt modelu `gpt-4o-mini`  
- **Prijíma** a zobrazuje odpoveď AI  
- **Overuje**, že vaše nastavenie funguje správne

**Kľúčová závislosť** (v `pom.xml`):  
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```
  
**Konfigurácia** (`application.yml`):  
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
  
## Zhrnutie

Skvelé! Teraz máte všetko nastavené:

- Províziou Azure AI Foundry modelov ako kód pomocou Bicep + `azd`  
- Spustené vaše Java vývojové prostredie (či už Codespaces, dev kontejnery alebo lokálne)  
- Pripojenie na Azure AI Foundry s overením bez kľúčov (Microsoft Entra ID) — žiadne API kľúče  
- Otestované všetko funguje jednoduchým príkladom komunikácie s modelom  

## Ďalšie kroky

[Kapitola 3: Základné techniky generatívnej AI](../03-CoreGenerativeAITechniques/README.md)

## Riešenie problémov

Máte problémy? Tu sú bežné chyby a ich riešenia:

- **Overenie zlyháva (401/403)?**  
  - Spustite `az login` — overovanie je bez kľúčov, musíte byť prihlásený  
  - Overte, že váš účet má rolu **Cognitive Services OpenAI User** na zdroji  
  - Ak ste práve provisionovali, počkajte chvíľu, kým sa priradenie role prejaví

- **Maven nenájdený?**  
  - Pri používaní dev kontajnerov/Codespaces by mal byť Maven predinštalovaný  
  - Pre lokálne nastavenie sa uistite, že máte nainštalované Java 21+ a Maven 3.9+  
  - Skúste `mvn --version` pre potvrdenie inštalácie

- **`azd` nenájdený alebo provisioning zlyháva?**  
  - Nainštalujte [Azure Developer CLI](https://aka.ms/azure-dev/install) a spustite `azd auth login`  
  - Vyberte región, kde je dostupný `gpt-4o-mini` (napr. `eastus2`)  
  - Pozrite [Azure AI Foundry sprievodcu nastavením](getting-started-azure-openai.md) pre viac detailov

- **Dev container sa nespúšťa?**  
  - Uistite sa, že Docker Desktop beží (pre lokálny vývoj)  
  - Skúste prebuildnúť kontajner: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **Chyby pri kompilácii aplikácie?**  
  - Uistite sa, že ste v správnom adresári: `02-SetupDevEnvironment/examples/basic-chat-azure`  
  - Skúste čistenie a znovu kompilovať: `mvn clean compile`

> **Potrebujete pomoc?**: Ak problémy pretrvávajú, otvorte issue v repozitári a radi vám pomôžeme.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Vyhlásenie o zodpovednosti**:
Tento dokument bol preložený pomocou AI prekladateľskej služby [Co-op Translator](https://github.com/Azure/co-op-translator). Hoci sa snažíme o presnosť, vezmite prosím na vedomie, že automatické preklady môžu obsahovať chyby alebo nepresnosti. Pôvodný dokument v jeho natívnom jazyku by mal byť považovaný za autoritatívny zdroj. Pre kritické informácie sa odporúča profesionálny ľudský preklad. Nie sme zodpovední za žiadne nedorozumenia alebo nesprávne interpretácie vyplývajúce z použitia tohto prekladu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->