# Menyiapkan Lingkungan Pengembangan untuk Azure AI Foundry

> Panduan ini menyiapkan model **Azure AI Foundry** untuk aplikasi AI Java dalam kursus ini, menggunakan autentikasi **tanpa kunci** (Microsoft Entra ID) — tidak perlu mengelola kunci API. Baru menggunakan alat ini? Mulai dengan [panduan lingkungan pengembangan](./README.md).

Panduan ini menyiapkan model **Azure AI Foundry** untuk aplikasi AI Java dalam kursus ini. Anda memiliki dua pilihan:

- **Opsi A — Penyediaan dengan `azd` + Bicep (direkomendasikan):** satu perintah untuk menerapkan akun Foundry dan model sebagai kode. Tidak perlu klik-klik di portal.
- **Opsi B — Membuat sumber daya secara manual** di portal Azure AI Foundry.

Kedua pilihan menggunakan **autentikasi tanpa kunci** (Microsoft Entra ID) — tidak ada kunci API yang perlu disalin atau bocor.

## Daftar Isi

- [Apa yang Dibuat](#apa-yang-dibuat)
- [Prasyarat](#prasyarat)
- [Opsi A: Penyediaan dengan azd + Bicep (Direkomendasikan)](#option-a-provision-with-azd--bicep-recommended)
- [Opsi B: Membuat Sumber Daya Secara Manual](#opsi-b-membuat-sumber-daya-secara-manual)
- [Konfigurasikan Lingkungan Anda](#konfigurasikan-lingkungan-anda)
- [Uji Pengaturan Anda](#uji-pengaturan-anda)
- [Apa Selanjutnya?](#apa-selanjutnya)
- [Sumber Daya](#sumber-daya)
- [Sumber Daya Tambahan](#sumber-daya-tambahan)

## Apa yang Dibuat

Template Bicep di [`infra/`](../../../02-SetupDevEnvironment/infra) menyediakan:

- Akun **Azure AI Foundry** (`Microsoft.CognitiveServices/accounts`, jenis `AIServices`) dengan sebuah proyek
- Sebuah deployment **chat** — `gpt-4o-mini`
- Sebuah deployment **embedding** — `text-embedding-3-small` (dipakai di bab-bab berikutnya)
- Sebuah **penugasan peran tanpa kunci** (`Cognitive Services OpenAI User`) sehingga Anda masuk dengan `az login` tanpa perlu mengelola kunci

## Prasyarat

- [Langganan Azure](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) dan [Maven 3.9+](https://maven.apache.org/download.cgi)

## Opsi A: Penyediaan dengan azd + Bicep (Direkomendasikan)

Dari folder `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# Masuk (kedua alat)
azd auth login
az login

# Menyediakan akun Foundry + penyebaran model
azd up
```

`azd` akan meminta **nama lingkungan** (misalnya `genai-java`) dan **wilayah**. Pilih wilayah dimana `gpt-4o-mini` dan `text-embedding-3-small` tersedia — misalnya `eastus2` atau `swedencentral`.

Setelah penyediaan selesai, azd:

1. Menerapkan semua yang didefinisikan di [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep).
2. Menjalankan hook pascapenyediaan yang menulis [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) dengan endpoint dan nama deployment Anda (tanpa rahasia).

> **Tip:** Jalankan ulang `azd up` kapan saja untuk menerapkan perubahan. Jalankan `azd down` untuk menghapus semuanya dan menghentikan biaya.

Untuk melihat pengaturan yang dihasilkan:

```bash
azd env get-values
```

Sekarang lompat ke [Uji Pengaturan Anda](#uji-pengaturan-anda).

## Opsi B: Membuat Sumber Daya Secara Manual

Lebih suka portal? Buat sumber daya secara manual:

1. Buka [portal Azure AI Foundry](https://ai.azure.com/) dan masuk.
2. **Buat proyek** (ini juga membuat sumber daya AI Foundry). Beri nama seperti `GenAIJava`.
3. Dalam proyek Anda, buka **Models + endpoints** → **Deploy model** → **Deploy base model**.
4. Deploy **gpt-4o-mini** (nama deployment `gpt-4o-mini`). Ulangi untuk **text-embedding-3-small** jika Anda ingin contoh embedding.
5. Dari **Overview**, salin **endpoint** (misalnya `https://<resource>.openai.azure.com/`).
6. Beri diri Anda akses tanpa kunci: di sumber daya, buka **Access control (IAM)** → **Add role assignment** → tetapkan **Cognitive Services OpenAI User** ke akun Anda.

> **Masih mengalami kesulitan?** Lihat [dokumentasi Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Konfigurasikan Lingkungan Anda

**Jika Anda menggunakan Opsi A (`azd up`)**, file pengaturan Anda sudah dibuat — tidak perlu konfigurasi lagi. Lewati ke [Uji Pengaturan Anda](#uji-pengaturan-anda).

**Jika Anda menggunakan Opsi B (manual)**, buat file `.env` contoh sendiri:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

Edit `.env` dengan endpoint Anda (tanpa kunci — autentikasi tanpa kunci):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Catatan keamanan:** Tidak ada kunci API yang perlu disimpan. Anda mengautentikasi dengan Microsoft Entra ID melalui `az login` (lokal) atau identitas terkelola (di Azure). File `.env` hanya berisi pengaturan yang tidak rahasia dan sudah dilindungi oleh `.gitignore`.

## Uji Pengaturan Anda

Pastikan Anda sudah masuk supaya autentikasi tanpa kunci bisa mendapatkan token, lalu jalankan contoh:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # jika Anda belum masuk
mvn clean spring-boot:run
```

Anda akan melihat respons dari model `gpt-4o-mini`!

> **Pengguna VS Code:** Tekan `F5` untuk menjalankan. Aplikasi akan memuat `.env` Anda secara otomatis.

> **Contoh lengkap:** Lihat [Contoh Basic Chat dengan Azure AI Foundry](./examples/basic-chat-azure/README.md) untuk detail dan pemecahan masalah.

## Apa Selanjutnya?

**Pengaturan selesai!** Sekarang Anda memiliki:
- Azure AI Foundry dengan `gpt-4o-mini` dan `text-embedding-3-small` yang sudah dideploy
- Autentikasi tanpa kunci (Microsoft Entra ID) — tanpa kunci untuk dikelola
- File `.env` lokal dengan endpoint dan nama deployment Anda
- Lingkungan pengembangan Java yang siap pakai

**Lanjutkan ke** [Bab 3: Teknik Inti Generatif AI](../03-CoreGenerativeAITechniques/README.md) untuk mulai membangun aplikasi AI!

## Sumber Daya

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Autentikasi tanpa kunci dengan Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Dokumentasi Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)
- [Dokumentasi Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Sumber Daya Tambahan

- [Unduh VS Code](https://code.visualstudio.com/Download)
- [Dapatkan Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Konfigurasi Dev Container](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Penafian**:
Dokumen ini telah diterjemahkan menggunakan layanan terjemahan AI [Co-op Translator](https://github.com/Azure/co-op-translator). Meskipun kami berupaya untuk mencapai akurasi, harap diketahui bahwa terjemahan otomatis mungkin mengandung kesalahan atau ketidakakuratan. Dokumen asli dalam bahasa aslinya harus dianggap sebagai sumber yang sah. Untuk informasi penting, disarankan menggunakan terjemahan profesional oleh manusia. Kami tidak bertanggung jawab atas kesalahpahaman atau penafsiran yang keliru yang timbul dari penggunaan terjemahan ini.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->