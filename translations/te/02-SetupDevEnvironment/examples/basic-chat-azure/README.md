# Azure AI Foundry తో ప్రాథమిక చాట్ - ఎండ్ టు ఎండ్ ఉదాహరణ

ఈ ఉదాహరణ కీ లెస్ సాక్ష్యీకరణ (Microsoft Entra ID) ఉపయోగించి **Azure AI Foundry** మోడల్ కు కనెక్ట్ చేసే సింపుల్ Spring Boot అప్లికేషన్ మరియు మీ సెటప్ ని పరీక్షిస్తుంది. ఇది Spring AI యొక్క `ChatClient` ని ఉపయోగిస్తుంది.

## సూచనలు

- [అవసరాలు](#అవసరాలు)
- [త్వరిత ప్రారంభం](#త్వరిత-ప్రారంభం)
- [సాక్ష్యీకరణ ఎలా పనిచేస్తుంది](#సాక్ష్యీకరణ-ఎలా-పనిచేస్తుంది)
- [అప్లికేషన్ ని నడపడం](#అప్లికేషన్-ను-నడపడం)
  - [Maven ఉపయోగించడం](#maven-ఉపయోగించడం)
  - [VS Code ఉపయోగించడం](#vs-code-ఉపయోగించడం)
  - [ఆశించిన అవుట్‌పుట్](#ఆశించిన-అవుట్‌పుట్)
- [కాన్ఫిగరేషను సూచిక](#కాన్ఫిగరేషన్-సూచిక)
  - [పరిసర వేరియబుల్స్](#పరిసర-వేరియబుల్స్)
  - [Spring కాన్ఫిగరేషన్](#spring-కాన్ఫిగరేషన్)
- [గందరగోళ పరిష్కారం](#గందరగోళ-పరిష్కారం)
  - [సాధారణ సమస్యలు](#సాధారణ-సమస్యలు)
  - [డీబగ్ మోడ్](#డీబగ్-మోడ్)
- [తర్వాతి దశలు](#తర్వాతి-దశలు)
- [మెరుగైన వనరులు](#మెరుగైన-వనరులు)

## అవసరాలు

ఈ ఉదాహరణ నడపడానికి ముందుగా కలిగి ఉండాలి:

- `gpt-4o-mini` మోడల్ కోసం ఒక Azure AI Foundry వనరు — దీన్ని `azd up` తో లేదా చేతితో [Azure AI Foundry సెటప్ గైడ్](../../getting-started-azure-openai.md) ద్వారా ప్రొవిజన్ చేయండి
- ఆ వనరు పై **Cognitive Services OpenAI User** పాత్ర (Bicep టెంప్లేట్లు మీరు కోసం నియమిస్తాయి)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), `az login` తో సైన్ ఇన్ అయి ఉండాలి
- Java 21+ మరియు Maven 3.9+ 

> **ఏ API కీ అవసరం లేదు** — సాక్ష్యీకరణ కీ లెస్ గా ఉంటుంది Microsoft Entra ID ద్వారా.

## త్వరిత ప్రారంభం

```bash
# 1. ప్రాజెక్టుకు నావిగేట్ చేయండి
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. కీ్లెస్ ఆథ్ టోకెన్‌ను పొందటానికి సైన్ ఇన్ అవ్వండి
az login

# 3. ఎండ్పాయింట్‌ని రూపొందించండి
#    - మీరు `azd up` ని నడిపితే, .env మీకు రాయబడ్డది (ఇది స్కిప్ చేయండి).
#    - లేకపోతే టెంప్లేట్‌ను కాపీ చేసి AZURE_OPENAI_ENDPOINTను సెట్ చేయండి:
cp .env.example .env

# 4. అప్లికేషన్‌ను నడపండి
mvn spring-boot:run
```

## సాక్ష్యీకరణ ఎలా పనిచేస్తుంది

ఈ ఉదాహరణ **Microsoft Entra ID** తో సాక్ష్యీకరించబడుతుంది — API కీ లేదు.

పేరుతో `spring.ai.azure.openai.endpoint` మాత్రమే సెట్ చేయబడి(api-key లేకుండా), Spring AI Azure OpenAI క్లయింట్ ని [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential) తో సృష్టిస్తుంది. ఆ క్రెడెన్షియల్ ఆటోమేటిగ్గా మీ స్థానికంగా ఉన్న `az login` సెషన్ నుండి లేదా Azure లో నడుస్తున్నప్పుడు ఒక మేనేజ్డ్ ఐడెంటిటీ నుండి టోకెన్ ని పొందుతుంది — కాబట్టి ఒకే కోడ్ రెండిటి ప్లేసులలో కూడా మార్పుల్లేకుండా పనిచేస్తుంది.

## అప్లికేషన్ ను నడపడం

### Maven ఉపయోగించడం

```bash
mvn spring-boot:run
```

### VS Code ఉపయోగించడం

1. ప్రాజెక్ట్ ను VS Code లో తెరవండి
2. `F5` నొక్కండి లేదా "Run and Debug" ప్యానెల్ ఉపయోగించండి
3. "Spring Boot-BasicChatApplication" కాన్ఫిగరేషన్ ఎంచుకోండి

> **గమనిక**: VS Code కాన్ఫిగరేషన్ మీ .env ఫైల్ ను ఆటోమేటీక్గా లోడ్ చేస్తుంది

### ఆశించిన అవుట్‌పుట్

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

## కాన్ఫిగరేషన్ సూచిక

### పరిసర వేరియబుల్స్

| వేరియబుల్ | వివరణ | అవసరం | ఉదాహరణ |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | Foundry (Azure OpenAI) ఎండపాయింట్ URL | అవును | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | చాట్ మోడల్ డిప్లాయ్‌మెంట్ పేరును | కాదు | `gpt-4o-mini` (డిఫాల్ట్) |

> API కీ వేరియబుల్ **లేదు** — సాక్ష్యీకరణ కీ లెస్ (Microsoft Entra ID ద్వారా `az login`).

### Spring కాన్ఫిగరేషన్

`application.yml` ఫైల్ లో కింది కాన్ఫిగరేషన్లు ఉంటాయి:
- **ఎండపాయింట్**: `${AZURE_OPENAI_ENDPOINT}` - పరిసర వేరియబుల్ నుండి
- **డిప్లాయ్‌మెంట్**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - పరిసర వేరియబుల్ నుండి, లేకుంటే డిఫాల్ట్
- **సాక్ష్యీకరణ**: కీ లెస్ — `api-key` సెట్ చేయలేదు, కాబట్టి Spring AI `DefaultAzureCredential` ఉపయోగిస్తుంది
- **తాపం**: `0.7` - సృజనాత్మకత నియంత్రణ (0.0 = స్థిరమైన, 1.0 = సృజనాత్మకమైనది)
- **గరిష్ట టోకెన్లు**: `500` - గరిష్ట ప్రతిస్పందన పొడవు

## గందరగోళ పరిష్కారం

### సాధారణ సమస్యలు

<details>
<summary><strong>లోపం: 401 / "PermissionDenied" / టోకెన్ లోపాలు</strong></summary>

- `az login` నడపండి — కీ లెస్ సాక్ష్యీకరణకు టోకెన్ కోసం యాక్టివ్ సైన్-ఇన్ అవసరం
- మీ ఖాతాకు ఆ వనరుపై **Cognitive Services OpenAI User** పాత్ర ఉన్నదని నిర్ధారించుకోండి
- మీరు పాత్రను అప్పుడే అప్పగించగా అయితే, అది వ్యాప్తి చెందడానికి కొద్దిసేపు వేచి ఉండండి
- మీరు సరైన టెనెంట్/సబ్స్క్రిప్షన్ లో ఉన్నారు అనేదానిని ధృవీకరించుకోండి (`az account show`)
</details>

<details>
<summary><strong>లోపం: "The endpoint is not valid" / కనెక్షన్ లోపాలు</strong></summary>

- `AZURE_OPENAI_ENDPOINT` పూర్తి ఆధార URL అని నిర్ధారించుకోండి (ఉదా. `https://your-resource.openai.azure.com/`)
- ట్రైలింగ్ స్లాష్ సరిగ్గా ఉంది కదా చూసుకోండి
- ఎండపాయింట్ మీ ప్రొవిజన్డ్ వనరుతో సరిపోతుందని ధృవీకరించుకోండి (`azd env get-values`)
</details>

<details>
<summary><strong>లోపం: "The deployment was not found"</strong></summary>

- `AZURE_OPENAI_DEPLOYMENT` ఏజ<A><B>కు డిప్లాయ్‌మెంట్ పేరుతో సరిపోతుందా అని వాటిల్లండి
- మోడల్ విజయవంతంగా డిప్లాయ్ అయిందని మరియు సక్రియంగా ఉందని చూడండి
- డిఫాల్ట్ డిప్లాయ్‌మెంట్ పేరు `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: పరిసర వేరియబుల్స్ లోడ్ కాదు అవడం</strong></summary>

- మీ `.env` ఫైల్ ప్రాజెక్ట్ రూట్ డైరెక్టరీలో ఉండాలి (ప్రమాణంతో `pom.xml` వంటి స్థాయిలో)
- VS Code యొక్క ఇంటిగ్రేటెడ్ టెర్మినల్ లో `mvn spring-boot:run` నడపండి
- VS Code Java ఎక్స్‌టెన్షన్ సరిగా ఇన్‌స్టాల్ అయిందా లేదో చూడండి
</details>

### డీబగ్ మోడ్

వివరమైన లాగింగ్ కోసం, `application.yml` లోని ఈ లైన్లను అన్ కామెంట్ చేయండి:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## తర్వాతి దశలు

**సెట్టప్ పూర్తయింది!** మీ నేర్చుకుంటున్న ప్రయాణం కొనసాగించండి:

[అధ్యాయం 3: కోర్ జనరేటివ్ AI సాంకేతికతలు](../../../03-CoreGenerativeAITechniques/README.md)

## మెరుగైన వనరులు

- [Spring AI Azure OpenAI డాక్యుమెంటేషన్](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Microsoft Entra ID తో కీ లెస్ సాక్ష్యీకరణ](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry పోర్టల్](https://ai.azure.com/)
- [Azure AI Foundry డాక్యుమెంటేషన్](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**అస్వీకరణ**:
ఈ పత్రం AI అనువాద సేవ [Co-op Translator](https://github.com/Azure/co-op-translator) ఉపయోగించి అనువదించబడింది. మేము ఖచ్చితత్వానికి ప్రయత్నిస్తున్నప్పటికీ, ఆటోమేటెడ్ అనువాదాలు తప్పులు లేదా అసమగ్రతలను కలిగి ఉండవచ్చు. దాని స్వదేశ భాషలో ఉన్న అసలు పత్రాన్ని అధికారం కలిగిన మూలంగా పరిగణించాలి. కీలకమైన సమాచారం కోసం, ప్రొఫెషనల్ మానవ అనువాదాన్ని సిఫారసు చేస్తాము. ఈ అనువాదం ఉపయోగం వల్ల కలిగే ఏవైనా అపార్థాలు లేదా తప్పుదారులు కోసం మేము బాధ్యత వహించము.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->