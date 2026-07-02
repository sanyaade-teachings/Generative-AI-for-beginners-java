# Opsætning af udviklingsmiljøet til Azure AI Foundry

> Denne guide opsætter **Azure AI Foundry**-modeller til Java AI-apps i dette kursus ved brug af **nøglefri** autentificering (Microsoft Entra ID) — ingen API-nøgler at administrere. Ny til værktøjerne? Start med [guiden til udviklingsmiljøet](./README.md).

Denne guide opsætter **Azure AI Foundry**-modeller til Java AI-apps i dette kursus. Du har to muligheder:

- **Mulighed A — Provision med `azd` + Bicep (anbefalet):** én kommando implementerer Foundry-kontoen og modeller som kode. Ingen portal-klik.
- **Mulighed B — Opret ressourcer manuelt** i Azure AI Foundry-portalen.

Begge valgmuligheder bruger **nøglefri autentificering** (Microsoft Entra ID) — der er ingen API-nøgler at kopiere eller lække.

## Indholdsfortegnelse

- [Hvad bliver oprettet](#hvad-bliver-oprettet)
- [Forudsætninger](#forudsætninger)
- [Mulighed A: Provision med azd + Bicep (anbefalet)](#option-a-provision-with-azd--bicep-recommended)
- [Mulighed B: Opret ressourcer manuelt](#mulighed-b-opret-ressourcer-manuelt)
- [Konfigurer dit miljø](#konfigurer-dit-miljø)
- [Test din opsætning](#test-din-opsætning)
- [Hvad nu?](#hvad-nu)
- [Ressourcer](#ressourcer)
- [Yderligere ressourcer](#yderligere-ressourcer)

## Hvad bliver oprettet

Bicep-skabelonerne i [`infra/`](../../../02-SetupDevEnvironment/infra) opretter:

- En **Azure AI Foundry**-konto (`Microsoft.CognitiveServices/accounts`, type `AIServices`) med et projekt
- En **chat**-udrulning — `gpt-4o-mini`
- En **embedding**-udrulning — `text-embedding-3-small` (bruges i senere kapitler)
- En **nøglefri rollefordeling** (`Cognitive Services OpenAI User`), så du logger ind med `az login` i stedet for at administrere nøgler

## Forudsætninger

- En [Azure-abonnement](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) og [Maven 3.9+](https://maven.apache.org/download.cgi)

## Mulighed A: Provision med azd + Bicep (anbefalet)

Fra mappen `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# Log ind (begge værktøjer)
azd auth login
az login

# Sørg for Foundry-kontoen + modeludrulninger
azd up
```

`azd` beder om et **miljønavn** (for eksempel `genai-java`) og en **region**. Vælg en region, hvor `gpt-4o-mini` og `text-embedding-3-small` er tilgængelige — for eksempel `eastus2` eller `swedencentral`.

Når provisioneringen er færdig, gør azd følgende:

1. Implementerer alt defineret i [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep).
2. Kører en post-provision hook, der skriver [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) med din endpoint og udrulningsnavne (ingen hemmeligheder).

> **Tip:** Kør `azd up` igen når som helst for at anvende ændringer. Kør `azd down` for at slette alt og stoppe forbruget.

For at se de genererede indstillinger:

```bash
azd env get-values
```

Spring nu videre til [Test din opsætning](#test-din-opsætning).

## Mulighed B: Opret ressourcer manuelt

Foretrækker du portalen? Opret ressourcerne manuelt:

1. Gå til [Azure AI Foundry-portalen](https://ai.azure.com/) og log ind.
2. **Opret et projekt** (dette opretter også en AI Foundry-ressource). Giv det et navn som `GenAIJava`.
3. I dit projekt, åbn **Models + endpoints** → **Deploy model** → **Deploy base model**.
4. Udrul **gpt-4o-mini** (udrulningsnavn `gpt-4o-mini`). Gentag for **text-embedding-3-small**, hvis du vil have embedding-eksemplerne.
5. Fra **Oversigt**, kopier **endpoint** (for eksempel `https://<resource>.openai.azure.com/`).
6. Giv dig selv nøglefri adgang: på ressourcen åbn **Access control (IAM)** → **Add role assignment** → tildel **Cognitive Services OpenAI User** til din konto.

> **Ikke helt i mål?** Se [Azure AI Foundry dokumentationen](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Konfigurer dit miljø

**Hvis du brugte Mulighed A (`azd up`)**, er din settings-fil allerede skrevet — der er ikke noget at konfigurere. Spring til [Test din opsætning](#test-din-opsætning).

**Hvis du brugte Mulighed B (manuel)**, opret `.env`-filen til eksemplet selv:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

Rediger `.env` med din endpoint (ingen nøgle — autentificering er nøglefri):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Sikkerhedsnote:** Der er ingen API-nøgle at gemme. Du autentificerer med Microsoft Entra ID via `az login` (lokalt) eller en managed identity (i Azure). `.env`-filen indeholder kun ikke-hemmelige indstillinger og er allerede dækket af `.gitignore`.

## Test din opsætning

Sørg for, at du er logget ind, så nøglefri autentificering kan hente et token, og kør så eksemplet:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # hvis du ikke allerede er logget ind
mvn clean spring-boot:run
```

Du skulle gerne se et svar fra `gpt-4o-mini`-modellen!

> **VS Code-brugere:** Tryk på `F5` for at køre. App’en indlæser din `.env` automatisk.

> **Fuldstændigt eksempel:** Se [Basic Chat med Azure AI Foundry-eksemplet](./examples/basic-chat-azure/README.md) for detaljer og fejlfinding.

## Hvad nu?

**Opsætningen er fuldført!** Du har nu:
- Azure AI Foundry med `gpt-4o-mini` og `text-embedding-3-small` udrullet
- Nøglefri autentificering (Microsoft Entra ID) — ingen nøgler at håndtere
- En lokal `.env` med din endpoint og udrulningsnavne
- Et Java-udviklingsmiljø klar til brug

**Fortsæt til** [Kapitel 3: Core Generative AI Techniques](../03-CoreGenerativeAITechniques/README.md) for at begynde at bygge AI-applikationer!

## Ressourcer

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Nøglefri autentificering med Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry Dokumentation](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI Dokumentation](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Yderligere ressourcer

- [Download VS Code](https://code.visualstudio.com/Download)
- [Hent Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Dev Container-konfiguration](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Ansvarsfraskrivelse**:
Dette dokument er blevet oversat ved hjælp af AI-oversættelsestjenesten [Co-op Translator](https://github.com/Azure/co-op-translator). Selvom vi bestræber os på nøjagtighed, skal du være opmærksom på, at automatiserede oversættelser kan indeholde fejl eller unøjagtigheder. Det originale dokument på dets oprindelige sprog bør betragtes som den autoritative kilde. For kritisk information anbefales professionel menneskelig oversættelse. Vi påtager os intet ansvar for misforståelser eller fejltolkninger, der opstår som følge af brugen af denne oversættelse.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->