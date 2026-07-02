# AGENTS.md

## Descripción del Proyecto

Este es un repositorio educativo para aprender desarrollo de IA Generativa con Java. Proporciona un curso práctico completo que cubre Modelos de Lenguaje Grandes (LLMs), ingeniería de prompts, embeddings, RAG (Generación aumentada por recuperación) y el Protocolo de Contexto de Modelo (MCP).

**Tecnologías clave:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI y SDKs de OpenAI

**Arquitectura:**
- Múltiples aplicaciones Spring Boot independientes organizadas por capítulos
- Proyectos de ejemplo que demuestran diferentes patrones de IA
- Preparado para GitHub Codespaces con contenedores de desarrollo preconfigurados

## Comandos de Configuración

### Requisitos Previos
- Java 21 o superior
- Maven 3.x
- Suscripción a Azure con un despliegue de modelo Azure AI Foundry (provisionar con `azd up`)
- Azure CLI (`az`) y Azure Developer CLI (`azd`), iniciados para autenticación sin clave

### Configuración del Entorno

**Opción 1: GitHub Codespaces (Recomendado)**
```bash
# Haz un fork del repositorio y crea un codespace desde la interfaz de GitHub
# El contenedor de desarrollo instalará automáticamente todas las dependencias
# Espera aproximadamente 2 minutos para la configuración del entorno
```

**Opción 2: Contenedor local de desarrollo**
```bash
# Clonar repositorio
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Abrir en VS Code con la extensión Dev Containers
# Reabrir en el contenedor cuando se solicite
```

**Opción 3: Configuración local**
```bash
# Instalar dependencias
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Verificar la instalación
java -version
mvn -version
```

### Configuración

**Configuración de Azure AI Foundry (sin clave, recomendado):**
```bash
# Provisione la cuenta de Foundry + despliegues de modelos como código
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd escribe examples/basic-chat-azure/.env con su endpoint (sin clave)
```

**Configuración manual de endpoint:**
```bash
# Si no usaste azd, configura el endpoint tú mismo (la autenticación permanece sin clave mediante az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Edita .env: AZURE_OPENAI_ENDPOINT=https://<recurso>.openai.azure.com/
```

## Flujo de Trabajo de Desarrollo

### Estructura del Proyecto
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

### Ejecución de Aplicaciones

**Ejecutar una aplicación Spring Boot:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Compilar un proyecto:**
```bash
cd [project-directory]
mvn clean install
```

**Iniciar el Servidor MCP Calculator:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# El servidor se ejecuta en http://localhost:8080
```

**Ejecutar Ejemplos Cliente:**
```bash
# Después de iniciar el servidor en otro terminal
cd 04-PracticalSamples/calculator

# Cliente MCP directo
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# Cliente potenciado por IA (requiere AZURE_OPENAI_ENDPOINT + az login)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Bot interactivo
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Recarga en caliente
Spring Boot DevTools está incluido en los proyectos que soportan recarga en caliente:
```bash
# Los cambios en los archivos Java se recargarán automáticamente al guardarlos
mvn spring-boot:run
```

## Instrucciones para Pruebas

### Ejecutar Pruebas

**Ejecutar todas las pruebas en un proyecto:**
```bash
cd [project-directory]
mvn test
```

**Ejecutar pruebas con salida detallada:**
```bash
mvn test -X
```

**Ejecutar clase de prueba específica:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Estructura de Pruebas
- Los archivos de prueba usan JUnit 5 (Jupiter)
- Las clases de prueba están ubicadas en `src/test/java/`
- Los ejemplos cliente en el proyecto calculator están en `src/test/java/com/microsoft/mcp/sample/client/`

### Pruebas Manuales
Muchos ejemplos son aplicaciones interactivas que requieren pruebas manuales:

1. Inicie la aplicación con `mvn spring-boot:run`
2. Pruebe endpoints o interactúe con la CLI
3. Verifique que el comportamiento esperado coincida con la documentación en el README.md de cada proyecto

### Pruebas con Azure AI Foundry
- Inicie sesión con `az login` antes de ejecutar ejemplos (autenticación sin clave)
- Asegúrese de que su cuenta tenga el rol Usuario de OpenAI de Cognitive Services en el recurso
- Pruebe el filtrado de contenido con el ejemplo de IA responsable en el Capítulo 5

## Guías de Estilo de Código

### Convenciones de Java
- **Versión de Java:** Java 21 con características modernas
- **Estilo:** Siga las convenciones estándar de Java
- **Nomenclatura:** 
  - Clases: PascalCase
  - Métodos/variables: camelCase
  - Constantes: UPPER_SNAKE_CASE
  - Nombres de paquetes: minúsculas

### Patrones de Spring Boot
- Usar `@Service` para lógica de negocio
- Usar `@RestController` para endpoints REST
- Configuración vía `application.yml` o `application.properties`
- Preferir variables de entorno en lugar de valores codificados
- Usar anotación `@Tool` para métodos expuestos por MCP

### Organización de Archivos
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

### Dependencias
- Gestionadas vía Maven `pom.xml`
- Spring AI BOM para gestión de versiones
- LangChain4j para integraciones de IA
- Spring Boot starter parent para dependencias de Spring

### Comentarios en Código
- Añadir JavaDoc para APIs públicas
- Incluir comentarios explicativos para interacciones complejas de IA
- Documentar claramente las descripciones de herramientas MCP

## Construcción y Despliegue

### Construcción de Proyectos

**Compilar sin pruebas:**
```bash
mvn clean install -DskipTests
```

**Compilar con todas las validaciones:**
```bash
mvn clean install
```

**Empaquetar la aplicación:**
```bash
mvn package
# Crea el JAR en el directorio target/
```

### Directorios de Salida
- Clases compiladas: `target/classes/`
- Clases de prueba: `target/test-classes/`
- Archivos JAR: `target/*.jar`
- Artefactos Maven: `target/`

### Configuración Específica por Entorno

**Desarrollo:**
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

**Producción:**
- Usar una identidad administrada en lugar de `az login` para autenticación sin clave
- Apuntar `AZURE_OPENAI_ENDPOINT` a su recurso Foundry de producción
- Gestionar configuración mediante variables de entorno o Azure Key Vault

### Consideraciones para el Despliegue
- Este es un repositorio educativo con aplicaciones de ejemplo
- No está diseñado para despliegue en producción tal cual
- Los ejemplos demuestran patrones para adaptar en producción
- Ver los README individuales de cada proyecto para notas de despliegue específicas

## Notas Adicionales

### Azure AI Foundry
- **Autenticación sin clave:** conéctese con Microsoft Entra ID — no hay claves API que gestionar
- **Provisionado como código:** Bicep + azd (`azd up`) crean la cuenta y despliegan modelos
- El mismo código compatible con OpenAI se ejecuta localmente (`az login`) y en Azure (identidad administrada)

### Trabajo con Múltiples Proyectos
Cada proyecto de ejemplo es independiente:
```bash
# Navegar al proyecto específico
cd 04-PracticalSamples/[project-name]

# Cada uno tiene su propio pom.xml y puede construirse de forma independiente
mvn clean install
```

### Problemas Comunes

**Desajuste de versión de Java:**
```bash
# Verificar Java 21
java -version
# Actualizar JAVA_HOME si es necesario
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Problemas descargando dependencias:**
```bash
# Borrar la caché de Maven y reintentar
rm -rf ~/.m2/repository
mvn clean install
```

**Endpoint o inicio de sesión no encontrados:**
```bash
# Establecer el endpoint en la sesión actual e iniciar sesión (sin clave)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# O usar un archivo .env en el directorio del proyecto
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Puerto ya en uso:**
```bash
# Spring Boot usa el puerto 8080 por defecto
# Cambiar en application.properties:
server.port=8081
```

### Soporte Multilenguaje
- Documentación disponible en más de 45 idiomas vía traducción automática
- Traducciones en el directorio `translations/`
- Traducción gestionada por workflow de GitHub Actions

### Ruta de Aprendizaje
1. Comience con [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. Siga los capítulos en orden (01 → 05)
3. Complete los ejemplos prácticos de cada capítulo
4. Explore los proyectos de ejemplo en el Capítulo 4
5. Aprenda prácticas de IA responsable en el Capítulo 5

### Contenedor de Desarrollo
El archivo `.devcontainer/devcontainer.json` configura:
- Entorno de desarrollo Java 21
- Maven preinstalado
- Extensiones Java para VS Code
- Herramientas Spring Boot
- Integración con GitHub Copilot
- Soporte Docker-in-Docker
- Azure CLI

### Consideraciones de Rendimiento
- Los despliegues de Azure AI Foundry tienen cuotas por minuto de tokens/solicitudes
- Usar tamaños de lote apropiados para embeddings
- Considerar caché para llamadas API repetidas
- Monitorizar uso de tokens para optimización de costos

### Notas de Seguridad
- Nunca comprometer archivos `.env` (ya están en `.gitignore`)
- Preferir autenticación sin clave (Microsoft Entra ID) sobre claves API
- Usar identidades administradas en Azure; `az login` para desarrollo local
- Seguir las guías de IA responsable en el Capítulo 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Descargo de responsabilidad**:
Este documento ha sido traducido utilizando el servicio de traducción automática [Co-op Translator](https://github.com/Azure/co-op-translator). Aunque nos esforzamos por la precisión, tenga en cuenta que las traducciones automatizadas pueden contener errores o inexactitudes. El documento original en su idioma nativo debe considerarse la fuente autorizada. Para información crítica, se recomienda una traducción profesional humana. No somos responsables de cualquier malentendido o interpretación errónea que surja del uso de esta traducción.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->