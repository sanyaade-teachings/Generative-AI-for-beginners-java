# Obrolan Dasar dengan Azure AI Foundry - Contoh End-to-End

Contoh ini adalah aplikasi Spring Boot sederhana yang terhubung ke model **Azure AI Foundry** menggunakan **autentikasi tanpa kunci** (Microsoft Entra ID) dan menguji pengaturan Anda. Ini menggunakan `ChatClient` dari Spring AI.

## Daftar Isi

- [Prasyarat](#prasyarat)
- [Mulai Cepat](#mulai-cepat)
- [Bagaimana Autentikasi Bekerja](#bagaimana-autentikasi-bekerja)
- [Menjalankan Aplikasi](#menjalankan-aplikasi)
  - [Menggunakan Maven](#menggunakan-maven)
  - [Menggunakan VS Code](#menggunakan-vs-code)
  - [Output yang Diharapkan](#output-yang-diharapkan)
- [Referensi Konfigurasi](#referensi-konfigurasi)
  - [Variabel Lingkungan](#variabel-lingkungan)
  - [Konfigurasi Spring](#konfigurasi-spring)
- [Pemecahan Masalah](#pemecahan-masalah)
  - [Masalah Umum](#masalah-umum)
  - [Mode Debug](#mode-debug)
- [Langkah Selanjutnya](#langkah-selanjutnya)
- [Sumber Daya](#sumber-daya)

## Prasyarat

Sebelum menjalankan contoh ini, pastikan Anda memiliki:

- Sumber daya Azure AI Foundry dengan deployment `gpt-4o-mini` — sediakan dengan `azd up` atau secara manual melalui [panduan pengaturan Azure AI Foundry](../../getting-started-azure-openai.md)
- Peran **Cognitive Services OpenAI User** pada sumber daya tersebut (template Bicep sudah menetapkannya untuk Anda)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), sudah masuk menggunakan `az login`
- Java 21+ dan Maven 3.9+

> **Tidak membutuhkan kunci API** — autentikasi tanpa kunci menggunakan Microsoft Entra ID.

## Mulai Cepat

```bash
# 1. Navigasi ke proyek
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Masuk agar otentikasi tanpa kunci dapat memperoleh token
az login

# 3. Konfigurasikan endpoint
#    - Jika Anda menjalankan `azd up`, .env telah dibuat untuk Anda (lewati ini).
#    - Jika tidak, salin template dan atur AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. Jalankan aplikasi
mvn spring-boot:run
```

## Bagaimana Autentikasi Bekerja

Contoh ini melakukan autentikasi dengan **Microsoft Entra ID** — tidak ada kunci API.

Jika hanya `spring.ai.azure.openai.endpoint` yang disetel (dan tanpa api-key), Spring AI membangun klien Azure OpenAI dengan [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). Kredensial tersebut secara otomatis menemukan token dari sesi `az login` lokal Anda, atau dari managed identity saat berjalan di Azure — jadi kode yang sama berfungsi di kedua lokasi tanpa perubahan.

## Menjalankan Aplikasi

### Menggunakan Maven

```bash
mvn spring-boot:run
```

### Menggunakan VS Code

1. Buka proyek di VS Code
2. Tekan `F5` atau gunakan panel "Run and Debug"
3. Pilih konfigurasi "Spring Boot-BasicChatApplication"

> **Catatan**: Konfigurasi VS Code secara otomatis memuat file .env Anda

### Output yang Diharapkan

```
Starting Basic Chat with Azure OpenAI...
Environment variables loaded successfully
Connecting to Azure OpenAI...
Sending prompt: What is AI in a short sentence? Max 100 words.

AI Response:
================
AI, or Artificial Intelligence, is the simulation of human intelligence in machines programmed to think and learn like humans.
================

Success! Azure OpenAI connection is working correctly.
```

## Referensi Konfigurasi

### Variabel Lingkungan

| Variabel | Deskripsi | Wajib | Contoh |
|----------|-----------|-------|--------|
| `AZURE_OPENAI_ENDPOINT` | URL endpoint Foundry (Azure OpenAI) | Ya | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Nama deployment model chat | Tidak | `gpt-4o-mini` (default) |

> Tidak ada variabel kunci API — autentikasi tanpa kunci (Microsoft Entra ID via `az login`).

### Konfigurasi Spring

File `application.yml` mengonfigurasi:
- **Endpoint**: `${AZURE_OPENAI_ENDPOINT}` - Dari variabel lingkungan
- **Deployment**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - Dari variabel lingkungan dengan fallback
- **Auth**: tanpa kunci — tidak ada `api-key` yang disetel, sehingga Spring AI menggunakan `DefaultAzureCredential`
- **Temperature**: `0.7` - Mengontrol kreativitas (0.0 = deterministik, 1.0 = kreatif)
- **Maks Token**: `500` - Panjang maksimal respons

## Pemecahan Masalah

### Masalah Umum

<details>
<summary><strong>Kesalahan: 401 / "PermissionDenied" / error token</strong></summary>

- Jalankan `az login` — autentikasi tanpa kunci membutuhkan sesi masuk aktif untuk mendapatkan token
- Pastikan akun Anda memiliki peran **Cognitive Services OpenAI User** pada sumber daya tersebut
- Jika baru saja menetapkan peran, tunggu beberapa menit agar propagasi selesai
- Pastikan Anda berada di tenant/subscription yang benar (`az account show`)
</details>

<details>
<summary><strong>Kesalahan: "The endpoint is not valid" / kesalahan koneksi</strong></summary>

- Pastikan `AZURE_OPENAI_ENDPOINT` adalah URL dasar lengkap (misalnya, `https://your-resource.openai.azure.com/`)
- Periksa konsistensi tanda slash di akhir URL
- Verifikasi endpoint sesuai dengan sumber daya yang Anda miliki (`azd env get-values`)
</details>

<details>
<summary><strong>Kesalahan: "The deployment was not found"</strong></summary>

- Periksa apakah `AZURE_OPENAI_DEPLOYMENT` sesuai dengan nama deployment di Azure
- Pastikan model sudah berhasil dideploy dan aktif
- Nama deployment default adalah `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: Variabel lingkungan tidak dimuat</strong></summary>

- Pastikan file `.env` berada di direktori root proyek (selevel dengan `pom.xml`)
- Coba jalankan `mvn spring-boot:run` dari terminal terintegrasi VS Code
- Periksa apakah ekstensi Java VS Code sudah terpasang dengan benar
</details>

### Mode Debug

Untuk mengaktifkan logging detail, uncomment baris-baris berikut di `application.yml`:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Langkah Selanjutnya

**Pengaturan Selesai!** Lanjutkan perjalanan pembelajaran Anda:

[Bab 3: Teknik Generative AI Inti](../../../03-CoreGenerativeAITechniques/README.md)

## Sumber Daya

- [Dokumentasi Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Autentikasi tanpa kunci dengan Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Portal Azure AI Foundry](https://ai.azure.com/)
- [Dokumentasi Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Penafian**:
Dokumen ini telah diterjemahkan menggunakan layanan terjemahan AI [Co-op Translator](https://github.com/Azure/co-op-translator). Meskipun kami berupaya untuk mencapai akurasi, harap diketahui bahwa terjemahan otomatis mungkin mengandung kesalahan atau ketidakakuratan. Dokumen asli dalam bahasa aslinya harus dianggap sebagai sumber yang sah. Untuk informasi penting, disarankan menggunakan terjemahan profesional oleh manusia. Kami tidak bertanggung jawab atas kesalahpahaman atau penafsiran yang keliru yang timbul dari penggunaan terjemahan ini.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->