# MCP Rechner Tutorial für Einsteiger

## Inhaltsverzeichnis

- [Was du lernen wirst](#was-du-lernen-wirst)
- [Voraussetzungen](#voraussetzungen)
- [Das Projektstruktur verstehen](#das-projektstruktur-verstehen)
- [Wesentliche Komponenten erklärt](#wesentliche-komponenten-erklärt)
  - [1. Hauptanwendung](#1-hauptanwendung)
  - [2. Rechner-Service](#2-rechner-service)
  - [3. Direkter MCP-Client](#3-direkter-mcp-client)
  - [4. KI-unterstützter Client](#4-ki-unterstützter-client)
- [Die Beispiele ausführen](#die-beispiele-ausführen)
- [Wie alles zusammen funktioniert](#wie-alles-zusammen-funktioniert)
- [Nächste Schritte](#nächste-schritte)

## Was du lernen wirst

Dieses Tutorial erklärt, wie man einen Rechner-Service mit dem Model Context Protocol (MCP) erstellt. Du wirst verstehen:

- Wie man einen Service erstellt, den KI als Werkzeug verwenden kann
- Wie man direkte Kommunikation mit MCP-Services einrichtet
- Wie KI-Modelle automatisch entscheiden können, welche Werkzeuge verwendet werden
- Der Unterschied zwischen direkten Protokollaufrufen und KI-gestützten Interaktionen

## Voraussetzungen

Bevor du beginnst, stelle sicher, dass du hast:
- Java 21 oder höher installiert
- Maven für das Abhängigkeitsmanagement
- Eine Azure AI Foundry Modellbereitstellung (bereitgestellt mit `azd up` — siehe [Kapitel 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- Die [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), angemeldet mit `az login` (schlüssellose Authentifizierung)
- Grundlegendes Verständnis von Java und Spring Boot

## Das Projektstruktur verstehen

Das Rechner-Projekt enthält mehrere wichtige Dateien:

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

## Wesentliche Komponenten erklärt

### 1. Hauptanwendung

**Datei:** `McpServerApplication.java`

Dies ist der Einstiegspunkt unseres Rechner-Services. Es handelt sich um eine Standard-Spring-Boot-Anwendung mit einer speziellen Ergänzung:

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
- Startet einen Spring Boot Webserver auf Port 8080
- Erstellt einen `ToolCallbackProvider`, der unsere Rechner-Methoden als MCP-Werkzeuge verfügbar macht
- Die `@Bean`-Annotation sagt Spring, diese Komponente zu verwalten, damit andere Teile sie nutzen können

### 2. Rechner-Service

**Datei:** `CalculatorService.java`

Hier findet die ganze Mathematik statt. Jede Methode ist mit `@Tool` markiert, um sie über MCP verfügbar zu machen:

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
2. **Klare Beschreibungen**: Jedes Tool hat eine Beschreibung, die KI-Modellen hilft zu verstehen, wann es verwendet werden soll
3. **Konsistentes Rückgabeformat**: Alle Operationen liefern menschenlesbare Strings wie „5.00 + 3.00 = 8.00“
4. **Fehlerbehandlung**: Division durch Null und negative Quadratwurzeln geben Fehlermeldungen zurück

**Verfügbare Operationen:**
- `add(a, b)` – Addiert zwei Zahlen
- `subtract(a, b)` – Subtrahiert zweite von erster Zahl
- `multiply(a, b)` – Multipliziert zwei Zahlen
- `divide(a, b)` – Teilt erste durch zweite Zahl (mit Null-Prüfung)
- `power(base, exponent)` – Erhöht Basis auf die Potenz Exponent
- `squareRoot(number)` – Berechnet Quadratwurzel (mit Negativ-Prüfung)
- `modulus(a, b)` – Gibt den Rest der Division zurück
- `absolute(number)` – Gibt den Absolutwert zurück
- `help()` – Liefert Informationen über alle Operationen

### 3. Direkter MCP-Client

**Datei:** `SDKClient.java`

Dieser Client kommuniziert direkt mit dem MCP-Server ohne KI zu verwenden. Er ruft manuell spezifische Rechnerfunktionen auf:

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
        
        // Bestimmte Taschenrechnerfunktionen aufrufen
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
1. **Verbindet** sich zum Rechner-Server unter `http://localhost:8080` mit dem Builder-Pattern
2. **Listet** alle verfügbaren Werkzeuge (unsere Rechnerfunktionen) auf
3. **Ruft auf** spezifische Funktionen mit exakten Parametern auf
4. **Gibt** die Ergebnisse direkt aus

**Hinweis:** Dieses Beispiel nutzt die Spring AI 1.1.0-SNAPSHOT-Dependency, die ein Builder-Pattern für den `WebFluxSseClientTransport` eingeführt hat. Wenn du eine ältere stabile Version nutzt, musst du möglicherweise den direkten Konstruktor verwenden.

**Wann du das benutzt:** Wenn du genau weißt, welche Berechnung du ausführen möchtest und sie programmatisch aufrufen willst.

### 4. KI-unterstützter Client

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

        // Verbinden Sie sich mit unserem Rechner-MCP-Server
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Zeigt, was die KI gerade macht
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Gewähren Sie der KI Zugriff auf unsere Rechner-Tools
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Erstellen Sie einen KI-Bot, der unseren Rechner verwenden kann
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
1. **Erstellt** eine KI-Modell-Verbindung mit deinem GitHub-Token
2. **Verbindet** die KI mit unserem Rechner-MCP-Server
3. **Gibt** der KI Zugriff auf alle unsere Rechner-Werkzeuge
4. **Ermöglicht** natürlichsprachige Anfragen wie „Berechne die Summe von 24,5 und 17,3“

**Die KI macht automatisch:**
- Versteht, dass du Zahlen addieren möchtest
- Wählt das Tool `add`
- Ruft `add(24.5, 17.3)` auf
- Gibt das Ergebnis in einer natürlichen Antwort zurück

## Die Beispiele ausführen

### Schritt 1: Rechner-Server starten

Melde dich zuerst an und setze deinen Azure AI Foundry Endpunkt (benötigt für den KI-Client — schlüssellose Authentifizierung, kein API-Schlüssel):

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

Starte den Server:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

Der Server läuft auf `http://localhost:8080`. Du solltest sehen:
```
Started McpServerApplication in X.XXX seconds
```

### Schritt 2: Mit direktem Client testen

In einem **NEUEN** Terminal, während der Server noch läuft, starte den direkten MCP-Client:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Du siehst eine Ausgabe wie:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Schritt 3: Mit KI-Client testen

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Du siehst, wie die KI automatisch Werkzeuge verwendet:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Schritt 4: MCP-Server schließen

Wenn du fertig bist, kannst du den KI-Client mit `Ctrl+C` im Terminal stoppen. Der MCP-Server läuft weiter, bis du ihn stoppst.
Um den Server zu beenden, drücke `Ctrl+C` im Terminal, in dem er läuft.

## Wie alles zusammen funktioniert

Hier ist der komplette Ablauf, wenn du die KI fragst: „Was ist 5 + 3?“:

1. **Du** stellst der KI eine Frage in natürlicher Sprache
2. **Die KI** analysiert die Anfrage und erkennt, dass du Addition möchtest
3. **Die KI** ruft den MCP-Server auf: `add(5.0, 3.0)`
4. **Der Rechner-Service** führt die Rechnung aus: `5.0 + 3.0 = 8.0`
5. **Der Rechner-Service** gibt zurück: `"5.00 + 3.00 = 8.00"`
6. **Die KI** erhält das Ergebnis und formuliert eine natürliche Antwort
7. **Du** bekommst: „Die Summe von 5 und 3 ist 8“

## Nächste Schritte

Für weitere Beispiele siehe [Kapitel 04: Praktische Beispiele](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Haftungsausschluss**:
Dieses Dokument wurde mit dem KI-Übersetzungsdienst [Co-op Translator](https://github.com/Azure/co-op-translator) übersetzt. Obwohl wir uns um Genauigkeit bemühen, beachten Sie bitte, dass automatisierte Übersetzungen Fehler oder Ungenauigkeiten enthalten können. Das Originaldokument in seiner Ursprungssprache gilt als maßgebliche Quelle. Bei kritischen Informationen wird eine professionelle menschliche Übersetzung empfohlen. Wir übernehmen keine Haftung für Missverständnisse oder Fehlinterpretationen, die aus der Verwendung dieser Übersetzung entstehen.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->