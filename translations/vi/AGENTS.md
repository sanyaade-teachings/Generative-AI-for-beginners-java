# AGENTS.md

## Tổng quan dự án

Đây là kho lưu trữ giáo dục để học phát triển Generative AI với Java. Nó cung cấp một khóa học thực hành toàn diện về Large Language Models (LLMs), kỹ thuật prompt, embeddings, RAG (Retrieval-Augmented Generation), và Model Context Protocol (MCP).

**Công nghệ chính:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI và các SDK OpenAI

**Kiến trúc:**
- Nhiều ứng dụng Spring Boot độc lập được tổ chức theo các chương
- Dự án mẫu trình diễn các mẫu AI khác nhau
- Sẵn sàng sử dụng GitHub Codespaces với các dev container được cấu hình sẵn

## Lệnh thiết lập

### Yêu cầu trước
- Java 21 trở lên
- Maven 3.x
- Đăng ký Azure với triển khai mô hình Azure AI Foundry (được cung cấp qua `azd up`)
- Azure CLI (`az`) và Azure Developer CLI (`azd`), đăng nhập để xác thực không cần khóa

### Thiết lập môi trường

**Lựa chọn 1: GitHub Codespaces (khuyến nghị)**
```bash
# Tạo nhánh của kho lưu trữ và tạo một không gian mã từ giao diện GitHub
# Container phát triển sẽ tự động cài đặt tất cả các phụ thuộc
# Chờ khoảng ~2 phút để thiết lập môi trường
```

**Lựa chọn 2: Dev Container cục bộ**
```bash
# Sao chép kho lưu trữ
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Mở trong VS Code với phần mở rộng Dev Containers
# Mở lại trong Container khi được nhắc
```

**Lựa chọn 3: Thiết lập cục bộ**
```bash
# Cài đặt các phụ thuộc
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Xác minh cài đặt
java -version
mvn -version
```

### Cấu hình

**Thiết lập Azure AI Foundry (không dùng khóa, khuyến nghị):**
```bash
# Cung cấp tài khoản Foundry + triển khai mô hình dưới dạng mã
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd ghi examples/basic-chat-azure/.env với điểm cuối của bạn (không có khóa)
```

**Cấu hình đầu cuối thủ công:**
```bash
# Nếu bạn không sử dụng azd, hãy tự đặt endpoint (xác thực vẫn không cần khóa thông qua az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Chỉnh sửa .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Quy trình phát triển

### Cấu trúc dự án
```
/
├── 01-IntroToGenAI/              # Chapter 1: Introduction
├── 02-SetupDevEnvironment/       # Chapter 2: Environment setup
│   └── examples/                 # Working examples
├── 03-CoreGenerativeAITechniques/ # Chapter 3: Core techniques
├── 04-PracticalSamples/          # Chapter 4: Sample projects
│   ├── calculator/               # MCP service example
│   ├── foundrylocal/            # Local model integration
│   └── petstory/                # Multi-modal app
├── 05-ResponsibleGenAI/         # Chapter 5: Responsible AI
└── translations/                # Multi-language support
```

### Chạy ứng dụng

**Chạy ứng dụng Spring Boot:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Xây dựng dự án:**
```bash
cd [project-directory]
mvn clean install
```

**Khởi động MCP Calculator Server:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# Máy chủ chạy trên http://localhost:8080
```

**Chạy ví dụ Client:**
```bash
# Sau khi khởi động máy chủ trong một cửa sổ terminal khác
cd 04-PracticalSamples/calculator

# Khách hàng MCP trực tiếp
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# Khách hàng sử dụng AI (yêu cầu AZURE_OPENAI_ENDPOINT + az login)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Bot tương tác
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Tải lại nóng
Spring Boot DevTools được tích hợp trong các dự án hỗ trợ tải lại nóng:
```bash
# Thay đổi trong các tệp Java sẽ tự động tải lại khi lưu
mvn spring-boot:run
```

## Hướng dẫn kiểm thử

### Chạy kiểm thử

**Chạy tất cả các kiểm thử trong một dự án:**
```bash
cd [project-directory]
mvn test
```

**Chạy kiểm thử với đầu ra chi tiết:**
```bash
mvn test -X
```

**Chạy một lớp kiểm thử cụ thể:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Cấu trúc kiểm thử
- Các tập tin kiểm thử sử dụng JUnit 5 (Jupiter)
- Các lớp kiểm thử nằm trong `src/test/java/`
- Ví dụ client trong dự án máy tính nằm trong `src/test/java/com/microsoft/mcp/sample/client/`

### Kiểm thử thủ công
Nhiều ví dụ là các ứng dụng tương tác yêu cầu kiểm thử thủ công:

1. Khởi động ứng dụng với `mvn spring-boot:run`
2. Kiểm thử các đầu cuối hoặc tương tác với CLI
3. Xác nhận hành vi mong đợi đúng theo tài liệu trong README.md mỗi dự án

### Kiểm thử với Azure AI Foundry
- Đăng nhập với `az login` trước khi chạy ví dụ (xác thực không cần khóa)
- Đảm bảo tài khoản của bạn có vai trò Cognitive Services OpenAI User trên tài nguyên
- Kiểm thử lọc nội dung với ví dụ responsible AI trong Chương 5

## Hướng dẫn phong cách mã nguồn

### Quy ước Java
- **Phiên bản Java:** Java 21 với các tính năng hiện đại
- **Phong cách:** Tuân theo quy ước Java chuẩn
- **Đặt tên:**
  - Lớp: PascalCase
  - Phương thức/biến: camelCase
  - Hằng số: UPPER_SNAKE_CASE
  - Tên package: chữ thường

### Mẫu Spring Boot
- Dùng `@Service` cho logic nghiệp vụ
- Dùng `@RestController` cho các endpoint REST
- Cấu hình qua `application.yml` hoặc `application.properties`
- Ưu tiên biến môi trường hơn giá trị cố định
- Sử dụng chú thích `@Tool` cho các phương thức được MCP phơi bày

### Tổ chức file
```
src/
├── main/
│   ├── java/
│   │   └── com/microsoft/[component]/
│   │       ├── [Component]Application.java
│   │       ├── config/
│   │       ├── controller/
│   │       ├── service/
│   │       └── exception/
│   └── resources/
│       ├── application.yml
│       └── static/
└── test/
    └── java/
        └── com/microsoft/[component]/
```

### Phụ thuộc
- Quản lý bằng Maven `pom.xml`
- Spring AI BOM để quản lý phiên bản
- LangChain4j cho tích hợp AI
- Spring Boot starter parent cho các phụ thuộc Spring

### Bình luận mã nguồn
- Thêm JavaDoc cho API công khai
- Bao gồm bình luận giải thích cho các tương tác AI phức tạp
- Ghi chú rõ ràng mô tả công cụ MCP

## Xây dựng và Triển khai

### Xây dựng dự án

**Xây dựng không chạy kiểm thử:**
```bash
mvn clean install -DskipTests
```

**Xây dựng với tất cả kiểm tra:**
```bash
mvn clean install
```

**Đóng gói ứng dụng:**
```bash
mvn package
# Tạo JAR trong thư mục target/
```

### Thư mục đầu ra
- Lớp đã biên dịch: `target/classes/`
- Lớp kiểm thử: `target/test-classes/`
- File JAR: `target/*.jar`
- Artifact Maven: `target/`

### Cấu hình riêng theo môi trường

**Phát triển:**
```yaml
# application.yml (keyless - no api-key; auth via DefaultAzureCredential)
spring:
  ai:
    azure:
      openai:
        endpoint: ${AZURE_OPENAI_ENDPOINT}
        chat:
          options:
            deployment-name: ${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}
```

**Sản xuất:**
- Dùng managed identity thay vì `az login` cho xác thực không khóa
- Trỏ `AZURE_OPENAI_ENDPOINT` đến tài nguyên Foundry production của bạn
- Quản lý cấu hình qua biến môi trường hoặc Azure Key Vault

### Lưu ý triển khai
- Đây là kho giáo dục với các ứng dụng mẫu
- Không được thiết kế để triển khai sản xuất trực tiếp
- Các ví dụ minh họa các mẫu để điều chỉnh cho sử dụng sản xuất
- Xem README các dự án riêng để biết chú ý triển khai cụ thể

## Ghi chú bổ sung

### Azure AI Foundry
- **Xác thực không khóa:** kết nối với Microsoft Entra ID — không cần quản lý khóa API
- **Cung cấp dưới dạng mã:** Bicep + azd (`azd up`) tạo tài khoản và triển khai mô hình
- Cùng mã tương thích với OpenAI chạy được cục bộ (`az login`) và trên Azure (managed identity)

### Làm việc với nhiều dự án
Mỗi dự án mẫu là độc lập:
```bash
# Điều hướng đến dự án cụ thể
cd 04-PracticalSamples/[project-name]

# Mỗi cái có pom.xml riêng và có thể được xây dựng độc lập
mvn clean install
```

### Vấn đề phổ biến

**Không khớp phiên bản Java:**
```bash
# Xác minh Java 21
java -version
# Cập nhật JAVA_HOME nếu cần thiết
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Sự cố tải phụ thuộc:**
```bash
# Xóa bộ nhớ đệm Maven và thử lại
rm -rf ~/.m2/repository
mvn clean install
```

**Không tìm thấy endpoint hoặc đăng nhập:**
```bash
# Đặt điểm cuối trong phiên hiện tại và đăng nhập (không khóa)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Hoặc sử dụng tệp .env trong thư mục dự án
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Port đã được sử dụng:**
```bash
# Spring Boot sử dụng cổng 8080 theo mặc định
# Thay đổi trong application.properties:
server.port=8081
```

### Hỗ trợ đa ngôn ngữ
- Tài liệu có sẵn hơn 45 ngôn ngữ qua dịch tự động
- Bản dịch trong thư mục `translations/`
- Dịch được quản lý qua workflow GitHub Actions

### Lộ trình học
1. Bắt đầu với [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. Theo dõi các chương theo thứ tự (01 → 05)
3. Hoàn thành các ví dụ thực hành trong mỗi chương
4. Khám phá dự án mẫu trong Chương 4
5. Học thực hành responsible AI trong Chương 5

### Container phát triển
`.devcontainer/devcontainer.json` cấu hình:
- Môi trường phát triển Java 21
- Maven được cài sẵn
- Các extension Java cho VS Code
- Công cụ Spring Boot
- Tích hợp GitHub Copilot
- Hỗ trợ Docker-in-Docker
- Azure CLI

### Lưu ý hiệu năng
- Triển khai Azure AI Foundry có hạn mức token/yêu cầu theo phút
- Dùng kích thước batch phù hợp cho embeddings
- Xem xét cache cho các cuộc gọi API lặp lại
- Theo dõi sử dụng token để tối ưu chi phí

### Lưu ý an ninh
- Không bao giờ commit file `.env` (đã có trong `.gitignore`)
- Ưu tiên xác thực không khóa (Microsoft Entra ID) thay cho khóa API
- Sử dụng managed identity trong Azure; `az login` cho phát triển cục bộ
- Tuân thủ hướng dẫn responsible AI trong Chương 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Tuyên bố miễn trừ trách nhiệm**:
Tài liệu này đã được dịch bằng dịch vụ dịch thuật AI [Co-op Translator](https://github.com/Azure/co-op-translator). Mặc dù chúng tôi cố gắng đảm bảo độ chính xác, xin lưu ý rằng bản dịch tự động có thể chứa lỗi hoặc sai sót. Tài liệu gốc bằng ngôn ngữ gốc nên được coi là nguồn tin chính thức. Đối với thông tin quan trọng, nên sử dụng dịch vụ dịch thuật chuyên nghiệp bởi con người. Chúng tôi không chịu trách nhiệm về bất kỳ hiểu lầm hoặc giải thích sai nào phát sinh từ việc sử dụng bản dịch này.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->