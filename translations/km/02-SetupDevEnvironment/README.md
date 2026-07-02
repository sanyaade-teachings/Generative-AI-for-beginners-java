# ការតំឡើងបរិយាកាសអភិវឌ្ឍសម្រាប់ Generative AI សម្រាប់ Java

> **ចាប់ផ្តើមឆាប់រហ័ស:** ផ្តល់សេវាម៉ូដែល AI របស់អ្នកលើ **Azure AI Foundry** ជាកូដ ជាមួយ Bicep + `azd` ក្នុងរយៈពេលតែប៉ុន្មាននាទី — មើល [មគ្គុទេសក៍តំឡើង Azure AI Foundry](getting-started-azure-openai.md)។ ការផ្ទៀងផ្ទាត់គឺ **គ្មានកូនសោ** (Microsoft Entra ID) ដូច្នេះគ្មាន API keys ត្រូវគ្រប់គ្រងទេ។

## អ្វីដែលអ្នកនឹងសិក្សា

- តំឡើងបរិយាកាសអភិវឌ្ឍ Java សម្រាប់កម្មវិធី AI
- ជ្រើសរើស និងកំណត់រចនាសម្ព័ន្ធបរិយាកាសអភិវឌ្ឍដែលអ្នកចូលចិត្ត (Cloud-first ជាមួយ Codespaces, local dev container ឬការតំឡើងសម្រូបពេញលេញក្នុងកុំព្យូទ័រ)
- សាកល្បងការតំឡើងឲ្យត្រូវការដោយភ្ជាប់ទៅម៉ូឌែល Azure AI Foundry

## តារាងមាតិកា

- [អ្វីដែលអ្នកនឹងសិក្សា](#អ្វីដែលអ្នកនឹងសិក្សា)
- [ប소개](#ប소개)
- [ជំហានទី 1៖ តំឡើងបរិយាកាសអភិវឌ្ឍរបស់អ្នក](#ជំហានទី-1៖-តំឡើងបរិយាកាសអភិវឌ្ឍរបស់អ្នក)
  - [ជម្រើស ក៖ GitHub Codespaces (ណែនាំ)](#ជម្រើស-ក៖-github-codespaces-ណែនាំ)
  - [ជម្រើស ខ៖ Local Dev Container](#ជម្រើស-ខ៖-local-dev-container)
  - [ជម្រើស គ៖ ប្រើការតំឡើងមូលដ្ឋានរបស់អ្នកដែលមានស្រាប់](#ជម្រើស-គ៖-ប្រើការតំឡើងមូលដ្ឋានរបស់អ្នកដែលមានស្រាប់)
- [ជំហានទី 2៖ ផ្តល់សេវា Azure AI Foundry](#ជំហានទី-2៖-ផ្តល់សេវា-azure-ai-foundry)
- [ជំហានទី 3៖ សាកល្បងការតំឡើងរបស់អ្នក](#ជំហានទី-3៖-សាកល្បងការតំឡើងរបស់អ្នក)
- [ការដោះស្រាយបញ្ហា](#ការដោះស្រាយបញ្ហា)
- [សង្ខេប](#សង្ខេប)
- [ជំហានបន្ទាប់](#ជំហានបន្ទាប់)

## ប소개

ជំពូកនេះនឹងណែនាំអ្នកពីរបៀបតំឡើងបរិយាកាសអភិវឌ្ឍ។ យើងនឹងប្រើ **Azure AI Foundry** សម្រាប់ម៉ូឌែលនៅក្នុងหลักสูตรនេះ។ អ្នកផ្តល់សេវាម៉ូឌែលជាកូដជាមួយ Bicep និង Azure Developer CLI (`azd`), បន្ទាប់មកភ្ជាប់ជាមួយ **keyless authentication** (Microsoft Entra ID) — គ្មានការចម្លងឬរលាយ API keys ទេ។

**គ្មានការតំឡើងក្នុងថាសមូលដ្ឋានចាំបាច់ទេ!** អ្នកអាចប្រើ GitHub Codespaces ដែលផ្តល់បរិយាកាសអភិវឌ្ឍពេញលេញតាមកម្មវិធីរកមើលរបស់អ្នក ហើយផ្តល់សេវាដល់ Foundry ពីទីនោះ។

យើងប្រើ **Azure AI Foundry** សម្រាប់หลักสูตรនេះព្រោះវា៖  
- **ផ្តល់សេវាជា​កូដ** — តែ `azd up` ធ្វើការតំឡើងគណនី និងការដាក់​ប្រើម៉ូឌែល  
- **គ្មានកូនសោ** — ផ្ទៀងផ្ទាត់ជាមួយការចុះឈ្មោះ Azure របស់អ្នក ឬ managed identity  
- **មានស្រាប់សម្រាប់ផលិតកម្ម** — កូដដូចគ្នារត់ដំណើរការ​នៅក្នុងថាស និងនៅ Azure  
- **អាចប្តូរបាន** — ប្តូរស្ដុកម៉ូឌែលតាមរយៈការផ្លាស់ប្តូរឈ្មោះការដាក់ប្រើ មិនមែនកូដរបស់អ្នកទេ

> **ចំណាំ:** ការដាក់ប្រើ Azure AI Foundry គិតតាមសញ្ញាតokens (បង់ប្រាក់តាមការប្រើប្រាស់)។ មើល [មគ្គុទេសក៍តំឡើង Azure AI Foundry](getting-started-azure-openai.md) សម្រាប់ព័ត៌មានអំពីការផ្តល់សេវា តំបន់ និងថ្លៃឈ្នួល។

## ជំហានទី 1៖ តំឡើងបរិយាកាសអភិវឌ្ឍរបស់អ្នក

<a name="quick-start-cloud"></a>

យើងបានបង្កើត dev container ដាក់រចនាសម្ព័ន្ធរួចហើយ ដើម្បីកាត់បន្ថយរយៈពេលតំឡើង និងធានាថាអ្នកមានឧបករណ៍ទាំងអស់ដែលត្រូវការសម្រាប់หลักสูตร Generative AI សម្រាប់ Java នេះ។ ជ្រើសរើសវិធីសាស្ត្រអភិវឌ្ឍដែលអ្នកចូលចិត្ត៖

### ជម្រើសក្នុងការតំឡើងបរិយាកាស៖

#### ជម្រើស ក៖ GitHub Codespaces (ណែនាំ)

**ចាប់ផ្តើមកូដក្នុងរយៈពេល 2 នាទី - គ្មានការតំឡើងក្នុងថាសអោយចាំបាច់!**

1. Fork ទិញ Repository នេះទៅគណនី GitHub របស់អ្នក  
   > **ចំណាំ**: ប្រសិនបើអ្នកចង់កែប្រែកំណត់រចនាសម្ព័ន្ធមូលដ្ឋាន សូមមើល [Dev Container Configuration](../../../.devcontainer/devcontainer.json)  
2. ចុច **Code** → ទៅតារាង **Codespaces** → ចុច **...** → **New with options...**  
3. ប្រើការ default – នេះជួយជ្រើសរើស **Dev container configuration**: **Generative AI Java Development Environment** devcontainer បង្កើតសម្រាប់หลักสูตรនេះ  
4. ចុច **Create codespace**  
5. រំពឹងរយៈពេល ~2 នាទីសម្រាប់បរិយាកាសត្រៀមរួច  
6. បន្តទៅ [ជំហានទី 2៖ ផ្តល់សេវា Azure AI Foundry](#ជំហានទី-2៖-ផ្តល់សេវា-azure-ai-foundry)

<img src="../../../translated_images/km/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: Codespaces submenu" width="50%">

<img src="../../../translated_images/km/image.833552b62eee7766.webp" alt="Screenshot: New with options" width="50%">

<img src="../../../translated_images/km/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: Create codespace options" width="50%">

> **អត្ថប្រយោជន៍នៃ Codespaces**:  
> - គ្មានការតំឡើងក្នុងថាសចាំបាច់  
> - ងាយរួមការងារបានលើឧបករណ៍ណាមួយដែលមានកម្មវិធីរុករកបណ្ដាញ  
> - រៀបចំរួចជាមុនជាមួយឧបករណ៍ និងផ្នែកពឹងផ្អែកទាំងអស់  
> - មានម៉ោងរហូតដល់ 60 ម៉ោងក្នុងមួយខែសម្រាប់គណនីផ្ទាល់ខ្លួន  
> - បរិយាកាសសម្រាប់អ្នករៀនទាំងអស់មានភាពស៊ាំនឹងគ្នា

#### ជម្រើស ខ៖ Local Dev Container

**សម្រាប់អ្នកអភិវឌ្ឍដែលចូលចិត្តអភិវឌ្ឍក្នុងថាសជាមួយ Docker**

1. Fork និង clone Repository នេះទៅក្នុងកុំព្យូទ័រមូលដ្ឋានរបស់អ្នក  
   > **ចំណាំ**៖ ប្រសិនបើអ្នកចង់កែប្រែកំណត់រចនាសម្ព័ន្ធមូលដ្ឋាន សូមមើល [Dev Container Configuration](../../../.devcontainer/devcontainer.json)  
2. ដំឡើង [Docker Desktop](https://www.docker.com/products/docker-desktop/) និង [VS Code](https://code.visualstudio.com/)  
3. ដំឡើង [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) ក្នុង VS Code  
4. បើកថត Repository ក្នុង VS Code  
5. ពេលមានសំណើសុំ ចុច **Reopen in Container** (ឬប្រើ `Ctrl+Shift+P` → "Dev Containers: Reopen in Container")  
6. រង់ចាំ container ត្រូវបានកសាង និងចាប់ផ្តើម  
7. បន្តទៅ [ជំហានទី 2៖ ផ្តល់សេវា Azure AI Foundry](#ជំហានទី-2៖-ផ្តល់សេវា-azure-ai-foundry)

<img src="../../../translated_images/km/devcontainer.21126c9d6de64494.webp" alt="Screenshot: Dev container setup" width="50%">

<img src="../../../translated_images/km/image-3.bf93d533bbc84268.webp" alt="Screenshot: Dev container build complete" width="50%">

#### ជម្រើស គ៖ ប្រើការតំឡើងមូលដ្ឋានរបស់អ្នកដែលមានស្រាប់

**សម្រាប់អ្នកអភិវឌ្ឍដែលមានបរិយាកាស Java រួចមកហើយ**

លក្ខខណ្ឌមុន៖  
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html)  
- [Maven 3.9+](https://maven.apache.org/download.cgi)  
- [VS Code](https://code.visualstudio.com) ឬ IDE ដែលអ្នកចូលចិត្ត

ជំហាន៖  
1. Clone Repository នេះទៅក្នុងកុំព្យូទ័ររបស់អ្នក  
2. បើកគម្រោងក្នុង IDE របស់អ្នក  
3. បន្តទៅ [ជំហានទី 2៖ ផ្តល់សេវា Azure AI Foundry](#ជំហានទី-2៖-ផ្តល់សេវា-azure-ai-foundry)

> **កិច្ច​ប្រឹក្សា​ពិបាក:** ប្រសិនបើអ្នកមានកុំព្យូទ័រសមត្ថភាពទាប ប៉ុន្តាស់ចង់ប្រើ VS Code ក្នុងកុំព្យូទ័រមូលដ្ឋាន អ្នកអាចប្រើ GitHub Codespaces! អ្នកអាចភ្ជាប់ VS Code ក្នុងថាសមូលដ្ឋានរបស់អ្នកទៅ Codespace ដែលផ្ទុកនៅលើបណ្ដាញសម្រាប់ចំណុចល្អបំផុតពីទាំងពីរ។  

<img src="../../../translated_images/km/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: created local devcontainer instance" width="50%">

## ជំហានទី 2៖ ផ្តល់សេវា Azure AI Foundry

ដាក់សេវាម៉ូដែល AI របស់หลักสูตรទៅ Azure AI Foundry ជា code។ ពី root folder នៃ repository៖

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```
  
`azd` នឹងសុំឲ្យបញ្ចូលឈ្មោះបរិយាកាស និងតំបន់, ផ្តល់សេវាគណនី Azure AI Foundry ជាមួយការដាក់ប្រើ `gpt-4o-mini` និង `text-embedding-3-small`, និងសរសេរឆ្លើយតបទៅក្នុងឯកសារ `.env` របស់ឧទាហរណ៍ — ទាំងអស់នេះជាមួយ **keyless** authentication (គ្មាន API keys)។

> **មើលណែនាំពេញលេញ:** សូមមើល [មគ្គុទេសក៍តំឡើង Azure AI Foundry](getting-started-azure-openai.md) សម្រាប់លក្ខខណ្ឌមុន, ជម្រើសដៃគូ (portal), ទិសដៅតំបន់, និងកំណត់សម្គាល់ថ្លៃឈ្នួល/ការសម្អាត។

## ជំហានទី 3៖ សាកល្បងការតំឡើងរបស់អ្នក

ក្រោយពេលម៉ូឌែល Foundry ត្រូវបានផ្តល់សេវា, សាកល្បងការភ្ជាប់ជាមួយកម្មវិធីឧទាហរណ៍នៅក្នុង [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure)។

1. បើកផ្ទាំងបញ្ជា (terminal) ក្នុងបរិយាកាសអភិវឌ្ឍរបស់អ្នក។
2. ទៅកាន់ឧទាហរណ៍:  
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
  
3. ធ្វើឲ្យប្រាកដថាអ្នកបានចុះឈ្មោះចូល (keyless auth ត្រូវការទ័កអ​ន):  
   ```bash
   az login
   ```
   > ប្រសិនបើអ្នកបានរត់ `azd up`, ឯកសារ `.env` ដែលមាន endpoint របស់អ្នកត្រូវបានសរសេរបានរួចហើយ។  
4. បើកកម្មវិធី:  
   ```bash
   mvn clean spring-boot:run
   ```
  
អ្នកគួរតែឃើញចម្លើយពីម៉ូឌែល `gpt-4o-mini`។

### ការយល់ដឹងពីកូដឧទាហរណ៍

ឧទាហរណ៍នៅក្រោម `examples/basic-chat-azure` គឺជា Spring Boot app ដែលប្រើ **Spring AI** ដើម្បីភ្ជាប់ទៅ Azure AI Foundry ជាមួយ keyless authentication។

**កូដនេះធ្វើអ្វី៖**  
- **ភ្ជាប់** ទៅ Azure AI Foundry ដោយប្រើការចុះឈ្មោះ Azure របស់អ្នក (Microsoft Entra ID) — គ្មាន API key  
- **ផ្ញើ** prompt ទៅម៉ូឌែល `gpt-4o-mini`  
- **ទទួល** និងបង្ហាញចម្លើយពី AI  
- **ផ្ទៀងផ្ទាត់** ថាការតំឡើងរបស់អ្នកដំណើរការល្អ

**ភាពពឹងផ្អែកសំខាន់** (នៅក្នុង `pom.xml`):  
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```
  
**ការកំណត់រចនាសម្ព័ន្ធ** (`application.yml`):  
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
  
## សង្ខេប

ល្អណាស់! ឥឡូវអ្នកបានតំឡើងរួចរាល់៖

- ផ្តល់សេវាម៉ូដែល Azure AI Foundry ជា code ជាមួយ Bicep + `azd`  
- រត់បរិយាកាសអភិវឌ្ឍ Java របស់អ្នក (តើវាជា Codespaces, dev containers ឬក្នុងថាស)  
- ភ្ជាប់ទៅ Azure AI Foundry ជាមួយ keyless authentication (Microsoft Entra ID) — គ្មាន API keys  
- សាកល្បងកូដឧទាហរណ៍សូចនាករ រាយការណ៍ទៅម៉ូឌែលរបស់អ្នកបាន

## ជំហានបន្ទាប់

[ជំពូក 3៖ បច្ចេកទេស Generative AI មូលដ្ឋាន](../03-CoreGenerativeAITechniques/README.md)

## ការដោះស្រាយបញ្ហា

មានបញ្ហាដែរទេ? នេះជាបញ្ហាសំរាប់ និងដំណោះស្រាយទូទៅ៖

- **Authentication បរាជ័យ (401/403)?**  
  - រត់ `az login` — authentication គ្មានកូនសោ ដូច្នេះត្រូវបានចុះឈ្មោះចូល  
  - ពិនិត្យមើលថាគណនីរបស់អ្នកមានតួនាទី **Cognitive Services OpenAI User** នៅលើធនធាន  
  - ប្រសិនបើអ្នកទើបផ្តល់សេវា រង់ចាំរយៈពេលមួយនាទីឲ្យតួនាទីត្រូវបានដាក់ចូល

- **មិនអាចឃើញ Maven?**  
  - ប្រសិនបើប្រើ dev containers/Codespaces, Maven គួរត្រូវបានដំឡើងរួច  
  - សម្រាប់ការតំឡើងក្នុងថាស អ្នកត្រូវដំឡើង Java 21+ និង Maven 3.9+  
  - ព្យាយាម `mvn --version` ដើម្បីពិនិត្យការដំឡើង

- **`azd` មិនបានឃើញ ឬការផ្តល់សេវាបរាជ័យ?**  
  - ដំឡើង [Azure Developer CLI](https://aka.ms/azure-dev/install) ហើយរត់ `azd auth login`  
  - ជ្រើសរើសតំបន់ដែលមាន `gpt-4o-mini` (ជា​ឧទាហរណ៍ `eastus2`)  
  - មើល [មគ្គុទេសក៍តំឡើង Azure AI Foundry](getting-started-azure-openai.md) សម្រាប់ព័ត៌មានលម្អិត

- **Dev container មិនចាប់ផ្តើម?**  
  - ប្រាកដថា Docker Desktop កំពុងដំណើរការ (សម្រាប់ការអភិវឌ្ឍក្នុងថាស)  
  - ព្យាយាមកសាង container ម្តងទៀត៖ `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **កំហុសកំចាត់កម្មវិធី?**  
  - ប្រាកដថាអ្នកនៅក្នុងថតត្រឹមត្រូវ៖ `02-SetupDevEnvironment/examples/basic-chat-azure`  
  - ព្យាយាមលាង និងកំចាត់៖ `mvn clean compile`

> **ត្រូវការជំនួយ?**៖ នៅតែមានបញ្ហាទេ? បើកប្រធានបទថ្មីក្នុង repository ហើយយើងនឹងជួយអ្នក។

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ការបដិសេធ**:
ឯកសារនេះត្រូវបានបម្លែងភាសា ដោយប្រើសេវាបម្លែងភាសា AI [Co-op Translator](https://github.com/Azure/co-op-translator)។ ទោះយើងខ្ញុំមានក្តីប្រាថ្នាឱ្យបានច្បាស់លាស់ តែសូមយល់ដឹងថាការបម្លែងដោយស្វ័យប្រវត្តិក៏អាចមានកំហុសឬភាពមិនត្រឹមត្រូវ។ ឯកសារដើមជាភាសាទីតាំងគួរត្រូវបានគេប្រើជាប្រភពច្បាស់លាស់។ សម្រាប់ព័ត៌មានសំខាន់ៗ សូមណែនាំឱ្យប្រើប្រាស់ការប្រែដោយមនុស្សជំនាញ។ យើងខ្ញុំមិនទទួលខុសត្រូវចំពោះការយល់ច្រឡំ ឬការបកស្រាយខុសបន្ទាប់ពីការប្រើប្រាស់ការបម្លែងនេះនោះទេ។
<!-- CO-OP TRANSLATOR DISCLAIMER END -->