# Het ontwikkelomgeving instellen voor Azure AI Foundry

> Deze handleiding stelt **Azure AI Foundry**-modellen in voor de Java AI-apps in deze cursus, met **keyless** authenticatie (Microsoft Entra ID) — geen API-sleutels om te beheren. Nieuw met de tooling? Begin met de [ontwikkelomgevinghandleiding](./README.md).

Deze handleiding stelt **Azure AI Foundry**-modellen in voor de Java AI-apps in deze cursus. Je hebt twee opties:

- **Optie A — Provisioneren met `azd` + Bicep (aanbevolen):** één opdracht zet het Foundry-account en modellen als code uit. Geen klikken in de portal.
- **Optie B — Bronnen handmatig aanmaken** in de Azure AI Foundry-portal.

Beide opties gebruiken **keyless authenticatie** (Microsoft Entra ID) — er zijn geen API-sleutels om te kopiëren of lekken.

## Inhoudsopgave

- [Wat wordt aangemaakt](#wat-wordt-aangemaakt)
- [Vereisten](#vereisten)
- [Optie A: Provisioneren met azd + Bicep (Aanbevolen)](#optie-a-provisioneren-met-azd--bicep-aanbevolen)
- [Optie B: Bronnen handmatig aanmaken](#optie-b-bronnen-handmatig-aanmaken)
- [Je omgeving configureren](#je-omgeving-configureren)
- [Je setup testen](#je-setup-testen)
- [Wat nu?](#wat-nu)
- [Bronnen](#bronnen)
- [Aanvullende bronnen](#aanvullende-bronnen)

## Wat wordt aangemaakt

De Bicep-sjablonen in [`infra/`](../../../02-SetupDevEnvironment/infra) maken aan:

- Een **Azure AI Foundry**-account (`Microsoft.CognitiveServices/accounts`, soort `AIServices`) met een project
- Een **chat**-implementatie — `gpt-4o-mini`
- Een **embedding**-implementatie — `text-embedding-3-small` (gebruikt in latere hoofdstukken)
- Een **keyless roltoewijzing** (`Cognitive Services OpenAI User`) zodat je inlogt met `az login` in plaats van sleutels te beheren

## Vereisten

- Een [Azure-abonnement](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) en [Maven 3.9+](https://maven.apache.org/download.cgi)

## Optie A: Provisioneren met azd + Bicep (Aanbevolen)

Vanaf de map `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# Aanmelden (beide tools)
azd auth login
az login

# Voorzie het Foundry-account + modelimplementaties
azd up
```

`azd` vraagt om een **omgevingsnaam** (bijvoorbeeld `genai-java`) en een **regio**. Kies een regio waar `gpt-4o-mini` en `text-embedding-3-small` beschikbaar zijn — bijvoorbeeld `eastus2` of `swedencentral`.

Als het provisioneren klaar is, doet azd het volgende:

1. Zet alles wat gedefinieerd is in [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep) uit.
2. Voert een post-provisioning hook uit die [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) schrijft met je endpoint en implementatienamen (geen geheimen).

> **Tip:** Voer `azd up` opnieuw uit om wijzigingen toe te passen. Voer `azd down` uit om alles te verwijderen en kosten te stoppen.

Om de gegenereerde instellingen te zien:

```bash
azd env get-values
```

Ga nu verder naar [Je setup testen](#je-setup-testen).

## Optie B: Bronnen handmatig aanmaken

Gebruik je liever de portal? Maak de bronnen handmatig aan:

1. Ga naar de [Azure AI Foundry portal](https://ai.azure.com/) en log in.
2. **Maak een project aan** (dit maakt ook een AI Foundry-resource aan). Geef het een naam zoals `GenAIJava`.
3. Open in je project **Modellen + endpoints** → **Model implementeren** → **Basis model implementeren**.
4. Implementeer **gpt-4o-mini** (implementatienaam `gpt-4o-mini`). Herhaal voor **text-embedding-3-small** als je de embedding voorbeelden wilt.
5. Kopieer vanuit **Overzicht** de **endpoint** (bijvoorbeeld `https://<resource>.openai.azure.com/`).
6. Geef jezelf keyless toegang: open bij de resource **Toegangsbeheer (IAM)** → **Roltoewijzing toevoegen** → wijs **Cognitive Services OpenAI User** toe aan je account.

> **Nog steeds problemen?** Zie de [Azure AI Foundry documentatie](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Je omgeving configureren

**Als je Optie A (`azd up`) gebruikt hebt**, is je instellingenbestand al geschreven — er is niets om te configureren. Ga door naar [Je setup testen](#je-setup-testen).

**Als je Optie B (handmatig) gebruikt hebt**, maak dan zelf het voorbeeld `.env`-bestand aan:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

Pas `.env` aan met je endpoint (geen sleutel — authenticatie is keyless):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Beveiligingsnotitie:** Er is geen API-sleutel om op te slaan. Je logt in met Microsoft Entra ID via `az login` (lokaal) of een managed identity (in Azure). Het `.env`-bestand bevat alleen niet-geheime instellingen en wordt al genegeerd door `.gitignore`.

## Je setup testen

Zorg dat je bent ingelogd, zodat keyless authenticatie een token kan ophalen, en voer dan het voorbeeld uit:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # als je nog niet bent ingelogd
mvn clean spring-boot:run
```

Je zou een antwoord van het `gpt-4o-mini`-model moeten zien!

> **VS Code-gebruikers:** Druk op `F5` om uit te voeren. De app laadt je `.env` automatisch.

> **Volledig voorbeeld:** Zie het [Basic Chat met Azure AI Foundry voorbeeld](./examples/basic-chat-azure/README.md) voor details en probleemoplossing.

## Wat nu?

**Setup compleet!** Je hebt nu:
- Azure AI Foundry met `gpt-4o-mini` en `text-embedding-3-small` uitgerold
- Keyless authenticatie (Microsoft Entra ID) — geen sleutels om te beheren
- Een lokaal `.env` met je endpoint en implementatienamen
- Een Java-ontwikkelomgeving klaar voor gebruik

**Ga verder naar** [Hoofdstuk 3: Core Generative AI Techniques](../03-CoreGenerativeAITechniques/README.md) om AI-applicaties te bouwen!

## Bronnen

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Keyless authenticatie met Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry-documentatie](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI-documentatie](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Aanvullende bronnen

- [Download VS Code](https://code.visualstudio.com/Download)
- [Download Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Dev Container-configuratie](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Disclaimer**:
Dit document is vertaald met behulp van de AI vertaaldienst [Co-op Translator](https://github.com/Azure/co-op-translator). Hoewel we streven naar nauwkeurigheid, dient u er rekening mee te houden dat geautomatiseerde vertalingen fouten of onnauwkeurigheden kunnen bevatten. Het originele document in de oorspronkelijke taal moet worden beschouwd als de gezaghebbende bron. Voor kritieke informatie wordt professionele menselijke vertaling aanbevolen. Wij zijn niet aansprakelijk voor eventuele misverstanden of verkeerde interpretaties die voortvloeien uit het gebruik van deze vertaling.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->