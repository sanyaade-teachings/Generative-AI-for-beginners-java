# جاوا کے لیے جنریٹیو AI کے لیے ترقیاتی ماحول کی ترتیب

> **فوری آغاز:** اپنے AI ماڈلز کو چند منٹوں میں بائسک + `azd` کے ساتھ بطور کوڈ **Azure AI Foundry** پر فراہم کریں — دیکھیں [Azure AI Foundry سیٹ اپ گائیڈ](getting-started-azure-openai.md)۔ توثیق **بغیر کنجی** (Microsoft Entra ID) ہے، لہٰذا کوئی API کلیدیں نہیں سنبھالنی پڑتی۔

## آپ کیا سیکھیں گے

- AI ایپلیکیشنز کے لیے جاوا ترقیاتی ماحول قائم کرنا
- اپنے پسندیدہ ترقیاتی ماحول کا انتخاب اور ترتیب دینا (Codespaces کے ساتھ کلاؤڈ پر مبنی، مقامی ڈیو کنٹینر، یا مکمل مقامی سیٹ اپ)
- Azure AI Foundry ماڈل سے جڑ کر اپنے سیٹ اپ کو ٹیسٹ کرنا

## فہرست مضامین

- [آپ کیا سیکھیں گے](#آپ-کیا-سیکھیں-گے)
- [تعارف](#تعارف)
- [مرحلہ 1: اپنا ترقیاتی ماحول قائم کریں](#مرحلہ-1-اپنا-ترقیاتی-ماحول-قائم-کریں)
  - [آپشن A: GitHub Codespaces (تجویز کردہ)](#آپشن-a-github-codespaces-تجویز-کردہ)
  - [آپشن B: مقامی ڈیو کنٹینر](#آپشن-b-مقامی-ڈیو-کنٹینر)
  - [آپشن C: اپنا موجودہ مقامی انسٹالیشن استعمال کریں](#آپشن-c-اپنا-موجودہ-مقامی-انسٹالیشن-استعمال-کریں)
- [مرحلہ 2: Azure AI Foundry کی فراہمی](#مرحلہ-2-azure-ai-foundry-فراہم-کریں)
- [مرحلہ 3: اپنے سیٹ اپ کا ٹیسٹ کریں](#مرحلہ-3-اپنے-سیٹ-اپ-کا-ٹیسٹ-کریں)
- [مسائل کا حل](#مسائل-کا-حل)
- [خلاصہ](#خلاصہ)
- [اگلے اقدامات](#اگلے-اقدامات)

## تعارف

یہ باب آپ کو ترقیاتی ماحول قائم کرنے میں رہنمائی کرے گا۔ ہم اس کورس کے دوران **Azure AI Foundry** کو ماڈلز کے لیے استعمال کریں گے۔ آپ ماڈلز کو بائسک اور Azure Developer CLI (`azd`) کے ساتھ کوڈ کی طرح فراہم کرتے ہیں، پھر **بغیر کنجی کی توثیق** (Microsoft Entra ID) کے ذریعے جڑتے ہیں — کوئی API کلیدیں نقل یا لیک نہیں کرنی پڑتی۔

**کوئی مقامی سیٹ اپ ضروری نہیں!** آپ GitHub Codespaces استعمال کر سکتے ہیں، جو آپ کے براؤزر میں مکمل ترقیاتی ماحول فراہم کرتا ہے، اور وہاں سے Foundry فراہم کر سکتے ہیں۔

ہم اس کورس کے لیے **Azure AI Foundry** استعمال کرتے ہیں کیونکہ یہ:
- **کوڈ کی طرح فراہم کردہ** – صرف ایک `azd up` اکاؤنٹ اور ماڈل کی تعیناتی کرتا ہے
- **بغیر کنجی کی** – اپنے Azure سائن ان یا مینیجڈ شناخت کے ساتھ توثیق کریں
- **پروڈکشن کے قابل** – وہی کوڈ لوکل اور Azure دونوں پر چلتا ہے
- **لچکدار** – ماڈلز کوڈ میں تبدیلی کیے بغیر تعیناتی کے نام سے تبدیل کریں

> **نوٹ**: Azure AI Foundry کی تعیناتیاں ٹوکنز کے حساب سے بل کی جاتی ہیں (ادا کیجیے جتنا استعمال کریں)۔ فراہمی، علاقے اور لاگت کی تفصیلات کے لیے [Azure AI Foundry سیٹ اپ گائیڈ](getting-started-azure-openai.md) دیکھیں۔

## مرحلہ 1: اپنا ترقیاتی ماحول قائم کریں

<a name="quick-start-cloud"></a>

ہم نے ایک پہلے سے ترتیب شدہ ترقیاتی کنٹینر بنایا ہے تاکہ سیٹ اپ کا وقت کم سے کم کیا جا سکے اور یقین دہانی ہو کہ آپ کے پاس اس جنریٹیو AI برائے جاوا کورس کے تمام ضروری آلات موجود ہیں۔ اپنی پسند کی ترقیاتی طریقہ کار منتخب کریں:

### ماحول کی ترتیب کے اختیارات:

#### آپشن A: GitHub Codespaces (تجویز کردہ)

**2 منٹ میں کوڈنگ شروع کریں - مقامی سیٹ اپ کی کوئی ضرورت نہیں!**

1. اس مجموعہ کو اپنے GitHub اکاؤنٹ پر فورک کریں  
   > **نوٹ**: اگر آپ بنیادی ترتیب میں ترمیم کرنا چاہتے ہیں تو [Dev Container Configuration](../../../.devcontainer/devcontainer.json) دیکھیں
2. پر کلک کریں **Code** → **Codespaces** ٹیب → **...** → **New with options...**
3. ڈیفالٹس استعمال کریں – یہ منتخب کرے گا **Dev container configuration**: **Generative AI Java Development Environment**، جو اس کورس کے لیے بنائی گئی کسٹم ڈیو کنٹینر ہے
4. پر کلک کریں **Create codespace**
5. تقریباً 2 منٹ انتظار کریں جب تک ماحول تیار نہ ہو جائے
6. آگے بڑھیں [مرحلہ 2: Azure AI Foundry فراہم کریں](#مرحلہ-2-azure-ai-foundry-فراہم-کریں)

<img src="../../../translated_images/ur/codespaces.9945ded8ceb431a5.webp" alt="اسکرین شاٹ: Codespaces سب مینو" width="50%">

<img src="../../../translated_images/ur/image.833552b62eee7766.webp" alt="اسکرین شاٹ: New with options" width="50%">

<img src="../../../translated_images/ur/codespaces-create.b44a36f728660ab7.webp" alt="اسکرین شاٹ: Create codespace کے اختیارات" width="50%">


> **Codespaces کے فوائد**:
> - کوئی مقامی انسٹالیشن ضروری نہیں
> - کسی بھی براؤزر والے آلے پر کام کرتا ہے
> - تمام اوزار اور انحصارات کے ساتھ پہلے سے ترتیب دیا گیا
> - ذاتی اکاؤنٹس کے لیے ماہانہ 60 گھنٹے مفت
> - تمام سیکھنے والوں کے لیے مستقل ماحول

#### آپشن B: مقامی ڈیو کنٹینر

**ڈاکر کے ذریعے مقامی ترقی کو ترجیح دینے والے ڈویلپرز کے لیے**

1. اس مجموعہ کو فورک اور کلون کریں اپنی مقامی مشین پر  
   > **نوٹ**: اگر آپ بنیادی ترتیب میں ترمیم کرنا چاہتے ہیں تو [Dev Container Configuration](../../../.devcontainer/devcontainer.json) دیکھیں
2. [Docker Desktop](https://www.docker.com/products/docker-desktop/) اور [VS Code](https://code.visualstudio.com/) انسٹال کریں
3. VS Code میں [Dev Containers توسیع](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) انسٹال کریں
4. VS Code میں مجموعہ فولڈر کھولیں
5. جب پوچھا جائے، کلک کریں **Reopen in Container** (یا `Ctrl+Shift+P` → "Dev Containers: Reopen in Container" استعمال کریں)
6. کنٹینر کے بننے اور شروع ہونے کا انتظار کریں
7. آگے بڑھیں [مرحلہ 2: Azure AI Foundry فراہم کریں](#مرحلہ-2-azure-ai-foundry-فراہم-کریں)

<img src="../../../translated_images/ur/devcontainer.21126c9d6de64494.webp" alt="اسکرین شاٹ: ڈیو کنٹینر سیٹ اپ" width="50%">

<img src="../../../translated_images/ur/image-3.bf93d533bbc84268.webp" alt="اسکرین شاٹ: ڈیو کنٹینر بلڈ مکمل" width="50%">

#### آپشن C: اپنا موجودہ مقامی انسٹالیشن استعمال کریں

**موجودہ جاوا ماحول کے ساتھ ڈویلپرز کے لیے**

ضروریات:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html) 
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) یا آپ کی پسندیدہ IDE

مراحل:
1. اس مجموعہ کو اپنی مقامی مشین پر کلون کریں
2. اپنے IDE میں پروجیکٹ کھولیں
3. آگے بڑھیں [مرحلہ 2: Azure AI Foundry فراہم کریں](#مرحلہ-2-azure-ai-foundry-فراہم-کریں)

> **پرو ٹپ**: اگر آپ کے پاس کم صلاحیت کی مشین ہے لیکن VS Code مقامی طور پر استعمال کرنا چاہتے ہیں، تو GitHub Codespaces استعمال کریں! آپ اپنا مقامی VS Code کلاؤڈ میں ہوسٹڈ Codespace سے جوڑ سکتے ہیں تاکہ دونوں کی بہترین خصوصیات حاصل ہوں۔

<img src="../../../translated_images/ur/image-2.fc0da29a6e4d2aff.webp" alt="اسکرین شاٹ: بنائی گئی مقامی ڈیو کنٹینر مثال" width="50%">


## مرحلہ 2: Azure AI Foundry فراہم کریں

کورس کے AI ماڈلز کو کوڈ کی طرح Azure AI Foundry پر تعینات کریں۔ مجموعہ کے روٹ سے:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```
  
`azd` ماحول کا نام اور علاقہ پوچھتا ہے، Azure AI Foundry اکاؤنٹ `gpt-4o-mini` اور `text-embedding-3-small` تعیناتیوں کے ساتھ فراہم کرتا ہے، اور اینڈپوائنٹ مثال کے `.env` میں لکھتا ہے — سب کچھ **بغیر کنجی** کی توثیق (کوئی API کی نہیں) کے ساتھ۔

> **مکمل گام بہ گام ہدایت:** ضروریات، دستی (پورٹل) متبادل، علاقائی رہنمائی، اور لاگت/صفائی کے نوٹس کے لیے [Azure AI Foundry سیٹ اپ گائیڈ](getting-started-azure-openai.md) دیکھیں۔

## مرحلہ 3: اپنے سیٹ اپ کا ٹیسٹ کریں

جب آپ کے Foundry ماڈلز فراہم ہو جائیں، تو [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) میں مثال ایپ کے ساتھ کنکشن کو ٹیسٹ کریں۔

1. اپنے ترقیاتی ماحول میں ٹرمینل کھولیں۔
2. مثال فولڈر میں جائیں:  
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
  
3. یقینی بنائیں کہ آپ سائن ان ہیں (بغیر کنجی کی توثیق کے لیے ٹوکن چاہیے):  
   ```bash
   az login
   ```
  
   > اگر آپ نے `azd up` چلایا ہے، تو آپ کے اینڈپوائنٹ کے ساتھ `.env` فائل پہلے ہی بن چکی ہے۔  
4. ایپلیکیشن چلائیں:  
   ```bash
   mvn clean spring-boot:run
   ```
  
آپ کو `gpt-4o-mini` ماڈل سے جواب دیکھنا چاہیے۔

### مثال کے کوڈ کو سمجھنا

`examples/basic-chat-azure` کے نیچے کی مثال ایک Spring Boot ایپ ہے جو **Spring AI** استعمال کرتی ہے Azure AI Foundry سے بغیر کنجی کی توثیق کے جڑنے کے لیے۔

**یہ کوڈ کیا کرتا ہے:**
- آپ کے Azure سائن ان (Microsoft Entra ID) کے ساتھ Azure AI Foundry سے جڑتا ہے — کوئی API کلید نہیں
- `gpt-4o-mini` ماڈل کو پرامپٹ بھیجتا ہے
- AI کے جواب کو وصول اور دکھاتا ہے
- آپ کے سیٹ اپ کی درستگی کو جانچتا ہے

**اہم انحصار** (`pom.xml` میں):  
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```
  
**کنفیگریشن** (`application.yml`):  
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
  
## خلاصہ

زبردست! آپ نے اب سب کچھ ترتیب دے دیا ہے:

- بائسک + `azd` کے ساتھ کوڈ کی طرح Azure AI Foundry ماڈلز فراہم کیے
- آپ کا جاوا ترقیاتی ماحول چل رہا ہے (چاہے Codespaces ہو، ڈیو کنٹینرز یا مقامی)
- Azure AI Foundry سے بغیر کنجی کی توثیق (Microsoft Entra ID) کے جڑے ہیں — کوئی API کلید نہیں
- ایک آسان مثال کے ساتھ سب کچھ کام کرتا ہے جس سے آپ کا ماڈل بات کرتا ہے

## اگلے اقدامات

[باب 3: بنیادی جنریٹیو AI تکنیکیں](../03-CoreGenerativeAITechniques/README.md)

## مسائل کا حل

مسائل پیش آ رہے ہیں؟ یہاں عام مسائل اور حل درج ہیں:

- **توثیق ناکام (401/403)؟**  
  - `az login` چلائیں — توثیق بغیر کنجی کی ہے، اس لیے آپ کا سائن ان ہونا ضروری ہے  
  - یہ تصدیق کریں کہ آپ کے اکاؤنٹ کو Cognitive Services OpenAI User کا کردار حاصل ہے  
  - اگر آپ نے ابھی تو تعینات کیا ہے، تو کردار کی تفویض پھیلنے کے لیے ایک منٹ انتظار کریں

- **Maven نہیں مل رہا؟**  
  - اگر dev containers/Codespaces استعمال کر رہے ہیں، تو Maven پہلے سے نصب ہونا چاہیے  
  - مقامی سیٹ اپ کے لیے، Java 21+ اور Maven 3.9+ انسٹال کریں  
  - `mvn --version` چلائیں تاکہ تنصیب کی تصدیق ہو

- **`azd` نہیں مل رہا یا فراہمی ناکام؟**  
  - [Azure Developer CLI](https://aka.ms/azure-dev/install) انسٹال کریں اور `azd auth login` چلائیں  
  - ایسا علاقہ منتخب کریں جہاں `gpt-4o-mini` دستیاب ہو (مثال کے طور پر `eastus2`)  
  - تفصیلات کے لیے [Azure AI Foundry سیٹ اپ گائیڈ](getting-started-azure-openai.md) دیکھیں

- **ڈیو کنٹینر شروع نہیں ہو رہا؟**  
  - یقینی بنائیں Docker Desktop چل رہا ہے (مقامی ترقی کے لیے)  
  - کنٹینر کو دوبارہ بنانے کی کوشش کریں: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **ایپلیکیشن کا کمپائل ایرر؟**  
  - یقینی بنائیں کہ صحیح ڈائریکٹری میں ہیں: `02-SetupDevEnvironment/examples/basic-chat-azure`  
  - صفائی اور دوبارہ تعمیر کریں: `mvn clean compile`

> **مدد چاہیے؟**: ابھی بھی مسائل ہیں؟ اس مجموعہ میں مسئلہ کھولیں، ہم آپ کی مدد کریں گے۔

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ڈس کلیمر**:
یہ دستاویز AI ترجمہ سروس [Co-op Translator](https://github.com/Azure/co-op-translator) کے ذریعے ترجمہ کی گئی ہے۔ جبکہ ہم درستگی کے لیے کوشاں ہیں، براہ کرم اس بات سے آگاہ رہیں کہ خودکار ترجمے میں غلطیاں یا عدم درستیاں ہو سکتی ہیں۔ اصل دستاویز اپنے مادری زبان میں مستند ماخذ سمجھی جائے گی۔ حساس معلومات کے لیے پیشہ ور انسانی ترجمہ کی سفارش کی جاتی ہے۔ اس ترجمے کے استعمال سے پیدا ہونے والی کسی بھی غلط فہمی یا غلط تشریح کی ذمہ داری ہم قبول نہیں کرتے۔
<!-- CO-OP TRANSLATOR DISCLAIMER END -->