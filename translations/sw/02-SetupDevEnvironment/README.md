# Kuweka Mazingira ya Maendeleo kwa AI ya Uzalishaji kwa Java

> **Mwanzoni Haraka:** Tumia mifano yako ya AI kwenye **Azure AI Foundry** kama msimbo kwa Bicep + `azd` kwa dakika chache — ona [Mwongozo wa Kuweka Azure AI Foundry](getting-started-azure-openai.md). Uthibitishaji ni **bila funguo** (Microsoft Entra ID), hivyo hakuna funguo za API za kusimamia.

## Utajifunza Nini

- Kuandaa mazingira ya maendeleo ya Java kwa programu za AI
- Kuchagua na kusanidi mazingira yako unayopendelea ya maendeleo (awali wingu kwa Codespaces, kontena ya maendeleo ya ndani, au usanidi kamili wa ndani)
- Kuangalia usanidi wako kwa kuungana na mfano wa Azure AI Foundry

## Jedwali la Maudhui

- [Utajifunza Nini](#utajifunza-nini)
- [Utangulizi](#utangulizi)
- [Hatua ya 1: Andaa Mazingira Yako ya Maendeleo](#hatua-1-andaa-mazingira-yako-ya-maendeleo)
  - [Chaguo A: GitHub Codespaces (Inapendekezwa)](#chaguo-a-github-codespaces-inapendekezwa)
  - [Chaguo B: Kontena ya Maendeleo ya Ndani](#chaguo-b-kontena-ya-maendeleo-ya-ndani)
  - [Chaguo C: Tumia Usanidi Wako wa Ndani Uliopo](#chaguo-c-tumia-usanidi-wako-wa-ndani-uliopo)
- [Hatua ya 2: Tumia Azure AI Foundry](#hatua-2-tumia-azure-ai-foundry)
- [Hatua ya 3: Jaribu Usanidi Wako](#hatua-3-jaribu-usanidi-wako)
- [Matatizo na Suluhisho](#matatizo-na-suluhisho)
- [Muhtasari](#muhtasari)
- [Hatua Zifuatazo](#hatua-zifuatazo)

## Utangulizi

Sura hii itakuongoza kupitia kuandaa mazingira ya maendeleo. Tutatumia **Azure AI Foundry** kwa mifano yote katika kozi hii. Unatuma mifano kama msimbo kwa Bicep na Azure Developer CLI (`azd`), kisha unajiunga kwa **uthibitishaji bila funguo** (Microsoft Entra ID) — hakuna funguo za API za kunakili au kuvuja.

**Hakuna usanidi wa ndani unaohitajika!** Unaweza kutumia GitHub Codespaces, ambayo hutoa mazingira kamili ya maendeleo kwenye kivinjari chako, na kuitumia Foundry kutoka hapo.

Tunatumia **Azure AI Foundry** kwa kozi hii kwa sababu ni:
- **Iliyopangwa kama msimbo** — `azd up` moja hutoa akaunti na uenezaji wa mifano
- **Bila funguo** — thibitisha kwa kuingia kwako kwa Azure au utambulisho uliosimamiwa
- **Tayari kwa uzalishaji** — msimbo ule ule unafanya kazi ndani na Azure
- **Rahisi kubadilisha** — badilisha mifano kwa kubadilisha jina la uenezaji, si msimbo wako

> **Kumbuka**: Uenezaji wa Azure AI Foundry unalipishwa kwa tokeni (lipa unapotumia). Angalia [mwongozo wa kuanzisha Azure AI Foundry](getting-started-azure-openai.md) kwa maelezo ya utoaji, eneo, na gharama.


## Hatua 1: Andaa Mazingira Yako ya Maendeleo

<a name="quick-start-cloud"></a>

Tumetengeneza kontena la maendeleo lililosanidiwa tayari ili kupunguza muda wa kuanzisha na kuhakikisha una zana zote zinazohitajika kwa kozi hii ya AI ya Uzalishaji kwa Java. Chagua njia unayopendelea ya maendeleo:

### Chaguzi za Kuandaa Mazingira:

#### Chaguo A: GitHub Codespaces (Inapendekezwa)

**Anza kuandika msimbo kwa dakika 2 - hakuna usanidi wa ndani unaohitajika!**

1. Toa nakala ya hifadhi hii kwenye akaunti yako ya GitHub
   > **Kumbuka**: Ikiwa unataka kuhariri usanidi wa msingi tafadhali angalia [Sanidi la Kontena la Maendeleo](../../../.devcontainer/devcontainer.json)
2. Bofya **Code** → tabua **Codespaces** → **...** → **New with options...**
3. Tumia chaguo za kawaida – hii itachagua **usanii wa kontena la maendeleo**: **Mazenviro ya Maendeleo ya AI ya Uzalishaji kwa Java** iliyotengenezwa maalum kwa kozi hii
4. Bofya **Create codespace**
5. Subiri takriban dakika 2 hadi mazingira yawe tayari
6. Endelea na [Hatua ya 2: Tumia Azure AI Foundry](#hatua-2-tumia-azure-ai-foundry)

<img src="../../../translated_images/sw/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: Codespaces submenu" width="50%">

<img src="../../../translated_images/sw/image.833552b62eee7766.webp" alt="Screenshot: New with options" width="50%">

<img src="../../../translated_images/sw/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: Create codespace options" width="50%">


> **Faida za Codespaces**:
> - Hakuna usanidi wa ndani unaohitajika
> - Hufanya kazi kwenye kifaa chochote chenye kivinjari
> - Imesanidiwa kabla na zana zote na utegemezi
> - Saa 60 za bure kwa mwezi kwa akaunti binafsi
> - Mazingira thabiti kwa wote wanaojifunza

#### Chaguo B: Kontena ya Maendeleo ya Ndani

**Kwa waendelezaji wanaopendelea maendeleo ya ndani kwa kutumia Docker**

1. Toa nakala na uweke hifadhi hii kwenye mashine yako ya ndani
   > **Kumbuka**: Ikiwa unataka kuhariri usanidi wa msingi tafadhali angalia [Sanidi la Kontena la Maendeleo](../../../.devcontainer/devcontainer.json)
2. Sakinisha [Docker Desktop](https://www.docker.com/products/docker-desktop/) na [VS Code](https://code.visualstudio.com/)
3. Sakinisha [kiendelezi cha Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) katika VS Code
4. Fungua folda ya hifadhi katika VS Code
5. Unapoulizwa, bofya **Reopen in Container** (au tumia `Ctrl+Shift+P` → "Dev Containers: Reopen in Container")
6. Subiri kontena lijenge na kuanzisha
7. Endelea na [Hatua ya 2: Tumia Azure AI Foundry](#hatua-2-tumia-azure-ai-foundry)

<img src="../../../translated_images/sw/devcontainer.21126c9d6de64494.webp" alt="Screenshot: Dev container setup" width="50%">

<img src="../../../translated_images/sw/image-3.bf93d533bbc84268.webp" alt="Screenshot: Dev container build complete" width="50%">

#### Chaguo C: Tumia Usanidi Wako wa Ndani Uliopo

**Kwa waendelezaji wenye mazingira ya Java waliowekeza tayari**

Mambo yanayohitajika:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html)
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) au IDE unayopendelea

Hatua:
1. Nakili hifadhi hii kwenye mashine yako ya ndani
2. Fungua mradi katika IDE yako
3. Endelea na [Hatua ya 2: Tumia Azure AI Foundry](#hatua-2-tumia-azure-ai-foundry)

> **Vidokezo vya Mtaalamu**: Kama una mashine yenye uwezo mdogo lakini unahitaji VS Code kwa ndani, tumia GitHub Codespaces! Unaweza kuunganisha VS Code yako ya ndani na Codespace iliyopo kwenye wingu kwa faida za pande zote mbili.

<img src="../../../translated_images/sw/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: created local devcontainer instance" width="50%">


## Hatua 2: Tumia Azure AI Foundry

Tuma mifano ya AI ya kozi kwenye Azure AI Foundry kama msimbo. Kutoka kwenye mzizi wa hifadhi:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```

`azd` linauliza jina la mazingira na eneo, linatuma akaunti ya Azure AI Foundry yenye uenezaji wa `gpt-4o-mini` na `text-embedding-3-small`, na linaandika kiunganishi kwenye `.env` ya mfano — yote kwa uthibitishaji wa **bila funguo** (hakuna funguo za API).

> **Mwongozo kamili:** Angalia [Mwongozo wa Kuweka Azure AI Foundry](getting-started-azure-openai.md) kwa mahitaji, mbadala ya mkono (portal), mwongozo wa eneo, na maelezo ya gharama/usafishaji.

## Hatua 3: Jaribu Usanidi Wako

Mara mifano yako ya Foundry itakapowekwa, jaribu muunganisho kwa programu ya mfano katika [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure).

1. Fungua terminal katika mazingira yako ya maendeleo.
2. Elekea kwenye mfano:
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
3. Hakikisha umeingia (uthibitishaji bila funguo unahitaji tokeni):
   ```bash
   az login
   ```
   > Ikiwa ulifanya `azd up`, faili `.env` yenye kiunganishi tayari imeandikwa.
4. Endesha programu:
   ```bash
   mvn clean spring-boot:run
   ```

Unapaswa kuona majibu kutoka kwenye mfano wa `gpt-4o-mini`.

### Kuelewa Msimbo wa Mfano

Mfano chini ya `examples/basic-chat-azure` ni programu ya Spring Boot inayotumia **Spring AI** kuungana na Azure AI Foundry kwa uthibitishaji bila funguo.

**Msimbo huu hufanya:**
- **Kuunganisha** na Azure AI Foundry kwa kutumia kuingia kwako Azure (Microsoft Entra ID) — hakuna funguo za API
- **Kutuma** ombi kwa mfano wa `gpt-4o-mini`
- **Kupokea** na kuonyesha jibu la AI
- **Kukagua** kuwa usanidi wako unafanya kazi sawa

**Tegemezi Mwaka** (katika `pom.xml`):
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```

**Usanidi** (`application.yml`):
```yaml
spring:
  ai:
    azure:
      openai:
        # Endpoint only - no api-key. Spring AI uses DefaultAzureCredential (keyless).
        endpoint: ${AZURE_OPENAI_ENDPOINT}
        chat:
          options:
            deployment-name: ${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}
```

## Muhtasari

Vizuri! Sasa una kila kitu kimesanidiwa:

- Mifano ya Azure AI Foundry tumeituma kama msimbo kwa Bicep + `azd`
- Mazingira yako ya maendeleo ya Java yanaendesha (iwe ni Codespaces, kontena za maendeleo, au ndani)
- Umeunganisha na Azure AI Foundry kwa uthibitishaji bila funguo (Microsoft Entra ID) — hakuna funguo za API
- Umejaribu kila kitu kinafanya kazi kwa mfano rahisi unaozungumza na mfano wako

## Hatua Zifuatazo

[Sura ya 3: Mbinu za Msingi za AI ya Uzalishaji](../03-CoreGenerativeAITechniques/README.md)

## Matatizo na Suluhisho

Una matatizo? Hapa kuna matatizo ya kawaida na suluhisho:

- **Uthibitishaji unaoshindwa (401/403)?** 
  - Endesha `az login` — uthibitishaji ni bila funguo, hivyo lazima uingie
  - Hakikisha akaunti yako ina jukumu la **Cognitive Services OpenAI User** kwenye rasilimali
  - Ikiwa umeanza tu, subiri dakika ili usambazaji wa jukumu ufanyike

- **Maven haioniwi?** 
  - Ikiwa unatumia kontena za maendeleo/Codespaces, Maven inapaswa kuwa imewekwa
  - Kwa usanidi wa ndani, hakikisha Java 21+ na Maven 3.9+ zimesakinishwa
  - Jaribu `mvn --version` kuthibitisha usakinishaji

- **`azd` haioniwi au uteuzi unashindwa?** 
  - Sakinisha [Azure Developer CLI](https://aka.ms/azure-dev/install) na endesha `azd auth login`
  - Chagua eneo ambapo `gpt-4o-mini` ipo (k.m. `eastus2`)
  - Angalia [mwongozo wa kuanzisha Azure AI Foundry](getting-started-azure-openai.md) kwa maelezo

- **Kontena ya maendeleo haianzi?** 
  - Hakikisha Docker Desktop inaendeshwa (kwa maendeleo ya ndani)
  - Jaribu kujenga upya kontena: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **Makosa ya kukusanya programu?**
  - Hakikisha uko katika folda sahihi: `02-SetupDevEnvironment/examples/basic-chat-azure`
  - Jaribu kusafisha na kujenga tena: `mvn clean compile`

> **Unahitaji msaada?**: Bado unapata matatizo? Fungua tatizo kwenye hifadhi tutakusaidia.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Kionyozo**:
Hati hii imetafsiriwa kwa kutumia huduma ya tafsiri ya AI [Co-op Translator](https://github.com/Azure/co-op-translator). Ingawa tunajitahidi kupata usahihi, tafadhali fahamu kwamba tafsiri za kiotomatiki zinaweza kuwa na makosa au upungufu wa usahihi. Hati ya asili katika lugha yake halisi inapaswa kuchukuliwa kama chanzo cha mamlaka. Kwa taarifa muhimu, tafsiri ya kitaalamu inayofanywa na binadamu inapendekezwa. Hatutojibu kwa kuelewa vibaya au tafsiri potofu zinazotokea kutokana na matumizi ya tafsiri hii.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->