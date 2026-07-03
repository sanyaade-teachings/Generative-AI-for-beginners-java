# AGENTS.md

## نظرة عامة على المشروع

هذا مستودع تعليمي لتعلم تطوير الذكاء الاصطناعي التوليدي باستخدام جافا. يوفر دورة شاملة عملية تغطي نماذج اللغة الكبيرة (LLMs)، هندسة التوجيه، التضمينات، RAG (التوليد المعزز بالإسترجاع)، وبروتوكول سياق النموذج (MCP).

**التقنيات الأساسية:**
- جافا 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- مافن
- LangChain4j
- Azure AI Foundry، Azure OpenAI، ومكتبات OpenAI SDK

**الهيكلية:**
- تطبيقات Spring Boot مستقلة متعددة منظمة حسب الفصول
- مشاريع عينة توضح أنماطًا مختلفة للذكاء الاصطناعي
- جاهز لـ GitHub Codespaces مع حاويات تطوير مهيأة مسبقًا

## أوامر الإعداد

### المتطلبات المسبقة
- جافا 21 أو أحدث
- مافن 3.x
- اشتراك Azure مع نشر نموذج Azure AI Foundry (يتم توفيره باستخدام `azd up`)
- Azure CLI (`az`) و Azure Developer CLI (`azd`)، مسجل دخول لدعم المصادقة بدون مفتاح

### إعداد البيئة

**الخيار 1: GitHub Codespaces (موصى به)**
```bash
# قم بتفريع المستودع وإنشاء مساحة تعليمات برمجية من واجهة GitHub
# سيقوم حاوية التطوير بتثبيت جميع التبعيات تلقائيًا
# انتظر حوالي دقيقتين لإعداد البيئة
```

**الخيار 2: حاوية تطوير محلية**
```bash
# استنساخ المستودع
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# افتح في VS Code مع امتداد حاويات التطوير
# أعد الفتح في الحاوية عند الطلب
```

**الخيار 3: الإعداد المحلي**
```bash
# تثبيت التبعيات
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# التحقق من التثبيت
java -version
mvn -version
```

### التكوين

**إعداد Azure AI Foundry (مصادقة بدون مفتاح، موصى بها):**
```bash
# توفير حساب Foundry + نشر النماذج ككود
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# يقوم azd بكتابة examples/basic-chat-azure/.env مع نقطة النهاية الخاصة بك (بدون مفتاح)
```

**تكوين نقطة النهاية يدويًا:**
```bash
# إذا لم تستخدم azd، قم بتعيين نقطة النهاية بنفسك (ستبقى المصادقة بدون مفتاح عبر az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# حرر .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## سير عمل التطوير

### هيكلية المشروع
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

### تشغيل التطبيقات

**تشغيل تطبيق Spring Boot:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**بناء المشروع:**
```bash
cd [project-directory]
mvn clean install
```

**بدء خادم MCP Calculator:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# الخادم يعمل على http://localhost:8080
```

**تشغيل أمثلة العميل:**
```bash
# بعد بدء الخادم في نافذة طرفية أخرى
cd 04-PracticalSamples/calculator

# عميل MCP مباشر
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# عميل معزز بالذكاء الاصطناعي (يتطلب AZURE_OPENAI_ENDPOINT + تسجيل دخول az)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# روبوت تفاعلي
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### إعادة التحميل السريع
تتضمن المشاريع التي تدعم إعادة التحميل السريع Spring Boot DevTools:
```bash
# سيتم إعادة التحميل تلقائيًا عند حفظ التغييرات في ملفات جافا
mvn spring-boot:run
```

## تعليمات الاختبار

### تشغيل الاختبارات

**تشغيل جميع الاختبارات في مشروع:**
```bash
cd [project-directory]
mvn test
```

**تشغيل الاختبارات مع مخرجات مفصلة:**
```bash
mvn test -X
```

**تشغيل فصل اختبار محدد:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### هيكلية الاختبار
- تستخدم ملفات الاختبار JUnit 5 (Jupiter)
- تقع فصول الاختبار في `src/test/java/`
- أمثلة العميل في مشروع الحاسبة توجد في `src/test/java/com/microsoft/mcp/sample/client/`

### الاختبار اليدوي
العديد من الأمثلة هي تطبيقات تفاعلية تتطلب اختبارًا يدويًا:

1. ابدأ التطبيق مع `mvn spring-boot:run`
2. اختبر نقاط النهاية أو تفاعل مع CLI
3. تحقق من توافق السلوك المتوقع مع الوثائق في كل ملف README.md الخاص بالمشروع

### الاختبار مع Azure AI Foundry
- سجل الدخول باستخدام `az login` قبل تشغيل الأمثلة (مصادقة بدون مفتاح)
- تأكد من أن حسابك يحمل دور مستخدم Cognitive Services OpenAI على المورد
- اختبر التصفية المحتوى مع مثال الذكاء الاصطناعي المسؤول في الفصل 5

## إرشادات نمط الكود

### قواعد جافا
- **إصدار جافا:** جافا 21 مع ميزات حديثة
- **النمط:** اتبع قواعد جافا القياسية
- **التسمية:** 
  - الفئات: PascalCase
  - الطرق/المتغيرات: camelCase
  - الثوابت: UPPER_SNAKE_CASE
  - أسماء الحزم: أحرف صغيرة

### أنماط Spring Boot
- استخدم `@Service` للمنطق التجاري
- استخدم `@RestController` لنقاط نهاية REST
- التكوين عبر `application.yml` أو `application.properties`
- تفضيل المتغيرات البيئية على القيم الثابتة
- استخدم التعليمة `@Tool` للطرق المكشوفة عبر MCP

### تنظيم الملفات
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

### الاعتمادات
- مدار عبر مافن `pom.xml`
- Spring AI BOM لإدارة الإصدارات
- LangChain4j للتكاملات AI
- Spring Boot starter parent لإدارة تبعيات Spring

### تعليقات الكود
- أضف JavaDoc لواجهات API العامة
- تضمين تعليقات توضيحية للتفاعلات المعقدة مع الذكاء الاصطناعي
- وثق أوصاف أدوات MCP بوضوح

## البناء والنشر

### بناء المشاريع

**البناء بدون اختبار:**
```bash
mvn clean install -DskipTests
```

**البناء مع جميع الفحوصات:**
```bash
mvn clean install
```

**حزم التطبيق:**
```bash
mvn package
# يُنشئ JAR في دليل target/
```

### مجلدات المخرجات
- الفئات المترجمة: `target/classes/`
- فئات الاختبار: `target/test-classes/`
- ملفات JAR: `target/*.jar`
- ملفات مافن: `target/`

### التكوين الخاص بالبيئة

**التطوير:**
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

**الإنتاج:**
- استخدم هوية مدارة بدلاً من `az login` للمصادقة بدون مفتاح
- وجه `AZURE_OPENAI_ENDPOINT` إلى مورد Foundry الإنتاجي الخاص بك
- إدارة التكوين عبر متغيرات البيئة أو Azure Key Vault

### اعتبارات النشر
- هذا مستودع تعليمي مع تطبيقات نموذجية
- غير مصمم للنشر في بيئة الإنتاج كما هو
- تعرض الأمثلة أنماطًا للتكيف مع الاستخدام في الإنتاج
- راجع ملفات README الخاصة بكل مشروع لملاحظات النشر الخاصة

## ملاحظات إضافية

### Azure AI Foundry
- **المصادقة بدون مفتاح:** الاتصال عبر Microsoft Entra ID — لا حاجة لإدارة مفاتيح API
- **تم توفيره ككود:** Bicep + azd (`azd up`) ينشئ الحساب ونماذج النشر
- نفس الكود المتوافق مع OpenAI يعمل محليًا (`az login`) وفي Azure (الهوية المدارة)

### العمل مع مشاريع متعددة
كل مشروع نموذجي مستقل:
```bash
# الانتقال إلى مشروع معين
cd 04-PracticalSamples/[project-name]

# لدى كل منها ملف pom.xml الخاص بها ويمكن بناؤها بشكل مستقل
mvn clean install
```

### المشكلات الشائعة

**عدم تطابق إصدار جافا:**
```bash
# تحقق من جافا 21
java -version
# حدّث JAVA_HOME إذا تطلب الأمر
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**مشاكل تنزيل التبعيات:**
```bash
# مسح ذاكرة التخزين المؤقت لـ Maven وأعد المحاولة
rm -rf ~/.m2/repository
mvn clean install
```

**عدم العثور على نقطة النهاية أو تسجيل الدخول:**
```bash
# تعيين نقطة النهاية في الجلسة الحالية وتسجيل الدخول (بدون مفتاح)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# أو استخدام ملف .env في دليل المشروع
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**المنفذ مستخدم بالفعل:**
```bash
# يستخدم Spring Boot المنفذ 8080 بشكل افتراضي
# التغيير في application.properties:
server.port=8081
```

### دعم متعدد اللغات
- التوثيق متوفر بـ 45+ لغة عبر الترجمة التلقائية
- الترجمات في مجلد `translations/`
- إدارة الترجمة عبر سير عمل GitHub Actions

### مسار التعلم
1. ابدأ مع [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. اتبع الفصول بالترتيب (01 → 05)
3. أكمل الأمثلة العملية في كل فصل
4. استكشف المشاريع النموذجية في الفصل 4
5. تعلم ممارسات الذكاء الاصطناعي المسؤول في الفصل 5

### حاوية التطوير
ملف `.devcontainer/devcontainer.json` يجهز:
- بيئة تطوير جافا 21
- مافن مثبت مسبقًا
- إضافات جافا لـ VS Code
- أدوات Spring Boot
- تكامل GitHub Copilot
- دعم Docker-in-Docker
- Azure CLI

### اعتبارات الأداء
- نشرات Azure AI Foundry لها حصص رموز/طلبات دقيقة
- استخدم أحجام دفعات مناسبة للتضمينات
- اعتبر التخزين المؤقت للنداءات المتكررة لواجهة API
- راقب استخدام الرموز لتحسين التكلفة

### ملاحظات الأمان
- لا تقم أبدًا بالتزام ملفات `.env` (مٌدرجة مسبقًا في `.gitignore`)
- فضّل المصادقة بدون مفتاح (Microsoft Entra ID) على مفاتيح API
- استخدم الهويات المدارة في Azure؛ استخدم `az login` للتطوير المحلي
- اتبع إرشادات الذكاء الاصطناعي المسؤول في الفصل 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**تنويه**:
تمت ترجمة هذا المستند باستخدام خدمة الترجمة بالذكاء الاصطناعي [Co-op Translator](https://github.com/Azure/co-op-translator). بينما نسعى للدقة، يرجى العلم أن الترجمات الآلية قد تحتوي على أخطاء أو عدم دقة. يجب اعتبار المستند الأصلي بلغته الأصلية المصدر الرسمي والمعتمد. للمعلومات الهامة، يُنصح بالاستعانة بترجمة بشرية محترفة. نحن غير مسؤولين عن أي سوء فهم أو تفسير ناتج عن استخدام هذه الترجمة.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->