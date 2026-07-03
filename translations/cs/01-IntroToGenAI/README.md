# Úvod do Generative AI - Java edice

## Co se naučíte

- **Základy Generative AI** včetně LLM, návrhu promptů, tokenů, embeddingů a vektorových databází
- **Porovnání nástrojů pro vývoj AI v Javě** včetně Azure OpenAI SDK, Spring AI a OpenAI Java SDK
- **Objevíte Model Context Protocol** a jeho roli v komunikaci AI agentů

## Obsah

- [Úvod](#úvod)
- [Rychlé zopakování konceptů Generative AI](#rychlé-zopakování-konceptů-generative-ai)
- [Přehled návrhu promptů](#přehled-návrhu-promptů)
- [Toky, embeddingy a agenti](#toky-embeddingy-a-agenti)
- [Nástroje a knihovny pro vývoj AI v Javě](#nástroje-a-knihovny-pro-vývoj-ai-v-javě)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [Shrnutí](#shrnutí)
- [Další kroky](#další-kroky)

## Úvod

Vítejte v první kapitole Generative AI pro začátečníky - Java edice! Toto základní lekce vás seznámí s klíčovými koncepty generativní AI a tím, jak s nimi pracovat v Javě. Naučíte se o základních stavebních blocích AI aplikací, včetně Velkých jazykových modelů (LLM), tokenů, embeddingů a AI agentů. Také prozkoumáme hlavní Java nástroje, které budete používat v průběhu kurzu.

### Rychlé zopakování konceptů Generative AI

Generativní AI je typ umělé inteligence, která vytváří nový obsah, jako je text, obrázky nebo kód, na základě vzorců a vztahů naučených z dat. Modely generativní AI mohou generovat lidsky podobné odpovědi, rozumět kontextu a někdy dokonce vytvořit obsah, který působí lidsky.

Při vývoji svých Java AI aplikací budete pracovat s **generativními AI modely** pro tvorbu obsahu. Některé schopnosti generativních AI modelů zahrnují:

- **Generování textu**: Tvorba lidsky vypadajícího textu pro chatboty, obsah a doplňování textu.
- **Generování a analýza obrázků**: Produkce realistických obrázků, vylepšení fotografií a detekce objektů.
- **Generování kódu**: Psaní úryvků kódu nebo skriptů.

Existují specifické typy modelů optimalizované pro různé úlohy. Například jak **malé jazykové modely (SLM)**, tak **velké jazykové modely (LLM)** dokážou generovat text, přičemž LLM obvykle nabízí lepší výkon u složitějších úloh. Pro úlohy související s obrazem používáte specializované vizuální modely nebo multimodální modely.

![Obrázek: Typy generativních AI modelů a jejich použití.](../../../translated_images/cs/llms.225ca2b8a0d34473.webp)

Samozřejmě, odpovědi těchto modelů nejsou vždy dokonalé. Pravděpodobně jste slyšeli o modelech, které „halucinují“ nebo generují nesprávné informace autoritativním způsobem. Ale můžete pomoci modelu generovat lepší odpovědi tím, že mu poskytnete jasné instrukce a kontext. Zde přichází na řadu **návrh promptů**.

#### Přehled návrhu promptů

Návrh promptů je praxe navrhování efektivních vstupů k nasměrování AI modelů k požadovaným výstupům. Zahrnuje:

- **Srozumitelnost**: Udělat instrukce jasné a jednoznačné.
- **Kontext**: Poskytnout nezbytné pozadí.
- **Omezení**: Specifikovat případná omezení nebo formáty.

Mezi nejlepší postupy návrhu promptů patří návrh promptu, jasné instrukce, rozdělení úlohy, one-shot a few-shot learning a ladění promptů. Testování různých promptů je nezbytné pro nalezení toho, co nejlépe funguje pro váš konkrétní případ použití.

Při vývoji aplikací budete pracovat s různými typy promptů:
- **Systémové prompty**: Nastavují základní pravidla a kontext chování modelu
- **Uživatelské prompty**: Vstupní data od uživatelů vaší aplikace
- **Asistenční prompty**: Odpovědi modelu založené na systémových a uživatelských promptech

> **Další informace**: Více o návrhu promptů najdete v [kapitole Prompt Engineering kurzu GenAI for Beginners](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### Toky, embeddingy a agenti

Při práci s generativními AI modely narazíte na pojmy jako **tokeny**, **embeddingy**, **agenti** a **Model Context Protocol (MCP)**. Zde je podrobný přehled těchto konceptů:

- **Tokeny**: Tokeny jsou nejmenší jednotky textu v modelu. Mohou to být slova, znaky nebo podslova. Tokeny se používají k reprezentaci textových dat ve formátu, kterému model rozumí. Například věta „The quick brown fox jumped over the lazy dog“ může být rozložena na tokeny jako ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] nebo ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"] v závislosti na strategii tokenizace.

![Obrázek: Příklad rozbití slov na tokeny v generativním AI](../../../translated_images/cs/tokens.6283ed277a2ffff4.webp)

Tokenizace je proces rozdělení textu na tyto menší jednotky. To je klíčové, protože modely pracují s tokeny místo s čistým textem. Počet tokenů v promptu ovlivňuje délku a kvalitu odpovědi, protože modely mají limity tokenů pro své kontextové okno (např. 128K tokenů pro GPT-4o celkový kontext včetně vstupu i výstupu).

  V Javě můžete použít knihovny jako OpenAI SDK, které tokenizaci automaticky řeší při odesílání požadavků na AI modely.

- **Embeddingy**: Embeddingy jsou vektorové reprezentace tokenů, které zachycují sémantický význam. Jsou to číselné reprezentace (obvykle pole desetinných čísel), které umožňují modelům chápat vztahy mezi slovy a generovat kontextově relevantní odpovědi. Podobná slova mají podobné embeddingy, což umožňuje modelu rozpoznávat synonyma a sémantické vztahy.

![Obrázek: Embeddingy](../../../translated_images/cs/embedding.398e50802c0037f9.webp)

  V Javě můžete embeddingy generovat pomocí OpenAI SDK nebo jiných knihoven podporujících generování embeddingů. Tyto embeddingy jsou klíčové pro úlohy jako sémantické vyhledávání, kdy chcete najít podobný obsah na základě významu spíše než přesných textových shod.

- **Vektorové databáze**: Vektorové databáze jsou specializované úložiště optimalizované pro embeddingy. Umožňují efektivní vyhledávání podobnosti a jsou zásadní pro vzory Retrieval-Augmented Generation (RAG), kde musíte najít relevantní informace z velkých datových sad na základě sémantické podobnosti namísto přesných shod.

![Obrázek: Architektura vektorové databáze zobrazující, jak jsou embeddingy ukládány a vyhledávány pro vyhledávání podobnosti.](../../../translated_images/cs/vector.f12f114934e223df.webp)

> **Poznámka**: V tomto kurzu se vektorovými databázemi nebudeme podrobně zabývat, ale stojí za zmínku, protože jsou často používány v reálných aplikacích.

- **Agenti & MCP**: Komponenty AI, které autonomně komunikují s modely, nástroji a externími systémy. Model Context Protocol (MCP) poskytuje standardizovaný způsob, jak agenti bezpečně přistupují k externím zdrojům dat a nástrojům. Více se naučíte v našem kurzu [MCP pro začátečníky](https://github.com/microsoft/mcp-for-beginners).

V Java AI aplikacích budete používat tokeny pro zpracování textu, embeddingy pro sémantické vyhledávání a RAG, vektorové databáze pro získávání dat a agenty s MCP pro vytváření inteligentních systémů používajících nástroje.

![Obrázek: Jak se prompt stane odpovědí – tokeny, vektory, volitelný RAG lookup, přemýšlení LLM a MCP agent vše v jednom rychlém toku.](../../../translated_images/cs/flow.f4ef62c3052d12a8.webp)

### Nástroje a knihovny pro vývoj AI v Javě

Java nabízí vynikající nástroje pro vývoj AI. Existují tři hlavní knihovny, které během kurzu prozkoumáme - OpenAI Java SDK, Azure OpenAI SDK a Spring AI.

Zde je rychlá tabulka ukazující, který SDK je použit v příkladech jednotlivých kapitol:

| Kapitola | Ukázka | SDK |
|---------|--------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**Odkazy na dokumentaci SDK:**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

OpenAI SDK je oficiální Java knihovna pro OpenAI API. Poskytuje jednoduché a konzistentní rozhraní pro interakci s modely OpenAI, což usnadňuje integraci AI funkcionalit do Java aplikací. Aplikace Pet Story a příklad Foundry Local v kapitole 4 demonstrují přístup pomocí OpenAI SDK společně s Azure AI Foundry.

#### Spring AI

Spring AI je komplexní framework, který přináší AI schopnosti do Spring aplikací a poskytuje konzistentní abstrakci napříč různými AI poskytovateli. Seamlessly se integruje do Spring ekosystému, což jej činí ideální volbou pro podnikové Java aplikace, které potřebují AI funkce.

Síla Spring AI spočívá v jeho bezproblémové integraci se Spring ekosystémem, což usnadňuje vytváření produkčně připravených AI aplikací pomocí známých Spring vzorů jako dependency injection, správa konfigurace a testovací frameworky. Spring AI budete používat v kapitolách 2 a 4 k vytváření aplikací, které využívají jak OpenAI, tak Model Context Protocol (MCP) Spring AI knihovny.

##### Model Context Protocol (MCP)

[Model Context Protocol (MCP)](https://modelcontextprotocol.io/) je vznikající standard, který umožňuje AI aplikacím bezpečně komunikovat s externími zdroji dat a nástroji. MCP poskytuje standardizovaný způsob, jak AI modely přistupují k kontextovým informacím a vykonávají akce ve vašich aplikacích.

V kapitole 4 vytvoříte jednoduchou MCP kalkulačku, která předvede základy Model Context Protocol se Spring AI, ukáže, jak vytvářet základní integrace nástrojů a architektury služeb.

#### Azure OpenAI Java SDK

Knihovna klienta Azure OpenAI pro Javu je adaptací OpenAI REST API, která poskytuje idiomatické rozhraní a integraci s celým Azure SDK ekosystémem. V kapitole 3 budete vytvářet aplikace používající Azure OpenAI SDK, včetně chat aplikací, volání funkcí a RAG (Retrieval-Augmented Generation).

> Poznámka: Azure OpenAI SDK zaostává za OpenAI Java SDK co do funkcí, proto pro budoucí projekty zvažte použití OpenAI Java SDK.

## Shrnutí

Tím jsme dokončili základy! Nyní rozumíte:

- Klíčovým konceptům za generativní AI - od LLM a návrhu promptů po tokeny, embeddingy a vektorové databáze
- Vašim možnostem nástrojů pro vývoj AI v Javě: Azure OpenAI SDK, Spring AI a OpenAI Java SDK
- Co je Model Context Protocol a jak umožňuje AI agentům pracovat s externími nástroji

## Další kroky

[2. kapitola: Nastavení vývojového prostředí](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Prohlášení o omezení odpovědnosti**:
Tento dokument byl přeložen pomocí AI překladatelské služby [Co-op Translator](https://github.com/Azure/co-op-translator). Přestože usilujeme o co největší přesnost, mějte prosím na paměti, že automatizované překlady mohou obsahovat chyby nebo nepřesnosti. Originální dokument v jeho mateřském jazyce by měl být považován za autoritativní zdroj. Pro kritické informace se doporučuje profesionální lidský překlad. Nejsme odpovědní za jakékoli nedorozumění nebo nesprávné interpretace vzniklé použitím tohoto překladu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->