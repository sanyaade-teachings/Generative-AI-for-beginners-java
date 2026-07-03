# บทนำสู่ Generative AI - ฉบับ Java

## สิ่งที่คุณจะได้เรียนรู้

- **พื้นฐานของ Generative AI** รวมถึง LLMs, การออกแบบ prompt, tokens, embeddings, และฐานข้อมูลเวกเตอร์
- **เปรียบเทียบเครื่องมือพัฒนา AI บน Java** รวมถึง Azure OpenAI SDK, Spring AI, และ OpenAI Java SDK
- **ค้นพบ Model Context Protocol** และบทบาทของมันในสื่อสารของเอเย่นต์ AI

## สารบัญ

- [บทนำ](#introduction)
- [ทบทวนแนวคิด Generative AI อย่างรวดเร็ว](#ทบทวนแนวคิด-generative-ai-อย่างรวดเร็ว)
- [ทบทวนการออกแบบ prompt](#ทบทวนการออกแบบ-prompt)
- [tokens, embeddings, และ agents](#tokens-embeddings-และ-agents)
- [เครื่องมือและไลบรารีสำหรับการพัฒนา AI บน Java](#เครื่องมือและไลบรารีสำหรับการพัฒนา-ai-บน-java)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [สรุป](#สรุป)
- [ขั้นตอนถัดไป](#ขั้นตอนถัดไป)

## Introduction

ยินดีต้อนรับสู่บทแรกของ Generative AI for Beginners - ฉบับ Java! บทเรียนพื้นฐานนี้จะแนะนำแนวคิดหลักของ Generative AI และวิธีการทำงานกับพวกมันโดยใช้ Java คุณจะได้เรียนรู้เกี่ยวกับองค์ประกอบสำคัญของแอปพลิเคชัน AI รวมถึง Large Language Models (LLMs), tokens, embeddings, และ AI agents เรายังจะสำรวจเครื่องมือหลักของ Java ที่คุณจะใช้ตลอดหลักสูตรนี้ด้วย

### ทบทวนแนวคิด Generative AI อย่างรวดเร็ว

Generative AI คือรูปแบบของปัญญาประดิษฐ์ที่สร้างเนื้อหาใหม่ เช่น ข้อความ รูปภาพ หรือโค้ด โดยอาศัยรูปแบบและความสัมพันธ์ที่เรียนรู้จากข้อมูล โมเดล Generative AI สามารถสร้างคำตอบที่เหมือนมนุษย์ เข้าใจบริบท และบางครั้งก็สร้างเนื้อหาที่ดูเหมือนมนุษย์สร้างขึ้น

เมื่อคุณพัฒนาแอปพลิเคชัน AI บน Java คุณจะทำงานกับ **โมเดล generative AI** เพื่อสร้างเนื้อหา ความสามารถบางอย่างของโมเดล generative AI ได้แก่:

- **การสร้างข้อความ**: สร้างข้อความที่คล้ายมนุษย์สำหรับแชทบอท เนื้อหา และการเติมข้อความ
- **การสร้างและวิเคราะห์ภาพ**: ผลิตภาพที่สมจริง ปรับปรุงภาพถ่าย และตรวจจับวัตถุ
- **การสร้างโค้ด**: เขียนโค้ดสั้นหรือสคริปต์

มีโมเดลเฉพาะที่ปรับแต่งสำหรับงานต่างๆ ตัวอย่างเช่น **Small Language Models (SLMs)** และ **Large Language Models (LLMs)** สามารถจัดการการสร้างข้อความได้ โดย LLMs มักให้ประสิทธิภาพดีกว่าสำหรับงานที่ซับซ้อน สำหรับงานที่เกี่ยวกับภาพ คุณจะใช้โมเดลวิชันเฉพาะทางหรือโมเดลมัลติ-โหมด

![Figure: Generative AI model types and use cases.](../../../translated_images/th/llms.225ca2b8a0d34473.webp)

แน่นอนว่าคำตอบจากโมเดลเหล่านี้ไม่ได้สมบูรณ์แบบตลอดเวลา คุณอาจเคยได้ยินเกี่ยวกับโมเดลที่ "หลอกหลอน" หรือสร้างข้อมูลที่ผิดพลาดในลักษณะที่มั่นใจ แต่คุณสามารถช่วยนำทางโมเดลเพื่อสร้างคำตอบที่ดียิ่งขึ้นโดยการให้คำแนะนำและบริบทที่ชัดเจน นั่นคือจุดที่ **prompt engineering** มีบทบาท

#### ทบทวนการออกแบบ prompt

Prompt engineering คือการออกแบบอินพุตที่มีประสิทธิภาพเพื่อชี้นำโมเดล AI ให้ได้ผลลัพธ์ที่ต้องการ ประกอบด้วย:

- **ความชัดเจน**: ทำให้คำสั่งชัดเจนและไม่กำกวม
- **บริบท**: ให้ข้อมูลเบื้องหลังที่จำเป็น
- **ข้อจำกัด**: ระบุข้อจำกัดหรือรูปแบบใดๆ

แนวทางปฏิบัติที่ดีที่สุดสำหรับ prompt engineering ได้แก่ การออกแบบ prompt, คำแนะนำที่ชัดเจน, การแยกงาน, การเรียนรู้แบบ one-shot และ few-shot, และการปรับจูน prompt การทดสอบ prompt ที่แตกต่างกันเป็นสิ่งสำคัญเพื่อค้นหาสิ่งที่เหมาะสมที่สุดสำหรับกรณีการใช้งานของคุณ

เมื่อพัฒนาแอปพลิเคชัน คุณจะทำงานกับ prompt ประเภทต่างๆ:
- **System prompts**: กำหนดกฎพื้นฐานและบริบทสำหรับพฤติกรรมของโมเดล
- **User prompts**: ข้อมูลอินพุตจากผู้ใช้แอปพลิเคชันของคุณ
- **Assistant prompts**: คำตอบของโมเดลที่อิงจากระบบและ user prompts

> **เรียนรู้เพิ่มเติม**: เรียนรู้เพิ่มเติมเกี่ยวกับ prompt engineering ใน [บท Prompt Engineering ของหลักสูตร GenAI for Beginners](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### tokens, embeddings, และ agents

เมื่อทำงานกับโมเดล generative AI คุณจะเจอคำว่า **tokens**, **embeddings**, **agents**, และ **Model Context Protocol (MCP)** นี่คือภาพรวมโดยละเอียดของแนวคิดเหล่านี้:

- **Tokens**: Tokens คือหน่วยข้อความที่เล็กที่สุดในโมเดล อาจเป็นคำ ตัวอักษร หรือ subwords Tokens ใช้แทนข้อมูลข้อความในรูปแบบที่โมเดลเข้าใจได้ ตัวอย่างประโยค "The quick brown fox jumped over the lazy dog" อาจถูกแยกเป็น tokens เช่น ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] หรือ ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"] ขึ้นอยู่กับกลยุทธ์การตัด token

![Figure: Generative AI tokens example of breaking words into tokens](../../../translated_images/th/tokens.6283ed277a2ffff4.webp)

การตัด token คือกระบวนการแบ่งข้อความเป็นหน่วยเล็กเหล่านี้ สิ่งนี้สำคัญเพราะโมเดลทำงานบน tokens แทนการใช้ข้อความดิบ จำนวน token ใน prompt ส่งผลต่อความยาวและคุณภาพของคำตอบ เนื่องจากโมเดลมีข้อจำกัดเรื่องจำนวน token ภายในหน้าต่างบริบท (เช่น 128K tokens สำหรับบริบททั้งหมดของ GPT-4o ที่รวมทั้งอินพุตและเอาต์พุต)

  ใน Java คุณสามารถใช้ไลบรารีเช่น OpenAI SDK เพื่อจัดการการตัด token อัตโนมัติขณะส่งคำขอไปยังโมเดล AI

- **Embeddings**: Embeddings คือการแทนข้อความในรูปแบบเวกเตอร์ที่จับความหมายเชิงสัญลักษณ์ พวกมันเป็นตัวแทนเชิงตัวเลข (โดยปกติเป็นอาเรย์ของค่าทศนิยม) ที่ช่วยให้โมเดลเข้าใจความสัมพันธ์ระหว่างคำและสร้างคำตอบที่มีบริบทเหมาะสม คำที่มีความหมายใกล้เคียงกันจะมี embeddings ที่คล้ายคลึงกัน ทำให้โมเดลเข้าใจแนวคิดเช่นคำเหมือนและความสัมพันธ์เชิงสัญลักษณ์

![Figure: Embeddings](../../../translated_images/th/embedding.398e50802c0037f9.webp)

  ใน Java คุณสามารถสร้าง embeddings โดยใช้ OpenAI SDK หรือไลบรารีอื่นๆ ที่รองรับการสร้าง embeddings embeddings เหล่านี้จำเป็นสำหรับงานเช่น semantic search ซึ่งต้องการค้นหาเนื้อหาที่คล้ายกันโดยอิงจากความหมายแทนการจับคู่ข้อความตรงๆ

- **ฐานข้อมูลเวกเตอร์**: ฐานข้อมูลเวกเตอร์คือระบบเก็บข้อมูลที่ออกแบบมาเฉพาะสำหรับ embeddings ช่วยให้การค้นหาความคล้ายคลึงทำได้อย่างมีประสิทธิภาพและสำคัญสำหรับรูปแบบ Retrieval-Augmented Generation (RAG) ซึ่งคุณต้องการค้นหาข้อมูลที่เกี่ยวข้องจากชุดข้อมูลขนาดใหญ่โดยอิงจากความหมายเชิงสัญลักษณ์แทนการจับคู่ข้อความตรงๆ

![Figure: Vector database architecture showing how embeddings are stored and retrieved for similarity search.](../../../translated_images/th/vector.f12f114934e223df.webp)

> **หมายเหตุ**: ในหลักสูตรนี้ เราจะไม่ครอบคลุมเรื่องฐานข้อมูลเวกเตอร์ แต่เห็นว่าน่าสนใจที่จะกล่าวถึงเพราะใช้กันอย่างแพร่หลายในแอปพลิเคชันจริง

- **Agents & MCP**: ส่วนประกอบ AI ที่โต้ตอบกับโมเดล เครื่องมือ และระบบภายนอกโดยอัตโนมัติ Model Context Protocol (MCP) มอบวิธีมาตรฐานสำหรับเอเย่นต์เพื่อเข้าถึงแหล่งข้อมูลภายนอกและเครื่องมืออย่างปลอดภัย เรียนรู้เพิ่มเติมในหลักสูตร [MCP for Beginners](https://github.com/microsoft/mcp-for-beginners)

ในแอปพลิเคชัน AI บน Java คุณจะใช้ tokens สำหรับประมวลผลข้อความ, embeddings สำหรับการค้นหาความหมายและ RAG, ฐานข้อมูลเวกเตอร์สำหรับการดึงข้อมูล, และ agents ที่ใช้ MCP เพื่อสร้างระบบอัจฉริยะที่ใช้เครื่องมือได้

![Figure: how a prompt becomes a reply—tokens, vectors, optional RAG lookup, LLM thinking, and an MCP agent all in one quick flow..](../../../translated_images/th/flow.f4ef62c3052d12a8.webp)

### เครื่องมือและไลบรารีสำหรับการพัฒนา AI บน Java

Java มีเครื่องมือพัฒนาที่ยอดเยี่ยมสำหรับ AI มีไลบรารีหลักสามตัวที่เราจะสำรวจตลอดหลักสูตร - OpenAI Java SDK, Azure OpenAI SDK, และ Spring AI

นี่คือตารางอ้างอิงเร็วแสดงว่า SDK ใดถูกใช้ในตัวอย่างของแต่ละบท:

| Chapter | Sample | SDK |
|---------|--------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**ลิงก์เอกสาร SDK:**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

OpenAI SDK คือไลบรารี Java อย่างเป็นทางการสำหรับ OpenAI API ให้อินเทอร์เฟซที่ง่ายและสม่ำเสมอในการโต้ตอบกับโมเดลของ OpenAI ทำให้ง่ายในการรวมความสามารถ AI เข้ากับแอปพลิเคชัน Java แอปพลิเคชัน Pet Story ในบทที่ 4 และตัวอย่าง Foundry Local แสดงวิธีใช้ OpenAI SDK ร่วมกับ Azure AI Foundry

#### Spring AI

Spring AI คือเฟรมเวิร์กครบวงจรที่นำความสามารถ AI เข้าสู่แอปพลิเคชัน Spring ให้ชั้นนามธรรมที่สอดคล้องกันทั่วผู้ให้บริการ AI ต่างๆ ผสานรวมอย่างไร้รอยต่อกับระบบนิเวศ Spring ทำให้เป็นตัวเลือกที่เหมาะสำหรับแอปพลิเคชัน Java ระดับองค์กรที่ต้องการความสามารถ AI

จุดแข็งของ Spring AI คือการบูรณาการอย่างไร้รอยต่อกับระบบนิเวศ Spring ทำให้ง่ายต่อการสร้างแอปพลิเคชัน AI พร้อมใช้งานในผลิตภัณฑ์จริงด้วยรูปแบบ Spring ที่คุ้นเคยเช่น การฉีดพึ่งพิง, การจัดการคอนฟิก, และกรอบการทดสอบ คุณจะใช้ Spring AI ในบทที่ 2 และ 4 เพื่อสร้างแอปพลิเคชันที่ใช้ได้ทั้ง OpenAI และ Model Context Protocol (MCP) ไลบรารี Spring AI

##### Model Context Protocol (MCP)

[Model Context Protocol (MCP)](https://modelcontextprotocol.io/) คือมาตรฐานใหม่ที่ช่วยให้แอปพลิเคชัน AI โต้ตอบกับแหล่งข้อมูลและเครื่องมือภายนอกอย่างปลอดภัย MCP ให้วิธีมาตรฐานสำหรับโมเดล AI ในการเข้าถึงข้อมูลบริบทและดำเนินการในแอปพลิเคชันของคุณ

ในบทที่ 4 คุณจะสร้างบริการเครื่องคิดเลข MCP ง่ายๆ ที่แสดงพื้นฐานของ Model Context Protocol ด้วย Spring AI แสดงวิธีสร้างการผสานเครื่องมือขั้นพื้นฐานและสถาปัตยกรรมบริการ

#### Azure OpenAI Java SDK

ไลบรารีลูกค้า Azure OpenAI สำหรับ Java เป็นการปรับใช้งาน REST API ของ OpenAI ที่ให้อินเทอร์เฟซที่คล้ายคลึงกับภาษาพร้อมการรวมเข้ากับระบบนิเวศ Azure SDK อื่นๆ ในบทที่ 3 คุณจะสร้างแอปพลิเคชันโดยใช้ Azure OpenAI SDK รวมถึงแอปแชท การเรียกฟังก์ชัน และรูปแบบ RAG (Retrieval-Augmented Generation)

> หมายเหตุ: Azure OpenAI SDK ล่าช้ากว่า OpenAI Java SDK ในเรื่องฟีเจอร์ ดังนั้นสำหรับโปรเจกต์ในอนาคตแนะนำให้ใช้ OpenAI Java SDK

## สรุป

นี่คือพื้นฐานทั้งหมด! ตอนนี้คุณเข้าใจ:

- แนวคิดหลักเบื้องหลัง generative AI - ตั้งแต่ LLMs และการออกแบบ prompt ถึง tokens, embeddings และฐานข้อมูลเวกเตอร์
- ตัวเลือกเครื่องมือพัฒนาสำหรับ Java AI: Azure OpenAI SDK, Spring AI, และ OpenAI Java SDK
- Model Context Protocol คืออะไรและช่วยให้เอเย่นต์ AI สามารถทำงานร่วมกับเครื่องมือภายนอกได้อย่างไร

## ขั้นตอนถัดไป

[บทที่ 2: การตั้งค่าสภาพแวดล้อมการพัฒนา](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**ปฏิเสธความรับผิดชอบ**:
เอกสารนี้ได้รับการแปลโดยใช้บริการแปลภาษา AI [Co-op Translator](https://github.com/Azure/co-op-translator) ขณะที่เราพยายามให้ความถูกต้อง โปรดทราบว่าการแปลโดยอัตโนมัติอาจมีข้อผิดพลาดหรือความไม่ถูกต้อง เอกสารต้นฉบับในภาษาต้นทางควรถูกพิจารณาเป็นแหล่งข้อมูลที่เชื่อถือได้ สำหรับข้อมูลที่สำคัญ แนะนำให้ใช้การแปลโดยมนุษย์มืออาชีพ เราไม่รับผิดชอบต่อความเข้าใจผิดหรือการตีความที่ผิดพลาดที่เกิดขึ้นจากการใช้การแปลนี้
<!-- CO-OP TRANSLATOR DISCLAIMER END -->