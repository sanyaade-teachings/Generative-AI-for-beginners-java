# 실용적인 응용 프로그램 및 프로젝트

## 학습 내용
이 섹션에서는 Java를 사용한 생성형 AI 개발 패턴을 보여주는 세 가지 실용적인 응용 프로그램을 시연합니다:
- 클라이언트 측과 서버 측 AI를 결합한 다중 모달 펫 스토리 생성기 만들기
- Foundry Local Spring Boot 데모를 사용한 로컬 AI 모델 통합 구현
- 계산기 예제를 사용하여 모델 컨텍스트 프로토콜(MCP) 서비스 개발

## 목차

- [소개](#소개)
  - [Foundry Local Spring Boot 데모](#foundry-local-spring-boot-데모)
  - [펫 스토리 생성기](#펫-스토리-생성기)
  - [MCP 계산기 서비스 (초보자용 MCP 데모)](#mcp-계산기-서비스-초보자용-mcp-데모)
- [학습 진행](#학습-진행)
- [요약](#요약)
- [다음 단계](#다음-단계)

## 소개

이 장에서는 Java로 생성형 AI 개발 패턴을 보여주는 <strong>샘플 프로젝트</strong>를 소개합니다. 각 프로젝트는 완전한 기능을 갖추고 있으며, 특정 AI 기술, 아키텍처 패턴 및 모범 사례를 시연하여 본인의 응용 프로그램에 적용할 수 있도록 합니다.

### Foundry Local Spring Boot 데모

<strong>[Foundry Local Spring Boot 데모](foundrylocal/README.md)</strong>는 <strong>OpenAI Java SDK</strong>를 사용하여 로컬 AI 모델과 통합하는 방법을 보여줍니다. Foundry Local(예: **Phi-4-mini**)에서 실행되는 모델에 연결하고 자동 모델 감지를 지원하여 클라우드 서비스에 의존하지 않고 AI 애플리케이션을 실행할 수 있습니다.

### 펫 스토리 생성기

<strong>[펫 스토리 생성기](petstory/README.md)</strong>는 <strong>다중 모달 AI 처리</strong>를 통해 창의적인 펫 스토리를 생성하는 재미있는 Spring Boot 웹 애플리케이션입니다. transformer.js를 사용한 브라우저 기반 AI 상호작용과 OpenAI SDK를 이용한 서버 측 처리를 결합합니다.

### MCP 계산기 서비스 (초보자용 MCP 데모)

<strong>[MCP 계산기 서비스](calculator/README.md)</strong>는 Spring AI를 이용한 <strong>모델 컨텍스트 프로토콜(MCP)</strong>의 간단한 시연입니다. MCP 개념을 초보자가 쉽게 이해할 수 있도록 기본 MCP 서버를 생성하고 MCP 클라이언트와 상호작용하는 방법을 보여줍니다.

## 학습 진행

이 프로젝트들은 이전 장의 개념을 바탕으로 구성되었습니다:

1. **간단하게 시작하기**: Foundry Local Spring Boot 데모부터 시작하여 로컬 모델과의 기본 AI 통합 이해
2. **상호작용 추가**: 펫 스토리 생성기로 다중 모달 AI 및 웹 기반 상호작용 학습
3. **MCP 기본 배우기**: MCP 계산기 서비스를 통해 모델 컨텍스트 프로토콜 기본 개념 이해

## 요약

수고하셨습니다! 실제 응용 프로그램을 이제 탐색했습니다:

- 브라우저와 서버 모두에서 작동하는 다중 모달 AI 경험
- 최신 Java 프레임워크 및 SDK를 이용한 로컬 AI 모델 통합
- 도구가 AI와 통합되는 방식을 보여주는 첫 번째 모델 컨텍스트 프로토콜 서비스

## 다음 단계

[5장: 책임감 있는 생성 AI](../05-ResponsibleGenAI/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**면책 조항**:
이 문서는 AI 번역 서비스 [Co-op Translator](https://github.com/Azure/co-op-translator)를 사용하여 번역되었습니다. 정확성을 기하기 위해 노력하고 있으나, 자동 번역은 오류나 부정확한 부분이 있을 수 있음을 유의하시기 바랍니다. 원본 문서의 원어본이 권위 있는 자료로 간주되어야 합니다. 중요한 정보의 경우, 전문가의 인간 번역을 권장합니다. 이 번역 사용으로 인해 발생하는 오해나 잘못된 해석에 대해 당사는 책임을 지지 않습니다.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->