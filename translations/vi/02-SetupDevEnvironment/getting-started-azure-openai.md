# Thiết Lập Môi Trường Phát Triển cho Azure AI Foundry

> Hướng dẫn này thiết lập các mô hình **Azure AI Foundry** cho các ứng dụng AI Java trong khóa học này, sử dụng xác thực **không cần khóa** (Microsoft Entra ID) — không cần quản lý khóa API. Mới dùng công cụ? Bắt đầu với [hướng dẫn môi trường phát triển](./README.md).

Hướng dẫn này thiết lập các mô hình **Azure AI Foundry** cho các ứng dụng AI Java trong khóa học này. Bạn có hai lựa chọn:

- **Lựa chọn A — Cung cấp với `azd` + Bicep (khuyến nghị):** một lệnh triển khai tài khoản và mô hình Foundry dưới dạng mã. Không cần thao tác trên portal.
- **Lựa chọn B — Tạo tài nguyên thủ công** trong portal Azure AI Foundry.

Cả hai cách đều sử dụng **xác thực không cần khóa** (Microsoft Entra ID) — không có khóa API nào để sao chép hay lộ ra.

## Mục Lục

- [Những gì được tạo ra](#những-gì-được-tạo-ra)
- [Yêu cầu trước](#yêu-cầu-trước)
- [Lựa chọn A: Cung cấp với azd + Bicep (Khuyến nghị)](#option-a-provision-with-azd--bicep-recommended)
- [Lựa chọn B: Tạo tài nguyên thủ công](#lựa-chọn-b-tạo-tài-nguyên-thủ-công)
- [Cấu hình môi trường của bạn](#cấu-hình-môi-trường-của-bạn)
- [Kiểm tra thiết lập của bạn](#kiểm-tra-thiết-lập-của-bạn)
- [Tiếp theo là gì?](#tiếp-theo-là-gì)
- [Tài nguyên](#tài-nguyên)
- [Tài nguyên bổ sung](#tài-nguyên-bổ-sung)

## Những gì được tạo ra

Các template Bicep trong [`infra/`](../../../02-SetupDevEnvironment/infra) triển khai:

- Một tài khoản **Azure AI Foundry** (`Microsoft.CognitiveServices/accounts`, loại `AIServices`) với một dự án
- Một bản triển khai **chat** — `gpt-4o-mini`
- Một bản triển khai **embedding** — `text-embedding-3-small` (sẽ dùng trong các chương sau)
- Một **phân quyền không cần khóa** (`Cognitive Services OpenAI User`) để bạn đăng nhập với `az login` thay vì quản lý khóa

## Yêu cầu trước

- Một [đăng ký Azure](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) và [Maven 3.9+](https://maven.apache.org/download.cgi)

## Lựa chọn A: Cung cấp với azd + Bicep (Khuyến nghị)

Từ thư mục `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# Đăng nhập (cả hai công cụ)
azd auth login
az login

# Cung cấp tài khoản Foundry + triển khai mô hình
azd up
```

`azd` sẽ yêu cầu bạn nhập **tên môi trường** (ví dụ `genai-java`) và **vùng**. Chọn vùng có `gpt-4o-mini` và `text-embedding-3-small` sẵn có — ví dụ `eastus2` hoặc `swedencentral`.

Khi quá trình cung cấp hoàn thành, azd sẽ:

1. Triển khai tất cả nội dung được định nghĩa trong [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep).
2. Chạy một hook sau cung cấp để ghi file [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) với endpoint và tên triển khai của bạn (không có thông tin bí mật).

> **Mẹo:** Chạy lại `azd up` bất cứ khi nào để áp dụng thay đổi. Chạy `azd down` để xóa mọi thứ và ngừng phát sinh chi phí.

Xem các thiết lập đã tạo:

```bash
azd env get-values
```

Bây giờ chuyển sang phần [Kiểm tra thiết lập của bạn](#kiểm-tra-thiết-lập-của-bạn).

## Lựa chọn B: Tạo tài nguyên thủ công

Muốn dùng portal? Tạo tài nguyên bằng tay:

1. Vào [portal Azure AI Foundry](https://ai.azure.com/) và đăng nhập.
2. **Tạo một dự án** (cũng tạo một tài nguyên AI Foundry). Đặt tên như `GenAIJava`.
3. Trong dự án, mở **Models + endpoints** → **Deploy model** → **Deploy base model**.
4. Triển khai **gpt-4o-mini** (tên triển khai `gpt-4o-mini`). Làm tương tự với **text-embedding-3-small** nếu muốn dùng ví dụ embedding.
5. Từ **Overview**, sao chép **endpoint** (ví dụ `https://<resource>.openai.azure.com/`).
6. Cấp quyền truy cập không cần khóa: trên tài nguyên, mở **Access control (IAM)** → **Add role assignment** → gán **Cognitive Services OpenAI User** cho tài khoản của bạn.

> **Vẫn gặp khó khăn?** Xem [tài liệu Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Cấu hình môi trường của bạn

**Nếu bạn dùng Lựa chọn A (`azd up`)**, file thiết lập đã được tạo sẵn — không cần cấu hình gì thêm. Bỏ qua và chuyển đến [Kiểm tra thiết lập của bạn](#kiểm-tra-thiết-lập-của-bạn).

**Nếu bạn dùng Lựa chọn B (thủ công)**, tự tạo file `.env` cho ví dụ:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

Chỉnh sửa `.env` với endpoint của bạn (không cần khóa — xác thực không khóa):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Ghi chú bảo mật:** Không có khóa API nào để lưu trữ. Bạn xác thực bằng Microsoft Entra ID qua `az login` (tại máy cục bộ) hoặc managed identity (trong Azure). File `.env` chỉ chứa các cài đặt không bí mật và đã được `.gitignore` bảo vệ.

## Kiểm tra thiết lập của bạn

Đảm bảo bạn đã đăng nhập để xác thực không khóa lấy token, rồi chạy ví dụ:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # nếu bạn chưa đăng nhập
mvn clean spring-boot:run
```

Bạn sẽ thấy phản hồi từ mô hình `gpt-4o-mini`!

> **Người dùng VS Code:** Nhấn `F5` để chạy. Ứng dụng tự động tải file `.env` của bạn.

> **Ví dụ đầy đủ:** Xem [Basic Chat với Azure AI Foundry example](./examples/basic-chat-azure/README.md) để biết chi tiết và khắc phục sự cố.

## Tiếp theo là gì?

**Thiết lập hoàn tất!** Bạn đã có:
- Azure AI Foundry với `gpt-4o-mini` và `text-embedding-3-small` đã được triển khai
- Xác thực không cần khóa (Microsoft Entra ID) — không cần quản lý khóa
- File `.env` cục bộ chứa endpoint và tên triển khai
- Môi trường phát triển Java sẵn sàng sử dụng

**Tiếp tục sang** [Chương 3: Kỹ Thuật AI Sinh Tạo Cốt Lõi](../03-CoreGenerativeAITechniques/README.md) để bắt đầu xây dựng ứng dụng AI!

## Tài nguyên

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Xác thực không cần khóa với Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Tài liệu Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)
- [Tài liệu Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Tài nguyên bổ sung

- [Tải VS Code](https://code.visualstudio.com/Download)
- [Tải Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Cấu hình Dev Container](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Tuyên bố miễn trừ trách nhiệm**:
Tài liệu này đã được dịch bằng dịch vụ dịch thuật AI [Co-op Translator](https://github.com/Azure/co-op-translator). Mặc dù chúng tôi cố gắng đảm bảo độ chính xác, xin lưu ý rằng bản dịch tự động có thể chứa lỗi hoặc sai sót. Tài liệu gốc bằng ngôn ngữ gốc nên được coi là nguồn tin chính thức. Đối với thông tin quan trọng, nên sử dụng dịch vụ dịch thuật chuyên nghiệp bởi con người. Chúng tôi không chịu trách nhiệm về bất kỳ hiểu lầm hoặc giải thích sai nào phát sinh từ việc sử dụng bản dịch này.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->