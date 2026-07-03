# AGENTS.md

## Gambaran Projek

Ini adalah repositori pendidikan untuk mempelajari pembangunan AI Generatif dengan Java. Ia menyediakan kursus praktikal yang komprehensif merangkumi Model Bahasa Besar (LLM), kejuruteraan arahan, penanaman, RAG (Generasi Beraugmen Pengambilan), dan Protokol Konteks Model (MCP).

**Teknologi Utama:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI, dan SDK OpenAI

**Senibina:**
- Beberapa aplikasi Spring Boot berdiri sendiri yang diatur mengikut bab
- Projek contoh yang menunjukkan pelbagai corak AI
- Sedia GitHub Codespaces dengan kontena pembangunan yang telah dikonfigurasikan

## Perintah Persediaan

### Prasyarat
- Java 21 atau lebih tinggi
- Maven 3.x
- Langganan Azure dengan penyebaran model Azure AI Foundry (disediakan dengan `azd up`)
- Azure CLI (`az`) dan Azure Developer CLI (`azd`), log masuk untuk pengesahan tanpa kunci

### Persediaan Persekitaran

**Pilihan 1: GitHub Codespaces (Disyorkan)**
```bash
# Fork repositori dan buat codespace dari UI GitHub
# Kontena dev akan memasang semua kebergantungan secara automatik
# Tunggu ~2 minit untuk penyediaan persekitaran
```

**Pilihan 2: Kontena Dev Tempatan**
```bash
# Klon repositori
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Buka dalam VS Code dengan sambungan Dev Containers
# Buka semula dalam Container apabila diminta
```

**Pilihan 3: Persediaan Tempatan**
```bash
# Pasang pergantungan
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Sahkan pemasangan
java -version
mvn -version
```

### Konfigurasi

**Persediaan Azure AI Foundry (tanpa kunci, disyorkan):**
```bash
# Sediakan akaun Foundry + penyebaran model sebagai kod
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd menulis examples/basic-chat-azure/.env dengan titik akhir anda (tanpa kunci)
```

**Konfigurasi endpoint manual:**
```bash
# Jika anda tidak menggunakan azd, tetapkan titik akhir sendiri (auth kekal tanpa kunci melalui az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Sunting .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Aliran Kerja Pembangunan

### Struktur Projek
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

**Membina projek:**
```bash
cd [project-directory]
mvn clean install
```

**Memulakan MCP Calculator Server:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# Pelayan berjalan di http://localhost:8080
```

**Menjalankan Contoh Pelanggan:**
```bash
# Selepas memulakan pelayan dalam terminal lain
cd 04-PracticalSamples/calculator

# Klien MCP langsung
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# Klien berkuasa AI (memerlukan AZURE_OPENAI_ENDPOINT + az login)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Bot interaktif
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Muat Semula Panas
Spring Boot DevTools disertakan dalam projek yang menyokong muat semula panas:
```bash
# Perubahan pada fail Java akan dimuat semula secara automatik apabila disimpan
mvn spring-boot:run
```

## Arahan Ujian

### Menjalankan Ujian

**Jalankan semua ujian dalam projek:**
```bash
cd [project-directory]
mvn test
```

**Jalankan ujian dengan output terperinci:**
```bash
mvn test -X
```

**Jalankan kelas ujian tertentu:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Struktur Ujian
- Fail ujian menggunakan JUnit 5 (Jupiter)
- Kelas ujian terletak dalam `src/test/java/`
- Contoh pelanggan dalam projek kalkulator berada di `src/test/java/com/microsoft/mcp/sample/client/`

### Ujian Manual
Banyak contoh adalah aplikasi interaktif yang memerlukan ujian manual:

1. Mulakan aplikasi dengan `mvn spring-boot:run`
2. Uji titik akhir atau berinteraksi dengan CLI
3. Sahkan tingkah laku yang dijangkakan sesuai dengan dokumentasi dalam README.md setiap projek

### Ujian dengan Azure AI Foundry
- Log masuk dengan `az login` sebelum menjalankan contoh (pengesahan tanpa kunci)
- Pastikan akaun anda mempunyai peranan Pengguna OpenAI Perkhidmatan Kognitif pada sumber tersebut
- Uji penapisan kandungan dengan contoh AI bertanggungjawab dalam Bab 5

## Garis Panduan Gaya Kod

### Konvensyen Java
- **Versi Java:** Java 21 dengan ciri baharu
- **Gaya:** Ikuti konvensyen Java standard
- **Penamaan:** 
  - Kelas: PascalCase
  - Kaedah/variabel: camelCase
  - Konstanta: UPPER_SNAKE_CASE
  - Nama pakej: huruf kecil

### Corak Spring Boot
- Gunakan `@Service` untuk logik perniagaan
- Gunakan `@RestController` untuk titik akhir REST
- Konfigurasi melalui `application.yml` atau `application.properties`
- Gunakan pembolehubah persekitaran lebih utama daripada nilai terikat keras
- Gunakan anotasi `@Tool` untuk kaedah yang didedahkan MCP

### Susunan Fail
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

### Pergantungan
- Diuruskan melalui Maven `pom.xml`
- BOM Spring AI untuk pengurusan versi
- LangChain4j untuk integrasi AI
- Induk permulaan Spring Boot untuk pergantungan Spring

### Komen Kod
- Tambah JavaDoc untuk API awam
- Sertakan komen penjelasan untuk interaksi AI yang kompleks
- Dokumentasikan penerangan alat MCP dengan jelas

## Pembinaan dan Penyebaran

### Membina Projek

**Bina tanpa ujian:**
```bash
mvn clean install -DskipTests
```

**Bina dengan semua pemeriksaan:**
```bash
mvn clean install
```

**Pakej aplikasi:**
```bash
mvn package
# Membuat JAR dalam direktori target/
```

### Direktori Output
- Kelas terkompilasi: `target/classes/`
- Kelas ujian: `target/test-classes/`
- Fail JAR: `target/*.jar`
- Artifak Maven: `target/`

### Konfigurasi Spesifik Persekitaran

**Pembangunan:**
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

**Pengeluaran:**
- Gunakan identiti terurus sebagai ganti `az login` untuk pengesahan tanpa kunci
- Tetapkan `AZURE_OPENAI_ENDPOINT` pada sumber Foundry pengeluaran anda
- Urus konfigurasi melalui pembolehubah persekitaran atau Azure Key Vault

### Pertimbangan Penyebaran
- Ini adalah repositori pendidikan dengan aplikasi contoh
- Tidak direka untuk penyebaran pengeluaran secara langsung
- Contoh menunjukkan corak untuk diadaptasi untuk penggunaan pengeluaran
- Lihat README projek masing-masing untuk nota penyebaran khusus

## Nota Tambahan

### Azure AI Foundry
- **Pengesahan tanpa kunci:** sambungkan dengan Microsoft Entra ID — tiada kunci API untuk diurus
- **Disediakan sebagai kod:** Bicep + azd (`azd up`) mencipta akaun dan penyebaran model
- Kod yang sama serasi OpenAI berjalan secara tempatan (`az login`) dan dalam Azure (identiti terurus)

### Bekerja dengan Beberapa Projek
Setiap projek contoh berdiri sendiri:
```bash
# Navigasi ke projek tertentu
cd 04-PracticalSamples/[project-name]

# Setiap satunya mempunyai pom.xml sendiri dan boleh dibina secara berdikari
mvn clean install
```

### Isu Umum

**Ketidakpadanan Versi Java:**
```bash
# Sahkan Java 21
java -version
# Kemas kini JAVA_HOME jika perlu
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Isu Muat Turun Pergantungan:**
```bash
# Bersihkan cache Maven dan cuba lagi
rm -rf ~/.m2/repository
mvn clean install
```

**Endpoint atau Log Masuk Tidak Ditemui:**
```bash
# Tetapkan titik akhir dalam sesi semasa dan log masuk (tanpa kunci)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Atau gunakan fail .env dalam direktori projek
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Port Sudah Digunakan:**
```bash
# Spring Boot menggunakan port 8080 secara lalai
# Tukar dalam application.properties:
server.port=8081
```

### Sokongan Pelbagai Bahasa
- Dokumentasi tersedia dalam 45+ bahasa melalui terjemahan automatik
- Terjemahan dalam direktori `translations/`
- Terjemahan diurus oleh aliran kerja GitHub Actions

### Laluan Pembelajaran
1. Mulakan dengan [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. Ikuti bab mengikut turutan (01 → 05)
3. Selesaikan contoh praktikal dalam setiap bab
4. Terokai projek contoh dalam Bab 4
5. Pelajari amalan AI bertanggungjawab dalam Bab 5

### Kontena Pembangunan
`.devcontainer/devcontainer.json` mengkonfigurasi:
- Persekitaran pembangunan Java 21
- Maven telah dipasang terlebih dahulu
- Sambungan Java VS Code
- Alat Spring Boot
- Integrasi GitHub Copilot
- Sokongan Docker dalam Docker
- Azure CLI

### Pertimbangan Prestasi
- Penyebaran Azure AI Foundry mempunyai kuota token/permintaan per minit
- Gunakan saiz kumpulan sesuai untuk penanaman
- Pertimbangkan pengepala untuk panggilan API berulang
- Pantau penggunaan token untuk pengoptimuman kos

### Nota Keselamatan
- Jangan sekali-kali melakukan komit fail `.env` (sudah dalam `.gitignore`)
- Pilih pengesahan tanpa kunci (Microsoft Entra ID) berbanding kunci API
- Gunakan identiti terurus di Azure; `az login` untuk pembangunan tempatan
- Ikuti garis panduan AI bertanggungjawab dalam Bab 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Penafian**:
Dokumen ini telah diterjemahkan menggunakan perkhidmatan terjemahan AI [Co-op Translator](https://github.com/Azure/co-op-translator). Walaupun kami berusaha untuk ketepatan, sila ambil maklum bahawa terjemahan automatik mungkin mengandungi kesilapan atau ketidaktepatan. Dokumen asal dalam bahasa asalnya harus dianggap sebagai sumber yang sahih. Untuk maklumat penting, terjemahan oleh manusia profesional adalah disyorkan. Kami tidak bertanggungjawab terhadap sebarang salah faham atau salah tafsir yang timbul daripada penggunaan terjemahan ini.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->