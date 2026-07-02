# Yeni Başlayanlar İçin Evcil Hayvan Hikayesi Oluşturucu Eğitimi

## İçindekiler

- [Gereksinimler](#gereksinimler)
- [Proje Yapısını Anlamak](#proje-yapısını-anlamak)
- [Temel Bileşenlerin Açıklaması](#temel-bileşenlerin-açıklaması)
  - [1. Ana Uygulama](#1-ana-uygulama)
  - [2. Web Denetleyici](#2-web-denetleyici)
  - [3. Hikaye Servisi](#3-hikaye-servisi)
  - [4. Web Şablonları](#4-web-şablonları)
  - [5. Yapılandırma](#5-yapılandırma)
- [Uygulamayı Çalıştırmak](#uygulamayı-çalıştırmak)
- [Hepsi Nasıl Birlikte Çalışır](#hepsi-nasıl-birlikte-çalışır)
- [Yapay Zeka Entegrasyonunu Anlamak](#yapay-zeka-entegrasyonunu-anlamak)
- [Sonraki Adımlar](#sonraki-adımlar)

## Gereksinimler

Başlamadan önce şunlara sahip olduğunuzdan emin olun:
- Java 21 veya üzeri kurulu
- Bağımlılık yönetimi için Maven
- Bir Azure AI Foundry model dağıtımı (bunu `azd up` ile sağlayın — bkz. [Bölüm 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), `az login` ile (anahtarsız kimlik doğrulama)
- Java, Spring Boot ve web geliştirmeye temel düzeyde hakimiyet

## Proje Yapısını Anlamak

Evcil hayvan hikayesi projesi birkaç önemli dosyaya sahiptir:

```
petstory/
├── src/main/java/com/example/petstory/
│   ├── PetStoryApplication.java       # Main Spring Boot application
│   ├── PetController.java             # Web request handler
│   ├── StoryService.java              # AI story generation service
│   └── SecurityConfig.java            # Security configuration
├── src/main/resources/
│   ├── application.properties         # App configuration
│   └── templates/
│       ├── index.html                 # Upload form page
│       └── result.html               # Story display page
└── pom.xml                           # Maven dependencies
```

## Temel Bileşenlerin Açıklaması

### 1. Ana Uygulama

**Dosya:** `PetStoryApplication.java`

Bu bizim Spring Boot uygulamasının giriş noktasıdır:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Yaptıkları:**
- `@SpringBootApplication` açıklaması otomatik yapılandırma ve bileşen taramayı etkinleştirir
- 8080 portunda gömülü bir web sunucusu (Tomcat) başlatır
- Gerekli tüm Spring bean'leri ve servisleri otomatik olarak oluşturur

### 2. Web Denetleyici

**Dosya:** `PetController.java`

Tüm web isteklerini ve kullanıcı etkileşimlerini yönetir:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // index.html şablonunu döndürür
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Girdi doğrulama
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Güvenlik için girdi temizleme
        String sanitizedDescription = sanitizeInput(description);
        
        // Hata yönetimi ile hikaye oluşturma
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // result.html şablonunu döndürür
            
        } catch (Exception e) {
            // AI başarısız olursa yedek hikaye kullan
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Uzunluğu sınırla
    }
}
```

**Ana özellikler:**

1. **Rota Yönetimi**: `@GetMapping("/")` yükleme formunu gösterir, `@PostMapping("/generate-story")` gönderimleri işler
2. **Girdi Doğrulama**: Boş açıklama ve uzunluk sınırı kontrolleri yapar
3. **Güvenlik**: Kullanıcı girdisini XSS saldırılarına karşı temizler
4. **Hata Yönetimi**: AI servisi başarısız olursa yedek hikayeler sağlar
5. **Model Bağlama**: Verileri Spring'in `Model` nesnesi aracılığıyla HTML şablonlarına geçirir

**Yedek Sistem:**
Denetleyici, AI servisi kullanılamadığında kullanılmak üzere önceden yazılmış hikaye şablonları içerir:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Tutarlı yanıtlar için açıklama karmasını kullanın
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Hikaye Servisi

**Dosya:** `StoryService.java`

Bu servis, anahtarsız kimlik doğrulama kullanarak Azure AI Foundry ile iletişim kurup hikayeler oluşturur:

```java
@Service
public class StoryService {
    
    private final OpenAIClient openAIClient;
    private final String modelName;
    
    public StoryService(@Value("${azure.openai.endpoint:}") String endpoint,
                       @Value("${azure.openai.deployment:gpt-4o-mini}") String modelName) {
        this.modelName = modelName;
        if (endpoint == null || endpoint.isBlank()) {
            endpoint = System.getenv("AZURE_OPENAI_ENDPOINT");
        }
        
        // Foundry'nin OpenAI uyumlu uç noktası /openai/v1/ altında yer alır
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Anahtarsız kimlik doğrulama Microsoft Entra ID ile (API anahtarı yok)
        DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
        this.openAIClient = OpenAIOkHttpClient.builder()
                .baseUrl(baseUrl)
                .credential(BearerTokenCredential.create(
                        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
                .build();
    }
    
    public String generateStory(String description) {
        String systemPrompt = "You are a creative storyteller who writes fun, " +
                             "family-friendly short stories about pets. " +
                             "Keep stories under 500 words and appropriate for all ages.";
        
        String userPrompt = "Write a fun short story about a pet described as: " + description;
        
        // AI isteğini yapılandır
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Yanıt uzunluğunu sınırla
                .temperature(0.8)          // Yaratıcılığı kontrol et (0.0-1.0)
                .build();
        
        // İstek gönder ve yanıt al
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Ana bileşenler:**

1. **OpenAI İstemcisi**: Azure AI Foundry için resmi OpenAI Java SDK'sını anahtarsız yapılandırılmış şekilde kullanır
2. **Sistem İsteği (Prompt)**: AI'nın aile dostu evcil hayvan hikayeleri yazmasını sağlar
3. **Kullanıcı İsteği (Prompt)**: Açıklamaya göre AI'ya tam olarak hangi hikayeyi yazacağını söyler
4. **Parametreler**: Hikaye uzunluğunu ve yaratıcılık seviyesini kontrol eder
5. **Hata Yönetimi**: Denetleyici tarafından yakalanan istisnalar fırlatır

### 4. Web Şablonları

**Dosya:** `index.html` (Yükleme Formu)

Kullanıcıların evcil hayvanlarını tanımladığı ana sayfa:

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Pet Story Generator</title>
    <!-- CSS styling -->
</head>
<body>
    <div class="container">
        <h1>Pet Story Generator</h1>
        <p>Describe your pet and we'll create a fun story about them!</p>
        
        <!-- Error message display -->
        <div th:if="${error}" class="error" th:text="${error}"></div>
        
        <!-- Story generation form -->
        <form action="/generate-story" method="post">
            <div class="form-group">
                <label for="description">Describe your pet:</label>
                <textarea id="description" name="description" 
                         placeholder="Tell us about your pet - what they look like, their personality, favorite activities..."
                         maxlength="1000" required></textarea>
            </div>
            <button type="submit" class="btn btn-primary">Generate Story</button>
        </form>
        
        <!-- Image upload section with client-side processing -->
        <div class="upload-section">
            <h2>Or Upload a Photo</h2>
            <input type="file" id="imageInput" accept="image/*" />
            <button onclick="analyzeImage()" class="upload-btn">Analyze Image</button>
        </div>
        
        <script>
            // Client-side image analysis using Transformers.js
            async function analyzeImage() {
                // Image processing code here
                // Generates description automatically from uploaded image
            }
        </script>
    </div>
</body>
</html>
```

**Dosya:** `result.html` (Hikaye Gösterimi)

Oluşturulan hikayeyi gösterir:

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Pet Story Result</title>
</head>
<body>
    <div class="container">
        <h1>Your Pet's Story</h1>
        
        <div class="result-section">
            <div class="result-label">Pet Description:</div>
            <div class="result-content" th:text="${caption}"></div>
        </div>
        
        <div class="result-section">
            <div class="result-label">Generated Story:</div>
            <div class="result-content" th:text="${story}"></div>
        </div>
        
        <div class="result-section" th:if="${analysisType}">
            <div class="result-label">Analysis Type:</div>
            <div class="result-content" th:text="${analysisType}"></div>
        </div>
        
        <a href="/" class="back-link">Generate Another Story</a>
    </div>
</body>
</html>
```

**Şablon özellikleri:**

1. **Thymeleaf Entegrasyonu**: Dinamik içerik için `th:` özniteliklerini kullanır
2. **Duyarlı Tasarım**: Mobil ve masaüstü için CSS stili
3. **Hata Yönetimi**: Kullanıcıya doğrulama hatalarını gösterir
4. **İstemci Tarafı İşleme**: Transformers.js kullanarak görsel analiz için JavaScript

### 5. Yapılandırma

**Dosya:** `application.properties`

Uygulama yapılandırma ayarları:

```properties
spring.application.name=pet-story-app

# File upload limits
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=10MB

# Logging configuration
logging.level.com.example.petstory=INFO

# Azure AI Foundry (keyless) configuration
azure.openai.endpoint=${AZURE_OPENAI_ENDPOINT:}
azure.openai.deployment=${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}
```

**Yapılandırma açıklaması:**

1. **Dosya Yükleme**: 10MB'a kadar görsellere izin verir
2. **Kayıt Tutma**: Çalışma sırasında hangi bilgilerin kaydedileceğini kontrol eder
3. **Azure AI Foundry**: Kullanılacak uç noktayı ve model dağıtımını belirtir (anahtarsız)
4. **Güvenlik**: Hassas bilgilerin açığa çıkmasını önlemek için hata yönetimi yapılandırması

## Uygulamayı Çalıştırmak

### Adım 1: Oturum Açın ve Uç Noktanızı Ayarlayın

Kimlik doğrulama anahtarsızdır (Microsoft Entra ID), bu yüzden API anahtarı yoktur. Oturum açın ve Foundry uç noktanızı ayarlayın:

**Windows (Komut İstemi):**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**Windows (PowerShell):**
```powershell
az login
$env:AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
```

**Linux/macOS:**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**Bunun Neden Gerekli Olduğu:**
- Azure AI Foundry, sorgu isteklerini Microsoft Entra ID ile doğrular
- Anahtarsız kimlik doğrulama, kaynak kod veya ortamda gizli bilgi olmaması demektir
- Hesabınızın ilgili kaynak üzerinde **Cognitive Services OpenAI User** rolüne sahip olması gerekir

### Adım 2: Derle ve Çalıştır

Proje dizinine gidin:
```bash
cd 04-PracticalSamples/petstory
```

Uygulamayı derleyin:
```bash
mvn clean compile
```

Sunucuyu başlatın:
```bash
mvn spring-boot:run
```

Uygulama `http://localhost:8080` adresinde başlayacaktır.

### Adım 3: Uygulamayı Test Edin

1. Tarayıcınızda `http://localhost:8080` adresini açın
2. Metin alanına evcil hayvanınızı tanımlayın (örneğin, "Top oynamayı seven enerjik bir golden retriever")
3. "Hikaye Oluştur" butonuna tıklayarak AI tarafından oluşturulan hikayeyi alın
4. Alternatif olarak, bir evcil hayvan resmi yükleyerek otomatik açıklama oluşturabilirsiniz
5. Evcil hayvan açıklamanıza dayalı yaratıcı hikayeyi görüntüleyin

## Hepsi Nasıl Birlikte Çalışır

Bir evcil hayvan hikayesi oluşturduğunuzda tam süreç şöyle işler:

1. **Kullanıcı Girişi**: Web formunda evcil hayvanınızı tanımlarsınız
2. **Form Gönderimi**: Tarayıcı, `/generate-story` adresine POST isteği gönderir
3. **Denetleyici İşlemi**: `PetController` girdiyi doğrular ve temizler
4. **AI Servisi Çağrısı**: `StoryService` isteği Azure AI Foundry modeline gönderir
5. **Hikaye Oluşumu**: AI, açıklamaya dayalı yaratıcı bir hikaye oluşturur
6. **Yanıt İşleme**: Denetleyici hikayeyi alır ve modele ekler
7. **Şablon Çizimi**: Thymeleaf `result.html` dosyasını hikayeyle işler
8. **Gösterim**: Kullanıcı tarayıcısında oluşturulan hikaye gösterilir

**Hata Yönetimi Akışı:**
AI servisi başarısız olursa:
1. Denetleyici hatayı yakalar
2. Önceden yazılmış yedek şablonlarla bir hikaye oluşturur
3. AI hizmetinin kullanılamadığına dair bir notla yedek hikayeyi gösterir
4. Kullanıcı yine de bir hikaye alır, böylece iyi bir kullanıcı deneyimi sağlanır

## Yapay Zeka Entegrasyonunu Anlamak

### Azure AI Foundry (anahtarsız)
Uygulama, anahtarsız kimlik doğrulama (Microsoft Entra ID) ile Azure AI Foundry kullanır:

```java
// Anahtarsız kimlik doğrulama - API anahtarı yok
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### İstek Mühendisliği
Servis, iyi sonuçlar almak için özenle hazırlanmış istekler kullanır:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Yanıt İşleme
AI yanıtı çıkarılır ve doğrulanır:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Sonraki Adımlar

Daha fazla örnek için bkz. [Bölüm 04: Pratik örnekler](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Feragatname**:
Bu belge, AI çeviri hizmeti [Co-op Translator](https://github.com/Azure/co-op-translator) kullanılarak çevrilmiştir. Doğruluk için çaba sarf etsek de, otomatik çevirilerin hata veya yanlışlık içerebileceğini lütfen unutmayınız. Orijinal belge, kendi dilinde yetkili kaynak olarak kabul edilmelidir. Kritik bilgiler için profesyonel insan çevirisi önerilir. Bu çevirinin kullanımı sonucu ortaya çıkabilecek yanlış anlamalardan veya yanlış yorumlamalardan sorumlu değiliz.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->