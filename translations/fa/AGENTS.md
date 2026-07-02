# AGENTS.md

## مرور پروژه

این یک مخزن آموزشی برای یادگیری توسعه هوش مصنوعی مولد با جاوا است. این دوره جامع عملی شامل مدل‌های زبانی بزرگ (LLMs)، مهندسی پرامپت، جاسازی‌ها، RAG (تولید با بازیابی تقویت‌شده) و پروتکل متن مدل (MCP) است.

**تکنولوژی‌های کلیدی:**
- جاوا ۲۱
- Spring Boot نسخه ۳.۵.x
- Spring AI نسخه ۱.۱.x
- Maven
- LangChain4j
- Azure AI Foundry، Azure OpenAI و SDKهای OpenAI

**معماری:**
- چندین برنامه مستقل Spring Boot سازماندهی شده بر اساس فصل‌ها
- پروژه‌های نمونه که الگوهای مختلف AI را نشان می‌دهند
- آماده GitHub Codespaces با کانتینرهای توسعه از پیش پیکربندی شده

## دستورات راه‌اندازی

### پیش‌نیازها
- جاوا ۲۱ یا بالاتر
- Maven نسخه ۳.x
- اشتراک Azure با استقرار مدل Azure AI Foundry (با `azd up` تهیه شده)
- Azure CLI (`az`) و Azure Developer CLI (`azd`)، وارد شده برای احراز هویت بدون کلید

### تنظیم محیط

**گزینه ۱: GitHub Codespaces (توصیه شده)**
```bash
# فورک مخزن و ایجاد یک محیط کد از طریق رابط گیت‌هاب
# کانتینر توسعه به صورت خودکار تمام وابستگی‌ها را نصب می‌کند
# حدود ۲ دقیقه برای راه‌اندازی محیط صبر کنید
```

**گزینه ۲: کانتینر توسعه محلی**
```bash
# کلون مخزن
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# باز کردن در VS Code با افزونه Dev Containers
# باز کردن مجدد در کانتینر هنگام درخواست
```

**گزینه ۳: راه‌اندازی محلی**
```bash
# نصب وابستگی‌ها
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# بررسی صحت نصب
java -version
mvn -version
```

### پیکربندی

**تنظیم Azure AI Foundry (بدون کلید، توصیه شده):**
```bash
# تامین حساب Foundry و استقرار مدل‌ها به صورت کد
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd فایل examples/basic-chat-azure/.env را با نقطه انتهایی شما (بدون کلید) می‌نویسد
```

**پیکربندی دستی نقطه انتهایی:**
```bash
# اگر از azd استفاده نکردید، نقطه پایان را خودتان تنظیم کنید (توکن ورود از طریق az login بدون کلید باقی می‌ماند)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# فایل .env را ویرایش کنید: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## جریان توسعه

### ساختار پروژه
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

### اجرای برنامه‌ها

**اجرای یک برنامه Spring Boot:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**ساخت پروژه:**
```bash
cd [project-directory]
mvn clean install
```

**راه‌اندازی سرور محاسبه‌گر MCP:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# سرور روی http://localhost:8080 اجرا می‌شود
```

**اجرای نمونه‌های مشتری:**
```bash
# پس از راه‌اندازی سرور در ترمینال دیگر
cd 04-PracticalSamples/calculator

# کلاینت مستقیم MCP
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# کلاینت مجهز به هوش مصنوعی (نیازمند AZURE_OPENAI_ENDPOINT و ورود az)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# ربات تعاملی
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### بارگذاری مجدد داغ
ابزار Spring Boot DevTools در پروژه‌هایی که از بارگذاری مجدد داغ پشتیبانی می‌کنند، گنجانده شده است:
```bash
# تغییرات در فایل‌های جاوا هنگام ذخیره به‌طور خودکار بارگذاری مجدد می‌شوند
mvn spring-boot:run
```

## دستورالعمل‌های آزمون

### اجرای تست‌ها

**اجرای همه تست‌ها در یک پروژه:**
```bash
cd [project-directory]
mvn test
```

**اجرای تست‌ها با خروجی مفصل:**
```bash
mvn test -X
```

**اجرای کلاس تست خاص:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### ساختار تست
- فایل‌های تست از JUnit 5 (Jupiter) استفاده می‌کنند
- کلاس‌های تست در مسیر `src/test/java/` قرار دارند
- نمونه‌های مشتری در پروژه محاسبه‌گر در مسیر `src/test/java/com/microsoft/mcp/sample/client/` هستند

### تست دستی
بسیاری از نمونه‌ها برنامه‌های تعاملی هستند که نیاز به تست دستی دارند:

1. برنامه را با دستور `mvn spring-boot:run` اجرا کنید
2. نقاط انتهایی را آزمایش کرده یا با CLI تعامل کنید
3. مطمئن شوید رفتار مورد انتظار با مستندات README.md هر پروژه مطابقت دارد

### تست با Azure AI Foundry
- قبل از اجرای نمونه‌ها با `az login` وارد شوید (احراز هویت بدون کلید)
- مطمئن شوید حساب شما نقش Cognitive Services OpenAI User را روی منبع دارد
- فیلتر کردن محتوا را با مثال مسئولانه AI در فصل ۵ آزمایش کنید

## راهنمای سبک کد

### قراردادهای جاوا
- **نسخه جاوا:** جاوا ۲۱ با ویژگی‌های مدرن
- **سبک:** پیروی از قراردادهای استاندارد جاوا
- **نامگذاری:** 
  - کلاس‌ها: PascalCase
  - متدها/متغیرها: camelCase
  - ثابت‌ها: UPPER_SNAKE_CASE
  - نام بسته‌ها: حروف کوچک

### الگوهای Spring Boot
- استفاده از `@Service` برای منطق کسب‌وکار
- استفاده از `@RestController` برای نقاط انتهایی REST
- پیکربندی از طریق `application.yml` یا `application.properties`
- ترجیح متغیرهای محیطی بر مقادیر سخت‌کد شده
- استفاده از annotation `@Tool` برای متدهای نمایان‌شده توسط MCP

### سازماندهی فایل‌ها
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

### وابستگی‌ها
- مدیریت‌شده از طریق Maven `pom.xml`
- Spring AI BOM برای مدیریت نسخه‌ها
- LangChain4j برای ادغام‌های AI
- والد استارتر Spring Boot برای وابستگی‌های Spring

### توضیحات کد
- اضافه کردن JavaDoc برای APIهای عمومی
- شامل کامنت‌های توضیحی برای تعاملات پیچیده AI
- مستندسازی واضح توصیف ابزارهای MCP

## ساخت و استقرار

### ساخت پروژه‌ها

**ساخت بدون تست:**
```bash
mvn clean install -DskipTests
```

**ساخت با همه بررسی‌ها:**
```bash
mvn clean install
```

**بسته‌بندی برنامه:**
```bash
mvn package
# ایجاد JAR در دایرکتوری target/
```

### دایرکتوری‌های خروجی
- کلاس‌های کامپایل شده: `target/classes/`
- کلاس‌های تست: `target/test-classes/`
- فایل‌های JAR: `target/*.jar`
- آرشیوهای Maven: `target/`

### پیکربندی مختص محیط

**توسعه:**
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

**تولید:**
- استفاده از یک هویت مدیریت شده به جای `az login` برای احراز هویت بدون کلید
- اشاره `AZURE_OPENAI_ENDPOINT` به منبع تولید Foundry شما
- مدیریت پیکربندی از طریق متغیرهای محیطی یا Azure Key Vault

### ملاحظات استقرار
- این یک مخزن آموزشی با برنامه‌های نمونه است
- برای استقرار به صورت مستقیم در تولید طراحی نشده است
- نمونه‌ها الگوهایی را برای تطبیق در استفاده تولیدی نشان می‌دهند
- نکات استقرار خاص را در READMEهای پروژه‌های جداگانه ببینید

## یادداشت‌های اضافی

### Azure AI Foundry
- **احراز هویت بدون کلید:** اتصال با Microsoft Entra ID — بدون کلید API برای مدیریت
- **تأمین به صورت کد:** Bicep + azd (`azd up`) حساب و استقرارهای مدل را ایجاد می‌کنند
- همان کد سازگار با OpenAI هم به صورت محلی (`az login`) و هم در Azure (هویت مدیریت شده) اجرا می‌شود

### کار با چند پروژه
هر پروژه نمونه به صورت مستقل است:
```bash
# به پروژه خاصی بروید
cd 04-PracticalSamples/[project-name]

# هر کدام فایل pom.xml مخصوص به خود را دارند و می‌توانند به طور مستقل ساخته شوند
mvn clean install
```

### مشکلات رایج

**اختلاف نسخه جاوا:**
```bash
# تأیید جاوا ۲۱
java -version
# در صورت نیاز، JAVA_HOME را به‌روزرسانی کنید
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**مشکلات دانلود وابستگی:**
```bash
# پاک کردن کش Maven و تلاش مجدد
rm -rf ~/.m2/repository
mvn clean install
```

**نقطه انتهایی یا ورود یافت نشد:**
```bash
# نقطه انتهایی را در جلسه جاری تنظیم کنید و وارد شوید (بدون کلید)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# یا از یک فایل .env در دایرکتوری پروژه استفاده کنید
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**پورت قبلاً در استفاده است:**
```bash
# اسپرینگ بوت به طور پیش‌فرض از پورت ۸۰۸۰ استفاده می‌کند
# تغییر در application.properties:
server.port=8081
```

### پشتیبانی چندزبانه
- مستندات به بیش از ۴۵ زبان با ترجمه خودکار در دسترس است
- ترجمه‌ها در پوشه `translations/`
- ترجمه توسط گردش کاری GitHub Actions مدیریت می‌شود

### مسیر یادگیری
1. با [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md) شروع کنید
2. فصل‌ها را به ترتیب دنبال کنید (۰۱ → ۰۵)
3. نمونه‌های عملی در هر فصل را به پایان برسانید
4. پروژه‌های نمونه در فصل ۴ را کاوش کنید
5. تمرین هوش مصنوعی مسئولانه را در فصل ۵ بیاموزید

### کانتینر توسعه
فایل `.devcontainer/devcontainer.json` شامل:
- محیط توسعه جاوا ۲۱
- Maven از پیش نصب شده
- افزونه‌های Java برای VS Code
- ابزارهای Spring Boot
- یکپارچه‌سازی GitHub Copilot
- پشتیبانی از Docker-in-Docker
- Azure CLI

### ملاحظات عملکرد
- استقرارهای Azure AI Foundry محدودیت‌های تعداد توکن/درخواست دقیقه‌ای دارند
- از اندازه دسته مناسب برای جاسازی‌ها استفاده کنید
- برای فراخوانی‌های مکرر API کش را مد نظر قرار دهید
- مصرف توکن را برای بهینه‌سازی هزینه زیر نظر بگیرید

### نکات امنیتی
- هرگز فایل‌های `.env` را کامیت نکنید (در `.gitignore` قرار دارند)
- احراز هویت بدون کلید (Microsoft Entra ID) را به جای کلیدهای API ترجیح دهید
- در Azure از هویت‌های مدیریت شده استفاده کنید؛ برای توسعه محلی از `az login`
- دستورالعمل‌های هوش مصنوعی مسئولانه را در فصل ۵ دنبال کنید

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**سلب مسئولیت**:
این سند با استفاده از سرویس ترجمه هوش مصنوعی [Co-op Translator](https://github.com/Azure/co-op-translator) ترجمه شده است. در حالی که ما در تلاش برای دقت هستیم، لطفاً توجه داشته باشید که ترجمه‌های خودکار ممکن است شامل خطاها یا نادرستی‌هایی باشند. سند اصلی به زبان مادری خود باید به عنوان منبع معتبر در نظر گرفته شود. برای اطلاعات حیاتی، ترجمه حرفه‌ای انسانی توصیه می‌شود. ما در قبال هرگونه سوء تفاهم یا برداشت نادرست ناشی از استفاده از این ترجمه مسئولیتی نداریم.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->