# Azure AI Foundry kūrimo aplinkos nustatymas

> Šiame vadove nustatomos **Azure AI Foundry** modeliai šio kurso Java AI programėlėms, naudojant **be rakto** autentifikavimą (Microsoft Entra ID) — nereikia tvarkyti API raktų. Naujas įrankių naudojime? Pradėkite nuo [kūrimo aplinkos vadovo](./README.md).

Šiame vadove nustatomos **Azure AI Foundry** modeliai šio kurso Java AI programėlėms. Turite du kelius:

- **A variantas — diegimas su `azd` + Bicep (rekomenduojama):** vienas komandos įvykdymas įdiegia Foundry paskyrą ir modelius kaip kodą. Nereikia naudoti portalo.
- **B variantas — ištekliai sukuriami rankiniu būdu** Azure AI Foundry portale.

Abi parinktys naudoja **be rakto autentifikavimą** (Microsoft Entra ID) — nereikia kopijuoti ar saugoti API raktų.

## Turinys

- [Kas sukuriama](#kas-sukuriama)
- [Prieš pradedant](#prieš-pradedant)
- [A variantas: diegimas su azd + Bicep (Rekomenduojama)](#option-a-provision-with-azd--bicep-recommended)
- [B variantas: išteklių kūrimas rankiniu būdu](#b-variantas-išteklių-kūrimas-rankiniu-būdu)
- [Jūsų aplinkos konfigūravimas](#jūsų-aplinkos-konfigūravimas)
- [Jūsų įrenginio testavimas](#jūsų-įrenginio-testavimas)
- [Kas toliau?](#kas-toliau)
- [Ištekliai](#ištekliai)
- [Papildomi ištekliai](#papildomi-ištekliai)

## Kas sukuriama

Bicep šablonai esančiame [`infra/`](../../../02-SetupDevEnvironment/infra) faile sukuria:

- **Azure AI Foundry** paskyrą (`Microsoft.CognitiveServices/accounts`, tipas `AIServices`) su projektu
- **pokalbio** diegimą — `gpt-4o-mini`
- **embedding** diegimą — `text-embedding-3-small` (naudojamas vėlesniuose skyriuose)
- **be rakto rolės priskyrimą** (`Cognitive Services OpenAI User`), kad galėtumėte prisijungti su `az login` vietoj rakto valdymo

## Prieš pradedant

- [Azure prenumerata](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) ir [Maven 3.9+](https://maven.apache.org/download.cgi)

## A variantas: diegimas su azd + Bicep (Rekomenduojama)

Iš `02-SetupDevEnvironment` aplanko:

```bash
cd 02-SetupDevEnvironment

# Prisijungti (abu įrankiai)
azd auth login
az login

# Paruošti Foundry paskyrą + modelių diegimus
azd up
```

`azd` paprašys įvesti **aplinkos pavadinimą** (pvz. `genai-java`) ir **regioną**. Pasirinkite regioną, kuriame yra `gpt-4o-mini` ir `text-embedding-3-small`, pavyzdžiui `eastus2` arba `swedencentral`.

Baigus diegimą, azd:

1. Įdiegia viską, kas aprašyta faile [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep).
2. Vykdo po diegimo skriptą, kuris įrašo [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) su jūsų galiniu tašku ir diegimų pavadinimais (be slaptų duomenų).

> **Patarimas:** Kartokite `azd up` bet kada, kai norite pritaikyti pakeitimus. Vykdykite `azd down`, kad ištrintumėte viską ir nutrauktumėte išlaidas.

Norėdami pamatyti sugeneruotus nustatymus:

```bash
azd env get-values
```

Dabar pereikite prie [Jūsų įrenginio testavimo](#jūsų-įrenginio-testavimas).

## B variantas: išteklių kūrimas rankiniu būdu

Mėgstate portalą? Sukurkite išteklius rankiniu būdu:

1. Nueikite į [Azure AI Foundry portalą](https://ai.azure.com/) ir prisijunkite.
2. **Sukurkite projektą** (tai taip pat sukuria AI Foundry resursą). Duokite jam pavadinimą, pvz., `GenAIJava`.
3. Projekto viduje atidarykite **Models + endpoints** → **Deploy model** → **Deploy base model**.
4. Įdiekite **gpt-4o-mini** (diegimo pavadinimas `gpt-4o-mini`). Jei norite naudoti embedding pavyzdžius, pakartokite su **text-embedding-3-small**.
5. Iš **Overview** nukopijuokite **endpoint** (pvz., `https://<resource>.openai.azure.com/`).
6. Suteikite sau be rakto prieigą: resurse atidarykite **Access control (IAM)** → **Add role assignment** → priskirkite sau rolę **Cognitive Services OpenAI User**.

> **Vis dar kyla problemų?** Peržiūrėkite [Azure AI Foundry dokumentaciją](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Jūsų aplinkos konfigūravimas

**Jeigu naudojote A variantą (`azd up`)**, jūsų nustatymų failas jau sukurtas — nieko papildomai konfigūruoti nereikia. Pereikite prie [Jūsų įrenginio testavimo](#jūsų-įrenginio-testavimas).

**Jeigu naudojote B variantą (rankiniu būdu)**, sukurkite pavyzdžio `.env` failą patys:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

Redaguokite `.env`, įrašydami savo endpoint (be rakto — autentifikacija be rakto):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Saugumo pastaba:** API rakto saugoti nereikia. Autentifikacija vyksta per Microsoft Entra ID su `az login` (vietoje) arba valdomą identitetą (Azure). `.env` faile laikomi tik neslaptieji nustatymai ir jis jau įtrauktas į `.gitignore`.

## Jūsų įrenginio testavimas

Įsitikinkite, kad esate prisijungę, kad be rakto autentifikacija gautų tokeną, tada paleiskite pavyzdį:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # jei dar nesate prisijungę
mvn clean spring-boot:run
```

Turėtumėte pamatyti atsakymą iš `gpt-4o-mini` modelio!

> **VS Code naudotojams:** Paspauskite `F5`, kad paleistumėte. Programėlė automatiškai užkraus `.env`.

> **Pilnas pavyzdys:** Žr. [Basic Chat su Azure AI Foundry pavyzdį](./examples/basic-chat-azure/README.md) su detaliais nurodymais ir trikčių šalinimu.

## Kas toliau?

**Nustatyta!** Dabar turite:
- Azure AI Foundry su diegtais `gpt-4o-mini` ir `text-embedding-3-small`
- Be rakto autentifikaciją (Microsoft Entra ID) — nereikia valdyti raktų
- Vietinį `.env` su jūsų galiniu tašku ir diegimų pavadinimais
- Paruoštą Java kūrimo aplinką

**Toliau tęskite į** [3 skyrių: Pagrindines Generatyvios AI technikas](../03-CoreGenerativeAITechniques/README.md) ir pradėkite kurti AI programas!

## Ištekliai

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Be rakto autentifikacija su Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry dokumentacija](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI dokumentacija](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Papildomi ištekliai

- [Atsisiųsti VS Code](https://code.visualstudio.com/Download)
- [Gauti Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Dev konteinerio konfigūracija](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Atsakomybės apribojimas**:
Šis dokumentas buvo išverstas naudojant dirbtinio intelekto vertimo paslaugą [Co-op Translator](https://github.com/Azure/co-op-translator). Nors siekiame tikslumo, prašome atkreipti dėmesį, kad automatiniai vertimai gali turėti klaidų ar netikslumų. Originalus dokumentas jo gimtąja kalba laikomas autoritetingu šaltiniu. Svarbiai informacijai rekomenduojama naudoti profesionalų žmogiškąjį vertimą. Mes neatsakome už jokius nesusipratimus ar neteisingą interpretaciją, kilusią naudojantis šiuo vertimu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->