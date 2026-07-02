# Gyakorlati Alkalmazások és Projektek

## Amit Megtanulsz
Ebben a szakaszban három gyakorlati alkalmazást mutatunk be, amelyek generatív AI fejlesztési mintákat szemléltetnek Java nyelven:
- Többmodális Kisállat Történet Generátor létrehozása, amely kliens- és szerveroldali AI-t kombinál
- Helyi AI modell integráció megvalósítása a Foundry Local Spring Boot demóval
- Model Context Protocol (MCP) szolgáltatás fejlesztése a Kalkulátor példával

## Tartalomjegyzék

- [Bevezetés](#bevezetés)
  - [Foundry Local Spring Boot Demó](#foundry-local-spring-boot-demó)
  - [Kisállat Történet Generátor](#kisállat-történet-generátor)
  - [MCP Kalkulátor Szolgáltatás (Kezdőknek Szóló MCP Demó)](#mcp-kalkulátor-szolgáltatás-kezdőknek-szóló-mcp-demó)
- [Tanulási Folyamat](#tanulási-folyamat)
- [Összegzés](#összegzés)
- [Következő Lépések](#következő-lépések)

## Bevezetés

Ez a fejezet **példaprojekteket** mutat be, amelyek generatív AI fejlesztési mintákat szemléltetnek Java nyelven. Minden projekt teljes funkcionalitású, és konkrét AI technológiákat, architekturális mintákat és legjobb gyakorlatokat mutat be, amelyeket saját alkalmazásaidhoz igazíthatsz.

### Foundry Local Spring Boot Demó

A **[Foundry Local Spring Boot Demó](foundrylocal/README.md)** bemutatja, hogyan lehet integrálódni helyi AI modellekkel a **OpenAI Java SDK** segítségével. Megmutatja, hogyan lehet csatlakozni a Foundry Local-on futó modellekhez (pl. **Phi-4-mini**), automatikus modellfelismeréssel, így AI alkalmazásokat futtathatsz anélkül, hogy felhőszolgáltatásoktól függnél.

### Kisállat Történet Generátor

A **[Kisállat Történet Generátor](petstory/README.md)** egy lebilincselő Spring Boot webalkalmazás, amely a **többmodális AI feldolgozást** demonstrálja kreatív kisállat történetek generálására. Kombinálja a kliens- és szerveroldali AI képességeket, a transformer.js-t használva böngészőalapú AI interakciókhoz, valamint az OpenAI SDK-t a szerveroldali feldolgozáshoz.

### MCP Kalkulátor Szolgáltatás (Kezdőknek Szóló MCP Demó)

Az **[MCP Kalkulátor Szolgáltatás](calculator/README.md)** egy egyszerű bemutató a **Model Context Protocol (MCP)** használatáról Spring AI-val. Kezdőknek szóló bevezetést nyújt az MCP fogalmakba, megmutatva, hogyan lehet létrehozni egy alap MCP szervert, amely MCP kliensekkel kommunikál.

## Tanulási Folyamat

Ezek a projektek úgy vannak megtervezve, hogy az előző fejezetek fogalmaira építsenek:

1. **Kezdd egyszerűen**: Indulj a Foundry Local Spring Boot demóval a helyi modellekkel való alapvető AI integráció megértéséhez
2. **Adj interaktivitást**: Haladj tovább a Kisállat Történet Generátorral a többmodális AI és a webes interakciók elsajátításához
3. **Tanuld meg az MCP alapjait**: Próbáld ki az MCP Kalkulátor Szolgáltatást a Model Context Protocol alapfogalmainak megértéséhez

## Összegzés

Szép munka! Most már megismertél néhány valós alkalmazást:

- Többmodális AI élmények, amelyek a böngészőben és a szerveren is működnek
- Helyi AI modell integráció modern Java keretrendszerekkel és SDK-kkal
- Az első Model Context Protocol szolgáltatásodat, hogy lásd, hogyan integrálódnak az eszközök az AI-val

## Következő Lépések

[5. fejezet: Felelős generatív AI](../05-ResponsibleGenAI/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Jogi nyilatkozat**:
Ez a dokumentum az AI fordítási szolgáltatás, a [Co-op Translator](https://github.com/Azure/co-op-translator) segítségével készült. Bár az pontosságra törekszünk, kérjük, vegye figyelembe, hogy az automatikus fordítások hibákat vagy pontatlanságokat tartalmazhatnak. Az eredeti dokumentum az anyanyelvén tekintendő hiteles forrásnak. Fontos információk esetén professzionális emberi fordítást javasolunk. Nem vállalunk felelősséget semmilyen félreértésért vagy téves értelmezésért, amely ebből a fordításból ered.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->