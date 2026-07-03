# Nastavenie vývojového prostredia pre Azure AI Foundry

> Tento sprievodca nastavuje modely **Azure AI Foundry** pre Java AI aplikácie v tomto kurze pomocou **overovania bez kľúčov** (Microsoft Entra ID) — nie je potrebné spravovať žiadne API kľúče. Ste nováčik v nástrojoch? Začnite so [sprievodcom vývojovým prostredím](./README.md).

Tento sprievodca nastavuje modely **Azure AI Foundry** pre Java AI aplikácie v tomto kurze. Máte dve možnosti:

- **Možnosť A — Provisionovanie pomocou `azd` + Bicep (odporúčané):** jeden príkaz nasadí Foundry účet a modely ako kód. Žiadne klikanie v portáli.
- **Možnosť B — Vytvorte zdroje manuálne** v portáli Azure AI Foundry.

Obe možnosti používajú **overovanie bez kľúčov** (Microsoft Entra ID) — nie je potrebné kopírovať alebo zverejňovať API kľúče.

## Obsah

- [Čo sa vytvorí](#čo-sa-vytvorí)
- [Požiadavky](#požiadavky)
- [Možnosť A: Provisionovanie pomocou azd + Bicep (odporúčané)](#option-a-provision-with-azd--bicep-recommended)
- [Možnosť B: Vytvorte zdroje manuálne](#možnosť-b-vytvorte-zdroje-manuálne)
- [Nastavte svoje prostredie](#nastavte-svoje-prostredie)
- [Otestujte nastavenie](#otestujte-nastavenie)
- [Čo je ďalej?](#čo-je-ďalej)
- [Zdroje](#zdroje)
- [Ďalšie zdroje](#ďalšie-zdroje)

## Čo sa vytvorí

Bicep šablóny v priečinku [`infra/`](../../../02-SetupDevEnvironment/infra) vytvoria:

- **Azure AI Foundry** účet (`Microsoft.CognitiveServices/accounts`, typ `AIServices`) s projektom
- Nasadenie **chat** — `gpt-4o-mini`
- Nasadenie **embedding** — `text-embedding-3-small` (použité v neskorších kapitolách)
- **Priradenie role bez kľúča** (`Cognitive Services OpenAI User`), aby ste sa mohli prihlasovať cez `az login` namiesto správy kľúčov

## Požiadavky

- [Predplatné Azure](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) a [Maven 3.9+](https://maven.apache.org/download.cgi)

## Možnosť A: Provisionovanie pomocou azd + Bicep (odporúčané)

Zo zložky `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# Prihlásiť sa (oba nástroje)
azd auth login
az login

# Poskytnite účet Foundry + nasadenie modelov
azd up
```

`azd` vás vyzve na zadanie **názvu prostredia** (napríklad `genai-java`) a **regiónu**. Vyberte región, kde sú dostupné `gpt-4o-mini` a `text-embedding-3-small` — napríklad `eastus2` alebo `swedencentral`.

Po dokončení provisionovania `azd`:

1. Nasadí všetko definované v [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep).
2. Spustí postprovision hook, ktorý zapíše [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) s vašou koncovou adresou a názvami nasadení (žiadne tajné údaje).

> **Tip:** Kedykoľvek spusťte znova `azd up` na aplikovanie zmien. Spustením `azd down` všetko zmažete a zastavíte generovanie nákladov.

Ak chcete vidieť vygenerované nastavenia:

```bash
azd env get-values
```

Teraz preskočte na [Otestujte nastavenie](#otestujte-nastavenie).

## Možnosť B: Vytvorte zdroje manuálne

Uprednostňujete portál? Vytvorte zdroje manuálne:

1. Prejdite na [Azure AI Foundry portál](https://ai.azure.com/) a prihláste sa.
2. **Vytvorte projekt** (tým sa vytvorí aj zdroj AI Foundry). Pomenujte ho napríklad `GenAIJava`.
3. Vo svojom projekte otvorte **Models + endpoints** → **Deploy model** → **Deploy base model**.
4. Nasadte **gpt-4o-mini** (názov nasadenia `gpt-4o-mini`). Rovnaký postup pre **text-embedding-3-small**, ak chcete použiť príklady s embeddingom.
5. Z **Overview** skopírujte **endpoint** (napríklad `https://<resource>.openai.azure.com/`).
6. Pridajte si prístup bez kľúča: na zdroji otvorte **Access control (IAM)** → **Add role assignment** → priraďte **Cognitive Services OpenAI User** svojmu účtu.

> **Máte stále problémy?** Pozrite si [dokumentáciu Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Nastavte svoje prostredie

**Ak ste použili Možnosť A (`azd up`)**, váš súbor s nastaveniami je už vytvorený — nie je potrebné nič nastavovať. Preskočte na [Otestujte nastavenie](#otestujte-nastavenie).

**Ak ste použili Možnosť B (manuálne)**, vytvorte si súbor `.env` z príkladu sami:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

Upravte `.env` so svojou koncovou adresou (bez kľúča — autentifikácia je bez kľúča):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Bezpečnostná poznámka:** Nie je potrebné ukladať žiadny API kľúč. Autentifikujete sa cez Microsoft Entra ID pomocou `az login` (lokálne) alebo spravovanú identitu (v Azure). Súbor `.env` obsahuje iba ne-tajné nastavenia a je už zahrnutý v `.gitignore`.

## Otestujte nastavenie

Uistite sa, že ste prihlásení, aby overovanie bez kľúča mohlo získať token, potom spustite príklad:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # ak ešte nie ste prihlásený
mvn clean spring-boot:run
```

Mali by ste vidieť odpoveď z modelu `gpt-4o-mini`!

> **Používatelia VS Code:** Stlačte `F5` na spustenie. Aplikácia automaticky načíta váš `.env`.

> **Úplný príklad:** Podrobnosti a riešenie problémov nájdete v [Basic Chat s Azure AI Foundry príklade](./examples/basic-chat-azure/README.md).

## Čo je ďalej?

**Nastavenie dokončené!** Teraz máte:
- Azure AI Foundry s nasadenými `gpt-4o-mini` a `text-embedding-3-small`
- Overovanie bez kľúča (Microsoft Entra ID) — žiadne kľúče na správu
- Lokálny `.env` so svojou koncovou adresou a názvami nasadení
- Vývojové prostredie Java pripravené na použitie

**Pokračujte na** [Kapitolu 3: Základné techniky generatívnej AI](../03-CoreGenerativeAITechniques/README.md) pre začiatok vývoja AI aplikácií!

## Zdroje

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Overovanie bez kľúča s Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Dokumentácia Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI Dokumentácia](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Ďalšie zdroje

- [Stiahnite si VS Code](https://code.visualstudio.com/Download)
- [Získajte Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Konfigurácia vývojového kontajnera](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Vyhlásenie o zodpovednosti**:
Tento dokument bol preložený pomocou AI prekladateľskej služby [Co-op Translator](https://github.com/Azure/co-op-translator). Hoci sa snažíme o presnosť, vezmite prosím na vedomie, že automatické preklady môžu obsahovať chyby alebo nepresnosti. Pôvodný dokument v jeho natívnom jazyku by mal byť považovaný za autoritatívny zdroj. Pre kritické informácie sa odporúča profesionálny ľudský preklad. Nie sme zodpovední za žiadne nedorozumenia alebo nesprávne interpretácie vyplývajúce z použitia tohto prekladu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->