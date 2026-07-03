# Setting Up the Development Environment for Azure AI Foundry

> Dis guide dey set up **Azure AI Foundry** models for the Java AI apps wey dey dis course, using **keyless** authentication (Microsoft Entra ID) — no API keys to manage. If you be new for the tooling? Start wit di [development environment guide](./README.md).

Dis guide dey set up **Azure AI Foundry** models for the Java AI apps for dis course. You get two ways:

- **Option A — Provision wit `azd` + Bicep (recommended):** one command go deploy the Foundry account and models as code. No portal clicking.
- **Option B — Create resources manually** for the Azure AI Foundry portal.

Both ways dey use **keyless authentication** (Microsoft Entra ID) — no API keys to copy or leak.

## Table of Contents

- [Wetn Go Turn](#wetn-go-turn)
- [Wetin You Need First](#wetin-you-need-first)
- [Option A: Provision wit azd + Bicep (Recommended)](#option-a-provision-with-azd--bicep-recommended)
- [Option B: Create Resources Manually](#option-b-create-resources-manually)
- [Configure Your Environment](#configure-your-environment)
- [Test Your Setup](#test-your-setup)
- [Wetn Next?](#wetn-next)
- [Resources](#resources)
- [Additional Resources](#additional-resources)

## Wetn Go Turn

The Bicep templates for [`infra/`](../../../02-SetupDevEnvironment/infra) go provide:

- One **Azure AI Foundry** account (`Microsoft.CognitiveServices/accounts`, kind `AIServices`) wit one project
- One **chat** deployment — `gpt-4o-mini`
- One **embedding** deployment — `text-embedding-3-small` (you go use am for later chapters)
- One **keyless role assignment** (`Cognitive Services OpenAI User`) so you fit sign in wit `az login` instead of manage keys

## Wetin You Need First

- One [Azure subscription](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) and [Maven 3.9+](https://maven.apache.org/download.cgi)

## Option A: Provision wit azd + Bicep (Recommended)

For the `02-SetupDevEnvironment` folder:

```bash
cd 02-SetupDevEnvironment

# Sign in (both tools)
azd auth login
az login

# Set up di Foundry account + model deployments
azd up
```

`azd` go ask for **environment name** (example `genai-java`) and **region**. Choose one region wey `gpt-4o-mini` and `text-embedding-3-small` dey available — example `eastus2` or `swedencentral`.

After the provisioning finish, azd go:

1. Deploy everything wey dey for [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep).
2. Run postprovision hook wey go write [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) wit your endpoint and deployment names (no secrets).

> **Tip:** You fit run `azd up` again anytime to apply changes. Run `azd down` to delete everything and stop tey cost.

To see the settings wey dem generate:

```bash
azd env get-values
```

Now make you waka go [Test Your Setup](#test-your-setup).

## Option B: Create Resources Manually

You like portal more? Create resources yourself:

1. Go the [Azure AI Foundry portal](https://ai.azure.com/) and sign in.
2. **Create one project** (dis one go also create AI Foundry resource). Give am name like `GenAIJava`.
3. For your project, open **Models + endpoints** → **Deploy model** → **Deploy base model**.
4. Deploy **gpt-4o-mini** (deployment name `gpt-4o-mini`). Do am again for **text-embedding-3-small** if you want to use embedding examples.
5. From **Overview**, copy the **endpoint** (example `https://<resource>.openai.azure.com/`).
6. Give yourself keyless access: for the resource, open **Access control (IAM)** → **Add role assignment** → assign **Cognitive Services OpenAI User** to your account.

> **Still dey find wahala?** Check the [Azure AI Foundry documentation](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Configure Your Environment

**If you use Option A (`azd up`)**, your settings file don already write — no need to configure anything. Skip go [Test Your Setup](#test-your-setup).

**If you use Option B (manual)**, create the example’s `.env` file yourself:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

Edit `.env` wit your endpoint (no key — authentication na keyless):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Security note:** No API key dey for storage. You dey authenticate wit Microsoft Entra ID via `az login` (local) or managed identity (for Azure). The `.env` file get only non-secret settings and e don already dey covered by `.gitignore`.

## Test Your Setup

Make sure say you don sign in so keyless auth fit collect token, then run the example:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # if you no don sign in before na
mvn clean spring-boot:run
```

You go see response from the `gpt-4o-mini` model!

> **VS Code users:** Press `F5` to run. The app go load your `.env` automatically.

> **Full example:** See the [Basic Chat wit Azure AI Foundry example](./examples/basic-chat-azure/README.md) for details and troubleshooting.

## Wetn Next?

**Setup don finish!** Now you get:

- Azure AI Foundry wit `gpt-4o-mini` and `text-embedding-3-small` deployed
- Keyless authentication (Microsoft Entra ID) — no keys to manage
- One local `.env` wit your endpoint and deployment names
- One Java development environment wey ready to use

**Continue to** [Chapter 3: Core Generative AI Techniques](../03-CoreGenerativeAITechniques/README.md) to start to build AI apps!

## Resources

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Keyless authentication wit Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry Documentation](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI Documentation](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Additional Resources

- [Download VS Code](https://code.visualstudio.com/Download)
- [Get Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Dev Container Configuration](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Disclaimer**:
Dis document don translate wit AI translation service [Co-op Translator](https://github.com/Azure/co-op-translator). Even tho we dey try make am correct, abeg make you know say automated translation fit get errors or mistakes. Di original document for dia own language na im be di correct source. For important info, make person wey sabi human translation do am. We no go responsible for any misunderstanding or wrong understanding wey fit happen because of dis translation.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->