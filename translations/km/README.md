# បច្ចេកវិទ្យាបង្កើត AI សម្រាប់អ្នកចាប់ផ្តើម - កំណែ Java
[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

![Generative AI for Beginners - Java Edition](../../translated_images/km/beg-genai-series.8b48be9951cc574c.webp)

**ពេលវេលាដំណើរការ**: វគ្គបណ្តុះបណ្តាលទាំងមូលអាចបញ្ចប់បានតាមអ៊ិនធឺណិតដោយមិនចាំបាច់ដំឡើងនៅក្នុងម៉ាស៊ីន។ ការតំឡើងបរិយាកាសប្រើប្រាស់យកពេល 2 នាទី ជាមួយការស្វែងរកឧទាហរណ៍ដែលត្រូវការពេល 1-3 ម៉ោងអាស្រ័យលើការស្វែងរកជ្រាលជ្រៅ។

> **ចាប់ផ្តើមយ៉ាងលឿន**

1. Fork ឃ្លាំងកូដនេះទៅគណនី GitHub របស់អ្នក
2. ចុច **Code** → ទំព័រ **Codespaces** → **...** → **New with options...**
3. ប្រើតម្លៃលំនាំដើម - នេះនឹងជ្រើសរើស Development container ដែលបានបង្កើតសម្រាប់វគ្គសិក្សានេះ
4. ចុច **Create codespace**
5. រង់ចាំ ~2 នាទីសម្រាប់បរិយាកាសត្រៀមរួច
6. ផ្លាស់ទៅត្រង់ [ជំពូក 2: Provision Azure AI Foundry](./02-SetupDevEnvironment/README.md#step-2-provision-azure-ai-foundry)

## គាំទ្រភាសាច្រើន

### គាំទ្រដោយ GitHub Action (ស្វ័យប្រវត្តិ និងតែងតែទាន់សម័យ)

<!-- CO-OP TRANSLATOR LANGUAGES TABLE START -->
[អារ៉ាបិក](../ar/README.md) | [បង្លាដី](../bn/README.md) | [ប៊ុលហ្គារី](../bg/README.md) | [ភាសាប៊ឺម៉ា (မြန်မာ)](../my/README.md) | [ចិន (ភាសាជាមូលដ្ឋាន)](../zh-CN/README.md) | [ចិន (ប្រពៃណី, ហុងកុង)](../zh-HK/README.md) | [ចិន (ប្រពៃណី, ម៉ាកាវ)](../zh-MO/README.md) | [ចិន (ប្រពៃណី, តៃវ៉ាន់)](../zh-TW/README.md) | [ក្រោអាទីស](../hr/README.md) | [ចិច](../cs/README.md) | [ដាណីម៉៉ាក់](../da/README.md) | [ហូឡង់](../nl/README.md) | [អេស្តូនី](../et/README.md) | [ហ្វินឡង់](../fi/README.md) | [បារាំង](../fr/README.md) | [អាល្លឺម៉ង់](../de/README.md) | [ក្រិច](../el/README.md) | [ហេប្រ៊ូវ](../he/README.md) | [ហិណ្ឌី](../hi/README.md) | [ហុងគ្រី](../hu/README.md) | [ឥណ្ឌូនេស៊ី](../id/README.md) | [អ៊ីតាលី](../it/README.md) | [ជប៉ុន](../ja/README.md) | [កណាដា](../kn/README.md) | [ខ្មែរ](./README.md) | [កូរ៉េ](../ko/README.md) | [លីទុយអានី](../lt/README.md) | [ម៉ាឡៃ](../ms/README.md) | [ម៉ាឡាឡាឡាំ](../ml/README.md) | [ម៉ារា�ធី](../mr/README.md) | [នេប៉ាល់](../ne/README.md) | [ភាសាពីជិន នីហជេរៀ](../pcm/README.md) | [ន័រវែជី](../no/README.md) | [បារშირ (ហ្វាស៊ី)](../fa/README.md) | [ប៉ូឡូញ](../pl/README.md) | [ព័រទុយហ្គាល់ (ប្រេស៊ីល)](../pt-BR/README.md) | [ព័រទុយហ្គាល់ (ប្រទេសព័រទុយហ្គាល់)](../pt-PT/README.md) | [ភាសាពុជាប៊ី (កុម្មុយហ្គ៊ី)](../pa/README.md) | [រូមានី](../ro/README.md) | [រុស្ស៊ី](../ru/README.md) | [សឺប៊ី (ស៊ីរិលិច)](../sr/README.md) | [ស្លូវ៉ាគ](../sk/README.md) | [ស្លូវេនី](../sl/README.md) | [ស្ប៉ាញ](../es/README.md) | [ស្វាហ៊ីលី](../sw/README.md) | [ស៊ុយអែត](../sv/README.md) | [តាឡាហ្គោ (ហ្វ៊ីលីពីន)](../tl/README.md) | [តាមីល](../ta/README.md) | [ទេលូហ្គូ](../te/README.md) | [ថៃ](../th/README.md) | [ទួរគី](../tr/README.md) | [អ៊ុយក្រែន](../uk/README.md) | [អ៊ួរឌូ](../ur/README.md) | [វៀតណាម](../vi/README.md)

> **ចង់បញ្ចូលកូដនៅក្នុងម៉ាស៊ីនរបស់អ្នកទេ?**
>
> ឃ្លាំងកូដនេះមានបទបកប្រែជាភាសា 50+ ដែលបន្ថែមទំហំទាញយកយ៉ាងច្រើន។ ដើម្បីបញ្ចូលដោយគ្មានការប្រែភាសា សូមប្រើ sparse checkout:
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
> វានាំឱ្យអ្នកមានគ្រប់អ្វីដែលអ្នកត្រូវការសម្រាប់បញ្ចប់វគ្គសិក្សានេះដោយការទាញយកលឿនជាងមុន។
<!-- CO-OP TRANSLATOR LANGUAGES TABLE END -->

## រចនាសម្ព័ន្ធវគ្គសិក្សា និងផ្លូវការសិក្សា

### **ជំពូកទី 1: ការជូនដំណឹងអំពីបច្ចេកវិទ្យាបង្កើត AI**
- **គំនិតមូលដ្ឋាន**: យល់ដឹងអំពីម៉ូដែលភាសាធំៗ, token, embeddings, និងសមត្ថភាព AI
- **បរិស្ថាន Java AI**: សង្ខេបអំពី Spring AI និង OpenAI SDKs
- **Model Context Protocol**: រាប់បញ្ចូល MCP និងតួនាទីរបស់វាក្នុងការទំនាក់ទំនងរវាងភ្នែកAI
- **ករណីការ​ប្រើប្រាស់ជាក់ស្តែង**: ស្ថានការ​ពិត​ជាក់ស្តែងរួមទាំង chatbot និងការបង្កើតមាតិកា
- **[→ ចាប់ផ្តើមជំពូក 1](./01-IntroToGenAI/README.md)**

### **ជំពូកទី 2: ការតំឡើងបរិយាកាសអភិវឌ្ឍន៍**
- **Azure AI Foundry**: Provision ការចេញផ្សាយម៉ូដែលជាគោលការណ៍កូដជាមួយ Bicep និង Azure Developer CLI (azd)
- **Spring Boot + Spring AI**: វិធីល្អបំផុតសម្រាប់អភិវឌ្ឍកម្មវិធី AI សម្រាប់សហគ្រាស
- **Keyless Authentication**: តភ្ជាប់ដោយសុវត្ថិភាពជាមួយ Microsoft Entra ID — មិនចាំបាច់មានកូនសោ API
- **ឧបករណ៍អភិវឌ្ឍន៍**: កុងតឺណ័រ Docker, VS Code, និងការកំណត់ GitHub Codespaces
- **[→ ចាប់ផ្តើមជំពូក 2](./02-SetupDevEnvironment/README.md)**

### **ជំពូកទី 3: បច្ចេកទេសស្នូលបង្កើត AI**
- **Prompt Engineering**: បច្ចេកទេសសម្រាប់ចម្លើយម៉ូដែល AI ល្អបំផុត
- **Embeddings និងប្រតិបត្តិការវ៉ិចទ័រ**: អនុវត្តស្វែងរកតាមអត្ថន័យ និងការផ្គូផ្គងស្រដៀងគ្នា
- **Retrieval-Augmented Generation (RAG)**: ផ្សំនឹង AI ជាមួយប្រភពទិន្នន័យផ្ទាល់ខ្លួន
- **Function Calling**: ពង្រីកសមត្ថភាព AI ជាមួយឧបករណ៍ និងផ្លាស៊ីនផ្ទាល់ខ្លួន
- **[→ ចាប់ផ្តើមជំពូក 3](./03-CoreGenerativeAITechniques/README.md)**

### **ជំពូកទី 4: ករណីប្រើប្រាស់ និងគម្រោងជាក់ស្តែង**
- **កម្មវិធីបង្កើតរឿងរ៉ាវសត្វចិញ្ចឹម** (`petstory/`): បង្កើតមាតិកាបែបច្នៃប្រឌិតជាមួយ Azure AI Foundry
- **ការតំណាង Foundry ក្នុងម៉ាស៊ីនថែមតែជិត** (`foundrylocal/`): ដំណើរការម៉ូដែល AI ក្នុងម៉ាស៊ីនដោយប្រើ OpenAI Java SDK
- **សេវាកម្មគណនា MCP** (`calculator/`): អនុវត្ត Model Context Protocol ជាមូលដ្ឋានជាមួយ Spring AI
- **[→ ចាប់ផ្តើមជំពូក 4](./04-PracticalSamples/README.md)**

### **ជំពូកទី 5: ការអភិវឌ្ឍ AI ដោយគោរពក្រមបញ្ញត្តិ**
- **Azure AI Foundry Content Safety**: សាកល្បងមេកានិចបំផុតក្នុងការត្រួតពិនិត្យមាតិកា និងសុវត្ថិភាព (ប្លុកតឹង និងការព្រមានទន់)
- **ការបង្ហាញសុវត្ថិភាព AI**: ឧទាហរណ៍ជាក់ស្តែងបង្ហាញពីរបៀបប្រព័ន្ធសុវត្ថិភាព AI សម័យថ្មីដំណើរការ
- **វិធីល្អបំផុត**: ណែនាំសម្រាប់ការអភិវឌ្ឍ និងចេញផ្សាយ AI យ៉ាងមានទំនួលខុសត្រូវ
- **[→ ចាប់ផ្តើមជំពូក 5](./05-ResponsibleGenAI/README.md)**

## ឯកសារឧបករណ៍បន្ថែម

<!-- CO-OP TRANSLATOR OTHER COURSES START -->
### LangChain
[![LangChain4j សម្រាប់អ្នកចាប់ផ្តើម](https://img.shields.io/badge/LangChain4j%20for%20Beginners-22C55E?style=for-the-badge&&labelColor=E5E7EB&color=0553D6)](https://aka.ms/langchain4j-for-beginners)
[![LangChain.js សម្រាប់អ្នកចាប់ផ្តើម](https://img.shields.io/badge/LangChain.js%20for%20Beginners-22C55E?style=for-the-badge&labelColor=E5E7EB&color=0553D6)](https://aka.ms/langchainjs-for-beginners?WT.mc_id=m365-94501-dwahlin)
[![LangChain សម្រាប់អ្នកចាប់ផ្តើម](https://img.shields.io/badge/LangChain%20for%20Beginners-22C55E?style=for-the-badge&labelColor=E5E7EB&color=0553D6)](https://github.com/microsoft/langchain-for-beginners?WT.mc_id=m365-94501-dwahlin)
---

### Azure / Edge / MCP / Agents
[![AZD សម្រាប់អ្នកចាប់ផ្តើម](https://img.shields.io/badge/AZD%20for%20Beginners-0078D4?style=for-the-badge&labelColor=E5E7EB&color=0078D4)](https://github.com/microsoft/AZD-for-beginners?WT.mc_id=academic-105485-koreyst)
[![Edge AI សម្រាប់អ្នកចាប់ផ្តើម](https://img.shields.io/badge/Edge%20AI%20for%20Beginners-00B8E4?style=for-the-badge&labelColor=E5E7EB&color=00B8E4)](https://github.com/microsoft/edgeai-for-beginners?WT.mc_id=academic-105485-koreyst)
[![MCP សម្រាប់អ្នកចាប់ផ្តើម](https://img.shields.io/badge/MCP%20for%20Beginners-009688?style=for-the-badge&labelColor=E5E7EB&color=009688)](https://github.com/microsoft/mcp-for-beginners?WT.mc_id=academic-105485-koreyst)
[![កម្មវិធីភ្នែក AI សម្រាប់អ្នកចាប់ផ្តើម](https://img.shields.io/badge/AI%20Agents%20for%20Beginners-00C49A?style=for-the-badge&labelColor=E5E7EB&color=00C49A)](https://github.com/microsoft/ai-agents-for-beginners?WT.mc_id=academic-105485-koreyst)

---
 
### បណ្តោះអាសន្ន Generative AI
[![Generative AI សម្រាប់អ្នកចាប់ផ្តើម](https://img.shields.io/badge/Generative%20AI%20for%20Beginners-8B5CF6?style=for-the-badge&labelColor=E5E7EB&color=8B5CF6)](https://github.com/microsoft/generative-ai-for-beginners?WT.mc_id=academic-105485-koreyst)
[![Generative AI (.NET)](https://img.shields.io/badge/Generative%20AI%20(.NET)-9333EA?style=for-the-badge&labelColor=E5E7EB&color=9333EA)](https://github.com/microsoft/Generative-AI-for-beginners-dotnet?WT.mc_id=academic-105485-koreyst)
[![Generative AI (Java)](https://img.shields.io/badge/Generative%20AI%20(Java)-C084FC?style=for-the-badge&labelColor=E5E7EB&color=C084FC)](https://github.com/microsoft/generative-ai-for-beginners-java?WT.mc_id=academic-105485-koreyst)
[![Generative AI (JavaScript)](https://img.shields.io/badge/Generative%20AI%20(JavaScript)-E879F9?style=for-the-badge&labelColor=E5E7EB&color=E879F9)](https://github.com/microsoft/generative-ai-with-javascript?WT.mc_id=academic-105485-koreyst)

---
 
### ការសិក្សាដើម
[![ML សម្រាប់អ្នកចាប់ផ្តើម](https://img.shields.io/badge/ML%20for%20Beginners-22C55E?style=for-the-badge&labelColor=E5E7EB&color=22C55E)](https://aka.ms/ml-beginners?WT.mc_id=academic-105485-koreyst)
[![Data Science សម្រាប់អ្នកចាប់ផ្តើម](https://img.shields.io/badge/Data%20Science%20for%20Beginners-84CC16?style=for-the-badge&labelColor=E5E7EB&color=84CC16)](https://aka.ms/datascience-beginners?WT.mc_id=academic-105485-koreyst)
[![AI សម្រាប់អ្នកចាប់ផ្តើម](https://img.shields.io/badge/AI%20for%20Beginners-A3E635?style=for-the-badge&labelColor=E5E7EB&color=A3E635)](https://aka.ms/ai-beginners?WT.mc_id=academic-105485-koreyst)
[![សុវត្ថិភាពបច្ចេកវិទ្យា (Cybersecurity) សម្រាប់អ្នកចាប់ផ្តើម](https://img.shields.io/badge/Cybersecurity%20for%20Beginners-F97316?style=for-the-badge&labelColor=E5E7EB&color=F97316)](https://github.com/microsoft/Security-101?WT.mc_id=academic-96948-sayoung)
[![អ្នកអភិវឌ្ឍន៍វេបសម្រាប់អ្នកចាប់ផ្តើម](https://img.shields.io/badge/Web%20Dev%20for%20Beginners-EC4899?style=for-the-badge&labelColor=E5E7EB&color=EC4899)](https://aka.ms/webdev-beginners?WT.mc_id=academic-105485-koreyst)
[![IoT សម្រាប់អ្នកចាប់ផ្តើម](https://img.shields.io/badge/IoT%20for%20Beginners-14B8A6?style=for-the-badge&labelColor=E5E7EB&color=14B8A6)](https://aka.ms/iot-beginners?WT.mc_id=academic-105485-koreyst)
[![ការអភិវឌ្ឍ XR សម្រាប់អ្នកចាប់ផ្តើម](https://img.shields.io/badge/XR%20Development%20for%20Beginners-38BDF8?style=for-the-badge&labelColor=E5E7EB&color=38BDF8)](https://github.com/microsoft/xr-development-for-beginners?WT.mc_id=academic-105485-koreyst)

---
 
### ស៊េរី Copilot
[![Copilot សម្រាប់ការសរសេរកម្មវិធីរួមជាមួយ AI](https://img.shields.io/badge/Copilot%20for%20AI%20Paired%20Programming-FACC15?style=for-the-badge&labelColor=E5E7EB&color=FACC15)](https://aka.ms/GitHubCopilotAI?WT.mc_id=academic-105485-koreyst)
[![Copilot សម្រាប់ C#/.NET](https://img.shields.io/badge/Copilot%20for%20C%23/.NET-FBBF24?style=for-the-badge&labelColor=E5E7EB&color=FBBF24)](https://github.com/microsoft/mastering-github-copilot-for-dotnet-csharp-developers?WT.mc_id=academic-105485-koreyst)
[![ការផ្សងព្រេង Copilot](https://img.shields.io/badge/Copilot%20Adventure-FDE68A?style=for-the-badge&labelColor=E5E7EB&color=FDE68A)](https://github.com/microsoft/CopilotAdventures?WT.mc_id=academic-105485-koreyst)
<!-- CO-OP TRANSLATOR OTHER COURSES END -->

## ទទួលបានជំនួយ

បើអ្នកឈ្នះឬមានសំណួរណាមួយអំពីការបង្កើតកម្មវិធី AI។ ចូលរួមជាមួយសិស្សរៀន និង អភិវឌ្ឍន៍ដែលមានបទពិសោធន៍ក្នុងការពិភាក្សាអំពី MCP។ វាជាសហគមន៍គាំទ្រដែលសំណួរត្រូវបានស្វាគមន៍ ហើយចំណេះដឹងត្រូវបានចែករំលែកដោយមិនគិតថ្លៃ។

[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

បើអ្នកមានមតិយោបល់ផលិតផល ឬ មានកំហុសពេលកំពុងបង្កើត សូមទៅ៖

[![Microsoft Foundry Developer Forum](https://img.shields.io/badge/GitHub-Microsoft_Foundry_Developer_Forum-blue?style=for-the-badge&logo=github&color=000000&logoColor=fff)](https://aka.ms/foundry/forum)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ការបដិសេធ**:
ឯកសារនេះត្រូវបានបម្លែងភាសា ដោយប្រើសេវាបម្លែងភាសា AI [Co-op Translator](https://github.com/Azure/co-op-translator)។ ទោះយើងខ្ញុំមានក្តីប្រាថ្នាឱ្យបានច្បាស់លាស់ តែសូមយល់ដឹងថាការបម្លែងដោយស្វ័យប្រវត្តិក៏អាចមានកំហុសឬភាពមិនត្រឹមត្រូវ។ ឯកសារដើមជាភាសាទីតាំងគួរត្រូវបានគេប្រើជាប្រភពច្បាស់លាស់។ សម្រាប់ព័ត៌មានសំខាន់ៗ សូមណែនាំឱ្យប្រើប្រាស់ការប្រែដោយមនុស្សជំនាញ។ យើងខ្ញុំមិនទទួលខុសត្រូវចំពោះការយល់ច្រឡំ ឬការបកស្រាយខុសបន្ទាប់ពីការប្រើប្រាស់ការបម្លែងនេះនោះទេ។
<!-- CO-OP TRANSLATOR DISCLAIMER END -->