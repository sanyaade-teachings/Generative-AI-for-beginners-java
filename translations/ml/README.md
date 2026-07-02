# ജനറേറ്റീവ് എ.ഐ ആരംഭക്കാർക്ക് - ജാവ് എഡിഷൻ
[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

![Generative AI for Beginners - Java Edition](../../translated_images/ml/beg-genai-series.8b48be9951cc574c.webp)

**സമയം:** മുഴുവൻ വർക്‌ഷോപ്പ് ഓൺലൈനിൽ പ്രാദേശിക സജ്ജീകരണം ആവശ്യമില്ലാതെ പൂർത്തിയാക്കാം. പരിസരം സജ്ജീകരിക്കാൻ 2 മിനിറ്റ് തുടങ്ങിയുള്ളവ, സാമ്പിൾസ് പരിശോധിക്കാൻ 1-3 മണിക്കൂർ വരെ ആവശ്യമായി വരാം, പരിശോധിക്കൽ ആഴമേതെ എന്താണെന്ന് ആശ്രയിച്ച്.

> **വേഗത്തിലുള്ള തുടങ്ങൽ**

1. ഈ റിപ്പോസിറ്ററി നിങ്ങളുടെ GitHub അക്കൗണ്ടിലേക്ക് ഫോർക്ക് ചെയ്യുക
2. **Code** → **Codespaces** ടാബ് → **...** → **New with options...** ക്ലിക്ക് ചെയ്യുക
3. ഡീഫോൾട്ടുകൾ ഉപയോഗിക്കുക – ഇത് ഈ കോഴ്‌സിനായി സൃഷ്ടിച്ച ഡെവലപ്പ്മെന്റ് കണ്ടെയ്‌നെർ തിരഞ്ഞെടുക്കും
4. **Create codespace** ക്ലിക്ക് ചെയ്യുക
5. പരിസരം റെഡി ആകാൻ ഏകദേശം ~2 മിനിറ്റ് കാത്തിരിക്കുക
6. നേരിട്ട് [ആദ്യ ഉദാഹരണത്തിലേക്ക്](./02-SetupDevEnvironment/README.md#step-2-create-a-github-personal-access-token) നിൽക്കുക

## ബഹുഭാഷാ പിന്തുണ

### GitHub Action വഴി പിന്തുണ (താനായി പ്രവർത്തിക്കുകയും എല്ലായ്പ്പോഴും പുതുക്കുകയും ചെയ്യും)

<!-- CO-OP TRANSLATOR LANGUAGES TABLE START -->
[അറബിക്ക്](../ar/README.md) | [ബംഗാളി](../bn/README.md) | [ബൾഗേറിയൻ](../bg/README.md) | [ബർമീസ് (മ്യാൻമാർ)](../my/README.md) | [ചൈനീസ് (സിംപ്ലിഫൈഡ്)](../zh-CN/README.md) | [ചൈനീസ് (റാണവഴി, ഹോങ്കോംഗ്)](../zh-HK/README.md) | [ചൈനീസ് (റാണവഴി, മക്കാവ്)](../zh-MO/README.md) | [ചൈനീസ് (റാണവഴി, তাইവാൻ)](../zh-TW/README.md) | [ക്രൊയേഷ്യൻ](../hr/README.md) | [ചെക്ക്](../cs/README.md) | [ഡാനിഷ്](../da/README.md) | [ഡച്ചുകൾ](../nl/README.md) | [എസ്റ്റോണിയൻ](../et/README.md) | [ഫിന്നിഷ്](../fi/README.md) | [ഫ്രഞ്ച്](../fr/README.md) | [ജർമ്മൻ](../de/README.md) | [ഗ്രീക്ക്](../el/README.md) | [ഹീബ്രു](../he/README.md) | [ഹിന്ദി](../hi/README.md) | [ഹംഗേറിയൻ](../hu/README.md) | [ഇന്തോനേഷ്യൻ](../id/README.md) | [ഇറ്റാലിയൻ](../it/README.md) | [ജാപനീസ്](../ja/README.md) | [കന്നഡ](../kn/README.md) | [ഖ്മർ](../km/README.md) | [കൊറിയൻ](../ko/README.md) | [ലിത്വാനിയൻ](../lt/README.md) | [മലേയാലം](./README.md) | [മാരത്തി](../mr/README.md) | [നെപ്പാളി](../ne/README.md) | [നൈജീരിയൻ പിഡ്ജിൻ](../pcm/README.md) | [നോർവീജിയൻ](../no/README.md) | [പേർഷ്യൻ (ഫാര്സി)](../fa/README.md) | [പോളിഷ്](../pl/README.md) | [പോർച്ചുഗീസി (ബ്രസീൽ)](../pt-BR/README.md) | [പോർച്ചുഗീസി (പോർച്ചുഗൽ)](../pt-PT/README.md) | [പഞ്ചാബി (ഗുരുമുഖി)](../pa/README.md) | [റോമാനിയൻ](../ro/README.md) | [റഷ്യൻ](../ru/README.md) | [സെർബിയൻ (സിറിലിക്ക്)](../sr/README.md) | [സ്ലോവാക്](../sk/README.md) | [സ്ലോവേനിയൻ](../sl/README.md) | [സ്പാനിഷ്](../es/README.md) | [സ്വാഹിലി](../sw/README.md) | [സ്വീഡിഷ്](../sv/README.md) | [ടാഗലോഗ് (ഫിലിപ്പീനോ)](../tl/README.md) | [തമിഴ്](../ta/README.md) | [തെലുങ്ക്](../te/README.md) | [തായ്](../th/README.md) | [ടർക്കിഷ്](../tr/README.md) | [ഉക്രൈന്യൻ](../uk/README.md) | [ഉർദ്](../ur/README.md) | [വിയറ്റ്നാമീസ്](../vi/README.md)

> **പ്രാദേശികമായി ക്ലോൺ ചെയ്യാൻ ആഗ്രഹിക്കുന്നുവോ?**
>
> ഈ റിപ്പോസിറ്ററിയിൽ 50-ത്തിലധികം ഭാഷാ പരിഭാഷകൾ ഉൾപ്പെടുന്നു അത് ഡൗൺലോഡ് വലുതാക്കുന്നു. പരിഭാഷകളില്ലാതെ ക്ലോൺ ചെയ്യാൻ sparse checkout ഉപയോഗിക്കുക:
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
> കോഴ്‌സ് പൂർണ്ണമായി ചെയ്യാൻ നിങ്ങൾക്ക് എന്തും ആവശ്യമായതു ലഭിക്കും കൂടാതെ ഡൗൺലോഡ് വേഗത്തിൽ കാണാം.
<!-- CO-OP TRANSLATOR LANGUAGES TABLE END -->

## കോഴ്‌സ് ഘടനയും പഠന മാർഗവും

### **അധ്യായം 1: ജനറേറ്റീവ് എ.ഐ ആരംഭം**
- **പ്രധാന ആശയങ്ങൾ**: വലിയ ഭാഷാ മോഡലുകൾ, ടോക്കണുകൾ, എംബഡിംഗുകൾ, എ.ഐ കഴിവുകൾ മനസിലാക്കൽ
- **ജാവ എ.ഐ ഇക്കോസിസ്റ്റം**: സ്പ്രിംഗ് എ.ഐ, ഓപ്പൺഎഐ SDK-കൾ അവലോകനം
- **മോഡൽ കോൺടെക്സ്റ്റ് പ്രോട്ടോകോൾ**: MCP ന്റെ പരിചയവും എ.ഐ ഏജന്റുകൾ തമ്മിലുള്ള ബന്ധത്തിൽ പങ്ക്
- **പ്രായോഗിക പ്രയോഗങ്ങൾ**: ചാറ്റ്ബോട്ടുകൾ, ഉള്ളടക്കം സൃഷ്ടിക്കൽ മുതലായ യാഥാർഥిక ദൃശ്യങ്ങൾ
- **[→ അധ്യായം 1 തുടങ്ങുക](./01-IntroToGenAI/README.md)**

### **അധ്യായം 2: ഡെവലപ്പ്മെന്റ് പരിസരം സജ്ജീകരണം**
- **അസ്യൂർ എ.ഐ ഫൗൻഡ്രി**: ബൈസിപ്പ് ഉപയോഗിച്ച് കോഡ് ആയി മോഡൽ ഡിസ്പ്ലോയ്മെന്റുകൾ സജ്ജീകരിക്കുക; അസ്യൂർ ഡെവലപ്പർ CLI (azd)
- **സ്പ്രിംഗ് ബൂട്ട് + സ്പ്രിംഗ് എ.ഐ**: എന്റർപ്രൈസ് എ.ഐ ആപ്ലിക്കേഷൻ വികസനത്തിന് മികച്ച രീതികൾ
- **കീലെസ് ഓതന്റിക്കേഷൻ**: മൈക്രോസോഫ്റ്റ് എന്റ്ര എ.ഐഡി ഉപയോഗിച്ച് സുരക്ഷിതമായ കണക്ഷൻ — API കീകൾ വേണ്ട
- **ഡെവലപ്പ്മെന്റ് ടൂളുകൾ**: ഡോക്കർ കണ്ടെയ്‌നറുകൾ, VS കോഡ്, GitHub Codespaces കോൺഫിഗറേഷൻ
- **[→ അധ്യായം 2 തുടങ്ങുക](./02-SetupDevEnvironment/README.md)**

### **അധ്യായം 3: ജനറേറ്റീവ് എ.ഐ മെയിൻ സാങ്കേതികവിദ്യകൾ**
- **പ്രോംപ്റ്റ് എൻജിനിയറിംഗ്**: ഉറ്റെടുത്ത വിധത്തിൽ എ.ഐ മോഡലുകളെ പ്രതികരിപ്പിക്കൽ
- **എംബഡിംഗുകൾ & വെക്ടർ ഓപ്പറേഷനുകൾ**: സെമാന്റിക് സെർച്ച്, സമാനത മാഷിങ് ഇമ്പ്ലിമെന്റേഷൻ
- **റിട്രിവൽ-ഓഗ്മെന്റഡ് ജനറേഷൻ (RAG)**: നിങ്ങളുടെ ഡാറ്റ ഉറവിടങ്ങളുമായി എ.ഐ സംയോജിപ്പിക്കുക
- **ഫംഗ്ഷൻ കോൾ**: ഒപ്പം ടൂളുകളും പ്ലഗിനുകളും ചേർത്തു എ.ഐ കഴിവുകൾ വിപുലീകരിക്കൽ
- **[→ അധ്യായം 3 തുടങ്ങുക](./03-CoreGenerativeAITechniques/README.md)**

### **അധ്യായം 4: പ്രായോഗിക പ്രോഗ്രാമുകളും പ്രോജക്ടുകളും**
- **പെറ്റ് സ്റ്റോറി ജനറേറ്റർ** (`petstory/`): അസ്യൂർ എ.ഐ ഫൗണ്ട്രിയുമായി സൃഷ്ടിപരമായ ഉള്ളടക്കം ജനന ഉദ്ഘാടനം
- **ഫൗണ്ട്രി ലൊക്കൽ ഡെമോ** (`foundrylocal/`): ഓപ്പൺഎഐ ജാവ SDK ഉപയോഗിച്ച് ലൊക്കൽ മോഡൽ ഇന്റഗ്രേഷൻ
- **MCP കാൽക്കുലേറ്റർ സർവീസ്** (`calculator/`): സ്പ്രിംഗ് എ.ഐ ഉപയോഗിച്ച് അടിസ്ഥാന മോഡൽ കോൺടെക്സ്റ്റ് പ്രോടோகോൾ നടപ്പാക്കൽ
- **[→ അധ്യായം 4 തുടങ്ങുക](./04-PracticalSamples/README.md)**

### **അധ്യായം 5: ഉത്തരവാദിത്വം പാലിച്ചുള്ള എ.ഐ വികസനം**
- **അസ്യൂർ എ.ഐ ഫൗണ്ട്രി ഉള്ളടക്കം സുരക്ഷ**: ബിൽറ്റിൻ ഉള്ളടക്കം ഫിൽട്ടറിംഗ്, സുരക്ഷാ സംവിധാനങ്ങൾ (ഹാർഡ് ബ്ലോക്കുകളും സോഫ്‌റ്റ് നിരസിക്കലും) പരീക്ഷിക്കുക
- **ഉത്തരവാദി എ.ഐ ഡെമോ**: ആധുനിക എ.ഐ സുരക്ഷാ സംവിധാനം എങ്ങനെ പ്രവർത്തിക്കുന്നു എന്നതിന്റെ പ്രായോഗിക ഉദാഹരണം
- **മികച്ച രീതികൾ**: സുസ്ഥിരവും ധാർമ്മികവുമായ എ.ഐ വികസനത്തിനുള്ള മാർഗ്ഗനിർദേശങ്ങൾ
- **[→ അധ്യായം 5 തുടങ്ങുക](./05-ResponsibleGenAI/README.md)**

## അധിക വിഭവങ്ങൾ

<!-- CO-OP TRANSLATOR OTHER COURSES START -->
### ലാംഗ്‌ചെയിൻ
[![LangChain4j for Beginners](https://img.shields.io/badge/LangChain4j%20for%20Beginners-22C55E?style=for-the-badge&&labelColor=E5E7EB&color=0553D6)](https://aka.ms/langchain4j-for-beginners)
[![LangChain.js for Beginners](https://img.shields.io/badge/LangChain.js%20for%20Beginners-22C55E?style=for-the-badge&labelColor=E5E7EB&color=0553D6)](https://aka.ms/langchainjs-for-beginners?WT.mc_id=m365-94501-dwahlin)
[![LangChain for Beginners](https://img.shields.io/badge/LangChain%20for%20Beginners-22C55E?style=for-the-badge&labelColor=E5E7EB&color=0553D6)](https://github.com/microsoft/langchain-for-beginners?WT.mc_id=m365-94501-dwahlin)
---

### അസ്യൂർ / എഡ്ജ് / MCP / ഏജന്റ്സ്
[![AZD for Beginners](https://img.shields.io/badge/AZD%20for%20Beginners-0078D4?style=for-the-badge&labelColor=E5E7EB&color=0078D4)](https://github.com/microsoft/AZD-for-beginners?WT.mc_id=academic-105485-koreyst)
[![Edge AI for Beginners](https://img.shields.io/badge/Edge%20AI%20for%20Beginners-00B8E4?style=for-the-badge&labelColor=E5E7EB&color=00B8E4)](https://github.com/microsoft/edgeai-for-beginners?WT.mc_id=academic-105485-koreyst)
[![MCP for Beginners](https://img.shields.io/badge/MCP%20for%20Beginners-009688?style=for-the-badge&labelColor=E5E7EB&color=009688)](https://github.com/microsoft/mcp-for-beginners?WT.mc_id=academic-105485-koreyst)
[![AI Agents for Beginners](https://img.shields.io/badge/AI%20Agents%20for%20Beginners-00C49A?style=for-the-badge&labelColor=E5E7EB&color=00C49A)](https://github.com/microsoft/ai-agents-for-beginners?WT.mc_id=academic-105485-koreyst)

---
 
### ജനറേറ്റീവ് എ.ഐ സീരീസ്
[![Generative AI for Beginners](https://img.shields.io/badge/Generative%20AI%20for%20Beginners-8B5CF6?style=for-the-badge&labelColor=E5E7EB&color=8B5CF6)](https://github.com/microsoft/generative-ai-for-beginners?WT.mc_id=academic-105485-koreyst)
[![Generative AI (.NET)](https://img.shields.io/badge/Generative%20AI%20(.NET)-9333EA?style=for-the-badge&labelColor=E5E7EB&color=9333EA)](https://github.com/microsoft/Generative-AI-for-beginners-dotnet?WT.mc_id=academic-105485-koreyst)
[![Generative AI (Java)](https://img.shields.io/badge/Generative%20AI%20(Java)-C084FC?style=for-the-badge&labelColor=E5E7EB&color=C084FC)](https://github.com/microsoft/generative-ai-for-beginners-java?WT.mc_id=academic-105485-koreyst)
[![Generative AI (JavaScript)](https://img.shields.io/badge/Generative%20AI%20(JavaScript)-E879F9?style=for-the-badge&labelColor=E5E7EB&color=E879F9)](https://github.com/microsoft/generative-ai-with-javascript?WT.mc_id=academic-105485-koreyst)

---
 
### കോർ ലേണിംഗ്
[![ML for Beginners](https://img.shields.io/badge/ML%20for%20Beginners-22C55E?style=for-the-badge&labelColor=E5E7EB&color=22C55E)](https://aka.ms/ml-beginners?WT.mc_id=academic-105485-koreyst)
[![Data Science for Beginners](https://img.shields.io/badge/Data%20Science%20for%20Beginners-84CC16?style=for-the-badge&labelColor=E5E7EB&color=84CC16)](https://aka.ms/datascience-beginners?WT.mc_id=academic-105485-koreyst)
[![AI for Beginners](https://img.shields.io/badge/AI%20for%20Beginners-A3E635?style=for-the-badge&labelColor=E5E7EB&color=A3E635)](https://aka.ms/ai-beginners?WT.mc_id=academic-105485-koreyst)
[![Cybersecurity for Beginners](https://img.shields.io/badge/Cybersecurity%20for%20Beginners-F97316?style=for-the-badge&labelColor=E5E7EB&color=F97316)](https://github.com/microsoft/Security-101?WT.mc_id=academic-96948-sayoung)

[![Web Dev for Beginners](https://img.shields.io/badge/Web%20Dev%20for%20Beginners-EC4899?style=for-the-badge&labelColor=E5E7EB&color=EC4899)](https://aka.ms/webdev-beginners?WT.mc_id=academic-105485-koreyst)
[![IoT for Beginners](https://img.shields.io/badge/IoT%20for%20Beginners-14B8A6?style=for-the-badge&labelColor=E5E7EB&color=14B8A6)](https://aka.ms/iot-beginners?WT.mc_id=academic-105485-koreyst)
[![XR Development for Beginners](https://img.shields.io/badge/XR%20Development%20for%20Beginners-38BDF8?style=for-the-badge&labelColor=E5E7EB&color=38BDF8)](https://github.com/microsoft/xr-development-for-beginners?WT.mc_id=academic-105485-koreyst)

---
 
### കോപൈലറ്റ് സീറീസ്
[![Copilot for AI Paired Programming](https://img.shields.io/badge/Copilot%20for%20AI%20Paired%20Programming-FACC15?style=for-the-badge&labelColor=E5E7EB&color=FACC15)](https://aka.ms/GitHubCopilotAI?WT.mc_id=academic-105485-koreyst)
[![Copilot for C#/.NET](https://img.shields.io/badge/Copilot%20for%20C%23/.NET-FBBF24?style=for-the-badge&labelColor=E5E7EB&color=FBBF24)](https://github.com/microsoft/mastering-github-copilot-for-dotnet-csharp-developers?WT.mc_id=academic-105485-koreyst)
[![Copilot Adventure](https://img.shields.io/badge/Copilot%20Adventure-FDE68A?style=for-the-badge&labelColor=E5E7EB&color=FDE68A)](https://github.com/microsoft/CopilotAdventures?WT.mc_id=academic-105485-koreyst)
<!-- CO-OP TRANSLATOR OTHER COURSES END -->

## സഹായം നേടുക

AI ആപ്പുകൾ നിർമ്മിക്കുന്നതിൽ നിങ്ങൾ തടസം അനുഭവപ്പെടുകയോ ഏതെങ്കിലും ചോദ്യം ഉണ്ടാകുകയോ ചെയ്താൽ. MCP-യിൽ അനുബന്ധ പഠിതാക്കളും പരിചയസമ്പന്നരും ആയ ഡവലപ്പർമാരൊപ്പം ചർച്ചകളിൽ ചേരുക. ചോദ്യം ചോദിക്കാൻ സ്വാഗതംാണ് ഇപ്പം, അറിവ് സ്വതന്ത്രമായി പങ്കുവെയ്ക്കപ്പെടുന്നു.

[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

നിങ്ങൾക്ക് ഉൽപ്പന്നത്തിൽ അഭിപ്രായം നൽകാനോ നിർമ്മിക്കുമ്പോൾ പിശകുകൾ ഉണ്ടാകുമ്പോൾ സന്ദർശിക്കുക:

[![Microsoft Foundry Developer Forum](https://img.shields.io/badge/GitHub-Microsoft_Foundry_Developer_Forum-blue?style=for-the-badge&logo=github&color=000000&logoColor=fff)](https://aka.ms/foundry/forum)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**അറിയിപ്പ്**:
ഈ രേഖ AI പരിഭാഷാ സേവനം [Co-op Translator](https://github.com/Azure/co-op-translator) ഉപയോഗിച്ച് പരിഭാഷപ്പെടുത്തിയതാണ്. ഞങ്ങൾ കൃത്യതയ്ക്കായി ശ്രമിക്കുന്നുവെങ്കിലും, ഓട്ടോമേറ്റഡ് പരിഭാഷകളിൽ പിഴവുകൾ അല്ലെങ്കിൽ തെറ്റായ വിവരങ്ങൾ ഉണ്ടാകാൻ സാധ്യതയുണ്ട്. അതിന്റെ സ്വാഭാവിക ഭാഷയിലുള്ള അസൽ രേഖയാണ് പ്രാമാണികമായ ഉറവിടമായി പരിഗണിക്കേണ്ടത്. നിർണായകമായ വിവരങ്ങൾക്ക്, പ്രൊഫഷണൽ മനുഷ്യ പരിഭാഷ ശുപാർശ ചെയ്യുന്നു. ഈ പരിഭാഷ ഉപയോഗിച്ച് ഉണ്ടാകുന്ന തെറ്റിദ്ധാരണകൾ അല്ലെങ്കിൽ തെറ്റായ വ്യാഖ്യാനങ്ങൾക്കായി ഞങ്ങൾ ഉത്തരവാദികളല്ല.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->