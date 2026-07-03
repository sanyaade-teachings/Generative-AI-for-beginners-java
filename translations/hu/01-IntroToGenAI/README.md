# Bevezetés a Generatív AI-ba - Java kiadás

## Amit megtanulsz

- **Generatív AI alapjai**, beleértve az LLM-eket, prompt tervezést, tokeneket, beágyazásokat és vektordatabázisokat
- **Java AI fejlesztőeszközök összehasonlítása**, köztük az Azure OpenAI SDK, Spring AI és OpenAI Java SDK
- **A Model Context Protocol felfedezése** és annak szerepe az AI ügynökök közötti kommunikációban

## Tartalomjegyzék

- [Bevezetés](#bevezetes)
- [Gyors áttekintés a generatív AI fogalmairól](#gyors-attekintes-a-generativ-ai-fogalmairol)
- [Prompt tervezés áttekintése](#prompt-tervezes-atekintese)
- [Tokenek, beágyazások és ügynökök](#tokenek-beagyazasok-es-ugynokok)
- [AI fejlesztőeszközök és könyvtárak Java számára](#ai-fejlesztoeszkozok-es-konyvtarak-java-szamara)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [Összefoglaló](#osszefoglalo)
- [Következő lépések](#kovetkezo-lepesek)

## Bevezetés

Üdvözölünk a Generatív AI kezdőknek - Java kiadás első fejezetében! Ez az alapozó lecke bevezetést nyújt a generatív AI alapvető fogalmaiba, és hogy hogyan dolgozhatsz velük Java használatával. Megismered az AI alkalmazások alapvető építőköveit, beleértve a Nagy Nyelvi Modelleket (LLM-ek), tokeneket, beágyazásokat és AI ügynököket. Feltérképezzük az alapvető Java eszközöket is, amelyeket a tanfolyam során fogsz használni.

### Gyors áttekintés a generatív AI fogalmairól

A generatív AI egy olyan mesterséges intelligencia, amely új tartalmakat hoz létre, például szöveget, képeket vagy kódot, az adatokból tanult minták és összefüggések alapján. A generatív AI modellek képesek emberihez hasonló válaszokat generálni, megérteni a kontextust, és néha akár emberinek tűnő tartalmakat készíteni.

Ahogy fejleszted a Java AI alkalmazásaidat, **generatív AI modellekkel** fogsz dolgozni a tartalom létrehozásához. A generatív AI modellek képességei közé tartozik:

- **Szöveggenerálás**: emberihez hasonló szövegek készítése chatbotok, tartalom vagy szövegkiegészítés céljára.
- **Képalkotás és elemzés**: valósághű képek létrehozása, fotók javítása, tárgyak felismerése.
- **Kódgenerálás**: kódrészletek vagy script-ek írása.

Speciális modellek vannak, amelyek optimalizáltak különböző feladatokra. Például mind a **kis nyelvi modellek (SLM-ek)**, mind a **nagy nyelvi modellek (LLM-ek)** képesek szöveggenerálásra, ahol az LLM-ek általában jobb teljesítményt nyújtanak bonyolultabb feladatoknál. Képhez kapcsolódó feladatokhoz speciális látásmodelleket vagy multimodális modelleket használnál.

![Ábra: Generatív AI modell típusok és alkalmazási területek.](../../../translated_images/hu/llms.225ca2b8a0d34473.webp)

Természetesen ezek a modellek sem mindig adnak tökéletes válaszokat. Valószínűleg hallottál már arról, hogy a modellek „hallucinálnak” vagy téves információkat generálnak meggyőző módon. De segíthetsz irányítani a modellt, hogy jobb válaszokat adjon, ha világos utasításokat és kontextust biztosítasz számára. Itt jön képbe a **prompt tervezés**.

#### Prompt tervezés áttekintése

A prompt tervezés az a gyakorlat, amikor hatékony bemeneteket készítesz az AI modellek számára, hogy azok a kívánt kimenetet hozzák létre. Ez magában foglalja:

- **Világosság**: egyértelmű és félreérthetetlen utasítások megadása.
- **Kontextus**: a szükséges háttérinformációk biztosítása.
- **Korlátozások**: bármilyen korlátozás vagy formátum meghatározása.

A prompt tervezés legjobb gyakorlatai közé tartozik a prompt tervezése, világos utasítások megadása, a feladat lebontása, egy- vagy néhánypéldányos tanulás, valamint prompt finomhangolás. Az eltérő promptok tesztelése elengedhetetlen, hogy megtaláld, mi működik legjobban a konkrét felhasználási esethez.

Alkalmazásfejlesztés során különböző prompt típusokkal dolgozol majd:
- **Rendszer promptok**: beállítják az alapvető szabályokat és a modell viselkedésének kontextusát
- **Felhasználói promptok**: az alkalmazásod felhasználóitól érkező bemeneti adatok
- **Asszisztens promptok**: a modell válaszai a rendszer és felhasználói promptok alapján

> **Tanulj többet**: Tudj meg többet a prompt tervezésről a [Prompt Engineering fejezetben a GenAI for Beginners kurzusban](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### Tokenek, beágyazások és ügynökök

Generatív AI modellekkel dolgozva találkozhatsz olyan fogalmakkal, mint a **tokenek**, **beágyazások**, **ügynökök** és a **Model Context Protocol (MCP)**. Íme egy részletes áttekintés ezekről a fogalmakról:

- **Tokenek**: A tokenek a szöveg legkisebb egységei egy modellben. Lehetnek szavak, karakterek vagy szótöredékek. A tokeneket arra használják, hogy a szöveges adatokat egy olyan formátumban képviseljék, amelyet a modell megért. Például az "A gyors barna róka átugrott a lusta kutya" mondat tokenekre bontható úgy, hogy ["A", " gyors", " barna", " róka", " átugrott", " a", " lusta", " kutya"] vagy tokenizálási stratégiától függően ["A", " gy", "ors", " ba", "rna", " ró", "ka", " át", "ugr", "ott", " a", " lu", "sta", " ku", "tya"].

![Ábra: Generatív AI tokenek példája a szavak tokenekre bontásáról](../../../translated_images/hu/tokens.6283ed277a2ffff4.webp)

A tokenizáció a szöveg darabolása ezekre az apró egységekre. Ez létfontosságú, mert a modellek tokeneken dolgoznak, nem nyers szövegen. Egy prompt tokenjeinek száma befolyásolja a modell válaszának hosszát és minőségét, mivel a modelleknek korlátuk van a tokenek számára a kontextusablakban (például a GPT-4o összkontekstusára 128K token, mind bemeneti mind kimeneti oldalon).

A Java nyelvben használhatsz könyvtárakat, például az OpenAI SDK-t, amely automatikusan kezeli a tokenizációt, amikor az AI modellekhez kérdéseket küldesz.

- **Beágyazások**: A beágyazások tokenek vektoros ábrázolásai, amelyek szemantikai jelentést ragadnak meg. Ezek numerikus ábrázolások (általában lebegőpontos számok tömbjei), amelyek lehetővé teszik a modell számára a szavak közötti kapcsolatok megértését és kontextuálisan releváns válaszok létrehozását. Hasonló szavak hasonló beágyazással rendelkeznek, így a modell képes megérteni a szinonimákat és szemantikai kapcsolatokat.

![Ábra: Beágyazások](../../../translated_images/hu/embedding.398e50802c0037f9.webp)

Java nyelven beágyazásokat generálhatsz az OpenAI SDK-val vagy más beágyazás-generálást támogató könyvtárakkal. Ezek a beágyazások kulcsfontosságúak olyan feladatokhoz, mint a szemantikai keresés, amikor hasonló tartalmakat akarsz megtalálni jelentés alapján, nem pedig szöveg szerinti egyezésre.

- **Vektordatabázisok**: A vektordatabázisok speciális adattároló rendszerek, amelyek optimalizáltak a beágyazások számára. Hatékony hasonlóságalapú keresést tesznek lehetővé és elengedhetetlenek a RAG (Retrieval-Augmented Generation) mintázatokhoz, ahol nagy adatbázisokból szemantikai hasonlóság alapján kell releváns információt találni, nem pedig pontos egyezés alapján.

![Ábra: Vektordatabázis architektúra, amely megmutatja, hogyan tárolják és kérdezik le a beágyazásokat hasonlóságkeresésre.](../../../translated_images/hu/vector.f12f114934e223df.webp)

> **Megjegyzés**: Ebben a tanfolyamban nem foglalkozunk részletesen a vektordatabázisokkal, de érdemes megemlíteni, mert gyakran használják őket valós alkalmazásokban.

- **Ügynökök és MCP**: Olyan AI komponensek, amelyek önállóan kommunikálnak modellekkel, eszközökkel és külső rendszerekkel. A Model Context Protocol (MCP) egy szabványosított mód arra, hogy az ügynökök biztonságosan hozzáférjenek külső adatforrásokhoz és eszközökhöz. Erről többet megtudhatsz a [MCP kezdőknek](https://github.com/microsoft/mcp-for-beginners) kurzusban.

Java AI alkalmazásokban a tokeneket szövegfeldolgozásra, a beágyazásokat szemantikai keresésre és RAG-re, a vektordatabázisokat adatlekérdezésre, valamint az ügynököket MCP-vel intelligens, eszközhasználó rendszerek építésére használod.

![Ábra: Hogyan válik egy prompt válasszá — tokenek, vektorok, opcionális RAG lekérdezés, LLM gondolkodás, és egy MCP ügynök mind egy gyors folyamatban.](../../../translated_images/hu/flow.f4ef62c3052d12a8.webp)

### AI fejlesztőeszközök és könyvtárak Java számára

Java kiváló eszközöket kínál AI fejlesztéshez. Három fő könyvtárat fogunk végigvenni ezen a tanfolyamon – OpenAI Java SDK, Azure OpenAI SDK és Spring AI.

Az alábbi táblázat bemutatja, mely SDK-t használjuk az egyes fejezetek példáiban:

| Fejezet | Példa | SDK |
|---------|--------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**SDK dokumentációs linkek:**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

Az OpenAI SDK az OpenAI API hivatalos Java könyvtára. Egyszerű és konzisztens interfészt kínál az OpenAI modellekkel való interakcióhoz, megkönnyítve az AI képességek integrálását Java alkalmazásokba. A 4. fejezet Pet Story alkalmazása és Foundry Local példája az OpenAI SDK megközelítését mutatja be Azure AI Foundry-val.

#### Spring AI

A Spring AI egy átfogó keretrendszer, amely AI képességeket hoz a Spring alkalmazásokba, egységes absztrakciós réteget biztosítva különböző AI szolgáltatók között. Zökkenőmentesen integrálódik a Spring ökoszisztémába, ideális választás vállalati Java alkalmazásokhoz, amelyeknek szükségük van AI funkciókra.

A Spring AI ereje a Spring ökoszisztémába való szoros integrációjában rejlik, megkönnyítve a gyártásra kész AI alkalmazások építését ismert Spring minták, mint például függőség-injektálás, konfigurációkezelés és tesztelési keretrendszerek használatával. A 2. és 4. fejezetben Spring AI-t használsz, hogy olyan alkalmazásokat építs, amelyek kihasználják az OpenAI-t és a Model Context Protocol (MCP) Spring AI könyvtárait.

##### Model Context Protocol (MCP)

A [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) egy feltörekvő szabvány, amely lehetővé teszi az AI alkalmazások számára, hogy biztonságosan kommunikáljanak külső adatforrásokkal és eszközökkel. Az MCP szabványosított módot biztosít az AI modellek számára a kontextuális információk elérésére és műveletek végrehajtására alkalmazásaidban.

A 4. fejezetben egy egyszerű MCP kalkulátor szolgáltatást építesz, amely bemutatja a Model Context Protocol alapjait Spring AI-val, megmutatva, hogyan hozhatsz létre alapvető eszközintegrációkat és szolgáltatás architektúrákat.

#### Azure OpenAI Java SDK

Az Azure OpenAI klienskönyvtár Java számára az OpenAI REST API-jainak adaptációja, amely idiomatikus interfészt és integrációt kínál az Azure SDK ökoszisztéma többi részével. A 3. fejezetben alkalmazásokat építesz az Azure OpenAI SDK-val, beleértve chatalkalmazásokat, funkcióhívásokat és RAG (Retrieval-Augmented Generation) mintákat.

> Megjegyzés: Az Azure OpenAI SDK a funkciók terén elmarad az OpenAI Java SDK-tól, így jövőbeli projektekhez érdemes az OpenAI Java SDK használatát fontolóra venni.

## Összefoglaló

Ezzel az alapokkal végeztél! Most már érted:

- A generatív AI mögötti alapvető fogalmakat – az LLM-eket és prompt tervezést, tokeneket, beágyazásokat és vektordatbázisokat
- A Java AI fejlesztéshez rendelkezésedre álló eszköztárakat: Azure OpenAI SDK, Spring AI és OpenAI Java SDK
- Mi az a Model Context Protocol, és hogyan teszi lehetővé az AI ügynököknek, hogy külső eszközökkel dolgozzanak

## Következő lépések

[2. fejezet: Fejlesztői környezet beállítása](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Jogi nyilatkozat**:
Ez a dokumentum az AI fordítási szolgáltatás, a [Co-op Translator](https://github.com/Azure/co-op-translator) segítségével készült. Bár az pontosságra törekszünk, kérjük, vegye figyelembe, hogy az automatikus fordítások hibákat vagy pontatlanságokat tartalmazhatnak. Az eredeti dokumentum az anyanyelvén tekintendő hiteles forrásnak. Fontos információk esetén professzionális emberi fordítást javasolunk. Nem vállalunk felelősséget semmilyen félreértésért vagy téves értelmezésért, amely ebből a fordításból ered.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->