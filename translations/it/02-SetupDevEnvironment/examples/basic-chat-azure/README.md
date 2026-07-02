# Chat di Base con Azure AI Foundry - Esempio End-to-End

Questo esempio è una semplice applicazione Spring Boot che si collega a un modello **Azure AI Foundry** utilizzando l’**autenticazione senza chiave** (Microsoft Entra ID) e testa la tua configurazione. Utilizza il `ChatClient` di Spring AI.

## Indice

- [Prerequisiti](#prerequisiti)
- [Avvio rapido](#avvio-rapido)
- [Come funziona l'autenticazione](#come-funziona-lautenticazione)
- [Esecuzione dell'applicazione](#esecuzione-dellapplicazione)
  - [Utilizzo di Maven](#utilizzo-di-maven)
  - [Utilizzo di VS Code](#utilizzo-di-vs-code)
  - [Output previsto](#output-previsto)
- [Riferimento di configurazione](#riferimento-di-configurazione)
  - [Variabili d'ambiente](#variabili-dambiente)
  - [Configurazione Spring](#configurazione-spring)
- [Risoluzione dei problemi](#risoluzione-dei-problemi)
  - [Problemi comuni](#problemi-comuni)
  - [Modalità di debug](#modalità-di-debug)
- [Passaggi successivi](#passaggi-successivi)
- [Risorse](#risorse)

## Prerequisiti

Prima di eseguire questo esempio, assicurati di avere:

- Una risorsa Azure AI Foundry con un deployment `gpt-4o-mini` — creala con `azd up` o manualmente tramite la [guida di configurazione di Azure AI Foundry](../../getting-started-azure-openai.md)
- Il ruolo **Cognitive Services OpenAI User** su quella risorsa (i template Bicep lo assegnano automaticamente)
- L’[Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), con accesso effettuato tramite `az login`
- Java 21+ e Maven 3.9+

> **Nessuna chiave API richiesta** — l’autenticazione è senza chiave tramite Microsoft Entra ID.

## Avvio rapido

```bash
# 1. Naviga al progetto
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Accedi in modo che l'autenticazione senza chiave possa ottenere un token
az login

# 3. Configura il punto di accesso
#    - Se hai eseguito `azd up`, .env è stato scritto per te (salta questo passaggio).
#    - Altrimenti copia il modello e imposta AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. Esegui l'applicazione
mvn spring-boot:run
```

## Come funziona l'autenticazione

Questo esempio si autentica con **Microsoft Entra ID** — non c’è alcuna chiave API.

Quando è impostato solo `spring.ai.azure.openai.endpoint` (e nessuna api-key), Spring AI crea il client Azure OpenAI con [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). Questa credenziale trova automaticamente un token dalla sessione locale `az login`, oppure da un’identità gestita se eseguito in Azure — così lo stesso codice funziona in entrambi gli ambienti senza modifiche.

## Esecuzione dell'applicazione

### Utilizzo di Maven

```bash
mvn spring-boot:run
```

### Utilizzo di VS Code

1. Apri il progetto in VS Code  
2. Premi `F5` o usa il pannello "Esegui e Debug"  
3. Seleziona la configurazione "Spring Boot-BasicChatApplication"

> **Nota**: La configurazione di VS Code carica automaticamente il tuo file .env

### Output previsto

```
Starting Basic Chat with Azure OpenAI...
Environment variables loaded successfully
Connecting to Azure OpenAI...
Sending prompt: What is AI in a short sentence? Max 100 words.

AI Response:
================
AI, or Artificial Intelligence, is the simulation of human intelligence in machines programmed to think and learn like humans.
================

Success! Azure OpenAI connection is working correctly.
```

## Riferimento di configurazione

### Variabili d'ambiente

| Variabile | Descrizione | Obbligatorio | Esempio |
|----------|-------------|--------------|---------|
| `AZURE_OPENAI_ENDPOINT` | URL endpoint Foundry (Azure OpenAI) | Sì | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Nome del deployment del modello chat | No | `gpt-4o-mini` (predefinito) |

> Non esiste nessuna variabile chiave API — l’autenticazione è senza chiave (Microsoft Entra ID tramite `az login`).

### Configurazione Spring

Il file `application.yml` configura:  
- **Endpoint**: `${AZURE_OPENAI_ENDPOINT}` - Dalla variabile d'ambiente  
- **Deployment**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - Dalla variabile d'ambiente con valore di fallback  
- **Autenticazione**: senza chiave — nessuna `api-key` impostata, quindi Spring AI usa `DefaultAzureCredential`  
- **Temperatura**: `0.7` - Controlla la creatività (0.0 = deterministico, 1.0 = creativo)  
- **Token massimi**: `500` - Lunghezza massima della risposta  

## Risoluzione dei problemi

### Problemi comuni

<details>
<summary><strong>Errore: 401 / "PermissionDenied" / errori di token</strong></summary>

- Esegui `az login` — l’autenticazione senza chiave necessita di un accesso attivo per ottenere un token  
- Verifica che il tuo account abbia il ruolo **Cognitive Services OpenAI User** sulla risorsa  
- Se hai appena assegnato il ruolo, attendi un minuto per la propagazione  
- Controlla di essere nel tenant/sottoscrizione corretti (`az account show`)  
</details>

<details>
<summary><strong>Errore: "L’endpoint non è valido" / errori di connessione</strong></summary>

- Assicurati che `AZURE_OPENAI_ENDPOINT` sia l’URL completo di base (es: `https://your-resource.openai.azure.com/`)  
- Controlla la coerenza della barra finale  
- Verifica che l’endpoint corrisponda alla risorsa provisionata (`azd env get-values`)  
</details>

<details>
<summary><strong>Errore: "Il deployment non è stato trovato"</strong></summary>

- Verifica che `AZURE_OPENAI_DEPLOYMENT` corrisponda a un nome di deployment in Azure  
- Controlla che il modello sia stato distribuito correttamente e sia attivo  
- Il nome predefinito del deployment è `gpt-4o-mini`  
</details>

<details>
<summary><strong>VS Code: le variabili d’ambiente non vengono caricate</strong></summary>

- Assicurati che il file `.env` si trovi nella directory radice del progetto (stesso livello di `pom.xml`)  
- Prova a eseguire `mvn spring-boot:run` nel terminale integrato di VS Code  
- Verifica che l’estensione Java per VS Code sia installata correttamente  
</details>

### Modalità di debug

Per abilitare il logging dettagliato, rimuovi il commento da queste righe nel `application.yml`:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Passaggi successivi

**Configurazione completata!** Continua il tuo percorso di apprendimento:

[Capitolo 3: Tecniche generative AI di base](../../../03-CoreGenerativeAITechniques/README.md)

## Risorse

- [Documentazione Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)  
- [Autenticazione senza chiave con Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)  
- [Portale Azure AI Foundry](https://ai.azure.com/)  
- [Documentazione Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Disclaimer**:
Questo documento è stato tradotto utilizzando il servizio di traduzione AI [Co-op Translator](https://github.com/Azure/co-op-translator). Sebbene ci impegniamo per garantire la precisione, si prega di notare che le traduzioni automatizzate possono contenere errori o imprecisioni. Il documento originale nella sua lingua nativa deve essere considerato la fonte autorevole. Per informazioni critiche, si raccomanda una traduzione professionale effettuata da un essere umano. Non siamo responsabili per eventuali malintesi o interpretazioni errate derivanti dall’uso di questa traduzione.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->