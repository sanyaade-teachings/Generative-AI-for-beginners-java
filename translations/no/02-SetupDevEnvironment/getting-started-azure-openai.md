# Sette opp utviklingsmiljøet for Azure AI Foundry

> Denne guiden setter opp **Azure AI Foundry**-modeller for Java AI-appene i dette kurset, med **nøkkelfri** autentisering (Microsoft Entra ID) — ingen API-nøkler å administrere. Ny til verktøyene? Start med [guiden for utviklingsmiljøet](./README.md).

Denne guiden setter opp **Azure AI Foundry**-modeller for Java AI-appene i dette kurset. Du har to veier:

- **Alternativ A — Provisionering med `azd` + Bicep (anbefalt):** én kommando som deployerer Foundry-kontoen og modellene som kode. Ingen portal-klikk.
- **Alternativ B — Opprett ressurser manuelt** i Azure AI Foundry-portalen.

Begge veier bruker **nøkkelfri autentisering** (Microsoft Entra ID) — det finnes ingen API-nøkler å kopiere eller lekke.

## Innholdsfortegnelse

- [Hva blir opprettet](#hva-blir-opprettet)
- [Forutsetninger](#forutsetninger)
- [Alternativ A: Provisionering med azd + Bicep (anbefalt)](#option-a-provision-with-azd--bicep-recommended)
- [Alternativ B: Opprett ressurser manuelt](#alternativ-b-opprett-ressurser-manuelt)
- [Konfigurer miljøet ditt](#konfigurer-miljøet-ditt)
- [Test oppsettet ditt](#test-oppsettet-ditt)
- [Hva kommer nå?](#hva-kommer-nå)
- [Ressurser](#ressurser)
- [Ekstra ressurser](#ekstra-ressurser)

## Hva blir opprettet

Bicep-maler i [`infra/`](../../../02-SetupDevEnvironment/infra) provisjonerer:

- En **Azure AI Foundry**-konto (`Microsoft.CognitiveServices/accounts`, type `AIServices`) med et prosjekt
- En **chat**-deployering — `gpt-4o-mini`
- En **embedding**-deployering — `text-embedding-3-small` (brukes i senere kapitler)
- En **nøkkelfri rolle-tilordning** (`Cognitive Services OpenAI User`) slik at du logger inn med `az login` i stedet for å håndtere nøkler

## Forutsetninger

- Et [Azure-abonnement](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) og [Maven 3.9+](https://maven.apache.org/download.cgi)

## Alternativ A: Provisionering med azd + Bicep (anbefalt)

Fra mappen `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# Logg inn (begge verktøy)
azd auth login
az login

# Opprett Foundry-konto + modellutrullinger
azd up
```

`azd` spør om et **miljønavn** (for eksempel `genai-java`) og en **region**. Velg en region hvor `gpt-4o-mini` og `text-embedding-3-small` er tilgjengelige — for eksempel `eastus2` eller `swedencentral`.

Når provisjoneringen er ferdig, vil azd:

1. Deployere alt definert i [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep).
2. Kjøre en postprovisjons-hook som skriver [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) med din endepunkt- og deployeringsnavn (ingen hemmeligheter).

> **Tips:** Kjør `azd up` på nytt når som helst for å anvende endringer. Kjør `azd down` for å slette alt og stoppe kostnader.

For å se de genererte innstillingene:

```bash
azd env get-values
```

Gå nå direkte til [Test oppsettet ditt](#test-oppsettet-ditt).

## Alternativ B: Opprett ressurser manuelt

Foretrekker du portalen? Opprett ressursene manuelt:

1. Gå til [Azure AI Foundry-portalen](https://ai.azure.com/) og logg inn.
2. **Opprett et prosjekt** (dette skaper også en AI Foundry-ressurs). Gi det et navn som `GenAIJava`.
3. I prosjektet ditt, åpne **Modeller + endepunkter** → **Deploy modell** → **Deploy base-modell**.
4. Deploy **gpt-4o-mini** (deploy-navn `gpt-4o-mini`). Gjenta for **text-embedding-3-small** om du ønsker embedding-eksemplene.
5. Fra **Oversikt**, kopier **endepunktet** (for eksempel `https://<resource>.openai.azure.com/`).
6. Gi deg selv nøkkelfri tilgang: på ressursen, åpne **Access control (IAM)** → **Legg til rolle-tilordning** → tilordne **Cognitive Services OpenAI User** til kontoen din.

> **Fremdeles problemer?** Se [dokumentasjonen for Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Konfigurer miljøet ditt

**Hvis du brukte alternativ A (`azd up`)**, er innstillingsfilen allerede skrevet — det er ingenting å konfigurere. Gå videre til [Test oppsettet ditt](#test-oppsettet-ditt).

**Hvis du brukte alternativ B (manuelt)**, lag `.env`-filen for eksempelet selv:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

Rediger `.env` med ditt endepunkt (ingen nøkkel — autentisering er nøkkelfri):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Sikkerhetsnotat:** Det finnes ingen API-nøkkel å lagre. Du autentiserer med Microsoft Entra ID via `az login` (lokalt) eller en managed identity (i Azure). `.env`-filen inneholder kun ikke-hemmelige innstillinger og er allerede dekket av `.gitignore`.

## Test oppsettet ditt

Sørg for at du er logget inn slik at nøkkelfri autentisering kan hente en token, og kjør så eksempelet:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # hvis du ikke allerede er logget inn
mvn clean spring-boot:run
```

Du bør se et svar fra `gpt-4o-mini`-modellen!

> **VS Code-brukere:** Trykk `F5` for å kjøre. Appen laster `.env` automatisk.

> **Fullt eksempel:** Se [Basic Chat med Azure AI Foundry-eksempelet](./examples/basic-chat-azure/README.md) for detaljer og feilsøking.

## Hva kommer nå?

**Oppsettet er komplett!** Du har nå:
- Azure AI Foundry med `gpt-4o-mini` og `text-embedding-3-small` deployert
- Nøkkelfri autentisering (Microsoft Entra ID) — ingen nøkler å administrere
- En lokal `.env` med ditt endepunkt og deployeringsnavn
- Et Java-utviklingsmiljø klart til bruk

**Fortsett til** [Kapittel 3: Kjerne-teknikker for Generativ AI](../03-CoreGenerativeAITechniques/README.md) for å begynne å bygge AI-applikasjoner!

## Ressurser

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Nøkkelfri autentisering med Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry-dokumentasjon](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI dokumentasjon](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Ekstra ressurser

- [Last ned VS Code](https://code.visualstudio.com/Download)
- [Last ned Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Dev container-konfigurasjon](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Ansvarsfraskrivelse**:
Dette dokumentet er oversatt ved hjelp av AI-oversettelsestjenesten [Co-op Translator](https://github.com/Azure/co-op-translator). Selv om vi streber etter nøyaktighet, vær oppmerksom på at automatiske oversettelser kan inneholde feil eller unøyaktigheter. Det opprinnelige dokumentet på originalspråket skal betraktes som den autoritative kilden. For kritisk informasjon anbefales profesjonell menneskelig oversettelse. Vi er ikke ansvarlige for eventuelle misforståelser eller feiltolkninger som oppstår ved bruk av denne oversettelsen.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->