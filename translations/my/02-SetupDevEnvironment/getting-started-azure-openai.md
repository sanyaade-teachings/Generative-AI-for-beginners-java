# Azure AI Foundry အတွက် ဖွံ့ဖြိုးရေးပတ်ဝန်းကျင် တပ်ဆင်ခြင်း

> ဒီလမ်းညွှန်က ဒီသင်ခန်းစာရှိ Java AI apps တွေအတွက် **Azure AI Foundry** မော်ဒယ်တွေကို **keyless** authentication (Microsoft Entra ID) ကို အသုံးပြုပြီး တပ်ဆင်ပေးမှာဖြစ်ပါတယ် — API key မလိုအပ်ပါဘူး။ အသစ်စတင်အသုံးပြုသူလား? [ဖွံ့ဖြိုးမှုပတ်ဝန်းကျင်လမ်းညွှန်](./README.md) ကနေ စတင်ပါ။

ဒီလမ်းညွှန်က ဒီသင်ခန်းစာရှိ Java AI apps အတွက် **Azure AI Foundry** မော်ဒယ်တွေကို တပ်ဆင်ပေးမှာဖြစ်ပါတယ်။ သင်မှာ နှစ်ခုရွေးချယ်စရာရှိပါတယ်-

- **အဆင့် A — `azd` + Bicep ဖြင့် Provision ပေးခြင်း (အကြံပြုချက်):** တစ်ချက် Command ဖြင့် Foundry အကောင့်နှင့် မော်ဒယ်များကို code အနေဖြင့် deploy ပေးသည်။ Portal တွင် နစ်နာစရာမလိုပါ။
- **အဆင့် B — Azure AI Foundry portal မှ resource များကို လက်ဖြင့် ဖန်တီးခြင်း။

နှစ်ခုလုံးအသုံးပြုရာတွင် **keyless authentication** (Microsoft Entra ID) ကိုသာ အသုံးပြုသည်။ API key မကူးယူခြင်း သို့မဟုတ် ဖျက်ဖျက်ခြင်းမရှိပါ။

## အကြောင်းအရာသီးခြား

- [ဘာတွေ ဖန်တီးမလဲ](#ဘာတွေ-ဖန်တီးမလဲ)
- [လိုအပ်ချက်များ](#လိုအပ်ချက်များ)
- [အဆင့် A: azd + Bicep ဖြင့် Provision ပေးခြင်း (အကြံပြုချက်)](#option-a-provision-with-azd--bicep-recommended)
- [အဆင့် B: Resource များကို လက်ဖြင့် ဖန်တီးခြင်း](#အဆင့်-b-resource-များကို-လက်ဖြင့်-ဖန်တီးခြင်း)
- [သင့်ပတ်ဝန်းကျင်ကို အသင့်ဖြစ်အောင် ပြင်ဆင်ခြင်း](#သင့်ပတ်ဝန်းကျင်ကို-အသင့်ဖြစ်အောင်-ပြင်ဆင်ခြင်း)
- [သင့်တပ်ဆင်မှုကို စမ်းသပ်ပါ](#သင့်တပ်ဆင်မှုကို-စမ်းသပ်ပါ)
- [နောက်တစ်ဆင့်မှာ ဘာတွေရှိမလဲ?](#နောက်တစ်ဆင့်မှာ-ဘာတွေရှိမလဲ)
- [အရင်းအမြစ်များ](#အရင်းအမြစ်များ)
- [အပိုအရင်းအမြစ်များ](#အပိုအရင်းအမြစ်များ)

## ဘာတွေ ဖန်တီးမလဲ

[`infra/`](../../../02-SetupDevEnvironment/infra) ထဲရှိ Bicep templates တွေက:

- **Azure AI Foundry** အကောင့် (`Microsoft.CognitiveServices/accounts`, kind `AIServices`) တစ်ခုနဲ့ project တစ်ခုကို ဖန်တီးပေးသည်
- **chat** deployment တစ်ခု — `gpt-4o-mini`
- **embedding** deployment တစ်ခု — `text-embedding-3-small` (နောက်ပိုင်းအခန်းများတွင် အသုံးပြုမည်)
- **keyless လုပ်ထုံးလုပ်နည်း** (`Cognitive Services OpenAI User`) တစ်ခု ပေးပြီး `az login` ဖြင့် login ဝင်နိုင်ပြီး key မပြီးလုပ်နှိပ်ရန် မလိုပါ

## လိုအပ်ချက်များ

- [Azure subscription](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) နှင့် [Maven 3.9+](https://maven.apache.org/download.cgi)

## အဆင့် A: azd + Bicep ဖြင့် Provision ပေးခြင်း (အကြံပြုချက်)

`02-SetupDevEnvironment` ဖိုဒါထဲကနေ:

```bash
cd 02-SetupDevEnvironment

# အကောင့်ဝင်ပါ (ကိရိယာနှစ်ခုလုံး)
azd auth login
az login

# Foundry အကောင့်နှင့် မော်ဒယ်ဖြန့်ချိမှုများကို သတ်မှတ်ပါ
azd up
```

`azd` က **environment name** (ဥပမာ `genai-java`) နဲ့ **region** ကို မေးတယ်။ `gpt-4o-mini` နဲ့ `text-embedding-3-small` ရနိုင်တဲ့ region ကို ရွေးပါ — ဥပမာ `eastus2` သို့မဟုတ် `swedencentral` ဖွင့်နိင်သည့် နေရာ။

Provision ပြီးဆုံးတဲ့အခါ azd က -

1. [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep) ထဲမှာသတ်မှတ်ထားတာအားလုံးကို deploy လုပ်ပေးပါလိမ့်မည်။
2. postprovision hook ကို run လုပ်ပြီး [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) ကို သင့် endpoint နဲ့ deployment နာမည်တွေနဲ့ ရေးထုတ်ပေးပါလိမ့်မယ် (secret မပါပါ).

> **အကြံပြုချက်။** ပြင်ဆင်မှုများကို ဖော်ပြရန် အချိန်တိုင်း `azd up` ကို ပြန်လည် run ပါ။ အားလုံး ဖျက်ရန်နှင့် ကုန်ကျစရိတ် မရှိအောင်ရပ်ရန် `azd down` ကို run ပါ။

ဖန်တီးပေးပြီးသော settings များကို ကြည့်ရန်:

```bash
azd env get-values
```

အခု [သင့်တပ်ဆင်မှုကို စမ်းသပ်ပါ](#သင့်တပ်ဆင်မှုကို-စမ်းသပ်ပါ) ကိုဆက်သွားပါ။

## အဆင့် B: Resource များကို လက်ဖြင့် ဖန်တီးခြင်း

Portal ကို မိမိကြိုက်လား? resource များကို လက်တွေ့ဖန်တီးပါ-

1. [Azure AI Foundry portal](https://ai.azure.com/) သို့ ဝင်ပြီး login ဝင်ပါ။
2. **Project တစ်ခု ဖန်တီးပါ** (ဒါက AI Foundry resource ကိုပါ ဖန်တီးပေးပါလိမ့်မယ်)။ နာမည် `GenAIJava` လိုပေးပါ။
3. သင့် project ထဲမှာ **Models + endpoints** → **Deploy model** → **Deploy base model** ကိုဖွင့်ပါ။
4. **gpt-4o-mini** ကို deploy လုပ်ပါ (deployment name `gpt-4o-mini`)။ embedding ဥပမာများအတွက် **text-embedding-3-small** ကိုလည်း ပြန်လုပ်ပါ။
5. **Overview** ထဲကနေ **endpoint** ကို ကူးယူပါ (ဥပမာ `https://<resource>.openai.azure.com/`)။
6. အကောင့်ကို keyless access ပေးရန် resource အပေါ် **Access control (IAM)** → **Add role assignment** → **Cognitive Services OpenAI User** ကို သတင်းအချက်အလက်ပေးထားသော အကောင့်၌ ချိန်းသတ်ပါ။

> **အကူအညီလိုရင်** [Azure AI Foundry စာရွက်စာတမ်း](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects) ကို ကြည့်ပါ။

## သင့်ပတ်ဝန်းကျင်ကို အသင့်ဖြစ်အောင် ပြင်ဆင်ခြင်း

**Option A (`azd up`) သုံးထားရင်** သင့် settings ဖိုင်ကို အကြောင်းတည်ပြီး ရေးထုတ်ပေးထားပါတယ် — ပြင်ဆင်ရန် မလိုပါ။ [သင့်တပ်ဆင်မှုကို စမ်းသပ်ပါ](#သင့်တပ်ဆင်မှုကို-စမ်းသပ်ပါ) ကို ဆက်သွားပါ။

**Option B (manual) သုံးထားရင်** ဥပမာ `.env` ဖိုင်ကို မိမိကိုယ်တိုင် ဖန်တီးပါ-

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

သင့် endpoint နဲ့ `.env` ကို ပြင်ဆင်ပါ (key မလို — auth က keyless ဖြစ်ပါတယ်) -

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **လုံခြုံရေးမှတ်ချက်။** API key ကို သိမ်းဆည်းထားရန် မလိုပါ။ Microsoft Entra ID ဖြင့် `az login` (ကိုယ်ရုံး) သို့မဟုတ် managed identity (Azure တွင်) တို့ အသုံးပြုကာ authenticate လုပ်သည်။ `.env` ဖိုင်မှာ secret မပါဝင်ဘဲ သာ သတ်မှတ်ချက်များကိုသာ ထည့်ထားပြီး `.gitignore` ဖြင့်ကာကွယ်ထားသည်။

## သင့်တပ်ဆင်မှုကို စမ်းသပ်ပါ

Keyless auth ကို token ရရှိစေရန် သင့်အကောင့်ကို login ဝင်ထားသည်ကို သေချာပါစေ၊ ထို့နောက် ဥပမာကို run ပါ -

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # သင်သည် အကောင့်သို့ မဝင်ထားသေးပါက
mvn clean spring-boot:run
```

`gpt-4o-mini` မော်ဒယ်က စကားပြန် တုံ့ပြန်မှုကို မြင်ရပါလိမ့်မယ်!

> **VS Code အသုံးပြုသူများ။** `F5` ကို နှိပ်၍ run ကြည့်ပါ။ app က သင့် `.env` ကို အလိုအလျောက် load လုပ်ပါသည်။

> **အပြည့်အစုံ ဥပမာ။** အသေးစိတ်နှင့် ပြသနာဖြေရှင်းမှုများအတွက် [Basic Chat with Azure AI Foundry example](./examples/basic-chat-azure/README.md) ကို ကြည့်ပါ။

## နောက်တစ်ဆင့်မှာ ဘာတွေရှိမလဲ?

**တပ်ဆင်ခြင်းပြီးဆုံးပါပြီ!** သင်မှာ

- Azure AI Foundry ဖြင့် `gpt-4o-mini` နှင့် `text-embedding-3-small` ကို deploy ပြီး
- Keyless authentication (Microsoft Entra ID) — key မထိန်းချုပ်ရ
- သင့် endpoint နဲ့ deployment နာမည်ပါ .env ဖိုင်တစ်ခုရှိပြီး
- Java ဖွံ့ဖြိုးရေးပတ်ဝန်းကျင် ပြင်ဆင်ပြီးရှိသည်

**ဆက်လုပ်ရန်** [အခန်း ၃: Core Generative AI Techniques](../03-CoreGenerativeAITechniques/README.md) ကို သွားပြီး AI applications တည်ဆောက်ခြင်း စတင်ပါ!

## အရင်းအမြစ်များ

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Microsoft Entra ID ဖြင့် Keyless authentication](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry စာရွက်စာတမ်း](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI စာရွက်စာတမ်း](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## အပိုအရင်းအမြစ်များ

- [VS Code အပ္ဒေါင်းလုပ်ပါ](https://code.visualstudio.com/Download)
- [Docker Desktop ရယူပါ](https://www.docker.com/products/docker-desktop)
- [Dev Container ဖွဲ့စည်းမှု](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ပြောကြားချက်**
ဤစာတမ်းကို AI ဘာသာပြန်ဝန်ဆောင်မှု [Co-op Translator](https://github.com/Azure/co-op-translator) အသုံးပြု၍ ဘာသာပြန်ထားပါသည်။ ကျွန်ုပ်တို့သည် တိကျမှန်ကန်မှုအတွက် ကြိုးပမ်းနေသော်လည်း၊ စက်ကိရိယာဘာသာပြန်ခြင်းများတွင် အမှားများ သို့မဟုတ် မှားယွင်းချက်များ ပါဝင်နိုင်ကြောင်း သတိပြုပါရန် လိုအပ်ပါသည်။ မူလစာတမ်းကို မူရင်းဘာသာဖြင့်သာ ယုံကြည်စိတ်ချရသော အချက်အလက်အဖြစ် သတ်မှတ်သင့်သည်။ အရေးကြီးသည့် သတင်းအချက်အလက်များအတွက် ပရော်ဖက်ရှင်နယ် လူသားဘာသာပြန်သူဝန်ဆောင်မှုကို အကြံပြုပါသည်။ ဤဘာသာပြန်ချက်ကို အသုံးပြုခြင်းမှ ဖြစ်ပေါ်လာသော နားလည်မှုကွာခြားမှုများ သို့မဟုတ် မမှန်ကန်သော အသုံးပြုမှုများအတွက် ကျွန်ုပ်တို့ တာဝန်မခံပါ။
<!-- CO-OP TRANSLATOR DISCLAIMER END -->