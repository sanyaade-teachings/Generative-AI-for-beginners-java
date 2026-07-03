# ಆರಂಭಿಕರಿಗಾಗಿ ಜನರೇಟಿವ್ AI - ಜಾವಾ ಆವೃತ್ತಿ
[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

![Generative AI for Beginners - Java Edition](../../translated_images/kn/beg-genai-series.8b48be9951cc574c.webp)

**ಸಮಯ ಬದ್ಧತೆ**: ಈ ವರ್ಕ್‌ಶಾಪ್ ಅನ್ನು ಎಲ್ಲಾ ಸ್ಥಳೀಯ ಸೆಟ್‌ಅಪ್ ಇಲ್ಲದೆ ಆನ್ಲೈನ್‌ನಲ್ಲಿ ಪೂರ್ಣಗೊಳ್ಳಿಸಬಹುದು. ಪರಿಸರ ಸೆಟ್‌ಅಪ್‌ಗೆ 2 ನಿಮಿಷಗಳು ಬೇಕಾಗುತ್ತದೆ, ಉದಾಹರಣೆಗಳನ್ನು ಅನ್ವೇಷಿಸಲು ಪರೀಕ್ಷಣೆ ಆಳವನ್ನು ಅವಲಂಬಿಸಿ 1-3 ಗಂಟೆಗಳ ಅವಧಿ ತೆಗೆದುಕೊಳ್ಳಬಹುದು.

> **ತ್ವರಿತ ಪ್ರಾರಂಭ**

1. ಈ ರೆಪೊಸಿಟರಿಯನ್ನು ನಿಮ್ಮ GitHub ಖಾತೆಗೆ ಫೋರ್ಕ್ ಮಾಡಿ  
2. **Code** → **Codespaces** ಟ್ಯಾಬ್ → **...** → **New with options...** ಕ್ಲಿಕ್ ಮಾಡಿ  
3. ಡೀಫಾಲ್ಟ್‌ಗಳನ್ನು ಬಳಸಿರಿ – ಇದರಿಂದ ಈ ಕೋರ್ಸ್‌ಗೆ ಸೃಷ್ಟಿಸಲಾದ ಡೆವಲಪ್‌ಮೆಂಟ್ ಕಂಟೇನರ್ ಆರಿಸಲಾಗುತ್ತದೆ  
4. **Create codespace** ಕ್ಲಿಕ್ ಮಾಡಿ  
5. ಪರಿಸರ ಸಿದ್ದವಾಗಲು ಸುಮಾರು 2 ನಿಮಿಷ ಕಾಯಿರಿ  
6. ನೇರವಾಗಿ [ಅಧ್ಯಾಯ 2: ಅಜೂರ್ AI ಫೌಂಡ್ರಿ ಪ್ರೊವಿಷನಿಂಗ್](./02-SetupDevEnvironment/README.md#step-2-provision-azure-ai-foundry) ಗೆ ಹಾರಾಟ ಮಾಡಿ

## ಬಹುಭಾಷಾ ಬೆಂಬಲ

### GitHub Action ಮೂಲಕ ಬೆಂಬಲ (ಸ್ವಯಂಚಾಲಿತ ಮತ್ತು ಸದಾಪ್ರತಿಷ್ಠಿತ)

<!-- CO-OP TRANSLATOR LANGUAGES TABLE START -->
[Arabic](../ar/README.md) | [Bengali](../bn/README.md) | [Bulgarian](../bg/README.md) | [Burmese (Myanmar)](../my/README.md) | [Chinese (Simplified)](../zh-CN/README.md) | [Chinese (Traditional, Hong Kong)](../zh-HK/README.md) | [Chinese (Traditional, Macau)](../zh-MO/README.md) | [Chinese (Traditional, Taiwan)](../zh-TW/README.md) | [Croatian](../hr/README.md) | [Czech](../cs/README.md) | [Danish](../da/README.md) | [Dutch](../nl/README.md) | [Estonian](../et/README.md) | [Finnish](../fi/README.md) | [French](../fr/README.md) | [German](../de/README.md) | [Greek](../el/README.md) | [Hebrew](../he/README.md) | [Hindi](../hi/README.md) | [Hungarian](../hu/README.md) | [Indonesian](../id/README.md) | [Italian](../it/README.md) | [Japanese](../ja/README.md) | [Kannada](./README.md) | [Khmer](../km/README.md) | [Korean](../ko/README.md) | [Lithuanian](../lt/README.md) | [Malay](../ms/README.md) | [Malayalam](../ml/README.md) | [Marathi](../mr/README.md) | [Nepali](../ne/README.md) | [Nigerian Pidgin](../pcm/README.md) | [Norwegian](../no/README.md) | [Persian (Farsi)](../fa/README.md) | [Polish](../pl/README.md) | [Portuguese (Brazil)](../pt-BR/README.md) | [Portuguese (Portugal)](../pt-PT/README.md) | [Punjabi (Gurmukhi)](../pa/README.md) | [Romanian](../ro/README.md) | [Russian](../ru/README.md) | [Serbian (Cyrillic)](../sr/README.md) | [Slovak](../sk/README.md) | [Slovenian](../sl/README.md) | [Spanish](../es/README.md) | [Swahili](../sw/README.md) | [Swedish](../sv/README.md) | [Tagalog (Filipino)](../tl/README.md) | [Tamil](../ta/README.md) | [Telugu](../te/README.md) | [Thai](../th/README.md) | [Turkish](../tr/README.md) | [Ukrainian](../uk/README.md) | [Urdu](../ur/README.md) | [Vietnamese](../vi/README.md)

> **ಸ್ಥಳೀಯವಾಗಿ ಕ್ಲೋನ್ ಮಾಡಬೇಕೆಂದು ಇಚ್ಛಿಸುತ್ತೀರಾ?**  
>  
> ಈ ರೆಪೊಸಿಟರಿಯಲ್ಲಿ 50+ ಭಾಷಾ ಅನುವಾದಗಳು ಸೇರಿವೆ, ಅವು ಡೌನ್‌ಲೋಡ್ ಗಾತ್ರವನ್ನು ಬಹಳಷ್ಟು ಹೆಚ್ಚಿಸುತ್ತವೆ. ಅನುವಾದಗಳಿಲ್ಲದೆ ಕ್ಲೋನ್ ಮಾಡಲು,sparse checkout ಬಳಸಿ:  
>  
> **Bash / macOS / Linux:**  
>> ```bash
> git clone --filter=blob:none --sparse https://github.com/microsoft/Generative-AI-for-beginners-java.git
> cd Generative-AI-for-beginners-java
> git sparse-checkout set --no-cone '/*' '!translations' '!translated_images'
> ```
>  
> **CMD (Windows):**  
>> ```cmd
> git clone --filter=blob:none --sparse https://github.com/microsoft/Generative-AI-for-beginners-java.git
> cd Generative-AI-for-beginners-java
> git sparse-checkout set --no-cone "/*" "!translations" "!translated_images"
> ```
>  
> ಇದು ನಿಮಗೆ ಕೋರ್ಸ್ ಪೂರೈಕೆಗಾಗಿ ಅಗತ್ಯವಿರುವ ಎಲ್ಲವನ್ನೂ much ವೇಗವಾಗಿ ಡೌನ್‌ಲೋಡ್ ಮಾಡಿಕೊಳ್ಳಲು ಅನುಮತಿಸುತ್ತದೆ.  
<!-- CO-OP TRANSLATOR LANGUAGES TABLE END -->

## ಕೋರ್ಸ್ ಮೂಲಾಕೃತಿಯು ಮತ್ತು ಕಲಿಕಾಶೀಲತೆ

### **ಅಧ್ಯಾಯ 1: ಜನರೇಟಿವ್ AI ಪರಿಚಯ**  
- **ಮೂಲಭೂತ ಪರಿಕಲ್ಪನೆಗಳು**: ದೊಡ್ಡ ಭಾಷಾ ಮಾದರಿಗಳು, ಟೋಕನ್ಗಳು, ಎम्बೆಡ್ಡಿಂಗ್‌ಗಳು ಮತ್ತು AI ಸಾಮರ್ಥ್ಯಗಳ ಅರಿವು  
- **ಜಾವಾ AI ಪರಿಸರ**: ಸ್ಪ್ರಿಂಗ್ AI ಮತ್ತು OpenAI SDKಗಳ ಅವಲೋಕನ  
- **ಮಾಡಲ್ ಕಾನ್‌ಟೆಕ್ಸ್ಟ್ ಪ್ರೋಟೋಕಾಲ್**: MCP ಪರಿಚಯ ಮತ್ತು AI ಏಜೆಂಟ್ ಸಂವಹನದಲ್ಲಿ ಅದರ ಪಾತ್ರ  
- **ಪ್ರಾಯೋಗಿಕ ಅನ್ವಯಗಳು**: ಚಾಟ್‌ಬೋಟ್ಗಳು ಮತ್ತು ವಿಷಯ ತಯಾರಿಕಾ ಸೆನ್ನೆರಿ  
- **[→ ಅಧ್ಯಾಯ 1 ಪ್ರಾರಂಭಿಸಿ](./01-IntroToGenAI/README.md)**

### **ಅಧ್ಯಾಯ 2: ಡೆವಲಪ್‌ಮೆಂಟ್ ಪರಿಸರ ಸೆಟ್‌ಅಪ್**  
- **ಅಜೂರ್ AI ಫೌಂಡ್ರಿ**: Bicep ಮತ್ತು ಅಜೂರ್ ಡೆವಲಪರ್ CLI (azd) ಬಳಸಿ ಮಾಡಲ್ ನಿಯೋಜನೆಗಳನ್ನು ಕೋಡ್ ಆಗಿ ಪ್ರೊವಿಷನ್ ಮಾಡಿ  
- **ಸ್ಪ್ರಿಂಗ್ ಬೂಟ್ + ಸ್ಪ್ರಿಂಗ್ AI**: ಎಂಟರ್ಪ್ರೈಸ್ AI ಅಪ್ಲಿಕೇಶನ್ ಅಭಿವೃದ್ಧಿಗಾಗಿ ಅತ್ಯುತ್ತಮ ಆಚಾರಗಳು  
- **ಕೀಲೆಸ್ ಪ್ರಾಮಾಣೀಕರಣ**: ಮೈಕ್ರೋಸಾಫ್ಟ್ ಎಂಟ್ರಾ ID ಮೂಲಕ ಭದ್ರತೆಯ ಸಹಿತ ಸಂಪರ್ಕ – ಯಾವುದೇ API ಕೀಗಳನ್ನು ನಿರ್ವಹಿಸುವ ಅವಶ್ಯಕತೆ ಇಲ್ಲ  
- **ಡೆವಲಪ್‌ಮೆಂಟ್ ಉಪಕರಣಗಳು**: ಡೋಕರ್ ಕಂಟೇನರ್‌ಗಳು, VS ಕೋಡ್ ಮತ್ತು GitHub Codespaces ಸಂರಚನೆ  
- **[→ ಅಧ್ಯಾಯ 2 ಪ್ರಾರಂಭಿಸಿ](./02-SetupDevEnvironment/README.md)**

### **ಅಧ್ಯಾಯ 3: ಮೂಲ ಜನರೇಟಿವ್ AI ತಂತ್ರಗಳು**  
- **ಪ್ರಾಂಪ್ಟ್ ಇಂಜಿನಿಯರಿಂಗ್**: ಅತ್ಯುತ್ತಮ AI ಮಾಡಲ್ ಪ್ರತಿಕ್ರಿಯೆಗಳಿಗೆ ತಂತ್ರಗಳು  
- **ಎಂಬೆಡ್ಡಿಂಗ್‌ಗಳು ಮತ್ತು ವೆಕ್ಟರ್ ಕಾರ್ಯಾಚರಣೆಗಳು**: ಸೆಯ್ಮಾಂಟಿಕ್ ಹುಡುಕಾಟ ಮತ್ತು ಸಾದೃಶ್ಯ ಹೊಂದಿಕೆಯನ್ನು ಅನ್ವಯಿಸಿ  
- **ರಿಟ್ರಿವಲ್-ಆಗ್ಮೆಂಟೆಡ್ ಜನರೇಶನ್ (RAG)**: AI ಮತ್ತು ನಿಮ್ಮದೇ ಡೇಟಾ ಮೂಲಗಳ ಸಂಯೋಜನೆ  
- **ಫಂಕ್ಷನ್ ಕಾಲಿಂಗ್**: ಕಸ್ಟಮ್ ಉಪಕರಣಗಳು ಮತ್ತು ಪ್ಲಗಿನ್‌ಗಳೊಂದಿಗೆ AI ಸಾಮರ್ಥ್ಯ ವಿಸ್ತರಣೆಯನ್ನ ಮಾಡಿ  
- **[→ ಅಧ್ಯಾಯ 3 ಪ್ರಾರಂಭಿಸಿ](./03-CoreGenerativeAITechniques/README.md)**

### **ಅಧ್ಯಾಯ 4: ಪ್ರಾಯೋಗಿಕ ಅನ್ವಯಗಳು ಮತ್ತು ಯೋಜನೆಗಳು**  
- **ಪೆಟ್ ಸ್ಟೋರಿ ಜನರೇಟರ್** (`petstory/`): ಅಜೂರ್ AI ಫೌಂಡ್ರಿ ಬಳಸಿ ಸೃಜನಾತ್ಮಕ ವಿಷಯ ತಯಾರಿಕೆ  
- **ಫೌಂಡ್ರಿ ಲೊಕಲ್ ಡೆಮೊ** (`foundrylocal/`): OpenAI ಜಾವಾ SDK ಬಳಸಿ ಸ್ಥಳೀಯ AI ಮಾಡಲ್ ಏಕೀಕರಣ  
- **MCP ಕ್ಯಾಲ್ಕ್ಯುಲೇಟರ್ ಸರ್ವಿಸ್** (`calculator/`): ಸ್ಪ್ರಿಂಗ್ AI ಬಳಸಿ ಮೂಲ ಮಾಡಲ್ ಕಾನ್‌ಟೆಕ್ಸ್ಟ್ ಪ್ರೋಟೋಕಾಲ್ ಅನ್ವಯಿಕೆ  
- **[→ ಅಧ್ಯಾಯ 4 ಪ್ರಾರಂಭಿಸಿ](./04-PracticalSamples/README.md)**

### **ಅಧ್ಯಾಯ 5: ಉತ್ತರದಾಯಕ AI ಅಭಿವೃದ್ಧಿ**  
- **ಅಜೂರ್ AI ಫೌಂಡ್ರಿ ವಿಷಯ ಭದ್ರತೆ**: ಒಳಗೊಂಡಿರುವ ವಿಷಯ ತಪಾಸಣೆ ಮತ್ತು ಭದ್ರತಾ ಯಾಂತ್ರಿಕೆಗಳ ಪರೀಕ್ಷೆ (ಕಠಿಣ ನಿರ್ಬಂಧಗಳು ಮತ್ತು ನರ್ಮ ನಿರಾಕರಣೆಗಳು)  
- **ಉತ್ತರದಾಯಕ AI ಡೆಮೊ**: ಆಧುನಿಕ AI ಭದ್ರತಾ ವ್ಯವಸ್ಥೆಗಳು ಹೇಗೆ ಕಾರ್ಯನಿರ್ವಹಿಸುತ್ತವೆ ಎಂಬ ನೇರ ಉದಾಹರಣೆ  
- **ಅತ್ಯುತ್ತಮ ಅಭ್ಯಾಸಗಳು**: ನೈತಿಕ AI ಅಭಿವೃದ್ಧಿ ಮತ್ತು ನಿಯೋಜನೆಗಾಗಿ ಅವಶ್ಯಕ ಮಾರ್ಗಸೂಚಿಗಳು  
- **[→ ಅಧ್ಯಾಯ 5 ಪ್ರಾರಂಭಿಸಿ](./05-ResponsibleGenAI/README.md)**

## ಹೆಚ್ಚುವರಿ સ્તೋತ್ರಗಳು

<!-- CO-OP TRANSLATOR OTHER COURSES START -->
### ಲಾಂಗ್ಚೈನ್  
[![LangChain4j for Beginners](https://img.shields.io/badge/LangChain4j%20for%20Beginners-22C55E?style=for-the-badge&&labelColor=E5E7EB&color=0553D6)](https://aka.ms/langchain4j-for-beginners)  
[![LangChain.js for Beginners](https://img.shields.io/badge/LangChain.js%20for%20Beginners-22C55E?style=for-the-badge&labelColor=E5E7EB&color=0553D6)](https://aka.ms/langchainjs-for-beginners?WT.mc_id=m365-94501-dwahlin)  
[![LangChain for Beginners](https://img.shields.io/badge/LangChain%20for%20Beginners-22C55E?style=for-the-badge&labelColor=E5E7EB&color=0553D6)](https://github.com/microsoft/langchain-for-beginners?WT.mc_id=m365-94501-dwahlin)  
---

### ಅಜೂರ್ / ಎಡ್ಜ್ / MCP / ಏಜೆಂಟ್‌ಗಳು  
[![AZD for Beginners](https://img.shields.io/badge/AZD%20for%20Beginners-0078D4?style=for-the-badge&labelColor=E5E7EB&color=0078D4)](https://github.com/microsoft/AZD-for-beginners?WT.mc_id=academic-105485-koreyst)  
[![Edge AI for Beginners](https://img.shields.io/badge/Edge%20AI%20for%20Beginners-00B8E4?style=for-the-badge&labelColor=E5E7EB&color=00B8E4)](https://github.com/microsoft/edgeai-for-beginners?WT.mc_id=academic-105485-koreyst)  
[![MCP for Beginners](https://img.shields.io/badge/MCP%20for%20Beginners-009688?style=for-the-badge&labelColor=E5E7EB&color=009688)](https://github.com/microsoft/mcp-for-beginners?WT.mc_id=academic-105485-koreyst)  
[![AI Agents for Beginners](https://img.shields.io/badge/AI%20Agents%20for%20Beginners-00C49A?style=for-the-badge&labelColor=E5E7EB&color=00C49A)](https://github.com/microsoft/ai-agents-for-beginners?WT.mc_id=academic-105485-koreyst)  

---  
 
### ಜನರೇಟಿವ್ AI ಸರಣಿಯು  
[![Generative AI for Beginners](https://img.shields.io/badge/Generative%20AI%20for%20Beginners-8B5CF6?style=for-the-badge&labelColor=E5E7EB&color=8B5CF6)](https://github.com/microsoft/generative-ai-for-beginners?WT.mc_id=academic-105485-koreyst)  
[![Generative AI (.NET)](https://img.shields.io/badge/Generative%20AI%20(.NET)-9333EA?style=for-the-badge&labelColor=E5E7EB&color=9333EA)](https://github.com/microsoft/Generative-AI-for-beginners-dotnet?WT.mc_id=academic-105485-koreyst)  
[![Generative AI (Java)](https://img.shields.io/badge/Generative%20AI%20(Java)-C084FC?style=for-the-badge&labelColor=E5E7EB&color=C084FC)](https://github.com/microsoft/generative-ai-for-beginners-java?WT.mc_id=academic-105485-koreyst)  
[![Generative AI (JavaScript)](https://img.shields.io/badge/Generative%20AI%20(JavaScript)-E879F9?style=for-the-badge&labelColor=E5E7EB&color=E879F9)](https://github.com/microsoft/generative-ai-with-javascript?WT.mc_id=academic-105485-koreyst)  

---  
 
### ಮೂಲಪಾಠ ಕಲಿಕೆ  
[![ML for Beginners](https://img.shields.io/badge/ML%20for%20Beginners-22C55E?style=for-the-badge&labelColor=E5E7EB&color=22C55E)](https://aka.ms/ml-beginners?WT.mc_id=academic-105485-koreyst)  
[![Data Science for Beginners](https://img.shields.io/badge/Data%20Science%20for%20Beginners-84CC16?style=for-the-badge&labelColor=E5E7EB&color=84CC16)](https://aka.ms/datascience-beginners?WT.mc_id=academic-105485-koreyst)  
[![AI for Beginners](https://img.shields.io/badge/AI%20for%20Beginners-A3E635?style=for-the-badge&labelColor=E5E7EB&color=A3E635)](https://aka.ms/ai-beginners?WT.mc_id=academic-105485-koreyst)  
[![Cybersecurity for Beginners](https://img.shields.io/badge/Cybersecurity%20for%20Beginners-F97316?style=for-the-badge&labelColor=E5E7EB&color=F97316)](https://github.com/microsoft/Security-101?WT.mc_id=academic-96948-sayoung)
[![Web Dev for Beginners](https://img.shields.io/badge/Web%20Dev%20for%20Beginners-EC4899?style=for-the-badge&labelColor=E5E7EB&color=EC4899)](https://aka.ms/webdev-beginners?WT.mc_id=academic-105485-koreyst)
[![IoT for Beginners](https://img.shields.io/badge/IoT%20for%20Beginners-14B8A6?style=for-the-badge&labelColor=E5E7EB&color=14B8A6)](https://aka.ms/iot-beginners?WT.mc_id=academic-105485-koreyst)
[![XR Development for Beginners](https://img.shields.io/badge/XR%20Development%20for%20Beginners-38BDF8?style=for-the-badge&labelColor=E5E7EB&color=38BDF8)](https://github.com/microsoft/xr-development-for-beginners?WT.mc_id=academic-105485-koreyst)

---
 
### Copilot ಸರಣಿ
[![Copilot for AI Paired Programming](https://img.shields.io/badge/Copilot%20for%20AI%20Paired%20Programming-FACC15?style=for-the-badge&labelColor=E5E7EB&color=FACC15)](https://aka.ms/GitHubCopilotAI?WT.mc_id=academic-105485-koreyst)
[![Copilot for C#/.NET](https://img.shields.io/badge/Copilot%20for%20C%23/.NET-FBBF24?style=for-the-badge&labelColor=E5E7EB&color=FBBF24)](https://github.com/microsoft/mastering-github-copilot-for-dotnet-csharp-developers?WT.mc_id=academic-105485-koreyst)
[![Copilot Adventure](https://img.shields.io/badge/Copilot%20Adventure-FDE68A?style=for-the-badge&labelColor=E5E7EB&color=FDE68A)](https://github.com/microsoft/CopilotAdventures?WT.mc_id=academic-105485-koreyst)
<!-- CO-OP TRANSLATOR OTHER COURSES END -->

## ಸಹಾಯ ಪಡೆಯುವುದು

ನೀವು ಅडकಿದರೆ ಅಥವಾ AI ಆ್ಯಪ್‌ಗಳನ್ನು ನಿರ್ಮಿಸುವ ಬಗ್ಗೆ ಯಾವುದೇ ಪ್ರಶ್ನೆಗಳಿದ್ದರೆ, MCP ಕುರಿತು ಚರ್ಚೆಗಳಲ್ಲಿ ಸಹ ಪಾಠಶಾಲಾ ವಿದ್ಯಾರ್ಥಿಗಳು ಮತ್ತು ಅನುಭವಸಂಪನ್ನ ಡೆವಲಪರ್‌ಗಳೊಂದಿಗೆ ಸೇರಿ. ಪ್ರಶ್ನೆಗಳಿಗೆ ಸ್ವಾಗತವಿರುವ ಹಾಗೂ ಜ್ಞಾನವು ಮುಕ್ತವಾಗಿ ಹಂಚಿಕೊಳ್ಳಲ್ಪಡುವ ಸಹಾಯಕ ಸಮುದಾಯ ಇದು.

[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

ನೀವು ಉತ್ಪನ್ನ ಪ್ರತಿಕ್ರಿಯೆ ಅಥವಾ ನಿರ್ಮಾಣದ ಸಮಯದಲ್ಲಿ ದೋಷಗಳನ್ನು ಹೊಂದಿದ್ದರೆ ದೃಷ್ಟಿಸಿರಿ:

[![Microsoft Foundry Developer Forum](https://img.shields.io/badge/GitHub-Microsoft_Foundry_Developer_Forum-blue?style=for-the-badge&logo=github&color=000000&logoColor=fff)](https://aka.ms/foundry/forum)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ಅಸ್ವೀಕಾರ**:
ಈ ದಸ್ತಾವೇಜು AI ಅನುವಾದ ಸೇವೆ [Co-op Translator](https://github.com/Azure/co-op-translator) ಬಳಸಿ ಅನುವಾದಿಸಲಾಗಿದೆ. ನಾವು ನಿಖರತೆಯನ್ನು ಸಾಧಿಸಲು ಪ್ರಯತ್ನಿಸುತ್ತಿದ್ದರೂ, ದಯವಿಟ್ಟು ಗಮನಿಸಿ, ಸ್ವಯಂಚಾಲಿತ ಅನುವಾದಗಳಲ್ಲಿ ದೋಷಗಳು ಅಥವಾ ಅಸಡ್ಡೆಗಳು ಇರಬಹುದು. ಮೂಲ ಭಾಷೆಯಲ್ಲಿರುವ ಮೂಲ ದಸ್ತಾವೇಜು ಪ್ರಾಮಾಣಿಕ ಮೂಲವೆಂದು ಪರಿಗಣಿಸಬೇಕು. ಪ್ರಮುಖ ಮಾಹಿತಿಗಾಗಿ, ವೃತ್ತಿಪರ ಮಾನವ ಅನುವಾದವನ್ನು ಶಿಫಾರಸು ಮಾಡಲಾಗುತ್ತದೆ. ಈ ಅನುವಾದವನ್ನು ಬಳಸುವ ಮೂಲಕ ಉಂಟಾಗುವ ಯಾವುದೇ ತಪ್ಪು ಅರ್ಥಗಳ ಅಥವಾ ತಪ್ಪು ವ್ಯಾಖ್ಯಾನಗಳ ಬಗ್ಗೆ ನಾವು ಹೊಣೆಗಾರರಲ್ಲ.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->