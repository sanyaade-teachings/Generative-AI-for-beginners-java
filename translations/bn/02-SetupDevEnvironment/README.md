# জেনারেটিভ AI for Java-এর জন্য ডেভেলপমেন্ট এনভায়রনমেন্ট সেট আপ করা

> **দ্রুত শুরু:** কয়েক মিনিটের মধ্যেই Bicep + `azd` দিয়ে কোড হিসেবে আপনার AI মডেলগুলি **Azure AI Foundry**-তে প্রোভিশন করুন — [Azure AI Foundry সেটআপ গাইড](getting-started-azure-openai.md) দেখুন। অথেন্টিকেশন **কীলেস** (Microsoft Entra ID), তাই কোনো API কী ম্যানেজ করতে হয় না।

## আপনি কি শিখবেন

- AI অ্যাপ্লিকেশনের জন্য একটি Java ডেভেলপমেন্ট এনভায়রনমেন্ট সেট আপ করা
- আপনার পছন্দসই ডেভেলপমেন্ট এনভায়রনমেন্ট নির্বাচন ও কনফিগার করা (ক্লাউড-প্রথম Codespaces দিয়ে, লোকাল ডেভ কন্টেইনার, অথবা পূর্ণলোকাল সেটআপ)
- Azure AI Foundry মডেলের সাথে সংযোগ করে আপনার সেটআপ পরীক্ষা করা

## বিষয়সূচি

- [আপনি কি শিখবেন](#আপনি-কি-শিখবেন)
- [পরিচিতি](#পরিচিতি)
- [ধাপ ১: আপনার ডেভেলপমেন্ট এনভায়রনমেন্ট সেট আপ করুন](#ধাপ-১-আপনার-ডেভেলপমেন্ট-এনভায়রনমেন্ট-সেট-আপ-করুন)
  - [অপশন A: GitHub Codespaces (পরামর্শকৃত)](#অপশন-a-github-codespaces-পরামর্শকৃত)
  - [অপশন B: লোকাল ডেভ কন্টেইনার](#অপশন-b-লোকাল-ডেভ-কন্টেইনার)
  - [অপশন C: আপনার বিদ্যমান লোকাল ইনস্টলেশন ব্যবহার করুন](#অপশন-c-আপনার-বিদ্যমান-লোকাল-ইনস্টলেশন-ব্যবহার-করুন)
- [ধাপ ২: Azure AI Foundry প্রোভিশন করুন](#ধাপ-২-azure-ai-foundry-প্রোভিশন-করুন)
- [ধাপ ৩: আপনার সেটআপ টেস্ট করুন](#ধাপ-৩-আপনার-সেটআপ-পরীক্ষা-করুন)
- [সমস্যা সমাধান](#সমস্যা-সমাধান)
- [সারাংশ](#সারাংশ)
- [পরবর্তী ধাপসমূহ](#পরবর্তী-ধাপসমূহ)

## পরিচিতি

এই অধ্যায়টি আপনাকে ডেভেলপমেন্ট এনভায়রনমেন্ট সেট আপ করার জন্য গাইড করবে। আমরা কোর্স জুড়ে মডেলগুলোর জন্য **Azure AI Foundry** ব্যবহার করব। আপনি Bicep এবং Azure Developer CLI (`azd`) দিয়ে কোড হিসেবে মডেলগুলি প্রোভিশন করবেন, তারপর **কীলেস অথেন্টিকেশনের** (Microsoft Entra ID) মাধ্যমে সংযোগ করবেন — কোনো API কী কপি বা ফাঁসের ঝামেলা নেই।

**লোকাল সেটআপের প্রয়োজন নেই!** আপনি GitHub Codespaces ব্যবহার করতে পারেন, যা আপনার ব্রাউজারে পূর্ণাঙ্গ ডেভেলপমেন্ট এনভায়রনমেন্ট প্রদান করে, এবং ওখান থেকেই Foundry প্রোভিশন করতে পারবেন।

আমরা এই কোর্সে **Azure AI Foundry** ব্যবহার করি কারণ এটি:
- **কোড হিসেবে প্রোভিশন করা হয়** — একবার `azd up` কমান্ড চালালে অ্যাকাউন্ট এবং মডেল ডিপ্লয়মেন্ট ডিপ্লয় হয়ে যায়
- **কীলেস** — Azure সাইন-ইন অথবা ম্যানেজড আইডেন্টিটির মাধ্যমে অথেন্টিকেট হয়
- **প্রোডাকশন-রেডি** — একই কোড লোকাল এবং Azure উভয় জায়গায় চলে
- **নমনীয়** — ডিপ্লয়মেন্ট নাম পরিবর্তন করে মডেল বদলায়, কোড বদলানোর দরকার নেই

> **মন্তব্য**: Azure AI Foundry ডিপ্লয়মেন্টগুলি প্রতি টোকেনের ভিত্তিতে বিল করা হয় (পে-অ্যাজ-ইউ-গো)। প্রোভিশনিং, রিজিয়ন এবং খরচ সম্পর্কিত বিস্তারিত জানতে [Azure AI Foundry সেটআপ গাইড](getting-started-azure-openai.md) দেখুন।

## ধাপ ১: আপনার ডেভেলপমেন্ট এনভায়রনমেন্ট সেট আপ করুন

<a name="quick-start-cloud"></a>

আমরা একটি প্রি-কনফিগার্ড ডেভ কন্টেইনার তৈরি করেছি যাতে সেটআপ সময় কম হয় এবং Generative AI for Java কোর্সের জন্য প্রয়োজনীয় সব টুল থাকে। আপনার পছন্দমত ডেভেলপমেন্ট পদ্ধতি নির্বাচন করুন:

### এনভায়রনমেন্ট সেটআপের বিকল্পসমূহ:

#### অপশন A: GitHub Codespaces (পরামর্শকৃত)

**২ মিনিটে কোডিং শুরু করুন - কোনো লোকাল সেটআপ প্রয়োজন নেই!**

1. এই রিপোজিটোরি আপনার GitHub অ্যাকাউন্টে fork করুন
   > **মন্তব্য**: যদি আপনি বেসিক কনফিগারেশন এডিট করতে চান তবে [Dev Container Configuration](../../../.devcontainer/devcontainer.json) দেখুন
2. ক্লিক করুন **Code** → **Codespaces** ট্যাবে → **...** → **New with options...**
3. ডিফল্ট ব্যবহার করুন – এটি **Dev container configuration** নির্বাচন করবে: এই কোর্সের জন্য তৈরি **Generative AI Java Development Environment** কাস্টম ডেভকন্টেইনার
4. ক্লিক করুন **Create codespace**
5. প্রায় ২ মিনিট অপেক্ষা করুন যতক্ষণ এনভায়রনমেন্ট প্রস্তুত হয়
6. এগিয়ে যান [ধাপ ২: Azure AI Foundry প্রোভিশন করুন](#ধাপ-২-azure-ai-foundry-প্রোভিশন-করুন)

<img src="../../../translated_images/bn/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: Codespaces submenu" width="50%">

<img src="../../../translated_images/bn/image.833552b62eee7766.webp" alt="Screenshot: New with options" width="50%">

<img src="../../../translated_images/bn/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: Create codespace options" width="50%">

> **Codespaces এর সুবিধাগুলো**:
> - কোনো লোকাল ইনস্টলেশন প্রয়োজন নেই
> - যেকোনো ব্রাউজারযুক্ত ডিভাইসে কাজ করে
> - সব টুল এবং ডিপেনডেন্সি প্রি-কনফিগারড
> - ব্যক্তিগত অ্যাকাউন্টের জন্য প্রতি মাসে ৬০ ঘন্টা ফ্রি
> - সকল শিক্ষার্থীর জন্য সামঞ্জস্যপূর্ণ এনভায়রনমেন্ট

#### অপশন B: লোকাল ডেভ কন্টেইনার

**ডকার দিয়ে লোকাল ডেভেলপমেন্ট পছন্দকারীদের জন্য**

1. এই রিপোজিটোরি fork করে আপনার লোকাল মেশিনে ক্লোন করুন
   > **মন্তব্য**: যদি আপনি বেসিক কনফিগারেশন এডিট করতে চান তবে [Dev Container Configuration](../../../.devcontainer/devcontainer.json) দেখুন
2. [Docker Desktop](https://www.docker.com/products/docker-desktop/) এবং [VS Code](https://code.visualstudio.com/) ইনস্টল করুন
3. VS Code-এ [Dev Containers এক্সটেনশন](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) ইনস্টল করুন
4. রিপোজিটোরি ফোল্ডারটি VS Code-এ খুলুন
5. প্রম্পট এলে ক্লিক করুন **Reopen in Container** (বা `Ctrl+Shift+P` → "Dev Containers: Reopen in Container" ব্যবহার করুন)
6. কন্টেইনার বিল্ড হয়ে শুরু হওয়া পর্যন্ত অপেক্ষা করুন
7. এগিয়ে যান [ধাপ ২: Azure AI Foundry প্রোভিশন করুন](#ধাপ-২-azure-ai-foundry-প্রোভিশন-করুন)

<img src="../../../translated_images/bn/devcontainer.21126c9d6de64494.webp" alt="Screenshot: Dev container setup" width="50%">

<img src="../../../translated_images/bn/image-3.bf93d533bbc84268.webp" alt="Screenshot: Dev container build complete" width="50%">

#### অপশন C: আপনার বিদ্যমান লোকাল ইনস্টলেশন ব্যবহার করুন

**বিদ্যমান Java এনভায়রনমেন্ট থাকা ডেভেলপারদের জন্য**

প্রয়োজনীয়:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html) 
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) অথবা আপনার পছন্দসই IDE

ধাপসমূহ:
1. এই রিপোজিটোরি আপনার লোকাল মেশিনে ক্লোন করুন
2. আপনার IDE-তে প্রকল্পটি খুলুন
3. এগিয়ে যান [ধাপ ২: Azure AI Foundry প্রোভিশন করুন](#ধাপ-২-azure-ai-foundry-প্রোভিশন-করুন)

> **প্রো টিপ**: আপনার কম্পিউটার যদি কম ক্ষমতার হয় কিন্তু লোকালি VS Code ব্যবহার করতে চান, তাহলে GitHub Codespaces ব্যবহার করুন! আপনি আপনার লোকাল VS Code থেকে ক্লাউড-হোস্টেড Codespace-এ সংযোগ করতে পারবেন, যা শ্রেষ্ঠ সুবিধা দেয়।

<img src="../../../translated_images/bn/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: created local devcontainer instance" width="50%">

## ধাপ ২: Azure AI Foundry প্রোভিশন করুন

কোর্সের AI মডেলগুলো কোড হিসেবে Azure AI Foundry-তে ডিপ্লয় করুন। রিপোজিটোরির রুট ফোল্ডারে:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```

`azd` একটি এনভায়রনমেন্ট নাম ও রিজিয়ন চায়, Azure AI Foundry অ্যাকাউন্ট প্রোভিশন করে `gpt-4o-mini` এবং `text-embedding-3-small` ডিপ্লয়মেন্টসহ, এবং উদাহরণের `.env` ফাইলে এন্ডপয়েন্ট লেখে — সব কীলেস অথেন্টিকেশনের মাধ্যমে (কোনো API কী নেই)।

> **পূর্ণ নির্দেশিকা:** প্রয়োজনীয়তা, ম্যানুয়াল (পোর্টাল) বিকল্প, রিজিয়ন নির্দেশনা, ও খরচ/পরিষ্কার সম্পর্কিত টিপস জানতে [Azure AI Foundry Setup Guide](getting-started-azure-openai.md) দেখুন।

## ধাপ ৩: আপনার সেটআপ পরীক্ষা করুন

যখন আপনার Foundry মডেলগুলো প্রোভিশন হয়ে যাবে, তখন উদাহরণ অ্যাপের মাধ্যমে সংযোগ পরীক্ষা করুন [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) এ।

1. আপনার ডেভেলপমেন্ট এনভায়রনমেন্টে টার্মিনাল খুলুন।
2. উদাহরণ ফোল্ডারে ক্লোন করুন:
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```

3. নিশ্চিত করুন আপনি সাইন ইন করেছেন (কীলেস অথেন্টিকেশন টোকেন প্রয়োজন):
   ```bash
   az login
   ```

   > যদি আপনি `azd up` চালিয়ে থাকেন, `.env` ফাইলে আপনার এন্ডপয়েন্ট ইতিমধ্যেই লেখা থাকবে।
4. অ্যাপ্লিকেশন চালান:
   ```bash
   mvn clean spring-boot:run
   ```

আপনি `gpt-4o-mini` মডেল থেকে একটি রেসপন্স দেখতে পাবেন।

### উদাহরণ কোড বোঝা

`examples/basic-chat-azure` এর নিচে থাকা উদাহরণটি একটি Spring Boot অ্যাপ, যা **Spring AI** ব্যবহার করে Azure AI Foundry-তে কীলেস অথেন্টিকেশন দিয়ে সংযোগ করে।

**এই কোডটি যা করে:**
- Azure AI Foundry-তে আপনার Azure সাইন-ইন (Microsoft Entra ID) ব্যবহার করে সংযোগ করে — কোন API কী লাগে না
- `gpt-4o-mini` মডেলে একটি প্রম্পট পাঠায়
- AI-এর প্রতিক্রিয়া গ্রহণ করে এবং প্রদর্শন করে
- আপনার সেটআপ সঠিকভাবে কাজ করছে কিনা যাচাই করে

**মূল ডিপেনডেন্সি** (`pom.xml` এ):
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```

**কনফিগারেশন** (`application.yml`):
```yaml
spring:
  ai:
    azure:
      openai:
        # Endpoint only - no api-key. Spring AI uses DefaultAzureCredential (keyless).
        endpoint: ${AZURE_OPENAI_ENDPOINT}
        chat:
          options:
            deployment-name: ${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}
```


## সারাংশ

দারুণ! এখন আপনার সবকিছু সেট আপ হয়ে গেছে:

- Bicep + `azd` দিয়ে কোড হিসেবে Azure AI Foundry মডেল প্রোভিশন করা হয়েছে
- আপনার Java ডেভেলপমেন্ট এনভায়রনমেন্ট চালু হয়ে গেছে (চাইলে Codespaces, ডেভ কন্টেইনার অথবা লোকাল)
- কীলেস অথেন্টিকেশন (Microsoft Entra ID) দিয়ে Azure AI Foundry-র সাথে সংযোগ
- একটি সহজ উদাহরণের মাধ্যমে পরীক্ষা করে দেখেছেন যেটি আপনার মডেলের সাথে কথা বলে

## পরবর্তী ধাপসমূহ

[অধ্যায় ৩: মূল জেনারেটিভ AI কৌশলসমূহ](../03-CoreGenerativeAITechniques/README.md)

## সমস্যা সমাধান

সমস্যা হচ্ছে? এখানে সাধারণ সমস্যাগুলো এবং তাদের সমাধান:

- **অথেন্টিকেশন ব্যর্থ (৪০১/৪০৩)?**
  - `az login` চালান — অথেন্টিকেশন কীলেস, তাই সাইন ইন থাকতে হবে
  - নিশ্চিত করুন আপনার অ্যাকাউন্টের কাছে রিসোর্সের উপর **Cognitive Services OpenAI User** রোল আছে
  - আপনি যদি সদ্য প্রোভিশন করে থাকেন, রোল এসাইনমেন্ট ছড়ানোর জন্য কিছুক্ষণ অপেক্ষা করুন

- **Maven পাওয়া যায়নি?**
  - ডেভ কন্টেইনার/Codespaces ব্যবহার করলে Maven প্রি-ইনস্টল থাকে
  - লোকাল সেটআপে নিশ্চিত করুন Java 21+ এবং Maven 3.9+ ইনস্টল আছে
  - `mvn --version` কমান্ড দিয়ে ইনস্টলেশন যাচাই করুন

- **`azd` পাওয়া যায়নি বা প্রোভিশনিং ব্যর্থ?**
  - [Azure Developer CLI](https://aka.ms/azure-dev/install) ইনস্টল করুন এবং `azd auth login` চালান
  - এমন একটি রিজিয়ন নির্বাচন করুন যেখানে `gpt-4o-mini` উপলব্ধ (যেমন `eastus2`)
  - বিস্তারিত জন্য [Azure AI Foundry setup guide](getting-started-azure-openai.md) দেখুন

- **ডেভ কন্টেইনার চালু হচ্ছে না?**
  - লোকাল ডেভেলপমেন্টের জন্য Docker Desktop চলছে কিনা নিশ্চিত করুন
  - কন্টেইনার পুনরায় বিল্ড করার চেষ্টা করুন: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **অ্যাপ্লিকেশনের কম্পাইলেশন এরর?**
  - সঠিক ডিরেক্টরিতে আছেন কিনা যাচাই করুন: `02-SetupDevEnvironment/examples/basic-chat-azure`
  - ক্লিন এবং রিবিল্ড করুন: `mvn clean compile`

> **সহায়তা প্রয়োজন?**: এখনও সমস্যা থাকলে রিপোজিটোরিতে একটি ইস্যু ওপেন করুন, আমরা সাহায্য করব।

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**অস্বীকৃতি**:
এই নথিটি AI অনুবাদ পরিষেবা [Co-op Translator](https://github.com/Azure/co-op-translator) ব্যবহার করে অনূদিত হয়েছে। যদিও আমরা শুদ্ধতার জন্য চেষ্টা করি, অনুগ্রহ করে মনে রাখবেন যে স্বয়ংক্রিয় অনুবাদে ত্রুটি বা অসঙ্গতি থাকতে পারে। মূল নথিটি তার স্বভাষায় কর্তৃত্বপূর্ণ উৎস হিসেবে বিবেচিত হওয়া উচিত। গুরুত্বপূর্ণ তথ্যের জন্য পেশাদার মানব অনুবাদ সুপারিশ করা হয়। এই অনুবাদের ব্যবহারে প্রয়োজনীয় ভুল বোঝাবুঝি বা ভুল ব্যাখ্যার জন্য আমরা দায়বদ্ধ নই।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->