# 초보자를 위한 MCP 계산기 튜토리얼

## 목차

- [배울 내용](#배울-내용)
- [사전 준비 사항](#사전-준비-사항)
- [프로젝트 구조 이해하기](#프로젝트-구조-이해하기)
- [핵심 구성 요소 설명](#핵심-구성-요소-설명)
  - [1. 메인 애플리케이션](#1-메인-애플리케이션)
  - [2. 계산기 서비스](#2-계산기-서비스)
  - [3. 직접 MCP 클라이언트](#3-직접-mcp-클라이언트)
  - [4. AI 기반 클라이언트](#4-ai-기반-클라이언트)
- [예제 실행하기](#예제-실행하기)
- [전체 작동 방식](#전체-작동-방식)
- [다음 단계](#다음-단계)

## 배울 내용

이 튜토리얼은 Model Context Protocol (MCP)을 사용하여 계산기 서비스를 구축하는 방법을 설명합니다. 다음을 이해하게 됩니다:

- AI가 도구로 사용할 수 있는 서비스 만드는 법
- MCP 서비스와 직접 통신 설정 방법
- AI 모델이 자동으로 사용할 도구를 선택하는 방법
- 직접 프로토콜 호출과 AI 지원 상호작용의 차이점

## 사전 준비 사항

시작하기 전에 다음을 확인하세요:
- Java 21 이상 설치
- 의존성 관리를 위한 Maven
- Azure AI Foundry 모델 배포 ( `azd up` 명령으로 프로비저닝 — [2장](../../02-SetupDevEnvironment/getting-started-azure-openai.md) 참조)
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), `az login`으로 로그인 (키 없는 인증)
- Java 및 Spring Boot 기본 이해

## 프로젝트 구조 이해하기

계산기 프로젝트에는 다음과 같은 중요한 파일들이 있습니다:

```
calculator/
├── src/main/java/com/microsoft/mcp/sample/server/
│   ├── McpServerApplication.java          # Main Spring Boot app
│   └── service/CalculatorService.java     # Calculator operations
└── src/test/java/com/microsoft/mcp/sample/client/
    ├── SDKClient.java                     # Direct MCP communication
    ├── LangChain4jClient.java            # AI-powered client
    └── Bot.java                          # Simple chat interface
```

## 핵심 구성 요소 설명

### 1. 메인 애플리케이션

**파일:** `McpServerApplication.java`

이 파일은 계산기 서비스의 진입점입니다. 기본적인 Spring Boot 애플리케이션이며, 특별히 다음 기능이 추가되어 있습니다:

```java
@SpringBootApplication
public class McpServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(McpServerApplication.class, args);
    }
    
    @Bean
    public ToolCallbackProvider calculatorTools(CalculatorService calculator) {
        return MethodToolCallbackProvider.builder().toolObjects(calculator).build();
    }
}
```

**하는 일:**
- 8080 포트에서 Spring Boot 웹 서버 시작
- 계산기 메서드를 MCP 도구로 사용할 수 있도록 `ToolCallbackProvider` 생성
- `@Bean` 어노테이션으로 Spring이 이 컴포넌트를 관리하고 다른 부분에서 사용 가능하게 함

### 2. 계산기 서비스

**파일:** `CalculatorService.java`

모든 수학 연산이 이곳에서 이뤄집니다. 각 메서드는 MCP를 통해 사용 가능하도록 `@Tool`로 표시되어 있습니다:

```java
@Service
public class CalculatorService {

    @Tool(description = "Add two numbers together")
    public String add(double a, double b) {
        double result = a + b;
        return formatResult(a, "+", b, result);
    }

    @Tool(description = "Subtract the second number from the first number")
    public String subtract(double a, double b) {
        double result = a - b;
        return formatResult(a, "-", b, result);
    }
    
    // 더 많은 계산기 연산...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**주요 특징:**

1. **`@Tool` 어노테이션**: 이 메서드가 외부 클라이언트가 호출할 수 있음을 MCP에 알림
2. **명확한 설명**: 각 도구는 AI 모델이 언제 사용해야 할지 이해할 수 있도록 설명이 있음
3. **일관된 반환 형식**: 모든 연산은 "5.00 + 3.00 = 8.00" 같은 읽기 쉬운 문자열을 반환함
4. **오류 처리**: 0으로 나누거나 음수 제곱근일 때 에러 메시지 반환

**사용 가능한 연산:**
- `add(a, b)` - 두 수 더하기
- `subtract(a, b)` - 두 번째 수를 첫 번째에서 빼기
- `multiply(a, b)` - 두 수 곱하기
- `divide(a, b)` - 첫 번째 수를 두 번째 수로 나누기 (0 체크 포함)
- `power(base, exponent)` - base를 exponent 제곱으로 올리기
- `squareRoot(number)` - 제곱근 계산 (음수 체크 포함)
- `modulus(a, b)` - 나머지 반환
- `absolute(number)` - 절대값 반환
- `help()` - 모든 연산 정보 반환

### 3. 직접 MCP 클라이언트

**파일:** `SDKClient.java`

이 클라이언트는 AI를 사용하지 않고 MCP 서버와 직접 통신합니다. 특정 계산기 함수들을 수동으로 호출합니다:

```java
public class SDKClient {
    
    public static void main(String[] args) {
        McpClientTransport transport = WebFluxSseClientTransport.builder(
            WebClient.builder().baseUrl("http://localhost:8080")
        ).build();
        new SDKClient(transport).run();
    }
    
    public void run() {
        var client = McpClient.sync(this.transport).build();
        client.initialize();
        
        // 사용 가능한 도구 목록
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // 특정 계산기 함수 호출
        CallToolResult resultAdd = client.callTool(
            new CallToolRequest("add", Map.of("a", 5.0, "b", 3.0))
        );
        System.out.println("Add Result = " + resultAdd);
        
        CallToolResult resultSqrt = client.callTool(
            new CallToolRequest("squareRoot", Map.of("number", 16.0))
        );
        System.out.println("Square Root Result = " + resultSqrt);
        
        client.closeGracefully();
    }
}
```

**하는 일:**
1. 빌더 패턴으로 `http://localhost:8080`에 실행 중인 계산기 서버에 연결
2. 사용 가능한 모든 도구(계산기 함수) 목록 조회
3. 정확한 매개변수로 특정 함수 호출
4. 결과를 바로 출력

**참고:** 이 예제는 Spring AI 1.1.0-SNAPSHOT 의존성을 사용하며, `WebFluxSseClientTransport`에 빌더 패턴이 도입되었습니다. 이전 안정 버전을 사용한다면 직접 생성자 호출이 필요할 수 있습니다.

**언제 사용하는가:** 특정 계산을 프로그래밍 방식으로 정확히 수행하고자 할 때.

### 4. AI 기반 클라이언트

**파일:** `LangChain4jClient.java`

이 클라이언트는 자동으로 어떤 계산기 도구를 사용할지 결정하는 AI 모델(GPT-4o-mini)을 사용합니다:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // AI 모델 설정 (Azure AI Foundry, Microsoft Entra ID를 통한 키리스 인증)
        String endpoint = System.getenv("AZURE_OPENAI_ENDPOINT");
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1";
        String token = new DefaultAzureCredentialBuilder().build()
                .getToken(new TokenRequestContext().addScopes("https://ai.azure.com/.default"))
                .block().getToken();
        ChatLanguageModel model = OpenAiOfficialChatModel.builder()
                .baseUrl(baseUrl)
                .apiKey(token)
                .modelName("gpt-4o-mini")
                .build();

        // 계산기 MCP 서버에 연결
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // AI가 수행 중인 작업 표시
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // AI에게 계산기 도구 접근 권한 부여
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // 계산기를 사용할 수 있는 AI 봇 생성
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // 이제 AI에게 자연어로 계산을 요청할 수 있음
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**하는 일:**
1. GitHub 토큰으로 AI 모델 연결 생성
2. AI를 MCP 서버(계산기)와 연결
3. AI가 모든 계산기 도구에 접근 가능하도록 설정
4. "24.5와 17.3의 합을 계산해줘" 같은 자연어 요청 허용

**AI가 자동으로:**
- 더하기를 해야 한다고 이해
- `add` 도구 선택
- `add(24.5, 17.3)` 호출
- 자연스러운 응답 형태로 결과 반환

## 예제 실행하기

### 1단계: 계산기 서버 시작

먼저 로그인하고 Azure AI Foundry 엔드포인트를 설정하세요 (AI 클라이언트에 필요 — 키 없는 인증, API 키 없음):

**Windows:**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**Linux/macOS:**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

서버 시작:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

서버가 `http://localhost:8080`에서 시작됩니다. 다음 메시지가 표시됩니다:
```
Started McpServerApplication in X.XXX seconds
```

### 2단계: 직접 클라이언트 테스트

서버가 실행 중인 상태에서 <strong>새 터미널</strong>을 열고 직접 MCP 클라이언트를 실행하세요:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

다음과 같은 출력이 나옵니다:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### 3단계: AI 클라이언트 테스트

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

AI가 자동으로 도구를 사용하는 모습을 볼 수 있습니다:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### 4단계: MCP 서버 종료

테스트가 끝나면 AI 클라이언트 터미널에서 `Ctrl+C`를 눌러 종료하세요. MCP 서버는 종료 명령까지 계속 실행됩니다.
서버를 종료하려면 실행 중인 터미널에서 `Ctrl+C`를 누르세요.

## 전체 작동 방식

AI에게 "5 + 3은 얼마야?"라고 물었을 때 전체 흐름은 다음과 같습니다:

1. <strong>사용자</strong>가 자연어로 AI에 질문
2. <strong>AI</strong>가 요청을 분석하고 덧셈을 원함을 인식
3. <strong>AI</strong>가 MCP 서버에 `add(5.0, 3.0)` 호출
4. <strong>계산기 서비스</strong>가 `5.0 + 3.0 = 8.0` 계산
5. <strong>계산기 서비스</strong>가 `"5.00 + 3.00 = 8.00"` 반환
6. <strong>AI</strong>가 결과를 받아 자연스러운 응답으로 포맷
7. <strong>사용자</strong>가 "5와 3의 합은 8입니다"라는 답변 받음

## 다음 단계

추가 예제는 [4장: 실용 예제](../README.md)에서 확인하세요

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**면책 조항**:
이 문서는 AI 번역 서비스 [Co-op Translator](https://github.com/Azure/co-op-translator)를 사용하여 번역되었습니다. 정확성을 기하기 위해 노력하고 있으나, 자동 번역은 오류나 부정확한 부분이 있을 수 있음을 유의하시기 바랍니다. 원본 문서의 원어본이 권위 있는 자료로 간주되어야 합니다. 중요한 정보의 경우, 전문가의 인간 번역을 권장합니다. 이 번역 사용으로 인해 발생하는 오해나 잘못된 해석에 대해 당사는 책임을 지지 않습니다.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->