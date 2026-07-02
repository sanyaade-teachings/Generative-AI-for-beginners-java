# Chat Cơ Bản với Azure AI Foundry - Ví Dụ Toàn Diện

Ví dụ này là một ứng dụng Spring Boot đơn giản kết nối với mô hình **Azure AI Foundry** sử dụng **xác thực không dùng khóa** (Microsoft Entra ID) và kiểm tra cài đặt của bạn. Nó sử dụng `ChatClient` của Spring AI.

## Mục Lục

- [Yêu Cầu Trước Khi Bắt Đầu](#yêu-cầu-trước-khi-bắt-đầu)
- [Bắt Đầu Nhanh](#bắt-đầu-nhanh)
- [Cách Xác Thực Hoạt Động](#cách-xác-thực-hoạt-động)
- [Chạy Ứng Dụng](#chạy-ứng-dụng)
  - [Sử Dụng Maven](#sử-dụng-maven)
  - [Sử Dụng VS Code](#sử-dụng-vs-code)
  - [Kết Quả Mong Đợi](#kết-quả-mong-đợi)
- [Tham Khảo Cấu Hình](#tham-khảo-cấu-hình)
  - [Biến Môi Trường](#biến-môi-trường)
  - [Cấu Hình Spring](#cấu-hình-spring)
- [Khắc Phục Sự Cố](#khắc-phục-sự-cố)
  - [Các Vấn Đề Thường Gặp](#các-vấn-đề-thường-gặp)
  - [Chế Độ Gỡ Lỗi](#chế-độ-gỡ-lỗi)
- [Bước Tiếp Theo](#bước-tiếp-theo)
- [Tài Nguyên](#tài-nguyên)

## Yêu Cầu Trước Khi Bắt Đầu

Trước khi chạy ví dụ này, hãy chắc chắn bạn có:

- Một tài nguyên Azure AI Foundry với một triển khai `gpt-4o-mini` — provision bằng `azd up` hoặc thủ công theo [hướng dẫn cài đặt Azure AI Foundry](../../getting-started-azure-openai.md)
- Vai trò **Cognitive Services OpenAI User** trên tài nguyên đó (các mẫu Bicep sẽ cấp quyền này cho bạn)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), đã đăng nhập với lệnh `az login`
- Java 21+ và Maven 3.9+

> **Không cần khóa API** — xác thực không dùng khóa qua Microsoft Entra ID.

## Bắt Đầu Nhanh

```bash
# 1. Điều hướng đến dự án
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Đăng nhập để xác thực không khóa có thể lấy token
az login

# 3. Cấu hình điểm cuối
#    - Nếu bạn đã chạy `azd up`, tệp .env đã được tạo sẵn (bỏ qua bước này).
#    - Nếu không, sao chép mẫu và đặt biến AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. Chạy ứng dụng
mvn spring-boot:run
```

## Cách Xác Thực Hoạt Động

Ví dụ này xác thực với **Microsoft Entra ID** — không có khóa API.

Khi chỉ đặt `spring.ai.azure.openai.endpoint` (và không có api-key), Spring AI tạo client Azure OpenAI với [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). Chứng thực đó tự động lấy token từ phiên đăng nhập `az login` cục bộ của bạn, hoặc từ managed identity khi chạy trong Azure — vậy nên cùng một đoạn mã hoạt động ở cả hai môi trường mà không cần thay đổi.

## Chạy Ứng Dụng

### Sử Dụng Maven

```bash
mvn spring-boot:run
```

### Sử Dụng VS Code

1. Mở dự án trong VS Code
2. Nhấn `F5` hoặc sử dụng bảng "Run and Debug"
3. Chọn cấu hình "Spring Boot-BasicChatApplication"

> **Lưu ý**: Cấu hình VS Code tự động tải file .env của bạn

### Kết Quả Mong Đợi

```
Starting Basic Chat with Azure OpenAI...
Environment variables loaded successfully
Connecting to Azure OpenAI...
Sending prompt: What is AI in a short sentence? Max 100 words.

AI Response:
================
AI, or Artificial Intelligence, is the simulation of human intelligence in machines programmed to think and learn like humans.
================

Success! Azure OpenAI connection is working correctly.
```

## Tham Khảo Cấu Hình

### Biến Môi Trường

| Biến | Mô Tả | Bắt Buộc | Ví Dụ |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | URL endpoint Foundry (Azure OpenAI) | Có | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Tên triển khai mô hình chat | Không | `gpt-4o-mini` (mặc định) |

> Không có biến khóa API — xác thực không dùng khóa (Microsoft Entra ID qua `az login`).

### Cấu Hình Spring

File `application.yml` cấu hình:
- **Endpoint**: `${AZURE_OPENAI_ENDPOINT}` - Lấy từ biến môi trường
- **Deployment**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - Lấy từ biến môi trường kèm giá trị dự phòng
- **Xác thực**: không dùng khóa — không đặt `api-key`, Spring AI dùng `DefaultAzureCredential`
- **Nhiệt độ**: `0.7` - Điều khiển độ sáng tạo (0.0 = xác định, 1.0 = sáng tạo)
- **Max Tokens**: `500` - Độ dài phản hồi tối đa

## Khắc Phục Sự Cố

### Các Vấn Đề Thường Gặp

<details>
<summary><strong>Lỗi: 401 / "PermissionDenied" / lỗi token</strong></summary>

- Chạy `az login` — xác thực không dùng khóa cần đăng nhập đang hoạt động để lấy token
- Xác nhận tài khoản có vai trò **Cognitive Services OpenAI User** trên tài nguyên
- Nếu bạn vừa cấp vai trò, chờ một phút để quyền được áp dụng
- Kiểm tra bạn đang ở đúng tenant/đăng ký (`az account show`)
</details>

<details>
<summary><strong>Lỗi: "The endpoint is not valid" / lỗi kết nối</strong></summary>

- Đảm bảo `AZURE_OPENAI_ENDPOINT` là URL gốc đầy đủ (ví dụ: `https://your-resource.openai.azure.com/`)
- Kiểm tra sự nhất quán của dấu gạch chéo cuối
- Xác minh endpoint khớp với tài nguyên bạn đã provision (`azd env get-values`)
</details>

<details>
<summary><strong>Lỗi: "The deployment was not found"</strong></summary>

- Xác nhận `AZURE_OPENAI_DEPLOYMENT` khớp với tên triển khai trong Azure
- Kiểm tra mô hình đã được triển khai và đang hoạt động
- Tên triển khai mặc định là `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: Biến môi trường không được nạp</strong></summary>

- Đảm bảo file `.env` nằm trong thư mục gốc dự án (cùng cấp với `pom.xml`)
- Thử chạy `mvn spring-boot:run` trong terminal tích hợp của VS Code
- Kiểm tra việc cài đặt extension Java trong VS Code đã đúng
</details>

### Chế Độ Gỡ Lỗi

Để bật ghi log chi tiết, bỏ comment các dòng này trong `application.yml`:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Bước Tiếp Theo

**Cài Đặt Hoàn Tất!** Tiếp tục hành trình học tập của bạn:

[Chương 3: Các Kỹ Thuật Core Generative AI](../../../03-CoreGenerativeAITechniques/README.md)

## Tài Nguyên

- [Tài liệu Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Xác thực không dùng khóa với Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Cổng thông tin Azure AI Foundry](https://ai.azure.com/)
- [Tài liệu Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Tuyên bố miễn trừ trách nhiệm**:
Tài liệu này đã được dịch bằng dịch vụ dịch thuật AI [Co-op Translator](https://github.com/Azure/co-op-translator). Mặc dù chúng tôi cố gắng đảm bảo độ chính xác, xin lưu ý rằng bản dịch tự động có thể chứa lỗi hoặc sai sót. Tài liệu gốc bằng ngôn ngữ gốc nên được coi là nguồn tin chính thức. Đối với thông tin quan trọng, nên sử dụng dịch vụ dịch thuật chuyên nghiệp bởi con người. Chúng tôi không chịu trách nhiệm về bất kỳ hiểu lầm hoặc giải thích sai nào phát sinh từ việc sử dụng bản dịch này.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->