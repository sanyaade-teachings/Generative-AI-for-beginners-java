# បច្ចេកវិទ្យា AI បង្កើតសម្រាប់អ្នកចាប់ផ្តើម - កំណត់ជាភាសា Java
[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

![Generative AI for Beginners - Java Edition](../../translated_images/km/beg-genai-series.8b48be9951cc574c.webp)

**ការប្តេជ្ញាចិត្តពេលវេលា**: វគ្គបង្រៀនទាំងមូលអាចបញ្ចប់តាមអនឡាញដោយគ្មានការតំឡើងក្នុងកន្លែង។ ការតំឡើងបរិយាកាសប្រើប្រាស់ចំណាយពេលប្រហែល 2 នាទី ហើយការស្វែងយល់ពីគំរូប្រើប្រាស់ពេលវេលា 1-3 ម៉ោងអាស្រ័យលើកម្រិតនៃការស្វែងយល់។

> **ការចាប់ផ្តើមយ៉ាងរហ័ស** 

1. កូនស្មើរ រកមូលដ្ឋានកូដនេះទៅគណនី GitHub របស់អ្នក
2. ចុច **Code** → ផ្ទាំង **Codespaces** → **...** → **New with options...**
3. ប្រើការកំណត់លំនាំដើម – នេះនឹងជ្រើសប្រអប់ភាសារអភិវឌ្ឍដែលបានបង្កើតសម្រាប់វគ្គនេះ
4. ចុច **Create codespace**
5. រង់ចាំប្រហែល ~2 នាទី សម្រាប់បរិយាកាសត្រៀមរួច
6. ទៅផ្ទាល់ទៅកាន់ [ឧទាហរណ៍ដំបូង](./02-SetupDevEnvironment/README.md#step-2-create-a-github-personal-access-token)

## ការគាំទ្រភាសាច្រើន

### គាំទ្រតាមរយៈ GitHub Action (ស្វ័យប្រវត្តិ & អាប់ដេតជានិច្ច)

<!-- CO-OP TRANSLATOR LANGUAGES TABLE START -->
[អារ៉ាប់](../ar/README.md) | [បង់ក្លា](../bn/README.md) | [ប៊ុលហ្គារី](../bg/README.md) | [ម្យ៉ាន់ម៉ា (ភាសាចិន)](../my/README.md) | [ចិន (សាមញ្ញ)](../zh-CN/README.md) | [ចិន (ប្រពៃណី, ហុងកុង)](../zh-HK/README.md) | [ចិន (ប្រពៃណី, ម៉ាកាវ)](../zh-MO/README.md) | [ចិន (ប្រពៃណី, តៃវ៉ាន់)](../zh-TW/README.md) | [ក្រូអាត](../hr/README.md) | [ចេច](../cs/README.md) | [ដាណឺម៉ា](../da/README.md) | [ដូជា](../nl/README.md) | [អេស្តូនី](../et/README.md) | [ហ្វិន្លង់](../fi/README.md) | [បារាំង](../fr/README.md) | [អាល្លឺម៉ង់](../de/README.md) | [ក្រេក](../el/README.md) | [ហេប្រ៊ូ](../he/README.md) | [ហិណ្ឌី](../hi/README.md) | [ហុងគ្រី](../hu/README.md) | [ឥណ្ឌូនេស៊ី](../id/README.md) | [អ៊ីតាលី](../it/README.md) | [ជប៉ុន](../ja/README.md) | [កណ្ណាដា](../kn/README.md) | [ខ្មែរ](./README.md) | [កូរ៉េ](../ko/README.md) | [ល៊ីទុយអាណា](../lt/README.md) | [ម៉ាឡេស៊ី](../ms/README.md) | [ម៉ាឡាលាម](../ml/README.md) | [ម៉ារាធី](../mr/README.md) | [នេប៉ាល](../ne/README.md) | [ភីជាំង​នៃ​ណាញជេរី](../pcm/README.md) | [ណ័រ៉េវ៉េល](../no/README.md) | [ភាសាព្រីស៊្យ៉ាន់ (ហ្វាស៊ី)](../fa/README.md) | [ប៉ូឡូញ](../pl/README.md) | [ប៉ូរទុយហ្គាល់ (ប្រេស៊ីល)](../pt-BR/README.md) | [ប៉ូរទុយហ្គាល់ (ប៉ូរទុយហ្គាល់)](../pt-PT/README.md) | [ពាណជាប៊ី (គួរ​មុខី)](../pa/README.md) | [រ៉ូមានី](../ro/README.md) | [រុស្ស៊ី](../ru/README.md) | [សឺរប៊ី (ស៊ីរីលិក)](../sr/README.md) | [ស្លូវ៉ាក់](../sk/README.md) | [ស្លូវេនី](../sl/README.md) | [អេស្ប៉ាញ](../es/README.md) | [ស្វាហ៊ីលី](../sw/README.md) | [ស៊ុយអែដ](../sv/README.md) | [តាឡាហ្គោ (ហ្វីលីពីណូ)](../tl/README.md) | [តាមីល](../ta/README.md) | [តេលូហ្គូ](../te/README.md) | [ថៃ](../th/README.md) | [ទួរគី](../tr/README.md) | [អ៊ុយក្រែន](../uk/README.md) | [អ៊ុយរឌូ](../ur/README.md) | [វៀតណាម](../vi/README.md)

> **ចូលចិត្តបញ្ចូលក្នុងកុំព្យូទ័រនៅក្នុងតំបន់មែនទេ?**
>
> មូលដ្ឋានកូដនេះមានការបកប្រែជាង 50+ ភាសា ដែលបង្កើនទំហំទាញយកយ៉ាងច្រើន។ ដើម្បីចម្លងគ្មានការបកប្រែ អ្នកអាចប្រើ sparse checkout៖
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
> នេះបង្រួមតម្លៃបញ្ចប់វគ្គដែលអ្នកត្រូវការជាមួយការទាញយកលឿនជាងមុន។
<!-- CO-OP TRANSLATOR LANGUAGES TABLE END -->

## រចនាសម្ព័ន្ធវគ្គបណ្តុះបណ្តាល និងផ្លូវសិក្សា

### **ជំពូកទី 1៖ ការណែនាំអំពី AI បង្កើត**
- **កំណត់គន្លឹះសំខាន់ៗ**៖ ការយល់ដឹងអំពីម៉ូឌែលភាសាធំៗ, តូគិន, embeddings និងសមត្ថភាព AI
- **បរិយាកាស AI Java**៖ ទិដ្ឋភាពទូលំទូលាយនៃ Spring AI និង OpenAI SDKs
- **ពិធីសាស្រ្ត Model Context Protocol**៖ ការណែនាំ MCP និងតួនាទីរបស់វាក្នុងការទំនាក់ទំនងតាមភ្នាក់ងារ AI
- **ការអនុវត្តនៅក្នុងជីវិតពិត**៖ ស្ថានភាពពិតប្រាកដ រួមទាំង chatbot និងការបង្កើតមាតិកា
- **[→ ចាប់ផ្តើមជំពូកទី 1](./01-IntroToGenAI/README.md)**

### **ជំពូកទី 2៖ ការតំឡើងបរិយាកាសអភិវឌ្ឍន៍**
- **Azure AI Foundry**៖ ផ្ដល់សេវាកម្មចែកចាយម៉ូឌែលជាកូដជាមួយ Bicep និង Azure Developer CLI (azd)
- **Spring Boot + Spring AI**៖ អនុវត្តលទ្ធផលល្អបំផុតសម្រាប់ការអភិវឌ្ឍកម្មវិធី AI សម្រាប់សហគ្រាស
- **សំងាត់គ្មានគ្រាប់សោ**៖ ភ្ជាប់ដោយសុវត្ថិភាពជាមួយ Microsoft Entra ID — គ្មានកូនសោ API ត្រូវគ្រប់គ្រង
- **ឧបករណ៍អភិវឌ្ឍន៍**៖ កុងតឺន័រ Docker, VS Code និងការកំណត់ GitHub Codespaces
- **[→ ចាប់ផ្តើមជំពូកទី 2](./02-SetupDevEnvironment/README.md)**

### **ជំពូកទី 3៖ បច្ចេកទេសចម្បងនៃ AI បង្កើត**
- **ការជំនាញបង្កើតលំនាំ**៖ បច្ចេកទេសសម្រាប់ការឆ្លើយតបម៉ូឌែល AI ឲ្យល្អបំផុត
- **Embeddings និងប្រតិបត្តិការវិចទ័រ**៖ អនុវត្តស្វែងរកអត្ថន័យ និងការប្រៀបធៀបលក្ខណៈស្រដៀង
- **Retrieval-Augmented Generation (RAG)**៖ បញ្ចូល AI ជាមួយប្រភពទិន្នន័យផ្ទាល់ខ្លួនរបស់អ្នក
- **ការហៅមុខងារ**៖ បន្ថែមសមត្ថភាព AI ជាមួយឧបករណ៍ និងផ្នែកបន្ថែមផ្ទាល់ខ្លួន
- **[→ ចាប់ផ្តើមជំពូកទី 3](./03-CoreGenerativeAITechniques/README.md)**

### **ជំពូកទី 4៖ ការអនុវត្តន៍ក្នុងជីវិតពិត និងគម្រោង**
- **កម្មវិធីបង្កើតរឿងច្រៀងសត្វចិញ្ចឹម** (`petstory/`): ការបង្កើតមាតិការចនាដោយ Azure AI Foundry
- **កម្មវិធីបង្ហាញ Foundry Local** (`foundrylocal/`): ការបញ្ចូលម៉ូឌែល AI ក្នុងតំបន់ជាមួយ OpenAI Java SDK
- **សេវាកម្មគណនាគ្រប់គ្រង MCP** (`calculator/`): អនុវត្តផ្នែក Model Context Protocol មូលដ្ឋានជាមួយ Spring AI
- **[→ ចាប់ផ្តើមជំពូកទី 4](./04-PracticalSamples/README.md)**

### **ជំពូកទី 5៖ ការអភិវឌ្ឍ AI ដ៏ទទួលខុសត្រូវ**
- **Azure AI Foundry សុវត្ថិភាពមាតិកា**៖ សាកល្បងការត្រួតពិនិត្យមាតិកា និងមុខងារសុវត្ថិភាព (ការរារាំងរឹងមាំ និងការបដិសេធប្រាក់ទន់)
- **ការបង្ហាញ AI សូមទណ្ឌកម្ម**៖ ឧទាហរណ៍ដាក់អាជីពណែនាំរបៀបដែលប្រព័ន្ធសុវត្ថិភាព AI សម័យទំនើបដំណើរការជាក់ស្តែង
- **វិធីសាស្រ្តល្អបំផុត**៖ យោបល់សំខាន់ៗសម្រាប់ការអភិវឌ្ឍ និងចែកចាយ AI ដោយមានទំនួលខុសត្រូវ
- **[→ ចាប់ផ្តើមជំពូកទី 5](./05-ResponsibleGenAI/README.md)**

## ឯកសារបន្ថែម

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
[![AI Agents សម្រាប់អ្នកចាប់ផ្តើម](https://img.shields.io/badge/AI%20Agents%20for%20Beginners-00C49A?style=for-the-badge&labelColor=E5E7EB&color=00C49A)](https://github.com/microsoft/ai-agents-for-beginners?WT.mc_id=academic-105485-koreyst)

---
 
### ស៊េរី Generative AI
[![Generative AI សម្រាប់អ្នកចាប់ផ្តើម](https://img.shields.io/badge/Generative%20AI%20for%20Beginners-8B5CF6?style=for-the-badge&labelColor=E5E7EB&color=8B5CF6)](https://github.com/microsoft/generative-ai-for-beginners?WT.mc_id=academic-105485-koreyst)
[![Generative AI (.NET)](https://img.shields.io/badge/Generative%20AI%20(.NET)-9333EA?style=for-the-badge&labelColor=E5E7EB&color=9333EA)](https://github.com/microsoft/Generative-AI-for-beginners-dotnet?WT.mc_id=academic-105485-koreyst)
[![Generative AI (Java)](https://img.shields.io/badge/Generative%20AI%20(Java)-C084FC?style=for-the-badge&labelColor=E5E7EB&color=C084FC)](https://github.com/microsoft/generative-ai-for-beginners-java?WT.mc_id=academic-105485-koreyst)
[![Generative AI (JavaScript)](https://img.shields.io/badge/Generative%20AI%20(JavaScript)-E879F9?style=for-the-badge&labelColor=E5E7EB&color=E879F9)](https://github.com/microsoft/generative-ai-with-javascript?WT.mc_id=academic-105485-koreyst)

---
 
### ការសិក្សាជាគោល
[![ML សម្រាប់អ្នកចាប់ផ្តើម](https://img.shields.io/badge/ML%20for%20Beginners-22C55E?style=for-the-badge&labelColor=E5E7EB&color=22C55E)](https://aka.ms/ml-beginners?WT.mc_id=academic-105485-koreyst)
[![វិទ្យាសាស្ត្រ​ទិន្នន័យ​សម្រាប់អ្នកចាប់ផ្តើម](https://img.shields.io/badge/Data%20Science%20for%20Beginners-84CC16?style=for-the-badge&labelColor=E5E7EB&color=84CC16)](https://aka.ms/datascience-beginners?WT.mc_id=academic-105485-koreyst)
[![AI សម្រាប់អ្នកចាប់ផ្តើម](https://img.shields.io/badge/AI%20for%20Beginners-A3E635?style=for-the-badge&labelColor=E5E7EB&color=A3E635)](https://aka.ms/ai-beginners?WT.mc_id=academic-105485-koreyst)
[![សន្តិសុខកុំព្យូទ័រសម្រាប់អ្នកចាប់ផ្តើម](https://img.shields.io/badge/Cybersecurity%20for%20Beginners-F97316?style=for-the-badge&labelColor=E5E7EB&color=F97316)](https://github.com/microsoft/Security-101?WT.mc_id=academic-96948-sayoung)

[![វិប ដេវ សម្រាប់ភាគីដំបូង](https://img.shields.io/badge/Web%20Dev%20for%20Beginners-EC4899?style=for-the-badge&labelColor=E5E7EB&color=EC4899)](https://aka.ms/webdev-beginners?WT.mc_id=academic-105485-koreyst)
[![IoT សម្រាប់ភាគីដំបូង](https://img.shields.io/badge/IoT%20for%20Beginners-14B8A6?style=for-the-badge&labelColor=E5E7EB&color=14B8A6)](https://aka.ms/iot-beginners?WT.mc_id=academic-105485-koreyst)
[![ការអភិវឌ្ឍ XR សម្រាប់ភាគីដំបូង](https://img.shields.io/badge/XR%20Development%20for%20Beginners-38BDF8?style=for-the-badge&labelColor=E5E7EB&color=38BDF8)](https://github.com/microsoft/xr-development-for-beginners?WT.mc_id=academic-105485-koreyst)

---
 
### ស៊េរី Copilot
[![Copilot សម្រាប់ការសរសេរកម្មវិធីរួមគ្នាជាមួយ AI](https://img.shields.io/badge/Copilot%20for%20AI%20Paired%20Programming-FACC15?style=for-the-badge&labelColor=E5E7EB&color=FACC15)](https://aka.ms/GitHubCopilotAI?WT.mc_id=academic-105485-koreyst)
[![Copilot សម្រាប់ C#/.NET](https://img.shields.io/badge/Copilot%20for%20C%23/.NET-FBBF24?style=for-the-badge&labelColor=E5E7EB&color=FBBF24)](https://github.com/microsoft/mastering-github-copilot-for-dotnet-csharp-developers?WT.mc_id=academic-105485-koreyst)
[![ការផ្សងព្រេង Copilot](https://img.shields.io/badge/Copilot%20Adventure-FDE68A?style=for-the-badge&labelColor=E5E7EB&color=FDE68A)](https://github.com/microsoft/CopilotAdventures?WT.mc_id=academic-105485-koreyst)
<!-- CO-OP TRANSLATOR OTHER COURSES END -->

## របៀបទទួលបានជំនួយ

បើអ្នកពិបាកឬមានសំណួរអំពីការបង្កើតកម្មវិធី AI។ ចូលរួមជាមួយអ្នករៀននិងអ្នកអភិវឌ្ឍដែលមានបទពិសោធន៍ក្នុងការពិភាក្សាអំពី MCP។ នេះគឺជាសហគមន៍គាំទ្រដែលសំណួរត្រូវបានទទួលយក និងចែករំលែកចំណេះដឹងដោយសេរី។

[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

ប្រសិនបើអ្នកមានមតិយោបល់អំពីផលិតផល ឬកំហុសពេលកំពុងបង្កើត សូមទៅកាន់៖

[![Microsoft Foundry Developer Forum](https://img.shields.io/badge/GitHub-Microsoft_Foundry_Developer_Forum-blue?style=for-the-badge&logo=github&color=000000&logoColor=fff)](https://aka.ms/foundry/forum)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ការបដិសេធ**:
ឯកសារនេះត្រូវបានបម្លែងភាសា ដោយប្រើសេវាបម្លែងភាសា AI [Co-op Translator](https://github.com/Azure/co-op-translator)។ ទោះយើងខ្ញុំមានក្តីប្រាថ្នាឱ្យបានច្បាស់លាស់ តែសូមយល់ដឹងថាការបម្លែងដោយស្វ័យប្រវត្តិក៏អាចមានកំហុសឬភាពមិនត្រឹមត្រូវ។ ឯកសារដើមជាភាសាទីតាំងគួរត្រូវបានគេប្រើជាប្រភពច្បាស់លាស់។ សម្រាប់ព័ត៌មានសំខាន់ៗ សូមណែនាំឱ្យប្រើប្រាស់ការប្រែដោយមនុស្សជំនាញ។ យើងខ្ញុំមិនទទួលខុសត្រូវចំពោះការយល់ច្រឡំ ឬការបកស្រាយខុសបន្ទាប់ពីការប្រើប្រាស់ការបម្លែងនេះនោះទេ។
<!-- CO-OP TRANSLATOR DISCLAIMER END -->