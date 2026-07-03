# Ứng Dụng Thực Tiễn & Dự Án

## Những Gì Bạn Sẽ Học
Trong phần này, chúng ta sẽ trình diễn ba ứng dụng thực tiễn thể hiện các mẫu phát triển AI sinh tạo với Java:
- Tạo một Trình Tạo Câu Chuyện Thú Cưng đa phương thức kết hợp AI phía khách và phía máy chủ
- Triển khai tích hợp mô hình AI cục bộ với bản demo Foundry Local Spring Boot
- Phát triển dịch vụ Model Context Protocol (MCP) với ví dụ Máy tính

## Mục Lục

- [Giới thiệu](#giới-thiệu)
  - [Demo Foundry Local Spring Boot](#demo-foundry-local-spring-boot)
  - [Trình Tạo Câu Chuyện Thú Cưng](#trình-tạo-câu-chuyện-thú-cưng)
  - [Dịch Vụ MCP Máy Tính (Demo MCP Thân Thiện Với Người Mới)](#dịch-vụ-mcp-máy-tính-demo-mcp-thân-thiện-với-người-mới)
- [Tiến Trình Học Tập](#tiến-trình-học-tập)
- [Tóm Tắt](#tóm-tắt)
- [Bước Tiếp Theo](#bước-tiếp-theo)

## Giới thiệu

Chương này trình bày **các dự án mẫu** thể hiện các mẫu phát triển AI sinh tạo với Java. Mỗi dự án đều hoạt động hoàn chỉnh và trình bày các công nghệ AI cụ thể, mẫu kiến trúc, và các thực hành tốt nhất mà bạn có thể áp dụng cho ứng dụng của mình.

### Demo Foundry Local Spring Boot

**[Demo Foundry Local Spring Boot](foundrylocal/README.md)** trình diễn cách tích hợp với các mô hình AI cục bộ sử dụng **OpenAI Java SDK**. Nó thể hiện việc kết nối với các mô hình chạy trên Foundry Local (ví dụ: **Phi-4-mini**), với phát hiện mô hình tự động, cho phép bạn chạy ứng dụng AI mà không cần phụ thuộc vào dịch vụ đám mây.

### Trình Tạo Câu Chuyện Thú Cưng

**[Trình Tạo Câu Chuyện Thú Cưng](petstory/README.md)** là một ứng dụng web Spring Boot hấp dẫn thể hiện **xử lý AI đa phương thức** để tạo ra các câu chuyện thú cưng sáng tạo. Ứng dụng kết hợp khả năng AI phía khách và phía máy chủ sử dụng transformer.js cho tương tác AI trên trình duyệt và OpenAI SDK cho xử lý phía máy chủ.

### Dịch Vụ MCP Máy Tính (Demo MCP Thân Thiện Với Người Mới)

**[Dịch Vụ MCP Máy Tính](calculator/README.md)** là một minh họa đơn giản của **Model Context Protocol (MCP)** sử dụng Spring AI. Nó cung cấp một giới thiệu thân thiện với người mới về các khái niệm MCP, cho thấy cách tạo một MCP Server cơ bản tương tác với các MCP client.

## Tiến Trình Học Tập

Các dự án này được thiết kế để xây dựng dựa trên các khái niệm từ các chương trước:

1. **Bắt đầu đơn giản**: Bắt đầu với Demo Foundry Local Spring Boot để hiểu tích hợp AI cơ bản với các mô hình cục bộ
2. **Thêm tính tương tác**: Tiến tới Trình Tạo Câu Chuyện Thú Cưng cho AI đa phương thức và tương tác web
3. **Học MCP cơ bản**: Thử nghiệm dịch vụ MCP Máy Tính để hiểu các nguyên tắc cơ bản của Model Context Protocol

## Tóm Tắt

Làm tốt lắm! Bây giờ bạn đã khám phá một số ứng dụng thực tế:

- Trải nghiệm AI đa phương thức hoạt động cả trên trình duyệt và máy chủ
- Tích hợp mô hình AI cục bộ sử dụng các framework và SDK Java hiện đại
- Dịch vụ Model Context Protocol đầu tiên của bạn để thấy cách các công cụ tích hợp với AI

## Bước Tiếp Theo

[Chương 5: AI Sinh Tạo Có Trách Nhiệm](../05-ResponsibleGenAI/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Tuyên bố miễn trừ trách nhiệm**:
Tài liệu này đã được dịch bằng dịch vụ dịch thuật AI [Co-op Translator](https://github.com/Azure/co-op-translator). Mặc dù chúng tôi cố gắng đảm bảo độ chính xác, xin lưu ý rằng bản dịch tự động có thể chứa lỗi hoặc sai sót. Tài liệu gốc bằng ngôn ngữ gốc nên được coi là nguồn tin chính thức. Đối với thông tin quan trọng, nên sử dụng dịch vụ dịch thuật chuyên nghiệp bởi con người. Chúng tôi không chịu trách nhiệm về bất kỳ hiểu lầm hoặc giải thích sai nào phát sinh từ việc sử dụng bản dịch này.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->