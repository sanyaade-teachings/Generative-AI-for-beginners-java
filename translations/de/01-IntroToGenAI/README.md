# Einführung in Generative KI – Java Edition

## Was Sie lernen werden

- **Grundlagen der Generativen KI** einschließlich LLMs, Prompt Engineering, Tokens, Embeddings und Vektordatenbanken
- **Vergleich von Java AI-Entwicklungswerkzeugen** einschließlich Azure OpenAI SDK, Spring AI und OpenAI Java SDK
- **Entdecken Sie das Model Context Protocol** und seine Rolle bei der Kommunikation von KI-Agenten

## Inhaltsverzeichnis

- [Einführung](#einführung)
- [Ein kurzer Überblick über Generative KI-Konzepte](#ein-kurzer-überblick-über-generative-ki-konzepte)
- [Rückblick auf Prompt Engineering](#rückblick-auf-prompt-engineering)
- [Tokens, Embeddings und Agenten](#tokens-embeddings-und-agenten)
- [KI-Entwicklungswerkzeuge und Bibliotheken für Java](#ki-entwicklungswerkzeuge-und-bibliotheken-für-java)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [Zusammenfassung](#zusammenfassung)
- [Nächste Schritte](#nächste-schritte)

## Einführung

Willkommen zum ersten Kapitel von Generative KI für Anfänger – Java Edition! Diese grundlegende Lektion führt Sie in die Kernkonzepte der generativen KI ein und zeigt, wie Sie mit ihnen in Java arbeiten. Sie lernen die wesentlichen Bausteine von KI-Anwendungen kennen, darunter Large Language Models (LLMs), Tokens, Embeddings und KI-Agenten. Außerdem erkunden wir die wichtigsten Java-Tools, die Sie im Verlauf dieses Kurses verwenden werden.

### Ein kurzer Überblick über Generative KI-Konzepte

Generative KI ist eine Art von künstlicher Intelligenz, die neue Inhalte erstellt, wie Text, Bilder oder Code, basierend auf Mustern und Beziehungen, die aus Daten gelernt wurden. Generative KI-Modelle können menschenähnliche Antworten generieren, den Kontext verstehen und manchmal sogar Inhalte erschaffen, die menschlich wirken.

Beim Entwickeln Ihrer Java-KI-Anwendungen arbeiten Sie mit **generativen KI-Modellen**, um Inhalte zu erstellen. Einige Fähigkeiten generativer KI-Modelle umfassen:

- **Textgenerierung**: Erstellen von menschenähnlichem Text für Chatbots, Inhalte und Textvervollständigung.
- **Bildgenerierung und -analyse**: Erzeugen realistischer Bilder, Verbessern von Fotos und Erkennen von Objekten.
- **Codegenerierung**: Schreiben von Code-Snippets oder Skripten.

Es gibt spezifische Modelltypen, die für verschiedene Aufgaben optimiert sind. Zum Beispiel können sowohl **Small Language Models (SLMs)** als auch **Large Language Models (LLMs)** Textgenerierung übernehmen, wobei LLMs typischerweise bessere Leistung bei komplexeren Aufgaben bieten. Für bildbezogene Aufgaben würden Sie spezialisierte Vision-Modelle oder multimodale Modelle verwenden.

![Figure: Generative AI model types and use cases.](../../../translated_images/de/llms.225ca2b8a0d34473.webp)

Natürlich sind die Antworten dieser Modelle nicht immer perfekt. Wahrscheinlich haben Sie schon von Modellen gehört, die „halluzinieren“ oder falsche Informationen auf autoritäre Weise generieren. Aber Sie können das Modell dabei unterstützen, bessere Antworten zu geben, indem Sie klare Anweisungen und Kontext bereitstellen. Hier kommt das **Prompt Engineering** ins Spiel.

#### Rückblick auf Prompt Engineering

Prompt Engineering ist die Praxis, effektive Eingaben zu gestalten, um KI-Modelle gezielt zu gewünschten Ausgaben zu führen. Es umfasst:

- **Klarheit**: Instruktionen klar und eindeutig machen.
- **Kontext**: Notwendige Hintergrundinformationen bereitstellen.
- **Einschränkungen**: Jegliche Begrenzungen oder Formate spezifizieren.

Zu den Best Practices zählen Prompt-Design, klare Anweisungen, Aufgabenzerlegung, One-Shot- und Few-Shot-Lernen sowie Prompt-Tuning. Verschiedene Prompts zu testen, ist essenziell, um herauszufinden, was für Ihren speziellen Anwendungsfall am besten funktioniert.

Beim Entwickeln von Anwendungen arbeiten Sie mit verschiedenen Prompt-Typen:
- **System-Prompts**: Legen die Grundregeln und den Kontext für das Verhalten des Modells fest
- **User-Prompts**: Die Eingabedaten von Ihren Anwender:innen
- **Assistant-Prompts**: Die Antworten des Modells basierend auf System- und User-Prompts

> **Mehr erfahren**: Mehr über Prompt Engineering erfahren Sie im [Kapitel Prompt Engineering des GenAI für Anfänger-Kurses](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### Tokens, Embeddings und Agenten

Beim Arbeiten mit generativen KI-Modellen stoßen Sie auf Begriffe wie **Tokens**, **Embeddings**, **Agenten** und **Model Context Protocol (MCP)**. Hier eine ausführliche Übersicht dieser Konzepte:

- **Tokens**: Tokens sind die kleinste Einheit von Text in einem Modell. Sie können Wörter, Zeichen oder Teilwörter sein. Tokens dienen dazu, Textdaten in einem Format darzustellen, das das Modell versteht. Zum Beispiel könnte der Satz „The quick brown fox jumped over the lazy dog“ tokenisiert werden als ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] oder ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"] je nach Tokenisierungsstrategie.

![Figure: Generative AI tokens example of breaking words into tokens](../../../translated_images/de/tokens.6283ed277a2ffff4.webp)

Tokenisierung ist der Prozess, Text in diese kleineren Einheiten zu zerlegen. Das ist entscheidend, weil Modelle auf Tokens und nicht auf Rohtext operieren. Die Anzahl der Tokens in einem Prompt beeinflusst die Länge und Qualität der Modellantwort, da Modelle eine Token-Begrenzung für ihr Kontextfenster haben (z. B. 128.000 Tokens für den gesamten Kontext von GPT-4o, inklusive Ein- und Ausgabe).

  In Java können Sie Bibliotheken wie das OpenAI SDK verwenden, um die Tokenisierung bei Anfragen an KI-Modelle automatisch zu handhaben.

- **Embeddings**: Embeddings sind Vektor-Darstellungen von Tokens, die semantische Bedeutung erfassen. Sie sind numerische Repräsentationen (typischerweise Arrays von Gleitkommazahlen), die es Modellen ermöglichen, die Beziehungen zwischen Wörtern zu verstehen und kontextuell relevante Antworten zu generieren. Ähnliche Wörter haben ähnliche Embeddings, wodurch das Modell Konzepte wie Synonyme und semantische Beziehungen begreift.

![Figure: Embeddings](../../../translated_images/de/embedding.398e50802c0037f9.webp)

  In Java können Sie Embeddings mit dem OpenAI SDK oder anderen Bibliotheken erzeugen, die Embedding-Erzeugung unterstützen. Diese Embeddings sind essenziell für Aufgaben wie semantische Suche, bei der Sie Inhalte nach Bedeutung und nicht nach exakten Textübereinstimmungen finden möchten.

- **Vektordatenbanken**: Vektordatenbanken sind spezialisierte Speichersysteme, die für Embeddings optimiert sind. Sie ermöglichen effiziente Ähnlichkeitssuche und sind wichtig für Retrieval-Augmented Generation (RAG)-Muster, bei denen relevante Informationen aus großen Datensätzen anhand semantischer Ähnlichkeit und nicht exakter Übereinstimmungen gefunden werden.

![Figure: Vector database architecture showing how embeddings are stored and retrieved for similarity search.](../../../translated_images/de/vector.f12f114934e223df.webp)

> **Hinweis**: In diesem Kurs behandeln wir keine Vektordatenbanken, erwähnen sie aber, da sie in realen Anwendungen häufig verwendet werden.

- **Agenten & MCP**: KI-Komponenten, die autonom mit Modellen, Tools und externen Systemen interagieren. Das Model Context Protocol (MCP) bietet eine standardisierte Möglichkeit, dass Agenten sicher auf externe Datenquellen und Tools zugreifen. Mehr dazu in unserem [MCP für Anfänger](https://github.com/microsoft/mcp-for-beginners)-Kurs.

In Java KI-Anwendungen verwenden Sie Tokens zur Textverarbeitung, Embeddings für semantische Suche und RAG, Vektordatenbanken für Datenabruf und Agenten mit MCP zum Aufbau intelligenter, toolnutzender Systeme.

![Figure: how a prompt becomes a reply—tokens, vectors, optional RAG lookup, LLM thinking, and an MCP agent all in one quick flow..](../../../translated_images/de/flow.f4ef62c3052d12a8.webp)

### KI-Entwicklungswerkzeuge und Bibliotheken für Java

Java bietet hervorragende Werkzeuge für die KI-Entwicklung. Es gibt drei Hauptbibliotheken, die wir im Verlauf dieses Kurses erkunden werden – OpenAI Java SDK, Azure OpenAI SDK und Spring AI.

Hier eine schnelle Referenztabelle, welche SDKs in den Beispielen der Kapitel verwendet werden:

| Kapitel | Beispiel | SDK |
|---------|----------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**Dokumentationslinks der SDKs:**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

Das OpenAI SDK ist die offizielle Java-Bibliothek für die OpenAI-API. Es bietet eine einfache und konsistente Schnittstelle zur Interaktion mit OpenAI-Modellen und erleichtert die Integration von KI-Funktionalitäten in Java-Anwendungen. Die Pet Story Anwendung und das Foundry Local Beispiel in Kapitel 4 demonstrieren den OpenAI SDK-Ansatz mit Azure AI Foundry.

#### Spring AI

Spring AI ist ein umfassendes Framework, das KI-Funktionalitäten in Spring-Anwendungen bringt und eine einheitliche Abstraktionsschicht über verschiedene KI-Anbieter hinweg bietet. Es integriert sich nahtlos in das Spring-Ökosystem und ist die ideale Wahl für Enterprise-Java-Anwendungen, die KI-Funktionalität benötigen.

Die Stärke von Spring AI liegt in der nahtlosen Integration in das Spring-Ökosystem, wodurch sich produktionsreife KI-Anwendungen mit vertrauten Spring-Mustern wie Dependency Injection, Konfigurationsmanagement und Testframeworks einfach erstellen lassen. Sie verwenden Spring AI in Kapitel 2 und 4, um Anwendungen zu bauen, die sowohl OpenAI als auch das Model Context Protocol (MCP) Spring AI-Bibliotheken nutzen.

##### Model Context Protocol (MCP)

Das [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) ist ein aufkommender Standard, der es KI-Anwendungen ermöglicht, sicher mit externen Datenquellen und Tools zu interagieren. MCP bietet eine standardisierte Möglichkeit, dass KI-Modelle auf kontextuelle Informationen zugreifen und Aktionen in Ihren Anwendungen ausführen können.

In Kapitel 4 bauen Sie einen einfachen MCP Taschenrechner-Service, der die Grundlagen des Model Context Protocol mit Spring AI demonstriert, und zeigt, wie man grundlegende Tool-Integrationen und Service-Architekturen erstellt.

#### Azure OpenAI Java SDK

Die Azure OpenAI Clientbibliothek für Java ist eine Anpassung der OpenAI REST APIs, die eine idiomatische Schnittstelle bietet und in das Azure SDK-Ökosystem integriert ist. In Kapitel 3 erstellen Sie Anwendungen mit dem Azure OpenAI SDK, darunter Chat-Anwendungen, Funktionsaufrufe und RAG (Retrieval-Augmented Generation) Muster.

> Hinweis: Das Azure OpenAI SDK hinkt dem OpenAI Java SDK im Funktionsumfang hinterher, daher sollten Sie für zukünftige Projekte das OpenAI Java SDK in Erwägung ziehen.

## Zusammenfassung

Damit sind die Grundlagen abgeschlossen! Sie verstehen nun:

- Die Kernkonzepte hinter generativer KI – von LLMs und Prompt Engineering bis hin zu Tokens, Embeddings und Vektordatenbanken
- Ihre Tool-Auswahlmöglichkeiten für Java KI-Entwicklung: Azure OpenAI SDK, Spring AI und OpenAI Java SDK
- Was das Model Context Protocol ist und wie es KI-Agenten ermöglicht, mit externen Tools zu arbeiten

## Nächste Schritte

[Kapitel 2: Einrichten der Entwicklungsumgebung](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Haftungsausschluss**:
Dieses Dokument wurde mit dem KI-Übersetzungsdienst [Co-op Translator](https://github.com/Azure/co-op-translator) übersetzt. Obwohl wir uns um Genauigkeit bemühen, beachten Sie bitte, dass automatisierte Übersetzungen Fehler oder Ungenauigkeiten enthalten können. Das Originaldokument in seiner Ursprungssprache gilt als maßgebliche Quelle. Bei kritischen Informationen wird eine professionelle menschliche Übersetzung empfohlen. Wir übernehmen keine Haftung für Missverständnisse oder Fehlinterpretationen, die aus der Verwendung dieser Übersetzung entstehen.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->