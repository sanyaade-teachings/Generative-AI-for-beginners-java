# Configuración del Entorno de Desarrollo para Azure AI Foundry

> Esta guía configura los modelos de **Azure AI Foundry** para las aplicaciones de IA en Java de este curso, usando autenticación **sin claves** (Microsoft Entra ID) — no hay claves de API que gestionar. ¿Nuevo en estas herramientas? Comienza con la [guía del entorno de desarrollo](./README.md).

Esta guía configura los modelos de **Azure AI Foundry** para las aplicaciones de IA en Java de este curso. Tienes dos opciones:

- **Opción A — Provisionar con `azd` + Bicep (recomendado):** un comando despliega la cuenta de Foundry y los modelos como código. Sin hacer clic en el portal.
- **Opción B — Crear los recursos manualmente** en el portal de Azure AI Foundry.

Ambas opciones usan autenticación **sin claves** (Microsoft Entra ID) — no hay claves de API que copiar o exponer.

## Tabla de Contenidos

- [Qué se crea](#qué-se-crea)
- [Requisitos Previos](#requisitos-previos)
- [Opción A: Provisionar con azd + Bicep (Recomendado)](#option-a-provision-with-azd--bicep-recommended)
- [Opción B: Crear Recursos Manualmente](#opción-b-crear-recursos-manualmente)
- [Configura Tu Entorno](#configura-tu-entorno)
- [Prueba Tu Configuración](#prueba-tu-configuración)
- [¿Qué Sigue?](#qué-sigue)
- [Recursos](#recursos)
- [Recursos Adicionales](#recursos-adicionales)

## Qué se crea

Las plantillas Bicep en [`infra/`](../../../02-SetupDevEnvironment/infra) crean:

- Una cuenta de **Azure AI Foundry** (`Microsoft.CognitiveServices/accounts`, tipo `AIServices`) con un proyecto
- Un despliegue de **chat** — `gpt-4o-mini`
- Un despliegue de **embedding** — `text-embedding-3-small` (usada en capítulos posteriores)
- Una **asignación de rol sin clave** (`Cognitive Services OpenAI User`) para que inicies sesión con `az login` en lugar de administrar claves

## Requisitos Previos

- Una [suscripción de Azure](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) y [Maven 3.9+](https://maven.apache.org/download.cgi)

## Opción A: Provisionar con azd + Bicep (Recomendado)

Desde la carpeta `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# Iniciar sesión (ambas herramientas)
azd auth login
az login

# Proporcionar la cuenta Foundry + despliegues de modelos
azd up
```

`azd` solicitará un **nombre de entorno** (por ejemplo `genai-java`) y una **región**. Elige una región donde estén disponibles `gpt-4o-mini` y `text-embedding-3-small` — por ejemplo, `eastus2` o `swedencentral`.

Cuando finalice la provisión, azd:

1. Despliega todo lo definido en [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep).
2. Ejecuta un hook postprovisionamiento que escribe [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) con tu endpoint y nombres de despliegue (sin secretos).

> **Consejo:** Ejecuta `azd up` nuevamente en cualquier momento para aplicar cambios. Ejecuta `azd down` para eliminar todo y dejar de generar costos.

Para ver los ajustes generados:

```bash
azd env get-values
```

Ahora ve a [Prueba Tu Configuración](#prueba-tu-configuración).

## Opción B: Crear Recursos Manualmente

¿Prefieres el portal? Crea los recursos manualmente:

1. Ve al [portal de Azure AI Foundry](https://ai.azure.com/) e inicia sesión.
2. **Crea un proyecto** (esto también crea un recurso de AI Foundry). Ponle un nombre como `GenAIJava`.
3. En tu proyecto, abre **Modelos + endpoints** → **Desplegar modelo** → **Desplegar modelo base**.
4. Despliega **gpt-4o-mini** (nombre de despliegue `gpt-4o-mini`). Repite para **text-embedding-3-small** si deseas los ejemplos de embedding.
5. Desde **Resumen**, copia el **endpoint** (por ejemplo `https://<recurso>.openai.azure.com/`).
6. Concede acceso sin clave: en el recurso, abre **Control de acceso (IAM)** → **Agregar asignación de rol** → asigna **Cognitive Services OpenAI User** a tu cuenta.

> **¿Todavía tienes problemas?** Consulta la [documentación de Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Configura Tu Entorno

**Si usaste la Opción A (`azd up`)**, tu archivo de configuración ya está creado — no necesitas hacer nada más. Salta a [Prueba Tu Configuración](#prueba-tu-configuración).

**Si usaste la Opción B (manual)**, crea tú mismo el archivo `.env` del ejemplo:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

Edita `.env` con tu endpoint (sin clave — la autenticación es sin clave):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Nota de seguridad:** No hay clave API que almacenar. Autenticas con Microsoft Entra ID mediante `az login` (localmente) o una identidad administrada (en Azure). El archivo `.env` contiene solo configuraciones no secretas y ya está cubierto por `.gitignore`.

## Prueba Tu Configuración

Asegúrate de haber iniciado sesión para que la autenticación sin clave obtenga un token, luego ejecuta el ejemplo:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # si aún no has iniciado sesión
mvn clean spring-boot:run
```

¡Deberías ver una respuesta del modelo `gpt-4o-mini`!

> **Usuarios de VS Code:** Presiona `F5` para ejecutar. La aplicación carga tu `.env` automáticamente.

> **Ejemplo completo:** Consulta el [Ejemplo Básico de Chat con Azure AI Foundry](./examples/basic-chat-azure/README.md) para detalles y solución de problemas.

## ¿Qué Sigue?

**¡Configuración completa!** Ahora tienes:
- Azure AI Foundry con `gpt-4o-mini` y `text-embedding-3-small` desplegados
- Autenticación sin claves (Microsoft Entra ID) — sin claves que gestionar
- Un archivo `.env` local con tu endpoint y nombres de despliegue
- Un entorno de desarrollo en Java listo para usar

**Continúa a** [Capítulo 3: Técnicas Básicas de IA Generativa](../03-CoreGenerativeAITechniques/README.md) para comenzar a construir aplicaciones de IA.

## Recursos

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Autenticación sin claves con Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Documentación de Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)
- [Documentación Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Recursos Adicionales

- [Descargar VS Code](https://code.visualstudio.com/Download)
- [Obtener Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Configuración del contenedor de desarrollo](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Descargo de responsabilidad**:
Este documento ha sido traducido utilizando el servicio de traducción automática [Co-op Translator](https://github.com/Azure/co-op-translator). Aunque nos esforzamos por la precisión, tenga en cuenta que las traducciones automatizadas pueden contener errores o inexactitudes. El documento original en su idioma nativo debe considerarse la fuente autorizada. Para información crítica, se recomienda una traducción profesional humana. No somos responsables de cualquier malentendido o interpretación errónea que surja del uso de esta traducción.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->