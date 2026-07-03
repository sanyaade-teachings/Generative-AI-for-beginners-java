# ஜெனரேட்டிவ் AI க்கான Java வளர்ச்சிக் சூழலை அமைத்தல்

> **விரைவான துவக்கம்:** Azure AI Foundry இல் உங்கள் AI மாதிரிகளை குறியீட்டுடன் Bicep + `azd` பயன்படுத்தி சில நிமிடங்களில் வாயிலாக ஏற்படுத்துங்கள் — [Azure AI Foundry அமைப்பு வழிகாட்டி](getting-started-azure-openai.md) ஐ பாருங்கள். அங்கீகாரம் **keyless** (Microsoft Entra ID) ஆகும், எனவே API விசைகள் நிர்வகிக்க தேவையில்லை.

## நீங்கள் கற்றுக்கொள்ளவிருக்கும் விஷயங்கள்

- AI செயலிகளுக்கான Java வளர்ச்சி சூழலை அமைத்தல்
- உங்கள் விருப்பமான வளர்ச்சி சூழலை தேர்ந்தெடுத்து அமைத்தல் (Codespaces உடன் மேக முதலில், உள்ளூர் dev container, அல்லது முழு உள்ளூர் அமைப்பு)
- Azure AI Foundry மாதிரிக்கு இணைந்து உங்கள் அமைப்பை பரிசோதித்தல்

## உள்ளடக்க அட்டவணை

- [நீங்கள் கற்றுக்கொள்ளவிருக்கும் விஷயங்கள்](#நீங்கள்-கற்றுக்கொள்ளவிருக்கும்-விஷயங்கள்)
- [அறிமுகம்](#அறிமுகம்)
- [படி 1: உங்கள் வளர்ச்சி சூழலை அமைத்தல்](#படி-1-உங்கள்-வளர்ச்சி-சூழலை-அமைத்தல்)
  - [விருப்பம் A: GitHub Codespaces (பரிந்துரைக்கப்பட்டது)](#விருப்பம்-a-github-codespaces-பரிந்துரைக்கப்பட்டது)
  - [விருப்பம் B: உள்ளூரான Dev Container](#விருப்பம்-b-உள்ளூரான-dev-container)
  - [விருப்பம் C: உங்கள் உள்ளூர் நிறுவலை பயன்படுத்துதல்](#விருப்பம்-c-உங்கள்-உள்ளூர்-நிறுவலை-பயன்படுத்துதல்)
- [படி 2: Azure AI Foundry அமைத்தல்](#படி-2-azure-ai-foundry-அமைத்தல்)
- [படி 3: உங்கள் அமைப்பை பரிசோதித்தல்](#படி-3-உங்கள்-அமைப்பை-பரிசோதித்தல்)
- [சிக்கல்களை தீர்க்குதல்](#சிக்கல்களை-தீர்க்குதல்)
- [சாராம்சம்](#சாராம்சம்)
- [அடுத்த படிகள்](#அடுத்த-படிகள்)

## அறிமுகம்

இந்த அத்தியாயம் நீங்கள் வளர்ச்சி சூழலை அமைப்பதில் வழிகாட்டும். இக்கற்கோர் முழுவதும் மாதிரிகளுக்காக **Azure AI Foundry** ஐ பயன்படுத்துவோம். நீங்கள் Bicep மற்றும் Azure Developer CLI (`azd`) மூலம் குறியீட்டுபடி மாதிரிகளை ஏற்படுத்துவீர்கள், பின்னர் **keyless authentication** (Microsoft Entra ID) மூலம் இணைவீர்கள் — எந்த API விசைகள் நகலெடுக்கவும் தேவையில்லை.

**உள்ளூர் அமைப்பு தேவையில்லை!** உங்கள் உலாவியில் முழுமையான வளர்ச்சி சூழலை வழங்க GitHub Codespaces ஐ பயன்படுத்தலாம், அங்கே இருந்து Foundry ஐ அமைக்கலாம்.

நாம் இந்த பாடத்துக்காக **Azure AI Foundry** ஐ பயன்படுத்துகிறோம் ஏனெனில் அது:
- **குறியீட்டாக Provision செய்யப்பட்டது** — ஒரு `azd up` கணக்கு மற்றும் மாதிரிகள் இடுகைகளை அமைக்கும்
- **Keyless** — உங்கள் Azure உள்நுழைவு அல்லது நிர்வகிக்கப்பட்ட அடையாளத்துடன் சான்று பெறலாம்
- **உற்பத்தித்திறன் வாய்ந்தது** — அதே குறியீடு உள்ளூரிலும் Azure இலும் இயங்கும்
- **அமைதியானது** — உங்கள் குறியீட்டை மாற்றாமல் மட்டும் இடுகை பெயரை மாற்றி மாதிரிகளை மாற்றலாம்

> **குறிப்பு**: Azure AI Foundry இடுகைகள் டோக்கன் அடிப்படையில் கட்டணம் செலுத்தப்படுகின்றன (pay-as-you-go). Provisioning, மண்டல மற்றும் செலவு விவரங்களுக்கு [Azure AI Foundry அமைப்பு வழிகாட்டி](getting-started-azure-openai.md) ஐ பாருங்கள்.


## படி 1: உங்கள் வளர்ச்சி சூழலை அமைத்தல்

<a name="quick-start-cloud"></a>

இந்த ஜெனரேட்டிவ் AI க்கான Java பயிற்சிக்கான தேவைகளை குறைக்க மற்றும் தேவையான அனைத்து கருவிகளையும் பெற்றிருக்க ஒரு முன்கூட்டிய dev container உருவாக்கப்பட்டுள்ளது. உங்கள் விருப்பமான வளர்ச்சி முறையை தேர்வுசெய்க:

### சூழல் அமைப்பு விருப்பங்கள்:

#### விருப்பம் A: GitHub Codespaces (பரிந்துரைக்கப்பட்டது)

**இரு நிமிடத்தில் குறியீடு துவங்குங்கள் - உள்ளூர் அமைப்பு தேவையில்லை!**

1. இந்த நிரலமைப்பை உங்கள் GitHub கணக்கிற்கு Fork செய்யவும்
   > **குறிப்பு**: அடிப்படை அமைப்பை திருத்த விரும்பினால் [Dev Container Configuration](../../../.devcontainer/devcontainer.json) ஐ பாருங்கள்
2. **Code** → **Codespaces** tab → **...** → **New with options...** கிளிக் செய்யவும்
3. இயல்புப்பிரிவுகளை பயன்படுத்தவும் – இது இந்த பாடத்துக்கான **Generative AI Java Development Environment** என்ற தனிப்பயன் devcontainer ஐ தேர்வு செய்யும்
4. **Create codespace** கிளிக் செய்யவும்
5. சூழல் தயாராக ~2 நிமிடம் காத்திருக்கவும்
6. [படி 2: Azure AI Foundry அமைத்தல்](#படி-2-azure-ai-foundry-அமைத்தல்) பக்கத்திற்கு செல்லவும்

<img src="../../../translated_images/ta/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: Codespaces submenu" width="50%">

<img src="../../../translated_images/ta/image.833552b62eee7766.webp" alt="Screenshot: New with options" width="50%">

<img src="../../../translated_images/ta/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: Create codespace options" width="50%">


> **Codespaces நன்மைகள்**:
> - உள்ளூர் நிறுவல் தேவையில்லை
> - எந்த உலாவியும் இயங்கும் சாதனத்தில் வேலை செய்யும்
> - அனைத்து கருவிகள் மற்றும் சார்புகளுடன் முன்பே கட்டமைக்கப்பட்டுள்ளது
> - தனிப்பட்ட கணக்குகளுக்கு மாதத்தில் இலவசமாக 60 மணி நேரம்
> - அனைத்து கற்றவர்களுக்கும் ஒரே விதமான சூழல்

#### விருப்பம் B: உள்ளூரான Dev Container

**Docker உடன் உள்ளூராக வளர்ச்சி செய்ய விரும்புவோருக்கு**

1. இந்த நிரலமைப்பை உங்கள் உள்ளூர் கணினிக்கு Fork மற்றும் Clone செய்யவும்
   > **குறிப்பு**: அடிப்படை அமைப்பை திருத்த விரும்பினால் [Dev Container Configuration](../../../.devcontainer/devcontainer.json) ஐ பாருங்கள்
2. [Docker Desktop](https://www.docker.com/products/docker-desktop/) மற்றும் [VS Code](https://code.visualstudio.com/) ஐ நிறுவவும்
3. VS Code இல் [Dev Containers விரிவுறுப்பை](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) நிறுவவும்
4. VS Code இல் நிரலமைப்பு கோப்புறையை திறக்கவும்
5. கேட்கப்பட்ட போது, **Reopen in Container** (அல்லது `Ctrl+Shift+P` → "Dev Containers: Reopen in Container") கிளிக் செய்யவும்
6. container உருவாக்கப்பட்டு துவங்குவதை காத்திருக்கவும்
7. [படி 2: Azure AI Foundry அமைத்தல்](#படி-2-azure-ai-foundry-அமைத்தல்) பக்கத்திற்கு செல்லவும்

<img src="../../../translated_images/ta/devcontainer.21126c9d6de64494.webp" alt="Screenshot: Dev container setup" width="50%">

<img src="../../../translated_images/ta/image-3.bf93d533bbc84268.webp" alt="Screenshot: Dev container build complete" width="50%">

#### விருப்பம் C: உங்கள் உள்ளூர் நிறுவலை பயன்படுத்துதல்

**ஊடுருவலான Java சூழலுடன் உள்ளூர் பைரவர்கள்**

முன் நிபந்தனைகள்:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html) 
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) அல்லது உங்கள் விருப்பமான IDE

படி:
1. இந்த நிரலமைப்பை உள்ளூர் கணினிக்கு Clone செய்யவும்
2. உங்கள் IDE இல் திட்டத்தை திறக்கவும்
3. [படி 2: Azure AI Foundry அமைத்தல்](#படி-2-azure-ai-foundry-அமைத்தல்) பக்கத்திற்கு செல்லவும்

> **திறமையான குறிப்பு**: குறைந்த திறன் வாய்ந்த கணினி இருந்தாலும் உங்கள் உள்ளூர் VS Code க்கான மேகத்தொகுதி Codespaces க்கு இணைக்கலாம், இதனால் இரண்டின் சிறப்பும் பெறலாம்.

<img src="../../../translated_images/ta/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: created local devcontainer instance" width="50%">


## படி 2: Azure AI Foundry அமைத்தல்

இந்த பாடத்திற்கான AI மாதிரிகளை Azure AI Foundry இல் குறியீட்டாக இடுகை செய்யும். நிரலமைப்பு ரூட்டில் இருந்து:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```

`azd` சூழல் பெயர் மற்றும் மண்டலத்தை கேட்டு, `gpt-4o-mini` மற்றும் `text-embedding-3-small` இடுகைகளுடன் Azure AI Foundry கணக்கை ஒதுக்கி, எடுத்துக்காட்டு `.env` கோப்பில் endpoint ஐ எழுதும் — அனைத்தும் **keyless** அங்கீகாரத்துடன் (API விசைகள் எதுவும் இல்லை).

> **முழு வழிகாட்டி:** தேவைகள், கையேடு (portal) மாற்று, மண்டல வழிகாட்டி மற்றும் செலவு/தடை குறிப்புகளுக்கு [Azure AI Foundry Setup Guide](getting-started-azure-openai.md) ஐப் பாருங்கள்.

## படி 3: உங்கள் அமைப்பை பரிசோதித்தல்

உங்கள் Foundry மாதிரிகள் ஒதுக்கப்பட்ட பிறகு, உதாரண செயலிக்கு இணைந்து [ `02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) இல் உள்ள app ஐ பரிசோதிக்கவும்.

1. உங்கள் வளர்ச்சி சூழலில் டெர்மினலை திறக்கவும்.
2. எடுத்துக்காட்டுத் துறைக்கு செல்லவும்:
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
3. உள்நுழைந்துள்ளீர்களா என்பதை சரிபார்க்கவும் (keyless அங்கீகாரம் டோக்கன் தேவை):
   ```bash
   az login
   ```
   > `azd up` ஓட்டப்பட்டிருந்தால், உங்கள் endpoint உடன் `.env` கோப்பு ஏற்கனவே எழுதப்பட்டிருக்கும்.
4. செயலியை ஓட்டவும்:
   ```bash
   mvn clean spring-boot:run
   ```

`gpt-4o-mini` மாதிரியிலிருந்து பதில் காண்பீர்கள்.

### எடுத்துக்காட்டு குறியீட்டை புரிந்துகொள்வது

`examples/basic-chat-azure` இல் உள்ள எடுத்துக்காட்டு Spring Boot செயலி ஆகும், இது **Spring AI** ஐ பயன்படுத்தி Azure AI Foundry உடன் keyless authentication மூலம் இணைகிறது.

**இந்த குறியீடு செய்கிறது:**
- Azure AI Foundry உடன் உங்கள் Azure உள்நுழைவைக் கொண்டு (Microsoft Entra ID) இணைக்கிறது — API விசை இல்லை
- `gpt-4o-mini` மாதிரிக்கு ஒரு வேட்கையை அனுப்புகிறது
- AI பதிலை பெறுகிறது மற்றும் காட்டுகிறது
- உங்கள் அமைப்பு சரியாக செயல்படுவதை உறுதிப்படுத்துகிறது

**முக்கிய சார்பு** (`pom.xml` க்குள்):
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```

**கான்பிகரேஷன்** (`application.yml`):
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

## சாராம்சம்

உதயமானது! இப்போது நீங்கள் எல்லாவற்றையும் அமைத்துள்ளீர்கள்:

- Bicep + `azd` மூலம் Azure AI Foundry மாதிரிகளை குறியீட்டாக ஒதுக்கீடு செய்துள்ளீர்கள்
- உங்கள் Java வளர்ச்சி சூழல் இயங்கும் (Codespaces, dev containers அல்லது உள்ளூர் அமைப்பினாலும்)
- Azure AI Foundry உடன் keyless authentication (Microsoft Entra ID) மூலம் இணைந்துள்ளீர்கள் — எந்த API விசைகளும் இல்லை
- உங்கள் மாதிரிக்கு பேசும் எளிய எடுத்துக்காட்டுடன் அனைத்தும் சரியாக செயல்படுவதை சோதித்துள்ளீர்கள்

## அடுத்த படிகள்

[அத்தியாயம் 3: Core Generative AI Techniques](../03-CoreGenerativeAITechniques/README.md)

## சிக்கல்களை தீர்க்குதல்

சிக்கல்கள் வருகிறதா? பொதுவான பிரச்சனைகள் மற்றும் தீர்வுகள் இங்கே:

- **அங்கீகாரம் தோல்வியடைந்துவிட்டதா (401/403)?** 
  - `az login` ஓட்டவும் — அங்கீகாரம் keyless ஆகும், உள்நுழைவது அவசியம்
  - உங்கள் கணக்கிற்கு அந்த வளத்தில் **Cognitive Services OpenAI User** பங்கை வழங்கப்பட்டுள்ளது என்று உறுதிசெய்யவும்
  - சமீபத்தில் ஒதுக்கப்பட்டிருந்தால், பங்கு ஒதுக்கல் பரவ நிமிடம் காத்திருக்கவும்

- **Maven கிடைக்கவில்லை?** 
  - dev containers/Codespaces பயன்படுத்தினால், Maven முன்கூட்டியே நிறுவப்பட்டுள்ளது
  - உள்ளூர் அமைப்பில் Java 21+ மற்றும் Maven 3.9+ நிறுவவும்
  - நிறுவல் சரிபார்க்க `mvn --version` ஓட்டவும்

- **`azd` கிடைக்கவில்லை அல்லது provisioning தோல்வி?** 
  - [Azure Developer CLI](https://aka.ms/azure-dev/install) ஐ நிறுவி `azd auth login` ஓட்டவும்
  - `gpt-4o-mini` கிடைக்கும் மண்டலகுள் ஒன்று தேர்வு செய்யவும் (உதா: `eastus2`)
  - விவரங்களுக்கு [Azure AI Foundry setup guide](getting-started-azure-openai.md) பாருங்கள்

- **Dev container துவங்கவில்லை?** 
  - (உள்ளூர் வளர்ச்சிக்காக) Docker Desktop இயங்குவதை உறுதிப்படுத்தவும்
  - container கட்டமைப்பை மறுவமைக்கவும்: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **செயலியின் தொகுப்பு பிழைகள்?**
  - சரியான கோப்புறையில் இருக்கிறீர்கள் என்பதை உறுதிப்படுத்து: `02-SetupDevEnvironment/examples/basic-chat-azure`
  - சுத்தம் செய்து மறுதொடக்கம் செய்க: `mvn clean compile`

> **உதவி தேவை?**: இன்னும் சிக்கல்கள் உள்ளனவா? நிரலமைப்பில் issue திறந்து எங்களுக்கு தெரியப்படுத்துங்கள், உதவுவோம்.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**மறுப்பு**:
இந்த ஆவணம் AI மொழிபெயர்ப்பு சேவை [Co-op Translator](https://github.com/Azure/co-op-translator) பயன்படுத்தி மொழிபெயர்க்கப்பட்டுள்ளது. நாங்கள் துல்லியத்திற்காக முயற்சி செய்துள்ளோம், ஆனால் தானாக செய்யப்படும் மொழிபெயர்ப்புகளில் பிழைகள் அல்லது தவறுகள் இருக்கலாம் என்பதை கவனத்தில் கொள்ளவும். அசல் ஆவணம் அதன் தாய்மொழியில் அதிகாரப்பூர்வ ஆதாரமாக கருதப்பட வேண்டும். முக்கியமான தகவல்களுக்கு, தொழில்நுட்பமான மனித மொழிபெயர்ப்பு பரிந்துரைக்கப்படுகிறது. இந்த மொழிபெயர்ப்பைப் பயன்படுத்துவதால் ஏற்படும் எந்த தவறான புரிதல்கள் அல்லது தவறான விளக்கத்திற்கும் நாங்கள் பொறுப்பில்வில்லை.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->