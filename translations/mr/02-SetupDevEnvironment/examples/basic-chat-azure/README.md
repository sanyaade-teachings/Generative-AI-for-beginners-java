# Azure AI Foundry सह मूलभूत चॅट - संपूर्ण उदाहरण

हे उदाहरण एक साधे Spring Boot अनुप्रयोग आहे जे **Azure AI Foundry** मॉडेलशी **कीलेस प्रमाणीकरण** (Microsoft Entra ID) वापरून कनेक्ट होते आणि तुमच्या सेटअपची चाचणी घेते. यामध्ये Spring AI चा `ChatClient` वापरला जातो.

## अनुक्रमणिका

- [पूर्वअटी](#पूर्वअटी)
- [त्वरित प्रारंभ](#त्वरित-प्रारंभ)
- [प्रमाणीकरण कसे कार्य करते](#प्रमाणीकरण-कसे-कार्य-करते)
- [अनुप्रयोग चालविणे](#अनुप्रयोग-चालविणे)
  - [Maven वापरून](#maven-वापरून)
  - [VS Code वापरून](#vs-code-वापरून)
  - [अपेक्षित आउटपुट](#अपेक्षित-आउटपुट)
- [संरचना संदर्भ](#संरचना-संदर्भ)
  - [पर्यावरण चल](#पर्यावरण-चल)
  - [स्प्रिंग संरचना](#स्प्रिंग-संरचना)
- [तक्रार निवारण](#तक्रार-निवारण)
  - [सामान्य समस्या](#सामान्य-समस्या)
  - [डिबग मोड](#डिबग-मोड)
- [पुढील पावले](#पुढील-पावले)
- [साधने](#साधने)

## पूर्वअटी

हे उदाहरण चालविण्यापूर्वी, या बाबींची खात्री करा:

- Azure AI Foundry संसाधन ज्यावर `gpt-4o-mini` तैनाती आहे — ते `azd up` ने किंवा [Azure AI Foundry सेटअप मार्गदर्शिका](../../getting-started-azure-openai.md) द्वारे मॅन्युअली तयार केलेले
- त्या संसाधनावर **Cognitive Services OpenAI User** भूमिका (Bicep टेम्प्लेट्स तुम्हाला ही भूमिका देतात)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli) इन्स्टॉल केलेले आणि `az login` ने साइन इन केलेले
- Java 21+ आणि Maven 3.9+

> **कुठल्याही API कीची गरज नाही** — प्रमाणीकरण Microsoft Entra ID द्वारा कीलेस आहे.

## त्वरित प्रारंभ

```bash
# 1. प्रकल्पाकडे जा
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. एखादा टोकन मिळविण्यासाठी कीलेस ऑथ मध्ये साइन इन करा
az login

# 3. एंडपॉइंट कॉन्फिगर करा
#    - जर तुम्ही `azd up` चालवले असेल, तर .env तुमच्यासाठी लिहिले गेले आहे (हे वगळा).
#    - अन्यथा टेम्पलेट कॉपी करा आणि AZURE_OPENAI_ENDPOINT सेट करा:
cp .env.example .env

# 4. अनुप्रयोग चालवा
mvn spring-boot:run
```

## प्रमाणीकरण कसे कार्य करते

हे उदाहरण **Microsoft Entra ID** सह प्रमाणीकरण करते — API की वापरली जात नाही.

जेव्हा फक्त `spring.ai.azure.openai.endpoint` सेट केलेले असते (आणि api-key नसते), तेव्हा Spring AI Azure OpenAI क्लायंट [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential) वापरून तयार करते. हे प्रमाणपत्र तुमच्या स्थानिक `az login` सत्रातून टोकन शोधते, किंवा Azure मध्ये चालताना व्यवस्थापित ओळखीतून टोकन मिळवते — त्यामुळे एकच कोड वेगवेगळ्या ठिकाणी कोणत्याही बदलाशिवाय चालतो.

## अनुप्रयोग चालविणे

### Maven वापरून

```bash
mvn spring-boot:run
```

### VS Code वापरून

1. VS Code मध्ये प्रोजेक्ट उघडा
2. `F5` दाबा किंवा "Run and Debug" पॅनेल वापरा
3. "Spring Boot-BasicChatApplication" कॉन्फिगरेशन निवडा

> **टीप**: VS Code कॉन्फिगरेशन स्वयंचलितपणे तुमची .env फाइल लोड करते

### अपेक्षित आउटपुट

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

## संरचना संदर्भ

### पर्यावरण चल

| चल | वर्णन | आवश्यक | उदाहरण |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | Foundry (Azure OpenAI) एंडपॉइंट URL | होय | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | चॅट मॉडेल तैनातीचे नाव | नाही | `gpt-4o-mini` (डिफॉल्ट) |

> API की चल **नाही** — प्रमाणीकरण कीलेस आहे (Microsoft Entra ID द्वारे `az login`).

### स्प्रिंग संरचना

`application.yml` फाईलमध्ये या गोष्टी संरचित आहेत:
- **एंडपॉइंट**: `${AZURE_OPENAI_ENDPOINT}` - पर्यावरण चलातून
- **तैनाती**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - पर्यावरण चलातून, जर नसेल तर डीफॉल्ट वापरले जाते
- **प्रमाणीकरण**: कीलेस — `api-key` सेट नाही, त्यामुळे Spring AI `DefaultAzureCredential` वापरते
- **टेम्परेचर**: `0.7` - सर्जनशीलतेचे नियंत्रण (0.0 = निश्चित, 1.0 = सर्जनशील)
- **कमाल टोकन्स**: `500` - कमाल प्रतिसादाची लांबी

## तक्रार निवारण

### सामान्य समस्या

<details>
<summary><strong>त्रुटी: 401 / "PermissionDenied" / टोकन त्रुटी</strong></summary>

- `az login` चालवा — कीलेस प्रमाणीकरणाला टोकन मिळवण्यासाठी सक्रिय साइन-इन आवश्यक आहे
- खात्याकडे त्या संसाधनावर **Cognitive Services OpenAI User** भूमिका आहे का तपासा
- भूमिका नुकतीच दिल्यास, लागू होण्यासाठी थोडा वेळ थांबा
- योग्य टेनंट/सबसक्रिप्शनमध्ये आहात का ते तपासा (`az account show`)
</details>

<details>
<summary><strong>त्रुटी: "एंडपॉइंट वैध नाही" / कनेक्शन समस्या</strong></summary>

- `AZURE_OPENAI_ENDPOINT` पूर्ण बेस URL असल्याची खात्री करा (उदा., `https://your-resource.openai.azure.com/`)
- ट्रेलिंग स्लॅशची सुसंगतता तपासा
- एंडपॉइंट तुमच्या तयार केलेल्या संसाधनाशी जुळते का ते तपासा (`azd env get-values`)
</details>

<details>
<summary><strong>त्रुटी: "तैनाती सापडली नाही"</strong></summary>

- `AZURE_OPENAI_DEPLOYMENT` योग्य तैनातीच्या नावाशी जुळते का तपासा
- मॉडेल यशस्वीपणे तैनात आणि सक्रिय आहे का हे तपासा
- डिफॉल्ट तैनात नाव `gpt-4o-mini` आहे
</details>

<details>
<summary><strong>VS Code: पर्यावरण चल लोड होत नाहीत</strong></summary>

- `.env` फाइल प्रोजेक्ट रूट डायरेक्टरीमध्ये आहे का तपासा (pom.xml प्रमाणेच स्तर)
- VS Code च्या एकत्रित टर्मिनलमध्ये `mvn spring-boot:run` चालवून पाहा
- VS Code Java विस्तार नीट स्थापित आहे का ते तपासा
</details>

### डिबग मोड

सविस्तर लॉगिंग सक्षम करण्यासाठी `application.yml` मधील हे ओळी अनकमेंट करा:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## पुढील पावले

**सेटअप पूर्ण!** तुमचा अध्ययन प्रवास सुरू ठेवा:

[अध्याय 3: मुख्य सर्जनशील AI तंत्रे](../../../03-CoreGenerativeAITechniques/README.md)

## साधने

- [Spring AI Azure OpenAI दस्तऐवज](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Microsoft Entra ID सह कीलेस प्रमाणीकरण](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry पोर्टल](https://ai.azure.com/)
- [Azure AI Foundry दस्तऐवज](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**अस्वीकरण**:
हा दस्तऐवज AI भाषांतर सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) चा वापर करून अनुवादित केला आहे. जरी आम्ही अचूकतेसाठी प्रयत्न करतो, तरी कृपया लक्षात घ्या की स्वयंचलित भाषांतरांमध्ये त्रुटी किंवा अचूकतेची कमतरता असू शकते. मूळ दस्तऐवज त्याच्या मूळ भाषेत अधिकृत स्रोत मानला पाहिजे. महत्त्वाची माहिती असल्यास, व्यावसायिक मानवी भाषांतराची शिफारस केली जाते. या भाषांतराच्या वापरामुळे उद्भवणाऱ्या कोणत्याही गैरसमज किंवा चुकीच्या अर्थलावणीसाठी आम्ही जबाबदार नाही.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->