# Introduktion til Generativ AI - Java Edition

## Hvad du vil lære

- **Grundlæggende om Generativ AI** inklusive LLM'er, prompt engineering, tokens, embeddings og vektordatabaser
- **Sammenlign Java AI udviklingsværktøjer** inklusive Azure OpenAI SDK, Spring AI og OpenAI Java SDK
- **Opdag Model Context Protocol** og dens rolle i AI-agenters kommunikation

## Indholdsfortegnelse

- [Introduktion](#introduktion)
- [En hurtig opfriskning af Generativ AI-koncepter](#en-hurtig-opfriskning-af-generativ-ai-koncepter)
- [Gennemgang af prompt engineering](#gennemgang-af-prompt-engineering)
- [Tokens, embeddings og agenter](#tokens-embeddings-og-agenter)
- [AI Udviklingsværktøjer og biblioteker til Java](#ai-udviklingsværktøjer-og-biblioteker-til-java)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [Opsummering](#opsummering)
- [Næste skridt](#næste-skridt)

## Introduktion

Velkommen til det første kapitel af Generativ AI for Begyndere - Java Edition! Denne grundlæggende lektion introducerer dig for kernekoncepterne i generativ AI og hvordan du arbejder med dem ved hjælp af Java. Du vil lære om de essentielle byggeklodser for AI-applikationer, inklusive Large Language Models (LLM'er), tokens, embeddings og AI-agenter. Vi vil også udforske de primære Java-værktøjer, du vil bruge gennem hele kurset.

### En hurtig opfriskning af Generativ AI-koncepter

Generativ AI er en type kunstig intelligens, der skaber nyt indhold, såsom tekst, billeder eller kode, baseret på mønstre og relationer lært fra data. Generative AI-modeller kan generere menneskelignende svar, forstå konteksten og nogle gange endda skabe indhold, der virker menneskeligt.

Når du udvikler dine Java AI-applikationer, vil du arbejde med **generative AI-modeller** for at skabe indhold. Nogle kapaciteter af generative AI-modeller inkluderer:

- **Tekstgenerering**: Udfærdigelse af menneskelignende tekst til chatbots, indhold og tekstfuldførelse.
- **Billedgenerering og analyse**: Fremstilling af realistiske billeder, forbedring af fotos og genkendelse af objekter.
- **Kodegenerering**: Skrivning af kodefragmenter eller scripts.

Der findes specifikke typer modeller, der er optimeret til forskellige opgaver. For eksempel kan både **Small Language Models (SLM'er)** og **Large Language Models (LLM'er)** håndtere tekstgenerering, hvor LLM'er typisk tilbyder bedre ydeevne for komplekse opgaver. Til billedrelaterede opgaver vil du bruge specialiserede vision-modeller eller multimodale modeller.

![Figure: Generative AI model types and use cases.](../../../translated_images/da/llms.225ca2b8a0d34473.webp)

Selvfølgelig er svarene fra disse modeller ikke altid perfekte. Du har sikkert hørt om modeller, der "hallucinerer" eller genererer ukorrekte oplysninger på en autoritativ måde. Men du kan hjælpe modellen med at generere bedre svar ved at give den klare instruktioner og kontekst. Det er her, **prompt engineering** kommer ind i billedet.

#### Gennemgang af prompt engineering

Prompt engineering er praksissen med at designe effektive input for at styre AI-modeller mod ønskede output. Det indebærer:

- **Klarhed**: At gøre instruktioner klare og entydige.
- **Kontekst**: At give nødvendig baggrundsinformation.
- **Begrænsninger**: At specificere eventuelle begrænsninger eller formater.

Nogle bedste praksisser for prompt engineering inkluderer promptdesign, klare instruktioner, opdeling af opgaver, one-shot og few-shot læring samt prompt tuning. Det er vigtigt at teste forskellige prompts for at finde ud af, hvad der fungerer bedst til dit specifikke anvendelsesområde.

Når du udvikler applikationer, vil du arbejde med forskellige typer prompts:
- **Systemprompts**: Sætter basale regler og kontekst for modellens opførsel
- **Brugerprompts**: Inputdata fra dine applikationsbrugere
- **Assistentprompts**: Modellens svar baseret på system- og brugerprompts

> **Lær mere**: Lær mere om prompt engineering i [Prompt Engineering-kapitlet i GenAI for Beginners-kurset](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### Tokens, embeddings og agenter

Når du arbejder med generative AI-modeller, vil du støde på termer som **tokens**, **embeddings**, **agenter** og **Model Context Protocol (MCP)**. Her er en detaljeret oversigt over disse koncepter:

- **Tokens**: Tokens er den mindste enhed af tekst i en model. De kan være ord, tegn eller dele af ord (subwords). Tokens bruges til at repræsentere tekstdata i et format, som modellen kan forstå. For eksempel kan sætningen "The quick brown fox jumped over the lazy dog" tokeniseres som ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] eller ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"], afhængigt af tokeniseringsstrategien.

![Figure: Generative AI tokens example of breaking words into tokens](../../../translated_images/da/tokens.6283ed277a2ffff4.webp)

Tokenisering er processen med at opdele tekst i disse mindre enheder. Dette er vigtigt, fordi modeller arbejder med tokens frem for rå tekst. Antallet af tokens i en prompt påvirker modellens svarlængde og kvalitet, da modeller har tokenbegrænsninger for deres kontekstvindue (f.eks. 128K tokens for GPT-4o's samlede kontekst, inklusive både input og output).

  I Java kan du bruge biblioteker som OpenAI SDK til automatisk at håndtere tokenisering, når du sender forespørgsler til AI-modeller.

- **Embeddings**: Embeddings er vektorrepræsentationer af tokens, der indfanger semantisk mening. De er numeriske repræsentationer (typisk arrays af flydende tal), der gør det muligt for modeller at forstå relationer mellem ord og generere kontekstuelle relevante svar. Lignende ord har lignende embeddings, hvilket gør det muligt for modellen at forstå begreber som synonymer og semantiske relationer.

![Figure: Embeddings](../../../translated_images/da/embedding.398e50802c0037f9.webp)

  I Java kan du generere embeddings ved hjælp af OpenAI SDK eller andre biblioteker, der understøtter generering af embeddings. Disse embeddings er essentielle til opgaver som semantisk søgning, hvor du ønsker at finde lignende indhold baseret på betydning snarere end nøjagtig tekstlig match.

- **Vektordatabaser**: Vektordatabaser er specialiserede lagersystemer optimeret til embeddings. De muliggør effektiv søgning efter ligheder og er afgørende for Retrieval-Augmented Generation (RAG)-mønstre, hvor du skal finde relevant information fra store datasæt baseret på semantisk lighed snarere end nøjagtige tekstmatches.

![Figure: Vector database architecture showing how embeddings are stored and retrieved for similarity search.](../../../translated_images/da/vector.f12f114934e223df.webp)

> **Note**: I dette kursus vil vi ikke dække vektordatabaser, men de er værd at nævne, da de ofte anvendes i virkelige applikationer.

- **Agenter & MCP**: AI-komponenter, der autonomt interagerer med modeller, værktøjer og eksterne systemer. Model Context Protocol (MCP) giver en standardiseret måde for agenter at få sikker adgang til eksterne datakilder og værktøjer. Lær mere i vores [MCP for Beginners](https://github.com/microsoft/mcp-for-beginners) kursus.

I Java AI-applikationer vil du bruge tokens til tekstbehandling, embeddings til semantisk søgning og RAG, vektordatabaser til dataopslag og agenter med MCP til at opbygge intelligente, værktøjsbrugende systemer.

![Figure: how a prompt becomes a reply—tokens, vectors, optional RAG lookup, LLM thinking, and an MCP agent all in one quick flow..](../../../translated_images/da/flow.f4ef62c3052d12a8.webp)

### AI Udviklingsværktøjer og biblioteker til Java

Java tilbyder fremragende værktøjer til AI-udvikling. Der er tre hovedbiblioteker, som vi vil udforske gennem hele kurset - OpenAI Java SDK, Azure OpenAI SDK, og Spring AI.

Her er en hurtig referencetabel, der viser hvilket SDK der bruges i hvert kapitel:

| Kapitel | Eksempel | SDK |
|---------|--------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**SDK Dokumentationslinks:**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

OpenAI SDK er det officielle Java-bibliotek til OpenAI API'et. Det giver en simpel og konsistent grænseflade til at interagere med OpenAI’s modeller, hvilket gør det nemt at integrere AI-kapabiliteter i Java-applikationer. Kapitel 4's Pet Story-applikation og Foundry Local-eksemplet demonstrerer OpenAI SDK-tilgangen med Azure AI Foundry.

#### Spring AI

Spring AI er et omfattende framework, der bringer AI-kapabiliteter til Spring-applikationer og giver et ensartet abstraktionslag på tværs af forskellige AI-udbydere. Det integreres sømløst med Spring-økosystemet, hvilket gør det til det ideelle valg for enterprise Java-applikationer, der har brug for AI-kapaciteter.

Spring AI’s styrke ligger i dets tætte integration med Spring-økosystemet, hvilket gør det nemt at bygge produktionsklare AI-applikationer med velkendte Spring-mønstre som dependency injection, konfigurationsstyring og testframeworks. Du vil bruge Spring AI i kapitel 2 og 4 til at bygge applikationer, der udnytter både OpenAI og Model Context Protocol (MCP) Spring AI biblioteker.

##### Model Context Protocol (MCP)

[Model Context Protocol (MCP)](https://modelcontextprotocol.io/) er en ny standard, der muliggør, at AI-applikationer sikkert kan interagere med eksterne datakilder og værktøjer. MCP giver en standardiseret måde for AI-modeller at få adgang til kontekstuel information og udføre handlinger i dine applikationer.

I kapitel 4 vil du bygge en simpel MCP kalkulator-service, der demonstrerer grundlæggende principper i Model Context Protocol med Spring AI, og viser hvordan man skaber basale værktøjsintegrationer og servicearkitekturer.

#### Azure OpenAI Java SDK

Azure OpenAI-klientbiblioteket til Java er en tilpasning af OpenAI’s REST API’er, som leverer en idiomatisk grænseflade og integration med resten af Azure SDK-økosystemet. I kapitel 3 vil du bygge applikationer ved hjælp af Azure OpenAI SDK, inklusive chat-applikationer, funktionskald og RAG (Retrieval-Augmented Generation) mønstre.

> Note: Azure OpenAI SDK halter lidt bagefter OpenAI Java SDK med hensyn til funktioner, så til fremtidige projekter bør du overveje at bruge OpenAI Java SDK.

## Opsummering

Sådan! Du har nu forstået:

- Kernekoncepterne bag generativ AI – fra LLM’er og prompt engineering til tokens, embeddings og vektordatabaser
- Dine værktøjsmuligheder til Java AI-udvikling: Azure OpenAI SDK, Spring AI og OpenAI Java SDK
- Hvad Model Context Protocol er, og hvordan det gør det muligt for AI-agenter at arbejde med eksterne værktøjer

## Næste skridt

[Kapitel 2: Opsætning af udviklingsmiljøet](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Ansvarsfraskrivelse**:
Dette dokument er blevet oversat ved hjælp af AI-oversættelsestjenesten [Co-op Translator](https://github.com/Azure/co-op-translator). Selvom vi bestræber os på nøjagtighed, skal du være opmærksom på, at automatiserede oversættelser kan indeholde fejl eller unøjagtigheder. Det originale dokument på dets oprindelige sprog bør betragtes som den autoritative kilde. For kritisk information anbefales professionel menneskelig oversættelse. Vi påtager os intet ansvar for misforståelser eller fejltolkninger, der opstår som følge af brugen af denne oversættelse.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->