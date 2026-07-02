# AGENTS.md

## Visão Geral do Projeto

Este é um repositório educativo para aprendizagem do desenvolvimento de IA Generativa com Java. Fornece um curso prático abrangente que cobre Grandes Modelos de Linguagem (LLMs), engenharia de prompt, embeddings, RAG (Generation Augmentada por Recuperação) e o Protocolo de Contexto de Modelo (MCP).

**Tecnologias Principais:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI e SDKs OpenAI

**Arquitetura:**
- Múltiplas aplicações Spring Boot independentes organizadas por capítulos
- Projetos de exemplo demonstrando diferentes padrões de IA
- Pronto para GitHub Codespaces com contêineres dev pré-configurados

## Comandos de Configuração

### Pré-requisitos
- Java 21 ou superior
- Maven 3.x
- Subscrição Azure com um deployment de modelo Azure AI Foundry (provisionar com `azd up`)
- Azure CLI (`az`) e Azure Developer CLI (`azd`), autenticados para auth sem chave

### Configuração de Ambiente

**Opção 1: GitHub Codespaces (Recomendado)**
```bash
# Faça um fork do repositório e crie um codespace a partir da interface do GitHub
# O contentor de desenvolvimento irá instalar automaticamente todas as dependências
# Aguarde cerca de 2 minutos para a configuração do ambiente
```

**Opção 2: Contêiner Dev Local**
```bash
# Clonar repositório
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Abrir no VS Code com a extensão Dev Containers
# Reabrir no Contentor quando solicitado
```

**Opção 3: Configuração Local**
```bash
# Instalar dependências
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Verificar instalação
java -version
mvn -version
```

### Configuração

**Configuração Azure AI Foundry (auth sem chave, recomendada):**
```bash
# Provisionar a conta Foundry + implementações de modelo como código
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd escreve examples/basic-chat-azure/.env com o seu endpoint (sem chave)
```

**Configuração manual de endpoint:**
```bash
# Se não usou azd, defina o endpoint você mesmo (a autenticação mantém-se sem chave via az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Edite .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Fluxo de Trabalho de Desenvolvimento

### Estrutura do Projeto
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

### Executar Aplicações

**Executar uma aplicação Spring Boot:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Compilar um projeto:**
```bash
cd [project-directory]
mvn clean install
```

**Iniciar o Servidor Calculadora MCP:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# O servidor funciona em http://localhost:8080
```

**Executar Exemplos Client:**
```bash
# Depois de iniciar o servidor noutro terminal
cd 04-PracticalSamples/calculator

# Cliente MCP direto
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# Cliente com IA (requer AZURE_OPENAI_ENDPOINT + az login)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Bot interativo
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Hot Reload
Spring Boot DevTools está incluído em projetos que suportam hot reload:
```bash
# As alterações nos ficheiros Java serão recarregadas automaticamente quando guardadas
mvn spring-boot:run
```

## Instruções de Teste

### Executar Testes

**Executar todos os testes de um projeto:**
```bash
cd [project-directory]
mvn test
```

**Executar testes com saída detalhada:**
```bash
mvn test -X
```

**Executar classe de teste específica:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Estrutura dos Testes
- Os ficheiros de teste usam JUnit 5 (Jupiter)
- As classes de teste localizam-se em `src/test/java/`
- Exemplos client no projeto calculadora estão em `src/test/java/com/microsoft/mcp/sample/client/`

### Teste Manual
Muitos exemplos são aplicações interativas que requerem teste manual:

1. Inicie a aplicação com `mvn spring-boot:run`
2. Teste os endpoints ou interaja com o CLI
3. Verifique se o comportamento esperado corresponde à documentação no README.md de cada projeto

### Teste com Azure AI Foundry
- Autentique-se com `az login` antes de executar exemplos (auth sem chave)
- Garanta que a sua conta tem o papel Cognitive Services OpenAI User no recurso
- Teste filtragem de conteúdo com o exemplo de IA responsável no Capítulo 5

## Diretrizes de Estilo de Código

### Convenções Java
- **Versão Java:** Java 21 com funcionalidades modernas
- **Estilo:** Seguir convenções padrão Java
- **Nomes:** 
  - Classes: PascalCase
  - Métodos/variáveis: camelCase
  - Constantes: UPPER_SNAKE_CASE
  - Nomes de pacotes: minúsculas

### Padrões Spring Boot
- Usar `@Service` para lógica de negócio
- Usar `@RestController` para endpoints REST
- Configuração via `application.yml` ou `application.properties`
- Preferir variáveis de ambiente a valores codificados
- Usar a anotação `@Tool` para métodos expostos pelo MCP

### Organização de Ficheiros
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

### Dependências
- Geridas via Maven `pom.xml`
- Spring AI BOM para gestão de versões
- LangChain4j para integrações de IA
- Spring Boot starter parent para dependências Spring

### Comentários no Código
- Adicionar JavaDoc para APIs públicas
- Incluir comentários explicativos para interações complexas de IA
- Documentar claramente descrições de ferramentas MCP

## Construção e Desdobramento

### Construir Projetos

**Construir sem testes:**
```bash
mvn clean install -DskipTests
```

**Construir com todas as verificações:**
```bash
mvn clean install
```

**Empacotar aplicação:**
```bash
mvn package
# Cria JAR no diretório target/
```

### Diretórios de Saída
- Classes compiladas: `target/classes/`
- Classes de teste: `target/test-classes/`
- Ficheiros JAR: `target/*.jar`
- Artefactos Maven: `target/`

### Configuração Específica ao Ambiente

**Desenvolvimento:**
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

**Produção:**
- Use uma identidade gerida em vez de `az login` para auth sem chave
- Aponte `AZURE_OPENAI_ENDPOINT` ao seu recurso Foundry de produção
- Gerencie configuração via variáveis de ambiente ou Azure Key Vault

### Considerações para Desdobramento
- Este é um repositório educativo com aplicações de exemplo
- Não projetado para desdobramento em produção tal qual está
- Os exemplos demonstram padrões para adaptar ao uso em produção
- Veja os READMEs de cada projeto para notas específicas de desdobramento

## Notas Adicionais

### Azure AI Foundry
- **Auth sem chave:** conecte com Microsoft Entra ID — sem necessidade de gerir chaves API
- **Provisionado como código:** Bicep + azd (`azd up`) criam a conta e os deployments dos modelos
- O mesmo código compatível com OpenAI corre localmente (`az login`) e na Azure (identidade gerida)

### Trabalho com Múltiplos Projetos
Cada projeto de exemplo é autónomo:
```bash
# Navegar para projeto específico
cd 04-PracticalSamples/[project-name]

# Cada um tem o seu próprio pom.xml e pode ser construído independentemente
mvn clean install
```

### Problemas Comuns

**Incompatibilidade de Versão Java:**
```bash
# Verificar Java 21
java -version
# Atualizar JAVA_HOME se necessário
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Problemas de Download de Dependências:**
```bash
# Limpar a cache do Maven e tentar novamente
rm -rf ~/.m2/repository
mvn clean install
```

**Endpoint ou Log-in Não Encontrado:**
```bash
# Defina o endpoint na sessão atual e inicie sessão (sem chave)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Ou utilize um ficheiro .env no diretório do projeto
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Porta Já Em Uso:**
```bash
# O Spring Boot usa a porta 8080 por defeito
# Alteração em application.properties:
server.port=8081
```

### Suporte Multilíngue
- Documentação disponível em mais de 45 idiomas via tradução automática
- Traduções na pasta `translations/`
- Tradução gerida por workflow GitHub Actions

### Caminho de Aprendizagem
1. Comece com [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. Siga os capítulos pela ordem (01 → 05)
3. Complete os exemplos práticos em cada capítulo
4. Explore os projetos de exemplo no Capítulo 4
5. Aprenda práticas de IA responsável no Capítulo 5

### Contêiner de Desenvolvimento
O `.devcontainer/devcontainer.json` configura:
- Ambiente de desenvolvimento Java 21
- Maven pré-instalado
- Extensões Java para VS Code
- Ferramentas Spring Boot
- Integração GitHub Copilot
- Suporte Docker-in-Docker
- Azure CLI

### Considerações de Performance
- Deployments Azure AI Foundry têm quotas por minuto de tokens/pedidos
- Use tamanhos de batch apropriados para embeddings
- Considere caching para chamadas repetidas à API
- Monitorize o uso de tokens para otimização de custos

### Notas de Segurança
- Nunca faça commit de ficheiros `.env` (já estão no `.gitignore`)
- Prefira auth sem chave (Microsoft Entra ID) em vez de chaves API
- Use identidades geridas na Azure; `az login` para desenvolvimento local
- Siga as diretrizes de IA responsável no Capítulo 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Aviso Legal**:
Este documento foi traduzido utilizando o serviço de tradução automática [Co-op Translator](https://github.com/Azure/co-op-translator). Embora nos esforcemos pela precisão, esteja ciente de que traduções automáticas podem conter erros ou imprecisões. O documento original na sua língua nativa deve ser considerado a fonte autorizada. Para informações críticas, recomenda-se tradução profissional humana. Não nos responsabilizamos por quaisquer mal-entendidos ou interpretações incorretas resultantes da utilização desta tradução.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->