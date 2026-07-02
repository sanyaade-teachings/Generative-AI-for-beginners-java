# Basis Chat met Azure AI Foundry - End-to-End Voorbeeld

Dit voorbeeld is een eenvoudige Spring Boot-applicatie die verbinding maakt met een **Azure AI Foundry**-model met behulp van **sleutelvrije authenticatie** (Microsoft Entra ID) en test je setup. Het gebruikt Spring AI's `ChatClient`.

## Inhoudsopgave

- [Vereisten](#vereisten)
- [Snel Starten](#snel-starten)
- [Hoe Authenticatie Werkt](#hoe-authenticatie-werkt)
- [De Applicatie Uitvoeren](#de-applicatie-uitvoeren)
  - [Maven Gebruiken](#maven-gebruiken)
  - [VS Code Gebruiken](#vs-code-gebruiken)
  - [Verwachte Uitvoer](#verwachte-uitvoer)
- [Configuratie Referentie](#configuratie-referentie)
  - [Omgevingsvariabelen](#omgevingsvariabelen)
  - [Spring Configuratie](#spring-configuratie)
- [Probleemoplossing](#probleemoplossing)
  - [Veelvoorkomende Problemen](#veelvoorkomende-problemen)
  - [Debug Modus](#debug-modus)
- [Volgende Stappen](#volgende-stappen)
- [Bronnen](#bronnen)

## Vereisten

Zorg ervoor dat je voor het uitvoeren van dit voorbeeld het volgende hebt:

- Een Azure AI Foundry-resource met een `gpt-4o-mini`-implementatie — maak deze aan met `azd up` of handmatig via de [Azure AI Foundry setup gids](../../getting-started-azure-openai.md)
- De **Cognitive Services OpenAI User** rol op die resource (de Bicep-sjablonen wijzen deze voor je toe)
- De [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), aangemeld met `az login`
- Java 21+ en Maven 3.9+

> **Geen API-sleutel nodig** — authenticatie is sleutelvrij via Microsoft Entra ID.

## Snel Starten

```bash
# 1. Navigeer naar het project
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Meld je aan zodat keyless authenticatie een token kan krijgen
az login

# 3. Configureer de endpoint
#    - Als je `azd up` hebt uitgevoerd, is .env voor je geschreven (sla dit over).
#    - Anders kopieer je de template en stel je AZURE_OPENAI_ENDPOINT in:
cp .env.example .env

# 4. Voer de toepassing uit
mvn spring-boot:run
```

## Hoe Authenticatie Werkt

Dit voorbeeld authenticeert met **Microsoft Entra ID** — er is geen API-sleutel.

Wanneer alleen `spring.ai.azure.openai.endpoint` is ingesteld (en geen api-key), bouwt Spring AI de Azure OpenAI-client met [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). Die credential vindt automatisch een token uit je lokale `az login` sessie, of uit een beheerde identiteit wanneer het in Azure draait — dus dezelfde code werkt op beide plaatsen zonder wijzigingen.

## De Applicatie Uitvoeren

### Maven Gebruiken

```bash
mvn spring-boot:run
```

### VS Code Gebruiken

1. Open het project in VS Code
2. Druk op `F5` of gebruik het "Run and Debug" paneel
3. Selecteer de configuratie "Spring Boot-BasicChatApplication"

> **Opmerking**: De VS Code-configuratie laadt automatisch je .env-bestand

### Verwachte Uitvoer

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

## Configuratie Referentie

### Omgevingsvariabelen

| Variabele | Beschrijving | Verplicht | Voorbeeld |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | Foundry (Azure OpenAI) endpoint URL | Ja | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Chat model implementatienaam | Nee | `gpt-4o-mini` (standaard) |

> Er is **geen** API-sleutelvariabele — authenticatie is sleutelvrij (Microsoft Entra ID via `az login`).

### Spring Configuratie

Het `application.yml` bestand configureert:
- **Endpoint**: `${AZURE_OPENAI_ENDPOINT}` - Uit omgevingsvariabele
- **Deployment**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - Uit omgevingsvariabele met fallback
- **Auth**: sleutelvrij — geen `api-key` ingesteld, dus Spring AI gebruikt `DefaultAzureCredential`
- **Temperature**: `0.7` - Stuurt creativiteit (0.0 = deterministisch, 1.0 = creatief)
- **Max Tokens**: `500` - Maximale responslengte

## Probleemoplossing

### Veelvoorkomende Problemen

<details>
<summary><strong>Fout: 401 / "PermissionDenied" / token fouten</strong></summary>

- Voer `az login` uit — sleutelvrije authenticatie vereist een actieve aanmelding voor een token
- Controleer of je account de rol **Cognitive Services OpenAI User** heeft op de resource
- Als je net de rol hebt toegewezen, wacht een minuut tot deze is doorgevoerd
- Controleer of je in de juiste tenant/abonnement bent (`az account show`)
</details>

<details>
<summary><strong>Fout: "The endpoint is not valid" / verbindingsproblemen</strong></summary>

- Zorg dat `AZURE_OPENAI_ENDPOINT` de volledige basis-URL is (bijv. `https://your-resource.openai.azure.com/`)
- Controleer consistentie van eventuele schuine strepen aan het eind
- Controleer of het endpoint overeenkomt met je geprovisioneerde resource (`azd env get-values`)
</details>

<details>
<summary><strong>Fout: "The deployment was not found"</strong></summary>

- Controleer of `AZURE_OPENAI_DEPLOYMENT` overeenkomt met een implementatienaam in Azure
- Controleer dat het model succesvol is geïmplementeerd en actief is
- De standaard implementatienaam is `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: Omgevingsvariabelen laden niet</strong></summary>

- Zorg dat je `.env` bestand in de hoofdmap van het project staat (op gelijke hoogte met `pom.xml`)
- Probeer `mvn spring-boot:run` uit te voeren in de geïntegreerde terminal van VS Code
- Controleer of de Java-extensie van VS Code correct is geïnstalleerd
</details>

### Debug Modus

Om gedetailleerde logging in te schakelen, haal de commentaartekens weg bij deze regels in `application.yml`:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Volgende Stappen

**Setup Voltooid!** Ga door met je leertraject:

[Hoofdstuk 3: Core Generative AI Techniques](../../../03-CoreGenerativeAITechniques/README.md)

## Bronnen

- [Spring AI Azure OpenAI Documentatie](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Sleutelvrije authenticatie met Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry Portal](https://ai.azure.com/)
- [Azure AI Foundry Documentatie](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Disclaimer**:
Dit document is vertaald met behulp van de AI vertaaldienst [Co-op Translator](https://github.com/Azure/co-op-translator). Hoewel we streven naar nauwkeurigheid, dient u er rekening mee te houden dat geautomatiseerde vertalingen fouten of onnauwkeurigheden kunnen bevatten. Het originele document in de oorspronkelijke taal moet worden beschouwd als de gezaghebbende bron. Voor kritieke informatie wordt professionele menselijke vertaling aanbevolen. Wij zijn niet aansprakelijk voor eventuele misverstanden of verkeerde interpretaties die voortvloeien uit het gebruik van deze vertaling.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->