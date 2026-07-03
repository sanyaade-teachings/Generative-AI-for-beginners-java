# AGENTS.md

## پروجیکٹ کا جائزہ

یہ جاوا کے ساتھ جنریٹو اے آئی کی ترقی سیکھنے کے لیے ایک تعلیمی ذخیرہ ہے۔ یہ بڑی زبان کے ماڈلز (LLMs)، پرامپٹ انجینئرنگ، ایمبیڈنگز، RAG (ریٹریول اگیمنٹڈ جنریشن)، اور ماڈل کانٹیکسٹ پروٹوکول (MCP) کا جامع عملی کورس فراہم کرتا ہے۔

**اہم ٹیکنالوجیز:**
- جاوا 21
- سپرنگ بوٹ 3.5.x
- سپرنگ AI 1.1.x
- میون
- لانگ چین4جے
- ایزور AI فاؤنڈری، ایزور اوپن AI، اور اوپن AI SDKs

**آرکیٹیکچر:**
- مضامین کے حساب سے منظم کئی اسٹینڈ اکیٹھ سپرنگ بوٹ ایپلیکیشنز
- مختلف AI پیٹرنز کو ظاہر کرنے والے نمونہ پروجیکٹس
- گٹ ہب کوڈ اسپیسز کے لیے تیار، پہلے سے کنفیگر کردہ ڈویلپمنٹ کنٹینرز کے ساتھ

## سیٹ اپ کمانڈز

### پری ریکوائرمنٹس
- جاوا 21 یا اس سے زیادہ
- میون 3.x
- ایزور سبسکرپشن جس میں ایزور AI فاؤنڈری ماڈل ڈپلائمنٹ ہو (از طریق `azd up` فراہم کریں)
- ایزور CLI (`az`) اور ایزور ڈیولپر CLI (`azd`)، کی لیس تصدیق کے لیے سائن ان

### ماحول کی ترتیب

**اختیار 1: گٹ ہب کوڈ اسپیسز (تجویز کردہ)**
```bash
# ریپازیٹری کو فورک کریں اور GitHub UI سے کوڈ اسپیس بنائیں
# ڈیولپمنٹ کنٹینر خود بخود تمام dependencies انسٹال کر دے گا
# ماحول کے سیٹ اپ کے لیے تقریباً ۲ منٹ انتظار کریں
```

**اختیار 2: لوکل ڈویلپمنٹ کنٹینر**
```bash
# مخزن کلون کریں
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Dev Containers توسیع کے ساتھ VS Code میں کھولیں
# جب کہا جائے تو کنٹینر میں دوبارہ کھولیں
```

**اختیار 3: لوکل سیٹ اپ**
```bash
# انحصارات انسٹال کریں
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# تنصیب کی تصدیق کریں
java -version
mvn -version
```

### کنفیگریشن

**ایزور AI فاؤنڈری سیٹ اپ (کی لیس، تجویز کردہ):**
```bash
# کوڈ کے طور پر Foundry اکاؤنٹ + ماڈل ڈپلائمنٹس مہیا کریں
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd آپ کے اینڈپوائنٹ (کوئی کلید نہیں) کے ساتھ examples/basic-chat-azure/.env لکھتا ہے
```

**مینول اینڈ پوائنٹ کنفیگریشن:**
```bash
# اگر آپ نے azd استعمال نہیں کیا تو خود اند پوائنٹ سیٹ کریں (auth az login کے ذریعے keyless رہتا ہے)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# .env ترمیم کریں: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## ترقیاتی ورک فلو

### پروجیکٹ کی ساخت
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

### ایپلیکیشنز چلانا

**سپرنگ بوٹ ایپلیکیشن چلانے کے لیے:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**پروجیکٹ بنانا:**
```bash
cd [project-directory]
mvn clean install
```

**MCP کیلکولیٹر سرور شروع کرنا:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# سرور http://localhost:8080 پر چل رہا ہے
```

**کلائنٹ کی مثالیں چلانا:**
```bash
# دوسرے ٹرمینل میں سرور شروع کرنے کے بعد
cd 04-PracticalSamples/calculator

# براہ راست MCP کلائنٹ
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# AI-چلایا گیا کلائنٹ (AZURE_OPENAI_ENDPOINT + az لاگ ان کی ضرورت ہے)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# انٹرایکٹو بوٹ
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### ہاٹ ری لوڈ
سپ رنگ بوٹ ڈیولپ ٹولز ان پروجیکٹس میں شامل ہیں جو ہاٹ ری لوڈ کو سپورٹ کرتے ہیں:
```bash
# جاوا فائلوں میں تبدیلیاں محفوظ کرنے پر خود بخود دوبارہ لوڈ ہو جائیں گی
mvn spring-boot:run
```

## ٹیسٹنگ ہدایات

### ٹیسٹس چلانا

**پروجیکٹ کے تمام ٹیسٹس چلائیں:**
```bash
cd [project-directory]
mvn test
```

**تفصیلی آؤٹ پٹ کے ساتھ ٹیسٹس چلائیں:**
```bash
mvn test -X
```

**مخصوص ٹیسٹ کلاس چلائیں:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### ٹیسٹ کی ساخت
- ٹیسٹ فائلیں JUnit 5 (Jupiter) استعمال کرتی ہیں
- ٹیسٹ کلاسز `src/test/java/` میں واقع ہیں
- کیلکولیٹر پروجیکٹ کی کلائنٹ مثالیں `src/test/java/com/microsoft/mcp/sample/client/` میں ہیں

### دستی ٹیسٹنگ
بہت سی مثالیں انٹرایکٹو ایپلیکیشنز ہیں جن کو دستی ٹیسٹ کرنے کی ضرورت ہے:

1. ایپلیکیشن کو `mvn spring-boot:run` سے شروع کریں
2. اینڈ پوائنٹس کو ٹیسٹ کریں یا CLI کے ساتھ تعامل کریں
3. ہر پروجیکٹ کے README.md میں دستاویزات کے مطابق توقع شدہ رویہ کی تصدیق کریں

### ایزور AI فاؤنڈری کے ساتھ ٹیسٹنگ
- مثالیں چلانے سے پہلے `az login` کے ساتھ سائن ان کریں (کی لیس تصدیق)
- یقینی بنائیں کہ آپ کے اکاؤنٹ کو Cognitive Services OpenAI User کا کردار دیا گیا ہے
- چپٹر 5 میں ذمہ دار AI کی مثال کے ساتھ کانٹینٹ فلٹرنگ کی جانچ کریں

## کوڈ اسٹائل گائیڈ لائنز

### جاوا کنونشنز
- **جاوا ورژن:** جاوا 21 جدید خصوصیات کے ساتھ
- **اسٹائل:** معیاری جاوا کنونشنز کی پیروی کریں
- **نامکاری:** 
  - کلاسز: PascalCase
  - میتھڈز/ویریبلز: camelCase
  - مستقلات: UPPER_SNAKE_CASE
  - پیکج کے نام: چھوٹے حروف میں

### سپرنگ بوٹ پیٹرنز
- کاروباری منطق کے لیے `@Service` استعمال کریں
- REST اینڈ پوائنٹس کے لیے `@RestController` استعمال کریں
- کنفیگریشن `application.yml` یا `application.properties` کے ذریعے
- ہارڈ کوڈڈ ویلیوز کی بجائے ماحولیاتی متغیرات کو ترجیح دیں
- MCP کے متعین طریقوں کے لیے `@Tool` اینوٹیشن استعمال کریں

### فائل کی ترتیب
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

### انحصارات
- میون `pom.xml` کے ذریعے منظم
- ورژن مینجمنٹ کے لیے سپرنگ AI BOM
- AI انٹیگریشن کے لیے لانگ چین4جے
- سپرنگ بوٹ سٹارٹر پیرنٹ سپرنگ انحصارات کے لیے

### کوڈ تبصرے
- عوامی APIs کے لیے جاوا ڈاک شامل کریں
- پیچیدہ AI تعاملات کے لیے وضاحتی تبصرے شامل کریں
- MCP ٹولز کی وضاحتیں واضح طور پر دستاویزی بنائیں

## تعمیر اور تنصیب

### پروجیکٹس بنانا

**ٹیسٹس کے بغیر بنانا:**
```bash
mvn clean install -DskipTests
```

**تمام جانچ کے ساتھ بنانا:**
```bash
mvn clean install
```

**ایپلیکیشن پیکیج کریں:**
```bash
mvn package
# target/ ڈائریکٹری میں JAR بناتا ہے
```

### آؤٹ پٹ ڈائریکٹریز
- مرتب شدہ کلاسز: `target/classes/`
- ٹیسٹ کلاسز: `target/test-classes/`
- جے اے آر فائلز: `target/*.jar`
- میون آرٹیفیکٹس: `target/`

### مخصوص ماحول کی کنفیگریشن

**ترقیاتی:**
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

**پروڈکشن:**
- کی لیس تصدیق کے لیے `az login` کی بجائے مینیجڈ شناخت استعمال کریں
- `AZURE_OPENAI_ENDPOINT` کو اپنے پروڈکشن فاؤنڈری وسائل کی طرف نشان زد کریں
- کنفیگریشن کو ماحولیاتی متغیرات یا ایزور کی وولٹ کے ذریعے منظم کریں

### تنصیب کے نکات
- یہ ایک تعلیمی ذخیرہ ہے جس میں نمونہ ایپلیکیشنز شامل ہیں
- جیسا ہے ویسا پروڈکشن میں تعیناتی کے لیے نہیں بنا
- نمونے پروڈکشن کے استعمال کے لیے پیٹرنز دکھاتے ہیں جن کے مطابق آپ ڈھال سکتے ہیں
- مخصوص تنصیب کی نوٹس کے لیے ہر پروجیکٹ کے README دیکھیے

## اضافی نوٹس

### ایزور AI فاؤنڈری
- **کی لیس تصدیق:** مائیکروسافٹ انٹرا ID کے ساتھ کنیکٹ کریں — کوئی API کیز مینج کرنے کی ضرورت نہیں
- **کوڈ کے طور پر پروویژن:** بائسک + azd (`azd up`) اکاؤنٹ اور ماڈل کی ڈپلائمنٹس بناتے ہیں
- وہی OpenAI-مطابق کوڈ لوکل (`az login`) اور ایزور (مینیجڈ شناخت) دونوں میں چلتا ہے

### متعدد پروجیکٹس کے ساتھ کام کرنا
ہر نمونہ پروجیکٹ اسٹینڈ اکیٹھ ہے:
```bash
# مخصوص پراجیکٹ پر جائیں
cd 04-PracticalSamples/[project-name]

# ہر ایک کا اپنا pom.xml ہے اور اسے آزادانہ طور پر بنایا جا سکتا ہے
mvn clean install
```

### عام مسائل

**جاوا ورژن کا میل نہ کھانا:**
```bash
# جاوا 21 کی تصدیق کریں
java -version
# اگر ضروری ہو تو JAVA_HOME کو اپ ڈیٹ کریں
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**انحصار ڈاؤن لوڈ کے مسائل:**
```bash
# میون کیشے صاف کریں اور پھر سے کوشش کریں
rm -rf ~/.m2/repository
mvn clean install
```

**اینڈ پوائنٹ یا سائن ان نہیں ملا:**
```bash
# موجودہ سیشن میں اینڈپوائنٹ سیٹ کریں اور سائن ان ہوں (بغیر کلید کے)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# یا پروجیکٹ ڈائریکٹری میں .env فائل استعمال کریں
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**پورٹ پہلے سے استعمال میں ہے:**
```bash
# اسپرنگ بوٹ بذریعہ ڈیفالٹ پورٹ 8080 استعمال کرتا ہے
# application.properties میں تبدیلی:
server.port=8081
```

### کثیر زبان سپورٹ
- 45+ زبانوں میں خودکار تراجم کے ذریعے دستاویزات دستیاب
- تراجم `translations/` ڈائریکٹری میں
- ترجمہ گٹ ہب ایکشنز ورک فلو کے ذریعے منظم

### سیکھنے کا راستہ
1. [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md) سے شروع کریں
2. مضامین کو ترتیب سے فالو کریں (01 → 05)
3. ہر چپٹر میں عملی مثالیں مکمل کریں
4. چپٹر 4 میں نمونہ پروجیکٹس کی تلاش کریں
5. چپٹر 5 میں ذمہ دار AI طریقے سیکھیں

### ڈویلپمنٹ کنٹینر
`.devcontainer/devcontainer.json` مندرجہ ذیل کو ترتیب دیتا ہے:
- جاوا 21 ڈیولپمنٹ ماحول
- پہلے سے انسٹال شدہ میون
- VS کوڈ جاوا ایکسٹینشنز
- سپرنگ بوٹ ٹولز
- گٹ ہب کوپائلٹ انٹیگریشن
- ڈاکر-ان-ڈاکر سپورٹ
- ایزور CLI

### کارکردگی کے عوامل
- ایزور AI فاؤنڈری کی ڈپلائمنٹس پر منٹ فی ٹوکن/ریکویسٹ کوٹہ محدود ہوتا ہے
- ایمبیڈنگز کے لیے مناسب بیچ سائز استعمال کریں
- بار بار API کالز کے لیے کیشنگ غور کریں
- لاگت کی بچت کے لیے ٹوکن کے استعمال کی نگرانی کریں

### سیکورٹی نوٹس
- کبھی بھی `.env` فائلز کمٹ نہ کریں (یہ پہلے سے `.gitignore` میں ہیں)
- کی لیس تصدیق (مائیکروسافٹ انٹرا ID) کو API کیز پر ترجیح دیں
- ایزور میں مینیجڈ شناختیں استعمال کریں؛ لوکل ڈویلپمنٹ کے لیے `az login` کریں
- چپٹر 5 میں ذمہ دار AI ہدایات پر عمل کریں

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ڈس کلیمر**:
یہ دستاویز AI ترجمہ سروس [Co-op Translator](https://github.com/Azure/co-op-translator) کے ذریعے ترجمہ کی گئی ہے۔ جبکہ ہم درستگی کے لیے کوشاں ہیں، براہ کرم اس بات سے آگاہ رہیں کہ خودکار ترجمے میں غلطیاں یا عدم درستیاں ہو سکتی ہیں۔ اصل دستاویز اپنے مادری زبان میں مستند ماخذ سمجھی جائے گی۔ حساس معلومات کے لیے پیشہ ور انسانی ترجمہ کی سفارش کی جاتی ہے۔ اس ترجمے کے استعمال سے پیدا ہونے والی کسی بھی غلط فہمی یا غلط تشریح کی ذمہ داری ہم قبول نہیں کرتے۔
<!-- CO-OP TRANSLATOR DISCLAIMER END -->