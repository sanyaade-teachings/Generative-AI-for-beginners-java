# Tutorial do Gerador de Histórias de Animais para Principiantes

## Índice

- [Pré-requisitos](#pré-requisitos)
- [Compreender a Estrutura do Projeto](#compreender-a-estrutura-do-projeto)
- [Componentes Principais Explicados](#componentes-principais-explicados)
  - [1. Aplicação Principal](#1-aplicação-principal)
  - [2. Controlador Web](#2-controlador-web)
  - [3. Serviço de Histórias](#3-serviço-de-histórias)
  - [4. Templates Web](#4-templates-web)
  - [5. Configuração](#5-configuração)
- [Executar a Aplicação](#executar-a-aplicação)
- [Como Tudo Funciona em Conjunto](#como-tudo-funciona-em-conjunto)
- [Compreender a Integração de IA](#compreender-a-integração-de-ia)
- [Próximos Passos](#próximos-passos)

## Pré-requisitos

Antes de começar, certifique-se de ter:
- Java 21 ou superior instalado
- Maven para gestão de dependências
- Um deployment de modelo Azure AI Foundry (provisione com `azd up` — veja [Capítulo 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), autenticado com `az login` (autenticação sem chave)
- Conhecimentos básicos de Java, Spring Boot e desenvolvimento web

## Compreender a Estrutura do Projeto

O projeto de histórias para animais tem vários ficheiros importantes:

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

**Ficheiro:** `PetStoryApplication.java`

Este é o ponto de entrada da nossa aplicação Spring Boot:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**O que isto faz:**
- A anotação `@SpringBootApplication` ativa a configuração automática e a deteção de componentes
- Inicia um servidor web incorporado (Tomcat) na porta 8080
- Cria automaticamente todos os beans e serviços Spring necessários

### 2. Controlador Web

**Ficheiro:** `PetController.java`

Este lida com todas as requisições web e interações do utilizador:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Devolve o modelo index.html
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
        
        // Sanitizar a entrada para segurança
        String sanitizedDescription = sanitizeInput(description);
        
        // Gerar história com tratamento de erros
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Devolve o modelo result.html
            
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
                   .substring(0, Math.min(input.length(), 500));  // Limitar o comprimento
    }
}
```

**Funcionalidades principais:**

1. **Gestão de Rotas**: `@GetMapping("/")` mostra o formulário de upload, `@PostMapping("/generate-story")` processa envios
2. **Validação de Entrada**: Verifica descrições vazias e limites de tamanho
3. **Segurança**: Sanitiza a entrada do utilizador para prevenir ataques XSS
4. **Gestão de Erros**: Fornece histórias de fallback quando o serviço de IA falha
5. **Ligação de Modelo**: Passa dados para templates HTML usando o `Model` do Spring

**Sistema de Fallback:**
O controlador inclui templates de histórias pré-escritas usados quando o serviço de IA não está disponível:

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

**Ficheiro:** `StoryService.java`

Este serviço comunica com o Azure AI Foundry para gerar histórias usando autenticação sem chave:

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
        
        // O endpoint compatível com OpenAI da Foundry está em /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Autenticação sem chave com Microsoft Entra ID (sem chave API)
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
        
        // Configurar o pedido à IA
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Limitar o comprimento da resposta
                .temperature(0.8)          // Controlar criatividade (0.0-1.0)
                .build();
        
        // Enviar pedido e obter resposta
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Componentes principais:**

1. **Cliente OpenAI**: Utiliza o SDK oficial OpenAI Java configurado para Azure AI Foundry (sem chave)
2. **Prompt do Sistema**: Define o comportamento da IA para escrever histórias familiares sobre animais
3. **Prompt do Utilizador**: Diz exatamente à IA que história escrever com base na descrição
4. **Parâmetros**: Controla o comprimento da história e o nível de criatividade
5. **Gestão de Erros**: Lança exceções que o controlador captura e trata

### 4. Templates Web

**Ficheiro:** `index.html` (Formulário de Upload)

A página principal onde os utilizadores descrevem os seus animais:

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

**Ficheiro:** `result.html` (Exibição da História)

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

**Características do Template:**

1. **Integração Thymeleaf**: Usa atributos `th:` para conteúdo dinâmico
2. **Design Responsivo**: Estilização CSS para dispositivos móveis e desktop
3. **Gestão de Erros**: Exibe erros de validação aos utilizadores
4. **Processamento Client-side**: JavaScript para análise de imagem (usando Transformers.js)

### 5. Configuração

**Ficheiro:** `application.properties`

Configurações da aplicação:

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

**Configuração explicada:**

1. **Upload de Ficheiros**: Permite imagens até 10MB
2. **Registo (Logging)**: Controla que informação é registada durante a execução
3. **Azure AI Foundry**: Especifica o endpoint e deployment do modelo a usar (autenticação sem chave)
4. **Segurança**: Configuração de tratamento de erros para evitar exposição de informação sensível

## Executar a Aplicação

### Passo 1: Autenticar e Definir o Endpoint

A autenticação é sem chave (Microsoft Entra ID), portanto não há chave API. Autentique-se e defina o endpoint Foundry:

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

**Porquê isto é necessário:**
- O Azure AI Foundry usa Microsoft Entra ID para autenticar pedidos de inferência
- Autenticação sem chave significa que não há segredos no código-fonte nem no ambiente
- A sua conta precisa da função **Cognitive Services OpenAI User** no recurso

### Passo 2: Construir e Executar

Navegue até à pasta do projeto:
```bash
cd 04-PracticalSamples/petstory
```

Construa a aplicação:
```bash
mvn clean compile
```

Inicie o servidor:
```bash
mvn spring-boot:run
```

A aplicação arrancará em `http://localhost:8080`.

### Passo 3: Testar a Aplicação

1. **Abra** `http://localhost:8080` no seu navegador
2. **Descreva** o seu animal na área de texto (por exemplo, "Um golden retriever brincalhão que adora buscar")
3. **Clique** em "Generate Story" para receber uma história gerada por IA
4. **Alternativamente**, carregue uma imagem do animal para gerar automaticamente uma descrição
5. **Veja** a história criativa baseada na descrição do seu animal

## Como Tudo Funciona em Conjunto

Aqui está o fluxo completo quando gera uma história para um animal:

1. **Entrada do Utilizador**: Descreve o seu animal no formulário web
2. **Submissão do Formulário**: O navegador envia uma requisição POST para `/generate-story`
3. **Processamento no Controlador**: O `PetController` valida e sanitiza a entrada
4. **Chamada ao Serviço de IA**: O `StoryService` envia pedido ao modelo Azure AI Foundry
5. **Geração da História**: A IA gera uma história criativa baseada na descrição
6. **Tratamento da Resposta**: O controlador recebe a história e adiciona-a ao modelo
7. **Renderização do Template**: O Thymeleaf renderiza o `result.html` com a história
8. **Exibição**: O utilizador vê a história gerada no seu navegador

**Fluxo de Tratamento de Erros:**
Se o serviço de IA falhar:
1. O controlador captura a exceção
2. Gera uma história de fallback usando templates pré-escritos
3. Exibe a história de fallback com uma nota sobre a indisponibilidade da IA
4. O utilizador recebe ainda assim uma história, garantindo boa experiência

## Compreender a Integração de IA

### Azure AI Foundry (sem chave)
A aplicação usa Azure AI Foundry com autenticação sem chave (Microsoft Entra ID):

```java
// Autenticação sem chave - sem chave API
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

Para mais exemplos, veja [Capítulo 04: Exemplos Práticos](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Aviso Legal**:
Este documento foi traduzido utilizando o serviço de tradução automática [Co-op Translator](https://github.com/Azure/co-op-translator). Embora nos esforcemos pela precisão, esteja ciente de que traduções automáticas podem conter erros ou imprecisões. O documento original na sua língua nativa deve ser considerado a fonte autorizada. Para informações críticas, recomenda-se tradução profissional humana. Não nos responsabilizamos por quaisquer mal-entendidos ou interpretações incorretas resultantes da utilização desta tradução.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->