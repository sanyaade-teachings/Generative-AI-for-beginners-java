# Menyediakan Persekitaran Pembangunan untuk Azure AI Foundry

> Panduan ini menyediakan model **Azure AI Foundry** untuk aplikasi AI Java dalam kursus ini, menggunakan pengesahan **tanpa kunci** (Microsoft Entra ID) — tiada kunci API untuk diurus. Baru dengan alat ini? Mulakan dengan [panduan persekitaran pembangunan](./README.md).

Panduan ini menyediakan model **Azure AI Foundry** untuk aplikasi AI Java dalam kursus ini. Anda mempunyai dua pilihan:

- **Pilihan A — Sediakan dengan `azd` + Bicep (disyorkan):** satu arahan untuk menyebarkan akaun Foundry dan model sebagai kod. Tiada klik portal.
- **Pilihan B — Cipta sumber secara manual** dalam portal Azure AI Foundry.

Kedua-dua pilihan menggunakan **pengesahan tanpa kunci** (Microsoft Entra ID) — tiada kunci API untuk disalin atau bocor.

## Jadual Kandungan

- [Apa yang Dicipta](#apa-yang-dicipta)
- [Pra-syarat](#pra-syarat)
- [Pilihan A: Sediakan dengan azd + Bicep (Disyorkan)](#option-a-provision-with-azd--bicep-recommended)
- [Pilihan B: Cipta Sumber Secara Manual](#pilihan-b-cipta-sumber-secara-manual)
- [Konfigurasikan Persekitaran Anda](#konfigurasikan-persekitaran-anda)
- [Uji Penyediaan Anda](#uji-penyediaan-anda)
- [Apa Seterusnya?](#apa-seterusnya)
- [Sumber](#sumber)
- [Sumber Tambahan](#sumber-tambahan)

## Apa yang Dicipta

Templat Bicep dalam [`infra/`](../../../02-SetupDevEnvironment/infra) menyediakan:

- Akaun **Azure AI Foundry** (`Microsoft.CognitiveServices/accounts`, jenis `AIServices`) dengan projek
- Penempatan **chat** — `gpt-4o-mini`
- Penempatan **embedding** — `text-embedding-3-small` (digunakan dalam bab-bab berikut)
- **Pentugasan peranan tanpa kunci** (`Cognitive Services OpenAI User`) supaya anda log masuk dengan `az login` dan bukannya mengurus kunci

## Pra-syarat

- [Langganan Azure](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) dan [Maven 3.9+](https://maven.apache.org/download.cgi)

## Pilihan A: Sediakan dengan azd + Bicep (Disyorkan)

Dari folder `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# Log masuk (kedua-dua alat)
azd auth login
az login

# Sediakan akaun Foundry + penyebaran model
azd up
```

`azd` akan meminta **nama persekitaran** (contohnya `genai-java`) dan **rantaian**. Pilih rantau yang menyediakan `gpt-4o-mini` dan `text-embedding-3-small` — contoh `eastus2` atau `swedencentral`.

Apabila penyediaan selesai, azd:

1. Menyebarkan segala yang ditakrifkan dalam [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep).
2. Menjalankan hook pascapenyediaan yang menulis [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) dengan nama titik hujung dan penempatan anda (tiada rahsia).

> **Petua:** Jalankan semula `azd up` bila-bila masa untuk memohon perubahan. Jalankan `azd down` untuk memadam semua dan berhenti menanggung kos.

Untuk melihat tetapan yang dijana:

```bash
azd env get-values
```

Sekarang teruskan ke [Uji Penyediaan Anda](#uji-penyediaan-anda).

## Pilihan B: Cipta Sumber Secara Manual

Lebih suka portal? Cipta sumber secara manual:

1. Pergi ke [portal Azure AI Foundry](https://ai.azure.com/) dan masuk.
2. **Cipta projek** (ini juga mencipta sumber AI Foundry). Beri nama seperti `GenAIJava`.
3. Dalam projek anda, buka **Models + endpoints** → **Deploy model** → **Deploy base model**.
4. Sebarkan **gpt-4o-mini** (nama penempatan `gpt-4o-mini`). Ulangi untuk **text-embedding-3-small** jika anda mahu contoh embedding.
5. Dari **Overview**, salin **endpoint** (contohnya `https://<resource>.openai.azure.com/`).
6. Beri diri anda akses tanpa kunci: pada sumber, buka **Access control (IAM)** → **Add role assignment** → tetapkan **Cognitive Services OpenAI User** kepada akaun anda.

> **Masih ada masalah?** Lihat [dokumentasi Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Konfigurasikan Persekitaran Anda

**Jika anda menggunakan Pilihan A (`azd up`)**, fail tetapan anda sudah ditulis — tiada yang perlu dikonfigurasikan. Teruskan ke [Uji Penyediaan Anda](#uji-penyediaan-anda).

**Jika anda menggunakan Pilihan B (manual)**, buat fail `.env` contoh sendiri:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

Sunting `.env` dengan titik hujung anda (tiada kunci — pengesahan tanpa kunci):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Nota keselamatan:** Tiada kunci API untuk disimpan. Anda mengesah dengan Microsoft Entra ID melalui `az login` (tempatan) atau identiti terurus (dalam Azure). Fail `.env` hanya menyimpan tetapan bukan rahsia dan sudah dilindungi oleh `.gitignore`.

## Uji Penyediaan Anda

Pastikan anda sudah masuk supaya pengesahan tanpa kunci boleh mendapatkan token, kemudian jalankan contoh:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # jika anda belum masuk
mvn clean spring-boot:run
```

Anda sepatutnya melihat respons dari model `gpt-4o-mini`!

> **Pengguna VS Code:** Tekan `F5` untuk jalankan. Aplikasi memuat `.env` anda secara automatik.

> **Contoh penuh:** Lihat [Contoh Chat Asas dengan Azure AI Foundry](./examples/basic-chat-azure/README.md) untuk butiran dan penyelesaian masalah.

## Apa Seterusnya?

**Penyediaan selesai!** Anda kini ada:
- Azure AI Foundry dengan `gpt-4o-mini` dan `text-embedding-3-small` disebarkan
- Pengesahan tanpa kunci (Microsoft Entra ID) — tiada kunci untuk diurus
- Fail `.env` tempatan dengan titik hujung dan nama penempatan anda
- Persekitaran pembangunan Java sedia digunakan

**Teruskan ke** [Bab 3: Teknik AI Generatif Teras](../03-CoreGenerativeAITechniques/README.md) untuk mula membina aplikasi AI!

## Sumber

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Pengesahan tanpa kunci dengan Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Dokumentasi Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)
- [Dokumentasi Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Sumber Tambahan

- [Muat turun VS Code](https://code.visualstudio.com/Download)
- [Dapatkan Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Konfigurasi Dev Container](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Penafian**:
Dokumen ini telah diterjemahkan menggunakan perkhidmatan terjemahan AI [Co-op Translator](https://github.com/Azure/co-op-translator). Walaupun kami berusaha untuk ketepatan, sila ambil maklum bahawa terjemahan automatik mungkin mengandungi kesilapan atau ketidaktepatan. Dokumen asal dalam bahasa asalnya harus dianggap sebagai sumber yang sahih. Untuk maklumat penting, terjemahan oleh manusia profesional adalah disyorkan. Kami tidak bertanggungjawab terhadap sebarang salah faham atau salah tafsir yang timbul daripada penggunaan terjemahan ini.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->