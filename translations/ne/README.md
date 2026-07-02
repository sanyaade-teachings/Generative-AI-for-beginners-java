# शुरुवातीहरूका लागि जेनेरेटिभ एआई - जाभा संस्करण
[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

![Generative AI for Beginners - Java Edition](../../translated_images/ne/beg-genai-series.8b48be9951cc574c.webp)

**समय प्रतिबद्धता**: सम्पूर्ण कार्यशाला अनलाइन पूरा गर्न सकिन्छ बिना स्थानीय सेटअप। वातावरण सेटअप गर्न २ मिनेट लाग्छ, नमूनाहरू अन्वेषण गर्न १-३ घण्टा लाग्न सक्छ अन्वेषणको गहिराइ अनुसार।

> **छिटो सुरु**

1. यो रिपोजिटोरी तपाईंको GitHub खातामा फोर्क गर्नुहोस्
2. क्लिक गर्नुहोस् **Code** → **Codespaces** ट्याब → **...** → **New with options...**
3. पूर्वनिर्धारित विकल्पहरू प्रयोग गर्नुहोस् – यसले यस कोर्सको लागि बनाइएको विकास कन्टेनर चयन गर्नेछ
4. क्लिक गर्नुहोस् **Create codespace**
5. ~२ मिनेट पर्खनुहोस् जबसम्म वातावरण तयार हुन्छ
6. सिधै जानुहोस् [पहिलो उदाहरण](./02-SetupDevEnvironment/README.md#step-2-create-a-github-personal-access-token) मा

## बहुभाषी समर्थन

### GitHub Action मार्फत समर्थित (स्वचालित र सधैं अद्यावधिक)

<!-- CO-OP TRANSLATOR LANGUAGES TABLE START -->
[अरबी](../ar/README.md) | [बंगाली](../bn/README.md) | [बुल्गेरियन](../bg/README.md) | [बर्मेली (म्यानमार)](../my/README.md) | [चिनियाँ (सरलीकृत)](../zh-CN/README.md) | [चिनियाँ (परम्परागत, हङकङ)](../zh-HK/README.md) | [चिनियाँ (परम्परागत, मकाऊ)](../zh-MO/README.md) | [चिनियाँ (परम्परागत, ताइवान)](../zh-TW/README.md) | [क्रोएशियन](../hr/README.md) | [चेच](../cs/README.md) | [डेन्स](../da/README.md) | [डच](../nl/README.md) | [इस्टोनियन](../et/README.md) | [फिन्निश](../fi/README.md) | [फ्रेन्च](../fr/README.md) | [जर्मन](../de/README.md) | [ग्रीक](../el/README.md) | [हिब्रू](../he/README.md) | [हिन्दी](../hi/README.md) | [हङ्गेरियन](../hu/README.md) | [इन्डोनेशियाली](../id/README.md) | [इतालियन](../it/README.md) | [जापानी](../ja/README.md) | [कन्नडा](../kn/README.md) | [ख्मेर](../km/README.md) | [कोरियन](../ko/README.md) | [लिथुआनियन](../lt/README.md) | [मलय](../ms/README.md) | [मलयालम](../ml/README.md) | [मराठी](../mr/README.md) | [नेपाली](./README.md) | [नाइजेरियन पिजिन](../pcm/README.md) | [नर्वेगियन](../no/README.md) | [फारसी (पर्सियन)](../fa/README.md) | [पोलिश](../pl/README.md) | [पुर्तगाली (ब्राजिल)](../pt-BR/README.md) | [पुर्तगाली (पुर्खाहरु)](../pt-PT/README.md) | [पन्जाबी (गुरुमुखी)](../pa/README.md) | [रोमानियन](../ro/README.md) | [रूसी](../ru/README.md) | [सर्बियन (सिरिलिक)](../sr/README.md) | [स्लोभाक](../sk/README.md) | [स्लोभेनियन](../sl/README.md) | [स्पानिश](../es/README.md) | [स्वाहिली](../sw/README.md) | [स्विडिश](../sv/README.md) | [ट्यागालोग (फिलिपिनो)](../tl/README.md) | [तमिल](../ta/README.md) | [तेलुगु](../te/README.md) | [थाई](../th/README.md) | [टर्किश](../tr/README.md) | [युक्रेनीयन](../uk/README.md) | [उर्दू](../ur/README.md) | [भियतनामी](../vi/README.md)

> **स्थानीय रूपमा क्लोन गर्न चाहनुहुन्छ?**
>
> यस रिपोजिटोरीमा ५० भन्दा धेरै भाषा अनुवादहरू समावेश छन् जसले डाउनलोड साइज धेरै बढाउँछ। अनुवाद बिना क्लोन गर्न स्पार्स चेकआउट प्रयोग गर्नुहोस्:
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
> यसले तपाईंलाई कोर्स पूरा गर्न आवश्यक सबै कुरा छिटो डाउनलोड गर्दै दिन्छ।
<!-- CO-OP TRANSLATOR LANGUAGES TABLE END -->

## कोर्स संरचना र सिकाइ मार्ग

### **अध्याय १: जेनेरेटिभ एआई परिचय**
- **मूल अवधारणाहरू**: ठूला भाषा मोडेलहरू, टोकनहरू, एम्बेडिङहरू, र एआई क्षमताहरू बुझ्न
- **जाभा एआई इकोसिस्टम**: Spring AI र OpenAI SDKs को अवलोकन
- **मोडेल कन्टेक्स्ट प्रोटोकल**: MCP परिचय र AI एजेन्ट सञ्चारमा यसको भूमिका
- **व्यावहारिक अनुप्रयोगहरू**: चैटबोटहरू र सामग्री सिर्जना सहित वास्तविक संसारका परिदृश्यहरू
- **[→ अध्याय १ सुरु गर्नुहोस्](./01-IntroToGenAI/README.md)**

### **अध्याय २: विकास वातावरण सेटअप**
- **Azure AI Foundry**: Bicep र Azure Developer CLI (azd) सँग मोडेल तैनाती कोडको रूपमा प्रदान गर्नुहोस्
- **Spring Boot + Spring AI**: उद्यम एआई अनुप्रयोग विकासका लागि उत्तम अभ्यासहरू
- **कीलेस प्रमाणीकरण**: Microsoft Entra ID सँग सुरक्षित रूपमा जडान गर्नुहोस् — कुनै API कुञ्जी व्यवस्थापन आवश्यक छैन
- **विकास उपकरणहरू**: Docker कन्टेनरहरू, VS Code, र GitHub Codespaces कन्फिगरेसन
- **[→ अध्याय २ सुरु गर्नुहोस्](./02-SetupDevEnvironment/README.md)**

### **अध्याय ३: मूल जेनेरेटिभ एआई प्रविधिहरू**
- **प्रम्प्ट इन्जिनियरिङ**: एआई मोडेल प्रतिक्रियाहरूका लागि प्रविधिहरू
- **एम्बेडिङ र भेक्टर अपरेसनहरू**: सेमेन्टिक खोज र समानता मिलाउने कार्यान्वयन गर्नुहोस्
- **रिट्रिभल-अग्मेन्टेड जेनेरेसन (RAG)**: एआईलाई आफ्नै डाटास्रोतहरूसँग जोड्नुहोस्
- **फङ्क्सन कलिङ**: कस्टम उपकरण र प्लगइनहरूसँग एआई क्षमताहरू विस्तार गर्नुहोस्
- **[→ अध्याय ३ सुरु गर्नुहोस्](./03-CoreGenerativeAITechniques/README.md)**

### **अध्याय ४: व्यावहारिक अनुप्रयोग र परियोजनाहरू**
- **पेट स्टोरी जेनेरेटर** (`petstory/`): Azure AI Foundry सँग सिर्जनात्मक सामग्री निर्माण
- **फाउण्ड्री स्थानीय डेमो** (`foundrylocal/`): OpenAI Java SDK सँग स्थानीय एआई मोडेल एकीकरण
- **MCP क्याल्कुलेटर सेवा** (`calculator/`): Spring AI सँग आधारभूत मोडेल कन्टेक्स्ट प्रोटोकल कार्यान्वयन
- **[→ अध्याय ४ सुरु गर्नुहोस्](./04-PracticalSamples/README.md)**

### **अध्याय ५: जिम्मेवार एआई विकास**
- **Azure AI Foundry सामग्री सुरक्षा**: निर्माण गरिएको सामग्री फिल्टरिङ र सुरक्षा संयन्त्रहरू परीक्षण गर्नुहोस् (कठोर रोकथाम र सौम्य अस्वीकृति)
- **जिम्मेवार एआई डेमो**: आधुनिक एआई सुरक्षा प्रणालीहरूले कसरी काम गर्छन् भन्ने व्यवहारिक उदाहरण
- **उत्तम अभ्यासहरू**: नैतिक एआई विकास र तैनातीका लागि आवश्यक मार्गनिर्देशनहरू
- **[→ अध्याय ५ सुरु गर्नुहोस्](./05-ResponsibleGenAI/README.md)**

## थप स्रोतहरू

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
 
### Generative AI Series
[![Generative AI for Beginners](https://img.shields.io/badge/Generative%20AI%20for%20Beginners-8B5CF6?style=for-the-badge&labelColor=E5E7EB&color=8B5CF6)](https://github.com/microsoft/generative-ai-for-beginners?WT.mc_id=academic-105485-koreyst)
[![Generative AI (.NET)](https://img.shields.io/badge/Generative%20AI%20(.NET)-9333EA?style=for-the-badge&labelColor=E5E7EB&color=9333EA)](https://github.com/microsoft/Generative-AI-for-beginners-dotnet?WT.mc_id=academic-105485-koreyst)
[![Generative AI (Java)](https://img.shields.io/badge/Generative%20AI%20(Java)-C084FC?style=for-the-badge&labelColor=E5E7EB&color=C084FC)](https://github.com/microsoft/generative-ai-for-beginners-java?WT.mc_id=academic-105485-koreyst)
[![Generative AI (JavaScript)](https://img.shields.io/badge/Generative%20AI%20(JavaScript)-E879F9?style=for-the-badge&labelColor=E5E7EB&color=E879F9)](https://github.com/microsoft/generative-ai-with-javascript?WT.mc_id=academic-105485-koreyst)

---
 
### कोर सिकाइ
[![ML for Beginners](https://img.shields.io/badge/ML%20for%20Beginners-22C55E?style=for-the-badge&labelColor=E5E7EB&color=22C55E)](https://aka.ms/ml-beginners?WT.mc_id=academic-105485-koreyst)
[![डेटा विज्ञान शुरुवातीहरूका लागि](https://img.shields.io/badge/Data%20Science%20for%20Beginners-84CC16?style=for-the-badge&labelColor=E5E7EB&color=84CC16)](https://aka.ms/datascience-beginners?WT.mc_id=academic-105485-koreyst)
[![एआई शुरुवातीहरूका लागि](https://img.shields.io/badge/AI%20for%20Beginners-A3E635?style=for-the-badge&labelColor=E5E7EB&color=A3E635)](https://aka.ms/ai-beginners?WT.mc_id=academic-105485-koreyst)
[![साइबर सुरक्षा शुरुवातीहरूका लागि](https://img.shields.io/badge/Cybersecurity%20for%20Beginners-F97316?style=for-the-badge&labelColor=E5E7EB&color=F97316)](https://github.com/microsoft/Security-101?WT.mc_id=academic-96948-sayoung)

[![प्रारम्भकर्ताहरूका लागि वेब विकास](https://img.shields.io/badge/Web%20Dev%20for%20Beginners-EC4899?style=for-the-badge&labelColor=E5E7EB&color=EC4899)](https://aka.ms/webdev-beginners?WT.mc_id=academic-105485-koreyst)
[![प्रारम्भकर्ताहरूका लागि IoT](https://img.shields.io/badge/IoT%20for%20Beginners-14B8A6?style=for-the-badge&labelColor=E5E7EB&color=14B8A6)](https://aka.ms/iot-beginners?WT.mc_id=academic-105485-koreyst)
[![प्रारम्भकर्ताहरूका लागि XR विकास](https://img.shields.io/badge/XR%20Development%20for%20Beginners-38BDF8?style=for-the-badge&labelColor=E5E7EB&color=38BDF8)](https://github.com/microsoft/xr-development-for-beginners?WT.mc_id=academic-105485-koreyst)

---
 
### कपालट सिरिज
[![एआई जोडी प्रोग्रामिङको लागि कपालट](https://img.shields.io/badge/Copilot%20for%20AI%20Paired%20Programming-FACC15?style=for-the-badge&labelColor=E5E7EB&color=FACC15)](https://aka.ms/GitHubCopilotAI?WT.mc_id=academic-105485-koreyst)
[![C#/.NET का लागि कपालट](https://img.shields.io/badge/Copilot%20for%20C%23/.NET-FBBF24?style=for-the-badge&labelColor=E5E7EB&color=FBBF24)](https://github.com/microsoft/mastering-github-copilot-for-dotnet-csharp-developers?WT.mc_id=academic-105485-koreyst)
[![कपालट साहसिक](https://img.shields.io/badge/Copilot%20Adventure-FDE68A?style=for-the-badge&labelColor=E5E7EB&color=FDE68A)](https://github.com/microsoft/CopilotAdventures?WT.mc_id=academic-105485-koreyst)
<!-- CO-OP TRANSLATOR OTHER COURSES END -->

## सहयोग प्राप्त गर्नुहोस्

यदि तपाईं अलमलमा परेका हुनुहुन्छ वा एआई एप्स बनाउन सम्बन्धी कुनै प्रश्नहरू छन् भने। MCP बारे छलफलमा साथी सिक्नेहरू र अनुभवी विकासकर्ताहरुमा जोडिनुहोस्। यो एक सहायक समुदाय हो जहाँ प्रश्नहरू स्वागतयोग्य छन् र ज्ञान स्वतन्त्र रूपमा साझा गरिन्छ।

[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

यदि तपाईंलाई उत्पादमा प्रतिक्रिया वा त्रुटिहरू छन् भने, कृपया जानुहोस्:

[![Microsoft Foundry Developer Forum](https://img.shields.io/badge/GitHub-Microsoft_Foundry_Developer_Forum-blue?style=for-the-badge&logo=github&color=000000&logoColor=fff)](https://aka.ms/foundry/forum)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**अस्वीकरण**:
यो दस्तावेज़ AI अनुवाद सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) प्रयोग गरेर अनुवाद गरिएको हो। हामी सही हुन प्रयास गर्छौं, तर कृपया जानकार हुनुस् कि स्वचालित अनुवादमा त्रुटिहरू वा अशुद्धताहरू हुन सक्छन्। मूल दस्तावेज़ यसको मूल भाषामा आधिकारिक स्रोत मानिनुपर्छ। महत्वपूर्ण जानकारीका लागि व्यावसायिक मानव अनुवाद सिफारिस गरिन्छ। यस अनुवादको प्रयोगबाट उत्पन्न कुनै पनि गलत बुझाइ वा त्रुटिको लागि हामी जिम्मेवार छैनौं।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->