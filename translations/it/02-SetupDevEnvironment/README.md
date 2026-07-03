# Configurare l'Ambiente di Sviluppo per Generative AI per Java

> **Avvio rapido:** Provisiona i tuoi modelli AI su **Azure AI Foundry** come codice con Bicep + `azd` in pochi minuti — consulta la [Guida alla Configurazione di Azure AI Foundry](getting-started-azure-openai.md). L'autenticazione è **senza chiavi** (Microsoft Entra ID), quindi non ci sono chiavi API da gestire.

## Cosa Imparerai

- Configurare un ambiente di sviluppo Java per applicazioni AI
- Scegliere e configurare l'ambiente di sviluppo preferito (cloud-first con Codespaces, container di sviluppo locale, o configurazione locale completa)
- Testare la configurazione collegandosi a un modello Azure AI Foundry

## Indice

- [Cosa Imparerai](#cosa-imparerai)
- [Introduzione](#introduzione)
- [Passo 1: Configura il Tuo Ambiente di Sviluppo](#passo-1-configura-il-tuo-ambiente-di-sviluppo)
  - [Opzione A: GitHub Codespaces (Consigliato)](#opzione-a-github-codespaces-consigliato)
  - [Opzione B: Container di Sviluppo Locale](#opzione-b-container-di-sviluppo-locale)
  - [Opzione C: Usa la Tua Installazione Locale Esistente](#opzione-c-usa-la-tua-installazione-locale-esistente)
- [Passo 2: Provisiona Azure AI Foundry](#passo-2-provisiona-azure-ai-foundry)
- [Passo 3: Testa la Tua Configurazione](#passo-3-testa-la-tua-configurazione)
- [Risoluzione dei Problemi](#risoluzione-dei-problemi)
- [Sommario](#sommario)
- [Prossimi Passi](#prossimi-passi)

## Introduzione

Questo capitolo ti guiderà nella configurazione di un ambiente di sviluppo. Utilizzeremo **Azure AI Foundry** per i modelli durante tutto il corso. Provisionerai i modelli come codice con Bicep e l'Azure Developer CLI (`azd`), poi ti connetterai con **autenticazione senza chiavi** (Microsoft Entra ID) — nessuna chiave API da copiare o rischiare di perdere.

**Nessuna configurazione locale richiesta!** Puoi usare GitHub Codespaces, che fornisce un ambiente di sviluppo completo nel tuo browser, e provisionare Foundry da lì.

Utilizziamo **Azure AI Foundry** per questo corso perché è:
- **Provisionato come codice** — un singolo `azd up` crea l'account e le distribuzioni dei modelli
- **Senza chiavi** — autenticati con il tuo accesso Azure o un'identità gestita
- **Pronto per la produzione** — lo stesso codice funziona localmente e su Azure
- **Flessibile** — cambia modello modificando il nome della distribuzione, non il codice

> **Nota**: Le distribuzioni di Azure AI Foundry sono fatturate per token (paghi solo quello che usi). Consulta la [guida alla configurazione di Azure AI Foundry](getting-started-azure-openai.md) per dettagli su provisioning, regione e costi.


## Passo 1: Configura il Tuo Ambiente di Sviluppo

<a name="quick-start-cloud"></a>

Abbiamo creato un container di sviluppo preconfigurato per minimizzare i tempi di configurazione e assicurarti di avere tutti gli strumenti necessari per questo corso Generative AI per Java. Scegli il tuo approccio di sviluppo preferito:

### Opzioni di Configurazione dell'Ambiente:

#### Opzione A: GitHub Codespaces (Consigliato)

**Inizia a programmare in 2 minuti - nessuna configurazione locale richiesta!**

1. Fai il fork di questo repository nel tuo account GitHub  
   > **Nota**: Se vuoi modificare la configurazione di base dai un'occhiata a [Dev Container Configuration](../../../.devcontainer/devcontainer.json)
2. Clicca su **Code** → scheda **Codespaces** → **...** → **New with options...**
3. Usa le impostazioni predefinite – questo selezionerà la **configurazione del dev container**: **Generative AI Java Development Environment** devcontainer personalizzato creato per questo corso
4. Clicca su **Create codespace**
5. Aspetta circa 2 minuti che l'ambiente sia pronto
6. Procedi a [Passo 2: Provisiona Azure AI Foundry](#passo-2-provisiona-azure-ai-foundry)

<img src="../../../translated_images/it/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: sottomenu Codespaces" width="50%">

<img src="../../../translated_images/it/image.833552b62eee7766.webp" alt="Screenshot: New with options" width="50%">

<img src="../../../translated_images/it/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: opzioni per creare codespace" width="50%">


> **Vantaggi dei Codespaces**:
> - Nessuna installazione locale richiesta
> - Funziona su qualsiasi dispositivo con un browser
> - Pre-configurato con tutti gli strumenti e le dipendenze
> - 60 ore gratis al mese per account personali
> - Ambiente coerente per tutti gli studenti

#### Opzione B: Container di Sviluppo Locale

**Per sviluppatori che preferiscono lo sviluppo locale con Docker**

1. Fai il fork e clona questo repository sulla tua macchina locale  
   > **Nota**: Se vuoi modificare la configurazione di base dai un'occhiata a [Dev Container Configuration](../../../.devcontainer/devcontainer.json)
2. Installa [Docker Desktop](https://www.docker.com/products/docker-desktop/) e [VS Code](https://code.visualstudio.com/)
3. Installa l'[estensione Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) in VS Code
4. Apri la cartella del repository in VS Code
5. Quando richiesto, clicca su **Reopen in Container** (o usa `Ctrl+Shift+P` → "Dev Containers: Reopen in Container")
6. Attendi che il container sia costruito e avviato
7. Procedi a [Passo 2: Provisiona Azure AI Foundry](#passo-2-provisiona-azure-ai-foundry)

<img src="../../../translated_images/it/devcontainer.21126c9d6de64494.webp" alt="Screenshot: configurazione dev container" width="50%">

<img src="../../../translated_images/it/image-3.bf93d533bbc84268.webp" alt="Screenshot: build dev container completato" width="50%">

#### Opzione C: Usa la Tua Installazione Locale Esistente

**Per sviluppatori con ambienti Java già configurati**

Prerequisiti:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html)
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) o il tuo IDE preferito

Passaggi:
1. Clona questo repository sulla tua macchina locale
2. Apri il progetto nel tuo IDE
3. Procedi a [Passo 2: Provisiona Azure AI Foundry](#passo-2-provisiona-azure-ai-foundry)

> **Consiglio Pro**: Se hai un computer con specifiche basse ma vuoi usare VS Code localmente, usa GitHub Codespaces! Puoi connettere la tua istanza locale di VS Code a un Codespace ospitato nel cloud per avere il meglio di entrambi i mondi.

<img src="../../../translated_images/it/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: istanza locale devcontainer creata" width="50%">


## Passo 2: Provisiona Azure AI Foundry

Distribuisci i modelli AI del corso su Azure AI Foundry come codice. Dalla root del repository:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```

`azd` ti chiede un nome per l'ambiente e una regione, provisiona un account Azure AI Foundry con le distribuzioni `gpt-4o-mini` e `text-embedding-3-small`, e scrive l'endpoint nel file `.env` dell'esempio — tutto con autenticazione **senza chiavi** (nessuna chiave API).

> **Guida completa:** Consulta la [Guida alla Configurazione di Azure AI Foundry](getting-started-azure-openai.md) per prerequisiti, un'alternativa manuale (portale), indicazioni sulla regione, e note su costi/pulizia.

## Passo 3: Testa la Tua Configurazione

Una volta che i modelli Foundry sono provisionati, testa la connessione con l'app di esempio in [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure).

1. Apri il terminale nel tuo ambiente di sviluppo.
2. Naviga nella cartella dell'esempio:
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
3. Assicurati di aver effettuato l'accesso (l'autenticazione senza chiavi richiede un token):
   ```bash
   az login
   ```
   > Se hai eseguito `azd up`, il file `.env` con il tuo endpoint è già stato scritto per te.
4. Esegui l'applicazione:
   ```bash
   mvn clean spring-boot:run
   ```

Dovresti vedere una risposta dal modello `gpt-4o-mini`.

### Comprendere il Codice di Esempio

L'esempio sotto `examples/basic-chat-azure` è un'app Spring Boot che usa **Spring AI** per connettersi ad Azure AI Foundry con autenticazione senza chiavi.

**Cosa fa questo codice:**
- **Si connette** ad Azure AI Foundry usando il tuo accesso Azure (Microsoft Entra ID) — nessuna chiave API
- **Invia** un prompt al modello `gpt-4o-mini`
- **Riceve** e mostra la risposta dell'AI
- **Verifica** che la configurazione funzioni correttamente

**Dipendenza Chiave** (in `pom.xml`):
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```

**Configurazione** (`application.yml`):
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

## Sommario

Perfetto! Ora hai tutto configurato:

- Modelli Azure AI Foundry provisionati come codice con Bicep + `azd`
- Ambiente di sviluppo Java attivo (Codespaces, dev container, o locale)
- Collegato a Azure AI Foundry con autenticazione senza chiavi (Microsoft Entra ID) — nessuna chiave API
- Testato tutto con un semplice esempio che comunica col modello

## Prossimi Passi

[Capitolo 3: Tecniche Core di Generative AI](../03-CoreGenerativeAITechniques/README.md)

## Risoluzione dei Problemi

Hai problemi? Ecco i problemi comuni e le soluzioni:

- **Autenticazione fallita (401/403)?**  
  - Esegui `az login` — l'autenticazione è senza chiavi, devi essere loggato
  - Verifica che il tuo account abbia il ruolo **Cognitive Services OpenAI User** sulla risorsa
  - Se hai appena provisionato, aspetta un minuto che l'assegnazione ruolo si propaghi

- **Maven non trovato?**  
  - Se usi dev container/Codespaces, Maven dovrebbe essere già installato
  - Per configurazione locale, assicurati di avere Java 21+ e Maven 3.9+ installati
  - Prova `mvn --version` per verificare

- **`azd` non trovato o provisioning fallisce?**  
  - Installa l'[Azure Developer CLI](https://aka.ms/azure-dev/install) ed esegui `azd auth login`
  - Scegli una regione dove `gpt-4o-mini` è disponibile (es. `eastus2`)
  - Consulta la [guida alla configurazione Azure AI Foundry](getting-started-azure-openai.md) per dettagli

- **Il dev container non parte?**  
  - Verifica che Docker Desktop sia in esecuzione (per sviluppo locale)
  - Prova a ricostruire il container: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **Errori di compilazione dell'applicazione?**
  - Assicurati di essere nella directory corretta: `02-SetupDevEnvironment/examples/basic-chat-azure`
  - Prova a pulire e ricompilare: `mvn clean compile`

> **Hai bisogno di aiuto?**: Hai ancora problemi? Apri un issue nel repository e ti aiuteremo.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Disclaimer**:
Questo documento è stato tradotto utilizzando il servizio di traduzione AI [Co-op Translator](https://github.com/Azure/co-op-translator). Sebbene ci impegniamo per garantire la precisione, si prega di notare che le traduzioni automatizzate possono contenere errori o imprecisioni. Il documento originale nella sua lingua nativa deve essere considerato la fonte autorevole. Per informazioni critiche, si raccomanda una traduzione professionale effettuata da un essere umano. Non siamo responsabili per eventuali malintesi o interpretazioni errate derivanti dall’uso di questa traduzione.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->