# Pagrindinis pokalbis su Azure AI Foundry – nuo pradžios iki pabaigos pavyzdys

Šis pavyzdys yra paprasta Spring Boot programa, kuri jungiasi prie **Azure AI Foundry** modelio naudodama **autentifikaciją be rakto** (Microsoft Entra ID) ir tikrina jūsų sąranką. Ji naudoja Spring AI `ChatClient`.

## Turinys

- [Prieš pradedant](#prieš-pradedant)
- [Greitas pradėjimas](#greitas-pradėjimas)
- [Kaip veikia autentifikacija](#kaip-veikia-autentifikacija)
- [Programos paleidimas](#programos-paleidimas)
  - [Naudojant Maven](#naudojant-maven)
  - [Naudojant VS Code](#naudojant-vs-code)
  - [Laukimas išvesties](#laukiama-išvestis)
- [Konfigūracijos nuoroda](#konfigūracijos-nuoroda)
  - [Aplinkos kintamieji](#aplinkos-kintamieji)
  - [Spring konfigūracija](#spring-konfigūracija)
- [Klaidų šalinimas](#klaidų-šalinimas)
  - [Dažnos problemos](#dažnos-problemos)
  - [Derinimo režimas](#derinimo-režimas)
- [Kiti žingsniai](#kiti-žingsniai)
- [Ištekliai](#ištekliai)

## Prieš pradedant

Prieš paleisdami šį pavyzdį, įsitikinkite, kad turite:

- Azure AI Foundry išteklių su `gpt-4o-mini` diegimu — įdiekite jį naudodami `azd up` arba rankiniu būdu per [Azure AI Foundry parengimo vadovą](../../getting-started-azure-openai.md)
- **Cognitive Services OpenAI User** vaidmenį tame išteklyje (jis automatiškai priskiriamas per Bicep šablonus)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), prisijungus naudodami `az login`
- Java 21+ ir Maven 3.9+

> **API raktas nereikalingas** — autentifikacija vykdoma be rakto per Microsoft Entra ID.

## Greitas pradėjimas

```bash
# 1. Pereikite prie projekto
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Prisijunkite, kad be rakto autentifikavimas galėtų gauti žetoną
az login

# 3. Konfigūruokite galinį tašką
#    - Jei vykdėte `azd up`, .env buvo parašytas už jus (praleiskite šį žingsnį).
#    - Priešingu atveju nukopijuokite šabloną ir nustatykite AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. Paleiskite programą
mvn spring-boot:run
```

## Kaip veikia autentifikacija

Šis pavyzdys autentifikuojasi su **Microsoft Entra ID** – API rakto nėra.

Kai nustatytas tik `spring.ai.azure.openai.endpoint` (ir nėra api-rakto), Spring AI sukuria Azure OpenAI klientą naudodama [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). Ši kredencialų tvarka automatiškai suranda tokeną iš jūsų vietinės `az login` sesijos arba iš valdomos tapatybės, kai veikia Azure aplinkoje – todėl tas pats kodas veikia abejose vietose be pakeitimų.

## Programos paleidimas

### Naudojant Maven

```bash
mvn spring-boot:run
```

### Naudojant VS Code

1. Atidarykite projektą VS Code
2. Paspauskite `F5` arba naudokite "Run and Debug" panelę
3. Pasirinkite "Spring Boot-BasicChatApplication" konfigūraciją

> **Pastaba**: VS Code konfigūracija automatiškai įkelia jūsų `.env` failą

### Laukiama išvestis

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

## Konfigūracijos nuoroda

### Aplinkos kintamieji

| Kintamasis | Aprašymas | Privalomas | Pavyzdys |
|------------|-----------|------------|----------|
| `AZURE_OPENAI_ENDPOINT` | Foundry (Azure OpenAI) galinio taško URL | Taip | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Pokalbio modelio diegimo pavadinimas | Ne | `gpt-4o-mini` (numatytasis) |

> Nėra **API rakto** kintamojo – autentifikacija vykdoma be rakto (Microsoft Entra ID per `az login`).

### Spring konfigūracija

`application.yml` faile konfigūruojama:
- **Endpoint**: `${AZURE_OPENAI_ENDPOINT}` – iš aplinkos kintamojo
- **Diegimas**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` – iš aplinkos kintamojo su numatytuoju
- **Autentifikacija**: be rakto – nėra nustatyto `api-key`, todėl Spring AI naudoja `DefaultAzureCredential`
- **Temperatūra**: `0.7` – valdo kūrybiškumą (0.0 = deterministinis, 1.0 = kūrybingas)
- **Maks. žetonai**: `500` – maksimalus atsakymo ilgis

## Klaidų šalinimas

### Dažnos problemos

<details>
<summary><strong>Klaida: 401 / "PermissionDenied" / tokeno klaidos</strong></summary>

- Paleiskite `az login` – autentifikacijai be rakto reikia aktyvaus prisijungimo tokenui gauti
- Patikrinkite, ar jūsų paskyra turi **Cognitive Services OpenAI User** vaidmenį tame išteklyje
- Jei ką tik priskyrėte vaidmenį, palaukite minutę, kol jis pritaikys korekcijas
- Patvirtinkite, kad esate tinkamame nuomininke/subscription (komanda `az account show`)
</details>

<details>
<summary><strong>Klaida: "Endpoint nėra galiojantis" / ryšio klaidos</strong></summary>

- Įsitikinkite, kad `AZURE_OPENAI_ENDPOINT` yra pilnas pagrindinis URL (pvz., `https://your-resource.openai.azure.com/`)
- Patikrinkite, ar nėra problemų su galiniu brūkšniu (/)
- Patikrinkite, ar galinis taškas atitinka jūsų paruoštą išteklių (`azd env get-values`)
</details>

<details>
<summary><strong>Klaida: "Diegimo nerasta"</strong></summary>

- Patikrinkite, ar `AZURE_OPENAI_DEPLOYMENT` atitinka diegimo pavadinimą Azure aplinkoje
- Įsitikinkite, kad modelis yra sėkmingai įdiegtas ir aktyvus
- Numatytoji diegimo pavadinimas yra `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: Aplinkos kintamieji neįkelti</strong></summary>

- Įsitikinkite, kad jūsų `.env` failas yra projekto šakniniame kataloge (lygyje su `pom.xml`)
- Pabandykite paleisti `mvn spring-boot:run` VS Code integruotoje terminalo aplinkoje
- Patikrinkite, ar VS Code Java plėtinys yra tinkamai įdiegtas
</details>

### Derinimo režimas

Norėdami įjungti išsamesnį žurnalavimą, atkomentuokite šias eilutes `application.yml` faile:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Kiti žingsniai

**Sąranka baigta!** Tęskite mokymosi kelią:

[3 skyrius: Pagrindinės generatyvios AI technikos](../../../03-CoreGenerativeAITechniques/README.md)

## Ištekliai

- [Spring AI Azure OpenAI dokumentacija](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Autentifikacija be rakto su Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry portalas](https://ai.azure.com/)
- [Azure AI Foundry dokumentacija](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Atsakomybės apribojimas**:
Šis dokumentas buvo išverstas naudojant dirbtinio intelekto vertimo paslaugą [Co-op Translator](https://github.com/Azure/co-op-translator). Nors siekiame tikslumo, prašome atkreipti dėmesį, kad automatiniai vertimai gali turėti klaidų ar netikslumų. Originalus dokumentas jo gimtąja kalba laikomas autoritetingu šaltiniu. Svarbiai informacijai rekomenduojama naudoti profesionalų žmogiškąjį vertimą. Mes neatsakome už jokius nesusipratimus ar neteisingą interpretaciją, kilusią naudojantis šiuo vertimu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->