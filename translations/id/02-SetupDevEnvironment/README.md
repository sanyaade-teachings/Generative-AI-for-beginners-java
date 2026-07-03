# Menyiapkan Lingkungan Pengembangan untuk Generative AI untuk Java

> **Mulai Cepat:** Siapkan model AI Anda di **Azure AI Foundry** sebagai kode dengan Bicep + `azd` dalam beberapa menit — lihat [Panduan Pengaturan Azure AI Foundry](getting-started-azure-openai.md). Autentikasi menggunakan **keyless** (Microsoft Entra ID), jadi tidak ada kunci API yang perlu dikelola.

## Apa yang Akan Anda Pelajari

- Menyiapkan lingkungan pengembangan Java untuk aplikasi AI
- Memilih dan mengonfigurasi lingkungan pengembangan favorit Anda (cloud-first dengan Codespaces, kontainer pengembangan lokal, atau pengaturan lokal penuh)
- Menguji pengaturan Anda dengan menghubungkan ke model Azure AI Foundry

## Daftar Isi

- [Apa yang Akan Anda Pelajari](#apa-yang-akan-anda-pelajari)
- [Pendahuluan](#pendahuluan)
- [Langkah 1: Siapkan Lingkungan Pengembangan Anda](#langkah-1-siapkan-lingkungan-pengembangan-anda)
  - [Pilihan A: GitHub Codespaces (Disarankan)](#pilihan-a-github-codespaces-disarankan)
  - [Pilihan B: Kontainer Dev Lokal](#pilihan-b-kontainer-dev-lokal)
  - [Pilihan C: Gunakan Instalasi Lokal Anda yang Ada](#pilihan-c-gunakan-instalasi-lokal-anda-yang-ada)
- [Langkah 2: Siapkan Azure AI Foundry](#langkah-2-siapkan-azure-ai-foundry)
- [Langkah 3: Uji Pengaturan Anda](#langkah-3-uji-pengaturan-anda)
- [Pemecahan Masalah](#pemecahan-masalah)
- [Ringkasan](#ringkasan)
- [Langkah Berikutnya](#langkah-berikutnya)

## Pendahuluan

Bab ini akan memandu Anda melalui penyiapan lingkungan pengembangan. Kami akan menggunakan **Azure AI Foundry** untuk model-model sepanjang kursus ini. Anda menyiapkan model sebagai kode dengan Bicep dan Azure Developer CLI (`azd`), lalu terhubung dengan **autentikasi keyless** (Microsoft Entra ID) — tanpa kunci API yang perlu disalin atau bocor.

**Tidak perlu pengaturan lokal!** Anda dapat menggunakan GitHub Codespaces, yang menyediakan lingkungan pengembangan lengkap di browser Anda, dan menyiapkan Foundry dari sana.

Kami menggunakan **Azure AI Foundry** untuk kursus ini karena:
- **Disiapkan sebagai kode** — satu `azd up` menerapkan akun dan penempatan model
- **Keyless** — autentikasi dengan sign-in Azure Anda atau identitas terkelola
- **Siap produksi** — kode yang sama dijalankan secara lokal dan di Azure
- **Fleksibel** — ganti model dengan mengubah nama penempatan, bukan kode Anda

> **Catatan**: Penempatan Azure AI Foundry dikenai biaya per token (bayar sesuai pemakaian). Lihat [panduan pengaturan Azure AI Foundry](getting-started-azure-openai.md) untuk detail penyiapan, wilayah, dan biaya.


## Langkah 1: Siapkan Lingkungan Pengembangan Anda

<a name="quick-start-cloud"></a>

Kami telah membuat kontainer pengembangan yang telah dikonfigurasi sebelumnya untuk meminimalkan waktu pengaturan dan memastikan Anda memiliki semua alat yang diperlukan untuk kursus Generative AI untuk Java ini. Pilih pendekatan pengembangan yang Anda sukai:

### Pilihan Pengaturan Lingkungan:

#### Pilihan A: GitHub Codespaces (Disarankan)

**Mulai coding dalam 2 menit - tanpa pengaturan lokal!**

1. Fork repositori ini ke akun GitHub Anda  
   > **Catatan**: Jika Anda ingin mengedit konfigurasi dasar, silakan lihat [Konfigurasi Kontainer Dev](../../../.devcontainer/devcontainer.json)
2. Klik **Code** → tab **Codespaces** → **...** → **New with options...**
3. Gunakan default – ini akan memilih **Konfigurasi Kontainer Dev**: **Lingkungan Pengembangan Generative AI Java** kontainer dev khusus yang dibuat untuk kursus ini
4. Klik **Create codespace**
5. Tunggu sekitar ~2 menit hingga lingkungan siap
6. Lanjut ke [Langkah 2: Siapkan Azure AI Foundry](#langkah-2-siapkan-azure-ai-foundry)

<img src="../../../translated_images/id/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: Codespaces submenu" width="50%">

<img src="../../../translated_images/id/image.833552b62eee7766.webp" alt="Screenshot: New with options" width="50%">

<img src="../../../translated_images/id/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: Create codespace options" width="50%">


> **Manfaat Codespaces**:
> - Tidak perlu instalasi lokal
> - Berfungsi di perangkat apa pun dengan browser
> - Sudah dikonfigurasi dengan semua alat dan dependensi
> - Gratis 60 jam per bulan untuk akun pribadi
> - Lingkungan konsisten untuk semua peserta

#### Pilihan B: Kontainer Dev Lokal

**Untuk pengembang yang lebih suka pengembangan lokal dengan Docker**

1. Fork dan clone repositori ini ke mesin lokal Anda  
   > **Catatan**: Jika Anda ingin mengedit konfigurasi dasar, silakan lihat [Konfigurasi Kontainer Dev](../../../.devcontainer/devcontainer.json)
2. Instal [Docker Desktop](https://www.docker.com/products/docker-desktop/) dan [VS Code](https://code.visualstudio.com/)
3. Instal ekstensi [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) di VS Code
4. Buka folder repositori di VS Code
5. Saat diminta, klik **Reopen in Container** (atau gunakan `Ctrl+Shift+P` → "Dev Containers: Reopen in Container")
6. Tunggu sampai kontainer dibangun dan berjalan
7. Lanjut ke [Langkah 2: Siapkan Azure AI Foundry](#langkah-2-siapkan-azure-ai-foundry)

<img src="../../../translated_images/id/devcontainer.21126c9d6de64494.webp" alt="Screenshot: Dev container setup" width="50%">

<img src="../../../translated_images/id/image-3.bf93d533bbc84268.webp" alt="Screenshot: Dev container build complete" width="50%">

#### Pilihan C: Gunakan Instalasi Lokal Anda yang Ada

**Untuk pengembang dengan lingkungan Java yang sudah ada**

Prasyarat:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html)  
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) atau IDE favorit Anda

Langkah:
1. Clone repositori ini ke mesin lokal Anda
2. Buka proyek di IDE Anda
3. Lanjut ke [Langkah 2: Siapkan Azure AI Foundry](#langkah-2-siapkan-azure-ai-foundry)

> **Tips Pro**: Jika Anda memiliki mesin dengan spesifikasi rendah tapi ingin menggunakan VS Code secara lokal, gunakan GitHub Codespaces! Anda bisa menghubungkan VS Code lokal Anda ke Codespace yang dihosting di cloud untuk mendapatkan kombinasi terbaik.

<img src="../../../translated_images/id/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: created local devcontainer instance" width="50%">


## Langkah 2: Siapkan Azure AI Foundry

Terapkan model AI kursus ini ke Azure AI Foundry sebagai kode. Dari root repositori:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```
  
`azd` akan meminta nama lingkungan dan wilayah, menyiapkan akun Azure AI Foundry dengan penempatan `gpt-4o-mini` dan `text-embedding-3-small`, dan menulis endpoint ke `.env` contoh — semua dengan autentikasi **keyless** (tanpa kunci API).

> **Panduan lengkap:** Lihat [Panduan Pengaturan Azure AI Foundry](getting-started-azure-openai.md) untuk prasyarat, alternatif manual (portal), panduan wilayah, serta catatan biaya/pembersihan.

## Langkah 3: Uji Pengaturan Anda

Setelah model Foundry Anda disiapkan, uji koneksi dengan aplikasi contoh di [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure).

1. Buka terminal di lingkungan pengembangan Anda.  
2. Masuk ke contoh:  
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
   
3. Pastikan Anda sudah masuk (autentikasi keyless memerlukan token):  
   ```bash
   az login
   ```
  
   > Jika Anda menjalankan `azd up`, file `.env` dengan endpoint Anda sudah dibuat otomatis.  
4. Jalankan aplikasi:  
   ```bash
   mvn clean spring-boot:run
   ```
  
Anda seharusnya melihat respons dari model `gpt-4o-mini`.

### Memahami Kode Contoh

Contoh di bawah `examples/basic-chat-azure` adalah aplikasi Spring Boot yang menggunakan **Spring AI** untuk terhubung ke Azure AI Foundry dengan autentikasi keyless.

**Apa yang dilakukan kode ini:**  
- **Terhubung** ke Azure AI Foundry menggunakan sign-in Azure Anda (Microsoft Entra ID) — tanpa kunci API  
- **Mengirim** prompt ke model `gpt-4o-mini`  
- **Menerima** dan menampilkan respons AI  
- **Memvalidasi** bahwa pengaturan Anda bekerja dengan benar

**Dependensi Utama** (di `pom.xml`):  
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```
  
**Konfigurasi** (`application.yml`):  
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
  
## Ringkasan

Bagus! Sekarang Anda sudah menyiapkan semuanya:

- Model Azure AI Foundry sudah disiapkan sebagai kode dengan Bicep + `azd`
- Lingkungan pengembangan Java Anda berjalan (baik itu Codespaces, kontainer dev, atau lokal)
- Terhubung ke Azure AI Foundry dengan autentikasi keyless (Microsoft Entra ID) — tanpa kunci API
- Menguji semuanya berfungsi dengan contoh sederhana yang berinteraksi dengan model Anda

## Langkah Berikutnya

[Bab 3: Teknik Core Generative AI](../03-CoreGenerativeAITechniques/README.md)

## Pemecahan Masalah

Mengalami masalah? Berikut masalah umum dan solusinya:

- **Autentikasi gagal (401/403)?**  
  - Jalankan `az login` — autentikasi keyless, jadi Anda harus sudah masuk  
  - Pastikan akun Anda memiliki peran **Cognitive Services OpenAI User** pada resource  
  - Jika baru saja menyiapkan, tunggu sebentar agar penugasan peran diterapkan

- **Maven tidak ditemukan?**  
  - Jika menggunakan kontainer dev/Codespaces, Maven sudah terpasang  
  - Untuk pengaturan lokal, pastikan Java 21+ dan Maven 3.9+ sudah terinstal  
  - Coba `mvn --version` untuk memeriksa instalasi

- **`azd` tidak ditemukan atau penyiapan gagal?**  
  - Pasang [Azure Developer CLI](https://aka.ms/azure-dev/install) dan jalankan `azd auth login`  
  - Pilih wilayah yang mendukung `gpt-4o-mini` (misal `eastus2`)  
  - Lihat [panduan pengaturan Azure AI Foundry](getting-started-azure-openai.md) untuk detailnya

- **Kontainer dev tidak berjalan?**  
  - Pastikan Docker Desktop berjalan (untuk pengembangan lokal)  
  - Coba bangun ulang kontainer: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **Kesalahan kompilasi aplikasi?**  
  - Pastikan Anda berada di direktori yang benar: `02-SetupDevEnvironment/examples/basic-chat-azure`  
  - Coba bersihkan dan bangun ulang: `mvn clean compile`

> **Butuh bantuan?**: Masih ada masalah? Buat isu di repositori dan kami akan membantu Anda.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Penafian**:
Dokumen ini telah diterjemahkan menggunakan layanan terjemahan AI [Co-op Translator](https://github.com/Azure/co-op-translator). Meskipun kami berupaya untuk mencapai akurasi, harap diketahui bahwa terjemahan otomatis mungkin mengandung kesalahan atau ketidakakuratan. Dokumen asli dalam bahasa aslinya harus dianggap sebagai sumber yang sah. Untuk informasi penting, disarankan menggunakan terjemahan profesional oleh manusia. Kami tidak bertanggung jawab atas kesalahpahaman atau penafsiran yang keliru yang timbul dari penggunaan terjemahan ini.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->