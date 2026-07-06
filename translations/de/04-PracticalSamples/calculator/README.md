# MCP Rechner Tutorial für Anfänger

## Inhaltsverzeichnis

- [Was Sie lernen werden](#was-sie-lernen-werden)
- [Voraussetzungen](#voraussetzungen)
- [Das Projektstruktur verstehen](#das-projektstruktur-verstehen)
- [Erklärung der Kernkomponenten](#erkl%C3%A4rung-der-kernkomponenten)
  - [1. Hauptanwendung](#1-hauptanwendung)
  - [2. Rechner-Service](#2-rechner-service)
  - [3. Direkter MCP-Client](#3-direkter-mcp-client)
  - [4. KI-basierter Client](#4-ki-basierter-client)
- [Ausführen der Beispiele](#ausf%C3%BChren-der-beispiele)
- [So funktioniert das Zusammenspiel](#so-funktioniert-das-zusammenspiel)
- [Nächste Schritte](#n%C3%A4chste-schritte)

## Was Sie lernen werden

Dieses Tutorial erklärt, wie man einen Rechner-Service mit dem Model Context Protocol (MCP) erstellt. Sie verstehen:

- Wie man einen Service erstellt, den KI als Werkzeug nutzen kann
- Wie man direkte Kommunikation mit MCP-Diensten einrichtet
- Wie KI-Modelle automatisch entscheiden, welche Werkzeuge verwendet werden
- Den Unterschied zwischen direkten Protokollaufrufen und KI-gestützten Interaktionen

## Voraussetzungen

Bevor Sie starten, stellen Sie sicher, dass Sie:
- Java 21 oder höher installiert haben
- Maven für das Abhängigkeitsmanagement nutzen
- Eine Azure AI Foundry Modellbereitstellung haben (stellen Sie sie mit `azd up` bereit — siehe [Kapitel 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- Die [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) installiert und mit `az login` angemeldet (authentifizierung ohne Schlüssel)
- Grundkenntnisse in Java und Spring Boot besitzen

## Das Projektstruktur verstehen

Das Rechnerprojekt hat mehrere wichtige Dateien:

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

## Erklärung der Kernkomponenten

### 1. Hauptanwendung

**Datei:** `McpServerApplication.java`

Dies ist der Einstiegspunkt unseres Rechner-Services. Es handelt sich um eine Standard-Spring-Boot-Anwendung mit einer besonderen Ergänzung:

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

**Was das bewirkt:**
- Startet einen Spring-Boot-Webserver auf Port 8080
- Erstellt einen `ToolCallbackProvider`, der unsere Rechner-Methoden als MCP-Werkzeuge verfügbar macht
- Die `@Bean`-Annotation teilt Spring mit, dass dies eine verwaltete Komponente ist, die andere Teile nutzen können

### 2. Rechner-Service

**Datei:** `CalculatorService.java`

Hier findet die gesamte Mathematik statt. Jede Methode ist mit `@Tool` versehen, um sie über MCP verfügbar zu machen:

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
    
    // Weitere Taschenrechneroperationen...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Wichtige Merkmale:**

1. **`@Tool` Annotation**: Signalisiert MCP, dass diese Methode von externen Clients aufgerufen werden kann
2. **Klare Beschreibungen**: Jedes Werkzeug hat eine Beschreibung, die KI-Modellen hilft zu verstehen, wann es verwendet werden soll
3. **Konsistentes Rückgabeformat**: Alle Operationen geben gut lesbare Strings zurück wie „5.00 + 3.00 = 8.00“
4. **Fehlerbehandlung**: Division durch Null und negative Quadratwurzeln geben Fehlermeldungen zurück

**Verfügbare Operationen:**
- `add(a, b)` - Addiert zwei Zahlen
- `subtract(a, b)` - Subtrahiert die zweite von der ersten Zahl
- `multiply(a, b)` - Multipliziert zwei Zahlen
- `divide(a, b)` - Dividiert die erste durch die zweite (mit Nullprüfung)
- `power(base, exponent)` - Potenziert Basis mit Exponent
- `squareRoot(number)` - Berechnet die Quadratwurzel (mit Negativprüfung)
- `modulus(a, b)` - Gibt den Rest der Division zurück
- `absolute(number)` - Gibt den Absolutwert zurück
- `help()` - Gibt Informationen über alle Operationen zurück

### 3. Direkter MCP-Client

**Datei:** `SDKClient.java`

Dieser Client kommuniziert direkt mit dem MCP-Server, ohne KI zu verwenden. Er ruft manuell bestimmte Rechnerfunktionen auf:

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
        
        // Verfügbare Werkzeuge auflisten
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Spezifische Taschenrechnerfunktionen aufrufen
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

**Was das bewirkt:**
1. **Verbindet** sich über das Builder-Pattern mit dem Rechner-Server unter `http://localhost:8080`
2. **Listet** alle verfügbaren Werkzeuge (unsere Rechnerfunktionen) auf
3. **Ruft** bestimmte Funktionen mit exakten Parametern auf
4. **Gibt** die Ergebnisse direkt aus

**Hinweis:** Dieses Beispiel verwendet die Spring AI 1.1.0-SNAPSHOT-Abhängigkeit, welche ein Builder-Pattern für den `WebFluxSseClientTransport` eingeführt hat. Falls Sie eine ältere stabile Version verwenden, müssen Sie möglicherweise den direkten Konstruktor verwenden.

**Wann verwenden:** Wenn Sie genau wissen, welche Berechnung Sie ausführen möchten und diese programmatisch aufrufen wollen.

### 4. KI-basierter Client

**Datei:** `LangChain4jClient.java`

Dieser Client verwendet ein KI-Modell (GPT-4o-mini), das automatisch entscheidet, welche Rechner-Werkzeuge verwendet werden:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Richten Sie das KI-Modell ein (Azure AI Foundry, schlüssellose Authentifizierung über Microsoft Entra ID)
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

        // Verbinden Sie sich mit unserem Calculator MCP-Server
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Zeigt, was die KI gerade macht
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Geben Sie der KI Zugriff auf unsere Calculator-Werkzeuge
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Erstellen Sie einen KI-Bot, der unseren Calculator verwenden kann
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Jetzt können wir die KI bitten, Berechnungen in natürlicher Sprache durchzuführen
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Was das bewirkt:**
1. **Erstellt** eine KI-Modellverbindung mit schlüsselloser Authentifizierung (Microsoft Entra ID)
2. **Verbindet** die KI mit unserem Rechner-MCP-Server
3. **Gibt** der KI Zugriff auf alle unsere Rechner-Werkzeuge
4. **Ermöglicht** natürliche Sprachabfragen wie „Berechne die Summe von 24,5 und 17,3“

**Die KI macht automatisch:**
- Versteht, dass Sie Zahlen addieren wollen
- Wählt das Werkzeug `add` aus
- Ruft `add(24.5, 17.3)` auf
- Gibt das Ergebnis in einer natürlichen Antwort zurück

## Ausführen der Beispiele

### Schritt 1: Starten Sie den Rechner-Server

Melden Sie sich zuerst an und setzen Sie Ihren Azure AI Foundry Endpunkt (benötigt für den KI-Client — authentifizierung ohne Schlüssel, kein API-Schlüssel):

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

Starten Sie den Server:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

Der Server startet unter `http://localhost:8080`. Sie sollten sehen:
```
Started McpServerApplication in X.XXX seconds
```

### Schritt 2: Testen mit direktem Client

In einem **NEUEN** Terminalfenster bei laufendem Server führen Sie den direkten MCP-Client aus:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Sie sehen eine Ausgabe ähnlich:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Schritt 3: Testen mit KI-Client

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Sie sehen, wie die KI automatisch Werkzeuge verwendet:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Schritt 4: Schließen Sie den MCP-Server

Wenn Sie fertig sind, können Sie den KI-Client mit `Strg+C` im Terminal stoppen. Der MCP-Server läuft weiter, bis Sie ihn stoppen.
Zum Beenden des Servers drücken Sie `Strg+C` im Terminal, in dem er läuft.

## So funktioniert das Zusammenspiel

Hier ist der komplette Ablauf, wenn Sie die KI fragen: „Was ist 5 + 3?“:

1. **Sie** fragen die KI in natürlicher Sprache
2. **Die KI** analysiert Ihre Anfrage und erkennt, dass Sie addieren wollen
3. **Die KI** ruft den MCP-Server auf: `add(5.0, 3.0)`
4. **Der Rechner-Service** führt aus: `5.0 + 3.0 = 8.0`
5. **Der Rechner-Service** gibt zurück: `"5.00 + 3.00 = 8.00"`
6. **Die KI** erhält das Ergebnis und formuliert eine natürliche Antwort
7. **Sie** bekommen: „Die Summe von 5 und 3 ist 8“

## Nächste Schritte

Für weitere Beispiele siehe [Kapitel 04: Praktische Beispiele](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Haftungsausschluss**:
Dieses Dokument wurde mit dem KI-Übersetzungsdienst [Co-op Translator](https://github.com/Azure/co-op-translator) übersetzt. Obwohl wir uns um Genauigkeit bemühen, beachten Sie bitte, dass automatisierte Übersetzungen Fehler oder Ungenauigkeiten enthalten können. Das Originaldokument in seiner Ursprungssprache gilt als maßgebliche Quelle. Bei kritischen Informationen wird eine professionelle menschliche Übersetzung empfohlen. Wir übernehmen keine Haftung für Missverständnisse oder Fehlinterpretationen, die aus der Verwendung dieser Übersetzung entstehen.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->