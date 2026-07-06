# ప్రారంభస్తుల కోసం జనరేటివ్ AI - జావా ఎడిషన్
[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

![ప్రారంభస్తుల కోసం జనరేటివ్ AI - జావా ఎడిషన్](../../translated_images/te/beg-genai-series.8b48be9951cc574c.webp)

**సమయం**: పూర్తిగా వర్క్‌షాప్ ఆన్‌లైన్‌లో స్థానిక సెటప్ లేకుండా పూర్తిచేయవచ్చు. పరిసర నియామకం 2 నిమిషాలు పడుతుంది, నమూనాలను అన్వేషించడంలో 1-3 గంటలు exploration లోతు ఆధారంగా అవసరం.

> **త్వరిత ప్రారంభం**

1. ఈ రిపోజిటరీని మీ GitHub ఖాతాకు Fork చేయండి  
2. **Code** → **Codespaces** ట్యాబ్ → **...** → **New with options...** క్లిక్ చేయండి  
3. అనుసరణ ఫలితాలను ఉపయోగించండి – ఇది కోర్సు కోసం రూపొందించిన Development container ను ఎంచుకుంటుంది  
4. **Create codespace** పై క్లిక్ చేయండి  
5. పరిసరాన్ని సిద్ధం అవ్వడానికి సుమారు 2 నిమిషాలు వేచి ఉండండి  
6. నేరుగా [అధ్యాయం 2: Azure AI Foundryని ప్రావిడ్షన్ చేయండి](./02-SetupDevEnvironment/README.md#step-2-provision-azure-ai-foundry) కు వెళ్లండి  

## బహుభాషా మద్దతు

### GitHub Action ద్వారా మద్దతు (స్వయంచాలక మరియు ఎప్పుడూ నవీకరణ)

<!-- CO-OP TRANSLATOR LANGUAGES TABLE START -->
[Arabic](../ar/README.md) | [Bengali](../bn/README.md) | [Bulgarian](../bg/README.md) | [Burmese (Myanmar)](../my/README.md) | [Chinese (Simplified)](../zh-CN/README.md) | [Chinese (Traditional, Hong Kong)](../zh-HK/README.md) | [Chinese (Traditional, Macau)](../zh-MO/README.md) | [Chinese (Traditional, Taiwan)](../zh-TW/README.md) | [Croatian](../hr/README.md) | [Czech](../cs/README.md) | [Danish](../da/README.md) | [Dutch](../nl/README.md) | [Estonian](../et/README.md) | [Finnish](../fi/README.md) | [French](../fr/README.md) | [German](../de/README.md) | [Greek](../el/README.md) | [Hebrew](../he/README.md) | [Hindi](../hi/README.md) | [Hungarian](../hu/README.md) | [Indonesian](../id/README.md) | [Italian](../it/README.md) | [Japanese](../ja/README.md) | [Kannada](../kn/README.md) | [Khmer](../km/README.md) | [Korean](../ko/README.md) | [Lithuanian](../lt/README.md) | [Malay](../ms/README.md) | [Malayalam](../ml/README.md) | [Marathi](../mr/README.md) | [Nepali](../ne/README.md) | [Nigerian Pidgin](../pcm/README.md) | [Norwegian](../no/README.md) | [Persian (Farsi)](../fa/README.md) | [Polish](../pl/README.md) | [Portuguese (Brazil)](../pt-BR/README.md) | [Portuguese (Portugal)](../pt-PT/README.md) | [Punjabi (Gurmukhi)](../pa/README.md) | [Romanian](../ro/README.md) | [Russian](../ru/README.md) | [Serbian (Cyrillic)](../sr/README.md) | [Slovak](../sk/README.md) | [Slovenian](../sl/README.md) | [Spanish](../es/README.md) | [Swahili](../sw/README.md) | [Swedish](../sv/README.md) | [Tagalog (Filipino)](../tl/README.md) | [Tamil](../ta/README.md) | [Telugu](./README.md) | [Thai](../th/README.md) | [Turkish](../tr/README.md) | [Ukrainian](../uk/README.md) | [Urdu](../ur/README.md) | [Vietnamese](../vi/README.md)

> **స్థానికంగా Clone చెయ్యాలని భావిస్తారా?**  
>  
> ఈ రిపోజిటరీలో 50+ భాషా అనువాదాలు ఉన్నాయి, ఇవి డౌన్లోడ్ పరిమాణాన్ని పెంచుతాయి. అనువాదాలు లేకుండా clone చేయడానికి sparse checkout ఉపయోగించండి:  
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
> ఇది కోర్సును పూర్తి చేయడానికి అవసరమైన అన్ని అంశాలను త్వరగా డౌన్లోడ్ చేసే రీతిలో అందిస్తుంది.  
<!-- CO-OP TRANSLATOR LANGUAGES TABLE END -->

## కోర్సు నిర్మాణం & నేర్చుకునే దారితి

### **అధ్యాయం 1: జనరేటివ్ AI పరిచయం**
- **ప్రధాన భావాలు**: పెద్ద భాషా మోడళ్లను, tokens, embeddings, AI సామర్థ్యాలను అర్థం చేసుకోవడం  
- **జావా AI ఇకోసిస్టం**: Spring AI మరియు OpenAI SDKలు అవలోకనం  
- **మోడల్ కాంటెక్స్ట్ ప్రోటోకాల్**: MCP పరిచయం మరియు AI ఏజెంట్ సంభాషణలో దాని పాత్ర  
- **ప్రయోగాత్మక అనువర్తనాలు**: చాట్‌బోట్లు మరియు కంటెంట్ త్రుటి ధరించిన వాస్తవ దృశ్యాలు  
- **[→ అధ్యాయం 1 ప్రారంభించండి](./01-IntroToGenAI/README.md)**

### **అధ్యాయం 2: అభివృద్ధి పరిసర ఏర్పాట్లు**
- **Azure AI Foundry**: Bicep, Azure Developer CLI (azd)తో కోడ్ రూపంలో మోడల్ మోపడం  
- **Spring Boot + Spring AI**: ఎంటర్‌ప్రైజ్ AI అప్లికేషన్ అభివృద్ధి ఉత్తమ విధానాలు  
- **కీలెస్ ప్రామాణీకరణ**: Microsoft Entra IDతో సురక్షితంగా కనెక్ట్ అవ్వడం — API కీలు అవసరం లేదు  
- **అభివృద్ధి సాధనాలు**: Docker కంటెయినర్లు, VS Code, GitHub Codespaces సెటప్  
- **[→ అధ్యాయం 2 ప్రారంభించండి](./02-SetupDevEnvironment/README.md)**

### **అధ్యాయం 3: ప్రాథమిక జనరేటివ్ AI సాంకేతికతలు**
- **ప్రాంప్ట్ ఇంజనీరింగ్**: ఉత్తమ AI మోడల్ ప్రతిస్పందనల కోసం సాంకేతికతలు  
- **ఎంబెడింగ్స్ & వెక్టార్ ఆపరేషన్స్**: అర్థవంతమైన శోధన మరియు సారూప్యత సరిచేసుకోవడం  
- **రేట్రీవల్-ఆగ్మెంటెడ్ జనరేషన్ (RAG)**: AIని మీ స్వంత డేటా మూలాలతో కలపడం  
- **ఫంక్షన్ కాల్లింగ్**: కస్టమ్ టూల్స్ మరియు ప్లగిన్లతో AI సామర్థ్యాలను విస్తరించడం  
- **[→ అధ్యాయం 3 ప్రారంభించండి](./03-CoreGenerativeAITechniques/README.md)**

### **అధ్యాయం 4: ప్రయోగాత్మక అనువర్తనాలు & ప్రాజెక్టులు**
- **పెట్ స్టోరీ జనరేటర్** (`petstory/`): Azure AI Foundryతో సృజనాత్మక కంటెంట్ జనరేషన్  
- **Foundry లోకల్ డెమో** (`foundrylocal/`): OpenAI జావా SDKతో లోకల్ AI మోడల్ సమ్మేళనం  
- **MCP క్యాల్క్యులేటర్ సర్వీస్** (`calculator/`): Spring AIతో ప్రాథమిక మోడల్ కాంటెక్స్ట్ ప్రోటోకాల్ అమలు  
- **[→ అధ్యాయం 4 ప్రారంభించండి](./04-PracticalSamples/README.md)**

### **అధ్యాయం 5: బాధ్యతాయుత AI అభివృద్ధి**
- **Azure AI Foundry కంటెంట్ సేఫ్టీ**: అంతర్గత కంటెంట్ ఫిల్టరింగ్ మరియు సేఫ్టీ యాంత్రికత (ధృఢతా నిరోధాలు మరియు సున్నిత తవ్వకాలు)  
- **బాధ్యతాయుత AI డెమో**: ఆధునిక AI సేఫ్టీ వ్యవస్థలు ఎలా పనిచేస్తాయో ప్రాక్టికల్ ఉదాహరణ  
- **ఉత్తమ విధానాలు**: నైతిక AI అభివృద్ధి మరియు అమలుకు మార్గదర్శకాలు  
- **[→ అధ్యాయం 5 ప్రారంభించండి](./05-ResponsibleGenAI/README.md)**

## అదనపు వనరులు

<!-- CO-OP TRANSLATOR OTHER COURSES START -->
### లాంగ్‌చైన్  
[![ప్రారంభస్తులకు LangChain4j](https://img.shields.io/badge/LangChain4j%20for%20Beginners-22C55E?style=for-the-badge&&labelColor=E5E7EB&color=0553D6)](https://aka.ms/langchain4j-for-beginners)  
[![ప్రారంభస్తులకు LangChain.js](https://img.shields.io/badge/LangChain.js%20for%20Beginners-22C55E?style=for-the-badge&labelColor=E5E7EB&color=0553D6)](https://aka.ms/langchainjs-for-beginners?WT.mc_id=m365-94501-dwahlin)  
[![ప్రారంభస్తులకు LangChain](https://img.shields.io/badge/LangChain%20for%20Beginners-22C55E?style=for-the-badge&labelColor=E5E7EB&color=0553D6)](https://github.com/microsoft/langchain-for-beginners?WT.mc_id=m365-94501-dwahlin)  
---

### Azure / ఎడ్జ్ / MCP / ఏజెంట్లు  
[![ప్రారంభస్తులకు AZD](https://img.shields.io/badge/AZD%20for%20Beginners-0078D4?style=for-the-badge&labelColor=E5E7EB&color=0078D4)](https://github.com/microsoft/AZD-for-beginners?WT.mc_id=academic-105485-koreyst)  
[![ప్రారంభస్తులకు ఎడ్జ్ AI](https://img.shields.io/badge/Edge%20AI%20for%20Beginners-00B8E4?style=for-the-badge&labelColor=E5E7EB&color=00B8E4)](https://github.com/microsoft/edgeai-for-beginners?WT.mc_id=academic-105485-koreyst)  
[![ప్రారంభస్తులకు MCP](https://img.shields.io/badge/MCP%20for%20Beginners-009688?style=for-the-badge&labelColor=E5E7EB&color=009688)](https://github.com/microsoft/mcp-for-beginners?WT.mc_id=academic-105485-koreyst)  
[![ప్రారంభస్తులకు AI ఏజెంట్లు](https://img.shields.io/badge/AI%20Agents%20for%20Beginners-00C49A?style=for-the-badge&labelColor=E5E7EB&color=00C49A)](https://github.com/microsoft/ai-agents-for-beginners?WT.mc_id=academic-105485-koreyst)  

---

### జనరేటివ్ AI సిరీస్  
[![ప్రారంభస్తులకు జనరేటివ్ AI](https://img.shields.io/badge/Generative%20AI%20for%20Beginners-8B5CF6?style=for-the-badge&labelColor=E5E7EB&color=8B5CF6)](https://github.com/microsoft/generative-ai-for-beginners?WT.mc_id=academic-105485-koreyst)  
[![జనరేటివ్ AI (.NET)](https://img.shields.io/badge/Generative%20AI%20(.NET)-9333EA?style=for-the-badge&labelColor=E5E7EB&color=9333EA)](https://github.com/microsoft/Generative-AI-for-beginners-dotnet?WT.mc_id=academic-105485-koreyst)  
[![జనరేటివ్ AI (Java)](https://img.shields.io/badge/Generative%20AI%20(Java)-C084FC?style=for-the-badge&labelColor=E5E7EB&color=C084FC)](https://github.com/microsoft/generative-ai-for-beginners-java?WT.mc_id=academic-105485-koreyst)  
[![జనరేటివ్ AI (JavaScript)](https://img.shields.io/badge/Generative%20AI%20(JavaScript)-E879F9?style=for-the-badge&labelColor=E5E7EB&color=E879F9)](https://github.com/microsoft/generative-ai-with-javascript?WT.mc_id=academic-105485-koreyst)  

---

### ప్రాథమిక అధ్యయనాలు  
[![ప్రారంభస్తులకు యంత్ర అభ్యాసం](https://img.shields.io/badge/ML%20for%20Beginners-22C55E?style=for-the-badge&labelColor=E5E7EB&color=22C55E)](https://aka.ms/ml-beginners?WT.mc_id=academic-105485-koreyst)  
[![ప్రారంభస్తులకు డేటా సైన్స్](https://img.shields.io/badge/Data%20Science%20for%20Beginners-84CC16?style=for-the-badge&labelColor=E5E7EB&color=84CC16)](https://aka.ms/datascience-beginners?WT.mc_id=academic-105485-koreyst)  
[![ప్రారంభస్తులకు AI](https://img.shields.io/badge/AI%20for%20Beginners-A3E635?style=for-the-badge&labelColor=E5E7EB&color=A3E635)](https://aka.ms/ai-beginners?WT.mc_id=academic-105485-koreyst)  
[![ప్రారంభస్తులకు సైబర్ సెక్యూరిటీ](https://img.shields.io/badge/Cybersecurity%20for%20Beginners-F97316?style=for-the-badge&labelColor=E5E7EB&color=F97316)](https://github.com/microsoft/Security-101?WT.mc_id=academic-96948-sayoung)  

[![Web Dev for Beginners](https://img.shields.io/badge/Web%20Dev%20for%20Beginners-EC4899?style=for-the-badge&labelColor=E5E7EB&color=EC4899)](https://aka.ms/webdev-beginners?WT.mc_id=academic-105485-koreyst)
[![IoT for Beginners](https://img.shields.io/badge/IoT%20for%20Beginners-14B8A6?style=for-the-badge&labelColor=E5E7EB&color=14B8A6)](https://aka.ms/iot-beginners?WT.mc_id=academic-105485-koreyst)
[![XR Development for Beginners](https://img.shields.io/badge/XR%20Development%20for%20Beginners-38BDF8?style=for-the-badge&labelColor=E5E7EB&color=38BDF8)](https://github.com/microsoft/xr-development-for-beginners?WT.mc_id=academic-105485-koreyst)

---
 
### కోపైలట్ సిరీస్
[![Copilot for AI Paired Programming](https://img.shields.io/badge/Copilot%20for%20AI%20Paired%20Programming-FACC15?style=for-the-badge&labelColor=E5E7EB&color=FACC15)](https://aka.ms/GitHubCopilotAI?WT.mc_id=academic-105485-koreyst)
[![Copilot for C#/.NET](https://img.shields.io/badge/Copilot%20for%20C%23/.NET-FBBF24?style=for-the-badge&labelColor=E5E7EB&color=FBBF24)](https://github.com/microsoft/mastering-github-copilot-for-dotnet-csharp-developers?WT.mc_id=academic-105485-koreyst)
[![Copilot Adventure](https://img.shields.io/badge/Copilot%20Adventure-FDE68A?style=for-the-badge&labelColor=E5E7EB&color=FDE68A)](https://github.com/microsoft/CopilotAdventures?WT.mc_id=academic-105485-koreyst)
<!-- CO-OP TRANSLATOR OTHER COURSES END -->

## సహాయం పొందడం

మీరు ఎక్కడైనా చిక్కుకున్నా లేదా AI యాప్స్‌ను రూపొందించడంలో ఏవైనా ప్రశ్నలు ఉన్నా, MCP గురించి చర్చల్లో అనుభవజ్ఞులైన డెవలపర్లు మరియు ఇతర అభ్యర్థులతో చేరండి. ఇది ప్రశ్నలను స్వాగతించే, జ్ఞానాన్ని స్వేచ్ఛగా పంచుకునే మద్దతుగా ఉన్న సంఘమై ఉంది.

[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

మీకు ఉత్పత్తి సూచనలు లేదా నిర్మాణ సమయంలో దోషాలు ఉంటే, దయచేసి సందర్శించండి:

[![Microsoft Foundry Developer Forum](https://img.shields.io/badge/GitHub-Microsoft_Foundry_Developer_Forum-blue?style=for-the-badge&logo=github&color=000000&logoColor=fff)](https://aka.ms/foundry/forum)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**అస్వీకరణ**:
ఈ పత్రం AI అనువాద సేవ [Co-op Translator](https://github.com/Azure/co-op-translator) ఉపయోగించి అనువదించబడింది. మేము ఖచ్చితత్వానికి ప్రయత్నిస్తున్నప్పటికీ, ఆటోమేటెడ్ అనువాదాలు తప్పులు లేదా అసమగ్రతలను కలిగి ఉండవచ్చు. దాని స్వదేశ భాషలో ఉన్న అసలు పత్రాన్ని అధికారం కలిగిన మూలంగా పరిగణించాలి. కీలకమైన సమాచారం కోసం, ప్రొఫెషనల్ మానవ అనువాదాన్ని సిఫారసు చేస్తాము. ఈ అనువాదం ఉపయోగం వల్ల కలిగే ఏవైనా అపార్థాలు లేదా తప్పుదారులు కోసం మేము బాధ్యత వహించము.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->