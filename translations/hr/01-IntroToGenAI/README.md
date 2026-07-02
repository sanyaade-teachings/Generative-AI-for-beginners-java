# Uvod u Generativnu AI - Java izdanje

## Što ćete naučiti

- **Osnove generativne AI** uključujući LLM-ove, prompt inženjering, tokene, ugradnje i vektorske baze podataka
- **Usporediti alate za razvoj Java AI** uključujući Azure OpenAI SDK, Spring AI i OpenAI Java SDK
- **Otkriti Model Context Protocol** i njegovu ulogu u komunikaciji AI agenata

## Sadržaj

- [Uvod](#uvod)
- [Brzi pregled koncepata generativne AI](#brzi-pregled-koncepata-generativne-ai)
- [Pregled prompt inženjeringa](#pregled-prompt-inženjeringa)
- [Tokeni, ugradnje i agenti](#tokeni-ugradnje-i-agentii)
- [Alati i biblioteke za razvoj AI-ja za Javu](#alati-i-biblioteke-za-razvoj-ai-ja-za-javu)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [Sažetak](#sažetak)
- [Sljedeći koraci](#sljedeći-koraci)

## Uvod

Dobrodošli u prvo poglavlje Generativne AI za početnike - Java izdanje! Ova osnivačka lekcija uvodi vas u osnovne koncepte generativne AI i kako s njima raditi koristeći Javu. Naučit ćete o ključnim sastavnim dijelovima AI aplikacija, uključujući Velike jezične modele (LLM-ove), tokene, ugradnje i AI agente. Također ćemo istražiti glavne Java alate koje ćete koristiti tijekom ovog tečaja.

### Brzi pregled koncepata generativne AI

Generativna AI je vrsta umjetne inteligencije koja stvara novi sadržaj, poput teksta, slika ili koda, na temelju obrazaca i odnosa naučenih iz podataka. Generativni AI modeli mogu generirati odgovore nalik ljudima, razumjeti kontekst i ponekad čak stvoriti sadržaj koji djeluje kao da ga je stvorio čovjek.

Razvijajući svoje Java AI aplikacije, radit ćete s **generativnim AI modelima** za stvaranje sadržaja. Neke mogućnosti generativnih AI modela uključuju:

- **Generiranje teksta**: Izrada teksta nalik ljudskom za chatbotove, sadržaj i dovršavanje teksta.
- **Generiranje i analiza slika**: Proizvodnja realističnih slika, poboljšavanje fotografija i detekcija objekata.
- **Generiranje koda**: Pisanje dijelova koda ili skripti.

Postoje specifične vrste modela koje su optimizirane za različite zadatke. Na primjer, i **Mali jezični modeli (SLM)** i **Veliki jezični modeli (LLM)** mogu obraditi generiranje teksta, pri čemu LLM-ovi obično nude bolju izvedbu za složenije zadatke. Za zadatke vezane uz slike koristili biste specijalizirane vizualne modele ili multimodalne modele.

![Figure: Generative AI model types and use cases.](../../../translated_images/hr/llms.225ca2b8a0d34473.webp)

Naravno, odgovori tih modela nisu uvijek savršeni. Vjerojatno ste čuli da modeli "haluciniraju" ili generiraju netočne informacije na autoritativan način. Ali možete pomoći modelu da generira bolje odgovore pružajući mu jasne upute i kontekst. Ovo je mjesto gdje dolazi **prompt inženjering**.

#### Pregled prompt inženjeringa

Prompt inženjering je praksa dizajniranja učinkovitih ulaza za usmjeravanje AI modela prema željenim izlazima. Uključuje:

- **Jasnoću**: Davanje jasnih i nedvosmislenih uputa.
- **Kontekst**: Pružanje potrebnih informacija u pozadini.
- **Ograničenja**: Navođenje bilo kakvih ograničenja ili formata.

Neke najbolje prakse za prompt inženjering uključuju dizajn prompta, jasne upute, razbijanje zadataka, učenje na jednom ili nekoliko primjera i podešavanje prompta. Testiranje različitih prompta ključno je za pronalaženje onoga što najbolje funkcionira za vaš specifični slučaj upotrebe.

Pri razvoju aplikacija radit ćete s različitim vrstama promptova:
- **Sistemski promptovi**: Postavljaju osnovna pravila i kontekst ponašanja modela
- **Korisnički promptovi**: Ulazni podaci od korisnika vaše aplikacije
- **Asistentski promptovi**: Modelovi odgovori na temelju sistemskih i korisničkih promptova

> **Saznajte više**: Saznajte više o prompt inženjeringu u [poglavlju o prompt inženjeringu tečaja GenAI za početnike](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### Tokeni, ugradnje i agenti

Kada radite s generativnim AI modelima, naići ćete na pojmove poput **tokena**, **ugradnji**, **agenata** i **Model Context Protocol (MCP)**. Evo detaljnog pregleda ovih koncepata:

- **Tokeni**: Tokeni su najmanja jedinica teksta u modelu. Mogu biti riječi, znakovi ili podriječi. Tokeni se koriste za predstavljanje tekstualnih podataka u formatu koji model može razumjeti. Na primjer, rečenica "The quick brown fox jumped over the lazy dog" može biti tokenizirana kao ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] ili ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"] ovisno o strategiji tokenizacije.

![Figure: Generative AI tokens example of breaking words into tokens](../../../translated_images/hr/tokens.6283ed277a2ffff4.webp)

Tokenizacija je proces razbijanja teksta na te manje jedinice. Ovo je ključno jer modeli rade s tokenima, a ne s neobrađenim tekstom. Broj tokena u promptu utječe na duljinu i kvalitetu modelovog odgovora jer modeli imaju limit broja tokena unutar kontekstnog prozora (npr. 128K tokena za ukupni kontekst GPT-4o, uključujući ulaz i izlaz).

  U Javi možete koristiti biblioteke poput OpenAI SDK-a za automatsko rukovanje tokenizacijom prilikom slanja zahtjeva AI modelima.

- **Ugradnje**: Ugradnje su vektorske reprezentacije tokena koje hvataju semantičko značenje. To su numeričke reprezentacije (obično nizovi decimalnih brojeva) koje omogućuju modelima razumijevanje odnosa između riječi i generiranje odgovora relevantnih kontekstu. Slične riječi imaju slične ugradnje, omogućujući modelu razumijevanje pojmova poput sinonima i semantičkih veza.

![Figure: Embeddings](../../../translated_images/hr/embedding.398e50802c0037f9.webp)

  U Javi možete generirati ugradnje koristeći OpenAI SDK ili druge biblioteke koje podržavaju generiranje ugradnji. Te su ugradnje ključne za zadatke poput semantičkog pretraživanja, gdje želite pronaći sličan sadržaj na temelju značenja, a ne točnih podudaranja teksta.

- **Vektorske baze podataka**: Vektorske baze podataka su specijalizirani sustavi za pohranu optimizirani za ugradnje. Omogućuju učinkovito pretraživanje sličnosti i ključni su za obrasce poput Pretraživanjem potpomognene generacije (RAG) gdje je potrebno pronaći relevantne informacije iz velikih skupova podataka na temelju semantičke sličnosti, a ne točnih podudaranja.

![Figure: Vector database architecture showing how embeddings are stored and retrieved for similarity search.](../../../translated_images/hr/vector.f12f114934e223df.webp)

> **Napomena**: U ovom tečaju nećemo pokrivati vektorske baze podataka, ali ih spominjemo jer se često koriste u stvarnim aplikacijama.

- **Agenti i MCP**: AI komponente koje autonomno komuniciraju s modelima, alatima i vanjskim sustavima. Model Context Protocol (MCP) pruža standardizirani način za agente da sigurno pristupe vanjskim izvorima podataka i alatima. Saznajte više u našem [tečaju MCP za početnike](https://github.com/microsoft/mcp-for-beginners).

U Java AI aplikacijama koristit ćete tokene za obradu teksta, ugradnje za semantičko pretraživanje i RAG, vektorske baze podataka za dohvat podataka i agente s MCP-om za izgradnju inteligentnih sustava koji koriste alate.

![Figure: how a prompt becomes a reply—tokens, vectors, optional RAG lookup, LLM thinking, and an MCP agent all in one quick flow..](../../../translated_images/hr/flow.f4ef62c3052d12a8.webp)

### Alati i biblioteke za razvoj AI-ja za Javu

Java nudi izvrsne alate za razvoj AI-ja. Postoje tri glavne biblioteke koje ćemo istražiti tijekom ovog tečaja - OpenAI Java SDK, Azure OpenAI SDK i Spring AI.

Evo kratke referentne tablice koja prikazuje koji se SDK koristi u primjerima svakog poglavlja:

| Poglavlje | Primjer | SDK |
|---------|--------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**Dokumentacijski linkovi za SDK:**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

OpenAI SDK je službena Java biblioteka za OpenAI API. Pruža jednostavno i konzistentno sučelje za interakciju s OpenAI modelima, olakšavajući integraciju AI mogućnosti u Java aplikacije. Aplikacija Pet Story i primjer Foundry Local iz poglavlja 4 demonstriraju pristup s OpenAI SDK-om u suradnji s Azure AI Foundry.

#### Spring AI

Spring AI je opsežan okvir koji donosi AI mogućnosti u Spring aplikacije, pružajući konzistentni sloj apstrakcije preko različitih AI pružatelja. Integrira se besprijekorno sa Spring ekosustavom, što ga čini idealnim izborom za enterprise Java aplikacije koje trebaju AI mogućnosti.

Snaga Spring AI leži u njegovoj besprijekornoj integraciji sa Spring ekosustavom, što olakšava izgradnju spremnih za produkciju AI aplikacija s poznatim Spring obrascima poput injekcije ovisnosti, upravljanja konfiguracijom i testnim okvirima. Koristit ćete Spring AI u poglavljima 2 i 4 za izradu aplikacija koje koriste i OpenAI i Model Context Protocol (MCP) Spring AI biblioteke.

##### Model Context Protocol (MCP)

[Model Context Protocol (MCP)](https://modelcontextprotocol.io/) je novi standard koji omogućuje AI aplikacijama sigurnu interakciju s vanjskim izvorima podataka i alatima. MCP pruža standardizirani način da AI modeli pristupe kontekstualnim informacijama i izvršavaju akcije u vašim aplikacijama.

U poglavlju 4, izgradit ćete jednostavan MCP kalkulator servis koji demonstrira osnove Model Context Protocola sa Spring AI-jem, pokazujući kako kreirati osnovne integracije alata i arhitekture servisa.

#### Azure OpenAI Java SDK

Azure OpenAI klijentska biblioteka za Javu je adaptacija OpenAI REST API-ja koja pruža idiomatsko sučelje i integraciju s ostatkom Azure SDK ekosustava. U poglavlju 3 izgradit ćete aplikacije koristeći Azure OpenAI SDK, uključujući chat aplikacije, pozivanje funkcija i RAG (Retrieval-Augmented Generation) obrasce.

> Napomena: Azure OpenAI SDK zaostaje za OpenAI Java SDK-em u smislu značajki, pa za buduće projekte razmotrite korištenje OpenAI Java SDK-a.

## Sažetak

Time smo završili osnove! Sada razumijete:

- Ključne koncepte generativne AI - od LLM-ova i prompt inženjeringa do tokena, ugradnji i vektorskih baza podataka
- Vaše opcije alata za razvoj Java AI: Azure OpenAI SDK, Spring AI i OpenAI Java SDK
- Što je Model Context Protocol i kako omogućuje AI agentima rad s vanjskim alatima

## Sljedeći koraci

[Poglavlje 2: Postavljanje razvojnih alata](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Napomena**:
Ovaj dokument je preveden korištenjem AI prevoditeljskog servisa [Co-op Translator](https://github.com/Azure/co-op-translator). Iako težimo točnosti, imajte na umu da automatski prijevodi mogu sadržavati greške ili netočnosti. Izvorni dokument na izvornom jeziku treba smatrati autoritativnim izvorom. Za važne informacije preporuča se profesionalni ljudski prijevod. Nismo odgovorni za bilo kakva nesporazumevanja ili pogrešne interpretacije koje proizlaze iz korištenja ovog prijevoda.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->