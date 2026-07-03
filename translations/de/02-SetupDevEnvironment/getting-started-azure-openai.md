# Einrichtung der Entwicklungsumgebung für Azure AI Foundry

> Diese Anleitung richtet die **Azure AI Foundry**-Modelle für die Java KI-Anwendungen in diesem Kurs ein und verwendet **schlüssellose** Authentifizierung (Microsoft Entra ID) — keine zu verwaltenden API-Schlüssel. Neu bei den Tools? Beginnen Sie mit der [Anleitung zur Entwicklungsumgebung](./README.md).

Diese Anleitung richtet die **Azure AI Foundry**-Modelle für die Java KI-Anwendungen in diesem Kurs ein. Sie haben zwei Wege:

- **Option A — Bereitstellung mit `azd` + Bicep (empfohlen):** Ein Befehl stellt das Foundry-Konto und die Modelle als Code bereit. Kein Klicken im Portal.
- **Option B — Ressourcen manuell erstellen** im Azure AI Foundry-Portal.

Beide Wege verwenden **schlüssellose Authentifizierung** (Microsoft Entra ID) — es müssen keine API-Schlüssel kopiert oder gefährdet werden.

## Inhaltsverzeichnis

- [Was wird erstellt](#was-wird-erstellt)
- [Voraussetzungen](#voraussetzungen)
- [Option A: Bereitstellung mit azd + Bicep (empfohlen)](#option-a-bereitstellung-mit-azd--bicep-empfohlen)
- [Option B: Ressourcen manuell erstellen](#option-b-ressourcen-manuell-erstellen)
- [Konfigurieren Ihrer Umgebung](#konfigurieren-ihrer-umgebung)
- [Testen Ihrer Einrichtung](#testen-ihrer-einrichtung)
- [Was kommt als nächstes?](#was-kommt-als-nächstes)
- [Ressourcen](#ressourcen)
- [Zusätzliche Ressourcen](#zusätzliche-ressourcen)

## Was wird erstellt

Die Bicep-Vorlagen in [`infra/`](../../../02-SetupDevEnvironment/infra) stellen bereit:

- Ein **Azure AI Foundry**-Konto (`Microsoft.CognitiveServices/accounts`, Art `AIServices`) mit einem Projekt
- Eine **Chat**-Bereitstellung — `gpt-4o-mini`
- Eine **Embedding**-Bereitstellung — `text-embedding-3-small` (wird in späteren Kapiteln verwendet)
- Eine **schlüssellose Rollenzuweisung** (`Cognitive Services OpenAI User`), damit Sie sich mit `az login` anmelden können, anstatt Schlüssel zu verwalten

## Voraussetzungen

- Ein [Azure-Abonnement](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) und [Maven 3.9+](https://maven.apache.org/download.cgi)

## Option A: Bereitstellung mit azd + Bicep (empfohlen)

Aus dem Ordner `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# Anmelden (beide Werkzeuge)
azd auth login
az login

# Das Foundry-Konto + Modelbereitstellungen bereitstellen
azd up
```

`azd` fragt nach einem **Umgebungsnamen** (z.B. `genai-java`) und einer **Region**. Wählen Sie eine Region, in der `gpt-4o-mini` und `text-embedding-3-small` verfügbar sind — zum Beispiel `eastus2` oder `swedencentral`.

Wenn die Bereitstellung abgeschlossen ist, führt azd aus:

1. Es stellt alles bereit, was in [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep) definiert ist.
2. Es führt einen Post-Provision-Hook aus, der [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) mit Ihrem Endpunkt und den Bereitstellungsnamen schreibt (keine Geheimnisse).

> **Tipp:** Führen Sie `azd up` jederzeit erneut aus, um Änderungen anzuwenden. Mit `azd down` löschen Sie alles und vermeiden weitere Kosten.

Um die generierten Einstellungen anzuzeigen:

```bash
azd env get-values
```

Springen Sie jetzt zu [Testen Ihrer Einrichtung](#testen-ihrer-einrichtung).

## Option B: Ressourcen manuell erstellen

Bevorzugen Sie das Portal? Erstellen Sie die Ressourcen manuell:

1. Gehen Sie zum [Azure AI Foundry-Portal](https://ai.azure.com/) und melden Sie sich an.
2. **Erstellen Sie ein Projekt** (dadurch wird auch eine AI Foundry-Ressource erstellt). Geben Sie ihm einen Namen wie `GenAIJava`.
3. Öffnen Sie in Ihrem Projekt **Modelle + Endpunkte** → **Modell bereitstellen** → **Basis-Modell bereitstellen**.
4. Stellen Sie **gpt-4o-mini** bereit (Bereitstellungsname `gpt-4o-mini`). Wiederholen Sie dies für **text-embedding-3-small**, wenn Sie die Embedding-Beispiele verwenden möchten.
5. Kopieren Sie von **Übersicht** den **Endpunkt** (z.B. `https://<resource>.openai.azure.com/`).
6. Gewähren Sie sich schlüssellosen Zugriff: Öffnen Sie auf der Ressource **Zugriffssteuerung (IAM)** → **Rollen zuweisen** → weisen Sie Ihrem Konto die Rolle **Cognitive Services OpenAI User** zu.

> **Haben Sie weiterhin Probleme?** Siehe die [Azure AI Foundry Dokumentation](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Konfigurieren Ihrer Umgebung

**Wenn Sie Option A (`azd up`) verwendet haben**, ist Ihre Einstellungsdatei bereits geschrieben — es gibt nichts zu konfigurieren. Überspringen Sie zu [Testen Ihrer Einrichtung](#testen-ihrer-einrichtung).

**Wenn Sie Option B (manuell)** verwendet haben, erstellen Sie die `.env`-Datei des Beispiels selbst:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

Bearbeiten Sie `.env` mit Ihrem Endpunkt (kein Schlüssel — die Authentifizierung ist schlüssellos):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Sicherheitshinweis:** Es gibt keinen API-Schlüssel zu speichern. Sie authentifizieren sich mit Microsoft Entra ID über `az login` (lokal) oder eine verwaltete Identität (in Azure). Die `.env`-Datei enthält nur nicht-geheime Einstellungen und ist bereits durch `.gitignore` geschützt.

## Testen Ihrer Einrichtung

Stellen Sie sicher, dass Sie angemeldet sind, damit die schlüssellose Authentifizierung ein Token erhalten kann, und führen Sie dann das Beispiel aus:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # wenn Sie noch nicht angemeldet sind
mvn clean spring-boot:run
```

Sie sollten eine Antwort vom `gpt-4o-mini` Modell sehen!

> **VS Code-Benutzer:** Drücken Sie `F5` zum Ausführen. Die App lädt Ihre `.env` automatisch.

> **Vollständiges Beispiel:** Siehe das [Basic Chat mit Azure AI Foundry Beispiel](./examples/basic-chat-azure/README.md) für Details und Fehlerbehebung.

## Was kommt als nächstes?

**Einrichtung abgeschlossen!** Sie haben jetzt:
- Azure AI Foundry mit `gpt-4o-mini` und `text-embedding-3-small` bereitgestellt
- Schlüssellose Authentifizierung (Microsoft Entra ID) — keine Schlüsselverwaltung
- Eine lokale `.env` mit Ihrem Endpunkt und den Bereitstellungsnamen
- Eine Java-Entwicklungsumgebung einsatzbereit

**Fahren Sie fort mit** [Kapitel 3: Kerntechniken der generativen KI](../03-CoreGenerativeAITechniques/README.md), um mit dem Erstellen von KI-Anwendungen zu beginnen!

## Ressourcen

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Schlüssellose Authentifizierung mit Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry Dokumentation](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI Dokumentation](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Zusätzliche Ressourcen

- [VS Code herunterladen](https://code.visualstudio.com/Download)
- [Docker Desktop holen](https://www.docker.com/products/docker-desktop)
- [Dev Container Konfiguration](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Haftungsausschluss**:
Dieses Dokument wurde mit dem KI-Übersetzungsdienst [Co-op Translator](https://github.com/Azure/co-op-translator) übersetzt. Obwohl wir uns um Genauigkeit bemühen, beachten Sie bitte, dass automatisierte Übersetzungen Fehler oder Ungenauigkeiten enthalten können. Das Originaldokument in seiner Ursprungssprache gilt als maßgebliche Quelle. Bei kritischen Informationen wird eine professionelle menschliche Übersetzung empfohlen. Wir übernehmen keine Haftung für Missverständnisse oder Fehlinterpretationen, die aus der Verwendung dieser Übersetzung entstehen.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->