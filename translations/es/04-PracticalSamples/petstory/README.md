# Tutorial del Generador de Historias de Mascotas para Principiantes

## Tabla de Contenidos

- [Requisitos previos](#requisitos-previos)
- [Comprendiendo la Estructura del Proyecto](#comprendiendo-la-estructura-del-proyecto)
- [Componentes Principales Explicados](#componentes-principales-explicados)
  - [1. Aplicación Principal](#1-aplicación-principal)
  - [2. Controlador Web](#2-controlador-web)
  - [3. Servicio de Historias](#3-servicio-de-historias)
  - [4. Plantillas Web](#4-plantillas-web)
  - [5. Configuración](#5-configuración)
- [Ejecución de la Aplicación](#ejecución-de-la-aplicación)
- [Cómo Funciona Todo Junto](#cómo-funciona-todo-junto)
- [Comprendiendo la Integración de IA](#comprendiendo-la-integración-de-ia)
- [Próximos Pasos](#próximos-pasos)

## Requisitos previos

Antes de comenzar, asegúrate de tener:
- Java 21 o superior instalado
- Maven para la gestión de dependencias
- Un despliegue de modelo Azure AI Foundry (provisiónalo con `azd up` — véase [Capítulo 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), con sesión iniciada mediante `az login` (autenticación sin clave)
- Conocimiento básico de Java, Spring Boot y desarrollo web

## Comprendiendo la Estructura del Proyecto

El proyecto de historia de mascotas tiene varios archivos importantes:

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

## Componentes Principales Explicados

### 1. Aplicación Principal

**Archivo:** `PetStoryApplication.java`

Este es el punto de entrada para nuestra aplicación Spring Boot:

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Qué hace esto:**
- La anotación `@SpringBootApplication` habilita la autoconfiguración y el escaneo de componentes
- Inicia un servidor web embebido (Tomcat) en el puerto 8080
- Crea automáticamente todos los beans y servicios necesarios de Spring

### 2. Controlador Web

**Archivo:** `PetController.java`

Este maneja todas las solicitudes web e interacciones del usuario:

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Devuelve la plantilla index.html
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Validación de entrada
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Sanitizar la entrada por seguridad
        String sanitizedDescription = sanitizeInput(description);
        
        // Generar historia con manejo de errores
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Devuelve la plantilla result.html
            
        } catch (Exception e) {
            // Usar historia de respaldo si la IA falla
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Limitar la longitud
    }
}
```

**Características clave:**

1. **Manejo de Rutas**: `@GetMapping("/")` muestra el formulario de carga, `@PostMapping("/generate-story")` procesa las presentaciones
2. **Validación de Entrada**: Verifica descripciones vacías y límites de longitud
3. **Seguridad**: Sanitiza la entrada del usuario para prevenir ataques XSS
4. **Manejo de Errores**: Proporciona historias alternativas cuando el servicio de IA falla
5. **Vinculación de Modelo**: Pasa datos a las plantillas HTML usando el `Model` de Spring

**Sistema Alternativo:**
El controlador incluye plantillas de historia preescritas que se usan cuando el servicio de IA no está disponible:

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Utilice hash de descripción para respuestas consistentes
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Servicio de Historias

**Archivo:** `StoryService.java`

Este servicio se comunica con Azure AI Foundry para generar historias usando autenticación sin clave:

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
        
        // El endpoint compatible con OpenAI de Foundry se encuentra en /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Autenticación sin clave con Microsoft Entra ID (sin clave API)
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
        
        // Configurar la solicitud de IA
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Limitar la longitud de la respuesta
                .temperature(0.8)          // Controlar la creatividad (0.0-1.0)
                .build();
        
        // Enviar la solicitud y obtener la respuesta
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Componentes clave:**

1. **Cliente OpenAI**: Usa el SDK oficial de OpenAI para Java configurado para Azure AI Foundry (sin clave)
2. **Indicador del Sistema**: Establece el comportamiento de la IA para escribir historias de mascotas aptas para la familia
3. **Indicador de Usuario**: Le dice a la IA exactamente qué historia escribir según la descripción
4. **Parámetros**: Controla la longitud de la historia y el nivel de creatividad
5. **Manejo de Errores**: Lanza excepciones que el controlador captura y maneja

### 4. Plantillas Web

**Archivo:** `index.html` (Formulario de carga)

La página principal donde los usuarios describen sus mascotas:

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

**Archivo:** `result.html` (Mostrar historia)

Muestra la historia generada:

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

**Características de la plantilla:**

1. **Integración con Thymeleaf**: Usa atributos `th:` para contenido dinámico
2. **Diseño Responsive**: Estilo CSS para móviles y escritorio
3. **Manejo de Errores**: Muestra errores de validación a los usuarios
4. **Procesamiento en el Cliente**: JavaScript para análisis de imágenes (usando Transformers.js)

### 5. Configuración

**Archivo:** `application.properties`

Configuraciones para la aplicación:

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

**Explicación de la configuración:**

1. **Carga de Archivos**: Permite imágenes de hasta 10MB
2. **Registro de Eventos (Logging)**: Controla qué información se registra durante la ejecución
3. **Azure AI Foundry**: Especifica el endpoint y despliegue de modelo a usar (autenticación sin clave)
4. **Seguridad**: Configuración de manejo de errores para evitar exponer información sensible

## Ejecución de la Aplicación

### Paso 1: Iniciar sesión y configurar tu endpoint

La autenticación es sin clave (Microsoft Entra ID), por lo que no hay clave API. Inicia sesión y configura tu endpoint de Foundry:

**Windows (Símbolo del sistema):**
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

**Por qué es necesario esto:**
- Azure AI Foundry usa Microsoft Entra ID para autenticar solicitudes de inferencia
- Autenticación sin clave significa que no hay secretos en el código fuente o ambiente
- Tu cuenta necesita el rol **Usuario de Cognitive Services OpenAI** en el recurso

### Paso 2: Compilar y ejecutar

Navega al directorio del proyecto:
```bash
cd 04-PracticalSamples/petstory
```

Compila la aplicación:
```bash
mvn clean compile
```

Inicia el servidor:
```bash
mvn spring-boot:run
```

La aplicación arrancará en `http://localhost:8080`.

### Paso 3: Probar la aplicación

1. **Abre** `http://localhost:8080` en tu navegador
2. **Describe** tu mascota en el área de texto (por ejemplo, "Un golden retriever juguetón que ama traer la pelota")
3. **Haz clic** en "Generar Historia" para obtener una historia generada por IA
4. **Alternativamente**, sube una imagen de tu mascota para generar automáticamente una descripción
5. **Visualiza** la historia creativa basada en la descripción de tu mascota

## Cómo Funciona Todo Junto

Aquí está el flujo completo cuando generas una historia de mascota:

1. **Entrada del Usuario**: Describes tu mascota en el formulario web
2. **Envío del Formulario**: El navegador envía una solicitud POST a `/generate-story`
3. **Procesamiento del Controlador**: `PetController` valida y sanitiza la entrada
4. **Llamada al Servicio de IA**: `StoryService` envía la solicitud al modelo de Azure AI Foundry
5. **Generación de la Historia**: La IA crea una historia creativa basada en la descripción
6. **Manejo de la Respuesta**: El controlador recibe la historia y la añade al modelo
7. **Renderizado de Plantilla**: Thymeleaf muestra `result.html` con la historia
8. **Visualización**: El usuario ve la historia generada en su navegador

**Flujo de Manejo de Errores:**
Si el servicio de IA falla:
1. El controlador captura la excepción
2. Genera una historia alternativa usando plantillas preescritas
3. Muestra la historia alternativa con una nota sobre la indisponibilidad de la IA
4. El usuario aún recibe una historia, asegurando una buena experiencia

## Comprendiendo la Integración de IA

### Azure AI Foundry (sin clave)
La aplicación usa Azure AI Foundry con autenticación sin clave (Microsoft Entra ID):

```java
// Autenticación sin clave - sin clave API
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Ingeniería de Prompt
El servicio usa prompts cuidadosamente diseñados para obtener buenos resultados:

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Procesamiento de Respuestas
Se extrae y valida la respuesta de la IA:

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Próximos Pasos

Para más ejemplos, ve a [Capítulo 04: Ejemplos prácticos](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Descargo de responsabilidad**:
Este documento ha sido traducido utilizando el servicio de traducción automática [Co-op Translator](https://github.com/Azure/co-op-translator). Aunque nos esforzamos por la precisión, tenga en cuenta que las traducciones automatizadas pueden contener errores o inexactitudes. El documento original en su idioma nativo debe considerarse la fuente autorizada. Para información crítica, se recomienda una traducción profesional humana. No somos responsables de cualquier malentendido o interpretación errónea que surja del uso de esta traducción.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->