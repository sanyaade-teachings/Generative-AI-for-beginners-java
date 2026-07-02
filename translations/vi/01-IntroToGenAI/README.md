# Giới thiệu về Generative AI - Phiên bản Java

## Những gì bạn sẽ học

- **Các kiến thức cơ bản về Generative AI** bao gồm LLMs, kỹ thuật xây dựng prompt, tokens, embeddings, và cơ sở dữ liệu vector
- **So sánh các công cụ phát triển AI trên Java** bao gồm Azure OpenAI SDK, Spring AI, và OpenAI Java SDK
- **Khám phá Model Context Protocol** và vai trò của nó trong giao tiếp của các tác nhân AI

## Mục lục

- [Giới thiệu](#giới-thiệu)
- [Làm mới nhanh các khái niệm về Generative AI](#làm-mới-nhanh-các-khái-niệm-về-generative-ai)
- [Xem lại kỹ thuật xây dựng prompt](#xem-lại-kỹ-thuật-xây-dựng-prompt)
- [Tokens, embeddings, và các tác nhân](#tokens-embeddings-và-các-tác-nhân)
- [Công cụ và thư viện phát triển AI cho Java](#công-cụ-và-thư-viện-phát-triển-ai-cho-java)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [Tóm tắt](#tóm-tắt)
- [Bước tiếp theo](#bước-tiếp-theo)

## Giới thiệu

Chào mừng bạn đến với chương đầu tiên của Generative AI cho Người Mới Bắt Đầu - Phiên bản Java! Bài học cơ bản này sẽ giới thiệu bạn về các khái niệm cốt lõi của generative AI và cách làm việc với chúng bằng Java. Bạn sẽ tìm hiểu về các thành phần xây dựng thiết yếu của ứng dụng AI, bao gồm Các mô hình ngôn ngữ lớn (LLMs), tokens, embeddings và các tác nhân AI. Chúng ta cũng sẽ khám phá các công cụ Java chính bạn sẽ dùng trong suốt khóa học này.

### Làm mới nhanh các khái niệm về Generative AI

Generative AI là một loại trí tuệ nhân tạo tạo ra nội dung mới, như văn bản, hình ảnh hoặc mã lập trình, dựa trên các mẫu và mối quan hệ học được từ dữ liệu. Các mô hình generative AI có thể tạo ra các câu trả lời giống như của con người, hiểu ngữ cảnh, và đôi khi thậm chí tạo ra nội dung có vẻ giống con người.

Khi phát triển các ứng dụng AI trên Java, bạn sẽ làm việc với **các mô hình generative AI** để tạo nội dung. Một số khả năng của các mô hình generative AI bao gồm:

- **Tạo văn bản**: Soạn thảo văn bản giống người cho chatbot, nội dung, và hoàn thành văn bản.
- **Tạo và phân tích hình ảnh**: Tạo ảnh chân thực, cải thiện ảnh, và phát hiện đối tượng.
- **Tạo mã lập trình**: Viết đoạn mã hoặc tập lệnh.

Có các loại mô hình cụ thể được tối ưu hóa cho các nhiệm vụ khác nhau. Ví dụ, cả **Mô hình Ngôn ngữ Nhỏ (SLMs)** và **Mô hình Ngôn ngữ Lớn (LLMs)** đều có thể xử lý tạo văn bản, với LLMs thường cho hiệu năng tốt hơn trong các nhiệm vụ phức tạp. Đối với các tác vụ liên quan đến hình ảnh, bạn sẽ sử dụng các mô hình chuyên biệt về thị giác hoặc mô hình đa phương tiện.

![Figure: Generative AI model types and use cases.](../../../translated_images/vi/llms.225ca2b8a0d34473.webp)

Dĩ nhiên, phản hồi từ các mô hình này không phải lúc nào cũng hoàn hảo. Có thể bạn đã nghe nói về các mô hình "ảo tưởng" hoặc tạo ra thông tin sai lệch một cách chính xác. Nhưng bạn có thể giúp hướng dẫn mô hình tạo ra câu trả lời tốt hơn bằng cách cung cấp cho chúng các chỉ dẫn rõ ràng và ngữ cảnh. Đây chính là lúc **kỹ thuật xây dựng prompt** trở nên quan trọng.

#### Xem lại kỹ thuật xây dựng prompt

Kỹ thuật xây dựng prompt là việc thiết kế các đầu vào hiệu quả nhằm hướng các mô hình AI tới các kết quả mong muốn. Nó bao gồm:

- **Rõ ràng**: Đưa ra hướng dẫn rõ ràng và không mơ hồ.
- **Ngữ cảnh**: Cung cấp thông tin nền cần thiết.
- **Ràng buộc**: Chỉ định các hạn chế hoặc định dạng cụ thể.

Một số thực hành tốt trong xây dựng prompt bao gồm thiết kế prompt, hướng dẫn rõ ràng, phân chia nhiệm vụ, học một-lần và học vài-lần, và tinh chỉnh prompt. Việc thử nghiệm các prompt khác nhau là cần thiết để tìm ra cách làm tốt nhất cho trường hợp sử dụng cụ thể của bạn.

Khi phát triển ứng dụng, bạn sẽ làm việc với các loại prompt khác nhau:
- **Prompt hệ thống**: Thiết lập các quy tắc cơ bản và ngữ cảnh cho hành vi của mô hình
- **Prompt người dùng**: Dữ liệu đầu vào từ người dùng ứng dụng của bạn
- **Prompt trợ lý**: Các phản hồi của mô hình dựa trên prompt hệ thống và người dùng

> **Tìm hiểu thêm**: Tìm hiểu thêm về kỹ thuật xây dựng prompt trong [Chương Prompt Engineering của khóa học GenAI cho Người Mới Bắt Đầu](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### Tokens, embeddings, và các tác nhân

Khi làm việc với các mô hình generative AI, bạn sẽ gặp các thuật ngữ như **tokens**, **embeddings**, **agents**, và **Model Context Protocol (MCP)**. Dưới đây là tổng quan chi tiết về các khái niệm này:

- **Tokens**: Tokens là đơn vị nhỏ nhất của văn bản trong một mô hình. Chúng có thể là từ, ký tự, hoặc các phần nhỏ của từ. Tokens được sử dụng để biểu diễn dữ liệu văn bản dưới dạng mà mô hình có thể hiểu được. Ví dụ, câu "The quick brown fox jumped over the lazy dog" có thể được tách token thành ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] hoặc ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"] tùy thuộc vào chiến lược token hóa.

![Figure: Generative AI tokens example of breaking words into tokens](../../../translated_images/vi/tokens.6283ed277a2ffff4.webp)

Quá trình token hóa là việc phân tách văn bản thành các đơn vị nhỏ này. Điều này rất quan trọng bởi vì mô hình hoạt động trên tokens thay vì văn bản nguyên bản. Số lượng tokens trong một prompt ảnh hưởng đến độ dài và chất lượng phản hồi của mô hình, vì mô hình có giới hạn tokens cho khoảng cửa sổ ngữ cảnh của nó (ví dụ: 128K tokens cho tổng ngữ cảnh của GPT-4o, bao gồm cả đầu vào lẫn đầu ra).

  Trong Java, bạn có thể sử dụng các thư viện như OpenAI SDK để xử lý token hóa tự động khi gửi yêu cầu tới mô hình AI.

- **Embeddings**: Embeddings là biểu diễn vector của tokens mang ý nghĩa ngữ nghĩa. Chúng là các biểu diễn số học (thông thường là mảng số thực dấu phẩy động) cho phép mô hình hiểu các mối quan hệ giữa từ và tạo ra phản hồi phù hợp theo ngữ cảnh. Những từ có ý nghĩa tương tự sẽ có embeddings gần nhau, giúp mô hình hiểu các khái niệm như đồng nghĩa và mối quan hệ ngữ nghĩa.

![Figure: Embeddings](../../../translated_images/vi/embedding.398e50802c0037f9.webp)

  Trong Java, bạn có thể tạo embeddings bằng OpenAI SDK hoặc các thư viện khác hỗ trợ tạo embedding. Các embeddings này rất quan trọng cho các tác vụ như tìm kiếm ngữ nghĩa, nơi bạn muốn tìm nội dung tương tự dựa trên ý nghĩa thay vì khớp chính xác về văn bản.

- **Cơ sở dữ liệu vector**: Cơ sở dữ liệu vector là các hệ thống lưu trữ chuyên biệt được tối ưu cho embeddings. Chúng cho phép tìm kiếm tương tự hiệu quả và rất quan trọng cho các mẫu Retrieval-Augmented Generation (RAG), nơi bạn cần tìm thông tin phù hợp từ các bộ dữ liệu lớn dựa trên sự tương đồng ngữ nghĩa thay vì khớp chính xác.

![Figure: Vector database architecture showing how embeddings are stored and retrieved for similarity search.](../../../translated_images/vi/vector.f12f114934e223df.webp)

> **Lưu ý**: Trong khóa học này, chúng ta sẽ không đề cập chi tiết về cơ sở dữ liệu vector nhưng thấy rằng chúng xứng đáng được nhắc tới vì chúng thường được sử dụng trong các ứng dụng thực tế.

- **Agents & MCP**: Các thành phần AI tương tác tự động với mô hình, công cụ và hệ thống bên ngoài. Model Context Protocol (MCP) cung cấp cách tiếp cận chuẩn hóa để các tác nhân truy cập an toàn vào các nguồn dữ liệu và công cụ bên ngoài. Tìm hiểu thêm trong khóa học [MCP cho Người Mới Bắt Đầu](https://github.com/microsoft/mcp-for-beginners).

Trong các ứng dụng AI Java, bạn sẽ sử dụng tokens cho xử lý văn bản, embeddings cho tìm kiếm ngữ nghĩa và RAG, cơ sở dữ liệu vector cho truy xuất dữ liệu, và các tác nhân với MCP để xây dựng hệ thống thông minh sử dụng công cụ.

![Figure: how a prompt becomes a reply—tokens, vectors, optional RAG lookup, LLM thinking, and an MCP agent all in one quick flow..](../../../translated_images/vi/flow.f4ef62c3052d12a8.webp)

### Công cụ và thư viện phát triển AI cho Java

Java cung cấp các công cụ tuyệt vời cho phát triển AI. Có ba thư viện chính mà chúng ta sẽ khám phá xuyên suốt khóa học này - OpenAI Java SDK, Azure OpenAI SDK, và Spring AI.

Dưới đây là bảng tham khảo nhanh cho thấy SDK nào được sử dụng trong các ví dụ của từng chương:

| Chương | Mẫu | SDK |
|---------|--------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**Liên kết tài liệu SDK:**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

OpenAI SDK là thư viện Java chính thức cho API OpenAI. Nó cung cấp một giao diện đơn giản và nhất quán để tương tác với các mô hình của OpenAI, giúp việc tích hợp khả năng AI vào các ứng dụng Java trở nên dễ dàng. Ứng dụng Pet Story và ví dụ Foundry Local trong Chương 4 minh họa cách sử dụng OpenAI SDK với Azure AI Foundry.

#### Spring AI

Spring AI là một framework toàn diện mang lại khả năng AI cho các ứng dụng Spring, cung cấp lớp trừu tượng thống nhất cho nhiều nhà cung cấp AI khác nhau. Nó tích hợp liền mạch với hệ sinh thái Spring, làm cho nó trở thành lựa chọn lý tưởng cho các ứng dụng Java doanh nghiệp cần năng lực AI.

Điểm mạnh của Spring AI là tích hợp sâu với hệ sinh thái Spring, giúp xây dựng các ứng dụng AI sẵn sàng cho sản xuất với các mẫu thiết kế Spring quen thuộc như tiêm phụ thuộc, quản lý cấu hình, và framework kiểm thử. Bạn sẽ sử dụng Spring AI trong Chương 2 và 4 để xây dựng các ứng dụng tận dụng cả OpenAI và các thư viện Spring AI Model Context Protocol (MCP).

##### Model Context Protocol (MCP)

[Model Context Protocol (MCP)](https://modelcontextprotocol.io/) là một tiêu chuẩn mới nổi giúp các ứng dụng AI tương tác an toàn với các nguồn dữ liệu và công cụ bên ngoài. MCP cung cấp cách tiếp cận chuẩn hóa để các mô hình AI truy cập thông tin ngữ cảnh và thực thi các hành động trong ứng dụng của bạn.

Trong Chương 4, bạn sẽ xây dựng một dịch vụ máy tính MCP đơn giản minh họa các nền tảng cơ bản của Model Context Protocol với Spring AI, cho thấy cách tạo tích hợp công cụ cơ bản và kiến trúc dịch vụ.

#### Azure OpenAI Java SDK

Thư viện khách Azure OpenAI cho Java là bản điều chỉnh các API REST của OpenAI cung cấp giao diện phù hợp theo ngôn ngữ và tích hợp với hệ sinh thái SDK Azure còn lại. Trong Chương 3, bạn sẽ xây dựng các ứng dụng sử dụng Azure OpenAI SDK, bao gồm ứng dụng chat, gọi hàm, và các mẫu RAG (Retrieval-Augmented Generation).

> Lưu ý: Azure OpenAI SDK chậm hơn OpenAI Java SDK về mặt tính năng, vì vậy với các dự án tương lai, hãy cân nhắc sử dụng OpenAI Java SDK.

## Tóm tắt

Đó là nền tảng cơ bản! Giờ bạn đã hiểu:

- Các khái niệm cốt lõi đằng sau generative AI - từ LLMs và kỹ thuật xây dựng prompt đến tokens, embeddings, và cơ sở dữ liệu vector
- Các lựa chọn bộ công cụ phát triển AI trên Java của bạn: Azure OpenAI SDK, Spring AI, và OpenAI Java SDK
- Model Context Protocol là gì và cách nó giúp các tác nhân AI làm việc với các công cụ bên ngoài

## Bước tiếp theo

[Chương 2: Thiết lập môi trường phát triển](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Tuyên bố miễn trừ trách nhiệm**:
Tài liệu này đã được dịch bằng dịch vụ dịch thuật AI [Co-op Translator](https://github.com/Azure/co-op-translator). Mặc dù chúng tôi cố gắng đảm bảo độ chính xác, xin lưu ý rằng bản dịch tự động có thể chứa lỗi hoặc sai sót. Tài liệu gốc bằng ngôn ngữ gốc nên được coi là nguồn tin chính thức. Đối với thông tin quan trọng, nên sử dụng dịch vụ dịch thuật chuyên nghiệp bởi con người. Chúng tôi không chịu trách nhiệm về bất kỳ hiểu lầm hoặc giải thích sai nào phát sinh từ việc sử dụng bản dịch này.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->