# Tutorial do Calculador MCP para Iniciantes

## Índice

- [O Que Você Vai Aprender](#o-que-você-vai-aprender)
- [Pré-requisitos](#pré-requisitos)
- [Entendendo a Estrutura do Projeto](#entendendo-a-estrutura-do-projeto)
- [Componentes Principais Explicados](#componentes-principais-explicados)
  - [1. Aplicação Principal](#1-aplicação-principal)
  - [2. Serviço da Calculadora](#2-serviço-da-calculadora)
  - [3. Cliente MCP Direto](#3-cliente-mcp-direto)
  - [4. Cliente com IA](#4-cliente-com-ia)
- [Executando os Exemplos](#executando-os-exemplos)
- [Como Tudo Funciona Junto](#como-tudo-funciona-junto)
- [Próximos Passos](#próximos-passos)

## O Que Você Vai Aprender

Este tutorial explica como construir um serviço de calculadora usando o Protocolo de Contexto de Modelo (MCP). Você vai entender:

- Como criar um serviço que a IA pode usar como uma ferramenta
- Como configurar comunicação direta com serviços MCP
- Como modelos de IA podem escolher automaticamente quais ferramentas usar
- A diferença entre chamadas diretas de protocolo e interações assistidas por IA

## Pré-requisitos

Antes de começar, certifique-se de que você possui:
- Java 21 ou superior instalado
- Maven para gerenciamento de dependências
- Uma implantação de modelo Azure AI Foundry (provisionada com `azd up` — veja [Capítulo 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- O [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), logado com `az login` (autenticação sem chave)
- Conhecimento básico de Java e Spring Boot

## Entendendo a Estrutura do Projeto

O projeto da calculadora possui vários arquivos importantes:

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

**Arquivo:** `McpServerApplication.java`

Este é o ponto de entrada do nosso serviço de calculadora. É uma aplicação padrão Spring Boot com uma adição especial:

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

**O que isso faz:**
- Inicializa um servidor web Spring Boot na porta 8080
- Cria um `ToolCallbackProvider` que torna nossos métodos de calculadora disponíveis como ferramentas MCP
- A anotação `@Bean` diz ao Spring para gerenciar este componente para que outras partes possam utilizá-lo

### 2. Serviço da Calculadora

**Arquivo:** `CalculatorService.java`

Aqui é onde toda a matemática acontece. Cada método é marcado com `@Tool` para torná-lo disponível via MCP:

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
    
    // Mais operações de calculadora...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Características principais:**

1. **Anotação `@Tool`**: Informa ao MCP que este método pode ser chamado por clientes externos
2. **Descrições Claras**: Cada ferramenta tem uma descrição que ajuda os modelos de IA a entender quando usá-la
3. **Formato Consistente de Retorno**: Todas as operações retornam strings legíveis, como "5.00 + 3.00 = 8.00"
4. **Tratamento de Erros**: Divisão por zero e raízes quadradas negativas retornam mensagens de erro

**Operações Disponíveis:**
- `add(a, b)` - Soma dois números
- `subtract(a, b)` - Subtrai o segundo do primeiro
- `multiply(a, b)` - Multiplica dois números
- `divide(a, b)` - Divide o primeiro pelo segundo (verificação de zero)
- `power(base, exponent)` - Eleva a base à potência do expoente
- `squareRoot(number)` - Calcula a raiz quadrada (verificação de número negativo)
- `modulus(a, b)` - Retorna o resto da divisão
- `absolute(number)` - Retorna o valor absoluto
- `help()` - Retorna informações sobre todas as operações

### 3. Cliente MCP Direto

**Arquivo:** `SDKClient.java`

Este cliente comunica-se diretamente com o servidor MCP sem usar IA. Ele chama manualmente funções específicas da calculadora:

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

**O que isso faz:**
1. **Conecta** ao servidor da calculadora em `http://localhost:8080` usando o padrão builder
2. **Lista** todas as ferramentas disponíveis (nossas funções da calculadora)
3. **Chama** funções específicas com parâmetros exatos
4. **Imprime** os resultados diretamente

**Nota:** Este exemplo usa a dependência Spring AI 1.1.0-SNAPSHOT, que introduziu um padrão builder para o `WebFluxSseClientTransport`. Se você estiver usando uma versão estável mais antiga, pode precisar usar o construtor direto em seu lugar.

**Quando usar:** Quando você sabe exatamente qual cálculo deseja executar e quer chamá-lo programaticamente.

### 4. Cliente com IA

**Arquivo:** `LangChain4jClient.java`

Este cliente usa um modelo de IA (GPT-4o-mini) que pode decidir automaticamente quais ferramentas da calculadora usar:

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

        // Conectar ao nosso servidor MCP da calculadora
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Mostra o que a IA está fazendo
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Dar à IA acesso às nossas ferramentas de calculadora
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Criar um bot de IA que pode usar nossa calculadora
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Agora podemos pedir para a IA fazer cálculos em linguagem natural
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**O que isso faz:**
1. **Cria** uma conexão com o modelo de IA usando seu token do GitHub
2. **Conecta** a IA ao nosso servidor MCP da calculadora
3. **Concede** à IA acesso a todas as ferramentas da calculadora
4. **Permite** pedidos em linguagem natural, como "Calcule a soma de 24.5 e 17.3"

**A IA automaticamente:**
- Entende que você quer somar números
- Escolhe a ferramenta `add`
- Chama `add(24.5, 17.3)`
- Retorna o resultado em uma resposta natural

## Executando os Exemplos

### Passo 1: Iniciar o Servidor da Calculadora

Primeiro, faça login e configure seu endpoint Azure AI Foundry (necessário para o cliente IA — autenticação sem chave, sem chave API):

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

O servidor iniciará em `http://localhost:8080`. Você deverá ver:
```
Started McpServerApplication in X.XXX seconds
```

### Passo 2: Testar com Cliente Direto

Em um terminal **NOVO** com o servidor ainda rodando, execute o cliente MCP direto:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Você verá uma saída como:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Passo 3: Testar com Cliente IA

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Você verá a IA usando ferramentas automaticamente:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Passo 4: Fechar o Servidor MCP

Quando terminar os testes, você pode parar o cliente IA pressionando `Ctrl+C` no terminal dele. O servidor MCP continuará rodando até que você pare.
Para parar o servidor, pressione `Ctrl+C` no terminal onde ele está rodando.

## Como Tudo Funciona Junto

Aqui está o fluxo completo quando você pergunta à IA "Quanto é 5 + 3?":

1. **Você** faz a pergunta à IA em linguagem natural
2. **A IA** analisa seu pedido e percebe que você quer fazer uma soma
3. **A IA** chama o servidor MCP: `add(5.0, 3.0)`
4. **O Serviço da Calculadora** realiza: `5.0 + 3.0 = 8.0`
5. **O Serviço da Calculadora** retorna: `"5.00 + 3.00 = 8.00"`
6. **A IA** recebe o resultado e formata uma resposta natural
7. **Você** recebe: "A soma de 5 e 3 é 8"

## Próximos Passos

Para mais exemplos, veja [Capítulo 04: Exemplos práticos](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Aviso Legal**:
Este documento foi traduzido usando o serviço de tradução por IA [Co-op Translator](https://github.com/Azure/co-op-translator). Embora nos esforcemos pela precisão, por favor, esteja ciente de que traduções automatizadas podem conter erros ou imprecisões. O documento original em seu idioma nativo deve ser considerado a fonte autorizada. Para informações críticas, recomenda-se tradução profissional humana. Não nos responsabilizamos por quaisquer mal-entendidos ou interpretações incorretas decorrentes do uso desta tradução.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->