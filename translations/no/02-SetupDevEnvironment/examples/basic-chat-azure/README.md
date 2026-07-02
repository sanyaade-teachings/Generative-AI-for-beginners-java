# Grunnleggende Chat med Azure AI Foundry - End-to-End Eksempel

Dette eksemplet er en enkel Spring Boot-applikasjon som kobler til en **Azure AI Foundry**-modell ved bruk av **nøkkelfri autentisering** (Microsoft Entra ID) og tester oppsettet ditt. Den bruker Spring AI sin `ChatClient`.

## Innholdsfortegnelse

- [Forutsetninger](#forutsetninger)
- [Rask Start](#rask-start)
- [Hvordan Autentisering Fungerer](#hvordan-autentisering-fungerer)
- [Kjøre Applikasjonen](#kjøre-applikasjonen)
  - [Bruke Maven](#bruke-maven)
  - [Bruke VS Code](#bruke-vs-code)
  - [Forventet Output](#forventet-output)
- [Konfigurasjonsreferanse](#konfigurasjonsreferanse)
  - [Miljøvariabler](#miljøvariabler)
  - [Spring Konfigurasjon](#spring-konfigurasjon)
- [Feilsøking](#feilsøking)
  - [Vanlige Problemer](#vanlige-problemer)
  - [Feilsøkingsmodus](#feilsøkingsmodus)
- [Neste Steg](#neste-steg)
- [Ressurser](#ressurser)

## Forutsetninger

Før du kjører dette eksemplet, forsikre deg om at du har:

- En Azure AI Foundry-ressurs med en `gpt-4o-mini` distribusjon — sett den opp med `azd up` eller manuelt via [Azure AI Foundry oppsettsveiledning](../../getting-started-azure-openai.md)
- Rollen **Cognitive Services OpenAI User** på denne ressursen (Bicep-malene tildeler denne for deg)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), innlogget med `az login`
- Java 21+ og Maven 3.9+

> **Ingen API-nøkkel kreves** — autentiseringen er nøkkelfri via Microsoft Entra ID.

## Rask Start

```bash
# 1. Naviger til prosjektet
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Logg inn slik at nøkkelfri autentisering kan få en token
az login

# 3. Konfigurer endepunktet
#    - Hvis du kjørte `azd up`, ble .env skrevet for deg (hopp over dette).
#    - Ellers kopier malen og sett AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. Kjør applikasjonen
mvn spring-boot:run
```

## Hvordan Autentisering Fungerer

Dette eksemplet autentiserer med **Microsoft Entra ID** — det kreves ikke API-nøkkel.

Når bare `spring.ai.azure.openai.endpoint` er satt (og ingen api-nøkkel), bygger Spring AI Azure OpenAI-klienten med [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). Den legitimasjonen finner automatisk en token fra din lokale `az login`-økt, eller fra en managed identity ved kjøring i Azure — så den samme koden fungerer på begge steder uten endringer.

## Kjøre Applikasjonen

### Bruke Maven

```bash
mvn spring-boot:run
```

### Bruke VS Code

1. Åpne prosjektet i VS Code
2. Trykk `F5` eller bruk "Run and Debug"-panelet
3. Velg "Spring Boot-BasicChatApplication" konfigurasjon

> **Merk**: VS Code-konfigurasjonen laster automatisk din .env-fil

### Forventet Output

```
Starting Basic Chat with Azure OpenAI...
Environment variables loaded successfully
Connecting to Azure OpenAI...
Sending prompt: What is AI in a short sentence? Max 100 words.

AI Response:
================
AI, or Artificial Intelligence, is the simulation of human intelligence in machines programmed to think and learn like humans.
================

Success! Azure OpenAI connection is working correctly.
```

## Konfigurasjonsreferanse

### Miljøvariabler

| Variabel | Beskrivelse | Påkrevd | Eksempel |
|----------|-------------|---------|----------|
| `AZURE_OPENAI_ENDPOINT` | Foundry (Azure OpenAI) endepunkt-URL | Ja | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Navn på chatmodell distribusjon | Nei | `gpt-4o-mini` (standard) |

> Det finnes **ingen** API-nøkkel variabel — autentisering er nøkkelfri (Microsoft Entra ID via `az login`).

### Spring Konfigurasjon

`application.yml`-filen konfigurerer:
- **Endepunkt**: `${AZURE_OPENAI_ENDPOINT}` - Fra miljøvariabel
- **Distribusjon**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - Fra miljøvariabel med fallback
- **Autentisering**: nøkkelfri — ingen `api-key` er satt, så Spring AI bruker `DefaultAzureCredential`
- **Temperatur**: `0.7` - Styrer kreativitet (0.0 = deterministisk, 1.0 = kreativ)
- **Maks tokens**: `500` - Maksimal svarlengde

## Feilsøking

### Vanlige Problemer

<details>
<summary><strong>Feil: 401 / "PermissionDenied" / tokenfeil</strong></summary>

- Kjør `az login` — nøkkelfri autentisering trenger en aktiv innlogging for å hente token
- Sjekk at kontoen din har rollen **Cognitive Services OpenAI User** på ressursen
- Hvis du nettopp har tildelt rollen, vent et minutt for at det skal slå igjennom
- Bekreft at du er i riktig tenant/abonnement (`az account show`)
</details>

<details>
<summary><strong>Feil: "The endpoint is not valid" / tilkoblingsfeil</strong></summary>

- Sørg for at `AZURE_OPENAI_ENDPOINT` er full base-URL (f.eks. `https://your-resource.openai.azure.com/`)
- Sjekk konsistens på skråstrek i slutten av URL
- Bekreft at endepunktet matcher din provisjonerte ressurs (`azd env get-values`)
</details>

<details>
<summary><strong>Feil: "The deployment was not found"</strong></summary>

- Sjekk at `AZURE_OPENAI_DEPLOYMENT` matcher et distribusjonsnavn i Azure
- Kontroller at modellen er vellykket distribuert og aktiv
- Standard distribusjonsnavn er `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: Miljøvariabler lastes ikke</strong></summary>

- Sørg for at `.env`-filen ligger i prosjektets rotmappe (samme nivå som `pom.xml`)
- Prøv å kjøre `mvn spring-boot:run` i VS Codes integrerte terminal
- Sjekk at VS Code Java-utvidelsen er riktig installert
</details>

### Feilsøkingsmodus

For å aktivere detaljert logging, fjern kommentaren på disse linjene i `application.yml`:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Neste Steg

**Oppsett Fullført!** Fortsett på læringsreisen:

[Kapittel 3: Kjerne Generative AI Teknikk](../../../03-CoreGenerativeAITechniques/README.md)

## Ressurser

- [Spring AI Azure OpenAI Dokumentasjon](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Nøkkelfri autentisering med Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry Portal](https://ai.azure.com/)
- [Azure AI Foundry Dokumentasjon](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Ansvarsfraskrivelse**:
Dette dokumentet er oversatt ved hjelp av AI-oversettelsestjenesten [Co-op Translator](https://github.com/Azure/co-op-translator). Selv om vi streber etter nøyaktighet, vær oppmerksom på at automatiske oversettelser kan inneholde feil eller unøyaktigheter. Det opprinnelige dokumentet på originalspråket skal betraktes som den autoritative kilden. For kritisk informasjon anbefales profesjonell menneskelig oversettelse. Vi er ikke ansvarlige for eventuelle misforståelser eller feiltolkninger som oppstår ved bruk av denne oversettelsen.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->