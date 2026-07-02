# Opsætning af udviklingsmiljøet til Generative AI for Java

> **Hurtig start:** Provisionér dine AI-modeller på **Azure AI Foundry** som kode med Bicep + `azd` på få minutter — se [Azure AI Foundry Setup Guide](getting-started-azure-openai.md). Autentificering er **nøglefri** (Microsoft Entra ID), så der er ingen API-nøgler, du skal administrere.

## Det lærer du

- Opsæt et Java-udviklingsmiljø til AI-applikationer
- Vælg og konfigurer dit foretrukne udviklingsmiljø (cloud-first med Codespaces, lokalt dev container eller fuld lokal opsætning)
- Test din opsætning ved at oprette forbindelse til en Azure AI Foundry-model

## Indholdsfortegnelse

- [Det lærer du](#det-lærer-du)
- [Introduktion](#introduktion)
- [Trin 1: Opsæt dit udviklingsmiljø](#trin-1-opsæt-dit-udviklingsmiljø)
  - [Mulighed A: GitHub Codespaces (anbefalet)](#mulighed-a-github-codespaces-anbefalet)
  - [Mulighed B: Lokal Dev Container](#mulighed-b-lokal-dev-container)
  - [Mulighed C: Brug din eksisterende lokale installation](#mulighed-c-brug-din-eksisterende-lokale-installation)
- [Trin 2: Provisionér Azure AI Foundry](#trin-2-provisionér-azure-ai-foundry)
- [Trin 3: Test din opsætning](#trin-3-test-din-opsætning)
- [Fejlfinding](#fejlfinding)
- [Opsummering](#opsummering)
- [Næste skridt](#næste-skridt)

## Introduktion

Dette kapitel guider dig gennem opsætning af et udviklingsmiljø. Vi bruger **Azure AI Foundry** til modellerne gennem hele kurset. Du provisionerer modellerne som kode med Bicep og Azure Developer CLI (`azd`), og opretter derefter forbindelse med **nøglefri autentificering** (Microsoft Entra ID) — ingen API-nøgler, der skal kopieres eller kan lækkes.

**Ingen lokal opsætning nødvendig!** Du kan bruge GitHub Codespaces, som leverer et komplet udviklingsmiljø i din browser, og provisionere Foundry derfra.

Vi bruger **Azure AI Foundry** til dette kursus, fordi det er:
- **Provisioneret som kode** — en enkelt `azd up` implementerer konto og model-implementeringer
- **Nøglefri** — autentificer med din Azure-login eller en administreret identitet
- **Produktionsklart** — samme kode kører lokalt og i Azure
- **Fleksibelt** — skift modeller ved at ændre et deploymentsnavn, ikke din kode

> **Bemærk**: Azure AI Foundry-implementeringer faktureres pr. token (betal efter forbrug). Se [Azure AI Foundry setup guide](getting-started-azure-openai.md) for provisionering, region og prisdetaljer.


## Trin 1: Opsæt dit udviklingsmiljø

<a name="quick-start-cloud"></a>

Vi har lavet en forkonfigureret udviklingscontainer for at minimere opsætningstid og sikre, at du har alle nødvendige værktøjer til dette Generative AI for Java kursus. Vælg din foretrukne udviklingstilgang:

### Valgmuligheder for miljøopsætning:

#### Mulighed A: GitHub Codespaces (anbefalet)

**Begynd at kode på 2 minutter – ingen lokal opsætning nødvendig!**

1. Fork dette repository til din GitHub-konto  
   > **Bemærk**: Hvis du vil redigere den grundlæggende konfiguration, så kig på [Dev Container Configuration](../../../.devcontainer/devcontainer.json)  
2. Klik på **Code** → fanen **Codespaces** → **...** → **New with options...**  
3. Brug standarderne – dette vælger **Dev container configuration**: **Generative AI Java Development Environment** custom devcontainer oprettet til dette kursus  
4. Klik på **Create codespace**  
5. Vent ca. 2 minutter, mens miljøet bliver klar  
6. Fortsæt til [Trin 2: Provisionér Azure AI Foundry](#trin-2-provisionér-azure-ai-foundry)

<img src="../../../translated_images/da/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: Codespaces undermenu" width="50%">

<img src="../../../translated_images/da/image.833552b62eee7766.webp" alt="Screenshot: New with options" width="50%">

<img src="../../../translated_images/da/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: Opret codespace muligheder" width="50%">


> **Fordele ved Codespaces**:  
> - Ingen lokal installation nødvendig  
> - Virker på alle enheder med en browser  
> - Forudkonfigureret med alle værktøjer og afhængigheder  
> - Gratis 60 timer pr. måned for personlige konti  
> - Konsistent miljø for alle deltagere  

#### Mulighed B: Lokal Dev Container

**For udviklere, der foretrækker lokal udvikling med Docker**

1. Fork og klon dette repository til din lokale maskine  
   > **Bemærk**: Hvis du vil redigere den grundlæggende konfiguration, så kig på [Dev Container Configuration](../../../.devcontainer/devcontainer.json)  
2. Installer [Docker Desktop](https://www.docker.com/products/docker-desktop/) og [VS Code](https://code.visualstudio.com/)  
3. Installer [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) i VS Code  
4. Åbn repository-mappen i VS Code  
5. Når du bliver spurgt, klik på **Reopen in Container** (eller brug `Ctrl+Shift+P` → "Dev Containers: Reopen in Container")  
6. Vent på, at containeren bygger og starter  
7. Fortsæt til [Trin 2: Provisionér Azure AI Foundry](#trin-2-provisionér-azure-ai-foundry)

<img src="../../../translated_images/da/devcontainer.21126c9d6de64494.webp" alt="Screenshot: Dev container opsætning" width="50%">

<img src="../../../translated_images/da/image-3.bf93d533bbc84268.webp" alt="Screenshot: Dev container build færdig" width="50%">

#### Mulighed C: Brug din eksisterende lokale installation

**For udviklere med eksisterende Java-miljøer**

Forudsætninger:  
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html)  
- [Maven 3.9+](https://maven.apache.org/download.cgi)  
- [VS Code](https://code.visualstudio.com) eller dit foretrukne IDE  

Trin:  
1. Klon dette repository til din lokale maskine  
2. Åbn projektet i dit IDE  
3. Fortsæt til [Trin 2: Provisionér Azure AI Foundry](#trin-2-provisionér-azure-ai-foundry)

> **Pro tip**: Hvis du har en maskine med lav specifikation, men ønsker VS Code lokalt, så brug GitHub Codespaces! Du kan forbinde din lokale VS Code til en cloud-hosted Codespace for at få det bedste af begge verdener.

<img src="../../../translated_images/da/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: oprettet lokal devcontainer instans" width="50%">


## Trin 2: Provisionér Azure AI Foundry

Deploy kursussets AI-modeller til Azure AI Foundry som kode. Fra repository-roden:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```
  
`azd` spørger om et miljønavn og en region, provisionerer en Azure AI Foundry-konto med `gpt-4o-mini` og `text-embedding-3-small` deployment, og skriver endpointet til eksemplets `.env` — alt med **nøglefri** autentificering (ingen API-nøgler).

> **Fuld gennemgang:** Se [Azure AI Foundry Setup Guide](getting-started-azure-openai.md) for forudsætninger, en manuel (portal) alternativ metode, regionsvejledning og omkostnings-/rydde-anvisninger.

## Trin 3: Test din opsætning

Når dine Foundry-modeller er provisioneret, test forbindelsen med eksempelsappen i [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure).

1. Åbn terminalen i dit udviklingsmiljø.  
2. Naviger til eksemplet:  
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
  
3. Sørg for, at du er logget ind (nøglefri auth kræver et token):  
   ```bash
   az login
   ```
  
   > Hvis du har kørt `azd up`, blev `.env`-filen med dit endpoint allerede skrevet til dig.  
4. Kør applikationen:  
   ```bash
   mvn clean spring-boot:run
   ```
  
Du bør se et svar fra `gpt-4o-mini` modellen.

### Forståelse af eksempel-koden

Eksemplet under `examples/basic-chat-azure` er en Spring Boot-app, der bruger **Spring AI** til at oprette forbindelse til Azure AI Foundry med nøglefri autentificering.

**Hvad denne kode gør:**  
- **Opretter forbindelse** til Azure AI Foundry med dit Azure-login (Microsoft Entra ID) — ingen API-nøgle  
- **Sender** et prompt til `gpt-4o-mini` modellen  
- **Modtager** og viser AI'ens svar  
- **Validerer**, at din opsætning virker korrekt  

**Nøgleafhængighed** (i `pom.xml`):  
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```
  
**Konfiguration** (`application.yml`):  
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
  
## Opsummering

Fantastisk! Du har nu alt sat op:  

- Provisioneret Azure AI Foundry-modeller som kode med Bicep + `azd`  
- Fået dit Java-udviklingsmiljø til at køre (uanset om det er Codespaces, dev containers eller lokalt)  
- Forbundet til Azure AI Foundry med nøglefri autentificering (Microsoft Entra ID) — ingen API-nøgler  
- Testet, at det hele virker med et simpelt eksempel, der taler med din model  

## Næste skridt

[Kapitel 3: Core Generative AI Techniques](../03-CoreGenerativeAITechniques/README.md)

## Fejlfinding

Har du problemer? Her er almindelige problemer og løsninger:

- **Autentificering fejler (401/403)?**  
  - Kør `az login` — autentificering er nøglefri, så du skal være logget ind  
  - Bekræft, at din konto har rollen **Cognitive Services OpenAI User** på ressourcen  
  - Hvis du lige har provisioneret, vent et øjeblik, så rollefordelingen kan slå igennem  

- **Maven ikke fundet?**  
  - Hvis du bruger dev containers/Codespaces, bør Maven være forudinstalleret  
  - Til lokal opsætning, sikre at Java 21+ og Maven 3.9+ er installeret  
  - Prøv `mvn --version` for at tjekke installation  

- **`azd` ikke fundet eller provisioning fejler?**  
  - Installer [Azure Developer CLI](https://aka.ms/azure-dev/install) og kør `azd auth login`  
  - Vælg en region hvor `gpt-4o-mini` er tilgængelig (fx `eastus2`)  
  - Se [Azure AI Foundry setup guide](getting-started-azure-openai.md) for detaljer  

- **Dev container starter ikke?**  
  - Sørg for Docker Desktop kører (for lokal udvikling)  
  - Prøv at genopbygge containeren: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"  

- **Kompileringsfejl i applikationen?**  
  - Sørg for du er i den rigtige mappe: `02-SetupDevEnvironment/examples/basic-chat-azure`  
  - Prøv at clean og compile: `mvn clean compile`  

> **Brug for hjælp?**: Har du stadig problemer? Opret en issue i repositoryet, så hjælper vi dig.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Ansvarsfraskrivelse**:
Dette dokument er blevet oversat ved hjælp af AI-oversættelsestjenesten [Co-op Translator](https://github.com/Azure/co-op-translator). Selvom vi bestræber os på nøjagtighed, skal du være opmærksom på, at automatiserede oversættelser kan indeholde fejl eller unøjagtigheder. Det originale dokument på dets oprindelige sprog bør betragtes som den autoritative kilde. For kritisk information anbefales professionel menneskelig oversættelse. Vi påtager os intet ansvar for misforståelser eller fejltolkninger, der opstår som følge af brugen af denne oversættelse.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->