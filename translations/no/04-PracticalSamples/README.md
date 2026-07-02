# Praktiske applikasjoner og prosjekter

## Hva du vil lære
I denne seksjonen vil vi demonstrere tre praktiske applikasjoner som viser generative AI-utviklingsmønstre med Java:
- Lage en multimodal Pet Story Generator som kombinerer klient-side og server-side AI
- Implementere lokal AI-modellintegrasjon med Foundry Local Spring Boot-demoen
- Utvikle en Model Context Protocol (MCP)-tjeneste med Calculator-eksempelet

## Innholdsfortegnelse

- [Introduksjon](#introduksjon)
  - [Foundry Local Spring Boot-demo](#foundry-local-spring-boot-demo)
  - [Pet Story Generator](#pet-story-generator)
  - [MCP Calculator Service (nybegynnervennlig MCP-demo)](#mcp-calculator-service-nybegynnervennlig-mcp-demo)
- [Læringsprogresjon](#læringsprogresjon)
- [Oppsummering](#oppsummering)
- [Neste steg](#neste-steg)

## Introduksjon

Dette kapittelet viser **eksempelsprosjekter** som demonstrerer generative AI-utviklingsmønstre med Java. Hvert prosjekt er fullt funksjonelt og demonstrerer spesifikke AI-teknologier, arkitekturmønstre og beste praksis som du kan tilpasse for dine egne applikasjoner.

### Foundry Local Spring Boot-demo

**[Foundry Local Spring Boot-demo](foundrylocal/README.md)** viser hvordan man integrerer med lokale AI-modeller ved hjelp av **OpenAI Java SDK**. Den demonstrerer tilkobling til modeller som kjører på Foundry Local (f.eks. **Phi-4-mini**), med automatisk modellgjenkjenning, slik at du kan kjøre AI-applikasjoner uten å være avhengig av skytjenester.

### Pet Story Generator

**[Pet Story Generator](petstory/README.md)** er en engasjerende Spring Boot-nettapp som demonstrerer **multimodal AI-behandling** for å generere kreative kjæledyrhistorier. Den kombinerer klient-side og server-side AI-funksjonalitet ved bruk av transformer.js for nettleserbaserte AI-interaksjoner og OpenAI SDK for server-side prosessering.

### MCP Calculator Service (nybegynnervennlig MCP-demo)

**[MCP Calculator Service](calculator/README.md)** er en enkel demonstrasjon av **Model Context Protocol (MCP)** ved bruk av Spring AI. Den gir en nybegynnervennlig introduksjon til MCP-konsepter og viser hvordan man lager en grunnleggende MCP-server som samhandler med MCP-klienter.

## Læringsprogresjon

Disse prosjektene er designet for å bygge videre på konsepter fra tidligere kapitler:

1. **Start enkelt**: Begynn med Foundry Local Spring Boot-demoen for å forstå grunnleggende AI-integrasjon med lokale modeller
2. **Legg til interaktivitet**: Gå videre til Pet Story Generator for multimodal AI og nettbaserte interaksjoner
3. **Lær MCP-grunnleggende**: Prøv MCP Calculator Service for å forstå fundamentale MCP-prinsipper

## Oppsummering

Bra jobbet! Du har nå utforsket noen ekte applikasjoner:

- Multimodale AI-opplevelser som fungerer både i nettleseren og på serveren
- Lokal AI-modellintegrasjon ved bruk av moderne Java-rammeverk og SDK-er
- Din første Model Context Protocol-tjeneste for å se hvordan verktøy integreres med AI

## Neste steg

[Kapittel 5: Ansvarlig generativ AI](../05-ResponsibleGenAI/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Ansvarsfraskrivelse**:
Dette dokumentet er oversatt ved hjelp av AI-oversettelsestjenesten [Co-op Translator](https://github.com/Azure/co-op-translator). Selv om vi streber etter nøyaktighet, vær oppmerksom på at automatiske oversettelser kan inneholde feil eller unøyaktigheter. Det opprinnelige dokumentet på originalspråket skal betraktes som den autoritative kilden. For kritisk informasjon anbefales profesjonell menneskelig oversettelse. Vi er ikke ansvarlige for eventuelle misforståelser eller feiltolkninger som oppstår ved bruk av denne oversettelsen.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->