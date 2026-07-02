# Perbualan Asas dengan Azure AI Foundry - Contoh End-to-End

Contoh ini adalah aplikasi Spring Boot mudah yang berhubung dengan model **Azure AI Foundry** menggunakan **pengesahan tanpa kunci** (Microsoft Entra ID) dan menguji tetapan anda. Ia menggunakan `ChatClient` dari Spring AI.

## Jadual Kandungan

- [Prasyarat](#prasyarat)
- [Mula Pantas](#mula-pantas)
- [Bagaimana Pengesahan Berfungsi](#bagaimana-pengesahan-berfungsi)
- [Menjalankan Aplikasi](#menjalankan-aplikasi)
  - [Menggunakan Maven](#menggunakan-maven)
  - [Menggunakan VS Code](#menggunakan-vs-code)
  - [Output Dijangka](#output-dijangka)
- [Rujukan Konfigurasi](#rujukan-konfigurasi)
  - [Pembolehubah Persekitaran](#pembolehubah-persekitaran)
  - [Konfigurasi Spring](#konfigurasi-spring)
- [Penyelesaian Masalah](#penyelesaian-masalah)
  - [Isu Biasa](#isu-biasa)
  - [Mod Debug](#mod-debug)
- [Langkah Seterusnya](#langkah-seterusnya)
- [Sumber](#sumber)

## Prasyarat

Sebelum menjalankan contoh ini, pastikan anda mempunyai:

- Sumber Azure AI Foundry dengan pelaksanaan `gpt-4o-mini` — sediakan dengan `azd up` atau secara manual melalui [panduan penyediaan Azure AI Foundry](../../getting-started-azure-openai.md)
- Peranan **Cognitive Services OpenAI User** pada sumber tersebut (templat Bicep menetapkan ini untuk anda)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), sudah log masuk dengan `az login`
- Java 21+ dan Maven 3.9+

> **Tiada kunci API diperlukan** — pengesahan adalah tanpa kunci melalui Microsoft Entra ID.

## Mula Pantas

```bash
# 1. Navigasi ke projek
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Log masuk supaya pengesahan tanpa kunci boleh mendapatkan token
az login

# 3. Konfigurasikan titik akhir
#    - Jika anda menjalankan `azd up`, .env telah ditulis untuk anda (langkau ini).
#    - Jika tidak, salin templat dan tetapkan AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. Jalankan aplikasi
mvn spring-boot:run
```

## Bagaimana Pengesahan Berfungsi

Contoh ini mengesahkan dengan **Microsoft Entra ID** — tiada kunci API digunakan.

Apabila hanya `spring.ai.azure.openai.endpoint` ditetapkan (dan tiada api-key), Spring AI membina klien Azure OpenAI dengan [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). Kredensial itu secara automatik mencari token daripada sesi `az login` anda secara tempatan, atau daripada pengenalan terurus apabila dijalankan di Azure — jadi kod yang sama berfungsi di kedua-dua tempat tanpa perubahan.

## Menjalankan Aplikasi

### Menggunakan Maven

```bash
mvn spring-boot:run
```

### Menggunakan VS Code

1. Buka projek dalam VS Code
2. Tekan `F5` atau gunakan panel "Run and Debug"
3. Pilih konfigurasi "Spring Boot-BasicChatApplication"

> **Nota**: Konfigurasi VS Code memuatkan fail .env anda secara automatik

### Output Dijangka

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

## Rujukan Konfigurasi

### Pembolehubah Persekitaran

| Pembolehubah | Penerangan | Wajib | Contoh |
|--------------|------------|--------|--------|
| `AZURE_OPENAI_ENDPOINT` | URL titik hujung Foundry (Azure OpenAI) | Ya | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Nama pelaksanaan model perbualan | Tidak | `gpt-4o-mini` (lalai) |

> Tiada pembolehubah kunci API — pengesahan adalah tanpa kunci (Microsoft Entra ID melalui `az login`).

### Konfigurasi Spring

Fail `application.yml` mengkonfigurasi:
- **Titik Hujung**: `${AZURE_OPENAI_ENDPOINT}` - Dari pembolehubah persekitaran
- **Pelaksanaan**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - Dari pembolehubah persekitaran dengan sandaran
- **Pengesahan**: tanpa kunci — tiada `api-key` ditetapkan, jadi Spring AI menggunakan `DefaultAzureCredential`
- **Suhu**: `0.7` - Mengawal kreativiti (0.0 = deterministik, 1.0 = kreatif)
- **Max Token**: `500` - Panjang maksimum respons

## Penyelesaian Masalah

### Isu Biasa

<details>
<summary><strong>Ralat: 401 / "PermissionDenied" / ralat token</strong></summary>

- Jalankan `az login` — pengesahan tanpa kunci memerlukan log masuk aktif untuk mendapatkan token
- Sahkan akaun anda mempunyai peranan **Cognitive Services OpenAI User** pada sumber tersebut
- Jika anda baru sahaja melantik peranan itu, tunggu beberapa minit untuk ia disebarkan
- Pastikan anda berada dalam penyewa/langganan yang betul (`az account show`)
</details>

<details>
<summary><strong>Ralat: "The endpoint is not valid" / ralat sambungan</strong></summary>

- Pastikan `AZURE_OPENAI_ENDPOINT` adalah URL asas penuh (contoh, `https://your-resource.openai.azure.com/`)
- Semak konsistensi garis miring akhir
- Sahkan titik hujung sepadan dengan sumber yang disediakan (`azd env get-values`)
</details>

<details>
<summary><strong>Ralat: "The deployment was not found"</strong></summary>

- Sahkan `AZURE_OPENAI_DEPLOYMENT` sepadan dengan nama pelaksanaan dalam Azure
- Periksa model telah berjaya dilaksanakan dan aktif
- Nama pelaksanaan lalai ialah `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: Pembolehubah persekitaran tidak dimuat</strong></summary>

- Pastikan fail `.env` anda berada di direktori root projek (paras yang sama dengan `pom.xml`)
- Cuba jalankan `mvn spring-boot:run` dalam terminal terbenam VS Code
- Semak sambungan Java VS Code telah dipasang dengan betul
</details>

### Mod Debug

Untuk menghidupkan log terperinci, nyahkomen baris berikut dalam `application.yml`:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Langkah Seterusnya

**Persediaan Lengkap!** Teruskan perjalanan pembelajaran anda:

[Bab 3: Teknik Teras AI Generatif](../../../03-CoreGenerativeAITechniques/README.md)

## Sumber

- [Dokumentasi Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Pengesahan tanpa kunci dengan Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Portal Azure AI Foundry](https://ai.azure.com/)
- [Dokumentasi Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Penafian**:
Dokumen ini telah diterjemahkan menggunakan perkhidmatan terjemahan AI [Co-op Translator](https://github.com/Azure/co-op-translator). Walaupun kami berusaha untuk ketepatan, sila ambil maklum bahawa terjemahan automatik mungkin mengandungi kesilapan atau ketidaktepatan. Dokumen asal dalam bahasa asalnya harus dianggap sebagai sumber yang sahih. Untuk maklumat penting, terjemahan oleh manusia profesional adalah disyorkan. Kami tidak bertanggungjawab terhadap sebarang salah faham atau salah tafsir yang timbul daripada penggunaan terjemahan ini.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->