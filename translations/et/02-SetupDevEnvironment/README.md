# Arenduskeskkonna seadistamine Generatiivse tehisintellekti jaoks Java-s

> **Kiirkäivitus:** Provisioni oma tehisintellekti mudelid **Azure AI Foundry’s** koodi abil Bicep + `azd` kaudu mõne minutiga — vaata [Azure AI Foundry seadistusjuhendit](getting-started-azure-openai.md). Autentimine on **võtmevaba** (Microsoft Entra ID), seega API võtmeid hallata ei ole vaja.

## Mida Sa Õpid

- Seada üles Java arenduskeskkond tehisintellekti rakenduste jaoks
- Valida ja konfigureerida eelistatud arenduskeskkond (pilvepõhine Codespaces, lokaalne arenduskonteiner või täielik kohalik seadistus)
- Testida oma seadistust, ühendudes Azure AI Foundry mudeliga

## Sisukord

- [Mida Sa Õpid](#mida-sa-õpid)
- [Sissejuhatus](#sissejuhatus)
- [1. samm: Seadista oma arenduskeskkond](#1-samm-seadista-oma-arenduskeskkond)
  - [Valik A: GitHub Codespaces (Soovitatav)](#valik-a-github-codespaces-soovitatav)
  - [Valik B: Kohalik arenduskonteiner](#valik-b-kohalik-arenduskonteiner)
  - [Valik C: Kasuta oma olemasolevat kohalikku installi](#valik-c-kasuta-oma-olemasolevat-kohalikku-installi)
- [2. samm: Provisioni Azure AI Foundry](#2-samm-provisioni-azure-ai-foundry)
- [3. samm: Testi oma seadistust](#3-samm-testi-oma-seadistust)
- [Probleemide lahendamine](#probleemide-lahendamine)
- [Kokkuvõte](#kokkuvõte)
- [Edasised sammud](#edasised-sammud)

## Sissejuhatus

See peatükk juhendab sind arenduskeskkonna seadistamisel. Selle kursuse jooksul kasutame **Azure AI Foundry** mudeleid. Provisionid mudelid koodi kaudu Bicepiga ja Azure Developer CLI-ga (`azd`), seejärel ühendud **võtmevaba autentimisega** (Microsoft Entra ID) — API võtmeid ei ole vaja kopeerida ega lekkida.

**Local setup pole vajalik!** Saad kasutada GitHub Codespaces’i, mis pakub täielikku arenduskeskkonda sinu brauseris ning seal provisionid Foundry.

Selle kursuse jaoks kasutame **Azure AI Foundry’d**, sest see on:
- **Provisionitud koodina** — üks `azd up` käsk juurutab konto ja mudeli juurutused
- **Võtmevaba** — autentimine toimub Azure sisselogimise või hallatava identiteedi kaudu
- **Tootmiskõlblik** — sama kood töötab nii lokaalselt kui ka Azure’is
- **Paindlik** — mudelite vahetus toimub vaid juurutuse nime muutmisega, mitte koodi muutmisega

> **Märkus**: Azure AI Foundry juurutustel arvestatakse tasu tokenite alusel (tasu kasutuse järgi). Vaata [Azure AI Foundry seadistusjuhendit](getting-started-azure-openai.md) juurutuse, regiooni ja kulude kohta.

## 1. samm: Seadista oma arenduskeskkond

<a name="quick-start-cloud"></a>

Oleme loonud eelkonfigureeritud arenduskonteineri, mis vähendab seadistusaega ja tagab, et sul on kõik vajalikud tööriistad selle Generatiivse tehisintellekti Java kursuse jaoks. Vali oma eelistatud arendusviis:

### Arenduskeskkonna seadistamise valikud:

#### Valik A: GitHub Codespaces (Soovitatav)

**Alusta kodeerimist 2 minutiga – lokaalne seadistus pole vajalik!**

1. Loo fork sellest repositori oma GitHubi kontole
   > **Märkus**: Kui soovid muuta baaskonfiguratsiooni, vaata [Arenduskonteineri konfiguratsiooni](../../../.devcontainer/devcontainer.json)
2. Klõpsa **Code** → **Codespaces** sakk → **...** → **New with options...**
3. Kasuta vaikeseadeid – see valib **Arenduskonteineri konfiguratsiooni**: **Generative AI Java Development Environment**, mis on selle kursuse jaoks loodud kohandatud devcontainer
4. Klõpsa **Create codespace**
5. Oota ~2 minutit, kuni keskkond on valmis
6. Liigu järgmisse sammu [2. samm: Provisioni Azure AI Foundry](#2-samm-provisioni-azure-ai-foundry)

<img src="../../../translated_images/et/codespaces.9945ded8ceb431a5.webp" alt="Ekraanipilt: Codespaces alammenüü" width="50%">

<img src="../../../translated_images/et/image.833552b62eee7766.webp" alt="Ekraanipilt: Uus koos valikutega" width="50%">

<img src="../../../translated_images/et/codespaces-create.b44a36f728660ab7.webp" alt="Ekraanipilt: Create codespace valikud" width="50%">


> **Codespaces'i eelised**:
> - Pole vaja kohalikku installi
> - Töötab mis tahes seadmes, kus on brauser
> - Eelkonfigureeritud kõikide tööriistade ja sõltuvustega
> - Tasuta 60 tundi kuus isiklikele kontodele
> - Ühtne keskkond kõigile õppijatele

#### Valik B: Kohalik Arenduskonteiner

**Arendajatele, kes eelistavad kohalikku arendust Dockeri abil**

1. Tee fork ja klooni see repo oma kohalikku arvutisse
   > **Märkus**: Kui soovid muuta baaskonfiguratsiooni, vaata [Arenduskonteineri konfiguratsiooni](../../../.devcontainer/devcontainer.json)
2. Paigalda [Docker Desktop](https://www.docker.com/products/docker-desktop/) ja [VS Code](https://code.visualstudio.com/)
3. Paigalda VS Code’i [Dev Containers laiendus](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
4. Ava repokataloog VS Code’is
5. Kui küsitakse, klõpsa **Reopen in Container** (või kasuta `Ctrl+Shift+P` → "Dev Containers: Reopen in Container")
6. Oota, kuni konteiner ehitatakse ja käivitub
7. Liigu järgmisse sammu [2. samm: Provisioni Azure AI Foundry](#2-samm-provisioni-azure-ai-foundry)

<img src="../../../translated_images/et/devcontainer.21126c9d6de64494.webp" alt="Ekraanipilt: Arenduskonteineri seadistamine" width="50%">

<img src="../../../translated_images/et/image-3.bf93d533bbc84268.webp" alt="Ekraanipilt: Arenduskonteineri ehitamine lõpetatud" width="50%">

#### Valik C: Kasuta oma olemasolevat kohalikku installi

**Arendajatele, kellel on olemasolev Java keskkond**

Eeltingimused:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html) 
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) või sinu eelistatud IDE

Sammud:
1. Klooni see repo oma kohalikku masinasse
2. Ava projekt oma IDE-s
3. Liigu järgmisse sammu [2. samm: Provisioni Azure AI Foundry](#2-samm-provisioni-azure-ai-foundry)

> **Nipp**: Kui sul on madalama specsiga masin, aga soovid VS Code’i lokaalselt, kasuta GitHub Codespaces’i! Sa saad ühendada oma lokaalse VS Code’i pilvepõhise Codespace’iga, et saada mõlema maailma parim.

<img src="../../../translated_images/et/image-2.fc0da29a6e4d2aff.webp" alt="Ekraanipilt: loodud kohalik devcontainer eksemplar" width="50%">


## 2. samm: Provisioni Azure AI Foundry

Juuruta selle kursuse tehisintellekti mudelid Azure AI Foundry’sse koodi abil. Repositori juurest:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```

`azd` küsib keskkonna nime ja regiooni, provisionib Azure AI Foundry konto koos `gpt-4o-mini` ja `text-embedding-3-small` juurutustega ning kirjutab lõpp-punkti näite `.env` faili — kõik see **võtmevaba** autentimisega (ilma API võtmeteta).

> **Täielik juhend:** Vaata [Azure AI Foundry seadistusjuhendit](getting-started-azure-openai.md) eeltingimuste, manuaalse (portaali) alternatiivi, regiooni juhiste ja kulude/koristamise märkuste jaoks.

## 3. samm: Testi oma seadistust

Kui su Foundry mudelid on provisionitud, testi ühendust näiterakendusega kataloogis [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure).

1. Ava terminal oma arenduskeskkonnas.
2. Liigu näiteni:
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
3. Veendu, et oled sisse logitud (võtmevaba autentimine vajab tokenit):
   ```bash
   az login
   ```
   > Kui jooksutasid `azd up`, kirjutati sinu lõpp-punkt juba `.env` faili.
4. Käivita rakendus:
   ```bash
   mvn clean spring-boot:run
   ```

Sa peaksid nägema vastust mudelilt `gpt-4o-mini`.

### Näite koodi mõistmine

Näide kataloogis `examples/basic-chat-azure` on Spring Boot rakendus, mis kasutab **Spring AI** ühendamaks Azure AI Foundry’ga võtmevaba autentimise abil.

**Mida see kood teeb:**
- **Ühendub** Azure AI Foundry’ga, kasutades sinu Azure sisselogimist (Microsoft Entra ID) — API võtmeteta
- **Saadab** päringu mudelile `gpt-4o-mini`
- **Vastuvõtab** ja kuvab tehisintellekti vastuse
- **Kontrollib**, et sinu seadistus töötab õigesti

**Peamine sõltuvus** (`pom.xml`):
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```

**Konfiguratsioon** (`application.yml`):
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

## Kokkuvõte

Suurepärane! Sul on nüüd kõik seadistatud:

- Provisionisid Azure AI Foundry mudeleid koodi kaudu Bicep + `azd` abil
- Käivitasid oma Java arenduskeskkonna (olgu see Codespaces, devcontainer või kohalik)
- Ühendusid Azure AI Foundry’ga võtmevaba autentimisega (Microsoft Entra ID) — ilma API võtmeteta
- Testisid, et kõik töötab lihtsa näitega, mis suhtleb mudeliga

## Edasised sammud

[3. peatükk: Core Generative AI Techniques](../03-CoreGenerativeAITechniques/README.md)

## Probleemide lahendamine

Probleemidega? Siin on levinumad vead ja lahendused:

- **Autentimine ei õnnestu (401/403)?** 
  - Käivita `az login` — autentimine on võtmevaba, seega pead olema sisse logitud
  - Kontrolli, et su kontol on ressurssidel roll **Cognitive Services OpenAI User**
  - Kui just provisionisid, oota minut, kuni rollimäärang kandub

- **Mavenit ei leitud?** 
  - Kui kasutad devkante/BliceSpacesi, on Maven eelinstallitud
  - Kohaliku seadistuse puhul veendu, et Java 21+ ja Maven 3.9+ on paigaldatud
  - Proovi `mvn --version` paigalduse kontrollimiseks

- **`azd` ei leitud või provisioning ebaõnnestus?** 
  - Paigalda [Azure Developer CLI](https://aka.ms/azure-dev/install) ja käivita `azd auth login`
  - Vali regiooni, kus `gpt-4o-mini` on saadaval (näiteks `eastus2`)
  - Vaata [Azure AI Foundry seadistusjuhendit](getting-started-azure-openai.md) lisainfo saamiseks

- **Dev konteinerit ei käivitata?** 
  - Veendu, et Docker Desktop töötab (kohalikuks arenduseks)
  - Proovi konteineri uuesti ehitada: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **Rakenduse kompileerimisvead?**
  - Veendu, et oled õiges kataloogis: `02-SetupDevEnvironment/examples/basic-chat-azure`
  - Proovi puhastada ja uuesti kompileerida: `mvn clean compile`

> **Vaja abi?**: Kui probleemid jätkuvad, ava reposse issue ja aitame sind.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Lahtiütlus**:
See dokument on tõlgitud kasutades AI tõlketeenust [Co-op Translator](https://github.com/Azure/co-op-translator). Kuigi me püüdleme täpsuse poole, palun pange tähele, et automatiseeritud tõlgetes võib esineda vigu või ebatäpsusi. Originaaldokument selle emakeeles tuleks pidada autoriteetseks allikaks. Olulise teabe puhul soovitatakse kasutada professionaalset inimtõlget. Me ei vastuta selle tõlkega seotud eksimustest või valesti mõistmistest.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->