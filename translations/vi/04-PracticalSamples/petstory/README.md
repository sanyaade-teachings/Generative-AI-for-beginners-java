# Hướng Dẫn Tạo Truyện Thú Cưng Cho Người Mới Bắt Đầu

## Mục Lục

- [Yêu Cầu Trước](#yêu-cầu-trước)
- [Hiểu Cấu Trúc Dự Án](#hiểu-cấu-trúc-dự-án)
- [Giải Thích Các Thành Phần Chính](#giải-thích-các-thành-phần-chính)
  - [1. Ứng Dụng Chính](#1-ứng-dụng-chính)
  - [2. Bộ Điều Khiển Web](#2-bộ-điều-khiển-web)
  - [3. Dịch Vụ Truyện](#3-dịch-vụ-truyện)
  - [4. Mẫu Web](#4-mẫu-web)
  - [5. Cấu Hình](#5-cấu-hình)
- [Chạy Ứng Dụng](#chạy-ứng-dụng)
- [Cách Tất Cả Hoạt Động Cùng Nhau](#cách-tất-cả-hoạt-động-cùng-nhau)
- [Hiểu Về Tích Hợp AI](#hiểu-về-tích-hợp-ai)
- [Bước Tiếp Theo](#bước-tiếp-theo)

## Yêu Cầu Trước

Trước khi bắt đầu, hãy đảm bảo bạn đã có:
- Java 21 hoặc cao hơn đã cài đặt
- Maven để quản lý các thư viện phụ thuộc
- Một triển khai mô hình Azure AI Foundry (cấu hình với `azd up` — xem [Chương 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), đăng nhập bằng `az login` (xác thực không cần khóa)
- Kiến thức cơ bản về Java, Spring Boot, và phát triển web

## Hiểu Cấu Trúc Dự Án

Dự án truyện thú cưng có một số tập tin quan trọng:

```
petstory/
├── src/main/java/com/example/petstory/
│   ├── PetStoryApplication.java       # Main Spring Boot application
│   ├── PetController.java             # Web request handler
│   ├── StoryService.java              # AI story generation service
│   └── SecurityConfig.java            # Security configuration
├── src/main/resources/
│   ├── application.properties         # App configuration
│   └── templates/
│       ├── index.html                 # Upload form page
│       └── result.html               # Story display page
└── pom.xml                           # Maven dependencies
```

## Giải Thích Các Thành Phần Chính

### 1. Ứng Dụng Chính

**Tập tin:** `PetStoryApplication.java`

Đây là điểm vào cho ứng dụng Spring Boot của chúng ta:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Công dụng của phần này:**
- Chú thích `@SpringBootApplication` cho phép tự động cấu hình và quét các thành phần
- Khởi động máy chủ web nhúng (Tomcat) trên cổng 8080
- Tự động tạo mọi bean và dịch vụ Spring cần thiết

### 2. Bộ Điều Khiển Web

**Tập tin:** `PetController.java`

Phần này xử lý tất cả yêu cầu web và tương tác của người dùng:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Trả về mẫu index.html
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Kiểm tra đầu vào
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Làm sạch đầu vào để đảm bảo an toàn
        String sanitizedDescription = sanitizeInput(description);
        
        // Tạo câu chuyện với xử lý lỗi
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Trả về mẫu result.html
            
        } catch (Exception e) {
            // Sử dụng câu chuyện dự phòng nếu AI thất bại
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Giới hạn độ dài
    }
}
```

**Các tính năng chính:**

1. **Xử lý tuyến đường**: `@GetMapping("/")` hiển thị form tải lên, `@PostMapping("/generate-story")` xử lý gửi dữ liệu
2. **Kiểm tra đầu vào**: Kiểm tra mô tả có trống và giới hạn độ dài
3. **Bảo mật**: Làm sạch dữ liệu người dùng để tránh tấn công XSS
4. **Xử lý lỗi**: Cung cấp truyện dự phòng khi dịch vụ AI không hoạt động
5. **Ràng buộc mô hình**: Truyền dữ liệu sang mẫu HTML bằng `Model` của Spring

**Hệ Thống Dự Phòng:**
Bộ điều khiển bao gồm các mẫu truyện đã viết sẵn, được dùng khi dịch vụ AI không khả dụng:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Sử dụng băm mô tả để có phản hồi nhất quán
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Dịch Vụ Truyện

**Tập tin:** `StoryService.java`

Dịch vụ này giao tiếp với Azure AI Foundry để tạo truyện bằng xác thực không cần khóa:

```java
@Service
public class StoryService {
    
    private final OpenAIClient openAIClient;
    private final String modelName;
    
    public StoryService(@Value("${azure.openai.endpoint:}") String endpoint,
                       @Value("${azure.openai.deployment:gpt-4o-mini}") String modelName) {
        this.modelName = modelName;
        if (endpoint == null || endpoint.isBlank()) {
            endpoint = System.getenv("AZURE_OPENAI_ENDPOINT");
        }
        
        // Điểm cuối tương thích OpenAI của Foundry nằm dưới /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Xác thực không cần khóa với Microsoft Entra ID (không cần khóa API)
        DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
        this.openAIClient = OpenAIOkHttpClient.builder()
                .baseUrl(baseUrl)
                .credential(BearerTokenCredential.create(
                        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
                .build();
    }
    
    public String generateStory(String description) {
        String systemPrompt = "You are a creative storyteller who writes fun, " +
                             "family-friendly short stories about pets. " +
                             "Keep stories under 500 words and appropriate for all ages.";
        
        String userPrompt = "Write a fun short story about a pet described as: " + description;
        
        // Cấu hình yêu cầu AI
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Giới hạn độ dài phản hồi
                .temperature(0.8)          // Kiểm soát độ sáng tạo (0.0-1.0)
                .build();
        
        // Gửi yêu cầu và nhận phản hồi
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Các thành phần chính:**

1. **OpenAI Client**: Sử dụng SDK Java OpenAI chính thức được cấu hình cho Azure AI Foundry (xác thực không khóa)
2. **Yêu cầu hệ thống**: Đặt hành vi của AI để viết truyện thân thiện với gia đình về thú cưng
3. **Yêu cầu người dùng**: Hướng dẫn AI viết câu chuyện chính xác dựa trên mô tả
4. **Tham số**: Điều khiển độ dài truyện và mức độ sáng tạo
5. **Xử lý lỗi**: Ném ngoại lệ mà bộ điều khiển bắt và xử lý

### 4. Mẫu Web

**Tập tin:** `index.html` (Form Tải Lên)

Trang chính nơi người dùng mô tả thú cưng của họ:

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Pet Story Generator</title>
    <!-- CSS styling -->
</head>
<body>
    <div class="container">
        <h1>Pet Story Generator</h1>
        <p>Describe your pet and we'll create a fun story about them!</p>
        
        <!-- Error message display -->
        <div th:if="${error}" class="error" th:text="${error}"></div>
        
        <!-- Story generation form -->
        <form action="/generate-story" method="post">
            <div class="form-group">
                <label for="description">Describe your pet:</label>
                <textarea id="description" name="description" 
                         placeholder="Tell us about your pet - what they look like, their personality, favorite activities..."
                         maxlength="1000" required></textarea>
            </div>
            <button type="submit" class="btn btn-primary">Generate Story</button>
        </form>
        
        <!-- Image upload section with client-side processing -->
        <div class="upload-section">
            <h2>Or Upload a Photo</h2>
            <input type="file" id="imageInput" accept="image/*" />
            <button onclick="analyzeImage()" class="upload-btn">Analyze Image</button>
        </div>
        
        <script>
            // Client-side image analysis using Transformers.js
            async function analyzeImage() {
                // Image processing code here
                // Generates description automatically from uploaded image
            }
        </script>
    </div>
</body>
</html>
```

**Tập tin:** `result.html` (Hiển Thị Truyện)

Hiển thị câu chuyện được tạo:

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Pet Story Result</title>
</head>
<body>
    <div class="container">
        <h1>Your Pet's Story</h1>
        
        <div class="result-section">
            <div class="result-label">Pet Description:</div>
            <div class="result-content" th:text="${caption}"></div>
        </div>
        
        <div class="result-section">
            <div class="result-label">Generated Story:</div>
            <div class="result-content" th:text="${story}"></div>
        </div>
        
        <div class="result-section" th:if="${analysisType}">
            <div class="result-label">Analysis Type:</div>
            <div class="result-content" th:text="${analysisType}"></div>
        </div>
        
        <a href="/" class="back-link">Generate Another Story</a>
    </div>
</body>
</html>
```

**Tính năng mẫu:**

1. **Tích hợp Thymeleaf**: Sử dụng thuộc tính `th:` cho nội dung động
2. **Thiết kế đáp ứng**: CSS cho điện thoại và máy tính để bàn
3. **Xử lý lỗi**: Hiển thị lỗi kiểm tra cho người dùng
4. **Xử lý phía client**: JavaScript để phân tích ảnh (dùng Transformers.js)

### 5. Cấu Hình

**Tập tin:** `application.properties`

Cài đặt cấu hình cho ứng dụng:

```properties
spring.application.name=pet-story-app

# File upload limits
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=10MB

# Logging configuration
logging.level.com.example.petstory=INFO

# Azure AI Foundry (keyless) configuration
azure.openai.endpoint=${AZURE_OPENAI_ENDPOINT:}
azure.openai.deployment=${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}
```

**Giải thích cấu hình:**

1. **Tải tập tin**: Cho phép hình ảnh lên đến 10MB
2. **Ghi nhật ký**: Điều khiển thông tin được ghi lại trong quá trình chạy
3. **Azure AI Foundry**: Chỉ định điểm cuối và triển khai mô hình sử dụng (xác thực không khóa)
4. **Bảo mật**: Cấu hình xử lý lỗi để không tiết lộ thông tin nhạy cảm

## Chạy Ứng Dụng

### Bước 1: Đăng Nhập và Đặt Điểm Cuối

Xác thực không khóa (Microsoft Entra ID), nên không có khoá API. Đăng nhập và đặt điểm cuối Foundry của bạn:

**Windows (Command Prompt):**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**Windows (PowerShell):**
```powershell
az login
$env:AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
```

**Linux/macOS:**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**Tại sao cần điều này:**
- Azure AI Foundry dùng Microsoft Entra ID để xác thực các yêu cầu suy luận
- Xác thực không khóa nghĩa là không có bí mật trong mã nguồn hoặc môi trường của bạn
- Tài khoản của bạn cần vai trò **Cognitive Services OpenAI User** trên tài nguyên

### Bước 2: Biên Dịch và Chạy

Đi đến thư mục dự án:
```bash
cd 04-PracticalSamples/petstory
```

Biên dịch ứng dụng:
```bash
mvn clean compile
```

Khởi động máy chủ:
```bash
mvn spring-boot:run
```

Ứng dụng sẽ khởi chạy tại `http://localhost:8080`.

### Bước 3: Thử Ứng Dụng

1. **Mở** `http://localhost:8080` trên trình duyệt
2. **Mô tả** thú cưng của bạn trong ô văn bản (ví dụ: "Một chú chó golden retriever nghịch ngợm thích nhặt bóng")
3. **Nhấn** "Generate Story" để nhận truyện do AI tạo
4. **Hoặc** tải lên ảnh thú cưng để tự động tạo mô tả
5. **Xem** câu chuyện sáng tạo dựa trên mô tả thú cưng của bạn

## Cách Tất Cả Hoạt Động Cùng Nhau

Dưới đây là quy trình đầy đủ khi bạn tạo câu chuyện cho thú cưng:

1. **Nhập liệu người dùng**: Bạn mô tả thú cưng trên form web
2. **Gửi form**: Trình duyệt gửi yêu cầu POST tới `/generate-story`
3. **Xử lý bộ điều khiển**: `PetController` kiểm tra và làm sạch đầu vào
4. **Gọi dịch vụ AI**: `StoryService` gửi yêu cầu tới mô hình Azure AI Foundry
5. **Tạo truyện**: AI tạo câu chuyện sáng tạo dựa trên mô tả
6. **Xử lý phản hồi**: Bộ điều khiển nhận truyện và thêm vào mô hình
7. **Kết xuất mẫu**: Thymeleaf hiển thị `result.html` với truyện
8. **Hiển thị**: Người dùng xem truyện được tạo trên trình duyệt

**Quy trình xử lý lỗi:**
Nếu dịch vụ AI không thành công:
1. Bộ điều khiển bắt ngoại lệ
2. Tạo truyện dự phòng dùng mẫu viết sẵn
3. Hiển thị truyện dự phòng và ghi chú AI không khả dụng
4. Người dùng vẫn nhận truyện, đảm bảo trải nghiệm tốt

## Hiểu Về Tích Hợp AI

### Azure AI Foundry (xác thực không khóa)
Ứng dụng sử dụng Azure AI Foundry với xác thực không khóa (Microsoft Entra ID):

```java
// Xác thực không khóa - không cần khóa API
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Kỹ thuật tạo yêu cầu (Prompt Engineering)
Dịch vụ dùng các yêu cầu được thiết kế kỹ để có kết quả tốt:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Xử lý phản hồi
Phản hồi của AI được trích xuất và kiểm tra:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Bước Tiếp Theo

Để xem thêm ví dụ, xem [Chương 04: Ví dụ thực tiễn](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Tuyên bố miễn trừ trách nhiệm**:
Tài liệu này đã được dịch bằng dịch vụ dịch thuật AI [Co-op Translator](https://github.com/Azure/co-op-translator). Mặc dù chúng tôi cố gắng đảm bảo độ chính xác, xin lưu ý rằng bản dịch tự động có thể chứa lỗi hoặc sai sót. Tài liệu gốc bằng ngôn ngữ gốc nên được coi là nguồn tin chính thức. Đối với thông tin quan trọng, nên sử dụng dịch vụ dịch thuật chuyên nghiệp bởi con người. Chúng tôi không chịu trách nhiệm về bất kỳ hiểu lầm hoặc giải thích sai nào phát sinh từ việc sử dụng bản dịch này.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->