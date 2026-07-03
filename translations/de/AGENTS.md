# AGENTS.md

## Projektübersicht

Dies ist ein edukatives Repository zum Erlernen der Generativen KI-Entwicklung mit Java. Es bietet einen umfassenden praktischen Kurs, der große Sprachmodelle (LLMs), Prompt Engineering, Einbettungen, RAG (Retrieval-Augmented Generation) und das Model Context Protocol (MCP) abdeckt.

**Schlüsseltechnologien:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI und OpenAI SDKs

**Architektur:**
- Mehrere eigenständige Spring Boot-Anwendungen, nach Kapiteln gegliedert
- Beispielprojekte, die verschiedene KI-Muster zeigen
- GitHub Codespaces-ready mit vorkonfigurierten Dev-Containern

## Setup Befehle

### Voraussetzungen
- Java 21 oder höher
- Maven 3.x
- Azure-Abonnement mit einem Azure AI Foundry Modell-Deployment (bereitstellen mit `azd up`)
- Azure CLI (`az`) und Azure Developer CLI (`azd`), angemeldet für schlüssellose Authentifizierung

### Umgebungseinrichtung

**Option 1: GitHub Codespaces (Empfohlen)**
```bash
# Forken Sie das Repository und erstellen Sie einen Codespace über die GitHub-Benutzeroberfläche
# Der Dev-Container installiert automatisch alle Abhängigkeiten
# Warten Sie etwa 2 Minuten auf die Einrichtung der Umgebung
```

**Option 2: Lokaler Dev Container**
```bash
# Repository klonen
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# In VS Code mit der Dev Containers-Erweiterung öffnen
# Beim Prompt im Container erneut öffnen
```

**Option 3: Lokale Einrichtung**
```bash
# Abhängigkeiten installieren
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Installation überprüfen
java -version
mvn -version
```

### Konfiguration

**Azure AI Foundry Einrichtung (schlüssellos, empfohlen):**
```bash
# Bereitstellung des Foundry-Kontos + Modellbereitstellungen als Code
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd schreibt examples/basic-chat-azure/.env mit deinem Endpunkt (kein Schlüssel)
```

**Manuelle Endpunkt-Konfiguration:**
```bash
# Wenn Sie azd nicht verwendet haben, legen Sie den Endpunkt selbst fest (die Authentifizierung bleibt schlüssel- und schlüssellos über az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Bearbeiten Sie .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Entwicklungsablauf

### Projektstruktur
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

### Anwendungen starten

**Eine Spring Boot-Anwendung ausführen:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Ein Projekt bauen:**
```bash
cd [project-directory]
mvn clean install
```

**MCP Calculator Server starten:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# Server läuft auf http://localhost:8080
```

**Client-Beispiele ausführen:**
```bash
# Nachdem der Server in einem anderen Terminal gestartet wurde
cd 04-PracticalSamples/calculator

# Direkter MCP-Client
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# KI-gestützter Client (erfordert AZURE_OPENAI_ENDPOINT + az login)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Interaktiver Bot
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Hot Reload
Spring Boot DevTools sind in Projekten enthalten, die Hot Reload unterstützen:
```bash
# Änderungen an Java-Dateien werden beim Speichern automatisch neu geladen
mvn spring-boot:run
```

## Testanweisungen

### Tests ausführen

**Alle Tests in einem Projekt ausführen:**
```bash
cd [project-directory]
mvn test
```

**Tests mit ausführlicher Ausgabe ausführen:**
```bash
mvn test -X
```

**Bestimmte Testklasse ausführen:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Teststruktur
- Testdateien nutzen JUnit 5 (Jupiter)
- Testklassen befinden sich in `src/test/java/`
- Client-Beispiele im Calculator-Projekt sind in `src/test/java/com/microsoft/mcp/sample/client/`

### Manuelles Testen
Viele Beispiele sind interaktive Anwendungen, die manuell getestet werden müssen:

1. Anwendung mit `mvn spring-boot:run` starten
2. Endpunkte testen oder mit der CLI interagieren
3. Überprüfen, ob das erwartete Verhalten mit der Dokumentation in der README.md jedes Projekts übereinstimmt

### Testen mit Azure AI Foundry
- Vor Ausführung der Beispiele mit `az login` anmelden (schlüssellose Authentifizierung)
- Sicherstellen, dass Ihr Konto die Rolle Cognitive Services OpenAI User auf der Ressource besitzt
- Inhaltsfilterung mit dem Responsible AI Beispiel in Kapitel 5 testen

## Code Style Richtlinien

### Java-Konventionen
- **Java-Version:** Java 21 mit modernen Features
- **Stil:** Standard-Java-Konventionen folgen
- **Benennung:** 
  - Klassen: PascalCase
  - Methoden/Variablen: camelCase
  - Konstanten: UPPER_SNAKE_CASE
  - Paketnamen: kleingeschrieben

### Spring Boot Muster
- Verwenden von `@Service` für Geschäftslogik
- Verwenden von `@RestController` für REST-Endpunkte
- Konfiguration über `application.yml` oder `application.properties`
- Umgebungsvariablen bevorzugt gegenüber hartcodierten Werten
- `@Tool` Annotation für MCP-exponierte Methoden verwenden

### Dateiorganisation
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

### Abhängigkeiten
- Verwaltet über Maven `pom.xml`
- Spring AI BOM für Versionsmanagement
- LangChain4j für KI-Integrationen
- Spring Boot Starter Parent für Spring-Abhängigkeiten

### Code-Kommentare
- JavaDoc für öffentliche APIs hinzufügen
- Erläuternde Kommentare für komplexe KI-Interaktionen einfügen
- MCP-Toolbeschreibungen klar dokumentieren

## Build und Deployment

### Projekte bauen

**Build ohne Tests:**
```bash
mvn clean install -DskipTests
```

**Build mit allen Prüfungen:**
```bash
mvn clean install
```

**Anwendung paketieren:**
```bash
mvn package
# Erstellt JAR im Verzeichnis target/
```

### Zielverzeichnisse
- Kompilierte Klassen: `target/classes/`
- Testklassen: `target/test-classes/`
- JAR-Dateien: `target/*.jar`
- Maven-Artefakte: `target/`

### Umgebungsbezogene Konfiguration

**Entwicklung:**
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

**Produktiv:**
- Statt `az login` für schlüssellose Authentifizierung verwaltete Identität verwenden
- `AZURE_OPENAI_ENDPOINT` auf Ihre produktive Foundry-Ressource verweisen
- Konfiguration über Umgebungsvariablen oder Azure Key Vault verwalten

### Deployment-Hinweise
- Dies ist ein edukatives Repository mit Beispielanwendungen
- Nicht für den produktiven Einsatz als Standalone gedacht
- Beispiele demonstrieren Muster, die für die Produktion angepasst werden können
- Für spezifische Deployment-Hinweise Einzelprojekte-README lesen

## Zusätzliche Hinweise

### Azure AI Foundry
- **Schlüssellose Authentifizierung:** Verbindung über Microsoft Entra ID — keine API-Schlüsselverwaltung erforderlich
- **Bereitstellung als Code:** Bicep + azd (`azd up`) erstellen Konto und Modell-Deployments
- Der selbe OpenAI-kompatible Code läuft lokal (`az login`) und in Azure (verwaltete Identität)

### Arbeiten mit mehreren Projekten
Jedes Beispielprojekt ist eigenständig:
```bash
# Navigiere zu einem bestimmten Projekt
cd 04-PracticalSamples/[project-name]

# Jedes hat seine eigene pom.xml und kann unabhängig gebaut werden
mvn clean install
```

### Häufige Probleme

**Java-Version stimmt nicht überein:**
```bash
# Überprüfen Sie Java 21
java -version
# Aktualisieren Sie JAVA_HOME bei Bedarf
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Probleme beim Herunterladen von Abhängigkeiten:**
```bash
# Maven-Cache löschen und erneut versuchen
rm -rf ~/.m2/repository
mvn clean install
```

**Endpunkt oder Anmeldung nicht gefunden:**
```bash
# Setze den Endpunkt in der aktuellen Sitzung und melde dich an (schlüssellos)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Oder verwende eine .env-Datei im Projektverzeichnis
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Port bereits in Verwendung:**
```bash
# Spring Boot verwendet standardmäßig Port 8080
# Änderung in application.properties:
server.port=8081
```

### Mehrsprachige Unterstützung
- Dokumentation in über 45 Sprachen durch automatisierte Übersetzung verfügbar
- Übersetzungen im Verzeichnis `translations/`
- Übersetzungsmanagement per GitHub Actions Workflow

### Lernpfad
1. Beginnen mit [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. Kapitel der Reihenfolge nach folgen (01 → 05)
3. Praktische Beispiele in jedem Kapitel abschließen
4. Beispielprojekte in Kapitel 4 erkunden
5. Responsible AI Praktiken in Kapitel 5 lernen

### Entwicklungscontainer
Die `.devcontainer/devcontainer.json` konfiguriert:
- Java 21 Entwicklungsumgebung
- Vorinstalliertes Maven
- VS Code Java-Erweiterungen
- Spring Boot Tools
- GitHub Copilot Integration
- Docker-in-Docker Unterstützung
- Azure CLI

### Performance-Überlegungen
- Azure AI Foundry Deployments haben Token-/Anfragekontingente pro Minute
- Passende Batchgrößen für Embeddings verwenden
- Caching für wiederholte API-Aufrufe in Betracht ziehen
- Token-Nutzung zur Kostenoptimierung überwachen

### Sicherheitshinweise
- Nie `.env`-Dateien committen (bereits in `.gitignore`)
- Schlüssellose Authentifizierung (Microsoft Entra ID) bevorzugen gegenüber API-Schlüsseln
- Verwaltete Identitäten in Azure nutzen; `az login` für lokale Entwicklung
- Verantwortungsbewusste KI-Richtlinien in Kapitel 5 befolgen

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Haftungsausschluss**:
Dieses Dokument wurde mit dem KI-Übersetzungsdienst [Co-op Translator](https://github.com/Azure/co-op-translator) übersetzt. Obwohl wir uns um Genauigkeit bemühen, beachten Sie bitte, dass automatisierte Übersetzungen Fehler oder Ungenauigkeiten enthalten können. Das Originaldokument in seiner Ursprungssprache gilt als maßgebliche Quelle. Bei kritischen Informationen wird eine professionelle menschliche Übersetzung empfohlen. Wir übernehmen keine Haftung für Missverständnisse oder Fehlinterpretationen, die aus der Verwendung dieser Übersetzung entstehen.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->