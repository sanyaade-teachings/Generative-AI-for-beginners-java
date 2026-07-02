# Tutorial Kalkulator MCP untuk Pemula

## Jadual Kandungan

- [Apa Yang Akan Anda Pelajari](#apa-yang-akan-anda-pelajari)
- [Prasyarat](#prasyarat)
- [Memahami Struktur Projek](#memahami-struktur-projek)
- [Komponen Utama Dijelaskan](#komponen-utama-dijelaskan)
  - [1. Aplikasi Utama](#1-aplikasi-utama)
  - [2. Perkhidmatan Kalkulator](#2-perkhidmatan-kalkulator)
  - [3. Klien MCP Terus](#3-klien-mcp-terus)
  - [4. Klien Berkuasa AI](#4-klien-berkuasa-ai)
- [Menjalankan Contoh](#menjalankan-contoh)
- [Bagaimana Semua Berfungsi Bersama](#bagaimana-semua-berfungsi-bersama)
- [Langkah Seterusnya](#langkah-seterusnya)

## Apa Yang Akan Anda Pelajari

Tutorial ini menerangkan cara membina perkhidmatan kalkulator menggunakan Model Context Protocol (MCP). Anda akan memahami:

- Cara mencipta perkhidmatan yang boleh digunakan AI sebagai alat
- Cara menyiapkan komunikasi terus dengan perkhidmatan MCP
- Bagaimana model AI boleh secara automatik memilih alat mana untuk digunakan
- Perbezaan antara panggilan protokol terus dan interaksi dibantu AI

## Prasyarat

Sebelum bermula, pastikan anda mempunyai:
- Java 21 atau lebih tinggi dipasang
- Maven untuk pengurusan pergantungan
- Penempatan model Azure AI Foundry (sediakan dengan `azd up` — lihat [Bab 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), log masuk dengan `az login` (pengesahan tanpa kunci)
- Pemahaman asas tentang Java dan Spring Boot

## Memahami Struktur Projek

Projek kalkulator mempunyai beberapa fail penting:

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

## Komponen Utama Dijelaskan

### 1. Aplikasi Utama

**Fail:** `McpServerApplication.java`

Ini adalah titik masuk untuk perkhidmatan kalkulator kami. Ia sebuah aplikasi Spring Boot standard dengan satu tambahan istimewa:

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

**Apa yang dilakukan ini:**
- Memulakan pelayan web Spring Boot pada port 8080
- Mencipta `ToolCallbackProvider` yang menjadikan kaedah kalkulator kami tersedia sebagai alat MCP
- Anotasi `@Bean` memberitahu Spring untuk menguruskan ini sebagai komponen yang boleh digunakan bahagian lain

### 2. Perkhidmatan Kalkulator

**Fail:** `CalculatorService.java`

Di sinilah semua pengiraan dibuat. Setiap kaedah ditandakan dengan `@Tool` untuk menjadikannya boleh diakses melalui MCP:

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

**Ciri-ciri utama:**

1. **Anotasi `@Tool`**: Ini memberitahu MCP bahawa kaedah ini boleh dipanggil oleh klien luar
2. **Keterangan Jelas**: Setiap alat mempunyai penerangan yang membantu model AI memahami bila untuk menggunakannya
3. **Format Pulangan Konsisten**: Semua operasi mengembalikan rentetan yang mudah dibaca seperti "5.00 + 3.00 = 8.00"
4. **Pengendalian Ralat**: Pembahagian dengan sifar dan punca kuasa dua negatif mengembalikan mesej ralat

**Operasi Yang Tersedia:**
- `add(a, b)` - Menambah dua nombor
- `subtract(a, b)` - Menolak nombor kedua daripada nombor pertama
- `multiply(a, b)` - Mendarab dua nombor
- `divide(a, b)` - Membahagikan nombor pertama dengan nombor kedua (dengan pemeriksaan sifar)
- `power(base, exponent)` - Menaikkan pangkalan ke kuasa eksponen
- `squareRoot(number)` - Mengira punca kuasa dua (dengan pemeriksaan negatif)
- `modulus(a, b)` - Mengembalikan baki pembahagian
- `absolute(number)` - Mengembalikan nilai mutlak
- `help()` - Mengembalikan maklumat tentang semua operasi

### 3. Klien MCP Terus

**Fail:** `SDKClient.java`

Klien ini bercakap terus dengan pelayan MCP tanpa menggunakan AI. Ia memanggil fungsi kalkulator tertentu secara manual:

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
        
        // Senarai alat yang tersedia
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

**Apa yang dilakukan ini:**
1. **Sambung** ke pelayan kalkulator di `http://localhost:8080` menggunakan corak pembina
2. **Senaraikan** semua alat yang tersedia (fungsi kalkulator kami)
3. **Panggil** fungsi tertentu dengan parameter tepat
4. **Cetak** keputusan secara langsung

**Nota:** Contoh ini menggunakan pergantungan Spring AI 1.1.0-SNAPSHOT, yang memperkenalkan corak pembina untuk `WebFluxSseClientTransport`. Jika anda menggunakan versi stabil lama, anda mungkin perlu menggunakan konstruktor terus.

**Bila digunakan:** Apabila anda tahu dengan tepat pengiraan yang ingin dilakukan dan mahu memanggilnya secara programatik.

### 4. Klien Berkuasa AI

**Fail:** `LangChain4jClient.java`

Klien ini menggunakan model AI (GPT-4o-mini) yang boleh secara automatik memutuskan alat kalkulator mana untuk digunakan:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Sediakan model AI (Azure AI Foundry, pengesahan tanpa kunci melalui Microsoft Entra ID)
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

        // Sambungkan ke pelayan kalkulator MCP kami
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Menunjukkan apa yang AI sedang lakukan
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Berikan AI akses kepada alat kalkulator kami
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Cipta bot AI yang boleh menggunakan kalkulator kami
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Kini kita boleh meminta AI melakukan pengiraan dalam bahasa semula jadi
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Apa yang dilakukan ini:**
1. **Mencipta** sambungan model AI menggunakan token GitHub anda
2. **Sambungkan** AI ke pelayan MCP kalkulator kami
3. **Beri** AI akses kepada semua alat kalkulator kami
4. **Benarkan** permintaan bahasa semula jadi seperti "Kira jumlah 24.5 dan 17.3"

**AI secara automatik:**
- Memahami anda mahu menambah nombor
- Memilih alat `add`
- Memanggil `add(24.5, 17.3)`
- Mengembalikan keputusan dalam respons semula jadi

## Menjalankan Contoh

### Langkah 1: Mulakan Pelayan Kalkulator

Mula-mula, log masuk dan tetapkan titik akhir Azure AI Foundry anda (diperlukan untuk klien AI — pengesahan tanpa kunci, tiada kunci API):

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

Mulakan pelayan:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

Pelayan akan bermula di `http://localhost:8080`. Anda akan melihat:
```
Started McpServerApplication in X.XXX seconds
```

### Langkah 2: Uji dengan Klien Terus

Dalam terminal **BARU** dengan pelayan masih berjalan, jalankan klien MCP terus:
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

Anda akan melihat AI secara automatik menggunakan alat:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Langkah 4: Tutup Pelayan MCP

Apabila selesai menguji, anda boleh hentikan klien AI dengan menekan `Ctrl+C` di terminalnya. Pelayan MCP akan terus berjalan sehingga anda hentikannya.
Untuk menghentikan pelayan, tekan `Ctrl+C` di terminal tempat ia berjalan.

## Bagaimana Semua Berfungsi Bersama

Berikut adalah aliran lengkap apabila anda bertanya kepada AI "Berapakah 5 + 3?":

1. **Anda** bertanya kepada AI dalam bahasa semula jadi
2. **AI** menganalisis permintaan anda dan sedar anda mahu penambahan
3. **AI** memanggil pelayan MCP: `add(5.0, 3.0)`
4. **Perkhidmatan Kalkulator** melakukan: `5.0 + 3.0 = 8.0`
5. **Perkhidmatan Kalkulator** mengembalikan: `"5.00 + 3.00 = 8.00"`
6. **AI** menerima keputusan dan memformat respons semula jadi
7. **Anda** mendapat: "Jumlah 5 dan 3 adalah 8"

## Langkah Seterusnya

Untuk lebih banyak contoh, lihat [Bab 04: Contoh praktikal](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Penafian**:
Dokumen ini telah diterjemahkan menggunakan perkhidmatan terjemahan AI [Co-op Translator](https://github.com/Azure/co-op-translator). Walaupun kami berusaha untuk ketepatan, sila ambil maklum bahawa terjemahan automatik mungkin mengandungi kesilapan atau ketidaktepatan. Dokumen asal dalam bahasa asalnya harus dianggap sebagai sumber yang sahih. Untuk maklumat penting, terjemahan oleh manusia profesional adalah disyorkan. Kami tidak bertanggungjawab terhadap sebarang salah faham atau salah tafsir yang timbul daripada penggunaan terjemahan ini.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->