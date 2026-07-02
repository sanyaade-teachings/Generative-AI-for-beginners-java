# Grundläggande chatt med Azure AI Foundry – steg-för-steg-exempel

Detta exempel är en enkel Spring Boot-applikation som ansluter till en **Azure AI Foundry**-modell med **nyckellös autentisering** (Microsoft Entra ID) och testar din uppsättning. Den använder Spring AI:s `ChatClient`.

## Innehållsförteckning

- [Förutsättningar](#förutsättningar)
- [Snabbstart](#snabbstart)
- [Hur autentisering fungerar](#hur-autentisering-fungerar)
- [Köra applikationen](#köra-applikationen)
  - [Använda Maven](#använda-maven)
  - [Använda VS Code](#använda-vs-code)
  - [Förväntad utdata](#förväntad-utdata)
- [Konfigurationsreferens](#konfigurationsreferens)
  - [Miljövariabler](#miljövariabler)
  - [Spring-konfiguration](#spring-konfiguration)
- [Felsökning](#felsökning)
  - [Vanliga problem](#vanliga-problem)
  - [Felsökningsläge](#felsökningsläge)
- [Nästa steg](#nästa-steg)
- [Resurser](#resurser)

## Förutsättningar

Innan du kör detta exempel, se till att du har:

- En Azure AI Foundry-resurs med en `gpt-4o-mini`-utplacering — skapa den med `azd up` eller manuellt via [Azure AI Foundry-uppstarts guiden](../../getting-started-azure-openai.md)
- Rollen **Cognitive Services OpenAI User** på den resursen (Bicep-mallarna tilldelar detta åt dig)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), inloggad med `az login`
- Java 21+ och Maven 3.9+

> **Ingen API-nyckel krävs** — autentisering sker nyckellöst via Microsoft Entra ID.

## Snabbstart

```bash
# 1. Navigera till projektet
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Logga in så att keyless auth kan få en token
az login

# 3. Konfigurera endpointen
#    - Om du körde `azd up`, skrevs .env för dig (hoppa över detta).
#    - Annars kopiera mallen och ange AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. Kör applikationen
mvn spring-boot:run
```

## Hur autentisering fungerar

Detta exempel autentiserar med **Microsoft Entra ID** — ingen API-nyckel används.

När endast `spring.ai.azure.openai.endpoint` är satt (och ingen api-nyckel), bygger Spring AI Azure OpenAI-klienten med [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). Denna credential hämtar automatiskt en token från din lokala `az login`-session eller från en hanterad identitet när den körs i Azure — så samma kod fungerar på båda ställena utan ändringar.

## Köra applikationen

### Använda Maven

```bash
mvn spring-boot:run
```

### Använda VS Code

1. Öppna projektet i VS Code  
2. Tryck på `F5` eller använd panelen "Run and Debug"  
3. Välj konfigurationen "Spring Boot-BasicChatApplication"

> **Observera**: VS Code-konfigurationen laddar automatiskt din .env-fil

### Förväntad utdata

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

## Konfigurationsreferens

### Miljövariabler

| Variabel | Beskrivning | Obligatorisk | Exempel |
|----------|-------------|--------------|---------|
| `AZURE_OPENAI_ENDPOINT` | Foundry (Azure OpenAI) endpoint-URL | Ja | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Namn på chattmodellens utplacering | Nej | `gpt-4o-mini` (standard) |

> Det finns **ingen** variabel för API-nyckel — autentisering är nyckellös (Microsoft Entra ID via `az login`).

### Spring-konfiguration

Filen `application.yml` konfigurerar:  
- **Endpoint**: `${AZURE_OPENAI_ENDPOINT}` - Från miljövariabel  
- **Deployment**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - Från miljövariabel med reservvärde  
- **Auth**: nyckellös — ingen `api-key` är satt, så Spring AI använder `DefaultAzureCredential`  
- **Temperature**: `0.7` - Styr kreativiteten (0.0 = deterministisk, 1.0 = kreativ)  
- **Max Tokens**: `500` - Maximal svarslängd  

## Felsökning

### Vanliga problem

<details>
<summary><strong>Fel: 401 / "PermissionDenied" / token-fel</strong></summary>

- Kör `az login` — nyckellös autentisering kräver aktiv inloggning för att få token  
- Kontrollera att ditt konto har rollen **Cognitive Services OpenAI User** på resursen  
- Om du nyligen tilldelade rollen, vänta en minut för att ändringen ska gälla  
- Kontrollera att du är i rätt tenant/abonnemang (`az account show`)  
</details>

<details>
<summary><strong>Fel: "The endpoint is not valid" / anslutningsfel</strong></summary>

- Säkerställ att `AZURE_OPENAI_ENDPOINT` är den fullständiga bas-URL:en (t.ex. `https://your-resource.openai.azure.com/`)  
- Kontrollera konsekvent snedstreck i slutet av URL:en  
- Verifiera att endpointen stämmer med din provisionerade resurs (`azd env get-values`)  
</details>

<details>
<summary><strong>Fel: "The deployment was not found"</strong></summary>

- Kontrollera att `AZURE_OPENAI_DEPLOYMENT` matchar ett utplaceringsnamn i Azure  
- Kontrollera att modellen är framgångsrikt utplacerad och aktiv  
- Standardnamnet för utplacering är `gpt-4o-mini`  
</details>

<details>
<summary><strong>VS Code: Miljövariabler laddas inte</strong></summary>

- Säkerställ att din `.env`-fil finns i projektets rotkatalog (samma nivå som `pom.xml`)  
- Prova att köra `mvn spring-boot:run` i VS Codes integrerade terminal  
- Kontrollera att Java-tillägget för VS Code är korrekt installerat  
</details>

### Felsökningsläge

För att aktivera detaljerad loggning, ta bort kommentaren från dessa rader i `application.yml`:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Nästa steg

**Installation slutförd!** Fortsätt din läranderesa:

[Kapitel 3: Kärntekniker för generativ AI](../../../03-CoreGenerativeAITechniques/README.md)

## Resurser

- [Spring AI Azure OpenAI-dokumentation](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)  
- [Nyckellös autentisering med Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)  
- [Azure AI Foundry-portal](https://ai.azure.com/)  
- [Azure AI Foundry-dokumentation](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Ansvarsfriskrivning**:
Detta dokument har översatts med hjälp av AI-översättningstjänsten [Co-op Translator](https://github.com/Azure/co-op-translator). Även om vi strävar efter noggrannhet, var vänlig notera att automatiska översättningar kan innehålla fel eller brister. Det ursprungliga dokumentet på dess modersmål bör betraktas som den auktoritativa källan. För kritisk information rekommenderas professionell mänsklig översättning. Vi ansvarar inte för några missförstånd eller feltolkningar som uppstår till följd av användningen av denna översättning.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->