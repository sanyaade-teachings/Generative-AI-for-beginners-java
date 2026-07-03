# AGENTS.md

## Ikhtisar Proyek

Ini adalah repositori edukasi untuk belajar pengembangan Generative AI dengan Java. Menyediakan kursus praktis komprehensif yang mencakup Large Language Models (LLM), rekayasa prompt, embeddings, RAG (Retrieval-Augmented Generation), dan Model Context Protocol (MCP).

**Teknologi Kunci:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI, dan SDK OpenAI

**Arsitektur:**
- Beberapa aplikasi Spring Boot mandiri yang diorganisir berdasarkan bab
- Proyek contoh yang menunjukkan pola AI berbeda
- Siap GitHub Codespaces dengan kontainer pengembangan yang sudah dikonfigurasi

## Perintah Setup

### Prasyarat
- Java 21 atau lebih tinggi
- Maven 3.x
- Langganan Azure dengan deployment model Azure AI Foundry (provisi dengan `azd up`)
- Azure CLI (`az`) dan Azure Developer CLI (`azd`), masuk untuk autentikasi tanpa kunci

### Setup Lingkungan

**Opsi 1: GitHub Codespaces (Direkomendasikan)**
```bash
# Buat repository cabang dan buat codespace dari UI GitHub
# Kontainer dev akan secara otomatis menginstal semua dependensi
# Tunggu sekitar 2 menit untuk pengaturan lingkungan
```

**Opsi 2: Dev Container Lokal**
```bash
# Kloning repositori
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Buka di VS Code dengan ekstensi Dev Containers
# Buka kembali di Container saat diminta
```

**Opsi 3: Setup Lokal**
```bash
# Pasang dependensi
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Verifikasi instalasi
java -version
mvn -version
```

### Konfigurasi

**Setup Azure AI Foundry (tanpa kunci, direkomendasikan):**
```bash
# Menyediakan akun Foundry + penyebaran model sebagai kode
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd menulis examples/basic-chat-azure/.env dengan endpoint Anda (tanpa kunci)
```

**Konfigurasi endpoint manual:**
```bash
# Jika Anda tidak menggunakan azd, atur titik akhir sendiri (otentikasi tetap tanpa kunci melalui az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Edit .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Alur Kerja Pengembangan

### Struktur Proyek
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

### Menjalankan Aplikasi

**Menjalankan aplikasi Spring Boot:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Membangun proyek:**
```bash
cd [project-directory]
mvn clean install
```

**Memulai MCP Calculator Server:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# Server berjalan pada http://localhost:8080
```

**Menjalankan Contoh Klien:**
```bash
# Setelah memulai server di terminal lain
cd 04-PracticalSamples/calculator

# Klien MCP langsung
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# Klien bertenaga AI (memerlukan AZURE_OPENAI_ENDPOINT + az login)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Bot interaktif
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Hot Reload
Spring Boot DevTools disertakan di proyek yang mendukung hot reload:
```bash
# Perubahan pada file Java akan otomatis dimuat ulang saat disimpan
mvn spring-boot:run
```

## Instruksi Pengujian

### Menjalankan Tes

**Jalankan semua tes dalam proyek:**
```bash
cd [project-directory]
mvn test
```

**Jalankan tes dengan output detail:**
```bash
mvn test -X
```

**Jalankan kelas tes tertentu:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Struktur Tes
- File tes menggunakan JUnit 5 (Jupiter)
- Kelas tes berada di `src/test/java/`
- Contoh klien di proyek calculator ada di `src/test/java/com/microsoft/mcp/sample/client/`

### Pengujian Manual
Banyak contoh adalah aplikasi interaktif yang membutuhkan pengujian manual:

1. Mulai aplikasi dengan `mvn spring-boot:run`
2. Uji endpoint atau interaksi dengan CLI
3. Verifikasi perilaku yang diharapkan sesuai dokumentasi di README.md masing-masing proyek

### Pengujian dengan Azure AI Foundry
- Masuk dengan `az login` sebelum menjalankan contoh (autentikasi tanpa kunci)
- Pastikan akun Anda punya peran Cognitive Services OpenAI User pada resource
- Uji penyaringan konten dengan contoh responsible AI di Bab 5

## Panduan Gaya Kode

### Konvensi Java
- **Versi Java:** Java 21 dengan fitur modern
- **Gaya:** Ikuti konvensi Java standar
- **Penamaan:** 
  - Kelas: PascalCase
  - Metode/variabel: camelCase
  - Konstanta: UPPER_SNAKE_CASE
  - Nama paket: huruf kecil

### Pola Spring Boot
- Gunakan `@Service` untuk logika bisnis
- Gunakan `@RestController` untuk endpoint REST
- Konfigurasi via `application.yml` atau `application.properties`
- Variabel lingkungan lebih disukai daripada nilai yang hard-coded
- Gunakan anotasi `@Tool` untuk metode yang diekspos oleh MCP

### Organisasi File
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

### Dependensi
- Dikelola melalui Maven `pom.xml`
- Spring AI BOM untuk manajemen versi
- LangChain4j untuk integrasi AI
- Spring Boot starter parent untuk dependensi Spring

### Komentar Kode
- Tambahkan JavaDoc untuk API publik
- Sertakan komentar penjelas untuk interaksi AI yang kompleks
- Dokumentasikan deskripsi alat MCP dengan jelas

## Build dan Deployment

### Membangun Proyek

**Bangun tanpa tes:**
```bash
mvn clean install -DskipTests
```

**Bangun dengan semua pengecekan:**
```bash
mvn clean install
```

**Bungkus aplikasi:**
```bash
mvn package
# Membuat JAR di direktori target/
```

### Direktori Output
- Kelas hasil kompilasi: `target/classes/`
- Kelas tes: `target/test-classes/`
- File JAR: `target/*.jar`
- Artefak Maven: `target/`

### Konfigurasi Spesifik Lingkungan

**Pengembangan:**
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

**Produksi:**
- Gunakan identitas terkelola daripada `az login` untuk autentikasi tanpa kunci
- Arahkan `AZURE_OPENAI_ENDPOINT` ke resource Foundry produksi Anda
- Kelola konfigurasi melalui variabel lingkungan atau Azure Key Vault

### Pertimbangan Deployment
- Ini repositori edukasi dengan aplikasi contoh
- Tidak dirancang untuk deployment produksi langsung
- Contoh menunjukkan pola untuk diadaptasi guna penggunaan produksi
- Lihat README proyek masing-masing untuk catatan deployment spesifik

## Catatan Tambahan

### Azure AI Foundry
- **Autentikasi tanpa kunci:** terhubung dengan Microsoft Entra ID — tanpa perlu mengelola API keys
- **Diprovinsi sebagai kode:** Bicep + azd (`azd up`) membuat akun dan deployment model
- Kode kompatibel OpenAI yang sama berjalan lokal (`az login`) dan di Azure (identitas terkelola)

### Bekerja dengan Banyak Proyek
Setiap proyek contoh berdiri sendiri:
```bash
# Navigasi ke proyek tertentu
cd 04-PracticalSamples/[project-name]

# Masing-masing memiliki pom.xml sendiri dan dapat dibangun secara mandiri
mvn clean install
```

### Masalah Umum

**Versi Java Tidak Cocok:**
```bash
# Verifikasi Java 21
java -version
# Perbarui JAVA_HOME jika diperlukan
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Masalah Unduh Dependensi:**
```bash
# Bersihkan cache Maven dan coba lagi
rm -rf ~/.m2/repository
mvn clean install
```

**Endpoint atau Sign-in Tidak Ditemukan:**
```bash
# Atur endpoint dalam sesi saat ini dan masuk (tanpa kunci)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Atau gunakan file .env di direktori proyek
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Port Sudah Digunakan:**
```bash
# Spring Boot menggunakan port 8080 secara default
# Ubah di application.properties:
server.port=8081
```

### Dukungan Multi-Bahasa
- Dokumentasi tersedia dalam 45+ bahasa melalui terjemahan otomatis
- Terjemahan ada di direktori `translations/`
- Terjemahan dikelola oleh workflow GitHub Actions

### Jalur Pembelajaran
1. Mulai dengan [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. Ikuti bab secara berurutan (01 → 05)
3. Selesaikan contoh praktis di setiap bab
4. Eksplorasi proyek contoh di Bab 4
5. Pelajari praktik responsible AI di Bab 5

### Kontainer Pengembangan
File `.devcontainer/devcontainer.json` mengonfigurasi:
- Lingkungan pengembangan Java 21
- Maven terpasang sebelumnya
- Ekstensi Java di VS Code
- Alat Spring Boot
- Integrasi GitHub Copilot
- Dukungan Docker-in-Docker
- Azure CLI

### Pertimbangan Performa
- Deployment Azure AI Foundry punya kuota token/permintaan per menit
- Gunakan ukuran batch yang sesuai untuk embeddings
- Pertimbangkan caching untuk panggilan API berulang
- Pantau penggunaan token untuk optimasi biaya

### Catatan Keamanan
- Jangan pernah commit file `.env` (sudah ada di `.gitignore`)
- Lebih pilih autentikasi tanpa kunci (Microsoft Entra ID) dibanding API keys
- Gunakan identitas terkelola di Azure; `az login` untuk pengembangan lokal
- Ikuti pedoman responsible AI di Bab 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Penafian**:
Dokumen ini telah diterjemahkan menggunakan layanan terjemahan AI [Co-op Translator](https://github.com/Azure/co-op-translator). Meskipun kami berupaya untuk mencapai akurasi, harap diketahui bahwa terjemahan otomatis mungkin mengandung kesalahan atau ketidakakuratan. Dokumen asli dalam bahasa aslinya harus dianggap sebagai sumber yang sah. Untuk informasi penting, disarankan menggunakan terjemahan profesional oleh manusia. Kami tidak bertanggung jawab atas kesalahpahaman atau penafsiran yang keliru yang timbul dari penggunaan terjemahan ini.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->