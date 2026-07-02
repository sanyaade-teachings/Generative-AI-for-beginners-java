# Įvadas į Generatyvinį DI – Java leidimas

## Ko Išmoksite

- **Generatyvinio DI pagrindai**, įskaitant LLM, užklausų inžineriją, žetonus, įterpimus ir vektorių duomenų bazes
- **Palyginkite Java DI kūrimo įrankius**, įskaitant Azure OpenAI SDK, Spring AI ir OpenAI Java SDK
- **Sužinokite apie Modelio Konteksto Protokolą** ir jo vaidmenį DI agentų komunikacijoje

## Turinys

- [Įvadas](#įvadas)
- [Greitas generatyvinio DI konceptų priminimas](#greitas-generatyvinio-di-konceptų-priminimas)
- [Užklausų inžinerijos apžvalga](#užklausų-inžinerijos-apžvalga)
- [Žetonai, įterpimai ir agentai](#žetonai-įterpimai-ir-agentai)
- [DI Kūrimo Įrankiai ir Bibliotekos Java kalbai](#di-kūrimo-įrankiai-ir-bibliotekos-java-kalbai)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [Santrauka](#santrauka)
- [Kiti Žingsniai](#kiti-žingsniai)

## Įvadas

Sveiki atvykę į pirmąją generatyvinio DI pradedantiesiems – Java leidimo – pamoką! Ši pagrindinė pamoka supažindins jus su pagrindinėmis generatyvinio DI sąvokomis ir kaip jas naudoti su Java. Išmoksite apie svarbiausius DI programų statybinius blokus, įskaitant Didelius Kalbos Modelius (LLM), žetonus, įterpimus ir DI agentus. Taip pat apžvelgsime pagrindinius Java įrankius, kuriuos naudosite viso kurso metu.

### Greitas generatyvinio DI konceptų priminimas

Generatyvinis DI yra dirbtinio intelekto tipas, kuris kuria naują turinį, pavyzdžiui, tekstą, vaizdus ar kodą, remdamasis išmoktomais duomenų modeliais ir ryšiais. Generatyviniai DI modeliai sugeba generuoti žmogaus panašias atsakymus, suprasti kontekstą ir kartais net kurti turinį, kuris atrodo kaip žmogaus sukurtas.

Kurdami savo Java DI programas, dirbsite su **generatyviniais DI modeliais** kurdami turinį. Kai kurios generatyvinių DI modelių galimybės apima:

- **Teksto Generavimas**: Žmogaus kalbai panašaus teksto kūrimas pokalbiams, turiniui ir teksto užbaigimui.
- **Vaizdų Generavimas ir Analizė**: Tikroviškų vaizdų kūrimas, nuotraukų gerinimas ir objektų atpažinimas.
- **Kodo Generavimas**: Kodo fragmentų ar scenarijų rašymas.

Yra specifinių modelių tipų, optimizuotų skirtingoms užduotims. Pavyzdžiui, tiek **Maži Kalbos Modeliai (SLM)**, tiek **Dideli Kalbos Modeliai (LLM)** gali apdoroti teksto generavimą, kur LLM dažniausiai suteikia geresnį našumą sudėtingesnėms užduotims. Vaizdų užduotims naudotumėte specializuotus regos modelius arba daugiarūšius modelius.

![Figura: Generatyvinio DI modelių tipai ir naudojimo atvejai.](../../../translated_images/lt/llms.225ca2b8a0d34473.webp)

Žinoma, šių modelių atsakymai nėra tobuli visada. Tikriausiai esate girdėję apie modelių „haliucinacijas“ arba klaidingos informacijos generavimą įtikinamu būdu. Bet galite padėti modeliams generuoti geresnius atsakymus, pateikdami aiškias instrukcijas ir kontekstą. Čia praverčia **užklausų inžinerija**.

#### Užklausų inžinerijos apžvalga

Užklausų inžinerija yra praktiką efektyviai kurti įvestis, vedančias DI modelius link norimų rezultatų. Ji apima:

- **Aiškumą**: Darant instrukcijas aiškias ir neambicingas.
- **Kontekstą**: Pateikiant reikalingą foninę informaciją.
- **Apribojimus**: Nurodant apribojimus ar formatus.

Kai kurios geros užklausų inžinerijos praktikos apima užklausų dizainą, aiškias instrukcijas, užduočių suskaidymą, vienkartinį ir kelių pavyzdžių mokymąsi bei užklausų derinimą. Testuoti skirtingas užklausas yra būtina norint rasti geriausią sprendimą jūsų specifiniam atvejui.

Kurdami programas, dirbsite su skirtingų tipų užklausomis:
- **Sistemos užklausos**: Nustato bazinius modelio elgesio taisykles ir kontekstą
- **Vartotojo užklausos**: Įvesties duomenys iš jūsų programos naudotojų
- **Asistentų užklausos**: Modelio atsakymai, paremti sisteminėmis ir vartotojo užklausomis

> **Sužinokite daugiau**: Sužinokite daugiau apie užklausų inžineriją [Generatyvinio DI pradedantiesiems kurso Užklausų inžinerijos skyriuje](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### Žetonai, įterpimai ir agentai

Dirbdami su generatyviniais DI modeliais, susidursite su terminais kaip **žetonai**, **įterpimai**, **agentai** ir **Modelio Konteksto Protokolas (MCP)**. Štai detali šių konceptų apžvalga:

- **Žetonai**: Žetonai yra mažiausios teksto dalys modelyje. Tai gali būti žodžiai, simboliai ar žodžių dalys. Žetonai naudojami tekstui atvaizduoti formatu, kurį modelis gali suprasti. Pavyzdžiui, sakinys „The quick brown fox jumped over the lazy dog“ gali būti suskaidytas į žetonus kaip ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] arba ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"], priklausomai nuo žetonizacijos strategijos.

![Figura: Generatyvinio DI žetonų pavyzdys žodžių suskaidymui į žetonus](../../../translated_images/lt/tokens.6283ed277a2ffff4.webp)

Žetonizacija yra tekstų suskaidymo į šiuos mažesnius vienetus procesas. Tai labai svarbu, nes modeliai veikia su žetonais, o ne žaliu tekstu. Žetonų skaičius užklausoje turi įtakos modelio atsakymo ilgiui ir kokybei, nes modeliai turi žetonų limitus savo konteksto lange (pvz., 128K žetonų GPT-4o bendram kontekstui, įskaitant tiek įvestį, tiek išvestį).

  Java kalba galite naudoti tokias bibliotekas kaip OpenAI SDK žetonizacijai automatizuoti siunčiant užklausas DI modeliams.

- **Įterpimai**: Įterpimai yra vektoriniai žetonų atvaizdai, kurie užfiksuoja semantinę prasmę. Tai skaitinės reprezentacijos (dažniausiai plaukiojančio kablelio skaičių masyvai), kurios leidžia modeliams suprasti žodžių santykius ir generuoti kontekstualiai aktualius atsakymus. Panašūs žodžiai turi panašius įterpimus, leidžiančius modeliui suprasti sinonimus ir semantinius ryšius.

![Figura: Įterpimai](../../../translated_images/lt/embedding.398e50802c0037f9.webp)

  Java kalboje galite generuoti įterpimus naudodami OpenAI SDK ar kitas įterpimų generavimą palaikančias bibliotekas. Šie įterpimai yra svarbūs užduotims, tokioms kaip semantinės paieškos, kai norite rasti panašų turinį pagal prasmę, o ne pagal tikslius teksto atitikmenis.

- **Vektorinės duomenų bazės**: Vektorinės duomenų bazės yra specializuotos saugyklos, optimizuotos įterpimams. Jos leidžia efektyviai ieškoti panašumų ir yra svarbios Retrieval-Augmented Generation (RAG) modeliams, kai reikia rasti aktualią informaciją dideliuose duomenų rinkiniuose pagal semantinį panašumą, o ne tikslius atitikmenis.

![Figura: Vektorinės duomenų bazės architektūra, rodanti, kaip įterpimai saugomi ir paimami panašumų paieškai.](../../../translated_images/lt/vector.f12f114934e223df.webp)

> **Pastaba**: Šiame kurse neapžvelgsime vektorinių duomenų bazių, bet paminime jas, nes jos dažnai naudojamos realiose programose.

- **Agentai ir MCP**: DI komponentai, kurie autonomiškai bendrauja su modeliais, įrankiais ir išorinėmis sistemomis. Modelio Konteksto Protokolas (MCP) suteikia standartizuotą būdą agentams saugiai naudotis išoriniais duomenų šaltiniais ir įrankiais. Sužinokite daugiau mūsų [MCP pradedantiesiems](https://github.com/microsoft/mcp-for-beginners) kurse.

Java DI programose naudosite žetonus teksto apdorojimui, įterpimus semantinei paieškai ir RAG, vektorines duomenų bazes duomenų gavimui bei agentus su MCP kuriant išmanias sistemas, naudojančias įrankius.

![Figura: kaip užklausa tampa atsakymu – žetonai, vektoriai, papildoma RAG paieška, LLM apdorojimas ir MCP agentas viename greitame sraute.](../../../translated_images/lt/flow.f4ef62c3052d12a8.webp)

### DI Kūrimo Įrankiai ir Bibliotekos Java kalbai

Java siūlo puikius įrankius DI kūrimui. Yra trys pagrindinės bibliotekos, kurias nagrinėsime viso kurso metu – OpenAI Java SDK, Azure OpenAI SDK ir Spring AI.

Čia pateiktame greitojo orientavimo lentelėje parodyta, kuris SDK naudojamas kiekvieno skyriaus pavyzdžiuose:

| Skyrius | Pavyzdys | SDK |
|---------|--------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**SDK Dokumentacijos Nuorodos:**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

OpenAI SDK yra oficiali Java biblioteka OpenAI API. Ji suteikia paprastą ir nuoseklią sąsają darbui su OpenAI modeliais, leidžianti lengvai integruoti DI galimybes į Java programas. 4-ojo skyriaus „Pet Story“ programa ir „Foundry Local“ pavyzdys demonstruoja OpenAI SDK naudojimą kartu su Azure AI Foundry.

#### Spring AI

Spring AI yra išsamus karkasas, kuris suteikia DI galimybes Spring programoms, suteikdamas nuoseklų abstrakcijos sluoksnį skirtingų DI tiekėjų integracijai. Jis sklandžiai integruojasi į Spring ekosistemą, todėl yra idealus pasirinkimas įmonių Java programoms, kurioms reikia DI funkcijų.

Spring AI stiprybė yra sklandi integracija į Spring ekosistemą, leidžianti lengvai kurti gamybinio lygio DI programas, naudojant pažįstamus Spring modelius, tokius kaip priklausomybių injekcija, konfigūracijos valdymas ir testavimo karkasai. Spring AI naudosite 2 ir 4 skyriuose kurdami programas, kurios naudoja tiek OpenAI, tiek Modelio Konteksto Protokolo (MCP) Spring AI bibliotekas.

##### Modelio Konteksto Protokolas (MCP)

[Modelio Konteksto Protokolas (MCP)](https://modelcontextprotocol.io/) yra kylančios standartas, leidžiantis DI programoms saugiai bendrauti su išoriniais duomenų šaltiniais ir įrankiais. MCP suteikia standartizuotą būdą DI modeliams pasiekti kontekstinę informaciją ir vykdyti veiksmus jūsų programose.

4-ajame skyriuje sukursite paprastą MCP skaičiuoklės paslaugą, kuri parodys Modelio Konteksto Protokolo pagrindus su Spring AI, demonstruodama, kaip kurti bazines įrankių integracijas ir paslaugų architektūras.

#### Azure OpenAI Java SDK

Azure OpenAI klientų biblioteka Java kalbai yra OpenAI REST API adaptacija, teikianti idiomatišką sąsają ir integraciją su likusia Azure SDK ekosistema. 3-ajame skyriuje kursite programas naudodami Azure OpenAI SDK, įskaitant pokalbių programas, funkcijų kvietimą ir RAG (Retrieval-Augmented Generation) modelius.

> Pastaba: Azure OpenAI SDK funkcionalumu atsilieka nuo OpenAI Java SDK, todėl būsimuose projektuose rekomenduojama naudoti OpenAI Java SDK.

## Santrauka

Čia baigiame pagrindus! Dabar suprantate:

- Pagrindines generatyvinio DI sąvokas – nuo LLM ir užklausų inžinerijos iki žetonų, įterpimų ir vektorinių duomenų bazių
- Jūsų Java DI kūrimo įrankių pasirinkimus: Azure OpenAI SDK, Spring AI ir OpenAI Java SDK
- Kas yra Modelio Konteksto Protokolas ir kaip jis leidžia DI agentams dirbti su išoriniais įrankiais

## Kiti Žingsniai

[2 skyrius: Kūrimo Aplinkos Paruošimas](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Atsakomybės apribojimas**:
Šis dokumentas buvo išverstas naudojant dirbtinio intelekto vertimo paslaugą [Co-op Translator](https://github.com/Azure/co-op-translator). Nors siekiame tikslumo, prašome atkreipti dėmesį, kad automatiniai vertimai gali turėti klaidų ar netikslumų. Originalus dokumentas jo gimtąja kalba laikomas autoritetingu šaltiniu. Svarbiai informacijai rekomenduojama naudoti profesionalų žmogiškąjį vertimą. Mes neatsakome už jokius nesusipratimus ar neteisingą interpretaciją, kilusią naudojantis šiuo vertimu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->