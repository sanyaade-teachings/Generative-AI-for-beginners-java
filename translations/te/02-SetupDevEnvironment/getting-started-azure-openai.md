# Azure AI Foundry కోసం అభివృద్ధి వాతావరణం సెట్ చేయడం

> ఈ గైడ్ ఈ కోర్సులోని జావా AI యాప్స్ కోసం **Azure AI Foundry** మోడల్స్‌ను **కీలెస్** ధృవీకరణ (Microsoft Entra ID) ఉపయోగించి సెట్ చేస్తుంది — నిర్వహించడానికి API కీలు అవసరం లేదు. టూల్లింగ్ కొత్తదా? [వికాస వాతావరణ గైడ్](./README.md) తో ప్రారంభించండి.

ఈ గైడ్ ఈ కోర్సులోని జావా AI యాప్స్ కోసం **Azure AI Foundry** మోడల్స్‌ను సెట్ చేస్తుంది. మీకు రెండు మార్గాలు ఉన్నాయి:

- **ఎంపిక A — `azd` + బైసప్ తో ప్రావిషనింగ్ (సిఫారసు):** ఒక ఆదేశం ద్వారా Foundry ఖాతా మరియు మోడల్స్ కోడ్ రూపంలో డిప్లాయ్ చేయబడతాయి. ఎటువంటి పోర్టల్ క్లికింగ్ అవసరం లేదు.
- **ఎంపిక B — Azure AI Foundry పోర్టల్‌లో మానవీయంగా వనరులను సృష్టించండి.**

రెండు మార్గాలు కూడా **కీలెస్ ధృవీకరణ** (Microsoft Entra ID) ఉపయోగిస్తాయి — కాపీ చేయడానికి లేదా లీక్ అయ్యే API కీలు ఉండవు.

## విభాగాల పట్టిక

- [ఏమి సృష్టించబడుతుంది](#ఏమి-సృష్టించబడుతుంది)
- [పూర్వాపరాలు](#పూర్వాపరాలు)
- [ఎంపిక A: azd + బైసప్ తో ప్రావిషనింగ్ (సిఫారసు)](#option-a-provision-with-azd--bicep-recommended)
- [ఎంపిక B: వనరులను మానవీయంగా సృష్టించడం](#ఎంపిక-b-వనరులను-మానవీయంగా-సృష్టించడం)
- [మీ వాతావరణాన్ని కాన్ఫిగర్ చేయండి](#మీ-వాతావరణాన్ని-కాన్ఫిగర్-చేయండి)
- [మీ సెటప్‌ని పరీక్షించండి](#మీ-సెటప్‌ని-పరీక్షించండి)
- [తరువాత ఏమిటి?](#తరువాత-ఏమిటి)
- [వనరులు](#వనరులు)
- [అదనపు వనరులు](#అదనపు-వనరులు)

## ఏమి సృష్టించబడుతుంది

[`infra/`](../../../02-SetupDevEnvironment/infra) లోని బైసప్ టెంప్లేట్లు:

- ఒక **Azure AI Foundry** ఖాతాను (`Microsoft.CognitiveServices/accounts`, రకం `AIServices`) ఒక ప్రాజెక్టుతో కలిసి సృష్టిస్తాయి
- ఒక **చాట్** డిప్లాయ్‌మెంట్ — `gpt-4o-mini`
- ఒక **ఎంబెడ్డింగ్** డిప్లాయ్‌మెంట్ — `text-embedding-3-small` (తరువాతి అధ్యాయాల్లో ఉపయోగిస్తారు)
- ఒక **కీలెస్ రోల్ అసైన్‌మెంట్** (`Cognitive Services OpenAI User`), అందువల్ల మీరు కీలు నిర్వహించకుండా `az login` తో సైన్ ఇన్ అవ్వవచ్చు

## పూర్వాపరాలు

- ఒక [Azure సబ్స్క్రిప్షన్](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) మరియు [Maven 3.9+](https://maven.apache.org/download.cgi)

## ఎంపిక A: azd + బైసప్ తో ప్రావిషనింగ్ (సిఫారసు)

`02-SetupDevEnvironment` ఫోల్డర్ నుండి:

```bash
cd 02-SetupDevEnvironment

# సైన్ ఇన్ చేయండి (రెండు టూల్స్ లేక)
azd auth login
az login

# ఫౌండ్రీ ఖాతా + మోడల్ అమన్స్‌కు ఏర్పాట్లు చేయండి
azd up
```
  
`azd` **వాతావరణం పేరు** (ఉదాహరణకి `genai-java`) మరియు **రిజియన్** కోసం అడుగుతుంది. `gpt-4o-mini` మరియు `text-embedding-3-small` అందుబాటులో ఉన్న రీజియన్‌ని ఎంచుకోండి — ఉదాహరణకి `eastus2` లేదా `swedencentral`.

ప్రావిషనింగ్ ముగిసిన తరువాత, azd:

1. [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep) లో నిర్వచించబడిన అన్ని అంశాలను డిప్లాయ్ చేస్తుంది.
2. తరువాత ప్రావిషనింగ్ హుక్ రన్ చేసి, [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) మీరు ఎండ్‌పాయింట్ మరియు డిప్లాయ్‌మెంట్ పేర్లతో (రహస్యాలు లేకుండా) రాస్తుంది.

> **సలహా:** మార్పుల అన్వయించడానికి ఎప్పుడైనా `azd up` మళ్ళీ నడిపించండి. మొత్తం డిలీట్ చేసి ఖర్చును నిలిపేందుకు `azd down` ని ఉపయోగించండి.

సృష్టించబడిన సెట్టింగ్స్ చూడడానికి:

```bash
azd env get-values
```
  
ఇప్పుడు [మీ సెటప్‌ని పరీక్షించండి](#మీ-సెటప్‌ని-పరీక్షించండి) కు దొరుకుద్దాం.

## ఎంపిక B: వనరులను మానవీయంగా సృష్టించడం

పోర్టల్‌ని ప్రాధాన్యం ఇస్తారా? వనరులను మీ చేతితో సృష్టించండి:

1. [Azure AI Foundry పోర్టల్](https://ai.azure.com/) కు వెళ్ళి సైన్ ఇన్ అవ్వండి.
2. **ప్రాజెక్ట్ సృష్టించండి** (ఇది ఒక AI Foundry వనరును కూడా సృష్టిస్తుంది). దీని పేరు `GenAIJava` వంటి ఇవ్వండి.
3. మీ ప్రాజెక్టులో, **Models + endpoints** → **Deploy model** → **Deploy base model** ని తెరవండి.
4. **gpt-4o-mini** (డిప్లాయ్‌మెంట్ పేరు `gpt-4o-mini`) డిప్లాయ్ చేయండి. ఎంబెడ్డింగ్ ఉదాహరణల కోసం **text-embedding-3-small** ని కూడా డిప్లాయ్ చేయండి.
5. **Overview** నుండి **ఎండ్‌పాయింట్** కాపీ చేసుకోండి (ఉదాహరణకి `https://<resource>.openai.azure.com/`).
6. మీకు కీలెస్ యాక్సెస్ ఇవ్వండి: వనరులో **Access control (IAM)** → **Add role assignment** → **Cognitive Services OpenAI User** మీ ఖాతాకు అసైన్ చేయండి.

> **ఏదైనా సమస్య ఉంటే?** [Azure AI Foundry డాక్యుమెంటేషన్](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects) చూడండి.

## మీ వాతావరణాన్ని కాన్ఫిగర్ చేయండి

**మీరు ఎంపిక A (`azd up`) ఉపయోగించి ఉంటే**, మీ సెట్టింగ్స్ ఫైల్ ఇప్పటికే రాయబడ్డాయి — ఎటువంటి కాన్ఫిగరేషన్ అవసరం లేదు. [మీ సెటప్‌ని పరీక్షించండి](#మీ-సెటప్‌ని-పరీక్షించండి) కి వెళ్లండి.

**మీరు ఎంపిక B (మానవీయ) ఉపయోగించి ఉంటే**, ఉదాహరణ `.env` ఫైల్‌ను మీరు మీరూ సృష్టించండి:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```
  
మీ ఎండ్‌పాయింట్ తో `.env`ను సవరించండి (కీ లేదు — ధృవీకరణ కీలెస్):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```
  
> **భద్రతా గమనిక:** నిల్వ చేయవలసిన API కీ లేదు. మీరు Microsoft Entra ID ద్వారా `az login` (লোকల్) లేదా మేనేజ్డ్ ఐడెంటిటీ (Azureలో) తో ధృవీకరించబడతారు. `.env` ఫైల్‌లో కేవలం రహస్యంలేని సెట్టింగ్స్ ఉంటాయి మరియు ఇది `.gitignore` ద్వారా కవర్ చేయబడింది.

## మీ సెటప్‌ని పరీక్షించండి

కీలెస్ ధృవీకరణ టోకెన్ పొందగలిగేందుకు మీరు సైన్ ఇన్ అయి ఉండాలి, ఆపై ఉదాహరణను నడుపండి:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # మీరు ఇప్పటికే సైన్ ఇన్ కాలేదైతే
mvn clean spring-boot:run
```
  
`gpt-4o-mini` మోడల్ నుండి స్పందన కనపడాలి!

> **VS Code వినియోగదారులు:** నడిపేందుకు `F5` ఒత్తండి. యాప్ మీ `.env` ని ఆటోమేటిక్ లోడ్ చేస్తుంది.

> **పూర్తి ఉదాహరణ:** వివరాలు మరియు సమస్య పరిష్కారానికి [Azure AI Foundry తో బెసిక్ చాట్ ఉదాహరణ](./examples/basic-chat-azure/README.md) చూడండి.

## తరువాత ఏమిటి?

**సెటప్ పూర్తయింది!** మీరు ఇప్పుడు కలిగి ఉన్నారు:  
- `gpt-4o-mini` మరియు `text-embedding-3-small` తో Azure AI Foundry డిప్లాయ్ చేయబడింది  
- కీలెస్ ధృవీకరణ (Microsoft Entra ID) — నిర్వహించాల్సిన కీలు లేవు  
- మీ ఎండ్‌పాయింట్ మరియు డిప్లాయ్‌మెంట్ పేర్లతో ఒక లోకల్ `.env`  
- జావా అభివృద్ధి వాతావరణం సిద్ధంగా ఉంది  

**కొనసాగండి** [అధ్యాయం 3: బేసిక్ జనరేటివ్ AI సాంకేతికతలు](../03-CoreGenerativeAITechniques/README.md) నుండి AI యాప్స్ నిర్మించడానికి!

## వనరులు

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Microsoft Entra ID తో కీలెస్ ధృవీకరణ](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry డాక్యుమెంటేషన్](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI డాక్యుమెంటేషన్](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## అదనపు వనరులు

- [VS Code డౌన్లోడ్](https://code.visualstudio.com/Download)
- [Docker Desktop పొందండి](https://www.docker.com/products/docker-desktop)
- [డెవ్ కంటైనర్ కాన్ఫిగరేషన్](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**అస్వీకరణ**:
ఈ పత్రం AI అనువాద సేవ [Co-op Translator](https://github.com/Azure/co-op-translator) ఉపయోగించి అనువదించబడింది. మేము ఖచ్చితత్వానికి ప్రయత్నిస్తున్నప్పటికీ, ఆటోమేటెడ్ అనువాదాలు తప్పులు లేదా అసమగ్రతలను కలిగి ఉండవచ్చు. దాని స్వదేశ భాషలో ఉన్న అసలు పత్రాన్ని అధికారం కలిగిన మూలంగా పరిగణించాలి. కీలకమైన సమాచారం కోసం, ప్రొఫెషనల్ మానవ అనువాదాన్ని సిఫారసు చేస్తాము. ఈ అనువాదం ఉపయోగం వల్ల కలిగే ఏవైనా అపార్థాలు లేదా తప్పుదారులు కోసం మేము బాధ్యత వహించము.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->