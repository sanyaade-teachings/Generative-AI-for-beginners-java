# Tutorial Generator Cerita Hewan Peliharaan untuk Pemula

## Daftar Isi

- [Prasyarat](#prasyarat)
- [Memahami Struktur Proyek](#memahami-struktur-proyek)
- [Penjelasan Komponen Inti](#penjelasan-komponen-inti)
  - [1. Aplikasi Utama](#1-aplikasi-utama)
  - [2. Pengontrol Web](#2-pengontrol-web)
  - [3. Layanan Cerita](#3-layanan-cerita)
  - [4. Template Web](#4-template-web)
  - [5. Konfigurasi](#5-konfigurasi)
- [Menjalankan Aplikasi](#menjalankan-aplikasi)
- [Cara Kerja Keseluruhan](#cara-kerja-keseluruhan)
- [Memahami Integrasi AI](#memahami-integrasi-ai)
- [Langkah Berikutnya](#langkah-berikutnya)

## Prasyarat

Sebelum memulai, pastikan Anda memiliki:
- Java 21 atau yang lebih tinggi terpasang
- Maven untuk manajemen ketergantungan
- Deployment model Azure AI Foundry (siapkan dengan `azd up` — lihat [Bab 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), masuk dengan `az login` (otomatis tanpa kunci)
- Pemahaman dasar tentang Java, Spring Boot, dan pengembangan web

## Memahami Struktur Proyek

Proyek cerita hewan peliharaan memiliki beberapa file penting:

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

## Penjelasan Komponen Inti

### 1. Aplikasi Utama

**File:** `PetStoryApplication.java`

Ini adalah titik masuk untuk aplikasi Spring Boot kita:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Fungsi ini:**
- Anotasi `@SpringBootApplication` mengaktifkan konfigurasi otomatis dan pemindaian komponen
- Memulai server web bawaan (Tomcat) pada port 8080
- Membuat semua bean dan layanan Spring yang diperlukan secara otomatis

### 2. Pengontrol Web

**File:** `PetController.java`

Ini menangani semua permintaan web dan interaksi pengguna:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Mengembalikan template index.html
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Validasi input
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Membersihkan input untuk keamanan
        String sanitizedDescription = sanitizeInput(description);
        
        // Menghasilkan cerita dengan penanganan kesalahan
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Mengembalikan template result.html
            
        } catch (Exception e) {
            // Gunakan cerita cadangan jika AI gagal
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Batasi panjang
    }
}
```

**Fitur utama:**

1. **Penanganan Rute**: `@GetMapping("/")` menampilkan form unggah, `@PostMapping("/generate-story")` memproses pengiriman
2. **Validasi Input**: Memeriksa deskripsi kosong dan batas panjang
3. **Keamanan**: Membersihkan input pengguna untuk mencegah serangan XSS
4. **Penanganan Kesalahan**: Menyediakan cerita cadangan saat layanan AI gagal
5. **Pengikatan Model**: Mengirim data ke template HTML menggunakan Spring `Model`

**Sistem Cadangan:**
Pengontrol mencakup template cerita yang sudah ditulis sebelumnya yang digunakan bila layanan AI tidak tersedia:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Gunakan hash deskripsi untuk respons yang konsisten
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Layanan Cerita

**File:** `StoryService.java`

Layanan ini berkomunikasi dengan Azure AI Foundry untuk menghasilkan cerita dengan otentikasi tanpa kunci:

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
        
        // Endpoint yang kompatibel dengan OpenAI Foundry berada di bawah /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Autentikasi tanpa kunci dengan Microsoft Entra ID (tanpa kunci API)
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
                .maxCompletionTokens(500)  // Batasi panjang respons
                .temperature(0.8)          // Kontrol kreativitas (0.0-1.0)
                .build();
        
        // Kirim permintaan dan dapatkan respons
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Komponen utama:**

1. **Klien OpenAI**: Menggunakan SDK Java OpenAI resmi yang dikonfigurasi untuk Azure AI Foundry (tanpa kunci)
2. **Prompt Sistem**: Menetapkan perilaku AI untuk menulis cerita hewan peliharaan yang ramah keluarga
3. **Prompt Pengguna**: Memberi tahu AI persis cerita apa yang harus dibuat berdasarkan deskripsi
4. **Parameter**: Mengatur panjang dan tingkat kreativitas cerita
5. **Penanganan Kesalahan**: Melempar pengecualian yang ditangani oleh pengontrol

### 4. Template Web

**File:** `index.html` (Form Unggah)

Halaman utama tempat pengguna mendeskripsikan hewan peliharaan mereka:

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

**File:** `result.html` (Tampilan Cerita)

Menampilkan cerita yang dihasilkan:

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

**Fitur template:**

1. **Integrasi Thymeleaf**: Menggunakan atribut `th:` untuk konten dinamis
2. **Desain Responsif**: Styling CSS untuk perangkat mobile dan desktop
3. **Penanganan Kesalahan**: Menampilkan kesalahan validasi ke pengguna
4. **Proses Klien**: JavaScript untuk analisis gambar (menggunakan Transformers.js)

### 5. Konfigurasi

**File:** `application.properties`

Pengaturan konfigurasi aplikasi:

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

**Penjelasan konfigurasi:**

1. **Unggah Berkas**: Mengizinkan gambar hingga 10MB
2. **Logging**: Mengontrol informasi yang dicatat selama eksekusi
3. **Azure AI Foundry**: Menentukan endpoint dan deployment model yang digunakan (autentikasi tanpa kunci)
4. **Keamanan**: Konfigurasi penanganan kesalahan agar tidak mengekspos informasi sensitif

## Menjalankan Aplikasi

### Langkah 1: Masuk dan Setel Endpoint Anda

Autentikasi menggunakan keyless (Microsoft Entra ID), jadi tanpa kunci API. Masuk dan setel endpoint Foundry Anda:

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

**Mengapa ini perlu:**
- Azure AI Foundry menggunakan Microsoft Entra ID untuk mengautentikasi permintaan inferensi
- Otentikasi tanpa kunci berarti tidak ada rahasia di kode sumber atau lingkungan Anda
- Akun Anda harus memiliki peran **Cognitive Services OpenAI User** pada resource tersebut

### Langkah 2: Build dan Jalankan

Masuk ke direktori proyek:
```bash
cd 04-PracticalSamples/petstory
```

Build aplikasi:
```bash
mvn clean compile
```

Mulai server:
```bash
mvn spring-boot:run
```

Aplikasi akan berjalan di `http://localhost:8080`.

### Langkah 3: Uji Aplikasi

1. **Buka** `http://localhost:8080` di browser Anda
2. **Jelaskan** hewan peliharaan Anda di area teks (misalnya, "Seekor golden retriever yang lincah dan suka mengambil bola")
3. **Klik** "Generate Story" untuk mendapatkan cerita AI yang dihasilkan
4. **Sebagai alternatif**, unggah gambar hewan peliharaan untuk otomatis menghasilkan deskripsi
5. **Lihat** cerita kreatif berdasarkan deskripsi hewan peliharaan Anda

## Cara Kerja Keseluruhan

Berikut alur lengkap saat Anda menghasilkan cerita hewan peliharaan:

1. **Input Pengguna**: Anda mendeskripsikan hewan peliharaan di form web
2. **Pengiriman Form**: Browser mengirim permintaan POST ke `/generate-story`
3. **Pemrosesan Pengontrol**: `PetController` memvalidasi dan membersihkan input
4. **Panggilan Layanan AI**: `StoryService` mengirim permintaan ke model Azure AI Foundry
5. **Pembuatan Cerita**: AI menghasilkan cerita kreatif berdasarkan deskripsi
6. **Penanganan Respons**: Pengontrol menerima cerita dan menambahkannya ke model
7. **Render Template**: Thymeleaf merender `result.html` dengan cerita tersebut
8. **Tampilan**: Pengguna melihat cerita yang dihasilkan di browser mereka

**Alur Penanganan Kesalahan:**
Jika layanan AI gagal:
1. Pengontrol menangkap pengecualian
2. Membuat cerita cadangan menggunakan template yang sudah dibuat
3. Menampilkan cerita cadangan dengan catatan bahwa AI tidak tersedia
4. Pengguna tetap mendapat cerita, memastikan pengalaman pengguna tetap baik

## Memahami Integrasi AI

### Azure AI Foundry (tanpa kunci)
Aplikasi menggunakan Azure AI Foundry dengan autentikasi tanpa kunci (Microsoft Entra ID):

```java
// Otentikasi tanpa kunci - tanpa kunci API
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Rekayasa Prompt
Layanan menggunakan prompt yang dirancang dengan cermat untuk mendapatkan hasil baik:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Pemrosesan Respons
Respons AI diambil dan divalidasi:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Langkah Berikutnya

Untuk contoh lebih lanjut, lihat [Bab 04: Contoh praktis](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Penafian**:
Dokumen ini telah diterjemahkan menggunakan layanan terjemahan AI [Co-op Translator](https://github.com/Azure/co-op-translator). Meskipun kami berupaya untuk mencapai akurasi, harap diketahui bahwa terjemahan otomatis mungkin mengandung kesalahan atau ketidakakuratan. Dokumen asli dalam bahasa aslinya harus dianggap sebagai sumber yang sah. Untuk informasi penting, disarankan menggunakan terjemahan profesional oleh manusia. Kami tidak bertanggung jawab atas kesalahpahaman atau penafsiran yang keliru yang timbul dari penggunaan terjemahan ini.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->