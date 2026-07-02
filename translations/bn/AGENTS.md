# AGENTS.md

## প্রকল্পের পর্যালোচনা

এটি জাভা দিয়ে জেনেরেটিভ এআই ডেভেলপমেন্ট শেখার জন্য একটি শিক্ষামূলক রেপোজিটরি। এটি লার্জ ল্যাংগুয়েজ মডেলস (LLMs), প্রম্পট ইঞ্জিনিয়ারিং, এম্বেডিংস, RAG (রিট্রিভাল-অগমেন্টেড জেনারেশন), এবং মডেল কন্টেক্সট প্রোটোকল (MCP) সহ একটি ব্যাপক হ্যান্ডস-অন কোর্স প্রদান করে।

**প্রধান প্রযুক্তিসমূহ:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI, এবং OpenAI SDKs

**স্থাপত্য:**
- অধ্যায় অনুসারে সংগঠিত একাধিক স্বাধীন Spring Boot অ্যাপ্লিকেশন
- বিভিন্ন এআই প্যাটার্ন প্রদর্শন করে নমুনা প্রকল্পসমূহ
- প্রি-কনফিগার করা ডেভ কন্টেইনারসহ GitHub Codespaces-রেডি

## সেটআপ কমান্ডসমূহ

### পূর্বশর্তসমূহ
- Java 21 বা তার উপরে
- Maven 3.x
- Azure AI Foundry মডেল ডিপ্লয়মেন্ট সহ একটি Azure সাবস্ক্রিপশন (`azd up` দিয়ে প্রোভিশন করুন)
- Azure CLI (`az`) এবং Azure Developer CLI (`azd`), কীলেস অথেন্টিকেশনের জন্য সাইন ইন করা অবস্থায়

### পরিবেশ সেটআপ

**অপশন ১: GitHub Codespaces (প্রস্তাবিত)**
```bash
# রিপোজিটরিটি ফর্ক করুন এবং GitHub UI থেকে একটি কোডস্পেস তৈরি করুন
# ডেভ কনটেইনার স্বয়ংক্রিয়ভাবে সমস্ত ডিপেনডেন্সি ইনস্টল করবে
# পরিবেশ সেটআপের জন্য প্রায় ২ মিনিট অপেক্ষা করুন
```

**অপশন ২: লোকাল ডেভ কন্টেইনার**
```bash
# রেপোজিটরি ক্লোন করুন
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Dev Containers এক্সটেনসন সহ VS Code এ খুলুন
# প্রম্পট দেওয়া হলে কনটেইনারে পুনরায় খুলুন
```

**অপশন ৩: লোকাল সেটআপ**
```bash
# ডিপেন্ডেন্সিগুলি ইনস্টল করুন
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# ইনস্টলেশন যাচাই করুন
java -version
mvn -version
```

### কনফিগারেশন

**Azure AI Foundry সেটআপ (কীলেস, প্রস্তাবিত):**
```bash
# Foundry অ্যাকাউন্ট + মডেল ডিপ্লয়মেন্ট কোড হিসেবে প্রস্তুত করুন
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd আপনার এন্ডপয়েন্ট সহ examples/basic-chat-azure/.env লেখে (কোনো কী নয়)
```

**ম্যানুয়াল এন্ডপয়েন্ট কনফিগ:**
```bash
# যদি আপনি azd ব্যবহার না করে থাকেন, তাহলে নিজে এন্ডপয়েন্ট সেট করুন (auth az login এর মাধ্যমে keyless থাকে)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# .env সম্পাদনা করুন: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## ডেভেলপমেন্ট ওয়ার্কফ্লো

### প্রকল্পের কাঠামো
```
/
├── 01-IntroToGenAI/              # Chapter 1: Introduction
├── 02-SetupDevEnvironment/       # Chapter 2: Environment setup
│   └── examples/                 # Working examples
├── 03-CoreGenerativeAITechniques/ # Chapter 3: Core techniques
├── 04-PracticalSamples/          # Chapter 4: Sample projects
│   ├── calculator/               # MCP service example
│   ├── foundrylocal/            # Local model integration
│   └── petstory/                # Multi-modal app
├── 05-ResponsibleGenAI/         # Chapter 5: Responsible AI
└── translations/                # Multi-language support
```

### অ্যাপ্লিকেশন চালানো

**Spring Boot অ্যাপ্লিকেশন চালানো:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**প্রকল্প বিল্ড করা:**
```bash
cd [project-directory]
mvn clean install
```

**MCP ক্যালকুলেটর সার্ভার শুরু করা:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# সার্ভার চলে http://localhost:8080 এ
```

**ক্লায়েন্ট উদাহরণ চালানো:**
```bash
# অন্য টার্মিনালে সার্ভার চালু করার পরে
cd 04-PracticalSamples/calculator

# সরাসরি MCP ক্লায়েন্ট
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# কৃত্রিম বুদ্ধিমত্তা চালিত ক্লায়েন্ট (AZURE_OPENAI_ENDPOINT + az login প্রয়োজন)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# ইন্টারেক্টিভ বট
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### হট রিলোড
যে প্রজেক্টগুলো হট রিলোড সমর্থন করে সেগুলোতে Spring Boot DevTools অন্তর্ভুক্ত আছে:
```bash
# জাভা ফাইল পরিবর্তনগুলি সংরক্ষিত হলে স্বয়ংক্রিয়ভাবে পুনরায় লোড হবে
mvn spring-boot:run
```

## পরীক্ষার নির্দেশাবলী

### টেস্ট চালানো

**একটি প্রকল্পের সব টেস্ট চালানো:**
```bash
cd [project-directory]
mvn test
```

**ভেরBOSE আউটপুট সহ টেস্ট চালানো:**
```bash
mvn test -X
```

**নির্দিষ্ট টেস্ট ক্লাস চালানো:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### টেস্ট কাঠামো
- টেস্ট ফাইলগুলো JUnit 5 (Jupiter) ব্যবহার করে
- টেস্ট ক্লাসগুলো `src/test/java/` এ অবস্থিত
- ক্যালকুলেটর প্রকল্পের ক্লায়েন্ট উদাহরণগুলো `src/test/java/com/microsoft/mcp/sample/client/` এ আছে

### ম্যানুয়াল টেস্টিং
অনেক উদাহরণই ইন্টারঅ্যাকটিভ অ্যাপ্লিকেশন যা ম্যানুয়াল টেস্টিং প্রয়োজন:

1. `mvn spring-boot:run` দিয়ে অ্যাপ্লিকেশন স্টার্ট করুন
2. এন্ডপয়েন্টগুলো টেস্ট করুন বা CLI এর সাথে ইন্টারঅ্যাক্ট করুন
3. প্রত্যাশিত আচরণ প্রতিটি প্রকল্পের README.md এর সাথে মেলাচ্ছে কিনা যাচাই করুন

### Azure AI Foundry এর সাথে টেস্টিং
- উদাহরণ চালানোর আগে `az login` দিয়ে সাইন ইন করুন (কীলেস অথ)
- নিশ্চিত করুন আপনার একাউন্টে রিসোর্সের জন্য Cognitive Services OpenAI User ভূমিকা আছে
- অধ্যায় 5 এর রেসপনসিবল AI উদাহরণের সাথে কনটেন্ট ফিল্টারিং টেস্ট করুন

## কোড স্টাইল গাইডলাইনস

### Java কনভেনশনস
- **Java ভার্সন:** Java 21 আধুনিক ফিচারসহ
- **স্টাইল:** স্ট্যান্ডার্ড Java কনভেনশন অনুযায়ী
- **নেমিং:**
  - ক্লাস: PascalCase
  - মেথড/ভেরিয়েবল: camelCase
  - কনস্ট্যান্ট: UPPER_SNAKE_CASE
  - প্যাকেজ নাম: ছোট হাতের

### Spring Boot প্যাটার্নস
- ব্যবসায়িক লজিকের জন্য `@Service` ব্যবহার করুন
- REST এন্ডপয়েন্টের জন্য `@RestController` ব্যবহার করুন
- কনফিগারেশন `application.yml` বা `application.properties` এর মাধ্যমে
- হার্ড-কোডেড মানের বদলে পরিবেশ ভেরিয়েবল প্রাধান্য দিন
- MCP-এক্সপোজড মেথডের জন্য `@Tool` অ্যানোটেশন ব্যবহার করুন

### ফাইল সংগঠন
```
src/
├── main/
│   ├── java/
│   │   └── com/microsoft/[component]/
│   │       ├── [Component]Application.java
│   │       ├── config/
│   │       ├── controller/
│   │       ├── service/
│   │       └── exception/
│   └── resources/
│       ├── application.yml
│       └── static/
└── test/
    └── java/
        └── com/microsoft/[component]/
```

### ডিপেন্ডেন্সিসমূহ
- Maven `pom.xml` এর মাধ্যমে ব্যবস্থাপনা
- Spring AI BOM ভার্সন ম্যানেজমেন্টের জন্য
- AI ইন্টিগ্রেশনের জন্য LangChain4j
- স্প্রিং ডিপেন্ডেন্সির জন্য Spring Boot স্টার্টার প্যারেন্ট

### কোড মন্তব্য
- পাবলিক API এর জন্য JavaDoc যোগ করুন
- জটিল AI ইন্টারঅ্যাকশনের জন্য ব্যাখ্যামূলক মন্তব্য দিন
- MCP টুল বর্ণনাগুলো স্পষ্টভাবে ডকুমেন্ট করুন

## বিল্ড এবং ডিপ্লয়মেন্ট

### প্রকল্প বিল্ড

**টেস্ট ছাড়া বিল্ড:**
```bash
mvn clean install -DskipTests
```

**সব চেক সহ বিল্ড:**
```bash
mvn clean install
```

**অ্যাপ্লিকেশন প্যাকেজিং:**
```bash
mvn package
# target/ ডিরেক্টরিতে JAR তৈরি করে
```

### আউটপুট ডিরেক্টরিসমূহ
- কম্পাইল করা ক্লাসসমূহ: `target/classes/`
- টেস্ট ক্লাসসমূহ: `target/test-classes/`
- জার ফাইলসমূহ: `target/*.jar`
- Maven আর্টিফ্যাক্টস: `target/`

### পরিবেশ-নির্দিষ্ট কনফিগারেশন

**ডেভেলপমেন্ট:**
```yaml
# application.yml (keyless - no api-key; auth via DefaultAzureCredential)
spring:
  ai:
    azure:
      openai:
        endpoint: ${AZURE_OPENAI_ENDPOINT}
        chat:
          options:
            deployment-name: ${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}
```

**প্রোডাকশন:**
- কীলেস অথের জন্য `az login` এর বদলে ম্যানেজড আইডেন্টিটি ব্যবহার করুন
- `AZURE_OPENAI_ENDPOINT` আপনার প্রোডাকশন Foundry রিসোর্স পয়েন্ট کرے
- কনফিগারেশন পরিবেশ ভেরিয়েবল বা Azure Key Vault এর মাধ্যমে পরিচালনা করুন

### ডিপ্লয়মেন্ট বিবেচনা
- এটি একটি শিক্ষামূলক রেপোজিটরি যার নমুনা অ্যাপ্লিকেশন রয়েছে
- সরাসরি প্রোডাকশনে ডিপ্লয় করার জন্য ডিজাইন করা হয়নি
- নমুনাগুলো প্রোডাকশন ব্যবহারের জন্য প্যাটার্ন দেখায়
- নির্দিষ্ট ডিপ্লয়মেন্ট নোটের জন্য প্রত্যেক প্রকল্পের README দেখুন

## অতিরিক্ত নোট

### Azure AI Foundry
- **কীলেস অথ:** Microsoft Entra ID দিয়ে সংযুক্ত — কোনো API কী ম্যানেজ করতে হবে না
- **কোড হিসেবে প্রোভিশন:** Bicep + azd (`azd up`) একাউন্ট ও মডেল ডিপ্লয়মেন্ট তৈরি করে
- একই OpenAI-কমপ্যাটিবল কোড লোকালি (`az login`) এবং Azure-এ (ম্যানেজড আইডেন্টিটি) চলে

### একাধিক প্রকল্পের সাথে কাজ
প্রত্যেক নমুনা প্রকল্প স্বাধীন:
```bash
# নির্দিষ্ট প্রজেক্টে নেভিগেট করুন
cd 04-PracticalSamples/[project-name]

# প্রতিটির নিজস্ব pom.xml রয়েছে এবং স্বাধীনভাবে তৈরি করা যেতে পারে
mvn clean install
```

### সাধারণ সমস্যা

**Java ভার্সন মিলছে না:**
```bash
# জাভা ২১ নিশ্চিত করুন
java -version
# প্রয়োজনে JAVA_HOME আপডেট করুন
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**ডিপেন্ডেন্সি ডাউনলোড সমস্যা:**
```bash
# মেভেন ক্যাশে পরিষ্কার করুন এবং পুনরায় চেষ্টা করুন
rm -rf ~/.m2/repository
mvn clean install
```

**এন্ডপয়েন্ট বা সাইন-ইন পাওয়া যায়নি:**
```bash
# বর্তমান সেশনে এন্ডপয়েন্ট সেট করুন এবং সাইন ইন করুন (কি ছাড়া)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# অথবা প্রকল্প ডিরেক্টরিতে একটি .env ফাইল ব্যবহার করুন
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**পোর্ট ইতিমধ্যে ব্যবহার হচ্ছে:**
```bash
# Spring Boot ডিফল্টরূপে 8080 পোর্ট ব্যবহার করে
# application.properties এ পরিবর্তন:
server.port=8081
```

### বহুভাষা সমর্থন
- ৪৫+ ভাষায় স্বয়ংক্রিয় অনুবাদের মাধ্যমে ডকুমেন্টেশন উপলব্ধ
- অনুবাদসমূহ `translations/` ডিরেক্টরিতে
- অনুবাদ GitHub Actions ওয়ার্কফ্লো দিয়ে পরিচালিত

### শিক্ষার পথ
1. [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md) থেকে শুরু করুন
2. অধ্যায়গুলো ক্রম অনুযায়ী অনুসরণ করুন (01 → 05)
3. প্রতিটি অধ্যায়ের হ্যান্ডস-অন উদাহরণ সম্পন্ন করুন
4. অধ্যায় ৪ এর নমুনা প্রকল্পগুলো অন্বেষণ করুন
5. অধ্যায় ৫ তে রেসপনসিবল AI অনুশীলন শিখুন

### ডেভেলপমেন্ট কন্টেইনার
`.devcontainer/devcontainer.json` কনফিগার করে:
- Java 21 ডেভেলপমেন্ট পরিবেশ
- Maven প্রি-ইনস্টল্ড
- VS Code Java এক্সটেনশনস
- Spring Boot টুলস
- GitHub Copilot ইন্টিগ্রেশন
- Docker-in-Docker সাপোর্ট
- Azure CLI

### পারফরমেন্স বিবেচনা
- Azure AI Foundry ডিপ্লয়মেন্টে প্রতি মিনিটে টোকেন/রিকোয়েস্ট কোটা থাকে
- এম্বেডিংসের জন্য উপযুক্ত ব্যাচ সাইজ ব্যবহার করুন
- পুনরাবৃত্ত API কলের জন্য ক্যাশিং বিবেচনা করুন
- খরচ অপ্টিমাইজেশনের জন্য টোকেন ব্যবহারের পর্যবেক্ষণ করুন

### নিরাপত্তা নোট
- কখনোই `.env` ফাইল কমিট করবেন না (ইতিমধ্যে `.gitignore` এ আছে)
- API কীয়ের পরিবর্তে কীলেস অথ (Microsoft Entra ID) প্রাধান্য দিন
- Azure-এ ম্যানেজড আইডেন্টিটি ব্যবহার করুন; লোকাল ডেভের জন্য `az login`
- অধ্যায় ৫ এর রেসপনসিবল AI গাইডলাইন অনুসরণ করুন

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**অস্বীকৃতি**:
এই নথিটি AI অনুবাদ পরিষেবা [Co-op Translator](https://github.com/Azure/co-op-translator) ব্যবহার করে অনূদিত হয়েছে। যদিও আমরা শুদ্ধতার জন্য চেষ্টা করি, অনুগ্রহ করে মনে রাখবেন যে স্বয়ংক্রিয় অনুবাদে ত্রুটি বা অসঙ্গতি থাকতে পারে। মূল নথিটি তার স্বভাষায় কর্তৃত্বপূর্ণ উৎস হিসেবে বিবেচিত হওয়া উচিত। গুরুত্বপূর্ণ তথ্যের জন্য পেশাদার মানব অনুবাদ সুপারিশ করা হয়। এই অনুবাদের ব্যবহারে প্রয়োজনীয় ভুল বোঝাবুঝি বা ভুল ব্যাখ্যার জন্য আমরা দায়বদ্ধ নই।
<!-- CO-OP TRANSLATOR DISCLAIMER END -->