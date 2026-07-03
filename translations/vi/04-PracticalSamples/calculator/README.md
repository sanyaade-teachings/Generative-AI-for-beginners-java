# Hướng Dẫn MCP Calculator cho Người Mới Bắt Đầu

## Mục Lục

- [Bạn Sẽ Học Gì](#bạn-sẽ-học-gì)
- [Yêu Cầu Tiên Quyết](#yêu-cầu-tiên-quyết)
- [Hiểu Về Cấu Trúc Dự Án](#hiểu-về-cấu-trúc-dự-án)
- [Giải Thích Các Thành Phần Chính](#giải-thích-các-thành-phần-chính)
  - [1. Ứng Dụng Chính](#1-ứng-dụng-chính)
  - [2. Dịch Vụ Máy Tính](#2-dịch-vụ-máy-tính)
  - [3. Khách Hàng MCP Trực Tiếp](#3-khách-hàng-mcp-trực-tiếp)
  - [4. Khách Hàng Được Hỗ Trợ AI](#4-khách-hàng-được-hỗ-trợ-ai)
- [Chạy Ví Dụ](#chạy-ví-dụ)
- [Cách Các Thành Phần Hoạt Động Cùng Nhau](#cách-các-thành-phần-hoạt-động-cùng-nhau)
- [Bước Tiếp Theo](#bước-tiếp-theo)

## Bạn Sẽ Học Gì

Hướng dẫn này giải thích cách xây dựng một dịch vụ máy tính sử dụng Giao Thức Ngữ Cảnh Mô Hình (Model Context Protocol - MCP). Bạn sẽ hiểu:

- Cách tạo dịch vụ mà AI có thể sử dụng như một công cụ
- Cách thiết lập giao tiếp trực tiếp với các dịch vụ MCP
- Cách AI tự động chọn công cụ cần dùng
- Sự khác biệt giữa gọi giao thức trực tiếp và tương tác được trợ giúp bởi AI

## Yêu Cầu Tiên Quyết

Trước khi bắt đầu, đảm bảo bạn có:
- Đã cài Java 21 trở lên
- Maven để quản lý thư viện phụ thuộc
- Triển khai mô hình Azure AI Foundry (cấu hình với `azd up` — xem [Chương 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), đăng nhập bằng `az login` (xác thực không cần khóa)
- Hiểu biết cơ bản về Java và Spring Boot

## Hiểu Về Cấu Trúc Dự Án

Dự án máy tính có một số tập tin quan trọng:

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

## Giải Thích Các Thành Phần Chính

### 1. Ứng Dụng Chính

**Tập tin:** `McpServerApplication.java`

Đây là điểm khởi đầu của dịch vụ máy tính. Nó là một ứng dụng Spring Boot tiêu chuẩn với một điểm đặc biệt:

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

**Việc này làm:**
- Khởi động server web Spring Boot trên cổng 8080
- Tạo `ToolCallbackProvider` để làm cho các phương thức máy tính của ta trở thành công cụ MCP
- Chú thích `@Bean` để Spring quản lý thành phần này và các phần khác có thể sử dụng

### 2. Dịch Vụ Máy Tính

**Tập tin:** `CalculatorService.java`

Nơi diễn ra tất cả các phép tính. Mỗi phương thức được đánh dấu với `@Tool` để có thể gọi qua MCP:

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
    
    // Thêm các phép tính của máy tính...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Tính năng chính:**

1. **Chú thích `@Tool`**: Thông báo MCP rằng phương thức này có thể được gọi bởi client bên ngoài
2. **Mô tả rõ ràng**: Mỗi công cụ có mô tả giúp các mô hình AI hiểu khi nào dùng nó
3. **Định dạng trả về nhất quán**: Tất cả trả về dạng chuỗi dễ đọc như "5.00 + 3.00 = 8.00"
4. **Xử lý lỗi**: Chia cho 0 hay căn bậc hai âm trả về thông báo lỗi

**Các phép tính hỗ trợ:**
- `add(a, b)` - Cộng hai số
- `subtract(a, b)` - Trừ số thứ hai khỏi số thứ nhất
- `multiply(a, b)` - Nhân hai số
- `divide(a, b)` - Chia số thứ nhất cho số thứ hai (có kiểm tra chia 0)
- `power(base, exponent)` - Lũy thừa cơ số với số mũ
- `squareRoot(number)` - Tính căn bậc hai (kiểm tra số âm)
- `modulus(a, b)` - Lấy phần dư phép chia
- `absolute(number)` - Lấy giá trị tuyệt đối
- `help()` - Trả về thông tin về tất cả các phép tính

### 3. Khách Hàng MCP Trực Tiếp

**Tập tin:** `SDKClient.java`

Client này giao tiếp trực tiếp với server MCP mà không dùng AI. Nó gọi các hàm máy tính cụ thể một cách thủ công:

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
        
        // Liệt kê các công cụ có sẵn
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Gọi các hàm máy tính cụ thể
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

**Việc này làm:**
1. **Kết nối** với server máy tính tại `http://localhost:8080` sử dụng builder pattern
2. **Liệt kê** tất cả các công cụ sẵn có (hàm máy tính)
3. **Gọi** các hàm cụ thể với tham số chính xác
4. **In** kết quả trực tiếp

**Lưu ý:** Ví dụ này dùng phụ thuộc Spring AI 1.1.0-SNAPSHOT, phiên bản này thêm builder pattern cho `WebFluxSseClientTransport`. Nếu bạn dùng phiên bản ổn định cũ, có thể phải dùng constructor trực tiếp.

**Khi nào dùng:** Khi bạn biết chính xác phép tính cần chạy và muốn gọi thủ công bằng mã.

### 4. Khách Hàng Được Hỗ Trợ AI

**Tập tin:** `LangChain4jClient.java`

Client này dùng mô hình AI (GPT-4o-mini) có thể tự động quyết định dùng công cụ máy tính nào:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Thiết lập mô hình AI (Azure AI Foundry, xác thực không khóa qua Microsoft Entra ID)
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

        // Kết nối với máy chủ MCP máy tính của chúng tôi
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Hiển thị những gì AI đang thực hiện
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Cấp quyền cho AI truy cập các công cụ máy tính của chúng tôi
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Tạo một bot AI có thể sử dụng máy tính của chúng tôi
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Bây giờ chúng ta có thể yêu cầu AI thực hiện các phép tính bằng ngôn ngữ tự nhiên
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Việc này làm:**
1. **Tạo** kết nối mô hình AI dùng xác thực không cần khóa (Microsoft Entra ID)
2. **Liên kết** AI với server MCP máy tính của ta
3. **Cho phép** AI truy cập vào tất cả các công cụ máy tính
4. **Cho phép** các yêu cầu ngôn ngữ tự nhiên như "Tính tổng của 24.5 và 17.3"

**AI tự động:**
- Hiểu rằng bạn muốn cộng các số
- Chọn công cụ `add`
- Gọi `add(24.5, 17.3)`
- Trả kết quả một câu trả lời tự nhiên

## Chạy Ví Dụ

### Bước 1: Khởi Động Server Máy Tính

Trước tiên, đăng nhập và đặt điểm cuối Azure AI Foundry (cần cho client AI — không cần khóa, không cần API key):

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

Khởi động server:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

Server sẽ chạy tại `http://localhost:8080`. Bạn sẽ thấy:
```
Started McpServerApplication in X.XXX seconds
```

### Bước 2: Thử Với Client Trực Tiếp

Mở một cửa sổ terminal **MỚI** trong khi server vẫn chạy, chạy client MCP trực tiếp:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Bạn sẽ thấy kết quả như:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Bước 3: Thử Với Client AI

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Bạn sẽ thấy AI tự động sử dụng công cụ:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Bước 4: Tắt Server MCP

Khi xong, bạn có thể dừng client AI bằng `Ctrl+C` trong terminal của nó. Server MCP sẽ tiếp tục chạy cho đến khi bạn dừng nó.
Để dừng server, nhấn `Ctrl+C` trong terminal nơi server đang chạy.

## Cách Các Thành Phần Hoạt Động Cùng Nhau

Dưới đây là quy trình đầy đủ khi bạn hỏi AI "5 + 3 bằng bao nhiêu?":

1. **Bạn** hỏi AI bằng ngôn ngữ tự nhiên
2. **AI** phân tích yêu cầu và hiểu bạn muốn phép cộng
3. **AI** gọi server MCP: `add(5.0, 3.0)`
4. **Dịch vụ Máy Tính** thực hiện: `5.0 + 3.0 = 8.0`
5. **Dịch vụ Máy Tính** trả về: `"5.00 + 3.00 = 8.00"`
6. **AI** nhận kết quả và chuyển thành câu trả lời tự nhiên
7. **Bạn** nhận được: "Tổng của 5 và 3 là 8"

## Bước Tiếp Theo

Để xem thêm ví dụ, xem [Chương 04: Ví Dụ Thực Tế](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Tuyên bố miễn trừ trách nhiệm**:
Tài liệu này đã được dịch bằng dịch vụ dịch thuật AI [Co-op Translator](https://github.com/Azure/co-op-translator). Mặc dù chúng tôi cố gắng đảm bảo độ chính xác, xin lưu ý rằng bản dịch tự động có thể chứa lỗi hoặc sai sót. Tài liệu gốc bằng ngôn ngữ gốc nên được coi là nguồn tin chính thức. Đối với thông tin quan trọng, nên sử dụng dịch vụ dịch thuật chuyên nghiệp bởi con người. Chúng tôi không chịu trách nhiệm về bất kỳ hiểu lầm hoặc giải thích sai nào phát sinh từ việc sử dụng bản dịch này.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->