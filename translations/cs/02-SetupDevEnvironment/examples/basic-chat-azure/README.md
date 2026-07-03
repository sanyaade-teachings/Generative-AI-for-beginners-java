# Základní chat s Azure AI Foundry – kompletní příklad

Tento příklad je jednoduchá aplikace Spring Boot, která se připojuje k modelu **Azure AI Foundry** pomocí **autentizace bez klíče** (Microsoft Entra ID) a testuje vaše nastavení. Používá Spring AI `ChatClient`.

## Obsah

- [Požadavky](#požadavky)
- [Rychlý start](#rychlý-start)
- [Jak funguje autentizace](#jak-funguje-autentizace)
- [Spuštění aplikace](#spuštění-aplikace)
  - [Použití Maven](#použití-maven)
  - [Použití VS Code](#použití-vs-code)
  - [Očekávaný výstup](#očekávaný-výstup)
- [Reference konfigurace](#reference-konfigurace)
  - [Proměnné prostředí](#proměnné-prostředí)
  - [Konfigurace Spring](#konfigurace-spring)
- [Řešení problémů](#řešení-problémů)
  - [Běžné problémy](#běžné-problémy)
  - [Režim ladění](#režim-ladění)
- [Další kroky](#další-kroky)
- [Zdroje](#zdroje)

## Požadavky

Před spuštěním tohoto příkladu se ujistěte, že máte:

- Azure AI Foundry zdroj s nasazením `gpt-4o-mini` — zprovozněte ho pomocí `azd up` nebo ručně přes [průvodce nastavením Azure AI Foundry](../../getting-started-azure-openai.md)
- roli **Cognitive Services OpenAI User** u tohoto zdroje (Bicep šablony ji přiřazují automaticky)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli) přihlášený pomocí `az login`
- Java 21+ a Maven 3.9+

> **API klíč není potřeba** — autentizace je bez klíče přes Microsoft Entra ID.

## Rychlý start

```bash
# 1. Přejděte do projektu
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Přihlaste se, aby klíčová autentizace mohla získat token
az login

# 3. Nakonfigurujte endpoint
#    - Pokud jste spustili `azd up`, .env byl automaticky vytvořen (přeskočte tento krok).
#    - Jinak zkopírujte šablonu a nastavte AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. Spusťte aplikaci
mvn spring-boot:run
```

## Jak funguje autentizace

Tento příklad se autentizuje pomocí **Microsoft Entra ID** — API klíč není vyžadován.

Když je nastaven pouze `spring.ai.azure.openai.endpoint` (a není uveden api-key), Spring AI sestaví Azure OpenAI klienta pomocí [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). Toto pověření automaticky získá token z vaší lokální relace `az login` nebo z managed identity při běhu v Azure — takže stejný kód funguje na obou místech bez úprav.

## Spuštění aplikace

### Použití Maven

```bash
mvn spring-boot:run
```

### Použití VS Code

1. Otevřete projekt ve VS Code
2. Stiskněte `F5` nebo použijte panel „Run and Debug“
3. Vyberte konfiguraci "Spring Boot-BasicChatApplication"

> **Poznámka**: Konfigurace VS Code automaticky načte váš soubor .env

### Očekávaný výstup

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

## Reference konfigurace

### Proměnné prostředí

| Proměnná | Popis | Povinná | Příklad |
|----------|-------|---------|---------|
| `AZURE_OPENAI_ENDPOINT` | URL koncového bodu Foundry (Azure OpenAI) | Ano | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Název nasazení chat modelu | Ne | `gpt-4o-mini` (výchozí) |

> Proměnná API klíče **neexistuje** — autentizace je bez klíče (Microsoft Entra ID přes `az login`).

### Konfigurace Spring

Soubor `application.yml` konfiguruje:
- **Endpoint**: `${AZURE_OPENAI_ENDPOINT}` - Z proměnné prostředí
- **Nasazení**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - Z proměnné prostředí s rezervní hodnotou
- **Autentizace**: bez klíče — není nastavena `api-key`, takže Spring AI používá `DefaultAzureCredential`
- **Teplota**: `0.7` - Řídí kreativitu (0.0 = deterministické, 1.0 = kreativní)
- **Maximální počet tokenů**: `500` - Maximální délka odpovědi

## Řešení problémů

### Běžné problémy

<details>
<summary><strong>Chyba: 401 / "PermissionDenied" / chyby s tokenem</strong></summary>

- Spusťte `az login` — autentizace bez klíče vyžaduje aktivní přihlášení pro získání tokenu
- Ověřte, že váš účet má roli **Cognitive Services OpenAI User** u zdroje
- Pokud jste roli právě přiřadili, počkejte chvíli na její propagaci
- Zkontrolujte, že jste ve správném tenantovi/předplatném (`az account show`)
</details>

<details>
<summary><strong>Chyba: "The endpoint is not valid" / chyby připojení</strong></summary>

- Ujistěte se, že `AZURE_OPENAI_ENDPOINT` je úplná základní URL (např. `https://your-resource.openai.azure.com/`)
- Zkontrolujte konzistenci koncového lomítka
- Ověřte, že endpoint odpovídá vašemu nakonfigurovanému zdroji (`azd env get-values`)
</details>

<details>
<summary><strong>Chyba: "The deployment was not found"</strong></summary>

- Ověřte, že `AZURE_OPENAI_DEPLOYMENT` odpovídá názvu nasazení v Azure
- Zkontrolujte, zda je model úspěšně nasazený a aktivní
- Výchozí název nasazení je `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: Nenačítají se proměnné prostředí</strong></summary>

- Ujistěte se, že váš `.env` soubor je v kořenovém adresáři projektu (na stejné úrovni jako `pom.xml`)
- Zkuste spustit `mvn spring-boot:run` v integrovaném terminálu VS Code
- Zkontrolujte, zda je správně nainstalované rozšíření VS Code pro Javu
</details>

### Režim ladění

Pro zapnutí podrobného logování odkomentujte tyto řádky v `application.yml`:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Další kroky

**Nastavení dokončeno!** Pokračujte ve svém vzdělávání:

[Kapitol 3: Základní generativní AI techniky](../../../03-CoreGenerativeAITechniques/README.md)

## Zdroje

- [Dokumentace Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Autentizace bez klíče pomocí Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Portál Azure AI Foundry](https://ai.azure.com/)
- [Dokumentace Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Prohlášení o omezení odpovědnosti**:
Tento dokument byl přeložen pomocí AI překladatelské služby [Co-op Translator](https://github.com/Azure/co-op-translator). Přestože usilujeme o co největší přesnost, mějte prosím na paměti, že automatizované překlady mohou obsahovat chyby nebo nepřesnosti. Originální dokument v jeho mateřském jazyce by měl být považován za autoritativní zdroj. Pro kritické informace se doporučuje profesionální lidský překlad. Nejsme odpovědní za jakékoli nedorozumění nebo nesprávné interpretace vzniklé použitím tohoto překladu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->