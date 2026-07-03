# Kehitysympäristön perustaminen Generative AI:ta varten Java-kielellä

> **Pika-aloitus:** Ota AI-mallit käyttöön **Azure AI Foundryssa** koodina Bicepillä + `azd`:llä muutamassa minuutissa — katso [Azure AI Foundry -aloitusopas](getting-started-azure-openai.md). Todennus on **avaimetonta** (Microsoft Entra ID), joten hallittavaa API-avainta ei ole.

## Mitä Opit

- Java-kehitysympäristön pystyttäminen AI-sovelluksia varten
- Kehitysympäristön valinta ja konfigurointi (pilvipohjainen Codespaces, paikallinen dev container tai täydellinen paikallinen asennus)
- Asetuksen testaaminen yhdistämällä Azure AI Foundry -malliin

## Sisällysluettelo

- [Mitä Opit](#mitä-opit)
- [Johdanto](#johdanto)
- [Vaihe 1: Kehitysympäristön Pystyttäminen](#vaihe-1-kehitysympäristön-pystyttäminen)
  - [Vaihtoehto A: GitHub Codespaces (Suositeltu)](#vaihtoehto-a-github-codespaces-suositeltu)
  - [Vaihtoehto B: Paikallinen Dev Container](#vaihtoehto-b-paikallinen-dev-container)
  - [Vaihtoehto C: Käytä Olemassa Olevaa Paikallista Asennustasi](#vaihtoehto-c-käytä-olemassa-olevaa-paikallista-asennustasi)
- [Vaihe 2: Azure AI Foundryn Käyttöönotto](#vaihe-2-azure-ai-foundryn-käyttöönotto)
- [Vaihe 3: Testaa Asetuksesi](#vaihe-3-testa-asetuksesi)
- [Vianetsintä](#vianetsintä)
- [Yhteenveto](#yhteenveto)
- [Seuraavat Askeleet](#seuraavat-askellet)

## Johdanto

Tässä luvussa ohjaamme sinua läpi kehitysympäristön perustamisen. Käytämme koko kurssin ajan **Azure AI Foundrya** malleihin. Otat mallit käyttöön koodina Bicepillä ja Azure Developer CLI:llä (`azd`), ja yhdistät **avaimettomalla todennuksella** (Microsoft Entra ID) — ei kopioitavia tai vuotavia API-avaimia.

**Paikallista asennusta ei tarvita!** Voit käyttää GitHub Codespacesia, joka tarjoaa täyden kehitysympäristön suoraan selaimessasi ja ottaa Foundryn käyttöön sieltä.

Käytämme tätä kurssia varten **Azure AI Foundrya**, koska se on:
- **Koodina käyttöön otettava** — yksi `azd up` ottaa käyttöön tilin ja mallin käyttöönotot
- **Avaimeton** — kirjaudu Azure-tunnuksellasi tai hallitulla identiteetillä
- **Tuotantovalmiina** — sama koodi toimii paikallisesti ja Azuressa
- **Joustava** — vaihda mallia muuttamalla käyttöönoton nimeä, ei koodia

> **Huomautus**: Azure AI Foundryn käyttöönotosta veloitetaan token-pohjaisesti (pay-as-you-go). Katso [Azure AI Foundryn käyttöönotto-opas](getting-started-azure-openai.md) käyttöönottovaiheet, aluevalinnat ja kustannustiedot.

## Vaihe 1: Kehitysympäristön Pystyttäminen

<a name="quick-start-cloud"></a>

Olemme luoneet valmiiksi konfiguroidun kehityskontin, joka minimoi asetusaikaa ja varmistaa, että sinulla on kaikki tarvittavat työkalut tälle Generative AI for Java -kurssille. Valitse haluamasi kehitystapa:

### Ympäristön pystytysvaihtoehdot:

#### Vaihtoehto A: GitHub Codespaces (Suositeltu)

**Aloita koodaus 2 minuutissa – ei paikallista asennusta!**

1. Forkkaa tämä repositorio GitHub-tilillesi  
   > **Huom:** Jos haluat muokata perusasetuksia, tutustu [Dev Container Configuration](../../../.devcontainer/devcontainer.json) -tiedostoon  
2. Klikkaa **Code** → **Codespaces** -välilehti → Klikkaa **...** → **New with options...**  
3. Käytä oletusasetuksia – tämä valitsee **Dev container configuration**: **Generative AI Java Development Environment** -mukautettu devcontainer tälle kurssille  
4. Klikkaa **Create codespace**  
5. Odota noin 2 minuuttia, että ympäristö on valmis  
6. Jatka [Vaiheeseen 2: Azure AI Foundryn Käyttöönotto](#vaihe-2-azure-ai-foundryn-käyttöönotto)  

<img src="../../../translated_images/fi/codespaces.9945ded8ceb431a5.webp" alt="Kuvakaappaus: Codespaces alivalikko" width="50%">

<img src="../../../translated_images/fi/image.833552b62eee7766.webp" alt="Kuvakaappaus: New with options" width="50%">

<img src="../../../translated_images/fi/codespaces-create.b44a36f728660ab7.webp" alt="Kuvakaappaus: Create codespace -valinnat" width="50%">

> **Codespacesin hyödyt**:  
> - Ei paikallista asennusta  
> - Toimii millä tahansa laitteella, jossa on selain  
> - Esikonfiguroitu kaikilla työkaluilla ja riippuvuuksilla  
> - Ilmaiset 60 tuntia per kuukausi henkilökohtaisille tileille  
> - Yhtenäinen ympäristö kaikille oppijoille  

#### Vaihtoehto B: Paikallinen Dev Container

**Kehittäjille, jotka haluavat työskennellä paikallisesti Dockerilla**

1. Forkkaa ja kloonaa tämä repositorio paikalliselle koneellesi  
   > **Huom:** Jos haluat muokata perusasetuksia, tutustu [Dev Container Configuration](../../../.devcontainer/devcontainer.json) -tiedostoon  
2. Asenna [Docker Desktop](https://www.docker.com/products/docker-desktop/) ja [VS Code](https://code.visualstudio.com/)  
3. Asenna [Dev Containers -laajennus](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) VS Codeen  
4. Avaa repositoriokansio VS Codessa  
5. Kun kehote ilmestyy, valitse **Reopen in Container** (tai käytä `Ctrl+Shift+P` → "Dev Containers: Reopen in Container")  
6. Odota, että kontti rakentuu ja käynnistyy  
7. Jatka [Vaiheeseen 2: Azure AI Foundryn Käyttöönotto](#vaihe-2-azure-ai-foundryn-käyttöönotto)  

<img src="../../../translated_images/fi/devcontainer.21126c9d6de64494.webp" alt="Kuvakaappaus: Dev container -asetukset" width="50%">

<img src="../../../translated_images/fi/image-3.bf93d533bbc84268.webp" alt="Kuvakaappaus: Dev container -rakennus valmis" width="50%">

#### Vaihtoehto C: Käytä Olemassa Olevaa Paikallista Asennustasi

**Kehittäjille, joilla on jo Java-ympäristö**

Esivaatimukset:  
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html)  
- [Maven 3.9+](https://maven.apache.org/download.cgi)  
- [VS Code](https://code.visualstudio.com) tai haluamasi IDE

Vaiheet:  
1. Kloonaa tämä repositorio paikalliselle koneellesi  
2. Avaa projekti IDE:ssäsi  
3. Jatka [Vaiheeseen 2: Azure AI Foundryn Käyttöönotto](#vaihe-2-azure-ai-foundryn-käyttöönotto)  

> **Vinkki:** Jos sinulla on heikkotehoinen kone mutta haluat käyttää VS Codea paikallisesti, käytä GitHub Codespacesia! Voit yhdistää paikallisen VS Coden pilvipohjaiseen Codespaceen parhaan kokemuksen saamiseksi.  

<img src="../../../translated_images/fi/image-2.fc0da29a6e4d2aff.webp" alt="Kuvakaappaus: luotu paikallinen devcontainer-instanssi" width="50%">

## Vaihe 2: Azure AI Foundryn Käyttöönotto

Ota kurssin AI-mallit käyttöön Azure AI Foundryyn koodina. Repositorion juurihakemistosta:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```
  
`azd` kysyy ympäristön nimen ja alueen, ottaa käyttöön Azure AI Foundry -tilin `gpt-4o-mini` ja `text-embedding-3-small` -käyttöönottoineen sekä kirjoittaa päätepisteen esimerkin `.env`-tiedostoon — kaikki **avaimettomalla** todennuksella (ei API-avaimia).

> **Täysi läpikäynti:** Katso [Azure AI Foundry -aloitusopas](getting-started-azure-openai.md) esivaatimuksista, manuaalisesta (portaali) vaihtoehdosta, alueohjeista sekä kustannus- ja siivousohjeista.

## Vaihe 3: Testaa Asetuksesi

Kun Foundry-mallit ovat valmiina, testaa yhteys esimerkkisovelluksella hakemistossa [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure).

1. Avaa terminaali kehitysympäristössäsi.  
2. Siirry esimerkkikansioon:  
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
  
3. Varmista, että olet kirjautunut sisään (avaimeton todennus tarvitsee tokenin):  
   ```bash
   az login
   ```
  
   > Jos olet suorittanut `azd up`, `.env`-tiedosto päätepisteellä on jo luotu.  
4. Käynnistä sovellus:  
   ```bash
   mvn clean spring-boot:run
   ```
  
Sinun pitäisi nähdä vastaus `gpt-4o-mini` -mallilta.

### Esimerkkikoodin Ymmärtäminen

`examples/basic-chat-azure` -kansiossa oleva esimerkki on Spring Boot -sovellus, joka käyttää **Spring AI**:ta yhdistääkseen Azure AI Foundryyn avaimettomalla todennuksella.

**Mitä tämä koodi tekee:**
- **Yhdistää** Azure AI Foundryyn Azure-kirjautumistunnuksellasi (Microsoft Entra ID) — ilman API-avainta  
- **Lähettää** kehotteen `gpt-4o-mini` -mallille  
- **Vastaanottaa** ja näyttää AI:n vastauksen  
- **Varmistaa** että asetuksesi toimivat oikein  

**Avainriippuvuus** (`pom.xml`):  
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```
  
**Konfiguraatio** (`application.yml`):  
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
  

## Yhteenveto

Hienoa! Nyt sinulla on kaikki valmiina:

- Otit Azure AI Foundryn mallit käyttöön koodina Bicepillä + `azd`:llä  
- Java-kehitysympäristösi toimii (oli se sitten Codespaces, devcontainer tai paikallinen)  
- Yhdistit Azure AI Foundryyn avaimettomalla todennuksella (Microsoft Entra ID) — ilman API-avaimia  
- Testasit kaiken toimivan yksinkertaisella esimerkillä, joka kommunikoi mallisi kanssa  

## Seuraavat Askeleet

[Luku 3: Keskeiset Generative AI -tekniikat](../03-CoreGenerativeAITechniques/README.md)

## Vianetsintä

Ongelmia? Tässä yleisimmät ongelmat ja ratkaisut:

- **Todennus epäonnistuu (401/403)?**  
  - Suorita `az login` — todennus on avaimetonta, joten kirjaudu sisään  
  - Varmista, että tililläsi on **Cognitive Services OpenAI User** -rooli resurssissa  
  - Jos otit juuri käyttöön, odota hetki, että roolijako leviää  

- **Maven ei löydy?**  
  - Dev containereissa/Codespacessa Mavenin pitäisi olla esiasennettu  
  - Paikallisessa asennuksessa varmista, että Java 21+ ja Maven 3.9+ on asennettu  
  - Tarkista komennolla `mvn --version`

- **`azd` ei löydy tai käyttöönotto epäonnistuu?**  
  - Asenna [Azure Developer CLI](https://aka.ms/azure-dev/install) ja suorita `azd auth login`  
  - Valitse alue, jolla `gpt-4o-mini` on saatavilla (esim. `eastus2`)  
  - Katso [Azure AI Foundry -aloitusopas](getting-started-azure-openai.md) lisätiedoista  

- **Dev container ei käynnisty?**  
  - Varmista, että Docker Desktop on käynnissä (paikallista kehitystä varten)  
  - Yritä rakentaa kontti uudelleen: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **Sovelluksen käännösvirheitä?**  
  - Varmista, että olet oikeassa hakemistossa: `02-SetupDevEnvironment/examples/basic-chat-azure`  
  - Yritä puhdistaa ja kääntää uudelleen: `mvn clean compile`

> **Tarvitsetko apua?**: Jos ongelmat jatkuvat, avaa issue repositoriossa, niin autamme sinua.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Vastuuvapauslauseke**:
Tämä asiakirja on käännetty käyttämällä tekoälypohjaista käännöspalvelua [Co-op Translator](https://github.com/Azure/co-op-translator). Vaikka pyrimme tarkkuuteen, otathan huomioon, että automaattiset käännökset saattavat sisältää virheitä tai epätarkkuuksia. Alkuperäinen asiakirja sen alkuperäiskielellä on virallinen lähde. Tärkeissä asioissa suositellaan ammattimaista ihmiskäännöstä. Emme ole vastuussa tämän käännöksen käytöstä aiheutuvista väärinymmärryksistä tai tulkinnoista.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->