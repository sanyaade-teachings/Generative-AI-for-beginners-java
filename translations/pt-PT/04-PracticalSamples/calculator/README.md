# Tutorial do Calculador MCP para Iniciantes

## Índice

- [O que Vai Aprender](#o-que-vai-aprender)
- [Pré-requisitos](#pré-requisitos)
- [Compreender a Estrutura do Projeto](#compreender-a-estrutura-do-projeto)
- [Componentes Principais Explicados](#componentes-principais-explicados)
  - [1. Aplicação Principal](#1-aplicação-principal)
  - [2. Serviço de Calculadora](#2-serviço-de-calculadora)
  - [3. Cliente MCP Direto](#3-cliente-mcp-direto)
  - [4. Cliente Alimentado por IA](#4-cliente-alimentado-por-ia)
- [Executar os Exemplos](#executar-os-exemplos)
- [Como Tudo Funciona em Conjunto](#como-tudo-funciona-em-conjunto)
- [Próximos Passos](#próximos-passos)

## O que Vai Aprender

Este tutorial explica como construir um serviço de calculadora utilizando o Protocolo de Contexto de Modelo (MCP). Vai compreender:

- Como criar um serviço que a IA pode usar como ferramenta
- Como configurar comunicação direta com serviços MCP
- Como os modelos de IA podem escolher automaticamente quais ferramentas usar
- A diferença entre chamadas de protocolo diretas e interações assistidas por IA

## Pré-requisitos

Antes de começar, certifique-se de que tem:
- Java 21 ou superior instalado
- Maven para gestão de dependências
- Uma implantação de modelo Azure AI Foundry (forneça-a com `azd up` — veja o [Capítulo 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- O [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), autenticado com `az login` (autenticação sem chave)
- Conhecimentos básicos de Java e Spring Boot

## Compreender a Estrutura do Projeto

O projeto da calculadora tem vários ficheiros importantes:

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

## Componentes Principais Explicados

### 1. Aplicação Principal

**Ficheiro:** `McpServerApplication.java`

Este é o ponto de entrada do nosso serviço de calculadora. É uma aplicação Spring Boot padrão com uma adição especial:

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

**O que isto faz:**
- Inicia um servidor web Spring Boot na porta 8080
- Cria um `ToolCallbackProvider` que torna os métodos da nossa calculadora disponíveis como ferramentas MCP
- A anotação `@Bean` indica ao Spring para gerir isto como um componente que outras partes podem usar

### 2. Serviço de Calculadora

**Ficheiro:** `CalculatorService.java`

Aqui é onde toda a matemática acontece. Cada método está marcado com `@Tool` para estar disponível via MCP:

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
    
    // Mais operações da calculadora...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Características principais:**

1. **Anotação `@Tool`**: Indica ao MCP que este método pode ser chamado por clientes externos
2. **Descrições Claras**: Cada ferramenta tem uma descrição que ajuda os modelos de IA a perceber quando usá-la
3. **Formato Consistente de Retorno**: Todas as operações retornam strings legíveis como "5.00 + 3.00 = 8.00"
4. **Gestão de Erros**: Divisão por zero e raízes quadradas negativas retornam mensagens de erro

**Operações Disponíveis:**
- `add(a, b)` - Soma dois números
- `subtract(a, b)` - Subtrai o segundo do primeiro
- `multiply(a, b)` - Multiplica dois números
- `divide(a, b)` - Divide o primeiro pelo segundo (com verificação de zero)
- `power(base, exponent)` - Eleva a base à potência do expoente
- `squareRoot(number)` - Calcula a raiz quadrada (com verificação de negativo)
- `modulus(a, b)` - Retorna o resto da divisão
- `absolute(number)` - Retorna o valor absoluto
- `help()` - Retorna informações sobre todas as operações

### 3. Cliente MCP Direto

**Ficheiro:** `SDKClient.java`

Este cliente comunica diretamente com o servidor MCP sem usar IA. Chama manualmente funções específicas da calculadora:

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
        
        // Listar ferramentas disponíveis
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Chamar funções específicas da calculadora
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

**O que isto faz:**
1. **Liga-se** ao servidor da calculadora em `http://localhost:8080` usando o padrão builder
2. **Lista** todas as ferramentas disponíveis (as funções da calculadora)
3. **Chama** funções específicas com parâmetros exatos
4. **Imprime** os resultados diretamente

**Nota:** Este exemplo usa a dependência Spring AI 1.1.0-SNAPSHOT, que introduziu um padrão builder para `WebFluxSseClientTransport`. Se usar uma versão estável mais antiga, pode ser necessário usar o construtor direto.

**Quando usar isto:** Quando souber exatamente que cálculo quer realizar e pretende chamá-lo programaticamente.

### 4. Cliente Alimentado por IA

**Ficheiro:** `LangChain4jClient.java`

Este cliente usa um modelo IA (GPT-4o-mini) que pode decidir automaticamente quais as ferramentas da calculadora a usar:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Configurar o modelo de IA (Azure AI Foundry, autenticação sem chave via Microsoft Entra ID)
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

        // Ligar ao nosso servidor MCP da calculadora
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Mostra o que a IA está a fazer
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Dar à IA acesso às nossas ferramentas de calculadora
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Criar um bot de IA que possa usar a nossa calculadora
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Agora podemos pedir à IA para fazer cálculos em linguagem natural
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**O que isto faz:**
1. **Cria** uma ligação ao modelo IA usando autenticação sem chave (Microsoft Entra ID)
2. **Liga** a IA ao nosso servidor MCP da calculadora
3. **Concede** à IA acesso a todas as ferramentas da calculadora
4. **Permite** pedidos em linguagem natural como "Calcular a soma de 24.5 e 17.3"

**A IA automaticamente:**
- Percebe que quer adicionar números
- Escolhe a ferramenta `add`
- Chama `add(24.5, 17.3)`
- Retorna o resultado numa resposta natural

## Executar os Exemplos

### Passo 1: Iniciar o Servidor da Calculadora

Primeiro, faça login e configure o endpoint Azure AI Foundry (necessário para o cliente IA — autenticação sem chave, sem chave API):

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

Inicie o servidor:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

O servidor iniciará em `http://localhost:8080`. Deve ver:
```
Started McpServerApplication in X.XXX seconds
```

### Passo 2: Testar com Cliente Direto

Num terminal **NOVO** enquanto o servidor ainda está a correr, execute o cliente MCP direto:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Verá uma saída como:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Passo 3: Testar com Cliente IA

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Verá a IA a usar as ferramentas automaticamente:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Passo 4: Fechar o Servidor MCP

Quando terminar os testes, pode fechar o cliente IA pressionando `Ctrl+C` no terminal dele. O servidor MCP continuará a correr até o parar.
Para parar o servidor, pressione `Ctrl+C` no terminal onde está a correr.

## Como Tudo Funciona em Conjunto

Eis o fluxo completo quando pergunta à IA "Quanto é 5 + 3?":

1. **Você** pergunta à IA em linguagem natural
2. **A IA** analisa o pedido e percebe que quer adição
3. **A IA** chama o servidor MCP: `add(5.0, 3.0)`
4. **O Serviço da Calculadora** calcula: `5.0 + 3.0 = 8.0`
5. **O Serviço da Calculadora** devolve: `"5.00 + 3.00 = 8.00"`
6. **A IA** recebe o resultado e formata uma resposta natural
7. **Você** recebe: "A soma de 5 e 3 é 8"

## Próximos Passos

Para mais exemplos, veja o [Capítulo 04: Exemplos práticos](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Aviso Legal**:
Este documento foi traduzido utilizando o serviço de tradução automática [Co-op Translator](https://github.com/Azure/co-op-translator). Embora nos esforcemos pela precisão, esteja ciente de que traduções automáticas podem conter erros ou imprecisões. O documento original na sua língua nativa deve ser considerado a fonte autorizada. Para informações críticas, recomenda-se tradução profissional humana. Não nos responsabilizamos por quaisquer mal-entendidos ou interpretações incorretas resultantes da utilização desta tradução.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->