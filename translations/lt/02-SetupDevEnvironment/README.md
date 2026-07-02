# Generatyvinio DI kūrimo aplinkos nustatymas Java kalbai

> **Greitas pradėjimas:** Pateikite savo DI modelius Azure AI Foundry kaip kodą naudodami Bicep + `azd` per kelias minutes – žr. [Azure AI Foundry diegimo vadovą](getting-started-azure-openai.md). Autentifikacija yra **be raktų** (Microsoft Entra ID), todėl API raktų valdymas nereikalingas.

## Ko išmoksite

- Sukursite Java kūrimo aplinką DI programoms
- Pasirinksite ir sukonfigūruosite pageidaujamą kūrimo aplinką (pirmiausia debesyje su Codespaces, vietinį kūrimo konteinerį arba pilną vietinę sąranką)
- Patikrinsite savo aplinką prisijungdami prie Azure AI Foundry modelio

## Turinys

- [Ko išmoksite](#ko-išmoksite)
- [Įvadas](#įvadas)
- [1 žingsnis: Sukurkite kūrimo aplinką](#1-žingsnis-sukurkite-kūrimo-aplinką)
  - [Pasirinkimas A: GitHub Codespaces (rekomenduojama)](#pasirinkimas-a-github-codespaces-rekomenduojama)
  - [Pasirinkimas B: Vietinis kūrimo konteineris](#pasirinkimas-b-vietinis-kūrimo-konteineris)
  - [Pasirinkimas C: Naudokite esamą vietinę diegimą](#pasirinkimas-c-naudokite-esamą-vietinę-diegimą)
- [2 žingsnis: Pateikite Azure AI Foundry](#2-žingsnis-pateikite-azure-ai-foundry)
- [3 žingsnis: Išbandykite savo aplinką](#3-žingsnis-išbandykite-savo-aplinką)
- [Klaidų šalinimas](#klaidų-šalinimas)
- [Santrauka](#santrauka)
- [Tolimesni veiksmai](#tolimesni-veiksmai)

## Įvadas

Šis skyrius parodys, kaip sukurti kūrimo aplinką. Šiame kurse naudosime **Azure AI Foundry** modeliams. Jūs pateikiate modelius kaip kodą su Bicep ir Azure Developer CLI (`azd`), tada jungiatės su **be raktų autentifikacija** (Microsoft Entra ID) – nereikia kopijuoti ar saugoti API raktų.

**Vietinė sąranka nereikalinga!** Galite naudoti GitHub Codespaces, kuris suteikia pilną kūrimo aplinką naršyklėje ir leidžia iš ten pateikti Foundry.

Šiam kursui naudojame **Azure AI Foundry**, nes jis:
- **Diegiamas kaip kodas** — vienu `azd up` įdiegiate paskyrą ir modelių diegimus
- **Be raktų** — autentifikuojatės su Azure prisijungimu arba valdomu identitetu
- **Paruoštas gamybai** — tas pats kodas veikia vietoje ir Azure
- **Lankstus** — pakeiskite modelius keičiant diegimo pavadinimą, o ne kodą

> **Pastaba:** Azure AI Foundry diegimai apmokestinami už tokenus (mokate pagal naudojimą). Žr. [Azure AI Foundry diegimo vadovą](getting-started-azure-openai.md) dėl sąrankos, regiono ir kainų.

## 1 žingsnis: Sukurkite kūrimo aplinką

<a name="quick-start-cloud"></a>

Sukūrėme iš anksto sukonfigūruotą kūrimo konteinerį, kad sumažintume sąrankos laiką ir užtikrintume, jog turite visus reikalingus įrankius šiame Generatyvinio DI Java kursui. Pasirinkite norimą kūrimo metodą:

### Aplinkos nustatymo pasirinkimai:

#### Pasirinkimas A: GitHub Codespaces (rekomenduojama)

**Pradėkite koduoti per 2 minutes – vietinė sąranka nereikalinga!**

1. Atšakokite (fork) šį saugyklą į savo GitHub paskyrą  
   > **Pastaba**: Jei norite redaguoti bazinę konfigūraciją, peržiūrėkite [Dev konteinerio konfigūraciją](../../../.devcontainer/devcontainer.json)  
2. Spustelėkite **Code** → skirtuką **Codespaces** → **...** → **New with options...**  
3. Naudokite numatytuosius nustatymus – pasirinktas bus **Dev konteinerio konfigūracija**: **Generatyvinio DI Java kūrimo aplinka**, sukurta specialiai šiam kursui  
4. Spustelėkite **Create codespace**  
5. Palaukite apie 2 minutes, kol aplinka bus paruošta  
6. Tęskite prie [2 žingsnio: Pateikite Azure AI Foundry](#2-žingsnis-pateikite-azure-ai-foundry)

<img src="../../../translated_images/lt/codespaces.9945ded8ceb431a5.webp" alt="Ekrano kopija: Codespaces poskiltis" width="50%">

<img src="../../../translated_images/lt/image.833552b62eee7766.webp" alt="Ekrano kopija: Naujas su parinktimis" width="50%">

<img src="../../../translated_images/lt/codespaces-create.b44a36f728660ab7.webp" alt="Ekrano kopija: Kurti codespace parinktis" width="50%">


> **Codespaces privalumai:**
> - Vietinė diegimas nereikalingas
> - Veikia bet kuriame įrenginyje su naršykle
> - Iš anksto sukonfigūruota su visais įrankiais ir priklausomybėmis
> - Asmeninėms paskyroms 60 nemokamų valandų per mėnesį
> - Nuosekli aplinka visiems besimokantiesiems

#### Pasirinkimas B: Vietinis kūrimo konteineris

**Skirta kūrėjams, kurie nori vietinės plėtros su Docker**

1. Atšakokite ir klonuokite šią saugyklą į savo vietinį kompiuterį  
   > **Pastaba**: Jei norite redaguoti bazinę konfigūraciją, peržiūrėkite [Dev konteinerio konfigūraciją](../../../.devcontainer/devcontainer.json)  
2. Įdiekite [Docker Desktop](https://www.docker.com/products/docker-desktop/) ir [VS Code](https://code.visualstudio.com/)  
3. Įdiekite [Dev Containers plėtinį](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) VS Code  
4. Atidarykite saugyklos aplanką VS Code  
5. Kai bus paraginta, spustelėkite **Reopen in Container** (arba naudokite `Ctrl+Shift+P` → "Dev Containers: Reopen in Container")  
6. Palaukite, kol konteineris bus sukurtas ir paleistas  
7. Tęskite prie [2 žingsnio: Pateikite Azure AI Foundry](#2-žingsnis-pateikite-azure-ai-foundry)

<img src="../../../translated_images/lt/devcontainer.21126c9d6de64494.webp" alt="Ekrano kopija: Dev konteinerio sąranka" width="50%">

<img src="../../../translated_images/lt/image-3.bf93d533bbc84268.webp" alt="Ekrano kopija: Dev konteinerio kūrimas baigtas" width="50%">

#### Pasirinkimas C: Naudokite esamą vietinę diegimą

**Skirta kūrėjams su esama Java aplinka**

Reikalavimai:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html)  
- [Maven 3.9+](https://maven.apache.org/download.cgi)  
- [VS Code](https://code.visualstudio.com) arba preferuojama IDE

Veiksmai:
1. Klonuokite šią saugyklą į savo vietinį kompiuterį  
2. Atidarykite projektą savo IDE  
3. Tęskite prie [2 žingsnio: Pateikite Azure AI Foundry](#2-žingsnis-pateikite-azure-ai-foundry)

> **Profesionalus patarimas:** Jei turite silpną kompiuterį, bet norite naudoti VS Code vietoje, naudokite GitHub Codespaces! Galite prijungti savo vietinį VS Code prie debesyje išlaikomo Codespace ir turėti geriausią iš abiejų pasaulių.

<img src="../../../translated_images/lt/image-2.fc0da29a6e4d2aff.webp" alt="Ekrano kopija: sukurtas vietinis dev konteinerio egzempliorius" width="50%">

## 2 žingsnis: Pateikite Azure AI Foundry

Diegkite kurso DI modelius į Azure AI Foundry kaip kodą. Iš saugyklos šakninio aplanko:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```
  
`azd` paprašys aplinkos pavadinimo ir regiono, pateiks Azure AI Foundry paskyrą su `gpt-4o-mini` ir `text-embedding-3-small` diegimais bei įrašys galinius taškus į pavyzdžio `.env` — visa tai su **be raktų** autentifikacija (be API raktų).

> **Pilnas vedlys:** Žr. [Azure AI Foundry diegimo vadovą](getting-started-azure-openai.md) dėl išankstinių sąlygų, rankinio (portalo) alternatyvos, regionų ir kainų/valymo pastabų.

## 3 žingsnis: Išbandykite savo aplinką

Kai Foundry modeliai bus įdiegti, patikrinkite ryšį su pavyzdine programa kataloge [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure).

1. Atidarykite terminalą savo kūrimo aplinkoje.  
2. Eikite į pavyzdį:  
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
  
3. Įsitikinkite, kad esate prisijungę (be raktų autentifikacijai reikalingas žetonas):  
   ```bash
   az login
   ```
  
   > Jei paleidote `azd up`, `.env` failas su jūsų galiniu tašku jau buvo sukurtas.  
4. Paleiskite programą:  
   ```bash
   mvn clean spring-boot:run
   ```
  
Turėtumėte pamatyti atsakymą iš `gpt-4o-mini` modelio.

### Pavyzdinio kodo supratimas

Pavyzdys `examples/basic-chat-azure` yra Spring Boot programa, kuri naudoja **Spring AI** prisijungimui prie Azure AI Foundry su be raktų autentifikacija.

**Ką daro šis kodas:**
- **Jungiasi** prie Azure AI Foundry su jūsų Azure prisijungimu (Microsoft Entra ID) – be API rakto  
- **Siunčia** užklausą `gpt-4o-mini` modeliui  
- **Gautą** atsakymą atvaizduoja  
- **Tikrina**, ar jūsų sąranka veikia tinkamai

**Pagrindinė priklausomybė** (faile `pom.xml`):  
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```
  
**Konfigūracija** (`application.yml`):  
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
  
## Santrauka

Puiku! Dabar turite viską sukonfigūruotą:

- Pateikėte Azure AI Foundry modelius kaip kodą su Bicep + `azd`  
- Paleidote Java kūrimo aplinką (Codespaces, kūrimo konteinerius ar vietinę)  
- Prisijungėte prie Azure AI Foundry su be raktų autentifikacija (Microsoft Entra ID) – be API raktų  
- Išbandėte veikimą su paprastu pavyzdžiu, kuris bendrauja su jūsų modeliu

## Tolimesni veiksmai

[3 skyrius: Pagrindinės generatyvinio DI technikos](../03-CoreGenerativeAITechniques/README.md)

## Klaidų šalinimas

Turite problemų? Štai dažniausios problemos ir jų sprendimai:

- **Autentifikacija nepavyksta (401/403)?**  
  - Vykdykite `az login` – autentifikacija be raktų, todėl turite būti prisijungę  
  - Patikrinkite, ar jūsų paskyra turi **Cognitive Services OpenAI User** rolę resurse  
  - Jei ką tik pateikėte paslaugą, palaukite minutę, kol rolės priskyrimas įsigalios  

- **Maven nerandamas?**  
  - Jei naudojate kūrimo konteinerius ar Codespaces, Maven turėtų būti iš anksto įdiegtas  
  - Vietinėje sąrangoje įsitikinkite, kad įdiegta Java 21+ ir Maven 3.9+  
  - Patikrinkite `mvn --version`  

- **`azd` nerandamas arba diegimas nepavyksta?**  
  - Įdiekite [Azure Developer CLI](https://aka.ms/azure-dev/install) ir vykdykite `azd auth login`  
  - Pasirinkite regioną, kuriame yra `gpt-4o-mini` (pvz., `eastus2`)  
  - Žr. [Azure AI Foundry diegimo vadovą](getting-started-azure-openai.md)  

- **Kūrimo konteineris nesikuria?**  
  - Įsitikinkite, kad Docker Desktop veikia (vietinei plėtrai)  
  - Bandykite perkurti konteinerį: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"  

- **Programos kompiliavimo klaidos?**  
  - Įsitikinkite, kad esate teisingame kataloge: `02-SetupDevEnvironment/examples/basic-chat-azure`  
  - Bandykite išvalyti ir iš naujo kompiliuoti: `mvn clean compile`  

> **Reikia pagalbos?**: Jei problemos nepavyksta išspręsti, atidarykite problemą repozitorijoje, mes jums padėsime.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Atsakomybės apribojimas**:
Šis dokumentas buvo išverstas naudojant dirbtinio intelekto vertimo paslaugą [Co-op Translator](https://github.com/Azure/co-op-translator). Nors siekiame tikslumo, prašome atkreipti dėmesį, kad automatiniai vertimai gali turėti klaidų ar netikslumų. Originalus dokumentas jo gimtąja kalba laikomas autoritetingu šaltiniu. Svarbiai informacijai rekomenduojama naudoti profesionalų žmogiškąjį vertimą. Mes neatsakome už jokius nesusipratimus ar neteisingą interpretaciją, kilusią naudojantis šiuo vertimu.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->