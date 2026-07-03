# Basic Chat mit Azure AI Foundry – End-to-End-Beispiel

Dieses Beispiel ist eine einfache Spring-Boot-Anwendung, die eine Verbindung zu einem **Azure AI Foundry**-Modell mit **schlüsselosem Authentifizieren** (Microsoft Entra ID) herstellt und Ihre Einrichtung testet. Es verwendet Spring AI's `ChatClient`.

## Inhaltsverzeichnis

- [Voraussetzungen](#voraussetzungen)
- [Schnellstart](#schnellstart)
- [Wie die Authentifizierung funktioniert](#wie-die-authentifizierung-funktioniert)
- [Anwendung ausführen](#anwendung-ausführen)
  - [Mit Maven](#mit-maven)
  - [Mit VS Code](#mit-vs-code)
  - [Erwartete Ausgabe](#erwartete-ausgabe)
- [Konfigurationsreferenz](#konfigurationsreferenz)
  - [Umgebungsvariablen](#umgebungsvariablen)
  - [Spring-Konfiguration](#spring-konfiguration)
- [Fehlerbehebung](#fehlerbehebung)
  - [Häufige Probleme](#häufige-probleme)
  - [Debug-Modus](#debug-modus)
- [Nächste Schritte](#nächste-schritte)
- [Ressourcen](#ressourcen)

## Voraussetzungen

Bevor Sie dieses Beispiel ausführen, stellen Sie sicher, dass Sie Folgendes haben:

- Eine Azure AI Foundry-Ressource mit einer `gpt-4o-mini` Bereitstellung – stellen Sie sie mit `azd up` bereit oder manuell über die [Azure AI Foundry Setup-Anleitung](../../getting-started-azure-openai.md)
- Die Rolle **Cognitive Services OpenAI User** für diese Ressource (wird durch die Bicep-Vorlagen zugewiesen)
- Die [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), angemeldet mit `az login`
- Java 21+ und Maven 3.9+

> **Kein API-Schlüssel erforderlich** — die Authentifizierung erfolgt schlüssellos über Microsoft Entra ID.

## Schnellstart

```bash
# 1. Navigiere zum Projekt
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Melde dich an, damit die schlüssellose Authentifizierung ein Token erhalten kann
az login

# 3. Konfiguriere den Endpunkt
#    - Wenn du `azd up` ausgeführt hast, wurde die .env für dich geschrieben (überspringe dies).
#    - Andernfalls kopiere die Vorlage und setze AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. Starte die Anwendung
mvn spring-boot:run
```

## Wie die Authentifizierung funktioniert

Dieses Beispiel authentifiziert sich mit **Microsoft Entra ID** — es wird kein API-Schlüssel verwendet.

Wenn nur `spring.ai.azure.openai.endpoint` gesetzt ist (und kein api-key), erstellt Spring AI den Azure OpenAI-Client mit [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). Diese Anmeldeinformation findet automatisch ein Token aus Ihrer lokalen `az login`-Sitzung oder aus einer verwalteten Identität bei Ausführung in Azure – somit funktioniert der gleiche Code an beiden Orten ohne Änderungen.

## Anwendung ausführen

### Mit Maven

```bash
mvn spring-boot:run
```

### Mit VS Code

1. Öffnen Sie das Projekt in VS Code
2. Drücken Sie `F5` oder verwenden Sie das "Run and Debug"-Panel
3. Wählen Sie die Konfiguration "Spring Boot-BasicChatApplication"

> **Hinweis**: Die VS Code-Konfiguration lädt automatisch Ihre .env-Datei

### Erwartete Ausgabe

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

## Konfigurationsreferenz

### Umgebungsvariablen

| Variable | Beschreibung | Erforderlich | Beispiel |
|----------|--------------|--------------|----------|
| `AZURE_OPENAI_ENDPOINT` | Foundry (Azure OpenAI) Endpunkt-URL | Ja | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Chat-Modell-Bereitstellungsname | Nein | `gpt-4o-mini` (Standard) |

> Es gibt **keine** API-Schlüssel-Variable — die Authentifizierung erfolgt schlüssellos (Microsoft Entra ID via `az login`).

### Spring-Konfiguration

Die Datei `application.yml` konfiguriert:
- **Endpunkt**: `${AZURE_OPENAI_ENDPOINT}` - Aus Umgebungsvariable
- **Bereitstellung**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - Aus Umgebungsvariable mit Fallback
- **Auth**: schlüssellos — kein `api-key` gesetzt, daher verwendet Spring AI `DefaultAzureCredential`
- **Temperatur**: `0.7` - Steuert die Kreativität (0.0 = deterministisch, 1.0 = kreativ)
- **Max Tokens**: `500` - Maximale Antwortlänge

## Fehlerbehebung

### Häufige Probleme

<details>
<summary><strong>Fehler: 401 / "PermissionDenied" / Token-Fehler</strong></summary>

- Führen Sie `az login` aus — schlüssellose Auth benötigt ein aktives Anmelden, um ein Token zu erhalten
- Überprüfen Sie, ob Ihr Konto die Rolle **Cognitive Services OpenAI User** für die Ressource hat
- Wenn Sie die Rolle gerade zugewiesen haben, warten Sie eine Minute, bis sie propagiert wird
- Stellen Sie sicher, dass Sie im richtigen Mandanten/Abonnement sind (`az account show`)
</details>

<details>
<summary><strong>Fehler: "Der Endpunkt ist nicht gültig" / Verbindungsfehler</strong></summary>

- Stellen Sie sicher, dass `AZURE_OPENAI_ENDPOINT` die vollständige Basis-URL ist (z.B. `https://your-resource.openai.azure.com/`)
- Prüfen Sie die Konsistenz des abschließenden Schrägstrichs
- Überprüfen Sie, dass der Endpunkt zu Ihrer bereitgestellten Ressource passt (`azd env get-values`)
</details>

<details>
<summary><strong>Fehler: "Die Bereitstellung wurde nicht gefunden"</strong></summary>

- Überprüfen Sie, ob `AZURE_OPENAI_DEPLOYMENT` mit einem Bereitstellungsnamen in Azure übereinstimmt
- Prüfen Sie, ob das Modell erfolgreich bereitgestellt und aktiv ist
- Der Standard-Bereitstellungsname ist `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: Umgebungsvariablen werden nicht geladen</strong></summary>

- Stellen Sie sicher, dass Ihre `.env`-Datei im Projekt-Stammverzeichnis liegt (auf derselben Ebene wie `pom.xml`)
- Versuchen Sie, `mvn spring-boot:run` im integrierten Terminal von VS Code auszuführen
- Prüfen Sie, ob die Java-Erweiterung in VS Code richtig installiert ist
</details>

### Debug-Modus

Um detaillierte Protokolle zu aktivieren, kommentieren Sie diese Zeilen in der `application.yml` aus:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Nächste Schritte

**Einrichtung abgeschlossen!** Führen Sie Ihre Lernreise fort:

[Kapitel 3: Kerntechniken der generativen KI](../../../03-CoreGenerativeAITechniques/README.md)

## Ressourcen

- [Spring AI Azure OpenAI Dokumentation](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Schlüssellose Authentifizierung mit Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry Portal](https://ai.azure.com/)
- [Azure AI Foundry Dokumentation](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Haftungsausschluss**:
Dieses Dokument wurde mit dem KI-Übersetzungsdienst [Co-op Translator](https://github.com/Azure/co-op-translator) übersetzt. Obwohl wir uns um Genauigkeit bemühen, beachten Sie bitte, dass automatisierte Übersetzungen Fehler oder Ungenauigkeiten enthalten können. Das Originaldokument in seiner Ursprungssprache gilt als maßgebliche Quelle. Bei kritischen Informationen wird eine professionelle menschliche Übersetzung empfohlen. Wir übernehmen keine Haftung für Missverständnisse oder Fehlinterpretationen, die aus der Verwendung dieser Übersetzung entstehen.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->