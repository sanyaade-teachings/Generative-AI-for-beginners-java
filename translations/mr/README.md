# सुरूवातीच्या लोकांसाठी जनरेटिव्ह AI - Java आवृत्ती
[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

![Generative AI for Beginners - Java Edition](../../translated_images/mr/beg-genai-series.8b48be9951cc574c.webp)

**वेळेची बांधीलकी**: संपूर्ण कार्यशाळा स्थानिक सेटअपशिवाय ऑनलाइन पूर्ण केली जाऊ शकते. पर्यावरण सेटअपसाठी 2 मिनिटे लागतात, तर नमुन्यांचा शोध घेण्यासाठी संशोधनाच्या खोलीवर अवलंबून 1-3 तास लागतात.

> **जलद सुरुवात**

1. ह्या रेपॉजिटरीला तुमच्या GitHub खात्यावर Fork करा  
2. **Code** → **Codespaces** टॅब → **...** → **New with options...** क्लिक करा  
3. डीफॉल्ट्स वापरा – त्यामुळे या कोर्ससाठी तयार केलेली विकास कंटेनर निवडले जाईल  
4. **Create codespace** क्लिक करा  
5. पर्यावरण तयार होण्यासाठी सुमारे 2 मिनिटे थांबा  
6. थेट [Chapter 2: Provision Azure AI Foundry](./02-SetupDevEnvironment/README.md#step-2-provision-azure-ai-foundry) येथे जा

## बहुभाषिक समर्थन

### GitHub Action द्वारे समर्थित (स्वयंचलित आणि नेहमी अद्ययावत)

<!-- CO-OP TRANSLATOR LANGUAGES TABLE START -->
[अरबी](../ar/README.md) | [बंगाली](../bn/README.md) | [बुल्गेरियन](../bg/README.md) | [बर्मी (म्यानमार)](../my/README.md) | [चिनी (सोपे)](../zh-CN/README.md) | [चिनी (परंपरागत, हॉंगकॉंग)](../zh-HK/README.md) | [चिनी (परंपरागत, मकाउ)](../zh-MO/README.md) | [चिनी (परंपरागत, तैवान)](../zh-TW/README.md) | [क्रोएशियन](../hr/README.md) | [चेक](../cs/README.md) | [डॅनिश](../da/README.md) | [डच](../nl/README.md) | [एस्टोनियन](../et/README.md) | [फिन्निश](../fi/README.md) | [फ्रेंच](../fr/README.md) | [जर्मन](../de/README.md) | [ग्रीक](../el/README.md) | [हिब्रू](../he/README.md) | [हिंदी](../hi/README.md) | [हंगेरियन](../hu/README.md) | [इंडोनेशियन](../id/README.md) | [इटालियन](../it/README.md) | [जपानी](../ja/README.md) | [कन्नड](../kn/README.md) | [खमेर](../km/README.md) | [कोरियन](../ko/README.md) | [लिथुआनियन](../lt/README.md) | [मलय](../ms/README.md) | [मल्याळम](../ml/README.md) | [मराठी](./README.md) | [नेपाली](../ne/README.md) | [नायजेरियन पिडगिन](../pcm/README.md) | [नॉर्वेजियन](../no/README.md) | [फारसी (पर्शियन)](../fa/README.md) | [पोलीश](../pl/README.md) | [पॉर्चुगिझ (ब्राझील)](../pt-BR/README.md) | [पॉर्चुगिझ (पुर्तगाल)](../pt-PT/README.md) | [पंजाबी (गुरमुखी)](../pa/README.md) | [रोमानियन](../ro/README.md) | [रशियन](../ru/README.md) | [सर्बियन (सिरिलिक)](../sr/README.md) | [स्लोव्हाक](../sk/README.md) | [स्लोव्हेनियन](../sl/README.md) | [स्पॅनिश](../es/README.md) | [स्वाहिली](../sw/README.md) | [स्वीडिश](../sv/README.md) | [टॅगालॉग (फिलीपिनो)](../tl/README.md) | [तामिळ](../ta/README.md) | [तेलुगू](../te/README.md) | [थाई](../th/README.md) | [तुर्की](../tr/README.md) | [युक्रेनियन](../uk/README.md) | [उर्दू](../ur/README.md) | [व्हिएतनामी](../vi/README.md)

> **स्थानिकरीतीने क्लोन करायचे आहे का?**
> 
> ह्या रेपॉजिटरीमध्ये 50+ भाषांतील अनुवादांचा समावेश आहे ज्यामुळे डाउनलोड आकार खूप वाढतो. अनुवादांशिवाय क्लोन करण्यासाठी sparse checkout वापरा:
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
> हे तुम्हाला कोर्स पूर्ण करण्यासाठी आवश्यक सर्व काही देतो आणि डाउनलोड खूप जलद होते.
<!-- CO-OP TRANSLATOR LANGUAGES TABLE END -->

## कोर्सची रचना व शिकण्याचा मार्ग

### **अध्याय 1: जनरेटिव्ह AI मध्ये परिचय**
- **मूल संकल्पना**: लार्ज लँग्वेज मॉडेल्स, टोकन्स, एम्बेडिंग्ज आणि AI क्षमता समजून घेणे  
- **Java AI पारिस्थितिकी तंत्र**: Spring AI आणि OpenAI SDK चे विहंगावलोकन  
- **मॉडेल संदर्भ प्रोटोकॉल**: MCP चा परिचय आणि AI एजंट संवादात त्याची भूमिका  
- **प्रायोगिक अनुप्रयोग**: चॅटबॉट्स आणि सामग्री निर्मितीसारख्या वास्तविक-विश्वातील प्रकरणांची माहिती  
- **[→ अध्याय 1 सुरू करा](./01-IntroToGenAI/README.md)**

### **अध्याय 2: विकास पर्यावरण सेटअप**
- **Azure AI Foundry**: Bicep आणि Azure Developer CLI (azd) सह मॉडेल डिप्लॉयमेंट कोड स्वरूपात प्राव्हिजन करा  
- **Spring Boot + Spring AI**: एंटरप्राइझ AI अनुप्रयोग विकासासाठी सर्वोत्तम पद्धती  
- **कीलेस प्रमाणीकरण**: Microsoft Entra ID सह सुरक्षितपणे कनेक्ट करा — API कीज नको व्यवस्थापित करायच्या  
- **विकास साधने**: Docker कंटेनर, VS Code, आणि GitHub Codespaces सेटिंग्ज  
- **[→ अध्याय 2 सुरू करा](./02-SetupDevEnvironment/README.md)**

### **अध्याय 3: मूलभूत जनरेटिव्ह AI तंत्रे**
- **प्रॉम्प्ट इंजिनिअरिंग**: AI मॉडेलच्या उत्तम प्रतिसादांसाठी तंत्रे  
- **एम्बेडिंग्ज आणि व्हेक्टर ऑपरेशन्स**: सिमॅंटिक शोध आणि साम्य जुळणी करण्यासाठी अंमलबजावणी करा  
- **रिट्रीव्हल-अगमेंटेड जनरेशन (RAG)**: तुमच्या स्वतःच्या डेटास्त्रोतांसह AI एकत्र करा  
- **फंक्शन कॉलिंग**: कस्टम टूल्स आणि प्लगईन्ससह AI क्षमता वाढवा  
- **[→ अध्याय 3 सुरू करा](./03-CoreGenerativeAITechniques/README.md)**

### **अध्याय 4: व्यावहारिक अनुप्रयोग व प्रकल्प**
- **पाळीव प्राणी कथा जनरेटर** (`petstory/`): Azure AI Foundry सह सर्जनशील कंटेंट निर्मिती  
- **Foundry लोकल डेमो** (`foundrylocal/`): OpenAI Java SDK सह स्थानिक AI मॉडेल इंटीग्रेशन  
- **MCP कॅल्क्युलेटर सेवा** (`calculator/`): Spring AI वापरून मूलभूत Model Context Protocol अंमलबजावणी  
- **[→ अध्याय 4 सुरू करा](./04-PracticalSamples/README.md)**

### **अध्याय 5: जबाबदार AI विकास**
- **Azure AI Foundry कंटेंट सुरक्षितता**: अंतर्निर्मित कंटेंट फिल्टरिंग आणि सुरक्षितता यंत्रणा तपासा (कठोर ब्लॉक्स आणि सौम्य नाकारणे)  
- **जबाबदार AI डेमो**: आधुनिक AI सुरक्षितता प्रणाली कशी कार्य करते याचे प्रत्यक्ष उदाहरण  
- **सर्वोत्तम पद्धती**: नैतिक AI विकास आणि वितरणासाठी आवश्यक मार्गदर्शिका  
- **[→ अध्याय 5 सुरू करा](./05-ResponsibleGenAI/README.md)**

## अतिरिक्त संसाधने

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
 
### जनरेटिव्ह AI सिरीज
[![Generative AI for Beginners](https://img.shields.io/badge/Generative%20AI%20for%20Beginners-8B5CF6?style=for-the-badge&labelColor=E5E7EB&color=8B5CF6)](https://github.com/microsoft/generative-ai-for-beginners?WT.mc_id=academic-105485-koreyst)
[![Generative AI (.NET)](https://img.shields.io/badge/Generative%20AI%20(.NET)-9333EA?style=for-the-badge&labelColor=E5E7EB&color=9333EA)](https://github.com/microsoft/Generative-AI-for-beginners-dotnet?WT.mc_id=academic-105485-koreyst)
[![Generative AI (Java)](https://img.shields.io/badge/Generative%20AI%20(Java)-C084FC?style=for-the-badge&labelColor=E5E7EB&color=C084FC)](https://github.com/microsoft/generative-ai-for-beginners-java?WT.mc_id=academic-105485-koreyst)
[![Generative AI (JavaScript)](https://img.shields.io/badge/Generative%20AI%20(JavaScript)-E879F9?style=for-the-badge&labelColor=E5E7EB&color=E879F9)](https://github.com/microsoft/generative-ai-with-javascript?WT.mc_id=academic-105485-koreyst)

---
 
### मुख्य शिक्षण
[![ML for Beginners](https://img.shields.io/badge/ML%20for%20Beginners-22C55E?style=for-the-badge&labelColor=E5E7EB&color=22C55E)](https://aka.ms/ml-beginners?WT.mc_id=academic-105485-koreyst)
[![Data Science for Beginners](https://img.shields.io/badge/Data%20Science%20for%20Beginners-84CC16?style=for-the-badge&labelColor=E5E7EB&color=84CC16)](https://aka.ms/datascience-beginners?WT.mc_id=academic-105485-koreyst)
[![AI for Beginners](https://img.shields.io/badge/AI%20for%20Beginners-A3E635?style=for-the-badge&labelColor=E5E7EB&color=A3E635)](https://aka.ms/ai-beginners?WT.mc_id=academic-105485-koreyst)
[![Cybersecurity for Beginners](https://img.shields.io/badge/Cybersecurity%20for%20Beginners-F97316?style=for-the-badge&labelColor=E5E7EB&color=F97316)](https://github.com/microsoft/Security-101?WT.mc_id=academic-96948-sayoung)

[![Web Dev for Beginners](https://img.shields.io/badge/Web%20Dev%20for%20Beginners-EC4899?style=for-the-badge&labelColor=E5E7EB&color=EC4899)](https://aka.ms/webdev-beginners?WT.mc_id=academic-105485-koreyst)
[![IoT for Beginners](https://img.shields.io/badge/IoT%20for%20Beginners-14B8A6?style=for-the-badge&labelColor=E5E7EB&color=14B8A6)](https://aka.ms/iot-beginners?WT.mc_id=academic-105485-koreyst)
[![XR Development for Beginners](https://img.shields.io/badge/XR%20Development%20for%20Beginners-38BDF8?style=for-the-badge&labelColor=E5E7EB&color=38BDF8)](https://github.com/microsoft/xr-development-for-beginners?WT.mc_id=academic-105485-koreyst)

---
 
### Copilot Series
[![Copilot for AI Paired Programming](https://img.shields.io/badge/Copilot%20for%20AI%20Paired%20Programming-FACC15?style=for-the-badge&labelColor=E5E7EB&color=FACC15)](https://aka.ms/GitHubCopilotAI?WT.mc_id=academic-105485-koreyst)
[![Copilot for C#/.NET](https://img.shields.io/badge/Copilot%20for%20C%23/.NET-FBBF24?style=for-the-badge&labelColor=E5E7EB&color=FBBF24)](https://github.com/microsoft/mastering-github-copilot-for-dotnet-csharp-developers?WT.mc_id=academic-105485-koreyst)
[![Copilot Adventure](https://img.shields.io/badge/Copilot%20Adventure-FDE68A?style=for-the-badge&labelColor=E5E7EB&color=FDE68A)](https://github.com/microsoft/CopilotAdventures?WT.mc_id=academic-105485-koreyst)
<!-- CO-OP TRANSLATOR OTHER COURSES END -->

## मदत मिळवणे

जर तुम्हाला अडचण येत असेल किंवा AI ऍप्स तयार करताना काही प्रश्न असतील. MCP बद्दल चर्चा करण्यासाठी सह-शिकणाऱ्यांशी आणि अनुभवी विकसकांशी सामील व्हा. ही एक सहायक समुदाय आहे जिथे प्रश्न विचारले जातात आणि ज्ञान मोकळ्या मनाने शेअर केले जाते.

[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

तुमच्याकडे उत्पादनाबाबत अभिप्राय किंवा त्रुटी असल्यास येथे भेट द्या:

[![Microsoft Foundry Developer Forum](https://img.shields.io/badge/GitHub-Microsoft_Foundry_Developer_Forum-blue?style=for-the-badge&logo=github&color=000000&logoColor=fff)](https://aka.ms/foundry/forum)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**अस्वीकरण**:
हा दस्तऐवज AI भाषांतर सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) चा वापर करून अनुवादित केला आहे. जरी आम्ही अचूकतेसाठी प्रयत्न करतो, तरी कृपया लक्षात घ्या की स्वयंचलित भाषांतरांमध्ये त्रुटी किंवा अचूकतेची कमतरता असू शकते. मूळ दस्तऐवज त्याच्या मूळ भाषेत अधिकृत स्रोत मानला पाहिजे. महत्त्वाची माहिती असल्यास, व्यावसायिक मानवी भाषांतराची शिफारस केली जाते. या भाषांतराच्या वापरामुळे उद्भवणाऱ्या कोणत्याही गैरसमज किंवा चुकीच्या अर्थलावणीसाठी आम्ही जबाबदार नाही.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->