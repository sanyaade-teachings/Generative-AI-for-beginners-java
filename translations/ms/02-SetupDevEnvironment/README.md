# Menyiapkan Persekitaran Pembangunan untuk Generative AI bagi Java

> **Mula Cepat:** Sediakan model AI anda di **Azure AI Foundry** sebagai kod dengan Bicep + `azd` dalam beberapa minit — lihat [Panduan Persediaan Azure AI Foundry](getting-started-azure-openai.md). Pengesahan adalah **tanpa kekunci** (Microsoft Entra ID), jadi tiada kunci API untuk diuruskan.

## Apa yang Anda Akan Pelajari

- Menyiapkan persekitaran pembangunan Java untuk aplikasi AI
- Memilih dan mengkonfigurasi persekitaran pembangunan pilihan anda (awan dahulu dengan Codespaces, bekas dev tempatan, atau tetapan penuh tempatan)
- Uji tetapan anda dengan menyambung kepada model Azure AI Foundry

## Jadual Kandungan

- [Apa yang Anda Akan Pelajari](#apa-yang-anda-akan-pelajari)
- [Pengenalan](#pengenalan)
- [Langkah 1: Menyiapkan Persekitaran Pembangunan Anda](#langkah-1-menyiapkan-persekitaran-pembangunan-anda)
  - [Pilihan A: GitHub Codespaces (Disyorkan)](#pilihan-a-github-codespaces-disyorkan)
  - [Pilihan B: Bekas Dev Tempatan](#pilihan-b-bekas-dev-tempatan)
  - [Pilihan C: Gunakan Pemasangan Tempatan Sedia Ada Anda](#pilihan-c-gunakan-pemasangan-tempatan-sedia-ada-anda)
- [Langkah 2: Sediakan Azure AI Foundry](#langkah-2-sediakan-azure-ai-foundry)
- [Langkah 3: Uji Tetapan Anda](#langkah-3-uji-tetapan-anda)
- [Penyelesaian Masalah](#penyelesaian-masalah)
- [Ringkasan](#ringkasan)
- [Langkah Seterusnya](#langkah-seterusnya)

## Pengenalan

Bab ini akan memandu anda melalui penyediaan persekitaran pembangunan. Kita akan menggunakan **Azure AI Foundry** untuk model sepanjang kursus ini. Anda menyediakan model sebagai kod dengan Bicep dan Azure Developer CLI (`azd`), kemudian sambung dengan **pengesahan tanpa kekunci** (Microsoft Entra ID) — tiada kunci API untuk disalin atau bocor.

**Tiada tetapan tempatan diperlukan!** Anda boleh gunakan GitHub Codespaces, yang menyediakan persekitaran pembangunan penuh dalam pelayar anda, dan menyediakan Foundry dari situ.

Kami menggunakan **Azure AI Foundry** untuk kursus ini kerana ia:
- **Disediakan sebagai kod** — satu `azd up` menyebarkan akaun dan penyebaran model
- **Tanpa kekunci** — sahkan dengan log masuk Azure anda atau identiti pengurus
- **Sedia untuk produksi** — kod yang sama berjalan secara tempatan dan di Azure
- **Fleksibel** — tukar model dengan mengubah nama penyebaran, bukan kod anda

> **Nota**: Penyebaran Azure AI Foundry dikenakan bayaran mengikut token (bayar ikut penggunaan). Lihat [panduan persediaan Azure AI Foundry](getting-started-azure-openai.md) untuk butiran penyediaan, rantau, dan kos.


## Langkah 1: Menyiapkan Persekitaran Pembangunan Anda

<a name="quick-start-cloud"></a>

Kami telah mencipta bekas pembangunan yang telah dipra-konfigurasi untuk meminimumkan masa penyediaan dan memastikan anda mempunyai semua alat yang diperlukan untuk kursus Generative AI bagi Java ini. Pilih pendekatan pembangunan pilihan anda:

### Pilihan Penyediaan Persekitaran:

#### Pilihan A: GitHub Codespaces (Disyorkan)

**Mula menulis kod dalam 2 minit - tiada tetapan tempatan diperlukan!**

1. Fork repositori ini ke akaun GitHub anda
   > **Nota**: Jika anda ingin mengubah config asas sila lihat [Dev Container Configuration](../../../.devcontainer/devcontainer.json)
2. Klik **Code** → tab **Codespaces** → **...** → **New with options...**
3. Gunakan lalai – ini akan memilih **Dev container configuration**: **Persekitaran Pembangunan Generative AI Java** custom devcontainer yang dibuat untuk kursus ini
4. Klik **Create codespace**
5. Tunggu kira-kira 2 minit sehingga persekitaran sedia
6. Teruskan ke [Langkah 2: Sediakan Azure AI Foundry](#langkah-2-sediakan-azure-ai-foundry)

<img src="../../../translated_images/ms/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: Codespaces submenu" width="50%">

<img src="../../../translated_images/ms/image.833552b62eee7766.webp" alt="Screenshot: New with options" width="50%">

<img src="../../../translated_images/ms/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: Create codespace options" width="50%">


> **Faedah Codespaces**:
> - Tiada pemasangan tempatan diperlukan
> - Berfungsi pada mana-mana peranti dengan pelayar
> - Dipra-konfigurasi dengan semua alat dan kebergantungan
> - 60 jam percuma sebulan untuk akaun peribadi
> - Persekitaran konsisten untuk semua pelajar

#### Pilihan B: Bekas Dev Tempatan

**Untuk pembangun yang lebih suka pembangunan tempatan dengan Docker**

1. Fork dan klon repositori ini ke mesin tempatan anda
   > **Nota**: Jika anda ingin mengubah config asas sila lihat [Dev Container Configuration](../../../.devcontainer/devcontainer.json)
2. Pasang [Docker Desktop](https://www.docker.com/products/docker-desktop/) dan [VS Code](https://code.visualstudio.com/)
3. Pasang [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) dalam VS Code
4. Buka folder repositori dalam VS Code
5. Apabila diminta, klik **Reopen in Container** (atau gunakan `Ctrl+Shift+P` → "Dev Containers: Reopen in Container")
6. Tunggu bekas dibina dan dimulakan
7. Teruskan ke [Langkah 2: Sediakan Azure AI Foundry](#langkah-2-sediakan-azure-ai-foundry)

<img src="../../../translated_images/ms/devcontainer.21126c9d6de64494.webp" alt="Screenshot: Dev container setup" width="50%">

<img src="../../../translated_images/ms/image-3.bf93d533bbc84268.webp" alt="Screenshot: Dev container build complete" width="50%">

#### Pilihan C: Gunakan Pemasangan Tempatan Sedia Ada Anda

**Untuk pembangun dengan persekitaran Java sedia ada**

Prasyarat:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html) 
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) atau IDE pilihan anda

Langkah:
1. Klon repositori ini ke mesin tempatan anda
2. Buka projek dalam IDE anda
3. Teruskan ke [Langkah 2: Sediakan Azure AI Foundry](#langkah-2-sediakan-azure-ai-foundry)

> **Petua Pro**: Jika anda mempunyai mesin berspesifikasi rendah tetapi mahu VS Code secara tempatan, gunakan GitHub Codespaces! Anda boleh sambungkan VS Code tempatan anda ke Codespace yang dihoskan di awan untuk yang terbaik dari kedua-dua dunia.

<img src="../../../translated_images/ms/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: created local devcontainer instance" width="50%">


## Langkah 2: Sediakan Azure AI Foundry

Sebarkan model AI kursus ke Azure AI Foundry sebagai kod. Dari akar repositori:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```

`azd` akan meminta nama persekitaran dan rantau, menyediakan akaun Azure AI Foundry dengan penyebaran `gpt-4o-mini` dan `text-embedding-3-small`, dan menulis titik hujung dalam `.env` contoh — semua dengan pengesahan **tanpa kekunci** (tiada kunci API).

> **Panduan penuh:** Lihat [Panduan Persediaan Azure AI Foundry](getting-started-azure-openai.md) untuk prasyarat, alternatif manual (portal), panduan rantau, dan nota kos/pembersihan.

## Langkah 3: Uji Tetapan Anda

Setelah model Foundry anda disediakan, uji sambungan dengan aplikasi contoh dalam [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure).

1. Buka terminal dalam persekitaran pembangunan anda.
2. Navigasi ke contoh:
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
3. Pastikan anda sudah log masuk (pengesahan tanpa kekunci memerlukan token):
   ```bash
   az login
   ```
   > Jika anda menjalankan `azd up`, fail `.env` dengan titik hujung anda sudah ditulis untuk anda.
4. Jalankan aplikasi:
   ```bash
   mvn clean spring-boot:run
   ```

Anda harus melihat respons daripada model `gpt-4o-mini`.

### Memahami Kod Contoh

Contoh di bawah `examples/basic-chat-azure` adalah aplikasi Spring Boot yang menggunakan **Spring AI** untuk menyambung ke Azure AI Foundry dengan pengesahan tanpa kekunci.

**Apa yang dilakukan kod ini:**
- **Menyambung** ke Azure AI Foundry menggunakan log masuk Azure anda (Microsoft Entra ID) — tanpa kunci API
- **Menghantar** prompt kepada model `gpt-4o-mini`
- **Menerima** dan memaparkan respons AI
- **Mengesahkan** tetapan anda berfungsi dengan betul

**Kebergantungan Utama** (dalam `pom.xml`):
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

Hebat! Anda kini mempunyai semua yang diperlukan:

- Model Azure AI Foundry disediakan sebagai kod dengan Bicep + `azd`
- Persekitaran pembangunan Java anda berjalan (sama ada Codespaces, bekas dev, atau tempatan)
- Disambung ke Azure AI Foundry dengan pengesahan tanpa kekunci (Microsoft Entra ID) — tiada kunci API
- Uji semuanya berfungsi dengan contoh mudah yang berbual dengan model anda

## Langkah Seterusnya

[Bab 3: Teknik Teras Generative AI](../03-CoreGenerativeAITechniques/README.md)

## Penyelesaian Masalah

Ada masalah? Berikut adalah masalah biasa dan penyelesaiannya:

- **Pengesahan gagal (401/403)?** 
  - Jalankan `az login` — pengesahan adalah tanpa kekunci, jadi anda mesti log masuk
  - Sahkan akaun anda mempunyai peranan **Cognitive Services OpenAI User** pada sumber
  - Jika baru sahaja menyediakan, tunggu sebentar untuk penganugerahan peranan disebarkan

- **Maven tidak dijumpai?** 
  - Jika menggunakan bekas dev/Codespaces, Maven sudah dipasang terlebih dahulu
  - Untuk tetapan tempatan, pastikan Java 21+ dan Maven 3.9+ dipasang
  - Cuba `mvn --version` untuk sahkan pemasangan

- **`azd` tidak ditemui atau penyediaan gagal?** 
  - Pasang [Azure Developer CLI](https://aka.ms/azure-dev/install) dan jalankan `azd auth login`
  - Pilih rantau di mana `gpt-4o-mini` tersedia (contoh `eastus2`)
  - Lihat [panduan persediaan Azure AI Foundry](getting-started-azure-openai.md) untuk butiran

- **Bekas dev tidak mula?** 
  - Pastikan Docker Desktop sedang berjalan (untuk pembangunan tempatan)
  - Cuba bina semula bekas: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **Ralat kompilasi aplikasi?**
  - Pastikan anda berada dalam direktori betul: `02-SetupDevEnvironment/examples/basic-chat-azure`
  - Cuba bersihkan dan bina semula: `mvn clean compile`

> **Perlu bantuan?**: Masih ada masalah? Buka isu dalam repositori dan kami akan membantu anda.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Penafian**:
Dokumen ini telah diterjemahkan menggunakan perkhidmatan terjemahan AI [Co-op Translator](https://github.com/Azure/co-op-translator). Walaupun kami berusaha untuk ketepatan, sila ambil maklum bahawa terjemahan automatik mungkin mengandungi kesilapan atau ketidaktepatan. Dokumen asal dalam bahasa asalnya harus dianggap sebagai sumber yang sahih. Untuk maklumat penting, terjemahan oleh manusia profesional adalah disyorkan. Kami tidak bertanggungjawab terhadap sebarang salah faham atau salah tafsir yang timbul daripada penggunaan terjemahan ini.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->