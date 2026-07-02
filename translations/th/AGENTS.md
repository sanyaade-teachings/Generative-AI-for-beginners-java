# AGENTS.md

## ภาพรวมโครงการ

นี่คือที่เก็บโค้ดเพื่อการศึกษาเกี่ยวกับการพัฒนา Generative AI ด้วย Java โดยมีหลักสูตรแบบลงมือทำที่ครอบคลุม Large Language Models (LLMs), prompt engineering, embeddings, RAG (Retrieval-Augmented Generation) และ Model Context Protocol (MCP)

**เทคโนโลยีหลัก:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI และ OpenAI SDKs

**สถาปัตยกรรม:**
- แอปพลิเคชัน Spring Boot หลายตัวแยกส่วนตามบทเรียน
- โครงการตัวอย่างที่แสดงแพตเทิร์น AI หลากหลาย
- พร้อมใช้งานกับ GitHub Codespaces ที่ตั้งค่าคอนเทนเนอร์สำหรับการพัฒนาไว้ล่วงหน้า

## คำสั่งติดตั้ง

### สิ่งที่ต้องเตรียม
- Java 21 หรือสูงกว่า
- Maven 3.x
- บัญชีสมาชิก Azure พร้อมการใช้งาน Azure AI Foundry และการติดตั้งโมเดล (ตั้งค่าด้วย `azd up`)
- Azure CLI (`az`) และ Azure Developer CLI (`azd`) พร้อมเซ็นชื่อเข้าระบบสำหรับ keyless auth

### การตั้งค่าสภาพแวดล้อม

**ตัวเลือก 1: GitHub Codespaces (แนะนำ)**
```bash
# Fork โครงการและสร้าง codespace จาก GitHub UI
# dev container จะติดตั้ง dependencies ทั้งหมดโดยอัตโนมัติ
# รอประมาณ ~2 นาทีสำหรับการตั้งค่าสภาพแวดล้อม
```

**ตัวเลือก 2: Dev Container ในเครื่อง**
```bash
# โคลนรีโพสิทอรี
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# เปิดใน VS Code พร้อมส่วนขยาย Dev Containers
# เปิดใหม่ในคอนเทนเนอร์เมื่อมีการแจ้งเตือน
```

**ตัวเลือก 3: การตั้งค่าในเครื่อง**
```bash
# ติดตั้ง dependencies
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# ตรวจสอบการติดตั้ง
java -version
mvn -version
```

### การกำหนดค่า

**การตั้งค่า Azure AI Foundry (keyless, แนะนำ):**
```bash
# จัดเตรียมบัญชี Foundry + การปรับใช้โมเดลเป็นโค้ด
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd เขียน examples/basic-chat-azure/.env ด้วยจุดสิ้นสุดของคุณ (ไม่มีคีย์)
```

**การตั้งค่า endpoint ด้วยตนเอง:**
```bash
# หากคุณไม่ได้ใช้ azd ให้ตั้งค่า endpoint ด้วยตัวเอง (การตรวจสอบสิทธิ์จะยังคงไม่ใช้คีย์ผ่าน az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# แก้ไข .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## เวิร์กโฟลว์การพัฒนา

### โครงสร้างโครงการ
```
/
├── 01-IntroToGenAI/              # Chapter 1: Introduction
├── 02-SetupDevEnvironment/       # Chapter 2: Environment setup
│   └── examples/                 # Working examples
├── 03-CoreGenerativeAITechniques/ # Chapter 3: Core techniques
├── 04-PracticalSamples/          # Chapter 4: Sample projects
│   ├── calculator/               # MCP service example
│   ├── foundrylocal/            # Local model integration
│   └── petstory/                # Multi-modal app
├── 05-ResponsibleGenAI/         # Chapter 5: Responsible AI
└── translations/                # Multi-language support
```

### การรันแอปพลิเคชัน

**รันแอป Spring Boot:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**สร้างโครงการ:**
```bash
cd [project-directory]
mvn clean install
```

**เริ่ม MCP Calculator Server:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# เซิร์ฟเวอร์ทำงานบน http://localhost:8080
```

**รันตัวอย่าง Client:**
```bash
# หลังจากเริ่มเซิร์ฟเวอร์ในเทอร์มินัลอื่น
cd 04-PracticalSamples/calculator

# ลูกค้า MCP ตรง
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# ลูกค้าที่ขับเคลื่อนด้วย AI (ต้องใช้ AZURE_OPENAI_ENDPOINT + az login)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# หุ่นยนต์ตอบโต้แบบอินเทอร์แอคทีฟ
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### การโหลดซ้ำแบบ Hot Reload
Spring Boot DevTools รวมอยู่ในโปรเจ็กต์ที่รองรับ hot reload:
```bash
# การเปลี่ยนแปลงไฟล์ Java จะโหลดใหม่โดยอัตโนมัติเมื่อบันทึกแล้ว
mvn spring-boot:run
```

## คำแนะนำการทดสอบ

### การรันการทดสอบ

**รันการทดสอบทั้งหมดในโครงการ:**
```bash
cd [project-directory]
mvn test
```

**รันการทดสอบพร้อมแสดงผลแบบละเอียด:**
```bash
mvn test -X
```

**รันคลาสทดสอบเฉพาะ:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### โครงสร้างการทดสอบ
- ไฟล์ทดสอบใช้ JUnit 5 (Jupiter)
- คลาสทดสอบอยู่ใน `src/test/java/`
- ตัวอย่าง client ในโครงการ calculator อยู่ใน `src/test/java/com/microsoft/mcp/sample/client/`

### การทดสอบด้วยตนเอง
หลายตัวอย่างเป็นแอปแบบโต้ตอบที่ต้องทดสอบด้วยตนเอง:

1. เริ่มแอปด้วย `mvn spring-boot:run`
2. ทดสอบ endpoints หรือโต้ตอบกับ CLI
3. ตรวจสอบให้แน่ใจว่าพฤติกรรมที่คาดหวังตรงกับเอกสาร README.md ของแต่ละโครงการ

### การทดสอบกับ Azure AI Foundry
- ลงชื่อเข้าใช้ด้วย `az login` ก่อนรันตัวอย่าง (keyless auth)
- ตรวจสอบว่าบัญชีของคุณมีบทบาท Cognitive Services OpenAI User บนทรัพยากรนั้น
- ทดสอบการกรองเนื้อหาด้วยตัวอย่าง responsible AI ในบทที่ 5

## แนวทางการเขียนโค้ด

### ข้อตกลง Java
- **เวอร์ชัน Java:** Java 21 พร้อมคุณสมบัติสมัยใหม่
- **สไตล์:** ปฏิบัติตามมาตรฐานข้อตกลง Java
- **การตั้งชื่อ:** 
  - คลาส: PascalCase
  - เมธอด/ตัวแปร: camelCase
  - ค่าคงที่: UPPER_SNAKE_CASE
  - ชื่อแพ็กเกจ: ตัวพิมพ์เล็กทั้งหมด

### แพตเทิร์น Spring Boot
- ใช้ `@Service` สำหรับตรรกะธุรกิจ
- ใช้ `@RestController` สำหรับ REST endpoints
- กำหนดค่าผ่าน `application.yml` หรือ `application.properties`
- ชอบใช้ environment variables มากกว่าค่าที่กำหนดในโค้ด
- ใช้ annotation `@Tool` สำหรับเมธอดที่เปิดเผยผ่าน MCP

### การจัดระเบียบไฟล์
```
src/
├── main/
│   ├── java/
│   │   └── com/microsoft/[component]/
│   │       ├── [Component]Application.java
│   │       ├── config/
│   │       ├── controller/
│   │       ├── service/
│   │       └── exception/
│   └── resources/
│       ├── application.yml
│       └── static/
└── test/
    └── java/
        └── com/microsoft/[component]/
```

### การจัดการ Dependency
- จัดการผ่าน Maven `pom.xml`
- Spring AI BOM สำหรับจัดการเวอร์ชัน
- LangChain4j สำหรับการเชื่อมต่อ AI
- Spring Boot starter parent สำหรับ dependency ของ Spring

### คอมเมนต์โค้ด
- เพิ่ม JavaDoc สำหรับ public APIs
- รวมคอมเมนต์อธิบายสำหรับการติดต่อกับ AI ที่ซับซ้อน
- บันทึกคำอธิบายเครื่องมือ MCP อย่างชัดเจน

## การสร้างและการนำขึ้นใช้งาน

### การสร้างโครงการ

**สร้างโดยไม่รันทดสอบ:**
```bash
mvn clean install -DskipTests
```

**สร้างพร้อมตรวจสอบทั้งหมด:**
```bash
mvn clean install
```

**แพ็กเกจแอปพลิเคชัน:**
```bash
mvn package
# สร้างไฟล์ JAR ในไดเรกทอรี target/
```

### โฟลเดอร์ผลลัพธ์
- คลาสที่คอมไพล์แล้ว: `target/classes/`
- คลาสทดสอบ: `target/test-classes/`
- ไฟล์ JAR: `target/*.jar`
- artifact ของ Maven: `target/`

### การกำหนดค่าสภาพแวดล้อมตามการใช้งาน

**สภาพแวดล้อมการพัฒนา:**
```yaml
# application.yml (keyless - no api-key; auth via DefaultAzureCredential)
spring:
  ai:
    azure:
      openai:
        endpoint: ${AZURE_OPENAI_ENDPOINT}
        chat:
          options:
            deployment-name: ${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}
```

**สภาพแวดล้อมการผลิต:**
- ใช้ managed identity แทน `az login` สำหรับ keyless auth
- ตั้งค่า `AZURE_OPENAI_ENDPOINT` ชี้ไปยัง Foundry resource ใน production
- จัดการการตั้งค่าผ่าน environment variables หรือ Azure Key Vault

### ข้อควรพิจารณาในการนำขึ้นใช้งาน
- นี่คือที่เก็บโค้ดเพื่อการศึกษาและตัวอย่างแอปพลิเคชัน
- ไม่ได้ออกแบบมาเพื่อนำขึ้นใช้งาน production โดยตรง
- ตัวอย่างแสดงแพตเทิร์นให้สามารถปรับใช้ใน production ได้
- ดู README ของแต่ละโปรเจ็กต์สำหรับหมายเหตุการนำขึ้นใช้งานเฉพาะ

## หมายเหตุเพิ่มเติม

### Azure AI Foundry
- **การยืนยันตัวตนแบบ keyless:** เชื่อมต่อกับ Microsoft Entra ID — ไม่มีคีย์ API ให้บริหารจัดการ
- **จัดเตรียมเป็นโค้ด:** Bicep + azd (`azd up`) สร้างบัญชีและการติดตั้งโมเดล
- โค้ดที่เข้ากันกับ OpenAI เดียวกันทำงานได้ทั้งในเครื่อง (`az login`) และบน Azure (managed identity)

### การทำงานกับหลายโปรเจ็กต์
แต่ละโปรเจ็กต์ตัวอย่างแยกส่วน:
```bash
# ไปยังโปรเจกต์เฉพาะ
cd 04-PracticalSamples/[project-name]

# แต่ละโปรเจกต์มีไฟล์ pom.xml ของตัวเองและสามารถสร้างได้อย่างอิสระ
mvn clean install
```

### ปัญหาทั่วไป

**ความไม่ตรงกันของเวอร์ชัน Java:**
```bash
# ตรวจสอบ Java 21
java -version
# อัปเดต JAVA_HOME หากจำเป็น
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**ปัญหาการดาวน์โหลด Dependency:**
```bash
# ล้างแคช Maven และลองใหม่อีกครั้ง
rm -rf ~/.m2/repository
mvn clean install
```

**พบปัญหา endpoint หรือการลงชื่อเข้าใช้ไม่เจอ:**
```bash
# ตั้งค่า endpoint ในเซสชันปัจจุบันและเข้าสู่ระบบ (ไม่ใช้คีย์)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# หรือใช้ไฟล์ .env ในไดเรกทอรีโปรเจกต์
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**พอร์ตถูกใช้งานแล้ว:**
```bash
# Spring Boot ใช้พอร์ต 8080 เป็นค่าเริ่มต้น
# เปลี่ยนใน application.properties:
server.port=8081
```

### การสนับสนุนหลายภาษา
- เอกสารพร้อมใช้งานใน 45+ ภาษา ผ่านการแปลอัตโนมัติ
- ไฟล์แปลอยู่ในไดเรกทอรี `translations/`
- การแปลจัดการด้วย workflow ของ GitHub Actions

### เส้นทางการเรียนรู้
1. เริ่มต้นที่ [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. เรียงตามลำดับบทเรียน (01 → 05)
3. ทำตัวอย่างลงมือทำในแต่ละบท
4. สำรวจโครงการตัวอย่างในบทที่ 4
5. เรียนรู้หลักการ responsible AI ในบทที่ 5

### คอนเทนเนอร์สำหรับการพัฒนา
ไฟล์ `.devcontainer/devcontainer.json` กำหนด:
- สภาพแวดล้อมพัฒนา Java 21
- ติดตั้ง Maven ล่วงหน้า
- ส่วนขยาย Java ของ VS Code
- เครื่องมือ Spring Boot
- การรวม GitHub Copilot
- รองรับ Docker-in-Docker
- Azure CLI

### ข้อควรพิจารณาด้านประสิทธิภาพ
- Azure AI Foundry มีโควต้านับโทเค็น/คำร้องต่อนาที
- ใช้ batch ขนาดเหมาะสมสำหรับ embeddings
- พิจารณาแคชผลลัพธ์สำหรับเรียก API ซ้ำ
- ตรวจสอบการใช้โทเค็นเพื่อประหยัดค่าใช้จ่าย

### หมายเหตุด้านความปลอดภัย
- หลีกเลี่ยงการ commit ไฟล์ `.env` (ไฟล์นี้มีใน `.gitignore` แล้ว)
- แนะนำใช้ keyless auth (Microsoft Entra ID) แทนคีย์ API
- ใช้ managed identities บน Azure; ใช้ `az login` สำหรับพัฒนาในเครื่อง
- ปฏิบัติตามแนวทาง responsible AI ในบทที่ 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ปฏิเสธความรับผิดชอบ**:
เอกสารนี้ได้รับการแปลโดยใช้บริการแปลภาษา AI [Co-op Translator](https://github.com/Azure/co-op-translator) ขณะที่เราพยายามให้ความถูกต้อง โปรดทราบว่าการแปลโดยอัตโนมัติอาจมีข้อผิดพลาดหรือความไม่ถูกต้อง เอกสารต้นฉบับในภาษาต้นทางควรถูกพิจารณาเป็นแหล่งข้อมูลที่เชื่อถือได้ สำหรับข้อมูลที่สำคัญ แนะนำให้ใช้การแปลโดยมนุษย์มืออาชีพ เราไม่รับผิดชอบต่อความเข้าใจผิดหรือการตีความที่ผิดพลาดที่เกิดขึ้นจากการใช้การแปลนี้
<!-- CO-OP TRANSLATOR DISCLAIMER END -->