# Setting Up the Development Environment for Generative AI for Java

> **Quick Start:** Provision your AI models on **Azure AI Foundry** as code with Bicep + `azd` for small time — see the [Azure AI Foundry Setup Guide](getting-started-azure-openai.md). Authentication no get key (Microsoft Entra ID), so no API keys wey you go manage.

## Wetin You Go Learn

- Set up Java development environment for AI applications
- Choose and configure your preferred development environment (cloud-first with Codespaces, local dev container, or full local setup)
- Test your setup by connecting to Azure AI Foundry model

## Table of Contents

- [Wetn You Go Learn](#wetin-you-go-learn)
- [Introduction](#introduction)
- [Step 1: Set Up Your Development Environment](#step-1-set-up-your-development-environment)
  - [Option A: GitHub Codespaces (Recommended)](#option-a-github-codespaces-recommended)
  - [Option B: Local Dev Container](#option-b-local-dev-container)
  - [Option C: Use Your Existing Local Installation](#option-c-use-your-existing-local-installation)
- [Step 2: Provision Azure AI Foundry](#step-2-provision-azure-ai-foundry)
- [Step 3: Test Your Setup](#step-3-test-your-setup)
- [Troubleshooting](#troubleshooting)
- [Summary](#summary)
- [Next Steps](#next-steps)

## Introduction

This chapter go guide you how to set up development environment. We go use **Azure AI Foundry** for all di models for dis course. You go provision di models as code with Bicep and Azure Developer CLI (`azd`), then connect with **keyless authentication** (Microsoft Entra ID) — no API keys wey you go copy or leak.

**No local setup necessary!** You fit use GitHub Codespaces, wey go give you full development environment inside your browser, and provision Foundry from there.

We dey use **Azure AI Foundry** for dis course because e:
- **Provisioned as code** — one `azd up` go deploy di account and model deployments
- **Keyless** — authenticate with your Azure sign-in or managed identity
- **Production-ready** — di same code fit run locally and for Azure
- **Flexible** — you fit swap models by changing deployment name, no be your code

> **Note**: Azure AI Foundry deployments dey charge by token (pay-as-you-go). See di [Azure AI Foundry setup guide](getting-started-azure-openai.md) for provisioning, region, and cost details.


## Step 1: Set Up Your Development Environment

<a name="quick-start-cloud"></a>

We don create preconfigured development container to reduce setup time and make sure say you get all di tools you need for dis Generative AI for Java course. Choose your preferred development approach:

### Environment Setup Options:

#### Option A: GitHub Codespaces (Recommended)

**Start to code for 2 minutes - no local setup necessary!**

1. Fork this repository to your GitHub account
   > **Note**: If you want change basic config, abeg check the [Dev Container Configuration](../../../.devcontainer/devcontainer.json)
2. Click **Code** → **Codespaces** tab → **...** → **New with options...**
3. Use the defaults – this one go select **Dev container configuration**: **Generative AI Java Development Environment** custom devcontainer created for this course
4. Click **Create codespace**
5. Wait ~2 minutes make environment ready
6. Go to [Step 2: Provision Azure AI Foundry](#step-2-provision-azure-ai-foundry)

<img src="../../../translated_images/pcm/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: Codespaces submenu" width="50%">

<img src="../../../translated_images/pcm/image.833552b62eee7766.webp" alt="Screenshot: New with options" width="50%">

<img src="../../../translated_images/pcm/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: Create codespace options" width="50%">


> **Benefits of Codespaces**:
> - No local installation necessary
> - E go work on any device wey get browser
> - Pre-configured with all tools and dependencies
> - Free 60 hours every month for personal accounts
> - Consistent environment for all learners

#### Option B: Local Dev Container

**For developers wey like local development with Docker**

1. Fork and clone this repository to your local machine
   > **Note**: If you want change basic config, abeg check the [Dev Container Configuration](../../../.devcontainer/devcontainer.json)
2. Install [Docker Desktop](https://www.docker.com/products/docker-desktop/) and [VS Code](https://code.visualstudio.com/)
3. Install the [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) for VS Code
4. Open the repository folder inside VS Code
5. When e ask, click **Reopen in Container** (or use `Ctrl+Shift+P` → "Dev Containers: Reopen in Container")
6. Wait make container build and start
7. Go to [Step 2: Provision Azure AI Foundry](#step-2-provision-azure-ai-foundry)

<img src="../../../translated_images/pcm/devcontainer.21126c9d6de64494.webp" alt="Screenshot: Dev container setup" width="50%">

<img src="../../../translated_images/pcm/image-3.bf93d533bbc84268.webp" alt="Screenshot: Dev container build complete" width="50%">

#### Option C: Use Your Existing Local Installation

**For developers wey don get Java environments properly**

Prerequisites:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html) 
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) or your preferred IDE

Steps:
1. Clone this repository to your local machine
2. Open the project inside your IDE
3. Go to [Step 2: Provision Azure AI Foundry](#step-2-provision-azure-ai-foundry)

> **Pro Tip**: If your machine low-spec but you want VS Code locally, use GitHub Codespaces! You fit connect your local VS Code to cloud-hosted Codespace make you get best of both worlds.

<img src="../../../translated_images/pcm/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: created local devcontainer instance" width="50%">


## Step 2: Provision Azure AI Foundry

Deploy di course AI models to Azure AI Foundry as code. From di repository root:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```

`azd` go ask you environment name and region, go provision Azure AI Foundry account with `gpt-4o-mini` and `text-embedding-3-small` deployments, write the endpoint inside di example `.env` — all this na with **keyless** authentication (no API keys).

> **Full walkthrough:** See the [Azure AI Foundry Setup Guide](getting-started-azure-openai.md) for prerequisites, manual (portal) alternative, region guidance, and cost/cleanup notes.

## Step 3: Test Your Setup

Once your Foundry models don provision, test di connection with the example app inside [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure).

1. Open terminal inside your development environment.
2. Go inside the example:
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
3. Make sure say you don sign in (keyless auth need token):
   ```bash
   az login
   ```
   > If you run `azd up`, `.env` file wey get your endpoint don already write for you.
4. Run the application:
   ```bash
   mvn clean spring-boot:run
   ```

You suppose see response from `gpt-4o-mini` model.

### How the Example Code Work

The example under `examples/basic-chat-azure` na Spring Boot app wey dey use **Spring AI** to connect Azure AI Foundry with keyless authentication.

**Wetin dis code dey do:**
- **Connect** to Azure AI Foundry using your Azure sign-in (Microsoft Entra ID) — no API key
- **Send** prompt to `gpt-4o-mini` model
- **Receive** and show AI response
- **Check** say your setup dey work well

**Key Dependency** (for `pom.xml`):
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

## Summary

Correct! You don set up everything:

- Provisioned Azure AI Foundry models as code with Bicep + `azd`
- Your Java development environment dey run (whether Codespaces, dev containers, or local)
- Connected to Azure AI Foundry with keyless authentication (Microsoft Entra ID) — no API keys
- Test all works with simple example wey dey talk to your model

## Next Steps

[Chapter 3: Core Generative AI Techniques](../03-CoreGenerativeAITechniques/README.md)

## Troubleshooting

Get wahala? Here na common problems and how to fix dem:

- **Authentication dey fail (401/403)?** 
  - Run `az login` — authentication no need key, you must sign in
  - Check say your account get **Cognitive Services OpenAI User** role on the resource
  - If you just provision, wait small make role assignment go full propagate

- **Maven no dey?** 
  - If you dey use dev containers/Codespaces, Maven dey pre-installed
  - For local setup, make sure Java 21+ and Maven 3.9+ dey installed
  - Run `mvn --version` to check

- **`azd` no dey or provisioning no dey work?** 
  - Install [Azure Developer CLI](https://aka.ms/azure-dev/install) and run `azd auth login`
  - Choose region wey `gpt-4o-mini` dey available (e.g. `eastus2`)
  - See [Azure AI Foundry setup guide](getting-started-azure-openai.md) for details

- **Dev container no dey start?** 
  - Make sure Docker Desktop dey run (for local development)
  - Try rebuild di container: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **Application get compile errors?**
  - Make sure you dey inside correct directory: `02-SetupDevEnvironment/examples/basic-chat-azure`
  - Try clean and rebuild: `mvn clean compile`

> **Need help?**: Still get wahala? Open issue inside repository, we go help you.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Disclaimer**:
Dis document don translate wit AI translation service [Co-op Translator](https://github.com/Azure/co-op-translator). Even tho we dey try make am correct, abeg make you know say automated translation fit get errors or mistakes. Di original document for dia own language na im be di correct source. For important info, make person wey sabi human translation do am. We no go responsible for any misunderstanding or wrong understanding wey fit happen because of dis translation.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->