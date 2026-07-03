# Peruskeskustelu Azure AI Foundryn kanssa – kokonaisvaltainen esimerkki

Tämä esimerkki on yksinkertainen Spring Boot -sovellus, joka yhdistää **Azure AI Foundry** -malliin käyttämällä **avaimetonta todennusta** (Microsoft Entra ID) ja testaa asetuksesi. Se käyttää Spring AI:n `ChatClient`-luokkaa.

## Sisällysluettelo

- [Esivaatimukset](#esivaatimukset)
- [Pika-aloitus](#pika-aloitus)
- [Miten todennus toimii](#miten-todennus-toimii)
- [Sovelluksen suorittaminen](#sovelluksen-suorittaminen)
  - [Mavenin käyttö](#mavenin-käyttö)
  - [VS Coden käyttö](#vs-coden-käyttö)
  - [Odotettu tuloste](#odotettu-tuloste)
- [Konfiguraatioiden viite](#konfiguraatioiden-viite)
  - [Ympäristömuuttujat](#ympäristömuuttujat)
  - [Spring-konfiguraatio](#spring-konfiguraatio)
- [Vianetsintä](#vianetsintä)
  - [Yleiset ongelmat](#yleiset-ongelmat)
  - [Vikailmoitukset](#vikailmoitukset)
- [Seuraavat askeleet](#seuraavat-askeleet)
- [Resurssit](#resurssit)

## Esivaatimukset

Ennen tämän esimerkin suorittamista varmista, että sinulla on:

- Azure AI Foundry -resurssi, jossa on `gpt-4o-mini` -käyttöönotto — ota se käyttöön komennolla `azd up` tai manuaalisesti [Azure AI Foundryn asennusohjeen](../../getting-started-azure-openai.md) avulla
- **Cognitive Services OpenAI User** -rooli kyseisessä resurssissa (Bicep-mallit asettavat tämän automaattisesti)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), kirjautuneena sisään komennolla `az login`
- Java 21+ ja Maven 3.9+

> **API-avainta ei vaadita** — todennus tapahtuu avaimettomasti Microsoft Entra ID:n kautta.

## Pika-aloitus

```bash
# 1. Siirry projektiin
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Kirjaudu sisään, jotta keyless-todennus voi saada tokenin
az login

# 3. Määritä päätepiste
#    - Jos kävit `azd up`, .env tiedosto on kirjoitettu puolestasi (hyppää tämä yli).
#    - Muussa tapauksessa kopioi mallipohja ja aseta AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. Suorita sovellus
mvn spring-boot:run
```

## Miten todennus toimii

Tässä esimerkissä todentaminen tapahtuu **Microsoft Entra ID:n** avulla — API-avainta ei tarvita.

Kun on asetettu vain `spring.ai.azure.openai.endpoint` (eikä api-avain), Spring AI rakentaa Azure OpenAI -asiakkaan käyttäen [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential) -todennustietoja. Tämä tunnistus hakee automaattisesti tokenin paikallisesta `az login` -sessionistasi tai hallitun identiteetin avulla, kun sovellus ajetaan Azure-ympäristössä — joten sama koodi toimii molemmissa ympäristöissä ilman muutoksia.

## Sovelluksen suorittaminen

### Mavenin käyttö

```bash
mvn spring-boot:run
```

### VS Coden käyttö

1. Avaa projekti VS Codessa
2. Paina `F5` tai käytä "Run and Debug" -paneelia
3. Valitse "Spring Boot-BasicChatApplication" -konfiguraatio

> **Huom:** VS Code -konfiguraatio lataa automaattisesti ympäristömuuttujat sisältävän .env -tiedoston

### Odotettu tuloste

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

## Konfiguraatioiden viite

### Ympäristömuuttujat

| Muuttuja | Kuvaus | Pakollinen | Esimerkki |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | Foundryn (Azure OpenAI) päätepisteen URL-osoite | Kyllä | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Chat-mallin käyttöönoton nimi | Ei | `gpt-4o-mini` (oletus) |

> API-avaimen muuttujaa **ei ole** — todennus on avaimetonta (Microsoft Entra ID `az login` -käytöllä).

### Spring-konfiguraatio

`application.yml`-tiedosto määrittelee:
- **Päätepisteen**: `${AZURE_OPENAI_ENDPOINT}` – ympäristömuuttujasta
- **Käyttöönoton**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` – ympäristömuuttujasta, oletuksella
- **Todennuksen**: avaimeton — ei asetettu `api-key`, joten Spring AI käyttää `DefaultAzureCredential`-luokkaa
- **Lämpötila**: `0.7` – ohjaa luovuutta (0.0 = deterministinen, 1.0 = luova)
- **Maksimi tokenit**: `500` – suurin vastauksen pituus

## Vianetsintä

### Yleiset ongelmat

<details>
<summary><strong>Virhe: 401 / "PermissionDenied" / token-virheet</strong></summary>

- Suorita `az login` — avaimeton todennus tarvitsee aktiivisen sisäänkirjautumisen saadakseen tokenin
- Varmista, että tililläsi on **Cognitive Services OpenAI User** -rooli kyseisessä resurssissa
- Jos rooli on juuri myönnetty, odota hetki, että se ehtii voimaan
- Tarkista, että olet oikeassa vuokralaisessa/tilaajassa (`az account show`)
</details>

<details>
<summary><strong>Virhe: "The endpoint is not valid" / yhteysvirheet</strong></summary>

- Varmista, että `AZURE_OPENAI_ENDPOINT` on koko perus-URL (esim. `https://your-resource.openai.azure.com/`)
- Tarkista, että osoitteen loppuosa (vinoviiva) on yhteneväinen
- Varmista, että osoite vastaa varattua resurssiasi (`azd env get-values`)
</details>

<details>
<summary><strong>Virhe: "The deployment was not found"</strong></summary>

- Varmista, että `AZURE_OPENAI_DEPLOYMENT` vastaa käyttöönoton nimeä Azuressa
- Tarkista, että malli on onnistuneesti otettu käyttöön ja aktiivinen
- Oletuskäyttöönoton nimi on `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: Ympäristömuuttujat eivät lataudu</strong></summary>

- Varmista, että .env-tiedosto on projektin juurihakemistossa (samalla tasolla kuin `pom.xml`)
- Kokeile suorittaa `mvn spring-boot:run` VS Coden integroidussa terminaalissa
- Tarkista, että VS Coden Java-laajennus on asennettu oikein
</details>

### Vikailmoitukset

Yksityiskohtaisen lokituksen käyttämiseksi ota nämä rivit käyttöön `application.yml`-tiedostossa:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Seuraavat askeleet

**Asennus valmis!** Jatka oppimismatkaasi:

[Luku 3: Keskeiset generatiivisen tekoälyn tekniikat](../../../03-CoreGenerativeAITechniques/README.md)

## Resurssit

- [Spring AI Azure OpenAI Dokumentaatio](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Avaimeton todennus Microsoft Entra ID:llä](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry Portaalin](https://ai.azure.com/)
- [Azure AI Foundry Dokumentaatio](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Vastuuvapauslauseke**:
Tämä asiakirja on käännetty käyttämällä tekoälypohjaista käännöspalvelua [Co-op Translator](https://github.com/Azure/co-op-translator). Vaikka pyrimme tarkkuuteen, otathan huomioon, että automaattiset käännökset saattavat sisältää virheitä tai epätarkkuuksia. Alkuperäinen asiakirja sen alkuperäiskielellä on virallinen lähde. Tärkeissä asioissa suositellaan ammattimaista ihmiskäännöstä. Emme ole vastuussa tämän käännöksen käytöstä aiheutuvista väärinymmärryksistä tai tulkinnoista.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->