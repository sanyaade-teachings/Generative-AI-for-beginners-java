# Tutorial sul Calcolatore MCP per Principianti

## Indice

- [Cosa Imparerai](#cosa-imparerai)
- [Prerequisiti](#prerequisiti)
- [Comprendere la Struttura del Progetto](#comprendere-la-struttura-del-progetto)
- [Spiegazione dei Componenti Principali](#spiegazione-dei-componenti-principali)
  - [1. Applicazione Principale](#1-applicazione-principale)
  - [2. Servizio Calcolatore](#2-servizio-calcolatore)
  - [3. Client MCP Diretto](#3-client-mcp-diretto)
  - [4. Client Alimentato da AI](#4-client-alimentato-da-ai)
- [Esecuzione degli Esempi](#esecuzione-degli-esempi)
- [Come Funziona Tutto Insieme](#come-funziona-tutto-insieme)
- [Prossimi Passi](#prossimi-passi)

## Cosa Imparerai

Questo tutorial spiega come costruire un servizio calcolatore utilizzando il Model Context Protocol (MCP). Capirai:

- Come creare un servizio che l'AI può usare come strumento
- Come configurare la comunicazione diretta con servizi MCP
- Come i modelli AI possono scegliere automaticamente quali strumenti usare
- La differenza tra chiamate dirette al protocollo e interazioni assistite dall'AI

## Prerequisiti

Prima di iniziare, assicurati di avere:
- Java 21 o superiore installato
- Maven per la gestione delle dipendenze
- Un deployment di modello Azure AI Foundry (provisiona con `azd up` — vedi [Capitolo 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- La [CLI di Azure](https://learn.microsoft.com/cli/azure/install-azure-cli), effettuato il login con `az login` (autenticazione senza chiave)
- Conoscenze base di Java e Spring Boot

## Comprendere la Struttura del Progetto

Il progetto calcolatore ha diversi file importanti:

```
calculator/
├── src/main/java/com/microsoft/mcp/sample/server/
│   ├── McpServerApplication.java          # Main Spring Boot app
│   └── service/CalculatorService.java     # Calculator operations
└── src/test/java/com/microsoft/mcp/sample/client/
    ├── SDKClient.java                     # Direct MCP communication
    ├── LangChain4jClient.java            # AI-powered client
    └── Bot.java                          # Simple chat interface
```

## Spiegazione dei Componenti Principali

### 1. Applicazione Principale

**File:** `McpServerApplication.java`

Questo è il punto di ingresso del nostro servizio calcolatore. È un'applicazione Spring Boot standard con un'aggiunta speciale:

```java
@SpringBootApplication
public class McpServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(McpServerApplication.class, args);
    }
    
    @Bean
    public ToolCallbackProvider calculatorTools(CalculatorService calculator) {
        return MethodToolCallbackProvider.builder().toolObjects(calculator).build();
    }
}
```

**Cosa fa:**
- Avvia un server web Spring Boot sulla porta 8080
- Crea un `ToolCallbackProvider` che rende disponibili i nostri metodi calcolatore come strumenti MCP
- L'annotazione `@Bean` dice a Spring di gestirlo come componente che altre parti possono utilizzare

### 2. Servizio Calcolatore

**File:** `CalculatorService.java`

Qui avviene tutta la matematica. Ogni metodo è annotato con `@Tool` per renderlo disponibile tramite MCP:

```java
@Service
public class CalculatorService {

    @Tool(description = "Add two numbers together")
    public String add(double a, double b) {
        double result = a + b;
        return formatResult(a, "+", b, result);
    }

    @Tool(description = "Subtract the second number from the first number")
    public String subtract(double a, double b) {
        double result = a - b;
        return formatResult(a, "-", b, result);
    }
    
    // Altre operazioni della calcolatrice...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Caratteristiche principali:**

1. **Annotazione `@Tool`**: Dice a MCP che questo metodo può essere chiamato da client esterni
2. **Descrizioni Chiare**: Ogni strumento ha una descrizione che aiuta i modelli AI a capire quando usarlo
3. **Formato di Ritorno Consistente**: Tutte le operazioni restituiscono stringhe leggibili come "5.00 + 3.00 = 8.00"
4. **Gestione Errori**: Divisione per zero e radice quadrata di numeri negativi restituiscono messaggi di errore

**Operazioni disponibili:**
- `add(a, b)` - Somma due numeri
- `subtract(a, b)` - Sottrae il secondo dal primo
- `multiply(a, b)` - Moltiplica due numeri
- `divide(a, b)` - Divide il primo per il secondo (controllo zero)
- `power(base, exponent)` - Eleva la base alla potenza dell'esponente
- `squareRoot(number)` - Calcola la radice quadrata (controllo negativo)
- `modulus(a, b)` - Restituisce il resto della divisione
- `absolute(number)` - Restituisce il valore assoluto
- `help()` - Restituisce informazioni su tutte le operazioni

### 3. Client MCP Diretto

**File:** `SDKClient.java`

Questo client comunica direttamente con il server MCP senza usare AI. Chiama manualmente funzioni specifiche del calcolatore:

```java
public class SDKClient {
    
    public static void main(String[] args) {
        McpClientTransport transport = WebFluxSseClientTransport.builder(
            WebClient.builder().baseUrl("http://localhost:8080")
        ).build();
        new SDKClient(transport).run();
    }
    
    public void run() {
        var client = McpClient.sync(this.transport).build();
        client.initialize();
        
        // Elenca gli strumenti disponibili
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Chiama funzioni specifiche della calcolatrice
        CallToolResult resultAdd = client.callTool(
            new CallToolRequest("add", Map.of("a", 5.0, "b", 3.0))
        );
        System.out.println("Add Result = " + resultAdd);
        
        CallToolResult resultSqrt = client.callTool(
            new CallToolRequest("squareRoot", Map.of("number", 16.0))
        );
        System.out.println("Square Root Result = " + resultSqrt);
        
        client.closeGracefully();
    }
}
```

**Cosa fa:**
1. **Si connette** al server calcolatore su `http://localhost:8080` usando il pattern builder
2. **Elenca** tutti gli strumenti disponibili (le nostre funzioni calcolatore)
3. **Chiama** funzioni specifiche con parametri esatti
4. **Stampa** i risultati direttamente

**Nota:** Questo esempio usa la dipendenza Spring AI 1.1.0-SNAPSHOT, che ha introdotto il pattern builder per `WebFluxSseClientTransport`. Se usi una versione stabile più vecchia, potresti dover usare il costruttore diretto.

**Quando usarlo:** Quando sai esattamente quale calcolo vuoi eseguire e vuoi chiamarlo programmaticamente.

### 4. Client Alimentato da AI

**File:** `LangChain4jClient.java`

Questo client usa un modello AI (GPT-4o-mini) che può decidere automaticamente quali strumenti calcolatore usare:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Configura il modello AI (Azure AI Foundry, autenticazione senza chiave tramite Microsoft Entra ID)
        String endpoint = System.getenv("AZURE_OPENAI_ENDPOINT");
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1";
        String token = new DefaultAzureCredentialBuilder().build()
                .getToken(new TokenRequestContext().addScopes("https://ai.azure.com/.default"))
                .block().getToken();
        ChatLanguageModel model = OpenAiOfficialChatModel.builder()
                .baseUrl(baseUrl)
                .apiKey(token)
                .modelName("gpt-4o-mini")
                .build();

        // Connetti al nostro server calcolatore MCP
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Mostra cosa sta facendo l'AI
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Dai all'AI accesso ai nostri strumenti calcolatore
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Crea un bot AI che può usare il nostro calcolatore
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Ora possiamo chiedere all'AI di fare calcoli in linguaggio naturale
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Cosa fa:**
1. **Crea** una connessione al modello AI usando autenticazione senza chiave (Microsoft Entra ID)
2. **Collega** l'AI al nostro server MCP calcolatore
3. **Dà** all'AI accesso a tutti i nostri strumenti calcolatore
4. **Permette** richieste in linguaggio naturale come "Calcola la somma di 24.5 e 17.3"

**L'AI automaticamente:**
- Capisce che vuoi sommare numeri
- Sceglie lo strumento `add`
- Chiama `add(24.5, 17.3)`
- Restituisce il risultato in una risposta naturale

## Esecuzione degli Esempi

### Passo 1: Avvia il Server Calcolatore

Prima, effettua il login e imposta il tuo endpoint Azure AI Foundry (necessario per il client AI — autenticazione senza chiave, nessuna API Key):

**Windows:**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**Linux/macOS:**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

Avvia il server:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

Il server si avvierà su `http://localhost:8080`. Dovresti vedere:
```
Started McpServerApplication in X.XXX seconds
```

### Passo 2: Testa con il Client Diretto

In un terminale **NUOVO** con il server ancora in esecuzione, esegui il client MCP diretto:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Vedrai un output come:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Passo 3: Testa con il Client AI

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Vedrai l'AI usare automaticamente gli strumenti:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Passo 4: Chiudi il Server MCP

Quando hai finito di testare, puoi fermare il client AI premendo `Ctrl+C` nel suo terminale. Il server MCP continuerà a funzionare finché non lo fermerai.
Per fermare il server, premi `Ctrl+C` nel terminale dove è in esecuzione.

## Come Funziona Tutto Insieme

Ecco il flusso completo quando chiedi all'AI "Quanto fa 5 + 3?":

1. **Tu** chiedi all'AI in linguaggio naturale
2. **L'AI** analizza la richiesta e capisce che vuoi sommare
3. **L'AI** chiama il server MCP: `add(5.0, 3.0)`
4. **Il Servizio Calcolatore** esegue: `5.0 + 3.0 = 8.0`
5. **Il Servizio Calcolatore** restituisce: `"5.00 + 3.00 = 8.00"`
6. **L'AI** riceve il risultato e crea una risposta naturale
7. **Tu** ottieni: "La somma di 5 e 3 è 8"

## Prossimi Passi

Per altri esempi, vedi [Capitolo 04: Esempi pratici](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Disclaimer**:
Questo documento è stato tradotto utilizzando il servizio di traduzione AI [Co-op Translator](https://github.com/Azure/co-op-translator). Sebbene ci impegniamo per garantire la precisione, si prega di notare che le traduzioni automatizzate possono contenere errori o imprecisioni. Il documento originale nella sua lingua nativa deve essere considerato la fonte autorevole. Per informazioni critiche, si raccomanda una traduzione professionale effettuata da un essere umano. Non siamo responsabili per eventuali malintesi o interpretazioni errate derivanti dall’uso di questa traduzione.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->