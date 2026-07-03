# Praktinės programėlės ir projektai

## Ko išmoksite
Šioje skiltyje demonstruosime tris praktines programėles, kurios iliustruoja generatyvinio DI kūrimo modelius su Java:
- Sukurkite multimodalinį augintinio istorijos generatorių, derinant kliento ir serverio pusės DI
- Įgyvendinkite vietinę DI modelio integraciją su Foundry Local Spring Boot demo
- Sukurkite Modelio konteksto protokolo (MCP) paslaugą su skaičiuotuvo pavyzdžiu

## Turinys

- [Įvadas](#įvadas)
  - [Foundry Local Spring Boot demonstracija](#foundry-local-spring-boot-demonstracija)
  - [Augintinio istorijos generatorius](#augintinio-istorijos-generatorius)
  - [MCP skaičiuotuvo paslauga (Pradedančiųjų MCP demonstracija)](#mcp-skaičiuotuvo-paslauga-pradedančiųjų-mcp-demonstracija)
- [Mokymosi eiga](#mokymosi-eiga)
- [Santrauka](#santrauka)
- [Kiti žingsniai](#kiti-žingsniai)

## Įvadas

Šis skyrius pristato **pavyzdinius projektus**, kurie demonstruoja generatyvinio DI kūrimo modelius su Java. Kiekvienas projektas yra pilnai veikiantis ir iliustruoja specifines DI technologijas, architektūrinius modelius ir gerąsias praktikas, kurias galite pritaikyti savo programose.

### Foundry Local Spring Boot demonstracija

**[Foundry Local Spring Boot demonstracija](foundrylocal/README.md)** demonstruoja, kaip integruotis su vietiniais DI modeliais naudojant **OpenAI Java SDK**. Ji parodo, kaip prisijungti prie modelių, veikiančių Foundry Local (pvz., **Phi-4-mini**), su automatinio modelio aptikimu, leidžiant jums vykdyti DI programėles be debesijos paslaugų.

### Augintinio istorijos generatorius

**[Augintinio istorijos generatorius](petstory/README.md)** yra patraukli Spring Boot žiniatinklio programa, iliustruojanti **multimodalinį DI apdorojimą** kūrybingoms augintinių istorijoms generuoti. Ji derina kliento ir serverio pusės DI galimybes naudodama transformer.js naršyklės DI sąveikai ir OpenAI SDK serverio apdorojimui.

### MCP skaičiuotuvo paslauga (Pradedančiųjų MCP demonstracija)

**[MCP skaičiuotuvo paslauga](calculator/README.md)** yra paprasta **Modelio konteksto protokolo (MCP)** demonstracija, naudojant Spring AI. Ji suteikia pradedančiųjų lygio įvadą į MCP koncepcijas, parodant, kaip sukurti paprastą MCP serverį, bendraujantį su MCP klientais.

## Mokymosi eiga

Šie projektai sukurti taip, kad plėtotų idėjas iš ankstesnių skyrių:

1. **Pradėkite paprastai**: pradėkite nuo Foundry Local Spring Boot demonstracijos, kad suprastumėte pagrindinę DI integraciją su vietiniais modeliais
2. **Įtraukite sąveiką**: tęskite su Augintinio istorijos generatoriumi dėl multimodalinio DI ir žiniatinklio sąveikų
3. **Išmokite MCP pagrindus**: išbandykite MCP skaičiuotuvo paslaugą, kad suprastumėte Modelio konteksto protokolo pagrindus

## Santrauka

Puiku! Dabar ištyrėte keletą realių programėlių:

- Multimodalinę DI patirtį, veikiančią tiek naršyklėje, tiek serveryje
- Vietinę DI modelio integraciją, naudojant modernias Java sistemas ir SDK
- Pirmąją savo Modelio konteksto protokolo paslaugą, kad pamatytumėte, kaip įrankiai integruojasi su DI

## Kiti žingsniai

[5 skyrius: Atsakingas generatyvinis DI](../05-ResponsibleGenAI/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Atsakomybės apribojimas**:
Šis dokumentas buvo išverstas naudojant dirbtinio intelekto vertimo paslaugą [Co-op Translator](https://github.com/Azure/co-op-translator). Nors siekiame tikslumo, prašome atkreipti dėmesį, kad automatiniai vertimai gali turėti klaidų ar netikslumų. Originalus dokumentas jo gimtąja kalba laikomas autoritetingu šaltiniu. Svarbiai informacijai rekomenduojama naudoti profesionalų žmogiškąjį vertimą. Mes neatsakome už jokius nesusipratimus ar neteisingą interpretaciją, kilusią naudojantis šiuo vertimu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->