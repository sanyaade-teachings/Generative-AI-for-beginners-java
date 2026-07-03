# Configuración del Entorno de Desarrollo para Generative AI para Java

> **Inicio Rápido:** Provisión tus modelos de IA en **Azure AI Foundry** como código con Bicep + `azd` en unos minutos — consulta la [Guía de Configuración de Azure AI Foundry](getting-started-azure-openai.md). La autenticación es **sin claves** (Microsoft Entra ID), así que no hay claves API que gestionar.

## Lo Que Aprenderás

- Configurar un entorno de desarrollo Java para aplicaciones de IA
- Elegir y configurar tu entorno de desarrollo preferido (primero en la nube con Codespaces, contenedor de desarrollo local o configuración local completa)
- Probar tu configuración conectándote a un modelo de Azure AI Foundry

## Tabla de Contenidos

- [Lo Que Aprenderás](#lo-que-aprenderás)
- [Introducción](#introducción)
- [Paso 1: Configura Tu Entorno de Desarrollo](#paso-1-configura-tu-entorno-de-desarrollo)
  - [Opción A: GitHub Codespaces (Recomendado)](#opción-a-github-codespaces-recomendado)
  - [Opción B: Contenedor de Desarrollo Local](#opción-b-contenedor-de-desarrollo-local)
  - [Opción C: Usa Tu Instalación Local Existente](#opción-c-usa-tu-instalación-local-existente)
- [Paso 2: Provisiona Azure AI Foundry](#paso-2-provisiona-azure-ai-foundry)
- [Paso 3: Prueba Tu Configuración](#paso-3-prueba-tu-configuración)
- [Solución de Problemas](#solución-de-problemas)
- [Resumen](#resumen)
- [Próximos Pasos](#próximos-pasos)

## Introducción

Este capítulo te guiará a través de la configuración de un entorno de desarrollo. Usaremos **Azure AI Foundry** para los modelos durante todo este curso. Provisión los modelos como código con Bicep y la CLI de desarrollador de Azure (`azd`), luego te conectas con **autenticación sin claves** (Microsoft Entra ID) — sin claves API que copiar o filtrar.

**¡No se requiere configuración local!** Puedes usar GitHub Codespaces, que proporciona un entorno completo de desarrollo en tu navegador, y desde ahí provisionar Foundry.

Usamos **Azure AI Foundry** para este curso porque es:
- **Provisionado como código** — un solo `azd up` despliega la cuenta y los despliegues del modelo
- **Sin claves** — autentícate con tu inicio de sesión Azure o una identidad administrada
- **Listo para producción** — el mismo código se ejecuta localmente y en Azure
- **Flexible** — cambia modelos cambiando un nombre de despliegue, no tu código

> **Nota**: Los despliegues de Azure AI Foundry se facturan por token (pago por uso). Consulta la [guía de configuración de Azure AI Foundry](getting-started-azure-openai.md) para detalles sobre provisión, región y costes.


## Paso 1: Configura Tu Entorno de Desarrollo

<a name="quick-start-cloud"></a>

Hemos creado un contenedor de desarrollo preconfigurado para minimizar el tiempo de configuración y asegurar que tengas todas las herramientas necesarias para este curso de Generative AI para Java. Elige tu enfoque de desarrollo preferido:

### Opciones de Configuración del Entorno:

#### Opción A: GitHub Codespaces (Recomendado)

**Comienza a programar en 2 minutos - ¡no se requiere configuración local!**

1. Haz fork de este repositorio en tu cuenta de GitHub
   > **Nota**: Si deseas editar la configuración básica, consulta la [Configuración del Contenedor de Desarrollo](../../../.devcontainer/devcontainer.json)
2. Haz clic en **Code** → pestaña **Codespaces** → **...** → **Nuevo con opciones...**
3. Usa las opciones por defecto – esto seleccionará la **Configuración del contenedor de desarrollo**: **Entorno de Desarrollo de Generative AI Java** contenedor personalizado creado para este curso
4. Haz clic en **Crear codespace**
5. Espera aproximadamente 2 minutos a que el entorno esté listo
6. Continúa al [Paso 2: Provisiona Azure AI Foundry](#paso-2-provisiona-azure-ai-foundry)

<img src="../../../translated_images/es/codespaces.9945ded8ceb431a5.webp" alt="Captura de pantalla: Submenú de Codespaces" width="50%">

<img src="../../../translated_images/es/image.833552b62eee7766.webp" alt="Captura de pantalla: Nuevo con opciones" width="50%">

<img src="../../../translated_images/es/codespaces-create.b44a36f728660ab7.webp" alt="Captura de pantalla: Opciones de creación de codespace" width="50%">


> **Beneficios de Codespaces**:
> - No se requiere instalación local
> - Funciona en cualquier dispositivo con un navegador
> - Preconfigurado con todas las herramientas y dependencias
> - 60 horas gratuitas al mes para cuentas personales
> - Entorno consistente para todos los aprendices

#### Opción B: Contenedor de Desarrollo Local

**Para desarrolladores que prefieren desarrollo local con Docker**

1. Haz fork y clona este repositorio a tu máquina local
   > **Nota**: Si deseas editar la configuración básica, consulta la [Configuración del Contenedor de Desarrollo](../../../.devcontainer/devcontainer.json)
2. Instala [Docker Desktop](https://www.docker.com/products/docker-desktop/) y [VS Code](https://code.visualstudio.com/)
3. Instala la [extensión de Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) en VS Code
4. Abre la carpeta del repositorio en VS Code
5. Cuando se te solicite, haz clic en **Reabrir en Contenedor** (o usa `Ctrl+Shift+P` → "Dev Containers: Reopen in Container")
6. Espera a que el contenedor se construya y arranque
7. Continúa al [Paso 2: Provisiona Azure AI Foundry](#paso-2-provisiona-azure-ai-foundry)

<img src="../../../translated_images/es/devcontainer.21126c9d6de64494.webp" alt="Captura de pantalla: Configuración de contenedor de desarrollo" width="50%">

<img src="../../../translated_images/es/image-3.bf93d533bbc84268.webp" alt="Captura de pantalla: Contenedor de desarrollo construido" width="50%">

#### Opción C: Usa Tu Instalación Local Existente

**Para desarrolladores con entornos Java existentes**

Requisitos previos:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html) 
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) o tu IDE preferido

Pasos:
1. Clona este repositorio a tu máquina local
2. Abre el proyecto en tu IDE
3. Continúa al [Paso 2: Provisiona Azure AI Foundry](#paso-2-provisiona-azure-ai-foundry)

> **Consejo Profesional**: Si tienes una máquina con bajas especificaciones pero quieres usar VS Code localmente, ¡usa GitHub Codespaces! Puedes conectar tu VS Code local a un Codespace alojado en la nube para tener lo mejor de ambos mundos.

<img src="../../../translated_images/es/image-2.fc0da29a6e4d2aff.webp" alt="Captura de pantalla: instancia de devcontainer local creada" width="50%">


## Paso 2: Provisiona Azure AI Foundry

Despliega los modelos de IA del curso en Azure AI Foundry como código. Desde la raíz del repositorio:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```

`azd` solicita un nombre de entorno y región, provisiona una cuenta Azure AI Foundry con despliegues `gpt-4o-mini` y `text-embedding-3-small`, y escribe el endpoint en el `.env` del ejemplo — todo con autenticación **sin claves** (sin claves API).

> **Guía completa:** Consulta la [Guía de Configuración de Azure AI Foundry](getting-started-azure-openai.md) para requisitos previos, alternativa manual (portal), recomendaciones de región y notas sobre costes y limpieza.

## Paso 3: Prueba Tu Configuración

Una vez que tus modelos Foundry estén provisionados, prueba la conexión con la app de ejemplo en [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure).

1. Abre la terminal en tu entorno de desarrollo.
2. Navega al ejemplo:
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
3. Asegúrate de estar conectado (la autenticación sin claves requiere un token):
   ```bash
   az login
   ```
   > Si ejecutaste `azd up`, el archivo `.env` con tu endpoint ya fue escrito para ti.
4. Ejecuta la aplicación:
   ```bash
   mvn clean spring-boot:run
   ```

Deberías ver una respuesta del modelo `gpt-4o-mini`.

### Entendiendo el Código de Ejemplo

El ejemplo en `examples/basic-chat-azure` es una app Spring Boot que usa **Spring AI** para conectarse a Azure AI Foundry con autenticación sin claves.

**Qué hace este código:**
- **Se conecta** a Azure AI Foundry usando tu inicio de sesión Azure (Microsoft Entra ID) — sin clave API
- **Envía** un prompt al modelo `gpt-4o-mini`
- **Recibe** y muestra la respuesta de la IA
- **Valida** que tu configuración funciona correctamente

**Dependencia clave** (en `pom.xml`):
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```

**Configuración** (`application.yml`):
```yaml
spring:
  ai:
    azure:
      openai:
        # Endpoint only - no api-key. Spring AI uses DefaultAzureCredential (keyless).
        endpoint: ${AZURE_OPENAI_ENDPOINT}
        chat:
          options:
            deployment-name: ${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}
```

## Resumen

¡Genial! Ahora tienes todo configurado:

- Modelos de Azure AI Foundry provisionados como código con Bicep + `azd`
- Tu entorno de desarrollo Java funcionando (ya sea Codespaces, contenedores de desarrollo o local)
- Conexión a Azure AI Foundry con autenticación sin claves (Microsoft Entra ID) — sin claves API
- Lo probaste todo con un ejemplo simple que se comunica con tu modelo

## Próximos Pasos

[Capítulo 3: Técnicas Básicas de Generative AI](../03-CoreGenerativeAITechniques/README.md)

## Solución de Problemas

¿Tienes problemas? Aquí los problemas y soluciones comunes:

- **¿Falla la autenticación (401/403)?** 
  - Ejecuta `az login` — la autenticación es sin claves, debes estar firmado
  - Verifica que tu cuenta tenga el rol **Cognitive Services OpenAI User** sobre el recurso
  - Si acabas de provisionar, espera un minuto para que se propague la asignación del rol

- **¿Maven no encontrado?** 
  - Si usas contenedores de desarrollo/Codespaces, Maven debe venir preinstalado
  - Para configuración local, asegúrate de tener Java 21+ y Maven 3.9+ instalados
  - Intenta `mvn --version` para verificar la instalación

- **¿`azd` no encontrado o falla la provisión?** 
  - Instala la [Azure Developer CLI](https://aka.ms/azure-dev/install) y ejecuta `azd auth login`
  - Elige una región donde `gpt-4o-mini` esté disponible (e.g. `eastus2`)
  - Consulta la [guía de Azure AI Foundry](getting-started-azure-openai.md) para detalles

- **¿El contenedor de desarrollo no inicia?** 
  - Asegúrate que Docker Desktop está corriendo (para desarrollo local)
  - Intenta reconstruir el contenedor: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **¿Errores al compilar la aplicación?**
  - Asegúrate de estar en el directorio correcto: `02-SetupDevEnvironment/examples/basic-chat-azure`
  - Intenta limpiar y recompilar: `mvn clean compile`

> **¿Necesitas ayuda?**: ¿Sigues teniendo problemas? Abre un issue en el repositorio y te ayudaremos.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Descargo de responsabilidad**:
Este documento ha sido traducido utilizando el servicio de traducción automática [Co-op Translator](https://github.com/Azure/co-op-translator). Aunque nos esforzamos por la precisión, tenga en cuenta que las traducciones automatizadas pueden contener errores o inexactitudes. El documento original en su idioma nativo debe considerarse la fuente autorizada. Para información crítica, se recomienda una traducción profesional humana. No somos responsables de cualquier malentendido o interpretación errónea que surja del uso de esta traducción.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->