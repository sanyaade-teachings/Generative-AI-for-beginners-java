# Praktické aplikácie a projekty

## Čo sa naučíte
V tejto časti si predvedieme tri praktické aplikácie, ktoré ukazujú vzory vývoja generatívnej AI v Jave:
- Vytvorte multimodálny generátor príbehov o domácich miláčikoch kombinujúci klientskú a serverovú AI
- Implementujte integráciu lokálneho AI modelu s ukážkou Foundry Local Spring Boot
- Vyvinúť službu Model Context Protocol (MCP) s príkladom kalkulačky

## Obsah

- [Úvod](#úvod)
  - [Foundry Local Spring Boot demo](#foundry-local-spring-boot-demo)
  - [Generátor príbehov o domácich miláčikoch](#generátor-príbehov-o-domácich-miláčikoch)
  - [MCP kalkulačková služba (začiatočnícka MCP ukážka)](#mcp-kalkulačková-služba-začiatočnícka-mcp-ukážka)
- [Postup učenia](#postup-ucenia)
- [Zhrnutie](#zhrnutie)
- [Ďalšie kroky](#ďalšie-kroky)

## Úvod

Táto kapitola predstavuje **príkladové projekty**, ktoré demonštrujú vzory vývoja generatívnej AI v Jave. Každý projekt je plne funkčný a ukazuje konkrétne AI technológie, architektonické vzory a osvedčené postupy, ktoré môžete prispôsobiť pre svoje vlastné aplikácie.

### Foundry Local Spring Boot demo

**[Foundry Local Spring Boot demo](foundrylocal/README.md)** demonštruje, ako integrovať lokálne AI modely pomocou **OpenAI Java SDK**. Ukazuje pripojenie k modelom bežiacim na Foundry Local (napr. **Phi-4-mini**) s automatickou detekciou modelu, čo vám umožní spúšťať AI aplikácie bez nutnosti spoliehať sa na cloudové služby.

### Generátor príbehov o domácich miláčikoch

**[Generátor príbehov o domácich miláčikoch](petstory/README.md)** je pútavá webová aplikácia Spring Boot, ktorá demonštruje **multimodálne spracovanie AI** pre generovanie kreatívnych príbehov o domácich miláčikoch. Kombinuje klientskú a serverovú AI schopnosť s využitím transformer.js pre AI interakcie v prehliadači a OpenAI SDK pre spracovanie na serveri.

### MCP kalkulačková služba (začiatočnícka MCP ukážka)

**[MCP kalkulačková služba](calculator/README.md)** je jednoduchou ukážkou **Model Context Protocol (MCP)** pomocou Spring AI. Poskytuje úvod pre začiatočníkov do konceptov MCP, ukazujúc, ako vytvoriť základný MCP server, ktorý komunikuje s MCP klientmi.

## Postup učenia

Tieto projekty sú navrhnuté tak, aby nadväzovali na koncepty z predchádzajúcich kapitol:

1. **Začnite jednoducho**: Začnite s Foundry Local Spring Boot demo, aby ste pochopili základnú integráciu AI s lokálnymi modelmi
2. **Pridajte interaktivitu**: Pokročte k generátoru príbehov o domácich miláčikoch pre multimodálnu AI a webové interakcie
3. **Naučte sa základy MCP**: Vyskúšajte MCP kalkulačkovú službu, aby ste pochopili základy Model Context Protocol

## Zhrnutie

Dobrý výkon! Teraz ste preskúmali niekoľko reálnych aplikácií:

- Multimodálne AI zážitky, ktoré fungujú v prehliadači aj na serveri
- Integráciu lokálneho AI modelu pomocou moderných Java rámcov a SDK
- Vašu prvú službu Model Context Protocol na pochopenie, ako nástroje integrujú AI

## Ďalšie kroky

[5. kapitola: Zodpovedná generatívna AI](../05-ResponsibleGenAI/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Vyhlásenie o zodpovednosti**:
Tento dokument bol preložený pomocou AI prekladateľskej služby [Co-op Translator](https://github.com/Azure/co-op-translator). Hoci sa snažíme o presnosť, vezmite prosím na vedomie, že automatické preklady môžu obsahovať chyby alebo nepresnosti. Pôvodný dokument v jeho natívnom jazyku by mal byť považovaný za autoritatívny zdroj. Pre kritické informácie sa odporúča profesionálny ľudský preklad. Nie sme zodpovední za žiadne nedorozumenia alebo nesprávne interpretácie vyplývajúce z použitia tohto prekladu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->