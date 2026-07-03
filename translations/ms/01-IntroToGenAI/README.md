# Pengenalan kepada Generative AI - Edisi Java

## Apa Yang Akan Anda Pelajari

- **Asas Generative AI** termasuk LLM, kejuruteraan prompt, token, embedding, dan pangkalan data vektor
- **Bandingkan alat pembangunan AI Java** termasuk Azure OpenAI SDK, Spring AI, dan OpenAI Java SDK
- **Temui Protokol Konteks Model** dan peranannya dalam komunikasi ejen AI

## Jadual Kandungan

- [Pengenalan](#pengenalan)
- [Segaran cepat tentang konsep Generative AI](#segaran-cepat-tentang-konsep-generative-ai)
- [Ulasan kejuruteraan prompt](#ulasan-kejuruteraan-prompt)
- [Token, embedding, dan ejen](#token-embedding-dan-ejen)
- [Alat dan Perpustakaan Pembangunan AI untuk Java](#alat-dan-perpustakaan-pembangunan-ai-untuk-java)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [Ringkasan](#ringkasan)
- [Langkah Seterusnya](#langkah-seterusnya)

## Pengenalan

Selamat datang ke bab pertama Generative AI untuk Pemula - Edisi Java! Pelajaran asas ini memperkenalkan anda kepada konsep teras generative AI dan cara menggunakannya dengan Java. Anda akan belajar tentang blok binaan penting untuk aplikasi AI, termasuk Large Language Models (LLM), token, embedding, dan ejen AI. Kami juga akan meneroka peralatan utama Java yang akan anda gunakan sepanjang kursus ini.

### Segaran cepat tentang konsep Generative AI

Generative AI adalah satu jenis kecerdasan buatan yang menghasilkan kandungan baru, seperti teks, imej, atau kod, berdasarkan corak dan hubungan yang dipelajari dari data. Model generative AI dapat menghasilkan respons seperti manusia, memahami konteks, dan kadangkala juga mencipta kandungan yang kelihatan manusiawi.

Semasa anda membangunkan aplikasi AI Java, anda akan bekerja dengan **model generative AI** untuk mencipta kandungan. Beberapa keupayaan model generative AI termasuk:

- **Penjanaan Teks**: Menghasilkan teks seperti manusia untuk chatbot, kandungan, dan pelengkap teks.
- **Penjanaan dan Analisis Imej**: Menghasilkan imej realistik, mempertingkatkan foto, dan mengesan objek.
- **Penjanaan Kod**: Menulis potongan kod atau skrip.

Terdapat jenis model tertentu yang dioptimumkan untuk tugas yang berbeza. Contohnya, kedua-dua **Small Language Models (SLM)** dan **Large Language Models (LLM)** boleh mengendalikan penjanaan teks, dengan LLM biasanya menawarkan prestasi lebih baik untuk tugas yang kompleks. Untuk tugas berkaitan imej, anda akan menggunakan model visi khusus atau model multi-modal.

![Figure: Generative AI model types and use cases.](../../../translated_images/ms/llms.225ca2b8a0d34473.webp)

Sudah tentu, respons daripada model-model ini tidak sempurna sepanjang masa. Anda mungkin pernah mendengar tentang model yang "berhalusinasi" atau menghasilkan maklumat yang salah dengan cara yang authoritative. Tetapi anda boleh membantu membimbing model untuk menghasilkan respons yang lebih baik dengan memberikan arahan dan konteks yang jelas. Di sinilah **kejuruteraan prompt** memainkan peranan.

#### Ulasan kejuruteraan prompt

Kejuruteraan prompt adalah amalan mereka bentuk input yang berkesan untuk membimbing model AI ke arah hasil yang dikehendaki. Ia melibatkan:

- **Kejelasan**: Membuat arahan yang jelas dan tidak samar.
- **Konteks**: Memberikan maklumat latar belakang yang diperlukan.
- **Sekatan**: Menentukan sebarang had atau format.

Beberapa amalan terbaik untuk kejuruteraan prompt termasuk reka bentuk prompt, arahan yang jelas, pecahan tugas, pembelajaran satu-shot dan few-shot, serta penalaan prompt. Menguji pelbagai prompt adalah penting untuk mengetahui apa yang terbaik bagi kes penggunaan khusus anda.

Semasa membangunkan aplikasi, anda akan bekerja dengan pelbagai jenis prompt:
- **System prompts**: Menetapkan peraturan asas dan konteks untuk perilaku model
- **User prompts**: Data input dari pengguna aplikasi anda
- **Assistant prompts**: Respons model berdasarkan system dan user prompts

> **Ketahui lebih lanjut**: Ketahui lebih lanjut tentang kejuruteraan prompt dalam [bab Kejuruteraan Prompt kursus GenAI untuk Pemula](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### Token, embedding, dan ejen

Apabila bekerja dengan model generative AI, anda akan menemui istilah seperti **token**, **embedding**, **ejen**, dan **Protokol Konteks Model (MCP)**. Berikut adalah gambaran terperinci tentang konsep ini:

- **Token**: Token adalah unit terkecil teks dalam model. Ia boleh menjadi perkataan, aksara, atau subword. Token digunakan untuk mewakili data teks dalam format yang boleh difahami model. Sebagai contoh, ayat "The quick brown fox jumped over the lazy dog" mungkin ditokenkan sebagai ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] atau ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"] bergantung pada strategi tokenisasi.

![Figure: Generative AI tokens example of breaking words into tokens](../../../translated_images/ms/tokens.6283ed277a2ffff4.webp)

Tokenisasi adalah proses memecah teks kepada unit-unit kecil ini. Ini penting kerana model beroperasi berdasarkan token dan bukan teks mentah. Bilangan token dalam prompt mempengaruhi panjang dan kualiti respons model, kerana model mempunyai had token untuk tetingkap konteks mereka (contohnya, 128K token untuk konteks total GPT-4o, termasuk input dan output).

  Dalam Java, anda boleh menggunakan perpustakaan seperti OpenAI SDK untuk mengendalikan tokenisasi secara automatik apabila menghantar permintaan ke model AI.

- **Embedding**: Embedding adalah representasi vektor token yang menangkap makna semantik. Ia adalah representasi berangka (biasanya tatasusunan nombor titik terapung) yang membolehkan model memahami hubungan antara perkataan dan menjana respons yang relevan secara konteks. Perkataan yang serupa mempunyai embedding yang serupa, membolehkan model memahami konsep seperti sinonim dan hubungan semantik.

![Figure: Embeddings](../../../translated_images/ms/embedding.398e50802c0037f9.webp)

  Dalam Java, anda boleh menjana embedding menggunakan OpenAI SDK atau perpustakaan lain yang menyokong penjanaan embedding. Embedding ini penting untuk tugas seperti carian semantik, di mana anda ingin mencari kandungan yang serupa berdasarkan makna dan bukan padanan teks tepat.

- **Pangkalan data vektor**: Pangkalan data vektor adalah sistem penyimpanan khusus yang dioptimumkan untuk embedding. Ia membolehkan carian kesamaan yang cekap dan sangat penting untuk corak Retrieval-Augmented Generation (RAG) di mana anda perlu mencari maklumat yang relevan daripada dataset besar berdasarkan kesamaan semantik dan bukan padanan tepat.

![Figure: Vector database architecture showing how embeddings are stored and retrieved for similarity search.](../../../translated_images/ms/vector.f12f114934e223df.webp)

> **Nota**: Dalam kursus ini, kita tidak akan meliputi Pangkalan data vektor tetapi anggap ia patut disebut kerana ia sering digunakan dalam aplikasi dunia sebenar.

- **Ejen & MCP**: Komponen AI yang berinteraksi secara autonomi dengan model, alat, dan sistem luaran. Model Context Protocol (MCP) menyediakan cara standard untuk ejen mengakses sumber data dan alat luaran dengan selamat. Ketahui lebih lanjut dalam kursus [MCP untuk Pemula](https://github.com/microsoft/mcp-for-beginners).

Dalam aplikasi AI Java, anda akan menggunakan token untuk pemprosesan teks, embedding untuk carian semantik dan RAG, pangkalan data vektor untuk pengambilan data, dan ejen dengan MCP untuk membina sistem pintar yang menggunakan alat.

![Figure: how a prompt becomes a reply—tokens, vectors, optional RAG lookup, LLM thinking, and an MCP agent all in one quick flow..](../../../translated_images/ms/flow.f4ef62c3052d12a8.webp)

### Alat dan Perpustakaan Pembangunan AI untuk Java

Java menawarkan peralatan cemerlang untuk pembangunan AI. Terdapat tiga perpustakaan utama yang akan kita terokai sepanjang kursus ini - OpenAI Java SDK, Azure OpenAI SDK, dan Spring AI.

Berikut adalah jadual rujukan cepat yang menunjukkan SDK mana digunakan dalam contoh setiap bab:

| Bab | Contoh | SDK |
|---------|--------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**Pautan Dokumentasi SDK:**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

OpenAI SDK adalah perpustakaan rasmi Java untuk OpenAI API. Ia menyediakan antara muka mudah dan konsisten untuk berinteraksi dengan model OpenAI, menjadikan integrasi keupayaan AI dalam aplikasi Java menjadi mudah. Aplikasi Pet Story dan contoh Foundry Local dalam Bab 4 menunjukkan pendekatan OpenAI SDK dengan Azure AI Foundry.

#### Spring AI

Spring AI adalah rangka kerja menyeluruh yang membawa keupayaan AI kepada aplikasi Spring, menyediakan lapisan abstraksi konsisten merentasi pelbagai penyedia AI. Ia berintegrasi dengan lancar dengan ekosistem Spring, menjadikannya pilihan ideal untuk aplikasi Java perusahaan yang memerlukan keupayaan AI.

Kekuatan Spring AI terletak pada integrasinya yang lancar dengan ekosistem Spring, memudahkan pembinaan aplikasi AI yang siap produksi dengan pola Spring yang biasa seperti suntikan kebergantungan, pengurusan konfigurasi, dan rangka kerja pengujian. Anda akan menggunakan Spring AI dalam Bab 2 dan 4 untuk membina aplikasi yang memanfaatkan kedua-dua OpenAI dan protokol Model Context Protocol (MCP) dengan perpustakaan Spring AI.

##### Protokol Konteks Model (MCP)

[Protokol Konteks Model (MCP)](https://modelcontextprotocol.io/) adalah standard muncul yang membolehkan aplikasi AI berinteraksi dengan selamat dengan sumber data dan alat luaran. MCP menyediakan cara standard untuk model AI mengakses maklumat kontekstual dan melaksanakan tindakan dalam aplikasi anda.

Dalam Bab 4, anda akan membina perkhidmatan kalkulator MCP mudah yang menunjukkan asas Protokol Konteks Model dengan Spring AI, menunjukkan cara membuat integrasi alat asas dan seni bina perkhidmatan.

#### Azure OpenAI Java SDK

Pustaka klien Azure OpenAI untuk Java adalah penyesuaian REST API OpenAI yang menyediakan antara muka idiomatik dan integrasi dengan ekosistem SDK Azure yang lain. Dalam Bab 3, anda akan membina aplikasi menggunakan Azure OpenAI SDK, termasuk aplikasi sembang, panggilan fungsi, dan corak RAG (Retrieval-Augmented Generation).

> Nota: Azure OpenAI SDK tertinggal di belakang OpenAI Java SDK dari segi ciri, oleh itu untuk projek masa depan, pertimbangkan menggunakan OpenAI Java SDK.

## Ringkasan

Itu menamatkan asas-asas! Anda kini memahami:

- Konsep teras di sebalik generative AI - dari LLM dan kejuruteraan prompt hingga token, embedding, dan pangkalan data vektor
- Pilihan alat pembangunan AI Java anda: Azure OpenAI SDK, Spring AI, dan OpenAI Java SDK
- Apa itu Protokol Konteks Model dan bagaimana ia membolehkan ejen AI bekerja dengan alat luaran

## Langkah Seterusnya

[Bab 2: Menyiapkan Persekitaran Pembangunan](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Penafian**:
Dokumen ini telah diterjemahkan menggunakan perkhidmatan terjemahan AI [Co-op Translator](https://github.com/Azure/co-op-translator). Walaupun kami berusaha untuk ketepatan, sila ambil maklum bahawa terjemahan automatik mungkin mengandungi kesilapan atau ketidaktepatan. Dokumen asal dalam bahasa asalnya harus dianggap sebagai sumber yang sahih. Untuk maklumat penting, terjemahan oleh manusia profesional adalah disyorkan. Kami tidak bertanggungjawab terhadap sebarang salah faham atau salah tafsir yang timbul daripada penggunaan terjemahan ini.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->