# Yeni Başlayanlar için MCP Hesap Makinesi Eğitimi

## İçindekiler

- [Neleri Öğreneceksiniz](#neleri-öğreneceksiniz)
- [Ön Koşullar](#ön-koşullar)
- [Proje Yapısını Anlamak](#proje-yapısını-anlamak)
- [Temel Bileşenler Açıklaması](#temel-bileşenler-açıklaması)
  - [1. Ana Uygulama](#1-ana-uygulama)
  - [2. Hesap Makinesi Servisi](#2-hesap-makinesi-servisi)
  - [3. Doğrudan MCP İstemcisi](#3-doğrudan-mcp-istemcisi)
  - [4. Yapay Zeka Destekli İstemci](#4-yapay-zeka-destekli-istemci)
- [Örneklerin Çalıştırılması](#örneklerin-çalıştırılması)
- [Hepsi Nasıl Birlikte Çalışır](#hepsi-nasıl-birlikte-çalışır)
- [Sonraki Adımlar](#sonraki-adımlar)

## Neleri Öğreneceksiniz

Bu eğitim, Model Context Protocol (MCP) kullanarak nasıl bir hesap makinesi servisi oluşturacağınızı açıklar. Şunları anlayacaksınız:

- Yapay zekanın araç olarak kullanabileceği bir servis oluşturma
- MCP servisleriyle doğrudan iletişim kurma ayarlamaları
- Yapay zeka modellerinin hangi araçları otomatik seçebileceği
- Doğrudan protokol çağrıları ile yapay zekâ destekli etkileşimler arasındaki fark

## Ön Koşullar

Başlamadan önce şunlara sahip olduğunuzdan emin olun:
- Java 21 veya daha yüksek sürümü yüklü
- Bağımlılık yönetimi için Maven
- Bir Azure AI Foundry model dağıtımı (sağlamak için `azd up` komutunu kullanın — bkz. [Bölüm 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) yüklü ve `az login` ile oturum açılmış (anahtarsız kimlik doğrulama)
- Java ve Spring Boot hakkında temel bilgi

## Proje Yapısını Anlamak

Hesap makinesi projesinde birkaç önemli dosya bulunmaktadır:

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

## Temel Bileşenler Açıklaması

### 1. Ana Uygulama

**Dosya:** `McpServerApplication.java`

Bu, hesap makinesi servisimizin giriş noktasıdır. Standart bir Spring Boot uygulamasıdır ancak bir özel ek içerir:

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

**Yaptıkları:**
- 8080 portunda Spring Boot web sunucusunu başlatır
- Hesap makinesi yöntemlerimizi MCP araçları olarak kullanılabilir hale getiren `ToolCallbackProvider` oluşturur
- `@Bean` anotasyonu, Spring’in bunu diğer bileşenlerin kullanabileceği bir bileşen olarak yönetmesini sağlar

### 2. Hesap Makinesi Servisi

**Dosya:** `CalculatorService.java`

Bütün matematik işlemleri burada gerçekleşir. Her yöntem `@Tool` ile işaretlenerek MCP üzerinden erişilebilir olur:

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
    
    // Daha fazla hesap makinesi işlemi...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Öne çıkan özellikler:**

1. **`@Tool` Anotasyonu:** Dış istemcilerin bu metodu çağırabileceğini MCP’ye bildirir
2. **Açık Açıklamalar:** Her aracın, yapay zekanın ne zaman kullanacağını anlamasına yardımcı olan açıklamaları vardır
3. **Tutarlı Dönüş Formatı:** Tüm işlemler "5.00 + 3.00 = 8.00" gibi insan tarafından okunabilir stringler döndürür
4. **Hata Yönetimi:** Sıfıra bölme ve negatif karekök hatalarını mesajlarla bildirir

**Mevcut İşlemler:**
- `add(a, b)` - İki sayıyı toplar
- `subtract(a, b)` - İkinci sayıdan birincisini çıkarır
- `multiply(a, b)` - İki sayıyı çarpar
- `divide(a, b)` - Birinci sayıyı ikinciye böler (sıfır kontrolü ile)
- `power(base, exponent)` - Taban sayıyı üssüne yükseltir
- `squareRoot(number)` - Karekök hesaplar (negatif kontrolü ile)
- `modulus(a, b)` - Bölümünden kalan değeri verir
- `absolute(number)` - Mutlak değeri verir
- `help()` - Tüm işlemler hakkında bilgi döner

### 3. Doğrudan MCP İstemcisi

**Dosya:** `SDKClient.java`

Bu istemci, yapay zeka kullanmadan MCP sunucusuna doğrudan konuşur. Belirli hesap makinesi fonksiyonlarını manuel olarak çağırır:

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
        
        // Mevcut araçları listele
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Belirli hesap makinesi fonksiyonlarını çağır
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

**Yaptıkları:**
1. Builder deseni kullanarak `http://localhost:8080` adresindeki hesap makinesi sunucusuna bağlanır
2. Tüm mevcut araçları listeler (hesap makinesi fonksiyonlarımız)
3. Belirli fonksiyonları tam parametrelerle çağırır
4. Sonuçları doğrudan yazdırır

**Not:** Bu örnek, `WebFluxSseClientTransport` için builder desenini tanıtan Spring AI 1.1.0-SNAPSHOT bağımlılığını kullanır. Daha eski stabil sürümler için doğrudan yapıcı kullanılmalıdır.

**Ne zaman kullanmalı:** Hangi hesaplamayı yapmak istediğinizi tam bildiğinizde ve bunu programatik olarak çağırmak istediğinizde.

### 4. Yapay Zeka Destekli İstemci

**Dosya:** `LangChain4jClient.java`

Bu istemci, hesap makinesi araçlarını otomatik seçebilen GPT-4o-mini modelini kullanır:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // AI modelini ayarlayın (Azure AI Foundry, Microsoft Entra ID aracılığıyla anahtarsız kimlik doğrulama)
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

        // Hesap makinesi MCP sunucumuza bağlanın
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // AI'nın ne yaptığını gösterir
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // AI'a hesap makinesi araçlarımıza erişim verin
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Hesap makinemizi kullanabilen bir AI botu oluşturun
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Artık AI'ya doğal dilde hesaplama yapmasını isteyebiliriz
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Yaptıkları:**
1. Anahtarsız kimlik doğrulama (Microsoft Entra ID) kullanarak yapay zeka model bağlantısı oluşturur
2. Yapay zekayı MCP hesap makinesi sunucusuna bağlar
3. Yapay zekaya tüm hesap makinesi araçlarına erişim sağlar
4. "24.5 ile 17.3'ün toplamını hesapla" gibi doğal dil isteklerine izin verir

**Yapay zeka otomatik olarak:**
- Toplama istediğinizi anlar
- `add` aracını seçer
- `add(24.5, 17.3)` çağrısı yapar
- Sonucu doğal bir yanıt olarak döner

## Örneklerin Çalıştırılması

### Adım 1: Hesap Makinesi Sunucusunu Başlatın

Öncelikle oturum açın ve Azure AI Foundry uç noktanızı ayarlayın (yapay zeka istemcisi için gerekli — anahtarsız kimlik doğrulama, API anahtarı yok):

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

Sunucuyu başlatın:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

Sunucu `http://localhost:8080` adresinde başlayacaktır. Şu görüntülenir:
```
Started McpServerApplication in X.XXX seconds
```

### Adım 2: Doğrudan İstemci ile Test Edin

Sunucu çalışırken **YENİ** bir terminal açın ve doğrudan MCP istemcisini çalıştırın:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Şöyle bir çıktı görürsünüz:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Adım 3: Yapay Zeka İstemcisi ile Test Edin

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Yapay zekanın araçları otomatik kullanışını görürsünüz:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Adım 4: MCP Sunucusunu Kapatın

Testiniz bittiğinde, yapay zeka istemcisini terminalinde `Ctrl+C` ile durdurabilirsiniz. MCP sunucusu ise siz durdurana kadar çalışmaya devam edecektir.
Sunucuyu durdurmak için çalıştığı terminalde `Ctrl+C` tuşlarına basın.

## Hepsi Nasıl Birlikte Çalışır

Yapay zekadan "5 + 3 nedir?" diye sordunuz diyelim:

1. **Siz** yapay zekaya doğal dilde soru sorarsınız
2. **Yapay zeka** isteğinizi analiz eder ve toplama istediğinizi anlar
3. **Yapay zeka** MCP sunucusuna çağrı yapar: `add(5.0, 3.0)`
4. **Hesap Makinesi Servisi** işlemi yapar: `5.0 + 3.0 = 8.0`
5. **Hesap Makinesi Servisi** sonucu döner: `"5.00 + 3.00 = 8.00"`
6. **Yapay zeka** sonucu alır ve doğal bir yanıt hazırlar
7. **Siz** "5 ile 3'ün toplamı 8'dir" yanıtını alırsınız

## Sonraki Adımlar

Daha fazla örnek için, bkz. [Bölüm 04: Pratik örnekler](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Feragatname**:
Bu belge, AI çeviri hizmeti [Co-op Translator](https://github.com/Azure/co-op-translator) kullanılarak çevrilmiştir. Doğruluk için çaba sarf etsek de, otomatik çevirilerin hata veya yanlışlık içerebileceğini lütfen unutmayınız. Orijinal belge, kendi dilinde yetkili kaynak olarak kabul edilmelidir. Kritik bilgiler için profesyonel insan çevirisi önerilir. Bu çevirinin kullanımı sonucu ortaya çıkabilecek yanlış anlamalardan veya yanlış yorumlamalardan sorumlu değiliz.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->