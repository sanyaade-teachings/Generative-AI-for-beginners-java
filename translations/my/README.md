# စတင်သင်ယူသူများအတွက် Generative AI - Java ဗားရှင်း
[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

![Generative AI for Beginners - Java Edition](../../translated_images/my/beg-genai-series.8b48be9951cc574c.webp)

**အချိန်ပေးဆောင်မှု**: အတန်းတစ်ခုလုံးကို တိုက်ရိုက်တက်ခြင်းမလိုဘဲ အွန်လိုင်းမှတဆင့် ပြီးမြောက်နိုင်ပါသည်။ ပတ်ဝန်းကျင်တည်ဆောက်ခြင်းမှာ မိနစ် ၂ ခန့်ယူပြီး၊ နမူနာများကို စူးစမ်းတာမှာ စူးစမ်းမှုအနက်မှတ်အလျားအလိုက် ၁-၃ နာရီ ကြာနိုင်သည်။

> **အမြန် စတင်ခြင်း** 

1. ဤဂစ်ဟတ် ရေပိုင်စုကို သင့် GitHub အကောင့်သို့ fork ဆွဲပါ
2. **Code** → **Codespaces** tab → **...** → **New with options...** ကို နှိပ်ပါ
3. ပုံမှန်အတိုင်း အသုံးပြုပြီး၊ ဤသင်တန်းအတွက် ဖန်တီးထားသော Development container ကို ရွေးချယ်ပါ
4. **Create codespace** ကိုနှိပ်ပါ
5. ပတ်ဝန်းကျင် အသင့်ဖြစ်ရန် မိနစ် ၂ ခန့် စောင့်ပါ
6. တိုက်ရိုက် [Chapter 2: Provision Azure AI Foundry](./02-SetupDevEnvironment/README.md#step-2-provision-azure-ai-foundry) သို့ ရောက်ရန်

## ဘာသာစကား အမျိုးမျိုးကို ထောက်ပံ့ခြင်း

### GitHub Action (အလိုအလျောက် နှင့် အမြဲနောက်ဆုံးပေါ်) မှတဆင့် ထောက်ပံ့ထားသည်

<!-- CO-OP TRANSLATOR LANGUAGES TABLE START -->
[Arabic](../ar/README.md) | [Bengali](../bn/README.md) | [Bulgarian](../bg/README.md) | [Burmese (Myanmar)](./README.md) | [Chinese (Simplified)](../zh-CN/README.md) | [Chinese (Traditional, Hong Kong)](../zh-HK/README.md) | [Chinese (Traditional, Macau)](../zh-MO/README.md) | [Chinese (Traditional, Taiwan)](../zh-TW/README.md) | [Croatian](../hr/README.md) | [Czech](../cs/README.md) | [Danish](../da/README.md) | [Dutch](../nl/README.md) | [Estonian](../et/README.md) | [Finnish](../fi/README.md) | [French](../fr/README.md) | [German](../de/README.md) | [Greek](../el/README.md) | [Hebrew](../he/README.md) | [Hindi](../hi/README.md) | [Hungarian](../hu/README.md) | [Indonesian](../id/README.md) | [Italian](../it/README.md) | [Japanese](../ja/README.md) | [Kannada](../kn/README.md) | [Khmer](../km/README.md) | [Korean](../ko/README.md) | [Lithuanian](../lt/README.md) | [Malay](../ms/README.md) | [Malayalam](../ml/README.md) | [Marathi](../mr/README.md) | [Nepali](../ne/README.md) | [Nigerian Pidgin](../pcm/README.md) | [Norwegian](../no/README.md) | [Persian (Farsi)](../fa/README.md) | [Polish](../pl/README.md) | [Portuguese (Brazil)](../pt-BR/README.md) | [Portuguese (Portugal)](../pt-PT/README.md) | [Punjabi (Gurmukhi)](../pa/README.md) | [Romanian](../ro/README.md) | [Russian](../ru/README.md) | [Serbian (Cyrillic)](../sr/README.md) | [Slovak](../sk/README.md) | [Slovenian](../sl/README.md) | [Spanish](../es/README.md) | [Swahili](../sw/README.md) | [Swedish](../sv/README.md) | [Tagalog (Filipino)](../tl/README.md) | [Tamil](../ta/README.md) | [Telugu](../te/README.md) | [Thai](../th/README.md) | [Turkish](../tr/README.md) | [Ukrainian](../uk/README.md) | [Urdu](../ur/README.md) | [Vietnamese](../vi/README.md)

> **ကိုယ်တိုင် ဒေါင်းလုတ်လုပ်ချင်ပါသလား?**
>
> ဤရေပိုင်စုတွင် ဘာသာစကား ၅၀ ကျော် ထည့်သွင်းထားပြီး ဒါကြောင့် ဒေါင်းလုတ်ဖိုင်အရွယ်အစား ကြီးတက်သည်။ ဘာသာပြန် အပါအဝင် မထည့်ဘဲ clone ဆွဲချင်လျှင် sparse checkout ကို အသုံးပြုပါ။
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
> ဤနည်းတူ သင်တန်း ပြီးမြောက်ရန် လိုအပ်သည့် အရာအားလုံးကို ပိုမိုလျင်မြန်စွာ ရယူနိုင်မှာ ဖြစ်သည်။
<!-- CO-OP TRANSLATOR LANGUAGES TABLE END -->

## သင်တန်း ဖွဲ့စည်းမှုနှင့် သင်ယူမှု လမ်းကြောင်း

### **အခန်း ၁: Generative AI မိတ်ဆက်ခြင်း**
- **အခြေခံအတွေးအခေါ်များ**: Language Models ကြီးများ၊ token များ၊ embeddings နှင့် AI ၏ အင်အားများကို နားလည်ခြင်း
- **Java AI ပတ်ဝန်းကျင်**: Spring AI နှင့် OpenAI SDK များအကြောင်းအကျဉ်း
- **Model Context Protocol**: MCP မိတ်ဆက်ခြင်းနှင့် AI agent များကြား ဆက်သွယ်မှုအတွက် အခန်းကဏ္ဍ
- **လက်တွေ့အသုံးပြုမှုများ**: ဆွေးနွေးခြင်းနည်းပညာများနှင့် အကြောင်းအရာ ဖန်တီးမှု အခြေအနေများ
- **[→ အခန်း ၁ စတင်ရန်](./01-IntroToGenAI/README.md)**

### **အခန်း ၂: ဖွံ့ဖြိုးတိုးတက်မှု ပတ်ဝန်းကျင် တည်ဆောက်ခြင်း**
- **Azure AI Foundry**: Bicep နှင့် Azure Developer CLI (azd) အသုံးပြု၍ မော်ဒယ် deployment များကို ကုတ်ဖြင့် provision လုပ်ခြင်း
- **Spring Boot + Spring AI**: စက်မှုလုပ်ငန်း AI လျှောက်လွှာ ဖွံ့ဖြိုးရေးအတွက် သင့်တော်သောနည်းလမ်းများ
- **နိုင်ငံသားအသုံးပြုထားခြင်း (Keyless Authentication)**: Microsoft Entra ID ဖြင့် လုံခြုံစိတ်ချစွာ ချိတ်ဆက်ခြင်း — API key မလိုအပ်ပါ
- **ဖွံ့ဖြိုးရေး ကိရိယာများ**: Docker container များ၊ VS Code နှင့် GitHub Codespaces ပြင်ဆင်ခြင်း
- **[→ အခန်း ၂ စတင်ရန်](./02-SetupDevEnvironment/README.md)**

### **အခန်း ၃: Core Generative AI နည်းလမ်းများ**
- **Prompt Engineering**: AI မော်ဒယ်များမှ အကောင်းဆုံး တုံ့ပြန်မှုရရှိအောင် နည်းဗျူဟာများ
- **Embeddings နှင့် Vector လုပ်ဆောင်ချက်များ**: အဓိပ္ပါယ်လိုက်ရှာဖွေခြင်းနှင့် ဆင်တူမှု ကိုက်ညီမှုလုပ်ငန်းများ
- **Retrieval-Augmented Generation (RAG)**: ကိုယ်ပိုင် ဒေတာအရင်းမြစ်များနှင့် AI ပေါင်းစပ်ခြင်း
- **Function Calling**: AI ၏ အင်အားများကို စိတ်ကြိုက် ကိရိယာများနှင့် plugin များဖြင့် တိုးချဲ့ခြင်း
- **[→ အခန်း ၃ စတင်ရန်](./03-CoreGenerativeAITechniques/README.md)**

### **အခန်း ၄: လက်တွေ့အသုံးပြုမှုများနှင့် ပရောဂျက်များ**
- **Pet Story Generator** (`petstory/`): Azure AI Foundry အသုံးပြု၍ ဖန်တီးမှုအကြောင်းအရာ ထုတ်လုပ်ခြင်း
- **Foundry Local Demo** (`foundrylocal/`): OpenAI Java SDK ဖြင့် ဒေသဆိုင်ရာ AI မော်ဒယ် ပေါင်းစပ်ခြင်း
- **MCP Calculator Service** (`calculator/`): Spring AI ဖြင့် Model Context Protocol အခြေခံ အကောင်အထည်ဖော်ခြင်း
- **[→ အခန်း ၄ စတင်ရန်](./04-PracticalSamples/README.md)**

### **အခန်း ၅: တာဝန်ယူသော AI ဖွံ့ဖြိုးရေး**
- **Azure AI Foundry အကြောင်းအရာ ဘေးကင်းရေး**: ဖန်တီးထားသော အကြောင်းအရာ စစ်ထုတ်ခြင်းနှင့် ဘေးကင်းရေး စနစ်များကို စမ်းသပ်ခြင်း (အခက်အခဲတားဆီးခြင်းနှင့် ပြောကြားမှု အနုတ်လက္ခဏာများ)
- **တာဝန်ယူသည့် AI မူကွဲ**: ခေတ်မှီ AI ဘေးကင်းရေးစနစ်များ ဂရင်းကြမ်းမြင်ကွင်းကို အမြင်ရစေသည့် လက်တွေ့ ဥပမာ
- **အကောင်းဆုံးသုံးစွဲရန် နည်းစနစ်များ**: တာဝန်ယူသော AI ဖွံ့ဖြိုးရေးနှင့် တပ်ဆင်ခြင်း အတွက် အခြေခံ ညွှန်ကြားချက်များ
- **[→ အခန်း ၅ စတင်ရန်](./05-ResponsibleGenAI/README.md)**

## အပိုဆောင်း အရင်းအမြစ်များ

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
 
### Core Learning
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

## အကူအညီ ရယူခြင်း

AI အက်ပ်တွေ ဖန်တီးရာမှာ စားနာမှုရှိသော်လည်း မေးခွန်းများ လွယ်ကူစွာမေးနိုင်တယ်။ MCP အကြောင်းကို ကျောင်းသားတွေနဲ့ အတွေ့အကြုံရှိ ထုတ်လုပ်သူတွေနဲ့ ဆွေးနွေးနိုင်တဲ့ အခန်းကဏ္ဍဖြစ်ပြီး မေးခွန်းတွေကို ကြိုဆိုပေးပြီး အသိပညာကို လွတ်လပ်စွာ မျှဝေပေးတယ်။

[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

ထုတ်ကုန် တုံ့ပြန်ချက်များ သို့မဟုတ် တုတ်ယုတ်ချက်များ ရှိပါက ဒီနေရာမှ ကြည့်ရှုပါ။

[![Microsoft Foundry Developer Forum](https://img.shields.io/badge/GitHub-Microsoft_Foundry_Developer_Forum-blue?style=for-the-badge&logo=github&color=000000&logoColor=fff)](https://aka.ms/foundry/forum)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ပြောကြားချက်**
ဤစာတမ်းကို AI ဘာသာပြန်ဝန်ဆောင်မှု [Co-op Translator](https://github.com/Azure/co-op-translator) အသုံးပြု၍ ဘာသာပြန်ထားပါသည်။ ကျွန်ုပ်တို့သည် တိကျမှန်ကန်မှုအတွက် ကြိုးပမ်းနေသော်လည်း၊ စက်ကိရိယာဘာသာပြန်ခြင်းများတွင် အမှားများ သို့မဟုတ် မှားယွင်းချက်များ ပါဝင်နိုင်ကြောင်း သတိပြုပါရန် လိုအပ်ပါသည်။ မူလစာတမ်းကို မူရင်းဘာသာဖြင့်သာ ယုံကြည်စိတ်ချရသော အချက်အလက်အဖြစ် သတ်မှတ်သင့်သည်။ အရေးကြီးသည့် သတင်းအချက်အလက်များအတွက် ပရော်ဖက်ရှင်နယ် လူသားဘာသာပြန်သူဝန်ဆောင်မှုကို အကြံပြုပါသည်။ ဤဘာသာပြန်ချက်ကို အသုံးပြုခြင်းမှ ဖြစ်ပေါ်လာသော နားလည်မှုကွာခြားမှုများ သို့မဟုတ် မမှန်ကန်သော အသုံးပြုမှုများအတွက် ကျွန်ုပ်တို့ တာဝန်မခံပါ။
<!-- CO-OP TRANSLATOR DISCLAIMER END -->