# Introduzione all'Intelligenza Artificiale Generativa - Edizione Java

## Cosa Imparerai

- **Fondamenti di Intelligenza Artificiale Generativa** inclusi LLM, prompt engineering, token, embedding e database vettoriali
- **Confronta gli strumenti di sviluppo AI per Java** tra cui Azure OpenAI SDK, Spring AI e OpenAI Java SDK
- **Scopri il Model Context Protocol** e il suo ruolo nella comunicazione degli agenti AI

## Sommario

- [Introduzione](#introduzione)
- [Un rapido ripasso sui concetti di Intelligenza Artificiale Generativa](#un-rapido-ripasso-sui-concetti-di-intelligenza-artificiale-generativa)
- [Revisione del prompt engineering](#revisione-del-prompt-engineering)
- [Token, embedding e agenti](#token-embedding-e-agenti)
- [Strumenti e librerie di sviluppo AI per Java](#strumenti-e-librerie-di-sviluppo-ai-per-java)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [Sintesi](#sintesi)
- [Prossimi Passi](#prossimi-passi)

## Introduzione

Benvenuto al primo capitolo di Intelligenza Artificiale Generativa per Principianti - Edizione Java! Questa lezione fondamentale ti introduce ai concetti base dell'AI generativa e come lavorare con essi usando Java. Imparerai i blocchi essenziali per costruire applicazioni AI, inclusi i Large Language Models (LLM), i token, gli embedding e gli agenti AI. Esploreremo anche gli strumenti principali Java che userai durante questo corso.

### Un rapido ripasso sui concetti di Intelligenza Artificiale Generativa

L'Intelligenza Artificiale Generativa è un tipo di intelligenza artificiale che crea nuovi contenuti, come testo, immagini o codice, basati su modelli e relazioni appresi dai dati. I modelli di AI generativa possono generare risposte simili a quelle umane, comprendere il contesto e talvolta perfino creare contenuti che sembrano umani.

Durante lo sviluppo delle tue applicazioni AI in Java, lavorerai con **modelli di AI generativa** per creare contenuti. Alcune capacità di questi modelli includono:

- **Generazione di testo**: Creazione di testo naturale per chatbot, contenuti e completamento testuale.
- **Generazione e analisi di immagini**: Produzione di immagini realistiche, miglioramento foto e riconoscimento di oggetti.
- **Generazione di codice**: Scrittura di snippet o script di codice.

Esistono tipi specifici di modelli ottimizzati per compiti differenti. Ad esempio, sia i **Small Language Models (SLM)** che i **Large Language Models (LLM)** possono gestire la generazione testuale, con gli LLM che in genere offrono migliori prestazioni per compiti complessi. Per attività legate alle immagini si usano modelli di visione specializzati o modelli multimodali.

![Figura: tipi di modelli AI generativi e casi d'uso.](../../../translated_images/it/llms.225ca2b8a0d34473.webp)

Naturalmente, le risposte di questi modelli non sono sempre perfette. Probabilmente hai sentito parlare di modelli che "allucinano" o generano informazioni errate in modo autorevole. Ma puoi guidare il modello a generare risposte migliori fornendogli istruzioni chiare e contesto. Qui entra in gioco il **prompt engineering**.

#### Revisione del prompt engineering

Il prompt engineering è la pratica di progettare input efficaci per indirizzare i modelli AI verso output desiderati. Comprende:

- **Chiarezza**: Rendere le istruzioni chiare e senza ambiguità.
- **Contesto**: Fornire le informazioni di base necessarie.
- **Vincoli**: Specificare eventuali limitazioni o formati.

Alcune best practice includono il design del prompt, istruzioni chiare, suddivisione del compito, apprendimento one-shot e few-shot e ottimizzazione del prompt. È essenziale testare diversi prompt per capire cosa funziona meglio per il tuo caso d'uso specifico.

Durante lo sviluppo di applicazioni, lavorerai con diversi tipi di prompt:
- **Prompt di sistema**: definiscono le regole base e il contesto per il comportamento del modello
- **Prompt utente**: dati di input forniti dagli utenti della tua applicazione
- **Prompt assistente**: risposte generate dal modello basate sui prompt di sistema e utente

> **Scopri di più**: Approfondisci il prompt engineering nel [capitolo Prompt Engineering del corso GenAI per Principianti](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### Token, embedding e agenti

Lavorando con i modelli di AI generativa incontrerai termini come **token**, **embedding**, **agenti** e **Model Context Protocol (MCP)**. Ecco un overview dettagliata di questi concetti:

- **Token**: I token sono l'unità minima di testo in un modello. Possono essere parole, caratteri o sottoparole. I token rappresentano i dati testuali in un formato comprensibile dal modello. Ad esempio, la frase "The quick brown fox jumped over the lazy dog" potrebbe essere tokenizzata come ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] oppure come ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"] a seconda della strategia di tokenizzazione.

![Figura: esempio di token AI generativi che suddivide le parole in token](../../../translated_images/it/tokens.6283ed277a2ffff4.webp)

La tokenizzazione è il processo che scompone il testo in queste unità più piccole. È cruciale perché i modelli operano sui token piuttosto che sul testo grezzo. Il numero di token in un prompt influisce sulla lunghezza e qualità della risposta del modello, poiché ogni modello ha un limite massimo di token per la sua finestra contestuale (es. 128K token per GPT-4o nel contesto totale, input e output inclusi).

  In Java, puoi utilizzare librerie come l’OpenAI SDK per gestire automaticamente la tokenizzazione quando invii richieste ai modelli AI.

- **Embedding**: Gli embedding sono rappresentazioni vettoriali dei token che catturano il significato semantico. Sono rappresentazioni numeriche (tipicamente array di numeri in virgola mobile) che consentono ai modelli di comprendere le relazioni tra le parole e generare risposte contestualmente rilevanti. Parole simili hanno embedding simili, permettendo al modello di capire concetti come sinonimi e relazioni semantiche.

![Figura: Embedding](../../../translated_images/it/embedding.398e50802c0037f9.webp)

  In Java, puoi generare embedding usando l’OpenAI SDK o altre librerie che supportano la generazione di embedding. Questi embedding sono essenziali per attività come la ricerca semantica, dove si vuole trovare contenuti simili basati sul significato invece che sulla corrispondenza esatta del testo.

- **Database vettoriali**: I database vettoriali sono sistemi di archiviazione specializzati ottimizzati per gli embedding. Consentono ricerche di similarità efficienti e sono fondamentali per i pattern di Generazione Aumentata da Recupero (RAG), in cui è necessario trovare informazioni rilevanti all’interno di grandi insiemi di dati basandosi sulla similarità semantica piuttosto che su corrispondenze esatte.

![Figura: architettura di un database vettoriale che mostra come gli embedding sono memorizzati e recuperati per la ricerca di similarità.](../../../translated_images/it/vector.f12f114934e223df.webp)

> **Nota**: In questo corso non tratteremo i database vettoriali, ma li menzioniamo perché sono di uso comune nelle applicazioni reali.

- **Agenti & MCP**: Componenti AI che interagiscono autonomamente con modelli, strumenti e sistemi esterni. Il Model Context Protocol (MCP) fornisce un modo standardizzato per consentire agli agenti di accedere in modo sicuro a fonti di dati esterne e strumenti. Scopri di più nel nostro corso [MCP per Principianti](https://github.com/microsoft/mcp-for-beginners).

Nelle applicazioni AI Java, userai token per la gestione del testo, embedding per la ricerca semantica e RAG, database vettoriali per il recupero dati e agenti con MCP per costruire sistemi intelligenti e integrati con strumenti.

![Figura: come un prompt diventa una risposta—token, vettori, opzionale ricerca RAG, pensiero LLM e un agente MCP, tutto in un unico flusso rapido..](../../../translated_images/it/flow.f4ef62c3052d12a8.webp)

### Strumenti e librerie di sviluppo AI per Java

Java offre ottimi strumenti per lo sviluppo AI. Tre librerie principali che esploreremo durante questo corso sono: OpenAI Java SDK, Azure OpenAI SDK e Spring AI.

Ecco una tabella di riferimento rapido che mostra quale SDK viene usato negli esempi di ogni capitolo:

| Capitolo | Esempio | SDK |
|---------|--------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**Link alla documentazione SDK:**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

L’OpenAI SDK è la libreria Java ufficiale per l’API OpenAI. Fornisce un’interfaccia semplice e coerente per interagire con i modelli OpenAI, facilitando l’integrazione delle capacità AI nelle applicazioni Java. L’applicazione Pet Story del capitolo 4 e l’esempio Foundry Local mostrano l’approccio OpenAI SDK con Azure AI Foundry.

#### Spring AI

Spring AI è un framework completo che porta le capacità AI nelle applicazioni Spring, offrendo un livello di astrazione coerente tra diversi provider AI. Si integra perfettamente con l’ecosistema Spring, rendendolo la scelta ideale per applicazioni Java aziendali che necessitano di funzionalità AI.

La forza di Spring AI sta nella sua integrazione fluida con l’ecosistema Spring, facilitando lo sviluppo di applicazioni AI pronte per la produzione con pattern familiari di Spring come l’iniezione delle dipendenze, gestione della configurazione e framework di test. Userai Spring AI nei capitoli 2 e 4 per costruire applicazioni che sfruttano sia OpenAI sia il Model Context Protocol (MCP) attraverso le librerie Spring AI.

##### Model Context Protocol (MCP)

Il [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) è uno standard emergente che consente alle applicazioni AI di interagire in modo sicuro con fonti di dati esterne e strumenti. MCP fornisce un modo standardizzato affinché i modelli AI accedano a informazioni contestuali ed eseguano azioni nelle applicazioni.

Nel Capitolo 4 costruirai un semplice servizio calcolatore MCP che dimostra i fondamenti del Model Context Protocol con Spring AI, mostrando come creare integrazioni base per strumenti e architetture di servizio.

#### Azure OpenAI Java SDK

La libreria client Azure OpenAI per Java è un’adattamento delle API REST di OpenAI che fornisce un’interfaccia idiomatica e integrazione con il resto dell’ecosistema SDK di Azure. Nel Capitolo 3 costruirai applicazioni usando Azure OpenAI SDK, incluse applicazioni di chat, chiamate a funzioni e pattern RAG (Retrieval-Augmented Generation).

> Nota: Azure OpenAI SDK è meno avanzato rispetto all’OpenAI Java SDK in termini di funzionalità, quindi per progetti futuri considera l’uso dell’OpenAI Java SDK.

## Sintesi

Ecco le basi concluse! Ora sai:

- I concetti chiave dietro l’AI generativa - dagli LLM e il prompt engineering a token, embedding e database vettoriali
- Le opzioni di toolkit per lo sviluppo AI in Java: Azure OpenAI SDK, Spring AI e OpenAI Java SDK
- Cos’è il Model Context Protocol e come consente agli agenti AI di lavorare con strumenti esterni

## Prossimi Passi

[Capitolo 2: Configurare l’Ambiente di Sviluppo](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Disclaimer**:
Questo documento è stato tradotto utilizzando il servizio di traduzione AI [Co-op Translator](https://github.com/Azure/co-op-translator). Sebbene ci impegniamo per garantire la precisione, si prega di notare che le traduzioni automatizzate possono contenere errori o imprecisioni. Il documento originale nella sua lingua nativa deve essere considerato la fonte autorevole. Per informazioni critiche, si raccomanda una traduzione professionale effettuata da un essere umano. Non siamo responsabili per eventuali malintesi o interpretazioni errate derivanti dall’uso di questa traduzione.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->