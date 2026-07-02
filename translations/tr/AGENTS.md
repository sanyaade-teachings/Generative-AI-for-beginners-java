# AGENTS.md

## Proje Genel Bakışı

Bu, Java ile Üretici Yapay Zeka geliştirmeyi öğrenmek için eğitsel bir depodur. Büyük Dil Modelleri (LLM'ler), prompt mühendisliği, gömme yapılar, RAG (Retrieval-Augmented Generation) ve Model Context Protocol (MCP) dahil olmak üzere kapsamlı, uygulamalı bir kurs sunar.

**Temel Teknolojiler:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI ve OpenAI SDK'ları

**Mimari:**
- Bölümler halinde düzenlenmiş birden fazla bağımsız Spring Boot uygulaması
- Farklı yapay zeka desenlerini gösteren örnek projeler
- Ön yapılandırılmış geliştirme konteynerleri ile GitHub Codespaces uyumlu

## Kurulum Komutları

### Önkoşullar
- Java 21 veya üstü
- Maven 3.x
- Azure AI Foundry model dağıtımı olan Azure aboneliği (`azd up` ile sağlayın)
- Azure CLI (`az`) ve Azure Developer CLI (`azd`), anahtarsız kimlik doğrulama için giriş yapılmış

### Ortam Kurulumu

**Seçenek 1: GitHub Codespaces (Önerilen)**
```bash
# Depoyu çatallayın ve GitHub UI'den bir kod alanı oluşturun
# Geliştirme konteyneri tüm bağımlılıkları otomatik olarak yükleyecektir
# Ortam kurulumu için ~2 dakika bekleyin
```

**Seçenek 2: Yerel Geliştirme Konteyneri**
```bash
# Depoyu klonla
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Dev Containers eklentisi ile VS Code'da aç
# İstendiğinde konteyner içinde yeniden aç
```

**Seçenek 3: Yerel Kurulum**
```bash
# Bağımlılıkları yükle
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Kurulumu doğrula
java -version
mvn -version
```

### Yapılandırma

**Azure AI Foundry Kurulumu (anahtarsız, önerilir):**
```bash
# Foundry hesabını ve model dağıtımlarını kod olarak sağlama
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd, endpoint’iniz ile examples/basic-chat-azure/.env dosyasını yazar (anahtar yok)
```

**Manuel uç nokta yapılandırması:**
```bash
# Eğer azd kullanmadıysanız, uç noktayı kendiniz ayarlayın (kimlik doğrulama, az login ile anahtarsız kalır)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# .env dosyasını düzenleyin: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Geliştirme İş Akışı

### Proje Yapısı
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

### Uygulamaları Çalıştırma

**Bir Spring Boot uygulamasını çalıştırma:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Projeyi derleme:**
```bash
cd [project-directory]
mvn clean install
```

**MCP Calculator Server'ı başlatma:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# Sunucu http://localhost:8080 üzerinde çalışıyor
```

**İstemci Örneklerini Çalıştırma:**
```bash
# Sunucuyu başka bir terminalde başlattıktan sonra
cd 04-PracticalSamples/calculator

# Doğrudan MCP istemcisi
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# Yapay zeka destekli istemci (AZURE_OPENAI_ENDPOINT + az giriş gerektirir)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Etkileşimli bot
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Hot Reload
Hot reload destekleyen projelerde Spring Boot DevTools dahildir:
```bash
# Java dosyalarındaki değişiklikler kaydedildiğinde otomatik olarak yeniden yüklenecektir
mvn spring-boot:run
```

## Test Talimatları

### Testleri Çalıştırma

**Bir projedeki tüm testleri çalıştırma:**
```bash
cd [project-directory]
mvn test
```

**Ayrıntılı çıktı ile testleri çalıştırma:**
```bash
mvn test -X
```

**Belirli bir test sınıfını çalıştırma:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Test Yapısı
- Test dosyaları JUnit 5 (Jupiter) kullanır
- Test sınıfları `src/test/java/` dizinindedir
- Calculator projesindeki istemci örnekleri `src/test/java/com/microsoft/mcp/sample/client/` yolundadır

### Manuel Test
Birçok örnek uygulama kullanıcı etkileşimi gerektirir:

1. Uygulamayı `mvn spring-boot:run` ile başlatın
2. Uç noktaları test edin veya CLI ile etkileşimde bulunun
3. Beklenen davranışın her projenin README.md dosyasındaki dokümantasyonla uyumlu olduğunu doğrulayın

### Azure AI Foundry ile Test
- Örnekleri çalıştırmadan önce `az login` ile giriş yapın (anahtarsız kimlik doğrulama)
- Hesabınızın ilgili kaynakta Cognitive Services OpenAI User rolü olduğundan emin olun
- 5. Bölümdeki sorumlu AI örneği ile içerik filtrelemeyi test edin

## Kod Stili Rehberi

### Java Konvansiyonları
- **Java Sürümü:** Java 21, modern özelliklerle
- **Stil:** Standart Java konvansiyonlarını takip edin
- **İsimlendirme:** 
  - Sınıflar: PascalCase
  - Metot/değişkenler: camelCase
  - Sabitler: UPPER_SNAKE_CASE
  - Paket isimleri: küçük harf

### Spring Boot Desenleri
- İş mantığı için `@Service` kullanın
- REST uç noktaları için `@RestController` kullanın
- Yapılandırma için `application.yml` veya `application.properties`
- Sert kodlanmış değerler yerine ortam değişkenlerini tercih edin
- MCP tarafından açığa çıkarılan metotlarda `@Tool` anotasyonu kullanın

### Dosya Organizasyonu
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

### Bağımlılıklar
- Maven `pom.xml` ile yönetilir
- Versiyon yönetimi için Spring AI BOM
- Yapay zeka entegrasyonları için LangChain4j
- Spring bağımlılıkları için Spring Boot starter parent

### Kod Yorumları
- Genel API'ler için JavaDoc ekleyin
- Karmaşık yapay zeka etkileşimlerini açıklayıcı yorumlarla destekleyin
- MCP araç açıklamalarını açıkça dokümante edin

## Derleme ve Dağıtım

### Projeleri Derleme

**Testler olmadan derleme:**
```bash
mvn clean install -DskipTests
```

**Tüm kontrollerle derleme:**
```bash
mvn clean install
```

**Uygulama paketleme:**
```bash
mvn package
# target/ dizininde JAR oluşturur
```

### Çıktı Dizini
- Derlenmiş sınıflar: `target/classes/`
- Test sınıfları: `target/test-classes/`
- JAR dosyaları: `target/*.jar`
- Maven artefaktları: `target/`

### Ortama Özgü Yapılandırma

**Geliştirme:**
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

**Prodüksiyon:**
- Anahtarsız kimlik doğrulama için `az login` yerine yönetilen kimlik kullanın
- `AZURE_OPENAI_ENDPOINT`'i prodüksiyon Foundry kaynağınıza yönlendirin
- Yapılandırmayı ortam değişkenleri veya Azure Key Vault ile yönetin

### Dağıtım Dikkatleri
- Bu eğitim amaçlı bir depo olup örnek uygulamalar içerir
- Üretim dağıtımı için doğrudan tasarlanmamıştır
- Örnekler, üretime uyarlanabilecek desenleri gösterir
- Özel dağıtım notları için proje README dosyalarına bakın

## Ek Notlar

### Azure AI Foundry
- **Anahtarsız kimlik doğrulama:** Microsoft Entra ID ile bağlantı — API anahtarı yönetimi yok
- **Kod olarak sağlama:** Bicep + azd (`azd up`) ile hesap ve model dağıtımları oluşturulur
- Aynı OpenAI uyumlu kod yerelde (`az login`) ve Azure’da (yönetilen kimlik) çalışır

### Çoklu Projelerle Çalışma
Her örnek proje bağımsızdır:
```bash
# Belirli projeye git
cd 04-PracticalSamples/[project-name]

# Her birinin kendi pom.xml dosyası vardır ve bağımsız olarak derlenebilir
mvn clean install
```

### Yaygın Sorunlar

**Java Sürüm Uyumsuzluğu:**
```bash
# Java 21 doğrulayın
java -version
# Gerekirse JAVA_HOME'u güncelleyin
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Bağımlılık İndirme Sorunları:**
```bash
# Maven önbelleğini temizleyin ve yeniden deneyin
rm -rf ~/.m2/repository
mvn clean install
```

**Uç Nokta veya Giriş Bulunamadı:**
```bash
# Geçerli oturumda uç noktayı ayarlayın ve oturum açın (anahtarsız)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Veya proje dizininde bir .env dosyası kullanın
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Port Zaten Kullanılıyor:**
```bash
# Spring Boot varsayılan olarak 8080 portunu kullanır
# application.properties dosyasında değişiklik:
server.port=8081
```

### Çok Dilli Destek
- Dokümantasyon 45+ dilde otomatik çeviri ile mevcut
- Çeviriler `translations/` dizininde
- Çeviri GitHub Actions iş akışı ile yönetilir

### Öğrenme Yolu
1. [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md) ile başlayın
2. Bölümleri sırasıyla takip edin (01 → 05)
3. Her bölümdeki uygulamalı örnekleri tamamlayın
4. 4. Bölümdeki örnek projeleri keşfedin
5. 5. Bölümde sorumlu AI uygulamalarını öğrenin

### Geliştirme Konteyneri
`.devcontainer/devcontainer.json` şunları yapılandırır:
- Java 21 geliştirme ortamı
- Önceden yüklü Maven
- VS Code Java eklentileri
- Spring Boot araçları
- GitHub Copilot entegrasyonu
- Docker içinde Docker desteği
- Azure CLI

### Performans Dikkatleri
- Azure AI Foundry dağıtımları dakikalık token/istek kotası ile sınırlıdır
- Gömme yapılar için uygun toplu boyutlar kullanın
- Tekrarlanan API çağrıları için önbellekleme düşünün
- Maliyet optimizasyonu için token kullanımını izleyin

### Güvenlik Notları
- `.env` dosyalarını asla commit etmeyin (zaten `.gitignore` içinde)
- API anahtarları yerine anahtarsız kimlik doğrulamayı tercih edin (Microsoft Entra ID)
- Azure’da yönetilen kimlikleri kullanın; yerel geliştirme için `az login`
- 5. Bölümdeki sorumlu AI yönergelerini takip edin

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Feragatname**:
Bu belge, AI çeviri hizmeti [Co-op Translator](https://github.com/Azure/co-op-translator) kullanılarak çevrilmiştir. Doğruluk için çaba sarf etsek de, otomatik çevirilerin hata veya yanlışlık içerebileceğini lütfen unutmayınız. Orijinal belge, kendi dilinde yetkili kaynak olarak kabul edilmelidir. Kritik bilgiler için profesyonel insan çevirisi önerilir. Bu çevirinin kullanımı sonucu ortaya çıkabilecek yanlış anlamalardan veya yanlış yorumlamalardan sorumlu değiliz.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->