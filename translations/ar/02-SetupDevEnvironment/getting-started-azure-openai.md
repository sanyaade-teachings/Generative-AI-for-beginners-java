# إعداد بيئة التطوير لـ Azure AI Foundry

> يشرح هذا الدليل إعداد نماذج **Azure AI Foundry** لتطبيقات Java AI في هذا المقرر، باستخدام المصادقة **بدون مفتاح** (Microsoft Entra ID) — لا مفاتيح API لإدارتها. جديد على الأدوات؟ ابدأ بـ [دليل بيئة التطوير](./README.md).

يشرح هذا الدليل إعداد نماذج **Azure AI Foundry** لتطبيقات Java AI في هذا المقرر. لديك مساران:

- **الخيار أ — التهيئة باستخدام `azd` + Bicep (موصى به):** أمر واحد ينشر حساب Foundry والنماذج ككود. لا حاجة للنقر في البوابة.
- **الخيار ب — إنشاء الموارد يدويًا** في بوابة Azure AI Foundry.

يستخدم كلا المسارين **المصادقة بدون مفتاح** (Microsoft Entra ID) — لا توجد مفاتيح API لنسخها أو تسريبها.

## جدول المحتويات

- [ما يتم إنشاؤه](#ما-يتم-إنشاؤه)
- [المتطلبات السابقة](#المتطلبات-السابقة)
- [الخيار أ: التهيئة باستخدام azd + Bicep (موصى به)](#الخيار-أ-التهيئة-باستخدام-azd--bicep-موصى-به)
- [الخيار ب: إنشاء الموارد يدويًا](#الخيار-ب-إنشاء-الموارد-يدويًا)
- [تهيئة بيئتك](#تهيئة-بيئتك)
- [اختبار إعدادك](#اختبار-إعدادك)
- [ما التالي؟](#ما-التالي)
- [الموارد](#الموارد)
- [الموارد الإضافية](#الموارد-الإضافية)

## ما يتم إنشاؤه

تقوم قوالب Bicep في [`infra/`](../../../02-SetupDevEnvironment/infra) بتهيئة:

- حساب **Azure AI Foundry** (`Microsoft.CognitiveServices/accounts`، النوع `AIServices`) مع مشروع
- نشر **الدردشة** — `gpt-4o-mini`
- نشر **التضمين** — `text-embedding-3-small` (يُستخدم في الفصول اللاحقة)
- تعيين دور **بدون مفتاح** (`Cognitive Services OpenAI User`) بحيث تقوم بتسجيل الدخول باستخدام `az login` بدلاً من إدارة المفاتيح

## المتطلبات السابقة

- [اشتراك Azure](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) و [Maven 3.9+](https://maven.apache.org/download.cgi)

## الخيار أ: التهيئة باستخدام azd + Bicep (موصى به)

من مجلد `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# تسجيل الدخول (كلا الأداتين)
azd auth login
az login

# توفير حساب Foundry + نشر النماذج
azd up
```
  
يطلب `azd` اسم **البيئة** (على سبيل المثال `genai-java`) و **المنطقة**. اختر منطقة يتوفر فيها `gpt-4o-mini` و `text-embedding-3-small` — على سبيل المثال `eastus2` أو `swedencentral`.

عند الانتهاء من التهيئة، يقوم azd بـ:

1. نشر كل شيء معرف في [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep).
2. تشغيل برنامج نصي بعد التهيئة الذي يكتب [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) مع نقطة النهاية وأسماء النشر الخاصة بك (دون أسرار).

> **نصيحة:** أعد تشغيل `azd up` في أي وقت لتطبيق التغييرات. شغل `azd down` لحذف كل شيء وإيقاف التكاليف.

لعرض الإعدادات التي تم إنشاؤها:

```bash
azd env get-values
```
  
تخطى الآن إلى [اختبار إعدادك](#اختبار-إعدادك).

## الخيار ب: إنشاء الموارد يدويًا

تفضل البوابة؟ أنشئ الموارد بنفسك:

1. انتقل إلى [بوابة Azure AI Foundry](https://ai.azure.com/) وسجّل الدخول.
2. **أنشئ مشروعًا** (يؤدي هذا أيضًا إلى إنشاء مورد AI Foundry). أعطه اسمًا مثل `GenAIJava`.
3. في مشروعك، افتح **النماذج + النقاط النهائية** → **نشر نموذج** → **نشر نموذج أساسي**.
4. انشر **gpt-4o-mini** (اسم النشر `gpt-4o-mini`). كرر لـ **text-embedding-3-small** إذا أردت أمثلة التضمين.
5. من **نظرة عامة**، انسخ **نقطة النهاية** (على سبيل المثال `https://<resource>.openai.azure.com/`).
6. امنح نفسك وصولًا بدون مفتاح: على المورد، افتح **التحكم في الوصول (IAM)** → **إضافة تعيين دور** → عيّن **Cognitive Services OpenAI User** على حسابك.

> **لا تزال تواجه مشكلة؟** راجع [وثائق Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## تهيئة بيئتك

**إذا استخدمت الخيار أ (`azd up`)**، تم كتابة ملف الإعدادات الخاص بك بالفعل — لا حاجة لأي تكوين. انتقل إلى [اختبار إعدادك](#اختبار-إعدادك).

**إذا استخدمت الخيار ب (يدويًا)**، أنشئ ملف `.env` الخاص بالمثال بنفسك:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```
  
حرر `.env` لنقطة النهاية الخاصة بك (بدون مفتاح — المصادقة بدون مفتاح):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```
  
> **ملاحظة أمان:** لا يوجد مفتاح API يجب تخزينه. تقوم بالمصادقة عبر Microsoft Entra ID بواسطة `az login` (محليًا) أو هوية مُدارة (في Azure). يحتوي ملف `.env` فقط على إعدادات غير سرية وهو مغطى بالفعل في `.gitignore`.

## اختبار إعدادك

تأكد من تسجيل الدخول حتى تتمكن المصادقة بدون مفتاح من الحصول على رمز، ثم شغل المثال:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # إذا لم تكن قد سجلت الدخول بالفعل
mvn clean spring-boot:run
```
  
يجب أن ترى استجابة من نموذج `gpt-4o-mini`!

> **مستخدمو VS Code:** اضغط `F5` للتشغيل. يقوم التطبيق بتحميل ملف `.env` تلقائيًا.

> **المثال الكامل:** راجع [مثال الدردشة الأساسية مع Azure AI Foundry](./examples/basic-chat-azure/README.md) للتفاصيل وحل المشاكل.

## ما التالي؟

**الإعداد مكتمل!** لديك الآن:  
- Azure AI Foundry مع `gpt-4o-mini` و `text-embedding-3-small` منشوران  
- المصادقة بدون مفتاح (Microsoft Entra ID) — لا مفاتيح لإدارتها  
- ملف `.env` محلي مع نقطة النهاية وأسماء النشر  
- بيئة تطوير Java جاهزة للعمل

**تابع إلى** [الفصل 3: تقنيات الذكاء الاصطناعي التوليدي الأساسية](../03-CoreGenerativeAITechniques/README.md) للبدء في بناء تطبيقات الذكاء الاصطناعي!

## الموارد

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [المصادقة بدون مفتاح مع Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [وثائق Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)
- [توثيق Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## الموارد الإضافية

- [تحميل VS Code](https://code.visualstudio.com/Download)
- [الحصول على Docker Desktop](https://www.docker.com/products/docker-desktop)
- [تكوين حاوية التطوير](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**تنويه**:
تمت ترجمة هذا المستند باستخدام خدمة الترجمة بالذكاء الاصطناعي [Co-op Translator](https://github.com/Azure/co-op-translator). بينما نسعى للدقة، يرجى العلم أن الترجمات الآلية قد تحتوي على أخطاء أو عدم دقة. يجب اعتبار المستند الأصلي بلغته الأصلية المصدر الرسمي والمعتمد. للمعلومات الهامة، يُنصح بالاستعانة بترجمة بشرية محترفة. نحن غير مسؤولين عن أي سوء فهم أو تفسير ناتج عن استخدام هذه الترجمة.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->