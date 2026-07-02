# Pagsasaayos ng Development Environment para sa Azure AI Foundry

> Itong gabay ay nagse-setup ng **Azure AI Foundry** models para sa Java AI apps sa kursong ito, gamit ang **keyless** na authentication (Microsoft Entra ID) — walang API keys na kailangang pamahalaan. Bago sa tooling? Magsimula sa [development environment guide](./README.md).

Itong gabay ay nagse-setup ng **Azure AI Foundry** models para sa Java AI apps sa kursong ito. Mayroon kang dalawang pagpipilian:

- **Option A — Magprovision gamit ang `azd` + Bicep (inirerekomenda):** isang utos lang ang nagde-deploy ng Foundry account at mga modelo bilang code. Walang pag-click sa portal.
- **Option B — Gawin ang mga resources nang mano-mano** sa Azure AI Foundry portal.

Parehong gumagamit ng **keyless authentication** (Microsoft Entra ID) — walang API keys na kailangang kopyahin o malantad.

## Table of Contents

- [Ano ang Nai-create](#ano-ang-nai-create)
- [Mga Kinakailangan](#mga-kinakailangan)
- [Option A: Provision gamit ang azd + Bicep (Inirerekomenda)](#option-a-provision-with-azd--bicep-recommended)
- [Option B: Gawin ang mga Resources nang Mano-mano](#option-b-gawin-ang-mga-resources-nang-mano-mano)
- [I-configure ang Iyong Kapaligiran](#i-configure-ang-iyong-kapaligiran)
- [Subukan ang Iyong Setup](#subukan-ang-iyong-setup)
- [Ano ang Susunod?](#ano-ang-susunod)
- [Mga Resources](#mga-resources)
- [Karagdagang Resources](#karagdagang-resources)

## Ano ang Nai-create

Ang mga Bicep templates sa [`infra/`](../../../02-SetupDevEnvironment/infra) ay nagpoprovide ng:

- Isang **Azure AI Foundry** account (`Microsoft.CognitiveServices/accounts`, kind `AIServices`) na may project
- Isang **chat** deployment — `gpt-4o-mini`
- Isang **embedding** deployment — `text-embedding-3-small` (ginagamit sa mga huling kabanata)
- Isang **keyless role assignment** (`Cognitive Services OpenAI User`) kaya makakapag-sign in ka gamit ang `az login` imbes na mag-manage ng mga keys

## Mga Kinakailangan

- Isang [Azure subscription](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) at [Maven 3.9+](https://maven.apache.org/download.cgi)

## Option A: Provision gamit ang azd + Bicep (Inirerekomenda)

Mula sa folder na `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# Mag-sign in (parehong tools)
azd auth login
az login

# I-provision ang Foundry account + model deployments
azd up
```

Hihingin ng `azd` ang isang **environment name** (halimbawa `genai-java`) at isang **region**. Pumili ng region kung saan available ang `gpt-4o-mini` at `text-embedding-3-small` — halimbawa `eastus2` o `swedencentral`.

Kapag natapos ang provisioning, ang azd ay:

1. Ide-deploy lahat ng nakasaad sa [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep).
2. Patatakbuhin ang postprovision hook na nagsusulat ng [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) na may iyong endpoint at pangalan ng deployments (walang mga sikreto).

> **Tip:** Patakbuhin muli ang `azd up` anumang oras para i-apply ang mga pagbabago. Patakbuhin ang `azd down` para burahin lahat at ihinto ang pag-incur ng gastos.

Para makita ang mga nagawang settings:

```bash
azd env get-values
```

Ngayon, tumalon sa [Test Your Setup](#subukan-ang-iyong-setup).

## Option B: Gawin ang mga Resources nang Mano-mano

Mas gusto ang portal? Gumawa ng mga resources nang manu-mano:

1. Pumunta sa [Azure AI Foundry portal](https://ai.azure.com/) at mag-sign in.
2. **Gumawa ng project** (ito rin ay gumagawa ng AI Foundry resource). Bigyan ito ng pangalan tulad ng `GenAIJava`.
3. Sa iyong project, buksan ang **Models + endpoints** → **Deploy model** → **Deploy base model**.
4. I-deploy ang **gpt-4o-mini** (deployment name ay `gpt-4o-mini`). Ulitin para sa **text-embedding-3-small** kung gusto mo ang embedding examples.
5. Mula sa **Overview**, kopyahin ang **endpoint** (halimbawa `https://<resource>.openai.azure.com/`).
6. Bigyan ang iyong sarili ng keyless access: sa resource, buksan ang **Access control (IAM)** → **Add role assignment** → i-assign ang **Cognitive Services OpenAI User** sa iyong account.

> **May problema pa?** Tingnan ang [Azure AI Foundry documentation](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## I-configure ang Iyong Kapaligiran

**Kung ginamit mo ang Option A (`azd up`)**, nakasulat na ang iyong settings file — walang kailangan i-configure. Tumalon na sa [Test Your Setup](#subukan-ang-iyong-setup).

**Kung ginamit mo ang Option B (manu-mano)**, gawin mo ang `.env` file ng example:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

I-edit ang `.env` gamit ang iyong endpoint (walang key — keyless ang auth):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Paalala sa seguridad:** Walang API key na dapat itago. Nag-a-authenticate ka gamit ang Microsoft Entra ID via `az login` (lokal) o managed identity (sa Azure). Ang `.env` file ay may lamang mga hindi sikreto na settings at sakop na ito ng `.gitignore`.

## Subukan ang Iyong Setup

Siguraduhing naka-sign in ka para makakuha ng token ang keyless auth, pagkatapos patakbuhin ang example:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # kung hindi ka pa naka-sign in
mvn clean spring-boot:run
```

Dapat kang makakita ng tugon mula sa `gpt-4o-mini` model!

> **Para sa mga gumagamit ng VS Code:** Pindutin ang `F5` para tumakbo. Awtomatiko nitong niloload ang iyong `.env`.

> **Buong example:** Tingnan ang [Basic Chat with Azure AI Foundry example](./examples/basic-chat-azure/README.md) para sa mga detalye at troubleshooting.

## Ano ang Susunod?

**Kumpleto na ang setup!** Ngayon ay mayroon ka nang:
- Azure AI Foundry na may `gpt-4o-mini` at `text-embedding-3-small` na naka-deploy
- Keyless authentication (Microsoft Entra ID) — walang mga keys na kailangang pamahalaan
- Lokal na `.env` file na may iyong endpoint at mga pangalan ng deployment
- Isang Java development environment na handa nang gamitin

**Magpatuloy sa** [Chapter 3: Core Generative AI Techniques](../03-CoreGenerativeAITechniques/README.md) para simulan ang paggawa ng AI applications!

## Mga Resources

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Keyless authentication gamit ang Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry Documentation](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI Documentation](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Karagdagang Resources

- [I-download ang VS Code](https://code.visualstudio.com/Download)
- [Kunin ang Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Dev Container Configuration](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Pagtatanggi**:
Ang dokumentong ito ay isinalin gamit ang serbisyo ng AI translation na [Co-op Translator](https://github.com/Azure/co-op-translator). Bagama't nagsusumikap kami para sa katumpakan, pakatandaan na ang awtomatikong pagsasalin ay maaaring maglaman ng mga pagkakamali o hindi pagkakatugma. Ang orihinal na dokumento sa orihinal nitong wika ang dapat ituring na pangunahing sanggunian. Para sa mahahalagang impormasyon, inirerekomenda ang propesyonal na pagsasalin ng tao. Hindi kami mananagot sa anumang maling pagkakaintindi o maling interpretasyon na nagmula sa paggamit ng pagsasaling ito.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->