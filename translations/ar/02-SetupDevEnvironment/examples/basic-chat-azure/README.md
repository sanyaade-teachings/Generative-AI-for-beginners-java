# الدردشة الأساسية مع Azure AI Foundry - مثال شامل من البداية للنهاية

هذا المثال هو تطبيق بسيط باستخدام Spring Boot يتصل بنموذج **Azure AI Foundry** باستخدام **المصادقة بدون مفتاح** (Microsoft Entra ID) ويختبر إعدادك. يستخدم عميل الدردشة `ChatClient` من Spring AI.

## جدول المحتويات

- [المتطلبات الأساسية](#المتطلبات-الأساسية)
- [بدء سريع](#بدء-سريع)
- [كيفية عمل المصادقة](#كيفية-عمل-المصادقة)
- [تشغيل التطبيق](#تشغيل-التطبيق)
  - [باستخدام Maven](#باستخدام-maven)
  - [باستخدام VS Code](#باستخدام-vs-code)
  - [الناتج المتوقع](#الناتج-المتوقع)
- [مرجع التهيئة](#مرجع-التهيئة)
  - [المتغيرات البيئية](#المتغيرات-البيئية)
  - [تهيئة Spring](#تهيئة-spring)
- [استكشاف الأخطاء وإصلاحها](#استكشاف-الأخطاء-وإصلاحها)
  - [المشاكل الشائعة](#المشاكل-الشائعة)
  - [وضع التصحيح](#وضع-التصحيح)
- [الخطوات التالية](#الخطوات-التالية)
- [الموارد](#الموارد)

## المتطلبات الأساسية

قبل تشغيل هذا المثال، تأكد من:

- وجود مورد Azure AI Foundry مع نشر `gpt-4o-mini` — قم بإعداده باستخدام `azd up` أو يدويًا عبر [دليل إعداد Azure AI Foundry](../../getting-started-azure-openai.md)
- دور **مستخدم خدمات الإدراك OpenAI** على هذا المورد (تقوم قوالب Bicep بتعيين هذا الدور لك)
- وجود [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli) مسجل الدخول باستخدام `az login`
- Java 21+ و Maven 3.9+

> **لا حاجة لمفتاح API** — المصادقة بدون مفتاح عبر Microsoft Entra ID.

## بدء سريع

```bash
# 1. انتقل إلى المشروع
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. قم بتسجيل الدخول حتى تتمكن المصادقة بدون مفتاح من الحصول على رمز
az login

# 3. قم بتكوين نقطة النهاية
#    - إذا قمت بتشغيل `azd up`، فقد تم كتابة ملف .env لك (تخط هذا).
#    - وإلا قم بنسخ القالب واضبط AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. شغّل التطبيق
mvn spring-boot:run
```

## كيفية عمل المصادقة

هذا المثال يقوم بالمصادقة باستخدام **Microsoft Entra ID** — لا يوجد مفتاح API.

عندما يتم تعيين `spring.ai.azure.openai.endpoint` فقط (دون وجود api-key)، يقوم Spring AI بإنشاء عميل Azure OpenAI باستخدام [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). هذه الاعتمادات تعثر تلقائيًا على رمز من جلسة `az login` المحلية، أو من هوية مُدارة عند التشغيل في Azure — لذلك يعمل نفس الكود في كلا المكانين دون تغييرات.

## تشغيل التطبيق

### باستخدام Maven

```bash
mvn spring-boot:run
```

### باستخدام VS Code

1. افتح المشروع في VS Code
2. اضغط على `F5` أو استخدم نافذة "تشغيل وتصحيح"
3. اختر تكوين "Spring Boot-BasicChatApplication"

> **ملاحظة**: يقوم تكوين VS Code بتحميل ملف .env تلقائيًا

### الناتج المتوقع

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

## مرجع التهيئة

### المتغيرات البيئية

| المتغير | الوصف | مطلوب | مثال |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | عنوان نقطة النهاية Foundry (Azure OpenAI) | نعم | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | اسم نشر نموذج الدردشة | لا | `gpt-4o-mini` (الافتراضي) |

> لا يوجد متغير مفتاح API — المصادقة بدون مفتاح (Microsoft Entra ID عبر `az login`).

### تهيئة Spring

ملف `application.yml` يقوم بتهيئة:
- **نقطة النهاية**: `${AZURE_OPENAI_ENDPOINT}` - من المتغير البيئي
- **النشر**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - من المتغير البيئي مع قيمة احتياطية
- **المصادقة**: بدون مفتاح — لا يتم تعيين `api-key`، لذا يستخدم Spring AI `DefaultAzureCredential`
- **درجة الحرارة**: `0.7` - تتحكم بالإبداع (0.0 = حتمي، 1.0 = إبداعي)
- **الحد الأقصى للرموز**: `500` - الحد الأقصى لطول الاستجابة

## استكشاف الأخطاء وإصلاحها

### المشاكل الشائعة

<details>
<summary><strong>خطأ: 401 / "PermissionDenied" / أخطاء الرمز</strong></summary>

- قم بتشغيل `az login` — المصادقة بدون مفتاح تحتاج إلى تسجيل دخول نشط للحصول على رمز
- تحقق من أن حسابك لديه دور **مستخدم خدمات الإدراك OpenAI** على المورد
- إذا قمت بتعيين الدور للتو، انتظر دقيقة حتى يتم تطبيقه
- تأكد من أنك في المستأجر/الاشتراك الصحيح (`az account show`)
</details>

<details>
<summary><strong>خطأ: "النقطة النهاية غير صحيحة" / أخطاء الاتصال</strong></summary>

- تأكد أن `AZURE_OPENAI_ENDPOINT` هو عنوان URL الأساسي الكامل (مثل `https://your-resource.openai.azure.com/`)
- تحقق من تناسق وجودة شرطة النهاية /
- تحقق من أن نقطة النهاية تطابق المورد الذي قمت بإعداده (`azd env get-values`)
</details>

<details>
<summary><strong>خطأ: "النشر غير موجود"</strong></summary>

- تحقق أن `AZURE_OPENAI_DEPLOYMENT` يطابق اسم نشر في Azure
- تحقق أن النموذج منشور وفعال بنجاح
- اسم النشر الافتراضي هو `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: متغيرات البيئة لا تُحمّل</strong></summary>

- تأكد من أن ملف `.env` موجود في جذر المشروع (على نفس مستوى `pom.xml`)
- جرب تشغيل `mvn spring-boot:run` في الطرفية المدمجة في VS Code
- تحقق من تثبيت امتداد Java في VS Code بشكل صحيح
</details>

### وضع التصحيح

لتمكين التسجيل المفصل، قم بإلغاء تعليق هذه الأسطر في `application.yml`:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## الخطوات التالية

**تم الانتهاء من الإعداد!** استمر في رحلة التعلم الخاصة بك:

[الفصل 3: تقنيات الذكاء التوليدي الأساسية](../../../03-CoreGenerativeAITechniques/README.md)

## الموارد

- [توثيق Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [المصادقة بدون مفتاح باستخدام Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [بوابة Azure AI Foundry](https://ai.azure.com/)
- [توثيق Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**تنويه**:
تمت ترجمة هذا المستند باستخدام خدمة الترجمة بالذكاء الاصطناعي [Co-op Translator](https://github.com/Azure/co-op-translator). بينما نسعى للدقة، يرجى العلم أن الترجمات الآلية قد تحتوي على أخطاء أو عدم دقة. يجب اعتبار المستند الأصلي بلغته الأصلية المصدر الرسمي والمعتمد. للمعلومات الهامة، يُنصح بالاستعانة بترجمة بشرية محترفة. نحن غير مسؤولين عن أي سوء فهم أو تفسير ناتج عن استخدام هذه الترجمة.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->