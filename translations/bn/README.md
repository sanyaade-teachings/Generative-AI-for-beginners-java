# নবীনদের জন্য জেনারেটিভ AI - জাভা সংস্করণ
[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

![নবীনদের জন্য জেনারেটিভ AI - জাভা সংস্করণ](../../translated_images/bn/beg-genai-series.8b48be9951cc574c.webp)

**সময় বৃদ্ধি**: পুরো ওয়ার্কশপটি স্থানীয় সেটআপ ছাড়াই অনলাইনে সম্পন্ন করা যেতে পারে। পরিবেশ সেটআপ ২ মিনিট সময় নেয়, এবং উদাহরণগুলি অনুসন্ধান করতে ১-৩ ঘণ্টা সময় লাগতে পারে, অনুসন্ধানের গভীরতার উপর নির্ভর করে।

> **দ্রুত শুরু**

১. এই রিপোজিটরিটি আপনার GitHub অ্যাকাউন্টে ফর্ক করুন  
২. ক্লিক করুন **Code** → **Codespaces** ট্যাব → **...** → **New with options...**  
৩. ডিফল্ট ব্যবহার করুন – এটি কোর্সের জন্য তৈরি ডেভেলপমেন্ট কন্টেইনার নির্বাচন করবে  
৪. ক্লিক করুন **Create codespace**  
৫. প্রায় ২ মিনিট অপেক্ষা করুন পরিবেশ প্রস্তুত হতে  
৬. সরাসরি যান [প্রথম উদাহরণে](./02-SetupDevEnvironment/README.md#step-2-create-a-github-personal-access-token)

## বহুভাষিক সমর্থন

### GitHub অ্যাকশনের মাধ্যমে সমর্থিত (স্বয়ংক্রিয় ও সর্বদা আপডেটেড)

<!-- CO-OP TRANSLATOR LANGUAGES TABLE START -->
[Arabic](../ar/README.md) | [Bengali](./README.md) | [Bulgarian](../bg/README.md) | [Burmese (Myanmar)](../my/README.md) | [Chinese (Simplified)](../zh-CN/README.md) | [Chinese (Traditional, Hong Kong)](../zh-HK/README.md) | [Chinese (Traditional, Macau)](../zh-MO/README.md) | [Chinese (Traditional, Taiwan)](../zh-TW/README.md) | [Croatian](../hr/README.md) | [Czech](../cs/README.md) | [Danish](../da/README.md) | [Dutch](../nl/README.md) | [Estonian](../et/README.md) | [Finnish](../fi/README.md) | [French](../fr/README.md) | [German](../de/README.md) | [Greek](../el/README.md) | [Hebrew](../he/README.md) | [Hindi](../hi/README.md) | [Hungarian](../hu/README.md) | [Indonesian](../id/README.md) | [Italian](../it/README.md) | [Japanese](../ja/README.md) | [Kannada](../kn/README.md) | [Khmer](../km/README.md) | [Korean](../ko/README.md) | [Lithuanian](../lt/README.md) | [Malay](../ms/README.md) | [Malayalam](../ml/README.md) | [Marathi](../mr/README.md) | [Nepali](../ne/README.md) | [Nigerian Pidgin](../pcm/README.md) | [Norwegian](../no/README.md) | [Persian (Farsi)](../fa/README.md) | [Polish](../pl/README.md) | [Portuguese (Brazil)](../pt-BR/README.md) | [Portuguese (Portugal)](../pt-PT/README.md) | [Punjabi (Gurmukhi)](../pa/README.md) | [Romanian](../ro/README.md) | [Russian](../ru/README.md) | [Serbian (Cyrillic)](../sr/README.md) | [Slovak](../sk/README.md) | [Slovenian](../sl/README.md) | [Spanish](../es/README.md) | [Swahili](../sw/README.md) | [Swedish](../sv/README.md) | [Tagalog (Filipino)](../tl/README.md) | [Tamil](../ta/README.md) | [Telugu](../te/README.md) | [Thai](../th/README.md) | [Turkish](../tr/README.md) | [Ukrainian](../uk/README.md) | [Urdu](../ur/README.md) | [Vietnamese](../vi/README.md)

> **স্থানীয়ভাবে ক্লোন করতে চান?**
>
> এই রিপোজিটরিটিতে ৫০+ ভাষায় অনুবাদ রয়েছে যা ডাউনলোড সাইজ যথেষ্ট বৃদ্ধি করে। অনুবাদ ছাড়া ক্লোন করতে স্পার্স চেকআউট ব্যবহার করুন:
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
> এটি আপনাকে দ্রুত ডাউনলোড সহ কোর্স সম্পন্ন করার জন্য প্রয়োজনীয় সবকিছু দেবে।
<!-- CO-OP TRANSLATOR LANGUAGES TABLE END -->

## কোর্স কাঠামো ও শেখার পথ

### **অধ্যায় ১: জেনারেটিভ AI এর পরিচিতি**
- **মূল ধারণা**: বড় ভাষা মডেল, টোকেন, এম্বেডিংস, এবং AI ক্ষমতার বোঝাপড়া  
- **জাভা AI ইকোসিস্টেম**: Spring AI এবং OpenAI SDKs এর ওভারভিউ  
- **মডেল কনটেক্সট প্রোটোকল**: MCP পরিচিতি এবং AI এজেন্ট যোগাযোগে এর ভূমিকা  
- **ব্যবহারিক প্রয়োগ**: চ্যাটবট ও বিষয়বস্তু তৈরি সহ বাস্তব জীবন পরিস্থিতি  
- **[→ অধ্যায় ১ শুরু করুন](./01-IntroToGenAI/README.md)**

### **অধ্যায় ২: উন্নয়ন পরিবেশ সেটআপ**
- **Azure AI Foundry**: Bicep ও Azure Developer CLI (azd) দ্বারা কোড হিসেবে মডেল ডিপ্লয়মেন্ট  
- **Spring Boot + Spring AI**: এন্টারপ্রাইজ AI অ্যাপ্লিকেশন উন্নয়নের সেরা অনুশীলন  
- **কীলেস অটেনটিকেশন**: Microsoft Entra ID দিয়ে নিরাপদ সংযোগ — API কী ম্যানেজ করতে হবে না  
- **উন্নয়ন সরঞ্জাম**: Docker কন্টেইনার, VS Code, ও GitHub Codespaces কনফিগারেশন  
- **[→ অধ্যায় ২ শুরু করুন](./02-SetupDevEnvironment/README.md)**

### **অধ্যায় ৩: মূল জেনারেটিভ AI প্রযুক্তি**
- **প্রম্পট ইঞ্জিনিয়ারিং**: AI মডেলের সর্বোত্তম প্রতিক্রিয়া পেতে কৌশল  
- **এম্বেডিংস ও ভেক্টর অপারেশনস**: সেমান্তিক সার্চ ও সাদৃশ্য মিলানো বাস্তবায়ন  
- **রিট্রিভাল-অগমেন্টেড জেনারেশন (RAG)**: AI ও নিজস্ব ডেটা সোর্স একত্রিত করা  
- **ফাংশন কলিং**: AI ক্ষমতা সম্প্রসারণ স্বনির্ধারিত টুল ও প্লাগইন দিয়ে  
- **[→ অধ্যায় ৩ শুরু করুন](./03-CoreGenerativeAITechniques/README.md)**

### **অধ্যায় ৪: ব্যবহারিক প্রয়োগ ও প্রকল্প**
- **পেট স্টোরি জেনারেটর** (`petstory/`): Azure AI Foundry দিয়ে সৃষ্টিশীল বিষয়বস্তু তৈরি  
- **Foundry লোকাল ডেমো** (`foundrylocal/`): OpenAI জাভা SDK সহ স্থানীয় AI মডেল ইন্টিগ্রেশন  
- **MCP ক্যালকুলেটর সার্ভিস** (`calculator/`): Spring AI এর সাথে মৌলিক Model Context Protocol বাস্তবায়ন  
- **[→ অধ্যায় ৪ শুরু করুন](./04-PracticalSamples/README.md)**

### **অধ্যায় ৫: দায়িত্বশীল AI উন্নয়ন**
- **Azure AI Foundry কন্টেন্ট সেফটি**: বিল্ট-ইন বিষয়বস্তু ফিল্টারিং ও সেফটি মেকানিজম পরীক্ষা (হার্ড ব্লক ও সফট প্রত্যাখ্যান)  
- **দায়িত্বশীল AI ডেমো**: আধুনিক AI সেফটি সিস্টেম কিভাবে কাজ করে তার হাতে-কলমে উদাহরণ  
- **সেরা অনুশীলন**: নৈতিক AI উন্নয়ন ও ডিপ্লয়মেন্টের অপরিহার্য নির্দেশিকা  
- **[→ অধ্যায় ৫ শুরু করুন](./05-ResponsibleGenAI/README.md)**

## অতিরিক্ত সম্পদ

<!-- CO-OP TRANSLATOR OTHER COURSES START -->
### ল্যাংচেইন  
[![LangChain4j নবীনদের জন্য](https://img.shields.io/badge/LangChain4j%20for%20Beginners-22C55E?style=for-the-badge&&labelColor=E5E7EB&color=0553D6)](https://aka.ms/langchain4j-for-beginners)  
[![LangChain.js নবীনদের জন্য](https://img.shields.io/badge/LangChain.js%20for%20Beginners-22C55E?style=for-the-badge&labelColor=E5E7EB&color=0553D6)](https://aka.ms/langchainjs-for-beginners?WT.mc_id=m365-94501-dwahlin)  
[![LangChain নবীনদের জন্য](https://img.shields.io/badge/LangChain%20for%20Beginners-22C55E?style=for-the-badge&labelColor=E5E7EB&color=0553D6)](https://github.com/microsoft/langchain-for-beginners?WT.mc_id=m365-94501-dwahlin)  
---

### Azure / Edge / MCP / এজেন্টস  
[![AZD নবীনদের জন্য](https://img.shields.io/badge/AZD%20for%20Beginners-0078D4?style=for-the-badge&labelColor=E5E7EB&color=0078D4)](https://github.com/microsoft/AZD-for-beginners?WT.mc_id=academic-105485-koreyst)  
[![Edge AI নবীনদের জন্য](https://img.shields.io/badge/Edge%20AI%20for%20Beginners-00B8E4?style=for-the-badge&labelColor=E5E7EB&color=00B8E4)](https://github.com/microsoft/edgeai-for-beginners?WT.mc_id=academic-105485-koreyst)  
[![MCP নবীনদের জন্য](https://img.shields.io/badge/MCP%20for%20Beginners-009688?style=for-the-badge&labelColor=E5E7EB&color=009688)](https://github.com/microsoft/mcp-for-beginners?WT.mc_id=academic-105485-koreyst)  
[![AI Agents নবীনদের জন্য](https://img.shields.io/badge/AI%20Agents%20for%20Beginners-00C49A?style=for-the-badge&labelColor=E5E7EB&color=00C49A)](https://github.com/microsoft/ai-agents-for-beginners?WT.mc_id=academic-105485-koreyst)  

---

### জেনারেটিভ AI সিরিজ  
[![নবীনদের জন্য জেনারেটিভ AI](https://img.shields.io/badge/Generative%20AI%20for%20Beginners-8B5CF6?style=for-the-badge&labelColor=E5E7EB&color=8B5CF6)](https://github.com/microsoft/generative-ai-for-beginners?WT.mc_id=academic-105485-koreyst)  
[![জেনারেটিভ AI (.NET)](https://img.shields.io/badge/Generative%20AI%20(.NET)-9333EA?style=for-the-badge&labelColor=E5E7EB&color=9333EA)](https://github.com/microsoft/Generative-AI-for-beginners-dotnet?WT.mc_id=academic-105485-koreyst)  
[![জেনারেটিভ AI (জাভা)](https://img.shields.io/badge/Generative%20AI%20(Java)-C084FC?style=for-the-badge&labelColor=E5E7EB&color=C084FC)](https://github.com/microsoft/generative-ai-for-beginners-java?WT.mc_id=academic-105485-koreyst)  
[![জেনারেটিভ AI (জাভাস্ক্রিপ্ট)](https://img.shields.io/badge/Generative%20AI%20(JavaScript)-E879F9?style=for-the-badge&labelColor=E5E7EB&color=E879F9)](https://github.com/microsoft/generative-ai-with-javascript?WT.mc_id=academic-105485-koreyst)  

---

### মূল শেখা  
[![নবীনদের জন্য এমএল](https://img.shields.io/badge/ML%20for%20Beginners-22C55E?style=for-the-badge&labelColor=E5E7EB&color=22C55E)](https://aka.ms/ml-beginners?WT.mc_id=academic-105485-koreyst)  
[![নবীনদের জন্য ডেটা সায়েন্স](https://img.shields.io/badge/Data%20Science%20for%20Beginners-84CC16?style=for-the-badge&labelColor=E5E7EB&color=84CC16)](https://aka.ms/datascience-beginners?WT.mc_id=academic-105485-koreyst)  
[![নবীনদের জন্য AI](https://img.shields.io/badge/AI%20for%20Beginners-A3E635?style=for-the-badge&labelColor=E5E7EB&color=A3E635)](https://aka.ms/ai-beginners?WT.mc_id=academic-105485-koreyst)  
[![নবীনদের জন্য সাইবারসিকিউরিটি](https://img.shields.io/badge/Cybersecurity%20for%20Beginners-F97316?style=for-the-badge&labelColor=E5E7EB&color=F97316)](https://github.com/microsoft/Security-101?WT.mc_id=academic-96948-sayoung)  

[![Web Dev for Beginners](https://img.shields.io/badge/Web%20Dev%20for%20Beginners-EC4899?style=for-the-badge&labelColor=E5E7EB&color=EC4899)](https://aka.ms/webdev-beginners?WT.mc_id=academic-105485-koreyst)
[![IoT for Beginners](https://img.shields.io/badge/IoT%20for%20Beginners-14B8A6?style=for-the-badge&labelColor=E5E7EB&color=14B8A6)](https://aka.ms/iot-beginners?WT.mc_id=academic-105485-koreyst)
[![XR Development for Beginners](https://img.shields.io/badge/XR%20Development%20for%20Beginners-38BDF8?style=for-the-badge&labelColor=E5E7EB&color=38BDF8)](https://github.com/microsoft/xr-development-for-beginners?WT.mc_id=academic-105485-koreyst)

---
 
### কপাইলট সিরিজ
[![Copilot for AI Paired Programming](https://img.shields.io/badge/Copilot%20for%20AI%20Paired%20Programming-FACC15?style=for-the-badge&labelColor=E5E7EB&color=FACC15)](https://aka.ms/GitHubCopilotAI?WT.mc_id=academic-105485-koreyst)
[![Copilot for C#/.NET](https://img.shields.io/badge/Copilot%20for%20C%23/.NET-FBBF24?style=for-the-badge&labelColor=E5E7EB&color=FBBF24)](https://github.com/microsoft/mastering-github-copilot-for-dotnet-csharp-developers?WT.mc_id=academic-105485-koreyst)
[![Copilot Adventure](https://img.shields.io/badge/Copilot%20Adventure-FDE68A?style=for-the-badge&labelColor=E5E7EB&color=FDE68A)](https://github.com/microsoft/CopilotAdventures?WT.mc_id=academic-105485-koreyst)
<!-- CO-OP TRANSLATOR OTHER COURSES END -->

## সাহায্য পাওয়া

যদি আপনি আটকে যান বা AI অ্যাপ তৈরি করার বিষয়ে কোনও প্রশ্ন থাকে। MCP নিয়ে আলোচনা করতে সহকর্মী শিক্ষার্থী এবং অভিজ্ঞ ডেভেলপারদের সাথে যোগ দিন। এটি একটি সহায়ক সম্প্রদায় যেখানে প্রশ্ন স্বাগত এবং জ্ঞান বিনামূল্যে ভাগ করা হয়।

[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

যদি আপনার পণ্য প্রতিক্রিয়া বা ত্রুটি থাকে তবে নির্মাণের সময় পরিদর্শন করুন:

[![Microsoft Foundry Developer Forum](https://img.shields.io/badge/GitHub-Microsoft_Foundry_Developer_Forum-blue?style=for-the-badge&logo=github&color=000000&logoColor=fff)](https://aka.ms/foundry/forum)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**অস্বীকৃতি**:
এই নথিটি AI অনুবাদ পরিষেবা [Co-op Translator](https://github.com/Azure/co-op-translator) ব্যবহার করে অনূদিত হয়েছে। যদিও আমরা শুদ্ধতার জন্য চেষ্টা করি, অনুগ্রহ করে মনে রাখবেন যে স্বয়ংক্রিয় অনুবাদে ত্রুটি বা অসঙ্গতি থাকতে পারে। মূল নথিটি তার স্বভাষায় কর্তৃত্বপূর্ণ উৎস হিসেবে বিবেচিত হওয়া উচিত। গুরুত্বপূর্ণ তথ্যের জন্য পেশাদার মানব অনুবাদ সুপারিশ করা হয়। এই অনুবাদের ব্যবহারে প্রয়োজনীয় ভুল বোঝাবুঝি বা ভুল ব্যাখ্যার জন্য আমরা দায়বদ্ধ নই।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->