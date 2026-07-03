# Java용 생성 AI 개발 환경 설정

> **빠른 시작:** 몇 분 만에 Bicep + `azd`로 코드 기반 Azure AI Foundry에 AI 모델을 프로비전하세요 — [Azure AI Foundry 설정 가이드](getting-started-azure-openai.md)를 참조하세요. 인증은 **키 없는 방식**(Microsoft Entra ID)이므로 API 키를 관리할 필요가 없습니다.

## 학습 내용

- AI 애플리케이션을 위한 Java 개발 환경 설정
- 선호하는 개발 환경 선택 및 구성(클라우드 우선 Codespaces, 로컬 개발 컨테이너 또는 완전 로컬 설치)
- Azure AI Foundry 모델에 연결하여 설정 테스트

## 목차

- [학습 내용](#학습-내용)
- [소개](#소개)
- [1단계: 개발 환경 설정](#1단계-개발-환경-설정)
  - [옵션 A: GitHub Codespaces (권장)](#옵션-a-github-codespaces-권장)
  - [옵션 B: 로컬 개발 컨테이너](#옵션-b-로컬-개발-컨테이너)
  - [옵션 C: 기존 로컬 설치 사용](#옵션-c-기존-로컬-설치-사용)
- [2단계: Azure AI Foundry 프로비전](#2단계-azure-ai-foundry-프로비전)
- [3단계: 설정 테스트](#3단계-설정-테스트)
- [문제 해결](#문제-해결)
- [요약](#요약)
- [다음 단계](#다음-단계)

## 소개

이 장에서는 개발 환경 설정 방법을 안내합니다. 전체 과정에서 모델은 <strong>Azure AI Foundry</strong>를 사용합니다. Bicep 및 Azure Developer CLI(`azd`)를 사용해 코드를 통해 모델을 프로비전하고, **키 없는 인증**(Microsoft Entra ID)으로 안전하게 연결합니다 — API 키를 복사하거나 노출할 필요가 없습니다.

**로컬 설치가 필요 없습니다!** 전체 개발 환경이 브라우저에서 제공되는 GitHub Codespaces를 이용할 수 있으며, 거기서 Foundry를 프로비전할 수 있습니다.

이 과정을 위해 <strong>Azure AI Foundry</strong>를 사용하는 이유:
- **코드로 프로비전** — `azd up` 한 번으로 계정과 모델 배포까지 완료
- **키 없는 인증** — Azure 로그인 또는 관리 ID로 인증
- **프로덕션 준비 완료** — 동일 코드가 로컬과 Azure에서 동작
- <strong>유연성</strong> — 배포 이름만 바꾸면 모델 전환 가능, 코드 변경 불필요

> <strong>참고</strong>: Azure AI Foundry 배포 비용은 토큰 사용량 기반입니다(종량제). 프로비전, 지역, 비용 세부사항은 [Azure AI Foundry 설정 가이드](getting-started-azure-openai.md)를 참고하세요.


## 1단계: 개발 환경 설정

<a name="quick-start-cloud"></a>

설정 시간을 최소화하고 Generative AI for Java 과정에 필요한 모든 도구가 포함된 사전 구성된 개발 컨테이너를 만들었습니다. 원하는 개발 방식을 선택하세요:

### 환경 설정 옵션:

#### 옵션 A: GitHub Codespaces (권장)

**2분 만에 코딩 시작 — 로컬 설치 불필요!**

1. 이 저장소를 GitHub 계정에 포크
   > <strong>참고</strong>: 기본 구성을 편집하려면 [Dev Container 구성](../../../.devcontainer/devcontainer.json)을 참조하세요
2. **Code** → **Codespaces** 탭 → **...** → **새 옵션으로 생성(New with options...)** 클릭
3. 기본값 사용 — 이 과정에 맞춘 **Generative AI Java Development Environment** 개발 컨테이너 구성이 선택됨
4. **Codespace 생성(Create codespace)** 클릭
5. 약 2분 기다려 환경 준비 완료
6. [2단계: Azure AI Foundry 프로비전](#2단계-azure-ai-foundry-프로비전) 진행

<img src="../../../translated_images/ko/codespaces.9945ded8ceb431a5.webp" alt="스크린샷: Codespaces 하위메뉴" width="50%">

<img src="../../../translated_images/ko/image.833552b62eee7766.webp" alt="스크린샷: 새 옵션으로 생성" width="50%">

<img src="../../../translated_images/ko/codespaces-create.b44a36f728660ab7.webp" alt="스크린샷: Codespace 생성 옵션" width="50%">


> **Codespaces의 장점**:
> - 로컬 설치 불필요
> - 브라우저가 있는 모든 기기에서 사용 가능
> - 모든 도구 및 종속성 사전 구성 완료
> - 개인 계정은 매월 60시간 무료 제공
> - 모든 학습자에게 일관된 환경 제공

#### 옵션 B: 로컬 개발 컨테이너

**Docker를 이용한 로컬 개발 환경을 선호하는 개발자용**

1. 이 저장소를 포크하고 로컬 컴퓨터에 클론
   > <strong>참고</strong>: 기본 구성을 편집하려면 [Dev Container 구성](../../../.devcontainer/devcontainer.json)을 참조하세요
2. [Docker Desktop](https://www.docker.com/products/docker-desktop/)과 [VS Code](https://code.visualstudio.com/) 설치
3. VS Code에서 [Dev Containers 확장](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) 설치
4. VS Code로 저장소 폴더 열기
5. 프롬프트가 뜨면 **컨테이너에서 다시 열기(Reopen in Container)** 클릭 (또는 `Ctrl+Shift+P` → "Dev Containers: Reopen in Container" 사용)
6. 컨테이너가 빌드되고 시작될 때까지 대기
7. [2단계: Azure AI Foundry 프로비전](#2단계-azure-ai-foundry-프로비전) 진행

<img src="../../../translated_images/ko/devcontainer.21126c9d6de64494.webp" alt="스크린샷: 개발 컨테이너 설정" width="50%">

<img src="../../../translated_images/ko/image-3.bf93d533bbc84268.webp" alt="스크린샷: 개발 컨테이너 빌드 완료" width="50%">

#### 옵션 C: 기존 로컬 설치 사용

**기존 Java 환경을 가진 개발자용**

필수 조건:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html)
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) 또는 선호하는 IDE

단계:
1. 이 저장소를 로컬 컴퓨터에 클론
2. IDE에서 프로젝트 열기
3. [2단계: Azure AI Foundry 프로비전](#2단계-azure-ai-foundry-프로비전) 진행

> <strong>팁</strong>: 저사양 머신에서 로컬 VS Code만 쓰고 싶다면 GitHub Codespaces를 활용하세요! 로컬 VS Code를 클라우드의 Codespace에 연결해 두 환경의 장점을 동시에 누릴 수 있습니다.

<img src="../../../translated_images/ko/image-2.fc0da29a6e4d2aff.webp" alt="스크린샷: 로컬 개발 컨테이너 인스턴스 생성됨" width="50%">


## 2단계: Azure AI Foundry 프로비전

과정의 AI 모델을 코드 기반으로 Azure AI Foundry에 배포합니다. 저장소 루트에서:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```
  
`azd`가 환경 이름과 지역을 묻고, `gpt-4o-mini` 및 `text-embedding-3-small` 배포가 포함된 Azure AI Foundry 계정을 프로비전하며, 예제의 `.env` 파일에 엔드포인트를 기록합니다 — 모두 **키 없는 인증**(API 키 없음)으로 수행됩니다.

> **전체 안내:** [Azure AI Foundry 설정 가이드](getting-started-azure-openai.md)에서 사전 조건, 수동(포털) 대안, 지역 안내, 비용 및 정리 참고.

## 3단계: 설정 테스트

Foundry 모델이 프로비전되면 [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure)에 있는 예제 앱으로 연결을 테스트하세요.

1. 개발 환경에서 터미널 열기
2. 예제 디렉터리로 이동:
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
  
3. 로그인되어 있는지 확인(키 없는 인증에는 토큰 필요):
   ```bash
   az login
   ```
  
   > `azd up`을 실행했다면 엔드포인트가 포함된 `.env` 파일이 이미 생성되어 있습니다.  
4. 애플리케이션 실행:
   ```bash
   mvn clean spring-boot:run
   ```
  
`gpt-4o-mini` 모델의 응답을 볼 수 있어야 합니다.

### 예제 코드 이해하기

`examples/basic-chat-azure`에 있는 예제는 Spring Boot 앱으로, <strong>Spring AI</strong>를 사용해 키 없는 인증으로 Azure AI Foundry에 연결합니다.

**이 코드가 하는 일**:
- Azure 로그인(Microsoft Entra ID)을 이용해 Azure AI Foundry에 <strong>연결</strong> — API 키 불필요  
- `gpt-4o-mini` 모델에 프롬프트를 <strong>전송</strong>  
- AI의 응답을 **수신 및 표시**  
- 설정이 제대로 되었는지 <strong>검증</strong>

**주요 의존성** (`pom.xml`):  
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```
  
<strong>구성</strong> (`application.yml`):  
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
  
## 요약

잘 하셨습니다! 다음을 모두 완료했습니다:

- Bicep + `azd`로 Azure AI Foundry 모델을 코드 기반 프로비전  
- Java 개발 환경 실행( Codespaces, 개발 컨테이너, 로컬 중 어떤 환경이든)  
- 키 없는 인증(Microsoft Entra ID)으로 Azure AI Foundry에 연결 — API 키 불필요  
- 모델과 간단하게 대화하는 예제로 모든 것이 정상 동작함을 확인

## 다음 단계

[3장: 핵심 생성 AI 기법](../03-CoreGenerativeAITechniques/README.md)

## 문제 해결

문제가 있나요? 자주 발생하는 문제와 해결책입니다:

- **인증 실패(401/403)?**  
  - `az login` 실행 — 인증은 키 없는 방식으로, 반드시 로그인 필요  
  - 계정에 해당 리소스에 대한 **Cognitive Services OpenAI User** 역할이 있는지 확인  
  - 방금 프로비전했다면 역할 할당 전파까지 1분 정도 대기  

- **Maven이 안 보이나?**  
  - 개발 컨테이너/ Codespaces 사용 중이면 Maven이 사전 설치되어 있음  
  - 로컬 설정 시 Java 21+, Maven 3.9+ 설치 여부 확인  
  - `mvn --version` 명령어로 설치 확인  

- **`azd` 명령어 없거나 프로비전 실패?**  
  - [Azure Developer CLI](https://aka.ms/azure-dev/install) 설치 후 `azd auth login` 실행  
  - `gpt-4o-mini`가 지원되는 지역 선택(예: `eastus2`)  
  - [Azure AI Foundry 설정 가이드](getting-started-azure-openai.md) 참고  

- **개발 컨테이너가 시작되지 않음?**  
  - Docker Desktop이 실행 중인지 확인(로컬 개발용)  
  - 컨테이너 재빌드 시도: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"  

- **애플리케이션 컴파일 오류?**  
  - 올바른 디렉터리인지 확인: `02-SetupDevEnvironment/examples/basic-chat-azure`  
  - 클린 후 재빌드 시도: `mvn clean compile`  

> **도움이 필요하신가요?** 문제가 계속되면 저장소에 이슈를 열어 문의하세요. 저희가 도와드리겠습니다.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**면책 조항**:
이 문서는 AI 번역 서비스 [Co-op Translator](https://github.com/Azure/co-op-translator)를 사용하여 번역되었습니다. 정확성을 기하기 위해 노력하고 있으나, 자동 번역은 오류나 부정확한 부분이 있을 수 있음을 유의하시기 바랍니다. 원본 문서의 원어본이 권위 있는 자료로 간주되어야 합니다. 중요한 정보의 경우, 전문가의 인간 번역을 권장합니다. 이 번역 사용으로 인해 발생하는 오해나 잘못된 해석에 대해 당사는 책임을 지지 않습니다.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->