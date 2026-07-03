# Tutorial do Gerador de Histórias de Animais de Estimação para Iniciantes

## Índice

- [Pré-requisitos](#pré-requisitos)
- [Entendendo a Estrutura do Projeto](#entendendo-a-estrutura-do-projeto)
- [Componentes Principais Explicados](#componentes-principais-explicados)
  - [1. Aplicação Principal](#1-aplicação-principal)
  - [2. Controlador Web](#2-controlador-web)
  - [3. Serviço de Histórias](#3-serviço-de-histórias)
  - [4. Templates Web](#4-templates-web)
  - [5. Configuração](#5-configuração)
- [Executando a Aplicação](#executando-a-aplicação)
- [Como Tudo Funciona Junto](#como-tudo-funciona-junto)
- [Entendendo a Integração com IA](#entendendo-a-integração-com-ia)
- [Próximos Passos](#próximos-passos)

## Pré-requisitos

Antes de começar, certifique-se de que você tem:
- Java 21 ou superior instalado
- Maven para gerenciamento de dependências
- Um deployment de modelo Azure AI Foundry (provisionado com `azd up` — veja [Capítulo 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), logado com `az login` (autenticação sem chave)
- Conhecimento básico de Java, Spring Boot e desenvolvimento web

## Entendendo a Estrutura do Projeto

O projeto de história de pet possui vários arquivos importantes:

```
petstory/
├── src/main/java/com/example/petstory/
│   ├── PetStoryApplication.java       # Main Spring Boot application
│   ├── PetController.java             # Web request handler
│   ├── StoryService.java              # AI story generation service
│   └── SecurityConfig.java            # Security configuration
├── src/main/resources/
│   ├── application.properties         # App configuration
│   └── templates/
│       ├── index.html                 # Upload form page
│       └── result.html               # Story display page
└── pom.xml                           # Maven dependencies
```

## Componentes Principais Explicados

### 1. Aplicação Principal

**Arquivo:** `PetStoryApplication.java`

Este é o ponto de entrada da nossa aplicação Spring Boot:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**O que isso faz:**
- A anotação `@SpringBootApplication` ativa a auto-configuração e escaneamento de componentes
- Inicia um servidor web embutido (Tomcat) na porta 8080
- Cria automaticamente todos os beans e serviços Spring necessários

### 2. Controlador Web

**Arquivo:** `PetController.java`

Ele gerencia todas as requisições web e interações do usuário:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Retorna o template index.html
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Validação de entrada
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Sanitizar entrada para segurança
        String sanitizedDescription = sanitizeInput(description);
        
        // Gerar história com tratamento de erro
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Retorna o template result.html
            
        } catch (Exception e) {
            // Usar história alternativa se a IA falhar
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Limitar o tamanho
    }
}
```

**Principais funcionalidades:**

1. **Gerenciamento de Rotas**: `@GetMapping("/")` exibe o formulário de upload, `@PostMapping("/generate-story")` processa os envios
2. **Validação de Entrada**: Verifica descrições vazias e limites de tamanho
3. **Segurança**: Sanitiza a entrada do usuário para prevenir ataques XSS
4. **Tratamento de Erros**: Fornece histórias alternativas quando o serviço de IA falha
5. **Binding de Modelo**: Passa dados para templates HTML usando o `Model` do Spring

**Sistema de Contingência:**
O controlador inclui templates de histórias pré-escritas que são usados quando o serviço de IA não está disponível:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Use hash de descrição para respostas consistentes
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Serviço de Histórias

**Arquivo:** `StoryService.java`

Este serviço se comunica com o Azure AI Foundry para gerar histórias usando autenticação sem chave:

```java
@Service
public class StoryService {
    
    private final OpenAIClient openAIClient;
    private final String modelName;
    
    public StoryService(@Value("${azure.openai.endpoint:}") String endpoint,
                       @Value("${azure.openai.deployment:gpt-4o-mini}") String modelName) {
        this.modelName = modelName;
        if (endpoint == null || endpoint.isBlank()) {
            endpoint = System.getenv("AZURE_OPENAI_ENDPOINT");
        }
        
        // O endpoint compatível com OpenAI do Foundry está localizado em /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Autenticação sem chave com Microsoft Entra ID (sem chave de API)
        DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
        this.openAIClient = OpenAIOkHttpClient.builder()
                .baseUrl(baseUrl)
                .credential(BearerTokenCredential.create(
                        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
                .build();
    }
    
    public String generateStory(String description) {
        String systemPrompt = "You are a creative storyteller who writes fun, " +
                             "family-friendly short stories about pets. " +
                             "Keep stories under 500 words and appropriate for all ages.";
        
        String userPrompt = "Write a fun short story about a pet described as: " + description;
        
        // Configurar a requisição de IA
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Limitar o tamanho da resposta
                .temperature(0.8)          // Controlar a criatividade (0.0-1.0)
                .build();
        
        // Enviar requisição e obter resposta
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Componentes principais:**

1. **Cliente OpenAI**: Usa o SDK oficial OpenAI Java configurado para Azure AI Foundry (sem chave)
2. **Prompt do Sistema**: Define o comportamento da IA para escrever histórias de pets familiares e apropriadas para todas as idades
3. **Prompt do Usuário**: Informa à IA exatamente qual história escrever com base na descrição
4. **Parâmetros**: Controla o tamanho da história e o nível de criatividade
5. **Tratamento de Erros**: Lança exceções que o controlador captura e gerencia

### 4. Templates Web

**Arquivo:** `index.html` (Formulário de Upload)

Página principal onde os usuários descrevem seus pets:

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Pet Story Generator</title>
    <!-- CSS styling -->
</head>
<body>
    <div class="container">
        <h1>Pet Story Generator</h1>
        <p>Describe your pet and we'll create a fun story about them!</p>
        
        <!-- Error message display -->
        <div th:if="${error}" class="error" th:text="${error}"></div>
        
        <!-- Story generation form -->
        <form action="/generate-story" method="post">
            <div class="form-group">
                <label for="description">Describe your pet:</label>
                <textarea id="description" name="description" 
                         placeholder="Tell us about your pet - what they look like, their personality, favorite activities..."
                         maxlength="1000" required></textarea>
            </div>
            <button type="submit" class="btn btn-primary">Generate Story</button>
        </form>
        
        <!-- Image upload section with client-side processing -->
        <div class="upload-section">
            <h2>Or Upload a Photo</h2>
            <input type="file" id="imageInput" accept="image/*" />
            <button onclick="analyzeImage()" class="upload-btn">Analyze Image</button>
        </div>
        
        <script>
            // Client-side image analysis using Transformers.js
            async function analyzeImage() {
                // Image processing code here
                // Generates description automatically from uploaded image
            }
        </script>
    </div>
</body>
</html>
```

**Arquivo:** `result.html` (Exibição da História)

Mostra a história gerada:

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Pet Story Result</title>
</head>
<body>
    <div class="container">
        <h1>Your Pet's Story</h1>
        
        <div class="result-section">
            <div class="result-label">Pet Description:</div>
            <div class="result-content" th:text="${caption}"></div>
        </div>
        
        <div class="result-section">
            <div class="result-label">Generated Story:</div>
            <div class="result-content" th:text="${story}"></div>
        </div>
        
        <div class="result-section" th:if="${analysisType}">
            <div class="result-label">Analysis Type:</div>
            <div class="result-content" th:text="${analysisType}"></div>
        </div>
        
        <a href="/" class="back-link">Generate Another Story</a>
    </div>
</body>
</html>
```

**Características do template:**

1. **Integração com Thymeleaf**: Usa atributos `th:` para conteúdo dinâmico
2. **Design Responsivo**: Estilos CSS para mobile e desktop
3. **Tratamento de Erros**: Exibe mensagens de validação para os usuários
4. **Processamento no Cliente**: JavaScript para análise de imagens (usando Transformers.js)

### 5. Configuração

**Arquivo:** `application.properties`

Configurações para a aplicação:

```properties
spring.application.name=pet-story-app

# File upload limits
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=10MB

# Logging configuration
logging.level.com.example.petstory=INFO

# Azure AI Foundry (keyless) configuration
azure.openai.endpoint=${AZURE_OPENAI_ENDPOINT:}
azure.openai.deployment=${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}
```

**Explicação da configuração:**

1. **Upload de Arquivos**: Permite imagens de até 10MB
2. **Log**: Controla as informações logadas durante a execução
3. **Azure AI Foundry**: Especifica o endpoint e o deployment do modelo a usar (autenticação sem chave)
4. **Segurança**: Configuração de tratamento de erros para evitar exposição de informações sensíveis

## Executando a Aplicação

### Passo 1: Faça Login e Configure seu Endpoint

A autenticação não usa chave (Microsoft Entra ID), então não há chave de API. Faça login e defina seu endpoint Foundry:

**Windows (Prompt de Comando):**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**Windows (PowerShell):**
```powershell
az login
$env:AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
```

**Linux/macOS:**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**Por que isso é necessário:**
- O Azure AI Foundry usa Microsoft Entra ID para autenticar requisições de inferência
- A autenticação sem chave significa que não há segredos no seu código-fonte ou ambiente
- Sua conta precisa da função **Cognitive Services OpenAI User** no recurso

### Passo 2: Build e Execução

Navegue até o diretório do projeto:
```bash
cd 04-PracticalSamples/petstory
```

Compile a aplicação:
```bash
mvn clean compile
```

Inicie o servidor:
```bash
mvn spring-boot:run
```

A aplicação iniciará em `http://localhost:8080`.

### Passo 3: Teste a Aplicação

1. **Abra** `http://localhost:8080` no seu navegador
2. **Descreva** seu pet na área de texto (exemplo: "Um golden retriever brincalhão que adora buscar bolas")
3. **Clique** em "Generate Story" para obter uma história gerada pela IA
4. **Ou**, envie uma imagem do pet para gerar automaticamente uma descrição
5. **Veja** a história criativa baseada na descrição do seu pet

## Como Tudo Funciona Junto

Aqui está o fluxo completo ao gerar uma história de pet:

1. **Entrada do Usuário**: Você descreve seu pet no formulário web
2. **Envio do Formulário**: O navegador envia uma requisição POST para `/generate-story`
3. **Processamento no Controlador**: `PetController` valida e sanitiza a entrada
4. **Chamada ao Serviço de IA**: `StoryService` envia a requisição ao modelo Azure AI Foundry
5. **Geração da História**: A IA cria uma história criativa baseada na descrição
6. **Recebimento da Resposta**: O controlador recebe a história e adiciona ao modelo
7. **Renderização do Template**: Thymeleaf renderiza `result.html` com a história
8. **Exibição**: O usuário vê a história gerada no navegador

**Fluxo de Tratamento de Erro:**
Se o serviço de IA falhar:
1. O controlador captura a exceção
2. Gera uma história alternativa usando templates pré-escritos
3. Exibe a história alternativa com uma nota sobre a indisponibilidade da IA
4. O usuário ainda recebe uma história, garantindo boa experiência

## Entendendo a Integração com IA

### Azure AI Foundry (sem chave)
A aplicação usa Azure AI Foundry com autenticação sem chave (Microsoft Entra ID):

```java
// Autenticação sem chave - sem chave de API
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Engenharia de Prompt
O serviço usa prompts cuidadosamente elaborados para obter bons resultados:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Processamento da Resposta
A resposta da IA é extraída e validada:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Próximos Passos

Para mais exemplos, veja [Capítulo 04: Exemplos práticos](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Aviso Legal**:
Este documento foi traduzido usando o serviço de tradução por IA [Co-op Translator](https://github.com/Azure/co-op-translator). Embora nos esforcemos pela precisão, por favor, esteja ciente de que traduções automatizadas podem conter erros ou imprecisões. O documento original em seu idioma nativo deve ser considerado a fonte autorizada. Para informações críticas, recomenda-se tradução profissional humana. Não nos responsabilizamos por quaisquer mal-entendidos ou interpretações incorretas decorrentes do uso desta tradução.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->