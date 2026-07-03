# தொடக்கத்திற்கான உருவாக்கும் AI - ஜாவா பதிப்பு
[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

![Generative AI for Beginners - Java Edition](../../translated_images/ta/beg-genai-series.8b48be9951cc574c.webp)

**நேரம் ஒதுக்கீடு**: முழு வேலைத்திட்டத்தை உள்ளூரில் அமைப்பதின்றி ஆன்லைனில் முடிக்கலாம். சூழல் அமைப்பு 2 நிமிடங்கள் எடுக்கிறது, மாதிரிகளை ஆராய 1-3 மணி நேரம் தேவை, ஆராய்ச்சி ஆழத்தை நோக்கி மாறுபடும்.

> **விரைவில் தொடங்கவும்** 

1. இந்த ரெப்பொசிடரியை உங்கள் GitHub கணக்கில் Fork செய்யவும்
2. கிளிக் செய்யவும் **Code** → **Codespaces** தாவல் → **...** → **New with options...**
3. இயல்புகளை பயன்படுத்தவும் – இது இந்த பாடத்திற்கான Development container ஐ தேர்ந்தெடுக்கும்
4. கிளிக் செய்யவும் **Create codespace**
5. சூழல் தயார் ஆக சுமார் 2 நிமிடங்கள் காத்திருக்கவும்
6. நேரடியாக செல்லவும் [முதல் உதாரணம்](./02-SetupDevEnvironment/README.md#step-2-create-a-github-personal-access-token)

## பன்மொழி ஆதரவு

### GitHub Action மூலம் ஆதரவு (தானாகவும் எப்போதும் புதுப்பிக்கும்)

<!-- CO-OP TRANSLATOR LANGUAGES TABLE START -->
[अरबी](../ar/README.md) | [பெங்காளி](../bn/README.md) | [புல்கேரியன்](../bg/README.md) | [பர்மீஸ் (மியான்மர்)](../my/README.md) | [சீனம் (எளிமைப்படுத்தப்பட்டது)](../zh-CN/README.md) | [சீனம் (சங்கீதார்மிகம், ஹாங்காங்)](../zh-HK/README.md) | [சீனம் (சங்கீதார்மிகம், मकाऊ)](../zh-MO/README.md) | [சீனம் (சங்கீதார்மிகம், தைவான்)](../zh-TW/README.md) | [குரோஷியன்](../hr/README.md) | [செக்](../cs/README.md) | [டேனிஷ்](../da/README.md) | [டச்சு](../nl/README.md) | [எஸ்டோனியன்](../et/README.md) | [பின்னிஷ்](../fi/README.md) | [பிரெஞ்சு](../fr/README.md) | [ஜெர்மன்](../de/README.md) | [கிரேக்கு](../el/README.md) | [ஹேப்ரூ](../he/README.md) | [ஹிந்தி](../hi/README.md) | [ஹங்கேரியன்](../hu/README.md) | [இந்தோனேஷியன்](../id/README.md) | [இத்தாலியன்](../it/README.md) | [ஜப்பானியன்](../ja/README.md) | [கன்னடம்](../kn/README.md) | [க்மர்](../km/README.md) | [கொரியன்](../ko/README.md) | [லித்துவேனியன்](../lt/README.md) | [மலாய்](../ms/README.md) | [மலையாளம்](../ml/README.md) | [மறாத்தி](../mr/README.md) | [நேபாளி](../ne/README.md) | [நைஜீரியன் பிஜின்](../pcm/README.md) | [நார்வேஜியன்](../no/README.md) | [பேர்ஷியன் (பார்ஸி)](../fa/README.md) | [போலிஸ்](../pl/README.md) | [போர்ச்சுகீஸ் (பிரேஜில்)](../pt-BR/README.md) | [போர்ச்சுகீஸ் (போர்ச்சுகல்)](../pt-PT/README.md) | [பஞ்சாபி (குருமுகி)](../pa/README.md) | [ரோமானியன்](../ro/README.md) | [ரஷியன்](../ru/README.md) | [செர்பியன் (சிரிலிக்)](../sr/README.md) | [ஸ்லோவாக்](../sk/README.md) | [ஸ்லோவேனியன்](../sl/README.md) | [ஸ்பானிஷ்](../es/README.md) | [ஸ்வாஹிலி](../sw/README.md) | [ஸ்வீடிஷ்](../sv/README.md) | [டகாலோ (பிலிபின்)](../tl/README.md) | [தமிழ்](./README.md) | [తెలుగు](../te/README.md) | [தாய்](../th/README.md) | [துருக்கி](../tr/README.md) | [உக்ரைனியன்](../uk/README.md) | [உருது](../ur/README.md) | [வியட்நாமீஸ்](../vi/README.md)

> **உள்ளூரில் க்ளோன் செய்ய விரும்புகிறீர்களா?**
>
> இந்த ரெப்பொசிடரி 50+ மொழி மொழிபெயர்ப்புகளைக் கொண்டுள்ளது, இது பதிவிறக்கும் அளவை பெரும். மொழிபெயர்ப்புகள் இல்லாமல் க்ளோன் செய்ய sparse checkout பயன்படுத்தவும்:
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
> இது படிப்பை நிறைவேற்ற தேவையான அனைத்தையும் உடனடி பதிவிறக்கம் உடன் வழங்கும்.
<!-- CO-OP TRANSLATOR LANGUAGES TABLE END -->

## பாடம் அமைப்பு மற்றும் கற்றல் பாதை

### **அத்தியாயம் 1: உருவாக்கும் AIக்கு அறிமுகம்**
- **முக்கிய தத்துவங்கள்**: பெரிய மொழி மாதிரிகள், குறியீடுகள், ஊடுருவல்கள் மற்றும் AI திறன்கள் பற்றி புரிதல்
- **ஜாவது AI சூழல்**: Spring AI மற்றும் OpenAI SDK களின் பார்வை
- **மாதிரி இணை தொடர்பு நடைமுறை**: MCP பற்றிய அறிமுகம் மற்றும் AI முகவர் தொடர்பில் அதன் பங்கு
- **நடைமுறை பயன்பாடுகள்**: வர்த்தக முகாமைகள் மற்றும் உள்ளடக்க உருவாக்கம் போன்ற யூதர்வுகளில்
- **[→ அத்தியாயம் 1 துவக்கம்](./01-IntroToGenAI/README.md)**

### **அத்தியாயம் 2: வளர்ச்சி சூழல் அமைப்பு**
- **Azure AI Foundry**: Bicep மற்றும் Azure Developer CLI (azd) கொண்டு மாதிரி வினியோகங்களை கூட்டு முயற்சியில் உருவாக்குதல்
- **Spring Boot + Spring AI**: நிறுவன AI பயன்பாடு மேம்பாட்டுக்கான சிறந்த நடைமுறைகள்
- **Keyless Authentication**: Microsoft Entra ID மூலம் பாதுகாப்பாக இணைப்பு — API விசைகளை மேலாண்மை செய்ய தேவையில்லை
- **வளார்ச்சி கருவிகள்**: டாக்கர் கட்டைகள், VS Code, மற்றும் GitHub Codespaces அமைப்பு
- **[→ அத்தியாயம் 2 துவக்கம்](./02-SetupDevEnvironment/README.md)**

### **அத்தியாயம் 3: முக்கிய உருவாக்கும் AI தொழில்நுட்பங்கள்**
- **ப்ராம்ட் இன்ஜினீயரிங்**: சிறந்த AI மாதிரி பதில்களுக்கான தொழில்நுட்பங்கள்
- **ஊடுருவல்கள் மற்றும் வெக்டர் செயல்கள்**: பொருள் தேடல் மற்றும் ஒத்திருக்கும் பொருட்களை செயல்படுத்தல்
- **திரும்பப்பெறல்-அட்டவணைத்தற்போது உருவாக்கல் (RAG)**: உங்கள் தரவு மூலம் AI ஐ இணைத்தல்
- **நிகழ_FUNCTION_CALL_**: தனிப்பயன் கருவிகள் மற்றும் பிளகின்களுடன் AI திறன்களை விரிவாக்கல்
- **[→ அத்தியாயம் 3 துவக்கம்](./03-CoreGenerativeAITechniques/README.md)**

### **அத்தியாயம் 4: நடைமுறை பயன்பாடுகள் மற்றும் திட்டங்கள்**
- **விலங்கு கதை உருவாக்கி** (`petstory/`): Azure AI Foundry மூலம் படைப்பாற்றலடைந்த உள்ளடக்கம் உருவாக்கல்
- **Foundry உள்ளூர் டெமோ** (`foundrylocal/`): OpenAI ஜாவா SDK உடன் உள்ளூர் AI மாதிரி ஒருங்கிணைப்பு
- **MCP கணக்கியல் சேவை** (`calculator/`): Spring AI உடன் அடிப்படை மாதிரி இணை தொடர்பு நடைமுறை நடைமுறை
- **[→ அத்தியாயம் 4 துவக்கம்](./04-PracticalSamples/README.md)**

### **அத்தியாயம் 5: பொறுப்பு AI மேம்பாடு**
- **Azure AI Foundry உள்ளடக்க பாதுகாப்பு**: உள்ளடக்க கட்டுப்பாடு மற்றும் பாதுகாப்பு முறைகளை சோதனை செய்க (கடுமையான தடைகள் மற்றும் மென்மையான நிராகரிப்புகள்)
- **பொறுப்பு AI டெமோ**: நவீன AI பாதுகாப்பு அமைப்புகள் நடைமுறையில் எப்படி செயல்படுகின்றன என்பதை கையால் புரிய படுத்துதல்
- **சிறந்த நடைமுறைகள்**: நேர்மையான AI மேம்பாடு மற்றும் பொருத்தத்திற்கு முக்கிய வழிகாட்டிகள்
- **[→ அத்தியாயம் 5 துவக்கம்](./05-ResponsibleGenAI/README.md)**

## கூடுதல் வளங்கள்

<!-- CO-OP TRANSLATOR OTHER COURSES START -->
### LangChain
[![LangChain4j for Beginners](https://img.shields.io/badge/LangChain4j%20for%20Beginners-22C55E?style=for-the-badge&&labelColor=E5E7EB&color=0553D6)](https://aka.ms/langchain4j-for-beginners)
[![LangChain.js for Beginners](https://img.shields.io/badge/LangChain.js%20for%20Beginners-22C55E?style=for-the-badge&labelColor=E5E7EB&color=0553D6)](https://aka.ms/langchainjs-for-beginners?WT.mc_id=m365-94501-dwahlin)
[![LangChain for Beginners](https://img.shields.io/badge/LangChain%20for%20Beginners-22C55E?style=for-the-badge&labelColor=E5E7EB&color=0553D6)](https://github.com/microsoft/langchain-for-beginners?WT.mc_id=m365-94501-dwahlin)
---

### Azure / Edge / MCP / முகவர்கள்
[![AZD for Beginners](https://img.shields.io/badge/AZD%20for%20Beginners-0078D4?style=for-the-badge&labelColor=E5E7EB&color=0078D4)](https://github.com/microsoft/AZD-for-beginners?WT.mc_id=academic-105485-koreyst)
[![Edge AI for Beginners](https://img.shields.io/badge/Edge%20AI%20for%20Beginners-00B8E4?style=for-the-badge&labelColor=E5E7EB&color=00B8E4)](https://github.com/microsoft/edgeai-for-beginners?WT.mc_id=academic-105485-koreyst)
[![MCP for Beginners](https://img.shields.io/badge/MCP%20for%20Beginners-009688?style=for-the-badge&labelColor=E5E7EB&color=009688)](https://github.com/microsoft/mcp-for-beginners?WT.mc_id=academic-105485-koreyst)
[![AI Agents for Beginners](https://img.shields.io/badge/AI%20Agents%20for%20Beginners-00C49A?style=for-the-badge&labelColor=E5E7EB&color=00C49A)](https://github.com/microsoft/ai-agents-for-beginners?WT.mc_id=academic-105485-koreyst)

---
 
### உருவாக்கும் AI தொடர்கதை
[![Generative AI for Beginners](https://img.shields.io/badge/Generative%20AI%20for%20Beginners-8B5CF6?style=for-the-badge&labelColor=E5E7EB&color=8B5CF6)](https://github.com/microsoft/generative-ai-for-beginners?WT.mc_id=academic-105485-koreyst)
[![Generative AI (.NET)](https://img.shields.io/badge/Generative%20AI%20(.NET)-9333EA?style=for-the-badge&labelColor=E5E7EB&color=9333EA)](https://github.com/microsoft/Generative-AI-for-beginners-dotnet?WT.mc_id=academic-105485-koreyst)
[![Generative AI (Java)](https://img.shields.io/badge/Generative%20AI%20(Java)-C084FC?style=for-the-badge&labelColor=E5E7EB&color=C084FC)](https://github.com/microsoft/generative-ai-for-beginners-java?WT.mc_id=academic-105485-koreyst)
[![Generative AI (JavaScript)](https://img.shields.io/badge/Generative%20AI%20(JavaScript)-E879F9?style=for-the-badge&labelColor=E5E7EB&color=E879F9)](https://github.com/microsoft/generative-ai-with-javascript?WT.mc_id=academic-105485-koreyst)

---
 
### கர்நிலை கற்றல்
[![ML for Beginners](https://img.shields.io/badge/ML%20for%20Beginners-22C55E?style=for-the-badge&labelColor=E5E7EB&color=22C55E)](https://aka.ms/ml-beginners?WT.mc_id=academic-105485-koreyst)
[![Data Science for Beginners](https://img.shields.io/badge/Data%20Science%20for%20Beginners-84CC16?style=for-the-badge&labelColor=E5E7EB&color=84CC16)](https://aka.ms/datascience-beginners?WT.mc_id=academic-105485-koreyst)
[![AI for Beginners](https://img.shields.io/badge/AI%20for%20Beginners-A3E635?style=for-the-badge&labelColor=E5E7EB&color=A3E635)](https://aka.ms/ai-beginners?WT.mc_id=academic-105485-koreyst)
[![Cybersecurity for Beginners](https://img.shields.io/badge/Cybersecurity%20for%20Beginners-F97316?style=for-the-badge&labelColor=E5E7EB&color=F97316)](https://github.com/microsoft/Security-101?WT.mc_id=academic-96948-sayoung)
[![வலை மேம்பாடு ஆரம்பிகளுக்கானது](https://img.shields.io/badge/Web%20Dev%20for%20Beginners-EC4899?style=for-the-badge&labelColor=E5E7EB&color=EC4899)](https://aka.ms/webdev-beginners?WT.mc_id=academic-105485-koreyst)
[![IOT ஆரம்பிகளுக்கானது](https://img.shields.io/badge/IoT%20for%20Beginners-14B8A6?style=for-the-badge&labelColor=E5E7EB&color=14B8A6)](https://aka.ms/iot-beginners?WT.mc_id=academic-105485-koreyst)
[![XR மேம்பாடு ஆரம்பிகளுக்கானது](https://img.shields.io/badge/XR%20Development%20for%20Beginners-38BDF8?style=for-the-badge&labelColor=E5E7EB&color=38BDF8)](https://github.com/microsoft/xr-development-for-beginners?WT.mc_id=academic-105485-koreyst)

---
 
### கோப்பைலட் தொடர்
[![நண்பருடன் AI கூட்டுப்பrogramming குக்கான கோப்பைலட்](https://img.shields.io/badge/Copilot%20for%20AI%20Paired%20Programming-FACC15?style=for-the-badge&labelColor=E5E7EB&color=FACC15)](https://aka.ms/GitHubCopilotAI?WT.mc_id=academic-105485-koreyst)
[![C#/.NET குக்கான கோப்பைலட்](https://img.shields.io/badge/Copilot%20for%20C%23/.NET-FBBF24?style=for-the-badge&labelColor=E5E7EB&color=FBBF24)](https://github.com/microsoft/mastering-github-copilot-for-dotnet-csharp-developers?WT.mc_id=academic-105485-koreyst)
[![கோப்பைலட் சாகசம்](https://img.shields.io/badge/Copilot%20Adventure-FDE68A?style=for-the-badge&labelColor=E5E7EB&color=FDE68A)](https://github.com/microsoft/CopilotAdventures?WT.mc_id=academic-105485-koreyst)
<!-- CO-OP TRANSLATOR OTHER COURSES END -->

## உதவி பெறுதல்

நீங்கள் சிக்கலில் இருந்தால் அல்லது AI செயலிகளைக் கட்டமைப்பதின் தொடர்பான ஏதேனும் கேள்விகள் இருந்தால், MCP பற்றி விவாதங்களில் பங்கேற்க fellow learners மற்றும் அனுபவமுள்ள மேம்படுத்திகளுடன் சேரவும். இது கேள்விகள் வரவேற்கப்படும் மற்றும் அறிவு திறப்பாக பகிரப்படும் ஆதரவான சமுதாயம் ஆகும்.

[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

நீங்கள் தயாரிப்பு பற்றிய கருத்துகள் அல்லது பிழைகள் இருந்தால் கட்டமைப்பதின் போது காண:

[![Microsoft Foundry Developer Forum](https://img.shields.io/badge/GitHub-Microsoft_Foundry_Developer_Forum-blue?style=for-the-badge&logo=github&color=000000&logoColor=fff)](https://aka.ms/foundry/forum)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**மறுப்பு**:
இந்த ஆவணம் AI மொழிபெயர்ப்பு சேவை [Co-op Translator](https://github.com/Azure/co-op-translator) பயன்படுத்தி மொழிபெயர்க்கப்பட்டுள்ளது. நாங்கள் துல்லியத்திற்காக முயற்சி செய்துள்ளோம், ஆனால் தானாக செய்யப்படும் மொழிபெயர்ப்புகளில் பிழைகள் அல்லது தவறுகள் இருக்கலாம் என்பதை கவனத்தில் கொள்ளவும். அசல் ஆவணம் அதன் தாய்மொழியில் அதிகாரப்பூர்வ ஆதாரமாக கருதப்பட வேண்டும். முக்கியமான தகவல்களுக்கு, தொழில்நுட்பமான மனித மொழிபெயர்ப்பு பரிந்துரைக்கப்படுகிறது. இந்த மொழிபெயர்ப்பைப் பயன்படுத்துவதால் ஏற்படும் எந்த தவறான புரிதல்கள் அல்லது தவறான விளக்கத்திற்கும் நாங்கள் பொறுப்பில்வில்லை.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->