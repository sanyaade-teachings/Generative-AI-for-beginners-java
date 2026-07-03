# راه‌اندازی محیط توسعه برای Azure AI Foundry

> این راهنما مدل‌های **Azure AI Foundry** را برای برنامه‌های جاوای هوش مصنوعی در این دوره با استفاده از احراز هویت بدون کلید (Microsoft Entra ID) تنظیم می‌کند — بدون نیاز به مدیریت کلیدهای API. به ابزارها تازه واردید؟ با [راهنمای محیط توسعه](./README.md) شروع کنید.

این راهنما مدل‌های **Azure AI Foundry** را برای برنامه‌های جاوای هوش مصنوعی در این دوره تنظیم می‌کند. شما دو مسیر دارید:

- **گزینه A — راه‌اندازی با `azd` + Bicep (توصیه شده):** با یک دستور حساب Foundry و مدل‌ها را به عنوان کد مستقر می‌کند. بدون کلیک در پرتال.
- **گزینه B — ایجاد منابع به صورت دستی** در پرتال Azure AI Foundry.

هر دو مسیر از **احراز هویت بدون کلید** (Microsoft Entra ID) استفاده می‌کنند — نیازی به کپی یا افشای کلیدهای API نیست.

## فهرست مطالب

- [چه مواردی ایجاد می‌شوند](#چه-مواردی-ایجاد-می‌شوند)
- [پیش‌نیازها](#پیش‌نیازها)
- [گزینه A: راه‌اندازی با azd + Bicep (توصیه شده)](#گزینه-a-راه‌اندازی-با-azd--bicep-توصیه-شده)
- [گزینه B: ایجاد منابع به صورت دستی](#گزینه-b-ایجاد-منابع-به-صورت-دستی)
- [پیکربندی محیط شما](#پیکربندی-محیط-شما)
- [آزمون تنظیمات شما](#آزمون-تنظیمات-شما)
- [بعدی چیست؟](#بعدی-چیست)
- [منابع](#منابع)
- [منابع اضافی](#منابع-اضافی)

## چه مواردی ایجاد می‌شوند

قالب‌های Bicep در [`infra/`](../../../02-SetupDevEnvironment/infra) موارد زیر را فراهم می‌کنند:

- یک حساب **Azure AI Foundry** (`Microsoft.CognitiveServices/accounts`، نوع `AIServices`) با یک پروژه
- یک استقرار **گفتگو** — `gpt-4o-mini`
- یک استقرار **تعبیه** — `text-embedding-3-small` (در فصل‌های بعدی استفاده می‌شود)
- یک **تخصیص نقش بدون کلید** (`Cognitive Services OpenAI User`) تا به جای مدیریت کلیدها با `az login` وارد شوید

## پیش‌نیازها

- یک [اشتراک Azure](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [جاوا ۲۱ به بالا](https://learn.microsoft.com/java/openjdk/download) و [Maven 3.9 به بالا](https://maven.apache.org/download.cgi)

## گزینه A: راه‌اندازی با azd + Bicep (توصیه شده)

از پوشه `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# ورود به سیستم (هر دو ابزار)
azd auth login
az login

# فراهم‌سازی حساب Foundry + استقرار مدل‌ها
azd up
```
  
`azd` از شما یک **نام محیط** (مثلاً `genai-java`) و یک **منطقه** می‌پرسد. منطقه‌ای را انتخاب کنید که `gpt-4o-mini` و `text-embedding-3-small` در آن در دسترس باشند — برای مثال `eastus2` یا `swedencentral`.

وقتی راه‌اندازی کامل شد، azd:

1. همه موارد تعریف شده در [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep) را مستقر می‌کند.  
2. یک قلاب پس از استقرار اجرا می‌کند که فایل [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) را با نام‌های نقطه انتهایی و استقرار شما می‌نویسد (بدون اسرار).

> **نکته:** هر موقع بخواهید تغییرات را اعمال کنید `azd up` را دوباره اجرا کنید. برای پاک کردن همه چیز و حذف هزینه `azd down` را اجرا کنید.

برای دیدن تنظیمات تولید شده:

```bash
azd env get-values
```
  
حالا به [آزمون تنظیمات شما](#آزمون-تنظیمات-شما) بروید.

## گزینه B: ایجاد منابع به صورت دستی

ترجیح می‌دهید از پرتال استفاده کنید؟ منابع را به صورت دستی ایجاد کنید:

1. به [پرتال Azure AI Foundry](https://ai.azure.com/) بروید و وارد شوید.  
2. **یک پروژه ایجاد کنید** (این نیز یک منبع AI Foundry می‌سازد). نامی مثل `GenAIJava` بدهید.  
3. در پروژه خود، به **Models + endpoints** → **Deploy model** → **Deploy base model** بروید.  
4. مدل **gpt-4o-mini** را مستقر کنید (نام استقرار `gpt-4o-mini`). برای مثال‌های تعبیه، همین کار را برای **text-embedding-3-small** تکرار کنید.  
5. از قسمت **Overview**، **endpoint** را کپی کنید (مثلاً `https://<resource>.openai.azure.com/`).  
6. دسترسی بدون کلید به خود بدهید: در منبع، به **Access control (IAM)** → **Add role assignment** بروید و نقش **Cognitive Services OpenAI User** را به حساب خود اختصاص دهید.

> **هنوز مشکل دارید؟** به [مستندات Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects) مراجعه کنید.

## پیکربندی محیط شما

**اگر گزینه A (`azd up`) را استفاده کرده‌اید**، فایل تنظیمات شما قبلاً نوشته شده — نیازی به پیکربندی نیست. به [آزمون تنظیمات شما](#آزمون-تنظیمات-شما) بروید.

**اگر گزینه B (دستی)** را استفاده کرده‌اید، فایل `.env` نمونه را خودتان ایجاد کنید:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```
  
فایل `.env` را با نقطه انتهایی خود ویرایش کنید (بدون کلید — احراز هویت بدون کلید است):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```
  
> **توجه امنیتی:** کلید API برای ذخیره وجود ندارد. شما با Microsoft Entra ID از طریق `az login` (محلی) یا یک هویت مدیریتی (در Azure) احراز هویت می‌کنید. فایل `.env` فقط تنظیمات غیرمحرمانه را نگه می‌دارد و در `.gitignore` لحاظ شده است.

## آزمون تنظیمات شما

مطمئن شوید وارد شده‌اید تا احراز هویت بدون کلید بتواند توکن دریافت کند، سپس نمونه را اجرا کنید:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # اگر هنوز وارد نشده‌اید
mvn clean spring-boot:run
```
  
باید پاسخی از مدل `gpt-4o-mini` ببینید!

> **کاربران VS Code:** برای اجرا `F5` را فشار دهید. برنامه فایل `.env` شما را به طور خودکار بارگیری می‌کند.

> **نمونه کامل:** برای جزئیات و رفع اشکال به [مثال گفتگوی پایه با Azure AI Foundry](./examples/basic-chat-azure/README.md) مراجعه کنید.

## بعدی چیست؟

**راه‌اندازی کامل شد!** شما اکنون دارید:  
- Azure AI Foundry با `gpt-4o-mini` و `text-embedding-3-small` مستقر شده  
- احراز هویت بدون کلید (Microsoft Entra ID) — بدون کلید برای مدیریت  
- یک فایل محلی `.env` با نقطه انتهایی و نام‌های استقرار  
- یک محیط توسعه جاوا آماده به کار

**ادامه دهید به** [فصل ۳: تکنیک‌های اصلی هوش مصنوعی مولد](../03-CoreGenerativeAITechniques/README.md) برای شروع ساخت برنامه‌های هوش مصنوعی!

## منابع

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)  
- [احراز هویت بدون کلید با Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)  
- [مستندات Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)  
- [مستندات Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)  
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## منابع اضافی

- [دانلود VS Code](https://code.visualstudio.com/Download)  
- [دریافت Docker Desktop](https://www.docker.com/products/docker-desktop)  
- [پیکربندی Dev Container](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**سلب مسئولیت**:
این سند با استفاده از سرویس ترجمه هوش مصنوعی [Co-op Translator](https://github.com/Azure/co-op-translator) ترجمه شده است. در حالی که ما در تلاش برای دقت هستیم، لطفاً توجه داشته باشید که ترجمه‌های خودکار ممکن است شامل خطاها یا نادرستی‌هایی باشند. سند اصلی به زبان مادری خود باید به عنوان منبع معتبر در نظر گرفته شود. برای اطلاعات حیاتی، ترجمه حرفه‌ای انسانی توصیه می‌شود. ما در قبال هرگونه سوء تفاهم یا برداشت نادرست ناشی از استفاده از این ترجمه مسئولیتی نداریم.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->