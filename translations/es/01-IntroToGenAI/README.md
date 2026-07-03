# Introducción a la IA Generativa - Edición Java

## Lo que aprenderás

- **Fundamentos de la IA Generativa** incluyendo LLMs, ingeniería de prompts, tokens, embeddings y bases de datos vectoriales
- **Comparar herramientas de desarrollo de IA en Java** incluyendo Azure OpenAI SDK, Spring AI y OpenAI Java SDK
- **Descubrir el Protocolo de Contexto del Modelo** y su papel en la comunicación de agentes de IA

## Tabla de Contenidos

- [Introducción](#introducción)
- [Un repaso rápido sobre conceptos de IA Generativa](#un-repaso-rápido-sobre-conceptos-de-ia-generativa)
- [Revisión de la ingeniería de prompts](#revisión-de-la-ingeniería-de-prompts)
- [Tokens, embeddings y agentes](#tokens-embeddings-y-agentes)
- [Herramientas y Bibliotecas de Desarrollo de IA para Java](#herramientas-y-bibliotecas-de-desarrollo-de-ia-para-java)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [Resumen](#resumen)
- [Próximos Pasos](#próximos-pasos)

## Introducción

¡Bienvenido al primer capítulo de IA Generativa para Principiantes - Edición Java! Esta lección fundamental te introduce a los conceptos principales de la IA generativa y cómo trabajar con ellos usando Java. Aprenderás sobre los bloques esenciales para construir aplicaciones de IA, incluyendo Modelos de Lenguaje Extensos (LLMs), tokens, embeddings y agentes de IA. También exploraremos las principales herramientas de Java que usarás a lo largo de este curso.

### Un repaso rápido sobre conceptos de IA Generativa

La IA generativa es un tipo de inteligencia artificial que crea contenido nuevo, como texto, imágenes o código, basado en patrones y relaciones aprendidas a partir de datos. Los modelos de IA generativa pueden generar respuestas similares a las humanas, entender el contexto y a veces incluso crear contenido que parece humano.

Mientras desarrollas tus aplicaciones de IA en Java, trabajarás con **modelos de IA generativos** para crear contenido. Algunas capacidades de estos modelos incluyen:

- **Generación de texto**: Crear texto similar al humano para chatbots, contenidos y completado de texto.
- **Generación y análisis de imágenes**: Producir imágenes realistas, mejorar fotos y detectar objetos.
- **Generación de código**: Escribir fragmentos de código o scripts.

Existen tipos específicos de modelos optimizados para diferentes tareas. Por ejemplo, tanto los **Modelos de Lenguaje Pequeños (SLMs)** como los **Modelos de Lenguaje Extensos (LLMs)** pueden manejar la generación de texto, con los LLMs ofreciendo generalmente mejor rendimiento para tareas complejas. Para tareas relacionadas con imágenes, usarías modelos especializados de visión o modelos multi-modales.

![Figura: Tipos de modelos de IA generativa y casos de uso.](../../../translated_images/es/llms.225ca2b8a0d34473.webp)

Por supuesto, las respuestas de estos modelos no son perfectas todo el tiempo. Probablemente hayas oído sobre modelos que "alucinan" o generan información incorrecta de manera autoritaria. Pero puedes ayudar a guiar al modelo para generar mejores respuestas proporcionándole instrucciones claras y contexto. Aquí es donde entra la **ingeniería de prompts**.

#### Revisión de la ingeniería de prompts

La ingeniería de prompts es la práctica de diseñar entradas efectivas para guiar a los modelos de IA hacia las salidas deseadas. Involucra:

- **Claridad**: Hacer que las instrucciones sean claras y sin ambigüedades.
- **Contexto**: Proporcionar información de fondo necesaria.
- **Restricciones**: Especificar cualquier limitación o formato.

Algunas mejores prácticas para la ingeniería de prompts incluyen el diseño de prompts, instrucciones claras, desglosado de tareas, aprendizaje one-shot y few-shot, y ajuste de prompts. Probar distintos prompts es esencial para encontrar lo que mejor funciona para tu caso específico.

Al desarrollar aplicaciones, trabajarás con diferentes tipos de prompts:
- **Prompts del sistema**: Establecen las reglas base y el contexto para el comportamiento del modelo
- **Prompts del usuario**: Los datos de entrada de los usuarios de tu aplicación
- **Prompts del asistente**: Las respuestas del modelo basadas en los prompts del sistema y usuario

> **Aprende más**: Aprende más sobre ingeniería de prompts en el [capítulo de Ingeniería de Prompts del curso GenAI para Principiantes](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### Tokens, embeddings y agentes

Al trabajar con modelos de IA generativa, encontrarás términos como **tokens**, **embeddings**, **agentes** y **Protocolo de Contexto del Modelo (MCP)**. Aquí tienes una descripción detallada de estos conceptos:

- **Tokens**: Los tokens son la unidad más pequeña de texto en un modelo. Pueden ser palabras, caracteres o subpalabras. Los tokens se usan para representar datos de texto en un formato que el modelo puede entender. Por ejemplo, la oración "The quick brown fox jumped over the lazy dog" podría tokenizarse como ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] o ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"] dependiendo de la estrategia de tokenización.

![Figura: Ejemplo de tokens de IA generativa al dividir palabras en tokens](../../../translated_images/es/tokens.6283ed277a2ffff4.webp)

La tokenización es el proceso de dividir el texto en estas unidades más pequeñas. Esto es crucial porque los modelos operan con tokens en lugar de texto bruto. La cantidad de tokens en un prompt afecta la longitud y calidad de la respuesta del modelo, ya que los modelos tienen límites de tokens para su ventana de contexto (por ejemplo, 128K tokens para el contexto total de GPT-4o, incluyendo tanto entrada como salida).

  En Java, puedes usar bibliotecas como el OpenAI SDK para manejar la tokenización automáticamente cuando envíes solicitudes a modelos de IA.

- **Embeddings**: Los embeddings son representaciones vectoriales de tokens que capturan el significado semántico. Son representaciones numéricas (típicamente arreglos de números con punto flotante) que permiten que los modelos entiendan relaciones entre palabras y generen respuestas contextualmente relevantes. Palabras similares tienen embeddings similares, lo que permite que el modelo entienda conceptos como sinónimos y relaciones semánticas.

![Figura: Embeddings](../../../translated_images/es/embedding.398e50802c0037f9.webp)

  En Java, puedes generar embeddings usando el OpenAI SDK u otras bibliotecas que soporten la generación de embeddings. Estos embeddings son esenciales para tareas como la búsqueda semántica, donde quieres encontrar contenido similar basado en el significado en lugar de coincidencias exactas de texto.

- **Bases de datos vectoriales**: Las bases de datos vectoriales son sistemas de almacenamiento especializados optimizados para embeddings. Permiten una búsqueda eficiente por similitud y son cruciales para patrones de Generación Aumentada por Recuperación (RAG) donde necesitas encontrar información relevante en grandes conjuntos de datos basada en similitud semántica en lugar de coincidencias exactas.

![Figura: Arquitectura de base de datos vectorial mostrando cómo se almacenan y recuperan embeddings para búsqueda por similitud.](../../../translated_images/es/vector.f12f114934e223df.webp)

> **Nota**: En este curso no cubriremos bases de datos vectoriales, pero creemos que vale la pena mencionarlas ya que son comúnmente usadas en aplicaciones del mundo real.

- **Agentes y MCP**: Componentes de IA que interactúan de forma autónoma con modelos, herramientas y sistemas externos. El Protocolo de Contexto del Modelo (MCP) ofrece una forma estandarizada para que los agentes accedan de manera segura a fuentes de datos y herramientas externas. Aprende más en nuestro curso [MCP para Principiantes](https://github.com/microsoft/mcp-for-beginners).

En aplicaciones de IA en Java, usarás tokens para el procesamiento de texto, embeddings para búsqueda semántica y RAG, bases de datos vectoriales para recuperación de datos, y agentes con MCP para construir sistemas inteligentes que usan herramientas.

![Figura: cómo un prompt se convierte en respuesta—tokens, vectores, búsqueda RAG opcional, pensamiento LLM y un agente MCP todo en un flujo rápido..](../../../translated_images/es/flow.f4ef62c3052d12a8.webp)

### Herramientas y Bibliotecas de Desarrollo de IA para Java

Java ofrece excelentes herramientas para el desarrollo de IA. Hay tres bibliotecas principales que exploraremos a lo largo del curso: OpenAI Java SDK, Azure OpenAI SDK y Spring AI.

Aquí una tabla de referencia rápida que muestra qué SDK se usa en los ejemplos de cada capítulo:

| Capítulo | Ejemplo | SDK |
|---------|--------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**Enlaces de documentación del SDK:**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

El OpenAI SDK es la biblioteca oficial en Java para la API de OpenAI. Proporciona una interfaz simple y consistente para interactuar con los modelos de OpenAI, facilitando la integración de capacidades de IA en aplicaciones Java. La aplicación "Pet Story" y el ejemplo "Foundry Local" del Capítulo 4 demuestran el enfoque del SDK OpenAI con Azure AI Foundry.

#### Spring AI

Spring AI es un marco integral que aporta capacidades de IA a aplicaciones Spring, proporcionando una capa de abstracción consistente a través de diferentes proveedores de IA. Se integra sin problemas con el ecosistema Spring, convirtiéndolo en la elección ideal para aplicaciones empresariales Java que requieren capacidades de IA.

La fortaleza de Spring AI radica en su integración fluida con el ecosistema Spring, facilitando la construcción de aplicaciones de IA listas para producción con patrones familiares de Spring como inyección de dependencias, gestión de configuración y frameworks de testing. Usarás Spring AI en los capítulos 2 y 4 para construir aplicaciones que aprovechan tanto OpenAI como el Protocolo de Contexto del Modelo (MCP) a través de las bibliotecas Spring AI.

##### Protocolo de Contexto del Modelo (MCP)

El [Protocolo de Contexto del Modelo (MCP)](https://modelcontextprotocol.io/) es un estándar emergente que permite a las aplicaciones de IA interactuar de manera segura con fuentes de datos y herramientas externas. MCP proporciona una manera estandarizada para que los modelos de IA accedan a información contextual y ejecuten acciones en tus aplicaciones.

En el Capítulo 4, construirás un servicio simple de calculadora MCP que demuestra los fundamentos del Protocolo de Contexto del Modelo con Spring AI, mostrando cómo crear integraciones básicas de herramientas y arquitecturas de servicios.

#### Azure OpenAI Java SDK

La biblioteca cliente Azure OpenAI para Java es una adaptación de las APIs REST de OpenAI que provee una interfaz idiomática e integración con el resto del ecosistema del SDK de Azure. En el capítulo 3, construirás aplicaciones usando el Azure OpenAI SDK, incluyendo aplicaciones de chat, llamadas a funciones y patrones RAG (Generación Aumentada por Recuperación).

> Nota: Azure OpenAI SDK está detrás del OpenAI Java SDK en cuanto a características, por lo que para proyectos futuros, considera usar el OpenAI Java SDK.

## Resumen

¡Eso concluye los fundamentos! Ahora entiendes:

- Los conceptos centrales detrás de la IA generativa - desde LLMs e ingeniería de prompts hasta tokens, embeddings y bases de datos vectoriales
- Tus opciones de herramientas para desarrollo de IA en Java: Azure OpenAI SDK, Spring AI y OpenAI Java SDK
- Qué es el Protocolo de Contexto del Modelo y cómo permite que los agentes de IA trabajen con herramientas externas

## Próximos Pasos

[Capítulo 2: Configuración del Entorno de Desarrollo](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Descargo de responsabilidad**:
Este documento ha sido traducido utilizando el servicio de traducción automática [Co-op Translator](https://github.com/Azure/co-op-translator). Aunque nos esforzamos por la precisión, tenga en cuenta que las traducciones automatizadas pueden contener errores o inexactitudes. El documento original en su idioma nativo debe considerarse la fuente autorizada. Para información crítica, se recomienda una traducción profesional humana. No somos responsables de cualquier malentendido o interpretación errónea que surja del uso de esta traducción.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->