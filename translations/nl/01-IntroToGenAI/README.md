# Introductie tot Generatieve AI - Java Editie

## Wat je zult leren

- **Fundamenten van Generatieve AI** inclusief LLM's, prompt engineering, tokens, embeddings en vector databases
- **Vergelijking van Java AI ontwikkeltools** zoals Azure OpenAI SDK, Spring AI en OpenAI Java SDK
- **Ontdek het Model Context Protocol** en de rol ervan in AI agent communicatie

## Inhoudsopgave

- [Introductie](#introductie)
- [Een korte opfrissing van Generatieve AI concepten](#een-korte-opfrissing-van-generatieve-ai-concepten)
- [Overzicht prompt engineering](#overzicht-prompt-engineering)
- [Tokens, embeddings en agents](#tokens-embeddings-en-agents)
- [AI ontwikkeltools en bibliotheken voor Java](#ai-ontwikkeltools-en-bibliotheken-voor-java)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [Samenvatting](#samenvatting)
- [Volgende stappen](#volgende-stappen)

## Introductie

Welkom bij het eerste hoofdstuk van Generatieve AI voor Beginners - Java Editie! Deze fundamentele les introduceert je in de kernconcepten van generatieve AI en hoe je hier mee kunt werken met Java. Je leert over de essentiële bouwstenen van AI-toepassingen, inclusief Large Language Models (LLM's), tokens, embeddings en AI agents. We verkennen ook de belangrijkste Java tools die je gedurende deze cursus zult gebruiken.

### Een korte opfrissing van Generatieve AI concepten

Generatieve AI is een vorm van kunstmatige intelligentie die nieuwe content creëert, zoals tekst, afbeeldingen of code, op basis van patronen en relaties die uit data zijn geleerd. Generatieve AI-modellen kunnen mensachtige antwoorden genereren, context begrijpen en soms zelfs content maken die menselijk lijkt.

Bij het ontwikkelen van je Java AI-toepassingen werk je met **generatieve AI-modellen** om content te creëren. Enkele mogelijkheden van generatieve AI-modellen zijn:

- **Tekstgeneratie**: Het maken van mensachtige tekst voor chatbots, content en tekstafmaking.
- **Afbeeldingsgeneratie en analyse**: Realistische afbeeldingen maken, foto's verbeteren en objecten detecteren.
- **Codegeneratie**: Codefragmenten of scripts schrijven.

Er bestaan specifieke modeltypen die geoptimaliseerd zijn voor verschillende taken. Bijvoorbeeld, zowel **Small Language Models (SLM's)** als **Large Language Models (LLM's)** kunnen tekstgeneratie aan, waarbij LLM's meestal betere prestaties leveren bij complexe taken. Voor beeldgerelateerde taken gebruik je gespecialiseerde vision-modellen of multimodale modellen.

![Figure: Generative AI model types and use cases.](../../../translated_images/nl/llms.225ca2b8a0d34473.webp)

Natuurlijk zijn de antwoorden van deze modellen niet altijd perfect. Je hebt waarschijnlijk gehoord van modellen die “hallucineren” of foutieve informatie op een autoritaire manier genereren. Maar je kunt het model helpen betere antwoorden te genereren door duidelijke instructies en context te geven. Dit is waar **prompt engineering** van pas komt.

#### Overzicht prompt engineering

Prompt engineering is de praktijk van het ontwerpen van effectieve inputs om AI-modellen naar gewenste outputs te leiden. Het omvat:

- **Helderheid**: Instructies duidelijk en ondubbelzinnig maken.
- **Context**: Noodzakelijke achtergrondinformatie verstrekken.
- **Beperkingen**: Eventuele limieten of formatten specificeren.

Enkele beste praktijken voor prompt engineering zijn promptontwerp, heldere instructies, taakopdeling, one-shot en few-shot leren, en prompt tuning. Het testen van verschillende prompts is essentieel om te vinden wat het beste werkt voor jouw specifieke gebruikssituatie.

Bij het ontwikkelen van toepassingen werk je met verschillende prompttypen:
- **Systeem-prompts**: Stellen de basisregels en context voor het gedrag van het model in
- **Gebruikersprompts**: De invoergegevens van je applicatiegebruikers
- **Assistent-prompts**: De antwoorden van het model gebaseerd op systeem- en gebruikersprompts

> **Meer weten**: Lees meer over prompt engineering in het [hoofdstuk Prompt Engineering van de GenAI voor Beginners cursus](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### Tokens, embeddings en agents

Wanneer je werkt met generatieve AI-modellen kom je termen tegen als **tokens**, **embeddings**, **agents** en **Model Context Protocol (MCP)**. Hier volgt een gedetailleerd overzicht van deze concepten:

- **Tokens**: Tokens zijn de kleinste eenheid tekst in een model. Ze kunnen woorden, karakters of subwoorden zijn. Tokens worden gebruikt om tekstdata te representeren in een formaat dat het model kan begrijpen. Bijvoorbeeld, de zin "The quick brown fox jumped over the lazy dog" kan worden getokeniseerd als ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] of ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"] afhankelijk van de tokenisatiestrategie.

![Figure: Generative AI tokens example of breaking words into tokens](../../../translated_images/nl/tokens.6283ed277a2ffff4.webp)

Tokenisatie is het proces waarbij tekst wordt opgedeeld in deze kleinere eenheden. Dit is cruciaal omdat modellen op tokens opereren in plaats van op ruwe tekst. Het aantal tokens in een prompt beïnvloedt de lengte en kwaliteit van het antwoord van het model, omdat modellen limieten hebben voor tokens in hun contextvenster (bijv. 128K tokens voor GPT-4o's totale context inclusief zowel input als output).

  In Java kun je bibliotheken gebruiken zoals de OpenAI SDK om tokenisatie automatisch af te handelen wanneer je verzoeken naar AI-modellen stuurt.

- **Embeddings**: Embeddings zijn vectorrepresentaties van tokens die semantische betekenis vastleggen. Het zijn numerieke representaties (meestal arrays van floating-point getallen) die modellen in staat stellen verbanden tussen woorden te begrijpen en contextueel relevante antwoorden te genereren. Soortgelijke woorden hebben vergelijkbare embeddings, waardoor het model begrippen als synoniemen en semantische relaties begrijpt.

![Figure: Embeddings](../../../translated_images/nl/embedding.398e50802c0037f9.webp)

  In Java kun je embeddings genereren met de OpenAI SDK of andere bibliotheken die embeddinggeneratie ondersteunen. Deze embeddings zijn essentieel voor taken zoals semantische zoekopdrachten, waarbij je content zoekt op basis van betekenis in plaats van exacte tekstmatches.

- **Vector databases**: Vector databases zijn gespecialiseerde opslagsystemen die geoptimaliseerd zijn voor embeddings. Ze maken efficiënte zoekopdrachten op gelijkenis mogelijk en zijn cruciaal voor Retrieval-Augmented Generation (RAG) patronen waarbij je relevante informatie uit grote datasets zoekt op basis van semantische gelijkenis in plaats van exacte overeenkomsten.

![Figure: Vector database architecture showing how embeddings are stored and retrieved for similarity search.](../../../translated_images/nl/vector.f12f114934e223df.webp)

> **Opmerking**: In deze cursus behandelen we Vector databases niet maar we vinden het belangrijk ze te noemen omdat ze vaak gebruikt worden in real-world toepassingen.

- **Agents & MCP**: AI-componenten die autonoom interageren met modellen, tools en externe systemen. Het Model Context Protocol (MCP) biedt een gestandaardiseerde manier voor agents om veilig toegang te krijgen tot externe databronnen en tools. Meer informatie vind je in onze [MCP for Beginners](https://github.com/microsoft/mcp-for-beginners) cursus.

In Java AI-toepassingen gebruik je tokens voor tekstverwerking, embeddings voor semantische zoekopdrachten en RAG, vector databases voor dataopvraging en agents met MCP om intelligente, tool-gebruikende systemen te bouwen.

![Figure: how a prompt becomes a reply—tokens, vectors, optional RAG lookup, LLM thinking, and an MCP agent all in one quick flow..](../../../translated_images/nl/flow.f4ef62c3052d12a8.webp)

### AI ontwikkeltools en bibliotheken voor Java

Java biedt uitstekende tools voor AI-ontwikkeling. Er zijn drie hoofd-bibliotheken die we door deze cursus heen zullen verkennen - OpenAI Java SDK, Azure OpenAI SDK en Spring AI.

Hier is een snelle referentietabel waarin staat welke SDK gebruikt wordt in de voorbeelden van elk hoofdstuk:

| Hoofdstuk | Voorbeeld | SDK |
|-----------|-----------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**SDK documentatielinks:**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

De OpenAI SDK is de officiële Java-bibliotheek voor de OpenAI API. Het biedt een eenvoudige en consistente interface om te interacteren met OpenAI's modellen, wat het eenvoudig maakt om AI-mogelijkheden te integreren in Java-toepassingen. Hoofdstuk 4's Pet Story applicatie en Foundry Local voorbeeld demonstreren de OpenAI SDK aanpak met Azure AI Foundry.

#### Spring AI

Spring AI is een uitgebreid framework dat AI-mogelijkheden brengt naar Spring applicaties en een consistente abstractielaag biedt over verschillende AI-leveranciers. Het integreert naadloos binnen het Spring-ecosysteem, wat het de ideale keuze maakt voor enterprise Java-applicaties die AI nodig hebben.

De kracht van Spring AI ligt in de naadloze integratie met het Spring-ecosysteem, waardoor het eenvoudig is om productieklare AI-applicaties te bouwen met vertrouwde Spring-patronen zoals dependency injection, configuratiebeheer en test-frameworks. Je gebruikt Spring AI in hoofdstuk 2 en 4 om applicaties te bouwen die zowel OpenAI als het Model Context Protocol (MCP) Spring AI bibliotheken gebruiken.

##### Model Context Protocol (MCP)

Het [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) is een opkomende standaard die AI-toepassingen in staat stelt om veilig te communiceren met externe databronnen en tools. MCP biedt een gestandaardiseerde manier voor AI-modellen om contextuele informatie te benaderen en acties uit te voeren in je applicaties.

In hoofdstuk 4 bouw je een eenvoudige MCP calculator service die de fundamenten van het Model Context Protocol met Spring AI demonstreert, en laat zien hoe je basis tool-integraties en service-architecturen maakt.

#### Azure OpenAI Java SDK

De Azure OpenAI clientbibliotheek voor Java is een aanpassing van de REST API's van OpenAI die een idiomatische interface en integratie biedt met de rest van het Azure SDK ecosysteem. In hoofdstuk 3 bouw je toepassingen met de Azure OpenAI SDK, waaronder chatapplicaties, function calling en RAG (Retrieval-Augmented Generation) patronen.

> Opmerking: Azure OpenAI SDK loopt qua features achter op de OpenAI Java SDK, dus voor toekomstige projecten overweeg het gebruik van de OpenAI Java SDK.

## Samenvatting

Dat waren de basisprincipes! Je begrijpt nu:

- De kernconcepten achter generatieve AI - van LLM's en prompt engineering tot tokens, embeddings en vector databases
- Je toolkitopties voor Java AI-ontwikkeling: Azure OpenAI SDK, Spring AI en OpenAI Java SDK
- Wat het Model Context Protocol is en hoe het AI agents in staat stelt met externe tools te werken

## Volgende stappen

[Hoofdstuk 2: Het inrichten van de ontwikkelomgeving](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Disclaimer**:
Dit document is vertaald met behulp van de AI vertaaldienst [Co-op Translator](https://github.com/Azure/co-op-translator). Hoewel we streven naar nauwkeurigheid, dient u er rekening mee te houden dat geautomatiseerde vertalingen fouten of onnauwkeurigheden kunnen bevatten. Het originele document in de oorspronkelijke taal moet worden beschouwd als de gezaghebbende bron. Voor kritieke informatie wordt professionele menselijke vertaling aanbevolen. Wij zijn niet aansprakelijk voor eventuele misverstanden of verkeerde interpretaties die voortvloeien uit het gebruik van deze vertaling.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->