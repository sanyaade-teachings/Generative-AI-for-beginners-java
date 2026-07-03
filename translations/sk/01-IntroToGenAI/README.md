# Úvod do generatívnej AI - Java edícia

## Čo sa naučíte

- **Základy generatívnej AI**, vrátane LLM, prompt engineering, tokenov, embeddingov a vektorových databáz
- **Porovnanie nástrojov na vývoj AI v Jave**, vrátane Azure OpenAI SDK, Spring AI a OpenAI Java SDK
- **Objavíte Model Context Protocol** a jeho úlohu v komunikácii AI agentov

## Obsah

- [Úvod](#úvod)
- [Rýchle zopakovanie pojmov generatívnej AI](#rýchle-zopakovanie-pojmov-generatívnej-ai)
- [Prehľad prompt engineeringu](#prehľad-prompt-engineeringu)
- [Tokény, embeddingy a agenti](#tokény-embeddingy-a-agenti)
- [Nástroje a knižnice pre AI vývoj v Jave](#nástroje-a-knižnice-pre-ai-vývoj-v-jave)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [Zhrnutie](#zhrnutie)
- [Ďalšie kroky](#ďalšie-kroky)

## Úvod

Vitajte v prvej kapitole Generatívnej AI pre začiatočníkov - Java edícia! Táto základná lekcia vás zoznámi s kľúčovými konceptmi generatívnej AI a ich používaním v Jave. Naučíte sa o základných stavebných prvkoch AI aplikácií, vrátane veľkých jazykových modelov (LLM), tokenov, embeddingov a AI agentov. Tiež preskúmame hlavné Java nástroje, ktoré budete počas kurzu používať.

### Rýchle zopakovanie pojmov generatívnej AI

Generatívna AI je typ umelej inteligencie, ktorá vytvára nový obsah, ako napríklad text, obrázky alebo kód, na základe vzorov a vzťahov naučených z dát. Generatívne AI modely dokážu generovať odpovede podobné ľudským, chápať kontext a niekedy dokonca vytvárať obsah, ktorý sa zdá byť ľudský.

Pri vývoji Java AI aplikácií budete pracovať s **generatívnymi AI modelmi** na tvorbu obsahu. Niektoré schopnosti generatívnych AI modelov sú:

- **Generovanie textu**: tvorba ľudsky pôsobiaceho textu pre chatbotov, obsah a doplnenie textu.
- **Generovanie a analýza obrázkov**: tvorba realistických obrázkov, vylepšovanie fotografií a detekcia objektov.
- **Generovanie kódu**: písanie útržkov alebo skriptov.

Existujú špecifické typy modelov optimalizované pre rôzne úlohy. Napríklad, ako **malé jazykové modely (SLM)**, tak aj **veľké jazykové modely (LLM)** dokážu spracovať generovanie textu, pričom LLM zvyčajne ponúkajú lepšie výsledky pre zložitejšie úlohy. Pre úlohy súvisiace s obrázkami by ste použili špecializované modely vizuálne alebo multimodálne modely.

![Obrázok: Typy generatívnych AI modelov a ich použitie.](../../../translated_images/sk/llms.225ca2b8a0d34473.webp)

Samozrejme, odpovede týchto modelov nie sú vždy dokonalé. Pravdepodobne ste počuli, že modely "halucinujú" alebo generujú nesprávne informácie s autoritatívnym tónom. Môžete však pomôcť nasmerovať model, aby generoval lepšie odpovede, poskytnutím jasných inštrukcií a kontextu. Tu prichádza na scénu **prompt engineering**.

#### Prehľad prompt engineeringu

Prompt engineering je prax navrhovania efektívnych vstupov, ktoré vedú AI modely k požadovaným výstupom. Zahrňuje:

- **Jasnosť**: robiť inštrukcie zrozumiteľné a jednoznačné.
- **Kontext**: poskytovať potrebné podklady.
- **Obmedzenia**: špecifikovať limitácie alebo formáty.

Medzi najlepšie praktiky pre prompt engineering patrí návrh promptov, jasné inštrukcie, rozdelenie úlohy, one-shot a few-shot učenie a ladenie promptov. Je dôležité testovať rôzne prompty, aby ste našli tie najvhodnejšie pre váš konkrétny prípad použitia.

Pri vývoji aplikácií budete pracovať s rôznymi typmi promptov:
- **Systémové prompty**: nastavujú základné pravidlá a kontext pre správanie modelu
- **Užívateľské prompty**: vstupné údaje od používateľov vašej aplikácie
- **Assistant prompty**: odpovede modelu založené na systémových a užívateľských promtoch

> **Viac informácií**: Viac o prompt engineeringu nájdete v [kapitole Prompt Engineering v kurze GenAI pre začiatočníkov](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### Tokény, embeddingy a agenti

Pri práci s generatívnymi AI modelmi narazíte na pojmy ako **tokény**, **embeddingy**, **agenti** a **Model Context Protocol (MCP)**. Tu je podrobný prehľad týchto konceptov:

- **Tokény**: Tokény sú najmenšou jednotkou textu v modeli. Môžu byť slovami, znakmi alebo subslovami. Tokény slúžia na reprezentáciu textových dát v podobe, ktorú model dokáže spracovať. Napríklad veta "The quick brown fox jumped over the lazy dog" môže byť tokenizovaná ako ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] alebo ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"] v závislosti od tokenizačnej stratégie.

![Obrázok: Príklad generatívnych AI tokenov, rozdelenie slov na tokeny](../../../translated_images/sk/tokens.6283ed277a2ffff4.webp)

Tokenizácia je proces rozdelenia textu na tieto menšie jednotky. Je to kľúčové, pretože modely pracujú s tokenmi, nie s holým textom. Počet tokenov v prompte ovplyvňuje dĺžku a kvalitu odpovede modelu, pretože modely majú limity tokenov pre svoj kontextový rámec (napríklad 128K tokenov pre celkový kontext GPT-4o, vrátane vstupu aj výstupu).

  V Jave môžete používať knižnice ako OpenAI SDK, ktoré tokenizáciu automaticky zvládajú pri odosielaní požiadaviek na AI modely.

- **Embeddingy**: Embeddingy sú vektorové reprezentácie tokenov, ktoré zachytávajú sémantický význam. Sú to numerické reprezentácie (zvyčajne polia čísel s pohyblivou desatinnou čiarkou), ktoré umožňujú modelom chápať vzťahy medzi slovami a generovať kontextovo relevantné odpovede. Podobné slová majú podobné embeddingy, čo modelu umožňuje pochopiť koncepty ako synonymá a sémantické vzťahy.

![Obrázok: Embeddingy](../../../translated_images/sk/embedding.398e50802c0037f9.webp)

  V Jave môžete embeddingy generovať pomocou OpenAI SDK alebo iných knižníc podporujúcich generovanie embeddingov. Tieto embeddingy sú nevyhnutné pre úlohy ako sémantické vyhľadávanie, kde chcete nájsť podobný obsah na základe významu, nie presnej zhody textu.

- **Vektorové databázy**: Vektorové databázy sú špecializované úložiská optimalizované pre embeddingy. Umožňujú efektívne vyhľadávanie podobnosti a sú veľmi dôležité pre pattern Retrieval-Augmented Generation (RAG), kde potrebujete nájsť relevantné informácie z veľkých dátových zdrojov na základe sémantickej podobnosti namiesto presnej zhodnosti.

![Obrázok: Architektúra vektorovej databázy zobrazujúca, ako sú embeddingy ukladané a vyhľadávané pre vyhľadávanie podobnosti.](../../../translated_images/sk/vector.f12f114934e223df.webp)

> **Poznámka**: V tomto kurze nebudeme pokrývať vektorové databázy, ale myslíme si, že stojí za to ich spomenúť, pretože sa bežne používajú v reálnych aplikáciách.

- **Agenti a MCP**: AI komponenty, ktoré autonómne komunikujú s modelmi, nástrojmi a externými systémami. Model Context Protocol (MCP) poskytuje štandardizovaný spôsob, ako môžu agenti bezpečne pristupovať k externým dátovým zdrojom a nástrojom. Viac sa dozviete v našom kurze [MCP pre začiatočníkov](https://github.com/microsoft/mcp-for-beginners).

V Java AI aplikáciách použijete tokény na spracovanie textu, embeddingy na sémantické vyhľadávanie a RAG, vektorové databázy na vyhľadávanie dát a agentov s MCP na budovanie inteligentných systémov používajúcich nástroje.

![Obrázok: Ako sa z promptu stáva odpoveď - tokény, vektory, voliteľné RAG vyhľadávanie, myslenie LLM a MCP agent v jednom rýchlom toku.](../../../translated_images/sk/flow.f4ef62c3052d12a8.webp)

### Nástroje a knižnice pre AI vývoj v Jave

Java ponúka vynikajúce nástroje na AI vývoj. Počas kurzu preskúmame tri hlavné knižnice - OpenAI Java SDK, Azure OpenAI SDK a Spring AI.

Tu je rýchly prehľad tabulky, ktorá ukazuje, ktorý SDK sa používa v príkladoch jednotlivých kapitol:

| Kapitola | Príklad | SDK |
|---------|---------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**Odkazy na dokumentáciu SDK:**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

OpenAI SDK je oficiálna Java knižnica pre OpenAI API. Poskytuje jednoduché a konzistentné rozhranie pre prácu s modelmi OpenAI, čo uľahčuje integráciu AI schopností do Java aplikácií. V kapitole 4 sú aplikácia Pet Story a príklad Foundry Local demonštráciou použitia OpenAI SDK s Azure AI Foundry.

#### Spring AI

Spring AI je komplexný framework, ktorý prináša AI schopnosti do Spring aplikácií a poskytuje konzistentnú abstrakciu naprieč rôznymi AI poskytovateľmi. Integruje sa hladko so Spring ekosystémom a je ideálny pre podnikové Java aplikácie využívajúce AI.

Silnou stránkou Spring AI je jeho bezproblémová integrácia so Spring ekosystémom, čo uľahčuje budovanie produkčne pripravených AI aplikácií pomocou známych Spring vzorov, ako sú dependency injection, správa konfigurácie a testovacie frameworky. Spring AI použijete v kapitolách 2 a 4 na tvorbu aplikácií využívajúcich OpenAI a Model Context Protocol (MCP) Spring AI knižnice.

##### Model Context Protocol (MCP)

[Model Context Protocol (MCP)](https://modelcontextprotocol.io/) je vznikajúci štandard, ktorý umožňuje AI aplikáciám bezpečne komunikovať s externými dátovými zdrojmi a nástrojmi. MCP poskytuje štandardizovaný spôsob, ako môžu AI modely pristupovať ku kontextovým informáciám a vykonávať akcie vo vašich aplikáciách.

V kapitole 4 vytvoríte jednoduchú MCP kalkulačkovú službu, ktorá demonštruje základy Model Context Protocol so Spring AI, ukazujúc, ako vytvárať základné integrácie nástrojov a architektúry služieb.

#### Azure OpenAI Java SDK

Azure OpenAI klientská knižnica pre Javu je adaptáciou OpenAI REST API, ktorá poskytuje idiomatické rozhranie a integráciu so zvyškom Azure SDK ekosystému. V kapitole 3 budete vytvárať aplikácie pomocou Azure OpenAI SDK, vrátane chatových aplikácií, volania funkcií a RAG (Retrieval-Augmented Generation) vzorov.

> Poznámka: Azure OpenAI SDK zaostáva za OpenAI Java SDK funkcionalitou, preto pre budúce projekty zvážte použitie OpenAI Java SDK.

## Zhrnutie

Tým sme zvládli základy! Teraz rozumiete:

- Kľúčovým konceptom generatívnej AI - od LLM a prompt engineeringu až po tokény, embeddingy a vektorové databázy
- Vašim možnostiam nástrojov pre Java AI vývoj: Azure OpenAI SDK, Spring AI a OpenAI Java SDK
- Čo je Model Context Protocol a ako umožňuje AI agentom pracovať s externými nástrojmi

## Ďalšie kroky

[Kapitola 2: Nastavenie vývojového prostredia](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Vyhlásenie o zodpovednosti**:
Tento dokument bol preložený pomocou AI prekladateľskej služby [Co-op Translator](https://github.com/Azure/co-op-translator). Hoci sa snažíme o presnosť, vezmite prosím na vedomie, že automatické preklady môžu obsahovať chyby alebo nepresnosti. Pôvodný dokument v jeho natívnom jazyku by mal byť považovaný za autoritatívny zdroj. Pre kritické informácie sa odporúča profesionálny ľudský preklad. Nie sme zodpovední za žiadne nedorozumenia alebo nesprávne interpretácie vyplývajúce z použitia tohto prekladu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->