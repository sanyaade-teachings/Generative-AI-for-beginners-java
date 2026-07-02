# Introducere în Generative AI - Ediția Java

## Ce vei învăța

- **Fundamentele Generative AI** inclusiv LLM-uri, ingineria prompturilor, tokeni, embeddings și baze de date vectoriale
- **Compară instrumentele de dezvoltare AI în Java** inclusiv Azure OpenAI SDK, Spring AI și OpenAI Java SDK
- **Descoperă Model Context Protocol** și rolul său în comunicarea agenților AI

## Cuprins

- [Introducere](#introducere)
- [O recapitulare rapidă a conceptelor Generative AI](#o-recapitulare-rapidă-a-conceptelor-generative-ai)
- [Revizuire inginerie prompturi](#revizuire-inginerie-prompturi)
- [Tokeni, embeddings și agenți](#tokeni-embeddings-și-agenți)
- [Instrumente și biblioteci pentru dezvoltare AI în Java](#instrumente-și-biblioteci-pentru-dezvoltare-ai-în-java)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [Sumar](#sumar)
- [Pașii următori](#pașii-următori)

## Introducere

Bine ai venit la primul capitol din Generative AI pentru Începători - Ediția Java! Această lecție fundamentală îți introduce conceptele de bază ale generative AI și cum să lucrezi cu ele folosind Java. Vei învăța despre blocurile esențiale ale aplicațiilor AI, inclusiv modele mari de limbaj (LLM), tokeni, embeddings și agenți AI. Vom explora și principalele instrumente Java pe care le vei folosi pe tot parcursul acestui curs.

### O recapitulare rapidă a conceptelor Generative AI

Generative AI este un tip de inteligență artificială care creează conținut nou, cum ar fi text, imagini sau cod, bazat pe tipare și relații învățate din date. Modelele generative AI pot genera răspunsuri asemănătoare cu cele umane, înțeleg contextul și uneori chiar creează conținut care pare uman.

Pe măsură ce dezvolți aplicații AI în Java, vei lucra cu **modele generative AI** pentru a crea conținut. Unele capabilități ale modelelor generative AI includ:

- **Generare text**: Crearea de texte asemănătoare cu cele umane pentru chatbot-uri, conținut și completarea textului.
- **Generare și analiză imagini**: Producerea de imagini realiste, îmbunătățirea fotografiilor și detectarea obiectelor.
- **Generare cod**: Scrierea de fragmente de cod sau scripturi.

Există tipuri specifice de modele optimizate pentru diverse sarcini. De exemplu, atât **Modele mici de limbaj (SLM)**, cât și **Modele mari de limbaj (LLM)** pot gestiona generarea de text, LLM-urile oferind de obicei performanțe mai bune pentru sarcini complexe. Pentru sarcinile legate de imagini, s-ar utiliza modele specializate de viziune sau modele multi-modale.

![Figure: Generative AI model types and use cases.](../../../translated_images/ro/llms.225ca2b8a0d34473.webp)

Desigur, răspunsurile acestor modele nu sunt perfecte tot timpul. Probabil ai auzit despre modele care „halucinează” sau generează informații incorecte într-un mod autoritar. Dar poți ghida modelul să genereze răspunsuri mai bune oferindu-i instrucțiuni clare și context. Aici intervine **ingineria prompturilor**.

#### Revizuire inginerie prompturi

Ingineria prompturilor este practica de a proiecta inputuri eficiente pentru a ghida modelele AI spre rezultatele dorite. Aceasta implică:

- **Claritate**: Instrucțiuni clare și fără ambiguități.
- **Context**: Furnizarea informațiilor de fundal necesare.
- **Constrângeri**: Specificarea oricăror limitări sau formate.

Unele bune practici pentru ingineria prompturilor includ designul promptului, instrucțiuni clare, împărțirea sarcinilor, învățare one-shot și few-shot și ajustarea prompturilor. Testarea diferitelor prompturi este esențială pentru a găsi ce funcționează cel mai bine pentru cazul tău specific.

La dezvoltarea aplicațiilor, vei lucra cu tipuri diferite de prompturi:
- **Prompturi de sistem**: stabilesc regulile și contextul de bază pentru comportamentul modelului
- **Prompturi de utilizator**: datele de intrare de la utilizatorii aplicației tale
- **Prompturi de asistent**: răspunsurile modelului bazate pe prompturile de sistem și utilizator

> **Află mai multe**: Află mai multe despre ingineria prompturilor în [capitolul Prompt Engineering al cursului GenAI for Beginners](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### Tokeni, embeddings și agenți

Când lucrezi cu modele generative AI, vei întâlni termeni precum **tokeni**, **embeddings**, **agenți** și **Model Context Protocol (MCP)**. Iată o privire detaliată asupra acestor concepte:

- **Tokeni**: Tokenii sunt cea mai mică unitate de text într-un model. Pot fi cuvinte, caractere sau subcuvinte. Tokenii sunt folosiți pentru a reprezenta textul într-un format pe care modelul îl poate înțelege. De exemplu, propoziția „The quick brown fox jumped over the lazy dog” ar putea fi tokenizată ca ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] sau ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"] în funcție de strategia de tokenizare.

![Figure: Generative AI tokens example of breaking words into tokens](../../../translated_images/ro/tokens.6283ed277a2ffff4.webp)

Tokenizarea este procesul de a împărți textul în aceste unități mai mici. Acest lucru este crucial deoarece modelele lucrează pe tokeni, nu pe text brut. Numărul de tokeni dintr-un prompt afectează lungimea și calitatea răspunsului modelului, deoarece modelele au limite de tokeni pentru fereastra de context (ex. 128K tokeni pentru contextul total al GPT-4o, incluzând atât inputul, cât și outputul).

  În Java, poți folosi biblioteci precum OpenAI SDK pentru a gestiona tokenizarea automat când trimiți cereri către modelele AI.

- **Embeddings**: Embeddings sunt reprezentări vectoriale ale tokenilor care captează semnificația semantică. Sunt reprezentări numerice (de obicei matrice de numere cu virgulă mobilă) care permit modelelor să înțeleagă relațiile dintre cuvinte și să genereze răspunsuri relevante contextual. Cuvintele similare au embeddings similare, permițând modelului să înțeleagă concepte precum sinonimele și relațiile semantice.

![Figure: Embeddings](../../../translated_images/ro/embedding.398e50802c0037f9.webp)

  În Java, poți genera embeddings folosind OpenAI SDK sau alte biblioteci care suportă generarea de embeddings. Aceste embeddings sunt esențiale pentru sarcini precum căutarea semantică, unde dorești să găsești conținut similar bazat pe semnificație și nu pe potriviri exacte de text.

- **Baze de date vectoriale**: Baze de date vectoriale sunt sisteme specializate de stocare optimizate pentru embeddings. Ele permit căutarea eficientă a similarității și sunt cruciale pentru modelele Retrieval-Augmented Generation (RAG), unde ai nevoie să găsești informații relevante din seturi mari de date bazat pe similaritatea semantică, nu pe potriviri exacte.

![Figure: Vector database architecture showing how embeddings are stored and retrieved for similarity search.](../../../translated_images/ro/vector.f12f114934e223df.webp)

> **Notă**: În acest curs, nu vom trata bazele de date vectoriale, dar le menționăm deoarece sunt utilizate frecvent în aplicații reale.

- **Agenți & MCP**: Componente AI care interacționează autonom cu modele, instrumente și sisteme externe. Model Context Protocol (MCP) oferă o modalitate standardizată ca agenții să acceseze în siguranță surse externe de date și instrumente. Află mai multe în cursul nostru [MCP for Beginners](https://github.com/microsoft/mcp-for-beginners).

În aplicațiile AI Java, vei folosi tokeni pentru procesarea textului, embeddings pentru căutarea semantică și RAG, baze de date vectoriale pentru regăsirea datelor și agenți cu MCP pentru a construi sisteme inteligente ce folosesc instrumente.

![Figure: how a prompt becomes a reply—tokens, vectors, optional RAG lookup, LLM thinking, and an MCP agent all in one quick flow..](../../../translated_images/ro/flow.f4ef62c3052d12a8.webp)

### Instrumente și biblioteci pentru dezvoltare AI în Java

Java oferă unelte excelente pentru dezvoltarea AI. Există trei biblioteci principale pe care le vom explora pe tot parcursul cursului: OpenAI Java SDK, Azure OpenAI SDK și Spring AI.

Iată un tabel de referință rapidă care arată ce SDK este folosit în exemplele din fiecare capitol:

| Capitol | Exemplu | SDK |
|---------|---------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**Linkuri documentație SDK:**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

OpenAI SDK este biblioteca oficială Java pentru API-ul OpenAI. Oferă o interfață simplă și coerentă pentru a interacționa cu modelele OpenAI, făcând ușoară integrarea capabilităților AI în aplicațiile Java. Aplicația Pet Story și exemplul Foundry Local din Capitolul 4 demonstrează abordarea OpenAI SDK cu Azure AI Foundry.

#### Spring AI

Spring AI este un cadru cuprinzător care aduce capabilități AI în aplicațiile Spring, oferind un strat de abstractizare consistent peste diferiți furnizori AI. Se integrează perfect cu ecosistemul Spring, făcându-l alegerea ideală pentru aplicațiile enterprise Java care au nevoie de capabilități AI.

Punctul forte al Spring AI constă în integrarea sa perfectă cu ecosistemul Spring, facilitând construirea de aplicații AI gata pentru producție folosind modele familiare Spring precum injecția de dependență, gestionarea configurațiilor și cadrele de testare. Vei folosi Spring AI în Capitolele 2 și 4 pentru a construi aplicații care valorifică atât OpenAI, cât și Model Context Protocol (MCP) prin bibliotecile Spring AI.

##### Model Context Protocol (MCP)

[Model Context Protocol (MCP)](https://modelcontextprotocol.io/) este un standard emergent care permite aplicațiilor AI să interacționeze securizat cu surse externe de date și instrumente. MCP oferă o metodă standardizată pentru ca modelele AI să acceseze informații contextuale și să execute acțiuni în aplicațiile tale.

În Capitolul 4, vei construi un simplu serviciu calculator MCP care demonstrează fundamentele Model Context Protocol cu Spring AI, arătând cum să creezi integrări de bază a instrumentelor și arhitecturi de servicii.

#### Azure OpenAI Java SDK

Biblioteca client Azure OpenAI pentru Java este o adaptare a API-urilor REST OpenAI care oferă o interfață idiomatică și integrare cu restul ecosistemului Azure SDK. În Capitolul 3, vei construi aplicații folosind Azure OpenAI SDK, inclusiv aplicații de chat, apelare de funcții și modele Retrieval-Augmented Generation (RAG).

> Notă: Azure OpenAI SDK este mai puțin avansat decât OpenAI Java SDK în ceea ce privește funcționalitățile, așa că pentru proiectele viitoare ia în considerare utilizarea OpenAI Java SDK.

## Sumar

Asta încheie bazele! Acum înțelegi:

- Conceputările de bază din spatele generative AI - de la LLM-uri și ingineria prompturilor la tokeni, embeddings și baze de date vectoriale
- Opțiunile tale de unelte pentru dezvoltarea AI în Java: Azure OpenAI SDK, Spring AI și OpenAI Java SDK
- Ce este Model Context Protocol și cum permite agenților AI să lucreze cu instrumente externe

## Pașii următori

[Capitolul 2: Configurarea mediului de dezvoltare](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Declinare a responsabilității**:
Acest document a fost tradus folosind serviciul de traducere AI [Co-op Translator](https://github.com/Azure/co-op-translator). În timp ce ne străduim pentru acuratețe, vă rugăm să rețineți că traducerile automate pot conține erori sau inexactități. Documentul original în limba sa nativă trebuie considerat sursa autorizată. Pentru informații critice, se recomandă traducerea profesională realizată de un om. Nu ne asumăm responsabilitatea pentru eventualele neînțelegeri sau interpretări greșite care decurg din utilizarea acestei traduceri.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->