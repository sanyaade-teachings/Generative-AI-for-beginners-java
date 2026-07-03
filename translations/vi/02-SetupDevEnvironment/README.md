# Thiết lập Môi trường Phát triển cho Generative AI dành cho Java

> **Bắt đầu Nhanh:** Cung cấp các mô hình AI của bạn trên **Azure AI Foundry** dưới dạng mã với Bicep + `azd` chỉ trong vài phút — xem [Hướng dẫn Thiết lập Azure AI Foundry](getting-started-azure-openai.md). Xác thực **không cần khóa** (Microsoft Entra ID), vì vậy không có khóa API để quản lý.

## Bạn Sẽ Học Được Gì

- Thiết lập môi trường phát triển Java cho ứng dụng AI
- Lựa chọn và cấu hình môi trường phát triển tùy thích (ưu tiên đám mây với Codespaces, container phát triển cục bộ, hoặc thiết lập cục bộ đầy đủ)
- Kiểm tra thiết lập bằng cách kết nối với mô hình Azure AI Foundry

## Mục Lục

- [Bạn Sẽ Học Được Gì](#bạn-sẽ-học-được-gì)
- [Giới Thiệu](#giới-thiệu)
- [Bước 1: Thiết Lập Môi Trường Phát Triển](#bước-1-thiết-lập-môi-trường-phát-triển)
  - [Lựa chọn A: GitHub Codespaces (Khuyến nghị)](#lựa-chọn-a-github-codespaces-khuyến-nghị)
  - [Lựa chọn B: Container Phát Triển Cục Bộ](#lựa-chọn-b-container-phát-triển-cục-bộ)
  - [Lựa chọn C: Sử Dụng Cài Đặt Cục Bộ Hiện Tại](#lựa-chọn-c-sử-dụng-cài-đặt-cục-bộ-hiện-tại)
- [Bước 2: Cung Cấp Azure AI Foundry](#bước-2-cung-cấp-azure-ai-foundry)
- [Bước 3: Kiểm Tra Thiết Lập](#bước-3-kiểm-tra-thiết-lập)
- [Khắc phục sự cố](#khắc-phục-sự-cố)
- [Tóm Tắt](#tóm-tắt)
- [Bước Tiếp Theo](#bước-tiếp-theo)

## Giới Thiệu

Chương này sẽ hướng dẫn bạn thiết lập môi trường phát triển. Chúng ta sẽ dùng **Azure AI Foundry** cho các mô hình trong suốt khóa học này. Bạn sẽ cung cấp các mô hình dưới dạng mã với Bicep và Azure Developer CLI (`azd`), rồi kết nối với **xác thực không cần khóa** (Microsoft Entra ID) — không cần sao chép hoặc lộ khóa API.

**Không cần thiết lập cục bộ!** Bạn có thể dùng GitHub Codespaces, cung cấp môi trường phát triển đầy đủ trong trình duyệt và cung cấp Foundry từ đó.

Chúng tôi sử dụng **Azure AI Foundry** cho khóa học này vì nó:
- **Cung cấp dưới dạng mã** — một lệnh `azd up` triển khai tài khoản và mô hình
- **Không cần khóa** — xác thực bằng đăng nhập Azure hoặc định danh được quản lý
- **Sẵn sàng cho sản xuất** — cùng mã chạy cục bộ và trên Azure
- **Linh hoạt** — thay mô hình chỉ bằng cách đổi tên triển khai, không cần đổi mã

> **Lưu ý**: Các triển khai Azure AI Foundry được tính phí theo token (trả theo mức sử dụng). Xem [hướng dẫn thiết lập Azure AI Foundry](getting-started-azure-openai.md) để biết chi tiết cung cấp, khu vực và chi phí.

## Bước 1: Thiết Lập Môi Trường Phát Triển

<a name="quick-start-cloud"></a>

Chúng tôi đã tạo một container phát triển được cấu hình sẵn để giảm thiểu thời gian thiết lập và đảm bảo bạn có tất cả công cụ cần thiết cho khóa học Generative AI cho Java này. Hãy chọn cách phát triển bạn thích:

### Lựa Chọn Thiết Lập Môi Trường:

#### Lựa chọn A: GitHub Codespaces (Khuyến nghị)

**Bắt đầu lập trình chỉ trong 2 phút - không cần thiết lập cục bộ!**

1. Fork kho lưu trữ này vào tài khoản GitHub của bạn  
   > **Lưu ý**: Nếu muốn chỉnh sửa cấu hình cơ bản, vui lòng xem [Cấu hình Dev Container](../../../.devcontainer/devcontainer.json)  
2. Nhấn **Code** → tab **Codespaces** → **...** → **Mới với tùy chọn...**  
3. Dùng mặc định – điều này sẽ chọn **Cấu hình Dev container**: **Môi trường Phát triển Generative AI Java** được tạo riêng cho khóa học này  
4. Nhấn **Tạo codespace**  
5. Đợi khoảng ~2 phút để môi trường sẵn sàng  
6. Tiếp tục tới [Bước 2: Cung cấp Azure AI Foundry](#bước-2-cung-cấp-azure-ai-foundry)

<img src="../../../translated_images/vi/codespaces.9945ded8ceb431a5.webp" alt="Ảnh chụp màn hình: submenu Codespaces" width="50%">

<img src="../../../translated_images/vi/image.833552b62eee7766.webp" alt="Ảnh chụp màn hình: Mới với tùy chọn" width="50%">

<img src="../../../translated_images/vi/codespaces-create.b44a36f728660ab7.webp" alt="Ảnh chụp màn hình: Tạo tùy chọn codespace" width="50%">

> **Lợi ích của Codespaces**:  
> - Không cần cài đặt cục bộ  
> - Hoạt động trên mọi thiết bị có trình duyệt  
> - Được cấu hình trước với tất cả công cụ và phụ thuộc  
> - Miễn phí 60 giờ mỗi tháng cho tài khoản cá nhân  
> - Môi trường đồng nhất cho tất cả học viên  

#### Lựa chọn B: Container Phát Triển Cục Bộ

**Dành cho nhà phát triển thích phát triển cục bộ với Docker**

1. Fork và clone kho lưu trữ này về máy của bạn  
   > **Lưu ý**: Nếu muốn chỉnh sửa cấu hình cơ bản, vui lòng xem [Cấu hình Dev Container](../../../.devcontainer/devcontainer.json)  
2. Cài đặt [Docker Desktop](https://www.docker.com/products/docker-desktop/) và [VS Code](https://code.visualstudio.com/)  
3. Cài đặt [extension Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) trong VS Code  
4. Mở thư mục kho lưu trữ trong VS Code  
5. Khi thấy nhắc, nhấn **Mở lại trong Container** (hoặc dùng `Ctrl+Shift+P` → "Dev Containers: Reopen in Container")  
6. Đợi container được xây dựng và khởi động  
7. Tiếp tục tới [Bước 2: Cung cấp Azure AI Foundry](#bước-2-cung-cấp-azure-ai-foundry)

<img src="../../../translated_images/vi/devcontainer.21126c9d6de64494.webp" alt="Ảnh chụp màn hình: thiết lập Dev container" width="50%">

<img src="../../../translated_images/vi/image-3.bf93d533bbc84268.webp" alt="Ảnh chụp màn hình: Xây dựng Dev container hoàn tất" width="50%">

#### Lựa chọn C: Sử Dụng Cài Đặt Cục Bộ Hiện Tại

**Dành cho nhà phát triển đã có môi trường Java**

Yêu cầu trước:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html)  
- [Maven 3.9+](https://maven.apache.org/download.cgi)  
- [VS Code](https://code.visualstudio.com) hoặc IDE bạn thích

Các bước:
1. Clone kho lưu trữ này về máy của bạn  
2. Mở dự án trong IDE của bạn  
3. Tiếp tục tới [Bước 2: Cung cấp Azure AI Foundry](#bước-2-cung-cấp-azure-ai-foundry)

> **Mẹo chuyên nghiệp**: Nếu máy bạn cấu hình thấp nhưng muốn dùng VS Code cục bộ, hãy dùng GitHub Codespaces! Bạn có thể kết nối VS Code cục bộ với Codespace trên đám mây, tận hưởng ưu điểm cả hai.

<img src="../../../translated_images/vi/image-2.fc0da29a6e4d2aff.webp" alt="Ảnh chụp màn hình: tạo phiên bản local devcontainer" width="50%">

## Bước 2: Cung Cấp Azure AI Foundry

Triển khai các mô hình AI trong khóa học lên Azure AI Foundry dưới dạng mã. Từ thư mục gốc kho lưu trữ:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```
  
`azd` sẽ yêu cầu tên môi trường và khu vực, cung cấp tài khoản Azure AI Foundry với các triển khai `gpt-4o-mini` và `text-embedding-3-small`, đồng thời ghi endpoint vào `.env` của ví dụ — tất cả với xác thực **không cần khóa** (không có khóa API).

> **Hướng dẫn chi tiết:** Xem [Hướng dẫn Thiết lập Azure AI Foundry](getting-started-azure-openai.md) để biết yêu cầu, phương pháp thủ công (portal), hướng dẫn chọn vùng và lưu ý về chi phí/dọn dẹp.

## Bước 3: Kiểm Tra Thiết Lập

Khi các mô hình Foundry của bạn đã được cung cấp, hãy kiểm tra kết nối với ứng dụng ví dụ trong [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure).

1. Mở terminal trong môi trường phát triển của bạn.  
2. Điều hướng tới ví dụ:  
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
  
3. Đảm bảo bạn đã đăng nhập (xác thực không khóa cần token):  
   ```bash
   az login
   ```
  
   > Nếu bạn đã chạy `azd up`, file `.env` chứa endpoint của bạn đã được ghi sẵn.  
4. Chạy ứng dụng:  
   ```bash
   mvn clean spring-boot:run
   ```
  
Bạn sẽ thấy câu trả lời từ mô hình `gpt-4o-mini`.

### Hiểu Về Mã Ví Dụ

Ví dụ trong `examples/basic-chat-azure` là ứng dụng Spring Boot dùng **Spring AI** để kết nối với Azure AI Foundry bằng xác thực không khóa.

**Mã này làm gì:**  
- **Kết nối** tới Azure AI Foundry bằng đăng nhập Azure của bạn (Microsoft Entra ID) — không cần khóa API  
- **Gửi** prompt tới mô hình `gpt-4o-mini`  
- **Nhận** và hiển thị phản hồi từ AI  
- **Xác nhận** thiết lập của bạn hoạt động chính xác

**Phụ thuộc chính** (trong `pom.xml`):  
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```
  
**Cấu hình** (`application.yml`):  
```yaml
spring:
  ai:
    azure:
      openai:
        # Endpoint only - no api-key. Spring AI uses DefaultAzureCredential (keyless).
        endpoint: ${AZURE_OPENAI_ENDPOINT}
        chat:
          options:
            deployment-name: ${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}
```
  
## Tóm Tắt

Tuyệt vời! Bạn đã có mọi thứ được thiết lập:

- Cung cấp mô hình Azure AI Foundry dưới dạng mã với Bicep + `azd`  
- Có môi trường phát triển Java chạy được (dù là Codespaces, container dev hoặc cục bộ)  
- Kết nối tới Azure AI Foundry với xác thực không khóa (Microsoft Entra ID) — không cần khóa API  
- Kiểm tra tất cả hoạt động với ví dụ đơn giản nói chuyện với mô hình của bạn

## Bước Tiếp Theo

[Chương 3: Kỹ Thuật Generative AI Cốt Lõi](../03-CoreGenerativeAITechniques/README.md)

## Khắc phục sự cố

Gặp vấn đề? Đây là các lỗi phổ biến và cách xử lý:

- **Xác thực thất bại (401/403)?**  
  - Chạy `az login` — xác thực không khóa nên bạn phải đăng nhập  
  - Kiểm tra tài khoản của bạn có vai trò **Cognitive Services OpenAI User** trên tài nguyên không  
  - Nếu bạn vừa mới cung cấp, đợi một phút cho vai trò được áp dụng

- **Không tìm thấy Maven?**  
  - Nếu dùng dev containers/Codespaces, Maven đã được cài sẵn  
  - Đối với thiết lập cục bộ, đảm bảo đã cài Java 21+ và Maven 3.9+  
  - Thử `mvn --version` để kiểm tra cài đặt

- **Không tìm thấy `azd` hoặc cung cấp thất bại?**  
  - Cài đặt [Azure Developer CLI](https://aka.ms/azure-dev/install) và chạy `azd auth login`  
  - Chọn khu vực có `gpt-4o-mini` (ví dụ `eastus2`)  
  - Xem [hướng dẫn thiết lập Azure AI Foundry](getting-started-azure-openai.md) để biết chi tiết

- **Dev container không khởi động?**  
  - Đảm bảo Docker Desktop đang chạy (cho phát triển cục bộ)  
  - Thử build lại container: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **Lỗi biên dịch ứng dụng?**  
  - Đảm bảo bạn đang ở đúng thư mục: `02-SetupDevEnvironment/examples/basic-chat-azure`  
  - Thử dọn sạch và build lại: `mvn clean compile`

> **Cần giúp đỡ?**: Vẫn gặp vấn đề? Hãy mở một issue trong kho lưu trữ và chúng tôi sẽ giúp bạn.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Tuyên bố miễn trừ trách nhiệm**:
Tài liệu này đã được dịch bằng dịch vụ dịch thuật AI [Co-op Translator](https://github.com/Azure/co-op-translator). Mặc dù chúng tôi cố gắng đảm bảo độ chính xác, xin lưu ý rằng bản dịch tự động có thể chứa lỗi hoặc sai sót. Tài liệu gốc bằng ngôn ngữ gốc nên được coi là nguồn tin chính thức. Đối với thông tin quan trọng, nên sử dụng dịch vụ dịch thuật chuyên nghiệp bởi con người. Chúng tôi không chịu trách nhiệm về bất kỳ hiểu lầm hoặc giải thích sai nào phát sinh từ việc sử dụng bản dịch này.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->