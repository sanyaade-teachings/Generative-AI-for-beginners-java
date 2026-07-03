# Azure AI Foundry کے لیے ترقیاتی ماحول مرتب کرنا

> یہ رہنما اس کورس میں جاوا AI ایپس کے لیے **Azure AI Foundry** ماڈلز کو **keyless** تصدیق (Microsoft Entra ID) کے استعمال سے سیٹ اپ کرتا ہے — کوئی API کیز منیج کرنے کی ضرورت نہیں۔ ٹولنگ کے بارے میں نیا ہیں؟ [ترقیاتی ماحول کی رہنما](./README.md) سے شروع کریں۔

یہ رہنما اس کورس میں جاوا AI ایپس کے لیے **Azure AI Foundry** ماڈلز کو سیٹ اپ کرتا ہے۔ آپ کے پاس دو راستے ہیں:

- **آپشن A — `azd` + Bicep کے ساتھ پرویژنس کریں (تجویز کردہ):** ایک کمانڈ سے Foundry اکاؤنٹ اور ماڈلز کو بطور کوڈ ڈیپلائے کرتا ہے۔ کوئی پورٹل کلک کرنے کی ضرورت نہیں۔
- **آپشن B — Azure AI Foundry پورٹل میں دستی طور پر وسائل بنائیں۔**

دونوں راستے **keyless تصدیق** (Microsoft Entra ID) استعمال کرتے ہیں — کوئی API کیز کاپی یا لیک نہیں ہوں گی۔

## فہرست مضامین

- [کیا بنایا جاتا ہے](#کیا-بنایا-جاتا-ہے)
- [ضروریات](#ضروریات)
- [آپشن A: azd + Bicep کے ساتھ پرویژنس (تجویز کردہ)](#option-a-provision-with-azd--bicep-recommended)
- [آپشن B: دستی طور پر وسائل بنائیں](#آپشن-b-دستی-طور-پر-وسائل-بنائیں)
- [اپنا ماحول ترتیب دیں](#اپنا-ماحول-ترتیب-دیں)
- [اپنے سیٹ اپ کی جانچ کریں](#اپنے-سیٹ-اپ-کی-جانچ-کریں)
- [اگلا کیا ہے؟](#اگلا-کیا-ہے؟)
- [وسائل](#وسائل)
- [اضافی وسائل](#اضافی-وسائل)

## کیا بنایا جاتا ہے

[`infra/`](../../../02-SetupDevEnvironment/infra) میں موجود Bicep ٹیمپلیٹس درج ذیل چیزیں فراہم کرتے ہیں:

- ایک **Azure AI Foundry** اکاؤنٹ (`Microsoft.CognitiveServices/accounts`, قسم `AIServices`) جس میں ایک پروجیکٹ
- ایک **چیٹ** ڈیپلائمنٹ — `gpt-4o-mini`
- ایک **ایمبیڈنگ** ڈیپلائمنٹ — `text-embedding-3-small` (جو بعد کے ابواب میں استعمال ہوگی)
- ایک **keyless رول تفویض** (`Cognitive Services OpenAI User`) تاکہ آپ `az login` کے ذریعے سائن ان کریں بغیر کیز کے انتظام کے

## ضروریات

- ایک [Azure سبسکرپشن](https://azure.microsoft.com/free/)
- [Azure ڈویلپر CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) اور [Maven 3.9+](https://maven.apache.org/download.cgi)

## آپشن A: azd + Bicep کے ساتھ پرویژنس (تجویز کردہ)

`02-SetupDevEnvironment` فولڈر سے:

```bash
cd 02-SetupDevEnvironment

# سائن ان کریں (دونوں ٹولز)
azd auth login
az login

# فاؤنڈری اکاؤنٹ + ماڈل کی تعیناتی فراہم کریں
azd up
```
  
`azd` آپ سے ایک **ماحول کا نام** (مثلاً `genai-java`) اور ایک **علاقہ** پوچھتا ہے۔ ایسا علاقہ منتخب کریں جہاں `gpt-4o-mini` اور `text-embedding-3-small` دستیاب ہوں — مثلاً `eastus2` یا `swedencentral`۔

پرویژنس مکمل ہونے پر، azd:

1. سب کچھ جو [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep) میں ڈیفائن کیا گیا ہے، ڈیپلائے کرتا ہے۔
2. ایک پوسٹ پروویژن ہک چلتا ہے جو آپ کے اینڈپوائنٹ اور ڈیپلائمنٹ ناموں کے ساتھ [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) لکھتا ہے (کوئی سیکریٹس نہیں)۔

> **مشورہ:** تبدیلیوں کو لاگو کرنے کے لیے کسی بھی وقت `azd up` دوبارہ چلائیں۔ سب کچھ حذف کرنے اور خرچ روکنے کے لیے `azd down` چلائیں۔

بنائی گئی ترتیبات دیکھنے کے لیے:

```bash
azd env get-values
```
  
اب [اپنے سیٹ اپ کی جانچ کریں](#اپنے-سیٹ-اپ-کی-جانچ-کریں) پر جائیں۔

## آپشن B: دستی طور پر وسائل بنائیں

پورٹل استعمال کرنا پسند ہے؟ تو وسائل خود بنائیں:

1. [Azure AI Foundry پورٹل](https://ai.azure.com/) پر جائیں اور سائن ان کریں۔
2. **پروجیکٹ بنائیں** (یہ AI Foundry ریسورس بھی بناتا ہے)۔ اسے `GenAIJava` جیسا کوئی نام دیں۔
3. اپنے پروجیکٹ میں، **Models + endpoints** → **Deploy model** → **Deploy base model** کھولیں۔
4. **gpt-4o-mini** (ڈیپلائمنٹ نام `gpt-4o-mini`) کو ڈیپلائے کریں۔ اگر ایمبیڈنگ کی مثالیں چاہییں تو **text-embedding-3-small** کے لیے بھی یہی کریں۔
5. **Overview** سے **endpoint** کاپی کریں (مثلاً `https://<resource>.openai.azure.com/`)۔
6. خود کو keyless رسائی دیں: ریسورس پر جائیں، **Access control (IAM)** → **Add role assignment** → **Cognitive Services OpenAI User** کو اپنے اکاؤنٹ کو تفویض کریں۔

> **ابھی بھی مسئلہ ہے؟** [Azure AI Foundry کی دستاویزات](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects) دیکھیں۔

## اپنا ماحول ترتیب دیں

**اگر آپ نے آپشن A (`azd up`) استعمال کیا ہے** تو آپ کی سیٹنگ فائل پہلے ہی لکھی جا چکی ہے — ترتیب دینے کی ضرورت نہیں۔ [اپنے سیٹ اپ کی جانچ کریں](#اپنے-سیٹ-اپ-کی-جانچ-کریں) پر جائیں۔

**اگر آپ نے آپشن B (دستی) استعمال کیا ہے** تو مثال کا `.env` فائل خود بنائیں:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```
  
اپنے اینڈپوائنٹ کے ساتھ `.env` ایڈیٹ کریں (کوئی کی نہیں — تصدیق keyless ہے):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```
  
> **سیکورٹی نوٹ:** کوئی API کی اسٹور کرنے کی ضرورت نہیں۔ آپ Microsoft Entra ID کے ذریعے `az login` (لوکل) یا managed identity (Azure میں) سے تصدیق کرتے ہیں۔ `.env` میں صرف غیر خفیہ ترتیبات ہوتی ہیں اور یہ پہلے ہی `.gitignore` میں شامل ہے۔

## اپنے سیٹ اپ کی جانچ کریں

یقینی بنائیں کہ آپ سائن ان ہیں تاکہ keyless تصدیق ٹوکن حاصل کر سکے، پھر مثال چلائیں:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # اگر آپ پہلے سے سائن ان نہیں ہیں
mvn clean spring-boot:run
```
  
آپ کو `gpt-4o-mini` ماڈل سے ردعمل دیکھنا چاہئے!

> **VS Code استعمال کرنے والے:** چلانے کے لیے `F5` دبائیں۔ ایپ خودکار طریقے سے آپ کی `.env` لوڈ کر لیتی ہے۔

> **مکمل مثال:** تفصیلات اور مسئلے حل کرنے کے لیے [Basic Chat with Azure AI Foundry example](./examples/basic-chat-azure/README.md) دیکھیں۔

## اگلا کیا ہے؟

**سیٹ اپ مکمل ہو گیا!** اب آپ کے پاس ہے:  
- Azure AI Foundry جس میں `gpt-4o-mini` اور `text-embedding-3-small` تعینات ہیں  
- Keyless تصدیق (Microsoft Entra ID) — کوئی کیز منیج کرنے کی ضرورت نہیں  
- ایک مقامی `.env` جس میں آپ کے اینڈپوائنٹ اور ڈیپلائمنٹ کے نام ہیں  
- تیار جاوا ترقیاتی ماحول  

**چلیں اگلے مرحلے کی طرف** [باب 3: کور جنریٹو AI تکنیکیں](../03-CoreGenerativeAITechniques/README.md) تاکہ AI ایپلیکیشنز بنانا شروع کریں!

## وسائل

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Microsoft Entra ID کے ساتھ Keyless تصدیق](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry دستاویزات](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI دستاویزات](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI جاوا SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## اضافی وسائل

- [VS Code ڈاؤن لوڈ کریں](https://code.visualstudio.com/Download)
- [Docker Desktop حاصل کریں](https://www.docker.com/products/docker-desktop)
- [Dev Container کی ترتیب](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ڈس کلیمر**:
یہ دستاویز AI ترجمہ سروس [Co-op Translator](https://github.com/Azure/co-op-translator) کے ذریعے ترجمہ کی گئی ہے۔ جبکہ ہم درستگی کے لیے کوشاں ہیں، براہ کرم اس بات سے آگاہ رہیں کہ خودکار ترجمے میں غلطیاں یا عدم درستیاں ہو سکتی ہیں۔ اصل دستاویز اپنے مادری زبان میں مستند ماخذ سمجھی جائے گی۔ حساس معلومات کے لیے پیشہ ور انسانی ترجمہ کی سفارش کی جاتی ہے۔ اس ترجمے کے استعمال سے پیدا ہونے والی کسی بھی غلط فہمی یا غلط تشریح کی ذمہ داری ہم قبول نہیں کرتے۔
<!-- CO-OP TRANSLATOR DISCLAIMER END -->