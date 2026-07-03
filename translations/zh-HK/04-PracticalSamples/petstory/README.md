# 寵物故事產生器初學者教學

## 目錄

- [先決條件](#先決條件)
- [了解專案結構](#了解專案結構)
- [核心組件說明](#核心組件說明)
  - [1. 主應用程式](#1-主應用程式)
  - [2. Web 控制器](#2-web-控制器)
  - [3. 故事服務](#3-故事服務)
  - [4. Web 範本](#4-web-範本)
  - [5. 設定](#5-設定)
- [執行應用程式](#執行應用程式)
- [整合運作流程](#整合運作流程)
- [了解 AI 整合](#了解-ai-整合)
- [後續步驟](#後續步驟)

## 先決條件

開始之前，請確保您已經具備：
- 安裝 Java 21 或更高版本
- Maven 來管理相依性
- 一個 Azure AI Foundry 模型部署（使用 `azd up` 進行配置 — 參見 [第二章](../../02-SetupDevEnvironment/getting-started-azure-openai.md)），並已用 `az login` 登入（無需金鑰驗證）
- 基本的 Java、Spring Boot 和網頁開發知識

## 了解專案結構

寵物故事專案包含以下重要檔案：

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

## 核心組件說明

### 1. 主應用程式

**檔案：** `PetStoryApplication.java`

這是我們 Spring Boot 應用程式的進入點：

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**功能說明：**
- `@SpringBootApplication` 註解啟用自動配置與元件掃描
- 啟動內嵌的 Tomcat 網頁伺服器，埠號為 8080
- 自動建立所有必要的 Spring Bean 與服務

### 2. Web 控制器

**檔案：** `PetController.java`

負責處理所有網頁請求與使用者互動：

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
        
        // 輸入驗證
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // 為安全起見清理輸入
        String sanitizedDescription = sanitizeInput(description);
        
        // 生成故事並處理錯誤
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // 返回 result.html 模板
            
        } catch (Exception e) {
            // 若 AI 失敗，使用備用故事
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // 限制長度
    }
}
```

**主要功能：**

1. <strong>路由處理</strong>：`@GetMapping("/")` 顯示上傳表單，`@PostMapping("/generate-story")` 處理表單提交
2. <strong>輸入驗證</strong>：檢查描述是否為空及長度限制
3. <strong>安全性</strong>：對使用者輸入進行消毒以防止 XSS 攻擊
4. <strong>錯誤處理</strong>：在 AI 服務失效時提供備用故事
5. <strong>模型繫結</strong>：使用 Spring 的 `Model` 傳遞資料到 HTML 範本

**備援系統：**
控制器包含預先寫好的故事範本，用於 AI 服務不可用時使用：

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // 使用描述哈希以獲得一致的回應
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. 故事服務

**檔案：** `StoryService.java`

此服務與 Azure AI Foundry 通訊，利用無金鑰認證生成故事：

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
        
        // Foundry 的 OpenAI 相容端點位於 /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // 使用 Microsoft Entra ID 進行無密鑰認證（無需 API 金鑰）
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
        
        // 配置 AI 請求
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // 限制回應長度
                .temperature(0.8)          // 控制創意度（0.0-1.0）
                .build();
        
        // 發送請求並獲取回應
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**主要組件：**

1. **OpenAI 客戶端**：使用官方 OpenAI Java SDK，針對 Azure AI Foundry（無金鑰）設定
2. <strong>系統提示</strong>：設定 AI 行為，編寫適合家庭的寵物故事
3. <strong>用戶提示</strong>：告訴 AI 根據描述編寫特定故事
4. <strong>參數控制</strong>：調整故事長度與創意等級
5. <strong>錯誤處理</strong>：拋出例外，供控制器攔截處理

### 4. Web 範本

**檔案：** `index.html`（上傳表單）

主頁面，使用者描述他們的寵物：

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

**檔案：** `result.html`（顯示故事）

呈現生成的故事：

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

**範本特點：**

1. **Thymeleaf 整合**：使用 `th:` 屬性呈現動態內容
2. <strong>響應式設計</strong>：CSS 支援行動與桌面裝置
3. <strong>錯誤提示</strong>：向使用者顯示驗證錯誤
4. <strong>客戶端處理</strong>：使用 JavaScript 結合 Transformers.js 進行影像分析

### 5. 設定

**檔案：** `application.properties`

應用程式的設定項目：

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

**設定說明：**

1. <strong>檔案上傳</strong>：允許最大為 10MB 的圖片上傳
2. <strong>日誌紀錄</strong>：控制執行期間的日誌資訊
3. **Azure AI Foundry**：指定端點與模型部署名稱（無金鑰驗證）
4. <strong>安全性</strong>：錯誤處理配置，避免曝光敏感資訊

## 執行應用程式

### 第一步：登入並設定端點

採用無金鑰驗證（Microsoft Entra ID），無需 API 金鑰。登入並設定 Foundry 端點：

**Windows（命令提示字元）：**
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

**原因說明：**
- Azure AI Foundry 使用 Microsoft Entra ID 進行推論請求的身份驗證
- 無金鑰認證表示不需在程式碼或環境中保存任何密鑰
- 您的帳號需具備該資源的 **認知服務 OpenAI 使用者** 角色

### 第二步：編譯與執行

切換到專案目錄：
```bash
cd 04-PracticalSamples/petstory
```

編譯應用程式：
```bash
mvn clean compile
```

啟動伺服器：
```bash
mvn spring-boot:run
```

應用程式將在 `http://localhost:8080` 啟動。

### 第三步：測試應用程式

1. <strong>開啟</strong> 瀏覽器並輸入 `http://localhost:8080`
2. <strong>描述</strong> 您的寵物（例如：「一隻愛玩取物的黃金獵犬」）
3. <strong>點擊</strong>「生成故事」以獲得 AI 生成的故事
4. <strong>或</strong> 上傳寵物圖片，自動產生描述
5. <strong>查看</strong> 根據描述創作的有趣故事

## 整合運作流程

以下是在產生寵物故事時的完整流程：

1. <strong>使用者輸入</strong>：您在網頁表單描述寵物
2. <strong>表單提交</strong>：瀏覽器送出 POST 請求至 `/generate-story`
3. <strong>控制器處理</strong>：`PetController` 驗證並清理輸入資料
4. **呼叫 AI 服務**：`StoryService` 向 Azure AI Foundry 模型發出請求
5. <strong>故事生成</strong>：AI 根據描述產出創意故事
6. <strong>回應處理</strong>：控制器接收故事並加入模型中
7. <strong>範本渲染</strong>：Thymeleaf 渲染 `result.html` 與故事內容
8. <strong>顯示故事</strong>：使用者在瀏覽器看到產生的故事

**錯誤處理流程：**
若 AI 服務失效時：
1. 控制器捕捉異常
2. 使用預設的故事範本生成備援故事
3. 顯示備援故事並提示 AI 目前不可用
4. 使用者仍能收到故事，確保良好體驗

## 了解 AI 整合

### Azure AI Foundry（無金鑰）

應用程式使用 Azure AI Foundry 的無金鑰認證（Microsoft Entra ID）：

```java
// 無須密鑰驗證 - 無 API 密鑰
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### 提示詞工程

服務利用精心設計的提示詞來獲得優質結果：

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### 回應處理

解析並驗證 AI 回應內容：

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## 後續步驟

更多範例請見 [第四章：實用範例](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
本文件由 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 翻譯而成。雖然我們致力於確保準確性，但請注意，機器自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應被視為權威來源。對於重要資訊，建議進行專業人工翻譯。我們不對因使用本翻譯而產生的任何誤解或誤釋承擔責任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->