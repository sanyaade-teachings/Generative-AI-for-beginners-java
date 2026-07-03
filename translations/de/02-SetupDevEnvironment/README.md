# Einrichtung der Entwicklungsumgebung für Generative KI für Java

> **Schnellstart:** Stellen Sie Ihre KI-Modelle auf **Azure AI Foundry** als Code mit Bicep + `azd` in wenigen Minuten bereit – siehe die [Azure AI Foundry Setup-Anleitung](getting-started-azure-openai.md). Die Authentifizierung ist **schlüssellos** (Microsoft Entra ID), daher müssen keine API-Schlüssel verwaltet werden.

## Was Sie lernen werden

- Einrichtung einer Java-Entwicklungsumgebung für KI-Anwendungen
- Auswahl und Konfiguration Ihrer bevorzugten Entwicklungsumgebung (Cloud-first mit Codespaces, lokaler Dev Container oder vollständige lokale Einrichtung)
- Testen Ihrer Einrichtung durch Verbindung zu einem Azure AI Foundry-Modell

## Inhaltsverzeichnis

- [Was Sie lernen werden](#was-sie-lernen-werden)
- [Einführung](#einführung)
- [Schritt 1: Einrichten Ihrer Entwicklungsumgebung](#schritt-1-einrichten-ihrer-entwicklungsumgebung)
  - [Option A: GitHub Codespaces (Empfohlen)](#option-a-github-codespaces-empfohlen)
  - [Option B: Lokaler Dev Container](#option-b-lokaler-dev-container)
  - [Option C: Verwendung Ihrer vorhandenen lokalen Installation](#option-c-verwendung-ihrer-vorhandenen-lokalen-installation)
- [Schritt 2: Azure AI Foundry bereitstellen](#schritt-2-azure-ai-foundry-bereitstellen)
- [Schritt 3: Testen Ihrer Einrichtung](#schritt-3-testen-ihrer-einrichtung)
- [Fehlerbehebung](#fehlerbehebung)
- [Zusammenfassung](#zusammenfassung)
- [Nächste Schritte](#nächste-schritte)

## Einführung

Dieses Kapitel führt Sie durch die Einrichtung einer Entwicklungsumgebung. Für die Modelle verwenden wir in diesem Kurs **Azure AI Foundry**. Sie stellen die Modelle als Code mit Bicep und der Azure Developer CLI (`azd`) bereit und verbinden sich dann mit **schlüsselloser Authentifizierung** (Microsoft Entra ID) – es müssen keine API-Schlüssel kopiert oder geleakt werden.

**Keine lokale Einrichtung erforderlich!** Sie können GitHub Codespaces nutzen, das eine vollwertige Entwicklungsumgebung im Browser bietet, und von dort Foundry bereitstellen.

Wir verwenden **Azure AI Foundry** für diesen Kurs, weil es:
- **Als Code bereitgestellt** wird — ein `azd up` deployt das Konto und die Model-Deployments
- **Schlüssellos** ist — Authentifizierung über Ihre Azure-Anmeldung oder eine verwaltete Identität
- **Produktionsreif** ist — derselbe Code läuft lokal und in Azure
- **Flexibel** ist — Modelle können durch Änderung eines Deployment-Namens getauscht werden, nicht des Codes

> **Hinweis**: Azure AI Foundry-Bereitstellungen werden pro Token abgerechnet (pay-as-you-go). Details zu Bereitstellung, Region und Kosten finden Sie in der [Azure AI Foundry Setup-Anleitung](getting-started-azure-openai.md).

## Schritt 1: Einrichten Ihrer Entwicklungsumgebung

<a name="quick-start-cloud"></a>

Wir haben einen vorkonfigurierten Entwicklungscontainer erstellt, um die Einrichtungszeit zu minimieren und sicherzustellen, dass Sie alle notwendigen Werkzeuge für diesen Generative AI for Java Kurs haben. Wählen Sie Ihre bevorzugte Entwicklungsvariante:

### Optionen zur Umgebungs-Einrichtung:

#### Option A: GitHub Codespaces (Empfohlen)

**Starten Sie in 2 Minuten mit dem Programmieren – keine lokale Einrichtung nötig!**

1. Forken Sie dieses Repository auf Ihr GitHub-Konto
   > **Hinweis**: Wenn Sie die Basiskonfiguration ändern möchten, werfen Sie bitte einen Blick auf die [Dev Container Konfiguration](../../../.devcontainer/devcontainer.json)
2. Klicken Sie auf **Code** → Tab **Codespaces** → **...** → **Neu mit Optionen...**
3. Verwenden Sie die Standardwerte – hierbei wird die **Dev Container-Konfiguration** ausgewählt: **Generative AI Java Development Environment**, ein speziell für diesen Kurs erstellter Devcontainer
4. Klicken Sie auf **Codespace erstellen**
5. Warten Sie ca. 2 Minuten, bis die Umgebung bereit ist
6. Fahren Sie mit [Schritt 2: Azure AI Foundry bereitstellen](#schritt-2-azure-ai-foundry-bereitstellen) fort

<img src="../../../translated_images/de/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: Codespaces-Untermenü" width="50%">

<img src="../../../translated_images/de/image.833552b62eee7766.webp" alt="Screenshot: Neu mit Optionen" width="50%">

<img src="../../../translated_images/de/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: Codespace-Erstellungsoptionen" width="50%">


> **Vorteile von Codespaces**:
> - Keine lokale Installation notwendig
> - Funktioniert auf jedem Gerät mit Browser
> - Vorgefertigt mit allen Werkzeugen und Abhängigkeiten
> - Kostenlose 60 Stunden pro Monat für persönliche Accounts
> - Einheitliche Umgebung für alle Lernenden

#### Option B: Lokaler Dev Container

**Für Entwickler, die gerne lokal mit Docker entwickeln**

1. Forken und klonen Sie dieses Repository auf Ihren lokalen Rechner
   > **Hinweis**: Wenn Sie die Basiskonfiguration ändern möchten, werfen Sie bitte einen Blick auf die [Dev Container Konfiguration](../../../.devcontainer/devcontainer.json)
2. Installieren Sie [Docker Desktop](https://www.docker.com/products/docker-desktop/) und [VS Code](https://code.visualstudio.com/)
3. Installieren Sie die [Dev Containers Erweiterung](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) in VS Code
4. Öffnen Sie den Repository-Ordner in VS Code
5. Wenn Sie dazu aufgefordert werden, klicken Sie auf **Im Container neu öffnen** (oder verwenden Sie `Ctrl+Shift+P` → "Dev Containers: Im Container neu öffnen")
6. Warten Sie, bis der Container gebaut und gestartet ist
7. Fahren Sie mit [Schritt 2: Azure AI Foundry bereitstellen](#schritt-2-azure-ai-foundry-bereitstellen) fort

<img src="../../../translated_images/de/devcontainer.21126c9d6de64494.webp" alt="Screenshot: Dev Container Einrichtung" width="50%">

<img src="../../../translated_images/de/image-3.bf93d533bbc84268.webp" alt="Screenshot: Dev Container Build abgeschlossen" width="50%">

#### Option C: Verwendung Ihrer vorhandenen lokalen Installation

**Für Entwickler mit vorhandener Java-Umgebung**

Voraussetzungen:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html) 
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) oder Ihre bevorzugte IDE

Schritte:
1. Klonen Sie dieses Repository auf Ihren lokalen Rechner
2. Öffnen Sie das Projekt in Ihrer IDE
3. Fahren Sie mit [Schritt 2: Azure AI Foundry bereitstellen](#schritt-2-azure-ai-foundry-bereitstellen) fort

> **Profi-Tipp**: Wenn Sie eine leistungsschwache Maschine haben, aber VS Code lokal verwenden möchten, nutzen Sie GitHub Codespaces! Sie können Ihre lokale VS Code-Instanz mit einem cloud-gehosteten Codespace verbinden und erhalten das Beste aus beiden Welten.

<img src="../../../translated_images/de/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: Erstellte lokale Devcontainer-Instanz" width="50%">

## Schritt 2: Azure AI Foundry bereitstellen

Stellen Sie die KI-Modelle des Kurses als Code in Azure AI Foundry bereit. Vom Repository-Stammverzeichnis aus:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```
  
`azd` fragt nach einem Umgebungsnamen und einer Region, stellt ein Azure AI Foundry-Konto mit den Deployments `gpt-4o-mini` und `text-embedding-3-small` bereit und schreibt den Endpunkt in die `.env` der Beispielanwendung — alles mit **schlüsselloser** Authentifizierung (keine API-Schlüssel).

> **Vollständige Anleitung:** Siehe die [Azure AI Foundry Setup-Anleitung](getting-started-azure-openai.md) für Voraussetzungen, eine manuelle (Portal-)Alternative, Regionsrichtlinien und Hinweise zu Kosten/Aufräumen.

## Schritt 3: Testen Ihrer Einrichtung

Nachdem Ihre Foundry-Modelle bereitgestellt sind, testen Sie die Verbindung mit der Beispielanwendung in [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure).

1. Öffnen Sie das Terminal in Ihrer Entwicklungsumgebung.
2. Navigieren Sie zum Beispiel:
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
3. Stellen Sie sicher, dass Sie angemeldet sind (schlüssellose Authentifizierung benötigt ein Token):
   ```bash
   az login
   ```
   > Wenn Sie `azd up` ausgeführt haben, wurde die `.env` mit Ihrem Endpunkt bereits für Sie angelegt.
4. Starten Sie die Anwendung:
   ```bash
   mvn clean spring-boot:run
   ```
  
Sie sollten eine Antwort vom Modell `gpt-4o-mini` sehen.

### Verständnis des Beispielcodes

Das Beispiel in `examples/basic-chat-azure` ist eine Spring Boot-Anwendung, die **Spring AI** verwendet, um mit Azure AI Foundry mit schlüsselloser Authentifizierung zu verbinden.

**Was dieser Code macht:**
- **Verbindet** sich mit Azure AI Foundry über Ihre Azure-Anmeldung (Microsoft Entra ID) – ohne API-Schlüssel
- **Sendet** eine Eingabeaufforderung (Prompt) an das Modell `gpt-4o-mini`
- **Empfängt** und zeigt die Antwort der KI an
- **Validiert**, dass Ihre Einrichtung korrekt funktioniert

**Wichtige Abhängigkeit** (in `pom.xml`):
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```
  
**Konfiguration** (`application.yml`):
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
  
## Zusammenfassung

Klasse! Sie haben jetzt alles eingerichtet:

- Azure AI Foundry-Modelle als Code mit Bicep + `azd` bereitgestellt
- Ihre Java-Entwicklungsumgebung läuft (egal ob Codespaces, Dev Container oder lokal)
- Verbindung zu Azure AI Foundry mit schlüsselloser Authentifizierung (Microsoft Entra ID) – keine API-Schlüssel
- Getestet, dass alles mit einem einfachen Beispiel funktioniert, das mit Ihrem Modell kommuniziert

## Nächste Schritte

[Kapitel 3: Kerntechniken der generativen KI](../03-CoreGenerativeAITechniques/README.md)

## Fehlerbehebung

Probleme? Hier sind häufige Probleme und Lösungen:

- **Authentifizierung schlägt fehl (401/403)?**
  - Führen Sie `az login` aus – die Authentifizierung ist schlüssellos, daher müssen Sie angemeldet sein
  - Vergewissern Sie sich, dass Ihr Konto die Rolle **Cognitive Services OpenAI User** für die Ressource hat
  - Wenn Sie gerade bereitgestellt haben, warten Sie eine Minute, bis die Rollenvergabe übernommen wurde

- **Maven nicht gefunden?**
  - Bei Verwendung von Dev Containers/Codespaces sollte Maven vorinstalliert sein
  - Für lokale Einrichtung: Stellen Sie sicher, dass Java 21+ und Maven 3.9+ installiert sind
  - Versuchen Sie `mvn --version`, um die Installation zu überprüfen

- **`azd` nicht gefunden oder Bereitstellung schlägt fehl?**
  - Installieren Sie die [Azure Developer CLI](https://aka.ms/azure-dev/install) und führen Sie `azd auth login` aus
  - Wählen Sie eine Region, in der `gpt-4o-mini` verfügbar ist (z.B. `eastus2`)
  - Details siehe die [Azure AI Foundry Setup-Anleitung](getting-started-azure-openai.md)

- **Dev Container startet nicht?**
  - Stellen Sie sicher, dass Docker Desktop läuft (für lokale Entwicklung)
  - Versuchen Sie den Container neu zu erstellen: `Ctrl+Shift+P` → "Dev Containers: Container neu erstellen"

- **Kompilierungsfehler in der Anwendung?**
  - Stellen Sie sicher, dass Sie im richtigen Verzeichnis sind: `02-SetupDevEnvironment/examples/basic-chat-azure`
  - Versuchen Sie, das Projekt zu reinigen und neu zu bauen: `mvn clean compile`

> **Brauchen Sie Hilfe?**: Haben Sie weiterhin Probleme? Öffnen Sie ein Issue im Repository, wir helfen Ihnen gerne weiter.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Haftungsausschluss**:
Dieses Dokument wurde mit dem KI-Übersetzungsdienst [Co-op Translator](https://github.com/Azure/co-op-translator) übersetzt. Obwohl wir uns um Genauigkeit bemühen, beachten Sie bitte, dass automatisierte Übersetzungen Fehler oder Ungenauigkeiten enthalten können. Das Originaldokument in seiner Ursprungssprache gilt als maßgebliche Quelle. Bei kritischen Informationen wird eine professionelle menschliche Übersetzung empfohlen. Wir übernehmen keine Haftung für Missverständnisse oder Fehlinterpretationen, die aus der Verwendung dieser Übersetzung entstehen.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->