# Základná konverzácia s Azure AI Foundry - kompletný príklad

Tento príklad je jednoduchá aplikácia Spring Boot, ktorá sa pripája k modelu **Azure AI Foundry** pomocou **overovania bez kľúča** (Microsoft Entra ID) a testuje vaše nastavenie. Používa Spring AI `ChatClient`.

## Obsah

- [Predpoklady](#prerekvizity)
- [Rýchly štart](#rychly-start)
- [Ako funguje overovanie](#ako-funguje-overovanie)
- [Spustenie aplikácie](#spustenie-aplikacie)
  - [Použitie Maven](#pouzitie-maven)
  - [Použitie VS Code](#pouzitie-vs-code)
  - [Očakávaný výstup](#ocakavany-vystup)
- [Referencia konfigurácie](#referencia-konfiguracie)
  - [Premenné prostredia](#premenne-prostredia)
  - [Spring konfigurácia](#spring-konfiguracia)
- [Riešenie problémov](#riesenie-problemov)
  - [Bežné problémy](#bezne-problemy)
  - [Režim ladenia](#rezim-ladenia)
- [Ďalšie kroky](#dalej-kroky)
- [Zdroje](#zdroje)

## Prerekvizity

Pred spustením tohto príkladu sa uistite, že máte:

- Zdroj Azure AI Foundry s nasadením `gpt-4o-mini` — nastavte ho pomocou `azd up` alebo manuálne cez [návod na nastavenie Azure AI Foundry](../../getting-started-azure-openai.md)
- Rola **Cognitive Services OpenAI User** pre tento zdroj (Bicep šablóny ju priraďujú automaticky)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), prihlásený pomocou `az login`
- Java 21+ a Maven 3.9+

> **Nie je potrebný žiadny API kľúč** — overovanie je bez kľúča cez Microsoft Entra ID.

## Rýchly štart

```bash
# 1. Prejdite do projektu
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Prihláste sa, aby mohla autentifikácia bez kľúča získať token
az login

# 3. Nakonfigurujte koncový bod
#    - Ak ste spustili `azd up`, .env bol pre vás napísaný (toto preskočte).
#    - Inak skopírujte šablónu a nastavte AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. Spustite aplikáciu
mvn spring-boot:run
```

## Ako funguje overovanie

Tento príklad sa overuje pomocou **Microsoft Entra ID** — API kľúč sa nepoužíva.

Keď je nastavený iba `spring.ai.azure.openai.endpoint` (a nie `api-key`), Spring AI vytvorí Azure OpenAI klienta s [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). Tento credential automaticky nájde token z vašej lokálnej relácie `az login` alebo z managed identity pri behu v Azure — takže ten istý kód funguje na oboch miestach bez zmien.

## Spustenie aplikácie

### Použitie Maven

```bash
mvn spring-boot:run
```

### Použitie VS Code

1. Otvorte projekt vo VS Code
2. Stlačte `F5` alebo použite panel "Run and Debug"
3. Vyberte konfiguráciu "Spring Boot-BasicChatApplication"

> **Poznámka**: Konfigurácia VS Code automaticky načíta váš .env súbor

### Očakávaný výstup

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

## Referencia konfigurácie

### Premenné prostredia

| Premenná | Popis | Povinné | Príklad |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | Adresa endpointu Foundry (Azure OpenAI) | Áno | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Názov nasadenia chat modelu | Nie | `gpt-4o-mini` (predvolené) |

> Nie je tu **žiadna** premenná pre API kľúč — overovanie je bez kľúča (Microsoft Entra ID cez `az login`).

### Spring konfigurácia

Súbor `application.yml` konfiguruje:
- **Endpoint**: `${AZURE_OPENAI_ENDPOINT}` - z premennej prostredia
- **Nasadenie**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - z premennej prostredia s náhradou
- **Overovanie**: bez kľúča — nie je nastavený `api-key`, takže Spring AI používa `DefaultAzureCredential`
- **Teplota**: `0.7` - ovplyvňuje kreativitu (0.0 = deterministické, 1.0 = kreatívne)
- **Maximálny počet tokenov**: `500` - maximálna dĺžka odpovede

## Riešenie problémov

### Bežné problémy

<details>
<summary><strong>Chyba: 401 / "PermissionDenied" / chyby tokenu</strong></summary>

- Spustite `az login` — overovanie bez kľúča vyžaduje aktívne prihlásenie na získanie tokenu
- Overte, či má váš účet rolu **Cognitive Services OpenAI User** pre daný zdroj
- Ak ste práve priradili rolu, počkajte chvíľu na jej propagáciu
- Potvrďte, že ste v správnom tenantovi/predplatnom (`az account show`)
</details>

<details>
<summary><strong>Chyba: "Endpoint nie je platný" / chyby pripojenia</strong></summary>

- Uistite sa, že `AZURE_OPENAI_ENDPOINT` je úplná základná URL (napr. `https://your-resource.openai.azure.com/`)
- Skontrolujte konzistenciu lomítka na konci
- Overte, či endpoint zodpovedá vášmu nasadenému zdroju (`azd env get-values`)
</details>

<details>
<summary><strong>Chyba: "Nasadenie nebolo nájdené"</strong></summary>

- Skontrolujte, či `AZURE_OPENAI_DEPLOYMENT` zodpovedá názvu nasadenia v Azure
- Overte, že model je úspešne nasadený a aktívny
- Predvolený názov nasadenia je `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: Premenné prostredia sa nenačítavajú</strong></summary>

- Uistite sa, že súbor `.env` je v koreňovom adresári projektu (na rovnakej úrovni ako `pom.xml`)
- Skúste spustiť `mvn spring-boot:run` v integrovanom termináli VS Code
- Skontrolujte, či je rozšírenie pre Javu vo VS Code správne nainštalované
</details>

### Režim ladenia

Ak chcete povoliť podrobné logy, odkomentujte tieto riadky v `application.yml`:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Ďalšie kroky

**Nastavenie dokončené!** Pokračujte vo vašom vzdelávaní:

[Kap. 3: Základné techniky generatívnej AI](../../../03-CoreGenerativeAITechniques/README.md)

## Zdroje

- [Spring AI Azure OpenAI dokumentácia](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Overovanie bez kľúča cez Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Portál Azure AI Foundry](https://ai.azure.com/)
- [Dokumentácia Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Vyhlásenie o zodpovednosti**:
Tento dokument bol preložený pomocou AI prekladateľskej služby [Co-op Translator](https://github.com/Azure/co-op-translator). Hoci sa snažíme o presnosť, vezmite prosím na vedomie, že automatické preklady môžu obsahovať chyby alebo nepresnosti. Pôvodný dokument v jeho natívnom jazyku by mal byť považovaný za autoritatívny zdroj. Pre kritické informácie sa odporúča profesionálny ľudský preklad. Nie sme zodpovední za žiadne nedorozumenia alebo nesprávne interpretácie vyplývajúce z použitia tohto prekladu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->