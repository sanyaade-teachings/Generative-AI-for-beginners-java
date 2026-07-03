# Utangulizi wa AI Inayozalisha - Toleo la Java

## Utakachojifunza

- **Misingi ya AI Inayozalisha** ikiwa ni pamoja na LLMs, uhandisi wa maelekezo, tokeni, embeddings, na hifadhidata za vekta
- **Kulinganisha zana za maendeleo ya AI za Java** ikiwa ni pamoja na Azure OpenAI SDK, Spring AI, na OpenAI Java SDK
- **Gundua Itifaki ya Muktadha wa Mfano** na jukumu lake katika mawasiliano ya mawakala wa AI

## Jedwali la Yaliyomo

- [Utangulizi](#utangulizi)
- [Kumbukumbu ya dhahiri ya dhana za AI Inayozalisha](#kumbukumbu-ya-dhahiri-ya-dhana-za-ai-inayozalisha)
- [Mapitio ya uhandisi wa maelekezo](#mapitio-ya-uhandisi-wa-maelekezo)
- [Tokeni, embeddings, na mawakala](#tokeni-embeddings-na-mawakala)
- [Zana za Maendeleo ya AI na Maktaba za Java](#zana-za-maendeleo-ya-ai-na-maktaba-za-java)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [Muhtasari](#muhtasari)
- [Hatua Zifuatazo](#hatua-zifuatazo)

## Utangulizi

Karibu katika sura ya kwanza ya AI Inayozalisha kwa Waanzilishi - Toleo la Java! Somo hili la msingi linakuanzisha kwa dhana za msingi za AI inayozalisha na jinsi ya kufanya kazi nazo kwa kutumia Java. Utajifunza kuhusu vipengele muhimu vya maombi ya AI, ikiwa ni pamoja na Modeli Kubwa za Lugha (LLMs), tokeni, embeddings, na mawakala wa AI. Pia tutachunguza zana kuu za Java utakazotumia katika kozi hii yote.

### Kumbukumbu ya dhahiri ya dhana za AI Inayozalisha

AI inayozalisha ni aina ya akili bandia inayounda maudhui mapya, kama vile maandishi, picha, au msimbo, kwa kuzingatia mifumo na uhusiano uliojifunza kutoka kwa data. Modeli za AI zinazozalisha zinaweza kutoa majibu yanayofanana na ya binadamu, kuelewa muktadha, na wakati mwingine hata kuunda maudhui yanayoonekana kuwa ya kibinadamu.

Unapoendeleza maombi yako ya AI kwa Java, utatumia **modeli za AI zinazozalisha** kuunda maudhui. Baadhi ya uwezo wa modeli hizi ni pamoja na:

- **Uundaji wa Maandishi**: Kuandika maandishi yanayofanana na ya kibinadamu kwa roboti za mazungumzo, maudhui, na kukamilisha maandishi.
- **Uundaji na Uchambuzi wa Picha**: Kutengeneza picha halisi, kuboresha picha, na kugundua vitu.
- **Uundaji wa Msimbo**: Kuandika vipande vya msimbo au maandishi ya programu.

Kuna aina maalum za modeli zilizoboreshwa kwa kazi mbalimbali. Kwa mfano, **Modeli Ndogo za Lugha (SLMs)** na **Modeli Kubwa za Lugha (LLMs)** zote zinaweza kushughulikia uundaji wa maandishi, huku LLMs zikitoa utendaji bora zaidi kwa kazi ngumu. Kwa kazi zinazohusiana na picha, utatumia modeli maalum za kuona au modeli za multimodal.

![Figure: Generative AI model types and use cases.](../../../translated_images/sw/llms.225ca2b8a0d34473.webp)

Bila shaka, majibu kutoka kwa modeli hizi si kamilifu kila wakati. Pengine umesikia kuhusu modeli "kufikirika" au kutoa habari zisizo sahihi kwa mtindo wa mwenye mamlaka. Lakini unaweza kusaidia kuelekeza modeli kutoa majibu bora kwa kuwapa maelekezo wazi na muktadha. Hapa ndipo **uhandisi wa maelekezo** unapoingilia.

#### Mapitio ya uhandisi wa maelekezo

Uhandisi wa maelekezo ni zoezi la kubuni pembejeo madhubuti ili kuelekeza modeli za AI kuelekea matokeo yanayotakiwa. Inahusisha:

- **Uwazi**: Kufanya maelekezo yawe wazi na yasiyo na utata.
- **Muktadha**: Kutoa taarifa muhimu za msingi.
- **Vikwazo**: Kueleza vizingiti au muundo wowote.

Mazingira bora ya uhandisi wa maelekezo ni pamoja na muundo wa maelekezo, maelekezo wazi, kugawanya kazi, kujifunza kwa mara moja au mara chache, na urekebishaji wa maelekezo. Kupima maelekezo tofauti ni muhimu kupata kinachofaa kwa matumizi yako maalum.

Unapoendelea kutengeneza maombi, utatumia aina tofauti za maelekezo:
- **Maelekezo ya mfumo**: Kuanzisha kanuni msingi na muktadha wa tabia ya modeli
- **Maelekezo ya mtumiaji**: Data ya pembejeo kutoka kwa watumiaji wa programu yako
- **Maelekezo ya msaidizi**: Majibu ya modeli yanayotokana na maelekezo ya mfumo na mtumiaji

> **Jifunze zaidi**: Jifunze zaidi kuhusu uhandisi wa maelekezo katika [Sura ya Uhandisi wa Maelekezo ya kozi ya GenAI kwa Waanzilishi](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### Tokeni, embeddings, na mawakala

Unapofanya kazi na modeli za AI zinazozalisha, utakumbana na maneno kama **tokeni**, **embeddings**, **mawakala**, na **Itifaki ya Muktadha wa Mfano (MCP)**. Hapa kuna muhtasari wa dhana hizi:

- **Tokeni**: Tokeni ni kipengee kidogo kabisa cha maandishi katika modeli. Zinaweza kuwa maneno, herufi, au sehemu za maneno. Tokeni hutumika kuwakilisha data ya maandishi katika muundo unaoweza kueleweka na modeli. Kwa mfano, sentensi "The quick brown fox jumped over the lazy dog" inaweza kugawanywa tokeni kama ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] au ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"] kulingana na mbinu ya kugawanya tokeni.

![Figure: Generative AI tokens example of breaking words into tokens](../../../translated_images/sw/tokens.6283ed277a2ffff4.webp)

Ugawaji tokeni ni mchakato wa kugawanya maandishi kuwa sehemu hizi ndogo. Hii ni muhimu kwa sababu modeli hufanya kazi kwa tokeni badala ya maandishi ghafi. Idadi ya tokeni katika maelekezo huathiri urefu na ubora wa jibu la modeli, kwani modeli zina vizingiti vya tokeni kwa dirisha lao la muktadha (kwa mfano, tokeni 128K kwa GPT-4o kwa jumla ya muktadha, ikiwa ni pamoja na pembejeo na matokeo).

  Katika Java, unaweza kutumia maktaba kama OpenAI SDK kushughulikia ugawaji tokeni moja kwa moja wakati wa kutuma maombi kwa modeli za AI.

- **Embeddings**: Embeddings ni uwakilishi wa vekta wa tokeni unaochukua maana ya kimuktadha. Ni uwakilishi wa takwimu (kawaida ni safu za nambari za desimali) unaowawezesha modeli kuelewa uhusiano kati ya maneno na kutoa majibu yanayofaa kwa muktadha. Maneno yanayofanana yana embeddings zinazofanana, kuruhusu modeli kuelewa dhana kama lugha ya maana na uhusiano wa kimantiki.

![Figure: Embeddings](../../../translated_images/sw/embedding.398e50802c0037f9.webp)

  Katika Java, unaweza kuzalisha embeddings kwa kutumia OpenAI SDK au maktaba nyingine zinazounga mkono uzalishaji wa embeddings. Embeddings hizi ni muhimu kwa kazi kama utafutaji wa maana, ambapo unataka kutafuta maudhui yanayofanana kulingana na maana badala ya kulinganisha maandishi kabisa.

- **Hifadhidata za vekta**: Hifadhidata za vekta ni mifumo maalum ya kuhifadhi iliyoboreshwa kwa embeddings. Huwezesha utafutaji wa ufanisi wa ulinganifu na ni muhimu kwa mifumo ya Retrieval-Augmented Generation (RAG) ambapo unahitaji kupata taarifa zinazohusiana kutoka kwa data kubwa kwa kuzingatia ulinganifu wa maana badala ya kulinganisha halisi.

![Figure: Vector database architecture showing how embeddings are stored and retrieved for similarity search.](../../../translated_images/sw/vector.f12f114934e223df.webp)

> **Kumbuka**: Katika kozi hii, hatutashughulikia hifadhidata za vekta lakini tunadhani ni vyema kutaja kwa kuwa hutumika sana katika maombi halisi.

- **Mawakala & MCP**: Vipengele vya AI vinavyojiendesha vinavyoshirikiana na modeli, zana, na mifumo ya nje. Itifaki ya Muktadha wa Mfano (MCP) hutoa njia ya kawaida kwa mawakala kupata kwa usalama vyanzo vya data na zana za nje. Jifunze zaidi katika kozi yetu ya [MCP kwa Waanzilishi](https://github.com/microsoft/mcp-for-beginners).

Katika maombi ya AI ya Java, utatumia tokeni kwa usindikaji wa maandishi, embeddings kwa utafutaji wa maana na RAG, hifadhidata za vekta kwa urejeshaji wa data, na mawakala na MCP kwa kujenga mifumo yenye akili inayotumia zana.

![Figure: how a prompt becomes a reply—tokens, vectors, optional RAG lookup, LLM thinking, and an MCP agent all in one quick flow..](../../../translated_images/sw/flow.f4ef62c3052d12a8.webp)

### Zana za Maendeleo ya AI na Maktaba za Java

Java inatoa zana bora za maendeleo ya AI. Kuna maktaba kuu tatu tutazozichunguza katika kozi hii yote - OpenAI Java SDK, Azure OpenAI SDK, na Spring AI.

Hapa ni jedwali la rejea la haraka linaonyesha SDK ipi inatumika katika mifano ya kila sura:

| Sura  | Mfano        | SDK                      |
|--------|--------------|--------------------------|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI  |
| 03-CoreGenerativeAITechniques | examples       | Azure OpenAI SDK       |
| 04-PracticalSamples | petstory      | OpenAI Java SDK        |
| 04-PracticalSamples | foundrylocal  | OpenAI Java SDK        |
| 04-PracticalSamples | calculator    | Spring AI MCP SDK + LangChain4j |

**Viungo vya Nyaraka za SDK:**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

OpenAI SDK ni maktaba rasmi ya Java kwa API ya OpenAI. Inatoa interface rahisi na thabiti ya kuingiliana na modeli za OpenAI, ikifanya iwe rahisi kuingiza uwezo wa AI katika maombi ya Java. Programu ya Sura ya 4 ya Hadithi ya Mnyama na mfano wa Foundry Local inaonyesha njia ya OpenAI SDK kwa Azure AI Foundry.

#### Spring AI

Spring AI ni mfumo kamili unaobeba uwezo wa AI kwa maombi ya Spring, ukitoa safu ya kawaida ya utoaji ufanisi kati ya watoa huduma tofauti za AI. Inaunganishwa vizuri na mfumo wa Spring, ikifanya iwe chaguo bora kwa maombi ya Java ya biashara yanayohitaji uwezo wa AI.

Nguvu ya Spring AI iko katika muunganisho wake wa kutatanisha na mfumo wa Spring, ikifanya rahisi kujenga maombi ya AI ya uzalishaji kwa kutumia mifumo ya kawaida ya Spring kama kuingiza utegemezi, usimamizi wa usanidi, na mifumo ya majaribio. Utatumia Spring AI katika Sura 2 na 4 kujenga maombi yanayotumia OpenAI pamoja na Maktaba za Model Context Protocol (MCP) za Spring AI.

##### Itifaki ya Muktadha wa Mfano (MCP)

[Itifaki ya Muktadha wa Mfano (MCP)](https://modelcontextprotocol.io/) ni kiwango kinachoibukia kinachowawezesha maombi ya AI kuingiliana kwa usalama na vyanzo vya data na zana za nje. MCP hutoa njia iliyosanifishwa kwa modeli za AI kupata taarifa za muktadha na kutekeleza vitendo katika maombi yako.

Katika Sura ya 4, utajenga huduma rahisi ya mkokoteni wa MCP inayoonyesha misingi ya Itifaki ya Muktadha wa Mfano na Spring AI, ikionyesha jinsi ya kuunda ushirikiano wa zana za msingi na usanifu wa huduma.

#### Azure OpenAI Java SDK

Maktaba ya mteja ya Azure OpenAI kwa Java ni muunganisho wa API za REST za OpenAI unaotoa interface inayozingatia dafuni pamoja na muingiliano na mfumo wa Azure SDK yote. Katika Sura ya 3, utajenga maombi ukiwa unatumia Azure OpenAI SDK, ikiwa ni pamoja na maombi ya mazungumzo, simu za kazi, na mifumo ya RAG (Retrieval-Augmented Generation).

> Kumbuka: Azure OpenAI SDK inapungua nyuma ya OpenAI Java SDK katika vipengele hivyo kwa miradi ya baadaye, fikiria kutumia OpenAI Java SDK.

## Muhtasari

Hii inahitimisha misingi! Sasa unaelewa:

- Dhana za msingi za AI inayozalisha - kutoka LLMs na uhandisi wa maelekezo hadi tokeni, embeddings, na hifadhidata za vekta
- Chaguzi zako za zana kwa maendeleo ya AI ya Java: Azure OpenAI SDK, Spring AI, na OpenAI Java SDK
- Ni nini Itifaki ya Muktadha wa Mfano na jinsi inavyowezesha mawakala wa AI kufanya kazi na zana za nje

## Hatua Zifuatazo

[Sura ya 2: Kusanidi Mazingira ya Maendeleo](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Kionyozo**:
Hati hii imetafsiriwa kwa kutumia huduma ya tafsiri ya AI [Co-op Translator](https://github.com/Azure/co-op-translator). Ingawa tunajitahidi kupata usahihi, tafadhali fahamu kwamba tafsiri za kiotomatiki zinaweza kuwa na makosa au upungufu wa usahihi. Hati ya asili katika lugha yake halisi inapaswa kuchukuliwa kama chanzo cha mamlaka. Kwa taarifa muhimu, tafsiri ya kitaalamu inayofanywa na binadamu inapendekezwa. Hatutojibu kwa kuelewa vibaya au tafsiri potofu zinazotokea kutokana na matumizi ya tafsiri hii.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->