# Kehitysympäristön määrittäminen Azure AI Foundrylle

> Tämä opas määrittää **Azure AI Foundry** -mallit tämän kurssin Java AI -sovelluksille käyttäen **avainvapaata** todennusta (Microsoft Entra ID) — API-avaimia ei tarvitse hallita. Uusi työkaluihin? Aloita [kehitysympäristöoppaasta](./README.md).

Tämä opas määrittää **Azure AI Foundry** -mallit tämän kurssin Java AI -sovelluksille. Sinulla on kaksi vaihtoehtoa:

- **Vaihtoehto A — Provisionointi `azd`:llä + Bicepillä (suositeltu):** yksi komento ottaa käyttöön Foundry-tilin ja mallit koodina. Ei portaalin napsuttelua.
- **Vaihtoehto B — Luo resurssit manuaalisesti** Azure AI Foundry -portaalissa.

Molemmat reitit käyttävät **avainvapaata todennusta** (Microsoft Entra ID) — API-avaimia ei tarvitse kopioida tai vuotaa.

## Sisällysluettelo

- [Mitä luodaan](#mitä-luodaan)
- [Ennakkoedellytykset](#ennakkoedellytykset)
- [Vaihtoehto A: Provisionointi azd:llä + Bicepillä (suositeltu)](#vaihtoehto-a-provisionointi-azdllä--bicepillä-suositeltu)
- [Vaihtoehto B: Luo resurssit manuaalisesti](#vaihtoehto-b-luo-resurssit-manuaalisesti)
- [Ympäristön konfigurointi](#ympäristön-konfigurointi)
- [Testaa asennuksesi](#testaa-asennuksesi)
- [Mitä seuraavaksi?](#mitä-seuraavaksi)
- [Resurssit](#resurssit)
- [Lisäresurssit](#lisäresurssit)

## Mitä luodaan

Bicep-mallit hakemistossa [`infra/`](../../../02-SetupDevEnvironment/infra) provisioivat:

- **Azure AI Foundry** -tilin (`Microsoft.CognitiveServices/accounts`, tyyppi `AIServices`) projektilla
- Chatin käyttöönoton — `gpt-4o-mini`
- Upotuksen käyttöönoton — `text-embedding-3-small` (käytetään myöhemmissä luvuissa)
- Avainvapaan roolimäärityksen (`Cognitive Services OpenAI User`), jotta kirjaudut sisään `az login` -komennolla ilman avainten hallintaa

## Ennakkoedellytykset

- [Azure-tilaus](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) ja [Maven 3.9+](https://maven.apache.org/download.cgi)

## Vaihtoehto A: Provisionointi azd:llä + Bicepillä (suositeltu)

Kansiosta `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# Kirjaudu sisään (molemmat työkalut)
azd auth login
az login

# Tarjoa Foundry-tili + mallien käyttöönotot
azd up
```

`azd` pyytää **ympäristön nimeä** (esimerkiksi `genai-java`) ja **aluetta**. Valitse alue, jolla `gpt-4o-mini` ja `text-embedding-3-small` ovat saatavilla — esimerkiksi `eastus2` tai `swedencentral`.

Kun provisiointi on valmis, azd:

1. Ota käyttöön kaikki, mikä on määritelty tiedostossa [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep).
2. Suorittaa jälkiprovisiointikoukun, joka kirjoittaa tiedoston [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) päätepisteelläsi ja käyttöönoton nimillä (ei salaisuuksia).

> **Vinkki:** Suorita `azd up` uudelleen milloin tahansa muutosten soveltamiseksi. Suorita `azd down` poistaaksesi kaiken ja lopettaaksesi kustannukset.

Näyttääksesi luodut asetukset:

```bash
azd env get-values
```

Siirry nyt kohtaan [Testaa asennuksesi](#testaa-asennuksesi).

## Vaihtoehto B: Luo resurssit manuaalisesti

Haluatko mieluummin portaalin? Luo resurssit käsin:

1. Mene [Azure AI Foundry -portaaliin](https://ai.azure.com/) ja kirjaudu sisään.
2. **Luo projekti** (tämä myös luo AI Foundry -resurssin). Anna nimi, kuten `GenAIJava`.
3. Avaa projektissasi **Models + endpoints** → **Deploy model** → **Deploy base model**.
4. Ota käyttöön **gpt-4o-mini** (käyttöönoton nimi `gpt-4o-mini`). Toista **text-embedding-3-small** varten, jos haluat upotus-esimerkit.
5. Kopioi **endpoint** (esim. `https://<resource>.openai.azure.com/`) kohdasta **Overview**.
6. Myönnä itsellesi avainvapaa käyttöoikeus: resurssilla avaa **Access control (IAM)** → **Add role assignment** → määritä **Cognitive Services OpenAI User** tilillesi.

> **Onko ongelmia?** Katso [Azure AI Foundryn dokumentaatio](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Ympäristön konfigurointi

**Jos käytit Vaihtoehtoa A (`azd up`)**, asetustiedosto on jo kirjoitettu — mitään konfiguroitavaa ei ole. Siirry kohtaan [Testaa asennuksesi](#testaa-asennuksesi).

**Jos käytit Vaihtoehtoa B (manuaalinen)**, luo esimerkin `.env` itse:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

Muokkaa `.env`-tiedostoa päätepisteelläsi (ei avainta — todennus on avainvapaa):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Turvallisuushuomautus:** API-avainta ei ole talletettavana. Todennat Microsoft Entra ID:llä `az login` -komennolla (paikallisesti) tai hallinnoidulla identiteetillä (Azuressa). `.env`-tiedosto sisältää vain ei-salaisia asetuksia ja on jo jätetty huomioimatta `.gitignore`-tiedostolla.

## Testaa asennuksesi

Varmista, että olet kirjautunut sisään, jotta avainvapaa todennus saa tokenin, ja suorita esimerkki:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # jos et ole jo kirjautunut sisään
mvn clean spring-boot:run
```

Näet vastauksen `gpt-4o-mini`-mallilta!

> **VS Code -käyttäjille:** Paina `F5` suorittaaksesi. Sovellus lataa `.env`-tiedoston automaattisesti.

> **Täydellinen esimerkki:** Katso [Basic Chat with Azure AI Foundry -esimerkki](./examples/basic-chat-azure/README.md) yksityiskohtiin ja vianmääritykseen.

## Mitä seuraavaksi?

**Asennus valmis!** Sinulla on nyt:
- Azure AI Foundry käyttöön otettuna `gpt-4o-mini` ja `text-embedding-3-small`-malleilla
- Avainvapaa todennus (Microsoft Entra ID) — ei avaimia hallittavana
- Paikallinen `.env` päätepisteellä ja käyttöönoton nimillä
- Java-kehitysympäristö valmiina käytettäväksi

**Jatka kohtaan** [Luku 3: Keskeiset generatiivisen tekoälyn tekniikat](../03-CoreGenerativeAITechniques/README.md) aloittaaksesi tekoälysovellusten rakentamisen!

## Resurssit

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Avainvapaa todennus Microsoft Entra ID:llä](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry -dokumentaatio](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI -dokumentaatio](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Lisäresurssit

- [Lataa VS Code](https://code.visualstudio.com/Download)
- [Hanki Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Dev Container -konfiguraatio](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Vastuuvapauslauseke**:
Tämä asiakirja on käännetty käyttämällä tekoälypohjaista käännöspalvelua [Co-op Translator](https://github.com/Azure/co-op-translator). Vaikka pyrimme tarkkuuteen, otathan huomioon, että automaattiset käännökset saattavat sisältää virheitä tai epätarkkuuksia. Alkuperäinen asiakirja sen alkuperäiskielellä on virallinen lähde. Tärkeissä asioissa suositellaan ammattimaista ihmiskäännöstä. Emme ole vastuussa tämän käännöksen käytöstä aiheutuvista väärinymmärryksistä tai tulkinnoista.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->