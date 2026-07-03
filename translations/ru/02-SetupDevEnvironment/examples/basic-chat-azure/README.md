# Базовый чат с Azure AI Foundry — пример от начала до конца

Этот пример — простое приложение Spring Boot, которое подключается к модели **Azure AI Foundry** с помощью **аутентификации без ключа** (Microsoft Entra ID) и тестирует вашу настройку. Используется `ChatClient` из Spring AI.

## Содержание

- [Требования](#требования)
- [Быстрый старт](#быстрый-старт)
- [Как работает аутентификация](#как-работает-аутентификация)
- [Запуск приложения](#запуск-приложения)
  - [Использование Maven](#использование-maven)
  - [Использование VS Code](#использование-vs-code)
  - [Ожидаемый вывод](#ожидаемый-вывод)
- [Справочник по настройке](#справочник-по-настройке)
  - [Переменные окружения](#переменные-окружения)
  - [Конфигурация Spring](#конфигурация-spring)
- [Устранение неполадок](#устранение-неполадок)
  - [Распространённые проблемы](#распространённые-проблемы)
  - [Режим отладки](#режим-отладки)
- [Следующие шаги](#следующие-шаги)
- [Ресурсы](#ресурсы)

## Требования

Перед запуском этого примера убедитесь, что у вас есть:

- Ресурс Azure AI Foundry с развертыванием `gpt-4o-mini` — создайте его с помощью `azd up` или вручную через [руководство по настройке Azure AI Foundry](../../getting-started-azure-openai.md)
- Роль **Cognitive Services OpenAI User** для этого ресурса (шаблоны Bicep назначают её автоматически)
- Установленная [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli) и выполнен вход через `az login`
- Java 21+ и Maven 3.9+

> **Ключ API не требуется** — аутентификация осуществляется без ключа через Microsoft Entra ID.

## Быстрый старт

```bash
# 1. Перейдите к проекту
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Войдите в систему, чтобы аутентификация без ключа могла получить токен
az login

# 3. Настройте конечную точку
#    - Если вы запускали `azd up`, файл .env был создан для вас (пропустите этот шаг).
#    - В противном случае скопируйте шаблон и установите AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. Запустите приложение
mvn spring-boot:run
```

## Как работает аутентификация

В этом примере используется аутентификация через **Microsoft Entra ID** — ключ API не используется.

Если задан только `spring.ai.azure.openai.endpoint` (и нет api-key), Spring AI создаёт клиент Azure OpenAI с помощью [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). Этот метод автоматически получает токен из вашей локальной сессии `az login` или через управляемую учётную запись при запуске в Azure — поэтому один и тот же код работает в обеих средах без изменений.

## Запуск приложения

### Использование Maven

```bash
mvn spring-boot:run
```

### Использование VS Code

1. Откройте проект в VS Code
2. Нажмите `F5` или используйте панель "Run and Debug"
3. Выберите конфигурацию "Spring Boot-BasicChatApplication"

> **Примечание**: Конфигурация VS Code автоматически загружает ваш файл .env

### Ожидаемый вывод

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

## Справочник по настройке

### Переменные окружения

| Переменная | Описание | Обязательно | Пример |
|------------|----------|-------------|--------|
| `AZURE_OPENAI_ENDPOINT` | URL конечной точки Foundry (Azure OpenAI) | Да | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Имя развертывания модели чата | Нет | `gpt-4o-mini` (по умолчанию) |

> **Нет** переменной для ключа API — аутентификация осуществляется без ключа (Microsoft Entra ID через `az login`).

### Конфигурация Spring

В файле `application.yml` настроено:
- **Endpoint**: `${AZURE_OPENAI_ENDPOINT}` — берётся из переменной окружения
- **Deployment**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` — из переменной окружения с запасным значением
- **Auth**: без ключа — `api-key` не установлен, поэтому Spring AI использует `DefaultAzureCredential`
- **Temperature**: `0.7` — регулирует креативность (0.0 = детерминированно, 1.0 = креативно)
- **Max Tokens**: `500` — максимальная длина ответа

## Устранение неполадок

### Распространённые проблемы

<details>
<summary><strong>Ошибка: 401 / "PermissionDenied" / ошибки токена</strong></summary>

- Выполните `az login` — для аутентификации без ключа необходима активная авторизация для получения токена
- Убедитесь, что ваша учётная запись имеет роль **Cognitive Services OpenAI User** для ресурса
- Если роль была назначена только что, подождите минуту для её распространения
- Проверьте правильность выбранного тенанта/подписки (`az account show`)
</details>

<details>
<summary><strong>Ошибка: "The endpoint is not valid" / ошибки соединения</strong></summary>

- Убедитесь, что `AZURE_OPENAI_ENDPOINT` — это полный базовый URL (например, `https://your-resource.openai.azure.com/`)
- Проверьте согласованность завершающего слеша
- Подтвердите, что конечная точка соответствует вашему развернутому ресурсу (`azd env get-values`)
</details>

<details>
<summary><strong>Ошибка: "The deployment was not found"</strong></summary>

- Проверьте, что `AZURE_OPENAI_DEPLOYMENT` совпадает с именем развертывания в Azure
- Удостоверьтесь, что модель успешно развернута и активна
- Имя развертывания по умолчанию — `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: Переменные окружения не загружаются</strong></summary>

- Убедитесь, что файл `.env` находится в корневой папке проекта (на том же уровне, что и `pom.xml`)
- Попробуйте запустить `mvn spring-boot:run` в встроенном терминале VS Code
- Проверьте, что расширение Java для VS Code установлено корректно
</details>

### Режим отладки

Чтобы включить подробное логирование, раскомментируйте эти строки в `application.yml`:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Следующие шаги

**Настройка завершена!** Продолжайте обучение:

[Глава 3: Основные техники генеративного ИИ](../../../03-CoreGenerativeAITechniques/README.md)

## Ресурсы

- [Документация Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Аутентификация без ключа с Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Портал Azure AI Foundry](https://ai.azure.com/)
- [Документация Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Отказ от ответственности**:
Этот документ был переведен с использованием сервиса машинного перевода [Co-op Translator](https://github.com/Azure/co-op-translator). Несмотря на наши усилия по обеспечению точности, имейте в виду, что автоматический перевод может содержать ошибки или неточности. Оригинальный документ на его исходном языке следует считать авторитетным источником. Для получения критически важной информации рекомендуется обратиться к профессиональному человеческому переводу. Мы не несем ответственности за любые недоразумения или неправильные толкования, возникшие в результате использования этого перевода.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->