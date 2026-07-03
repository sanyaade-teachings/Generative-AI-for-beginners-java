# নবশিকিদের জন্য জেনারেটিভ এআই - জাভা সংস্করণ
[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

![Generative AI for Beginners - Java Edition](../../translated_images/bn/beg-genai-series.8b48be9951cc574c.webp)

**সময় ব্যয়**: পুরো কর্মশালা অনলাইনে সম্পন্ন করা যেতে পারে কোনও স্থানীয় সেটআপ ছাড়াই। পরিবেশ সেটআপ ২ মিনিট নিবে, আর নমুনাগুলো অন্বেষণের জন্য অন্বেষণের গভীরতার উপর নির্ভর করে ১-৩ ঘণ্টা লাগতে পারে।

> **দ্রুত শুরু**

১. এই রিপোজিটরিটি আপনার গিটহাব অ্যাকাউন্টে ফর্ক করুন  
২. **Code** → **Codespaces** ট্যাবে ক্লিক করুন → **...** → **New with options...**  
৩. ডিফল্ট ব্যবহার করুন – এটি এই কোর্সের জন্য তৈরি ডেভেলপমেন্ট কন্টেইনার নির্বাচন করবে  
৪. **Create codespace** ক্লিক করুন  
৫. পরিবেশ প্রস্তুতির জন্য ~২ মিনিট অপেক্ষা করুন  
৬. সরাসরি [অধ্যায় ২: Azure AI Foundry প্রোভিশন করুন](./02-SetupDevEnvironment/README.md#step-2-provision-azure-ai-foundry) এ_jump_ করুন

## বহুভাষী সমর্থন

### গিটহাব অ্যাকশনের মাধ্যমে সমর্থিত (স্বয়ংক্রিয় ও সর্বদা আপডেটেড)

<!-- CO-OP TRANSLATOR LANGUAGES TABLE START -->
[Arabic](../ar/README.md) | [Bengali](./README.md) | [Bulgarian](../bg/README.md) | [Burmese (Myanmar)](../my/README.md) | [Chinese (Simplified)](../zh-CN/README.md) | [Chinese (Traditional, Hong Kong)](../zh-HK/README.md) | [Chinese (Traditional, Macau)](../zh-MO/README.md) | [Chinese (Traditional, Taiwan)](../zh-TW/README.md) | [Croatian](../hr/README.md) | [Czech](../cs/README.md) | [Danish](../da/README.md) | [Dutch](../nl/README.md) | [Estonian](../et/README.md) | [Finnish](../fi/README.md) | [French](../fr/README.md) | [German](../de/README.md) | [Greek](../el/README.md) | [Hebrew](../he/README.md) | [Hindi](../hi/README.md) | [Hungarian](../hu/README.md) | [Indonesian](../id/README.md) | [Italian](../it/README.md) | [Japanese](../ja/README.md) | [Kannada](../kn/README.md) | [Khmer](../km/README.md) | [Korean](../ko/README.md) | [Lithuanian](../lt/README.md) | [Malay](../ms/README.md) | [Malayalam](../ml/README.md) | [Marathi](../mr/README.md) | [Nepali](../ne/README.md) | [Nigerian Pidgin](../pcm/README.md) | [Norwegian](../no/README.md) | [Persian (Farsi)](../fa/README.md) | [Polish](../pl/README.md) | [Portuguese (Brazil)](../pt-BR/README.md) | [Portuguese (Portugal)](../pt-PT/README.md) | [Punjabi (Gurmukhi)](../pa/README.md) | [Romanian](../ro/README.md) | [Russian](../ru/README.md) | [Serbian (Cyrillic)](../sr/README.md) | [Slovak](../sk/README.md) | [Slovenian](../sl/README.md) | [Spanish](../es/README.md) | [Swahili](../sw/README.md) | [Swedish](../sv/README.md) | [Tagalog (Filipino)](../tl/README.md) | [Tamil](../ta/README.md) | [Telugu](../te/README.md) | [Thai](../th/README.md) | [Turkish](../tr/README.md) | [Ukrainian](../uk/README.md) | [Urdu](../ur/README.md) | [Vietnamese](../vi/README.md)

> **লোকাল ক্লোন পছন্দ করেন?**
>
> এই রিপোজিটরিতে ৫০+ ভাষার অনুবাদ রয়েছে যার ফলে ডাউনলোড সাইজ যথেষ্ট বৃদ্ধি পায়। অনুবাদ ছাড়া ক্লোন করতে sparse checkout ব্যবহার করুন:
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
> এটি আপনাকে দ্রুত ডাউনলোডের মাধ্যমে কোর্স সম্পূর্ণ করার জন্য প্রয়োজনীয় সবকিছু প্রদান করবে।
<!-- CO-OP TRANSLATOR LANGUAGES TABLE END -->

## কোর্স কাঠামো ও শেখার পথ

### **অধ্যায় ১: জেনারেটিভ এআই পরিচিতি**
- **মূল ধারণা**: বৃহৎ ভাষা মডেল, টোকেন, এমবেডিং, এবং এআই সক্ষমতা বোঝা  
- **জাভা এআই ইকোসিস্টেম**: Spring AI এবং OpenAI SDK-এর ওভারভিউ  
- **মডেল কনটেক্সট প্রোটোকল**: MCP পরিচিতি ও এআই এজেন্ট কমিউনিকেশনে এর ভূমিকা  
- **প্রায়োগিক অ্যাপ্লিকেশন**: চ্যাটবট এবং কনটেন্ট জেনারেশনের বাস্তব উদাহরণ  
- **[→ অধ্যায় ১ শুরু করুন](./01-IntroToGenAI/README.md)**

### **অধ্যায় ২: ডেভেলপমেন্ট এনভায়রনমেন্ট সেটআপ**
- **Azure AI Foundry**: Bicep এবং Azure Developer CLI (azd) দিয়ে কোড হিসাবে মডেল ডিপ্লয়মেন্ট প্রোভিশন करना  
- **Spring Boot + Spring AI**: এন্টারপ্রাইজ AI অ্যাপ্লিকেশন ডেভেলপমেন্টের সেরা অভ্যাস  
- **কীলেস অথেনটিকেশন**: Microsoft Entra ID-র মাধ্যমে নিরাপদ সংযোগ — কোনও API কী ব্যবস্থাপনা নেই  
- **ডেভেলপমেন্ট সরঞ্জাম**: ডকার কন্টেইনার, VS Code, এবং GitHub Codespaces কনফিগারেশন  
- **[→ অধ্যায় ২ শুরু করুন](./02-SetupDevEnvironment/README.md)**

### **অধ্যায় ৩: মূল জেনারেটিভ এআই কৌশলসমূহ**
- **প্রম্পট ইঞ্জিনিয়ারিং**: AI মডেলের সেরা প্রতিক্রিয়া পাওয়ার কৌশল  
- **এমবেডিং এবং ভেক্টর অপারেশন**: সেমেন্টিক সার্চ এবং সাদৃশ্যমূলক মিল বাস্তবায়ন  
- **রিট্রিভাল-অগমেন্টেড জেনারেশন (RAG)**: নিজের ডাটা সোর্সের সঙ্গে AI একত্রিত করা  
- **ফাংশন কলিং**: কাস্টম টুল এবং প্লাগইনের মাধ্যমে AI ক্ষমতা সম্প্রসারণ  
- **[→ অধ্যায় ৩ শুরু করুন](./03-CoreGenerativeAITechniques/README.md)**

### **অধ্যায় ৪: ব্যবহারিক অ্যাপ্লিকেশন ও প্রকল্পসমূহ**
- **পেট স্টোরি জেনারেটর** (`petstory/`): Azure AI Foundry ব্যবহার করে সৃজনশীল কনটেন্ট জেনারেশন  
- **Foundry Local ডেমো** (`foundrylocal/`): OpenAI Java SDK-র সঙ্গে স্থানীয় AI মডেল ইন্টিগ্রেশন  
- **MCP ক্যালকুলেটর সার্ভিস** (`calculator/`): Spring AI দিয়ে মৌলিক মডেল কনটেক্সট প্রোটোকল বাস্তবায়ন  
- **[→ অধ্যায় ৪ শুরু করুন](./04-PracticalSamples/README.md)**

### **অধ্যায় ৫: দায়িত্বশীল AI ডেভেলপমেন্ট**
- **Azure AI Foundry কনটেন্ট সেফটি**: অন্তর্নির্মিত কনটেন্ট ফিল্টারিং ও সুরক্ষা ব্যবস্থার (হার্ড ব্লক এবং সফট রিফিউজাল) টেস্ট  
- **দায়িত্বশীল AI ডেমো**: আধুনিক AI সেফটি সিস্টেম কিভাবে কাজ করে তা দেখানো একটি হ্যান্ডস-অন উদাহরণ  
- **সেরা চর্চা**: নৈতিক AI ডেভেলপমেন্ট এবং ডিপ্লয়মেন্টের গুরুত্বপূর্ণ নির্দেশনাবলী  
- **[→ অধ্যায় ৫ শুরু করুন](./05-ResponsibleGenAI/README.md)**

## অতিরিক্ত সম্পদ

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

### কোপাইলট সিরিজ
[![Copilot for AI Paired Programming](https://img.shields.io/badge/Copilot%20for%20AI%20Paired%20Programming-FACC15?style=for-the-badge&labelColor=E5E7EB&color=FACC15)](https://aka.ms/GitHubCopilotAI?WT.mc_id=academic-105485-koreyst)
[![Copilot for C#/.NET](https://img.shields.io/badge/Copilot%20for%20C%23/.NET-FBBF24?style=for-the-badge&labelColor=E5E7EB&color=FBBF24)](https://github.com/microsoft/mastering-github-copilot-for-dotnet-csharp-developers?WT.mc_id=academic-105485-koreyst)
[![Copilot Adventure](https://img.shields.io/badge/Copilot%20Adventure-FDE68A?style=for-the-badge&labelColor=E5E7EB&color=FDE68A)](https://github.com/microsoft/CopilotAdventures?WT.mc_id=academic-105485-koreyst)
<!-- CO-OP TRANSLATOR OTHER COURSES END -->

## সাহায্য পাবার উপায়

আপনি যদি আটকে যান বা AI অ্যাপ তৈরির ব্যাপারে কোনও প্রশ্ন থাকে, তবে MCP সম্পর্কে আলোচনা করতে সহপাঠী শিক্ষার্থী এবং অভিজ্ঞ ডেভেলপারদের সঙ্গে যোগ দিন। এটি একটি সহায়ক কমিউনিটি যেখানে প্রশ্ন করা স্বাগত এবং জ্ঞান বিনা দ্বিধায় শেয়ার করা হয়।

[![Microsoft Foundry Discord](https://dcbadge.limes.pink/api/server/nTYy5BXMWG)](https://discord.gg/nTYy5BXMWG)

আপনি যদি পণ্য প্রতিক্রিয়া বা ত্রুটি সম্পর্কে জানান দিতে চান, ভ্রমণ করুন:

[![Microsoft Foundry Developer Forum](https://img.shields.io/badge/GitHub-Microsoft_Foundry_Developer_Forum-blue?style=for-the-badge&logo=github&color=000000&logoColor=fff)](https://aka.ms/foundry/forum)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**অস্বীকৃতি**:
এই নথিটি AI অনুবাদ পরিষেবা [Co-op Translator](https://github.com/Azure/co-op-translator) ব্যবহার করে অনূদিত হয়েছে। যদিও আমরা শুদ্ধতার জন্য চেষ্টা করি, অনুগ্রহ করে মনে রাখবেন যে স্বয়ংক্রিয় অনুবাদে ত্রুটি বা অসঙ্গতি থাকতে পারে। মূল নথিটি তার স্বভাষায় কর্তৃত্বপূর্ণ উৎস হিসেবে বিবেচিত হওয়া উচিত। গুরুত্বপূর্ণ তথ্যের জন্য পেশাদার মানব অনুবাদ সুপারিশ করা হয়। এই অনুবাদের ব্যবহারে প্রয়োজনীয় ভুল বোঝাবুঝি বা ভুল ব্যাখ্যার জন্য আমরা দায়বদ্ধ নই।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->