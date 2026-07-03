# Johdanto generatiiviseen tekoälyyn – Java-versio

## Mitä opit

- **Generatiivisen tekoälyn perusteet**, mukaan lukien LLM-mallit, prompt-tekniikka, tokenit, upotukset ja vektoritietokannat
- **Vertaa Java-tekoälykehityksen työkaluja** kuten Azure OpenAI SDK, Spring AI ja OpenAI Java SDK
- **Tutustu Model Context Protocoliin** ja sen rooliin tekoälyagenttien viestinnässä

## Sisällysluettelo

- [Johdanto](#johdanto)
- [Nopea kertaus generatiivisen tekoälyn käsitteistä](#nopea-kertaus-generatiivisen-tekoälyn-käsitteistä)
- [Prompt-tekniikan tarkastelu](#prompt-tekniikan-tarkastelu)
- [Tokenit, upotukset ja agentit](#tokenit-upotukset-ja-agentit)
- [Tekoälykehityksen työkalut ja kirjastot Javalle](#tekoälykehityksen-työkalut-ja-kirjastot-javalle)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [Yhteenveto](#yhteenveto)
- [Seuraavat askeleet](#seuraavat-askellet)

## Johdanto

Tervetuloa Generatiivinen tekoäly aloittelijoille – Java-versio -kirjan ensimmäiseen lukuun! Tämä peruskurssi esittelee sinulle generatiivisen tekoälyn keskeiset käsitteet sekä miten työskennellä niiden kanssa Javalla. Opit tekoälysovellusten olennaiset rakennuspalikat, kuten suuret kielimallit (LLM), tokenit, upotukset ja tekoälyagentit. Tutustumme myös tärkeimpiin Java-työkaluihin, joita käytät kurssin aikana.

### Nopea kertaus generatiivisen tekoälyn käsitteistä

Generatiivinen tekoäly on tekoälyn muoto, joka luo uutta sisältöä, kuten tekstiä, kuvia tai koodia, oppimiensa kuvioiden ja suhteiden perusteella. Generatiiviset tekoälymallit voivat tuottaa ihmismäisiä vastauksia, ymmärtää kontekstia ja joskus jopa luoda sisältöä, joka vaikuttaa ihmismäiseltä.

Kun kehität Java-pohjaisia tekoälysovelluksia, työskentelet **generatiivisten tekoälymallien** kanssa sisällön luomisessa. Joitakin generatiivisten mallien kyvykkyyksiä ovat:

- **Tekstinluonti**: Ihmismäisen tekstin luominen chatbotteihin, sisältöön ja tekstin täydentämiseen.
- **Kuvien luonti ja analyysi**: Realististen kuvien tuottaminen, valokuvien parantaminen ja kohteiden tunnistus.
- **Koodin luonti**: Koodinpätkien tai skriptien kirjoittaminen.

Mallityyppejä on erilaisia, jotka on optimoitu eri tehtäviin. Esimerkiksi sekä **pienet kielimallit (SLM)** että **suuret kielimallit (LLM)** voivat käsitellä tekstinluontia, mutta LLM:t tarjoavat yleensä paremman suorituskyvyn monimutkaisemmissa tehtävissä. Kuvatehtäviin käytetään erikoistuneita näkömalleja tai monimuotoisia malleja.

![Kuva: Generatiivisten tekoälymallien tyypit ja käyttötapaukset.](../../../translated_images/fi/llms.225ca2b8a0d34473.webp)

Tietenkään mallien vastaukset eivät ole aina täydellisiä. Olennaista on, että olet ehkä kuullut malleista, jotka "hallusinoivat" eli tuottavat virheellistä tietoa hyvin vakuuttavalla tavalla. Mutta voit auttaa ohjaamaan mallia tuottamaan parempia vastauksia antamalla sille selkeät ohjeet ja kontekstin. Tässä kohtaa **prompt-tekniikka** tulee kuvaan.

#### Prompt-tekniikan tarkastelu

Prompt-tekniikka tarkoittaa tehokkaiden syötteiden suunnittelua, jotta tekoälymallit ohjautuvat haluttuihin tuloksiin. Se sisältää:

- **Selkeys**: Ohjeiden selkeä ja yksiselitteinen muotoilu.
- **Konteksti**: Tarvittavan taustatiedon tarjoaminen.
- **Rajoitteet**: Mahdollisten rajoitusten tai formaattien määrittäminen.

Hyviä käytäntöjä prompt-tekniikassa ovat promptin suunnittelu, selkeät ohjeet, tehtävän pilkkominen osiin, yksi- tai muutaman esimerkin opetus sekä promptin viritys. Erilaisten promptien testaaminen on olennainen osa parhaan tuloksen löytämistä.

Sovelluksia kehitettäessä käytät eri tyyppisiä promtteja:
- **Järjestelmän promtit**: Mallin käyttäytymisen perustason säännöt ja konteksti
- **Käyttäjän promptit**: Sovelluksen käyttäjien syöttämät tiedot
- **Avustajan promptit**: Mallin vastaukset järjestelmän ja käyttäjän promptien perusteella

> **Lue lisää**: Lue lisää prompt-tekniikasta [GenAI for Beginners -kurssin Prompt Engineering -luvussa](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### Tokenit, upotukset ja agentit

Työskennellessäsi generatiivisten tekoälymallien kanssa kohtaat termejä kuten **tokenit**, **upotukset**, **agentit** ja **Model Context Protocol (MCP)**. Tässä yksityiskohtainen yleiskatsaus näihin käsitteisiin:

- **Tokenit**: Tokenit ovat pienin tekstin yksikkö mallissa. Ne voivat olla sanoja, merkkejä tai osasanoja. Tokenit edustavat tekstidataa muodossa, jonka malli ymmärtää. Esimerkiksi lause "The quick brown fox jumped over the lazy dog" saatetaan tokenisoida muotoon ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] tai ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"] tokenisointistrategiasta riippuen.

![Kuva: Esimerkki generatiivisen tekoälyn tokeneista, joissa sanat jaetaan tokeneiksi](../../../translated_images/fi/tokens.6283ed277a2ffff4.webp)

Tokenisointi tarkoittaa tekstin pilkkomista näihin pienempiin yksiköihin. Tämä on tärkeää, koska mallit käsittelevät tokeneita eikä raakatekstiä. Tokenimäärä promptissa vaikuttaa mallin vastauksen pituuteen ja laatuun, sillä malleilla on token-rajoitukset konteksti-ikkunassaan (esim. GPT-4o:n kokonaiskonteksti sisältäen syötteen ja vastauksen on 128K tokenia).

  Javassa voit käyttää kirjastoja kuten OpenAI SDK tokenisoinnin hoitamiseen automaattisesti, kun lähetät pyyntöjä tekoälymalleille.

- **Upotukset**: Upotukset ovat tokenien vektoriesityksiä, jotka kuvaavat semanttista merkitystä. Ne ovat numeerisia esityksiä (yleensä liukulukutaulukoita), joiden avulla mallit ymmärtävät sanojen välisiä suhteita ja luovat kontekstuaalisesti merkityksellisiä vastauksia. Samankaltaisilla sanoilla on samankaltaiset upotukset, jolloin malli voi ymmärtää konsepteja kuten synonyymejä ja semanttisia suhteita.

![Kuva: Upotukset](../../../translated_images/fi/embedding.398e50802c0037f9.webp)

  Javassa voit luoda upotuksia käyttämällä OpenAI SDK:ta tai muita upotusten generointia tukevia kirjastoja. Upotukset ovat olennaisia tehtävissä kuten semanttinen haku, jossa haluat löytää merkitykseltään samankaltaista sisältöä tarkkojen tekstiosumien sijaan.

- **Vektoritietokannat**: Vektoritietokannat ovat erityisesti upotuksia varten optimoituja tallennusjärjestelmiä. Ne mahdollistavat tehokkaan samankaltaisten sisältöjen haun ja ovat tärkeitä Retrieval-Augmented Generation (RAG) -mallin toteutuksessa, jossa etsitään merkitykseltään oleellista tietoa suurista tietojoukoista tarkkojen tekstiosumien sijaan.

![Kuva: Vektoritietokannan arkkitehtuuri, joka näyttää kuinka upotukset tallennetaan ja haetaan samankaltaishakuun.](../../../translated_images/fi/vector.f12f114934e223df.webp)

> **Huom:** Tässä kurssissa emme kata vektoritietokantoja, mutta mainitsemme ne, koska niitä käytetään yleisesti käytännön sovelluksissa.

- **Agentit & MCP**: Tekoälykomponentit, jotka itsenäisesti vuorovaikuttavat mallien, työkalujen ja ulkoisten järjestelmien kanssa. Model Context Protocol (MCP) tarjoaa standardoidun tavan agenttien turvalliseen pääsyyn ulkoisiin datalähteisiin ja työkaluihin. Lue lisää [MCP for Beginners](https://github.com/microsoft/mcp-for-beginners) -kurssiltamme.

Java-pohjaisissa tekoälysovelluksissa käytät tokeneita tekstinkäsittelyyn, upotuksia semanttiseen hakuun ja RAGiin, vektoritietokantoja tiedonhakuun sekä agentteja MCP:n avulla rakentamaan älykkäitä, työkaluja hyödyntäviä järjestelmiä.

![Kuva: Kuinka prompti muuttuu vastaukseksi—tokenit, vektorit, valinnainen RAG-haku, LLM:n ajattelu ja MCP-agentti yhdellä nopealla prosessilla.](../../../translated_images/fi/flow.f4ef62c3052d12a8.webp)

### Tekoälykehityksen työkalut ja kirjastot Javalle

Java tarjoaa erinomaiset työkalut tekoälykehitykseen. Tässä kurssissa tutustumme kolmeen pääkirjastoon – OpenAI Java SDK, Azure OpenAI SDK ja Spring AI.

Tässä pikainen referenssitaulukko, joka näyttää minkä SDK:n jokaisen luvun esimerkeissä käytetään:

| Luku | Esimerkki | SDK |
|---------|--------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**SDK-dokumentaatiolinkit:**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

OpenAI SDK on virallinen Java-kirjasto OpenAI-API:lle. Se tarjoaa yksinkertaisen ja yhtenäisen rajapinnan OpenAI:n mallien käyttöön, jolloin tekoälyominaisuuksien integroiminen Java-sovelluksiin on helppoa. Luvun 4 Pet Story -sovellus ja Foundry Local -esimerkki demonstroivat OpenAI SDK:n käyttöä Azure AI Foundry -ympäristössä.

#### Spring AI

Spring AI on kattava kehys, joka tuo tekoälyominaisuudet Spring-sovelluksiin tarjoten yhtenäisen abstraktiokerroksen eri tekoälypalveluiden välillä. Se integroituu saumattomasti Spring-ekosysteemiin, joten se on ihanteellinen valinta yritystason Java-sovelluksiin, jotka tarvitsevat tekoälykyvykkyyksiä.

Spring AI:n vahvuus on sen saumattomassa integroinnissa Spring-ekosysteemiin, jolloin tuotantovalmiiden tekoälysovellusten rakentaminen tutuin Spring-kuvioin kuten riippuvuuksien injektointi, konfiguraation hallinta ja testaustyökalut on helppoa. Käytät Spring AI:ta luvuissa 2 ja 4 rakentaaksesi sovelluksia, jotka hyödyntävät sekä OpenAI:ta että Model Context Protocolin (MCP) Spring AI -kirjastoja.

##### Model Context Protocol (MCP)

[Model Context Protocol (MCP)](https://modelcontextprotocol.io/) on nouseva standardi, joka mahdollistaa tekoälysovellusten turvallisen vuorovaikutuksen ulkoisten tietolähteiden ja työkalujen kanssa. MCP tarjoaa standardoidun tavan tekoälymallien pääsyyn kontekstuaaliseen tietoon ja toimintojen suorittamiseen sovelluksissasi.

Luvussa 4 rakennat yksinkertaisen MCP-laskinpalvelun, joka demonstroi Model Context Protocolin perusteita Spring AI:n avulla näyttäen, miten tehdä perusintegraatioita työkaluihin ja palveluarkkitehtuureihin.

#### Azure OpenAI Java SDK

Azure OpenAI -asiakas kirjasto Javalle on OpenAI:n REST-rajapintojen adaptaatio, joka tarjoaa idiomaattisen rajapinnan ja integroinnin osaksi Azure SDK -ekosysteemiä. Luvussa 3 rakennat sovelluksia käyttäen Azure OpenAI SDK:ta, mukaan lukien chat-sovelluksia, funktiokutsuja ja RAG-malleja (Retrieval-Augmented Generation).

> Huom: Azure OpenAI SDK jää jäljessä ominaisuuksissa OpenAI Java SDK:sta, joten tulevaisuuden projekteissa kannattaa harkita OpenAI Java SDK:n käyttöä.

## Yhteenveto

Näin pääset perustan yli! Ymmärrät nyt:

- Generatiivisen tekoälyn ydinkäsitteet – LLM:stä ja prompt-tekniikasta tokeneihin, upotuksiin ja vektoritietokantoihin
- Java-tekoälykehityksen työkaluvaihtoehdot: Azure OpenAI SDK, Spring AI ja OpenAI Java SDK
- Mikä Model Context Protocol on ja miten se mahdollistaa tekoälyagenttien toiminnan ulkoisten työkalujen kanssa

## Seuraavat askeleet

[Luku 2: Kehitysympäristön pystyttäminen](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Vastuuvapauslauseke**:
Tämä asiakirja on käännetty käyttämällä tekoälypohjaista käännöspalvelua [Co-op Translator](https://github.com/Azure/co-op-translator). Vaikka pyrimme tarkkuuteen, otathan huomioon, että automaattiset käännökset saattavat sisältää virheitä tai epätarkkuuksia. Alkuperäinen asiakirja sen alkuperäiskielellä on virallinen lähde. Tärkeissä asioissa suositellaan ammattimaista ihmiskäännöstä. Emme ole vastuussa tämän käännöksen käytöstä aiheutuvista väärinymmärryksistä tai tulkinnoista.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->