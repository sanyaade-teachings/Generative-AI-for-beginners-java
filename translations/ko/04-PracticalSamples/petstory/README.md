# 초보자를 위한 반려동물 이야기 생성기 튜토리얼

## 목차

- [필수 조건](#필수-조건)
- [프로젝트 구조 이해](#프로젝트-구조-이해)
- [핵심 구성 요소 설명](#핵심-구성-요소-설명)
  - [1. 메인 애플리케이션](#1-메인-애플리케이션)
  - [2. 웹 컨트롤러](#2-웹-컨트롤러)
  - [3. 스토리 서비스](#3-스토리-서비스)
  - [4. 웹 템플릿](#4-웹-템플릿)
  - [5. 설정](#5-설정)
- [애플리케이션 실행하기](#애플리케이션-실행하기)
- [전체 작동 원리](#전체-작동-원리)
- [AI 통합 이해하기](#ai-통합-이해하기)
- [다음 단계](#다음-단계)

## 필수 조건

시작하기 전에, 다음이 준비되어 있는지 확인하세요:
- Java 21 이상 설치
- 의존성 관리를 위한 Maven
- Azure AI Foundry 모델 배포 ( `azd up` 명령어로 프로비저닝 — [2장](../../02-SetupDevEnvironment/getting-started-azure-openai.md) 참고), `az login` 으로 로그인 (키리스 인증)
- Java, Spring Boot, 웹 개발에 대한 기본 지식

## 프로젝트 구조 이해

반려동물 이야기 프로젝트는 여러 중요한 파일로 구성되어 있습니다:

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

## 핵심 구성 요소 설명

### 1. 메인 애플리케이션

**파일:** `PetStoryApplication.java`

Spring Boot 애플리케이션의 진입점입니다:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**주요 역할:**
- `@SpringBootApplication` 어노테이션으로 자동 설정 및 컴포넌트 스캔 활성화
- 내장 웹 서버(Tomcat)를 8080 포트에서 실행
- 필요한 모든 Spring 빈 및 서비스 자동 생성

### 2. 웹 컨트롤러

**파일:** `PetController.java`

웹 요청과 사용자 상호작용을 처리합니다:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // index.html 템플릿을 반환합니다
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // 입력 검증
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // 보안을 위해 입력값을 정리합니다
        String sanitizedDescription = sanitizeInput(description);
        
        // 오류 처리를 포함한 이야기 생성
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // result.html 템플릿을 반환합니다
            
        } catch (Exception e) {
            // AI 실패 시 대체 이야기 사용
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // 길이 제한
    }
}
```

**핵심 기능:**

1. **라우트 처리**: `@GetMapping("/")`는 업로드 폼 표시, `@PostMapping("/generate-story")`는 제출 처리
2. **입력 검증**: 빈 설명 및 길이 제한 확인
3. <strong>보안</strong>: 교차 사이트 스크립팅(XSS) 공격 방지 위해 사용자 입력 정화
4. **오류 처리**: AI 서비스 실패 시 대체 이야기 제공
5. **모델 바인딩**: Spring `Model`로 HTML 템플릿에 데이터 전달

**대체 시스템:**
컨트롤러에는 AI 서비스가 사용 불가능할 때 사용하는 미리 작성된 이야기 템플릿이 포함되어 있습니다:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // 일관된 응답을 위해 설명 해시를 사용하세요
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. 스토리 서비스

**파일:** `StoryService.java`

이 서비스는 키리스 인증으로 Azure AI Foundry와 통신하여 이야기를 생성합니다:

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
        
        // Foundry의 OpenAI 호환 엔드포인트는 /openai/v1/ 아래에 있습니다
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Microsoft Entra ID로 키 없는 인증(API 키 필요 없음)
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
        
        // AI 요청 구성
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // 응답 길이 제한
                .temperature(0.8)          // 창의성 조절(0.0-1.0)
                .build();
        
        // 요청을 보내고 응답 받기
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**주요 구성 요소:**

1. **OpenAI 클라이언트**: Azure AI Foundry용 공식 OpenAI Java SDK를 사용 (키리스)
2. **시스템 프롬프트**: AI가 가족 친화적인 반려동물 이야기를 쓰도록 설정
3. **사용자 프롬프트**: 설명에 따라 AI에 정확히 어떤 이야기를 쓸지 지시
4. <strong>파라미터</strong>: 이야기 길이와 창의성 수준 제어
5. **오류 처리**: 컨트롤러가 처리할 예외를 던짐

### 4. 웹 템플릿

**파일:** `index.html` (업로드 폼)

사용자가 반려동물을 설명하는 메인 페이지:

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

**파일:** `result.html` (스토리 표시)

생성된 이야기를 보여줍니다:

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

**템플릿 특징:**

1. **Thymeleaf 통합**: 동적 콘텐츠를 위한 `th:` 속성 사용
2. **반응형 디자인**: 모바일 및 데스크톱용 CSS 스타일링
3. **오류 처리**: 사용자에게 검증 오류 표시
4. **클라이언트 사이드 처리**: 이미지 분석용 JavaScript (Transformers.js 사용)

### 5. 설정

**파일:** `application.properties`

애플리케이션 설정:

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

**설정 상세:**

1. **파일 업로드**: 최대 10MB 이미지 허용
2. <strong>로깅</strong>: 실행 중 기록할 정보 제어
3. **Azure AI Foundry**: 사용될 엔드포인트와 모델 배포 지정 (키리스 인증)
4. <strong>보안</strong>: 민감한 정보 노출 방지 위한 오류 처리 설정

## 애플리케이션 실행하기

### 1단계: 로그인 및 엔드포인트 설정

인증은 키리스(Microsoft Entra ID)로 이루어지며 API 키가 필요 없습니다. 로그인 후 Foundry 엔드포인트를 설정하세요:

**Windows (명령 프롬프트):**
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

**이 단계가 필요한 이유:**
- Azure AI Foundry는 Microsoft Entra ID를 사용해 추론 요청을 인증합니다
- 키리스 인증으로 소스 코드나 환경에 비밀값이 없습니다
- 계정에 **Cognitive Services OpenAI User** 역할이 리소스에 할당되어 있어야 합니다

### 2단계: 빌드 및 실행

프로젝트 디렉토리로 이동하세요:
```bash
cd 04-PracticalSamples/petstory
```

애플리케이션 빌드:
```bash
mvn clean compile
```

서버 시작:
```bash
mvn spring-boot:run
```

애플리케이션은 `http://localhost:8080` 에서 실행됩니다.

### 3단계: 애플리케이션 테스트

1. 브라우저에서 `http://localhost:8080` 열기
2. 텍스트 영역에 반려동물 설명 입력 (예: "공을 가져오는 것을 좋아하는 장난꾸러기 골든 리트리버")
3. "Generate Story" 버튼 클릭하여 AI가 생성한 이야기 받기
4. 또는 반려동물 사진을 업로드하여 자동으로 설명 생성
5. 반려동물 설명을 바탕으로 창의적인 이야기 보기

## 전체 작동 원리

반려동물 이야기를 생성할 때의 전체 흐름:

1. **사용자 입력**: 웹 폼에서 반려동물을 설명
2. **폼 제출**: 브라우저가 `/generate-story`에 POST 요청 전송
3. **컨트롤러 처리**: `PetController`가 입력 검증 및 정화 수행
4. **AI 서비스 호출**: `StoryService`가 Azure AI Foundry 모델에 요청 발송
5. **이야기 생성**: AI가 설명을 바탕으로 창의적 이야기 생성
6. **응답 처리**: 컨트롤러가 이야기를 받아 모델에 추가
7. **템플릿 렌더링**: Thymeleaf가 `result.html`을 이야기와 함께 렌더링
8. <strong>표시</strong>: 사용자가 브라우저에서 생성된 이야기 확인

**오류 처리 흐름:**
AI 서비스 실패 시:
1. 컨트롤러가 예외를 잡음
2. 미리 작성된 템플릿으로 대체 이야기 생성
3. AI 사용 불가 알림과 함께 대체 이야기 표시
4. 사용자 경험을 유지하며 이야기를 제공

## AI 통합 이해하기

### Azure AI Foundry (키리스)
애플리케이션은 Microsoft Entra ID를 이용한 키리스 인증으로 Azure AI Foundry를 활용합니다:

```java
// 키 없는 인증 - API 키 없음
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### 프롬프트 구성
더 나은 결과를 위해 정성껏 작성된 프롬프트를 사용합니다:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### 응답 처리
AI의 응답을 추출하고 검증합니다:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## 다음 단계

추가 예제는 [4장: 실용 샘플](../README.md)에서 확인하세요

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**면책 조항**:
이 문서는 AI 번역 서비스 [Co-op Translator](https://github.com/Azure/co-op-translator)를 사용하여 번역되었습니다. 정확성을 기하기 위해 노력하고 있으나, 자동 번역은 오류나 부정확한 부분이 있을 수 있음을 유의하시기 바랍니다. 원본 문서의 원어본이 권위 있는 자료로 간주되어야 합니다. 중요한 정보의 경우, 전문가의 인간 번역을 권장합니다. 이 번역 사용으로 인해 발생하는 오해나 잘못된 해석에 대해 당사는 책임을 지지 않습니다.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->