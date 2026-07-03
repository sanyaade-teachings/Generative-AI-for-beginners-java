# Sette opp utviklingsmiljøet for Generativ AI for Java

> **Rask start:** Provisionér AI-modellene dine på **Azure AI Foundry** som kode med Bicep + `azd` på noen få minutter — se [Oppsettveiledning for Azure AI Foundry](getting-started-azure-openai.md). Autentisering er **nøkkelfri** (Microsoft Entra ID), så det er ingen API-nøkler å håndtere.

## Hva du vil lære

- Sette opp et Java-utviklingsmiljø for AI-applikasjoner
- Velge og konfigurere ditt foretrukne utviklingsmiljø (sky-først med Codespaces, lokal dev container eller full lokal oppsett)
- Teste oppsettet ditt ved å koble til en Azure AI Foundry-modell

## Innholdsfortegnelse

- [Hva du vil lære](#hva-du-vil-lære)
- [Introduksjon](#introduksjon)
- [Steg 1: Sett opp utviklingsmiljøet ditt](#steg-1-sett-opp-utviklingsmiljøet-ditt)
  - [Alternativ A: GitHub Codespaces (anbefalt)](#alternativ-a-github-codespaces-anbefalt)
  - [Alternativ B: Lokal dev container](#alternativ-b-lokal-dev-container)
  - [Alternativ C: Bruk din eksisterende lokale installasjon](#alternativ-c-bruk-din-eksisterende-lokale-installasjon)
- [Steg 2: Provision Azure AI Foundry](#steg-2-provision-azure-ai-foundry)
- [Steg 3: Test oppsettet ditt](#steg-3-test-oppsettet-ditt)
- [Feilsøking](#feilsøking)
- [Oppsummering](#oppsummering)
- [Neste steg](#neste-steg)

## Introduksjon

Dette kapitlet vil guide deg gjennom hvordan du setter opp et utviklingsmiljø. Vi bruker **Azure AI Foundry** for modellene gjennom hele kurset. Du provisionerer modellene som kode med Bicep og Azure Developer CLI (`azd`), deretter kobler du til med **nøkkelfri autentisering** (Microsoft Entra ID) — ingen API-nøkler å kopiere eller lekke.

**Ingen lokal oppsett nødvendig!** Du kan bruke GitHub Codespaces, som tilbyr et fullt utviklingsmiljø i nettleseren din, og provisionere Foundry derfra.

Vi bruker **Azure AI Foundry** for dette kurset fordi det er:
- **Provisionert som kode** — ett `azd up` deployerer konto og modellimplementeringer
- **Nøkkelfri** — autentiser med Azure-pålogging eller en managed identity
- **Produsert for produksjon** — samme kode kjører lokalt og i Azure
- **Fleksibelt** — bytt modeller ved å endre et deploy-navn, ikke koden din

> **Merk**: Azure AI Foundry-implementeringer faktureres per token (betal etter bruk). Se [Oppsettveiledning for Azure AI Foundry](getting-started-azure-openai.md) for provisjonering, region og kostnadsdetaljer.


## Steg 1: Sett opp utviklingsmiljøet ditt

<a name="quick-start-cloud"></a>

Vi har laget en forhåndskonfigurert utviklingscontainer for å minimere oppsettstid og sikre at du har alle nødvendige verktøy for dette Generative AI for Java-kurset. Velg din foretrukne utviklingstilnærming:

### Miljøoppsettalternativer:

#### Alternativ A: GitHub Codespaces (anbefalt)

**Start koding på 2 minutter - ingen lokal oppsett nødvendig!**

1. Fork dette repositoryet til din GitHub-konto
   > **Merk**: Hvis du vil redigere grunnkonfigurasjonen, ta en titt på [Dev Container Configuration](../../../.devcontainer/devcontainer.json)
2. Klikk **Code** → fanen **Codespaces** → **...** → **Ny med alternativer...**
3. Bruk standardinnstillingene – dette vil velge **Dev container-konfigurasjonen**: **Generative AI Java Development Environment** tilpasset devcontainer laget for dette kurset
4. Klikk **Create codespace**
5. Vent ~2 minutter på at miljøet er klart
6. Fortsett til [Steg 2: Provision Azure AI Foundry](#steg-2-provision-azure-ai-foundry)

<img src="../../../translated_images/no/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: Codespaces submenu" width="50%">

<img src="../../../translated_images/no/image.833552b62eee7766.webp" alt="Screenshot: New with options" width="50%">

<img src="../../../translated_images/no/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: Create codespace options" width="50%">


> **Fordeler med Codespaces**:
> - Ingen lokal installasjon nødvendig
> - Fungerer på alle enheter med nettleser
> - Forhåndskonfigurert med alle verktøy og avhengigheter
> - Gratis 60 timer per måned for personlige kontoer
> - Konsistent miljø for alle brukere

#### Alternativ B: Lokal dev container

**For utviklere som foretrekker lokal utvikling med Docker**

1. Fork og klon dette repositoryet til din lokale maskin
   > **Merk**: Hvis du vil redigere grunnkonfigurasjonen, ta en titt på [Dev Container Configuration](../../../.devcontainer/devcontainer.json)
2. Installer [Docker Desktop](https://www.docker.com/products/docker-desktop/) og [VS Code](https://code.visualstudio.com/)
3. Installer [Dev Containers-utvidelsen](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) i VS Code
4. Åpne repository-mappen i VS Code
5. Når du blir spurt, klikk **Reopen in Container** (eller bruk `Ctrl+Shift+P` → "Dev Containers: Reopen in Container")
6. Vent til containeren bygges og starter
7. Fortsett til [Steg 2: Provision Azure AI Foundry](#steg-2-provision-azure-ai-foundry)

<img src="../../../translated_images/no/devcontainer.21126c9d6de64494.webp" alt="Screenshot: Dev container setup" width="50%">

<img src="../../../translated_images/no/image-3.bf93d533bbc84268.webp" alt="Screenshot: Dev container build complete" width="50%">

#### Alternativ C: Bruk din eksisterende lokale installasjon

**For utviklere med eksisterende Java-miljøer**

Forutsetninger:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html) 
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) eller din foretrukne IDE

Steg:
1. Klon dette repositoryet til din lokale maskin
2. Åpne prosjektet i din IDE
3. Fortsett til [Steg 2: Provision Azure AI Foundry](#steg-2-provision-azure-ai-foundry)

> **Proftips**: Hvis du har en maskin med lav spesifikasjon, men vil ha VS Code lokalt, bruk GitHub Codespaces! Du kan koble din lokale VS Code til en sky-hostet Codespace for det beste av begge verdener.

<img src="../../../translated_images/no/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: created local devcontainer instance" width="50%">


## Steg 2: Provision Azure AI Foundry

Deploy kursets AI-modeller til Azure AI Foundry som kode. Fra repo-rot:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```

`azd` ber om et miljønavn og region, provisjonerer en Azure AI Foundry-konto med `gpt-4o-mini` og `text-embedding-3-small` deployeringer, og skriver endepunktet inn i eksempelets `.env` — alt med **nøkkelfri** autentisering (ingen API-nøkler).

> **Full gjennomgang:** Se [Oppsettveiledning for Azure AI Foundry](getting-started-azure-openai.md) for forutsetninger, et manuelt (portal) alternativ, regionanbefaling, og kostnads-/oppryddingsnotater.

## Steg 3: Test oppsettet ditt

Når Foundry-modellene dine er provisionert, test tilkoblingen med eksempelappen i [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure).

1. Åpne terminalen i utviklingsmiljøet ditt.
2. Naviger til eksempelet:
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
3. Sørg for at du er innlogget (nøkkelfri autentisering krever token):
   ```bash
   az login
   ```
   > Hvis du kjørte `azd up`, ble `.env`-filen med endepunktet allerede skrevet for deg.
4. Kjør applikasjonen:
   ```bash
   mvn clean spring-boot:run
   ```

Du bør se et svar fra `gpt-4o-mini`-modellen.

### Forstå eksempelkoden

Eksemplet under `examples/basic-chat-azure` er en Spring Boot-app som bruker **Spring AI** for å koble til Azure AI Foundry med nøkkelfri autentisering.

**Dette gjør koden:**
- **Kobler til** Azure AI Foundry med din Azure-pålogging (Microsoft Entra ID) — ingen API-nøkkel
- **Sender** en prompt til `gpt-4o-mini`-modellen
- **Mottar** og viser AI-svaret
- **Validerer** at oppsettet ditt fungerer riktig

**Nøkkelavhengighet** (i `pom.xml`):
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```

**Konfigurasjon** (`application.yml`):
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

## Oppsummering

Flott! Nå har du alt satt opp:

- Provisionert Azure AI Foundry-modeller som kode med Bicep + `azd`
- Fått Java-utviklingsmiljøet ditt i gang (enten Codespaces, dev containers eller lokalt)
- Knyttet til Azure AI Foundry med nøkkelfri autentisering (Microsoft Entra ID) — ingen API-nøkler
- Testet at alt fungerer med et enkelt eksempel som kommuniserer med modellen din

## Neste steg

[Kapittel 3: Kjerne teknikker for generativ AI](../03-CoreGenerativeAITechniques/README.md)

## Feilsøking

Har du problemer? Her er vanlige problemer og løsninger:

- **Autentisering feiler (401/403)?** 
  - Kjør `az login` — autentisering er nøkkelfri, så du må være logget inn
  - Bekreft at kontoen din har rollen **Cognitive Services OpenAI User** på ressursen
  - Hvis du nettopp provisjonerte, vent ett minutt på at rolle-tilordningen sprer seg

- **Maven ikke funnet?** 
  - Bruker du dev containers/Codespaces, skal Maven være forhåndsinstallert
  - For lokalt oppsett, sørg for at Java 21+ og Maven 3.9+ er installert
  - Prøv `mvn --version` for å sjekke installasjonen

- **`azd` ikke funnet eller provisjonering feiler?** 
  - Installer [Azure Developer CLI](https://aka.ms/azure-dev/install) og kjør `azd auth login`
  - Velg en region hvor `gpt-4o-mini` er tilgjengelig (f.eks. `eastus2`)
  - Se [Oppsettveiledning for Azure AI Foundry](getting-started-azure-openai.md) for detaljer

- **Dev container starter ikke?** 
  - Sørg for at Docker Desktop kjører (lokal utvikling)
  - Prøv å bygge containeren på nytt: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **Kompileringsfeil i applikasjonen?**
  - Sørg for at du er i riktig katalog: `02-SetupDevEnvironment/examples/basic-chat-azure`
  - Prøv å rense og bygge på nytt: `mvn clean compile`

> **Trenger du hjelp?**: Fortsatt problemer? Åpne et issue i repositoryen så hjelper vi deg.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Ansvarsfraskrivelse**:
Dette dokumentet er oversatt ved hjelp av AI-oversettelsestjenesten [Co-op Translator](https://github.com/Azure/co-op-translator). Selv om vi streber etter nøyaktighet, vær oppmerksom på at automatiske oversettelser kan inneholde feil eller unøyaktigheter. Det opprinnelige dokumentet på originalspråket skal betraktes som den autoritative kilden. For kritisk informasjon anbefales profesjonell menneskelig oversettelse. Vi er ikke ansvarlige for eventuelle misforståelser eller feiltolkninger som oppstår ved bruk av denne oversettelsen.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->