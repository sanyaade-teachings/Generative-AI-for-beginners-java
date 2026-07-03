# Grundlæggende Chat med Azure AI Foundry - End-to-End Eksempel

Dette eksempel er en simpel Spring Boot-applikation, der forbinder til en **Azure AI Foundry**-model ved hjælp af **keyless authentication** (Microsoft Entra ID) og tester din opsætning. Den bruger Spring AI's `ChatClient`.

## Indholdsfortegnelse

- [Forudsætninger](#forudsætninger)
- [Kom hurtigt i gang](#kom-hurtigt-i-gang)
- [Hvordan autentificering virker](#hvordan-autentificering-virker)
- [Kørsel af applikationen](#kørsel-af-applikationen)
  - [Brug af Maven](#brug-af-maven)
  - [Brug af VS Code](#brug-af-vs-code)
  - [Forventet output](#forventet-output)
- [Konfigurationsreference](#konfigurationsreference)
  - [Miljøvariabler](#miljøvariabler)
  - [Spring-konfiguration](#spring-konfiguration)
- [Fejlfinding](#fejlfinding)
  - [Almindelige problemer](#almindelige-problemer)
  - [Debug-tilstand](#debug-tilstand)
- [Næste skridt](#næste-skridt)
- [Ressourcer](#ressourcer)

## Forudsætninger

Før du kører dette eksempel, skal du sikre dig, at du har:

- En Azure AI Foundry-ressource med en `gpt-4o-mini`-udrulning — opret den med `azd up` eller manuelt via [Azure AI Foundry opsætningsvejledning](../../getting-started-azure-openai.md)
- Rollen **Cognitive Services OpenAI User** på den ressource (de Bicep-skabeloner tildeler denne til dig)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), logget ind med `az login`
- Java 21+ og Maven 3.9+

> **Ingen API-nøgle krævet** — autentificering sker keyless via Microsoft Entra ID.

## Kom hurtigt i gang

```bash
# 1. Naviger til projektet
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Log ind, så keyless auth kan få et token
az login

# 3. Konfigurer slutpunktet
#    - Hvis du kørte `azd up`, blev .env skrevet for dig (spring dette over).
#    - Ellers kopier skabelonen og sæt AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. Kør applikationen
mvn spring-boot:run
```

## Hvordan autentificering virker

Dette eksempel autentificerer med **Microsoft Entra ID** — der er ingen API-nøgle.

Når kun `spring.ai.azure.openai.endpoint` er sat (og ingen api-nøgle), bygger Spring AI Azure OpenAI-klienten med [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). Den credential finder automatisk en token fra din `az login` session lokalt eller fra en managed identity, når den kører i Azure — så den samme kode virker begge steder uden ændringer.

## Kørsel af applikationen

### Brug af Maven

```bash
mvn spring-boot:run
```

### Brug af VS Code

1. Åbn projektet i VS Code
2. Tryk på `F5` eller brug "Run and Debug"-panelet
3. Vælg "Spring Boot-BasicChatApplication" konfigurationen

> **Bemærk**: VS Code-konfigurationen indlæser automatisk din .env-fil

### Forventet output

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

## Konfigurationsreference

### Miljøvariabler

| Variabel | Beskrivelse | Krævet | Eksempel |
|----------|-------------|---------|----------|
| `AZURE_OPENAI_ENDPOINT` | Foundry (Azure OpenAI) endpoint URL | Ja | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Navn på chat model-udrulning | Nej | `gpt-4o-mini` (standard) |

> Der findes **ingen** API-nøgle-variabel — autentificering sker keyless (Microsoft Entra ID via `az login`).

### Spring-konfiguration

`application.yml`-filen konfigurerer:
- **Endpoint**: `${AZURE_OPENAI_ENDPOINT}` - Fra miljøvariabel
- **Deployment**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - Fra miljøvariabel med fallback
- **Auth**: keyless — ingen `api-key` sat, så Spring AI bruger `DefaultAzureCredential`
- **Temperature**: `0.7` - Kontrollerer kreativitet (0.0 = deterministisk, 1.0 = kreativ)
- **Max Tokens**: `500` - Maksimal svarlængde

## Fejlfinding

### Almindelige problemer

<details>
<summary><strong>Fejl: 401 / "PermissionDenied" / token fejl</strong></summary>

- Kør `az login` — keyless auth kræver aktiv login for at hente en token
- Bekræft, at din konto har rollen **Cognitive Services OpenAI User** på ressourcen
- Hvis du lige har tildelt rollen, vent et øjeblik for udbredelse
- Bekræft, at du er i det rigtige tenant/abonnement (`az account show`)
</details>

<details>
<summary><strong>Fejl: "The endpoint is not valid" / forbindelsesfejl</strong></summary>

- Sørg for, at `AZURE_OPENAI_ENDPOINT` er den fulde basis-URL (f.eks. `https://your-resource.openai.azure.com/`)
- Tjek konvention for afsluttende skråstreg
- Bekræft at endpoint matcher din udrullede ressource (`azd env get-values`)
</details>

<details>
<summary><strong>Fejl: "The deployment was not found"</strong></summary>

- Bekræft, at `AZURE_OPENAI_DEPLOYMENT` matcher et udrulningsnavn i Azure
- Tjek at modellen er succesfuldt udrullet og aktiv
- Standardnavnet for udrulning er `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: Miljøvariabler indlæses ikke</strong></summary>

- Sørg for, at din `.env`-fil er i projektets rodmappe (samme niveau som `pom.xml`)
- Prøv at køre `mvn spring-boot:run` i VS Codes integrerede terminal
- Tjek at VS Code Java-udvidelsen er korrekt installeret
</details>

### Debug-tilstand

For at aktivere detaljeret logning skal du afkommentere disse linjer i `application.yml`:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Næste skridt

**Opsætningen er fuldført!** Fortsæt din læringsrejse:

[Kapitel 3: Core Generative AI Techniques](../../../03-CoreGenerativeAITechniques/README.md)

## Ressourcer

- [Spring AI Azure OpenAI Dokumentation](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Keyless autentificering med Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry Portal](https://ai.azure.com/)
- [Azure AI Foundry Dokumentation](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Ansvarsfraskrivelse**:
Dette dokument er blevet oversat ved hjælp af AI-oversættelsestjenesten [Co-op Translator](https://github.com/Azure/co-op-translator). Selvom vi bestræber os på nøjagtighed, skal du være opmærksom på, at automatiserede oversættelser kan indeholde fejl eller unøjagtigheder. Det originale dokument på dets oprindelige sprog bør betragtes som den autoritative kilde. For kritisk information anbefales professionel menneskelig oversættelse. Vi påtager os intet ansvar for misforståelser eller fejltolkninger, der opstår som følge af brugen af denne oversættelse.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->