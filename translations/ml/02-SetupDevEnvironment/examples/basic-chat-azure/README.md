# ആസ്യൂർ AI ഫൗണ്ടറിയുമായി അടിസ്ഥാന ചാറ്റ് - തുടക്കം മുതൽ അവസാനം വരെ ഉദാഹരണം

ഈ ഉദാഹരണം ഒരു ലളിതമായ Spring Boot അപ്ലിക്കേഷനാണ്, അതു **ആസ്യൂർ AI ഫൗണ്ടറി** മോഡലുമായി **കീലെസ് ഓതന്റിക്കേഷൻ** (Microsoft Entra ID) ഉപയോഗിച്ച് കണക്റ്റ് ചെയ്ത് നിങ്ങളുടെ സെറ്റപ്പ് പരിശോധിക്കുന്നു. ഇത് Spring AI യുടെ `ChatClient` ഉപയോഗിക്കുന്നു.

## ഉള്ളടക്ക പട്ടിക

- [ആവശ്യമായ вещകൾ](#ആവശ്യമായ-вещകൾ)
- [ശീഘ്രം ആരംഭിക്കൽ](#ശീഘ്രം-ആരംഭിക്കൽ)
- [ഓതന്റിക്കേഷൻ എങ്ങനെ പ്രവർത്തിക്കുന്നു](#ഓതന്റിക്കേഷൻ-എങ്ങനെ-പ്രവർത്തിക്കുന്നു)
- [അപ്ലിക്കേഷൻ പ്രവർത്തിപ്പിക്കൽ](#അപ്ലിക്കേഷൻ-പ്രവർത്തിപ്പിക്കൽ)
  - [Maven ഉപയോഗിച്ച്](#maven-ഉപയോഗിച്ച്)
  - [VS Code ഉപയോഗിച്ച്](#vs-code-ഉപയോഗിച്ച്)
  - [പ്രതീക്ഷിക്കുന്ന ഔട്ട്പുട്ട്](#പ്രതീക്ഷിക്കുന്ന-ഔട്ട്പുട്ട്)
- [കോൺഫിഗറേഷൻ റഫറൻസ്](#കോൺഫിഗറേഷൻ-റഫറൻസ്)
  - [പരിസ്ഥിതി വേരിയബിളുകൾ](#പരിസ്ഥിതി-വേരിയബിളുകൾ)
  - [സ്പ്രിംഗ് കോൺഫിഗറേഷൻ](#സ്പ്രിംഗ്-കോൺഫിഗറേഷൻ)
- [പ്രശ്നപരിഹാരം](#പ്രശ്നപരിഹാരം)
  - [സാധാരണ തകരാർ](#സാധാരണ-തകരാർ)
  - [ഡീബഗ് മോഡ്](#ഡീബഗ്-മോഡ്)
- [അടുത്ത ഘട്ടങ്ങൾ](#അടുത്ത-ഘട്ടങ്ങൾ)
- [സ്രോതസ്സ്](#സ്രോതസ്സ്)

## ആവശ്യമായ вещകൾ

ഈ ഉദാഹരണം പ്രവർത്തിപ്പിക്കുന്നതിന് മുമ്പ് ഉറപ്പുവരുത്തുക:

- `gpt-4o-mini` ൽ ഡിപ്ലോയ് ചെയ്ത ആസ്യൂർ AI ഫൗണ്ടറി റിസോഴ്‌സ് — `azd up` ഉപയോഗിച്ച് അല്ലെങ്കിൽ കൈമാറി [Azure AI Foundry സെറ്റ് അപ് ഗൈഡ്](../../getting-started-azure-openai.md) വഴി ക്രമീകരിക്കുക
- ആ റിസോഴ്‌സിലെ **Cognitive Services OpenAI User** പിൻവശം (Bicep ടെംപ്ലേറ്റുകൾ ഇത് നിങ്ങൾക്കായി നിയോഗിക്കുന്നു)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), `az login` ഉപയോഗിച്ച് സൈൻ ഇൻ ചെയ്തിരിക്കണം
- Java 21+ ഒപ്പം Maven 3.9+

> **API കി വേണ്ട** — ഓതന്റിക്കേഷൻ Microsoft Entra ID മുഖേന കീലെസാണ്.

## ശീഘ്രം ആരംഭിക്കൽ

```bash
# 1. പ്രോജക്ടിലേക്ക് നവിഗേറ്റുചെയ്യുക
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. കീലെസ് ഓതന്റിക്കേഷൻ ടോക്കൺ ലഭിക്കാൻ സൈൻറെ ഇൻ ചെയ്യുക
az login

# 3. എന്റ്പോയിന്റ് കോൺഫിഗർചെയ്യുക
#    - നിങ്ങൾ `azd up` പ്രവർത്തിച്ചതെങ്കിൽ, .env നിങ്ങളുവേണ്ടി എഴുതി (ഇത് ഒഴിവാക്കാം).
#    - അല്ലെങ്കിൽ ട็มപ്ലേറ്റ് പകർത്തി AZURE_OPENAI_ENDPOINT സജ്ജമാക്കുക:
cp .env.example .env

# 4. ആപ്ലിക്കേഷൻ പ്രവർത്തിപ്പിക്കുക
mvn spring-boot:run
```

## ഓതന്റിക്കേഷൻ എങ്ങനെ പ്രവർത്തിക്കുന്നു

ഈ ഉദാഹരണം **Microsoft Entra ID** ഉപയോഗിച്ച് ഓതന്റിക്കെയാണ് ചെയ്യുന്നത് — API കി ഇല്ല.

`spring.ai.azure.openai.endpoint` മാത്രം സെറ്റ് ചെയ്താൽ (api-key ഇല്ലാതെ), Spring AI Azure OpenAI ക്ലയന്റ് [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential) ഉപയോഗിച്ച് നിർമിക്കുന്നു. ആ ക്രെഡൻഷ്യൽ താങ്കളുടെ ലോക്കൽ `az login` സെഷനിൽ നിന്ന് അല്ലെങ്കിൽ ആസ്യൂറിൽ പ്രവർത്തിക്കുമ്പോൾ മാനേജ്ഡ് ഐഡന്റിറ്റിയിൽ നിന്ന് ടോക്കൺ കണ്ടെത്തും — അതിനാൽ ഒരേ കോഡ് രണ്ടു വശത്തും മാറ്റങ്ങൾ ഇല്ലാതെ പ്രവർത്തിക്കും.

## അപ്ലിക്കേഷൻ പ്രവർത്തിപ്പിക്കൽ

### Maven ഉപയോഗിച്ച്

```bash
mvn spring-boot:run
```

### VS Code ഉപയോഗിച്ച്

1. VS Codeൽ പ്രൊജക്റ്റ് തുറക്കുക
2. `F5` അമർത്തുക അല്ലെങ്കിൽ "Run and Debug" പാനൽ ഉപയോഗിക്കുക
3. "Spring Boot-BasicChatApplication" കോൺഫിഗറേഷൻ തിരഞ്ഞെടുക്കുക

> **നോട്ടിസ്**: VS Code കോൺഫിഗറേഷൻ നിങ്ങളുടെ .env ഫയൽ സ്വയം ലോഡ് ചെയ്യുന്നു

### പ്രതീക്ഷിക്കുന്ന ഔട്ട്പുട്ട്

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

## കോൺഫിഗറേഷൻ റഫറൻസ്

### പരിസ്ഥിതി വേരിയബിളുകൾ

| വേരിയബിള് | വിവരണം | ആവശ്യമാണ് | ഉദാഹരണം |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | ഫൗണ്ടറി (ആസ്യൂർ OpenAI) എന്റ്പോയിന്റ് URL | ഇൻ | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | ചാറ്റ് മോഡൽ ഡിപ്ലോയ്മെന്റ് നാമം | ഇല്ല | `gpt-4o-mini` (ഡീഫോൾട്ട്) |

> API കി വേരിയബിള് **ഇല്ല** — ഓതന്റിക്കേഷൻ കിലെസാണ് (Microsoft Entra ID `az login` മുഖേന).

### സ്പ്രിംഗ് കോൺഫിഗറേഷൻ

`application.yml` ഫയൽ കേന്ദ്രീകരിക്കുന്നു:
- **എന്റ്പോയിന്റ്**: `${AZURE_OPENAI_ENDPOINT}` - പരിസ്ഥിതി വേരിയബിളിൽ നിന്ന്
- **ഡിപ്ലോയ്മെന്റ്**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - പരിസ്ഥിതി വേരിയബിളിൽ നിന്നുള്ള എന്നിവാശ്രയത്തോടെ
- **ഓത്**: കീലെസ് — `api-key` ഇല്ലാതെ Spring AI `DefaultAzureCredential` ഉപയോഗിക്കുന്നു
- **ടെംപറേച്ചർ**: `0.7` - സൃഷ്ടിപരമായ ക്രമീകരണം (0.0 = നിർണ്ണായകമായ, 1.0 = സൃഷ്ടിപരമായ)
- **മാക്സ് ടോക്കൺസ്**: `500` - പരമാവധി പ്രതികരണ ദൈർഘ്യം

## പ്രശ്നപരിഹാരം

### സാധാരണ തകരാർ

<details>
<summary><strong>പിശക്: 401 / "PermissionDenied" / ടോക്കൺ പിശകുകൾ</strong></summary>

- `az login` നടത്തുക — കീലെസ് ഓത് ടോക്കൺ ലഭിക്കാൻ സജീവമായ സൈൻഇൻ ആവശ്യമുണ്ട്
- നിങ്ങളുടെ അക്കൗണ്ടിന് ആ റിസോഴ്‌സിൽ **Cognitive Services OpenAI User** റോളുണ്ട് എന്ന് സ്ഥിരീകരിക്കുക
- റോളിന് കഴിഞ്ഞു കൃത്യമായി പ്രചരിക്കാൻ ഒരാഴ്‌ച കാത്തിരിക്കുക
- ശരിയായ ടെനന്റ്/സബ്‌സ്‌ക്രിപ്ഷനിൽ എടുത്തിരിക്കുന്നത് ഉറപ്പാക്കുക (`az account show`)
</details>

<details>
<summary><strong>പിശക്: "The endpoint is not valid" / കണക്ഷൻ പിശകുകൾ</strong></summary>

- `AZURE_OPENAI_ENDPOINT` പൂർണപ്പെട്ട ബേസ്സ് URL ആയിരിക്കണം (ഉദാ: `https://your-resource.openai.azure.com/`)
- ട്രെയിലിംഗ് സ്ലാഷ് സ്ഥിരം വെയ്ക്കുന്നുണ്ടോ പരിശോധന നടത്തുക
- എൻഡ്‌പോയിന്റ് provision ചെയ്ത റിസോഴ്‌സുമായി ഒത്തിരിക്കണമെന്ന് സ്ഥിരീകരിക്കുക (`azd env get-values`)
</details>

<details>
<summary><strong>പിശക്: "The deployment was not found"</strong></summary>

- `AZURE_OPENAI_DEPLOYMENT` ആസ്യൂറിൽ ഉള്ള ഒരു ഡിപ്ലോയ്മെന്റ് നാമവുമായി പൊരുത്തപ്പെടുന്നുണ്ടോ പരിശോധിക്കുക
- മോഡൽ വിജയകരമായി ഡിപ്ലോയ് ചെയ്തതും സജീവമായതും ആയിരിക്കണം
- ഡീഫോൾട്ട് ഡിപ്ലോയ്മെന്റ് നാമം `gpt-4o-mini` ആണ്
</details>

<details>
<summary><strong>VS Code: പരിസ്ഥിതി വേരിയബിളുകൾ ലോഡ് ആവുന്നില്ല</strong></summary>

- `.env` ഫയൽ പ്രൊജക്റ്റ് റൂട്ട് ഡയറക്ടറിയിൽ (pom.xml ഉം ഉള്ള ലെവലിൽ) ഉണ്ടെന്ന് ഉറപ്പാക്കുക
- VS Code സംയോജിത ടെർമിനലിൽ `mvn spring-boot:run` പരീക്ഷിക്കുക
- VS Code ജാവ എക്സ്റ്റൻഷൻ ശരിയായി ഇൻസ്റ്റാൾ ചെയ്തിട്ടുണ്ടെന്ന് പരിശോധിക്കുക
</details>

### ഡീബഗ് മോഡ്

വിശദമായ ലോക്കിംഗ് സജ്ജമാക്കാൻ `application.yml`ൽ ഈ വരികൾ അനകൂലമാക്കുക:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## അടുത്ത ഘട്ടങ്ങൾ

**സജ്ജീകരണം പൂർത്തിയായി!** നിങ്ങളുടെ പഠനയാത്ര തുടർന്നു:

[അധ്യാപന ഭാഗം 3: കോർ ജനറേറ്റീവ് AI സാങ്കേതിക വിദ്യകൾ](../../../03-CoreGenerativeAITechniques/README.md)

## സ്രോതസ്സ്

- [Spring AI ആസ്യൂർ OpenAI ഡോക്യുമെന്റേഷൻ](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Microsoft Entra ID ഉപയോഗിച്ച് കീലെസ് ഓതന്റിക്കേഷൻ](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [ആസ്യൂർ AI ഫൗണ്ടറി പോർട്ടൽ](https://ai.azure.com/)
- [ആസ്യൂർ AI ഫൗണ്ടറി ഡോക്യുമെന്റേഷൻ](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**അറിയിപ്പ്**:
ഈ രേഖ AI പരിഭാഷാ സേവനം [Co-op Translator](https://github.com/Azure/co-op-translator) ഉപയോഗിച്ച് പരിഭാഷപ്പെടുത്തിയതാണ്. ഞങ്ങൾ കൃത്യതയ്ക്കായി ശ്രമിക്കുന്നുവെങ്കിലും, ഓട്ടോമേറ്റഡ് പരിഭാഷകളിൽ പിഴവുകൾ അല്ലെങ്കിൽ തെറ്റായ വിവരങ്ങൾ ഉണ്ടാകാൻ സാധ്യതയുണ്ട്. അതിന്റെ സ്വാഭാവിക ഭാഷയിലുള്ള അസൽ രേഖയാണ് പ്രാമാണികമായ ഉറവിടമായി പരിഗണിക്കേണ്ടത്. നിർണായകമായ വിവരങ്ങൾക്ക്, പ്രൊഫഷണൽ മനുഷ്യ പരിഭാഷ ശുപാർശ ചെയ്യുന്നു. ഈ പരിഭാഷ ഉപയോഗിച്ച് ഉണ്ടാകുന്ന തെറ്റിദ്ധാരണകൾ അല്ലെങ്കിൽ തെറ്റായ വ്യാഖ്യാനങ്ങൾക്കായി ഞങ്ങൾ ഉത്തരവാദികളല്ല.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->