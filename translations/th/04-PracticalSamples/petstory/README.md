# คำแนะนำการสร้างเรื่องราวสัตว์เลี้ยงสำหรับผู้เริ่มต้น

## สารบัญ

- [สิ่งที่ต้องมี](#สิ่งที่ต้องมี)
- [ทำความเข้าใจโครงสร้างโปรเจกต์](#ทำความเข้าใจโครงสร้างโปรเจกต์)
- [อธิบายส่วนประกอบหลัก](#อธิบายส่วนประกอบหลัก)
  - [1. แอปพลิเคชันหลัก](#1-แอปพลิเคชันหลัก)
  - [2. ตัวควบคุมเว็บ](#2-ตัวควบคุมเว็บ)
  - [3. บริการเรื่องราว](#3-บริการเรื่องราว)
  - [4. แม่แบบเว็บ](#4-แม่แบบเว็บ)
  - [5. การตั้งค่า](#5-การตั้งค่า)
- [การรันแอปพลิเคชัน](#การรันแอปพลิเคชัน)
- [วิธีการทำงานร่วมกันทั้งหมด](#วิธีการทำงานร่วมกันทั้งหมด)
- [ทำความเข้าใจการผสานรวม AI](#ทำความเข้าใจการผสานรวม-ai)
- [ขั้นตอนถัดไป](#ขั้นตอนถัดไป)

## สิ่งที่ต้องมี

ก่อนเริ่มต้น ให้แน่ใจว่าคุณมี:
- ติดตั้ง Java 21 หรือสูงกว่า
- Maven สำหรับจัดการ dependencies
- การปรับใช้โมเดล Azure AI Foundry (ตั้งค่าโดยใช้ `azd up` — ดู [บทที่ 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)) และเข้าระบบด้วย `az login` (การยืนยันตัวตนแบบ keyless)
- ความเข้าใจพื้นฐานเกี่ยวกับ Java, Spring Boot, และการพัฒนาเว็บ

## ทำความเข้าใจโครงสร้างโปรเจกต์

โปรเจกต์เรื่องราวสัตว์เลี้ยงมีไฟล์สำคัญหลายไฟล์:

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

## อธิบายส่วนประกอบหลัก

### 1. แอปพลิเคชันหลัก

**ไฟล์:** `PetStoryApplication.java`

นี่คือจุดเริ่มต้นของแอป Spring Boot ของเรา:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**สิ่งที่ทำ:**
- ตัวกำกับ `@SpringBootApplication` เปิดใช้การกำหนดค่าอัตโนมัติและสแกนคอมโพเนนต์
- เริ่มเว็บเซิร์ฟเวอร์ฝังตัว (Tomcat) ที่พอร์ต 8080
- สร้าง Spring beans และบริการทั้งหมดโดยอัตโนมัติ

### 2. ตัวควบคุมเว็บ

**ไฟล์:** `PetController.java`

จัดการคำร้องขอเว็บและปฏิสัมพันธ์ของผู้ใช้ทั้งหมด:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // ส่งกลับเทมเพลต index.html
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // การตรวจสอบความถูกต้องของอินพุต
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // กำจัดข้อมูลอินพุตเพื่อความปลอดภัย
        String sanitizedDescription = sanitizeInput(description);
        
        // สร้างเรื่องราวพร้อมจัดการข้อผิดพลาด
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // ส่งกลับเทมเพลต result.html
            
        } catch (Exception e) {
            // ใช้เรื่องราวสำรองหาก AI ล้มเหลว
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // จำกัดความยาว
    }
}
```

**ฟีเจอร์หลัก:**

1. **จัดการเส้นทาง**: `@GetMapping("/")` แสดงฟอร์มอัปโหลด, `@PostMapping("/generate-story")` ประมวลผลการส่งข้อมูล
2. **ตรวจสอบอินพุต**: ตรวจสอบคำบรรยายที่ว่างเปล่าและจำกัดความยาว
3. **ความปลอดภัย**: ล้างข้อมูลผู้ใช้เพื่อป้องกันการโจมตี XSS
4. **จัดการข้อผิดพลาด**: ให้เรื่องราวสำรองเมื่อบริการ AI ล้มเหลว
5. **ผูกโมเดล**: ส่งข้อมูลไปยังแม่แบบ HTML โดยใช้ `Model` ของ Spring

**ระบบสำรอง:**
ตัวควบคุมมีแม่แบบเรื่องราวที่เขียนไว้ล่วงหน้าซึ่งใช้เมื่อบริการ AI ไม่พร้อมใช้งาน:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // ใช้แฮชคำอธิบายเพื่อความสม่ำเสมอของการตอบสนอง
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. บริการเรื่องราว

**ไฟล์:** `StoryService.java`

บริการนี้สื่อสารกับ Azure AI Foundry เพื่อสร้างเรื่องราวโดยใช้การยืนยันตัวตนแบบ keyless:

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
        
        // จุดเชื่อมต่อที่เข้ากันได้กับ OpenAI ของ Foundry อยู่ภายใต้ /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // การตรวจสอบสิทธิ์แบบไม่ใช้คีย์ด้วย Microsoft Entra ID (ไม่ต้องใช้ API key)
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
        
        // กำหนดการร้องขอ AI
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // จำกัดความยาวของการตอบกลับ
                .temperature(0.8)          // ควบคุมความคิดสร้างสรรค์ (0.0-1.0)
                .build();
        
        // ส่งคำขอและรับการตอบกลับ
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**ส่วนประกอบหลัก:**

1. **ลูกค้า OpenAI**: ใช้ SDK Java อย่างเป็นทางการของ OpenAI ที่ตั้งค่าสำหรับ Azure AI Foundry (แบบ keyless)
2. **ระบบ Prompt**: ตั้งพฤติกรรม AI ให้เขียนเรื่องราวสัตว์เลี้ยงในแบบเหมาะสมกับครอบครัว
3. **ผู้ใช้ Prompt**: บอก AI ว่าเขียนเรื่องอะไรโดยอิงจากคำบรรยาย
4. **พารามิเตอร์**: ควบคุมความยาวและระดับความสร้างสรรค์ของเรื่อง
5. **การจัดการข้อผิดพลาด**: ขว้างข้อยกเว้นที่ตัวควบคุมจับและจัดการ

### 4. แม่แบบเว็บ

**ไฟล์:** `index.html` (ฟอร์มอัปโหลด)

หน้าหลักที่ผู้ใช้บรรยายสัตว์เลี้ยงของตน:

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

**ไฟล์:** `result.html` (แสดงเรื่องราว)

แสดงเรื่องราวที่สร้างขึ้น:

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

**ฟีเจอร์ของแม่แบบ:**

1. **ผสาน Thymeleaf**: ใช้แอตทริบิวต์ `th:` สำหรับเนื้อหาไดนามิก
2. **ออกแบบตอบสนอง**: การจัดแต่ง CSS สำหรับมือถือและเดสก์ท็อป
3. **จัดการข้อผิดพลาด**: แสดงข้อผิดพลาดจากการตรวจสอบข้อมูลให้ผู้ใช้ดู
4. **ประมวลผลฝั่งไคลเอนต์**: JavaScript สำหรับวิเคราะห์ภาพ (ใช้ Transformers.js)

### 5. การตั้งค่า

**ไฟล์:** `application.properties`

การตั้งค่าการกำหนดค่าสำหรับแอป:

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

**อธิบายการตั้งค่า:**

1. **อัปโหลดไฟล์**: อนุญาตภาพขนาดสูงสุด 10MB
2. **การบันทึกข้อมูล**: ควบคุมข้อมูลที่บันทึกขณะรัน
3. **Azure AI Foundry**: ระบุ endpoint และการปรับใช้โมเดลที่ใช้ (การยืนยันตัวตนแบบ keyless)
4. **ความปลอดภัย**: การตั้งค่าจัดการข้อผิดพลาดเพื่อไม่เผยข้อมูลสำคัญ

## การรันแอปพลิเคชัน

### ขั้นตอนที่ 1: เข้าสู่ระบบและตั้งค่า Endpoint

การยืนยันตัวตนแบบ keyless (Microsoft Entra ID) จึงไม่มี API key ลงชื่อเข้าใช้และตั้งค่า endpoint ของ Foundry:

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

**เหตุผลที่ต้องตั้งค่า:**
- Azure AI Foundry ใช้ Microsoft Entra ID ในการยืนยันคำขอการอนุมาน
- การยืนยันตัวตนแบบ keyless หมายถึงไม่มีความลับในซอร์สโค้ดหรือสิ่งแวดล้อม
- บัญชีของคุณต้องมีบทบาท **Cognitive Services OpenAI User** บนทรัพยากร

### ขั้นตอนที่ 2: สร้างและรัน

ไปยังไดเรกทอรีโปรเจกต์:
```bash
cd 04-PracticalSamples/petstory
```

สร้างแอป:
```bash
mvn clean compile
```

เริ่มเซิร์ฟเวอร์:
```bash
mvn spring-boot:run
```

แอปจะเริ่มที่ `http://localhost:8080`

### ขั้นตอนที่ 3: ทดสอบแอปพลิเคชัน

1. **เปิด** `http://localhost:8080` ในเว็บเบราว์เซอร์ของคุณ
2. **บรรยาย** สัตว์เลี้ยงของคุณในพื้นที่ข้อความ (เช่น "สุนัขพันธุ์โกลเด้น รีทรีฟเวอร์ ซุกซนและชอบเอาของมาให้")
3. **คลิก** "Generate Story" เพื่อรับเรื่องราวที่ AI สร้างขึ้น
4. **หรือ** อัปโหลดภาพสัตว์เลี้ยงเพื่อทำคำบรรยายโดยอัตโนมัติ
5. **ดู** เรื่องราวสร้างสรรค์ตามคำบรรยายของสัตว์เลี้ยงคุณ

## วิธีการทำงานร่วมกันทั้งหมด

นี่คือกระบวนการทั้งหมดเมื่อคุณสร้างเรื่องราวสัตว์เลี้ยง:

1. **ข้อมูลผู้ใช้**: คุณบรรยายสัตว์เลี้ยงในฟอร์มเว็บ
2. **ส่งฟอร์ม**: เบราว์เซอร์ส่งคำขอ POST ไปที่ `/generate-story`
3. **การประมวลผลตัวควบคุม**: `PetController` ตรวจสอบและล้างข้อมูลอินพุต
4. **เรียกบริการ AI**: `StoryService` ส่งคำขอไปยังโมเดล Azure AI Foundry
5. **สร้างเรื่องราว**: AI สร้างเรื่องราวที่สร้างสรรค์จากคำบรรยาย
6. **จัดการตอบกลับ**: ตัวควบคุมรับเรื่องราวและเพิ่มลงในโมเดล
7. **เรนเดอร์แม่แบบ**: Thymeleaf แสดงผล `result.html` พร้อมเรื่องราว
8. **แสดงผล**: ผู้ใช้เห็นเรื่องราวที่สร้างในเบราว์เซอร์

**กระบวนการจัดการข้อผิดพลาด:**
ถ้าบริการ AI ล้มเหลว:
1. ตัวควบคุมจับข้อยกเว้น
2. สร้างเรื่องสำรองโดยใช้แม่แบบที่เขียนไว้ล่วงหน้า
3. แสดงเรื่องสำรองพร้อมแจ้งเตือนว่า AI ใช้งานไม่ได้
4. ผู้ใช้ยังได้เรื่องราวเพื่อประสบการณ์ที่ดี

## ทำความเข้าใจการผสานรวม AI

### Azure AI Foundry (keyless)
แอปนี้ใช้ Azure AI Foundry ด้วยการยืนยันตัวตนแบบ keyless (Microsoft Entra ID):

```java
// การตรวจสอบสิทธิ์แบบไม่ใช้คีย์ - ไม่มีคีย์ API
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### การออกแบบ Prompt
บริการใช้ prompt ที่รังสรรค์อย่างระมัดระวังเพื่อผลลัพธ์ที่ดี:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### การประมวลผลตอบกลับ
ตอบกลับจาก AI จะถูกดึงข้อมูลและตรวจสอบ:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## ขั้นตอนถัดไป

สำหรับตัวอย่างเพิ่มเติม ดู [บทที่ 04: ตัวอย่างใช้งานจริง](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ปฏิเสธความรับผิดชอบ**:
เอกสารนี้ได้รับการแปลโดยใช้บริการแปลภาษา AI [Co-op Translator](https://github.com/Azure/co-op-translator) ขณะที่เราพยายามให้ความถูกต้อง โปรดทราบว่าการแปลโดยอัตโนมัติอาจมีข้อผิดพลาดหรือความไม่ถูกต้อง เอกสารต้นฉบับในภาษาต้นทางควรถูกพิจารณาเป็นแหล่งข้อมูลที่เชื่อถือได้ สำหรับข้อมูลที่สำคัญ แนะนำให้ใช้การแปลโดยมนุษย์มืออาชีพ เราไม่รับผิดชอบต่อความเข้าใจผิดหรือการตีความที่ผิดพลาดที่เกิดขึ้นจากการใช้การแปลนี้
<!-- CO-OP TRANSLATOR DISCLAIMER END -->