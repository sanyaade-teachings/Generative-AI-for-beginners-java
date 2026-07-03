# ការឆាត់ជាមួយ Azure AI Foundry ដំបូង - ឧទាហរណ៍ពីចូលដល់ចេញ

ឧទាហរណ៍នេះគឺជាកម្មវិធី Spring Boot ងាយៗមួយដែលភ្ជាប់ទៅនឹងម៉ូឌែល **Azure AI Foundry** ដោយប្រើ **ការផ្ទៀងផ្ទាត់ដោយគ្មាន key** (Microsoft Entra ID) ហើយសាកល្បងការតំឡើងរបស់អ្នក។ វាប្រើ `ChatClient` របស់ Spring AI។

## អត្ថបទខ្លីៗ

- [លក្ខខណ្ឌមុន](#លក្ខខណ្ឌមុន)
- [ចាប់ផ្តើមយ៉ាងរហ័ស](#ចាប់ផ្តើមយ៉ាងរហ័ស)
- [របៀបផ្ទៀងផ្ទាត់ដំណើរការ](#របៀបផ្ទៀងផ្ទាត់ដំណើរការ)
- [ការរត់កម្មវិធី](#ការរត់កម្មវិធី)
  - [ការប្រើ Maven](#ការប្រើ-maven)
  - [ការប្រើ VS Code](#ការប្រើ-vs-code)
  - [លទ្ធផលដែលរំពឹងទុក](#លទ្ធផលដែលរំពឹងទុក)
- [យោងការកំណត់](#យោងការកំណត់)
  - [អថេរ​បរិស្ថាន](#អថេរ​បរិស្ថាន)
  - [ការកំណត់ Spring](#ការកំណត់-spring)
- [ការដោះស្រាយបញ្ហា](#ការដោះស្រាយបញ្ហា)
  - [បញ្ហាសកម្មជាច្រើន](#បញ្ហាសកម្មជាច្រើន)
  - [ម៉ូដធ្វើបញ្ហាដោះស្រាយ](#ម៉ូដធ្វើបញ្ហាដោះស្រាយ)
- [ជំហានបន្ទាប់](#ជំហានបន្ទាប់)
- [ធនធាន](#ធនធាន)

## លក្ខខណ្ឌមុន

មុនពេលដំណើរការឧទាហរណ៍នេះ សូមប្រាកដថាអ្នកមាន៖

- វត្ថុធាតុ Azure AI Foundry ជាមួយ `gpt-4o-mini` ដែលបានបញ្ចូល — បង្កើតវាជាមួយ `azd up` ឬដោយដៃតាម [មគ្គុទេសក៍ការតំឡើង Azure AI Foundry](../../getting-started-azure-openai.md)
- តួនាទី **Cognitive Services OpenAI User** លើវត្ថុធាតុនោះ (ទំព័រផ្ទុក Bicep បានផ្ដល់តួនាទីនេះសម្រាប់អ្នក)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli) ដែលបានចុះឈ្មោះចូលជាមួយ `az login`
- Java 21+ និង Maven 3.9+

> **មិនត្រូវការផ្ទៀងផ្ទាត់កូនសោ API** — ការផ្ទៀងផ្ទាត់គឺគ្មាន key ដោយ Microsoft Entra ID។

## ចាប់ផ្តើមយ៉ាងរហ័ស

```bash
# 1. នាវីហ្គែត​ទៅគម្រោង
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. ចូលគណនី ដើម្បីអោយការផ្ទៀងផ្ទាត់ឥតគ្រាប់អាចទទួលបានសញ្ញា Token
az login

# 3. កំណត់រចនាសម្ព័ន្ធចុងបញ្ចប់
#    - បើអ្នកបានរត់ `azd up` .env ត្រូវបានសរសេរដល់អ្នក (រំលងនេះ)
#    - បើមិនដូច្នោះទេ ចម្លងគំរូ និងកំណត់ AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. រត់កម្មវិធី
mvn spring-boot:run
```

## របៀបផ្ទៀងផ្ទាត់ដំណើរការ

ឧទាហរណ៍នេះផ្ទៀងផ្ទាត់ជាមួយ **Microsoft Entra ID** — មិនមានកូនសោ API ទេ។

ពេលមានតែ `spring.ai.azure.openai.endpoint` តែប៉ុណ្ណោះ (ហើយគ្មាន api-key) Spring AI នឹងបង្កើតកាឡាយអាជីព Azure OpenAI ជាមួយ [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential)។ វាប្រើនៅចំណុចនោះដោយស្វ័យប្រវត្តិបាន token ពីសម័យ `az login` របស់អ្នកនៅក្នុងតំបន់របស់អ្នក ឬពី managed identity នៅពេលរត់ក្នុង Azure — ដូច្នេះកូដដូចគ្នានេះដំណើរការបានទាំងពីរប្រភេទដោយគ្មានការផ្លាស់ប្ដូរ។

## ការរត់កម្មវិធី

### ការប្រើ Maven

```bash
mvn spring-boot:run
```

### ការប្រើ VS Code

1. បើកគំរូនៅក្នុង VS Code
2. ចុច `F5` ឬប្រើផ្ទាំង "Run and Debug"
3. ជ្រើសរើសការកំណត់ "Spring Boot-BasicChatApplication"

> **ចំណាំ**: ការកំណត់ VS Code នេះបញ្ចូលឯកសារ .env របស់អ្នកដោយស្វ័យប្រវត្តិ។

### លទ្ធផលដែលរំពឹងទុក

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

## យោងការកំណត់

### អថេរ​បរិស្ថាន

| អថេរ | ការពិពណ៌នា | ត្រូវការ | ឧទាហរណ៍ |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | ស្ថានីយ៍ endpoint របស់ Foundry (Azure OpenAI) | ត្រូវការ | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | ឈ្មោះការចេញផ្សាយម៉ូឌែលជជែក | មិនត្រូវការ | `gpt-4o-mini` (លំនាំដើម) |

> មិនមានអថេរកូនសោ API នោះទេ — ការផ្ទៀងផ្ទាត់គឺគ្មាន key (Microsoft Entra ID តាមរយៈ `az login`)។

### ការកំណត់ Spring

ឯកសារ `application.yml` កំណត់៖
- **Endpoint**: `${AZURE_OPENAI_ENDPOINT}` - ពីអថេរបរិស្ថាន
- **Deployment**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - ពីអថេរបរិស្ថានជាមួយ fallback
- **Auth**: គ្មាន key — មិនមាន `api-key` ដាក់ទេ ដូច្នេះ Spring AI ប្រើ `DefaultAzureCredential`
- **សីតុណ្ហភាព**: `0.7` - គ្រប់គ្រងភាពច្នៃប្រឌិត (0.0 = ពាក់ព័ន្ធត្រឹមត្រូវ, 1.0 = មានច្នៃប្រឌិត)
- **Max Tokens**: `500` - ប្រវែងឆ្លើយតបអតិបរមា

## ការដោះស្រាយបញ្ហា

### បញ្ហាសកម្មជាច្រើន

<details>
<summary><strong>កំហុស: 401 / "PermissionDenied" / កំហុស token</strong></summary>

- រត់ `az login` — ការផ្ទៀងផ្ទាត់គ្មាន key ត្រូវការចុះឈ្មោះចូលសកម្មដើម្បីទទួលបាន token
- ផ្ទៀងផ្ទាត់ថាអក្សរារបស់អ្នកមានតួនាទី **Cognitive Services OpenAI User** លើវត្ថុធាតុ
- ប្រសិនបើអ្នកទើបតែផ្ដល់តួនាទី សូមរង់ចាំមួយនាទីដើម្បីឲ្យវាបន្តផ្សាយ
- បញ្ជាក់ថាអ្នកនៅក្នុង tenant/subscription ត្រឹមត្រូវ (`az account show`)
</details>

<details>
<summary><strong>កំហុស: "The endpoint is not valid" / កំហុសភ្ជាប់</strong></summary>

- បញ្ជាក់ថា `AZURE_OPENAI_ENDPOINT` គឺ URL ពិស្តារពេញលេញ (ឧ. `https://your-resource.openai.azure.com/`)
- ពិនិត្យសញ្ញាផ្តល់ដាច់បញ្ចប់សុទ្ធតែត្រូវគ្នា
- ផ្ទៀងផ្ទាត់ endpoint ត្រូវតាមវត្ថុធាតុដែលបានបង្កើត (`azd env get-values`)
</details>

<details>
<summary><strong>កំហុស: "The deployment was not found"</strong></summary>

- ផ្ទៀងផ្ទាត់ថា `AZURE_OPENAI_DEPLOYMENT` ត្រូវនឹងឈ្មោះការចេញផ្សាយក្នុង Azure
- ពិនិត្យមើលថាម៉ូឌែលបានចេញផ្សាយដោយជោគជ័យនិងសកម្មទេ
- ឈ្មោះការចេញផ្សាយលំនាំដើមគឺ `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: អថេរបរិស្ថានមិនបានផ្ទុក</strong></summary>

- បញ្ជាក់ឲ្យឯកសារ `.env` របស់អ្នកនៅក្នុងថត root របស់គំរូ (នៅកម្រិតដូចគ្នាជាមួយ `pom.xml`)
- ព្យាយាមរត់ `mvn spring-boot:run` ក្នុង Terminal រួមបញ្ចូលរបស់ VS Code
- ពិនិត្យមើលថា Extension Java របស់ VS Code បានដំឡើងត្រឹមត្រូវ
</details>

### ម៉ូដធ្វើបញ្ហាដោះស្រាយ

ដើម្បីបើកការចុះបញ្ជីលម្អិត សូមយកការអធិប្បាយខាងក្រោមក្នុង `application.yml`៖

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## ជំហានបន្ទាប់

**ការតំឡើងរួចរាល់!** សូមបន្តការសិក្សារបស់អ្នក៖

[ជំពូក 3: បច្ចេកទេស Core Generative AI](../../../03-CoreGenerativeAITechniques/README.md)

## ធនធាន

- [ឯកសារ Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [ការផ្ទៀងផ្ទាត់គ្មាន key ជាមួយ Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [ផតฟ័รม Azure AI Foundry](https://ai.azure.com/)
- [ឯកសារ Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ការបដិសេធ**:
ឯកសារនេះត្រូវបានបម្លែងភាសា ដោយប្រើសេវាបម្លែងភាសា AI [Co-op Translator](https://github.com/Azure/co-op-translator)។ ទោះយើងខ្ញុំមានក្តីប្រាថ្នាឱ្យបានច្បាស់លាស់ តែសូមយល់ដឹងថាការបម្លែងដោយស្វ័យប្រវត្តិក៏អាចមានកំហុសឬភាពមិនត្រឹមត្រូវ។ ឯកសារដើមជាភាសាទីតាំងគួរត្រូវបានគេប្រើជាប្រភពច្បាស់លាស់។ សម្រាប់ព័ត៌មានសំខាន់ៗ សូមណែនាំឱ្យប្រើប្រាស់ការប្រែដោយមនុស្សជំនាញ។ យើងខ្ញុំមិនទទួលខុសត្រូវចំពោះការយល់ច្រឡំ ឬការបកស្រាយខុសបន្ទាប់ពីការប្រើប្រាស់ការបម្លែងនេះនោះទេ។
<!-- CO-OP TRANSLATOR DISCLAIMER END -->