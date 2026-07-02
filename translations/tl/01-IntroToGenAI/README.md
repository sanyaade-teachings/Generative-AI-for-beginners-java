# Panimula sa Generative AI - Edisyon ng Java

## Ano ang Matututunan Mo

- **Mga pundasyon ng Generative AI** kabilang ang LLMs, prompt engineering, tokens, embeddings, at vector databases
- **Paghahambing ng mga kasangkapan sa pagbuo ng Java AI** kabilang ang Azure OpenAI SDK, Spring AI, at OpenAI Java SDK
- **Tuklasin ang Model Context Protocol** at ang papel nito sa komunikasyon ng AI agent

## Talaan ng Nilalaman

- [Panimula](#panimula)
- [Isang mabilis na refresher sa mga konsepto ng Generative AI](#isang-mabilis-na-refresher-sa-mga-konsepto-ng-generative-ai)
- [Review sa prompt engineering](#review-sa-prompt-engineering)
- [Mga tokens, embeddings, at agents](#mga-tokens-embeddings-at-agents)
- [Mga Kasangkapan at Library ng AI Development para sa Java](#mga-kasangkapan-at-library-ng-ai-development-para-sa-java)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [Buod](#buod)
- [Mga Susunod na Hakbang](#mga-susunod-na-hakbang)

## Panimula

Maligayang pagdating sa unang kabanata ng Generative AI para sa Mga Nagsisimula - Edisyon ng Java! Ipinapakilala sa lekser na ito ang mga pangunahing konsepto ng generative AI at kung paano ito gamitin gamit ang Java. Malalaman mo ang mahahalagang bahagi ng mga AI application, kabilang ang Large Language Models (LLMs), tokens, embeddings, at AI agents. Tatalakayin din natin ang pangunahing mga kasangkapan sa Java na gagamitin mo sa buong kurso.

### Isang mabilis na refresher sa mga konsepto ng Generative AI

Ang Generative AI ay isang uri ng artipisyal na intelihensiya na lumilikha ng bagong nilalaman, tulad ng teksto, mga imahe, o code, base sa mga pattern at ugnayan na natutunan mula sa datos. Kayang bumuo ng mga generative AI models ng mga tugon na halos katulad ng tao, maunawaan ang konteksto, at minsan pa ay lumikha ng nilalaman na tila gawa ng tao.

Habang bumubuo ka ng iyong mga Java AI application, gagamit ka ng **generative AI models** upang gumawa ng nilalaman. Ilan sa mga kakayahan ng generative AI models ay ang:

- **Pagbuo ng Teksto**: Paggawa ng human-like na teksto para sa mga chatbot, nilalaman, at text completion.
- **Pagbuo at Pagsusuri ng Imahe**: Gumawa ng makatotohanang mga imahe, pagbutihin ang mga larawan, at matukoy ang mga bagay.
- **Pagbuo ng Code**: Pagsulat ng mga code snippet o script.

May mga partikular na uri ng modelo na na-optimize para sa iba't ibang gawain. Halimbawa, parehong kaya ng **Small Language Models (SLMs)** at **Large Language Models (LLMs)** ang pagbuo ng teksto, ngunit karaniwang nagbibigay ng mas mahusay na performance ang LLMs para sa mas kumplikadong gawain. Para sa mga gawain na may kinalaman sa imahe, gagamit ka ng mga espesyal na vision models o multi-modal models.

![Figure: Mga uri at gamit ng generative AI models.](../../../translated_images/tl/llms.225ca2b8a0d34473.webp)

Siyempre, hindi palaging perpekto ang mga tugon mula sa mga modelong ito. Marahil ay narinig mo na ang tungkol sa mga modelong "hallucinating" o naglalabas ng maling impormasyon nang may awtoridad. Ngunit maaari mong gabayan ang modelo upang makabuo ng mas mahusay na mga tugon sa pamamagitan ng pagbibigay ng malinaw na mga utos at konteksto. Dito pumapasok ang **prompt engineering**.

#### Review sa prompt engineering

Ang prompt engineering ay ang pagsasanay ng pagdidisenyo ng epektibong mga input upang gabayan ang mga AI model papunta sa nais na output. Kasama dito ang:

- **Kalinawan**: Gumawa ng mga tagubilin na malinaw at walang kalabuan.
- **Konteksto**: Magbigay ng kinakailangang background na impormasyon.
- **Mga Limitasyon**: Tukuyin ang anumang mga limitasyon o format.

Ilan sa mga best practice para sa prompt engineering ay ang disenyo ng prompt, malinaw na mga tagubilin, paghahati-hati ng gawain, one-shot at few-shot learning, at prompt tuning. Mahalaga ang pagsusubok ng iba’t ibang prompt upang mahanap kung ano ang pinakamahusay para sa iyong partikular na kaso.

Sa pagbuo ng mga application, gagamit ka ng iba't ibang uri ng prompt:
- **System prompts**: Nagse-set ng mga batayang patakaran at konteksto para sa kilos ng modelo
- **User prompts**: Ang input data mula sa iyong mga user ng application
- **Assistant prompts**: Mga tugon ng modelo base sa system at user prompts

> **Matuto pa**: Matuto pa tungkol sa prompt engineering sa [Prompt Engineering chapter ng GenAI for Beginners course](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### Mga tokens, embeddings, at agents

Kapag nagtatrabaho sa mga generative AI model, makakaharap ka sa mga terminong katulad ng **tokens**, **embeddings**, **agents**, at **Model Context Protocol (MCP)**. Narito ang detalyadong pagpapaliwanag ng mga konseptong ito:

- **Tokens**: Ang mga token ang pinakamaliit na yunit ng teksto sa isang modelo. Maaari silang mga salita, karakter, o bahagi ng salita. Ginagamit ang tokens para irepresenta ang data ng teksto sa format na mauunawaan ng modelo. Halimbawa, ang pangungusap na "The quick brown fox jumped over the lazy dog" ay maaaring hatiin bilang ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] o ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"] depende sa estratehiya ng tokenization.

![Figure: Halimbawa ng paghiwa-hiwalay ng salita sa mga tokens](../../../translated_images/tl/tokens.6283ed277a2ffff4.webp)

Ang tokenization ay ang proseso ng paghahati ng teksto sa mga maliliit na yunit na ito. Mahalaga ito dahil gumagana ang mga modelo sa tokens imbes na raw text. Ang bilang ng tokens sa isang prompt ay nakakaapekto sa haba at kalidad ng tugon ng modelo, dahil may mga hangganan ng token ang mga modelo para sa kanilang context window (hal. 128K tokens para sa kabuuang konteksto ng GPT-4o, kasama ang input at output).

  Sa Java, maaari mong gamitin ang mga library tulad ng OpenAI SDK para awtomatikong pamahalaan ang tokenization kapag nagpapadala ng mga request sa mga AI model.

- **Embeddings**: Ang embeddings ay mga vector na representasyon ng mga tokens na sumasaklaw sa kahulugang semantiko. Ito ay mga numerikal na representasyon (karaniwang mga array ng floating-point na numero) na nagbibigay-daan sa mga modelo na maunawaan ang mga ugnayan ng mga salita at makabuo ng mga tugon na may kaugnayang konteksto. May pagkakapareho ang mga salita sa embeddings upang matulungan ang modelo na maunawaan ang mga konsepto tulad ng mga sinonimo at mga relasyon sa semantika.

![Figure: Embeddings](../../../translated_images/tl/embedding.398e50802c0037f9.webp)

  Sa Java, maaari kang gumawa ng embeddings gamit ang OpenAI SDK o iba pang mga library na sumusuporta sa paggawa ng embeddings. Mahalaga ang mga embeddings para sa mga gawain tulad ng semantic search, kung saan nais mong hanapin ang mga katulad na nilalaman base sa kahulugan imbes sa eksaktong pagkapareho ng teksto.

- **Vector databases**: Ang mga vector database ay mga espesyal na sistema ng imbakan na naka-optimize para sa embeddings. Pinapadali nila ang mabilis na paghahanap ng pagkakatulad at mahalaga para sa mga pattern ng Retrieval-Augmented Generation (RAG) kung saan kailangan mong maghanap ng kaugnay na impormasyon mula sa malalaking dataset base sa semantikong pagkakatulad at hindi eksaktong tugma.

![Figure: Arkitektura ng vector database na nagpapakita kung paano iniimbak at kinukuha ang embeddings para sa paghahanap ng pagkakatulad.](../../../translated_images/tl/vector.f12f114934e223df.webp)

> **Tandaan**: Hindi namin tatalakayin ang mga Vector database sa kursong ito ngunit sulit na banggitin dahil madalas itong gamitin sa mga totoong aplikasyon.

- **Agents at MCP**: Mga bahagi ng AI na awtonomong nakikipag-ugnayan sa mga modelo, mga kasangkapan, at mga panlabas na sistema. Ang Model Context Protocol (MCP) ay nagbibigay ng isang standardized na paraan para sa mga agent na ligtas na ma-access ang mga panlabas na pinagmumulan ng datos at kasangkapan. Matuto pa sa aming [MCP for Beginners](https://github.com/microsoft/mcp-for-beginners) na kurso.

Sa mga Java AI application, gagamitin mo ang tokens para sa pagproseso ng teksto, embeddings para sa semantic search at RAG, vector databases para sa retrieval ng data, at agents na may MCP para makabuo ng matatalinong systemang gumagamit ng mga kasangkapan. 

![Figure: Paano ang prompt ay nagiging sagot—tokens, vectors, opsyonal na RAG lookup, pag-iisip ng LLM, at isang MCP agent lahat sa isang mabilis na daloy.](../../../translated_images/tl/flow.f4ef62c3052d12a8.webp)

### Mga Kasangkapan at Library ng AI Development para sa Java

Nag-aalok ang Java ng mahusay na kasangkapan para sa AI development. Mayroong tatlong pangunahing mga library na tatalakayin natin sa buong kursong ito - OpenAI Java SDK, Azure OpenAI SDK, at Spring AI.

Narito ang isang mabilis na talaan na nagpapakita kung aling SDK ang ginagamit sa mga halimbawa sa bawat kabanata:

| Kabanata | Halimbawa | SDK |
|---------|--------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**Mga Link sa Dokumentasyon ng SDK:**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

Ang OpenAI SDK ay ang opisyal na Java library para sa OpenAI API. Nagbibigay ito ng simple at pare-parehong interface para makipag-ugnayan sa mga modelo ng OpenAI, kaya madaling i-integrate ang AI capabilities sa mga Java application. Ipinapakita ng Pet Story application at Foundry Local na halimbawa sa Kabanata 4 ang paggamit ng OpenAI SDK kasama ang Azure AI Foundry.

#### Spring AI

Ang Spring AI ay isang komprehensibong framework na nagdadala ng AI capabilities sa mga Spring application, nagbibigay ng pare-parehong abstraction layer sa iba't ibang AI providers. Ito ay seamless na nakikipag-integrate sa Spring ecosystem, na ginagawa itong ideal na piliin para sa mga enterprise Java application na nangangailangan ng AI capabilities.

Ang lakas ng Spring AI ay nasa kanyang seamless integration sa Spring ecosystem, kaya madali kang makabuo ng production-ready AI application gamit ang mga pamilyar na pattern ng Spring tulad ng dependency injection, configuration management, at testing frameworks. Gagamitin mo ang Spring AI sa Kabanata 2 at 4 para bumuo ng mga application na gumagamit ng parehong OpenAI at Model Context Protocol (MCP) Spring AI libraries.

##### Model Context Protocol (MCP)

Ang [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) ay isang umuusbong na standard na nagbibigay-daan sa mga AI application na ligtas na makipag-ugnayan sa mga panlabas na pinagkukunan ng data at mga kasangkapan. Nagbibigay ang MCP ng standardized na paraan para ma-access ng AI models ang contextual information at maisagawa ang mga aksyon sa iyong mga application.

Sa Kabanata 4, bubuuin mo ang isang simpleng MCP calculator service na nagpapakita ng mga pundasyon ng Model Context Protocol gamit ang Spring AI, ipinapakita kung paano gumawa ng mga basic tool integrations at service architectures.

#### Azure OpenAI Java SDK

Ang Azure OpenAI client library para sa Java ay isang adaptasyon ng OpenAI REST APIs na nagbibigay ng idiomatic na interface at integrasyon sa iba pang bahagi ng Azure SDK ecosystem. Sa Kabanata 3, gagawa ka ng mga application gamit ang Azure OpenAI SDK, kasama na ang chat applications, function calling, at mga pattern ng RAG (Retrieval-Augmented Generation).

> Tandaan: Ang Azure OpenAI SDK ay nai-lag kumpara sa OpenAI Java SDK sa mga tampok kaya para sa mga susunod na proyekto, isaalang-alang ang paggamit ng OpenAI Java SDK.

## Buod

Dito nagtatapos ang mga pundasyon! Ngayon ay nauunawaan mo na ang:

- Mga pangunahing konsepto sa likod ng generative AI - mula sa LLMs at prompt engineering hanggang sa tokens, embeddings, at vector databases
- Mga opsyon ng toolkit para sa Java AI development: Azure OpenAI SDK, Spring AI, at OpenAI Java SDK
- Ano ang Model Context Protocol at kung paano nito pinapagana ang AI agents na magtrabaho gamit ang mga panlabas na kasangkapan

## Mga Susunod na Hakbang

[Chapter 2: Setting Up the Development Environment](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Pagtatanggi**:
Ang dokumentong ito ay isinalin gamit ang serbisyo ng AI translation na [Co-op Translator](https://github.com/Azure/co-op-translator). Bagama't nagsusumikap kami para sa katumpakan, pakatandaan na ang awtomatikong pagsasalin ay maaaring maglaman ng mga pagkakamali o hindi pagkakatugma. Ang orihinal na dokumento sa orihinal nitong wika ang dapat ituring na pangunahing sanggunian. Para sa mahahalagang impormasyon, inirerekomenda ang propesyonal na pagsasalin ng tao. Hindi kami mananagot sa anumang maling pagkakaintindi o maling interpretasyon na nagmula sa paggamit ng pagsasaling ito.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->