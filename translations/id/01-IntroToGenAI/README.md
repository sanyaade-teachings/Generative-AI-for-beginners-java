# Pendahuluan ke Generative AI - Edisi Java

## Apa yang Akan Anda Pelajari

- **Dasar-dasar Generative AI** termasuk LLM, rekayasa prompt, token, embeddings, dan basis data vektor
- **Membandingkan alat pengembangan AI Java** termasuk Azure OpenAI SDK, Spring AI, dan OpenAI Java SDK
- **Mengenal Model Context Protocol** dan perannya dalam komunikasi agen AI

## Daftar Isi

- [Pendahuluan](#pendahuluan)
- [Penyegaran cepat tentang konsep Generative AI](#penyegaran-cepat-tentang-konsep-generative-ai)
- [Ulasan rekayasa prompt](#ulasan-rekayasa-prompt)
- [Token, embeddings, dan agen](#token-embeddings-dan-agen)
- [Alat dan Perpustakaan Pengembangan AI untuk Java](#alat-dan-perpustakaan-pengembangan-ai-untuk-java)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [Ringkasan](#ringkasan)
- [Langkah Selanjutnya](#langkah-selanjutnya)

## Pendahuluan

Selamat datang di bab pertama Generative AI untuk Pemula - Edisi Java! Pelajaran dasar ini memperkenalkan Anda pada konsep inti generative AI dan cara bekerja dengannya menggunakan Java. Anda akan mempelajari blok bangunan esensial aplikasi AI, termasuk Large Language Models (LLM), token, embeddings, dan agen AI. Kami juga akan mengeksplorasi alat utama Java yang akan Anda gunakan sepanjang kursus ini.

### Penyegaran cepat tentang konsep Generative AI

Generative AI adalah jenis kecerdasan buatan yang menciptakan konten baru, seperti teks, gambar, atau kode, berdasarkan pola dan hubungan yang dipelajari dari data. Model Generative AI dapat menghasilkan respons seperti manusia, memahami konteks, dan terkadang bahkan menciptakan konten yang tampak menyerupai manusia.

Saat Anda mengembangkan aplikasi AI Java Anda, Anda akan bekerja dengan **model generative AI** untuk membuat konten. Beberapa kemampuan model generative AI meliputi:

- **Pembuatan Teks**: Membuat teks seperti manusia untuk chatbot, konten, dan penyelesaian teks.
- **Pembuatan dan Analisis Gambar**: Menghasilkan gambar realistis, meningkatkan foto, dan mendeteksi objek.
- **Pembuatan Kode**: Menulis potongan kode atau skrip.

Ada jenis model tertentu yang dioptimalkan untuk tugas berbeda. Misalnya, baik **Small Language Models (SLM)** maupun **Large Language Models (LLM)** dapat menangani pembuatan teks, dengan LLM biasanya menawarkan performa lebih baik untuk tugas kompleks. Untuk tugas yang terkait dengan gambar, Anda akan menggunakan model visi khusus atau model multi-modal.

![Figure: Generative AI model types and use cases.](../../../translated_images/id/llms.225ca2b8a0d34473.webp)

Tentu saja, respons dari model ini tidak selalu sempurna. Anda mungkin pernah mendengar tentang model yang "halusinasi" atau menghasilkan informasi yang salah dengan cara yang meyakinkan. Namun Anda dapat membantu membimbing model untuk menghasilkan respons yang lebih baik dengan memberikan instruksi dan konteks yang jelas. Di sinilah **rekayasa prompt** berperan.

#### Ulasan rekayasa prompt

Rekayasa prompt adalah praktik merancang input efektif untuk membimbing model AI menuju keluaran yang diinginkan. Ini melibatkan:

- **Kejelasan**: Membuat instruksi jelas dan tidak ambigu.
- **Konteks**: Memberikan informasi latar belakang yang diperlukan.
- **Keterbatasan**: Menentukan batasan atau format apapun.

Beberapa praktik terbaik rekayasa prompt termasuk desain prompt, instruksi jelas, pemecahan tugas, one-shot dan few-shot learning, serta penyetelan prompt. Menguji berbagai prompt sangat penting untuk menemukan yang paling cocok untuk kasus penggunaan spesifik Anda.

Saat mengembangkan aplikasi, Anda akan bekerja dengan jenis prompt yang berbeda:
- **Prompt sistem**: Menetapkan aturan dasar dan konteks perilaku model
- **Prompt pengguna**: Data input dari pengguna aplikasi Anda
- **Prompt asisten**: Respons model berdasarkan prompt sistem dan pengguna

> **Pelajari lebih lanjut**: Pelajari lebih dalam tentang rekayasa prompt di [bab Prompt Engineering dalam kursus GenAI for Beginners](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### Token, embeddings, dan agen

Saat bekerja dengan model generative AI, Anda akan menemui istilah seperti **token**, **embeddings**, **agen**, dan **Model Context Protocol (MCP)**. Berikut penjelasan rinci tentang konsep-konsep ini:

- **Token**: Token adalah unit terkecil dari teks dalam sebuah model. Mereka bisa berupa kata, karakter, atau sub-kata. Token digunakan untuk merepresentasikan data teks dalam format yang dapat dipahami model. Contohnya, kalimat "The quick brown fox jumped over the lazy dog" mungkin di-tokenisasi sebagai ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] atau ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"] tergantung strategi tokenisasi.

![Figure: Generative AI tokens example of breaking words into tokens](../../../translated_images/id/tokens.6283ed277a2ffff4.webp)

Tokenisasi adalah proses memecah teks menjadi unit-unit kecil ini. Ini penting karena model beroperasi pada token, bukan teks mentah. Jumlah token dalam sebuah prompt memengaruhi panjang dan kualitas respons model, karena model memiliki batas token pada jendela konteksnya (misalnya, 128K token untuk konteks total GPT-4o, termasuk input dan output).

  Dalam Java, Anda dapat menggunakan perpustakaan seperti OpenAI SDK untuk menangani tokenisasi secara otomatis saat mengirim permintaan ke model AI.

- **Embeddings**: Embeddings adalah representasi vektor dari token yang menangkap makna semantik. Mereka adalah representasi numerik (biasanya array angka floating-point) yang memungkinkan model memahami hubungan antar kata dan menghasilkan respons yang relevan secara kontekstual. Kata-kata yang mirip memiliki embeddings yang mirip, sehingga model dapat memahami konsep seperti sinonim dan hubungan semantik.

![Figure: Embeddings](../../../translated_images/id/embedding.398e50802c0037f9.webp)

  Dalam Java, Anda dapat menghasilkan embeddings menggunakan OpenAI SDK atau perpustakaan lain yang mendukung pembuatan embedding. Embeddings ini penting untuk tugas seperti pencarian semantik, di mana Anda ingin menemukan konten yang serupa berdasarkan makna, bukan kecocokan teks tepat.

- **Basis data vektor**: Basis data vektor adalah sistem penyimpanan khusus yang dioptimalkan untuk embeddings. Mereka memungkinkan pencarian kemiripan yang efisien dan sangat penting untuk pola Retrieval-Augmented Generation (RAG) di mana Anda perlu menemukan informasi relevan dari dataset besar berdasarkan kemiripan semantik daripada kecocokan tepat.

![Figure: Vector database architecture showing how embeddings are stored and retrieved for similarity search.](../../../translated_images/id/vector.f12f114934e223df.webp)

> **Catatan**: Dalam kursus ini, kami tidak akan membahas basis data vektor tetapi menganggap mereka layak disebut karena sering digunakan dalam aplikasi dunia nyata.

- **Agen & MCP**: Komponen AI yang berinteraksi secara otonom dengan model, alat, dan sistem eksternal. Model Context Protocol (MCP) menyediakan cara standar untuk agen mengakses sumber data eksternal dan alat secara aman. Pelajari lebih lanjut di kursus kami [MCP untuk Pemula](https://github.com/microsoft/mcp-for-beginners).

Dalam aplikasi AI Java, Anda akan menggunakan token untuk pemrosesan teks, embeddings untuk pencarian semantik dan RAG, basis data vektor untuk pengambilan data, dan agen dengan MCP untuk membangun sistem cerdas yang menggunakan alat.

![Figure: how a prompt becomes a reply—tokens, vectors, optional RAG lookup, LLM thinking, and an MCP agent all in one quick flow..](../../../translated_images/id/flow.f4ef62c3052d12a8.webp)

### Alat dan Perpustakaan Pengembangan AI untuk Java

Java menyediakan alat yang sangat baik untuk pengembangan AI. Ada tiga perpustakaan utama yang akan kami jelajahi sepanjang kursus ini - OpenAI Java SDK, Azure OpenAI SDK, dan Spring AI.

Berikut tabel referensi cepat yang menunjukkan SDK mana digunakan dalam contoh setiap bab:

| Bab | Contoh | SDK |
|---------|--------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**Tautan Dokumentasi SDK:**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

OpenAI SDK adalah perpustakaan resmi Java untuk API OpenAI. Ini menyediakan antarmuka sederhana dan konsisten untuk berinteraksi dengan model OpenAI, memudahkan integrasi kemampuan AI ke dalam aplikasi Java. Aplikasi Pet Story dan contoh Foundry Local di Bab 4 menunjukkan pendekatan OpenAI SDK dengan Azure AI Foundry.

#### Spring AI

Spring AI adalah kerangka kerja komprehensif yang menghadirkan kemampuan AI ke aplikasi Spring, menyediakan lapisan abstraksi konsisten di berbagai penyedia AI. Ia terintegrasi mulus dengan ekosistem Spring, menjadikannya pilihan ideal untuk aplikasi Java perusahaan yang membutuhkan kemampuan AI.

Kekuatan Spring AI terletak pada integrasi mulus dengan ekosistem Spring, membuatnya mudah membangun aplikasi AI siap produksi dengan pola Spring yang familiar seperti dependency injection, manajemen konfigurasi, dan kerangka kerja pengujian. Anda akan menggunakan Spring AI di Bab 2 dan 4 untuk membangun aplikasi menggunakan OpenAI dan Model Context Protocol (MCP) libraries di Spring AI.

##### Model Context Protocol (MCP)

[Model Context Protocol (MCP)](https://modelcontextprotocol.io/) adalah standar yang sedang berkembang yang memungkinkan aplikasi AI berinteraksi dengan aman dengan sumber data dan alat eksternal. MCP menyediakan cara standar bagi model AI untuk mengakses informasi kontekstual dan mengeksekusi tindakan dalam aplikasi Anda.

Di Bab 4, Anda akan membangun layanan kalkulator MCP sederhana yang menunjukkan dasar-dasar Model Context Protocol dengan Spring AI, memperlihatkan cara membuat integrasi alat dasar dan arsitektur layanan.

#### Azure OpenAI Java SDK

Perpustakaan klien Azure OpenAI untuk Java adalah adaptasi dari REST API OpenAI yang menyediakan antarmuka idiomatik dan integrasi dengan ekosistem SDK Azure lainnya. Di Bab 3, Anda akan membangun aplikasi menggunakan Azure OpenAI SDK, termasuk aplikasi chat, panggilan fungsi, dan pola RAG (Retrieval-Augmented Generation).

> Catatan: Azure OpenAI SDK tertinggal dari OpenAI Java SDK dalam hal fitur, jadi untuk proyek mendatang, pertimbangkan menggunakan OpenAI Java SDK.

## Ringkasan

Itulah dasar-dasarnya! Sekarang Anda mengerti:

- Konsep inti di balik generative AI - dari LLM dan rekayasa prompt hingga token, embeddings, dan basis data vektor
- Pilihan toolkit Anda untuk pengembangan AI Java: Azure OpenAI SDK, Spring AI, dan OpenAI Java SDK
- Apa itu Model Context Protocol dan bagaimana ia memungkinkan agen AI bekerja dengan alat eksternal

## Langkah Selanjutnya

[Bab 2: Menyiapkan Lingkungan Pengembangan](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Penafian**:
Dokumen ini telah diterjemahkan menggunakan layanan terjemahan AI [Co-op Translator](https://github.com/Azure/co-op-translator). Meskipun kami berupaya untuk mencapai akurasi, harap diketahui bahwa terjemahan otomatis mungkin mengandung kesalahan atau ketidakakuratan. Dokumen asli dalam bahasa aslinya harus dianggap sebagai sumber yang sah. Untuk informasi penting, disarankan menggunakan terjemahan profesional oleh manusia. Kami tidak bertanggung jawab atas kesalahpahaman atau penafsiran yang keliru yang timbul dari penggunaan terjemahan ini.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->