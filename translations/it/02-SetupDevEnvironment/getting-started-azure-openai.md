# Configurare l'Ambiente di Sviluppo per Azure AI Foundry

> Questa guida configura i modelli di **Azure AI Foundry** per le app Java AI di questo corso, utilizzando l'autenticazione **senza chiave** (Microsoft Entra ID) — nessuna API key da gestire. Sei nuovo agli strumenti? Inizia con la [guida all'ambiente di sviluppo](./README.md).

Questa guida configura i modelli di **Azure AI Foundry** per le app Java AI di questo corso. Hai due percorsi:

- **Opzione A — Provision con `azd` + Bicep (consigliato):** un comando distribuisce l'account Foundry e i modelli come codice. Nessun clic nel portale.
- **Opzione B — Crea le risorse manualmente** nel portale Azure AI Foundry.

Entrambi i percorsi utilizzano l'autenticazione **senza chiave** (Microsoft Entra ID) — non ci sono API key da copiare o divulgare.

## Indice

- [Cosa viene creato](#cosa-viene-creato)
- [Prerequisiti](#prerequisiti)
- [Opzione A: Provision con azd + Bicep (Consigliato)](#option-a-provision-with-azd--bicep-recommended)
- [Opzione B: Creare le Risorse Manualmente](#opzione-b-creare-le-risorse-manualmente)
- [Configura il Tuo Ambiente](#configura-il-tuo-ambiente)
- [Testa la Tua Configurazione](#testa-la-tua-configurazione)
- [Cosa Fare Dopo?](#cosa-fare-dopo)
- [Risorse](#risorse)
- [Risorse Aggiuntive](#risorse-aggiuntive)

## Cosa viene creato

I template Bicep in [`infra/`](../../../02-SetupDevEnvironment/infra) creano:

- Un account **Azure AI Foundry** (`Microsoft.CognitiveServices/accounts`, tipo `AIServices`) con un progetto
- Una distribuzione **chat** — `gpt-4o-mini`
- Una distribuzione **embedding** — `text-embedding-3-small` (usata nei capitoli successivi)
- Un **assegnamento di ruolo senza chiave** (`Cognitive Services OpenAI User`) così ti autentichi con `az login` invece di gestire le chiavi

## Prerequisiti

- Un [abbonamento Azure](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) e [Maven 3.9+](https://maven.apache.org/download.cgi)

## Opzione A: Provision con azd + Bicep (Consigliato)

Dalla cartella `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# Accedi (entrambi gli strumenti)
azd auth login
az login

# Provisiona l'account Foundry + distribuzioni del modello
azd up
```

`azd` ti chiederà un **nome ambiente** (ad esempio `genai-java`) e una **regione**. Scegli una regione dove sono disponibili `gpt-4o-mini` e `text-embedding-3-small` — per esempio `eastus2` o `swedencentral`.

Quando il provisioning è terminato, azd:

1. Distribuisce tutto ciò che è definito in [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep).
2. Esegue uno script post-provisioning che scrive [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) con il tuo endpoint e i nomi delle distribuzioni (nessun segreto).

> **Consiglio:** Rilancia `azd up` in qualsiasi momento per applicare modifiche. Esegui `azd down` per eliminare tutto ed evitare costi.

Per vedere le impostazioni generate:

```bash
azd env get-values
```

Ora passa a [Testa la Tua Configurazione](#testa-la-tua-configurazione).

## Opzione B: Creare le Risorse Manualmente

Preferisci il portale? Crea le risorse manualmente:

1. Vai al [portale Azure AI Foundry](https://ai.azure.com/) e accedi.
2. **Crea un progetto** (crea anche una risorsa AI Foundry). Dagli un nome come `GenAIJava`.
3. Nel progetto apri **Modelli + endpoint** → **Distribuisci modello** → **Distribuisci modello base**.
4. Distribuisci **gpt-4o-mini** (nome distribuzione `gpt-4o-mini`). Ripeti per **text-embedding-3-small** se vuoi gli esempi embedding.
5. Da **Panoramica**, copia l'**endpoint** (per esempio `https://<risorsa>.openai.azure.com/`).
6. Concedi a te stesso accesso senza chiave: nella risorsa, apri **Controllo accessi (IAM)** → **Aggiungi assegnazione ruolo** → assegna **Cognitive Services OpenAI User** al tuo account.

> **Hai ancora problemi?** Vedi la [documentazione Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Configura il Tuo Ambiente

**Se hai usato l'Opzione A (`azd up`)**, il file di configurazione è già creato — non devi configurare nulla. Passa a [Testa la Tua Configurazione](#testa-la-tua-configurazione).

**Se hai usato l'Opzione B (manuale)**, crea tu stesso il file `.env` dell’esempio:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

Modifica `.env` con il tuo endpoint (senza chiave — l'autenticazione è senza chiave):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Nota sulla sicurezza:** Non c’è nessuna API key da conservare. Ti autentichi con Microsoft Entra ID tramite `az login` (localmente) o un’identità gestita (in Azure). Il file `.env` contiene solo impostazioni non segrete ed è già incluso in `.gitignore`.

## Testa la Tua Configurazione

Assicurati di aver effettuato l’accesso così l’autenticazione senza chiave può ottenere un token, quindi esegui l’esempio:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # se non hai ancora effettuato l'accesso
mvn clean spring-boot:run
```

Dovresti vedere una risposta dal modello `gpt-4o-mini`!

> **Utenti VS Code:** Premi `F5` per eseguire. L’app carica automaticamente il file `.env`.

> **Esempio completo:** Vedi il [Basic Chat con Azure AI Foundry](./examples/basic-chat-azure/README.md) per dettagli e risoluzione problemi.

## Cosa Fare Dopo?

**Configurazione completata!** Ora hai:
- Azure AI Foundry con `gpt-4o-mini` e `text-embedding-3-small` distribuiti
- Autenticazione senza chiave (Microsoft Entra ID) — nessuna chiave da gestire
- Un file `.env` locale con il tuo endpoint e i nomi di distribuzione
- Un ambiente di sviluppo Java pronto all’uso

**Continua con** [Capitolo 3: Tecniche Core di Generative AI](../03-CoreGenerativeAITechniques/README.md) per iniziare a costruire applicazioni AI!

## Risorse

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Autenticazione senza chiave con Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Documentazione Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)
- [Documentazione Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Risorse Aggiuntive

- [Scarica VS Code](https://code.visualstudio.com/Download)
- [Scarica Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Configurazione Dev Container](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Disclaimer**:
Questo documento è stato tradotto utilizzando il servizio di traduzione AI [Co-op Translator](https://github.com/Azure/co-op-translator). Sebbene ci impegniamo per garantire la precisione, si prega di notare che le traduzioni automatizzate possono contenere errori o imprecisioni. Il documento originale nella sua lingua nativa deve essere considerato la fonte autorevole. Per informazioni critiche, si raccomanda una traduzione professionale effettuata da un essere umano. Non siamo responsabili per eventuali malintesi o interpretazioni errate derivanti dall’uso di questa traduzione.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->