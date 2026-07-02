# Chat básico con Azure AI Foundry - Ejemplo de principio a fin

Este ejemplo es una aplicación simple de Spring Boot que se conecta a un modelo de **Azure AI Foundry** usando **autenticación sin clave** (Microsoft Entra ID) y prueba tu configuración. Usa el `ChatClient` de Spring AI.

## Tabla de contenido

- [Requisitos previos](#requisitos-previos)
- [Inicio rápido](#inicio-rápido)
- [Cómo funciona la autenticación](#cómo-funciona-la-autenticación)
- [Ejecutando la aplicación](#ejecutando-la-aplicación)
  - [Usando Maven](#usando-maven)
  - [Usando VS Code](#usando-vs-code)
  - [Salida esperada](#salida-esperada)
- [Referencia de configuración](#referencia-de-configuración)
  - [Variables de entorno](#variables-de-entorno)
  - [Configuración de Spring](#configuración-de-spring)
- [Resolución de problemas](#resolución-de-problemas)
  - [Problemas comunes](#problemas-comunes)
  - [Modo de depuración](#modo-de-depuración)
- [Siguientes pasos](#siguientes-pasos)
- [Recursos](#recursos)

## Requisitos previos

Antes de ejecutar este ejemplo, asegúrate de tener:

- Un recurso de Azure AI Foundry con un despliegue `gpt-4o-mini` — provisionado con `azd up` o manualmente mediante la [guía de configuración de Azure AI Foundry](../../getting-started-azure-openai.md)
- El rol **Cognitive Services OpenAI User** en ese recurso (las plantillas Bicep asignan esto automáticamente)
- La [CLI de Azure (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), con sesión iniciada usando `az login`
- Java 21+ y Maven 3.9+

> **No se requiere clave API** — la autenticación es sin clave mediante Microsoft Entra ID.

## Inicio rápido

```bash
# 1. Navega al proyecto
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Inicia sesión para que la autenticación sin clave pueda obtener un token
az login

# 3. Configura el endpoint
#    - Si ejecutaste `azd up`, se escribió .env por ti (omite esto).
#    - De lo contrario, copia la plantilla y configura AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. Ejecuta la aplicación
mvn spring-boot:run
```

## Cómo funciona la autenticación

Este ejemplo se autentica con **Microsoft Entra ID** — no hay clave API.

Cuando solo se configura `spring.ai.azure.openai.endpoint` (y no la api-key), Spring AI crea el cliente Azure OpenAI con [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). Esta credencial automáticamente obtiene un token de tu sesión local de `az login`, o de una identidad administrada cuando se ejecuta en Azure — por lo que el mismo código funciona en ambos entornos sin cambios.

## Ejecutando la aplicación

### Usando Maven

```bash
mvn spring-boot:run
```

### Usando VS Code

1. Abre el proyecto en VS Code
2. Presiona `F5` o usa el panel "Run and Debug"
3. Selecciona la configuración "Spring Boot-BasicChatApplication"

> **Nota**: La configuración de VS Code carga automáticamente tu archivo .env

### Salida esperada

```
Starting Basic Chat with Azure OpenAI...
Environment variables loaded successfully
Connecting to Azure OpenAI...
Sending prompt: What is AI in a short sentence? Max 100 words.

AI Response:
================
AI, or Artificial Intelligence, is the simulation of human intelligence in machines programmed to think and learn like humans.
================

Success! Azure OpenAI connection is working correctly.
```

## Referencia de configuración

### Variables de entorno

| Variable | Descripción | Requerida | Ejemplo |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | URL del endpoint de Foundry (Azure OpenAI) | Sí | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Nombre del despliegue del modelo de chat | No | `gpt-4o-mini` (por defecto) |

> No existe una variable para clave API — la autenticación es sin clave (Microsoft Entra ID vía `az login`).

### Configuración de Spring

El archivo `application.yml` configura:
- **Endpoint**: `${AZURE_OPENAI_ENDPOINT}` - Desde variable de entorno
- **Despliegue**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - Desde variable de entorno con valor por defecto
- **Autenticación**: sin clave — no se establece `api-key`, por lo que Spring AI usa `DefaultAzureCredential`
- **Temperatura**: `0.7` - Controla creatividad (0.0 = determinista, 1.0 = creativo)
- **Tokens máximos**: `500` - Longitud máxima de respuesta

## Resolución de problemas

### Problemas comunes

<details>
<summary><strong>Error: 401 / "PermissionDenied" / errores de token</strong></summary>

- Ejecuta `az login` — la autenticación sin clave necesita una sesión activa para obtener token
- Verifica que tu cuenta tenga el rol **Cognitive Services OpenAI User** en el recurso
- Si acabas de asignar el rol, espera un minuto para que se propague
- Confirma que estás en el tenant/suscripción correctos (`az account show`)
</details>

<details>
<summary><strong>Error: "El endpoint no es válido" / errores de conexión</strong></summary>

- Asegúrate que `AZURE_OPENAI_ENDPOINT` es la URL base completa (e.g., `https://your-resource.openai.azure.com/`)
- Verifica la consistencia con la barra diagonal al final
- Confirma que el endpoint coincide con tu recurso provisionado (`azd env get-values`)
</details>

<details>
<summary><strong>Error: "No se encontró el despliegue"</strong></summary>

- Verifica que `AZURE_OPENAI_DEPLOYMENT` coincide con un nombre de despliegue en Azure
- Asegúrate que el modelo está desplegado y activo
- El nombre por defecto del despliegue es `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: Variables de entorno no cargan</strong></summary>

- Asegúrate que el archivo `.env` esté en el directorio raíz del proyecto (al mismo nivel que `pom.xml`)
- Intenta ejecutar `mvn spring-boot:run` en la terminal integrada de VS Code
- Verifica que la extensión de Java para VS Code esté correctamente instalada
</details>

### Modo de depuración

Para habilitar registro detallado, descomenta estas líneas en `application.yml`:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Siguientes pasos

**¡Configuración completa!** Continúa tu aprendizaje:

[Capítulo 3: Técnicas centrales de IA generativa](../../../03-CoreGenerativeAITechniques/README.md)

## Recursos

- [Documentación Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Autenticación sin clave con Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Portal Azure AI Foundry](https://ai.azure.com/)
- [Documentación Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Descargo de responsabilidad**:
Este documento ha sido traducido utilizando el servicio de traducción automática [Co-op Translator](https://github.com/Azure/co-op-translator). Aunque nos esforzamos por la precisión, tenga en cuenta que las traducciones automatizadas pueden contener errores o inexactitudes. El documento original en su idioma nativo debe considerarse la fuente autorizada. Para información crítica, se recomienda una traducción profesional humana. No somos responsables de cualquier malentendido o interpretación errónea que surja del uso de esta traducción.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->