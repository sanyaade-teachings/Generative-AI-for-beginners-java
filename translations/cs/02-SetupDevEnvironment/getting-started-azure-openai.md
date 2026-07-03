# Nastavení vývojového prostředí pro Azure AI Foundry

> Tento průvodce nastavuje modely **Azure AI Foundry** pro Java AI aplikace v tomto kurzu pomocí **keyless** autentizace (Microsoft Entra ID) — žádné API klíče k spravování. Jste v nástrojích noví? Začněte s [průvodcem vývojovým prostředím](./README.md).

Tento průvodce nastavuje modely **Azure AI Foundry** pro Java AI aplikace v tomto kurzu. Máte dvě možnosti:

- **Možnost A — Proviste pomocí `azd` + Bicep (doporučeno):** jeden příkaz nasadí Foundry účet a modely jako kód. Žádné klikání v portálu.
- **Možnost B — Vytvořte zdroje ručně** v Azure AI Foundry portálu.

Obě varianty používají **keyless autentizaci** (Microsoft Entra ID) — nejsou žádné API klíče k zkopírování nebo úniku.

## Obsah

- [Co se vytvoří](#co-se-vytvoří)
- [Požadavky](#požadavky)
- [Možnost A: Provision pomocí azd + Bicep (doporučeno)](#option-a-provision-with-azd--bicep-recommended)
- [Možnost B: Vytvoření zdrojů ručně](#možnost-b-vytvoření-zdrojů-ručně)
- [Konfigurace vašeho prostředí](#konfigurace-vašeho-prostředí)
- [Otestujte své nastavení](#otestujte-své-nastavení)
- [Co dál?](#co-dál)
- [Zdroje](#zdroje)
- [Další zdroje](#další-zdroje)

## Co se vytvoří

Bicep šablony v [`infra/`](../../../02-SetupDevEnvironment/infra) vytvoří:

- **Azure AI Foundry** účet (`Microsoft.CognitiveServices/accounts`, druh `AIServices`) s projektem
- Nasazení **chatu** — `gpt-4o-mini`
- Nasazení **embeddingu** — `text-embedding-3-small` (používá se v pozdějších kapitolách)
- **Keyless role assignment** (`Cognitive Services OpenAI User`), díky čemuž se přihlásíte pomocí `az login` místo správy klíčů

## Požadavky

- [Azure předplatné](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) a [Maven 3.9+](https://maven.apache.org/download.cgi)

## Možnost A: Provision pomocí azd + Bicep (doporučeno)

Z adresáře `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# Přihlásit se (oba nástroje)
azd auth login
az login

# Zajistit účet Foundry + nasazení modelů
azd up
```

`azd` vás vyzve k zadání **jména prostředí** (například `genai-java`) a **regionu**. Vyberte region, kde jsou dostupné `gpt-4o-mini` a `text-embedding-3-small` — například `eastus2` nebo `swedencentral`.

Po dokončení provisioningu `azd`:

1. Nasadí vše, co je definováno v [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep).
2. Spustí postprovision hook, který zapíše [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) s vaším endpointem a jmény nasazení (žádná tajemství).

> **Tip:** Kdykoliv proveďte znovu `azd up` pro aplikaci změn. Spusťte `azd down` pro odstranění všeho a zastavení nákladů.

Pro zobrazení vygenerovaných nastavení:

```bash
azd env get-values
```

Nyní přejděte na [Otestujte své nastavení](#otestujte-své-nastavení).

## Možnost B: Vytvoření zdrojů ručně

Dáte přednost portálu? Vytvořte zdroje ručně:

1. Přejděte na [Azure AI Foundry portál](https://ai.azure.com/) a přihlaste se.
2. **Vytvořte projekt** (to zároveň vytvoří Azure AI Foundry zdroj). Pojmenujte jej například `GenAIJava`.
3. Ve svém projektu otevřete **Models + endpoints** → **Deploy model** → **Deploy base model**.
4. Nasadíte **gpt-4o-mini** (jméno nasazení `gpt-4o-mini`). Opakujte pro **text-embedding-3-small**, pokud chcete příklady embeddingu.
5. Z **Přehledu** (Overview) zkopírujte **endpoint** (například `https://<resource>.openai.azure.com/`).
6. Udělte si přístup bez klíče: na zdroji otevřete **Access control (IAM)** → **Add role assignment** → přiřaďte roli **Cognitive Services OpenAI User** na váš účet.

> **Stále máte problémy?** Podívejte se na [dokumentaci Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Konfigurace vašeho prostředí

**Pokud jste použili Možnost A (`azd up`)**, váš konfigurační soubor je již zapsán — není třeba nic nastavovat. Přeskočte na [Otestujte své nastavení](#otestujte-své-nastavení).

**Pokud jste použili Možnost B (ručně)**, vytvořte `.env` soubor příkladu sami:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

Upravte `.env` s vaším endpointem (žádný klíč — autentizace je keyless):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Bezpečnostní upozornění:** Žádný API klíč není potřeba ukládat. Autentizujete se pomocí Microsoft Entra ID přes `az login` (lokálně) nebo spravovanou identitou (v Azure). `.env` soubor obsahuje pouze nezabezpečená nastavení a je již v `.gitignore`.

## Otestujte své nastavení

Ujistěte se, že jste přihlášeni, aby keyless autentizace získala token, poté spusťte příklad:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # pokud ještě nejste přihlášeni
mvn clean spring-boot:run
```

Měli byste vidět odpověď od modelu `gpt-4o-mini`!

> **Uživatelé VS Code:** Stiskněte `F5` pro spuštění. Aplikace automaticky načte váš `.env`.

> **Plný příklad:** Podívejte se na [Basic Chat s Azure AI Foundry příklad](./examples/basic-chat-azure/README.md) pro podrobnosti a řešení problémů.

## Co dál?

**Nastavení dokončeno!** Nyní máte:
- Nasazený Azure AI Foundry s `gpt-4o-mini` a `text-embedding-3-small`
- Keyless autentizaci (Microsoft Entra ID) — žádné klíče ke správě
- Lokální `.env` s vaším endpointem a jmény nasazení
- Java vývojové prostředí připravené k použití

**Pokračujte do** [Kapitoly 3: Základní generativní AI techniky](../03-CoreGenerativeAITechniques/README.md) a začněte tvořit AI aplikace!

## Zdroje

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Keyless autentizace s Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Dokumentace Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI dokumentace](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Další zdroje

- [Stáhnout VS Code](https://code.visualstudio.com/Download)
- [Získat Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Konfigurace Dev Container](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Prohlášení o omezení odpovědnosti**:
Tento dokument byl přeložen pomocí AI překladatelské služby [Co-op Translator](https://github.com/Azure/co-op-translator). Přestože usilujeme o co největší přesnost, mějte prosím na paměti, že automatizované překlady mohou obsahovat chyby nebo nepřesnosti. Originální dokument v jeho mateřském jazyce by měl být považován za autoritativní zdroj. Pro kritické informace se doporučuje profesionální lidský překlad. Nejsme odpovědní za jakékoli nedorozumění nebo nesprávné interpretace vzniklé použitím tohoto překladu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->