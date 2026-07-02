# Introduksjon til generativ AI - Java-utgave

## Hva du vil lære

- **Grunnleggende om generativ AI** inkludert LLM-er, prompt engineering, tokens, embeddings og vektordatabaser
- **Sammenligning av Java AI-utviklingsverktøy** inkludert Azure OpenAI SDK, Spring AI og OpenAI Java SDK
- **Oppdag Model Context Protocol** og dens rolle i kommunikasjon mellom AI-agenter

## Innholdsfortegnelse

- [Introduksjon](#introduksjon)
- [En rask oppfriskning av generative AI-konsepter](#en-rask-oppfriskning-av-generative-ai-konsepter)
- [Gjennomgang av prompt engineering](#gjennomgang-av-prompt-engineering)
- [Tokens, embeddings og agenter](#tokens-embeddings-og-agenter)
- [AI-utviklingsverktøy og biblioteker for Java](#ai-utviklingsverktøy-og-biblioteker-for-java)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [Oppsummering](#oppsummering)
- [Neste steg](#neste-steg)

## Introduksjon

Velkommen til det første kapittelet i Generativ AI for nybegynnere - Java-utgave! Denne grunnleggende leksjonen introduserer deg for kjernene av generativ AI og hvordan du kan jobbe med det ved hjelp av Java. Du vil lære om de essensielle byggeklossene for AI-applikasjoner, inkludert store språkmodeller (LLM-er), tokens, embeddings og AI-agenter. Vi vil også utforske hovedverktøyene i Java som du vil bruke gjennom hele kurset.

### En rask oppfriskning av generative AI-konsepter

Generativ AI er en type kunstig intelligens som skaper nytt innhold, som tekst, bilder eller kode, basert på mønstre og relasjoner lært fra data. Generative AI-modeller kan generere menneskelignende svar, forstå kontekst og noen ganger til og med lage innhold som virker menneskelig.

Når du utvikler dine Java AI-applikasjoner, vil du jobbe med **generative AI-modeller** for å lage innhold. Noen av funksjonene til generative AI-modeller inkluderer:

- **Tekstgenerering**: Lage menneskelignende tekst for chatboter, innhold og tekstfullføring.
- **Bildegenerering og analyse**: Lage realistiske bilder, forbedre bilder og oppdage objekter.
- **Kodegenerering**: Skrive kodesnutter eller skript.

Det finnes spesifikke typer modeller optimalisert for ulike oppgaver. For eksempel kan både **små språkmodeller (SLM)** og **store språkmodeller (LLM)** håndtere tekstgenerering, hvor LLM-er vanligvis gir bedre ytelse for komplekse oppgaver. For bildeoppgaver bruker du spesialiserte synsmodeller eller multimodale modeller.

![Figure: Generative AI model types and use cases.](../../../translated_images/no/llms.225ca2b8a0d34473.webp)

Selvfølgelig er ikke svarene fra disse modellene perfekte hele tiden. Du har sikkert hørt om modeller som "hallusinerer" eller genererer feilaktig informasjon på en autoritativ måte. Men du kan hjelpe til med å styre modellen til å generere bedre svar ved å gi den klare instruksjoner og kontekst. Dette er der **prompt engineering** kommer inn.

#### Gjennomgang av prompt engineering

Prompt engineering er praksisen med å designe effektive innspill for å styre AI-modeller mot ønskede resultater. Det innebærer:

- **Klarhet**: Gjøre instruksjonene klare og entydige.
- **Kontekst**: Gi nødvendig bakgrunnsinformasjon.
- **Begrensninger**: Spesifisere eventuelle begrensninger eller formater.

Noen beste praksiser for prompt engineering inkluderer design av prompt, klare instrukser, oppdeling av oppgaver, one-shot og few-shot læring, og prompt-tuning. Det er viktig å teste forskjellige prompts for å finne det som fungerer best for ditt spesifikke bruksområde.

Når du utvikler applikasjoner, vil du jobbe med forskjellige prompt-typer:
- **Systemprompter**: Setter grunnreglene og konteksten for modellens oppførsel
- **Brukerprompter**: Inndata fra brukerne av applikasjonen din
- **Assistentprompter**: Modellens svar basert på system- og brukerprompter

> **Lær mer**: Lær mer om prompt engineering i [Prompt Engineering-kapitlet i GenAI for Beginners-kurset](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### Tokens, embeddings og agenter

Når du jobber med generative AI-modeller, vil du støte på begreper som **tokens**, **embeddings**, **agenter** og **Model Context Protocol (MCP)**. Her er en detaljert oversikt over disse konseptene:

- **Tokens**: Tokens er den minste enheten av tekst i en modell. De kan være ord, tegn eller delord. Tokens brukes for å representere tekstdata i et format som modellen kan forstå. For eksempel kan setningen "The quick brown fox jumped over the lazy dog" bli tokenisert som ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] eller ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"] avhengig av tokeniseringsstrategi.

![Figure: Generative AI tokens example of breaking words into tokens](../../../translated_images/no/tokens.6283ed277a2ffff4.webp)

Tokenisering er prosessen med å bryte ned tekst i disse mindre enhetene. Dette er avgjørende fordi modeller opererer på tokens i stedet for rå tekst. Antallet tokens i en prompt påvirker både svarlengden og kvaliteten, ettersom modeller har tokenbegrensninger for kontekstvinduet deres (f.eks. 128K tokens for GPT-4o sitt totale kontekst, inkludert både inn- og utdata).

  I Java kan du bruke biblioteker som OpenAI SDK for å håndtere tokenisering automatisk når du sender forespørsler til AI-modeller.

- **Embeddings**: Embeddings er vektorrepresentasjoner av tokens som fanger semantisk betydning. De er numeriske representasjoner (typisk arrayer av flyttall) som gjør det mulig for modeller å forstå relasjoner mellom ord og generere kontekstuelt relevante svar. Lignende ord har lignende embeddings, noe som lar modellen forstå konsepter som synonymer og semantiske relasjoner.

![Figure: Embeddings](../../../translated_images/no/embedding.398e50802c0037f9.webp)

  I Java kan du generere embeddings ved hjelp av OpenAI SDK eller andre biblioteker som støtter embeddeingsgenerering. Disse embeddings er essensielle for oppgaver som semantisk søk, hvor du ønsker å finne lignende innhold basert på mening snarere enn eksakt tekstmatch.

- **Vektordatabaser**: Vektordatabaser er spesialiserte lagringssystemer optimalisert for embeddings. De muliggjør effektiv søk etter likhet og er viktige for Retrieval-Augmented Generation (RAG)-mønstre, hvor du trenger å finne relevant informasjon fra store datasett basert på semantisk likhet snarere enn eksakte treff.

![Figure: Vector database architecture showing how embeddings are stored and retrieved for similarity search.](../../../translated_images/no/vector.f12f114934e223df.webp)

> **Merk**: I dette kurset dekker vi ikke vektordatabaser, men mener det er verdt å nevne siden de ofte brukes i virkelige applikasjoner.

- **Agenter & MCP**: AI-komponenter som autonomt interagerer med modeller, verktøy og eksterne systemer. Model Context Protocol (MCP) gir en standardisert måte for agenter å trygt få tilgang til eksterne datakilder og verktøy. Lær mer i vårt [MCP for Beginners](https://github.com/microsoft/mcp-for-beginners)-kurs.

I Java AI-applikasjoner vil du bruke tokens for tekstbehandling, embeddings for semantisk søk og RAG, vektordatabaser for datainnhenting, og agenter med MCP for å bygge intelligente systemer som bruker verktøy.

![Figure: how a prompt becomes a reply—tokens, vectors, optional RAG lookup, LLM thinking, and an MCP agent all in one quick flow..](../../../translated_images/no/flow.f4ef62c3052d12a8.webp)

### AI-utviklingsverktøy og biblioteker for Java

Java tilbyr utmerkede utviklingsverktøy for AI. Det finnes tre hovedbiblioteker vi vil utforske gjennom kurset - OpenAI Java SDK, Azure OpenAI SDK og Spring AI.

Her er en rask tabell som viser hvilket SDK som brukes i eksemplene i hvert kapittel:

| Kapittel | Eksempel | SDK |
|---------|--------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**SDK-dokumentasjonslenker:**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

OpenAI SDK er det offisielle Java-biblioteket for OpenAI API. Det gir et enkelt og konsistent grensesnitt for å samhandle med OpenAIs modeller, noe som gjør det lett å integrere AI-funksjoner i Java-applikasjoner. Kapittel 4s Pet Story-applikasjon og Foundry Local-eksempelet demonstrerer OpenAI SDK-tilnærmingen med Azure AI Foundry.

#### Spring AI

Spring AI er et omfattende rammeverk som bringer AI-funksjonalitet til Spring-applikasjoner, og tilbyr et konsistent abstraksjonslag på tvers av forskjellige AI-leverandører. Det integreres sømløst med Spring-økosystemet, noe som gjør det til det ideelle valget for virksomhets-Java-applikasjoner som trenger AI-funksjoner.

Spring AI sin styrke ligger i sin sømløse integrasjon med Spring-økosystemet, og gjør det enkelt å bygge produksjonsklare AI-applikasjoner med kjente Spring-mønstre som avhengighetsinjeksjon, konfigurasjonsstyring og testingsrammeverk. Du vil bruke Spring AI i kapittel 2 og 4 for å bygge applikasjoner som utnytter både OpenAI og Model Context Protocol (MCP) Spring AI-biblioteker.

##### Model Context Protocol (MCP)

[Model Context Protocol (MCP)](https://modelcontextprotocol.io/) er en kommende standard som gjør det mulig for AI-applikasjoner å samhandle sikkert med eksterne datakilder og verktøy. MCP gir en standardisert måte for AI-modeller å få tilgang til kontekstuell informasjon og utføre handlinger i applikasjonene dine.

I kapittel 4 vil du bygge en enkel MCP kalkulator-tjeneste som demonstrerer grunnprinsippene for Model Context Protocol med Spring AI, og viser hvordan lage grunnleggende verktøy-integrasjoner og tjenestearkitektur.

#### Azure OpenAI Java SDK

Azure OpenAI-klientbiblioteket for Java er en tilpasning av OpenAIs REST API-er som gir et idiomatisk grensesnitt og integrasjon med resten av Azure SDK-økosystemet. I kapittel 3 vil du bygge applikasjoner ved hjelp av Azure OpenAI SDK, inkludert chatteapplikasjoner, funksjonskall og RAG (Retrieval-Augmented Generation)-mønstre.

> Merk: Azure OpenAI SDK ligger etter OpenAI Java SDK når det gjelder funksjonalitet, så for fremtidige prosjekter bør du vurdere å bruke OpenAI Java SDK.

## Oppsummering

Det var grunnlaget! Nå forstår du:

- Kjernene bak generativ AI - fra LLM-er og prompt engineering til tokens, embeddings og vektordatabaser
- Dine verktøyalternativer for Java AI-utvikling: Azure OpenAI SDK, Spring AI og OpenAI Java SDK
- Hva Model Context Protocol er, og hvordan det gjør det mulig for AI-agenter å jobbe med eksterne verktøy

## Neste steg

[Kapittel 2: Sette opp utviklingsmiljøet](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Ansvarsfraskrivelse**:
Dette dokumentet er oversatt ved hjelp av AI-oversettelsestjenesten [Co-op Translator](https://github.com/Azure/co-op-translator). Selv om vi streber etter nøyaktighet, vær oppmerksom på at automatiske oversettelser kan inneholde feil eller unøyaktigheter. Det opprinnelige dokumentet på originalspråket skal betraktes som den autoritative kilden. For kritisk informasjon anbefales profesjonell menneskelig oversettelse. Vi er ikke ansvarlige for eventuelle misforståelser eller feiltolkninger som oppstår ved bruk av denne oversettelsen.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->