# Azure AI Foundry-ന്റെ ഡെവലപ്പ്മെന്റ് പരിസ്ഥിതി ക്രമീകരിക്കൽ

> ഈ ഗൈഡ് ഈ കോഴ്സിലുള്ള ജാവ AI അപ്ലിക്കേഷനുകൾക്കായി **Azure AI Foundry** മോഡലുകൾ ക്രമീകരിക്കുന്നു, **കീലെസ്** ഓത്തന്റിക്കേഷൻ (Microsoft Entra ID) ഉപയോഗിച്ച് — മാനേജുചെയ്യാൻ API കീകൾ ആവശ്യമില്ല. ടൂളിംഗിൽ പുതിയവനാണോ? [ഡെവലപ്പ്മെന്റ് പരിസ്ഥിതിയുടെ ഗൈഡ്](./README.md) ആരംഭിക്കുക.

ഈ ഗൈഡ് ഈ കോഴ്സിലുള്ള ജാവ AI അപ്ലിക്കേഷനുകൾക്കായി **Azure AI Foundry** മോഡലുകൾ ക്രമീകരിക്കുന്നു. നിങ്ങൾക്കു രണ്ടാമത്തെ മാർഗ്ഗങ്ങളുണ്ട്:

- **ഓപ്ഷൻ A — `azd` + Bicep ഉപയോഗിച്ച് പ്രൊവിഷൻ ചെയ്യൽ (പരാമർശിതം):** ഒരു കമാൻഡിൽ Foundry അക്കൗണ്ടും മോഡലുകളും കോഡായി ഡിപ്ലോയ് ചെയ്യുന്നു. പോർട്ടലിൽ ക്ലിക്ക് ആവശ്യമില്ല.
- **ഓപ്ഷൻ B — Azure AI Foundry പോർട്ടലിൽ മാനുവലായി റിസോഴ്സുകൾ സൃഷ്ടിക്കുക.**

രണ്ടു മാർഗ്ഗങ്ങളും **കീലെസ് ഓത്തന്റിക്കേഷൻ** (Microsoft Entra ID) ഉപയോഗിക്കുന്നു — API കീകൾ കോപ്പി ചെയ്‌തോ ലीकായോ ചെയ്യേണ്ടതില്ല.

## ഉള്ളടക്ക പട്ടിക

- [എന്താണ് സൃഷ്ടിക്കപ്പെടുന്നത്](#എന്താണ്-സൃഷ്ടിക്കപ്പെടുന്നത്)
- [അവശ്യങ്ങൾ](#ആവശ്യമുള്ളവ)
- [ഓപ്ഷൻ A: azd + Bicep ഉപയോഗിച്ച് പ്രൊവിഷൻ (പരാമർശിതം)](#option-a-provision-with-azd--bicep-recommended)
- [ഓപ്ഷൻ B: റിസോഴ്സുകൾ മാനുവലായി സൃഷ്ടിക്കുക](#ഓപ്ഷൻ-b-റിസോഴ്സുകൾ-മാനുവലായി-സൃഷ്ടിക്കുക)
- [നിങ്ങളുടെ പരിസ്ഥിതി ക്രമീകരിക്കുക](#നിങ്ങളുടെ-പരിസ്ഥിതിയെ-ക്രമീകരിക്കുക)
- [നിങ്ങളുടെ ക്രമീകരണം പരിശോധിക്കാം](#നിങ്ങളുടെ-ക്രമീകരണം-പരിശോധിക്കാം)
- [അടുത്തത് എന്ത്?](#അടുത്തത്-എന്ത്)
- [സൊമഗ്രങ്ങൾ](#റിസോഴ്സുകൾ)
- [അധിക സ്രോതസ്സുകൾ](#അധിക-സ്രോതസ്സുകൾ)

## എന്താണ് സൃഷ്ടിക്കപ്പെടുന്നത്

[`infra/`](../../../02-SetupDevEnvironment/infra) ൽ ഉള്ള Bicep ടെംപ്ലേറ്റുകൾ താഴെ പറയുന്നവ പ്രൊവിഷൻ ചെയ്യുന്നു:

- ഒരു **Azure AI Foundry** അക്കൗണ്ട് (`Microsoft.CognitiveServices/accounts`, തരം `AIServices`) ആയുള്ള ഒരു പ്രോജക്ട്
- ഒരു **ചാറ്റ്** ഡിപ്ലോയ്മെന്റ് — `gpt-4o-mini`
- ഒരു **എംബെഡ്ഡിംഗ്** ഡിപ്ലോയ്മെന്റ് — `text-embedding-3-small` (പിന്നീട് അദ്ധ്യായങ്ങളിൽ ഉപയോഗിക്കും)
- ഒരു **കീലെസ് റോളു നിയോഗം** (`Cognitive Services OpenAI User`) നിങ്ങൾക്ക് കീകൾ കൈകാര്യം ചെയ്യാതെ `az login` ഉപയോഗിച്ച് സൈൻ ഇൻ ചെയ്യാൻ

## ആവശ്യമുള്ളവ

- ഒരു [Azure സബ്സ്ക്രിപ്ഷൻ](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) കൂടാതെ [Maven 3.9+](https://maven.apache.org/download.cgi)

## ഓപ്ഷൻ A: azd + Bicep ഉപയോഗിച്ച് പ്രൊവിഷൻ (പരാമർശിതം)

`02-SetupDevEnvironment` ഫോളഡറിൽ നിന്ന്:

```bash
cd 02-SetupDevEnvironment

# സൈൻ ഇൻ ചെയ്യുക (രണ്ടു ടൂളുകളും)
azd auth login
az login

# ഫൗണ്ട്രി അക്കൗണ്ട് + മോഡൽ ഡിപ്ലോയ്മെന്റുകൾ ഒരുക്കുക
azd up
```


`azd` ഒരു **പരിസ്ഥിതി പേര്** (ഉദാഹരണത്തിന് `genai-java`) കൂടാതെ **റിയോൻ** ചോദിക്കും. `gpt-4o-mini` ഒപ്പം `text-embedding-3-small` ലഭ്യമായ റീജിയൻ തിരഞ്ഞെടുക്കുക — ഉദാഹരണത്തിന് `eastus2` അല്ലെങ്കിൽ `swedencentral`.

പ്രൊവിഷനിംഗ് പൂർത്തിയാകുമ്പോൾ, azd:

1. [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep) ൽ നിർവചിച്ച എല്ലാ στοιχείων ഡിപ്ലോയ്ന് ചെയ്യും.
2. ഒരു പോസ്റ്റ് പ്രൊവിഷൻ ഹുക്ക് ഓടിക്കും, നിങ്ങളുടെ എന്റ്പോയിന്റ്, ഡിപ്ലോയ്മെന്റ് നാമങ്ങൾ ഉള്ള [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) എഴുതും (രഹസ്യങ്ങൾ ഇല്ലാതെ).

> **ടിപ്പ്:** മാറ്റങ്ങൾ പ്രയോഗിക്കാൻ എപ്പോഴും `azd up` വീണ്ടും ഓടിക്കാൻ കഴിയും. ഓരോന്നും ഡിലീറ്റ് ചെയ്ത് ചെലവ് അവസാനിപ്പിക്കാൻ `azd down` ഓടിക്കാം.

സൃഷ്ടിച്ച ക്രമീകരണങ്ങൾ കാണാൻ:

```bash
azd env get-values
```


ഇപ്പൊൾ [നിങ്ങളുടെ ക്രമീകരണം പരിശോധിക്കുക](#നിങ്ങളുടെ-ക്രമീകരണം-പരിശോധിക്കാം) എന്നതിലേക്ക് പോവാം.

## ഓപ്ഷൻ B: റിസോഴ്സുകൾ മാനുവലായി സൃഷ്ടിക്കുക

പോർട്ടൽ ഉപയോഗിക്കണോ? റിസോഴ്സുകൾ കൈയാൽ സൃഷ്ടിക്കുക:

1. [Azure AI Foundry പോർട്ടലിൽ](https://ai.azure.com/) ലോഗിൻ ചെയ്യുക.
2. **പ്രോജക്ട് സൃഷ്ടിക്കുക** (ഇത് ഒരു AI Foundry റിസോഴ്സും സൃഷ്ടിക്കും). ഉദാഹരണമായി `GenAIJava` എന്ന പേര് നൽകുക.
3. നിങ്ങളുടെ പ്രോജക്ടിൽ, **Models + endpoints** → **Deploy model** → **Deploy base model** തുറക്കുക.
4. **gpt-4o-mini** ഡിപ്ലോയ് ചെയ്യുക (ഡിപ്ലോയ്മെന്റ് പേര് `gpt-4o-mini`). എംബെഡ്ഡിംഗ് ഉദാഹരണങ്ങൾക്കായി **text-embedding-3-small** ആവശ്യമെങ്കിൽ അതും ആവർത്തിക്കുക.
5. **Overview** ൽ നിന്ന് **endpoint** കോപ്പി ചെയ്യുക (ഉദാഹരണം `https://<resource>.openai.azure.com/`).
6. കീലെസ് ആക്‌സസ് അനുവദിക്കുക: റിസോഴ്സിൽ **Access control (IAM)** → **Add role assignment** → **Cognitive Services OpenAI User** നിങ്ങൾക്ക് നിയോഗിക്കുക.

> **ഇത്തരം പ്രശ്നങ്ങൾ തുടരുന്നു?** [Azure AI Foundry ഡോക്യുമെന്റേഷൻ](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects) കാണുക.

## നിങ്ങളുടെ പരിസ്ഥിതിയെ ക്രമീകരിക്കുക

**ഓപ്ഷൻ A (`azd up`) നിങ്ങൾ ഉപയോഗിച്ചാൽ**, ക്രമീകരണ ഫയൽ നിങ്ങളുടെ വേണ്ടി എഴുതിയിട്ടുണ്ട് — ക്രമീകരണമുണ്ടാക്കേണ്ടതില്ല. [നിങ്ങളുടെ ക്രമീകരണം പരിശോധിക്കുക](#നിങ്ങളുടെ-ക്രമീകരണം-പരിശോധിക്കാം) എന്നതിലേക്ക് പോകൂ.

**ഓപ്ഷൻ B (മാനുവൽ) നിങ്ങൾ ഉപയോഗിച്ചാൽ**, ഉദാഹരണത്തിന്റെ `.env` ഫയൽ സ്വയം സൃഷ്ടിക്കുക:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```


കീ ഇല്ലാത്ത your എന്റ്പോയിന്റ് ഉപയോഗിച്ച് `.env` എഡിറ്റ് ചെയ്യുക:

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```


> **സുരക്ഷാ കുറിപ്പ്:** API കീകളും സൂക്ഷിക്കേണ്ടതില്ല. Microsoft Entra ID വഴി `az login` (ലോകലായി) അല്ലെങ്കിൽ Azureയിൽ മാനേജുചെയ്ത ഐഡന്റിറ്റിയിലൂടെ ഓത്തന്റിക്കേഷൻ നടക്കുന്നു. `.env` ഫയൽ രഹസ്യമല്ലാത്ത ക്രമീകരണങ്ങൾ മാത്രമാണ് അകത്തു, ഇത് `.gitignore` ൽ ഉൾപ്പടെ സംരക്ഷിച്ചിട്ടുണ്ട്.

## നിങ്ങളുടെ ക്രമീകരണം പരിശോധിക്കാം

കീലെസ് ഓത്തന്റിക്കേഷൻ ടോക്കൺ കിട്ടാൻ സൈൻ ഇൻ ചെയ്തിരിക്കുന്നുവെന്ന് ഉറപ്പാക്കുക, ശേഷം ഉദാഹരണം ഓടിക്കുക:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # നിങ്ങൾ ഇതിനകം സൈൻ ഇൻ ചെയ്തിട്ടില്ലെങ്കിൽ
mvn clean spring-boot:run
```


`gpt-4o-mini` മോഡലിൽ നിന്ന് പ്രതികരണം കാണാനാകും!

> **VS Code ഉപയോഗിക്കുന്നവർ:** ഓടിക്കാൻ `F5` അമർത്തുക. അയപ്പ് നിങ്ങളുടെ `.env` സ്വയം ലോഡ് ചെയ്യും.

> **സമ്പൂർണ ഉദാഹരണം:** വിശദീകരണങ്ങൾക്കും പ്രശ്നപരിഹാരത്തിനുമായി [Basic Chat with Azure AI Foundry ഉദാഹരണം](./examples/basic-chat-azure/README.md) കാണുക.

## അടുത്തത് എന്ത്?

**ക്രമീകരണം പൂർത്തിയായി!** ഇപ്പൊഴുള്ളത്:
- `gpt-4o-mini` ഒപ്പം `text-embedding-3-small` ഉപയോഗിച്ച Azure AI Foundry ഡിപ്ലോയ ചെയ്തിട്ടുണ്ട്
- കീകൾ കൈകാര്യം ചെയ്യേണ്ടതില്ലാത്ത ഓത്തന്റിക്കേഷൻ (Microsoft Entra ID)
- നിങ്ങളുടെ എന്റ്പോയിന്റ്, ഡിപ്ലോയ്മെന്റ് പേരുകൾ അടങ്ങിയ ഒരു ലോക്കൽ `.env`
- ജാവ വികസന പരിസ്ഥിതി തയ്യാറായി

**ആരംഭിക്കാൻ** [അദ്ധ്യായം 3: കോർ ജനറേറ്റീവ് AI സാങ്കേതികവിദ്യകൾ](../03-CoreGenerativeAITechniques/README.md) കാണൂ!

## റിസോഴ്സുകൾ

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Microsoft Entra ID ഉപയോഗിച്ച കീലെസ് ഓത്തന്റിക്കേഷൻ](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry ഡോക്യുമെന്റേഷൻ](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI ഡോക്യുമെന്റേഷൻ](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## അധിക സ്രോതസ്സുകൾ

- [VS Code ഡൗൺലോഡ് ചെയ്യുക](https://code.visualstudio.com/Download)
- [Docker Desktop ലഭിക്കുക](https://www.docker.com/products/docker-desktop)
- [Dev Container ക്രമീകരണം](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**അറിയിപ്പ്**:
ഈ രേഖ AI പരിഭാഷാ സേവനം [Co-op Translator](https://github.com/Azure/co-op-translator) ഉപയോഗിച്ച് പരിഭാഷപ്പെടുത്തിയതാണ്. ഞങ്ങൾ കൃത്യതയ്ക്കായി ശ്രമിക്കുന്നുവെങ്കിലും, ഓട്ടോമേറ്റഡ് പരിഭാഷകളിൽ പിഴവുകൾ അല്ലെങ്കിൽ തെറ്റായ വിവരങ്ങൾ ഉണ്ടാകാൻ സാധ്യതയുണ്ട്. അതിന്റെ സ്വാഭാവിക ഭാഷയിലുള്ള അസൽ രേഖയാണ് പ്രാമാണികമായ ഉറവിടമായി പരിഗണിക്കേണ്ടത്. നിർണായകമായ വിവരങ്ങൾക്ക്, പ്രൊഫഷണൽ മനുഷ്യ പരിഭാഷ ശുപാർശ ചെയ്യുന്നു. ഈ പരിഭാഷ ഉപയോഗിച്ച് ഉണ്ടാകുന്ന തെറ്റിദ്ധാരണകൾ അല്ലെങ്കിൽ തെറ്റായ വ്യാഖ്യാനങ്ങൾക്കായി ഞങ്ങൾ ഉത്തരവാദികളല്ല.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->