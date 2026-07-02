# AGENTS.md

## Visão Geral do Projeto

Este é um repositório educacional para aprendizado de desenvolvimento de IA Generativa com Java. Ele oferece um curso prático completo que abrange Grandes Modelos de Linguagem (LLMs), engenharia de prompts, embeddings, RAG (Geração Incrementada por Recuperação) e o Protocolo de Contexto de Modelo (MCP).

**Tecnologias Principais:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI, e SDKs OpenAI

**Arquitetura:**
- Múltiplas aplicações Spring Boot independentes organizadas por capítulos
- Projetos de exemplo demonstrando diferentes padrões de IA
- Pronto para GitHub Codespaces com contêineres de desenvolvimento pré-configurados

## Comandos de Configuração

### Pré-requisitos
- Java 21 ou superior
- Maven 3.x
- Assinatura Azure com uma implantação de modelo Azure AI Foundry (provisione com `azd up`)
- Azure CLI (`az`) e Azure Developer CLI (`azd`), autenticados para auth sem chave

### Configuração do Ambiente

**Opção 1: GitHub Codespaces (Recomendado)**
```bash
# Faça um fork do repositório e crie um codespace a partir da interface do GitHub
# O contêiner de desenvolvimento instalará automaticamente todas as dependências
# Aguarde cerca de 2 minutos para a configuração do ambiente
```

**Opção 2: Contêiner de Desenvolvimento Local**
```bash
# Clonar repositório
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Abrir no VS Code com extensão Dev Containers
# Reabrir no Container quando solicitado
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

**Configuração Azure AI Foundry (sem chave, recomendado):**
```bash
# Provisionar a conta Foundry + implantações de modelo como código
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd grava examples/basic-chat-azure/.env com seu endpoint (sem chave)
```

**Configuração manual do endpoint:**
```bash
# Se você não usou azd, defina o endpoint você mesmo (a autenticação permanece sem chave via az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Edite o .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
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

### Executando Aplicações

**Executando uma aplicação Spring Boot:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Construindo um projeto:**
```bash
cd [project-directory]
mvn clean install
```

**Iniciando o Servidor Calculadora MCP:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# O servidor roda em http://localhost:8080
```

**Executando Exemplos de Cliente:**
```bash
# Após iniciar o servidor em outro terminal
cd 04-PracticalSamples/calculator

# Cliente MCP direto
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# Cliente com IA (requer AZURE_OPENAI_ENDPOINT + az login)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Bot interativo
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Hot Reload
Spring Boot DevTools está incluído nos projetos que suportam hot reload:
```bash
# Alterações em arquivos Java serão recarregadas automaticamente ao salvar
mvn spring-boot:run
```

## Instruções de Teste

### Executando Testes

**Executar todos os testes em um projeto:**
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

### Estrutura de Testes
- Arquivos de teste usam JUnit 5 (Jupiter)
- Classes de teste localizadas em `src/test/java/`
- Exemplos de cliente no projeto calculator estão em `src/test/java/com/microsoft/mcp/sample/client/`

### Teste Manual
Muitos exemplos são aplicações interativas que requerem testes manuais:

1. Inicie a aplicação com `mvn spring-boot:run`
2. Teste endpoints ou interaja com a CLI
3. Verifique se o comportamento esperado corresponde à documentação em cada README.md do projeto

### Testando com Azure AI Foundry
- Faça login com `az login` antes de executar os exemplos (auth sem chave)
- Certifique-se de que sua conta tenha o papel Cognitive Services OpenAI User no recurso
- Teste o filtro de conteúdo com o exemplo de IA responsável no Capítulo 5

## Diretrizes de Estilo de Código

### Convenções Java
- **Versão do Java:** Java 21 com recursos modernos
- **Estilo:** Siga as convenções padrão do Java
- **Nomenclatura:** 
  - Classes: PascalCase
  - Métodos/variáveis: camelCase
  - Constantes: UPPER_SNAKE_CASE
  - Nomes de pacotes: minúsculos

### Padrões Spring Boot
- Use `@Service` para lógica de negócio
- Use `@RestController` para endpoints REST
- Configuração via `application.yml` ou `application.properties`
- Variáveis de ambiente preferidas em vez de valores hard-coded
- Use anotação `@Tool` para métodos expostos pelo MCP

### Organização de Arquivos
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
- Gerenciado via Maven `pom.xml`
- Spring AI BOM para gerenciamento de versões
- LangChain4j para integrações com IA
- Spring Boot starter parent para dependências Spring

### Comentários no Código
- Adicione JavaDoc para APIs públicas
- Inclua comentários explicativos para interações complexas de IA
- Documente claramente as descrições das ferramentas MCP

## Build e Deploy

### Compilando Projetos

**Build sem testes:**
```bash
mvn clean install -DskipTests
```

**Build com todas as verificações:**
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
- Arquivos JAR: `target/*.jar`
- Artefatos Maven: `target/`

### Configuração Específica de Ambiente

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
- Use uma identidade gerenciada em vez de `az login` para autenticação sem chave
- Aponte `AZURE_OPENAI_ENDPOINT` para seu recurso Foundry de produção
- Gerencie configuração via variáveis de ambiente ou Azure Key Vault

### Considerações para Deploy
- Este é um repositório educacional com aplicações de exemplo
- Não projetado para deploy em produção como está
- Exemplos demonstram padrões para adaptação ao uso em produção
- Veja os READMEs dos projetos individuais para notas específicas de deploy

## Notas Adicionais

### Azure AI Foundry
- **Autenticação sem chave:** conecte com Microsoft Entra ID — sem necessidade de gerenciar chaves API
- **Provisionado como código:** Bicep + azd (`azd up`) criam a conta e implantações de modelo
- O mesmo código compatível com OpenAI funciona localmente (`az login`) e no Azure (identidade gerenciada)

### Trabalhando com Múltiplos Projetos
Cada projeto de exemplo é independente:
```bash
# Navegar para projeto específico
cd 04-PracticalSamples/[project-name]

# Cada um tem seu próprio pom.xml e pode ser compilado independentemente
mvn clean install
```

### Problemas Comuns

**Incompatibilidade de Versão Java:**
```bash
# Verifique o Java 21
java -version
# Atualize o JAVA_HOME se necessário
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Problemas no Download de Dependências:**
```bash
# Limpar o cache do Maven e tentar novamente
rm -rf ~/.m2/repository
mvn clean install
```

**Endpoint ou Login Não Encontrado:**
```bash
# Defina o endpoint na sessão atual e faça login (sem chave)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Ou use um arquivo .env no diretório do projeto
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Porta Já em Uso:**
```bash
# O Spring Boot usa a porta 8080 por padrão
# Alteração em application.properties:
server.port=8081
```

### Suporte Multilíngue
- Documentação disponível em 45+ idiomas via tradução automática
- Traduções na pasta `translations/`
- Tradução gerenciada por workflow do GitHub Actions

### Caminho de Aprendizado
1. Comece com [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. Siga os capítulos em ordem (01 → 05)
3. Complete os exemplos práticos em cada capítulo
4. Explore projetos de exemplo no Capítulo 4
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
- Implantações Azure AI Foundry têm cotas por minuto de tokens/solicitações
- Use tamanhos de lote apropriados para embeddings
- Considere caching para chamadas repetidas de API
- Monitore o uso de tokens para otimizar custos

### Notas de Segurança
- Nunca faça commit de arquivos `.env` (já no `.gitignore`)
- Prefira autenticação sem chave (Microsoft Entra ID) em vez de chaves de API
- Use identidades gerenciadas no Azure; `az login` para desenvolvimento local
- Siga diretrizes de IA responsável no Capítulo 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Aviso Legal**:
Este documento foi traduzido usando o serviço de tradução por IA [Co-op Translator](https://github.com/Azure/co-op-translator). Embora nos esforcemos pela precisão, por favor, esteja ciente de que traduções automatizadas podem conter erros ou imprecisões. O documento original em seu idioma nativo deve ser considerado a fonte autorizada. Para informações críticas, recomenda-se tradução profissional humana. Não nos responsabilizamos por quaisquer mal-entendidos ou interpretações incorretas decorrentes do uso desta tradução.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->