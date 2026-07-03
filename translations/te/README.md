# ప్రారంభికుల కోసం జనరేటివ్ AI - జావా ఎడిషన్
[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

![ప్రారంభికుల కోసం జనరేటివ్ AI - జావా ఎడిషన్](../../translated_images/te/beg-genai-series.8b48be9951cc574c.webp)

**సమయ ప్రతిబద్దత**: ఇంటర్నెట్ కనెక్షన్ ఉన్న చోట ఎలాంటి లోకల్ సెటప్ లేకుండా మొత్తం వర్క్‌షాప్ పూర్తి చేసుకోవచ్చు. వాతావరణం సెటప్ 2 నిమిషాలు పడుతుంది, సాంపిల్స్ ని అన్వేషించడానికి 1-3 గంటలు జరిగే అవకాశం ఉంటుంది, అన్వేషణ లోతుపైన ఆధారపడి.

> **త్వరిత ప్రారంభం**

1. ఈ రిపొజిటరీని మీ GitHub ఖాతాకు Fork చేయండి
2. **Code** → **Codespaces** ట్యాబ్ → **...** → **New with options...** పై క్లిక్ చేయండి
3. డిఫాల్ట్స్ ఉపయోగించండి – ఇది ఈ కోర్సు కోసం సృష్టించిన డెవలప్‌మెంట్ కంటైనర్‌ను ఎంచుకుంటుంది
4. **Create codespace** పై క్లిక్ చేయండి
5. వాతావరణం రేడీ అయ్యే వరకు సుమారు 2 నిమిషాలు వేచి ఉండండి
6. నేరుగా [మొదటి ఉదాహరణకు](./02-SetupDevEnvironment/README.md#step-2-create-a-github-personal-access-token) జంప్ అవ్వండి

## బహుభాషా మద్దతు

### GitHub Action ద్వారా మద్దతు (ఆటోమేటెడ్ & ఎప్పుడూ అప్డేట్‌కాబడి ఉండే)

<!-- CO-OP TRANSLATOR LANGUAGES TABLE START -->
[అరబిక్](../ar/README.md) | [బెంగాలీ](../bn/README.md) | [బడ్గారియన్](../bg/README.md) | [బర్మీస్ (మయన్మార్)](../my/README.md) | [చైనా (సింప్లిఫైడ్)](../zh-CN/README.md) | [చైనా (పారంపర్య, హాంకాంగ్)](../zh-HK/README.md) | [చైనా (పారంపర్య, మాకావ్)](../zh-MO/README.md) | [చైనా (పారంపర్య, తైవాన్)](../zh-TW/README.md) | [క్రోవేటియన్](../hr/README.md) | [చెక్](../cs/README.md) | [డానిష్](../da/README.md) | [డచ్](../nl/README.md) | [ఎస్టోనియన్](../et/README.md) | [ఫినిష్](../fi/README.md) | [ఫ్రెంచ్](../fr/README.md) | [జర్మన్](../de/README.md) | [గ్రీకు](../el/README.md) | [ఇబ్రాయీ](../he/README.md) | [హిందీ](../hi/README.md) | [హంజేరియన్](../hu/README.md) | [ఇండోనేషియన్](../id/README.md) | [ఇటాలియన్](../it/README.md) | [జపనీస్](../ja/README.md) | [కన్నడ](../kn/README.md) | [ఖ్మేర్](../km/README.md) | [కొరియన్](../ko/README.md) | [లిథువేనియన్](../lt/README.md) | [మలే](../ms/README.md) | [మలయాళం](../ml/README.md) | [మరాఠి](../mr/README.md) | [నేపాళీ](../ne/README.md) | [నైజీరియన్ పిగిన్](../pcm/README.md) | [నార్వేజియన్](../no/README.md) | [పర్షియన్ (ఫార్సీ)](../fa/README.md) | [పోలిష్](../pl/README.md) | [పోర్చుగీస్ (బ్రెజిల్)](../pt-BR/README.md) | [పోర్చుగీస్ (పోర్చుగల్)](../pt-PT/README.md) | [పంజాబీ (గుర్ముఖీ)](../pa/README.md) | [రోమాన్స్](../ro/README.md) | [రష్యన్](../ru/README.md) | [సెర్బియన్ (సిరిలిక్)](../sr/README.md) | [స్లోవాక్](../sk/README.md) | [స్లోవేనియన్](../sl/README.md) | [స్పానిష్](../es/README.md) | [స్వాహిలి](../sw/README.md) | [స్వీడిష్](../sv/README.md) | [తగాలాగ్ (ఫిలిపినో)](../tl/README.md) | [తమిళ్](../ta/README.md) | [తెలుగు](./README.md) | [థాయ్](../th/README.md) | [తుర్కిష్](../tr/README.md) | [ఉక్రేనియన్](../uk/README.md) | [ఉర్దూ](../ur/README.md) | [వియత్నామీస్](../vi/README.md)

> **లోకల్‌గా క్లోన్ చేయాలనుకుంటున్నారా?**
>
> ఈ రిపొజిటరీ 50+ ప్రాంతీయ భాషా అనువాదాలను కలిగి ఉంది, ఇది డౌన్లోడ్ సైజును ముఖ్యంగా పెంచుతుంది. అనువాదాల बिना క్లోన్ చేసుకోవడానికి, స్పార్స్ చెకౌట్ ఉపయోగించండి:
>
> **Bash / macOS / Linux:**
> ```bash
> git clone --filter=blob:none --sparse https://github.com/microsoft/Generative-AI-for-beginners-java.git
> cd Generative-AI-for-beginners-java
> git sparse-checkout set --no-cone '/*' '!translations' '!translated_images'
> ```
>
> **CMD (విండోస్):**
> ```cmd
> git clone --filter=blob:none --sparse https://github.com/microsoft/Generative-AI-for-beginners-java.git
> cd Generative-AI-for-beginners-java
> git sparse-checkout set --no-cone "/*" "!translations" "!translated_images"
> ```
>
> ఇది కోర్సులో పూర్తి చేయడానికి అన్ని అవసరాలనూ తక్కువ సమయ డౌన్లోడ్‌తో మీకు అందిస్తుంది.
<!-- CO-OP TRANSLATOR LANGUAGES TABLE END -->

## కోర్సు నిర్మాణం & నేర్చుకునే దారి

### **అధ్యాయం 1: జనరేటివ్ AI కు పరిచయం**
- **ప్రధాన భావనలు**: పెద్ద భాషా నమూనాలు, టోకెన్లు, ఏంబెడ్డింగ్స్, AI సామర్థ్యాలు అర్థం చేసుకోవడం
- **జావా AI ఎకోసిస్టమ్**: Spring AI మరియు OpenAI SDK ల సమీక్ష
- **మోడల్ కాంటెక్స్ట్ ప్రోటోకాల్**: MCP పరిచయం మరియు AI ఏజెంట్ కమ్యూనికేషన్‌లో పాత్ర
- **వాస్తవ అనువర్తనాలు**: చాట్‌బాట్లు మరియు కంటెంట్ సృష్టి సహా నిజ జీవిత సన్నివేశాలు
- **[→ అధ్యాయం 1 ప్రారంభించండి](./01-IntroToGenAI/README.md)**

### **అధ్యాయం 2: డెవలప్‌మెంట్ వాతావరణం సెటప్**
- **Azure AI Foundry**: Bicep మరియు Azure Developer CLI (azd) తో మోడల్ డిప్లాయ్‌మెంట్‌లను కోడుగా ప్రొవైజన్ చేయండి
- **Spring Boot + Spring AI**: ఎంటర్‌ప్రైజ్ AI అప్లికేషన్ అభివృద్ధి ఉత్తమ పద్ధతులు
- **కీలెస్ నిర్ధారణ**: Microsoft Entra ID తో సురక్షితంగా కనెక్ట్ అవ్వండి — ఏ API కీలు నిర్వహించాల్సిన అవసరం లేదు
- **డెవలప్‌మెంట్ టూల్స్**: Docker కంటైనర్లు, VS కోడ్, మరియు GitHub Codespaces కాన్ఫిగరేషన్
- **[→ అధ్యాయం 2 ప్రారంభించండి](./02-SetupDevEnvironment/README.md)**

### **అధ్యాయం 3: కోర్ జనరేటివ్ AI సాంకేతికతలు**
- **ప్రాంప్ట్ ఇంజినీరింగ్**: ఉత్తమ AI మోడల్ ప్రతిస్పందనల కోసం సాంకేతికతలు
- **ఏంబెడ్డింగ్స్ & వెక్టర్ ఆపరేషన్లు**: సెమాంటిక్ సెర్చ్ మరియు సారూప్య మ్యాచ్ అమలు చేయండి
- **రిట్రీవల్-ఆగ్మెంటెడ్ జనరేషన్ (RAG)**: AI ని మీ స్వంత డేటా మూలాలతో కలిపి ఉపయోగించండి
- **ఫంక్షన్ కాల్లింగ్**: కస్టమ్ టూల్స్ మరియు ప్లగిన్లతో AI సామర్థ్యాలు పెంచండి
- **[→ అధ్యాయం 3 ప్రారంభించండి](./03-CoreGenerativeAITechniques/README.md)**

### **అధ్యాయం 4: వాస్తవ అనువర్తనాలు & ప్రాజెక్టులు**
- **పెట్ స్టోరీ జనరేటర్** (`petstory/`): Azure AI Foundryతో సృజనాత్మక కంటెంట్ సృష్టి
- **Foundry Local డెమో** (`foundrylocal/`): OpenAI జావా SDKతో స్థానిక AI మోడల్ ఇంటిగ్రేషన్
- **MCP కాల్క్యులేటర్ సర్వీస్** (`calculator/`): Spring AIతో ప్రాథమిక మోడల్ కాంటెక్స్ట్ ప్రోటోకాల్ అమలు
- **[→ అధ్యాయం 4 ప్రారంభించండి](./04-PracticalSamples/README.md)**

### **అధ్యాయం 5: బాధ్యతాయుత AI అభివృద్ధి**
- **Azure AI Foundry కంటెంట్ సేఫ్టి**: బిల్ట్-ఇన్ కంటెంట్ ఫిల్టరింగ్ మరియు సేఫ్టి యంత్రాంగాలు (హార్డ్ బ్లాక్స్ మరియు సాఫ్ట్ రిఫ్యూజల్స్) పరీక్షించండి
- **బాధ్యతాయుత AI డెమో**: ఆధునిక AI సేఫ్టి వ్యవస్థలు ప్రాక్టీస్‌లో ఎలా పనిచేస్తాయో చూపించే హ్యాండ్స్-ఆన్ ఉదాహరణ
- **ఉత్తమ పద్ధతులు**: నైతిక AI అభివృద్ధి మరియు డిప్లాయ్‌మెంట్ కోసం ముఖ్య మార్గదర్శకాలు
- **[→ అధ్యాయం 5 ప్రారంభించండి](./05-ResponsibleGenAI/README.md)**

## అదనపు వనరులు

<!-- CO-OP TRANSLATOR OTHER COURSES START -->
### లాంగ్‌చైన్
[![ప్రారంభికుల కోసం LangChain4j](https://img.shields.io/badge/LangChain4j%20for%20Beginners-22C55E?style=for-the-badge&&labelColor=E5E7EB&color=0553D6)](https://aka.ms/langchain4j-for-beginners)
[![ప్రారంభికుల కోసం LangChain.js](https://img.shields.io/badge/LangChain.js%20for%20Beginners-22C55E?style=for-the-badge&labelColor=E5E7EB&color=0553D6)](https://aka.ms/langchainjs-for-beginners?WT.mc_id=m365-94501-dwahlin)
[![ప్రారంభికుల కోసం LangChain](https://img.shields.io/badge/LangChain%20for%20Beginners-22C55E?style=for-the-badge&labelColor=E5E7EB&color=0553D6)](https://github.com/microsoft/langchain-for-beginners?WT.mc_id=m365-94501-dwahlin)
---

### Azure / Edge / MCP / ఏజెంట్లు
[![ప్రారంభికుల కోసం AZD](https://img.shields.io/badge/AZD%20for%20Beginners-0078D4?style=for-the-badge&labelColor=E5E7EB&color=0078D4)](https://github.com/microsoft/AZD-for-beginners?WT.mc_id=academic-105485-koreyst)
[![ప్రారంభికుల కోసం Edge AI](https://img.shields.io/badge/Edge%20AI%20for%20Beginners-00B8E4?style=for-the-badge&labelColor=E5E7EB&color=00B8E4)](https://github.com/microsoft/edgeai-for-beginners?WT.mc_id=academic-105485-koreyst)
[![ప్రారంభికుల కోసం MCP](https://img.shields.io/badge/MCP%20for%20Beginners-009688?style=for-the-badge&labelColor=E5E7EB&color=009688)](https://github.com/microsoft/mcp-for-beginners?WT.mc_id=academic-105485-koreyst)
[![ప్రారంభికుల కోసం AI ఏజెంట్లు](https://img.shields.io/badge/AI%20Agents%20for%20Beginners-00C49A?style=for-the-badge&labelColor=E5E7EB&color=00C49A)](https://github.com/microsoft/ai-agents-for-beginners?WT.mc_id=academic-105485-koreyst)

---
 
### జనరేటివ్ AI సిరీస్
[![ప్రారంభికుల కోసం జనరేటివ్ AI](https://img.shields.io/badge/Generative%20AI%20for%20Beginners-8B5CF6?style=for-the-badge&labelColor=E5E7EB&color=8B5CF6)](https://github.com/microsoft/generative-ai-for-beginners?WT.mc_id=academic-105485-koreyst)
[![జనరేటివ్ AI (.NET)](https://img.shields.io/badge/Generative%20AI%20(.NET)-9333EA?style=for-the-badge&labelColor=E5E7EB&color=9333EA)](https://github.com/microsoft/Generative-AI-for-beginners-dotnet?WT.mc_id=academic-105485-koreyst)
[![జనరేటివ్ AI (జావా)](https://img.shields.io/badge/Generative%20AI%20(Java)-C084FC?style=for-the-badge&labelColor=E5E7EB&color=C084FC)](https://github.com/microsoft/generative-ai-for-beginners-java?WT.mc_id=academic-105485-koreyst)
[![జనరేటివ్ AI (జావాస్క్రిప్ట్)](https://img.shields.io/badge/Generative%20AI%20(JavaScript)-E879F9?style=for-the-badge&labelColor=E5E7EB&color=E879F9)](https://github.com/microsoft/generative-ai-with-javascript?WT.mc_id=academic-105485-koreyst)

---
 
### కోర్ లెర్నింగ్
[![ప్రారంభికుల కోసం ML](https://img.shields.io/badge/ML%20for%20Beginners-22C55E?style=for-the-badge&labelColor=E5E7EB&color=22C55E)](https://aka.ms/ml-beginners?WT.mc_id=academic-105485-koreyst)
[![ప్రారంభికుల కోసం డేటా సైన్స్](https://img.shields.io/badge/Data%20Science%20for%20Beginners-84CC16?style=for-the-badge&labelColor=E5E7EB&color=84CC16)](https://aka.ms/datascience-beginners?WT.mc_id=academic-105485-koreyst)
[![ప్రారంభికుల కోసం AI](https://img.shields.io/badge/AI%20for%20Beginners-A3E635?style=for-the-badge&labelColor=E5E7EB&color=A3E635)](https://aka.ms/ai-beginners?WT.mc_id=academic-105485-koreyst)
[![ప్రారంభికుల కోసం సైబర్‌సెక్యూరిటీ](https://img.shields.io/badge/Cybersecurity%20for%20Beginners-F97316?style=for-the-badge&labelColor=E5E7EB&color=F97316)](https://github.com/microsoft/Security-101?WT.mc_id=academic-96948-sayoung)

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

మీరు అడ్డుకున్నట్లయితే లేదా AI యాప్స్ నిర్మాణంపై ఏవైనా ప్రశ్నలు ఉంటే. MCP గురించి చర్చల్లో సహచర విద్యార్థులు మరియు అనుభవజ్ఞులైన డెవలపర్లతో చేరండి. ప్రశ్నలను స్వాగతించే, జ్ఞానాన్ని స్వేచ్ఛగా పంచుకునే ఒక సహాయక సమాజం ఇది.

[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

మీకు ప్రోడెక్ట్ అభిప్రాయం లేదా బిల్డింగ్ సమయంలో లోపాలు ఉంటే సందర్శించండి:

[![Microsoft Foundry Developer Forum](https://img.shields.io/badge/GitHub-Microsoft_Foundry_Developer_Forum-blue?style=for-the-badge&logo=github&color=000000&logoColor=fff)](https://aka.ms/foundry/forum)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**అస్వీకరణ**:
ఈ పత్రం AI అనువాద సేవ [Co-op Translator](https://github.com/Azure/co-op-translator) ఉపయోగించి అనువదించబడింది. మేము ఖచ్చితత్వానికి ప్రయత్నిస్తున్నప్పటికీ, ఆటోమేటెడ్ అనువాదాలు తప్పులు లేదా అసమగ్రతలను కలిగి ఉండవచ్చు. దాని స్వదేశ భాషలో ఉన్న అసలు పత్రాన్ని అధికారం కలిగిన మూలంగా పరిగణించాలి. కీలకమైన సమాచారం కోసం, ప్రొఫెషనల్ మానవ అనువాదాన్ని సిఫారసు చేస్తాము. ఈ అనువాదం ఉపయోగం వల్ల కలిగే ఏవైనా అపార్థాలు లేదా తప్పుదారులు కోసం మేము బాధ్యత వహించము.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->