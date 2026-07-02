# Praktilised rakendused ja projektid

## Mida sa õpid
Selles osas demonstreerime kolme praktilist rakendust, mis tutvustavad generatiivse tehisintellekti arendusmustreid Java abil:
- Loo mitme modaalne lemmiklooma loo generaator, kombineerides kliendi- ja serveripoolset tehisintellekti
- Rakenda kohaliku tehisintellekti mudeli integratsioon Foundry Local Spring Boot demo abil
- Arenda Model Context Protocol (MCP) teenus kalkulaatori näitega

## Sisukord

- [Sissejuhatus](#sissejuhatus)
  - [Foundry Local Spring Boot Demo](#foundry-local-spring-boot-demo)
  - [Lemmiklooma loo generaator](#lemmiklooma-loo-generaator)
  - [MCP kalkulaatori teenus (alustajale MCP demo)](#mcp-kalkulaatori-teenus-alustajale-mcp-demo)
- [Õppimise järjestus](#õppimise-järjestus)
- [Kokkuvõte](#kokkuvõte)
- [Järgmised sammud](#järgmised-sammud)

## Sissejuhatus

See peatükk esitleb **näidisprojekte**, mis demonstreerivad generatiivse tehisintellekti arendusmustreid Java abil. Iga projekt on täielikult töövõimeline ja näitab spetsiifilisi tehisintellekti tehnoloogiaid, arhitektuurimustreid ja parimaid praktikaid, mida saad kohandada oma rakenduste jaoks.

### Foundry Local Spring Boot Demo

**[Foundry Local Spring Boot Demo](foundrylocal/README.md)** demonstreerib, kuidas integreerida kohalike tehisintellekti mudelitega kasutades **OpenAI Java SDK-d**. See näitab mudelite ühendamist, mis töötavad Foundry Local'is (nt **Phi-4-mini**), automaatse mudeli tuvastamisega, võimaldades sul käivitada tehisintellekti rakendusi ilma pilveteenustest sõltumata.

### Lemmiklooma loo generaator

**[Lemmiklooma loo generaator](petstory/README.md)** on kaasahaarav Spring Boot veebirakendus, mis demonstreerib **mitme modaalset tehisintellekti töötlemist**, et luua loomingulisi lemmiklooma lugusid. See ühendab kliendi- ja serveripoolsed tehisintellekti võimalused, kasutades transformer.js'i brauseripõhiseks AI interaktsiooniks ning OpenAI SDK-d serveripoolseks töötlemiseks.

### MCP kalkulaatori teenus (alustajale MCP demo)

**[MCP kalkulaatori teenus](calculator/README.md)** on lihtne näide **Model Context Protocol'i (MCP)** rakendamisest Spring AI abil. See pakub algajale sõbralikku sissejuhatust MCP kontseptsioonidesse, näidates, kuidas luua lihtne MCP server, mis suhtleb MCP klientidega.

## Õppimise järjestus

Need projektid on mõeldud eelnevate peatükkide kontseptsioonidele tuginedes:

1. **Alusta lihtsast**: alusta Foundry Local Spring Boot demonstreerimisest, et mõista kohalike mudelitega AI integreerimist
2. **Lisa interaktiivsust**: liigu edasi Lemmiklooma loo generaatori juurde, et kogeda mitme modaalset AI ja veebipõhist interaktsioonimist
3. **Õpi MCP põhitõdesid**: proovi MCP kalkulaatori teenust, et mõista Model Context Protocoli aluseid

## Kokkuvõte

Tubli töö! Oled nüüd uurinud päris rakendusi:

- Mitme modaalne tehisintellekti kogemus, mis töötab nii brauseris kui ka serveris
- Kohalike tehisintellekti mudelite integratsioon kaasaegsete Java raamistikute ja SDK-de abil
- Sinu esimene Model Context Protocol teenus, et näha, kuidas tööriistad AI-ga integreeruvad

## Järgmised sammud

[Peatükk 5: Vastutustundlik generatiivne tehisintellekt](../05-ResponsibleGenAI/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Lahtiütlus**:
See dokument on tõlgitud kasutades AI tõlketeenust [Co-op Translator](https://github.com/Azure/co-op-translator). Kuigi me püüdleme täpsuse poole, palun pange tähele, et automatiseeritud tõlgetes võib esineda vigu või ebatäpsusi. Originaaldokument selle emakeeles tuleks pidada autoriteetseks allikaks. Olulise teabe puhul soovitatakse kasutada professionaalset inimtõlget. Me ei vastuta selle tõlkega seotud eksimustest või valesti mõistmistest.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->