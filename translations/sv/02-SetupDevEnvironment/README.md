# Installera utvecklingsmiljön för Generativ AI för Java

> **Snabbstart:** Tillhandahåll dina AI-modeller på **Azure AI Foundry** som kod med Bicep + `azd` på några minuter — se [Azure AI Foundry Setup Guide](getting-started-azure-openai.md). Autentisering är **nyckellös** (Microsoft Entra ID), så det finns inga API-nycklar att hantera.

## Vad du kommer att lära dig

- Sätta upp en Java-utvecklingsmiljö för AI-applikationer
- Välja och konfigurera din föredragna utvecklingsmiljö (moln-först med Codespaces, lokal dev container eller full lokal installation)
- Testa din installation genom att koppla upp dig mot en Azure AI Foundry-modell

## Innehållsförteckning

- [Vad du kommer att lära dig](#vad-du-kommer-att-lära-dig)
- [Introduktion](#introduktion)
- [Steg 1: Sätt upp din utvecklingsmiljö](#steg-1-sätt-upp-din-utvecklingsmiljö)
  - [Alternativ A: GitHub Codespaces (Rekommenderat)](#alternativ-a-github-codespaces-rekommenderat)
  - [Alternativ B: Lokal Dev Container](#alternativ-b-lokal-dev-container)
  - [Alternativ C: Använd din befintliga lokala installation](#alternativ-c-använd-din-befintliga-lokala-installation)
- [Steg 2: Tillhandahåll Azure AI Foundry](#steg-2-tillhandahåll-azure-ai-foundry)
- [Steg 3: Testa din installation](#steg-3-testa-din-installation)
- [Felsökning](#felsökning)
- [Sammanfattning](#sammanfattning)
- [Nästa steg](#nästa-steg)

## Introduktion

Det här kapitlet kommer att vägleda dig genom att sätta upp en utvecklingsmiljö. Vi kommer att använda **Azure AI Foundry** för modellerna genom hela kursen. Du tillhandahåller modellerna som kod med Bicep och Azure Developer CLI (`azd`), och ansluter sedan med **nyckellös autentisering** (Microsoft Entra ID) — inga API-nycklar att kopiera eller läcka.

**Ingen lokal installation krävs!** Du kan använda GitHub Codespaces, som tillhandahåller en full utvecklingsmiljö i din webbläsare och provisionerar Foundry därifrån.

Vi använder **Azure AI Foundry** i denna kurs eftersom det är:
- **Tillhandahållet som kod** — en `azd up` distribuerar kontot och modelldistributionerna
- **Nyckellöst** — autentisera med ditt Azure-inloggning eller en hanterad identitet
- **Produktionredo** — samma kod körs lokalt och i Azure
- **Flexibelt** — byt modeller genom att ändra ett distributionnamn, inte din kod

> **Notera**: Azure AI Foundry-distributioner debiteras per token (pay-as-you-go). Se [Azure AI Foundry setup guide](getting-started-azure-openai.md) för information om tillhandahållande, region och kostnader.

## Steg 1: Sätt upp din utvecklingsmiljö

<a name="quick-start-cloud"></a>

Vi har skapat en förkonfigurerad utvecklingscontainer för att minimera installationstiden och säkerställa att du har alla nödvändiga verktyg för denna Generative AI for Java-kurs. Välj din föredragna utvecklingsmetod:

### Alternativ för miljöinställning:

#### Alternativ A: GitHub Codespaces (Rekommenderat)

**Börja koda på 2 minuter - ingen lokal installation krävs!**

1. Forka detta repository till ditt GitHub-konto  
   > **Notera**: Om du vill redigera grundkonfigurationen, ta en titt på [Dev Container Configuration](../../../.devcontainer/devcontainer.json)
2. Klicka på **Code** → fliken **Codespaces** → **...** → **New with options...**
3. Använd standardinställningarna – detta kommer att välja **Dev container configuration**: **Generative AI Java Development Environment**, specialanpassad devcontainer för denna kurs
4. Klicka på **Create codespace**
5. Vänta cirka 2 minuter tills miljön är redo
6. Fortsätt till [Steg 2: Tillhandahåll Azure AI Foundry](#steg-2-tillhandahåll-azure-ai-foundry)

<img src="../../../translated_images/sv/codespaces.9945ded8ceb431a5.webp" alt="Skärmdump: Codespaces undermeny" width="50%">

<img src="../../../translated_images/sv/image.833552b62eee7766.webp" alt="Skärmdump: New with options" width="50%">

<img src="../../../translated_images/sv/codespaces-create.b44a36f728660ab7.webp" alt="Skärmdump: Skapa codespace-inställningar" width="50%">


> **Fördelar med Codespaces**:  
> - Ingen lokal installation krävs  
> - Fungerar på vilken enhet som helst med webbläsare  
> - Förkonfigurerat med alla verktyg och beroenden  
> - Gratis 60 timmar per månad för personliga konton  
> - Konsistent miljö för alla deltagare  

#### Alternativ B: Lokal Dev Container

**För utvecklare som föredrar lokal utveckling med Docker**

1. Forka och klona detta repository till din lokala dator  
   > **Notera**: Om du vill redigera grundkonfigurationen, ta en titt på [Dev Container Configuration](../../../.devcontainer/devcontainer.json)
2. Installera [Docker Desktop](https://www.docker.com/products/docker-desktop/) och [VS Code](https://code.visualstudio.com/)
3. Installera [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) i VS Code
4. Öppna repository-mappen i VS Code
5. Klicka på **Reopen in Container** när du uppmanas (eller använd `Ctrl+Shift+P` → "Dev Containers: Reopen in Container")
6. Vänta tills containern byggs och startar
7. Fortsätt till [Steg 2: Tillhandahåll Azure AI Foundry](#steg-2-tillhandahåll-azure-ai-foundry)

<img src="../../../translated_images/sv/devcontainer.21126c9d6de64494.webp" alt="Skärmdump: Dev container-inställning" width="50%">

<img src="../../../translated_images/sv/image-3.bf93d533bbc84268.webp" alt="Skärmdump: Dev container byggd klar" width="50%">

#### Alternativ C: Använd din befintliga lokala installation

**För utvecklare med befintliga Java-miljöer**

Förutsättningar:  
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html)  
- [Maven 3.9+](https://maven.apache.org/download.cgi)  
- [VS Code](https://code.visualstudio.com) eller din föredragna IDE

Steg:  
1. Klona detta repository till din lokala dator  
2. Öppna projektet i din IDE  
3. Fortsätt till [Steg 2: Tillhandahåll Azure AI Foundry](#steg-2-tillhandahåll-azure-ai-foundry)

> **Proffstips**: Har du en lågpresterande maskin men vill använda VS Code lokalt? Använd GitHub Codespaces! Du kan koppla din lokala VS Code till en molnhostad Codespace för det bästa av två världar.

<img src="../../../translated_images/sv/image-2.fc0da29a6e4d2aff.webp" alt="Skärmdump: skapad lokal devcontainer-instans" width="50%">

## Steg 2: Tillhandahåll Azure AI Foundry

Distribuera kursens AI-modeller till Azure AI Foundry som kod. Från repositorys rotmapp:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```
  
`azd` frågar efter ett miljönamn och region, tillhandahåller ett Azure AI Foundry-konto med distributioner för `gpt-4o-mini` och `text-embedding-3-small`, och skriver endpoint i exempelns `.env` — allt med **nyckellös** autentisering (inga API-nycklar).

> **Full genomgång:** Se [Azure AI Foundry Setup Guide](getting-started-azure-openai.md) för förutsättningar, ett manuellt (portal) alternativ, regionråd och kostnads-/rengöringsinformation.

## Steg 3: Testa din installation

När dina Foundry-modeller är tillhandahållna, testa anslutningen med exempelappen i [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure).

1. Öppna terminalen i din utvecklingsmiljö.  
2. Navigera till exemplet:  
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
  
3. Säkerställ att du är inloggad (nyckellös autentisering kräver token):  
   ```bash
   az login
   ```
  
   > Om du körde `azd up` skrevs `.env`-filen med din endpoint redan automatiskt.  
4. Kör applikationen:  
   ```bash
   mvn clean spring-boot:run
   ```
  
Du bör se ett svar från `gpt-4o-mini`-modellen.

### Förståelse av exempel-koden

Exemplet under `examples/basic-chat-azure` är en Spring Boot-app som använder **Spring AI** för att ansluta till Azure AI Foundry med nyckellös autentisering.

**Vad denna kod gör:**  
- **Ansluter** till Azure AI Foundry med ditt Azure-inlogg (Microsoft Entra ID) — ingen API-nyckel  
- **Skickar** en prompt till `gpt-4o-mini` modellen  
- **Tar emot** och visar AI:ns svar  
- **Validerar** att din installation fungerar korrekt  

**Viktig beroende** (i `pom.xml`):  
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```
  
**Konfiguration** (`application.yml`):  
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
  

## Sammanfattning

Bra jobbat! Nu har du allt installerat:

- Tillhandahållna Azure AI Foundry-modeller som kod med Bicep + `azd`  
- Din Java-utvecklingsmiljö igång (vare sig det är Codespaces, dev containers eller lokalt)  
- Ansluten till Azure AI Foundry med nyckellös autentisering (Microsoft Entra ID) — inga API-nycklar  
- Testat att allt fungerar med ett enkelt exempel som kommunicerar med din modell

## Nästa steg

[Kapitel 3: Kärntekniker för Generativ AI](../03-CoreGenerativeAITechniques/README.md)

## Felsökning

Problem? Här är vanliga problem och lösningar:

- **Autentisering misslyckas (401/403)?**  
  - Kör `az login` — autentisering är nyckellös, så du måste vara inloggad  
  - Verifiera att ditt konto har rollen **Cognitive Services OpenAI User** på resursen  
  - Om du just provisionerat, vänta en minut för att rolltilldelningen ska spridas

- **Maven hittas inte?**  
  - Om du använder dev containers/Codespaces ska Maven vara förinstallerat  
  - För lokal installation, säkerställ att Java 21+ och Maven 3.9+ är installerade  
  - Prova `mvn --version` för att verifiera installation

- **`azd` hittas inte eller provisionering misslyckas?**  
  - Installera [Azure Developer CLI](https://aka.ms/azure-dev/install) och kör `azd auth login`  
  - Välj en region där `gpt-4o-mini` är tillgänglig (t.ex. `eastus2`)  
  - Se [Azure AI Foundry setup guide](getting-started-azure-openai.md) för detaljer

- **Dev container startar inte?**  
  - Kontrollera att Docker Desktop körs (för lokal utveckling)  
  - Försök bygga om containern: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **Kompileringsfel i applikationen?**  
  - Se till att du är i rätt katalog: `02-SetupDevEnvironment/examples/basic-chat-azure`  
  - Försök rengöra och bygga om: `mvn clean compile`

> **Behöver du hjälp?**: Har du fortfarande problem? Öppna ett ärende i repositoryt så hjälper vi dig.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Ansvarsfriskrivning**:
Detta dokument har översatts med hjälp av AI-översättningstjänsten [Co-op Translator](https://github.com/Azure/co-op-translator). Även om vi strävar efter noggrannhet, var vänlig notera att automatiska översättningar kan innehålla fel eller brister. Det ursprungliga dokumentet på dess modersmål bör betraktas som den auktoritativa källan. För kritisk information rekommenderas professionell mänsklig översättning. Vi ansvarar inte för några missförstånd eller feltolkningar som uppstår till följd av användningen av denna översättning.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->