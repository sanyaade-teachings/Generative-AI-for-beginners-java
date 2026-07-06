# ஆரம்பிகளுக்கான உருவாக்கும் AI - ஜாவா பதிப்பு
[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

![Generative AI for Beginners - Java Edition](../../translated_images/ta/beg-genai-series.8b48be9951cc574c.webp)

**நேர ஒதுக்கீடு**: இந்த அனைத்து பணிமூலகமும் உள்ளூர்தான அமைப்பின்றி இணையத்தின் மூலம் முடிக்கலாம். சுற்றுச்சூழல் அமைக்க 2 நிமிடங்கள் ஆகும், மாதிரிகள் ஆய்வு அடிப்படையில் 1-3 மணி நேரம் தேவைப்படும்.

> **வேகமான துவக்கம்**

1. இந்த மாதிரிபட்டியை உங்கள் GitHub கணக்கிற்கு Fork செய்யவும்
2. கிளிக் செய்யவும் **Code** → **Codespaces** தாவல் → **...** → **New with options...**
3. இயல்பானவற்றைப் பயன்படுத்தவும் – இது இந்த பாடத்திற்கான Development container ஐத் தேர்வு செய்யும்
4. கிளிக் செய்யவும் **Create codespace**
5. சூழல் தயாராக ~2 நிமிடங்கள் காத்திருங்கள்
6. நேரடியாக செல் [அத்தியாயம் 2: Azure AI Foundry வழங்குதல்](./02-SetupDevEnvironment/README.md#step-2-provision-azure-ai-foundry)

## பன்மொழி ஆதரவு

### GitHub Action மூலம் ஆதரிக்கப்படுகிறது (தானியங்கிய மற்றும் எப்பொழுதும் புதுப்பிக்கப்படும்)

<!-- CO-OP TRANSLATOR LANGUAGES TABLE START -->
[அரபு](../ar/README.md) | [பெங்காலி](../bn/README.md) | [பல்கேரியன்](../bg/README.md) | [புர்மீஸ் (மியான்மை)](../my/README.md) | [சீனம் (எளிமைப்படுத்தப்பட்டது)](../zh-CN/README.md) | [சீனம் (பண்டைய, ஹாங்காங்)](../zh-HK/README.md) | [சீனம் (பண்டைய, மாகாவ்)](../zh-MO/README.md) | [சீனம் (பண்டைய, தைவான்)](../zh-TW/README.md) | [குரோஷியன்](../hr/README.md) | [செக்](../cs/README.md) | [டேனிஷ்](../da/README.md) | [டச்சு](../nl/README.md) | [எஸ்டோனியன்](../et/README.md) | [பினிஷ்](../fi/README.md) | [பிரஞ்சு](../fr/README.md) | [ஜெர்மன்](../de/README.md) | [கிரேக்கம்](../el/README.md) | [ஹீப்ரூ](../he/README.md) | [ஹிந்தி](../hi/README.md) | [ஹுங்கேரியன்](../hu/README.md) | [இந்தோனேசியன்](../id/README.md) | [இத்தாலியன்](../it/README.md) | [ஜப்பானியம்](../ja/README.md) | [கன்னட](../kn/README.md) | [க்மர்](../km/README.md) | [கொரியன்](../ko/README.md) | [லிடுவேனியன்](../lt/README.md) | [மலாய்](../ms/README.md) | [மலையாளம்](../ml/README.md) | [மராத்தி](../mr/README.md) | [நேபாளி](../ne/README.md) | [நைஜீரியன் பிட்ஜின்](../pcm/README.md) | [நார்வேஜியன்](../no/README.md) | [போர்சியன் (பார்ஸி)](../fa/README.md) | [போலிஷ்](../pl/README.md) | [போர்ச்சுகீஸ் (பிரேசில்)](../pt-BR/README.md) | [போர்ச்சுகீஸ் (போர்ச்சுகல்)](../pt-PT/README.md) | [பஞ்சாபி (குருமுகхи)](../pa/README.md) | [ரோமேனியன்](../ro/README.md) | [ரஷியன்](../ru/README.md) | [செர்பியன் (சிரிலிக்)](../sr/README.md) | [ஸ்லோவாக்](../sk/README.md) | [ஸ்லோவேனியன்](../sl/README.md) | [ஸ்பானிஷ்](../es/README.md) | [ஸ்வாலி](../sw/README.md) | [ஸ்வீடிஷ்](../sv/README.md) | [தமிழ்](./README.md) | [తెలుగు](../te/README.md) | [தை](../th/README.md) | [துருக்கி](../tr/README.md) | [உக்ரைனியன்](../uk/README.md) | [உருது](../ur/README.md) | [வியட்நாமீஸ்](../vi/README.md)

> **உள்ளூராகக் கிளோன் செய்ய விரும்புகிறீர்களா?**
>
> இந்த மாதிரிப்பகுதியில் 50+ மொழி மொழியாக்கங்கள் சேர்க்கப்பட்டுள்ளன, இது பதிவிறக்கும் அளவை பெரிதாக அதிகரிக்கிறது. மொழிபெயர்ப்புக்கள் இல்லாமல் கிளோன் செய்ய sparse checkout ஐப் பயன்படுத்தவும்:
>
> **Bash / macOS / Linux:**
> ```bash
> git clone --filter=blob:none --sparse https://github.com/microsoft/Generative-AI-for-beginners-java.git
> cd Generative-AI-for-beginners-java
> git sparse-checkout set --no-cone '/*' '!translations' '!translated_images'
> ```
>
> **CMD (Windows):**
> ```cmd
> git clone --filter=blob:none --sparse https://github.com/microsoft/Generative-AI-for-beginners-java.git
> cd Generative-AI-for-beginners-java
> git sparse-checkout set --no-cone "/*" "!translations" "!translated_images"
> ```
>
> இது உங்களுக்கு பாடத்தை முடிக்க தேவையான அனைத்தையும் மிகவும் விரைவான பதிவிறக்கம் உடன் வழங்கும்.
<!-- CO-OP TRANSLATOR LANGUAGES TABLE END -->

## பாடம் அமைப்பு & கல்வி பாதை

### **அத்தியாயம் 1: உருவாக்கும் AI அறிமுகம்**
- **முக்கிய கருத்துக்கள்**: பெரிய மொழி மாதிரிகள், டோக்கன்கள், ஆழ்மைகள் மற்றும் AI திறன்கள் புரிதல்
- **ஜாவா AI சூழல்**: Spring AI மற்றும் OpenAI SDKகள் அறிமுகம்
- **மாதிரி சூழல் நெறிமுறை**: MCP அறிமுகம் மற்றும் AI முகவர் தொடர்பில் பங்கு
- **நிகழ்நிலை பயன்பாடுகள்**: அரட்டை மெஷின்கள் மற்றும் உள்ளடக்க உருவாக்கம் போன்ற உலகைப் பொருந்தும் காட்சிகள்
- **[→ அத்தியாயம் 1 துவங்கு](./01-IntroToGenAI/README.md)**

### **அத்தியாயம் 2: மேம்பாட்டு சூழல் அமைப்பு**
- **Azure AI Foundry**: Bicep மற்றும் Azure Developer CLI (azd) மூலம் மாதிரி பராமரிப்பு வழங்கல்
- **Spring Boot + Spring AI**: நிறுவன AI செயல்பாடு மேம்பாட்டு சிறந்த நடைமுறைகள்
- **கீ இல்லா அங்கீகாரம்**: Microsoft Entra ID உடன் பாதுகாப்பாக இணைப்பு — API விசைகள் தேவையில்லை
- **மேம்பாட்டு கருவிகள்**: Docker கன்டெயினர்களும், VS Code, GitHub Codespaces அமைப்பும்
- **[→ அத்தியாயம் 2 துவங்கு](./02-SetupDevEnvironment/README.md)**

### **அத்தியாயம் 3: மைய உருவாக்கும் AI தொழில்நுட்பங்கள்**
- **தூண்டுகோல் பொறியியல்**: சிறந்த AI மாதிரி பதில்கள் பெற மரபுகள்
- **ஆழ்மைகள் & வெக்டர் செயல்பாடுகள்**: அர்த்தவாய்ந்த தேடல் மற்றும் ஒத்திசைவு பொருத்தம் செயல்படுத்துதல்
- **திருப்பி-வாய்ந்த உருவாக்கம் (RAG)**: உங்கள் சொந்த தரவுகளுடன் AI ஐ இணைத்தல்
- **செயலி அழைப்பு**: தனிப்பயன் கருவிகளும் பிளகின்களும் கொண்டு AI திறன்களை விரிவாக்குதல்
- **[→ அத்தியாயம் 3 துவங்கு](./03-CoreGenerativeAITechniques/README.md)**

### **அத்தியாயம் 4: நடைமுறை பயன்பாடுகள் & திட்டங்கள்**
- **விலங்கு கதை உருவாக்கி** (`petstory/`): Azure AI Foundry உடன் படைப்பாற்றல் உள்ளடக்க உருவாக்கம்
- **Foundry உள்ளூர் டெமோ** (`foundrylocal/`): OpenAI ஜாவா SDK உடன் உள்ளூர் AI மாதிரி ஒருங்கிணைப்பு
- **MCPCalculator சேவை** (`calculator/`): Spring AI உடன் அடிப்படை மாதிரி சூழல் நெறிமுறை அமல்படுத்தல்
- **[→ அத்தியாயம் 4 துவங்கு](./04-PracticalSamples/README.md)**

### **அத்தியாயம் 5: பொறுப்பேற்ற AI மேம்பாடு**
- **Azure AI Foundry உள்ளடக்க பாதுகாப்பு**: கட்டமைக்கப்பட்ட உள்ளடக்க வடிகட்டுதல் மற்றும் பாதுகாப்பு முறைமைகள் (கடுமையான தடைகள் மற்றும் மெலிந்த மறுப்பு)
- **பொறுப்பான AI டெமோ**: நவீன AI பாதுகாப்பு முறைகள் நடைமுறையில் எப்படி செயல்படுகின்றன ஐ காட்டும் உதாரணம்
- **சிறந்த நடைமுறைகள்**: நெறிமுறை AI மேம்பாடு மற்றும் பராமரிப்புக்கான முக்கிய வழிகாட்டிகள்
- **[→ அத்தியாயம் 5 துவங்கு](./05-ResponsibleGenAI/README.md)**

## கூடுதல் வளங்கள்

<!-- CO-OP TRANSLATOR OTHER COURSES START -->
### LangChain
[![LangChain4j for Beginners](https://img.shields.io/badge/LangChain4j%20for%20Beginners-22C55E?style=for-the-badge&&labelColor=E5E7EB&color=0553D6)](https://aka.ms/langchain4j-for-beginners)
[![LangChain.js for Beginners](https://img.shields.io/badge/LangChain.js%20for%20Beginners-22C55E?style=for-the-badge&labelColor=E5E7EB&color=0553D6)](https://aka.ms/langchainjs-for-beginners?WT.mc_id=m365-94501-dwahlin)
[![LangChain for Beginners](https://img.shields.io/badge/LangChain%20for%20Beginners-22C55E?style=for-the-badge&labelColor=E5E7EB&color=0553D6)](https://github.com/microsoft/langchain-for-beginners?WT.mc_id=m365-94501-dwahlin)
---

### Azure / Edge / MCP / Agents
[![AZD for Beginners](https://img.shields.io/badge/AZD%20for%20Beginners-0078D4?style=for-the-badge&labelColor=E5E7EB&color=0078D4)](https://github.com/microsoft/AZD-for-beginners?WT.mc_id=academic-105485-koreyst)
[![Edge AI for Beginners](https://img.shields.io/badge/Edge%20AI%20for%20Beginners-00B8E4?style=for-the-badge&labelColor=E5E7EB&color=00B8E4)](https://github.com/microsoft/edgeai-for-beginners?WT.mc_id=academic-105485-koreyst)
[![MCP for Beginners](https://img.shields.io/badge/MCP%20for%20Beginners-009688?style=for-the-badge&labelColor=E5E7EB&color=009688)](https://github.com/microsoft/mcp-for-beginners?WT.mc_id=academic-105485-koreyst)
[![AI Agents for Beginners](https://img.shields.io/badge/AI%20Agents%20for%20Beginners-00C49A?style=for-the-badge&labelColor=E5E7EB&color=00C49A)](https://github.com/microsoft/ai-agents-for-beginners?WT.mc_id=academic-105485-koreyst)

---
 
### உருவாக்கும் AI தொடர்
[![Generative AI for Beginners](https://img.shields.io/badge/Generative%20AI%20for%20Beginners-8B5CF6?style=for-the-badge&labelColor=E5E7EB&color=8B5CF6)](https://github.com/microsoft/generative-ai-for-beginners?WT.mc_id=academic-105485-koreyst)
[![Generative AI (.NET)](https://img.shields.io/badge/Generative%20AI%20(.NET)-9333EA?style=for-the-badge&labelColor=E5E7EB&color=9333EA)](https://github.com/microsoft/Generative-AI-for-beginners-dotnet?WT.mc_id=academic-105485-koreyst)
[![Generative AI (Java)](https://img.shields.io/badge/Generative%20AI%20(Java)-C084FC?style=for-the-badge&labelColor=E5E7EB&color=C084FC)](https://github.com/microsoft/generative-ai-for-beginners-java?WT.mc_id=academic-105485-koreyst)
[![Generative AI (JavaScript)](https://img.shields.io/badge/Generative%20AI%20(JavaScript)-E879F9?style=for-the-badge&labelColor=E5E7EB&color=E879F9)](https://github.com/microsoft/generative-ai-with-javascript?WT.mc_id=academic-105485-koreyst)

---
 
### மையக் கற்றல்
[![ML for Beginners](https://img.shields.io/badge/ML%20for%20Beginners-22C55E?style=for-the-badge&labelColor=E5E7EB&color=22C55E)](https://aka.ms/ml-beginners?WT.mc_id=academic-105485-koreyst)
[![Data Science for Beginners](https://img.shields.io/badge/Data%20Science%20for%20Beginners-84CC16?style=for-the-badge&labelColor=E5E7EB&color=84CC16)](https://aka.ms/datascience-beginners?WT.mc_id=academic-105485-koreyst)
[![AI for Beginners](https://img.shields.io/badge/AI%20for%20Beginners-A3E635?style=for-the-badge&labelColor=E5E7EB&color=A3E635)](https://aka.ms/ai-beginners?WT.mc_id=academic-105485-koreyst)
[![Cybersecurity for Beginners](https://img.shields.io/badge/Cybersecurity%20for%20Beginners-F97316?style=for-the-badge&labelColor=E5E7EB&color=F97316)](https://github.com/microsoft/Security-101?WT.mc_id=academic-96948-sayoung)

[![Web Dev for Beginners](https://img.shields.io/badge/Web%20Dev%20for%20Beginners-EC4899?style=for-the-badge&labelColor=E5E7EB&color=EC4899)](https://aka.ms/webdev-beginners?WT.mc_id=academic-105485-koreyst)
[![IoT for Beginners](https://img.shields.io/badge/IoT%20for%20Beginners-14B8A6?style=for-the-badge&labelColor=E5E7EB&color=14B8A6)](https://aka.ms/iot-beginners?WT.mc_id=academic-105485-koreyst)
[![XR Development for Beginners](https://img.shields.io/badge/XR%20Development%20for%20Beginners-38BDF8?style=for-the-badge&labelColor=E5E7EB&color=38BDF8)](https://github.com/microsoft/xr-development-for-beginners?WT.mc_id=academic-105485-koreyst)

---
 
### கோபைலட் தொடர்
[![Copilot for AI Paired Programming](https://img.shields.io/badge/Copilot%20for%20AI%20Paired%20Programming-FACC15?style=for-the-badge&labelColor=E5E7EB&color=FACC15)](https://aka.ms/GitHubCopilotAI?WT.mc_id=academic-105485-koreyst)
[![Copilot for C#/.NET](https://img.shields.io/badge/Copilot%20for%20C%23/.NET-FBBF24?style=for-the-badge&labelColor=E5E7EB&color=FBBF24)](https://github.com/microsoft/mastering-github-copilot-for-dotnet-csharp-developers?WT.mc_id=academic-105485-koreyst)
[![Copilot Adventure](https://img.shields.io/badge/Copilot%20Adventure-FDE68A?style=for-the-badge&labelColor=E5E7EB&color=FDE68A)](https://github.com/microsoft/CopilotAdventures?WT.mc_id=academic-105485-koreyst)
<!-- CO-OP TRANSLATOR OTHER COURSES END -->

## உதவி பெறுதல்

நீங்கள் சிக்கிக் கொண்டால் அல்லது AI செயலிகளைக் கட்டமைப்பதில் ஏதேனும் கேள்விகள் இருந்தால், MCP குறித்து கலந்துரையாடல்களில் நண்பர்கள் மற்றும் அனுபவம் வாய்ந்த டெவலப்பர்களுடன் சேருங்கள். கேள்விகளை அமைதியான சமுதாயமாகவும் அறிவை சுதந்திரமாக பகிர்ந்துகொள்ளும் இடமாகவும் இது உள்ளது.

[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

தயாரிப்பின் கருத்துக்கள் அல்லது பிழைகள் இருந்தால், கட்டமைப்பின் போது கீழ்கண்ட இடத்திற்கு செல்லவும்:

[![Microsoft Foundry Developer Forum](https://img.shields.io/badge/GitHub-Microsoft_Foundry_Developer_Forum-blue?style=for-the-badge&logo=github&color=000000&logoColor=fff)](https://aka.ms/foundry/forum)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**மறுப்பு**:
இந்த ஆவணம் AI மொழிபெயர்ப்பு சேவை [Co-op Translator](https://github.com/Azure/co-op-translator) பயன்படுத்தி மொழிபெயர்க்கப்பட்டுள்ளது. நாங்கள் துல்லியத்திற்காக முயற்சி செய்துள்ளோம், ஆனால் தானாக செய்யப்படும் மொழிபெயர்ப்புகளில் பிழைகள் அல்லது தவறுகள் இருக்கலாம் என்பதை கவனத்தில் கொள்ளவும். அசல் ஆவணம் அதன் தாய்மொழியில் அதிகாரப்பூர்வ ஆதாரமாக கருதப்பட வேண்டும். முக்கியமான தகவல்களுக்கு, தொழில்நுட்பமான மனித மொழிபெயர்ப்பு பரிந்துரைக்கப்படுகிறது. இந்த மொழிபெயர்ப்பைப் பயன்படுத்துவதால் ஏற்படும் எந்த தவறான புரிதல்கள் அல்லது தவறான விளக்கத்திற்கும் நாங்கள் பொறுப்பில்வில்லை.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->