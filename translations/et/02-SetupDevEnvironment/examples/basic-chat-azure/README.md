# Põhiline vestlus Azure AI Foundry'ga - näide lõpust lõpuni

See näide on lihtne Spring Boot rakendus, mis ühendub **Azure AI Foundry** mudeliga, kasutades **võtmeta autentimist** (Microsoft Entra ID) ja testib teie seadistust. Kasutab Spring AI `ChatClient`i.

## Sisukord

- [Nõuded](#nõuded)
- [Kiire stardijuhend](#kiire-stardijuhend)
- [Kuidas autentimine töötab](#kuidas-autentimine-töötab)
- [Rakenduse käivitamine](#rakenduse-käivitamine)
  - [Maveni kasutamine](#maveni-kasutamine)
  - [VS Code'i kasutamine](#vs-codei-kasutamine)
  - [Oodatud väljund](#oodatud-väljund)
- [Konfiguratsioonijuhend](#konfiguratsioonijuhend)
  - [Keskkonnamuutujad](#keskkonnamuutujad)
  - [Springi konfiguratsioon](#springi-konfiguratsioon)
- [Probleemide lahendamine](#probleemide-lahendamine)
  - [Tavalised probleemid](#tavalised-probleemid)
  - [Silumisrežiim](#silumisrežiim)
- [Järgmised sammud](#järgmised-sammud)
- [Võimalused](#võimalused)

## Nõuded

Enne selle näite käivitamist veenduge, et teil on:

- Azure AI Foundry ressurss koos `gpt-4o-mini` juurutusega — looge see käsuga `azd up` või käsitsi via [Azure AI Foundry seadistamisjuhend](../../getting-started-azure-openai.md)
- Sellel ressursil **Cognitive Services OpenAI User** roll (Bicep mallid määravad selle teile automaatselt)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), sisse logitud käsuga `az login`
- Java 21+ ja Maven 3.9+

> **API-võtit ei nõuta** — autentimine toimub võtmeta Microsoft Entra ID kaudu.

## Kiire stardijuhend

```bash
# 1. Navigeeri projekti
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Logi sisse, et võtmeta autentimine saaks tokeni
az login

# 3. Konfigureeri lõpp-punkt
#    - Kui sa jooksutasid `azd up`, siis .env kirjutati sinu jaoks (loe see vahele).
#    - Vastasel juhul kopeeri mall ja määra AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. Käivita rakendus
mvn spring-boot:run
```

## Kuidas autentimine töötab

See näide autentib end **Microsoft Entra ID** abil — API-võtit ei kasutata.

Kui on määratud ainult `spring.ai.azure.openai.endpoint` (ja pole api-key'd), loob Spring AI Azure OpenAI kliendi [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential) abil. See mandaad automaatselt leiab teie kohalikust `az login` sessioonist tokeni või haldatu identiteedi kaudu, kui töötab Azure'is — nii toimib sama kood mõlemas keskkonnas ilma muudatusteta.

## Rakenduse käivitamine

### Maveni kasutamine

```bash
mvn spring-boot:run
```

### VS Code'i kasutamine

1. Avage projekt VS Code'is
2. Vajutage `F5` või kasutage paneeli "Run and Debug"
3. Valige konfiguratsioon "Spring Boot-BasicChatApplication"

> **Märkus**: VS Code konfiguratsioon laadib automaatselt teie .env faili

### Oodatud väljund

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

## Konfiguratsioonijuhend

### Keskkonnamuutujad

| Muutuja | Kirjeldus | Kohustuslik | Näide |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | Foundry (Azure OpenAI) lõpp-punkti URL | Jah | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Vestlusmudeli juurutuse nimi | Ei | `gpt-4o-mini` (vaikimisi) |

> API-võtme muutujat **puudub** — autentimine on võtmeta (Microsoft Entra ID kaudu `az login`).

### Springi konfiguratsioon

Fail `application.yml` seadistab:
- **Lõpp-punkt**: `${AZURE_OPENAI_ENDPOINT}` - Keskkonnamuutuja põhjal
- **Juurutus**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - Keskkonnamuutuja koos varuväärtusega
- **Autentimine**: võtmeta — pole `api-key` määratud, Spring AI kasutab `DefaultAzureCredential`i
- **Temperatuur**: `0.7` - Kontroller loovust (0.0 = deterministlik, 1.0 = loov)
- **Maksimaalne sümbolite arv**: `500` - Maksimaalne vastuse pikkus

## Probleemide lahendamine

### Tavalised probleemid

<details>
<summary><strong>Viga: 401 / "PermissionDenied" / tokeni vead</strong></summary>

- Käivitage `az login` — võtmeta autentimine vajab aktiivset sisselogimist tokeni saamiseks
- Kontrollige, et teie kontol oleks sellel ressursil **Cognitive Services OpenAI User** roll
- Kui just määrasite rolli, oodake minut, et see jõustuks
- Veenduge, et olete õiges tenant/ tellimuses (`az account show`)
</details>

<details>
<summary><strong>Viga: "Lõpp-punkt ei kehti" / ühenduse vead</strong></summary>

- Veenduge, et `AZURE_OPENAI_ENDPOINT` on täielik põhja-URL (nt `https://your-resource.openai.azure.com/`)
- Kontrollige, kas kaldkriips lõpus on järjekindel
- Kontrollige, et lõpp-punkt vastab loodud ressursile (`azd env get-values`)
</details>

<details>
<summary><strong>Viga: "Juurutus ei leitud"</strong></summary>

- Kontrollige, et `AZURE_OPENAI_DEPLOYMENT` vastab Azure juurutuse nimele
- Kontrollige, et mudel on edukalt juurutatud ja aktiivne
- Vaikimisi juurutuse nimi on `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: Keskkonnamuutujad ei laadi</strong></summary>

- Veenduge, et `.env` fail asub projekti juurkaustas (samas tasemel kui `pom.xml`)
- Proovige käivitada `mvn spring-boot:run` VS Code'i integreeritud terminalis
- Kontrollige, et VS Code Java laiendus on õigesti paigaldatud
</details>

### Silumisrežiim

Põhjaliku logimise lubamiseks kommenteerige read `application.yml` failis:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Järgmised sammud

**Seadistus on lõpetatud!** Jätkake oma õppeteekonda:

[3. peatükk: Tuumik generatiivse tehisintellekti tehnikad](../../../03-CoreGenerativeAITechniques/README.md)

## Võimalused

- [Spring AI Azure OpenAI dokumentatsioon](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Võtmeta autentimine Microsoft Entra ID-ga](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry portaal](https://ai.azure.com/)
- [Azure AI Foundry dokumentatsioon](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Lahtiütlus**:
See dokument on tõlgitud kasutades AI tõlketeenust [Co-op Translator](https://github.com/Azure/co-op-translator). Kuigi me püüdleme täpsuse poole, palun pange tähele, et automatiseeritud tõlgetes võib esineda vigu või ebatäpsusi. Originaaldokument selle emakeeles tuleks pidada autoriteetseks allikaks. Olulise teabe puhul soovitatakse kasutada professionaalset inimtõlget. Me ei vastuta selle tõlkega seotud eksimustest või valesti mõistmistest.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->