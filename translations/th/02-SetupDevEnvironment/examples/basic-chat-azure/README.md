# ตัวอย่างการแชทพื้นฐานกับ Azure AI Foundry - ตัวอย่างตั้งแต่ต้นจนจบ

ตัวอย่างนี้เป็นแอปพลิเคชัน Spring Boot ง่ายๆ ที่เชื่อมต่อกับโมเดล **Azure AI Foundry** โดยใช้ **การตรวจสอบสิทธิ์แบบไม่ใช้คีย์** (Microsoft Entra ID) และทดสอบการตั้งค่าของคุณ โดยใช้ `ChatClient` ของ Spring AI

## สารบัญ

- [ข้อกำหนดเบื้องต้น](#ข้อกำหนดเบื้องต้น)
- [เริ่มต้นอย่างรวดเร็ว](#เริ่มต้นอย่างรวดเร็ว)
- [วิธีการทำงานของการตรวจสอบสิทธิ์](#วิธีการทำงานของการตรวจสอบสิทธิ์)
- [การรันแอปพลิเคชัน](#การรันแอปพลิเคชัน)
  - [การใช้ Maven](#การใช้-maven)
  - [การใช้ VS Code](#การใช้-vs-code)
  - [ผลลัพธ์ที่คาดหวัง](#ผลลัพธ์ที่คาดหวัง)
- [เอกสารอ้างอิงการตั้งค่า](#เอกสารอ้างอิงการตั้งค่า)
  - [ตัวแปรสภาพแวดล้อม](#ตัวแปรสภาพแวดล้อม)
  - [การตั้งค่า Spring](#การตั้งค่า-spring)
- [การแก้ไขปัญหา](#การแก้ไขปัญหา)
  - [ปัญหาที่พบบ่อย](#ปัญหาที่พบบ่อย)
  - [โหมดดีบัก](#โหมดดีบัก)
- [ขั้นตอนถัดไป](#ขั้นตอนถัดไป)
- [แหล่งข้อมูล](#แหล่งข้อมูล)

## ข้อกำหนดเบื้องต้น

ก่อนที่จะรันตัวอย่างนี้ ให้ตรวจสอบว่าคุณมี:

- ทรัพยากร Azure AI Foundry พร้อมการปรับใช้ `gpt-4o-mini` — สร้างโดยใช้ `azd up` หรือทำด้วยตนเองตาม [คำแนะนำการตั้งค่า Azure AI Foundry](../../getting-started-azure-openai.md)
- บทบาท **Cognitive Services OpenAI User** บนทรัพยากรนั้น (เทมเพลต Bicep จะกำหนดบทบาทนี้ให้คุณ)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli) ลงชื่อเข้าใช้ด้วย `az login`
- Java 21+ และ Maven 3.9+

> **ไม่ต้องใช้คีย์ API** — การตรวจสอบสิทธิ์ใช้แบบไม่ต้องใช้คีย์ผ่าน Microsoft Entra ID

## เริ่มต้นอย่างรวดเร็ว

```bash
# 1. ไปที่โปรเจกต์
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. ลงชื่อเข้าใช้เพื่อให้การยืนยันตัวตนแบบไม่มีคีย์สามารถรับโทเค็นได้
az login

# 3. กำหนดค่า endpoint
#    - หากคุณรัน `azd up` ไฟล์ .env จะถูกเขียนให้คุณ (ข้ามขั้นตอนนี้)
#    - หากไม่เช่นนั้นให้คัดลอกแม่แบบและตั้งค่า AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. รันแอปพลิเคชัน
mvn spring-boot:run
```

## วิธีการทำงานของการตรวจสอบสิทธิ์

ตัวอย่างนี้ตรวจสอบสิทธิ์ด้วย **Microsoft Entra ID** — ไม่มีคีย์ API

เมื่อกำหนดเพียง `spring.ai.azure.openai.endpoint` เท่านั้น (และไม่มี api-key) Spring AI จะสร้างไคลเอนต์ Azure OpenAI ด้วย [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential) ซึ่งข้อมูลประจำตัวนี้จะดึงโทเค็นจากเซสชัน `az login` ของคุณที่เครื่อง หรือจาก managed identity เมื่อรันใน Azure — ดังนั้นโค้ดเดียวกันนี้จึงทำงานได้ทั้งสองที่โดยไม่ต้องเปลี่ยนแปลง

## การรันแอปพลิเคชัน

### การใช้ Maven

```bash
mvn spring-boot:run
```

### การใช้ VS Code

1. เปิดโปรเจกต์ใน VS Code
2. กด `F5` หรือใช้แผง "Run and Debug"
3. เลือกการตั้งค่า "Spring Boot-BasicChatApplication"

> **หมายเหตุ**: การตั้งค่า VS Code จะโหลดไฟล์ .env ของคุณโดยอัตโนมัติ

### ผลลัพธ์ที่คาดหวัง

```
Starting Basic Chat with Azure OpenAI...
Environment variables loaded successfully
Connecting to Azure OpenAI...
Sending prompt: What is AI in a short sentence? Max 100 words.

AI Response:
================
AI, or Artificial Intelligence, is the simulation of human intelligence in machines programmed to think and learn like humans.
================

Success! Azure OpenAI connection is working correctly.
```

## เอกสารอ้างอิงการตั้งค่า

### ตัวแปรสภาพแวดล้อม

| ตัวแปร | คำอธิบาย | จำเป็น | ตัวอย่าง |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | URL จุดเชื่อมต่อ Foundry (Azure OpenAI) | ใช่ | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | ชื่อการปรับใช้โมเดลแชท | ไม่ | `gpt-4o-mini` (ค่าเริ่มต้น) |

> ไม่มีตัวแปรคีย์ API — การตรวจสอบสิทธิ์เป็นแบบไม่ใช้คีย์ (Microsoft Entra ID ผ่าน `az login`)

### การตั้งค่า Spring

ไฟล์ `application.yml` กำหนดค่า:
- **Endpoint**: `${AZURE_OPENAI_ENDPOINT}` - จากตัวแปรสภาพแวดล้อม
- **Deployment**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - จากตัวแปรสภาพแวดล้อมพร้อมค่าตั้งต้น
- **Auth**: แบบไม่ใช้คีย์ — ไม่มีการตั้งค่า `api-key` ดังนั้น Spring AI จึงใช้ `DefaultAzureCredential`
- **Temperature**: `0.7` - ควบคุมความสร้างสรรค์ (0.0 = แน่นอน, 1.0 = สร้างสรรค์)
- **Max Tokens**: `500` - ความยาวตอบกลับสูงสุด

## การแก้ไขปัญหา

### ปัญหาที่พบบ่อย

<details>
<summary><strong>ข้อผิดพลาด: 401 / "PermissionDenied" / ข้อผิดพลาดโทเค็น</strong></summary>

- รัน `az login` — การตรวจสอบสิทธิ์แบบไม่ใช้คีย์ต้องการการลงชื่อเข้าใช้ที่ใช้งานอยู่เพื่อรับโทเค็น
- ตรวจสอบว่าบัญชีของคุณมีบทบาท **Cognitive Services OpenAI User** บนทรัพยากร
- หากเพิ่งกำหนดบทบาท รอประมาณหนึ่งนาทีเพื่อให้มีผล
- ยืนยันว่าคุณอยู่ใน tenant/subscription ที่ถูกต้อง (`az account show`)
</details>

<details>
<summary><strong>ข้อผิดพลาด: "The endpoint is not valid" / ข้อผิดพลาดการเชื่อมต่อ</strong></summary>

- ตรวจสอบว่า `AZURE_OPENAI_ENDPOINT` เป็น URL พื้นฐานเต็มรูปแบบ (เช่น `https://your-resource.openai.azure.com/`)
- ตรวจสอบความสม่ำเสมอของเครื่องหมายทับหลัง
- ยืนยันว่า endpoint ตรงกับทรัพยากรที่คุณตั้งค่าไว้ (`azd env get-values`)
</details>

<details>
<summary><strong>ข้อผิดพลาด: "The deployment was not found"</strong></summary>

- ตรวจสอบว่า `AZURE_OPENAI_DEPLOYMENT` ตรงกับชื่อการปรับใช้ใน Azure
- ตรวจสอบว่าโมเดลถูกปรับใช้และใช้งานได้สำเร็จ
- ชื่อการปรับใช้เริ่มต้นคือ `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: ตัวแปรสภาพแวดล้อมไม่โหลด</strong></summary>

- ตรวจสอบว่าไฟล์ `.env` อยู่ที่โฟลเดอร์รากของโปรเจกต์ (ระดับเดียวกับ `pom.xml`)
- ลองรัน `mvn spring-boot:run` ในเทอร์มินัลแบบบูรณาการของ VS Code
- ตรวจสอบว่าได้ติดตั้งส่วนขยาย Java ของ VS Code อย่างถูกต้อง
</details>

### โหมดดีบัก

เพื่อเปิดใช้งานการบันทึกข้อมูลโดยละเอียด ให้ยกเลิกการคอมเมนต์บรรทัดเหล่านี้ใน `application.yml`:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## ขั้นตอนถัดไป

**การตั้งค่าเสร็จสมบูรณ์!** ดำเนินการเรียนรู้ของคุณต่อไป:

[บทที่ 3: เทคนิค Generative AI หลัก](../../../03-CoreGenerativeAITechniques/README.md)

## แหล่งข้อมูล

- [เอกสาร Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [การตรวจสอบสิทธิ์แบบไม่ใช้คีย์ด้วย Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [พอร์ทัล Azure AI Foundry](https://ai.azure.com/)
- [เอกสาร Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ปฏิเสธความรับผิดชอบ**:
เอกสารนี้ได้รับการแปลโดยใช้บริการแปลภาษา AI [Co-op Translator](https://github.com/Azure/co-op-translator) ขณะที่เราพยายามให้ความถูกต้อง โปรดทราบว่าการแปลโดยอัตโนมัติอาจมีข้อผิดพลาดหรือความไม่ถูกต้อง เอกสารต้นฉบับในภาษาต้นทางควรถูกพิจารณาเป็นแหล่งข้อมูลที่เชื่อถือได้ สำหรับข้อมูลที่สำคัญ แนะนำให้ใช้การแปลโดยมนุษย์มืออาชีพ เราไม่รับผิดชอบต่อความเข้าใจผิดหรือการตีความที่ผิดพลาดที่เกิดขึ้นจากการใช้การแปลนี้
<!-- CO-OP TRANSLATOR DISCLAIMER END -->