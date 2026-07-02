# Azure AI Foundry க்கான மேம்பாட்டு சூழலை அமைத்தல்

> இந்த வழிகாட்டி இந்த பாடத்திட்டத்தில் உள்ள ஜாவா AI செயலிகளுக்கான **Azure AI Foundry** மாதிரிகளை **keyless** அங்கீகாரத்துடன் (Microsoft Entra ID) அமைக்கிறது — நிர்வகிக்க API விசைகள் தேவையில்லை. கருவி முறையில் புதியவரா? [மேம்பாட்டு சூழல் வழிகாட்டியுடன்](./README.md) தொடங்குங்கள்.

இந்த வழிகாட்டி இந்த பாடத்திட்டத்தில் உள்ள ஜாவா AI செயலிகளுக்கான **Azure AI Foundry** மாதிரிகளை அமைக்கிறது. உங்களுக்கு இரண்டு பாதைகள் உள்ளன:

- **விருப்பம் A — `azd` + Bicep உடன் வழங்கல் (பரிந்துரைக்கப்படுகிறது):** ஒரு கட்டளை மூலம் Foundry கணக்கு மற்றும் மாதிரிகள் குறியீட்டாக ஐயமிடப்படுகின்றன. எந்த போர்டல் கிளிக்கலும் தேவையில்லை.
- **விருப்பம் B — Azure AI Foundry போர்டலில் உள்ள வளங்களை கையேடு மூலம் உருவாக்குதல்.**

இருபதும் **keyless அங்கீகாரத்தை** (Microsoft Entra ID) பயன்படுத்துகிறது — காபி செய்வதற்கும் கசிவதற்கும் API விசைகள் இல்லை.

## உள்ளடக்கம்

- [என்ன உருவாக்கப்படுகிறது](#என்ன-உருவாக்கப்படுகிறது)
- [முன் நபுட்படுத்தல்கள்](#முன்-நபுட்படுத்தல்கள்)
- [விருப்பம் A: azd + Bicep உடன் வழங்கல் (பரிந்துரைக்கப்பட்டது)](#option-a-provision-with-azd--bicep-recommended)
- [விருப்பம் B: வளங்களை கையேடு மூலம் உருவாக்குதல்](#விருப்பம்-b-வளங்களை-கையேடு-மூலம்-உருவாக்குதல்)
- [உங்கள் சூழலை அமைத்தல்](#உங்கள்-சூழலை-அமைத்தல்)
- [உங்கள் அமைப்பை சோதனை செய்தல்](#உங்கள்-அமைப்பை-சோதனை-செய்தல்)
- [அடுத்து என்ன?](#அடுத்து-என்ன)
- [வளங்கள்](#வளங்கள்)
- [கூடுதல் வளங்கள்](#கூடுதல்-வளங்கள்)

## என்ன உருவாக்கப்படுகிறது

[`infra/`](../../../02-SetupDevEnvironment/infra) என்ற Bicep வார்ப்புருக்கள்:

- ஒரு **Azure AI Foundry** கணக்கு (`Microsoft.CognitiveServices/accounts`, வகை `AIServices`) ஒரு திட்டத்துடன்
- ஒரு **chat** வழங்கல் — `gpt-4o-mini`
- ஒரு **embedding** வழங்கல் — `text-embedding-3-small` (பின்னர் மாநிலங்களில் பயன்படுத்தப்படும்)
- ஒரு **keyless роль ஒதுக்கீடு** (`Cognitive Services OpenAI User`) எனவே நீங்கள் விசைகளை நிர்வகிக்காமல் `az login` மூலம் நுழைய முடியும்

## முன் நபுட்படுத்தல்கள்

- ஒரு [Azure சந்தா](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) மற்றும் [Maven 3.9+](https://maven.apache.org/download.cgi)

## விருப்பம் A: azd + Bicep உடன் வழங்கல் (பரிந்துரைக்கப்பட்டது)

`02-SetupDevEnvironment` கோப்பகத்திலிருந்து:

```bash
cd 02-SetupDevEnvironment

# உள்நுழையவும் (இரு கருவிகளும்)
azd auth login
az login

# Foundry கணக்கையும் மாதிரி பராமரிப்புகளையும் ஏற்பாடு செய்யவும்
azd up
```

`azd` ஒரு **சூழல் பெயர்** (எடுத்துக்காட்டு `genai-java`) மற்றும் ஒரு **பகுதி** கேட்கும். `gpt-4o-mini` மற்றும் `text-embedding-3-small` கிடைக்கும் பகுதியை தேர்ந்தெடுக்கவும் — எடுத்துக்காட்டு `eastus2` அல்லது `swedencentral`.

வழங்கல் முடிந்தவுடன், azd:

1. [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep) இல் வரையறுக்கப்பட்ட அனைத்தையும் ஐயமிடுகிறது.
2. உங்கள் முடிவு மற்றும் வழங்கல் பெயர்களுடன் (இணைப்புகள் இல்லை) [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) எழுதும் ஒரு பிந்தி வழங்கல் கூக்கு செயல்படுத்துகிறது.

> **சூழல்:** மாற்றங்களைப் பிரயோகிக்க `azd up` ஐ எப்போதும் மீண்டும் இயக்குங்கள். எல்லாவற்றையும் அழிக்க `azd down` இயக்கவும்.

உற்பத்தி செய்யப்பட்ட அமைப்புகளை பார்க்க:

```bash
azd env get-values
```

இப்பொழுது [உங்கள் அமைப்பை சோதனை செய்யவும்](#உங்கள்-அமைப்பை-சோதனை-செய்தல்) செல்.

## விருப்பம் B: வளங்களை கையேடு மூலம் உருவாக்குதல்

போர்டலை விரும்புகிறீர்களா? வளங்களை கையேடாக உருவாக்க:

1. [Azure AI Foundry போர்டல்](https://ai.azure.com/) செல்லவும் மற்றும் நுழையவும்.
2. **ஒரு திட்டத்தை உருவாக்கவும்** (இதுவும் AI Foundry வளத்தை உருவாக்கும்). அதற்கு `GenAIJava` போன்ற பெயர் கொடுக்கவும்.
3. உங்கள் திட்டத்தில், **Models + endpoints** → **Deploy model** → **Deploy base model** திறக்கவும்.
4. **gpt-4o-mini** ஐ (வழங்கல் பெயர் `gpt-4o-mini`) வழங்கவும். நீங்கள் embedding எடுத்துக்காட்டுகளைப் பயன்படுத்த விரும்பினால் **text-embedding-3-small** ஐ மீண்டும் வழங்கவும்.
5. **Overview** இல் இருந்து **endpoint** ஐ (எடுத்துக்காட்டு `https://<resource>.openai.azure.com/`) நகலெடுக்கவும்.
6. விசையில்லா அணுகல் அளிக்க: வளத்தில் **Access control (IAM)** → **Add role assignment** → உங்கள் கணக்கிற்கு **Cognitive Services OpenAI User** ஒதுக்கவும்.

> **இனிமேல் பிரச்சனை இருந்தால்?** [Azure AI Foundry ஆவணங்களை](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects) பார்க்கவும்.

## உங்கள் சூழலை அமைத்தல்

**நீங்கள் விருப்பம் A (`azd up`) பயன்படுத்தியிருந்தால்**, உங்கள் அமைப்புப் கோப்பு ஏற்கனவே எழுதப்பட்டுள்ளது — எந்த அமைப்பும் தேவையில்லை. [உங்கள் அமைப்பை சோதனை செய்யவும்](#உங்கள்-அமைப்பை-சோதனை-செய்தல்) க்கு உடனே செல்.

**நீங்கள் விருப்பம் B (கையேடு) பயன்படுத்தினால்**, எடுத்துக்காட்டின் `.env` கோப்பை உங்கள் கையால் உருவாக்கி எழுதவும்:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

உங்கள் endpoint ஐ `.env` இல் தொகுக்கவும் (விசை இல்லை — அங்கீகாரம் keyless):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **பாதுகாப்பு குறிப்பு:** சேமிக்க API விசை இல்லை. நீங்கள் Microsoft Entra ID மூலம் `az login` (உங்கள் இயந்திரத்தில்) அல்லது மேலாண்மை அடையாளம் (Azure இல்) மூலம் அங்கீகரிக்கிறீர்கள். `.env` கோப்பில் இரகசியமற்ற அமைப்புகள் மட்டும் உள்ளன மற்றும் արդեն `.gitignore` மூலம் பாதுகாக்கப்பட்டுள்ளன.

## உங்கள் அமைப்பை சோதனை செய்தல்

விசையில்லா அங்கீகாரம் ஒரு டோக்கனை பெற நீங்கள் நுழைந்துள்ளீர்கள் என்பதை உறுதிப்படுத்தி, எடுத்துக்காட்டை இயக்கு:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # நீங்கள் ஏற்கனவே உள்நுழைக்கவில்லை என்றால்
mvn clean spring-boot:run
```

நீங்கள் `gpt-4o-mini` மாதிரியிலிருந்து பதிலை காண்வீர்கள்!

> **VS Code பயனர்கள்:** இயக்க F5 என்ற விசையை அழுத்தவும். செயலி உங்கள் `.env` ஐ தானாக ஏற்றுகிறது.

> **முழு எடுத்துக்காட்டு:** [Azure AI Foundry உடன் அடிப்படை உரையாடல் எடுத்துக்காட்டு](./examples/basic-chat-azure/README.md) விவரங்களுக்கும் பிழைத்திருத்தத்திற்கும்.

## அடுத்து என்ன?

**அமைப்பு முடிந்துச்சென்றது!** இப்போது உங்களுக்கு உள்ளது:
- `gpt-4o-mini` மற்றும் `text-embedding-3-small` உடன் Azure AI Foundry
- விசைகளின் தேவையில்லாத அங்கீகாரம் (Microsoft Entra ID)
- உங்கள் endpoint மற்றும் வழங்கல் பெயர்களுடன் உள்ளூர் `.env`
- தயாராக உள்ள ஜாவா மேம்பாட்டு சூழல்

**தொடரவும்** [அத்தியாயம் 3: மூல உயிரியியல் AI நுட்பங்கள்](../03-CoreGenerativeAITechniques/README.md) க்கு AI செயலிகளை உருவாக்க ஆரம்பிக்க!

## வளங்கள்

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Microsoft Entra ID உடன் Keyless அங்கீகாரம்](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry ஆவணங்கள்](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI ஆவணங்கள்](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI ஜாவா SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## கூடுதல் வளங்கள்

- [VS Code பதிவிறக்கு](https://code.visualstudio.com/Download)
- [Docker Desktop பெறுக](https://www.docker.com/products/docker-desktop)
- [Dev Container அமைப்பு](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**மறுப்பு**:
இந்த ஆவணம் AI மொழிபெயர்ப்பு சேவை [Co-op Translator](https://github.com/Azure/co-op-translator) பயன்படுத்தி மொழிபெயர்க்கப்பட்டுள்ளது. நாங்கள் துல்லியத்திற்காக முயற்சி செய்துள்ளோம், ஆனால் தானாக செய்யப்படும் மொழிபெயர்ப்புகளில் பிழைகள் அல்லது தவறுகள் இருக்கலாம் என்பதை கவனத்தில் கொள்ளவும். அசல் ஆவணம் அதன் தாய்மொழியில் அதிகாரப்பூர்வ ஆதாரமாக கருதப்பட வேண்டும். முக்கியமான தகவல்களுக்கு, தொழில்நுட்பமான மனித மொழிபெயர்ப்பு பரிந்துரைக்கப்படுகிறது. இந்த மொழிபெயர்ப்பைப் பயன்படுத்துவதால் ஏற்படும் எந்த தவறான புரிதல்கள் அல்லது தவறான விளக்கத்திற்கும் நாங்கள் பொறுப்பில்வில்லை.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->