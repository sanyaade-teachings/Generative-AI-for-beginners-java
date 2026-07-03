# Uvod v Generativno AI - Java izdaja

## Kaj se boste naučili

- **Osnove generativne AI**, vključno z LLM-ji, inženirstvom pozivov, tokeni, vdelavami in vektorskimi bazami podatkov
- **Primerjava orodij za razvoj AI v Javi**, vključno z Azure OpenAI SDK, Spring AI in OpenAI Java SDK
- **Odkrijte Model Context Protocol** in njegovo vlogo v komunikaciji AI agentov

## Kazalo

- [Uvod](#uvod)
- [Hiter pregled konceptov generativne AI](#hiter-pregled-konceptov-generativne-ai)
- [Pregled inženirstva pozivov](#pregled-inženirstva-pozivov)
- [Tokeni, vdelave in agenti](#tokeni-vdelave-in-agenti)
- [Orodja in knjižnice za razvoj AI v Javi](#orodja-in-knjižnice-za-razvoj-ai-v-javi)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [Povzetek](#povzetek)
- [Naslednji koraki](#naslednji-koraki)

## Uvod

Dobrodošli v prvi poglavju Generativne AI za začetnike - Java izdaja! Ta temeljna lekcija vas bo seznanila s ključnimi koncepti generativne AI in kako z njimi delati z uporabo Jave. Naučili se boste o bistvenih gradnikih AI aplikacij, vključno z velikimi jezikovnimi modeli (LLM), tokeni, vdelavami in AI agenti. Prav tako bomo raziskali glavna orodja za Javo, ki jih boste uporabljali skozi celoten tečaj.

### Hiter pregled konceptov generativne AI

Generativna AI je vrsta umetne inteligence, ki ustvarja novo vsebino, kot so besedila, slike ali koda, na osnovi vzorcev in odnosov, naučenih iz podatkov. Generativni AI modeli lahko ustvarjajo človeško podobne odzive, razumejo kontekst in včasih celo proizvajajo vsebino, ki deluje kot človeška.

Med razvojem vaših Java AI aplikacij boste uporabljali **generativne AI modele** za ustvarjanje vsebine. Nekatere sposobnosti generativnih AI modelov vključujejo:

- **Ustvarjanje besedila**: Oblikovanje človeško podobnega besedila za klepetalne bote, vsebine in dokončanje besedil.
- **Ustvarjanje in analiza slik**: Priprava realističnih slik, izboljšava fotografij in zaznavanje objektov.
- **Generiranje kode**: Pisanje kodnih odlomkov ali skript.

Obstajajo specifične vrste modelov, optimizirane za različne naloge. Na primer, tako **Majhni jezikovni modeli (SLM)** kot tudi **Veliki jezikovni modeli (LLM)** lahko obdelujejo ustvarjanje besedila, pri čemer LLM-ji običajno nudijo boljšo zmogljivost za kompleksne naloge. Za naloge, povezane s slikami, bi uporabili specializirane modele vida ali multimodalne modele.

![Slika: vrste generativnih AI modelov in primeri uporabe.](../../../translated_images/sl/llms.225ca2b8a0d34473.webp)

Seveda odzivi teh modelov niso vedno popolni. Verjetno ste že slišali za modele, ki "halucinirajo" oziroma ustvarjajo napačne informacije na avtoritativen način. Pomagate jim lahko, da ustvarijo boljše odzive, če jim zagotovite jasna navodila in kontekst. Tu pride do izraza **inženirstvo pozivov**.

#### Pregled inženirstva pozivov

Inženirstvo pozivov je praksa oblikovanja učinkovitih vhodov za usmerjanje AI modelov k željenim izhodom. Vključuje:

- **Jasnost**: Navodila naj bodo jasna in nedvoumna.
- **Kontekst**: Zagotavljanje potrebnih ozadnih informacij.
- **Omejitve**: Določanje morebitnih omejitev ali formatov.

Nekatere najboljše prakse za inženirstvo pozivov vključujejo oblikovanje pozivov, jasna navodila, razdelitev nalog, učenje na enem ali nekaj primerih in prilagajanje pozivov. Preizkušanje različnih pozivov je bistveno za najdbo najboljšega pristopa za vaš specifični primer uporabe.

Med razvojem aplikacij boste uporabljali različne vrste pozivov:
- **Sistemski pozivi**: Nastavijo osnovna pravila in kontekst vedenja modela
- **Uporabniški pozivi**: Vhodni podatki od uporabnikov vaše aplikacije
- **Asistentski pozivi**: Modelovi odgovori, narejeni na podlagi sistemskih in uporabniških pozivov

> **Več informacij**: Več o inženirstvu pozivov si lahko preberete v [poglavju o inženirstvu pozivov v tečaju Generativna AI za začetnike](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### Tokeni, vdelave in agenti

Pri delu z generativnimi AI modeli boste naleteli na izraze, kot so **tokeni**, **vdelave**, **agenti** in **Model Context Protocol (MCP)**. Tukaj je podroben pregled teh pojmov:

- **Tokeni**: Tokeni so najmanjša enota besedila v modelu. Lahko so besede, znaki ali delne besede. Tokeni se uporabljajo za predstavitev besedilnih podatkov v obliki, ki jo model lahko razume. Na primer, stavek "The quick brown fox jumped over the lazy dog" je lahko razdeljen na tokene kot ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] ali ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"], odvisno od strategije tokenizacije.

![Slika: primer tokenov v generativni AI, kako se besede razbijejo v tokene](../../../translated_images/sl/tokens.6283ed277a2ffff4.webp)

Tokenizacija je proces razčlenjevanja besedila na te manjše enote. To je ključno, saj modeli delujejo na tokenih in ne na surovem besedilu. Število tokenov v pozivu vpliva na dolžino in kakovost modelovega odziva, ker modeli imajo omejitve glede števila tokenov v kontekstnem oknu (npr. 128K tokenov za GPT-4o celoten kontekst, vključno z vhodom in izhodom).

  V Javi lahko uporabite knjižnice, kot je OpenAI SDK, ki samodejno obravnava tokenizacijo pri pošiljanju zahtevkov modelom.

- **Vdelave**: Vdelave so vektorske predstavitve tokenov, ki zajemajo semantični pomen. So numerične predstavitve (običajno polja plavajočih števil), ki omogočajo modelom razumevanje povezav med besedami in generiranje kontekstualno ustreznih odgovorov. Podobne besede imajo podobne vdelave, kar modelu omogoča razumevanje pojmov, kot so sopomenke in semantični odnosi.

![Slika: vdelave](../../../translated_images/sl/embedding.398e50802c0037f9.webp)

  V Javi lahko vdelave generirate z uporabo OpenAI SDK ali drugih knjižnic, ki podpirajo generiranje vdelav. Te vdelave so bistvene za naloge, kot je semantično iskanje, kjer želite najti podobno vsebino na podlagi pomena in ne izključno z besedilnim ujemanjem.

- **Vektorske baze podatkov**: Vektorske baze podatkov so specializirani sistemi za shranjevanje, optimizirani za vdelave. Omogočajo učinkovito iskanje podobnosti in so ključni za vzorce Retrieval-Augmented Generation (RAG), kjer je treba najti relevantne informacije iz velikih zbirk podatkov na podlagi semantične podobnosti namesto točnih ujemanj.

![Slika: arhitektura vektorske baze podatkov, ki prikazuje, kako se vdelave shranjujejo in pridobivajo za iskanje podobnosti.](../../../translated_images/sl/vector.f12f114934e223df.webp)

> **Opomba**: V tem tečaju ne bomo pokrivali vektorskih baz podatkov, vendar jih omenjamo, ker se pogosto uporabljajo v realnih aplikacijah.

- **Agenti in MCP**: AI komponente, ki avtonomno sodelujejo z modeli, orodji in zunanjimi sistemi. Model Context Protocol (MCP) zagotavlja standardiziran način, da agenti varno dostopajo do zunanjih virov podatkov in orodij. Več o tem se naučite v našem tečaju [MCP za začetnike](https://github.com/microsoft/mcp-for-beginners).

V Java AI aplikacijah boste uporabili tokene za obdelavo besedila, vdelave za semantično iskanje in RAG, vektorske baze za iskanje podatkov ter agente z MCP za gradnjo inteligentnih sistemov, ki uporabljajo orodja.

![Slika: kako poziv postane odgovor — tokeni, vektorji, opcijsko RAG iskanje, razmišljanje LLM in MCP agent v enem hitrem toku..](../../../translated_images/sl/flow.f4ef62c3052d12a8.webp)

### Orodja in knjižnice za razvoj AI v Javi

Java ponuja odlična orodja za razvoj AI. Obstajajo tri glavne knjižnice, ki jih bomo raziskali skozi ta tečaj - OpenAI Java SDK, Azure OpenAI SDK in Spring AI.

Tukaj je hiter referenčni tabelarični pregled, katero SDK se uporablja v primerih posameznih poglavij:

| Poglavje | Vzorec | SDK |
|---------|--------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**Povezave do dokumentacije SDK:**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

OpenAI SDK je uradna Java knjižnica za OpenAI API. Omogoča enostaven in dosleden vmesnik za delo z modeli OpenAI, kar omogoča enostavno integracijo AI zmožnosti v Java aplikacije. Aplikacija Pet Story iz 4. poglavja in primer Foundry Local prikazujeta pristop OpenAI SDK skupaj z Azure AI Foundry.

#### Spring AI

Spring AI je celovit okvir, ki prinaša AI zmogljivosti v Spring aplikacije, zagotavljajoč dosledno abstrakcijo med različnimi AI ponudniki. Se brezhibno integrira v Spring ekosistem, zaradi česar je idealna izbira za podjetniške Java aplikacije, ki potrebujejo AI zmogljivosti.

Moč Spring AI je v njegovi elegantni integraciji s Spring ekosistemom, kar omogoča enostavno gradnjo produkcijsko pripravljenih AI aplikacij z znanimi Spring vzorci, kot so odvisnostna injekcija, upravljanje konfiguracije in testni okvirji. Spring AI boste uporabljali v 2. in 4. poglavju za gradnjo aplikacij, ki uporabljajo tako OpenAI kot Model Context Protocol (MCP) Spring AI knjižnice.

##### Model Context Protocol (MCP)

[Model Context Protocol (MCP)](https://modelcontextprotocol.io/) je nastajajoči standard, ki omogoča AI aplikacijam, da varno sodelujejo z zunanjimi viri podatkov in orodji. MCP zagotavlja standardiziran način, da AI modeli dostopajo do kontekstualnih informacij in izvajajo akcije v vaših aplikacijah.

V 4. poglavju boste zgradili preprosto MCP kalkulator storitev, ki prikazuje osnovne principe Model Context Protocol s Spring AI in prikazuje, kako ustvariti osnovne integracije orodij in arhitekture storitev.

#### Azure OpenAI Java SDK

Azure OpenAI knjižnica za Javo je prilagoditev OpenAI REST API-jev, ki omogoča idiomatičen vmesnik in integracijo z ostalim Azure SDK ekosistemom. V 3. poglavju boste razvijali aplikacije z uporabo Azure OpenAI SDK, vključno s klepetalnimi aplikacijami, klici funkcij in vzorci RAG (Retrieval-Augmented Generation).

> Opomba: Azure OpenAI SDK zaostaja za OpenAI Java SDK glede funkcionalnosti, zato za prihodnje projekte razmislite o uporabi OpenAI Java SDK.

## Povzetek

To je konec osnov! Zdaj razumete:

- Ključne koncepte generativne AI - od LLM-jev in inženirstva pozivov do tokenov, vdelav in vektorskih baz podatkov
- Vaše možnosti orodij za razvoj AI v Javi: Azure OpenAI SDK, Spring AI in OpenAI Java SDK
- Kaj je Model Context Protocol in kako omogoča AI agentom delo z zunanjimi orodji

## Naslednji koraki

[Poglavje 2: Nastavitev razvojnega okolja](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Omejitev odgovornosti**:
Ta dokument je bil preveden z uporabo AI prevajalske storitve [Co-op Translator](https://github.com/Azure/co-op-translator). Čeprav si prizadevamo za natančnost, vas prosimo, da upoštevate, da avtomatizirani prevodi lahko vsebujejo napake ali netočnosti. Izvirni dokument v njegovem izvirnem jeziku je treba obravnavati kot avtoritativni vir. Za kritične informacije je priporočljiv strokovni človeški prevod. Ne odgovarjamo za morebitna nesporazume ali napačne interpretacije, ki izhajajo iz uporabe tega prevoda.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->