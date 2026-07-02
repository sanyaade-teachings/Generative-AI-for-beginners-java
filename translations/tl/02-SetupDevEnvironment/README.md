# Pag-setup ng Development Environment para sa Generative AI para sa Java

> **Mabilisang Simula:** Mag-provision ng iyong mga AI model sa **Azure AI Foundry** bilang code gamit ang Bicep + `azd` sa loob ng ilang minuto — tingnan ang [Azure AI Foundry Setup Guide](getting-started-azure-openai.md). Ang authentication ay **keyless** (Microsoft Entra ID), kaya walang API key na kailangang pamahalaan.

## Ano ang Malalaman Mo

- Mag-setup ng Java development environment para sa mga AI application
- Piliin at i-configure ang iyong nais na development environment (cloud-first gamit ang Codespaces, local dev container, o buong local setup)
- Subukan ang iyong setup sa pamamagitan ng pagkonekta sa Azure AI Foundry model

## Nilalaman ng Talahanayan

- [Ano ang Malalaman Mo](#ano-ang-malalaman-mo)
- [Panimula](#panimula)
- [Hakbang 1: I-setup ang Iyong Development Environment](#hakbang-1-i-setup-ang-iyong-development-environment)
  - [Opsyon A: GitHub Codespaces (Inirerekomenda)](#opsyon-a-github-codespaces-inirerekomenda)
  - [Opsyon B: Local Dev Container](#opsyon-b-local-dev-container)
  - [Opsyon C: Gamitin ang Iyong Kasalukuyang Lokal na Instalasyon](#opsyon-c-gamitin-ang-iyong-kasalukuyang-lokal-na-instalasyon)
- [Hakbang 2: Mag-provision ng Azure AI Foundry](#hakbang-2-mag-provision-ng-azure-ai-foundry)
- [Hakbang 3: Subukan ang Iyong Setup](#hakbang-3-subukan-ang-iyong-setup)
- [Pag-troubleshoot](#pag-troubleshoot)
- [Buod](#buod)
- [Mga Susunod na Hakbang](#mga-susunod-na-hakbang)

## Panimula

Ang kabanatang ito ay gagabay sa iyo sa pag-setup ng isang development environment. Gagamitin natin ang **Azure AI Foundry** para sa mga model sa kabuuan ng kursong ito. Mag-provision ka ng mga model bilang code gamit ang Bicep at ang Azure Developer CLI (`azd`), saka magkonekta gamit ang **keyless authentication** (Microsoft Entra ID) — walang mga API key na kailangang kopyahin o ipamahagi.

**Hindi kailangan ng lokal na setup!** Maaari mong gamitin ang GitHub Codespaces, na nagbibigay ng kumpletong development environment sa iyong browser, at mag-provision ng Foundry mula doon.

Ginagamit natin ang **Azure AI Foundry** para sa kursong ito dahil ito ay:
- **Provisioned bilang code** — isang `azd up` lang ang nagpapadala ng account at mga deployment ng model
- **Keyless** — mag-authenticate gamit ang iyong Azure sign-in o isang managed identity
- **Handa na para sa produksyon** — parehas na code ang tumatakbo nang lokal at sa Azure
- **Flexible** — palitan ang mga model sa pamamagitan ng pagpapalit ng pangalan ng deployment, hindi ng iyong code

> **Tandaan**: Sinisingil ang mga Azure AI Foundry deployment per token (pay-as-you-go). Tingnan ang [Azure AI Foundry setup guide](getting-started-azure-openai.md) para sa detalye ng provisioning, rehiyon, at gastos.

## Hakbang 1: I-setup ang Iyong Development Environment

<a name="quick-start-cloud"></a>

Nagawa naming isang preconfigured development container upang mabawasan ang oras ng setup at matiyak na mayroon kang lahat ng kinakailangang tools para sa kursong Generative AI para sa Java. Piliin ang gusto mong paraan ng pag-develop:

### Mga Opsyon para sa Setup ng Environment:

#### Opsyon A: GitHub Codespaces (Inirerekomenda)

**Magsimulang mag-code sa loob ng 2 minuto - walang kailangang lokal na setup!**

1. I-fork ang repositoryong ito sa iyong GitHub account
   > **Tandaan**: Kung nais mong i-edit ang basic config, tingnan ang [Dev Container Configuration](../../../.devcontainer/devcontainer.json)
2. I-click ang **Code** → tab na **Codespaces** → **...** → **New with options...**
3. Gamitin ang mga default – pipiliin nito ang **Dev container configuration**: **Generative AI Java Development Environment** na custom devcontainer para sa kursong ito
4. I-click ang **Create codespace**
5. Maghintay ng mga ~2 minuto hanggang maging handa ang environment
6. Magpatuloy sa [Hakbang 2: Mag-provision ng Azure AI Foundry](#hakbang-2-mag-provision-ng-azure-ai-foundry)

<img src="../../../translated_images/tl/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: Codespaces submenu" width="50%">

<img src="../../../translated_images/tl/image.833552b62eee7766.webp" alt="Screenshot: New with options" width="50%">

<img src="../../../translated_images/tl/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: Create codespace options" width="50%">

> **Mga Benepisyo ng Codespaces**:
> - Hindi kailangan ng lokal na instalasyon
> - Gumagana sa kahit anong device na may browser
> - Pre-configured na may lahat ng tools at dependencies
> - Libreng 60 oras kada buwan para sa mga personal na account
> - Konsistent na environment para sa lahat ng mag-aaral

#### Opsyon B: Local Dev Container

**Para sa mga developer na mas gustong local development gamit ang Docker**

1. I-fork at i-clone ang repositoryong ito sa iyong lokal na makina
   > **Tandaan**: Kung nais mong i-edit ang basic config, tingnan ang [Dev Container Configuration](../../../.devcontainer/devcontainer.json)
2. I-install ang [Docker Desktop](https://www.docker.com/products/docker-desktop/) at [VS Code](https://code.visualstudio.com/)
3. I-install ang [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) sa VS Code
4. Buksan ang folder ng repository sa VS Code
5. Kapag na-prompt, i-click ang **Reopen in Container** (o gamitin ang `Ctrl+Shift+P` → "Dev Containers: Reopen in Container")
6. Maghintay habang binubuo at nagsisimula ang container
7. Magpatuloy sa [Hakbang 2: Mag-provision ng Azure AI Foundry](#hakbang-2-mag-provision-ng-azure-ai-foundry)

<img src="../../../translated_images/tl/devcontainer.21126c9d6de64494.webp" alt="Screenshot: Dev container setup" width="50%">

<img src="../../../translated_images/tl/image-3.bf93d533bbc84268.webp" alt="Screenshot: Dev container build complete" width="50%">

#### Opsyon C: Gamitin ang Iyong Kasalukuyang Lokal na Instalasyon

**Para sa mga developer na may kasalukuyang Java environment**

Mga kinakailangan:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html)
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) o ang iyong preferred na IDE

Mga hakbang:
1. I-clone ang repositoryong ito sa iyong lokal na makina
2. Buksan ang proyekto sa iyong IDE
3. Magpatuloy sa [Hakbang 2: Mag-provision ng Azure AI Foundry](#hakbang-2-mag-provision-ng-azure-ai-foundry)

> **Pro Tip**: Kung mababa ang specs ng iyong makina pero gusto mo ng lokal na VS Code, gamitin ang GitHub Codespaces! Maaari mong ikonekta ang iyong lokal na VS Code sa isang cloud-hosted Codespace para sa pinakamahusay na karanasan.

<img src="../../../translated_images/tl/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: created local devcontainer instance" width="50%">

## Hakbang 2: Mag-provision ng Azure AI Foundry

I-deploy ang mga AI model ng kurso sa Azure AI Foundry bilang code. Mula sa pinaka-ugat ng repository:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```

Hinahanap ng `azd` ang pangalan ng environment at rehiyon, nagpo-provision ng Azure AI Foundry account na may `gpt-4o-mini` at `text-embedding-3-small` na mga deployment, at sinusulat ang endpoint sa `.env` ng halimbawa — lahat gamit ang **keyless** authentication (walang API key).

> **Buong walkthrough:** Tingnan ang [Azure AI Foundry Setup Guide](getting-started-azure-openai.md) para sa mga kinakailangan, manual (portal) na alternatibo, gabay sa rehiyon, at mga tala sa gastos/cleanup.

## Hakbang 3: Subukan ang Iyong Setup

Kapag naka-provision na ang iyong Foundry models, subukan ang koneksyon gamit ang example app sa [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure).

1. Buksan ang terminal sa iyong development environment.
2. Pumunta sa example:
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
3. Siguraduhing naka-sign in ka (kailangan ang keyless auth ng token):
   ```bash
   az login
   ```
   > Kung pinatakbo mo ang `azd up`, naisulat na ang `.env` file kasama ang iyong endpoint para sa iyo.
4. Patakbuhin ang aplikasyon:
   ```bash
   mvn clean spring-boot:run
   ```

Dapat kang makakita ng tugon mula sa `gpt-4o-mini` na model.

### Pag-unawa sa Example Code

Ang example sa ilalim ng `examples/basic-chat-azure` ay isang Spring Boot app na gumagamit ng **Spring AI** para kumonekta sa Azure AI Foundry gamit ang keyless authentication.

**Ginagawa ng code na ito:**
- **Kumokonekta** sa Azure AI Foundry gamit ang iyong Azure sign-in (Microsoft Entra ID) — walang API key
- **Nagpapadala** ng prompt sa `gpt-4o-mini` model
- **Tumatanggap** at ipinapakita ang tugon ng AI
- **Tinitiyak** na maayos ang iyong setup

**Pangunahing Dependency** (sa `pom.xml`):
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```

**Configuration** (`application.yml`):
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


## Buod

Magaling! Naka-setup mo na ang lahat:

- Na-provision ang Azure AI Foundry models bilang code gamit ang Bicep + `azd`
- Napatakbo ang iyong Java development environment (Codespaces, dev containers, o local man)
- Nakakonekta sa Azure AI Foundry gamit ang keyless authentication (Microsoft Entra ID) — walang API keys
- Nasubukan na ang lahat gamit ang isang simpleng halimbawa na nakikipag-usap sa iyong model

## Mga Susunod na Hakbang

[Kabanata 3: Mga Pangunahing Teknik ng Generative AI](../03-CoreGenerativeAITechniques/README.md)

## Pag-troubleshoot

May problema? Narito ang mga karaniwang isyu at solusyon:

- **Palpak ang authentication (401/403)?**
  - Patakbuhin ang `az login` — keyless ang authentication kaya kailangan naka-sign in ka
  - Siguraduhing mayroong **Cognitive Services OpenAI User** role ang iyong account sa resource
  - Kung kakapag-provision mo lang, maghintay ng isang minuto para ma-propagate ang role assignment

- **Hindi makita ang Maven?**
  - Kung gumagamit ng dev containers/Codespaces, pre-installed na ang Maven
  - Para sa lokal na setup, siguraduhing naka-install ang Java 21+ at Maven 3.9+
  - Subukang patakbuhin ang `mvn --version` para i-verify ang instalasyon

- **Hindi makita ang `azd` o pumapalya ang provisioning?**
  - I-install ang [Azure Developer CLI](https://aka.ms/azure-dev/install) at patakbuhin ang `azd auth login`
  - Piliin ang rehiyon kung saan available ang `gpt-4o-mini` (hal. `eastus2`)
  - Tingnan ang [Azure AI Foundry setup guide](getting-started-azure-openai.md) para sa mga detalye

- **Hindi nagsisimula ang Dev container?**
  - Siguraduhing tumatakbo ang Docker Desktop (para sa lokal na development)
  - Subukang i-rebuild ang container: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **Mga error sa pag-compile ng aplikasyon?**
  - Siguraduhing nasa tamang direktoryo ka: `02-SetupDevEnvironment/examples/basic-chat-azure`
  - Subukang linisin at i-rebuild: `mvn clean compile`

> **Kailangan ng tulong?**: May problema pa rin? Gumawa ng issue sa repository at tutulungan ka namin.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Pagtatanggi**:
Ang dokumentong ito ay isinalin gamit ang serbisyo ng AI translation na [Co-op Translator](https://github.com/Azure/co-op-translator). Bagama't nagsusumikap kami para sa katumpakan, pakatandaan na ang awtomatikong pagsasalin ay maaaring maglaman ng mga pagkakamali o hindi pagkakatugma. Ang orihinal na dokumento sa orihinal nitong wika ang dapat ituring na pangunahing sanggunian. Para sa mahahalagang impormasyon, inirerekomenda ang propesyonal na pagsasalin ng tao. Hindi kami mananagot sa anumang maling pagkakaintindi o maling interpretasyon na nagmula sa paggamit ng pagsasaling ito.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->