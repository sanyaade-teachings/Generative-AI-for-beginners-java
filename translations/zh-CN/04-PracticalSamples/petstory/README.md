# 宠物故事生成器初学者教程

## 目录

- [先决条件](#先决条件)
- [了解项目结构](#了解项目结构)
- [核心组件讲解](#核心组件讲解)
  - [1. 主应用程序](#1-主应用程序)
  - [2. 网络控制器](#2-网络控制器)
  - [3. 故事服务](#3-故事服务)
  - [4. 网络模板](#4-网络模板)
  - [5. 配置](#5-配置)
- [运行应用程序](#运行应用程序)
- [整体工作流程](#整体工作流程)
- [理解AI集成](#理解ai集成)
- [下一步](#下一步)

## 先决条件

开始之前，请确保您具备以下条件：
- 安装了 Java 21 或更高版本
- 安装了用于依赖管理的 Maven
- 已部署 Azure AI Foundry 模型（通过 `azd up` 进行配置—参见 [第2章](../../02-SetupDevEnvironment/getting-started-azure-openai.md)），并使用 `az login` 登录（无密钥认证）
- 具备 Java、Spring Boot 和网页开发的基础知识

## 了解项目结构

宠物故事项目包含几个重要的文件：

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

## 核心组件讲解

### 1. 主应用程序

**文件：** `PetStoryApplication.java`

这是我们 Spring Boot 应用程序的入口：

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**功能说明：**
- `@SpringBootApplication` 注解启用自动配置和组件扫描
- 在 8080 端口启动嵌入式（Tomcat）Web 服务器
- 自动创建所有必要的 Spring Bean 和服务

### 2. 网络控制器

**文件：** `PetController.java`

负责处理所有网络请求和用户交互：

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // 返回 index.html 模板
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // 输入验证
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // 对输入进行安全过滤
        String sanitizedDescription = sanitizeInput(description);
        
        // 生成故事并处理错误
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // 返回 result.html 模板
            
        } catch (Exception e) {
            // 如果 AI 失败则使用备用故事
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // 限制长度
    }
}
```

**关键功能：**

1. <strong>路由处理</strong>：`@GetMapping("/")` 显示上传表单，`@PostMapping("/generate-story")` 处理提交
2. <strong>输入验证</strong>：检查描述是否为空和长度限制
3. <strong>安全性</strong>：对用户输入进行清理，防止 XSS 攻击
4. <strong>错误处理</strong>：AI 服务失败时提供备用故事
5. <strong>模型绑定</strong>：使用 Spring 的 `Model` 向 HTML 模板传递数据

**备用系统：**
当 AI 服务不可用时，控制器使用预先编写的故事模板：

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // 使用描述哈希以获得一致的响应
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. 故事服务

**文件：** `StoryService.java`

该服务与 Azure AI Foundry 通过无密钥认证进行通信以生成故事：

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
        
        // Foundry 的 OpenAI 兼容端点位于 /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // 使用 Microsoft Entra ID 进行无密钥身份验证（无 API 密钥）
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
        
        // 配置 AI 请求
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // 限制响应长度
                .temperature(0.8)          // 控制创造力（0.0-1.0）
                .build();
        
        // 发送请求并获取响应
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**关键组件：**

1. **OpenAI 客户端**：使用官方的 OpenAI Java SDK，配置为 Azure AI Foundry（无密钥）
2. <strong>系统提示</strong>：设置 AI 行为，编写适合家庭的宠物故事
3. <strong>用户提示</strong>：告诉 AI 根据描述编写故事
4. <strong>参数</strong>：控制故事长度和创意水平
5. <strong>错误处理</strong>：抛出异常，供控制器捕获和处理

### 4. 网络模板

**文件：** `index.html`（上传表单）

主页面，用户在这里描述他们的宠物：

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

**文件：** `result.html`（故事展示）

显示生成的故事：

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

**模板特点：**

1. **Thymeleaf 集成**：使用 `th:` 属性实现动态内容
2. <strong>响应式设计</strong>：CSS 样式支持移动端和桌面端
3. <strong>错误处理</strong>：显示验证错误信息给用户
4. <strong>客户端处理</strong>：使用 JavaScript 进行图像分析（基于 Transformers.js）

### 5. 配置

**文件：** `application.properties`

应用程序的配置设置：

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

**配置说明：**

1. <strong>文件上传</strong>：允许最大 10MB 图片
2. <strong>日志</strong>：控制执行时记录的信息
3. **Azure AI Foundry**：指定使用的端点和模型部署（无密钥认证）
4. <strong>安全性</strong>：错误处理配置，避免暴露敏感信息

## 运行应用程序

### 步骤 1：登录并设置终结点

认证采用无密钥方式（Microsoft Entra ID），无需 API 密钥。请登录并设置您的 Foundry 终结点：

**Windows（命令提示符）：**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**Windows（PowerShell）：**
```powershell
az login
$env:AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
```

**Linux/macOS：**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**为什么需要这样做：**
- Azure AI Foundry 使用 Microsoft Entra ID 认证推断请求
- 无密钥认证意味着源代码或环境中无需存储密钥
- 您的账户需要在资源上拥有 **认知服务 OpenAI 用户** 角色

### 步骤 2：构建并运行

切换到项目目录：
```bash
cd 04-PracticalSamples/petstory
```

构建应用：
```bash
mvn clean compile
```

启动服务器：
```bash
mvn spring-boot:run
```

应用将在 `http://localhost:8080` 启动。

### 步骤 3：测试应用

1. <strong>打开</strong> 浏览器访问 `http://localhost:8080`
2. <strong>描述</strong> 您的宠物（例如：“一只喜欢捡球的活泼金毛”）
3. <strong>点击</strong> “生成故事” 获取 AI 生成的故事
4. <strong>或者</strong> 上传宠物图片自动生成描述
5. <strong>查看</strong> 根据宠物描述创作的有趣故事

## 整体工作流程

生成宠物故事的完整流程如下：

1. <strong>用户输入</strong>：您在网页表单中描述宠物
2. <strong>表单提交</strong>：浏览器发送 POST 请求到 `/generate-story`
3. <strong>控制器处理</strong>：`PetController` 验证并清理输入内容
4. **AI 服务调用**：`StoryService` 向 Azure AI Foundry 模型发送请求
5. <strong>故事生成</strong>：AI 根据描述创作故事
6. <strong>响应处理</strong>：控制器接收故事并添加到模型中
7. <strong>模板渲染</strong>：Thymeleaf 渲染 `result.html` 页面显示故事
8. <strong>显示</strong>：用户浏览器中显示生成的故事

**错误处理流程：**
如果 AI 服务失败：
1. 控制器捕获异常
2. 使用预先写好的模板生成备用故事
3. 显示备用故事并附有 AI 服务不可用的提示
4. 用户仍可获得故事，确保良好的用户体验

## 理解AI集成

### Azure AI Foundry（无密钥）
应用使用 Azure AI Foundry，采用无密钥认证（Microsoft Entra ID）：

```java
// 无密钥认证 - 无需 API 密钥
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### 提示工程
服务使用精心设计的提示以获得良好结果：

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### 响应处理
提取并验证 AI 响应：

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## 下一步

更多示例，参见 [第04章：实用示例](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免责声明**：
本文件由 AI 翻译服务 [Co-op Translator](https://github.com/Azure/co-op-translator) 翻译完成。尽管我们力求准确，但请注意，自动翻译可能包含错误或不准确之处。原始语言版文件应视为权威来源。对于重要信息，建议使用专业人工翻译。我们对因使用本翻译而产生的任何误解或误释不承担责任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->