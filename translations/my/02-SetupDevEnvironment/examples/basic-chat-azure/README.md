# Azure AI Foundry ဖြင့် အခြေခံစကားပြောခြင်း - အစမှအဆုံးနမူနာ

ဤနမူနာသည် Spring Boot လျှောက်လွှာတစ်ခုဖြစ်ပြီး **Azure AI Foundry** မော်ဒယ်နှင့် **keyless authentication** (Microsoft Entra ID) ဖြင့် ချိတ်ဆက်ပြီး သင့်စနစ်ကို စမ်းသပ်သည်။ Spring AI ၏ `ChatClient` ကို အသုံးပြုသည်။

## အကြောင်းအရာ စာရင်း

- [လိုအပ်ချက်များ](#လိုအပ်ချက်များ)
- [အမြန်စတART](#အမြန်စတart)
- [Authentication မည်သို့ အလုပ်လုပ်သည်](#authentication-မည်သို့-အလုပ်လုပ်သည်)
- [လျှောက်လွှာ ပြေးနေခြင်း](#လျှောက်လွှာ-ပြေးနေခြင်း)
  - [Maven ကို အသုံးပြု၍](#maven-ကို-အသုံးပြု၍)
  - [VS Code ကို အသုံးပြု၍](#vs-code-ကို-အသုံးပြု၍)
  - [မျှော်မှန်းထားသည့် အထွက်အာ](#မျှော်မှန်းထားသည့်-အထွက်အာ)
- [ဆက်တင် အညွှန်း](#ဆက်တင်-အညွှန်း)
  - [ပတ်ဝန်းကျင် အပြောင်းအလဲများ](#ပတ်ဝန်းကျင်-အပြောင်းအလဲများ)
  - [Spring ဆက်တင်](#spring-ဆက်တင်)
- [ပြဿနာဖြေရှင်းခြင်း](#ပြဿနာဖြေရှင်းခြင်း)
  - [ပုံမှန် ပြဿနာများ](#ပုံမှန်-ပြဿနာများ)
  - [Debug mode](#debug-mode)
- [နောက်တစ်ခြား လုပ်ဆောင်ရန်များ](#နောက်တစ်ခြား-လုပ်ဆောင်ရန်များ)
- [အရင်းအမြစ်များ](#အရင်းအမြစ်များ)

## လိုအပ်ချက်များ

ဤနမူနာကို လုပ်ဆောင်မီ အောက်ပါများကို သေချာစေရန် -

- `gpt-4o-mini` deployment ပါသည့် Azure AI Foundry အရင်းအမြစ် တစ်ခု — `azd up` ဖြင့် သို့မဟုတ် လက်ဖြင့် [Azure AI Foundry setup guide](../../getting-started-azure-openai.md) မှတစ်ဆင့် တပ်ဆင်ထားရန်
- အဆိုပါ အရင်းအမြစ်တွင် **Cognitive Services OpenAI User** အခန်းကဏ္ဍ ရှိရန် (Bicep template များက သင့်အတွက်သတ်မှတ်ပေးသည်)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli) ကို `az login` ဖြင့် စာရင်းဝင်ပြီး အသုံးပြုထားရန်
- Java 21+ နှင့် Maven 3.9+

> **API key မလိုအပ်ပါ** — authentication သည် Microsoft Entra ID ဖြင့် keyless ဖြစ်သည်။

## အမြန်စတART

```bash
# 1။ ပရောဂျက်သို့ သွားပါ
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2။ keyless auth သည် token ရနိုင်ရန် လက်မှတ်စနစ်ဖြင့် ဝင်ရောက်ပါ
az login

# 3။ endpoint ကို ပြင်ဆင်ပါ
#    - သင် `azd up` ကို စဉ်ဆက်မရိုက်ခဲ့လျှင်၊ .env ကို သင်အတွက် ရေးသားခဲ့သည် (ဤအပိုင်းကို ကျော်ပါ)
#    - မဟုတ်ပါက template ကို ကူးယူပြီး AZURE_OPENAI_ENDPOINT ကို သတ်မှတ်ပါ
cp .env.example .env

# 4။ အက်ပလိကေးရှင်းကို ပြေးပါ
mvn spring-boot:run
```

## Authentication မည်သို့ အလုပ်လုပ်သည်

ဤနမူနာတွင် **Microsoft Entra ID** ဖြင့် authentication လုပ်သည် — API key မရှိပါ။

`spring.ai.azure.openai.endpoint` သာ သတ်မှတ်ပြီး (api-key မပါက) Spring AI သည် [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential) ဖြင့် Azure OpenAI client ကို ဆောက်လုပ်ပါသည်။ ၎င်း credential သည် သင့် `az login` စက်တင်တောင့်မှ သို့မဟုတ် Azure တွင် ပြေးနေစဉ် managed identity မှ token ကို အလိုအလျောက် ရှာဖွေသည် — ထိုကြောင့် တူညီသောကုဒ်သည် ပြောင်းလဲခြင်းမရှိဘဲ နှစ်နေရာစလုံးတွင် အလုပ်လုပ်နိုင်သည်။

## လျှောက်လွှာ ပြေးနေခြင်း

### Maven ကို အသုံးပြု၍

```bash
mvn spring-boot:run
```

### VS Code ကို အသုံးပြု၍

1. VS Code တွင် project ကို ဖွင့်ပါ
2. `F5` ကို နှိပ်ပါ သို့မဟုတ် "Run and Debug" panel ကို အသုံးပြုပါ
3. "Spring Boot-BasicChatApplication" configuration ကို ရွေးချယ်ပါ

> **မှတ်ချက်**: VS Code configuration သည် ကိုယ်တိုင် .env ဖိုင်ကို အလိုအလျောက် load လုပ်ပါသည်

### မျှော်မှန်းထားသည့် အထွက်အာ

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

## ဆက်တင် အညွှန်း

### ပတ်ဝန်းကျင် အပြောင်းအလဲများ

| Variable | ဖော်ပြချက် | လိုအပ်သည် | နမူနာ |
|----------|------------|------------|--------|
| `AZURE_OPENAI_ENDPOINT` | Foundry (Azure OpenAI) endpoint URL | မရှိမဖြစ် | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | စကားပြော မော်ဒယ် deployment နာမည် | မလိုအပ် | `gpt-4o-mini` (default) |

> API key variable မရှိပါ — authentication သည် keyless ဖြစ်သည် (Microsoft Entra ID ဖြင့် `az login` ကို အသုံးပြုသည်)။

### Spring ဆက်တင်

`application.yml` ဖိုင်တွင် အောက်‌ပါများ သတ်မှတ်ထားသည် -
- **Endpoint**: `${AZURE_OPENAI_ENDPOINT}` - ပတ်ဝန်းကျင်အပြောင်းအလဲမှရရှိ
- **Deployment**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - ပတ်ဝန်းကျင်အပြောင်းအလဲမှရရှိပြီး fallback ပါ
- **Auth**: keyless — `api-key` မထည့်ထားလို့ Spring AI သည် `DefaultAzureCredential` ကို အသုံးပြုသည်
- **Temperature**: `0.7` - ဖန်တီးမှု ထိန်းချုပ်မှု (0.0 = စနစ်တကျ, 1.0 = ဖန်တီးမှုအများဆုံး)
- **Max Tokens**: `500` - အပြန်ပြောစာလုံးအများဆုံး

## ပြဿနာဖြေရှင်းခြင်း

### ပုံမှန် ပြဿနာများ

<details>
<summary><strong>Error: 401 / "PermissionDenied" / token အမှားများ</strong></summary>

- `az login` ဖြင့် ပြေးကြည့်ပါ — keyless auth အတွက် လက်ရှိလက်မှတ်ပါသော စာရင်းဝင်မှု လိုအပ်သည်
- သင့်အကောင့်တွင် အဆိုပါ အရင်းအမြစ်ပေါ် **Cognitive Services OpenAI User** အခန်းကဏ္ဍ ရှိကြောင်း အတည်ပြုပါ
- အခန်းကဏ္ဍသတ်မှတ်ပြီး မကြာမီ စောင့်ပါ (တစ်မိနစ်ခန့်)
- သင့် tenant / subscription မှာ မှန်ကန်စွာရှိကြောင်း စစ်ဆေးပါ (`az account show`)
</details>

<details>
<summary><strong>Error: "The endpoint is not valid" / ချိတ်ဆက်မှု အမှားများ</strong></summary>

- `AZURE_OPENAI_ENDPOINT` သည် အပြည့်အစုံ base URL ဖြစ်ကြောင်း (ဥပမာ `https://your-resource.openai.azure.com/`) သေချာစေပါ
- slash အဆုံး (/) အညီ သေချာစစ်ဆေးပါ
- endpoint သည် သင့်ပူဇော်ထားသော အရင်းအမြစ်နှင့် ကိုက်ညီကြောင်း စစ်ဆေးပါ (`azd env get-values`)
</details>

<details>
<summary><strong>Error: "The deployment was not found"</strong></summary>

- `AZURE_OPENAI_DEPLOYMENT` သည် Azure တွင် ရှိသော deployment နာမည်နှင့် ကိုက်ညီကြောင်း စစ်ဆေးပါ
- မော်ဒယ်သည် ကောင်းစွာ တပ်ဆင်ပြီး အသုံးပြုနိုင်ခွင့် ရှိကြောင်း အတည်ပြုပြီးဖြစ်ပါစေ
- မူလ deployment နာမည်မှာ `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: ပတ်ဝန်းကျင် အပြောင်းအလဲများ မတင်ပြီးခြင်း</strong></summary>

- `.env` ဖိုင်သည် project ရင်းမြစ် ဒါနဲ့တူညီတဲ့ directory (pom.xml တိုင်း) တွင် ရှိသည်ကို သေချာစေပါ
- VS Code integrated terminal မှာ `mvn spring-boot:run` ကို စမ်းကြည့်ပါ
- VS Code Java extension က တတ်ပါသည် ကိုအတည်ပြုပါ
</details>

### Debug mode

အသေးစိတ် မှတ်တမ်းတင်ရန် `application.yml` တွင် အောက်ပါလိုင်းများကို uncomment လုပ်ပါ -

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## နောက်တစ်ခြား လုပ်ဆောင်ရန်များ

**တပ်ဆင်မှု ပြီးစီးပါပြီ!** သင်ယူမှု ခရီးကို ဆက်လက် လုပ်ဆောင်ပါ -

[အခန်း ၃: Core Generative AI Techniques](../../../03-CoreGenerativeAITechniques/README.md)

## အရင်းအမြစ်များ

- [Spring AI Azure OpenAI စာတမ်းများ](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Microsoft Entra ID ဖြင့် Keyless authentication](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry Portal](https://ai.azure.com/)
- [Azure AI Foundry စာတမ်းများ](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ပြောကြားချက်**
ဤစာတမ်းကို AI ဘာသာပြန်ဝန်ဆောင်မှု [Co-op Translator](https://github.com/Azure/co-op-translator) အသုံးပြု၍ ဘာသာပြန်ထားပါသည်။ ကျွန်ုပ်တို့သည် တိကျမှန်ကန်မှုအတွက် ကြိုးပမ်းနေသော်လည်း၊ စက်ကိရိယာဘာသာပြန်ခြင်းများတွင် အမှားများ သို့မဟုတ် မှားယွင်းချက်များ ပါဝင်နိုင်ကြောင်း သတိပြုပါရန် လိုအပ်ပါသည်။ မူလစာတမ်းကို မူရင်းဘာသာဖြင့်သာ ယုံကြည်စိတ်ချရသော အချက်အလက်အဖြစ် သတ်မှတ်သင့်သည်။ အရေးကြီးသည့် သတင်းအချက်အလက်များအတွက် ပရော်ဖက်ရှင်နယ် လူသားဘာသာပြန်သူဝန်ဆောင်မှုကို အကြံပြုပါသည်။ ဤဘာသာပြန်ချက်ကို အသုံးပြုခြင်းမှ ဖြစ်ပေါ်လာသော နားလည်မှုကွာခြားမှုများ သို့မဟုတ် မမှန်ကန်သော အသုံးပြုမှုများအတွက် ကျွန်ုပ်တို့ တာဝန်မခံပါ။
<!-- CO-OP TRANSLATOR DISCLAIMER END -->