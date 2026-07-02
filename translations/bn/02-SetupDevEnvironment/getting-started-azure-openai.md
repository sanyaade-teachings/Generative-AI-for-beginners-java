# Azure AI Foundry এর জন্য ডেভেলপমেন্ট পরিবেশ সেট আপ করা

> এই গাইডটি এই কোর্সের জাভা AI অ্যাপগুলির জন্য **Azure AI Foundry** মডেলগুলি সেট আপ করে, **keyless** প্রমাণীকরণ (Microsoft Entra ID) ব্যবহার করে — কোনও API কী পরিচালনা করার প্রয়োজন নেই। টুলিংয়ে নতুন? শুরু করুন [ডেভেলপমেন্ট পরিবেশ গাইড](./README.md) থেকে।

এই গাইডটি এই কোর্সের জাভা AI অ্যাপগুলির জন্য **Azure AI Foundry** মডেলগুলি সেট আপ করে। আপনার দুইটি পথ রয়েছে:

- **পছন্দ A — `azd` + Bicep এর মাধ্যমে Provision (প্রস্তাবিত):** এক কমান্ডে Foundry অ্যাকাউন্ট এবং মডেলগুলি কোড হিসাবে ডিপ্লয় হয়। কোনও পোর্টাল ক্লিক করতে হবে না।
- **পছন্দ B — Azure AI Foundry পোর্টালে ম্যানুয়ালি রিসোর্স তৈরি করুন।**

দুই পথেই **keyless authentication** (Microsoft Entra ID) ব্যবহার করা হয় — কোনও API কী কপি বা ফাঁস করার প্রয়োজন নেই।

## বিষয় সমূহের সূচি

- [কি তৈরি হয়](#কি-তৈরি-হয়)
- [প্রয়োজনীয়তা](#প্রয়োজনীয়তা)
- [পছন্দ A: azd + Bicep দিয়ে Provision (প্রস্তাবিত)](#option-a-provision-with-azd--bicep-recommended)
- [পছন্দ B: ম্যানুয়ালি রিসোর্স তৈরি করা](#পছন্দ-b-ম্যানুয়ালি-রিসোর্স-তৈরি-করা)
- [আপনার পরিবেশ কনফিগার করুন](#আপনার-পরিবেশ-কনফিগার-করুন)
- [আপনার সেটআপ পরীক্ষা করুন](#আপনার-সেটআপ-পরীক্ষা-করুন)
- [পরবর্তী কি?](#পরবর্তী-কি)
- [রিসোর্স](#রিসোর্স)
- [অতিরিক্ত রিসোর্স](#অতিরিক্ত-রিসোর্স)

## কি তৈরি হয়

[`infra/`](../../../02-SetupDevEnvironment/infra) এর Bicep টেমপ্লেটগুলি provision করে:

- একটি **Azure AI Foundry** অ্যাকাউন্ট (`Microsoft.CognitiveServices/accounts`, ধরন `AIServices`) একটি প্রকল্পসহ
- একটি **chat** ডিপ্লয়মেন্ট — `gpt-4o-mini`
- একটি **embedding** ডিপ্লয়মেন্ট — `text-embedding-3-small` (পরে অধ্যায়ে ব্যবহৃত)
- একটি **keyless role assignment** (`Cognitive Services OpenAI User`) যাতে আপনি কী ম্যানেজ না করে `az login` দিয়ে সাইন ইন করতে পারেন

## প্রয়োজনীয়তা

- একটি [Azure সাবস্ক্রিপশন](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) এবং [Maven 3.9+](https://maven.apache.org/download.cgi)

## পছন্দ A: azd + Bicep দিয়ে Provision (প্রস্তাবিত)

`02-SetupDevEnvironment` ফোল্ডার থেকে:

```bash
cd 02-SetupDevEnvironment

# সাইন ইন (উভয় টুল)
azd auth login
az login

# ফাউন্ড্রি অ্যাকাউন্ট + মডেল ডিপ্লয়মেন্ট প্রদান করুন
azd up
```

`azd` আপনাকে একটি **পরিবেশ নাম** (উদাহরণস্বরূপ `genai-java`) এবং একটি **রিজিওন** এর জন্য প্রম্পট দেয়। এমন একটি রিজিওন বেছে নিন যেখানে `gpt-4o-mini` এবং `text-embedding-3-small` উপলব্ধ — যেমন `eastus2` বা `swedencentral`।

Provisioning শেষ হলে, azd:

1. [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep)-এ সংজ্ঞায়িত সবকিছু ডিপ্লয় করে।
2. একটি postprovision হুক চালায় যা আপনার endpoint এবং deployment নামসমূহ সহ [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) ফাইল লেখে (কোনো সিক্রেট নয়)।

> **টিপ:** কোনও সময় পরিবর্তন প্রয়োগ করতে আবার `azd up` রান করুন। সবকিছু মুছে দিতে এবং খরচ বন্ধ করতে `azd down` রান করুন।

উত্পন্ন সেটিংস দেখতে:

```bash
azd env get-values
```

এখন [আপনার সেটআপ পরীক্ষা করুন](#আপনার-সেটআপ-পরীক্ষা-করুন) এ যান।

## পছন্দ B: ম্যানুয়ালি রিসোর্স তৈরি করা

পোর্টাল ব্যবহার করতে চান? নিজে নিজে রিসোর্স তৈরি করুন:

1. [Azure AI Foundry পোর্টালে](https://ai.azure.com/) যান এবং সাইন ইন করুন।
2. **একটি প্রকল্প তৈরি করুন** (এটি একটি AI Foundry রিসোর্সও তৈরি করে)। একটি নাম দিন যেমন `GenAIJava`।
3. আপনার প্রকল্পে, **Models + endpoints** → **Deploy model** → **Deploy base model** খুলুন।
4. **gpt-4o-mini** (ডিপ্লয়মেন্ট নাম `gpt-4o-mini`) ডিপ্লয় করুন। ইচ্ছা করলে embedding উদাহরণের জন্য **text-embedding-3-small** এর জন্যও পুনরাবৃত্তি করুন।
5. **Overview** থেকে **endpoint** কপি করুন (উদাহরণস্বরূপ `https://<resource>.openai.azure.com/`)।
6. নিজেকে keyless access দিন: রিসোর্সে, **Access control (IAM)** → **Add role assignment** → আপনার অ্যাকাউন্টে **Cognitive Services OpenAI User** অ্যাসাইন করুন।

> **এখনও সমস্যা হচ্ছে?** দেখুন [Azure AI Foundry ডকুমেন্টেশন](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects)।

## আপনার পরিবেশ কনফিগার করুন

**যদি আপনি পছন্দ A (`azd up`) ব্যবহার করে থাকেন**, আপনার সেটিংস ফাইল ইতিমধ্যেই লেখা হয়েছে — কনফিগার করার কিছু নেই। [আপনার সেটআপ পরীক্ষা করুন](#আপনার-সেটআপ-পরীক্ষা-করুন) এ যান।

**যদি আপনি পছন্দ B (ম্যানুয়াল) ব্যবহার করে থাকেন**, উদাহরণের `.env` ফাইল নিজে তৈরি করুন:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

`.env` এ এডিট করুন আপনার endpoint সহ (কোনো কী নয় — অথেন্টিকেশন keyless):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **সিকিউরিটি নোট:** সংরক্ষণ করার জন্য কোনও API কী নেই। আপনি Microsoft Entra ID দিয়ে `az login` (লোকালি) বা একটি managed identity (Azure এ) দিয়ে প্রমাণীকরণ করেন। `.env` ফাইলে শুধুমাত্র নন-সিক্রেট সেটিংস থাকে এবং এটি ইতিমধ্যেই `.gitignore` দ্বারা কভার করা হয়।

## আপনার সেটআপ পরীক্ষা করুন

নিশ্চিত করুন আপনি সাইন ইন করেছেন যাতে keyless auth টোকেন পেতে পারে, তারপর উদাহরণ রান করুন:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # যদি আপনি ইতিমধ্যেই সাইন ইন না করে থাকেন
mvn clean spring-boot:run
```

আপনি `gpt-4o-mini` মডেল থেকে একটি প্রতিক্রিয়া দেখতে পাবেন!

> **VS Code ব্যবহারকারী:** রান করতে `F5` চাপুন। অ্যাপটি আপনার `.env` স্বয়ংক্রিয়ভাবে লোড করে।

> **সম্পূর্ণ উদাহরণ:** বিস্তারিত এবং সমস্যা সমাধানের জন্য দেখুন [Azure AI Foundry এর সাথে Basic Chat উদাহরণ](./examples/basic-chat-azure/README.md)।

## পরবর্তী কি?

**সেটআপ সম্পন্ন!** আপনার এখন আছে:
- `gpt-4o-mini` এবং `text-embedding-3-small` সহ Azure AI Foundry ডিপ্লয় করা
- Keyless authentication (Microsoft Entra ID) — কী ম্যানেজ করার প্রয়োজন নেই
- একটি লোকাল `.env` আপনার endpoint এবং deployment নামসহ
- জাভা ডেভেলপমেন্ট পরিবেশ প্রস্তুত

**শুরু করুন** [অধ্যায় ৩: Core Generative AI Techniques](../03-CoreGenerativeAITechniques/README.md) থেকে AI অ্যাপ্লিকেশন তৈরি করতে!

## রিসোর্স

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Microsoft Entra ID সহ Keyless authentication](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry ডকুমেন্টেশন](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI ডকুমেন্টেশন](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## অতিরিক্ত রিসোর্স

- [VS Code ডাউনলোড করুন](https://code.visualstudio.com/Download)
- [Docker Desktop পান](https://www.docker.com/products/docker-desktop)
- [Dev Container কনফিগারেশন](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**অস্বীকৃতি**:
এই নথিটি AI অনুবাদ পরিষেবা [Co-op Translator](https://github.com/Azure/co-op-translator) ব্যবহার করে অনূদিত হয়েছে। যদিও আমরা শুদ্ধতার জন্য চেষ্টা করি, অনুগ্রহ করে মনে রাখবেন যে স্বয়ংক্রিয় অনুবাদে ত্রুটি বা অসঙ্গতি থাকতে পারে। মূল নথিটি তার স্বভাষায় কর্তৃত্বপূর্ণ উৎস হিসেবে বিবেচিত হওয়া উচিত। গুরুত্বপূর্ণ তথ্যের জন্য পেশাদার মানব অনুবাদ সুপারিশ করা হয়। এই অনুবাদের ব্যবহারে প্রয়োজনীয় ভুল বোঝাবুঝি বা ভুল ব্যাখ্যার জন্য আমরা দায়বদ্ধ নই।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->