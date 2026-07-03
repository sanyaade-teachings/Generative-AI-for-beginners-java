# คู่มือการใช้เครื่องคิดเลข MCP สำหรับมือใหม่

## สารบัญ

- [สิ่งที่คุณจะได้เรียนรู้](#สิ่งที่คุณจะได้เรียนรู้)
- [ข้อกำหนดเบื้องต้น](#ข้อกำหนดเบื้องต้น)
- [ทำความเข้าใจโครงสร้างโปรเจกต์](#ทำความเข้าใจโครงสร้างโปรเจกต์)
- [อธิบายส่วนประกอบหลัก](#อธิบายส่วนประกอบหลัก)
  - [1. แอปพลิเคชันหลัก](#1-แอปพลิเคชันหลัก)
  - [2. บริการเครื่องคิดเลข](#2-บริการเครื่องคิดเลข)
  - [3. ไคลเอนต์ MCP แบบตรง](#3-ไคลเอนต์-mcp-แบบตรง)
  - [4. ไคลเอนต์ที่ใช้ AI](#4-ไคลเอนต์ที่ใช้-ai)
- [การรันตัวอย่าง](#การรันตัวอย่าง)
- [การทำงานร่วมกันทั้งหมด](#การทำงานร่วมกันทั้งหมด)
- [ขั้นตอนถัดไป](#ขั้นตอนถัดไป)

## สิ่งที่คุณจะได้เรียนรู้

คำแนะนำนี้อธิบายวิธีสร้างบริการเครื่องคิดเลขโดยใช้ Model Context Protocol (MCP) คุณจะได้เข้าใจ:

- วิธีสร้างบริการที่ AI สามารถใช้เป็นเครื่องมือ
- วิธีตั้งค่าการสื่อสารโดยตรงกับบริการ MCP
- วิธีที่โมเดล AI เลือกใช้เครื่องมือโดยอัตโนมัติ
- ความแตกต่างระหว่างการเรียกโปรโตคอลโดยตรงกับการโต้ตอบที่มี AI ช่วยเหลือ

## ข้อกำหนดเบื้องต้น

ก่อนเริ่มต้น ตรวจสอบให้แน่ใจว่าคุณมี:
- Java 21 หรือสูงกว่า ติดตั้งไว้
- Maven สำหรับจัดการ dependencies
- การปรับใช้โมเดล Azure AI Foundry (สร้างด้วยคำสั่ง `azd up` — ดู [บทที่ 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) ลงชื่อเข้าใช้ด้วย `az login` (แก้ไขการพิสูจน์ตัวตนแบบไม่มีคีย์)
- ความเข้าใจพื้นฐานเกี่ยวกับ Java และ Spring Boot

## ทำความเข้าใจโครงสร้างโปรเจกต์

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

นี่คือจุดเริ่มต้นของบริการเครื่องคิดเลขของเรา เป็นแอปพลิเคชัน Spring Boot มาตรฐานที่มีการเพิ่มพิเศษหนึ่งอย่าง:

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

**สิ่งที่ทำ:**
- เริ่มต้นเว็บเซิร์ฟเวอร์ Spring Boot ที่พอร์ต 8080
- สร้าง `ToolCallbackProvider` ที่ทำให้เมธอดเครื่องคิดเลขของเราพร้อมใช้งานเป็นเครื่องมือ MCP
- คำสั่ง `@Bean` บอกให้ Spring จัดการคอมโพเนนต์นี้เพื่อให้ส่วนอื่นใช้ได้

### 2. บริการเครื่องคิดเลข

**ไฟล์:** `CalculatorService.java`

ที่นี่คือจุดที่คำนวณทั้งหมดเกิดขึ้น โดยแต่ละเมธอดจะถูกติดแท็กด้วย `@Tool` เพื่อให้สามารถเรียกใช้ผ่าน MCP:

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

1. **คำสั่ง `@Tool`**: บอก MCP ว่าสามารถเรียกเมธอดนี้โดยไคลเอนต์ภายนอกได้
2. **คำอธิบายชัดเจน**: แต่ละเครื่องมือมีคำอธิบายช่วยให้โมเดล AI เข้าใจเวลาที่ควรใช้งาน
3. **รูปแบบผลลัพธ์สม่ำเสมอ**: ทุกการดำเนินการส่งคืนเป็นสตริงที่อ่านง่าย เช่น "5.00 + 3.00 = 8.00"
4. **จัดการข้อผิดพลาด**: หารด้วยศูนย์และรากที่เป็นลบจะคืนข้อความแจ้งข้อผิดพลาด

**การดำเนินการที่มี:**
- `add(a, b)` - บวกเลขสองจำนวน
- `subtract(a, b)` - ลบจำนวนที่สองจากจำนวนแรก
- `multiply(a, b)` - คูณเลขสองจำนวน
- `divide(a, b)` - หารจำนวนแรกด้วยจำนวนที่สอง (ตรวจสอบศูนย์)
- `power(base, exponent)` - ยกกำลังฐานด้วยเลขชี้กำลัง
- `squareRoot(number)` - คำนวณรากที่สอง (ตรวจสอบค่าติดลบ)
- `modulus(a, b)` - คืนเศษจากการหาร
- `absolute(number)` - คืนค่าค่าสัมบูรณ์
- `help()` - คืนข้อมูลเกี่ยวกับการดำเนินการทั้งหมด

### 3. ไคลเอนต์ MCP แบบตรง

**ไฟล์:** `SDKClient.java`

ไคลเอนต์นี้สื่อสารโดยตรงกับเซิร์ฟเวอร์ MCP โดยไม่ใช้ AI โดยเรียกฟังก์ชันเครื่องคิดเลขเฉพาะด้วยตนเอง:

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
        
        // แสดงรายการเครื่องมือที่มี
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // เรียกใช้ฟังก์ชันเครื่องคำนวณเฉพาะ
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

**สิ่งที่ทำ:**
1. **เชื่อมต่อ** กับเซิร์ฟเวอร์เครื่องคิดเลขที่ `http://localhost:8080` โดยใช้รูปแบบ builder
2. **แสดงรายการ** เครื่องมือทั้งหมด (ฟังก์ชันเครื่องคิดเลขของเรา)
3. **เรียกใช้** ฟังก์ชันเฉพาะพร้อมพารามิเตอร์ที่ระบุ
4. **แสดงผลลัพธ์** โดยตรง

**หมายเหตุ:** ตัวอย่างนี้ใช้ dependency Spring AI 1.1.0-SNAPSHOT ซึ่งเพิ่มรูปแบบ builder สำหรับ `WebFluxSseClientTransport` หากคุณใช้เวอร์ชันเก่าที่เสถียร อาจต้องใช้ constructor โดยตรงแทน

**เมื่อต้องใช้:** เมื่อคุณรู้ว่าต้องการคำนวณอะไรและต้องการเรียกใช้โปรแกรมโดยตรง

### 4. ไคลเอนต์ที่ใช้ AI

**ไฟล์:** `LangChain4jClient.java`

ไคลเอนต์นี้ใช้โมเดล AI (GPT-4o-mini) ที่สามารถตัดสินใจโดยอัตโนมัติว่าเครื่องมือเครื่องคิดเลขใดควรใช้:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // ตั้งค่ารูปแบบ AI (Azure AI Foundry, การยืนยันตัวตนแบบไม่ต้องใช้คีย์ผ่าน Microsoft Entra ID)
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

        // ตอนนี้เราสามารถขอให้ AI ทำการคำนวณด้วยภาษาธรรมชาติได้แล้ว
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**สิ่งที่ทำ:**
1. **สร้าง** การเชื่อมต่อโมเดล AI โดยใช้การพิสูจน์ตัวตนแบบไม่มีคีย์ (Microsoft Entra ID)
2. **เชื่อมต่อ** AI กับเซิร์ฟเวอร์ MCP เครื่องคิดเลขของเรา
3. **มอบ** สิทธิ์ให้ AI ใช้เครื่องมือเครื่องคิดเลขทั้งหมดของเรา
4. **อนุญาต** ให้ส่งคำขอแบบภาษาธรรมชาติ เช่น "คำนวณผลรวมของ 24.5 และ 17.3"

**AI ทำงานโดยอัตโนมัติ:**
- เข้าใจว่าคุณต้องการบวกเลข
- เลือกเครื่องมือ `add`
- เรียก `add(24.5, 17.3)`
- ส่งผลลัพธ์ในรูปแบบคำตอบธรรมชาติ

## การรันตัวอย่าง

### ขั้นตอนที่ 1: เริ่มเซิร์ฟเวอร์เครื่องคิดเลข

ก่อนอื่น ลงชื่อเข้าใช้และตั้งค่า Azure AI Foundry endpoint ของคุณ (จำเป็นสำหรับไคลเอนต์ AI — การพิสูจน์ตัวตนแบบไม่มีคีย์ ไม่มี API key):

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

เริ่มเซิร์ฟเวอร์:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

เซิร์ฟเวอร์จะเริ่มต้นที่ `http://localhost:8080` คุณควรเห็น:
```
Started McpServerApplication in X.XXX seconds
```

### ขั้นตอนที่ 2: ทดสอบด้วยไคลเอนต์แบบตรง

ในเทอร์มินัล **ใหม่** โดยเซิร์ฟเวอร์ยังทำงานอยู่ ให้รันไคลเอนต์ MCP แบบตรง:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

คุณจะเห็นผลลัพธ์เช่น:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### ขั้นตอนที่ 3: ทดสอบด้วยไคลเอนต์ AI

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

คุณจะเห็นว่า AI ใช้เครื่องมือโดยอัตโนมัติ:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### ขั้นตอนที่ 4: ปิดเซิร์ฟเวอร์ MCP

เมื่อทดสอบเสร็จ คุณสามารถหยุดไคลเอนต์ AI ได้โดยกด `Ctrl+C` ในเทอร์มินัลของมัน เซิร์ฟเวอร์ MCP จะยังคงทำงานจนกว่าคุณจะหยุด
เพื่อหยุดเซิร์ฟเวอร์ ให้กด `Ctrl+C` ในเทอร์มินัลที่รันเซิร์ฟเวอร์อยู่

## การทำงานร่วมกันทั้งหมด

นี่คือลำดับขั้นตอนเมื่อคุณถาม AI ว่า "5 + 3 เท่ากับอะไร?":

1. **คุณ** ถาม AI ด้วยภาษาธรรมชาติ
2. **AI** วิเคราะห์คำขอและรู้ว่าคุณต้องการบวกเลข
3. **AI** เรียกเซิร์ฟเวอร์ MCP: `add(5.0, 3.0)`
4. **บริการเครื่องคิดเลข** คำนวณ: `5.0 + 3.0 = 8.0`
5. **บริการเครื่องคิดเลข** ส่งคืน: `"5.00 + 3.00 = 8.00"`
6. **AI** รับผลและจัดรูปแบบคำตอบเป็นธรรมชาติ
7. **คุณ** ได้รับคำตอบ: "ผลรวมของ 5 และ 3 คือ 8"

## ขั้นตอนถัดไป

สำหรับตัวอย่างเพิ่มเติม โปรดดู [บทที่ 04: ตัวอย่างใช้งานจริง](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ปฏิเสธความรับผิดชอบ**:
เอกสารนี้ได้รับการแปลโดยใช้บริการแปลภาษา AI [Co-op Translator](https://github.com/Azure/co-op-translator) ขณะที่เราพยายามให้ความถูกต้อง โปรดทราบว่าการแปลโดยอัตโนมัติอาจมีข้อผิดพลาดหรือความไม่ถูกต้อง เอกสารต้นฉบับในภาษาต้นทางควรถูกพิจารณาเป็นแหล่งข้อมูลที่เชื่อถือได้ สำหรับข้อมูลที่สำคัญ แนะนำให้ใช้การแปลโดยมนุษย์มืออาชีพ เราไม่รับผิดชอบต่อความเข้าใจผิดหรือการตีความที่ผิดพลาดที่เกิดขึ้นจากการใช้การแปลนี้
<!-- CO-OP TRANSLATOR DISCLAIMER END -->