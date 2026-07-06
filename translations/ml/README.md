# തുടക്കക്കാര്‍ക്കുള്ള ജനറേറ്റീവ് AI - ജാവ എഡിഷന്‍  
[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

![Generative AI for Beginners - Java Edition](../../translated_images/ml/beg-genai-series.8b48be9951cc574c.webp)

**സമയം ബാധ്യത**: മുഴുവന്‍ വർക്ക്‌ഷോപ്പ് ലൊക്കല്‍ സെറ്റപ്പ് കൂടാതെ ഓൺലൈനിൽ പൂർത്തിയാക്കാവുന്നതാണ്. പരിസ്ഥിതിയുടെ സെറ്റപ്പ് 2 മിനിട്ട് സമയം എടുക്കും, സാമ്പിളുകൾ പര്യവേക്ഷണം ചെയ്യുന്നത് പര്യവേക്ഷണത്തിന്റെ ആഴമനുസരിച്ച് 1-3 മണിക്കൂർ എടുക്കാം.

> **ദ്രുത ആരംഭം** 

1. ഈ റിപ്പോസിറ്ററി നിങ്ങളുടെ GitHub അക്കൗണ്ടിലേക്ക് ഫോർക്കുചെയ്യുക  
2. ക്ലിക്കുചെയ്യുക **Code** → **Codespaces** ടാബ് → **...** → **New with options...**  
3. ഡിഫോൾട്ടുകൾ ഉപയോഗിക്കുക – ഇത് ഈ കോഴ്സിനായി സൃഷ്ടിച്ച ഡെവലപ്പ്മന്റ് കണ്ടെയ്‌നറെ തിരഞ്ഞെടുക്കും  
4. ക്ലിക്കുചെയ്യുക **Create codespace**  
5. പരിസ്ഥിതി സജ്ജമാകുന്നതിന് ഏകദേശം 2 മിനിട്ട് കാത്തിരിക്കുക  
6. നേരിട്ട് ചെന്ന് തുടങ്ങുക [അധ്യായം 2: Azure AI Foundry പ്രൊവിഷൻ ചെയ്യുക](./02-SetupDevEnvironment/README.md#step-2-provision-azure-ai-foundry)

## ബഹുഭാഷാ പിന്തുണ

### GitHub ആക്ഷൻ വഴി പിന്തുണ (Automation & എപ്പോഴും അപ്ഡേറ്റ്)

<!-- CO-OP TRANSLATOR LANGUAGES TABLE START -->
[Arabic](../ar/README.md) | [Bengali](../bn/README.md) | [Bulgarian](../bg/README.md) | [Burmese (Myanmar)](../my/README.md) | [Chinese (Simplified)](../zh-CN/README.md) | [Chinese (Traditional, Hong Kong)](../zh-HK/README.md) | [Chinese (Traditional, Macau)](../zh-MO/README.md) | [Chinese (Traditional, Taiwan)](../zh-TW/README.md) | [Croatian](../hr/README.md) | [Czech](../cs/README.md) | [Danish](../da/README.md) | [Dutch](../nl/README.md) | [Estonian](../et/README.md) | [Finnish](../fi/README.md) | [French](../fr/README.md) | [German](../de/README.md) | [Greek](../el/README.md) | [Hebrew](../he/README.md) | [Hindi](../hi/README.md) | [Hungarian](../hu/README.md) | [Indonesian](../id/README.md) | [Italian](../it/README.md) | [Japanese](../ja/README.md) | [Kannada](../kn/README.md) | [Khmer](../km/README.md) | [Korean](../ko/README.md) | [Lithuanian](../lt/README.md) | [Malay](../ms/README.md) | [Malayalam](./README.md) | [Marathi](../mr/README.md) | [Nepali](../ne/README.md) | [Nigerian Pidgin](../pcm/README.md) | [Norwegian](../no/README.md) | [Persian (Farsi)](../fa/README.md) | [Polish](../pl/README.md) | [Portuguese (Brazil)](../pt-BR/README.md) | [Portuguese (Portugal)](../pt-PT/README.md) | [Punjabi (Gurmukhi)](../pa/README.md) | [Romanian](../ro/README.md) | [Russian](../ru/README.md) | [Serbian (Cyrillic)](../sr/README.md) | [Slovak](../sk/README.md) | [Slovenian](../sl/README.md) | [Spanish](../es/README.md) | [Swahili](../sw/README.md) | [Swedish](../sv/README.md) | [Tagalog (Filipino)](../tl/README.md) | [Tamil](../ta/README.md) | [Telugu](../te/README.md) | [Thai](../th/README.md) | [Turkish](../tr/README.md) | [Ukrainian](../uk/README.md) | [Urdu](../ur/README.md) | [Vietnamese](../vi/README.md)

> **പ്രാദേശികമായി ക്ലോൺ ചെയ്യണമെന്ന് ആഗ്രഹിക്കുന്നുവോ?**
>
> ഈ റിപ്പോസിറ്ററിയിൽ 50+ ഭാഷാ പരിഭാഷകൾ ഉൾക്കൊള്ളിച്ചിരിക്കുന്നത്, അത് ഡൗൺലോഡ് വലിപ്പം വർദ്ധിപ്പിക്കുന്നു. പരിഭാഷകൾ ഇല്ലാതെ ക്ലോൺ ചെയ്യുന്നതിന്, സ്പാർസ് ചെക്ക്ഔട്ട് ഉപയോഗിക്കുക:
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
> ഇത് കോഴ്സ് പൂർത്തിയാക്കാൻ ആവശ്യമുള്ള എല്ലാ വസ്തുക്കളും വളരെ വേഗത്തിൽ ഡൗൺലോഡു ചെയ്യാൻ സഹായിക്കും.
<!-- CO-OP TRANSLATOR LANGUAGES TABLE END -->

## കോഴ്‌സ് ഘടനയും പഠനപഥവും

### **അധ്യായം 1: ജനറേറ്റീവ് AI-ക്ക് പരിചയം**
- **പ്രധാന ആശയങ്ങൾ**: വലിയ ഭാഷാ മോഡലുകൾ, ടോക്കൺസ്, എംബെഡ്ഡിംഗ്സ്, AI ശേഷികൾ മനസ്സിലാക്കൽ  
- **Java AI ഇക്കോസിസ്റ്റം**: Spring AI, OpenAI SDK-കളുടെ അവലോകനം  
- **മോഡൽ കോൺടെക്സ് പ്രോട്ടോക്കോൾ**: MCP-ന്റെ പരിചയം, AI ഏജന്റ് ആശയവിനിമയത്തിൽ അതിന്റെ പങ്ക്  
- **പ്രായോഗിക ഉപയോഗങ്ങൾ**: ചാറ്റ്ബോട്ടുകൾ, ഉള്ളടക്ക സൃഷ്ടി ഉൾപ്പെടെയുള്ള യാഥാർത്ഥ്യ ഘടകങ്ങൾ  
- **[→ ആരംഭിക്കുക അധ്യായം 1](./01-IntroToGenAI/README.md)**

### **അധ്യായം 2: ഡെവലപ്പ്മെന്റ് പരിസ്ഥിതി സ്ഥാപനം**
- **Azure AI Foundry**: Bicep, Azure Developer CLI (azd) ഉപയോഗിച്ച് മോഡൽ ഡിപ്ലോയ്മെന്റുകൾ കോഡായി പ്രൊവിഷൻ ചെയ്യുക  
- **Spring Boot + Spring AI**: എന്റർപ്രൈസ് AI അപ്ലിക്കേഷൻ വികസനത്തിനുള്ള മികച്ച രീതികൾ  
- **കീലെസ് ഓത്തന്റിക്കേഷന്‍**: Microsoft Entra ID-യുമായി സുരക്ഷിതമായി ബന്ധിപ്പിക്കുക — API കീകൾ കൈകാര്യം ചെയ്യേണ്ടതില്ല  
- **ഡെവലപ്പ്മെന്റ് ടൂളുകൾ**: Docker കണ്ടെയ്‌നറുകൾ, VS Code, GitHub Codespaces കോൺഫിഗറേഷൻ  
- **[→ ആരംഭിക്കുക അധ്യായം 2](./02-SetupDevEnvironment/README.md)**

### **അധ്യായം 3: കേർ ജനറേറ്റീവ് AI സാങ്കേതിക വിദ്യകൾ**
- **പ്രോംപ്റ്റ് എൻജിനീയറിങ്**: AI മോഡൽ പരിപൂർണമായ പ്രതികരണങ്ങൾക്ക് സാങ്കേതിക വിദ്യകൾ  
- **എംബെഡ്ഡിംഗ്സ് & വെക്റ്റർ കാൽപ്പനകൾ**: സൈമാന്റിക് തിരച്ചിൽ, സമാനതാ പൊരുത്തപ്പെടുത്തൽ നടപ്പാക്കുക  
- **റിട്രീവൽ-ഓഗ്മെന്റഡ് ജനറേഷൻ (RAG)**: നിങ്ങളുടെ സ്വന്തം ഡാറ്റാ സ്രോതസുകളുമായി AI സംയോജിപ്പിക്കുക  
- **ഫങ്‌ഷൻ കോൾ**: കോസ്റ്റം ടൂളുകൾ, പ്ലഗിനുകൾ ഉപയോഗിച്ച് AI ശേഷികൾ വിപുലീകരിക്കുക  
- **[→ ആരംഭിക്കുക അധ്യായം 3](./03-CoreGenerativeAITechniques/README.md)**

### **അധ്യായം 4: പ്രായോഗിക ഉപയോഗങ്ങളും പദ്ധതികളും**
- **പാൾ സ്റ്റോറി ജനറേറ്റർ** (`petstory/`): Azure AI Foundry ഉപയോഗിച്ച് സൃഷ്‌ടിപരമായ ഉള്ളടക്ക നിർമ്മിതിയ്‌ക്ക്  
- **Foundry ലോക്കൽ ഡെമോ** (`foundrylocal/`): OpenAI Java SDK-യുമായി ലൊക്കൽ AI മോഡൽ സംയോജനം  
- **MCP കാൽക്കുലേറ്റർ സർവീസ്** (`calculator/`): Spring AI ഉപയോഗിച്ച് അടിസ്ഥാന Model Context Protocol നടപ്പാക്കൽ  
- **[→ ആരംഭിക്കുക അധ്യായം 4](./04-PracticalSamples/README.md)**

### **അധ്യായം 5: ഉത്തരവാദിത്വമുള്ള AI വികസനം**
- **Azure AI Foundry ഉള്ളടക്ക സുരക്ഷ**: ഘടിപ്പിച്ച ഉള്ളടക്ക ഫിൽട്ടറിങ്ങും സുരക്ഷായന്ത്രങ്ങളും പരീക്ഷിക്കുക (ഹാർഡ് ബ്ലോക്കുകളും സോഫ്‌റ്റ് നിരാസനകളും)  
- **ഉത്തരവാദിത്വമുള്ള AI ഡെമോ**: ആധുനിക AI സുരക്ഷാ സംവിധാനങ്ങൾ പ്രയോഗത്തിൽ എങ്ങനെ പ്രവർത്തിക്കുന്നു എന്നത് കാണിക്കുന്ന കൈകാര്യം  
- **മികച്ച രീതികൾ**: നീതിപരമായ AI വികസനത്തിനും വിനിയോഗത്തിനും ആവശ്യമായ മാർഗ്ഗനിർദ്ദേശങ്ങൾ  
- **[→ ആരംഭിക്കുക അധ്യായം 5](./05-ResponsibleGenAI/README.md)**

## അധിക സ്രോതസുകൾ

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
 
### ജനറേറ്റീവ് AI സീരീസ്  
[![Generative AI for Beginners](https://img.shields.io/badge/Generative%20AI%20for%20Beginners-8B5CF6?style=for-the-badge&labelColor=E5E7EB&color=8B5CF6)](https://github.com/microsoft/generative-ai-for-beginners?WT.mc_id=academic-105485-koreyst)  
[![Generative AI (.NET)](https://img.shields.io/badge/Generative%20AI%20(.NET)-9333EA?style=for-the-badge&labelColor=E5E7EB&color=9333EA)](https://github.com/microsoft/Generative-AI-for-beginners-dotnet?WT.mc_id=academic-105485-koreyst)  
[![Generative AI (Java)](https://img.shields.io/badge/Generative%20AI%20(Java)-C084FC?style=for-the-badge&labelColor=E5E7EB&color=C084FC)](https://github.com/microsoft/generative-ai-for-beginners-java?WT.mc_id=academic-105485-koreyst)  
[![Generative AI (JavaScript)](https://img.shields.io/badge/Generative%20AI%20(JavaScript)-E879F9?style=for-the-badge&labelColor=E5E7EB&color=E879F9)](https://github.com/microsoft/generative-ai-with-javascript?WT.mc_id=academic-105485-koreyst)  

---
 
### കോർ പഠനം  
[![ML for Beginners](https://img.shields.io/badge/ML%20for%20Beginners-22C55E?style=for-the-badge&labelColor=E5E7EB&color=22C55E)](https://aka.ms/ml-beginners?WT.mc_id=academic-105485-koreyst)  
[![Data Science for Beginners](https://img.shields.io/badge/Data%20Science%20for%20Beginners-84CC16?style=for-the-badge&labelColor=E5E7EB&color=84CC16)](https://aka.ms/datascience-beginners?WT.mc_id=academic-105485-koreyst)  
[![AI for Beginners](https://img.shields.io/badge/AI%20for%20Beginners-A3E635?style=for-the-badge&labelColor=E5E7EB&color=A3E635)](https://aka.ms/ai-beginners?WT.mc_id=academic-105485-koreyst)  
[![Cybersecurity for Beginners](https://img.shields.io/badge/Cybersecurity%20for%20Beginners-F97316?style=for-the-badge&labelColor=E5E7EB&color=F97316)](https://github.com/microsoft/Security-101?WT.mc_id=academic-96948-sayoung)
[![Web Dev for Beginners](https://img.shields.io/badge/Web%20Dev%20for%20Beginners-EC4899?style=for-the-badge&labelColor=E5E7EB&color=EC4899)](https://aka.ms/webdev-beginners?WT.mc_id=academic-105485-koreyst)
[![IoT for Beginners](https://img.shields.io/badge/IoT%20for%20Beginners-14B8A6?style=for-the-badge&labelColor=E5E7EB&color=14B8A6)](https://aka.ms/iot-beginners?WT.mc_id=academic-105485-koreyst)
[![XR Development for Beginners](https://img.shields.io/badge/XR%20Development%20for%20Beginners-38BDF8?style=for-the-badge&labelColor=E5E7EB&color=38BDF8)](https://github.com/microsoft/xr-development-for-beginners?WT.mc_id=academic-105485-koreyst)

---
 
### കോപൈലറ്റ് സീരീസ്
[![Copilot for AI Paired Programming](https://img.shields.io/badge/Copilot%20for%20AI%20Paired%20Programming-FACC15?style=for-the-badge&labelColor=E5E7EB&color=FACC15)](https://aka.ms/GitHubCopilotAI?WT.mc_id=academic-105485-koreyst)
[![Copilot for C#/.NET](https://img.shields.io/badge/Copilot%20for%20C%23/.NET-FBBF24?style=for-the-badge&labelColor=E5E7EB&color=FBBF24)](https://github.com/microsoft/mastering-github-copilot-for-dotnet-csharp-developers?WT.mc_id=academic-105485-koreyst)
[![Copilot Adventure](https://img.shields.io/badge/Copilot%20Adventure-FDE68A?style=for-the-badge&labelColor=E5E7EB&color=FDE68A)](https://github.com/microsoft/CopilotAdventures?WT.mc_id=academic-105485-koreyst)
<!-- CO-OP TRANSLATOR OTHER COURSES END -->

## സഹായം ലഭിക്കുക

നിങ്ങൾ കുടുങ്ങുകയോ AI ആപ്പുകൾ നിർമ്മിക്കുന്നതുമായി ബന്ധപ്പെട്ട ഏതെങ്കിലും ചോദ്യങ്ങളുണ്ടോ? MCPയെക്കുറിച്ച് fellow learners-ഉം പരിചയസമ്പന്നരായ ഡെവലപ്പർമാരും ചർച്ച ചെയ്യുന്നതിൽ പങ്കുചേരുക. ചോദ്യങ്ങൾക്ക് സ്വാഗതം ചെയ്യപ്പെടുന്ന, അറിവ് സ്വതന്ത്രമായി പങ്കിടപ്പെടുന്ന ഒരു പിന്തുണയുള്ള കമ്മ്യൂണിറ്റി ആണ് ഇത്.

[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

നിങ്ങൾക്ക് ഉത്പന്ന ഫീഡ്ബാക്കോ നിർമ്മാണ സമയത്ത് പിശകുകളോ ഉണ്ടെങ്കിൽ സന്ദർശിക്കുക:

[![Microsoft Foundry Developer Forum](https://img.shields.io/badge/GitHub-Microsoft_Foundry_Developer_Forum-blue?style=for-the-badge&logo=github&color=000000&logoColor=fff)](https://aka.ms/foundry/forum)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**അറിയിപ്പ്**:
ഈ രേഖ AI പരിഭാഷാ സേവനം [Co-op Translator](https://github.com/Azure/co-op-translator) ഉപയോഗിച്ച് പരിഭാഷപ്പെടുത്തിയതാണ്. ഞങ്ങൾ കൃത്യതയ്ക്കായി ശ്രമിക്കുന്നുവെങ്കിലും, ഓട്ടോമേറ്റഡ് പരിഭാഷകളിൽ പിഴവുകൾ അല്ലെങ്കിൽ തെറ്റായ വിവരങ്ങൾ ഉണ്ടാകാൻ സാധ്യതയുണ്ട്. അതിന്റെ സ്വാഭാവിക ഭാഷയിലുള്ള അസൽ രേഖയാണ് പ്രാമാണികമായ ഉറവിടമായി പരിഗണിക്കേണ്ടത്. നിർണായകമായ വിവരങ്ങൾക്ക്, പ്രൊഫഷണൽ മനുഷ്യ പരിഭാഷ ശുപാർശ ചെയ്യുന്നു. ഈ പരിഭാഷ ഉപയോഗിച്ച് ഉണ്ടാകുന്ന തെറ്റിദ്ധാരണകൾ അല്ലെങ്കിൽ തെറ്റായ വ്യാഖ്യാനങ്ങൾക്കായി ഞങ്ങൾ ഉത്തരവാദികളല്ല.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->