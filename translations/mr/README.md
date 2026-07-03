# सुरुवातीसाठी जनरेटिव्ह AI - Java आवृत्ती
[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

![सुरुवातीसाठी जनरेटिव्ह AI - Java आवृत्ती](../../translated_images/mr/beg-genai-series.8b48be9951cc574c.webp)

**वेळेची बांधणी**: संपूर्ण कार्यशाळा ऑनलाइन, स्थानिक सेटअपशिवाय पूर्ण केली जाऊ शकते. पर्यावरण सेटअपसाठी २ मिनिटे लागतात, तर नमुने अभ्यासण्यासाठी १-३ तास लागतात, हे शोधण्यात लागणाऱ्या वेळेनुसार बदलते.

> **त्वरित सुरुवात**

1. या रेपॉझिटरीला आपल्या GitHub खात्यात Fork करा
2. क्लिक करा **Code** → **Codespaces** टॅब → **...** → **New with options...**
3. डिफॉल्ट वापरा – यामुळे या कोर्ससाठी तयार केलेला Development container निवडला जाईल
4. क्लिक करा **Create codespace**
5. पर्यावरण तयार होईपर्यंत सुमारे २ मिनिटे प्रतीक्षा करा
6. थेट जा [पहिल्या उदाहरणाकडे](./02-SetupDevEnvironment/README.md#step-2-create-a-github-personal-access-token)

## बहुभाषिक समर्थन

### GitHub Action द्वारे समर्थित (स्वयंचलित आणि सदैव अद्ययावत)

<!-- CO-OP TRANSLATOR LANGUAGES TABLE START -->
[अरबी](../ar/README.md) | [बंगाली](../bn/README.md) | [बल्गेरियन](../bg/README.md) | [बर्मी (म्यानमार)](../my/README.md) | [चिनी (साधी)](../zh-CN/README.md) | [चिनी (परंपरागत, हाँगकाँग)](../zh-HK/README.md) | [चिनी (परंपरागत, मकाऊ)](../zh-MO/README.md) | [चिनी (परंपरागत, तैवान)](../zh-TW/README.md) | [क्रोएशियन](../hr/README.md) | [चेक](../cs/README.md) | [डॅनिश](../da/README.md) | [डच](../nl/README.md) | [एस्टोनियन](../et/README.md) | [फिन्निश](../fi/README.md) | [फ्रेंच](../fr/README.md) | [जर्मन](../de/README.md) | [ग्रीक](../el/README.md) | [हिब्रू](../he/README.md) | [हिंदी](../hi/README.md) | [हंगेरियन](../hu/README.md) | [इंडोनेशियन](../id/README.md) | [इटालियन](../it/README.md) | [जपानी](../ja/README.md) | [कन्नड](../kn/README.md) | [खमेर](../km/README.md) | [कोरियन](../ko/README.md) | [लिथुआनियन](../lt/README.md) | [मलय](../ms/README.md) | [मलयाळम](../ml/README.md) | [मराठी](./README.md) | [नेपाली](../ne/README.md) | [नायजेरियन पिजिन](../pcm/README.md) | [नॉर्वेजियन](../no/README.md) | [पर्शियन (फारसी)](../fa/README.md) | [पोलिश](../pl/README.md) | [पोर्तुगीज (ब्राझील)](../pt-BR/README.md) | [पोर्तुगीज (पोर्तुगाल)](../pt-PT/README.md) | [पंजाबी (गुरमुखी)](../pa/README.md) | [रोमानियन](../ro/README.md) | [रशियन](../ru/README.md) | [सर्बियन (सिरिलिक)](../sr/README.md) | [स्लोव्हाक](../sk/README.md) | [स्लोव्हेनियन](../sl/README.md) | [स्पॅनिश](../es/README.md) | [स्वाहिली](../sw/README.md) | [स्वीडिश](../sv/README.md) | [टॅगलॉग (फिलिपिनो)](../tl/README.md) | [तमिळ](../ta/README.md) | [तेलुगू](../te/README.md) | [थाई](../th/README.md) | [तुर्किश](../tr/README.md) | [युक्रेनीयन](../uk/README.md) | [उर्दू](../ur/README.md) | [व्हिएतनामी](../vi/README.md)

> **स्थानिकरित्या क्लोन करायला प्राधान्य द्यायचं?**
>
> या रेपॉझिटरीमध्ये ५०+ भाषांतरं आहेत ज्यामुळे डाउनलोडचा आकार फार वाढतो. भाषांतरांशिवाय क्लोन करण्यासाठी sparse checkout वापरा:
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
> यामुळे कोर्स पूर्ण करण्यासाठी आवश्यक असलेले सर्व काही अधिक वेगाने डाउनलोड करता येईल.
<!-- CO-OP TRANSLATOR LANGUAGES TABLE END -->

## कोर्स संरचना आणि शिक्षण मार्ग

### **प्रकरण १: जनरेटिव्ह AI ची ओळख**
- **मुख्य संकल्पना**: मोठ्या भाषा मॉडेल्स, टोकन्स, एम्बेडिंग्ज आणि AI क्षमता समजावून घेणे
- **Java AI पर्यावरण**: Spring AI आणि OpenAI SDK चा आढावा
- **मॉडेल कॉन्टेक्स्ट प्रोटोकॉल**: MCP ची ओळख व AI एजंट संवादातील त्याचा उपयोग
- **व्यावहारिक वापर**: चॅटबॉट्स आणि कंटेंट जनरेशनसारख्या वास्तविक जगातील परिस्थिती
- **[→ प्रकरण १ सुरू करा](./01-IntroToGenAI/README.md)**

### **प्रकरण २: विकास पर्यावरण सेटअप**
- **Azure AI Foundry**: Bicep आणि Azure Developer CLI (azd) वापरून मॉडेल तैनाती कोडमध्ये तयार करा
- **Spring Boot + Spring AI**: एंटरप्राइज AI अॅप्लिकेशन विकासासाठी सर्वोत्तम प्रथांचे पालन
- **कीलेस प्रमाणीकरण**: Microsoft Entra ID द्वारे सुरक्षित कनेक्शन — API कीज नाहीत
- **विकास साधने**: Docker कंटेनर्स, VS Code, आणि GitHub Codespaces कॉन्फिगरेशन
- **[→ प्रकरण २ सुरू करा](./02-SetupDevEnvironment/README.md)**

### **प्रकरण ३: कोअर जनरेटिव्ह AI तंत्रे**
- **प्रॉम्प्ट इंजिनिअरिंग**: AI मॉडेल उत्तम प्रतिसादांसाठी तंत्रे
- **एम्बेडिंग्ज आणि व्हेक्टर ऑपरेशन्स**: सेमँटिक शोध आणि साम्य निवडक राबवणे
- **रिट्रीव्हल-ऑगमेंटेड जनरेशन (RAG)**: AI ला तुमच्या स्वतःच्या डेटाबेससह एकत्र करा
- **फंक्शन कॉलिंग**: कस्टम टूल्स आणि प्लगइन्ससह AI क्षमता वाढवा
- **[→ प्रकरण ३ सुरू करा](./03-CoreGenerativeAITechniques/README.md)**

### **प्रकरण ४: व्यावहारिक अनुप्रयोग व प्रकल्प**
- **पेट स्टोरी जनरेटर** (`petstory/`): Azure AI Foundry वापरून सर्जनशील कंटेंट निर्मिती
- **Foundry Local Demo** (`foundrylocal/`): OpenAI Java SDK सह स्थानिक AI मॉडेल एकत्रीकरण
- **MCP कॅलक्युलेटर सेवा** (`calculator/`): Spring AI सह बेसिक मॉडेल कॉन्टेक्स्ट प्रोटोकॉल अमलबजावणी
- **[→ प्रकरण ४ सुरू करा](./04-PracticalSamples/README.md)**

### **प्रकरण ५: जबाबदार AI विकास**
- **Azure AI Foundry कंटेंट सुरक्षा**: अंगभूत कंटेंट फिल्टरिंग आणि सुरक्षा यंत्रणा तपासा (हॉर्ड ब्लॉक्स आणि सौम्य नकार)
- **जबाबदार AI डेमो**: आधुनिक AI सुरक्षा प्रणाली प्रत्यक्षात कशी कार्य करते हे दर्शवणारे उदाहरण
- **सर्वोत्तम प्रथा**: नैतिक AI विकास आणि तैनातीसाठी आवश्यक मार्गदर्शक तत्त्वे
- **[→ प्रकरण ५ सुरू करा](./05-ResponsibleGenAI/README.md)**

## अतिरिक्त स्रोत

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

आपण अडकले असाल किंवा AI अप्लिकेशन्स तयार करताना काही प्रश्न असतील तर. इतर शिकणाऱ्या आणि अनुभवी विकासकांबरोबर MCP बद्दल चर्चा करा. ही एक सहायक समुदाय आहे जिथे प्रश्नांना स्वागत आहे आणि ज्ञान मोकळेपणाने वाटले जाते.

[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

जर आपल्याकडे उत्पादनाबाबत अभिप्राय किंवा बांधणी करताना त्रुटी असतील तर भेट द्या:

[![Microsoft Foundry Developer Forum](https://img.shields.io/badge/GitHub-Microsoft_Foundry_Developer_Forum-blue?style=for-the-badge&logo=github&color=000000&logoColor=fff)](https://aka.ms/foundry/forum)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**अस्वीकरण**:
हा दस्तऐवज AI भाषांतर सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) चा वापर करून अनुवादित केला आहे. जरी आम्ही अचूकतेसाठी प्रयत्न करतो, तरी कृपया लक्षात घ्या की स्वयंचलित भाषांतरांमध्ये त्रुटी किंवा अचूकतेची कमतरता असू शकते. मूळ दस्तऐवज त्याच्या मूळ भाषेत अधिकृत स्रोत मानला पाहिजे. महत्त्वाची माहिती असल्यास, व्यावसायिक मानवी भाषांतराची शिफारस केली जाते. या भाषांतराच्या वापरामुळे उद्भवणाऱ्या कोणत्याही गैरसमज किंवा चुकीच्या अर्थलावणीसाठी आम्ही जबाबदार नाही.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->