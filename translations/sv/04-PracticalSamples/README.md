# Praktiska tillämpningar & projekt

## Vad du kommer att lära dig
I detta avsnitt visar vi tre praktiska tillämpningar som demonstrerar utvecklingsmönster för generativ AI med Java:
- Skapa en multimodal Pet Story Generator som kombinerar klientbaserad och serverbaserad AI
- Implementera lokal AI-modellintegrering med Foundry Local Spring Boot-demo
- Utveckla en Model Context Protocol (MCP)-tjänst med kalkylatorexemplet

## Innehållsförteckning

- [Introduktion](#introduktion)
  - [Foundry Local Spring Boot-demo](#foundry-local-spring-boot-demo)
  - [Pet Story Generator](#pet-story-generator)
  - [MCP Calculator Service (Nybörjarvänlig MCP-demo)](#mcp-calculator-service-nybörjarvänlig-mcp-demo)
- [Lärande progression](#lärande-progression)
- [Sammanfattning](#sammanfattning)
- [Nästa steg](#nästa-steg)

## Introduktion

Detta kapitel visar **exempelprojekt** som demonstrerar utvecklingsmönster för generativ AI med Java. Varje projekt är fullt fungerarande och demonstrerar specifika AI-teknologier, arkitekturmönster och bästa praxis som du kan anpassa för dina egna applikationer.

### Foundry Local Spring Boot-demo

**[Foundry Local Spring Boot-demo](foundrylocal/README.md)** visar hur man integrerar med lokala AI-modeller med hjälp av **OpenAI Java SDK**. Den visar anslutning till modeller som körs på Foundry Local (t.ex. **Phi-4-mini**), med automatisk modelldetektering, vilket gör att du kan köra AI-applikationer utan att vara beroende av molntjänster.

### Pet Story Generator

**[Pet Story Generator](petstory/README.md)** är en engagerande Spring Boot-webbapplikation som demonstrerar **multimodal AI-behandling** för att generera kreativa djurhistorier. Den kombinerar klientbaserade och serverbaserade AI-funktioner med transformer.js för AI-interaktioner i webbläsaren och OpenAI SDK för serverbaserad bearbetning.

### MCP Calculator Service (Nybörjarvänlig MCP-demo)

**[MCP Calculator Service](calculator/README.md)** är en enkel demonstration av **Model Context Protocol (MCP)** med Spring AI. Den ger en nybörjarvänlig introduktion till MCP-koncept, där den visar hur man skapar en grundläggande MCP-server som interagerar med MCP-klienter.

## Lärande progression

Dessa projekt är utformade för att bygga vidare på koncept från tidigare kapitel:

1. **Börja enkelt**: Börja med Foundry Local Spring Boot-demo för att förstå grundläggande AI-integrering med lokala modeller
2. **Lägg till interaktivitet**: Fortsätt med Pet Story Generator för multimodal AI och webb-baserade interaktioner
3. **Lär dig MCP-grunder**: Prova MCP Calculator Service för att förstå grunderna i Model Context Protocol

## Sammanfattning

Bra jobbat! Du har nu utforskat några riktiga applikationer:

- Multimodala AI-upplevelser som fungerar både i webbläsaren och på servern
- Lokal AI-modellintegrering med moderna Java-ramverk och SDK:er
- Din första Model Context Protocol-tjänst för att se hur verktyg integreras med AI

## Nästa steg

[Kapitel 5: Ansvarsfull generativ AI](../05-ResponsibleGenAI/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Ansvarsfriskrivning**:
Detta dokument har översatts med hjälp av AI-översättningstjänsten [Co-op Translator](https://github.com/Azure/co-op-translator). Även om vi strävar efter noggrannhet, var vänlig notera att automatiska översättningar kan innehålla fel eller brister. Det ursprungliga dokumentet på dess modersmål bör betraktas som den auktoritativa källan. För kritisk information rekommenderas professionell mänsklig översättning. Vi ansvarar inte för några missförstånd eller feltolkningar som uppstår till följd av användningen av denna översättning.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->