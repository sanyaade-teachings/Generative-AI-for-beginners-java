# 초보자를 위한 MCP 계산기 튜토리얼

## 목차

- [학습할 내용](#학습할-내용)
- [필수 조건](#필수-조건)
- [프로젝트 구조 이해하기](#프로젝트-구조-이해하기)
- [핵심 구성 요소 설명](#핵심-구성-요소-설명)
  - [1. 메인 애플리케이션](#1-메인-애플리케이션)
  - [2. 계산기 서비스](#2-계산기-서비스)
  - [3. 직접 MCP 클라이언트](#3-직접-mcp-클라이언트)
  - [4. AI 기반 클라이언트](#4-ai-기반-클라이언트)
- [예제 실행하기](#예제-실행하기)
- [전체 작동 원리](#전체-작동-원리)
- [다음 단계](#다음-단계)

## 학습할 내용

이 튜토리얼은 모델 컨텍스트 프로토콜(Model Context Protocol, MCP)을 사용하여 계산기 서비스를 구축하는 방법을 설명합니다. 다음을 이해하게 됩니다:

- AI가 도구로 사용할 수 있는 서비스를 만드는 방법
- MCP 서비스와 직접 통신 설정하는 방법
- AI 모델이 자동으로 어떤 도구를 사용할지 선택하는 방법
- 직접 프로토콜 호출과 AI 지원 상호작용의 차이점

## 필수 조건

시작하기 전에 다음을 준비하세요:
- Java 21 이상 설치
- 의존성 관리를 위한 Maven
- Azure AI Foundry 모델 배포 ( `azd up`로 프로비저닝 — [2장](../../02-SetupDevEnvironment/getting-started-azure-openai.md) 참고)
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) 설치 및 `az login`으로 로그인 (키리스 인증)
- Java 및 Spring Boot 기본 이해

## 프로젝트 구조 이해하기

계산기 프로젝트에는 몇 가지 중요한 파일이 있습니다:

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

이 파일은 계산기 서비스의 진입점입니다. 표준 Spring Boot 애플리케이션이면서 특별한 추가 사항이 하나 있습니다:

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

**이것이 하는 일:**
- 포트 8080에서 Spring Boot 웹 서버 시작
- 계산기 메서드를 MCP 도구로 사용할 수 있게 만드는 `ToolCallbackProvider` 생성
- `@Bean` 어노테이션으로 이 컴포넌트를 스프링이 관리하도록 지정하여 다른 부분에서 사용할 수 있도록 함

### 2. 계산기 서비스

**파일:** `CalculatorService.java`

모든 수학 연산이 여기서 이루어집니다. 각 메서드는 `@Tool`로 표시되어 MCP를 통해 사용 가능합니다:

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
    
    // 더 많은 계산기 작업들...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**주요 기능:**

1. **`@Tool` 어노테이션**: MCP에 외부 클라이언트가 이 메서드를 호출할 수 있음을 알림
2. **명확한 설명**: 각 도구에는 AI 모델이 언제 사용할지 이해할 수 있는 설명이 포함됨
3. **일관된 반환 형식**: 모든 연산은 "5.00 + 3.00 = 8.00"과 같은 사람이 읽을 수 있는 문자열 반환
4. **오류 처리**: 0으로 나누기와 음수 제곱근은 오류 메시지를 반환

**사용 가능한 연산:**
- `add(a, b)` - 두 수를 더함
- `subtract(a, b)` - 두 번째 수를 첫 번째 수에서 뺌
- `multiply(a, b)` - 두 수를 곱함
- `divide(a, b)` - 첫 번째 수를 두 번째 수로 나눔 (0 체크 포함)
- `power(base, exponent)` - base의 exponent 제곱
- `squareRoot(number)` - 제곱근 계산 (음수 체크 포함)
- `modulus(a, b)` - 나머지 값 반환
- `absolute(number)` - 절대값 반환
- `help()` - 모든 연산에 대한 정보 반환

### 3. 직접 MCP 클라이언트

**파일:** `SDKClient.java`

이 클라이언트는 AI를 사용하지 않고 MCP 서버와 직접 통신합니다. 특정 계산기 기능을 수동으로 호출합니다:

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

**이것이 하는 일:**
1. 빌더 패턴을 사용하여 `http://localhost:8080`의 계산기 서버에 연결
2. 사용 가능한 모든 도구(계산기 함수) 목록 조회
3. 정확한 매개변수를 사용하여 특정 함수 호출
4. 결과를 바로 출력

**참고:** 이 예제는 `WebFluxSseClientTransport`를 위한 빌더 패턴을 도입한 Spring AI 1.1.0-SNAPSHOT 의존성을 사용합니다. 이전 안정 버전을 사용하는 경우 직접 생성자를 사용해야 할 수 있습니다.

**이용 시기:** 수행하려는 계산이 정확히 정해져 있고 프로그래밍 방식으로 호출하고 싶을 때.

### 4. AI 기반 클라이언트

**파일:** `LangChain4jClient.java`

이 클라이언트는 AI 모델(GPT-4o-mini)을 사용하여 어떤 계산기 도구를 사용할지 자동으로 결정합니다:

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

        // 우리의 계산기 MCP 서버에 연결
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // AI가 무엇을 하고 있는지 보여줌
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // AI에게 우리의 계산기 도구에 대한 접근 권한 부여
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // 우리의 계산기를 사용할 수 있는 AI 봇 생성
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // 이제 자연어로 AI에게 계산을 요청할 수 있음
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**이것이 하는 일:**
1. 키리스 인증(Microsoft Entra ID)을 사용하여 AI 모델 연결 생성
2. AI를 계산기 MCP 서버에 연결
3. AI에 계산기 도구 전체 접근 권한 부여
4. "24.5와 17.3의 합을 계산해줘" 같은 자연어 요청 허용

**AI는 자동으로:**
- 덧셈을 원하는 것을 이해
- `add` 도구 선택
- `add(24.5, 17.3)` 호출
- 자연스러운 응답으로 결과 반환

## 예제 실행하기

### 단계 1: 계산기 서버 시작

먼저 Azure AI Foundry 엔드포인트를 설정하세요 (AI 클라이언트용, 키리스 인증, API 키 불필요):

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

서버가 `http://localhost:8080`에서 시작됩니다. 다음과 같은 메시지가 표시됩니다:
```
Started McpServerApplication in X.XXX seconds
```

### 단계 2: 직접 클라이언트로 테스트

서버가 실행 중인 상태에서 <strong>새 터미널</strong>을 열고 직접 MCP 클라이언트를 실행하세요:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

다음과 같은 출력이 표시됩니다:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### 단계 3: AI 클라이언트로 테스트

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

AI가 도구를 자동으로 사용하는 모습을 확인할 수 있습니다:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### 단계 4: MCP 서버 종료

테스트를 마치면 AI 클라이언트 터미널에서 `Ctrl+C`를 눌러 종료할 수 있습니다. MCP 서버는 별도로 종료할 때까지 계속 실행됩니다.
서버를 종료하려면 실행 중인 터미널에서 `Ctrl+C`를 누르세요.

## 전체 작동 원리

AI에게 "5 + 3은 무엇인가?"라고 물었을 때의 전체 흐름:

1. <strong>사용자</strong>가 자연어로 AI에 질문
2. <strong>AI</strong>가 요청을 분석하여 덧셈임을 파악
3. <strong>AI</strong>가 MCP 서버에 `add(5.0, 3.0)` 호출
4. <strong>계산기 서비스</strong>가 `5.0 + 3.0 = 8.0` 계산
5. <strong>계산기 서비스</strong>가 `"5.00 + 3.00 = 8.00"` 결과 반환
6. <strong>AI</strong>가 결과를 받아 자연스러운 응답으로 포맷팅
7. <strong>사용자</strong>가 "5와 3의 합은 8입니다"라는 답변을 받음

## 다음 단계

더 많은 예제는 [4장: 실용 샘플](../README.md)에서 확인하세요.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**면책 조항**:
이 문서는 AI 번역 서비스 [Co-op Translator](https://github.com/Azure/co-op-translator)를 사용하여 번역되었습니다. 정확성을 기하기 위해 노력하고 있으나, 자동 번역은 오류나 부정확한 부분이 있을 수 있음을 유의하시기 바랍니다. 원본 문서의 원어본이 권위 있는 자료로 간주되어야 합니다. 중요한 정보의 경우, 전문가의 인간 번역을 권장합니다. 이 번역 사용으로 인해 발생하는 오해나 잘못된 해석에 대해 당사는 책임을 지지 않습니다.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->