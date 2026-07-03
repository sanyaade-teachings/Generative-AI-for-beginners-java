# ការតំឡើងបរិយាកាសអភិវឌ្ឍសម្រាប់ Azure AI Foundry

> មគ្គុទេសក៍នេះកំណត់រចនាសម្ព័ន្ធម៉ូដែល **Azure AI Foundry** សម្រាប់កម្មវិធី AI Java នៅក្នុងវគ្គសិក្សានេះ ដោយប្រើការផ្ទៀងផ្ទាត់អត្តសញ្ញាណ **keyless** (Microsoft Entra ID) — គ្មានសោ API ត្រូវគ្រប់គ្រងទេ។ ថ្មីជាមួយឧបករណ៍នេះ? ចាប់ផ្តើមជាមួយ [មគ្គុទេសក៍បរិយាកាសអភិវឌ្ឍ](./README.md)។

មគ្គុទេសក៍នេះកំណត់រចនាសម្ព័ន្ធម៉ូដែល **Azure AI Foundry** សម្រាប់កម្មវិធី AI Java នៅក្នុងវគ្គសិក្សានេះ។ អ្នកមានពីរផ្លូវ៖

- **ជម្រើស A — តំឡើងជាមួយ `azd` + Bicep (ណែនាំ):** ពាក្យបញ្ជាមួយសម្រាប់ដាក់បណ្តុំគណនី Foundry និងម៉ូដែលជាកូដ។ គ្មានការចុចយ៉ាងណានៅលើផតថលទេ។
- **ជម្រើស B — បង្កើតធនធានដោយដៃ** ក្នុងផតថល Azure AI Foundry។

ផ្លូវទាំងពីរប្រើការផ្ទៀងផ្ទាត់អត្តសញ្ញាណ **keyless** (Microsoft Entra ID) — គ្មានសោ API ត្រូវចម្លងឬរំលាយ។

## មាតិកា

- [អ្វីដែលត្រូវបានបង្កើត](#អ្វីដែលត្រូវបានបង្កើត)
- [លក្ខខណ្ឌមុន](#លក្ខខណ្ឌមុន)
- [ជម្រើស A: តំឡើងជាមួយ azd + Bicep (ណែនាំ)](#option-a-provision-with-azd--bicep-recommended)
- [ជម្រើស B: បង្កើតធនធានដោយដៃ](#ជម្រើស-b-បង្កើតធនធានដោយដៃ)
- [បត់បែនបរិយាកាសរបស់អ្នក](#បត់បែនបរិយាកាសរបស់អ្នក)
- [សាកល្បងការតំឡើងរបស់អ្នក](#សាកល្បងការតំឡើងរបស់អ្នក)
- [តើអ្វីជាដំណាក់កាលបន្ទាប់?](#តើអ្វីជាដំណាក់កាលបន្ទាប់)
- [ធនធាន](#ធនធាន)
- [ធនធានបន្ថែម](#ធនធានបន្ថែម)

## អ្វីដែលត្រូវបានបង្កើត

ទំរង់ Bicep ក្នុង [`infra/`](../../../02-SetupDevEnvironment/infra) បង្កើត៖

- គណនី **Azure AI Foundry** (`Microsoft.CognitiveServices/accounts`, ប្រភេទ `AIServices`) ជាមួយគម្រោងមួយ
- ការដាក់បញ្ចូល **chat** — `gpt-4o-mini`
- ការដាក់បញ្ចូល **embedding** — `text-embedding-3-small` (ប្រើនៅវគ្គបន្ទាប់)
- ការតែងតាំងតួនាទី **keyless** (`Cognitive Services OpenAI User`) ដូច្នេះអ្នកអាចចូលដោយ `az login` ជំនួសការគ្រប់គ្រងសោ

## លក្ខខណ្ឌមុន

- ការជាវ [Azure](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) និង [Maven 3.9+](https://maven.apache.org/download.cgi)

## ជម្រើស A: តំឡើងជាមួយ azd + Bicep (ណែនាំ)

ចូលទៅកាន់ថត `02-SetupDevEnvironment`៖

```bash
cd 02-SetupDevEnvironment

# ចូលបន្ទាន់ (ឧបករណ៍ទាំងពីរ)
azd auth login
az login

# រៀបចំគណនី Foundry + ការចាក់ផ្សាយម៉ូដែល
azd up
```

`azd` នឹងស្នើឲ្យបញ្ចូល **ឈ្មោះបរិយាកាស** (ឧ. `genai-java`) និង **តំបន់**។ ជ្រើសតំបន់ដែលមាន `gpt-4o-mini` និង `text-embedding-3-small` — ឧ. `eastus2` ឬ `swedencentral`។

ពេលការតំឡើងបញ្ចប់ azd៖

1. ដាក់បញ្ចូលគ្រប់យ៉ាងដែលបានកំណត់នៅក្នុង [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep)។
2. រត់ព្រឹត្តិការណ៍បន្ទាប់ពេលបញ្ចប់ (postprovision hook) ដែលសរសេរ [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) ជាមួយឈ្មោះចំណុចចុងបញ្ចប់ និងឈ្មោះការដាក់បញ្ចូល (គ្មានអាថ៌កំបាំង)។

> **កន្លែង​ពិចារណា៖** ធ្វើការ `azd up` ម្តងទៀតគ្រប់ពេល ដើម្បីអនុវត្តន៍ការផ្លាស់ប្តូរ។ ប្រើ `azd down` ដើម្បីលុបចោលគ្រប់យ៉ាង និងបញ្ឈប់ការចំណាយ។

ដើម្បីមើលការកំណត់ដែលបានបង្កើត៖

```bash
azd env get-values
```

ឥឡូវនេះរោលទៅ [សាកល្បងការតំឡើងរបស់អ្នក](#សាកល្បងការតំឡើងរបស់អ្នក)។

## ជម្រើស B: បង្កើតធនធានដោយដៃ

ចូលចិត្តផតថលទេ? បង្កើតធនធានដោយដៃ៖

1. ចូលទៅកាន់ [ផតថល Azure AI Foundry](https://ai.azure.com/) ហើយចូលគណនី។
2. **បង្កើតគម្រោង** (វាក៏បង្កើតធនធាន AI Foundry)។ ផ្ដល់ឈ្មោះដូចជា `GenAIJava`។
3. នៅក្នុងគម្រោងរបស់អ្នក បើក **ម៉ូដែល + ចំណុចបញ្ចប់** → **ដាក់បញ្ចូលម៉ូដែល** → **ដាក់បញ្ចូលម៉ូដែលមេ**។
4. ដាក់បញ្ចូល **gpt-4o-mini** (ឈ្មោះការដាក់បញ្ចូល `gpt-4o-mini`)។ ធ្វើឡើងម្តងទៀតសម្រាប់ **text-embedding-3-small** ប្រសិនបើចង់ប្រើឧទាហរណ៍ embedding។
5. ពីក្រោម **ទាំងមូល** ចម្លង **ចំណុចបញ្ចប់** (ឧ. `https://<resource>.openai.azure.com/`)។
6. ផ្ដល់សិទ្ធិ keyless ដល់ខ្លួនឯង៖ នៅលើធនធាន បើក **ការត្រួតពិនិត្យការចូល (IAM)** → **បន្ថែមការតែងតាំងតួនាទី** → ផ្ដល់តួនាទី **Cognitive Services OpenAI User** ជាគណនីរបស់អ្នក។

> **នៅតែមានបញ្ហា?** មើលឯកសាររបស់ [Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects)។

## បត់បែនបរិយាកាសរបស់អ្នក

**បើអ្នកប្រើជម្រើស A (`azd up`)** ឯកសារកំណត់របស់អ្នកបានសរសេរអ្នកហើយ — មិនចាំបាច់កំណត់ឲ្យបន្ថែមទេ។ រំលងទៅ [សាកល្បងការតំឡើងរបស់អ្នក](#សាកល្បងការតំឡើងរបស់អ្នក)។

**បើអ្នកប្រើជម្រើស B (ដោយដៃ)** បង្កើតឯកសារ `.env` នៃឧទាហរណ៍ដោយខ្លួនឯង៖

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

កែសម្រួល `.env` ជាមួយចំណុចបញ្ចប់របស់អ្នក (គ្មានសោ — ការផ្ទៀងផ្ទាត់គឺ keyless)៖

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **ចំណាំសុវត្ថិភាព៖** គ្មានសោ API ត្រូវរក្សាទុកទេ។ អ្នកផ្ទៀងផ្ទាត់អត្តសញ្ញាណជាមួយ Microsoft Entra ID តាមរយៈ `az login` (នៅក្នុងម៉ាស៊ីនមូលដ្ឋាន) ឬ managed identity (នៅក្នុង Azure)។ ឯកសារ `.env` រក្សាទុកតែការកំណត់ដែលមិនមែនជាអាថ៌កំបាំងប៉ុណ្ណោះ ហើយត្រូវបានគ្របដណ្តប់ដោយ `.gitignore`។

## សាកល្បងការតំឡើងរបស់អ្នក

ធ្វើឲ្យប្រាកដថាអ្នកបានចូលគណនី ដើម្បីការផ្ទៀងផ្ទាត់អត្តសញ្ញាណ keyless អាចយក token បាន បន្ទាប់មករត់ឧទាហរណ៍៖

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # បើអ្នកមិនទាន់បានចូលគណនី
mvn clean spring-boot:run
```

អ្នកគួរតែឃើញចម្លើយពីម៉ូដែល `gpt-4o-mini`!

> **អ្នកប្រើ VS Code:** ចុច `F5` ដើម្បីរត់។ កម្មវិធីបញ្ចូលឯកសារ `.env` ដោយស្វ័យប្រវត្តិ។

> **ឧទាហរណ៍ពេញលេញ៖** មើល [ឧទាហរណ៍ Basic Chat ជាមួយ Azure AI Foundry](./examples/basic-chat-azure/README.md) សម្រាប់ព្រឹត្តិការណ៍/ការដោះស្រាយបញ្ហា។

## តើអ្វីជាដំណាក់កាលបន្ទាប់?

**ការតំឡើងបានបញ្ចប់!** ឥឡូវនេះអ្នកមាន៖
- Azure AI Foundry ជាមួយ `gpt-4o-mini` និង `text-embedding-3-small` ត្រូវបានដាក់បញ្ចូល
- ការផ្ទៀងផ្ទាត់អត្តសញ្ញាណ keyless (Microsoft Entra ID) — គ្មានសោត្រូវគ្រប់គ្រង
- ឯកសារ `.env` មួយសម្រាប់ចំណុចបញ្ចប់ និងឈ្មោះការដាក់បញ្ចូល
- បរិយាកាសអភិវឌ្ឍ Java ដែលត្រៀមរួច

**បន្តទៅ** [ជំពូក ៣៖ ជំនាញ AI សំណង់ចម្បង](../03-CoreGenerativeAITechniques/README.md) ដើម្បីចាប់ផ្តើមបង្កើតកម្មវិធី AI!

## ធនធាន

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [ការផ្ទៀងផ្ទាត់ keyless ជាមួយ Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [ឯកសារ Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)
- [ឯកសារ Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## ធនធានបន្ថែម

- [ទាញយក VS Code](https://code.visualstudio.com/Download)
- [ទាញយក Docker Desktop](https://www.docker.com/products/docker-desktop)
- [ការកំណត់ Dev Container](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ការបដិសេធ**:
ឯកសារនេះត្រូវបានបម្លែងភាសា ដោយប្រើសេវាបម្លែងភាសា AI [Co-op Translator](https://github.com/Azure/co-op-translator)។ ទោះយើងខ្ញុំមានក្តីប្រាថ្នាឱ្យបានច្បាស់លាស់ តែសូមយល់ដឹងថាការបម្លែងដោយស្វ័យប្រវត្តិក៏អាចមានកំហុសឬភាពមិនត្រឹមត្រូវ។ ឯកសារដើមជាភាសាទីតាំងគួរត្រូវបានគេប្រើជាប្រភពច្បាស់លាស់។ សម្រាប់ព័ត៌មានសំខាន់ៗ សូមណែនាំឱ្យប្រើប្រាស់ការប្រែដោយមនុស្សជំនាញ។ យើងខ្ញុំមិនទទួលខុសត្រូវចំពោះការយល់ច្រឡំ ឬការបកស្រាយខុសបន្ទាប់ពីការប្រើប្រាស់ការបម្លែងនេះនោះទេ។
<!-- CO-OP TRANSLATOR DISCLAIMER END -->