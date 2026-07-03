# راه‌اندازی محیط توسعه برای هوش مصنوعی مولد جاوا

> **شروع سریع:** مدل‌های هوش مصنوعی خود را در **Azure AI Foundry** به صورت کد با Bicep + `azd` ظرف چند دقیقه فراهم کنید — راهنمای [Azure AI Foundry Setup Guide](getting-started-azure-openai.md) را ببینید. احراز هویت **بدون کلید** (Microsoft Entra ID) است، بنابراین نیازی به مدیریت کلیدهای API ندارید.

## آنچه خواهید آموخت

- راه‌اندازی محیط توسعه جاوا برای برنامه‌های هوش مصنوعی
- انتخاب و پیکربندی محیط توسعه دلخواه خود (اولویت ابری با Codespaces، کانتینر توسعه محلی، یا راه‌اندازی کامل محلی)
- آزمایش تنظیمات خود با اتصال به مدل Azure AI Foundry

## فهرست مطالب

- [آنچه خواهید آموخت](#آنچه-خواهید-آموخت)
- [مقدمه](#مقدمه)
- [مرحله ۱: راه‌اندازی محیط توسعه](#مرحله-۱-راه‌اندازی-محیط-توسعه)
  - [گزینه الف: GitHub Codespaces (توصیه شده)](#گزینه-الف-github-codespaces-توصیه-شده)
  - [گزینه ب: کانتینر توسعه محلی](#گزینه-ب-کانتینر-توسعه-محلی)
  - [گزینه ج: استفاده از نصب محلی موجود](#گزینه-ج-استفاده-از-نصب-محلی-موجود)
- [مرحله ۲: فراهم‌سازی Azure AI Foundry](#مرحله-۲-فراهم‌سازی-azure-ai-foundry)
- [مرحله ۳: آزمایش تنظیمات](#مرحله-۳-آزمایش-تنظیمات)
- [عیب‌یابی](#عیب‌یابی)
- [خلاصه](#خلاصه)
- [گام‌های بعدی](#گام‌های-بعدی)

## مقدمه

این فصل شما را در راه‌اندازی محیط توسعه راهنمایی می‌کند. در طول این دوره، از **Azure AI Foundry** برای مدل‌ها استفاده خواهیم کرد. شما مدل‌ها را به صورت کد با Bicep و Azure Developer CLI (`azd`) فراهم می‌کنید، سپس با احراز هویت **بدون کلید** (Microsoft Entra ID) متصل می‌شوید — بدون نیاز به کپی یا لو رفتن کلیدهای API.

**راه‌اندازی محلی لازم نیست!** می‌توانید از GitHub Codespaces استفاده کنید که محیط توسعه کامل را در مرورگر شما فراهم می‌کند و از آنجا Foundry را فراهم می‌کند.

ما در این دوره از **Azure AI Foundry** استفاده می‌کنیم زیرا:
- **به صورت کد فراهم می‌شود** — تنها با یک `azd up` حساب و استقرار مدل‌ها را اعمال می‌کند
- **بدون کلید** — با ورود به Azure یا هویت مدیریت شده احراز هویت می‌کند
- **آماده تولید** — همان کد هم به صورت محلی و هم در Azure اجرا می‌شود
- **انعطاف‌پذیر** — مدل‌ها را با تغییر نام استقرار، نه کد خود، عوض کنید

> **نکته:** هزینه استقرارهای Azure AI Foundry به ازای هر توکن است (پرداخت به ازای مصرف). برای جزئیات فراهم‌سازی، منطقه و هزینه‌ها به [Azure AI Foundry setup guide](getting-started-azure-openai.md) مراجعه کنید.


## مرحله ۱: راه‌اندازی محیط توسعه

<a name="quick-start-cloud"></a>

ما یک کانتینر توسعه پیش‌پیکربندی‌شده ایجاد کردیم تا زمان راه‌اندازی را کاهش داده و اطمینان حاصل کنیم که همه ابزارهای لازم برای این دوره هوش مصنوعی مولد جاوا را دارید. رویکرد توسعه دلخواه خود را انتخاب کنید:

### گزینه‌های راه‌اندازی محیط:

#### گزینه الف: GitHub Codespaces (توصیه شده)

**در ۲ دقیقه شروع به کدنویسی کنید - نیازی به راه‌اندازی محلی نیست!**

1. این مخزن را به حساب GitHub خود فورک کنید  
   > **نکته:** اگر می‌خواهید تنظیمات پایه را ویرایش کنید، به [Dev Container Configuration](../../../.devcontainer/devcontainer.json) مراجعه کنید
2. روی **Code** کلیک کنید → تب **Codespaces** → **...** → **New with options...**
3. پیش‌فرض‌ها را نگه دارید – این تنظیمات **Dev container configuration**: **محیط توسعه جاوا هوش مصنوعی مولد** که برای این دوره ساخته شده را انتخاب می‌کند
4. روی **Create codespace** کلیک کنید
5. حدود ۲ دقیقه منتظر بمانید تا محیط آماده شود
6. به [مرحله ۲: فراهم‌سازی Azure AI Foundry](#مرحله-۲-فراهم‌سازی-azure-ai-foundry) بروید

<img src="../../../translated_images/fa/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: Codespaces submenu" width="50%">

<img src="../../../translated_images/fa/image.833552b62eee7766.webp" alt="Screenshot: New with options" width="50%">

<img src="../../../translated_images/fa/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: Create codespace options" width="50%">


> **مزایای Codespaces**:
> - نیازی به نصب محلی نیست
> - روی هر دستگاهی با مرورگر کار می‌کند
> - با همه ابزارها و وابستگی‌ها پیش‌پیکربندی شده است
> - ۶۰ ساعت رایگان در ماه برای حساب‌های شخصی
> - محیط یکنواخت برای همه فراگیران

#### گزینه ب: کانتینر توسعه محلی

**برای توسعه‌دهندگانی که ترجیح می‌دهند توسعه محلی با داکر انجام دهند**

1. این مخزن را فورک و کلون کنید به دستگاه محلی خود  
   > **نکته:** اگر می‌خواهید تنظیمات پایه را ویرایش کنید، به [Dev Container Configuration](../../../.devcontainer/devcontainer.json) مراجعه کنید
2. [Docker Desktop](https://www.docker.com/products/docker-desktop/) و [VS Code](https://code.visualstudio.com/) را نصب کنید
3. افزونه [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) را در VS Code نصب کنید
4. پوشه مخزن را در VS Code باز کنید
5. وقتی درخواست شد، روی **Reopen in Container** کلیک کنید (یا از `Ctrl+Shift+P` → "Dev Containers: Reopen in Container" استفاده کنید)
6. منتظر بمانید تا کانتینر ساخته و راه‌اندازی شود
7. به [مرحله ۲: فراهم‌سازی Azure AI Foundry](#مرحله-۲-فراهم‌سازی-azure-ai-foundry) بروید

<img src="../../../translated_images/fa/devcontainer.21126c9d6de64494.webp" alt="Screenshot: Dev container setup" width="50%">

<img src="../../../translated_images/fa/image-3.bf93d533bbc84268.webp" alt="Screenshot: Dev container build complete" width="50%">

#### گزینه ج: استفاده از نصب محلی موجود

**برای توسعه‌دهندگانی که محیط جاوا موجود دارند**

پیش‌نیازها:  
- [جاوا ۲۱ به بالا](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html)  
- [Maven 3.9 به بالا](https://maven.apache.org/download.cgi)  
- [VS Code](https://code.visualstudio.com) یا IDE دلخواه شما

مراحل:  
1. این مخزن را روی دستگاه محلی خود کلون کنید  
2. پروژه را در IDE خود باز کنید  
3. به [مرحله ۲: فراهم‌سازی Azure AI Foundry](#مرحله-۲-فراهم‌سازی-azure-ai-foundry) بروید

> **نکته حرفه‌ای**: اگر دستگاه کم‌قدرتی دارید اما می‌خواهید VS Code محلی داشته باشید، از GitHub Codespaces استفاده کنید! می‌توانید VS Code محلی را به Codespace میزبان ابری متصل کنید تا بهترین‌های هر دو را داشته باشید.

<img src="../../../translated_images/fa/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: created local devcontainer instance" width="50%">


## مرحله ۲: فراهم‌سازی Azure AI Foundry

مدل‌های هوش مصنوعی دوره را به Azure AI Foundry به صورت کد استقرار دهید. از ریشه مخزن:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```
  
`azd` نام محیط و منطقه را درخواست می‌کند، حساب Azure AI Foundry را با استقرارهای `gpt-4o-mini` و `text-embedding-3-small` فراهم می‌کند، و نقطه پایان را در فایل `.env` نمونه می‌نویسد — همه با احراز هویت **بدون کلید** (بدون کلید API).

> **راهنمای کامل:** برای پیش‌نیازها، جایگزین دستی (پرتال)، راهنمای منطقه و نکات هزینه/پاکسازی به [Azure AI Foundry Setup Guide](getting-started-azure-openai.md) مراجعه کنید.

## مرحله ۳: آزمایش تنظیمات

پس از فراهم‌سازی مدل‌های Foundry، اتصال را با اپلیکیشن نمونه در [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) تست کنید.

1. ترمینال را در محیط توسعه خود باز کنید.  
2. به مثال بروید:  
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
3. مطمئن شوید وارد سیستم شده‌اید (احراز هویت بدون کلید به توکن نیاز دارد):  
   ```bash
   az login
   ```
   > اگر `azd up` را اجرا کرده‌اید، فایل `.env` با نقطه پایان شما قبلاً نوشته شده است.  
4. برنامه را اجرا کنید:  
   ```bash
   mvn clean spring-boot:run
   ```
  
شما باید پاسخ از مدل `gpt-4o-mini` را ببینید.

### درک کد نمونه

نمونه زیر `examples/basic-chat-azure` یک برنامه Spring Boot است که با **Spring AI** برای اتصال به Azure AI Foundry با احراز هویت بدون کلید استفاده می‌شود.

**کدی که اینجا است انجام می‌دهد:**  
- **اتصال** به Azure AI Foundry با ورود Azure شما (Microsoft Entra ID) — بدون کلید API  
- **ارسال** درخواست به مدل `gpt-4o-mini`  
- **دریافت** و نمایش پاسخ هوش مصنوعی  
- **اعتبارسنجی** اینکه تنظیمات به درستی کار می‌کند

**وابستگی کلیدی** (در `pom.xml`):  
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```
  
**پیکربندی** (`application.yml`):  
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
  
## خلاصه

عالی! حالا همه چیز را راه‌اندازی کرده‌اید:

- مدل‌های Azure AI Foundry را به صورت کد با Bicep + `azd` فراهم کرده‌اید  
- محیط توسعه جاوا خود را (چه Codespaces، کانتینر توسعه یا محلی) راه‌اندازی کرده‌اید  
- به Azure AI Foundry با احراز هویت بدون کلید (Microsoft Entra ID) متصل شده‌اید — بدون کلید API  
- همه چیز را با نمونه ساده‌ای که با مدل شما صحبت می‌کند آزمایش کرده‌اید

## گام‌های بعدی

[فصل ۳: تکنیک‌های اصلی هوش مصنوعی مولد](../03-CoreGenerativeAITechniques/README.md)

## عیب‌یابی

مشکل دارید؟ اینجا مشکلات رایج و راه‌حل‌ها:

- **احراز هویت موفق نیست (401/403)?**  
  - اجرای `az login` — احراز هویت بدون کلید است، پس باید وارد شده باشید  
  - بررسی کنید حساب شما نقش **Cognitive Services OpenAI User** را روی منبع دارد  
  - اگر تازه فراهم کرده‌اید، چند دقیقه صبر کنید تا انتساب نقش اعمال شود

- **Maven پیدا نمی‌شود؟**  
  - اگر از کانتینرهای dev یا Codespaces استفاده می‌کنید، Maven باید پیش‌نصب شده باشد  
  - در تنظیم محلی، اطمینان حاصل کنید جاوا ۲۱+ و Maven 3.9+ نصب شده‌اند  
  - با اجرای `mvn --version` نصب را بررسی کنید

- **`azd` پیدا نمی‌شود یا فراهم‌سازی موفق نیست؟**  
  - [Azure Developer CLI](https://aka.ms/azure-dev/install) را نصب کنید و `azd auth login` را اجرا کنید  
  - منطقه‌ای را انتخاب کنید که `gpt-4o-mini` در آن موجود است (مثلاً `eastus2`)  
  - برای جزئیات به [Azure AI Foundry setup guide](getting-started-azure-openai.md) مراجعه کنید

- **کانتینر توسعه شروع نمی‌شود؟**  
  - مطمئن شوید Docker Desktop در حال اجرا است (برای توسعه محلی)  
  - سعی کنید کانتینر را بازسازی کنید: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **خطاهای کامپایل اپلیکیشن؟**  
  - مطمئن شوید در دایرکتوری صحیح هستید: `02-SetupDevEnvironment/examples/basic-chat-azure`  
  - سعی کنید پاکسازی و بازسازی کنید: `mvn clean compile`

> **نیاز به کمک دارید؟**: هنوز مشکل دارید؟ در مخزن یک Issue باز کنید و ما به شما کمک خواهیم کرد.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**سلب مسئولیت**:
این سند با استفاده از سرویس ترجمه هوش مصنوعی [Co-op Translator](https://github.com/Azure/co-op-translator) ترجمه شده است. در حالی که ما در تلاش برای دقت هستیم، لطفاً توجه داشته باشید که ترجمه‌های خودکار ممکن است شامل خطاها یا نادرستی‌هایی باشند. سند اصلی به زبان مادری خود باید به عنوان منبع معتبر در نظر گرفته شود. برای اطلاعات حیاتی، ترجمه حرفه‌ای انسانی توصیه می‌شود. ما در قبال هرگونه سوء تفاهم یا برداشت نادرست ناشی از استفاده از این ترجمه مسئولیتی نداریم.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->