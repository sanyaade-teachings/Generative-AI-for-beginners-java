# Azure AI Foundry এর সাথে বেসিক চ্যাট - শুরু থেকে শেষ পর্যন্ত উদাহরণ

এই উদাহরণটি একটি সহজ Spring Boot অ্যাপ্লিকেশন যা **Azure AI Foundry** মডেলের সাথে **কী-মুক্ত প্রমাণীকরণের** (Microsoft Entra ID) মাধ্যমে সংযুক্ত হয় এবং আপনার সেটআপ পরীক্ষা করে। এটি Spring AI এর `ChatClient` ব্যবহার করে।

## বিষয়বস্তু সূচী

- [প্রয়োজনীয়তাসমূহ](#প্রয়োজনীয়তাসমূহ)
- [দ্রুত শুরু](#দ্রুত-শুরু)
- [কিভাবে প্রমাণীকরণ কাজ করে](#কিভাবে-প্রমাণীকরণ-কাজ-করে)
- [অ্যাপ্লিকেশন চালানো](#অ্যাপ্লিকেশন-চালানো)
  - [Maven ব্যবহার করে](#maven-ব্যবহার-করে)
  - [VS Code ব্যবহার করে](#vs-code-ব্যবহার-করে)
  - [প্রত্যাশিত আউটপুট](#প্রত্যাশিত-আউটপুট)
- [কনফিগারেশন রেফারেন্স](#কনফিগারেশন-রেফারেন্স)
  - [পরিবেশ পরিবর্তনশীলসমূহ](#পরিবেশ-পরিবর্তনশীলসমূহ)
  - [Spring কনফিগারেশন](#spring-কনফিগারেশন)
- [সমস্যা সমাধান](#সমস্যা-সমাধান)
  - [সাধারণ সমস্যা](#সাধারণ-সমস্যা)
  - [ডিবাগ মোড](#ডিবাগ-মোড)
- [পরবর্তী ধাপ](#পরবর্তী-ধাপ)
- [সম্পদগুলি](#সম্পদগুলি)

## প্রয়োজনীয়তাসমূহ

এই উদাহরণটি চালানোর আগে নিশ্চিত করুন যে আপনি:

- একটি Azure AI Foundry রিসোর্স আছে যার মধ্যে একটি `gpt-4o-mini` ডিপ্লয়মেন্ট — এটি `azd up` দিয়ে বা ম্যানুয়ালি [Azure AI Foundry সেটআপ গাইড](../../getting-started-azure-openai.md) থেকে প্রদান করতে পারেন
- ঐ রিসোর্সে **Cognitive Services OpenAI User** ভূমিকা আছে (Bicep টেমপ্লেটগুলো আপনাকে এটি ধার্য করে)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli) ইন্সটল এবং `az login` দিয়ে সাইন ইন করেছেন
- Java 21+ এবং Maven 3.9+ আছে

> **কোন API কী প্রয়োজন নেই** — প্রমাণীকরণ কী-মুক্ত Microsoft Entra ID এর মাধ্যমে।

## দ্রুত শুরু

```bash
# ১. প্রকল্পে যান
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# ২. টোকেন পেতে সাইন ইন করুন যাতে কীলেস অথ করার সুবিধা পাওয়া যায়
az login

# ৩. এন্ডপয়েন্ট কনফিগার করুন
#    - যদি আপনি `azd up` চালিয়ে থাকেন, .env আপনাকে দেওয়া হয়েছে (এটি ছাড়াই যেতে পারেন)।
#    - অন্যথায় টেমপ্লেটটি কপি করুন এবং AZURE_OPENAI_ENDPOINT সেট করুন:
cp .env.example .env

# ৪. অ্যাপ্লিকেশন চালান
mvn spring-boot:run
```

## কিভাবে প্রমাণীকরণ কাজ করে

এই উদাহরণটি **Microsoft Entra ID** এর মাধ্যমে প্রমাণীকরণ করে — কোন API কী নেই।

যখন শুধুমাত্র `spring.ai.azure.openai.endpoint` সেট থাকে (আর কোন api-key না থাকে), তখন Spring AI Azure OpenAI ক্লায়েন্ট তৈরি করে [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential) ব্যবহার করে। ঐ ক্রেডেনশিয়াল স্বয়ংক্রিয়ভাবে আপনার `az login` সেশনের টোকেন স্থানীয়ভাবে বা Azure তে চালানোর সময় একটি ম্যানেজড আইডেন্টিটি থেকে পায় — তাই একই কোড উভয় জায়গায় কোনো পরিবর্তন ছাড়াই কাজ করে।

## অ্যাপ্লিকেশন চালানো

### Maven ব্যবহার করে

```bash
mvn spring-boot:run
```

### VS Code ব্যবহার করে

1. VS Code-এ প্রজেক্টটি খুলুন
2. `F5` চাপুন অথবা "Run and Debug" প্যানেল ব্যবহার করুন
3. "Spring Boot-BasicChatApplication" কনফিগারেশন নির্বাচন করুন

> **মন্তব্য**: VS Code কনফিগারেশনটি স্বয়ংক্রিয়ভাবে আপনার .env ফাইল লোড করে

### প্রত্যাশিত আউটপুট

```
Starting Basic Chat with Azure OpenAI...
Environment variables loaded successfully
Connecting to Azure OpenAI...
Sending prompt: What is AI in a short sentence? Max 100 words.

AI Response:
================
AI, or Artificial Intelligence, is the simulation of human intelligence in machines programmed to think and learn like humans.
================

Success! Azure OpenAI connection is working correctly.
```

## কনফিগারেশন রেফারেন্স

### পরিবেশ পরিবর্তনশীলসমূহ

| পরিবর্তনশীল | বর্ণনা | প্রয়োজনীয় | উদাহরণ |
|------------|---------|------------|---------|
| `AZURE_OPENAI_ENDPOINT` | Foundry (Azure OpenAI) এন্ডপয়েন্ট URL | হ্যাঁ | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | চ্যাট মডেলের ডিপ্লয়মেন্টের নাম | না | `gpt-4o-mini` (ডিফল্ট) |

> একটি API কী পরিবর্তনশীল **নেই** — প্রমাণীকরণ কী-মুক্ত (Microsoft Entra ID এর মাধ্যমে `az login`)।

### Spring কনফিগারেশন

`application.yml` ফাইলে কনফিগার করা হয়েছে:
- **এন্ডপয়েন্ট**: `${AZURE_OPENAI_ENDPOINT}` - পরিবেশ পরিবর্তনশীল থেকে
- **ডিপ্লয়মেন্ট**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - পরিবেশ পরিবর্তনশীল থেকে এবং ব্যাকআপ সহ
- **প্রমাণীকরণ**: কী-মুক্ত — `api-key` সেট করা হয়নি, তাই Spring AI `DefaultAzureCredential` ব্যবহার করে
- **তাপমাত্রা**: `0.7` - সৃষ্টিশীলতায় নিয়ন্ত্রণ করে (0.0 = নির্ধারিত, 1.0 = সৃষ্টিশীল)
- **সর্বোচ্চ টোকেন সংখ্যা**: `500` - সর্বোচ্চ উত্তর দৈর্ঘ্য

## সমস্যা সমাধান

### সাধারণ সমস্যা

<details>
<summary><strong>ত্রুটি: 401 / "PermissionDenied" / টোকেনের ত্রুটি</strong></summary>

- `az login` চালান — কী-মুক্ত প্রমাণীকরণ সক্রিয় লগইন প্রয়োজন টোকেন পেতে
- নিশ্চিত করুন আপনার অ্যাকাউন্টে ঐ রিসোর্সের উপর **Cognitive Services OpenAI User** ভূমিকা আছে
- আপনি যদি সম্প্রতি ভূমিকা ধার্য করে থাকেন, অপেক্ষা করুন একটি মিনিটে প্রোপাগেট হতে
- নিশ্চিত করুন আপনি সঠিক টেন্যান্ট/সাবস্ক্রিপশনে আছেন (`az account show`)
</details>

<details>
<summary><strong>ত্রুটি: "The endpoint is not valid" / সংযোগ ত্রুটি</strong></summary>

- নিশ্চিত করুন `AZURE_OPENAI_ENDPOINT` পূর্ণ বেস URL (যেমন, `https://your-resource.openai.azure.com/`) 
- ট্রেইলিং স্ল্যাশের সঙ্গতি পরীক্ষা করুন
- নিশ্চিত করুন এন্ডপয়েন্ট আপনার প্রদানকৃত রিসোর্সের সাথে মেলে (`azd env get-values`)
</details>

<details>
<summary><strong>ত্রুটি: "The deployment was not found"</strong></summary>

- নিশ্চিত করুন `AZURE_OPENAI_DEPLOYMENT` Azure-এ একটি ডিপ্লয়মেন্টের নামের সাথে মিলে
- চেক করুন মডেল সফলভাবে ডিপ্লয় এবং সক্রিয় আছে
- ডিফল্ট ডিপ্লয়মেন্টের নাম হলো `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: পরিবেশ পরিবর্তনশীল লোড হচ্ছে না</strong></summary>

- নিশ্চিত করুন আপনার `.env` ফাইল প্রজেক্টের মূল ডিরেক্টরিতে আছে (`pom.xml` এর সাথে সমতল)
- VS Code এর ইন্টিগ্রেটেড টার্মিনালে `mvn spring-boot:run` চালানোর চেষ্টা করুন
- চেক করুন VS Code Java এক্সটেনশন সঠিকভাবে ইন্সটল হয়েছে কিনা
</details>

### ডিবাগ মোড

বিস্তৃত লগিং চালু করতে `application.yml` এ নিম্নলিখিত লাইনগুলো আনকমেন্ট করুন:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## পরবর্তী ধাপ

**সেটআপ সম্পন্ন!** আপনার শেখার যাত্রা চালিয়ে যান:

[অধ্যায় ৩: কোর জেনারেটিভ AI প্রযুক্তি](../../../03-CoreGenerativeAITechniques/README.md)

## সম্পদগুলি

- [Spring AI Azure OpenAI ডকুমেন্টেশন](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Microsoft Entra ID এর মাধ্যমে কী-মুক্ত প্রমাণীকরণ](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry পোর্টাল](https://ai.azure.com/)
- [Azure AI Foundry ডকুমেন্টেশন](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**অস্বীকৃতি**:
এই নথিটি AI অনুবাদ পরিষেবা [Co-op Translator](https://github.com/Azure/co-op-translator) ব্যবহার করে অনূদিত হয়েছে। যদিও আমরা শুদ্ধতার জন্য চেষ্টা করি, অনুগ্রহ করে মনে রাখবেন যে স্বয়ংক্রিয় অনুবাদে ত্রুটি বা অসঙ্গতি থাকতে পারে। মূল নথিটি তার স্বভাষায় কর্তৃত্বপূর্ণ উৎস হিসেবে বিবেচিত হওয়া উচিত। গুরুত্বপূর্ণ তথ্যের জন্য পেশাদার মানব অনুবাদ সুপারিশ করা হয়। এই অনুবাদের ব্যবহারে প্রয়োজনীয় ভুল বোঝাবুঝি বা ভুল ব্যাখ্যার জন্য আমরা দায়বদ্ধ নই।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->