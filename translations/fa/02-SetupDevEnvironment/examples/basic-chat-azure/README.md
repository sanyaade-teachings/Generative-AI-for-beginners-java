# چت پایه با Azure AI Foundry - نمونه انتها به انتها

این نمونه یک برنامه ساده Spring Boot است که به مدل **Azure AI Foundry** با استفاده از **احراز هویت بدون کلید** (Microsoft Entra ID) متصل می‌شود و تنظیمات شما را آزمایش می‌کند. از `ChatClient` در Spring AI استفاده می‌کند.

## فهرست مطالب

- [پیش‌نیازها](#پیش‌نیازها)
- [شروع سریع](#شروع-سریع)
- [نحوه عملکرد احراز هویت](#نحوه-عملکرد-احراز-هویت)
- [اجرای برنامه](#اجرای-برنامه)
  - [استفاده از Maven](#استفاده-از-maven)
  - [استفاده از VS Code](#استفاده-از-vs-code)
  - [خروجی مورد انتظار](#خروجی-مورد-انتظار)
- [مرجع پیکربندی](#مرجع-پیکربندی)
  - [متغیرهای محیطی](#متغیرهای-محیطی)
  - [پیکربندی Spring](#پیکربندی-spring)
- [رفع اشکال](#رفع-اشکال)
  - [مشکلات رایج](#مشکلات-رایج)
  - [حالت اشکال‌زدایی](#حالت-اشکال‌زدایی)
- [گام‌های بعدی](#گام‌های-بعدی)
- [منابع](#منابع)

## پیش‌نیازها

قبل از اجرای این نمونه، مطمئن شوید که:

- یک منبع Azure AI Foundry با استقرار `gpt-4o-mini` دارید — آن را با `azd up` فراهم کنید یا دستی از طریق [راهنمای راه‌اندازی Azure AI Foundry](../../getting-started-azure-openai.md)
- نقش **Cognitive Services OpenAI User** در آن منبع دارید (قالب‌های Bicep این نقش را برای شما اختصاص می‌دهند)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli) را نصب کرده و با `az login` وارد شده‌اید
- Java 21+ و Maven 3.9+ نصب شده است

> **کلید API لازم نیست** — احراز هویت بدون کلید از طریق Microsoft Entra ID انجام می‌شود.

## شروع سریع

```bash
# ۱. به پروژه بروید
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# ۲. وارد شوید تا احراز هویت بدون کلید بتواند توکن بگیرد
az login

# ۳. نقطه انتهایی را پیکربندی کنید
#    - اگر دستور `azd up` را اجرا کردید، فایل .env برای شما نوشته شده است (این مرحله را رد کنید).
#    - در غیر این صورت قالب را کپی کرده و AZURE_OPENAI_ENDPOINT را تنظیم کنید:
cp .env.example .env

# ۴. برنامه را اجرا کنید
mvn spring-boot:run
```

## نحوه عملکرد احراز هویت

این نمونه با **Microsoft Entra ID** احراز هویت می‌کند — کلید API وجود ندارد.

وقتی فقط `spring.ai.azure.openai.endpoint` تنظیم شده باشد (و کلید API نباشد)، Spring AI کلاینت Azure OpenAI را با [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential) می‌سازد. این اعتبارنامه به طور خودکار توکنی را از جلسه `az login` شما به صورت محلی یا از هویت مدیریت شده هنگام اجرا در Azure پیدا می‌کند — بنابراین همان کد در هر دو محیط بدون تغییر کار می‌کند.

## اجرای برنامه

### استفاده از Maven

```bash
mvn spring-boot:run
```

### استفاده از VS Code

1. پروژه را در VS Code باز کنید
2. کلید `F5` را بزنید یا از پنل "Run and Debug" استفاده کنید
3. پیکربندی "Spring Boot-BasicChatApplication" را انتخاب کنید

> **توجه**: پیکربندی VS Code به طور خودکار فایل .env شما را بارگذاری می‌کند

### خروجی مورد انتظار

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

## مرجع پیکربندی

### متغیرهای محیطی

| متغیر | توضیحات | الزامی | مثال |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | آدرس پایانه Foundry (Azure OpenAI) | بله | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | نام استقرار مدل چت | خیر | `gpt-4o-mini` (پیش‌فرض) |

> متغیر کلید API **وجود ندارد** — احراز هویت بدون کلید (Microsoft Entra ID از طریق `az login`) است.

### پیکربندی Spring

فایل `application.yml` تنظیمات زیر را انجام می‌دهد:
- **پایانه**: `${AZURE_OPENAI_ENDPOINT}` - از متغیر محیطی
- **استقرار**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - از متغیر محیطی با مقدار پیش‌فرض
- **احراز هویت**: بدون کلید — کلید API تنظیم نشده، پس Spring AI از `DefaultAzureCredential` استفاده می‌کند
- **دمای پاسخ**: `0.7` - کنترل خلاقیت (0.0 = قطعی، 1.0 = خلاقانه)
- **حداکثر توکن‌ها**: `500` - حداکثر طول پاسخ

## رفع اشکال

### مشکلات رایج

<details>
<summary><strong>خطا: 401 / "PermissionDenied" / خطاهای توکن</strong></summary>

- دستور `az login` را اجرا کنید — احراز هویت بدون کلید به ورود فعال برای دریافت توکن نیاز دارد
- بررسی کنید حساب شما نقش **Cognitive Services OpenAI User** را بر روی منبع دارد
- اگر همین الان نقش را اختصاص داده‌اید، یک دقیقه صبر کنید تا اعمال شود
- تأیید کنید در tenant/اشتراک مناسب هستید (`az account show`)
</details>

<details>
<summary><strong>خطا: "The endpoint is not valid" / خطاهای اتصال</strong></summary>

- مطمئن شوید `AZURE_OPENAI_ENDPOINT` آدرس کامل پایه است (مثلاً `https://your-resource.openai.azure.com/`)
- سازگاری در پایان آدرس با اسلش بررسی شود
- مطمئن شوید پایانه با منبع فراهم شده شما مطابقت دارد (`azd env get-values`)
</details>

<details>
<summary><strong>خطا: "The deployment was not found"</strong></summary>

- بررسی کنید `AZURE_OPENAI_DEPLOYMENT` با نام یک استقرار در Azure مطابقت دارد
- تأیید کنید مدل با موفقیت مستقر و فعال است
- نام پیش‌فرض استقرار `gpt-4o-mini` است
</details>

<details>
<summary><strong>VS Code: بارگذاری نشدن متغیرهای محیطی</strong></summary>

- مطمئن شوید فایل `.env` در ریشه پروژه (سطح همان `pom.xml`) قرار دارد
- تلاش کنید `mvn spring-boot:run` را در ترمینال داخلی VS Code اجرا کنید
- بررسی کنید افزونه Java برای VS Code به درستی نصب شده باشد
</details>

### حالت اشکال‌زدایی

برای فعال‌سازی نمایش دقیق لاگ‌ها، این خطوط را در `application.yml` لغو کامنت کنید:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## گام‌های بعدی

**راه‌اندازی کامل است!** مسیر یادگیری خود را ادامه دهید:

[فصل ۳: تکنیک‌های اصلی هوش مصنوعی مولد](../../../03-CoreGenerativeAITechniques/README.md)

## منابع

- [مستندات Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [احراز هویت بدون کلید با Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [پرتال Azure AI Foundry](https://ai.azure.com/)
- [مستندات Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**سلب مسئولیت**:
این سند با استفاده از سرویس ترجمه هوش مصنوعی [Co-op Translator](https://github.com/Azure/co-op-translator) ترجمه شده است. در حالی که ما در تلاش برای دقت هستیم، لطفاً توجه داشته باشید که ترجمه‌های خودکار ممکن است شامل خطاها یا نادرستی‌هایی باشند. سند اصلی به زبان مادری خود باید به عنوان منبع معتبر در نظر گرفته شود. برای اطلاعات حیاتی، ترجمه حرفه‌ای انسانی توصیه می‌شود. ما در قبال هرگونه سوء تفاهم یا برداشت نادرست ناشی از استفاده از این ترجمه مسئولیتی نداریم.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->