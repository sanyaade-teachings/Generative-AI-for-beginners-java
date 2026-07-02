# Het Ontwikkelomgeving Instellen voor Generative AI voor Java

> **Snelle Start:** Voorzie je AI-modellen op **Azure AI Foundry** als code met Bicep + `azd` binnen enkele minuten — zie de [Azure AI Foundry Setup Guide](getting-started-azure-openai.md). Authenticatie is **keyless** (Microsoft Entra ID), dus er zijn geen API-sleutels om te beheren.

## Wat Je Zal Leren

- Een Java-ontwikkelomgeving instellen voor AI-toepassingen
- Je voorkeursontwikkelomgeving kiezen en configureren (cloud-first met Codespaces, lokale dev-container of volledige lokale installatie)
- Je setup testen door verbinding te maken met een Azure AI Foundry-model

## Inhoudsopgave

- [Wat Je Zal Leren](#wat-je-zal-leren)
- [Introductie](#introductie)
- [Stap 1: Je Ontwikkelomgeving Instellen](#stap-1-je-ontwikkelomgeving-instellen)
  - [Optie A: GitHub Codespaces (Aanbevolen)](#optie-a-github-codespaces-aanbevolen)
  - [Optie B: Lokale Dev Container](#optie-b-lokale-dev-container)
  - [Optie C: Gebruik Je Bestaande Lokale Installatie](#optie-c-gebruik-je-bestaande-lokale-installatie)
- [Stap 2: Azure AI Foundry Voorzien](#stap-2-azure-ai-foundry-voorzien)
- [Stap 3: Je Setup Testen](#stap-3-je-setup-testen)
- [Probleemoplossing](#probleemoplossing)
- [Samenvatting](#samenvatting)
- [Volgende Stappen](#volgende-stappen)

## Introductie

Dit hoofdstuk begeleidt je bij het opzetten van een ontwikkelomgeving. We gebruiken gedurende deze cursus **Azure AI Foundry** voor de modellen. Je voorziet de modellen als code met Bicep en de Azure Developer CLI (`azd`), en maakt vervolgens verbinding met **keyless authenticatie** (Microsoft Entra ID) — geen API-sleutels om te kopiëren of lekken.

**Geen lokale setup vereist!** Je kunt GitHub Codespaces gebruiken, dat een volledige ontwikkelomgeving in je browser biedt, en Foundry daarvandaan voorzien.

We gebruiken **Azure AI Foundry** voor deze cursus omdat het:
- **Als code te voorzien is** — één `azd up` zet het account en de modeldeployments op
- **Keyless** is — authenticeren met je Azure-aanmelding of een beheerde identiteit
- **Productierijp** is — dezelfde code draait lokaal en in Azure
- **Flexibel** is — verwissel modellen door alleen een deployment-naam te wijzigen, niet je code

> **Opmerking**: Azure AI Foundry-deployments worden per token gefactureerd (pay-as-you-go). Zie de [Azure AI Foundry setup guide](getting-started-azure-openai.md) voor details over provisioning, regio en kosten.


## Stap 1: Je Ontwikkelomgeving Instellen

<a name="quick-start-cloud"></a>

We hebben een vooraf geconfigureerde ontwikkelcontainer gemaakt om de setup-tijd te minimaliseren en ervoor te zorgen dat je alle benodigde tools hebt voor deze Generative AI voor Java-cursus. Kies je voorkeursontwikkelwijze:

### Opties voor Omgevingssetup:

#### Optie A: GitHub Codespaces (Aanbevolen)

**Begin binnen 2 minuten met coderen — geen lokale setup nodig!**

1. Fork deze repository naar je GitHub-account
   > **Opmerking**: Wil je de basisconfiguratie aanpassen, kijk dan naar de [Dev Container Configuration](../../../.devcontainer/devcontainer.json)
2. Klik op **Code** → tabblad **Codespaces** → **...** → **New with options...**
3. Gebruik de standaardinstellingen – hiermee wordt de **Dev container-configuratie** geselecteerd: **Generative AI Java Development Environment** custom devcontainer die voor deze cursus is gemaakt
4. Klik op **Create codespace**
5. Wacht ongeveer 2 minuten tot de omgeving klaar is
6. Ga verder met [Stap 2: Azure AI Foundry Voorzien](#stap-2-azure-ai-foundry-voorzien)

<img src="../../../translated_images/nl/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: Codespaces submenu" width="50%">

<img src="../../../translated_images/nl/image.833552b62eee7766.webp" alt="Screenshot: New with options" width="50%">

<img src="../../../translated_images/nl/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: Create codespace options" width="50%">


> **Voordelen van Codespaces**:
> - Geen lokale installatie nodig
> - Werkt op elk apparaat met een browser
> - Vooraf geconfigureerd met alle tools en dependencies
> - Gratis 60 uur per maand voor persoonlijke accounts
> - Consistente omgeving voor alle cursisten

#### Optie B: Lokale Dev Container

**Voor ontwikkelaars die lokaal willen ontwikkelen met Docker**

1. Fork en clone deze repository naar je lokale machine
   > **Opmerking**: Wil je de basisconfiguratie aanpassen, kijk dan naar de [Dev Container Configuration](../../../.devcontainer/devcontainer.json)
2. Installeer [Docker Desktop](https://www.docker.com/products/docker-desktop/) en [VS Code](https://code.visualstudio.com/)
3. Installeer de [Dev Containers-extensie](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) in VS Code
4. Open de repositorymap in VS Code
5. Klik bij de prompt op **Reopen in Container** (of gebruik `Ctrl+Shift+P` → "Dev Containers: Reopen in Container")
6. Wacht tot de container gebouwd is en gestart is
7. Ga verder met [Stap 2: Azure AI Foundry Voorzien](#stap-2-azure-ai-foundry-voorzien)

<img src="../../../translated_images/nl/devcontainer.21126c9d6de64494.webp" alt="Screenshot: Dev container setup" width="50%">

<img src="../../../translated_images/nl/image-3.bf93d533bbc84268.webp" alt="Screenshot: Dev container build complete" width="50%">

#### Optie C: Gebruik Je Bestaande Lokale Installatie

**Voor ontwikkelaars met bestaande Java-omgevingen**

Vereisten:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html) 
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) of je favoriete IDE

Stappen:
1. Clone deze repository naar je lokale machine
2. Open het project in je IDE
3. Ga verder met [Stap 2: Azure AI Foundry Voorzien](#stap-2-azure-ai-foundry-voorzien)

> **Pro Tip**: Heb je een machine met lage specificaties maar wil je toch VS Code lokaal gebruiken, dan is GitHub Codespaces ideaal! Je kunt je lokale VS Code verbinden met een cloud-hosted Codespace voor het beste van twee werelden.

<img src="../../../translated_images/nl/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: created local devcontainer instance" width="50%">


## Stap 2: Azure AI Foundry Voorzien

Deploy de AI-modellen van de cursus naar Azure AI Foundry als code. Vanuit de root van de repository:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```

`azd` vraagt om een omgevingsnaam en regio, voorziet een Azure AI Foundry-account met `gpt-4o-mini` en `text-embedding-3-small` deployments, en schrijft het endpoint weg in de `.env` van het voorbeeld — allemaal met **keyless** authenticatie (geen API-sleutels).

> **Volledige walkthrough:** Zie de [Azure AI Foundry Setup Guide](getting-started-azure-openai.md) voor vereisten, een handmatige (portaal) alternatieve methode, regiogids en kosten/opschoningsnotities.

## Stap 3: Je Setup Testen

Zodra je Foundry-modellen zijn voorzien, test je de verbinding met de voorbeeldapp in [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure).

1. Open de terminal in je ontwikkelomgeving.
2. Navigeer naar het voorbeeld:
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
3. Zorg dat je bent aangemeld (keyless auth heeft een token nodig):
   ```bash
   az login
   ```
   > Als je `azd up` hebt uitgevoerd, is het `.env`-bestand met je endpoint al voor je geschreven.
4. Start de applicatie:
   ```bash
   mvn clean spring-boot:run
   ```

Je zou een reactie van het `gpt-4o-mini` model moeten zien.

### Het Begrijpen van de Voorbeeldcode

De example onder `examples/basic-chat-azure` is een Spring Boot-app die **Spring AI** gebruikt om met keyless authenticatie verbinding te maken met Azure AI Foundry.

**Wat deze code doet:**
- **Verbindt** met Azure AI Foundry met je Azure-aanmelding (Microsoft Entra ID) — geen API-sleutel nodig
- **Verstuurt** een prompt naar het `gpt-4o-mini` model
- **Ontvangt** en toont de AI-respons
- **Valideert** dat je setup correct functioneert

**Belangrijke Dependency** (in `pom.xml`):
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```

**Configuratie** (`application.yml`):
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

## Samenvatting

Geweldig! Je hebt nu alles ingesteld:

- Azure AI Foundry-modellen voorzien als code met Bicep + `azd`
- Je Java-ontwikkelomgeving draaiende gekregen (of dat nu Codespaces, dev containers of lokaal is)
- Verbonden met Azure AI Foundry met keyless authenticatie (Microsoft Entra ID) — geen API-sleutels
- Alles getest met een eenvoudig voorbeeld dat met je model communiceert

## Volgende Stappen

[Hoofdstuk 3: Kerntechnieken van Generative AI](../03-CoreGenerativeAITechniques/README.md)

## Probleemoplossing

Problemen? Hier veelvoorkomende problemen en oplossingen:

- **Authenticatie mislukt (401/403)?** 
  - Voer `az login` uit — authenticatie is keyless, je moet dus aangemeld zijn
  - Controleer of je account de rol **Cognitive Services OpenAI User** op de resource heeft
  - Als je net hebt voorzien, wacht een minuut voor roltoewijzing om door te komen

- **Maven niet gevonden?** 
  - Bij gebruik van dev containers/Codespaces is Maven vooraf geïnstalleerd
  - Voor lokale setup, zorg dat Java 21+ en Maven 3.9+ zijn geïnstalleerd
  - Probeer `mvn --version` om installatie te controleren

- **`azd` niet gevonden of provisioning mislukt?** 
  - Installeer de [Azure Developer CLI](https://aka.ms/azure-dev/install) en voer `azd auth login` uit
  - Kies een regio waar `gpt-4o-mini` beschikbaar is (bijv. `eastus2`)
  - Zie de [Azure AI Foundry setup guide](getting-started-azure-openai.md) voor details

- **Dev container start niet?** 
  - Zorg dat Docker Desktop draait (voor lokale ontwikkeling)
  - Probeer de container opnieuw te bouwen: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **Compilatiefouten in applicatie?**
  - Zorg dat je in de juiste directory bent: `02-SetupDevEnvironment/examples/basic-chat-azure`
  - Probeer schoon te maken en opnieuw te bouwen: `mvn clean compile`

> **Hulp nodig?**: Nog steeds problemen? Open een issue in de repository en we helpen je verder.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Disclaimer**:
Dit document is vertaald met behulp van de AI vertaaldienst [Co-op Translator](https://github.com/Azure/co-op-translator). Hoewel we streven naar nauwkeurigheid, dient u er rekening mee te houden dat geautomatiseerde vertalingen fouten of onnauwkeurigheden kunnen bevatten. Het originele document in de oorspronkelijke taal moet worden beschouwd als de gezaghebbende bron. Voor kritieke informatie wordt professionele menselijke vertaling aanbevolen. Wij zijn niet aansprakelijk voor eventuele misverstanden of verkeerde interpretaties die voortvloeien uit het gebruik van deze vertaling.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->