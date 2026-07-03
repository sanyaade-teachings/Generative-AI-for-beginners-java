# การตั้งค่าสภาพแวดล้อมการพัฒนาสำหรับ Azure AI Foundry

> ไกด์นี้ตั้งค่าโมเดล **Azure AI Foundry** สำหรับแอป AI ภาษา Java ในหลักสูตรนี้ โดยใช้การตรวจสอบสิทธิ์แบบ **keyless** (Microsoft Entra ID) — ไม่มีคีย์ API ให้จัดการ หากคุณยังใหม่กับเครื่องมือนี้? เริ่มต้นได้ที่ [ไกด์สภาพแวดล้อมการพัฒนา](./README.md)

ไกด์นี้ตั้งค่าโมเดล **Azure AI Foundry** สำหรับแอป AI ภาษา Java ในหลักสูตรนี้ คุณมีทางเลือกสองแบบ:

- **ตัวเลือก A — Provision ด้วย `azd` + Bicep (แนะนำ):** คำสั่งเดียวจะปรับใช้บัญชี Foundry และโมเดลเป็นโค้ด ไม่มีการคลิกในพอร์ทัล
- **ตัวเลือก B — สร้างทรัพยากรด้วยตนเอง** ในพอร์ทัล Azure AI Foundry

ทั้งสองทางเลือกใช้การตรวจสอบสิทธิ์แบบ **keyless** (Microsoft Entra ID) — ไม่มีคีย์ API ให้คัดลอกหรือรั่วไหล

## สารบัญ

- [สิ่งที่จะถูกสร้างขึ้น](#สิ่งที่จะถูกสร้างขึ้น)
- [ข้อกำหนดเบื้องต้น](#ข้อกำหนดเบื้องต้น)
- [ตัวเลือก A: Provision ด้วย azd + Bicep (แนะนำ)](#option-a-provision-with-azd--bicep-recommended)
- [ตัวเลือก B: สร้างทรัพยากรด้วยตนเอง](#ตัวเลือก-b-สร้างทรัพยากรด้วยตนเอง)
- [ตั้งค่าสภาพแวดล้อมของคุณ](#ตั้งค่าสภาพแวดล้อมของคุณ)
- [ทดสอบการตั้งค่าของคุณ](#ทดสอบการตั้งค่าของคุณ)
- [ขั้นตอนต่อไป?](#ขั้นตอนต่อไป)
- [แหล่งข้อมูล](#แหล่งข้อมูล)
- [แหล่งข้อมูลเพิ่มเติม](#แหล่งข้อมูลเพิ่มเติม)

## สิ่งที่จะถูกสร้างขึ้น

แม่แบบ Bicep ใน [`infra/`](../../../02-SetupDevEnvironment/infra) จะ provision:

- บัญชี **Azure AI Foundry** (`Microsoft.CognitiveServices/accounts`, ชนิด `AIServices`) พร้อมโปรเจกต์
- การปรับใช้ **chat** — `gpt-4o-mini`
- การปรับใช้ **embedding** — `text-embedding-3-small` (ใช้ในบทต่อไป)
- การมอบหมายบทบาทแบบ **keyless** (`Cognitive Services OpenAI User`) เพื่อให้คุณลงชื่อเข้าใช้ด้วย `az login` แทนการจัดการคีย์

## ข้อกำหนดเบื้องต้น

- [การสมัครใช้งาน Azure](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) และ [Maven 3.9+](https://maven.apache.org/download.cgi)

## ตัวเลือก A: Provision ด้วย azd + Bicep (แนะนำ)

จากโฟลเดอร์ `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# ลงชื่อเข้าใช้ (ทั้งสองเครื่องมือ)
azd auth login
az login

# จัดเตรียมบัญชี Foundry + การปรับใช้โมเดล
azd up
```

`azd` จะร้องขอ **ชื่อสภาพแวดล้อม** (ตัวอย่างเช่น `genai-java`) และ **ภูมิภาค** เลือกภูมิภาคที่มี `gpt-4o-mini` และ `text-embedding-3-small` — เช่น `eastus2` หรือ `swedencentral`

เมื่อ provisioning เสร็จสิ้น azd จะ:

1. ปรับใช้ทุกอย่างที่กำหนดใน [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep)
2. รัน postprovision hook ที่เขียน [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) ด้วยชื่อจุดสิ้นสุดและชื่อการปรับใช้ของคุณ (ไม่มีความลับ)

> **เคล็ดลับ:** รันคำสั่ง `azd up` อีกครั้งเมื่อใดก็ได้เพื่อใช้การเปลี่ยนแปลง รัน `azd down` เพื่อลบทุกอย่างและหยุดเกิดค่าใช้จ่าย

เพื่อดูการตั้งค่าที่สร้างขึ้น:

```bash
azd env get-values
```

ตอนนี้ข้ามไปที่ [ทดสอบการตั้งค่าของคุณ](#ทดสอบการตั้งค่าของคุณ)

## ตัวเลือก B: สร้างทรัพยากรด้วยตนเอง

ชอบใช้พอร์ทัล? สร้างทรัพยากรด้วยตัวเอง:

1. ไปที่ [พอร์ทัล Azure AI Foundry](https://ai.azure.com/) และลงชื่อเข้าใช้
2. **สร้างโปรเจกต์** (สิ่งนี้จะสร้างทรัพยากร AI Foundry ด้วย) ตั้งชื่อเช่น `GenAIJava`
3. ในโปรเจกต์ของคุณ เปิด **Models + endpoints** → **Deploy model** → **Deploy base model**
4. ปรับใช้ **gpt-4o-mini** (ชื่อการปรับใช้ `gpt-4o-mini`) ทำซ้ำสำหรับ **text-embedding-3-small** หากคุณต้องการตัวอย่าง embedding
5. จาก **ภาพรวม** คัดลอก **endpoint** (เช่น `https://<resource>.openai.azure.com/`)
6. มอบสิทธิ์ keyless ให้ตัวเอง: ในทรัพยากร เปิด **การควบคุมการเข้าถึง (IAM)** → **เพิ่มบทบาท** → มอบหมาย **Cognitive Services OpenAI User** ให้กับบัญชีของคุณ

> **ยังมีปัญหา?** ดูที่ [เอกสาร Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects)

## ตั้งค่าสภาพแวดล้อมของคุณ

**หากคุณใช้ตัวเลือก A (`azd up`)** ไฟล์ตั้งค่าของคุณจะถูกเขียนไว้แล้ว — ไม่มีอะไรต้องตั้งค่า ข้ามไปที่ [ทดสอบการตั้งค่าของคุณ](#ทดสอบการตั้งค่าของคุณ)

**หากคุณใช้ตัวเลือก B (ด้วยตนเอง)** ให้สร้างไฟล์ `.env` ของตัวอย่างด้วยตัวเอง:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

แก้ไข `.env` ด้วย endpoint ของคุณ (ไม่มีคีย์ — ใช้การตรวจสอบสิทธิ์แบบ keyless):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **บันทึกความปลอดภัย:** ไม่มีคีย์ API ที่ต้องเก็บ คุณตรวจสอบสิทธิ์ด้วย Microsoft Entra ID ผ่าน `az login` (ในเครื่อง) หรือ managed identity (ใน Azure) ไฟล์ `.env` มีแต่การตั้งค่าไม่ลับและถูกละไว้ใน `.gitignore` แล้ว

## ทดสอบการตั้งค่าของคุณ

ตรวจสอบให้แน่ใจว่าคุณได้ลงชื่อเข้าใช้เพื่อให้การตรวจสอบสิทธิ์แบบ keyless ได้รับโทเค็น จากนั้นรันตัวอย่าง:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # ถ้าคุณยังไม่ได้เข้าสู่ระบบ
mvn clean spring-boot:run
```

คุณจะเห็นการตอบกลับจากโมเดล `gpt-4o-mini`!

> **ผู้ใช้ VS Code:** กด `F5` เพื่อรัน แอปจะโหลด `.env` ของคุณโดยอัตโนมัติ

> **ตัวอย่างเต็ม:** ดู [Basic Chat with Azure AI Foundry example](./examples/basic-chat-azure/README.md) สำหรับรายละเอียดและการแก้ไขปัญหา

## ขั้นตอนต่อไป?

**ตั้งค่าเสร็จสมบูรณ์!** ตอนนี้คุณมี:
- Azure AI Foundry พร้อม `gpt-4o-mini` และ `text-embedding-3-small` ถูกปรับใช้
- การตรวจสอบสิทธิ์แบบ keyless (Microsoft Entra ID) — ไม่มีคีย์ให้จัดการ
- ไฟล์ `.env` ในเครื่องพร้อม endpoint และชื่อการปรับใช้ของคุณ
- สภาพแวดล้อมการพัฒนา Java พร้อมใช้งาน

**ดำเนินการต่อไปที่** [บทที่ 3: เทคนิค AI สร้างสรรค์หลัก](../03-CoreGenerativeAITechniques/README.md) เพื่อเริ่มสร้างแอป AI!

## แหล่งข้อมูล

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [การตรวจสอบสิทธิ์แบบ keyless ด้วย Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [เอกสาร Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)
- [เอกสาร Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## แหล่งข้อมูลเพิ่มเติม

- [ดาวน์โหลด VS Code](https://code.visualstudio.com/Download)
- [รับ Docker Desktop](https://www.docker.com/products/docker-desktop)
- [การกำหนดค่านี้ Dev Container](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ปฏิเสธความรับผิดชอบ**:
เอกสารนี้ได้รับการแปลโดยใช้บริการแปลภาษา AI [Co-op Translator](https://github.com/Azure/co-op-translator) ขณะที่เราพยายามให้ความถูกต้อง โปรดทราบว่าการแปลโดยอัตโนมัติอาจมีข้อผิดพลาดหรือความไม่ถูกต้อง เอกสารต้นฉบับในภาษาต้นทางควรถูกพิจารณาเป็นแหล่งข้อมูลที่เชื่อถือได้ สำหรับข้อมูลที่สำคัญ แนะนำให้ใช้การแปลโดยมนุษย์มืออาชีพ เราไม่รับผิดชอบต่อความเข้าใจผิดหรือการตีความที่ผิดพลาดที่เกิดขึ้นจากการใช้การแปลนี้
<!-- CO-OP TRANSLATOR DISCLAIMER END -->