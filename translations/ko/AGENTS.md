# AGENTS.md

## Project Overview

이 저장소는 Java로 생성 AI 개발을 학습하기 위한 교육용 저장소입니다. 대규모 언어 모델(LLM), 프롬프트 엔지니어링, 임베딩, RAG(검색 강화 생성), 모델 컨텍스트 프로토콜(MCP)을 포괄적으로 다루는 실습 과정을 제공합니다.

**주요 기술:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI 및 OpenAI SDK

**아키텍처:**
- 챕터별로 구성된 여러 독립형 Spring Boot 애플리케이션
- 다양한 AI 패턴을 시연하는 샘플 프로젝트
- 사전 구성된 개발 컨테이너와 함께 GitHub Codespaces 지원

## Setup Commands

### Prerequisites
- Java 21 이상
- Maven 3.x
- Azure AI Foundry 모델 배포가 포함된 Azure 구독 (`azd up`으로 프로비저닝)
- Azure CLI(`az`) 및 Azure Developer CLI(`azd`), 키리스 인증을 위해 로그인 완료

### Environment Setup

**옵션 1: GitHub Codespaces (권장)**
```bash
# 저장소를 포크하고 GitHub UI에서 코드스페이스를 만듭니다
# 개발 컨테이너가 모든 종속성을 자동으로 설치합니다
# 환경 설정을 위해 약 2분간 기다립니다
```

**옵션 2: 로컬 개발 컨테이너**
```bash
# 저장소 복제
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Dev Containers 확장 프로그램으로 VS Code에서 열기
# 요청 시 컨테이너에서 다시 열기
```

**옵션 3: 로컬 설정**
```bash
# 종속성 설치
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# 설치 확인
java -version
mvn -version
```

### Configuration

**Azure AI Foundry 설정 (키리스, 권장):**
```bash
# Foundry 계정 및 모델 배포를 코드로 프로비저닝합니다
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd는 예제/basic-chat-azure/.env 파일을 엔드포인트로 작성합니다 (키 없음)
```

**수동 엔드포인트 구성:**
```bash
# azd를 사용하지 않았다면, 엔드포인트를 직접 설정하세요 (인증은 az 로그인으로 키 없이 유지됩니다)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# .env 파일을 수정하세요: AZURE_OPENAI_ENDPOINT=https://<리소스>.openai.azure.com/
```

## Development Workflow

### 프로젝트 구조
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

### 애플리케이션 실행

**Spring Boot 애플리케이션 실행:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**프로젝트 빌드:**
```bash
cd [project-directory]
mvn clean install
```

**MCP 계산기 서버 시작:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# 서버가 http://localhost:8080 에서 실행됩니다
```

**클라이언트 예제 실행:**
```bash
# 다른 터미널에서 서버를 시작한 후
cd 04-PracticalSamples/calculator

# 직접 MCP 클라이언트
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# AI 기반 클라이언트 (AZURE_OPENAI_ENDPOINT + az 로그인 필요)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# 대화형 봇
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### 핫 리로드
핫 리로드를 지원하는 프로젝트에는 Spring Boot DevTools가 포함되어 있습니다:
```bash
# 자바 파일의 변경 사항은 저장 시 자동으로 다시 로드됩니다
mvn spring-boot:run
```

## Testing Instructions

### 테스트 실행

**프로젝트 내 모든 테스트 실행:**
```bash
cd [project-directory]
mvn test
```

**상세 출력으로 테스트 실행:**
```bash
mvn test -X
```

**특정 테스트 클래스 실행:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### 테스트 구조
- 테스트 파일은 JUnit 5 (Jupiter)를 사용
- 테스트 클래스는 `src/test/java/`에 위치
- 계산기 프로젝트의 클라이언트 예제는 `src/test/java/com/microsoft/mcp/sample/client/`에 위치

### 수동 테스트
많은 예제는 수동 테스트가 필요한 인터랙티브한 애플리케이션입니다:

1. `mvn spring-boot:run`으로 애플리케이션 시작
2. 엔드포인트 테스트 또는 CLI와 상호 작용
3. 각 프로젝트의 README.md에 명시된 문서와 예상 동작 확인

### Azure AI Foundry로 테스트하기
- 예제 실행 전에 `az login`으로 로그인 (키리스 인증)
- 계정에 해당 리소스의 Cognitive Services OpenAI 사용자 역할이 있는지 확인
- 5장 책임 있는 AI 예제로 콘텐츠 필터링 테스트

## Code Style Guidelines

### Java 관례
- **Java 버전:** 최신 기능을 갖춘 Java 21
- **스타일:** 표준 Java 관례 준수
- **명명 규칙:** 
  - 클래스: PascalCase
  - 메서드/변수: camelCase
  - 상수: UPPER_SNAKE_CASE
  - 패키지명: 소문자

### Spring Boot 패턴
- 비즈니스 로직에 `@Service` 사용
- REST 엔드포인트에 `@RestController` 사용
- `application.yml` 또는 `application.properties`로 구성
- 하드코딩 값 대신 환경 변수 사용 권장
- MCP 노출 메서드에는 `@Tool` 애노테이션 사용

### 파일 구성
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

### 종속성
- Maven `pom.xml`로 관리
- 버전 관리를 위한 Spring AI BOM 사용
- AI 통합을 위한 LangChain4j 사용
- Spring 종속성을 위한 Spring Boot 스타터 부모

### 코드 주석
- 공개 API에 JavaDoc 추가
- 복잡한 AI 상호작용에 대한 설명 주석 포함
- MCP 도구 설명 명확하게 문서화

## Build and Deployment

### 프로젝트 빌드

**테스트 없이 빌드:**
```bash
mvn clean install -DskipTests
```

**모든 검사와 함께 빌드:**
```bash
mvn clean install
```

**애플리케이션 패키징:**
```bash
mvn package
# target/ 디렉토리에 JAR 파일을 생성합니다
```

### 출력 디렉터리
- 컴파일된 클래스: `target/classes/`
- 테스트 클래스: `target/test-classes/`
- JAR 파일: `target/*.jar`
- Maven 아티팩트: `target/`

### 환경별 구성

**개발:**
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

**운영:**
- 키리스 인증 시 `az login` 대신 관리 ID 사용
- `AZURE_OPENAI_ENDPOINT`를 운영 Foundry 리소스에 맞게 설정
- 환경 변수 또는 Azure Key Vault를 통한 구성 관리

### 배포 고려사항
- 이 저장소는 교육용이며 샘플 애플리케이션 포함
- 즉시 운영 환경 배포용으로 설계되지 않음
- 샘플은 운영 환경용 적응 패턴 시연용
- 개별 프로젝트 README에서 특정 배포 참고사항 확인

## Additional Notes

### Azure AI Foundry
- **키리스 인증:** Microsoft Entra ID로 연결 – 관리할 API 키 없음
- **코드로 프로비저닝:** Bicep 및 azd (`azd up`)로 계정 및 모델 배포 생성
- 동일한 OpenAI 호환 코드를 로컬(`az login`) 및 Azure(관리 ID)에서 실행 가능

### 여러 프로젝트 작업
각 샘플 프로젝트는 독립적입니다:
```bash
# 특정 프로젝트로 이동
cd 04-PracticalSamples/[project-name]

# 각각은 자체 pom.xml을 가지고 있으며 독립적으로 빌드할 수 있습니다
mvn clean install
```

### 자주 발생하는 문제

**Java 버전 불일치:**
```bash
# 자바 21 확인
java -version
# 필요하면 JAVA_HOME 업데이트
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**종속성 다운로드 문제:**
```bash
# Maven 캐시를 지우고 다시 시도하세요
rm -rf ~/.m2/repository
mvn clean install
```

**엔드포인트 또는 로그인 실패:**
```bash
# 현재 세션에서 엔드포인트를 설정하고 로그인합니다 (키리스)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# 또는 프로젝트 디렉터리에 .env 파일을 사용하세요
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**포트가 이미 사용 중:**
```bash
# Spring Boot는 기본적으로 포트 8080을 사용합니다
# application.properties에서 변경:
server.port=8081
```

### 다국어 지원
- 45개 이상의 언어로 자동 번역된 문서 제공
- 번역은 `translations/` 디렉터리에 있음
- 번역은 GitHub Actions 워크플로우로 관리

### 학습 경로
1. [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)로 시작
2. 챕터 순서대로 진행 (01 → 05)
3. 각 챕터의 실습 예제 완료
4. 4장의 샘플 프로젝트 탐색
5. 5장에서 책임 있는 AI 실습 학습

### 개발 컨테이너
`.devcontainer/devcontainer.json` 구성:
- Java 21 개발 환경
- 사전 설치된 Maven
- VS Code Java 확장
- Spring Boot 도구
- GitHub Copilot 통합
- 도커 내 도커 지원
- Azure CLI 포함

### 성능 고려사항
- Azure AI Foundry 배포는 분당 토큰/요청 쿼터 제한 있음
- 임베딩에 적절한 배치 크기 사용
- 반복 API 호출 시 캐싱 고려
- 토큰 사용량 모니터링으로 비용 최적화

### 보안 참고사항
- `.env` 파일 커밋 금지 (.gitignore에 이미 포함)
- API 키보다 키리스 인증(Microsoft Entra ID) 권장
- Azure에서는 관리 ID 사용, 로컬 개발에는 `az login`
- 5장의 책임 있는 AI 가이드라인 준수

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**면책 조항**:
이 문서는 AI 번역 서비스 [Co-op Translator](https://github.com/Azure/co-op-translator)를 사용하여 번역되었습니다. 정확성을 기하기 위해 노력하고 있으나, 자동 번역은 오류나 부정확한 부분이 있을 수 있음을 유의하시기 바랍니다. 원본 문서의 원어본이 권위 있는 자료로 간주되어야 합니다. 중요한 정보의 경우, 전문가의 인간 번역을 권장합니다. 이 번역 사용으로 인해 발생하는 오해나 잘못된 해석에 대해 당사는 책임을 지지 않습니다.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->