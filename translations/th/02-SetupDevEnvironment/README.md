# การตั้งค่าสภาพแวดล้อมการพัฒนาสำหรับ Generative AI สำหรับ Java

> **เริ่มต้นอย่างรวดเร็ว:** จัดเตรียมโมเดล AI ของคุณบน **Azure AI Foundry** เป็นโค้ดด้วย Bicep + `azd` ในไม่กี่นาที — ดูได้ที่ [Azure AI Foundry Setup Guide](getting-started-azure-openai.md) การรับรองความถูกต้องเป็นแบบ **ไม่ต้องใช้คีย์** (Microsoft Entra ID) ดังนั้นจึงไม่มีคีย์ API ให้จัดการ

## สิ่งที่คุณจะได้เรียนรู้

- ตั้งค่าสภาพแวดล้อมการพัฒนา Java สำหรับแอปพลิเคชัน AI
- เลือกและกำหนดค่าสภาพแวดล้อมการพัฒนาที่คุณชอบ (เน้นคลาวด์ด้วย Codespaces, คอนเทนเนอร์พัฒนาท้องถิ่น หรือการตั้งค่าท้องถิ่นเต็มรูปแบบ)
- ทดสอบการตั้งค่าของคุณโดยการเชื่อมต่อกับโมเดล Azure AI Foundry

## สารบัญ

- [สิ่งที่คุณจะได้เรียนรู้](#สิ่งที่คุณจะได้เรียนรู้)
- [บทนำ](#บทนำ)
- [ขั้นตอนที่ 1: ตั้งค่าสภาพแวดล้อมการพัฒนา](#ขั้นตอนที่-1-ตั้งค่าสภาพแวดล้อมการพัฒนา)
  - [ตัวเลือก A: GitHub Codespaces (แนะนำ)](#ตัวเลือก-a-github-codespaces-แนะนำ)
  - [ตัวเลือก B: คอนเทนเนอร์พัฒนาท้องถิ่น](#ตัวเลือก-b-คอนเทนเนอร์พัฒนาท้องถิ่น)
  - [ตัวเลือก C: ใช้การติดตั้งในเครื่องที่มีอยู่](#ตัวเลือก-c-ใช้การติดตั้งในเครื่องที่มีอยู่)
- [ขั้นตอนที่ 2: จัดเตรียม Azure AI Foundry](#ขั้นตอนที่-2-จัดเตรียม-azure-ai-foundry)
- [ขั้นตอนที่ 3: ทดสอบการตั้งค่าของคุณ](#ขั้นตอนที่-3-ทดสอบการตั้งค่าของคุณ)
- [การแก้ไขปัญหา](#การแก้ไขปัญหา)
- [สรุป](#สรุป)
- [ขั้นตอนถัดไป](#ขั้นตอนถัดไป)

## บทนำ

บทนี้จะพาคุณผ่านขั้นตอนการตั้งค่าสภาพแวดล้อมการพัฒนา เราจะใช้ **Azure AI Foundry** สำหรับโมเดลตลอดหลักสูตรนี้ คุณจะจัดเตรียมโมเดลเป็นโค้ดด้วย Bicep และ Azure Developer CLI (`azd`) จากนั้นเชื่อมต่อด้วยการรับรองความถูกต้องแบบ **ไม่ต้องใช้คีย์** (Microsoft Entra ID) — ไม่มีคีย์ API ให้คัดลอกหรือรั่วไหล

**ไม่ต้องตั้งค่าในเครื่อง!** คุณสามารถใช้ GitHub Codespaces ซึ่งให้สภาพแวดล้อมการพัฒนาเต็มรูปแบบในเบราว์เซอร์ของคุณ และจัดเตรียม Foundry จากที่นั่น

เราใช้ **Azure AI Foundry** สำหรับหลักสูตรนี้เพราะ:
- **จัดเตรียมเป็นโค้ด** — ใช้ `azd up` เพียงคำสั่งเดียวก็ปรับใช้บัญชีและการปรับใช้โมเดลได้
- **ไม่ต้องใช้คีย์** — รับรองความถูกต้องด้วยการลงชื่อเข้าใช้ Azure หรือ managed identity
- **พร้อมสำหรับงานจริง** — โค้ดเดียวกันทำงานได้ทั้งในเครื่องและบน Azure
- **ยืดหยุ่น** — เปลี่ยนโมเดลได้โดยแก้ชื่อการปรับใช้ ไม่ต้องแก้โค้ดของคุณ

> **หมายเหตุ**: การปรับใช้ Azure AI Foundry จะคิดค่าบริการตามจำนวนโทเค็น (จ่ายตามการใช้งาน) ดูได้ที่ [Azure AI Foundry setup guide](getting-started-azure-openai.md) สำหรับรายละเอียดการจัดเตรียม ภูมิภาค และค่าใช้จ่าย


## ขั้นตอนที่ 1: ตั้งค่าสภาพแวดล้อมการพัฒนา

<a name="quick-start-cloud"></a>

เราได้สร้างคอนเทนเนอร์พัฒนาที่กำหนดล่วงหน้าเพื่อช่วยลดเวลาการตั้งค่าและให้แน่ใจว่าคุณมีเครื่องมือที่จำเป็นทั้งหมดสำหรับหลักสูตร Generative AI for Java นี้ เลือกวิธีการพัฒนาที่คุณชอบ:

### ตัวเลือกในการตั้งค่าสภาพแวดล้อม:

#### ตัวเลือก A: GitHub Codespaces (แนะนำ)

**เริ่มเขียนโค้ดได้ใน 2 นาที - ไม่ต้องตั้งค่าในเครื่อง!**

1. Fork รีโพซิทอรีนี้ไปยังบัญชี GitHub ของคุณ
   > **หมายเหตุ**: หากคุณต้องการแก้ไขการตั้งค่าพื้นฐาน โปรดดูที่ [Dev Container Configuration](../../../.devcontainer/devcontainer.json)
2. คลิก **Code** → แท็บ **Codespaces** → **...** → **New with options...**
3. ใช้ค่าดีฟอลต์ – เลือก **Dev container configuration**: **Generative AI Java Development Environment** ซึ่งเป็น devcontainer แบบกำหนดเองสำหรับหลักสูตรนี้
4. คลิก **Create codespace**
5. รอประมาณ 2 นาทีจนกว่าสภาพแวดล้อมจะพร้อมใช้งาน
6. ดำเนินการต่อไปที่ [ขั้นตอนที่ 2: จัดเตรียม Azure AI Foundry](#ขั้นตอนที่-2-จัดเตรียม-azure-ai-foundry)

<img src="../../../translated_images/th/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: Codespaces submenu" width="50%">

<img src="../../../translated_images/th/image.833552b62eee7766.webp" alt="Screenshot: New with options" width="50%">

<img src="../../../translated_images/th/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: Create codespace options" width="50%">


> **ข้อดีของ Codespaces**:
> - ไม่ต้องติดตั้งในเครื่อง
> - ใช้งานได้บนอุปกรณ์ใดก็ได้ที่ใช้เบราว์เซอร์
> - ตั้งค่าล่วงหน้าพร้อมด้วยเครื่องมือและ dependencies ทั้งหมด
> - ฟรี 60 ชั่วโมงต่อเดือนสำหรับบัญชีส่วนตัว
> - สภาพแวดล้อมที่สอดคล้องกันสำหรับผู้เรียนทุกคน

#### ตัวเลือก B: คอนเทนเนอร์พัฒนาท้องถิ่น

**สำหรับนักพัฒนาที่ชอบพัฒนาท้องถิ่นด้วย Docker**

1. Fork และโคลนรีโพซิทอรีนี้ไปยังเครื่องของคุณ
   > **หมายเหตุ**: หากคุณต้องการแก้ไขการตั้งค่าพื้นฐาน โปรดดูที่ [Dev Container Configuration](../../../.devcontainer/devcontainer.json)
2. ติดตั้ง [Docker Desktop](https://www.docker.com/products/docker-desktop/) และ [VS Code](https://code.visualstudio.com/)
3. ติดตั้ง [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) ใน VS Code
4. เปิดโฟลเดอร์รีโพซิทอรีใน VS Code
5. เมื่อมีข้อความแจ้ง ให้คลิก **Reopen in Container** (หรือใช้ `Ctrl+Shift+P` → "Dev Containers: Reopen in Container")
6. รอให้คอนเทนเนอร์สร้างและเริ่มทำงาน
7. ดำเนินการต่อไปที่ [ขั้นตอนที่ 2: จัดเตรียม Azure AI Foundry](#ขั้นตอนที่-2-จัดเตรียม-azure-ai-foundry)

<img src="../../../translated_images/th/devcontainer.21126c9d6de64494.webp" alt="Screenshot: Dev container setup" width="50%">

<img src="../../../translated_images/th/image-3.bf93d533bbc84268.webp" alt="Screenshot: Dev container build complete" width="50%">

#### ตัวเลือก C: ใช้การติดตั้งในเครื่องที่มีอยู่

**สำหรับนักพัฒนาที่มีสภาพแวดล้อม Java อยู่แล้ว**

ข้อกำหนดล่วงหน้า:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html) 
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) หรือ IDE ที่คุณชอบ

ขั้นตอน:
1. โคลนรีโพซิทอรีนี้ไปยังเครื่องของคุณ
2. เปิดโปรเจกต์ใน IDE ของคุณ
3. ดำเนินการต่อไปที่ [ขั้นตอนที่ 2: จัดเตรียม Azure AI Foundry](#ขั้นตอนที่-2-จัดเตรียม-azure-ai-foundry)

> **เคล็ดลับมือโปร**: ถ้าคุณมีเครื่องที่สเปคต่ำแต่ต้องการใช้ VS Code ท้องถิ่น ใช้ GitHub Codespaces! คุณสามารถเชื่อมต่อ VS Code ท้องถิ่นของคุณกับ Codespace บนคลาวด์ เพื่อรับประโยชน์จากทั้งสองโลก

<img src="../../../translated_images/th/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: created local devcontainer instance" width="50%">


## ขั้นตอนที่ 2: จัดเตรียม Azure AI Foundry

ปรับใช้โมเดล AI ของหลักสูตรไปยัง Azure AI Foundry เป็นโค้ด จาก root ของรีโพซิทอรี:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```

`azd` จะถามชื่อสภาพแวดล้อมและภูมิภาค จากนั้นจัดเตรียมบัญชี Azure AI Foundry พร้อมการปรับใช้โมเดล `gpt-4o-mini` และ `text-embedding-3-small` และเขียน endpoint ลงในไฟล์ `.env` ของตัวอย่าง — ทั้งหมดนี้ด้วยการรับรองความถูกต้องแบบ **ไม่ต้องใช้คีย์** (ไม่มีคีย์ API)

> **คำแนะนำแบบเต็ม:** ดูที่ [Azure AI Foundry Setup Guide](getting-started-azure-openai.md) สำหรับข้อกำหนดล่วงหน้า วิธีการแบบแมนนวล (portal) คำแนะนำภูมิภาค และบันทึกค่าใช้จ่าย/การล้างข้อมูล

## ขั้นตอนที่ 3: ทดสอบการตั้งค่าของคุณ

เมื่อโมเดล Foundry ของคุณถูกจัดเตรียมแล้ว ให้ทดสอบการเชื่อมต่อด้วยแอปตัวอย่างใน [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure)

1. เปิดเทอร์มินัลในสภาพแวดล้อมการพัฒนาของคุณ
2. ไปยังตัวอย่าง:
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
3. ตรวจสอบว่าคุณได้ลงชื่อเข้าใช้แล้ว (การรับรองความถูกต้องแบบไม่ต้องใช้คีย์ต้องใช้โทเค็น):
   ```bash
   az login
   ```
   > หากคุณรัน `azd up` ไฟล์ `.env` ที่มี endpoint ของคุณก็ถูกเขียนไว้แล้ว
4. รันแอปพลิเคชัน:
   ```bash
   mvn clean spring-boot:run
   ```

คุณควรเห็นการตอบสนองจากโมเดล `gpt-4o-mini`

### ทำความเข้าใจโค้ดตัวอย่าง

ตัวอย่างใน `examples/basic-chat-azure` คือแอป Spring Boot ที่ใช้ **Spring AI** เพื่อเชื่อมต่อกับ Azure AI Foundry ด้วยการรับรองความถูกต้องแบบไม่ใช้คีย์

**โค้ดนี้ทำอะไร:**
- **เชื่อมต่อ** กับ Azure AI Foundry ผ่านการลงชื่อเข้าใช้ Azure ของคุณ (Microsoft Entra ID) — ไม่ต้องใช้คีย์ API
- **ส่ง** prompt ไปยังโมเดล `gpt-4o-mini`
- **รับ** และแสดงผลการตอบกลับของ AI
- **ตรวจสอบ** ว่าการตั้งค่าของคุณทำงานถูกต้อง

**ไลบรารีสำคัญ** (ใน `pom.xml`):
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```

**การกำหนดค่า** (`application.yml`):
```yaml
spring:
  ai:
    azure:
      openai:
        # Endpoint only - no api-key. Spring AI uses DefaultAzureCredential (keyless).
        endpoint: ${AZURE_OPENAI_ENDPOINT}
        chat:
          options:
            deployment-name: ${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}
```

## สรุป

เยี่ยม! ตอนนี้คุณได้ตั้งค่าทุกอย่างเรียบร้อยแล้ว:

- จัดเตรียมโมเดล Azure AI Foundry เป็นโค้ดด้วย Bicep + `azd`
- ได้รันสภาพแวดล้อมการพัฒนา Java ของคุณ (ไม่ว่าจะเป็น Codespaces, dev containers หรือท้องถิ่น)
- เชื่อมต่อกับ Azure AI Foundry ด้วยการรับรองความถูกต้องแบบไม่ใช้คีย์ (Microsoft Entra ID) — ไม่มีคีย์ API
- ทดสอบทุกอย่างทำงานด้วยตัวอย่างง่าย ๆ ที่สื่อสารกับโมเดลของคุณ

## ขั้นตอนถัดไป

[บทที่ 3: เทคนิค Generative AI หลัก](../03-CoreGenerativeAITechniques/README.md)

## การแก้ไขปัญหา

มีปัญหาใช่ไหม? นี่คือปัญหาทั่วไปและวิธีแก้ไข:

- **รับรองความถูกต้องล้มเหลว (401/403)?**
  - รัน `az login` — การรับรองความถูกต้องเป็นแบบไม่ใช้คีย์ คุณต้องลงชื่อเข้าใช้
  - ตรวจสอบว่าบัญชีของคุณมีบทบาท **Cognitive Services OpenAI User** ในทรัพยากรนั้น
  - หากเพิ่งจัดเตรียม ให้รอสักครู่ให้ระบบมอบหมายบทบาทเสร็จสมบูรณ์

- **ไม่พบ Maven?**
  - หากใช้ dev containers/Codespaces จะติดตั้ง Maven มาให้แล้ว
  - สำหรับการตั้งค่าในเครื่อง ให้แน่ใจว่าได้ติดตั้ง Java 21+ และ Maven 3.9+
  - ลองรัน `mvn --version` เพื่อตรวจสอบการติดตั้ง

- **ไม่พบ `azd` หรือการจัดเตรียมล้มเหลว?**
  - ติดตั้ง [Azure Developer CLI](https://aka.ms/azure-dev/install) และรัน `azd auth login`
  - เลือกภูมิภาคที่ `gpt-4o-mini` มีให้บริการ (เช่น `eastus2`)
  - ดู [Azure AI Foundry setup guide](getting-started-azure-openai.md) สำหรับรายละเอียด

- **คอนเทนเนอร์พัฒนาไม่เริ่มทำงาน?**
  - ตรวจสอบว่า Docker Desktop กำลังทำงานอยู่ (สำหรับพัฒนาท้องถิ่น)
  - ลองสร้างคอนเทนเนอร์ใหม่: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **ข้อผิดพลาดในการคอมไพล์แอปพลิเคชัน?**
  - ตรวจสอบว่าคุณอยู่ในไดเรกทอรีที่ถูกต้อง: `02-SetupDevEnvironment/examples/basic-chat-azure`
  - ลองทำความสะอาดและคอมไพล์ใหม่: `mvn clean compile`

> **ต้องการความช่วยเหลือ?**: หากยังมีปัญหาอยู่ เปิด issue ในรีโพซิทอรีและเราจะช่วยคุณเอง

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ปฏิเสธความรับผิดชอบ**:
เอกสารนี้ได้รับการแปลโดยใช้บริการแปลภาษา AI [Co-op Translator](https://github.com/Azure/co-op-translator) ขณะที่เราพยายามให้ความถูกต้อง โปรดทราบว่าการแปลโดยอัตโนมัติอาจมีข้อผิดพลาดหรือความไม่ถูกต้อง เอกสารต้นฉบับในภาษาต้นทางควรถูกพิจารณาเป็นแหล่งข้อมูลที่เชื่อถือได้ สำหรับข้อมูลที่สำคัญ แนะนำให้ใช้การแปลโดยมนุษย์มืออาชีพ เราไม่รับผิดชอบต่อความเข้าใจผิดหรือการตีความที่ผิดพลาดที่เกิดขึ้นจากการใช้การแปลนี้
<!-- CO-OP TRANSLATOR DISCLAIMER END -->