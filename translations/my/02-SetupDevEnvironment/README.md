# Java အတွက် Generative AI ဖန်တီးမှုဖွံ့ဖြိုးရေး ပတ်ဝန်းကျင် တပ်ဆင်ခြင်း

> **အဆင်ပြေစွာ စတင်ခြင်း:** AI မော်ဒယ်များကို **Azure AI Foundry** တွင် Bicep + `azd` ဖြင့် ကုဒ်အဖြစ် နည်းနည်း သုံအတွင်း စတင်အသုံးပြုပါ — [Azure AI Foundry Setup Guide](getting-started-azure-openai.md) ကိုကြည့်ပါ။ အတည်ပြုခြင်းမှာ **keyless** (Microsoft Entra ID) ဖြစ်သောကြောင့် API key မလိုအပ်ပါ။

## သင်ထိတွေ့မည့် အကြောင်းအရာများ

- AI application များအတွက် Java ဖွံ့ဖြိုးရေးပတ်ဝန်းကျင် တပ်ဆင်ခြင်း
- သင့်နှစ်သက်ရာ ဖွံ့ဖြိုးရေးပတ်ဝန်းကျင် (Codespaces ဖြင့် cloud-first, အလုပ်လုပ်ဖို့ Dev container, ဒါမှမဟုတ် ဒါယ့်ကိုယ်တိုင် local ဖြင့် စတင်ခြင်း) ရွေးချယ်၍ ချိန်ညှိခြင်း
- သင့် အသင့်တင်ခြင်းကို Azure AI Foundry မော်ဒယ်တစ်ခုနှင့် ချိတ်ဆက်၍ စမ်းသပ်ခြင်း

## အကြောင်းအရာ စဉ်

- [သင်ထိတွေ့မည့် အကြောင်းအရာများ](#သင်ထိတွေ့မည့်-အကြောင်းအရာများ)
- [နိဒါန်း](#နိဒါန်း)
- [အဆင့် ၁: ဖွံ့ဖြိုးရေးပတ်ဝန်းကျင် တပ်ဆင်ခြင်း](#အဆင့်-၁-ဖွံ့ဖြိုးရေး-ပတ်ဝန်းကျင်-တပ်ဆင်ခြင်း)
  - [ရွေးချယ်မှု A: GitHub Codespaces (အကြံပြု)](#ရွေးချယ်မှု-a-github-codespaces-အကြံပြု)
  - [ရွေးချယ်မှု B: Local Dev Container](#ရွေးချယ်မှု-b-local-dev-container)
  - [ရွေးချယ်မှု C: သင့်ရဲ့ ရှိပြီးသား Local တပ်ဆင်ထားမှု အသုံးပြုခြင်း](#ရွေးချယ်မှု-c-သင့်ရဲ့-ရှိပြီးသား-local-တပ်ဆင်မှု-အသုံးပြုခြင်း)
- [အဆင့် ၂: Azure AI Foundry Provision ပြုလုပ်ခြင်း](#အဆင့်-၂-azure-ai-foundry-provision-ပြုလုပ်ခြင်း)
- [အဆင့် ၃: သင့် Setup ကို စမ်းသပ်ခြင်း](#အဆင့်-၃-သင့်-setup-ကို-စမ်းသပ်ခြင်း)
- [ပြဿနာဖြေရှင်းခြင်း](#ပြဿနာဖြေရှင်းခြင်း)
- [အနှစ်ချုပ်](#အနှစ်ချုပ်)
- [နောက်တစ်ဆင့်များ](#နောက်တစ်ဆင့်များ)

## နိဒါန်း

ဤအခန်းတွင် ဖွံ့ဖြိုးရေးပတ်ဝန်းကျင် တပ်ဆင်ခြင်းကို လမ်းညွှန်ပါမည်။ ထိုကိစ္စအားလုံးတွင် **Azure AI Foundry** ကိုများစွာ အသုံးပြုမည်ဖြစ်သည်။ သင်သည် မော်ဒယ်များကို Bicep နှင့် Azure Developer CLI (`azd`) ဖြင့် ကုဒ်အဖြစ် provision ပြုလုပ်ပြီး၊ **keyless authentication** (Microsoft Entra ID) ဖြင့် ချိတ်ဆက်မည်ဖြစ်သည် — API key မလိုအပ်ပါ၊ ကူးယူခြင်း မရှိပါ။

**Local တပ်ဆင်မှု မလိုအပ်ပါ!** GitHub Codespaces ကို သုံးနိုင်ပြီး၊ ၎င်းမှာ browser တစ်ခုထဲတွင် ပြည့်စုံသော ဖွံ့ဖြိုးရေးပတ်ဝန်းကျင် တပ်ဆင်ထားပြီး Foundry ကိုဆိုက်ရောက် provision လုပ်နိုင်ပါသည်။

ဒီသင်တန်းတွင် **Azure AI Foundry** ကို သုံးကြတာ အကြောင်းကတော့-
- **ကုဒ်အဖြစ် provision ပြုလုပ်တယ်** — `azd up` တစ်ကြိမ် run ဖို့သာလိုပြီး အကောင့်နဲ့ မော်ဒယ် deployment များ deploy လုပ်ပေးတယ်
- **keyless authentication** — Azure သို့ sign-in ဖို့ သို့မဟုတ် managed identity နဲ့ authenticate လုပ်တယ်
- **ပရော်ဒပ်ရှင်း အသင့်ဖြစ်** — ဒါကို local မှာ run ပေးနိုင်တယ်၊ Azure မှာ run ပေးနိုင်တယ်
- **ညာဘက်ကောင်းသော ရွေးချယ်စရာ** — မော်ဒယ်များ အစားထိုးချိန်မှာ deployment အမည်ကိုပြောင်းရုံပဲ၊ မိမိကုဒ်ကို မပြောင်းရပါဘူး

> **မှတ်ချက်**: Azure AI Foundry deployments များမှာ token အရေအတွက်အလိုက် ဘားလ်ဖြစ်ကြောင်း (pay-as-you-go) ဖြစ်သည်။ Provisioning, region, နှင့် ကုန်ကျစရိတ် အသေးစိတ်အတွက် [Azure AI Foundry setup guide](getting-started-azure-openai.md) ကိုကြည့်ပါ။

## အဆင့် ၁: ဖွံ့ဖြိုးရေး ပတ်ဝန်းကျင် တပ်ဆင်ခြင်း

<a name="quick-start-cloud"></a>

Generative AI for Java သင်တန်းမှာ လိုအပ်တဲ့ ကိရိယာတွေကို စုံလင်စွာ ထည့်သွင်းထားသော ပရီးကွန်ဖစ်(ရှင်း) ဖွံ့ဖြိုးရေး container တစ်ခုကို ရှာဖွေပြီး သင်၏ ဖွံ့ဖြိုးရေးနည်းလမ်းကို ရွေးချယ်ပါ။

### ပတ်ဝန်းကျင်တပ်ဆင်ခြင်း ရွေးချယ်စရာများ -

#### ရွေးချယ်မှု A: GitHub Codespaces (အကြံပြု)

**၂ မိနစ်အတွင်း ကုဒ်ရေးစတင်ပါ — local တပ်ဆင်မှု မလိုအပ်ပါ!**

1. ဤ repository ကို သင့် GitHub အကောင့်သို့ fork ပြုလုပ်ပါ
   > **မှတ်ချက်**: မူလ config ကို ပြင်ချင်ပါက [Dev Container Configuration](../../../.devcontainer/devcontainer.json) ကို ကြည့်ရှုပါ
2. **Code** → **Codespaces** ဇယား → **...** → **New with options...** ကို နှိပ်ပါ
3. မူကြမ်း သတ်မှတ်ချက်များကို အသုံးချပါ – ၎င်းသည် **Dev container configuration** အဖြစ် **Generative AI Java Development Environment** ကိုရွေးပါ၊ သင်တန်းအတွက် ဖန်တီးထားသော custom devcontainer ဖြစ်ပါသည်
4. **Create codespace** ကို နှိပ်ပါ
5. ပတ်ဝန်းကျင် အသင့်ရှိရန် ~၂ မိနစ်စောင့်ပါ
6. [အဆင့် ၂: Azure AI Foundry Provision ပြုလုပ်ခြင်း](#အဆင့်-၂-azure-ai-foundry-provision-ပြုလုပ်ခြင်း) သို့ ရောက်ရှိပါ

<img src="../../../translated_images/my/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: Codespaces submenu" width="50%">

<img src="../../../translated_images/my/image.833552b62eee7766.webp" alt="Screenshot: New with options" width="50%">

<img src="../../../translated_images/my/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: Create codespace options" width="50%">

> **Codespaces ၏ အားသာချက်များ**:
> - Local တပ်ဆင်မှု မလိုအပ်ပါ
> - ဘရောက်ဇာရှိရာ စက်တစ်လုံးလုံးတွင် အသုံးပြုနိုင်သည်
> - ကိရိယာများနှင့် တိုးချဲ့ထည့်သွင်းမှုများအားလုံး မူကြားပြင်ဆင်ပြီး
> - ကိုယ်ပိုင်အကောင့်များအတွက် တစ်လလျှင် လွတ်လပ်စွာ ၆၀ နာရီ
> - သင်ယူသူအားလုံးအတွက် တည်ကြည့် ပတ်ဝန်းကျင်

#### ရွေးချယ်မှု B: Local Dev Container

**Docker ဖြင့် local development ကိုနှစ်သက်သည့် ဖွံ့ဖြိုးသူများအတွက်**

1. ဤ repository ကို သင့် local ကွန်ပျူတာသို့ fork နှင့် clone ပြုလုပ်ပါ
   > **မှတ်ချက်**: မူလ config ကို ပြင်ချင်ရင် [Dev Container Configuration](../../../.devcontainer/devcontainer.json) ကို ကြည့်ရှုပါ
2. [Docker Desktop](https://www.docker.com/products/docker-desktop/) နှင့် [VS Code](https://code.visualstudio.com/) ကို တပ်ဆင်ပါ
3. VS Code တွင် [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) ကို တပ်ဆင်ပါ
4. Repository ဖိုတာကို VS Code ထဲ၌ ဖွင့်ပါ
5. ဖိတ်ကြားချက်ရလာရင် **Reopen in Container** ကိုနှိပ်ပါ (သို့မဟုတ် `Ctrl+Shift+P` → "Dev Containers: Reopen in Container" ကို အသုံးပြုပါ)
6. Container ရုပ်ပုံဖန်တီးပြီး စတင်မှုကို စောင့်ပါ
7. [အဆင့် ၂: Azure AI Foundry Provision ပြုလုပ်ခြင်း](#အဆင့်-၂-azure-ai-foundry-provision-ပြုလုပ်ခြင်း) သို့ ရောက်ရှိပါ

<img src="../../../translated_images/my/devcontainer.21126c9d6de64494.webp" alt="Screenshot: Dev container setup" width="50%">

<img src="../../../translated_images/my/image-3.bf93d533bbc84268.webp" alt="Screenshot: Dev container build complete" width="50%">

#### ရွေးချယ်မှု C: သင့်ရဲ့ ရှိပြီးသား Local တပ်ဆင်မှု အသုံးပြုခြင်း

**ရှိပြီးသား Java ပတ်ဝန်းကျင်ပေါ်မှာ ဖွံ့ဖြိုးသူများအတွက်**

လိုအပ်ချက်များ-
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html)
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) သို့မဟုတ် မနှစ်သက်ရာ IDE

ခြေလှမ်းများ-
1. ဤ repository ကို သင့် local ကွန်ပျူတာသို့ clone ချပြီး ဖွင့်ပါ
2. Project ကို သင့် IDE တွင် ဖွင့်ပါ
3. [အဆင့် ၂: Azure AI Foundry Provision ပြုလုပ်ခြင်း](#အဆင့်-၂-azure-ai-foundry-provision-ပြုလုပ်ခြင်း) သို့ ရောက်ရှိပါ

> **အကြံပြုချက်**: သင့်စက် specs ต่ำသည်၊ ဒါပေမယ့် VS Code ကို local မှာ အသုံးချချင်တာဖြစ်လျှင် GitHub Codespaces ကို သုံးပါ! ကိုယ့် local VS Code ကို cloud-hosted Codespace နဲ့ ချိတ်ဆက်ပြီး နှစ်ဘက်စလုံး အကောင်းဆုံးကို ရနိုင်ပါတယ်။

<img src="../../../translated_images/my/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: created local devcontainer instance" width="50%">

## အဆင့် ၂: Azure AI Foundry Provision ပြုလုပ်ခြင်း

သင်တန်းအတွက် AI မော်ဒယ်များကို Azure AI Foundry တွင် ကုဒ်အဖြစ် deploy လုပ်ပါ။ Repository ရဲ့ root မှာ:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```


`azd` က environment အမည်နဲ့ region ကို မေးမြန်းပြီး Azure AI Foundry အကောင့် ကို `gpt-4o-mini` နဲ့ `text-embedding-3-small` deployments တွေနဲ့ provision ပြုလုပ်သည်။ နမူနာ `.env` ဖိုင်ထဲမှာ endpoint ကို ရေးသွင်းပေးသည် — အားလုံး keyless authentication ဖြင့် (API key မလိုအပ်ပါ)။

> **အပြည့်အစုံ လမ်းညွှန်ချက်:** [Azure AI Foundry Setup Guide](getting-started-azure-openai.md) မှာ မလိုအပ်သည့် အချက်အလက်များ၊ မန်ဝယ် (portal) နည်းလမ်း၊ region ညွှန်ကြားချက်များ၊ နှင့် ကုန်ကျစရိတ်/ရှင်းလင်းရေးမှတ်စုများ ပါရှိသည်။

## အဆင့် ၃: သင့် Setup ကို စမ်းသပ်ခြင်း

Foundry မော်ဒယ်များ provision ပြီးပါက၊ နမူနာအက်ပ်ကို [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) ထဲမှ စမ်းသပ်ပါ။

1. ဖွံ့ဖြိုးရေး ပတ်ဝန်းကျင် ထဲမှ terminal ကို ဖွင့်ပါ
2. နမူနာလမ်းကြောင်းသို့ သွားပါ
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
3. စစ်ဆေးပါ သင် signed-in ဖြစ်ပါပြီလား (keyless auth သည် token လိုအပ်သည်)-
   ```bash
   az login
   ```
   > သင် `azd up` ကို run လုပ်ပြီးဖြစ်ပါက `.env` ဖိုင်ထဲမှာ endpoint ရေးသားပြီးသားဖြစ်ပါပြီ။
4. အက်ပ်ကို run ပါ-
   ```bash
   mvn clean spring-boot:run
   ```


သင်သည် `gpt-4o-mini` မော်ဒယ်မှ တုံ့ပြန်ချက်တစ်ခုကိုမြင်ရမည်။

### နမူနာကုဒ် နားလည်ခြင်း

`examples/basic-chat-azure` မှာရှိသည့် နမူနာက Spring Boot app တစ်ခုဖြစ်ပြီး Azure AI Foundry နှင့် keyless authentication ဖြင့် ချိတ်ဆက်ရန် **Spring AI** ကို အသုံးပြုထားသည်။

**အဆိုပါကုဒ် စီမံချက်များ:**
- **အာဇူးပေးနေသည်** Azure AI Foundry ကို သင့် Azure sign-in (Microsoft Entra ID) ဖြင့် ချိတ်ဆက်သည် — API key မလိုအပ်ပါ
- **prompt တစ်ခု စာပို့သည်** `gpt-4o-mini` မော်ဒယ်သို့
- **AI ထံမှ တုံ့ပြန်ချက်ကို လက်ခံပြီး ပြသသည်**
- **သင့် setup မှန်ကန်ကြောင်း သက်သေပြသည်**

**အဓိက ပံ့ပိုးမှု** (`pom.xml` မှာ):
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```


**ချိန်ညှိမှု** (`application.yml`):
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


## အနှစ်ချုပ်

ကောင်းပါပြီ! သင့်မှာ လုံးဝ ပြင်ဆင်ပြီးဖြစ်ပါပြီ-

- Azure AI Foundry မော်ဒယ်များကို ကုဒ်ဖြင့် Bicep + `azd` ဖြင့် provision ပြုလုပ်ခြင်း
- သင့် Java ဖွံ့ဖြိုးရေး ပတ်ဝန်းကျင် အလုပ်ထဲမလားခြင်း (Codespaces, dev containers, ဒါမှမဟုတ် local)
- Azure AI Foundry နှင့် keyless authentication (Microsoft Entra ID) ဖြင့် ချိတ်ဆက်မှု—API key မလိုအပ်ပါ
- မော်ဒယ်ကို ပြောဆိုပြီး လွယ်ကူသော နမူနာဖြင့် စစ်ဆေးပြီး အလုပ်လုပ်မှုတိုက်တွန်းမှု

## နောက်တစ်ဆင့်များ

[အခန်း ၃: Core Generative AI Techniques](../03-CoreGenerativeAITechniques/README.md)

## ပြဿနာဖြေရှင်းခြင်း

ပြဿနာရှိပါသလား? အောက်ပါ ပြဿနာများနှင့် ဖြေရှင်းနည်းများကို ကြည့်ပါ-

- **Authentication မအောင်မြင်နေပါသလား (401/403)?**
  - `az login` ကို run ပါ — authentication သည် keyless ဖြစ်သည်၊ သင့်အကောင့်ကို မှန်ကန်စွာ login ခဲ့ထားရမည်
  - သင့် အကောင့်တွင် resource ပေါ်ရှိ **Cognitive Services OpenAI User** သတ်မှတ်ချက် ရှိကြောင်း စစ်ဆေးပါ
  - သင် provision ပြုလုပ်ပြီးကြသည့် မိနစ်အနည်းငယ်စောင့်ပါ၊ role assignment သည် ဖြန့်ဖြူးတော့မည်ဖြစ်သည်

- **Maven ကို မတွေ့မြင်ပါသလား?**
  - dev containers / Codespaces ကို သုံးပါက Maven သည် မျှော်လင့်ထားသလို တပ်ဆင်ထားသည်
  - local တပ်ဆင်မှုအတွက် Java 21+ နှင့် Maven 3.9+ တပ်ဆင်ရေးကို သေချာစွာ စစ်ဆေးပါ
  - `mvn --version` ဖြင့် တပ်ဆင်မှုကို စစ်ဆေးပါ

- **`azd` မတွေ့ရ သို့မဟုတ် provisioning မအောင်မြင်ပါသလား?**
  - [Azure Developer CLI](https://aka.ms/azure-dev/install) ကို တပ်ဆင်ပြီး `azd auth login` ကို run ပါ
  - `gpt-4o-mini` ဝန်ဆောင်မှုရှိတဲ့ region ကို ရွေးချယ်ပါ (ဥပမာ: `eastus2`)
  - [Azure AI Foundry setup guide](getting-started-azure-openai.md) တွင် အသေးစိတ်ဖော်ပြထားသည်

- **Dev container က စတင်မနေပါသလား?**
  - Docker Desktop သည် လည်ပတ်နေကြောင်း သေချာစေရန် (local dev အတွက်)
  - container ကို ပြန်ဖန်တီးရန် ကြိုးစားပါ: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **အက်ပ် ဖွဲ့စည်းမှုနဲ့ ပြဿနာရှိပါသလား?**
  - သင် ၀င်ထားသည့် ဖိုဒါ သို့ `02-SetupDevEnvironment/examples/basic-chat-azure` ဖြစ်ကြောင်း အတည်ပြုပါ
  - သန့်ရှင်း၍ ပြန်ဖွဲ့ရန်: `mvn clean compile`

> **အကူအညီလိုပါသလား?**: ပြဿနာရှိနေဆဲလျှင် repository ထဲတွင် issue တစ်ခုဖွင့်ပါ၊ ကျွန်ုပ်တို့ကူညီပေးပါမည်။

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ပြောကြားချက်**
ဤစာတမ်းကို AI ဘာသာပြန်ဝန်ဆောင်မှု [Co-op Translator](https://github.com/Azure/co-op-translator) အသုံးပြု၍ ဘာသာပြန်ထားပါသည်။ ကျွန်ုပ်တို့သည် တိကျမှန်ကန်မှုအတွက် ကြိုးပမ်းနေသော်လည်း၊ စက်ကိရိယာဘာသာပြန်ခြင်းများတွင် အမှားများ သို့မဟုတ် မှားယွင်းချက်များ ပါဝင်နိုင်ကြောင်း သတိပြုပါရန် လိုအပ်ပါသည်။ မူလစာတမ်းကို မူရင်းဘာသာဖြင့်သာ ယုံကြည်စိတ်ချရသော အချက်အလက်အဖြစ် သတ်မှတ်သင့်သည်။ အရေးကြီးသည့် သတင်းအချက်အလက်များအတွက် ပရော်ဖက်ရှင်နယ် လူသားဘာသာပြန်သူဝန်ဆောင်မှုကို အကြံပြုပါသည်။ ဤဘာသာပြန်ချက်ကို အသုံးပြုခြင်းမှ ဖြစ်ပေါ်လာသော နားလည်မှုကွာခြားမှုများ သို့မဟုတ် မမှန်ကန်သော အသုံးပြုမှုများအတွက် ကျွန်ုပ်တို့ တာဝန်မခံပါ။
<!-- CO-OP TRANSLATOR DISCLAIMER END -->