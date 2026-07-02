# AGENTS.md

## Project Overview

ဤသည်မှာ Java နှင့် Generative AI ဖြံ့ဖြိုးမှုကို သင်ယူရန် အသိပညာပေး repository ဖြစ်သည်။ ၎င်းသည် Large Language Models (LLMs), prompt engineering, embeddings, RAG (Retrieval-Augmented Generation), နှင့် Model Context Protocol (MCP) အပါအဝင် ကျယ်ပြန့်သော လက်တွေ့သင်တန်းကို ပေးစွမ်းသည်။

**အဓိကနည်းပညာများ:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI, နှင့် OpenAI SDKs

**ဖွဲ့စည်းပုံ:**
- အခန်းများအလိုက် စီစဉ်ထားသည့် မတူညီသော standalone Spring Boot applications များ
- AI ပုံစံကွဲပြားမှုများကို ပြသသည့် နမူနာ projects များ
- GitHub Codespaces-ready  ဖြစ်ပြီး dev containers များကို ကြိုတင် ဖွဲ့စည်းထားသည်

## Setup Commands

### Prerequisites
- Java 21 သို့မဟုတ် အထက်
- Maven 3.x
- Azure subscription နှင့် Azure AI Foundry model deployment (အတည်ပြုရန် `azd up` ဖြင့် provision)
- Azure CLI (`az`) နှင့် Azure Developer CLI (`azd`), keyless auth အတွက် စာရင်းဝင်ထား

### Environment Setup

**Option 1: GitHub Codespaces (အကြံပြု)**
```bash
# GitHub UI မှ repository ကို fork လုပ်ပြီး codespace တစ်ခု ဖန်တီးပါ
# dev container သည် အလိုအလျောက် dependency အားလုံးကို 설치 လုပ်ပေးပါမည်
# ပတ်ဝန်းကျင် သတ်မှတ်ခြင်းအတွက် ~2 မိနစ် ခန့် စောင့်ဆိုင်းပါ
```

**Option 2: Local Dev Container**
```bash
# စာကြည့်တိုက်ကို မိတ္တူယူပါ
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Dev Containers အပြင်အဆင်ဖြင့် VS Code တွင် ဖွင့်ပါ
# အဆိုပြုသောအခါ Container ထဲတွင် ထပ်မံဖွင့်ပါ
```

**Option 3: Local Setup**
```bash
# ကြိုတင်လိုအပ်ချက်များ ထည့်သွင်းပါ
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# ထည့်သွင်းမှုကို အတည်ပြုပါ
java -version
mvn -version
```

### Configuration

**Azure AI Foundry Setup (keyless, အကြံပြု):**
```bash
# Foundry အကောင့်နှင့်မော်ဒယ်တင်သွင်းမှုများကို ကုဒ်အနေနှင့် ပြုလုပ်ပါ
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd သည် သင့် endpoint ဖြင့် examples/basic-chat-azure/.env ကို ရေးသားသည် (key မပါ)
```

**Manual endpoint config:**
```bash
# သင် azd ကို မအသုံးပြုခဲ့ပါက endpoint ကို ကိုယ်ပိုင်သတ်မှတ်ပါ (auth သည် az login မှတဆင့် keyless ဖြစ်နေပါသည်)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# .env ကို တည်းဖြတ်ပါ: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Development Workflow

### Project Structure
```
/
├── 01-IntroToGenAI/              # Chapter 1: Introduction
├── 02-SetupDevEnvironment/       # Chapter 2: Environment setup
│   └── examples/                 # Working examples
├── 03-CoreGenerativeAITechniques/ # Chapter 3: Core techniques
├── 04-PracticalSamples/          # Chapter 4: Sample projects
│   ├── calculator/               # MCP service example
│   ├── foundrylocal/            # Local model integration
│   └── petstory/                # Multi-modal app
├── 05-ResponsibleGenAI/         # Chapter 5: Responsible AI
└── translations/                # Multi-language support
```

### Running Applications

**Spring Boot application တစ်ခု ထည့်မောင်းခြင်း:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Project တစ်ခု တည်ဆောက်ခြင်း:**
```bash
cd [project-directory]
mvn clean install
```

**MCP Calculator Server စတင်ခြင်း:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# ဆာဗာကို http://localhost:8080 မှာ အလုပ်လုပ်နေသည်။
```

**Client ဥပမာများ မောင်းနှင်ခြင်း:**
```bash
# အခြား terminal တစ်ခုတွင် server ကို စတင်ပြီးနောက်
cd 04-PracticalSamples/calculator

# တိုက်ရိုက် MCP client
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# AI မောင်းနှင်ထားသော client (AZURE_OPENAI_ENDPOINT နှင့် az login လိုအပ်သည်)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# အပြန်အလှန်ဆက်သွယ်နိုင်သော bot
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Hot Reload
Spring Boot DevTools သည် hot reload ကို ထောက်ပံ့သော projects များတွင် ပါဝင်သည်။
```bash
# Java ဖိုင်များကို ပြင်ဆင်ပြီး သိမ်းဆည်းသောအခါ အလိုအလျောက် ပြန်လည်လုပ်ဆောင်ပါမည်
mvn spring-boot:run
```

## Testing Instructions

### Running Tests

**Project တစ်ခုအတွင်း စမ်းသပ်ချက်အားလုံး အပြေး:**
```bash
cd [project-directory]
mvn test
```

**Verbose output ဖြင့် စမ်းသပ်ချက်များ လည်ပတ်ခြင်း:**
```bash
mvn test -X
```

**သတ်မှတ်ထားသည့် test class ကို တစ်ခုချင်း run ပြုလုပ်ခြင်း:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Test Structure
- စမ်းသပ်ဖိုင်များတွင် JUnit 5 (Jupiter) အသုံးပြုသည်
- စမ်းသပ် class များကို `src/test/java/` တွင် တည်ရှိသည်
- calculator project မှ client ဥပမာများကို `src/test/java/com/microsoft/mcp/sample/client/` တွင် ရှာနိုင်သည်

### Manual Testing
အများစုသော ဥပမာများသည် ဆက်သွယ်သုံးစွဲရသော applications ဖြစ်၍ manual testing လိုအပ်သည်။

1. `mvn spring-boot:run` ဖြင့် application ကို စတင်ပါ
2. endpoint များကို စမ်းသပ်ရန် သို့မဟုတ် CLI နှင့် ဆက်သွယ်ရန်
3. ယူဆထားသောအပြုအမူသည် တခြား project များ၏ README.md တွင် ဖော်ပြထားသည့် လက်တွေ့အချက်အလက်နှင့် ကိုက်ညီမှု သေချာစစ်ဆေးပါ

### Testing with Azure AI Foundry
- ဥပမာများ run မည်မှာ `az login` ဖြင့် စာရင်းဝင်ထားရန် (keyless auth)
- သင်၏ အကောင့်တွင် Cognitive Services OpenAI User အခန်းကဏ္ဍ ရှိမှုရှိရန် အာမခံပါ
- Chapter 5 တွင် ပါဝင်သည့် 책임있는 AI ဥပမာဖြင့် content filtering စမ်းသပ်မှု ပြုလုပ်နိုင်သည်

## Code Style Guidelines

### Java Conventions
- **Java Version:** Java 21 နှင့် အဆင့်မြင့် function များ
- **Style:** Java စံပြ သတ်မှတ်ချက်များကို လိုက်နာပါ
- **Naming:**
  - Classes: PascalCase
  - Methods/variables: camelCase
  - Constants: UPPER_SNAKE_CASE
  - Package names: lowercase

### Spring Boot Patterns
- နယ်ပယ် logic အတွက် `@Service` ကို အသုံးပြုပါ
- REST endpoints အတွက် `@RestController` ကို အသုံးပြုပါ
- `application.yml` သို့မဟုတ် `application.properties` ဖြင့် ပြင်ဆင်မှု စီမံပါ
- ပြောင်းလဲမှုများအတွက် Environment variables များကို ဦးစားပေးအသုံးပြုပါ
- MCP method များအတွက် `@Tool` annotation ကို အသုံးပြုပါ

### File Organization
```
src/
├── main/
│   ├── java/
│   │   └── com/microsoft/[component]/
│   │       ├── [Component]Application.java
│   │       ├── config/
│   │       ├── controller/
│   │       ├── service/
│   │       └── exception/
│   └── resources/
│       ├── application.yml
│       └── static/
└── test/
    └── java/
        └── com/microsoft/[component]/
```

### Dependencies
- Maven `pom.xml` ဖြင့် စီမံခန့်ခွဲသည်
- Spring AI BOM version စီမံကိန်း
- LangChain4j ဖြင့် AI အပြင်အဆင်များ ပံ့ပိုးမှု
- Spring Boot starter parent ကို Spring နည်းပညာများအတွက် အသုံးပြုသည်

### Code Comments
- အများပြည်သူ API များအတွက် JavaDoc ထည့်ပါ
- ခက်ခဲသော AI လုပ်ဆောင်ချက်များအတွက် ရှင်းပြချက်များ ထည့်သွင်းပါ
- MCP tool အသုံးပြုပုံ အကြောင်းအရာများကို သေချာဖော်ပြပါ

## Build and Deployment

### Building Projects

**စမ်းသပ်မပါဘဲ build လုပ်ခြင်း:**
```bash
mvn clean install -DskipTests
```

**စစ်ဆေးမှုအားလုံးပါဝင်သော build:**
```bash
mvn clean install
```

**Application package ပြုလုပ်ခြင်း:**
```bash
mvn package
# target/ ဖိုလ်ဒါတွင် JAR ဖိုင်ကို ဖန်တီးသည်။
```

### Output Directories
- ပြုပြင်ပြီးသော classes: `target/classes/`
- စမ်းသပ် classes: `target/test-classes/`
- JAR ဖိုင်များ: `target/*.jar`
- Maven artifacts: `target/`

### Environment-Specific Configuration

**ဖွံ့ဖြိုးရေး:**
```yaml
# application.yml (keyless - no api-key; auth via DefaultAzureCredential)
spring:
  ai:
    azure:
      openai:
        endpoint: ${AZURE_OPENAI_ENDPOINT}
        chat:
          options:
            deployment-name: ${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}
```

**ထုတ်လုပ်ရေး:**
- keyless auth အတွက် `az login` မသုံးဘဲ managed identity ကို အသုံးပြုပါ
- `AZURE_OPENAI_ENDPOINT` ကို ထုတ်လုပ်ရေး Foundry resource အတိုင်း ချိန်ညှိပါ
- configuration တွေကို environment variables သို့မဟုတ် Azure Key Vault ဖြင့် စီမံပါ

### Deployment Considerations
- ဤသည်မှာ နည်းပညာပညာရေး repository ဖြစ်ပြီး နမူနာ application များ ပါဝင်သည်
- မျက်မှောက်အနေဖြင့် ထုတ်လုပ်ရေး deployment မဟုတ်ပါ
- နမူနာများတွင် ထုတ်လုပ်ရေးအတွက် ထပ်ဆင့်ဖွဲ့စည်းမှုများ ကို ပုံပြင်ထားပါသည်
- အသေးစိတ် deployment မှတ်ချက်များအတွက် တစ်ခုချင်း project ရဲ့ README များကို ကြည့်ပါ

## Additional Notes

### Azure AI Foundry
- **Keyless auth:** Microsoft Entra ID နှင့် ချိတ်ဆက်သည် — API key မလိုအပ်ပါ
- **Provisioned as code:** Bicep + azd (`azd up`) ဖြင့် အကောင့်နှင့် model deployment များဖန်တီးသည်
- ထို OpenAI-compatible code သည် မိမိ ဒေသတွင် (အသုံးပြုရန် `az login`) နှင့် Azure (managed identity) တွင်လည်း အလုပ်လုပ်သည်

### Working with Multiple Projects
နမူနာ project တစ်ခုချင်းစီ သည် standalone ဖြစ်ပါသည်။
```bash
# သတ်မှတ်ထားသော ပရောဂျက်သို့ သွားပါ
cd 04-PracticalSamples/[project-name]

# တစ်ခုချင်းစီမှာ ကိုယ်ပိုင် pom.xml ရှိပြီး အလိုအလျောက် တည်ဆောက်နိုင်ပါသည်
mvn clean install
```

### Common Issues

**Java Version မကိုက်ညီမှု:**
```bash
# Java 21 ကို အတည်ပြုပါ
java -version
# လိုအပ်လျှင် JAVA_HOME ကို အပ်ဒိတ်လုပ်ပါ။
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Dependency ဒေါင်းလုပ် လုပ်ရာ ပြဿနာ:**
```bash
# Maven cache ကို ရှင်းလင်းပြီး ထပ်မံကြိုးစားပါ။
rm -rf ~/.m2/repository
mvn clean install
```

**Endpoint သို့မဟုတ် Sign-in မတွေ့ရှိနိုင်ခြင်း:**
```bash
# လတ်တလော အစည်းအဝေးတွင် အဆုံးအမှတ်ကို သတ်မှတ်ပြီး လော့ဂ်အင်ရယူပါ(သော့ချက်မလိုဘဲ)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# ဒါမှမဟုတ် project ဖိုလ်ဒါအတွင်းရှိ .env ဖိုင်ကို အသုံးပြုပါ
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Port သုံးပြီးသား ဖြစ်နေခြင်း:**
```bash
# Spring Boot သည် ပုံမှန်အားဖြင့် port 8080 ကိုအသုံးပြုသည်
# application.properties တွင် ပြောင်းလဲရန်:
server.port=8081
```

### Multi-Language Support
- ၄၅ ကျော်ဘာသာစကားများတွင် automated ဘာသာပြန် စနစ်ဖြင့် သက်ဆိုင်ရာစာရွက်စာတမ်း ရရှိနိုင်သည်
- ဘာသာပြန်များကို `translations/` ဖိုလ်ဒါတွင် သိမ်းဆည်းထားသည်
- ဘာသာပြန်လုပ်ငန်းကို GitHub Actions workflow ဖြင့် စီမံသည်

### Learning Path
1. [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md) မှ စတင်ပါ
2. အခန်းများနည်းလမ်းစဉ်အတိုင်း လိုက်နာခြင်း (01 → 05)
3. အခန်းတိုင်းတွင် လက်တွေ့ စမ်းသပ်မှုများ ပြည့်စုံအောင် လုပ်ဆောင်ပါ
4. Chapter 4 တွင် နမူနာ projects များကို လေ့လာပါ
5. Chapter 5 တွင် တာဝန်ရှိသော AI လေ့လာမှုများ လုပ်ဆောင်ပါ

### Development Container
`.devcontainer/devcontainer.json` တွင် ထည့်သွင်းထားသော configuration များမှာ:
- Java 21 ဖွံ့ဖြိုးရေးပတ်ဝန်းက်
- Maven ကြိုတင် ထည့်သွင်းထားခြင်း
- VS Code Java extensions များ
- Spring Boot tools များ
- GitHub Copilot integration
- Docker-in-Docker ထောက်ပံ့မှု
- Azure CLI

### Performance Considerations
- Azure AI Foundry deployments များတွင် တစ်မိနစ်စီ token/request အရေအတွက် ကန့်သတ်ထားသည်
- embeddings များအတွက် အထူးသင့်လျော်သော batch size ကို သုံးပါ
- API call များ အကြိမ်များပြန် အသုံးပြုပါက caching ကို စဉ်းစားပါ
- သုံးစွဲ token အရေအတွက်ကို စောင့်ကြည့်ကာ ကုန်ကျစရိတ် သက်သာစေပါ

### Security Notes
- `.env` ဖိုင်များကို ဘယ်တော့မှ commit မလုပ်ပါ (အပြင် `.gitignore` တွင် ပါဝင်သည်)
- API key များထက် keyless auth (Microsoft Entra ID) ကို ဦးစားပေး အသုံးပြုပါ
- Azure တွင် managed identities အသုံးပြုသည်၊ ဒေသတွင်းဖွံ့ဖြိုးရေးအတွက် `az login` ကို သုံးပါ
- Chapter 5 တွင် ပါ ်သော တာဝန်ရှိသော AI လမ်းညွှန်ချက်များကို လိုက်နာပါ

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ပြောကြားချက်**
ဤစာတမ်းကို AI ဘာသာပြန်ဝန်ဆောင်မှု [Co-op Translator](https://github.com/Azure/co-op-translator) အသုံးပြု၍ ဘာသာပြန်ထားပါသည်။ ကျွန်ုပ်တို့သည် တိကျမှန်ကန်မှုအတွက် ကြိုးပမ်းနေသော်လည်း၊ စက်ကိရိယာဘာသာပြန်ခြင်းများတွင် အမှားများ သို့မဟုတ် မှားယွင်းချက်များ ပါဝင်နိုင်ကြောင်း သတိပြုပါရန် လိုအပ်ပါသည်။ မူလစာတမ်းကို မူရင်းဘာသာဖြင့်သာ ယုံကြည်စိတ်ချရသော အချက်အလက်အဖြစ် သတ်မှတ်သင့်သည်။ အရေးကြီးသည့် သတင်းအချက်အလက်များအတွက် ပရော်ဖက်ရှင်နယ် လူသားဘာသာပြန်သူဝန်ဆောင်မှုကို အကြံပြုပါသည်။ ဤဘာသာပြန်ချက်ကို အသုံးပြုခြင်းမှ ဖြစ်ပေါ်လာသော နားလည်မှုကွာခြားမှုများ သို့မဟုတ် မမှန်ကန်သော အသုံးပြုမှုများအတွက် ကျွန်ုပ်တို့ တာဝန်မခံပါ။
<!-- CO-OP TRANSLATOR DISCLAIMER END -->