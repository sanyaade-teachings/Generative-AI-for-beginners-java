# Käytännön sovellukset ja projektit

## Mitä opit
Tässä osiossa demonstroidaan kolme käytännön sovellusta, jotka esittelevät generatiivisen tekoälyn kehitysmalleja Javalla:
- Luo monimodaalinen lemmikkitarinageneraattori yhdistämällä asiakas- ja palvelinpuolen tekoäly
- Toteuta paikallinen tekoälymallin integrointi Foundry Local Spring Boot -demolla
- Kehitä Model Context Protocol (MCP) -palvelu laskin-esimerkillä

## Sisällysluettelo

- [Johdanto](#johdanto)
  - [Foundry Local Spring Boot Demo](#foundry-local-spring-boot-demo)
  - [Lemmikkitarinageneraattori](#lemmikkitarinageneraattori)
  - [MCP Calculator Service (aloittelijaystävällinen MCP-demo)](#mcp-calculator-service-aloittelijaystävällinen-mcp-demo)
- [Oppimiskulku](#oppimiskulku)
- [Yhteenveto](#yhteenveto)
- [Seuraavat askeleet](#seuraavat-askeleet)

## Johdanto

Tämä luku esittelee **näyteprojekteja**, jotka demonstroivat generatiivisen tekoälyn kehitysmalleja Javalla. Jokainen projekti on täysin toimiva ja esittelee tiettyjä tekoälyteknologioita, arkkitehtuurimalleja ja parhaita käytäntöjä, joita voit soveltaa omissa sovelluksissasi.

### Foundry Local Spring Boot Demo

**[Foundry Local Spring Boot Demo](foundrylocal/README.md)** näyttää, miten integroidutaan paikallisiin tekoälymalleihin käyttämällä **OpenAI Java SDK:ta**. Se esittelee yhteyden muodostamista malleihin, jotka pyörivät Foundry Localissa (esim. **Phi-4-mini**), automaattisella mallintunnistuksella, mikä mahdollistaa tekoälysovellusten ajamisen ilman pilvipalveluihin tukeutumista.

### Lemmikkitarinageneraattori

**[Lemmikkitarinageneraattori](petstory/README.md)** on mukaansatempaava Spring Boot -verkkosovellus, joka demonstroi **monimodaalista tekoälykäsittelyä** luovan lemmikkitarinan tuottamiseksi. Se yhdistää asiakas- ja palvelinpuolen tekoälyominaisuudet käyttäen transformer.js-kirjastoa selaimessa toimiviin tekoälytoimintoihin ja OpenAI SDK:ta palvelinpuolen käsittelyyn.

### MCP Calculator Service (aloittelijaystävällinen MCP-demo)

**[MCP Calculator Service](calculator/README.md)** on yksinkertainen demonstraatio **Model Context Protocolista (MCP)** Spring AI:ta käyttäen. Se tarjoaa aloittelijaystävällisen johdannon MCP-käsitteisiin näyttäen, miten luodaan perus MCP-palvelin, joka kommunikoi MCP-asiakkaiden kanssa.

## Oppimiskulku

Nämä projektit on suunniteltu rakentamaan aiempien lukujen käsitteiden varaan:

1. **Aloita yksinkertaisesti**: Aloita Foundry Local Spring Boot Demolla ymmärtääksesi paikallisen mallin perusin­tegraation
2. **Lisää vuorovaikutteisuutta**: Siirry Lemmikkitarinageneraattoriin monimodaalisen tekoälyn ja verkkopohjaisten vuorovaikutusten parissa
3. **Opiskele MCP:n perusteet**: Kokeile MCP Calculator Servicea ymmärtääksesi Model Context Protocolin perusteet

## Yhteenveto

Hyvin tehty! Olet nyt tutustunut todellisiin sovelluksiin:

- Monimodaaliset tekoälykokemukset, jotka toimivat sekä selaimessa että palvelimella
- Paikallinen tekoälymallin integrointi nykyaikaisilla Java-kehyksillä ja SDK:lla
- Ensimmäinen Model Context Protocol -palvelusi, joka näyttää työkalujen integroinnin tekoälyyn

## Seuraavat askeleet

[Luku 5: Vastuullinen generatiivinen tekoäly](../05-ResponsibleGenAI/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Vastuuvapauslauseke**:
Tämä asiakirja on käännetty käyttämällä tekoälypohjaista käännöspalvelua [Co-op Translator](https://github.com/Azure/co-op-translator). Vaikka pyrimme tarkkuuteen, otathan huomioon, että automaattiset käännökset saattavat sisältää virheitä tai epätarkkuuksia. Alkuperäinen asiakirja sen alkuperäiskielellä on virallinen lähde. Tärkeissä asioissa suositellaan ammattimaista ihmiskäännöstä. Emme ole vastuussa tämän käännöksen käytöstä aiheutuvista väärinymmärryksistä tai tulkinnoista.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->