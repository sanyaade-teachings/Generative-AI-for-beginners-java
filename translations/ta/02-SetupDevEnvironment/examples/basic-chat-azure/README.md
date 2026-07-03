# அடிப்படையான சாட் விமர்சனம் Azure AI Foundry உடன் - முழுமையான உதாரணம்

இந்த உதாரணம் ஒரு எளிய Spring Boot பயன்பாட்டாகும், இது **Azure AI Foundry** மாதிரியுடன் **keyless authentication** (Microsoft Entra ID) மூலம் இணைத்து, உங்கள் அமைப்பை சோதிக்கிறது. இது Spring AI யின் `ChatClient`-ஐ பயன்படுத்துகிறது.

## உள்ளடக்கப்பட்ட பட்டியல்

- [தேவையானவை](#தேவையானவை)
- [வேகமான துவக்கம்](#வேகமான-துவக்கம்)
- [அங்கீகாரம் எப்படி வேலை செய்கிறது](#அங்கீகாரம்-எப்படி-வேலை-செய்கிறது)
- [பயன்பாட்டை இயக்குதல்](#பயன்பாட்டை-இயக்குதல்)
  - [Maven பயன்படுத்துதல்](#maven-பயன்படுத்துதல்)
  - [VS Code பயன்படுத்துதல்](#vs-code-பயன்படுத்துதல்)
  - [எதிர்பார்க்கப்படும் வெளியீடு](#எதிர்பார்க்கப்படும்-வெளியீடு)
- [கான்பிகரேஷன் குறிப்பு](#கான்பிகரேஷன்-குறிப்பு)
  - [சுற்றுச்சூழல் மாறிலிகள்](#சுற்றுச்சூழல்-மாறிலிகள்)
  - [Spring அமைப்பு](#spring-அமைப்பு)
- [பிரச்சினைகள் மற்றும் தீர்வுகள்](#பிரச்சினைகள்-மற்றும்-தீர்வுகள்)
  - [பொதுவான பிரச்சினைகள்](#பொதுவான-பிரச்சினைகள்)
  - [டெபக் மோடு](#டெபக்-மோடு)
- [அடுத்த படிகள்](#அடுத்த-படிகள்)
- [வளங்கள்](#வளங்கள்)

## தேவையானவை

இந்த உதாரணத்தை இயக்குவதற்கு முன், பின்வருவன இருப்பதை உறுதி செய்யவும்:

- ஒரு Azure AI Foundry வளம் மற்றும் அதற்கான `gpt-4o-mini` வழங்கல் — இதனை `azd up` மூலம் அல்லது [Azure AI Foundry அமைப்பு கையேடு](../../getting-started-azure-openai.md) மூலம் கைமுறையாக ஏற்படுத்தவும்
- அந்த வளத்தில் **Cognitive Services OpenAI User** பங்கு (Bicep விபரக்கோப்புகள் இதை தானாக ஒதுக்குகிறது)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), `az login` மூலம் உள்நுழைந்திருக்க வேண்டும்
- Java 21+ மற்றும் Maven 3.9+

> **APi விசை தேவையில்லை** — அங்கீகாரம் Microsoft Entra ID மூலம் உள்ளீடற்ற முறையில் நடக்கிறது.

## வேகமான துவக்கம்

```bash
# 1. திட்டத்திற்குச் செல்லவும்
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. சான்றளிக்கவும், அதனால் keyless auth ஒரு டோக்கனைப் பெறலாம்
az login

# 3. கடைசிப் புள்ளியை அமைக்கவும்
#    - நீங்கள் `azd up` இயக்கித்து இருந்தால், .env உங்களுக்கு எழுதப்பட்டிருக்கும் (இந்த படியை தவிர்க்கவும்).
#    - இல்லையெனில், மாதிரியை நகலெடுத்து AZURE_OPENAI_ENDPOINT ஐ அமைக்கவும்:
cp .env.example .env

# 4. செயலியை இயக்கவும்
mvn spring-boot:run
```

## அங்கீகாரம் எப்படி வேலை செய்கிறது

இந்த உதாரணம் **Microsoft Entra ID** மூலம் அங்கிகாரம் செய்கிறது — API விசை இல்லை.

`spring.ai.azure.openai.endpoint` மட்டும் அமைக்கப்பட்டு (api-key இல்லாமல்) இருக்கும் போது, Spring AI `DefaultAzureCredential` மூலம் Azure OpenAI வாடிக்கையாளரை உருவாக்குகிறது. அந்த அங்கீகாரம் உங்கள் உள்ளூர் `az login` அமர்வு அல்லது Azure இல் இயங்கும் தனிமையான அடையாளத்திலிருந்து தானாக டோக்கனை பெறுகிறது — எனவே ஒரே குறியீடு எந்த இடத்திலும் மாற்றமின்றி இயங்கும்.

## பயன்பாட்டை இயக்குதல்

### Maven பயன்படுத்துதல்

```bash
mvn spring-boot:run
```

### VS Code பயன்படுத்துதல்

1. VS Code-ல் திட்டத்தைத் திறக்கவும்
2. `F5` அழுத்தவோ "Run and Debug" பலகையைப் பயன்படுத்தவோ செய்யவும்
3. "Spring Boot-BasicChatApplication" கட்டமைப்பைத் தேர்ந்தெடுக்கவும்

> **கவனம்**: VS Code கட்டமைப்பு தானாக உங்கள் .env கோப்பை ஏற்றுகிறது

### எதிர்பார்க்கப்படும் வெளியீடு

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

## கான்பிகரேஷன் குறிப்பு

### சுற்றுச்சூழல் மாறிலிகள்

| மாறி | விளக்கம் | அவசியம் | உதாரணம் |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | Foundry (Azure OpenAI) இடைநிலை URL | ஆம் | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | சாட் மாதிரி வழங்கல் பெயர் | இல்லை | `gpt-4o-mini` (இயல்புநிலை) |

> API விசை மாறி **இல்லை** — அங்கீகாரம் keyless (Microsoft Entra ID மூலம் `az login`).

### Spring அமைப்பு

`application.yml` கோப்பு தொகுத்துள்ளது:
- **இடைநிலை**: `${AZURE_OPENAI_ENDPOINT}` - சுற்றுச்சூழல் மாறியிலிருந்து
- **வழங்கல்**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - சுற்றுச்சூழல் மாறியிலிருந்து (Fallback உடன்)
- **அங்கீகாரம்**: keyless — `api-key` அமைக்கப்படவில்லை, எனவே Spring AI `DefaultAzureCredential` பயன்படுத்துகிறது
- **வெப்பநிலை**: `0.7` - புதுமையை கட்டுப்படுத்துகிறது (0.0 = தீர்மானிக்கப்பட்டது, 1.0 = புதுமையானது)
- **அதிகபட்ச டோக்கன்கள்**: `500` - அதிகபட்ச பதில் நீளம்

## பிரச்சினைகள் மற்றும் தீர்வுகள்

### பொதுவான பிரச்சினைகள்

<details>
<summary><strong>பிழை: 401 / "PermissionDenied" / டோக்கன் பிழைகள்</strong></summary>

- `az login` இயக்கவும் — keyless அங்கீகாரம் டோக்கனை பெற செயல்பாட்டிலான உள்நுழைவை தேவைப்படுகிறது
- உங்கள் கணக்கில் **Cognitive Services OpenAI User** பங்கு உள்ளதா என்பதை உறுதி செய்யவும்
- நீங்கள் பங்கு கொடுத்துவிட்டால், பரவ விரலுக்கு சில நிமிடங்கள் காத்திருக்கவும்
- நீங்கள் சரியான வீடு/சந்தாவிலிருக்கிறீர்களா என்பதை உறுதி செய்யவும் (`az account show`)
</details>

<details>
<summary><strong>பிழை: "The endpoint is not valid" / இணைப்பு பிழைகள்</strong></summary>

- `AZURE_OPENAI_ENDPOINT` முழு அடிப்படை URL ஆக இருக்க வேண்டும் (உதாரணம்: `https://your-resource.openai.azure.com/`)
- இடைநிலை URL பகுதி போதுமானதும் சரியானதும் இருந்திருப்பதை சரிபார்க்கவும்
- இடைநிலை உங்கள் வழங்கிய வளத்துடன் பொருந்துகிறதா என்பதை சரிபார்க்கவும் (`azd env get-values`)
</details>

<details>
<summary><strong>பிழை: "The deployment was not found"</strong></summary>

- `AZURE_OPENAI_DEPLOYMENT` Azure இல் வழங்கும் பெயருடன் பொருந்துகிறதா என்பதை சரிபார்க்கவும்
- மாதிரி வெற்றிகரமாக வெளியிடப்பட்டு செயல்பட்டுள்ளதாக இருக்கிறதா என உறுதி செய்யவும்
- இயல்புநிலை வழங்கல் பெயர் `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: சுற்றுச்சூழல் மாறிகள் ஏற்றப்படவில்லை</strong></summary>

- உங்கள் `.env` கோப்பு திட்டத்தின் முதன்மையான அடைவு (pom.xml உடன் ஒரே அளவு) இல் உள்ளது என்பதை உறுதி செய்யவும்
- VS Code ஒருங்கிணைக்கப்பட்ட டெர்மினலில் `mvn spring-boot:run` இயக்கு முயற்சிக்கவும்
- VS Code Java நீட்சியமைப்புகள் சரியாக நிறுவப்பட்டுள்ளதா என்பதை சரிபார்க்கவும்
</details>

### டெபக் மோடு

விரிவான பதிவு பதிவு செய்ய, `application.yml` இல் பின்வரும் வரிசைகளை செயலிழக்க விடவும்:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## அடுத்த படிகள்

**அமைப்பு முடிந்தது!** உங்கள் கற்றல் பயணத்தை தொடரவும்:

[Chapter 3: Core Generative AI Techniques](../../../03-CoreGenerativeAITechniques/README.md)

## வளங்கள்

- [Spring AI Azure OpenAI ஆவணம்](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Microsoft Entra ID மூலம் keyless authentication](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry க்கு போர்ட்டர்](https://ai.azure.com/)
- [Azure AI Foundry ஆவணம்](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**மறுப்பு**:
இந்த ஆவணம் AI மொழிபெயர்ப்பு சேவை [Co-op Translator](https://github.com/Azure/co-op-translator) பயன்படுத்தி மொழிபெயர்க்கப்பட்டுள்ளது. நாங்கள் துல்லியத்திற்காக முயற்சி செய்துள்ளோம், ஆனால் தானாக செய்யப்படும் மொழிபெயர்ப்புகளில் பிழைகள் அல்லது தவறுகள் இருக்கலாம் என்பதை கவனத்தில் கொள்ளவும். அசல் ஆவணம் அதன் தாய்மொழியில் அதிகாரப்பூர்வ ஆதாரமாக கருதப்பட வேண்டும். முக்கியமான தகவல்களுக்கு, தொழில்நுட்பமான மனித மொழிபெயர்ப்பு பரிந்துரைக்கப்படுகிறது. இந்த மொழிபெயர்ப்பைப் பயன்படுத்துவதால் ஏற்படும் எந்த தவறான புரிதல்கள் அல்லது தவறான விளக்கத்திற்கும் நாங்கள் பொறுப்பில்வில்லை.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->