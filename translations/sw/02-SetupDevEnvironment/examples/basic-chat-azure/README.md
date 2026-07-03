# Mazungumzo ya Msingi na Azure AI Foundry - Mfano wa Mwisho kwa Mwisho

Mfano huu ni programu rahisi ya Spring Boot inayounganisha na mfano wa **Azure AI Foundry** kwa kutumia **uthibitishaji bila funguo** (Microsoft Entra ID) na kujaribu usanidi wako. Inatumia `ChatClient` ya Spring AI.

## Jedwali la Yaliyomo

- [Mahitaji](#mahitaji)
- [Anza Haraka](#anza-haraka)
- [Jinsi Uthibitishaji Unavyofanya Kazi](#jinsi-uthibitishaji-unavyofanya-kazi)
- [Kukimbia Programu](#kukimbia-programu)
  - [Kutumia Maven](#kutumia-maven)
  - [Kutumia VS Code](#kutumia-vs-code)
  - [Matokeo Yanayotarajiwa](#matokeo-yanayotarajiwa)
- [Marejeleo ya Usanidi](#marejeleo-ya-usanidi)
  - [Mazingira ya Kazi](#mazingira-ya-kazi)
  - [Usanidi wa Spring](#usanidi-wa-spring)
- [Kutatua Matatizo](#kutatua-matatizo)
  - [Masuala ya Kawaida](#masuala-ya-kawaida)
  - [Hali ya Kurekebisha Makosa](#hali-ya-kurekebisha-makosa)
- [Hatua Zifuatazo](#hatua-zifuatazo)
- [Rasilimali](#rasilimali)

## Mahitaji

Kabla ya kukimbia mfano huu, hakikisha una:

- Rasilimali ya Azure AI Foundry yenye usambazaji wa `gpt-4o-mini` — itengeneze kwa kutumia `azd up` au kwa mkono kupitia [mwongozo wa usanidi wa Azure AI Foundry](../../getting-started-azure-openai.md)
- Nafasi ya **Cognitive Services OpenAI User** kwenye rasilimali hiyo (vigezo vya Bicep vinakupa hii)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), umeingia kwa `az login`
- Java 21+ na Maven 3.9+

> **Haina hitaji la funguo ya API** — uthibitishaji ni bila funguo kupitia Microsoft Entra ID.

## Anza Haraka

```bash
# 1. Elekea kwenye mradi
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Ingia ili uthibitishaji usiotumia funguo upate tokeni
az login

# 3. Sanidi mwisho wa huduma
#    - Ikiwa uliendesha `azd up`, .env ilianzishwa kwa ajili yako (ruka hii).
#    - Vinginevyo nakili kiolezo na weka AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. Endesha programu
mvn spring-boot:run
```

## Jinsi Uthibitishaji Unavyofanya Kazi

Mfano huu unathibitisha kwa kutumia **Microsoft Entra ID** — hakuna funguo ya API.

Wakati `spring.ai.azure.openai.endpoint` tu imewekwa (na hakuna api-key), Spring AI huunda mteja wa Azure OpenAI kwa kutumia [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). Kadi hiyo hujipata tokeni moja kwa moja kutoka kwa kikao chako cha `az login` kimoja kwa moja, au kutoka kwa kitambulisho kilichosimamiwa unapotumia Azure — hivyo msimbo huo unafanya kazi sehemu zote bila mabadiliko.

## Kukimbia Programu

### Kutumia Maven

```bash
mvn spring-boot:run
```

### Kutumia VS Code

1. Fungua mradi katika VS Code
2. Bonyeza `F5` au tumia jopo la "Run and Debug"
3. Chagua usanidi wa "Spring Boot-BasicChatApplication"

> **Kumbuka**: Usanidi wa VS Code hujipakiza faili lako la .env moja kwa moja

### Matokeo Yanayotarajiwa

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

## Marejeleo ya Usanidi

### Mazingira ya Kazi

| Kigezo | Maelezo | Kinahitajika | Mfano |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | Anuani ya Foundry (Azure OpenAI) | Ndio | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Jina la usambazaji wa mfano wa mazungumzo | Hapana | `gpt-4o-mini` (chaguo-msingi) |

> Hakuna kigezo cha funguo ya API — uthibitishaji ni bila funguo (Microsoft Entra ID kupitia `az login`).

### Usanidi wa Spring

Faili ya `application.yml` inaweka:
- **Anuani ya Huduma**: `${AZURE_OPENAI_ENDPOINT}` - Kutoka kwa kigezo cha mazingira
- **Usambazaji**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - Kutoka kwa kigezo cha mazingira na mbadala
- **Uthibitishaji**: bila funguo — hakuna `api-key` imewekwa, hivyo Spring AI inatumia `DefaultAzureCredential`
- **Joto**: `0.7` - Kudhibiti ubunifu (0.0 = uhakika, 1.0 = ubunifu)
- **Hadi Tokeni**: `500` - Urefu wa juu wa jibu

## Kutatua Matatizo

### Masuala ya Kawaida

<details>
<summary><strong>Kosa: 401 / "PermissionDenied" / makosa ya tokeni</strong></summary>

- Endesha `az login` — uthibitishaji bila funguo unahitaji kuingia kuweza kupata tokeni
- Hakikisha akaunti yako ina nafasi ya **Cognitive Services OpenAI User** kwenye rasilimali
- Ikiwa umeweka nafasi hiyo hivi karibuni, subiri dakika ili iweze kusambaa
- Thibitisha uko kwenye tena/usanidi sahihi (`az account show`)
</details>

<details>
<summary><strong>Kosa: "Anuani ya huduma si halali" / makosa ya muunganisho</strong></summary>

- Hakikisha `AZURE_OPENAI_ENDPOINT` ni URL kamili ya msingi (mfano, `https://your-resource.openai.azure.com/`)
- Angalia usawa wa mshale wa mwisho
- Thibitisha anuani hiyo inalingana na rasilimali uliyotengeneza (`azd env get-values`)
</details>

<details>
<summary><strong>Kosa: "Usambazaji haukupatikana"</strong></summary>

- Thibitisha `AZURE_OPENAI_DEPLOYMENT` ni jina la usambazaji linalopatikana Azure
- Angalia mfano umewekewa kazi na umeanzishwa
- Jina la usambazaji la chaguo-msingi ni `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: Mazingira ya kazi hayaendi kufululiza</strong></summary>

- Hakikisha faili yako ya `.env` iko kwenye mzizi wa mradi (kando na `pom.xml`)
- Jaribu kuendesha `mvn spring-boot:run` kwenye terminal ya VS Code
- Angalia kwamba kuongeza la Java la VS Code limewekwa vizuri
</details>

### Hali ya Kurekebisha Makosa

Ili kuwezesha kumbukumbu za kina, toa alama kutoka mistari hii katika `application.yml`:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Hatua Zifuatazo

**Usanidi Umekamilika!** Endelea na safari yako ya kujifunza:

[Sura ya 3: Mbinu za Msingi za AI za Uumbaji](../../../03-CoreGenerativeAITechniques/README.md)

## Rasilimali

- [Nyaraka za Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Uthibitishaji bila funguo na Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Portal ya Azure AI Foundry](https://ai.azure.com/)
- [Nyaraka za Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Kionyozo**:
Hati hii imetafsiriwa kwa kutumia huduma ya tafsiri ya AI [Co-op Translator](https://github.com/Azure/co-op-translator). Ingawa tunajitahidi kupata usahihi, tafadhali fahamu kwamba tafsiri za kiotomatiki zinaweza kuwa na makosa au upungufu wa usahihi. Hati ya asili katika lugha yake halisi inapaswa kuchukuliwa kama chanzo cha mamlaka. Kwa taarifa muhimu, tafsiri ya kitaalamu inayofanywa na binadamu inapendekezwa. Hatutojibu kwa kuelewa vibaya au tafsiri potofu zinazotokea kutokana na matumizi ya tafsiri hii.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->