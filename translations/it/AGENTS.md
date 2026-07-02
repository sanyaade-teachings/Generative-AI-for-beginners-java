# AGENTS.md

## Panoramica del Progetto

Questo è un repository didattico per imparare lo sviluppo di Generative AI con Java. Fornisce un corso pratico completo che copre Large Language Models (LLM), ingegneria dei prompt, embeddings, RAG (Retrieval-Augmented Generation) e il Model Context Protocol (MCP).

**Tecnologie Chiave:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI e OpenAI SDK

**Architettura:**
- Molteplici applicazioni Spring Boot standalone organizzate per capitoli
- Progetti di esempio che dimostrano diversi pattern AI
- Pronto per GitHub Codespaces con contenitori dev preconfigurati

## Comandi di Configurazione

### Prerequisiti
- Java 21 o superiore
- Maven 3.x
- Abbonamento Azure con un deployment modello Azure AI Foundry (provisionato con `azd up`)
- Azure CLI (`az`) e Azure Developer CLI (`azd`), con accesso effettuato per autenticazione senza chiavi

### Configurazione Ambiente

**Opzione 1: GitHub Codespaces (Consigliato)**
```bash
# Effettua il fork del repository e crea un codespace dall'interfaccia GitHub
# Il container di sviluppo installerà automaticamente tutte le dipendenze
# Attendi circa 2 minuti per la configurazione dell'ambiente
```

**Opzione 2: Container Dev Locale**
```bash
# Clona il repository
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Apri in VS Code con l'estensione Dev Containers
# Riapri nel container quando richiesto
```

**Opzione 3: Configurazione Locale**
```bash
# Installa le dipendenze
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Verifica l'installazione
java -version
mvn -version
```

### Configurazione

**Configurazione Azure AI Foundry (senza chiavi, consigliato):**
```bash
# Provisionare l'account Foundry + distribuzioni del modello come codice
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd scrive examples/basic-chat-azure/.env con il tuo endpoint (senza chiave)
```

**Config manuale endpoint:**
```bash
# Se non hai utilizzato azd, imposta tu stesso l'endpoint (l'autenticazione rimane senza chiave tramite az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Modifica .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Workflow di Sviluppo

### Struttura del Progetto
```
/
├── 01-IntroToGenAI/              # Chapter 1: Introduction
├── 02-SetupDevEnvironment/       # Chapter 2: Environment setup
│   └── examples/                 # Working examples
├── 03-CoreGenerativeAITechniques/ # Chapter 3: Core techniques
├── 04-PracticalSamples/          # Chapter 4: Sample projects
│   ├── calculator/               # MCP service example
│   ├── foundrylocal/            # Local model integration
│   └── petstory/                # Multi-modal app
├── 05-ResponsibleGenAI/         # Chapter 5: Responsible AI
└── translations/                # Multi-language support
```

### Avvio Applicazioni

**Esecuzione di un'applicazione Spring Boot:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Costruzione di un progetto:**
```bash
cd [project-directory]
mvn clean install
```

**Avvio del Server MCP Calculator:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# Il server gira su http://localhost:8080
```

**Esecuzione degli esempi Client:**
```bash
# Dopo aver avviato il server in un altro terminale
cd 04-PracticalSamples/calculator

# Client MCP diretto
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# Client potenziato da AI (richiede AZURE_OPENAI_ENDPOINT + az login)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Bot interattivo
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Hot Reload
Spring Boot DevTools è incluso nei progetti che supportano il caricamento a caldo:
```bash
# Le modifiche ai file Java verranno ricaricate automaticamente al salvataggio
mvn spring-boot:run
```

## Istruzioni per il Testing

### Esecuzione dei Test

**Esegui tutti i test di un progetto:**
```bash
cd [project-directory]
mvn test
```

**Esegui test con output dettagliato:**
```bash
mvn test -X
```

**Esegui una specifica classe di test:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Struttura dei Test
- I file di test utilizzano JUnit 5 (Jupiter)
- Le classi di test si trovano in `src/test/java/`
- Gli esempi client nel progetto calculator sono in `src/test/java/com/microsoft/mcp/sample/client/`

### Testing Manuale
Molti esempi sono applicazioni interattive che richiedono test manuali:

1. Avvia l'applicazione con `mvn spring-boot:run`
2. Testa gli endpoint o interagisci con la CLI
3. Verifica che il comportamento atteso corrisponda alla documentazione in ogni README.md del progetto

### Testing con Azure AI Foundry
- Effettua il login con `az login` prima di eseguire gli esempi (autenticazione senza chiavi)
- Assicurati che il tuo account abbia il ruolo Cognitive Services OpenAI User sulla risorsa
- Testa il filtro dei contenuti con l'esempio di responsible AI nel Capitolo 5

## Linee Guida di Stile del Codice

### Convenzioni Java
- **Versione Java:** Java 21 con funzionalità moderne
- **Stile:** Seguire le convenzioni Java standard
- **Nomenclatura:** 
  - Classi: PascalCase
  - Metodi/variabili: camelCase
  - Costanti: UPPER_SNAKE_CASE
  - Nomi di package: minuscolo

### Pattern Spring Boot
- Usare `@Service` per la logica di business
- Usare `@RestController` per gli endpoint REST
- Configurazione tramite `application.yml` o `application.properties`
- Preferire variabili ambiente rispetto a valori hard-coded
- Usare l'annotazione `@Tool` per i metodi esposti da MCP

### Organizzazione File
```
src/
├── main/
│   ├── java/
│   │   └── com/microsoft/[component]/
│   │       ├── [Component]Application.java
│   │       ├── config/
│   │       ├── controller/
│   │       ├── service/
│   │       └── exception/
│   └── resources/
│       ├── application.yml
│       └── static/
└── test/
    └── java/
        └── com/microsoft/[component]/
```

### Dipendenze
- Gestite tramite Maven `pom.xml`
- Spring AI BOM per la gestione delle versioni
- LangChain4j per integrazioni AI
- Spring Boot starter parent per dipendenze Spring

### Commenti nel Codice
- Aggiungere JavaDoc per API pubbliche
- Includere commenti esplicativi per interazioni AI complesse
- Documentare chiaramente le descrizioni degli strumenti MCP

## Build e Distribuzione

### Costruzione dei Progetti

**Build senza test:**
```bash
mvn clean install -DskipTests
```

**Build con tutti i controlli:**
```bash
mvn clean install
```

**Pacchettizzazione applicazione:**
```bash
mvn package
# Crea il JAR nella directory target/
```

### Cartelle di Output
- Classi compilate: `target/classes/`
- Classi di test: `target/test-classes/`
- File JAR: `target/*.jar`
- Artefatti Maven: `target/`

### Configurazione Specifica per Ambiente

**Sviluppo:**
```yaml
# application.yml (keyless - no api-key; auth via DefaultAzureCredential)
spring:
  ai:
    azure:
      openai:
        endpoint: ${AZURE_OPENAI_ENDPOINT}
        chat:
          options:
            deployment-name: ${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}
```

**Produzione:**
- Usare un'identità gestita invece di `az login` per l'autenticazione senza chiavi
- Impostare `AZURE_OPENAI_ENDPOINT` sulla risorsa Foundry di produzione
- Gestire la configurazione tramite variabili ambiente o Azure Key Vault

### Considerazioni per la Distribuzione
- Questo è un repository didattico con applicazioni di esempio
- Non è progettato per la distribuzione in produzione così com’è
- Gli esempi dimostrano pattern da adattare per uso in produzione
- Consultare i README dei singoli progetti per note specifiche sulla distribuzione

## Note Aggiuntive

### Azure AI Foundry
- **Autenticazione senza chiavi:** connessione con Microsoft Entra ID — nessuna chiave API da gestire
- **Provisioning come codice:** Bicep + azd (`azd up`) creano account e deployment modello
- Lo stesso codice compatibile OpenAI funziona localmente (`az login`) e su Azure (identità gestita)

### Lavorare con Progetti Multipli
Ogni progetto di esempio è standalone:
```bash
# Naviga al progetto specifico
cd 04-PracticalSamples/[project-name]

# Ognuno ha il proprio pom.xml e può essere costruito indipendentemente
mvn clean install
```

### Problemi Comuni

**Versione Java non corrispondente:**
```bash
# Verificare Java 21
java -version
# Aggiornare JAVA_HOME se necessario
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Problemi di download dipendenze:**
```bash
# Cancella la cache di Maven e riprova
rm -rf ~/.m2/repository
mvn clean install
```

**Endpoint o accesso non trovati:**
```bash
# Impostare l'endpoint nella sessione corrente ed effettuare l'accesso (senza chiave)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Oppure utilizzare un file .env nella directory del progetto
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Porta già in uso:**
```bash
# Spring Boot usa la porta 8080 di default
# Modifica in application.properties:
server.port=8081
```

### Supporto Multilingua
- Documentazione disponibile in oltre 45 lingue tramite traduzione automatica
- Traduzioni nella cartella `translations/`
- Traduzione gestita tramite workflow GitHub Actions

### Percorso di Apprendimento
1. Inizia con [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. Segui i capitoli in ordine (01 → 05)
3. Completa gli esempi pratici in ogni capitolo
4. Esplora i progetti di esempio nel Capitolo 4
5. Impara le pratiche di responsible AI nel Capitolo 5

### Container di Sviluppo
Il file `.devcontainer/devcontainer.json` configura:
- Ambiente di sviluppo Java 21
- Maven preinstallato
- Estensioni Java per VS Code
- Strumenti Spring Boot
- Integrazione GitHub Copilot
- Supporto Docker-in-Docker
- Azure CLI

### Considerazioni sulle Prestazioni
- I deployment Azure AI Foundry hanno quote token/richiesta al minuto
- Usare dimensioni batch appropriate per gli embeddings
- Considerare caching per chiamate API ripetute
- Monitorare l’uso dei token per ottimizzare i costi

### Note sulla Sicurezza
- Non committare mai file `.env` (già in `.gitignore`)
- Preferire autenticazione senza chiavi (Microsoft Entra ID) invece delle chiavi API
- Usare identità gestite su Azure; `az login` per sviluppo locale
- Seguire le linee guida di responsible AI nel Capitolo 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Disclaimer**:
Questo documento è stato tradotto utilizzando il servizio di traduzione AI [Co-op Translator](https://github.com/Azure/co-op-translator). Sebbene ci impegniamo per garantire la precisione, si prega di notare che le traduzioni automatizzate possono contenere errori o imprecisioni. Il documento originale nella sua lingua nativa deve essere considerato la fonte autorevole. Per informazioni critiche, si raccomanda una traduzione professionale effettuata da un essere umano. Non siamo responsabili per eventuali malintesi o interpretazioni errate derivanti dall’uso di questa traduzione.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->