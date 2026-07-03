# Tutorial Kalkulator MCP untuk Pemula

## Daftar Isi

- [Apa yang Akan Anda Pelajari](#apa-yang-akan-anda-pelajari)
- [Prasyarat](#prasyarat)
- [Memahami Struktur Proyek](#memahami-struktur-proyek)
- [Penjelasan Komponen Inti](#penjelasan-komponen-inti)
  - [1. Aplikasi Utama](#1-aplikasi-utama)
  - [2. Layanan Kalkulator](#2-layanan-kalkulator)
  - [3. Klien MCP Langsung](#3-klien-mcp-langsung)
  - [4. Klien Berbasis AI](#4-klien-berbasis-ai)
- [Menjalankan Contoh](#menjalankan-contoh)
- [Bagaimana Semua Bekerja Bersama](#bagaimana-semua-bekerja-bersama)
- [Langkah Selanjutnya](#langkah-selanjutnya)

## Apa yang Akan Anda Pelajari

Tutorial ini menjelaskan cara membangun layanan kalkulator menggunakan Model Context Protocol (MCP). Anda akan memahami:

- Cara membuat layanan yang dapat digunakan AI sebagai alat
- Cara mengatur komunikasi langsung dengan layanan MCP
- Cara model AI dapat secara otomatis memilih alat yang akan digunakan
- Perbedaan antara panggilan protokol langsung dan interaksi dengan bantuan AI

## Prasyarat

Sebelum memulai, pastikan Anda sudah:
- Terpasang Java 21 atau lebih tinggi
- Maven untuk manajemen dependensi
- Penempatan model Azure AI Foundry (siapkan dengan `azd up` — lihat [Bab 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), masuk dengan `az login` (autentikasi tanpa kunci)
- Pemahaman dasar tentang Java dan Spring Boot

## Memahami Struktur Proyek

Proyek kalkulator memiliki beberapa file penting:

```
calculator/
├── src/main/java/com/microsoft/mcp/sample/server/
│   ├── McpServerApplication.java          # Main Spring Boot app
│   └── service/CalculatorService.java     # Calculator operations
└── src/test/java/com/microsoft/mcp/sample/client/
    ├── SDKClient.java                     # Direct MCP communication
    ├── LangChain4jClient.java            # AI-powered client
    └── Bot.java                          # Simple chat interface
```

## Penjelasan Komponen Inti

### 1. Aplikasi Utama

**File:** `McpServerApplication.java`

Ini adalah titik masuk layanan kalkulator kita. Ini adalah aplikasi Spring Boot standar dengan satu tambahan khusus:

```java
@SpringBootApplication
public class McpServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(McpServerApplication.class, args);
    }
    
    @Bean
    public ToolCallbackProvider calculatorTools(CalculatorService calculator) {
        return MethodToolCallbackProvider.builder().toolObjects(calculator).build();
    }
}
```

**Yang dilakukan ini:**
- Menjalankan server web Spring Boot pada port 8080
- Membuat `ToolCallbackProvider` yang membuat metode kalkulator kita tersedia sebagai alat MCP
- Anotasi `@Bean` memberitahu Spring untuk mengelola ini sebagai komponen yang dapat digunakan oleh bagian lain

### 2. Layanan Kalkulator

**File:** `CalculatorService.java`

Di sinilah semua perhitungan dilakukan. Setiap metode diberi tanda `@Tool` agar tersedia melalui MCP:

```java
@Service
public class CalculatorService {

    @Tool(description = "Add two numbers together")
    public String add(double a, double b) {
        double result = a + b;
        return formatResult(a, "+", b, result);
    }

    @Tool(description = "Subtract the second number from the first number")
    public String subtract(double a, double b) {
        double result = a - b;
        return formatResult(a, "-", b, result);
    }
    
    // Lebih banyak operasi kalkulator...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Fitur utama:**

1. **Anotasi `@Tool`**: Memberitahu MCP bahwa metode ini dapat dipanggil oleh klien eksternal
2. **Deskripsi Jelas**: Setiap alat memiliki deskripsi yang membantu model AI memahami kapan harus menggunakan
3. **Format Keluaran Konsisten**: Semua operasi mengembalikan string yang mudah dibaca seperti "5.00 + 3.00 = 8.00"
4. **Penanganan Error**: Pembagian dengan nol dan akar kuadrat negatif mengembalikan pesan error

**Operasi yang Tersedia:**
- `add(a, b)` - Menambahkan dua angka
- `subtract(a, b)` - Mengurangi angka kedua dari angka pertama
- `multiply(a, b)` - Mengalikan dua angka
- `divide(a, b)` - Membagi angka pertama dengan angka kedua (dengan pengecekan nol)
- `power(base, exponent)` - Memangkatkan basis dengan eksponen
- `squareRoot(number)` - Menghitung akar kuadrat (dengan pengecekan negatif)
- `modulus(a, b)` - Mengembalikan sisa pembagian
- `absolute(number)` - Mengembalikan nilai absolut
- `help()` - Mengembalikan informasi tentang semua operasi

### 3. Klien MCP Langsung

**File:** `SDKClient.java`

Klien ini berkomunikasi langsung ke server MCP tanpa menggunakan AI. Ia memanggil fungsi kalkulator tertentu secara manual:

```java
public class SDKClient {
    
    public static void main(String[] args) {
        McpClientTransport transport = WebFluxSseClientTransport.builder(
            WebClient.builder().baseUrl("http://localhost:8080")
        ).build();
        new SDKClient(transport).run();
    }
    
    public void run() {
        var client = McpClient.sync(this.transport).build();
        client.initialize();
        
        // Daftar alat yang tersedia
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Panggil fungsi kalkulator tertentu
        CallToolResult resultAdd = client.callTool(
            new CallToolRequest("add", Map.of("a", 5.0, "b", 3.0))
        );
        System.out.println("Add Result = " + resultAdd);
        
        CallToolResult resultSqrt = client.callTool(
            new CallToolRequest("squareRoot", Map.of("number", 16.0))
        );
        System.out.println("Square Root Result = " + resultSqrt);
        
        client.closeGracefully();
    }
}
```

**Yang dilakukan ini:**
1. **Menghubungkan** ke server kalkulator di `http://localhost:8080` menggunakan pola builder
2. **Mendaftar** semua alat yang tersedia (fungsi kalkulator kita)
3. **Memanggil** fungsi tertentu dengan parameter yang tepat
4. **Mencetak** hasil secara langsung

**Catatan:** Contoh ini menggunakan dependensi Spring AI 1.1.0-SNAPSHOT, yang memperkenalkan pola builder untuk `WebFluxSseClientTransport`. Jika Anda menggunakan versi stabil yang lebih lama, mungkin perlu menggunakan konstruktor langsung.

**Kapan menggunakan ini:** Saat Anda tahu persis perhitungan yang ingin dilakukan dan ingin memanggilnya secara programatis.

### 4. Klien Berbasis AI

**File:** `LangChain4jClient.java`

Klien ini menggunakan model AI (GPT-4o-mini) yang dapat secara otomatis memutuskan alat kalkulator mana yang akan digunakan:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Siapkan model AI (Azure AI Foundry, autentikasi tanpa kunci melalui Microsoft Entra ID)
        String endpoint = System.getenv("AZURE_OPENAI_ENDPOINT");
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1";
        String token = new DefaultAzureCredentialBuilder().build()
                .getToken(new TokenRequestContext().addScopes("https://ai.azure.com/.default"))
                .block().getToken();
        ChatLanguageModel model = OpenAiOfficialChatModel.builder()
                .baseUrl(baseUrl)
                .apiKey(token)
                .modelName("gpt-4o-mini")
                .build();

        // Sambungkan ke server kalkulator MCP kami
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Menunjukkan apa yang sedang dilakukan AI
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Beri akses AI ke alat kalkulator kami
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Buat bot AI yang dapat menggunakan kalkulator kami
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Sekarang kita dapat meminta AI untuk melakukan perhitungan dalam bahasa alami
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Yang dilakukan ini:**
1. **Membuat** koneksi model AI menggunakan token GitHub Anda
2. **Menghubungkan** AI ke server MCP kalkulator kita
3. **Memberi** AI akses ke semua alat kalkulator kita
4. **Memungkinkan** permintaan berbahasa alami seperti "Hitung jumlah 24.5 dan 17.3"

**AI secara otomatis:**
- Memahami Anda ingin menjumlahkan angka
- Memilih alat `add`
- Memanggil `add(24.5, 17.3)`
- Mengembalikan hasil dalam respons alami

## Menjalankan Contoh

### Langkah 1: Mulai Server Kalkulator

Pertama, masuk dan atur endpoint Azure AI Foundry Anda (diperlukan untuk klien AI — autentikasi tanpa kunci, tanpa API key):

**Windows:**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**Linux/macOS:**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

Mulai server:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

Server akan mulai pada `http://localhost:8080`. Anda akan melihat:
```
Started McpServerApplication in X.XXX seconds
```

### Langkah 2: Uji dengan Klien Langsung

Di terminal **BARU** dengan server masih berjalan, jalankan klien MCP langsung:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Anda akan melihat output seperti:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Langkah 3: Uji dengan Klien AI

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Anda akan melihat AI menggunakan alat secara otomatis:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Langkah 4: Tutup Server MCP

Setelah selesai pengujian, Anda dapat menghentikan klien AI dengan menekan `Ctrl+C` di terminalnya. Server MCP akan tetap berjalan sampai Anda menghentikannya.
Untuk menghentikan server, tekan `Ctrl+C` di terminal tempat server berjalan.

## Bagaimana Semua Bekerja Bersama

Berikut alur lengkap ketika Anda menanyakan pada AI "Berapa 5 + 3?":

1. **Anda** bertanya pada AI dalam bahasa alami
2. **AI** menganalisis permintaan dan menyadari Anda ingin penjumlahan
3. **AI** memanggil server MCP: `add(5.0, 3.0)`
4. **Layanan Kalkulator** melakukan: `5.0 + 3.0 = 8.0`
5. **Layanan Kalkulator** mengembalikan: `"5.00 + 3.00 = 8.00"`
6. **AI** menerima hasil dan memformat respons alami
7. **Anda** mendapatkan: "Jumlah dari 5 dan 3 adalah 8"

## Langkah Selanjutnya

Untuk contoh lebih lanjut, lihat [Bab 04: Contoh praktis](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Penafian**:
Dokumen ini telah diterjemahkan menggunakan layanan terjemahan AI [Co-op Translator](https://github.com/Azure/co-op-translator). Meskipun kami berupaya untuk mencapai akurasi, harap diketahui bahwa terjemahan otomatis mungkin mengandung kesalahan atau ketidakakuratan. Dokumen asli dalam bahasa aslinya harus dianggap sebagai sumber yang sah. Untuk informasi penting, disarankan menggunakan terjemahan profesional oleh manusia. Kami tidak bertanggung jawab atas kesalahpahaman atau penafsiran yang keliru yang timbul dari penggunaan terjemahan ini.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->