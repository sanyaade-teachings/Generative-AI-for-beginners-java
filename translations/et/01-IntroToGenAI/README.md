# Sissejuhatus generatiivse tehisintellekti - Java väljaanne

## Mida sa õpid

- **Generatiivse tehisintellekti alused** sisaldades LLM-e, prompti disaini, tokeneid, manuseid ja vektandmebaase
- **Võrrelda Java AI arendustööriistu** sealhulgas Azure OpenAI SDK, Spring AI ja OpenAI Java SDK
- **Avasta mudelikonteksti protokoll** ja selle roll AI agentide kommunikatsioonis

## Sisukord

- [Sissejuhatus](#sissejuhatus)
- [Kiire kordus generatiivse tehisintellekti mõistetest](#kiire-kordus-generatiivse-tehisintellekti-mõistetest)
- [Prompti disaini ülevaade](#prompti-disaini-ülevaade)
- [Tokendid, manused ja agendid](#tokendid-manused-ja-agendid)
- [AI arendustööriistad ja teegid Javale](#ai-arendustööriistad-ja-teegid-javale)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [Kokkuvõte](#kokkuvõte)
- [Järgmised sammud](#järgmised-sammud)

## Sissejuhatus

Tere tulemast generatiivse tehisintellekti algajate esimesele peatükile - Java väljaanne! See aluste tund tutvustab sulle generatiivse tehisintellekti põhikontseptsioone ja kuidas nendega Java abil töötada. Õpid tehisintellekti rakenduste olulisi koostisosi, sealhulgas suuri keelemudeleid (LLM-id), tokeneid, manuseid ja AI agente. Samuti tutvustame põhilisi Java tööriistu, mida kasutad kogu kursuse vältel.

### Kiire kordus generatiivse tehisintellekti mõistetest

Generatiivne tehisintellekt on tehisintellekti tüüp, mis loob uut sisu, nagu tekst, pildid või kood, põhinedes andmetest õpitud mustritel ja seostel. Generatiivsed mudelid saavad genereerida inimlaadseid vastuseid, mõista konteksti ja mõnikord isegi luua inimeselaadset sisu.

Java AI rakendusi arendades töötad sa **generatiivsete AI mudelitega**, et luua sisu. Mõned generatiivsete mudelite võimed:

- **Teksti genereerimine**: inimlaadse teksti koostamine vestlusrobotitele, sisule ja tekstide täiendamisele.
- **Pildi genereerimine ja analüüs**: realistlike piltide loomine, fotode parandamine ja objektide tuvastamine.
- **Koodi genereerimine**: koodiplokkide või skriptide kirjutamine.

On erinevat tüüpi mudeleid, mis on optimeeritud erinevateks ülesanneteks. Näiteks nii **väiksed keelemudelid (SLM-id)** kui ka **suured keelemudelid (LLM-id)** suudavad teostada teksti genereerimist, kusjuures LLM-id tavaliselt pakuvad paremat jõudlust keeruliste ülesannete puhul. Piltidega seotud ülesannete jaoks kasutatakse spetsialiseeritud nägemismudeleid või multimodaalseid mudeleid.

![Joonis: generatiivsete AI mudelite tüübid ja kasutusjuhtumid.](../../../translated_images/et/llms.225ca2b8a0d34473.webp)

Muidugi pole mudelite vastused alati täiuslikud. Sa oled tõenäoliselt kuulnud, et mudelid "hallutsineerivad" või genereerivad valesid andmeid autoriteetsel viisil. Kuid sa saad mudelit juhtida paremate vastuste poole, pakkudes neile selgeid juhiseid ja konteksti. Siin tuleb mängu **prompti disain**.

#### Prompti disaini ülevaade

Prompti disain on tõhusate sisendite loomise praktika, et suunata AI mudeleid soovitud väljundite poole. See hõlmab:

- **Selgus**: juhiste selge ja mitmetimõistetamatu esitamine.
- **Kontekst**: vajaliku taustinfo pakkumine.
- **Piirangud**: piirangute või vormingute määratlemine.

Mõned parimad praktikad prompti disainis on prompti kujundamine, selged juhised, ülesannete jaotamine, ühe ja mõne näite põhine õppimine ning prompti häälestamine. Erinevate promptide testimine on oluline, et leida sinu spetsiifilisele kasutusjuhtumile parim lahendus.

Rakenduste loomisel töötad erinevat tüüpi promptidega:
- **Süsteemi promptid**: määravad mudeli käitumise põhireeglid ja konteksti
- **Kasutaja promptid**: kasutajate sisestatud andmed sinu rakenduses
- **Assistiendi promptid**: mudeli vastused süsteemi ja kasutaja promptide põhjal

> **Loe lisaks**: Loe prompti disaini kohta rohkem [GenAI algajate kursuse prompti disaini peatükist](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### Tokendid, manused ja agendid

Generatiivsete AI mudelitega töötades kohtad termineid nagu **tokendid**, **manused**, **agendid** ja **mudelikonteksti protokoll (MCP)**. Siin on detailne ülevaade neist mõistetest:

- **Tokendid**: Tokendid on mudeli väikseimad tekstielemendid. Need võivad olla sõnad, tähemärgid või alamsõnad. Tokeneid kasutatakse tekstandmete esitamiseks formaadis, mida mudel mõistab. Näiteks lause "The quick brown fox jumped over the lazy dog" võidakse tokenizeerida kujul ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] või ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"] sõltuvalt tokeniseerimise strateegiast.

![Joonis: generatiivsete AI tokenite näide sõnade tükeldamisest tokeniteks](../../../translated_images/et/tokens.6283ed277a2ffff4.webp)

Tokeniseerimine on protsess, mille käigus tekst jagatakse väiksemateks osadeks. See on oluline, sest mudelid töötlevad tokeneid, mitte toorteksti. Tokenite arv promptis mõjutab mudeli vastuse pikkust ja kvaliteeti, kuna mudelitel on tokenipiirangud kontekstiaknas (nt 128K tokenit GPT-4o jaoks, mis sisaldab nii sisendi kui väljundi tokenid).

  Javas saad kasutada teeke nagu OpenAI SDK, mis teostab tokeniseerimise automaatselt, kui saadad AI mudelitele päringuid.

- **Manused**: Manused on tokenite vektorkujutised, mis haaravad semantilist tähendust. Need on arvulised esindused (tavaliselt ujukomaarvude massiivid), mis võimaldavad mudelitel mõista sõnadevahelisi suhteid ja genereerida kontekstitundlikke vastuseid. Sarnased sõnad omavad sarnaseid manuseid, võimaldades mudelil mõista sünonüüme ja semantilisi seoseid.

![Joonis: Manused](../../../translated_images/et/embedding.398e50802c0037f9.webp)

  Javas saad manuseid genereerida OpenAI SDK või teiste teekide abil, mis toetavad manuste loomist. Need on hädavajalikud semantiliste otsingute jaoks, kus soovid leida sarnast sisu tähenduse põhjal, mitte täpse tekstilise vaste järgi.

- **Vektandmebaasid**: Vektandmebaasid on spetsialiseeritud andmesalvestussüsteemid, mis on optimeeritud manuste hoidmiseks. Nende abil saab efektiivselt teostada sarnasusotsinguid ja need on üliolulised toetusgeneratsioonis (RAG), kus on vaja leida sobivat infot suurtest andmekogudest semantilise sarnasuse alusel, mitte täpse vaste järgi.

![Joonis: Vektandmebaasi arhitektuur, mis näitab, kuidas manuseid hoitakse ja otsitakse sarnasuse leidmiseks.](../../../translated_images/et/vector.f12f114934e223df.webp)

> **Märkus**: Selles kursuses me vektandmebaase ei käsitle põhjalikult, kuid väärib mainimist, sest neid kasutatakse laialdaselt reaalses maailmas.

- **Agendid ja MCP**: AI komponendid, mis autonoomselt suhtlevad mudelite, tööriistade ja välissüsteemidega. Mudelikonteksti protokoll (MCP) annab standardiseeritud viisi, kuidas agendid saavad turvaliselt ligipääsu välistele andmeallikatele ja tööriistadele. Loe rohkem meie [MCP algajate kursusest](https://github.com/microsoft/mcp-for-beginners).

Java AI rakendustes kasutad sa tokeneid tekstitöötluseks, manuseid semantiliseks otsinguks ja RAG-ks, vektandmebaase andmete taastamiseks ning agente koos MCP-ga nutikate süsteemide loomiseks, mis kasutavad tööriistu.

![Joonis: kuidas promptist saab vastus — tokendid, vektorid, valikuline RAG otsing, LLM mõtlemine ja MCP agent kiire voona.](../../../translated_images/et/flow.f4ef62c3052d12a8.webp)

### AI arendustööriistad ja teegid Javale

Java pakub suurepärast AI arenduse tööriistakomplekti. Peamised kolm teeki, mida selle kursuse jooksul vaatleme, on OpenAI Java SDK, Azure OpenAI SDK ja Spring AI.

Siin on kiire viide tabelis, millist SDK-d kasutatakse iga peatüki näidetes:

| Peatükk | Näide | SDK |
|---------|--------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**SDK dokumentatsiooni lingid:**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

OpenAI SDK on ametlik Java teek OpenAI API jaoks. See pakub lihtsat ja järjepidevat liidest OpenAI mudelitega suhtlemiseks, võimaldades sul hõlpsasti integreerida AI võimeid Java rakendustesse. Peatüki 4 Pet Story rakendus ja Foundry Local näide demonstreerivad OpenAI SDK kasutamist koos Azure AI Foundry'ga.

#### Spring AI

Spring AI on terve raamistik, mis toob AI võimed Springi rakendustesse, pakkudes ühtset abstraktsioonikihte erinevate AI pakkujate vahel. See integreerub sujuvalt Springi ökosüsteemiga, muutes selle ideaalseks valikuks ärilisele Java arendusele, mis vajab AI võimalusi.

Spring AI tugevuseks on selle sujuv integreerimine Springi ökosüsteemiga, muutes lihtsaks tootmiskõlbulike AI rakenduste loomise tuttavate Springi mustrite, nagu sõltuvussüstimine, konfiguratsioonihaldus ja testimisraamistikud abil. Kasutad Spring AI-t peatükkides 2 ja 4, et luua rakendusi, mis toetavad nii OpenAI kui ka Model Context Protocol (MCP) Spring AI teeke.

##### Mudelikonteksti protokoll (MCP)

[Model Context Protocol (MCP)](https://modelcontextprotocol.io/) on tekkiv standard, mis võimaldab AI rakendustel turvaliselt suhelda väliste andmeallikate ja tööriistadega. MCP pakub standarditud viisi, kuidas AI mudelid saavad ligipääsu kontekstuaalsele infole ja täita sinu rakendustes toiminguid.

Peatükis 4 ehitad lihtsa MCP kalkulaatori teenuse, mis demonstreerib Mudelikonteksti protokolli aluseid koos Spring AI-ga, näidates, kuidas luua põhilisi tööriista integratsioone ja teenuse arhitektuure.

#### Azure OpenAI Java SDK

Azure OpenAI klienditeek Javale on OpenAI REST API-de kohandatud versioon, mis pakub idiomaatilist liidest ja integreerub ülejäänud Azure SDK ökosüsteemiga. Peatükis 3 ehitad rakendusi kasutades Azure OpenAI SDK-d, sealhulgas vestlusrakendusi, funktsioonikutsumist ja RAG (Retrieval-Augmented Generation) mustreid.

> Märkus: Azure OpenAI SDK on funktsionaalsuses OpenAI Java SDK-st maha jäänud, seega tulevaste projektide jaoks soovitame kaaluda OpenAI Java SDK kasutamist.

## Kokkuvõte

See lõpetab põhialused! Nüüd mõistad:

- Generatiivse AI põhikontseptsioone - alates LLM-idest ja prompti disainist kuni tokenite, manuste ja vektandmebaasideni
- Teie tööriistakomplekti valikud Java AI arenduseks: Azure OpenAI SDK, Spring AI ja OpenAI Java SDK
- Mis on Model Context Protocol ja kuidas see võimaldab AI agentidel töötada väliste tööriistadega

## Järgmised sammud

[Peatükk 2: Arenduskeskkonna seadistamine](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Lahtiütlus**:
See dokument on tõlgitud kasutades AI tõlketeenust [Co-op Translator](https://github.com/Azure/co-op-translator). Kuigi me püüdleme täpsuse poole, palun pange tähele, et automatiseeritud tõlgetes võib esineda vigu või ebatäpsusi. Originaaldokument selle emakeeles tuleks pidada autoriteetseks allikaks. Olulise teabe puhul soovitatakse kasutada professionaalset inimtõlget. Me ei vastuta selle tõlkega seotud eksimustest või valesti mõistmistest.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->