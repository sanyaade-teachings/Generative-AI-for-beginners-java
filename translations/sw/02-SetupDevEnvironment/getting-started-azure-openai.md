# Kuweka Mazingira ya Maendeleo kwa Azure AI Foundry

> Mwongozo huu unaweka mifano ya **Azure AI Foundry** kwa programu za AI za Java katika kozi hii, ukitumia uthibitishaji wa **bila funguo** (Microsoft Entra ID) — hakuna funguo za API za kusimamia. Je, wewe ni mpya katika zana hizi? Anza na [mwongozo wa mazingira ya maendeleo](./README.md).

Mwongozo huu unaweka mifano ya **Azure AI Foundry** kwa programu za AI za Java katika kozi hii. Una njia mbili:

- **Chaguo A — Kuweka na `azd` + Bicep (inayopendekezwa):** amri moja inasambaza akaunti ya Foundry na mifano kama msimbo. Hakuna kubofya kwenye portal.
- **Chaguo B — Tengeneza rasilimali kwa mkono** katika portal ya Azure AI Foundry.

Njia zote mbili zinatumia **uthibitishaji bila funguo** (Microsoft Entra ID) — hakuna funguo za API za kunakili au kuvuja.

## Orodha ya Maudhui

- [Kile Kinachoundwa](#kile-kinachoundwa)
- [Mahitaji](#mahitaji)
- [Chaguo A: Kuweka na azd + Bicep (Inayopendekezwa)](#option-a-provision-with-azd--bicep-recommended)
- [Chaguo B: Tengeneza Rasilimali kwa Mkono](#chaguo-b-tengeneza-rasilimali-kwa-mkono)
- [Sanidi Mazingira Yako](#sanidi-mazingira-yako)
- [Jaribu Usanidi Wako](#jaribu-usanidi-wako)
- [Nini Kifuatayo?](#nini-kifuatayo)
- [Rasilimali](#rasilimali)
- [Rasilimali Zaidi](#rasilimali-zaidi)

## Kile Kinachoundwa

Mifano ya Bicep katika [`infra/`](../../../02-SetupDevEnvironment/infra) inatoa:

- Akaunti ya **Azure AI Foundry** (`Microsoft.CognitiveServices/accounts`, aina `AIServices`) yenye mradi
- Uwekaji wa **chat** — `gpt-4o-mini`
- Uwekaji wa **embedding** — `text-embedding-3-small` (inayotumiwa katika sura zijazo)
- Uteuzi wa jukumu la **bila funguo** (`Cognitive Services OpenAI User`) ili usajili ufanyike kwa `az login` badala ya kusimamia funguo

## Mahitaji

- [Usajili wa Azure](https://azure.microsoft.com/free/)
- [CLI ya Azure Developer (`azd`)](https://aka.ms/azure-dev/install)
- [CLI ya Azure (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) na [Maven 3.9+](https://maven.apache.org/download.cgi)

## Chaguo A: Kuweka na azd + Bicep (Inayopendekezwa)

Kutoka folda `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# Ingia (zana zote mbili)
azd auth login
az login

# Tayarisha akaunti ya Foundry + usambazaji wa mifano
azd up
```

`azd` itauliza kwa jina la **mazingira** (kwa mfano `genai-java`) na **eneo**. Chagua eneo ambapo `gpt-4o-mini` na `text-embedding-3-small` vinapatikana — kwa mfano `eastus2` au `swedencentral`.

Unapomaliza kuweka, azd:

1. Inasambaza kila kitu kilichobainishwa katika [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep).
2. Inaendesha hook ya baada ya kuweka inayotengeneza [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) na maelezo yako ya kiungo na majina ya uwekaji (bila siri).

> **Vidokezo:** Rudia `azd up` wakati wowote kufanyia mabadiliko. Tumia `azd down` kufuta kila kitu na kuacha kushindwa na gharama.

Ili kuona mipangilio iliyoundwa:

```bash
azd env get-values
```

Sasa ruka kwenda [Jaribu Usanidi Wako](#jaribu-usanidi-wako).

## Chaguo B: Tengeneza Rasilimali kwa Mkono

Unapendelea portal? Tengeneza rasilimali kwa mkono:

1. Nenda kwa [portal ya Azure AI Foundry](https://ai.azure.com/) na ingia.
2. **Tengeneza mradi** (hii pia inaweka rasilimali ya AI Foundry). Mpe jina kama `GenAIJava`.
3. Katika mradi wako, fungua **Models + endpoints** → **Deploy model** → **Deploy base model**.
4. Weka **gpt-4o-mini** (jina la uwekaji `gpt-4o-mini`). Rudia kwa **text-embedding-3-small** ikiwa unataka mifano ya embedding.
5. Kutoka **Overview**, nakili **kiunganishi** (kwa mfano `https://<resource>.openai.azure.com/`).
6. Jipa ufikiaji wa keyless: kwenye rasilimali, fungua **Access control (IAM)** → **Add role assignment** → panga **Cognitive Services OpenAI User** kwa akaunti yako.

> **Bado una shida?** Angalia [nyaraka za Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Sanidi Mazingira Yako

**Ikiwa ulitumia Chaguo A (`azd up`)**, faili lako la mipangilio tayari limeandikwa — hakuna cha kusanidi. Ruka kwenda [Jaribu Usanidi Wako](#jaribu-usanidi-wako).

**Ikiwa ulitumia Chaguo B (mkono)**, tengeneza faili ya `.env` ya mfano mwenyewe:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

Hariri `.env` na kiungo chako (bila funguo — uthibitisho ni bila funguo):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Kumbusho la usalama:** Hakuna funguo ya API ya kuhifadhi. Unathibitishwa na Microsoft Entra ID kupitia `az login` (kibinafsi) au umahiri ulioendeshwa (katika Azure). Faili ya `.env` inashikilia mipangilio isiyo na siri na tayari iko chini ya `.gitignore`.

## Jaribu Usanidi Wako

Hakikisha umeingia ili uthibitisho wa keyless upate tokeni, kisha endesha mfano:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # ikiwa bado hujajiandikisha
mvn clean spring-boot:run
```

Unapaswa kuona jibu kutoka kwa mfano wa `gpt-4o-mini`!

> **Watumiaji wa VS Code:** Bonyeza `F5` kuendesha. Programu inachukua `.env` yako moja kwa moja.

> **Mfano kamili:** Tazama [Mfano wa Mazungumzo ya Msingi na Azure AI Foundry](./examples/basic-chat-azure/README.md) kwa maelezo na kutatua matatizo.

## Nini Kifuatayo?

**Usanidi umekamilika!** Sasa una:
- Azure AI Foundry na `gpt-4o-mini` na `text-embedding-3-small` vimesambazwa
- Uthibitishaji bila funguo (Microsoft Entra ID) — hakuna funguo za kusimamia
- Faili la ndani `.env` lenye kiungo chako na majina ya uwekaji
- Mazingira ya maendeleo ya Java tayari kwa matumizi

**Endelea na** [Sura 3: Mbinu za AI Kuu ya Uzalishaji](../03-CoreGenerativeAITechniques/README.md) kuanza kujenga programu za AI!

## Rasilimali

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Uthibitishaji bila funguo na Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Nyaraka za Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)
- [Nyaraka za Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Rasilimali Zaidi

- [Pakua VS Code](https://code.visualstudio.com/Download)
- [Pata Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Mipangilio ya Dev Container](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Kionyozo**:
Hati hii imetafsiriwa kwa kutumia huduma ya tafsiri ya AI [Co-op Translator](https://github.com/Azure/co-op-translator). Ingawa tunajitahidi kupata usahihi, tafadhali fahamu kwamba tafsiri za kiotomatiki zinaweza kuwa na makosa au upungufu wa usahihi. Hati ya asili katika lugha yake halisi inapaswa kuchukuliwa kama chanzo cha mamlaka. Kwa taarifa muhimu, tafsiri ya kitaalamu inayofanywa na binadamu inapendekezwa. Hatutojibu kwa kuelewa vibaya au tafsiri potofu zinazotokea kutokana na matumizi ya tafsiri hii.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->