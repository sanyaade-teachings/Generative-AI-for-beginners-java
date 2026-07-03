# 생성형 AI 소개 - 자바 편

## 학습 내용

- LLM, 프롬프트 엔지니어링, 토큰, 임베딩, 벡터 데이터베이스를 포함한 **생성형 AI 기초**
- Azure OpenAI SDK, Spring AI, OpenAI Java SDK를 포함한 **자바 AI 개발 도구 비교**
- AI 에이전트 통신에서의 역할인 **모델 컨텍스트 프로토콜** 발견

## 목차

- [소개](#소개)
- [생성형 AI 개념 간단 복습](#생성형-ai-개념-간단-복습)
- [프롬프트 엔지니어링 검토](#프롬프트-엔지니어링-검토)
- [토큰, 임베딩, 에이전트](#토큰-임베딩-에이전트)
- [자바용 AI 개발 도구 및 라이브러리](#자바용-ai-개발-도구-및-라이브러리)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [요약](#요약)
- [다음 단계](#다음-단계)

## 소개

생성형 AI 입문 - 자바 편의 첫 장에 오신 것을 환영합니다! 이 기초 수업에서는 생성형 AI의 핵심 개념과 자바를 사용해 이를 다루는 방법을 소개합니다. 대형 언어 모델(LLM), 토큰, 임베딩, AI 에이전트 등 AI 애플리케이션의 필수 빌딩 블록에 대해 학습합니다. 또한 이 과정 전반에 사용할 주요 자바 도구도 살펴봅니다.

### 생성형 AI 개념 간단 복습

생성형 AI는 데이터에서 학습한 패턴과 관계를 기반으로 텍스트, 이미지 또는 코드를 새로 생성하는 인공지능 유형입니다. 생성형 AI 모델은 사람과 유사한 응답을 생성하고 맥락을 이해하며 경우에 따라 사람이 쓴 것 같은 콘텐츠를 만들어 내기도 합니다.

자바 AI 애플리케이션을 개발할 때 <strong>생성형 AI 모델</strong>을 사용해 콘텐츠를 생성합니다. 생성형 AI 모델의 기능은 다음과 같습니다:

- **텍스트 생성**: 챗봇, 콘텐츠, 텍스트 완성을 위한 사람 같은 텍스트 작성
- **이미지 생성 및 분석**: 사실적인 이미지 생성, 사진 향상, 객체 감지
- **코드 생성**: 코드 조각이나 스크립트 작성

특정 작업에 최적화된 모델 유형도 있습니다. 예를 들어, <strong>소형 언어 모델(SLM)</strong>과 **대형 언어 모델(LLM)** 모두 텍스트 생성을 처리할 수 있으나, LLM이 일반적으로 복잡한 작업에서 더 나은 성능을 제공합니다. 이미지 관련 작업에는 특화된 비전 모델이나 멀티모달 모델을 사용합니다.

![Figure: Generative AI model types and use cases.](../../../translated_images/ko/llms.225ca2b8a0d34473.webp)

물론, 이러한 모델들의 응답이 항상 완벽한 것은 아닙니다. 모델이 잘못된 정보를 권위 있게 생성하는, 이른바 "환각" 현상에 대해 들어보셨을 것입니다. 하지만 명확한 지시와 맥락을 제공하면 모델이 더 나은 응답을 생성하도록 유도할 수 있습니다. 이것이 바로 <strong>프롬프트 엔지니어링</strong>입니다.

#### 프롬프트 엔지니어링 검토

프롬프트 엔지니어링은 AI 모델이 원하는 출력으로 안내되도록 효과적인 입력을 설계하는 실습입니다. 여기에 포함되는 요소는 다음과 같습니다:

- <strong>명확성</strong>: 지시사항을 명확하고 모호하지 않게 작성
- <strong>맥락</strong>: 필요한 배경 정보를 제공
- **제약 조건**: 제한사항이나 형식을 명시

프롬프트 엔지니어링 모범 사례는 프롬프트 설계, 명확한 지시, 작업 분할, 원샷 및 소수 샷 학습, 프롬프트 튜닝 등이 있으며, 다양한 프롬프트를 테스트하여 특정 사용 사례에 가장 적합한 방식을 찾는 것이 중요합니다.

애플리케이션 개발 시 다음과 같은 다양한 프롬프트 유형을 사용합니다:
- **시스템 프롬프트**: 모델 행동의 기본 규칙과 맥락 설정
- **사용자 프롬프트**: 애플리케이션 사용자의 입력 데이터
- **어시스턴트 프롬프트**: 시스템과 사용자 프롬프트에 기반한 모델의 응답

> **자세히 알아보기**: [생성형 AI 입문 코스의 프롬프트 엔지니어링 챕터](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)에서 프롬프트 엔지니어링을 자세히 학습하세요.

#### 토큰, 임베딩, 에이전트

생성형 AI 모델 작업 시 <strong>토큰</strong>, <strong>임베딩</strong>, <strong>에이전트</strong>, <strong>모델 컨텍스트 프로토콜(MCP)</strong>라는 용어를 접하게 됩니다. 다음은 이 개념들에 대한 상세 개요입니다:

- <strong>토큰</strong>: 모델의 가장 작은 텍스트 단위입니다. 단어, 문자 또는 서브워드일 수 있습니다. 토큰은 모델이 이해할 수 있는 형식으로 텍스트 데이터를 표현하는 데 사용됩니다. 예를 들어, 문장 "The quick brown fox jumped over the lazy dog"는 토큰화 전략에 따라 ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] 또는 ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"]로 나뉠 수 있습니다.

![Figure: Generative AI tokens example of breaking words into tokens](../../../translated_images/ko/tokens.6283ed277a2ffff4.webp)

토큰화는 텍스트를 이러한 작은 단위로 분해하는 과정입니다. 모델은 원시 텍스트 대신 토큰을 기반으로 작동하기 때문에 중요합니다. 프롬프트 안의 토큰 수는 모델의 응답 길이와 품질에 영향을 미치며, 모델은 컨텍스트 창에 토큰 제한이 있습니다(예: GPT-4o는 입력과 출력을 포함해 총 128K 토큰).

  자바에서는 OpenAI SDK와 같은 라이브러리를 사용해 AI 모델에 요청할 때 자동으로 토큰화를 처리할 수 있습니다.

- <strong>임베딩</strong>: 임베딩은 토큰의 벡터 표현으로 의미를 포착합니다. 임베딩은 모델이 단어 간 관계를 이해하고 문맥에 맞는 응답을 생성할 수 있도록 하는 수치 표현(일반적으로 부동 소수점 수 배열)입니다. 의미가 비슷한 단어는 유사한 임베딩을 가지므로, 모델이 동의어 및 의미 관계를 이해할 수 있습니다.

![Figure: Embeddings](../../../translated_images/ko/embedding.398e50802c0037f9.webp)

  자바에서는 OpenAI SDK 또는 임베딩 생성을 지원하는 기타 라이브러리를 사용해 임베딩을 생성할 수 있습니다. 임베딩은 의미 기반 검색과 같이 정확한 텍스트 일치가 아닌 의미에 근거해 유사한 콘텐츠를 찾는 작업에 필수적입니다.

- **벡터 데이터베이스**: 벡터 데이터베이스는 임베딩 저장에 최적화된 특수 저장 시스템입니다. 이들은 효율적인 유사도 검색을 가능하게 하며, 의미적 유사성에 기반해 대규모 데이터셋에서 관련 정보를 찾는 검색 강화 생성(RAG) 패턴에서 중요합니다.

![Figure: Vector database architecture showing how embeddings are stored and retrieved for similarity search.](../../../translated_images/ko/vector.f12f114934e223df.webp)

> <strong>참고</strong>: 본 과정에서는 벡터 데이터베이스를 다루지 않지만, 실제 애플리케이션에서 흔히 사용되므로 언급할 가치가 있습니다.

- **에이전트 & MCP**: 모델, 도구, 외부 시스템과 자율적으로 상호작용하는 AI 구성 요소입니다. 모델 컨텍스트 프로토콜(MCP)은 에이전트가 외부 데이터 소스와 도구에 안전하게 접근할 수 있는 표준화된 방법을 제공합니다. 자세한 내용은 [초보자를 위한 MCP](https://github.com/microsoft/mcp-for-beginners) 코스를 참조하세요.

자바 AI 애플리케이션에서는 텍스트 처리에 토큰을, 의미 검색 및 RAG에 임베딩을, 데이터 검색에 벡터 데이터베이스를, 지능적 도구 사용 시스템 구축에 MCP를 활용하는 에이전트를 사용합니다.

![Figure: how a prompt becomes a reply—tokens, vectors, optional RAG lookup, LLM thinking, and an MCP agent all in one quick flow..](../../../translated_images/ko/flow.f4ef62c3052d12a8.webp)

### 자바용 AI 개발 도구 및 라이브러리

자바는 AI 개발을 위한 훌륭한 도구를 제공합니다. 이 과정에서 살펴볼 주요 라이브러리는 OpenAI Java SDK, Azure OpenAI SDK, Spring AI 세 가지입니다.

각 장별 예제에서 사용하는 SDK를 한눈에 보여주는 표는 다음과 같습니다:

| 장 | 샘플 | SDK |
|---------|--------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**SDK 문서 링크:**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

OpenAI SDK는 OpenAI API를 위한 공식 자바 라이브러리입니다. OpenAI 모델과 상호작용하는 단순하고 일관된 인터페이스를 제공해 자바 애플리케이션에 AI 기능을 쉽게 통합할 수 있습니다. 4장 Pet Story 애플리케이션과 Foundry Local 예제에서 OpenAI SDK 방식을 Azure AI Foundry와 함께 다룹니다.

#### Spring AI

Spring AI는 다양한 AI 제공자를 대상으로 일관된 추상화 계층을 제공하여 Spring 애플리케이션에 AI 기능을 도입하는 포괄적 프레임워크입니다. Spring 생태계와 원활하게 통합되며, AI 기능이 필요한 엔터프라이즈 자바 애플리케이션에 이상적인 선택입니다.

Spring AI의 강점은 익숙한 의존성 주입, 구성 관리, 테스트 프레임워크 같은 Spring 패턴을 이용해 프로덕션 수준 AI 애플리케이션 개발을 쉽게 해 준다는 점입니다. 2장과 4장에서 OpenAI와 모델 컨텍스트 프로토콜(MCP)용 Spring AI 라이브러리를 활용하는 애플리케이션을 구축합니다.

##### 모델 컨텍스트 프로토콜(MCP)

[모델 컨텍스트 프로토콜(MCP)](https://modelcontextprotocol.io/)은 AI 애플리케이션이 외부 데이터 소스와 도구와 안전하게 상호작용할 수 있게 하는 신흥 표준입니다. MCP는 AI 모델이 맥락 정보에 접근하고 애플리케이션에서 작업을 실행할 수 있는 표준화된 방법을 제공합니다.

4장에서는 Spring AI로 모델 컨텍스트 프로토콜의 기본 개념을 시연하는 간단한 MCP 계산기 서비스를 구축하며, 기본 도구 통합과 서비스 아키텍처 구현 방법을 보여줍니다.

#### Azure OpenAI Java SDK

Azure OpenAI 자바 클라이언트 라이브러리는 OpenAI REST API를 기반으로 하며, 관용적인 인터페이스와 Azure SDK 생태계와 통합을 제공합니다. 3장에서는 Azure OpenAI SDK를 사용해 챗 애플리케이션, 함수 호출, RAG(검색 강화 생성) 패턴을 구현합니다.

> 참고: 기능 면에서 Azure OpenAI SDK는 OpenAI Java SDK에 비해 뒤처지므로 앞으로 프로젝트에선 OpenAI Java SDK 사용을 고려하세요.

## 요약

기초 과정을 마쳤습니다! 이제 다음을 이해하게 되었습니다:

- LLM, 프롬프트 엔지니어링, 토큰, 임베딩, 벡터 데이터베이스 등 생성형 AI의 핵심 개념
- Azure OpenAI SDK, Spring AI, OpenAI Java SDK 등 자바 AI 개발 툴킷 선택지
- 모델 컨텍스트 프로토콜이 무엇이며 AI 에이전트가 외부 도구를 활용하는 방법

## 다음 단계

[2장: 개발 환경 설정](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**면책 조항**:
이 문서는 AI 번역 서비스 [Co-op Translator](https://github.com/Azure/co-op-translator)를 사용하여 번역되었습니다. 정확성을 기하기 위해 노력하고 있으나, 자동 번역은 오류나 부정확한 부분이 있을 수 있음을 유의하시기 바랍니다. 원본 문서의 원어본이 권위 있는 자료로 간주되어야 합니다. 중요한 정보의 경우, 전문가의 인간 번역을 권장합니다. 이 번역 사용으로 인해 발생하는 오해나 잘못된 해석에 대해 당사는 책임을 지지 않습니다.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->