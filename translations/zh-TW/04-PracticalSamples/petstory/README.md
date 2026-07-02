# 寵物故事生成器初學者教學

## 目錄

- [先決條件](#先決條件)
- [了解專案結構](#了解專案結構)
- [核心組件說明](#核心組件說明)
  - [1. 主應用程式](#1-主應用程式)
  - [2. 網頁控制器](#2-網頁控制器)
  - [3. 故事服務](#3-故事服務)
  - [4. 網頁範本](#4-網頁範本)
  - [5. 設定](#5-設定)
- [執行應用程式](#執行應用程式)
- [運作流程說明](#運作流程說明)
- [理解 AI 整合](#理解-ai-整合)
- [後續步驟](#後續步驟)

## 先決條件

開始之前，請確保您已經具備：
- 安裝 Java 21 或更高版本
- 具備 Maven 依賴管理工具
- Azure AI Foundry 模型部署（使用 `azd up` 進行佈署 — 請參閱[第二章](../../02-SetupDevEnvironment/getting-started-azure-openai.md)），並使用 `az login` 登入（無需 API 金鑰的驗證）
- 基本的 Java、Spring Boot 及網頁開發知識

## 了解專案結構

寵物故事專案包含幾個重要檔案：

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
- 啟動嵌入式網頁伺服器 (Tomcat)，監聽 8080 埠號
- 自動建立所有必需的 Spring Bean 與服務

### 2. 網頁控制器

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
        return "index";  // 回傳 index.html 範本
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
        
        // 針對安全性清理輸入
        String sanitizedDescription = sanitizeInput(description);
        
        // 生成故事並處理錯誤
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // 回傳 result.html 範本
            
        } catch (Exception e) {
            // 如果 AI 失敗則使用備用故事
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

1. <strong>路由處理</strong>：`@GetMapping("/")` 顯示上傳表單，`@PostMapping("/generate-story")` 處理提交請求
2. <strong>輸入驗證</strong>：檢查描述是否為空或超過限制長度
3. <strong>安全性</strong>：清理使用者輸入，防止 XSS 攻擊
4. <strong>錯誤處理</strong>：當 AI 服務失敗時提供備用故事
5. <strong>模型綁定</strong>：使用 Spring 的 `Model` 將資料傳給 HTML 範本

**備用系統：**
控制器包含預先撰寫的故事範本，當 AI 服務無法使用時將使用這些範本：

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // 使用描述雜湊以確保回應一致性
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. 故事服務

**檔案：** `StoryService.java`

此服務透過無需金鑰的認證，與 Azure AI Foundry 進行故事生成的通訊：

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
        
        // 使用 Microsoft Entra ID 進行無金鑰身份驗證（無需 API 密鑰）
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
        
        // 設定 AI 請求
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // 限制回應長度
                .temperature(0.8)          // 控制創意度（0.0-1.0）
                .build();
        
        // 發送請求並取得回應
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**關鍵元件：**

1. **OpenAI 用戶端**：使用官方 OpenAI Java SDK，且針對 Azure AI Foundry（無需金鑰）進行設定
2. <strong>系統提示</strong>：設定 AI 的行為為撰寫適合家庭閱讀的寵物故事
3. <strong>使用者提示</strong>：告訴 AI 根據描述精確產生故事
4. <strong>參數</strong>：控制故事長度及創意程度
5. <strong>錯誤處理</strong>：拋出例外，供控制器捕捉並處理

### 4. 網頁範本

**檔案：** `index.html`（上傳表單）

使用者描述寵物的主要頁面：

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

**檔案：** `result.html`（故事顯示）

顯示生成出來的故事：

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

**範本特色：**

1. **Thymeleaf 整合**：使用 `th:` 屬性支援動態內容
2. <strong>響應式設計</strong>：CSS 樣式同時適用於行動裝置及桌面環境
3. <strong>錯誤回饋</strong>：顯示驗證錯誤給使用者
4. <strong>前端處理</strong>：JavaScript 進行圖片分析（使用 Transformers.js）

### 5. 設定

**檔案：** `application.properties`

應用程式的設定參數：

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

1. <strong>檔案上傳</strong>：允許最大 10MB 的圖片上傳
2. <strong>日誌紀錄</strong>：控制執行時哪些資訊會被寫入日誌
3. **Azure AI Foundry**：指定使用的端點與模型部署（無需金鑰認證）
4. <strong>安全性</strong>：錯誤處理設定避免暴露敏感資料

## 執行應用程式

### 第一步：登入並設定端點

採用無需金鑰的認證（Microsoft Entra ID），無需 API 金鑰。請登入並指定您的 Foundry 端點：

**Windows (命令提示字元)：**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**Windows (PowerShell)：**
```powershell
az login
$env:AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
```

**Linux/macOS：**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**必要原因：**
- Azure AI Foundry 使用 Microsoft Entra ID 驗證推論請求
- 無需金鑰認證，即不需在原始碼或環境變數中放置秘密
- 您的帳戶在該資源需要具備 **Cognitive Services OpenAI User** 角色權限

### 第二步：編譯並執行

切換至專案目錄：
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

應用程式會在 `http://localhost:8080` 啟動。

### 第三步：測試應用程式

1. <strong>開啟</strong> 瀏覽器並輸入 `http://localhost:8080`
2. <strong>描述</strong> 您的寵物（例如：「一隻喜歡玩接球的黃金獵犬」）
3. <strong>點擊</strong> 「Generate Story」取得 AI 生成的故事
4. <strong>或者</strong> 上傳寵物圖片，自動生成描述
5. <strong>瀏覽</strong> 根據描述產生的創意故事

## 運作流程說明

當您產生寵物故事時的完整流程如下：

1. <strong>使用者輸入</strong>：在網頁表單描述您的寵物
2. <strong>表單提交</strong>：瀏覽器送出 POST 請求至 `/generate-story`
3. <strong>控制器處理</strong>：`PetController` 驗證並清理輸入資料
4. **AI 服務呼叫**：`StoryService` 向 Azure AI Foundry 模型發出請求
5. <strong>故事生成</strong>：AI 根據描述產生創意故事
6. <strong>回應處理</strong>：控制器接收故事並加入模型
7. <strong>範本渲染</strong>：Thymeleaf 產生 `result.html` 並帶入故事內容
8. <strong>顯示結果</strong>：使用者在瀏覽器看到產生的故事

**錯誤處理流程：**
若 AI 服務失敗：
1. 控制器捕捉例外狀況
2. 使用預先撰寫的備用故事範本進行生成
3. 顯示備用故事並註明 AI 服務目前無法使用
4. 確保使用者仍能取得故事，維持良好使用體驗

## 理解 AI 整合

### Azure AI Foundry（無需金鑰）
應用程式採用 Azure AI Foundry 結合無需金鑰認證（Microsoft Entra ID）：

```java
// 無鑰匙認證 - 無 API 金鑰
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### 提示語工程
服務使用精心設計的提示語取得良好結果：

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### 回應處理
從 AI 回應中擷取並驗證有效資料：

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## 後續步驟

更多範例請見 [第四章：實務範例](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**免責聲明**：
此文件已使用 AI 翻譯服務 [Co-op Translator](https://github.com/Azure/co-op-translator) 進行翻譯。雖然我們努力追求準確性，但請注意自動翻譯可能包含錯誤或不準確之處。原始文件的母語版本應視為權威來源。對於關鍵資訊，建議採用專業人工翻譯。我們不對因使用此翻譯所產生的任何誤解或誤譯承擔責任。
<!-- CO-OP TRANSLATOR DISCLAIMER END -->