# Installera utvecklingsmiljön för Azure AI Foundry

> Denna guide ställer in **Azure AI Foundry**-modeller för Java AI-apparna i denna kurs, med **nyckellös** autentisering (Microsoft Entra ID) — inga API-nycklar att hantera. Ny med verktygen? Börja med [guider för utvecklingsmiljön](./README.md).

Denna guide ställer in **Azure AI Foundry**-modeller för Java AI-apparna i denna kurs. Du har två vägar:

- **Alternativ A — Provisionera med `azd` + Bicep (rekommenderas):** ett kommando distribuerar Foundry-kontot och modellerna som kod. Ingen klickning i portalen.
- **Alternativ B — Skapa resurser manuellt** i Azure AI Foundry-portalen.

Båda vägarna använder **nyckellös autentisering** (Microsoft Entra ID) — det finns inga API-nycklar att kopiera eller läcka.

## Innehållsförteckning

- [Vad som skapas](#vad-som-skapas)
- [Förutsättningar](#förutsättningar)
- [Alternativ A: Provisionering med azd + Bicep (Rekommenderas)](#option-a-provision-with-azd--bicep-recommended)
- [Alternativ B: Skapa resurser manuellt](#alternativ-b-skapa-resurser-manuellt)
- [Konfigurera din miljö](#konfigurera-din-miljö)
- [Testa din installation](#testa-din-installation)
- [Vad händer härnäst?](#vad-händer-härnäst)
- [Resurser](#resurser)
- [Ytterligare resurser](#ytterligare-resurser)

## Vad som skapas

Bicep-mallarna i [`infra/`](../../../02-SetupDevEnvironment/infra) provisionerar:

- Ett **Azure AI Foundry**-konto (`Microsoft.CognitiveServices/accounts`, sort `AIServices`) med ett projekt
- En **chat**-distribution — `gpt-4o-mini`
- En **embedding**-distribution — `text-embedding-3-small` (används i senare kapitel)
- En **nyckellös rolltilldelning** (`Cognitive Services OpenAI User`) så att du loggar in med `az login` istället för att hantera nycklar

## Förutsättningar

- Ett [Azure-abonnemang](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) och [Maven 3.9+](https://maven.apache.org/download.cgi)

## Alternativ A: Provisionering med azd + Bicep (Rekommenderas)

Från mappen `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# Logga in (båda verktygen)
azd auth login
az login

# Tilldela Foundry-kontot + modellutplaceringar
azd up
```

`azd` frågar efter ett **miljönamn** (t.ex. `genai-java`) och en **region**. Välj en region där `gpt-4o-mini` och `text-embedding-3-small` finns tillgängliga — till exempel `eastus2` eller `swedencentral`.

När provisioneringen är klar, gör azd följande:

1. Distribuerar allt som definieras i [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep).
2. Kör en post-provision hook som skriver [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) med din endpoint och distributionsnamn (inga hemligheter).

> **Tips:** Kör `azd up` igen när som helst för att tillämpa ändringar. Kör `azd down` för att ta bort allt och sluta generera kostnader.

För att se de genererade inställningarna:

```bash
azd env get-values
```

Hoppa nu till [Testa din installation](#testa-din-installation).

## Alternativ B: Skapa resurser manuellt

Föredrar du portalen? Skapa resurserna manuellt:

1. Gå till [Azure AI Foundry-portalen](https://ai.azure.com/) och logga in.
2. **Skapa ett projekt** (det skapar också en AI Foundry-resurs). Ge det ett namn som `GenAIJava`.
3. I ditt projekt, öppna **Models + endpoints** → **Deploy model** → **Deploy base model**.
4. Distribuera **gpt-4o-mini** (distributionsnamn `gpt-4o-mini`). Upprepa för **text-embedding-3-small** om du vill ha embedding-exemplen.
5. Från **Översikt**, kopiera **endpoint** (t.ex. `https://<resource>.openai.azure.com/`).
6. Ge dig själv nyckellös åtkomst: på resursen, öppna **Access control (IAM)** → **Add role assignment** → tilldela **Cognitive Services OpenAI User** till ditt konto.

> **Har du fortfarande problem?** Se [dokumentationen för Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Konfigurera din miljö

**Om du använde Alternativ A (`azd up`)** är din inställningsfil redan skapad — inget att konfigurera. Hoppa till [Testa din installation](#testa-din-installation).

**Om du använde Alternativ B (manuellt)**, skapa exempelns `.env`-fil själv:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

Redigera `.env` med din endpoint (ingen nyckel — autentisering är nyckellös):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Säkerhetsnotis:** Det finns ingen API-nyckel att lagra. Du autentiserar med Microsoft Entra ID via `az login` (lokalt) eller en hanterad identitet (i Azure). `.env`-filen innehåller bara icke-hemliga inställningar och är redan undantagen i `.gitignore`.

## Testa din installation

Se till att du är inloggad så att nyckellös autentisering kan få en token, sedan kör exemplet:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # om du inte redan är inloggad
mvn clean spring-boot:run
```

Du bör se ett svar från `gpt-4o-mini`-modellen!

> **VS Code-användare:** Tryck på `F5` för att köra. Appen läser automatiskt in din `.env`.

> **Fullständigt exempel:** Se [Basic Chat with Azure AI Foundry-exemplet](./examples/basic-chat-azure/README.md) för detaljer och felsökning.

## Vad händer härnäst?

**Installation klar!** Nu har du:
- Azure AI Foundry med `gpt-4o-mini` och `text-embedding-3-small` distribuerade
- Nyckellös autentisering (Microsoft Entra ID) — inga nycklar att hantera
- En lokal `.env` med din endpoint och distributionsnamn
- En Java-utvecklingsmiljö redo att använda

**Fortsätt till** [Kapitel 3: Kärntekniker för generativ AI](../03-CoreGenerativeAITechniques/README.md) för att börja bygga AI-applikationer!

## Resurser

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Nyckellös autentisering med Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry-dokumentation](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI-dokumentation](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Ytterligare resurser

- [Ladda ner VS Code](https://code.visualstudio.com/Download)
- [Skaffa Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Dev Container-konfiguration](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Ansvarsfriskrivning**:
Detta dokument har översatts med hjälp av AI-översättningstjänsten [Co-op Translator](https://github.com/Azure/co-op-translator). Även om vi strävar efter noggrannhet, var vänlig notera att automatiska översättningar kan innehålla fel eller brister. Det ursprungliga dokumentet på dess modersmål bör betraktas som den auktoritativa källan. För kritisk information rekommenderas professionell mänsklig översättning. Vi ansvarar inte för några missförstånd eller feltolkningar som uppstår till följd av användningen av denna översättning.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->