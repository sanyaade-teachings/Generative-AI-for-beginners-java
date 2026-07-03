# Azure AI Foundry साठी विकास वातावरण सेट करणे

> हा मार्गदर्शक या कोर्समधील Java AI अ‍ॅपसाठी **Azure AI Foundry** मॉडेल सेट करतो, **कीलेस** प्रमाणीकरण (Microsoft Entra ID) वापरून — व्यवस्थापित करण्यासाठी कोणतेही API की नाहीत. टूलिंगबाबत नवीन आहात? सुरू करा [विकास वातावरण मार्गदर्शक](./README.md) पासून.

हा मार्गदर्शक या कोर्समधील Java AI अ‍ॅपसाठी **Azure AI Foundry** मॉडेल सेट करतो. तुमच्याकडे दोन पर्याय आहेत:

- **पर्याय A — `azd` + Bicep सोबत वितरण (शिफारसीय):** एका कमांडने Foundry खाते आणि मॉडेल्स कोड म्हणून वितरित करा. कोणतीही पोर्टल क्लिकिंग नाही.
- **पर्याय B — Azure AI Foundry पोर्टलकडून संसाधने स्वतः बनवा.**

दोन्ही मार्गांनी **कीलेस प्रमाणीकरण** (Microsoft Entra ID) वापरले जाते — कोणत्याही API की कॉपी किंवा लीक होत नाहीत.

## अनुक्रमणिका

- [काय तयार होते](#काय-तयार-होते)
- [पूर्व आवश्यक अटी](#पूर्व-आवश्यक-अटी)
- [पर्याय A: azd + Bicep सह वितरण (शिफारसीय)](#option-a-provision-with-azd--bicep-recommended)
- [पर्याय B: संसाधने स्वतः तयार करा](#पर्याय-b-संसाधने-स्वतः-तयार-करा)
- [तुमचे वातावरण कॉन्फिगर करा](#तुमचे-वातावरण-कॉन्फिगर-करा)
- [तुमची सेटअप तपासा](#तुमची-सेटअप-तपासा)
- [पुढे काय?](#पुढे-काय)
- [संसाधने](#संसाधने)
- [अतिरिक्त संसाधने](#अतिरिक्त-संसाधने)

## काय तयार होते

[`infra/`](../../../02-SetupDevEnvironment/infra) मधील Bicep टेम्पलेट:

- एक **Azure AI Foundry** खाते (`Microsoft.CognitiveServices/accounts`, kind `AIServices`) प्रोजेक्टसह तयार करते
- एक **चॅट** वितरण — `gpt-4o-mini`
- एक **एम्बेडिंग** वितरण — `text-embedding-3-small` (नंतरच्या प्रकरणांत वापरले जाते)
- एक **कीलेस भूमिकेचे नियुक्ती** (`Cognitive Services OpenAI User`), ज्यामुळे तुम्ही `az login` वापरून साइन इन करू शकता, की व्यवस्थापित न करता

## पूर्व आवश्यक अटी

- एक [Azure सदस्यता](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) आणि [Maven 3.9+](https://maven.apache.org/download.cgi)

## पर्याय A: azd + Bicep सह वितरण (शिफारसीय)

`02-SetupDevEnvironment` फोल्डरमधून:

```bash
cd 02-SetupDevEnvironment

# साइन इन करा (दोन्ही साधने)
azd auth login
az login

# फाउंड्री खाते + मॉडेल डिप्लॉयमेंट्सची तरतूद करा
azd up
```

`azd` आपल्याला **पर्यावरणाचे नाव** (उदा. `genai-java`) आणि **प्रदेश** विचारतो. `gpt-4o-mini` आणि `text-embedding-3-small` उपलब्ध असलेल्या प्रदेशाचा निवड करा — उदा. `eastus2` किंवा `swedencentral`.

वितरण पूर्ण झाल्यावर, azd:

1. [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep) मध्ये परिभाषित सर्वकाही तैनात करतो.
2. एक पोस्टप्रोव्हिजन हुक चालवतो जे [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) मध्ये तुमचा endpoint आणि वितरण नावे (कोणतीही गुपिते नाहीत) लिहितो.

> **टीप:** बदल लागू करण्यासाठी कोणत्याही वेळी `azd up` पुन्हा चालवा. सर्व काही हटवण्यासाठी आणि खर्च थांबवण्यासाठी `azd down` चालवा.

निर्मित सेटिंग्ज पाहण्यासाठी:

```bash
azd env get-values
```

आता [तुमची सेटअप तपासा](#तुमची-सेटअप-तपासा) येथे जा.

## पर्याय B: संसाधने स्वतः तयार करा

पोर्तल पसंत करता? संसाधने हाताने तयार करा:

1. [Azure AI Foundry पोर्टल](https://ai.azure.com/) ला जा आणि साइन इन करा.
2. **एक प्रोजेक्ट तयार करा** (यामुळे AI Foundry संसाधन देखील तयार होते). त्याला `GenAIJava` सारखे नाव द्या.
3. आपल्या प्रोजेक्टमध्ये, **मॉडेल्स + एंडपॉइंट्स** → **मॉडेल तैनात करा** → **आधार मॉडेल तैनात करा** उघडा.
4. **gpt-4o-mini** तैनात करा (तैनातीचे नाव `gpt-4o-mini`). दाखल्यांसाठी **text-embedding-3-small** साठी पुन्हा करा.
5. **अवलोकन** कडून **एंडपॉइंट** कॉपी करा (उदा. `https://<resource>.openai.azure.com/`).
6. स्वतःला कीलेस प्रवेश द्या: संसाधनावर जा, **Access control (IAM)** → **Add role assignment** → आपले खाते योजनेसाठी **Cognitive Services OpenAI User** ने नियुक्त करा.

> **अजून समस्या आहे?** पाहा [Azure AI Foundry दस्तऐवज](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## तुमचे वातावरण कॉन्फिगर करा

**जर तुम्ही पर्याय A (`azd up`) वापरले असेल**, तर तुमचे सेटिंग्ज फायली आधीच लिहिल्या गेलेल्या आहेत — कॉन्फिगर करण्यासाठी काही नाही. [तुमची सेटअप तपासा](#तुमची-सेटअप-तपासा) येथे जा.

**जर तुम्ही पर्याय B (हाताने) वापरले असेल**, तर उदाहरणाचा `.env` फाईल स्वतः तयार करा:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

`.env` मध्ये तुमचा एंडपॉइंट एडिट करा (की नाही — प्रमाणीकरण कीलेस आहे):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **सुरक्षा नोट:** संग्रहित करण्यासाठी कोणतीही API की नाही. तुम्ही Microsoft Entra ID द्वारे `az login` (स्थानिक) किंवा व्यवस्थापित ओळख (Azure मध्ये) वापरून प्रमाणीकरण करता. `.env` फाइलमध्ये केवळ गैर-गुपित सेटिंग्ज असतात आणि ती `.gitignore` मध्ये आधीच समाविष्ट आहे.

## तुमची सेटअप तपासा

प्रथम तुम्ही साइन इन आहात हे सुनिश्चित करा जेणेकरून कीलेस प्रमाणीकरण टोकन मिळवू शकेल, नंतर उदाहरण चालवा:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # जर आपण अद्याप साइन इन केलेले नसाल तर
mvn clean spring-boot:run
```

तुम्हाला `gpt-4o-mini` मॉडेलकडून प्रतिसाद दिसेल!

> **VS Code वापरकर्ते:** चालवण्यासाठी `F5` दाबा. अ‍ॅप आपोआप तुमच्या `.env` ला लोड करतो.

> **पूर्ण उदाहरण:** तपशील आणि समस्या निवारणासाठी [Basic Chat with Azure AI Foundry उदाहरण](./examples/basic-chat-azure/README.md) पहा.

## पुढे काय?

**सेटअप पूर्ण!** तुमच्याकडे आता आहे:
- Azure AI Foundry सह `gpt-4o-mini` आणि `text-embedding-3-small` तैनात केलेले
- कीलेस प्रमाणीकरण (Microsoft Entra ID) — कोणतीही की व्यवस्थापित करायची नाही
- तुमचा एंडपॉइंट आणि वितरण नावे असलेली स्थानिक `.env`
- Java विकास वातावरण तयार आहे

**सुरू करा** [तृतीय प्रकरण: कोअर जनरेटिव्ह AI तंत्रज्ञान](../03-CoreGenerativeAITechniques/README.md) AI अ‍ॅप्लिकेशन्स तयार करण्यासाठी!

## संसाधने

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Microsoft Entra ID सह कीलेस प्रमाणीकरण](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry दस्तऐवज](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI दस्तऐवज](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## अतिरिक्त संसाधने

- [VS Code डाउनलोड करा](https://code.visualstudio.com/Download)
- [Docker Desktop मिळवा](https://www.docker.com/products/docker-desktop)
- [Dev Container कॉन्फिगरेशन](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**अस्वीकरण**:
हा दस्तऐवज AI भाषांतर सेवा [Co-op Translator](https://github.com/Azure/co-op-translator) चा वापर करून अनुवादित केला आहे. जरी आम्ही अचूकतेसाठी प्रयत्न करतो, तरी कृपया लक्षात घ्या की स्वयंचलित भाषांतरांमध्ये त्रुटी किंवा अचूकतेची कमतरता असू शकते. मूळ दस्तऐवज त्याच्या मूळ भाषेत अधिकृत स्रोत मानला पाहिजे. महत्त्वाची माहिती असल्यास, व्यावसायिक मानवी भाषांतराची शिफारस केली जाते. या भाषांतराच्या वापरामुळे उद्भवणाऱ्या कोणत्याही गैरसमज किंवा चुकीच्या अर्थलावणीसाठी आम्ही जबाबदार नाही.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->