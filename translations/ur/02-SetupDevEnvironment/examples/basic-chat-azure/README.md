# Azure AI Foundry کے ساتھ بنیادی چیٹ - مکمل مثال

یہ مثال ایک سادہ Spring Boot ایپلیکیشن ہے جو **Azure AI Foundry** ماڈل سے **کلید سے پاک تصدیق** (Microsoft Entra ID) کے ذریعے جڑتی ہے اور آپ کی ترتیب کا تجربہ کرتی ہے۔ یہ Spring AI کے `ChatClient` کا استعمال کرتی ہے۔

## فہرست مضامین

- [ضروریات](#ضروریات)
- [فوری آغاز](#فوری-آغاز)
- [تصدیق کیسے کام کرتی ہے](#تصدیق-کیسے-کام-کرتی-ہے)
- [ایپلیکیشن چلانا](#ایپلیکیشن-چلانا)
  - [Maven کا استعمال](#maven-کا-استعمال)
  - [VS Code کا استعمال](#vs-code-کا-استعمال)
  - [متوقع نتیجہ](#متوقع-نتیجہ)
- [ترتیب کا حوالہ](#ترتیب-کا-حوالہ)
  - [ماحولیاتی متغیرات](#ماحولیاتی-متغیرات)
  - [Spring کی ترتیب](#spring-کی-ترتیب)
- [مسائل کا حل](#مسائل-کا-حل)
  - [عام مسائل](#عام-مسائل)
  - [ڈیبگ موڈ](#ڈیبگ-موڈ)
- [اگلے اقدامات](#اگلے-اقدامات)
- [وسائل](#وسائل)

## ضروریات

اس مثال کو چلانے سے پہلے یقینی بنائیں کہ آپ نے:

- Azure AI Foundry کا ایک وسیلہ جس میں `gpt-4o-mini` تعیناتی ہو — اسے `azd up` سے فراہم کریں یا دستی طور پر [Azure AI Foundry سیٹ اپ گائیڈ](../../getting-started-azure-openai.md) کے ذریعے کریں
- اس وسیلہ پر **Cognitive Services OpenAI User** کا رول ہونا (Bicep ٹیمپلیٹس آپ کے لیے فراہم کرتے ہیں)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli) اور `az login` کے ذریعے سائن ان ہونا
- Java 21+ اور Maven 3.9+

> **کوئی API کلید درکار نہیں** — تصدیق Microsoft Entra ID کے ذریعے کلید سے پاک ہے۔

## فوری آغاز

```bash
# 1. پروجیکٹ پر جائیں
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. سائن ان کریں تاکہ کی لیس توثیق ٹوکن حاصل کر سکے
az login

# 3. اینڈپوائنٹ کو ترتیب دیں
#    - اگر آپ نے `azd up` چلایا تھا، تو .env آپ کے لیے لکھا گیا تھا (اسے چھوڑ دیں).
#    - ورنہ ٹیمپلیٹ کو کاپی کریں اور AZURE_OPENAI_ENDPOINT سیٹ کریں:
cp .env.example .env

# 4. ایپلیکیشن چلائیں
mvn spring-boot:run
```

## تصدیق کیسے کام کرتی ہے

یہ مثال **Microsoft Entra ID** کے ساتھ تصدیق کرتی ہے — کوئی API کلید نہیں ہے۔

جب صرف `spring.ai.azure.openai.endpoint` سیٹ ہوتی ہے (اور کوئی api-key نہیں)، تو Spring AI Azure OpenAI کلائنٹ کو [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential) کے ساتھ بناتا ہے۔ یہ اس سند کو خودکار طور پر آپ کے `az login` سیشن سے لوکل ٹوکن تلاش کرتا ہے، یا جب Azure میں چل رہا ہو تو Managed Identity سے — اس طرح ایک ہی کوڈ دونوں جگہ بغیر تبدیلی کے کام کرتا ہے۔

## ایپلیکیشن چلانا

### Maven کا استعمال

```bash
mvn spring-boot:run
```

### VS Code کا استعمال

1. VS Code میں پروجیکٹ کھولیں
2. `F5` دبائیں یا "Run and Debug" پینل استعمال کریں
3. "Spring Boot-BasicChatApplication" کنفیگریشن منتخب کریں

> **نوٹ**: VS Code کنفیگریشن خودکار طور پر آپ کی `.env` فائل لوڈ کر لیتا ہے

### متوقع نتیجہ

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

## ترتیب کا حوالہ

### ماحولیاتی متغیرات

| متغیر | وضاحت | ضروری | مثال |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | Foundry (Azure OpenAI) کا اینڈپوائنٹ URL | ہاں | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | چیٹ ماڈل کی تعیناتی کا نام | نہیں | `gpt-4o-mini` (ڈفالٹ) |

> **کوئی** API کلید متغیر نہیں ہے — تصدیق کلید سے پاک ہے (Microsoft Entra ID کے ذریعے `az login`).

### Spring کی ترتیب

`application.yml` فائل کی ترتیب:
- **اینڈپوائنٹ**: `${AZURE_OPENAI_ENDPOINT}` - ماحولیاتی متغیر سے
- **تعیناتی**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - ماحولیاتی متغیر سے، متبادل کے ساتھ
- **تصدیق**: کلید سے پاک — کوئی `api-key` نہیں، اس لیے Spring AI `DefaultAzureCredential` استعمال کرتا ہے
- **Temperature**: `0.7` - تخلیقی صلاحیت کو کنٹرول کرتا ہے (0.0 = قطعیت پسند، 1.0 = تخلیقی)
- **زیادہ سے زیادہ ٹوکنز**: `500` - زیادہ سے زیادہ جواب کی لمبائی

## مسائل کا حل

### عام مسائل

<details>
<summary><strong>غلطی: 401 / "PermissionDenied" / ٹوکن کی غلطیاں</strong></summary>

- `az login` چلائیں — کلید سے پاک تصدیق کو ٹوکن حاصل کرنے کے لیے ایک فعال سائن ان کی ضرورت ہے
- تصدیق کریں کہ آپ کے اکاؤنٹ پر وسیلہ کے لیے **Cognitive Services OpenAI User** کا رول ہے
- اگر آپ نے ابھی رول تفویض کیا ہے تو اس کے پھیلاؤ کیلئے ایک منٹ انتظار کریں
- جانچ کریں کہ آپ درست ٹیننٹ/سبسکرپشن میں ہیں (`az account show`)
</details>

<details>
<summary><strong>غلطی: "The endpoint is not valid" / کنکشن کی غلطیاں</strong></summary>

- یقینی بنائیں کہ `AZURE_OPENAI_ENDPOINT` مکمل بنیادی URL ہے (مثلاً `https://your-resource.openai.azure.com/`)
- ٹریلنگ سلیش کی مطابقت چیک کریں
- تصدیق کریں کہ اینڈپوائنٹ آپ کے فراہم کردہ وسیلہ سے میل کھاتا ہے (`azd env get-values`)
</details>

<details>
<summary><strong>غلطی: "The deployment was not found"</strong></summary>

- تصدیق کریں کہ `AZURE_OPENAI_DEPLOYMENT` Azure میں کسی تعیناتی کے نام سے میل کھاتا ہو
- جانچ کریں کہ ماڈل کامیابی سے تعینات اور فعال ہے
- ڈیفالٹ تعیناتی کا نام `gpt-4o-mini` ہے
</details>

<details>
<summary><strong>VS Code: ماحولیاتی متغیرات لوڈ نہیں ہو رہے</strong></summary>

- یقینی بنائیں کہ آپ کی `.env` فائل پروجیکٹ کی روٹ ڈائریکٹری میں ہے (وہیں جہاں `pom.xml` ہے)
- VS Code کی انٹیگریٹڈ ٹرمینل میں `mvn spring-boot:run` چلانے کی کوشش کریں
- چیک کریں کہ VS Code Java ایکسٹینشن صحیح طریقے سے انسٹال ہے
</details>

### ڈیبگ موڈ

تفصیلی لاگنگ کو فعال کرنے کے لیے `application.yml` میں یہ لائنیں ان کومینٹ کریں:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## اگلے اقدامات

**سیٹ اپ مکمل!** اپنی سیکھنے کا سفر جاری رکھیں:

[باب 3: بنیادی تخلیقی AI تکنیکیں](../../../03-CoreGenerativeAITechniques/README.md)

## وسائل

- [Spring AI Azure OpenAI دستاویزات](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Microsoft Entra ID کے ساتھ کلید سے پاک تصدیق](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry پورٹل](https://ai.azure.com/)
- [Azure AI Foundry دستاویزات](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ڈس کلیمر**:
یہ دستاویز AI ترجمہ سروس [Co-op Translator](https://github.com/Azure/co-op-translator) کے ذریعے ترجمہ کی گئی ہے۔ جبکہ ہم درستگی کے لیے کوشاں ہیں، براہ کرم اس بات سے آگاہ رہیں کہ خودکار ترجمے میں غلطیاں یا عدم درستیاں ہو سکتی ہیں۔ اصل دستاویز اپنے مادری زبان میں مستند ماخذ سمجھی جائے گی۔ حساس معلومات کے لیے پیشہ ور انسانی ترجمہ کی سفارش کی جاتی ہے۔ اس ترجمے کے استعمال سے پیدا ہونے والی کسی بھی غلط فہمی یا غلط تشریح کی ذمہ داری ہم قبول نہیں کرتے۔
<!-- CO-OP TRANSLATOR DISCLAIMER END -->