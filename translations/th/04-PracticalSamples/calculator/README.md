# MCP Calculator Tutorial สำหรับผู้เริ่มต้น

## สารบัญ

- [สิ่งที่คุณจะได้เรียนรู้](#สิ่งที่คุณจะได้เรียนรู้)
- [ข้อกำหนดเบื้องต้น](#ข้อกำหนดเบื้องต้น)
- [ทำความเข้าใจกับโครงสร้างโปรเจกต์](#ทำความเข้าใจกับโครงสร้างโปรเจกต์)
- [อธิบายส่วนประกอบหลัก](#อธิบายส่วนประกอบหลัก)
  - [1. แอปพลิเคชันหลัก](#1-แอปพลิเคชันหลัก)
  - [2. บริการเครื่องคิดเลข](#2-บริการเครื่องคิดเลข)
  - [3. ไคลเอ็นต์ MCP โดยตรง](#3-ไคลเอ็นต์-mcp-โดยตรง)
  - [4. ไคลเอ็นต์ที่ขับเคลื่อนด้วย AI](#4-ไคลเอ็นต์ที่ขับเคลื่อนด้วย-ai)
- [การรันตัวอย่าง](#การรันตัวอย่าง)
- [การทำงานร่วมกันทั้งหมด](#การทำงานร่วมกันทั้งหมด)
- [ขั้นตอนถัดไป](#ขั้นตอนถัดไป)

## สิ่งที่คุณจะได้เรียนรู้

บทเรียนนี้อธิบายวิธีการสร้างบริการเครื่องคิดเลขโดยใช้ Model Context Protocol (MCP) คุณจะได้เข้าใจ:

- วิธีสร้างบริการที่ AI สามารถใช้เป็นเครื่องมือ
- วิธีตั้งค่าการสื่อสารโดยตรงกับบริการ MCP
- วิธีที่โมเดล AI สามารถเลือกใช้เครื่องมือโดยอัตโนมัติ
- ความแตกต่างระหว่างการเรียกโปรโตคอลโดยตรงและการมีปฏิสัมพันธ์ที่ช่วยโดย AI

## ข้อกำหนดเบื้องต้น

ก่อนเริ่มต้น ให้แน่ใจว่าคุณมี:
- ติดตั้ง Java 21 หรือสูงกว่า
- Maven สำหรับจัดการไลบรารี
- การปรับใช้โมเดล Azure AI Foundry (จัดเตรียมด้วย `azd up` — ดู [บทที่ 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) ลงชื่อเข้าใช้ด้วย `az login` (การยืนยันตัวตนแบบไม่มีคีย์)
- ความรู้พื้นฐานเกี่ยวกับ Java และ Spring Boot

## ทำความเข้าใจกับโครงสร้างโปรเจกต์

โปรเจกต์เครื่องคิดเลขมีไฟล์สำคัญหลายไฟล์:

```
calculator/
├── src/main/java/com/microsoft/mcp/sample/server/
│   ├── McpServerApplication.java          # Main Spring Boot app
│   └── service/CalculatorService.java     # Calculator operations
└── src/test/java/com/microsoft/mcp/sample/client/
    ├── SDKClient.java                     # Direct MCP communication
    ├── LangChain4jClient.java            # AI-powered client
    └── Bot.java                          # Simple chat interface
```

## อธิบายส่วนประกอบหลัก

### 1. แอปพลิเคชันหลัก

**ไฟล์:** `McpServerApplication.java`

นี่คือจุดเริ่มต้นของบริการเครื่องคิดเลขของเรา เป็นแอป Spring Boot ปกติที่มีการเพิ่มฟีเจอร์พิเศษหนึ่งอย่าง:

```java
@SpringBootApplication
public class McpServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(McpServerApplication.class, args);
    }
    
    @Bean
    public ToolCallbackProvider calculatorTools(CalculatorService calculator) {
        return MethodToolCallbackProvider.builder().toolObjects(calculator).build();
    }
}
```

**สิ่งที่ฟีเจอร์นี้ทำ:**
- เริ่มต้นเว็บเซิร์ฟเวอร์ Spring Boot ที่พอร์ต 8080
- สร้าง `ToolCallbackProvider` ที่ทำให้เมธอดเครื่องคิดเลขของเราอยู่ในรูปแบบเครื่องมือ MCP
- คำสั่ง `@Bean` บอกให้ Spring จัดการเป็นคอมโพเนนต์ที่ส่วนอื่นๆ สามารถใช้ได้

### 2. บริการเครื่องคิดเลข

**ไฟล์:** `CalculatorService.java`

ที่นี่คือที่เกิดการคำนวณทั้งหมด เมธอดแต่ละตัวถูกทำเครื่องหมายด้วย `@Tool` เพื่อให้ใช้ผ่าน MCP ได้:

```java
@Service
public class CalculatorService {

    @Tool(description = "Add two numbers together")
    public String add(double a, double b) {
        double result = a + b;
        return formatResult(a, "+", b, result);
    }

    @Tool(description = "Subtract the second number from the first number")
    public String subtract(double a, double b) {
        double result = a - b;
        return formatResult(a, "-", b, result);
    }
    
    // การดำเนินการเครื่องคิดเลขเพิ่มเติม...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**คุณสมบัติสำคัญ:**

1. **คำสั่ง `@Tool`**: บอก MCC ว่าเมธอดนี้สามารถถูกเรียกโดยไคลเอ็นต์ภายนอก
2. **คำอธิบายชัดเจน**: แต่ละเครื่องมือมีคำอธิบายช่วยให้โมเดล AI เข้าใจว่าเมื่อใดควรใช้งาน
3. **รูปแบบผลลัพธ์ที่สม่ำเสมอ**: การดำเนินการทุกอย่างส่งกลับเป็นสตริงที่มนุษย์เข้าใจ เช่น "5.00 + 3.00 = 8.00"
4. **การจัดการข้อผิดพลาด**: การหารด้วยศูนย์และการหาค่ารากที่เป็นลบจะแจ้งข้อความผิดพลาด

**การดำเนินการที่มีอยู่:**
- `add(a, b)` - บวกสองจำนวน
- `subtract(a, b)` - ลบจำนวนที่สองจากจำนวนแรก
- `multiply(a, b)` - คูณสองจำนวน
- `divide(a, b)` - หารจำนวนแรกด้วยจำนวนที่สอง (ตรวจสอบศูนย์)
- `power(base, exponent)` - การยกกำลังฐานด้วยเลขยกกำลัง
- `squareRoot(number)` - หาค่ารากที่สอง (ตรวจสอบค่าลบ)
- `modulus(a, b)` - คืนเศษจากการหาร
- `absolute(number)` - คืนค่ามูลค่าที่แน่นอน (ค่าสัมบูรณ์)
- `help()` - ให้ข้อมูลเกี่ยวกับการดำเนินการทั้งหมด

### 3. ไคลเอ็นต์ MCP โดยตรง

**ไฟล์:** `SDKClient.java`

ไคลเอ็นต์นี้สื่อสารโดยตรงกับเซิร์ฟเวอร์ MCP โดยไม่ใช้ AI เรียกฟังก์ชันเครื่องคิดเลขเฉพาะด้วยตนเอง:

```java
public class SDKClient {
    
    public static void main(String[] args) {
        McpClientTransport transport = WebFluxSseClientTransport.builder(
            WebClient.builder().baseUrl("http://localhost:8080")
        ).build();
        new SDKClient(transport).run();
    }
    
    public void run() {
        var client = McpClient.sync(this.transport).build();
        client.initialize();
        
        // แสดงรายชื่อเครื่องมือที่มี
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // เรียกใช้ฟังก์ชันเครื่องคิดเลขเฉพาะเจาะจง
        CallToolResult resultAdd = client.callTool(
            new CallToolRequest("add", Map.of("a", 5.0, "b", 3.0))
        );
        System.out.println("Add Result = " + resultAdd);
        
        CallToolResult resultSqrt = client.callTool(
            new CallToolRequest("squareRoot", Map.of("number", 16.0))
        );
        System.out.println("Square Root Result = " + resultSqrt);
        
        client.closeGracefully();
    }
}
```

**สิ่งที่ฟีเจอร์นี้ทำ:**
1. **เชื่อมต่อ** กับเซิร์ฟเวอร์เครื่องคิดเลขที่ `http://localhost:8080` โดยใช้รูปแบบตัวสร้าง (builder pattern)
2. **แสดงรายการ** เครื่องมือทั้งหมด (ฟังก์ชันของเครื่องคิดเลขของเรา)
3. **เรียกใช้** ฟังก์ชันเฉพาะด้วยพารามิเตอร์ที่เจาะจง
4. **พิมพ์** ผลลัพธ์โดยตรง

**หมายเหตุ:** ตัวอย่างนี้ใช้ไลบรารี Spring AI 1.1.0-SNAPSHOT ที่แนะนำรูปแบบตัวสร้างสำหรับ `WebFluxSseClientTransport` หากคุณใช้เวอร์ชันเสถียรก่อนหน้านี้ อาจต้องใช้คอนสตรัคเตอร์โดยตรงแทน

**เมื่อใดควรใช้:** เมื่อคุณรู้ว่าต้องการคำนวณอะไรและต้องการเรียกใช้โปรแกรมโดยตรง

### 4. ไคลเอ็นต์ที่ขับเคลื่อนด้วย AI

**ไฟล์:** `LangChain4jClient.java`

ไคลเอ็นต์นี้ใช้โมเดล AI (GPT-4o-mini) ที่สามารถเลือกใช้เครื่องมือเครื่องคิดเลขโดยอัตโนมัติ:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // ตั้งค่าโมเดล AI (Azure AI Foundry, การรับรองความถูกต้องแบบไม่ใช้คีย์ผ่าน Microsoft Entra ID)
        String endpoint = System.getenv("AZURE_OPENAI_ENDPOINT");
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1";
        String token = new DefaultAzureCredentialBuilder().build()
                .getToken(new TokenRequestContext().addScopes("https://ai.azure.com/.default"))
                .block().getToken();
        ChatLanguageModel model = OpenAiOfficialChatModel.builder()
                .baseUrl(baseUrl)
                .apiKey(token)
                .modelName("gpt-4o-mini")
                .build();

        // เชื่อมต่อกับเซิร์ฟเวอร์เครื่องคิดเลข MCP ของเรา
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // แสดงสิ่งที่ AI กำลังทำอยู่
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // ให้ AI เข้าถึงเครื่องมือเครื่องคิดเลขของเรา
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // สร้างบอท AI ที่สามารถใช้เครื่องคิดเลขของเราได้
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // ตอนนี้เราสามารถขอให้ AI คำนวณด้วยภาษาธรรมชาติได้แล้ว
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**สิ่งที่ฟีเจอร์นี้ทำ:**
1. **สร้าง** การเชื่อมต่อกับโมเดล AI โดยใช้โทเค็น GitHub ของคุณ
2. **เชื่อมต่อ** AI กับเซิร์ฟเวอร์ MCP เครื่องคิดเลขของเรา
3. **ให้** AI เข้าถึงเครื่องมือทั้งหมดของเครื่องคิดเลขเรา
4. **อนุญาต** การร้องขอเป็นภาษาธรรมชาติ เช่น "Calculate the sum of 24.5 and 17.3"

**AI จะทำโดยอัตโนมัติ:**
- เข้าใจว่าคุณต้องการบวกเลข
- เลือกเครื่องมือ `add`
- เรียก `add(24.5, 17.3)`
- ส่งผลลัพธ์กลับในรูปแบบการตอบสนองที่เป็นธรรมชาติ

## การรันตัวอย่าง

### ขั้นตอนที่ 1: เริ่มเซิร์ฟเวอร์เครื่องคิดเลข

ก่อนอื่น ลงชื่อเข้าใช้และตั้งค่าจุดเชื่อมต่อ Azure AI Foundry ของคุณ (จำเป็นสำหรับไคลเอ็นต์ AI — ยืนยันตัวตนแบบไม่มีคีย์ ไม่มีคีย์ API):

**Windows:**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**Linux/macOS:**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

เริ่มต้นเซิร์ฟเวอร์:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

เซิร์ฟเวอร์จะเริ่มที่ `http://localhost:8080` คุณจะเห็น:
```
Started McpServerApplication in X.XXX seconds
```

### ขั้นตอนที่ 2: ทดสอบกับไคลเอ็นต์โดยตรง

ในเทอร์มินัล **ใหม่** ขณะที่เซิร์ฟเวอร์ยังทำงาน อยู่ ให้รันไคลเอ็นต์ MCP โดยตรง:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

คุณจะเห็นผลลัพธ์ประมาณนี้:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### ขั้นตอนที่ 3: ทดสอบกับไคลเอ็นต์ AI

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

คุณจะเห็น AI ใช้งานเครื่องมือโดยอัตโนมัติ:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### ขั้นตอนที่ 4: ปิดเซิร์ฟเวอร์ MCP

เมื่อทดสอบเสร็จแล้ว คุณสามารถหยุดไคลเอ็นต์ AI ได้โดยกด `Ctrl+C` ในเทอร์มินัลของมัน เซิร์ฟเวอร์ MCP จะยังทำงานจนกว่าคุณจะหยุดมัน
เพื่อหยุดเซิร์ฟเวอร์ ให้กด `Ctrl+C` ในเทอร์มินัลที่เซิร์ฟเวอร์กำลังทำงานอยู่

## การทำงานร่วมกันทั้งหมด

นี่คือขั้นตอนทั้งหมดเมื่อคุณถาม AI ว่า "5 + 3 เท่ากับเท่าไร?":

1. **คุณ** ถาม AI เป็นภาษาธรรมชาติ
2. **AI** วิเคราะห์คำขอและเข้าใจว่าคุณต้องการบวกเลข
3. **AI** เรียกเซิร์ฟเวอร์ MCP: `add(5.0, 3.0)`
4. **บริการเครื่องคิดเลข** คำนวณ: `5.0 + 3.0 = 8.0`
5. **บริการเครื่องคิดเลข** ส่งกลับ: `"5.00 + 3.00 = 8.00"`
6. **AI** รับผลและจัดรูปแบบคำตอบเป็นภาษาธรรมชาติ
7. **คุณ** ได้รับคำตอบ: "ผลรวมของ 5 และ 3 คือ 8"

## ขั้นตอนถัดไป

สำหรับตัวอย่างเพิ่มเติม ดูที่ [บทที่ 04: ตัวอย่างการใช้งานจริง](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ปฏิเสธความรับผิดชอบ**:
เอกสารนี้ได้รับการแปลโดยใช้บริการแปลภาษา AI [Co-op Translator](https://github.com/Azure/co-op-translator) ขณะที่เราพยายามให้ความถูกต้อง โปรดทราบว่าการแปลโดยอัตโนมัติอาจมีข้อผิดพลาดหรือความไม่ถูกต้อง เอกสารต้นฉบับในภาษาต้นทางควรถูกพิจารณาเป็นแหล่งข้อมูลที่เชื่อถือได้ สำหรับข้อมูลที่สำคัญ แนะนำให้ใช้การแปลโดยมนุษย์มืออาชีพ เราไม่รับผิดชอบต่อความเข้าใจผิดหรือการตีความที่ผิดพลาดที่เกิดขึ้นจากการใช้การแปลนี้
<!-- CO-OP TRANSLATOR DISCLAIMER END -->