# Introduktion till Generativ AI - Java Edition

## Vad du kommer att lära dig

- **Grundläggande om generativ AI** inklusive LLMs, prompt engineering, tokens, embeddings och vektordatabaser
- **Jämföra Java AI-utvecklingsverktyg** inklusive Azure OpenAI SDK, Spring AI och OpenAI Java SDK
- **Upptäck Model Context Protocol** och dess roll i AI-agenters kommunikation

## Innehållsförteckning

- [Introduktion](#introduktion)
- [En snabb repetition av Generative AI-koncept](#en-snabb-repetition-av-generative-ai-koncept)
- [Översikt av prompt engineering](#översikt-av-prompt-engineering)
- [Tokens, embeddings och agenter](#tokens-embeddings-och-agenter)
- [AI-utvecklingsverktyg och bibliotek för Java](#ai-utvecklingsverktyg-och-bibliotek-för-java)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [Sammanfattning](#sammanfattning)
- [Nästa steg](#nästa-steg)

## Introduktion

Välkommen till första kapitlet i Generative AI för nybörjare - Java Edition! Denna grundläggande lektion introducerar dig till de centrala koncepten i generativ AI och hur man arbetar med dem med Java. Du kommer att lära dig om de viktiga byggstenarna i AI-applikationer, inklusive stora språkmodeller (LLMs), tokens, embeddings och AI-agenter. Vi kommer också att utforska de primära Java-verktygen du kommer att använda genom hela kursen.

### En snabb repetition av Generative AI-koncept

Generativ AI är en typ av artificiell intelligens som skapar nytt innehåll, såsom text, bilder eller kod, baserat på mönster och relationer som modellen lärt sig från data. Generativa AI-modeller kan generera mänskliga svar, förstå kontext och ibland till och med skapa innehåll som verkar mänskligt.

När du utvecklar dina Java AI-applikationer kommer du att arbeta med **generativa AI-modeller** för att skapa innehåll. Några kapabiliteter hos generativa AI-modeller inkluderar:

- **Textgenerering**: Skapa mänskliglik text för chattbotar, innehåll och textkomplettering.
- **Bildgenerering och analys**: Producera realistiska bilder, förbättra foton och upptäcka objekt.
- **Kodgenerering**: Skriva kodexempel eller skript.

Det finns specifika typer av modeller som optimerats för olika uppgifter. Till exempel kan både **små språkmodeller (SLMs)** och **stora språkmodeller (LLMs)** hantera textgenerering, där LLMs vanligtvis erbjuder bättre prestanda för komplexa uppgifter. För bildrelaterade uppgifter använder man specialiserade visionsmodeller eller multimodala modeller.

![Figure: Generative AI model types and use cases.](../../../translated_images/sv/llms.225ca2b8a0d34473.webp)

Självklart är inte svaren från dessa modeller perfekta hela tiden. Du har förmodligen hört talas om att modeller "hallucinerar" eller genererar felaktig information på ett auktoritativt sätt. Men du kan hjälpa modellen att generera bättre svar genom att ge den tydliga instruktioner och kontext. Det är här **prompt engineering** kommer in.

#### Översikt av prompt engineering

Prompt engineering är praxis för att designa effektiva indata för att styra AI-modeller mot önskade utdata. Det innebär:

- **Tydlighet**: Göra instruktionerna klara och entydiga.
- **Kontext**: Ge nödvändig bakgrundsinformation.
- **Begränsningar**: Specificera eventuella begränsningar eller format.

Några bästa metoder för prompt engineering inkluderar promptdesign, tydliga instruktioner, uppdelning av uppgifter, one-shot och few-shot learning, samt promptjustering. Det är viktigt att testa olika prompts för att hitta vad som fungerar bäst för din specifika användning.

När du utvecklar applikationer kommer du att arbeta med olika typer av prompts:
- **Systemprompts**: Sätter basregler och kontext för modellens beteende
- **Användarprompts**: Indata från dina applikationsanvändare
- **Assistansprompts**: Modellens svar baserade på system- och användarprompts

> **Lär dig mer**: Läs mer om prompt engineering i [Prompt Engineering-kapitlet i GenAI för nybörjare-kursen](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### Tokens, embeddings och agenter

När du arbetar med generativa AI-modeller kommer du att möta termer som **tokens**, **embeddings**, **agenter** och **Model Context Protocol (MCP)**. Här är en detaljerad översikt över dessa koncept:

- **Tokens**: Tokens är den minsta textenheten i en modell. Det kan vara ord, tecken eller delord. Tokens används för att representera textdata i ett format som modellen kan förstå. Till exempel kan meningen "The quick brown fox jumped over the lazy dog" tokeniseras som ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] eller ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"] beroende på tokeniseringsstrategi.

![Figure: Generative AI tokens example of breaking words into tokens](../../../translated_images/sv/tokens.6283ed277a2ffff4.webp)

Tokenisering är processen att dela upp text i dessa mindre enheter. Detta är avgörande eftersom modeller arbetar på tokens istället för rå text. Antalet tokens i en prompt påverkar modellens svarslängd och kvalitet, eftersom modeller har tokenbegränsningar för deras kontextfönster (t.ex. 128K tokens för GPT-4o:s totala kontext inklusive både indata och utdata).

  I Java kan du använda bibliotek som OpenAI SDK för att automatiskt hantera tokenisering när du skickar förfrågningar till AI-modeller.

- **Embeddings**: Embeddings är vektorrepresentationer av tokens som fångar semantisk mening. Det är numeriska representationer (vanligtvis arrayer av flyttal) som låter modellen förstå relationer mellan ord och generera kontextuellt relevanta svar. Liknande ord har liknande embeddings, vilket möjliggör för modellen att förstå begrepp som synonymer och semantiska relationer.

![Figure: Embeddings](../../../translated_images/sv/embedding.398e50802c0037f9.webp)

  I Java kan du generera embeddings med OpenAI SDK eller andra bibliotek som stödjer embeddinggenerering. Dessa embeddings är viktiga för uppgifter som semantisk sökning, där du vill hitta liknande innehåll baserat på betydelse snarare än exakt textmatchning.

- **Vektordatabaser**: Vektordatabaser är specialiserade lagringssystem optimerade för embeddings. De möjliggör effektiv likhetssökning och är viktiga för Retrieval-Augmented Generation (RAG) mönster där du behöver hitta relevant information från stora datamängder baserat på semantisk likhet snarare än exakta träffar.

![Figure: Vector database architecture showing how embeddings are stored and retrieved for similarity search.](../../../translated_images/sv/vector.f12f114934e223df.webp)

> **Obs**: I denna kurs täcker vi inte vektordatabaser men nämner dem eftersom de ofta används i verkliga tillämpningar.

- **Agenter och MCP**: AI-komponenter som autonomt interagerar med modeller, verktyg och externa system. Model Context Protocol (MCP) erbjuder ett standardiserat sätt för agenter att säkert få tillgång till externa datakällor och verktyg. Lär dig mer i vår [MCP för nybörjare](https://github.com/microsoft/mcp-for-beginners) kurs.

I Java AI-applikationer använder du tokens för textbehandling, embeddings för semantisk sökning och RAG, vektordatabaser för datahämtning och agenter med MCP för att bygga intelligenta system som använder verktyg.

![Figure: how a prompt becomes a reply—tokens, vectors, optional RAG lookup, LLM thinking, and an MCP agent all in one quick flow..](../../../translated_images/sv/flow.f4ef62c3052d12a8.webp)

### AI-utvecklingsverktyg och bibliotek för Java

Java erbjuder utmärkta verktyg för AI-utveckling. Det finns tre huvudbibliotek som vi kommer att utforska genom hela kursen - OpenAI Java SDK, Azure OpenAI SDK och Spring AI.

Här är en snabb referenstabell som visar vilket SDK som används i varje kapitels exempel:

| Kapitel | Exempel | SDK |
|---------|---------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**SDK-dokumentationslänkar:**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

OpenAI SDK är det officiella Java-biblioteket för OpenAI API:et. Det ger ett enkelt och konsekvent gränssnitt för att interagera med OpenAIs modeller, vilket gör det enkelt att integrera AI-funktioner i Java-applikationer. Kapitel 4:s Pet Story-applikation och Foundry Local-exemplet visar OpenAI SDK-ansatsen med Azure AI Foundry.

#### Spring AI

Spring AI är ett omfattande ramverk som ger AI-funktioner till Spring-applikationer och erbjuder ett konsekvent abstraktionslager över olika AI-leverantörer. Det integreras sömlöst med Spring-ekosystemet, vilket gör det till det idealiska valet för företags-Java-applikationer som behöver AI-funktioner.

Spring AI:s styrka ligger i dess smidiga integration med Spring-ekosystemet, vilket gör det enkelt att bygga produktionsfärdiga AI-applikationer med bekanta Spring-mönster som dependency injection, konfigurationshantering och testningsramverk. Du kommer att använda Spring AI i kapitel 2 och 4 för att bygga applikationer som utnyttjar både OpenAI och Model Context Protocol (MCP) Spring AI-biblioteken.

##### Model Context Protocol (MCP)

[Model Context Protocol (MCP)](https://modelcontextprotocol.io/) är en framväxande standard som möjliggör för AI-applikationer att interagera säkert med externa datakällor och verktyg. MCP erbjuder ett standardiserat sätt för AI-modeller att få tillgång till kontextuell information och utföra åtgärder i dina applikationer.

I kapitel 4 bygger du en enkel MCP-räknartjänst som demonstrerar grunderna i Model Context Protocol med Spring AI, och visar hur man skapar grundläggande verktygsintegrationer och tjänstearkitekturer.

#### Azure OpenAI Java SDK

Azure OpenAI-klientbiblioteket för Java är en anpassning av OpenAIs REST-API:er som erbjuder ett idiomatiskt gränssnitt och integration med resten av Azure SDK-ekosystemet. I kapitel 3 bygger du applikationer med Azure OpenAI SDK, inklusive chattapplikationer, funktionsanrop och RAG (Retrieval-Augmented Generation) mönster.

> Obs: Azure OpenAI SDK ligger efter OpenAI Java SDK när det gäller funktioner, så överväg att använda OpenAI Java SDK för framtida projekt.

## Sammanfattning

Det avslutar grunderna! Du förstår nu:

- De grundläggande koncepten bakom generativ AI - från LLMs och prompt engineering till tokens, embeddings och vektordatabaser
- Dina verktygsalternativ för Java AI-utveckling: Azure OpenAI SDK, Spring AI och OpenAI Java SDK
- Vad Model Context Protocol är och hur det gör det möjligt för AI-agenter att arbeta med externa verktyg

## Nästa steg

[Kapitel 2: Sätta upp utvecklingsmiljön](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Ansvarsfriskrivning**:
Detta dokument har översatts med hjälp av AI-översättningstjänsten [Co-op Translator](https://github.com/Azure/co-op-translator). Även om vi strävar efter noggrannhet, var vänlig notera att automatiska översättningar kan innehålla fel eller brister. Det ursprungliga dokumentet på dess modersmål bör betraktas som den auktoritativa källan. För kritisk information rekommenderas professionell mänsklig översättning. Vi ansvarar inte för några missförstånd eller feltolkningar som uppstår till följd av användningen av denna översättning.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->