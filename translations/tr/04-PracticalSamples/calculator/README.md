# Yeni Başlayanlar için MCP Hesaplayıcı Eğitimi

## İçindekiler

- [Neler Öğreneceksiniz](#neler-öğreneceksiniz)
- [Ön Koşullar](#ön-koşullar)
- [Proje Yapısını Anlamak](#proje-yapısını-anlamak)
- [Temel Bileşenlerin Açıklaması](#temel-bileşenlerin-açıklaması)
  - [1. Ana Uygulama](#1-ana-uygulama)
  - [2. Hesaplayıcı Servisi](#2-hesaplayıcı-servisi)
  - [3. Doğrudan MCP İstemcisi](#3-doğrudan-mcp-istemcisi)
  - [4. Yapay Zekâ Destekli İstemci](#4-yapay-zekâ-destekli-istemci)
- [Örneklerin Çalıştırılması](#örneklerin-çalıştırılması)
- [Hepsi Birlikte Nasıl Çalışır](#hepsi-birlikte-nasıl-çalışır)
- [Sonraki Adımlar](#sonraki-adımlar)

## Neler Öğreneceksiniz

Bu eğitim, Model Context Protocol (MCP) kullanarak bir hesaplayıcı servisi nasıl oluşturacağınızı açıklar. Şunları anlayacaksınız:

- Yapay zekanın araç olarak kullanabileceği bir servis nasıl oluşturulur
- MCP servisleriyle doğrudan iletişim nasıl kurulur
- Yapay zekâ modellerinin hangi araçları otomatik olarak seçebileceği
- Doğrudan protokol çağrıları ile yapay zekâ destekli etkileşimler arasındaki fark

## Ön Koşullar

Başlamadan önce emin olun:
- Java 21 veya üzeri yüklü
- Bağımlılık yönetimi için Maven
- Bir Azure AI Foundry model dağıtımı (tezgahda `azd up` ile oluşturun — bkz. [Bölüm 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) yüklü ve `az login` ile giriş yapılmış (anahtarsız kimlik doğrulama)
- Java ve Spring Boot hakkında temel bilgi

## Proje Yapısını Anlamak

Hesaplayıcı projesinin birkaç önemli dosyası vardır:

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

## Temel Bileşenlerin Açıklaması

### 1. Ana Uygulama

**Dosya:** `McpServerApplication.java`

Bu, hesaplayıcı servisimizin giriş noktasıdır. Standart bir Spring Boot uygulaması olup bir özel ek içerir:

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

**Yaptığı:**
- 8080 portunda bir Spring Boot web sunucusu başlatır
- Hesaplayıcı yöntemlerimizi MCP araçları olarak kullanılabilir yapan bir `ToolCallbackProvider` oluşturur
- `@Bean` notasyonu, Spring'in bunu diğer bileşenler tarafından kullanılacak bir bileşen olarak yönetmesini sağlar

### 2. Hesaplayıcı Servisi

**Dosya:** `CalculatorService.java`

Bütün matematik işlemlerinin yapıldığı yerdir. Her yöntem `@Tool` ile işaretlenmiştir ve MCP üzerinden erişilebilir:

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

1. **`@Tool` Notasyonu**: Dış istemcilerin bu yöntemi çağırabileceğini MCP'ye bildirir
2. **Anlaşılır Açıklamalar**: Her aracın, yapay zekâ modellerinin ne zaman kullanacağını anlamasına yardımcı olan açıklaması vardır
3. **Tutarlı Dönen Format**: Tüm işlemler, "5.00 + 3.00 = 8.00" gibi insan tarafından okunabilir stringler döner
4. **Hata Yönetimi**: Sıfıra bölme ve negatif karekök için hata mesajları döner

**Mevcut İşlemler:**
- `add(a, b)` - İki sayıyı toplar
- `subtract(a, b)` - İkinci sayıdan birincisini çıkarır
- `multiply(a, b)` - İki sayıyı çarpar
- `divide(a, b)` - Birinci sayıyı ikinciye böler (sıfır kontrolü ile)
- `power(base, exponent)` - Tabanı üs kuvvetine yükseltir
- `squareRoot(number)` - Kare kökünü hesaplar (negatif kontrolü ile)
- `modulus(a, b)` - Bölümden kalanını verir
- `absolute(number)` - Mutlak değeri verir
- `help()` - Tüm işlemler hakkında bilgi verir

### 3. Doğrudan MCP İstemcisi

**Dosya:** `SDKClient.java`

Bu istemci, yapay zekâ kullanmadan doğrudan MCP sunucusuyla konuşur. Belirli hesaplayıcı fonksiyonlarını manuel olarak çağırır:

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

**Yaptığı:**
1. `http://localhost:8080` adresindeki hesaplayıcı sunucusuna builder deseni ile bağlanır
2. Tüm kullanılabilir araçları (hesaplayıcı fonksiyonlarımız) listeler
3. Belirli fonksiyonları tam parametrelerle çağırır
4. Sonuçları doğrudan yazdırır

**Not:** Bu örnek, `WebFluxSseClientTransport` için bir builder deseni getiren Spring AI 1.1.0-SNAPSHOT bağımlılığını kullanır. Daha eski kararlı sürüm kullanıyorsanız doğrudan yapıcıyı kullanmanız gerekebilir.

**Ne zaman kullanılır:** Hangi hesaplamayı yapmak istediğinizi tam olarak bildiğinizde ve programatik olarak çağırmak istediğinizde.

### 4. Yapay Zekâ Destekli İstemci

**Dosya:** `LangChain4jClient.java`

Bu istemci, hangi hesaplayıcı araçlarının kullanılacağına otomatik karar verebilen GPT-4o-mini AI modelini kullanır:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // AI modelini kurun (Azure AI Foundry, Microsoft Entra ID ile anahtarsız kimlik doğrulama)
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

        // AI'ya hesap makinesi araçlarımıza erişim verin
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Hesap makinemizi kullanabilen bir AI botu oluşturun
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Artık AI'dan doğal dilde hesaplamalar yapmasını isteyebiliriz
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Yaptığı:**
1. GitHub token'ınızla bir yapay zeka model bağlantısı oluşturur
2. Yapay zekayı hesaplayıcı MCP sunucumuza bağlar
3. Yapay zekaya tüm hesaplayıcı araçlarımıza erişim verir
4. "24.5 ile 17.3'ün toplamını hesapla" gibi doğal dilde istekleri kabul eder

**Yapay zekâ otomatik olarak:**
- Toplama işlemi istediğinizi anlar
- `add` aracını seçer
- `add(24.5, 17.3)` çağrısı yapar
- Sonucu doğal bir cevap olarak döner

## Örneklerin Çalıştırılması

### 1. Adım: Hesaplayıcı Sunucusunu Başlatın

Öncelikle oturum açın ve Azure AI Foundry uç noktanızı ayarlayın (AI istemcisi için gerekli — anahtarsız kimlik doğrulama, API anahtarı yok):

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

Sunucu `http://localhost:8080` adresinde başlatılacak. Şunu görmelisiniz:
```
Started McpServerApplication in X.XXX seconds
```

### 2. Adım: Doğrudan İstemci ile Test Edin

Sunucu çalışırken **yeni** bir terminalde doğrudan MCP istemcisini çalıştırın:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Aşağıdakine benzer çıktı göreceksiniz:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### 3. Adım: Yapay Zekâ İstemcisiyle Test Edin

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Yapay zekânın otomatik araç kullanımını göreceksiniz:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### 4. Adım: MCP Sunucusunu Kapatın

Testiniz tamamlandığında, yapay zekâ istemcisini terminalinde `Ctrl+C` ile durdurabilirsiniz. MCP sunucusu siz kapatana kadar çalışmaya devam eder.
Sunucuyu durdurmak için, çalıştığı terminalde `Ctrl+C` tuşlayın.

## Hepsi Birlikte Nasıl Çalışır

Yapay zekâya "5 + 3 nedir?" diye sorduğunuzda tam akış şu şekildedir:

1. **Siz** yapay zekâya doğal dilde sorarsınız
2. **Yapay zekâ** isteğinizi analiz edip toplama istediğinizi anlar
3. **Yapay zekâ** MCP sunucusunu çağırır: `add(5.0, 3.0)`
4. **Hesaplayıcı Servisi** işlemi yapar: `5.0 + 3.0 = 8.0`
5. **Hesaplayıcı Servisi** sonucu döner: `"5.00 + 3.00 = 8.00"`
6. **Yapay zekâ** sonucu alır ve doğal bir yanıt oluşturur
7. **Siz** alırsınız: "5 ile 3'ün toplamı 8'dir"

## Sonraki Adımlar

Daha fazla örnek için bakınız [Bölüm 04: Pratik örnekler](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Feragatname**:
Bu belge, AI çeviri hizmeti [Co-op Translator](https://github.com/Azure/co-op-translator) kullanılarak çevrilmiştir. Doğruluk için çaba sarf etsek de, otomatik çevirilerin hata veya yanlışlık içerebileceğini lütfen unutmayınız. Orijinal belge, kendi dilinde yetkili kaynak olarak kabul edilmelidir. Kritik bilgiler için profesyonel insan çevirisi önerilir. Bu çevirinin kullanımı sonucu ortaya çıkabilecek yanlış anlamalardan veya yanlış yorumlamalardan sorumlu değiliz.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->