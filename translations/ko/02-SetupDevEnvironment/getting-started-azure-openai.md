# Azure AI Foundry 개발 환경 설정

> 이 가이드는 이 과정의 Java AI 앱용 **Azure AI Foundry** 모델을 **키 없는** 인증(Microsoft Entra ID)을 사용해 설정합니다 — 관리할 API 키가 필요 없습니다. 도구가 처음인가요? [개발 환경 가이드](./README.md)부터 시작하세요.

이 가이드는 이 과정의 Java AI 앱용 **Azure AI Foundry** 모델을 설정합니다. 두 가지 경로가 있습니다:

- **옵션 A — `azd` + Bicep으로 프로비저닝(권장):** 한 번의 명령으로 Foundry 계정과 모델을 코드로 배포합니다. 포털 클릭 불필요.
- **옵션 B — Azure AI Foundry 포털에서 리소스를 수동으로 생성하기**

두 경로 모두 **키 없는 인증**(Microsoft Entra ID)을 사용하므로 복사하거나 노출할 API 키가 없습니다.

## 목차

- [생성되는 항목](#생성되는-항목)
- [필수 조건](#필수-조건)
- [옵션 A: azd + Bicep으로 프로비저닝(권장)](#option-a-provision-with-azd--bicep-recommended)
- [옵션 B: 리소스 수동 생성](#옵션-b-리소스-수동-생성)
- [환경 구성](#환경-구성)
- [설정 테스트](#설정-테스트)
- [다음 단계](#다음-단계)
- [리소스](#리소스)
- [추가 리소스](#추가-리소스)

## 생성되는 항목

[`infra/`](../../../02-SetupDevEnvironment/infra)의 Bicep 템플릿은 다음을 프로비저닝합니다:

- 프로젝트가 포함된 **Azure AI Foundry** 계정(`Microsoft.CognitiveServices/accounts`, kind `AIServices`)
- <strong>채팅</strong> 배포 — `gpt-4o-mini`
- <strong>임베딩</strong> 배포 — `text-embedding-3-small` (나중 챕터에서 사용)
- 키를 관리하지 않고 `az login`으로 로그인할 수 있는 **키 없는 역할 할당**(`Cognitive Services OpenAI User`)

## 필수 조건

- [Azure 구독](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) 및 [Maven 3.9+](https://maven.apache.org/download.cgi)

## 옵션 A: azd + Bicep으로 프로비저닝(권장)

`02-SetupDevEnvironment` 폴더에서:

```bash
cd 02-SetupDevEnvironment

# 로그인 (두 도구 모두)
azd auth login
az login

# Foundry 계정 및 모델 배포 준비
azd up
```
  
`azd`는 **환경 이름**(예: `genai-java`)과 <strong>지역</strong>을 묻습니다. `gpt-4o-mini`와 `text-embedding-3-small`이 사용 가능한 지역을 선택하세요 — 예를 들어 `eastus2` 또는 `swedencentral`.

프로비저닝이 끝나면 azd는:

1. [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep)에 정의된 모든 항목을 배포합니다.
2. 포스트 프로비저닝 훅을 실행해 [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure)에 엔드포인트와 배포 이름을 기록합니다(비밀 정보 없음).

> **팁:** 변경 사항을 적용하려면 언제든지 `azd up`을 재실행하세요. 모든 리소스를 삭제하고 비용 발생을 중단하려면 `azd down`을 실행하세요.

생성된 설정을 보려면:

```bash
azd env get-values
```
  
이제 [설정 테스트](#설정-테스트)로 건너뛰세요.

## 옵션 B: 리소스 수동 생성

포털을 선호하시나요? 수동으로 리소스를 생성하세요:

1. [Azure AI Foundry 포털](https://ai.azure.com/)에 로그인하세요.
2. **프로젝트 생성**(이 과정에서 AI Foundry 리소스도 생성됩니다). 이름은 `GenAIJava` 같은 이름을 붙이세요.
3. 프로젝트에서 **Models + endpoints** → **Deploy model** → <strong>Deploy base model</strong>을 엽니다.
4. **gpt-4o-mini**(배포 이름 `gpt-4o-mini`)를 배포합니다. 임베딩 예제를 원하면 <strong>text-embedding-3-small</strong>도 반복하여 배포하세요.
5. <strong>개요(Overview)</strong>에서 <strong>엔드포인트</strong>(예: `https://<resource>.openai.azure.com/`)를 복사합니다.
6. 키 없는 액세스 권한 부여: 리소스의 **접근 제어(IAM)** → **역할 할당 추가** → 계정에 **Cognitive Services OpenAI User** 역할을 할당합니다.

> **문제가 계속되나요?** [Azure AI Foundry 문서](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects)를 참고하세요.

## 환경 구성

**옵션 A (`azd up`)를 사용했다면**, 설정 파일이 이미 작성되어 있어 구성할 필요가 없습니다. [설정 테스트](#설정-테스트)로 건너뛰세요.

**옵션 B (수동)를 사용했다면**, 예제 `.env` 파일을 직접 만드세요:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```
  
`.env`를 편집해 엔드포인트를 넣으세요(키는 없습니다 — 인증은 키 없음):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```
  
> **보안 참고:** 저장할 API 키가 없습니다. `az login`(로컬) 또는 관리 ID(Azure 내)를 통해 Microsoft Entra ID로 인증합니다. `.env` 파일은 비밀이 아닌 설정만 포함하며 이미 `.gitignore`에 포함되어 있습니다.

## 설정 테스트

키 없는 인증이 토큰을 얻을 수 있도록 로그인되어 있는지 확인한 후 예제를 실행하세요:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # 아직 로그인하지 않은 경우
mvn clean spring-boot:run
```
  
`gpt-4o-mini` 모델의 응답을 볼 수 있을 것입니다!

> **VS Code 사용자:** `F5`를 눌러 실행하세요. 앱이 `.env`를 자동으로 로드합니다.

> **전체 예제:** 자세한 내용과 문제 해결은 [Azure AI Foundry와 기본 채팅 예제](./examples/basic-chat-azure/README.md)를 참고하세요.

## 다음 단계

**설정 완료!** 이제 다음이 준비되었습니다:  
- `gpt-4o-mini`와 `text-embedding-3-small`이 배포된 Azure AI Foundry  
- 관리할 키가 없는 인증(Microsoft Entra ID)  
- 엔드포인트와 배포 이름이 포함된 로컬 `.env`  
- Java 개발 환경

**AI 앱 개발을 시작하려면** [3장: 핵심 생성 AI 기법](../03-CoreGenerativeAITechniques/README.md)으로 계속하세요!

## 리소스

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Microsoft Entra ID로 키 없는 인증](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry 문서](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI 문서](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## 추가 리소스

- [VS Code 다운로드](https://code.visualstudio.com/Download)
- [Docker Desktop 받기](https://www.docker.com/products/docker-desktop)
- [Dev Container 구성](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**면책 조항**:
이 문서는 AI 번역 서비스 [Co-op Translator](https://github.com/Azure/co-op-translator)를 사용하여 번역되었습니다. 정확성을 기하기 위해 노력하고 있으나, 자동 번역은 오류나 부정확한 부분이 있을 수 있음을 유의하시기 바랍니다. 원본 문서의 원어본이 권위 있는 자료로 간주되어야 합니다. 중요한 정보의 경우, 전문가의 인간 번역을 권장합니다. 이 번역 사용으로 인해 발생하는 오해나 잘못된 해석에 대해 당사는 책임을 지지 않습니다.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->