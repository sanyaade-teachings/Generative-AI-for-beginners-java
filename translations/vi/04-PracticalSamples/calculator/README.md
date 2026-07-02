# Hướng Dẫn Máy Tính MCP Cho Người Mới Bắt Đầu

## Mục Lục

- [Bạn Sẽ Học Được Gì](#bạn-sẽ-học-được-gì)
- [Yêu Cầu Trước Khi Bắt Đầu](#yêu-cầu-trước-khi-bắt-đầu)
- [Hiểu Cấu Trúc Dự Án](#hiểu-cấu-trúc-dự-án)
- [Giải Thích Các Thành Phần Cốt Lõi](#giải-thích-các-thành-phần-cốt-lõi)
  - [1. Ứng Dụng Chính](#1-ứng-dụng-chính)
  - [2. Dịch Vụ Máy Tính](#2-dịch-vụ-máy-tính)
  - [3. Khách Hàng MCP Trực Tiếp](#3-khách-hàng-mcp-trực-tiếp)
  - [4. Khách Hàng Sử Dụng AI](#4-khách-hàng-sử-dụng-ai)
- [Chạy Các Ví Dụ](#chạy-các-ví-dụ)
- [Cách Tất Cả Hoạt Động Cùng Nhau](#cách-tất-cả-hoạt-động-cùng-nhau)
- [Bước Tiếp Theo](#bước-tiếp-theo)

## Bạn Sẽ Học Được Gì

Hướng dẫn này giải thích cách xây dựng một dịch vụ máy tính sử dụng Giao Thức Ngữ Cảnh Mô Hình (Model Context Protocol - MCP). Bạn sẽ hiểu:

- Cách tạo một dịch vụ mà AI có thể dùng như một công cụ
- Cách thiết lập giao tiếp trực tiếp với các dịch vụ MCP
- Cách các mô hình AI có thể tự động chọn các công cụ để sử dụng
- Sự khác biệt giữa các cuộc gọi giao thức trực tiếp và các tương tác có trợ giúp AI

## Yêu Cầu Trước Khi Bắt Đầu

Trước khi bắt đầu, đảm bảo bạn đã có:
- Java 21 hoặc cao hơn được cài đặt
- Maven để quản lý phụ thuộc
- Một triển khai mô hình Azure AI Foundry (cung cấp bằng lệnh `azd up` — xem [Chương 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), đã đăng nhập bằng `az login` (xác thực không dùng key)
- Kiến thức cơ bản về Java và Spring Boot

## Hiểu Cấu Trúc Dự Án

Dự án máy tính có một số file quan trọng:

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

## Giải Thích Các Thành Phần Cốt Lõi

### 1. Ứng Dụng Chính

**File:** `McpServerApplication.java`

Đây là điểm vào của dịch vụ máy tính của chúng ta. Đây là ứng dụng Spring Boot tiêu chuẩn với một bổ sung đặc biệt:

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

**Điều này làm gì:**
- Khởi động một máy chủ web Spring Boot trên cổng 8080
- Tạo một `ToolCallbackProvider` làm cho các phương thức máy tính của chúng ta có thể truy cập như các công cụ MCP
- Chú thích `@Bean` bảo Spring quản lý thành phần này để các phần khác có thể sử dụng

### 2. Dịch Vụ Máy Tính

**File:** `CalculatorService.java`

Đây là nơi diễn ra tất cả các phép toán. Mỗi phương thức được đánh dấu với `@Tool` để có thể sử dụng qua MCP:

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
    
    // Thêm các phép toán máy tính...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Các tính năng chính:**

1. **Chú thích `@Tool`**: Nói với MCP rằng phương thức này có thể được gọi bởi các khách hàng bên ngoài
2. **Mô tả rõ ràng**: Mỗi công cụ có mô tả giúp các mô hình AI hiểu khi nào nên sử dụng
3. **Định dạng trả về nhất quán**: Tất cả các phép toán trả về chuỗi dễ đọc như "5.00 + 3.00 = 8.00"
4. **Xử lý lỗi**: Chia cho zero và căn bậc hai âm trả về thông báo lỗi

**Các phép toán có sẵn:**
- `add(a, b)` - Cộng hai số
- `subtract(a, b)` - Trừ số thứ hai khỏi số thứ nhất
- `multiply(a, b)` - Nhân hai số
- `divide(a, b)` - Chia số thứ nhất cho số thứ hai (kiểm tra chia cho zero)
- `power(base, exponent)` - Lũy thừa cơ số với số mũ
- `squareRoot(number)` - Tính căn bậc hai (kiểm tra số âm)
- `modulus(a, b)` - Trả về phần dư phép chia
- `absolute(number)` - Trả về giá trị tuyệt đối
- `help()` - Trả về thông tin về tất cả các phép toán

### 3. Khách Hàng MCP Trực Tiếp

**File:** `SDKClient.java`

Khách hàng này giao tiếp trực tiếp với máy chủ MCP mà không dùng AI. Nó gọi thủ công các hàm máy tính cụ thể:

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

**Điều này làm gì:**
1. **Kết nối** với máy chủ máy tính tại `http://localhost:8080` sử dụng builder pattern
2. **Liệt kê** tất cả các công cụ có sẵn (các hàm máy tính của chúng ta)
3. **Gọi** các hàm cụ thể với đối số chính xác
4. **In** kết quả trực tiếp

**Lưu ý:** Ví dụ này dùng phụ thuộc Spring AI 1.1.0-SNAPSHOT, bổ sung builder pattern cho `WebFluxSseClientTransport`. Nếu bạn dùng phiên bản cũ hơn ổn định, có thể cần dùng constructor trực tiếp.

**Khi nào sử dụng:** Khi bạn biết chính xác phép tính muốn thực hiện và muốn gọi nó bằng lập trình.

### 4. Khách Hàng Sử Dụng AI

**File:** `LangChain4jClient.java`

Khách hàng này dùng mô hình AI (GPT-4o-mini) có thể tự động quyết định công cụ máy tính nào sẽ dùng:

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

        // Kết nối với máy chủ máy tính MCP của chúng tôi
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Hiển thị những gì AI đang làm
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Cấp quyền truy cập cho AI vào các công cụ máy tính của chúng tôi
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

**Điều này làm gì:**
1. **Tạo** kết nối mô hình AI sử dụng token GitHub của bạn
2. **Kết nối** AI với máy chủ MCP của máy tính
3. **Cung cấp** cho AI quyền truy cập tất cả các công cụ máy tính của chúng ta
4. **Cho phép** các yêu cầu ngôn ngữ tự nhiên như "Tính tổng của 24.5 và 17.3"

**AI tự động:**
- Hiểu bạn muốn cộng số
- Chọn công cụ `add`
- Gọi `add(24.5, 17.3)`
- Trả kết quả dưới dạng câu trả lời tự nhiên

## Chạy Các Ví Dụ

### Bước 1: Khởi Động Máy Chủ Máy Tính

Đầu tiên, đăng nhập và thiết lập endpoint Azure AI Foundry của bạn (cần cho khách hàng AI — xác thực không cần key, không cần API key):

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

Khởi động máy chủ:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

Máy chủ sẽ chạy trên `http://localhost:8080`. Bạn sẽ thấy:
```
Started McpServerApplication in X.XXX seconds
```

### Bước 2: Thử Với Khách Hàng Trực Tiếp

Mở một terminal **MỚI** trong khi máy chủ vẫn chạy, chạy khách hàng MCP trực tiếp:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Bạn sẽ thấy đầu ra như:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Bước 3: Thử Với Khách Hàng AI

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Bạn sẽ thấy AI tự động sử dụng các công cụ:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Bước 4: Đóng Máy Chủ MCP

Khi bạn kết thúc thử nghiệm, có thể dừng khách hàng AI bằng cách nhấn `Ctrl+C` trong terminal của nó. Máy chủ MCP sẽ tiếp tục chạy cho đến khi bạn dừng nó.
Để dừng máy chủ, nhấn `Ctrl+C` trong terminal đang chạy máy chủ.

## Cách Tất Cả Hoạt Động Cùng Nhau

Dưới đây là luồng hoàn chỉnh khi bạn hỏi AI "5 + 3 bằng bao nhiêu?":

1. **Bạn** hỏi AI bằng ngôn ngữ tự nhiên
2. **AI** phân tích yêu cầu và nhận ra bạn muốn cộng
3. **AI** gọi máy chủ MCP: `add(5.0, 3.0)`
4. **Dịch Vụ Máy Tính** thực hiện phép tính: `5.0 + 3.0 = 8.0`
5. **Dịch Vụ Máy Tính** trả về: `"5.00 + 3.00 = 8.00"`
6. **AI** nhận kết quả và định dạng câu trả lời tự nhiên
7. **Bạn** nhận được: "Tổng của 5 và 3 là 8"

## Bước Tiếp Theo

Để xem thêm các ví dụ, hãy xem [Chương 04: Mẫu thực tế](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Tuyên bố miễn trừ trách nhiệm**:
Tài liệu này đã được dịch bằng dịch vụ dịch thuật AI [Co-op Translator](https://github.com/Azure/co-op-translator). Mặc dù chúng tôi cố gắng đảm bảo độ chính xác, xin lưu ý rằng bản dịch tự động có thể chứa lỗi hoặc sai sót. Tài liệu gốc bằng ngôn ngữ gốc nên được coi là nguồn tin chính thức. Đối với thông tin quan trọng, nên sử dụng dịch vụ dịch thuật chuyên nghiệp bởi con người. Chúng tôi không chịu trách nhiệm về bất kỳ hiểu lầm hoặc giải thích sai nào phát sinh từ việc sử dụng bản dịch này.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->