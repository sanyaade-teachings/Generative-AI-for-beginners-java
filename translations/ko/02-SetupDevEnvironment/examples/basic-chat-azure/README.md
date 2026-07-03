# Azure AI Foundry와 함께하는 기본 채팅 - 종단 간 예제

이 예제는 **키리스 인증**(Microsoft Entra ID)을 사용하여 **Azure AI Foundry** 모델에 연결하고 설정을 테스트하는 간단한 Spring Boot 애플리케이션입니다. Spring AI의 `ChatClient`를 사용합니다.

## 목차

- [필수 조건](#필수-조건)
- [빠른 시작](#빠른-시작)
- [인증 작동 방식](#인증-작동-방식)
- [애플리케이션 실행](#애플리케이션-실행)
  - [Maven 사용](#maven-사용)
  - [VS Code 사용](#vs-code-사용)
  - [예상 출력](#예상-출력)
- [구성 참조](#구성-참조)
  - [환경 변수](#환경-변수)
  - [스프링 구성](#스프링-구성)
- [문제 해결](#문제-해결)
  - [일반 문제](#일반-문제)
  - [디버그 모드](#디버그-모드)
- [다음 단계](#다음-단계)
- [리소스](#리소스)

## 필수 조건

이 예제를 실행하기 전에 다음을 확인하세요:

- `gpt-4o-mini` 배포가 있는 Azure AI Foundry 리소스 — `azd up`으로 프로비저닝하거나 [Azure AI Foundry 설정 가이드](../../getting-started-azure-openai.md)를 통해 수동으로 설정
- 해당 리소스에 대한 **Cognitive Services OpenAI 사용자** 역할 (Bicep 템플릿이 자동으로 할당)
- `az login`으로 로그인된 [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- Java 21 이상 및 Maven 3.9 이상

> **API 키 불필요** — 인증은 Microsoft Entra ID를 통한 키리스 방식입니다.

## 빠른 시작

```bash
# 1. 프로젝트로 이동하세요
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. 키리스 인증이 토큰을 받을 수 있도록 로그인하세요
az login

# 3. 엔드포인트를 구성하세요
#    - 만약 `azd up`을 실행했다면, .env 파일이 자동으로 작성되었습니다 (이 단계는 건너뛰세요).
#    - 그렇지 않으면 템플릿을 복사하고 AZURE_OPENAI_ENDPOINT를 설정하세요.
cp .env.example .env

# 4. 애플리케이션을 실행하세요
mvn spring-boot:run
```

## 인증 작동 방식

이 예제는 <strong>Microsoft Entra ID</strong>로 인증합니다 — API 키가 없습니다.

`spring.ai.azure.openai.endpoint`만 설정되어 있고 api-key가 없으면, Spring AI는 [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential)을 사용하여 Azure OpenAI 클라이언트를 빌드합니다. 이 자격 증명은 로컬에서 `az login` 세션의 토큰을 자동으로 찾거나, Azure에서 실행 시 관리된 ID에서 토큰을 가져옵니다 — 따라서 동일한 코드가 두 환경 모두에서 변경 없이 작동합니다.

## 애플리케이션 실행

### Maven 사용

```bash
mvn spring-boot:run
```

### VS Code 사용

1. VS Code에서 프로젝트를 엽니다
2. `F5` 키를 누르거나 "실행 및 디버그" 패널을 사용합니다
3. "Spring Boot-BasicChatApplication" 구성을 선택합니다

> <strong>참고</strong>: VS Code 구성은 자동으로 `.env` 파일을 로드합니다

### 예상 출력

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

## 구성 참조

### 환경 변수

| 변수 | 설명 | 필수 | 예시 |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | Foundry (Azure OpenAI) 엔드포인트 URL | 예 | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | 채팅 모델 배포 이름 | 아니오 | `gpt-4o-mini` (기본값) |

> API 키 변수가 <strong>없음</strong> — 인증은 키리스 방식(Microsoft Entra ID via `az login`)입니다.

### 스프링 구성

`application.yml` 파일은 다음을 설정합니다:
- <strong>엔드포인트</strong>: `${AZURE_OPENAI_ENDPOINT}` - 환경 변수에서 가져옴
- <strong>배포</strong>: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - 환경 변수에서 가져오며 기본값 사용
- <strong>인증</strong>: 키리스 — `api-key` 미설정 시 Spring AI가 `DefaultAzureCredential` 사용
- <strong>온도</strong>: `0.7` - 창의성 조절 (0.0 = 결정적, 1.0 = 창의적)
- **최대 토큰**: `500` - 최대 응답 길이

## 문제 해결

### 일반 문제

<details>
<summary><strong>오류: 401 / "PermissionDenied" / 토큰 오류</strong></summary>

- `az login` 실행 — 키리스 인증은 활성 로그인 세션이 필요합니다
- 계정에 리소스에 대한 **Cognitive Services OpenAI 사용자** 역할이 있는지 확인
- 역할을 방금 할당한 경우, 전파될 때까지 잠시 기다리세요
- 올바른 테넌트/구독에 있는지 확인 (`az account show`)
</details>

<details>
<summary><strong>오류: "엔드포인트가 유효하지 않음" / 연결 오류</strong></summary>

- `AZURE_OPENAI_ENDPOINT`가 전체 기본 URL인지 확인 (예: `https://your-resource.openai.azure.com/`)
- 슬래시( / ) 일관성 확인
- 프로비저닝된 리소스와 엔드포인트가 일치하는지 확인 (`azd env get-values`)
</details>

<details>
<summary><strong>오류: "배포를 찾을 수 없음"</strong></summary>

- `AZURE_OPENAI_DEPLOYMENT`가 Azure 내 실제 배포 이름과 일치하는지 확인
- 모델이 성공적으로 배포되고 활성 상태인지 확인
- 기본 배포 이름은 `gpt-4o-mini`입니다
</details>

<details>
<summary><strong>VS Code: 환경 변수 로드가 안 될 때</strong></summary>

- `.env` 파일이 프로젝트 루트 디렉터리(`pom.xml`과 같은 위치)에 있는지 확인
- VS Code 통합 터미널에서 `mvn spring-boot:run` 실행 시도
- VS Code Java 확장이 올바르게 설치되어 있는지 확인
</details>

### 디버그 모드

자세한 로깅을 활성화하려면 `application.yml`에서 아래 라인의 주석을 해제하세요:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## 다음 단계

**설정 완료!** 학습 여정을 계속 진행하세요:

[3장: 핵심 생성 AI 기법](../../../03-CoreGenerativeAITechniques/README.md)

## 리소스

- [Spring AI Azure OpenAI 문서](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Microsoft Entra ID를 이용한 키리스 인증](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry 포털](https://ai.azure.com/)
- [Azure AI Foundry 문서](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**면책 조항**:
이 문서는 AI 번역 서비스 [Co-op Translator](https://github.com/Azure/co-op-translator)를 사용하여 번역되었습니다. 정확성을 기하기 위해 노력하고 있으나, 자동 번역은 오류나 부정확한 부분이 있을 수 있음을 유의하시기 바랍니다. 원본 문서의 원어본이 권위 있는 자료로 간주되어야 합니다. 중요한 정보의 경우, 전문가의 인간 번역을 권장합니다. 이 번역 사용으로 인해 발생하는 오해나 잘못된 해석에 대해 당사는 책임을 지지 않습니다.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->