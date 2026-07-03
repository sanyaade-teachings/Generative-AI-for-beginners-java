# Tutorial Penjana Cerita Haiwan Peliharaan untuk Pemula

## Jadual Kandungan

- [Prasyarat](#prasyarat)
- [Memahami Struktur Projek](#memahami-struktur-projek)
- [Komponen Teras Diterangkan](#komponen-teras-diterangkan)
  - [1. Aplikasi Utama](#1-aplikasi-utama)
  - [2. Pengawal Web](#2-pengawal-web)
  - [3. Perkhidmatan Cerita](#3-perkhidmatan-cerita)
  - [4. Templat Web](#4-templat-web)
  - [5. Konfigurasi](#5-konfigurasi)
- [Menjalankan Aplikasi](#menjalankan-aplikasi)
- [Bagaimana Kesemuanya Berfungsi Bersama](#bagaimana-kesemuanya-berfungsi-bersama)
- [Memahami Integrasi AI](#memahami-integrasi-ai)
- [Langkah Seterusnya](#langkah-seterusnya)

## Prasyarat

Sebelum bermula, pastikan anda mempunyai:
- Java 21 atau lebih tinggi dipasang
- Maven untuk pengurusan pergantungan
- Penempatan model Azure AI Foundry (sediakan dengan `azd up` — lihat [Bab 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), log masuk dengan `az login` (pengesahan tanpa kunci)
- Pemahaman asas tentang Java, Spring Boot, dan pembangunan web

## Memahami Struktur Projek

Projek cerita haiwan peliharaan mempunyai beberapa fail penting:

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

## Komponen Teras Diterangkan

### 1. Aplikasi Utama

**Fail:** `PetStoryApplication.java`

Ini adalah titik masuk untuk aplikasi Spring Boot kami:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Apa yang dilakukan ini:**
- Anotasi `@SpringBootApplication` mengaktifkan konfigurasi automatik dan pengimbasan komponen
- Memulakan pelayan web terbina dalam (Tomcat) di port 8080
- Membuat semua bean dan perkhidmatan Spring yang diperlukan secara automatik

### 2. Pengawal Web

**Fail:** `PetController.java`

Ini mengendalikan semua permintaan web dan interaksi pengguna:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Memulangkan templat index.html
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Pengesahan input
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Membersihkan input untuk keselamatan
        String sanitizedDescription = sanitizeInput(description);
        
        // Jana cerita dengan pengendalian ralat
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Memulangkan templat result.html
            
        } catch (Exception e) {
            // Gunakan cerita gantian jika AI gagal
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Hadkan panjang
    }
}
```

**Ciri utama:**

1. **Pengurusan Laluan**: `@GetMapping("/")` memaparkan borang muat naik, `@PostMapping("/generate-story")` memproses penghantaran
2. **Pengesahan Input**: Memeriksa keterangan kosong dan had panjang
3. **Keselamatan**: Membersihkan input pengguna untuk mengelakkan serangan XSS
4. **Pengendalian Ralat**: Menyediakan cerita gantian apabila perkhidmatan AI gagal
5. **Pengesahan Model**: Memindahkan data ke templat HTML menggunakan Spring `Model`

**Sistem Gantian:**
Pengawal termasuk templat cerita siap guna yang digunakan apabila perkhidmatan AI tidak tersedia:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Gunakan hash keterangan untuk respons yang konsisten
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Perkhidmatan Cerita

**Fail:** `StoryService.java`

Perkhidmatan ini berkomunikasi dengan Azure AI Foundry untuk menjana cerita menggunakan pengesahan tanpa kunci:

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
        
        // Titik akhir OpenAI yang serasi dengan Foundry terletak di bawah /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Pengesahan tanpa kunci dengan Microsoft Entra ID (tiada kunci API)
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
        
        // Konfigurasikan permintaan AI
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Hadkan panjang respons
                .temperature(0.8)          // Kawal kreativiti (0.0-1.0)
                .build();
        
        // Hantar permintaan dan dapatkan respons
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Komponen utama:**

1. **Klien OpenAI**: Menggunakan SDK rasmi OpenAI Java yang dikonfigurasikan untuk Azure AI Foundry (tanpa kunci)
2. **Prompt Sistem**: Menetapkan tingkah laku AI untuk menulis cerita haiwan peliharaan mesra keluarga
3. **Prompt Pengguna**: Memberitahu AI tepat apa cerita yang hendak ditulis berdasarkan keterangan
4. **Parameter**: Mengawal panjang cerita dan tahap kreativiti
5. **Pengendalian Ralat**: Melempar pengecualian yang ditangkap dan diurus oleh pengawal

### 4. Templat Web

**Fail:** `index.html` (Borang Muat Naik)

Halaman utama di mana pengguna menerangkan haiwan peliharaan mereka:

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

**Fail:** `result.html` (Paparan Cerita)

Memaparkan cerita yang dihasilkan:

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

**Ciri templat:**

1. **Integrasi Thymeleaf**: Menggunakan atribut `th:` untuk kandungan dinamik
2. **Reka Bentuk Responsif**: Gaya CSS untuk mudah alih dan desktop
3. **Pengendalian Ralat**: Memaparkan ralat pengesahan kepada pengguna
4. **Pemprosesan Pihak Klien**: JavaScript untuk analisis imej (menggunakan Transformers.js)

### 5. Konfigurasi

**Fail:** `application.properties`

Tetapan konfigurasi untuk aplikasi:

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

**Konfigurasi diterangkan:**

1. **Muat Naik Fail**: Membenarkan imej sehingga 10MB
2. **Pencatatan**: Mengawal maklumat yang dicatat semasa pelaksanaan
3. **Azure AI Foundry**: Menentukan titik hujung dan penempatan model yang digunakan (pengesahan tanpa kunci)
4. **Keselamatan**: Konfigurasi pengendalian ralat untuk mengelakkan pendedahan maklumat sensitif

## Menjalankan Aplikasi

### Langkah 1: Log Masuk dan Tetapkan Titik Hujung Anda

Pengesahan adalah tanpa kunci (Microsoft Entra ID), jadi tiada kunci API. Log masuk dan tetapkan titik hujung Foundry anda:

**Windows (Command Prompt):**
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

**Mengapa ini diperlukan:**
- Azure AI Foundry menggunakan Microsoft Entra ID untuk mengesahkan permintaan inferens
- Pengesahan tanpa kunci bermaksud tiada rahsia dalam kod sumber atau persekitaran anda
- Akaun anda perlu mempunyai peranan **Cognitive Services OpenAI User** pada sumber tersebut

### Langkah 2: Bina dan Jalankan

Navigasi ke direktori projek:
```bash
cd 04-PracticalSamples/petstory
```

Bina aplikasi:
```bash
mvn clean compile
```

Mulakan pelayan:
```bash
mvn spring-boot:run
```

Aplikasi akan bermula di `http://localhost:8080`.

### Langkah 3: Uji Aplikasi

1. **Buka** `http://localhost:8080` dalam pelayar anda
2. **Terangkan** haiwan peliharaan anda dalam ruang teks (contoh, "Seekor golden retriever yang suka bermain dan ambil barang")
3. **Klik** "Generate Story" untuk mendapatkan cerita yang dijana AI
4. **Atau**, muat naik imej haiwan peliharaan untuk menjana keterangan secara automatik
5. **Lihat** cerita kreatif berdasarkan keterangan haiwan peliharaan anda

## Bagaimana Kesemuanya Berfungsi Bersama

Berikut adalah aliran lengkap apabila anda menjana cerita haiwan peliharaan:

1. **Input Pengguna**: Anda menerangkan haiwan peliharaan pada borang web
2. **Penghantaran Borang**: Pelayar menghantar permintaan POST ke `/generate-story`
3. **Pemprosesan Pengawal**: `PetController` mengesahkan dan membersihkan input
4. **Panggilan Perkhidmatan AI**: `StoryService` menghantar permintaan ke model Azure AI Foundry
5. **Penjanaan Cerita**: AI menjana cerita kreatif berdasarkan keterangan
6. **Pengendalian Respons**: Pengawal menerima cerita dan menambahkannya ke model
7. **Penjanaan Templat**: Thymeleaf memaparkan `result.html` dengan cerita tersebut
8. **Paparan**: Pengguna melihat cerita yang dijana dalam pelayar mereka

**Aliran Pengendalian Ralat:**
Jika perkhidmatan AI gagal:
1. Pengawal menangkap pengecualian
2. Menjana cerita gantian menggunakan templat siap
3. Memaparkan cerita gantian dengan nota tentang ketidaktersediaan AI
4. Pengguna masih mendapat cerita, memastikan pengalaman pengguna yang baik

## Memahami Integrasi AI

### Azure AI Foundry (tanpa kunci)
Aplikasi menggunakan Azure AI Foundry dengan pengesahan tanpa kunci (Microsoft Entra ID):

```java
// Pengesahan tanpa kunci - tiada kunci API
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Kejuruteraan Prompt
Perkhidmatan menggunakan prompt yang direka dengan teliti untuk mendapatkan hasil yang baik:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Pemprosesan Respons
Respons AI diekstrak dan disahkan:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Langkah Seterusnya

Untuk lebih banyak contoh, lihat [Bab 04: Contoh praktikal](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Penafian**:
Dokumen ini telah diterjemahkan menggunakan perkhidmatan terjemahan AI [Co-op Translator](https://github.com/Azure/co-op-translator). Walaupun kami berusaha untuk ketepatan, sila ambil maklum bahawa terjemahan automatik mungkin mengandungi kesilapan atau ketidaktepatan. Dokumen asal dalam bahasa asalnya harus dianggap sebagai sumber yang sahih. Untuk maklumat penting, terjemahan oleh manusia profesional adalah disyorkan. Kami tidak bertanggungjawab terhadap sebarang salah faham atau salah tafsir yang timbul daripada penggunaan terjemahan ini.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->