# Настройка среды разработки для Azure AI Foundry

> В этом руководстве описывается настройка моделей **Azure AI Foundry** для Java AI приложений из этого курса с использованием **аутентификации без ключей** (Microsoft Entra ID) — никаких ключей API для управления. Новичок в инструментах? Начните с [руководства по настройке среды разработки](./README.md).

Это руководство настраивает модели **Azure AI Foundry** для Java AI приложений этого курса. У вас есть два варианта:

- **Вариант A — Provision с помощью `azd` + Bicep (рекомендуется):** одна команда развертывает учетную запись Foundry и модели как код. Никаких кликов в портале.
- **Вариант B — Создать ресурсы вручную** в портале Azure AI Foundry.

Оба варианта используют **аутентификацию без ключей** (Microsoft Entra ID) — нет необходимости копировать или раскрывать ключи API.

## Содержание

- [Что создается](#что-создается)
- [Требования](#требования)
- [Вариант A: Provision с azd + Bicep (рекомендуется)](#option-a-provision-with-azd--bicep-recommended)
- [Вариант B: Создание ресурсов вручную](#вариант-b-создание-ресурсов-вручную)
- [Настройка вашей среды](#настройка-вашей-среды)
- [Проверка настройки](#проверка-настройки)
- [Что дальше?](#что-дальше)
- [Ресурсы](#ресурсы)
- [Дополнительные ресурсы](#дополнительные-ресурсы)

## Что создается

Bicep-шаблоны из [`infra/`](../../../02-SetupDevEnvironment/infra) развертывают:

- Учетную запись **Azure AI Foundry** (`Microsoft.CognitiveServices/accounts`, тип `AIServices`) с проектом
- Развертывание **чата** — `gpt-4o-mini`
- Развертывание **встраивания** — `text-embedding-3-small` (используется в последующих главах)
- Назначение роли **без ключей** (`Cognitive Services OpenAI User`), чтобы вы могли входить с помощью `az login` вместо управления ключами

## Требования

- [Подписка Azure](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) и [Maven 3.9+](https://maven.apache.org/download.cgi)

## Вариант A: Provision с azd + Bicep (рекомендуется)

Из папки `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# Войти в систему (для обоих инструментов)
azd auth login
az login

# Настройка учетной записи Foundry и развертывание моделей
azd up
```

`azd` запросит **имя среды** (например, `genai-java`) и **регион**. Выберите регион, где доступны `gpt-4o-mini` и `text-embedding-3-small` — например, `eastus2` или `swedencentral`.

После завершения развертывания azd:

1. Развертывает всё, что определено в [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep).
2. Выполняет postprovision hook, который записывает [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) с вашим endpoint и именами развертываний (без секретов).

> **Совет:** Повторно запускайте `azd up` для применения изменений. Запустите `azd down` чтобы удалить все и прекратить начисление затрат.

Чтобы увидеть сгенерированные настройки:

```bash
azd env get-values
```

Теперь перейдите к разделу [Проверка настройки](#проверка-настройки).

## Вариант B: Создание ресурсов вручную

Предпочитаете портал? Создайте ресурсы вручную:

1. Перейдите на [портал Azure AI Foundry](https://ai.azure.com/) и войдите.
2. **Создайте проект** (это также создаст ресурс AI Foundry). Дайте ему имя, например, `GenAIJava`.
3. В вашем проекте откройте **Models + endpoints** → **Deploy model** → **Deploy base model**.
4. Разверните **gpt-4o-mini** (имя развертывания `gpt-4o-mini`). Повторите для **text-embedding-3-small**, если хотите использовать примеры с встраиванием.
5. В разделе **Overview** скопируйте **endpoint** (например, `https://<resource>.openai.azure.com/`).
6. Предоставьте себе доступ без ключей: в ресурсе откройте **Access control (IAM)** → **Add role assignment** → назначьте **Cognitive Services OpenAI User** вашей учетной записи.

> **Возникают проблемы?** См. [документацию Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Настройка вашей среды

**Если вы использовали Вариант A (`azd up`)**, ваш файл настроек уже создан — ничего не нужно настраивать. Перейдите к [Проверке настройки](#проверка-настройки).

**Если вы использовали Вариант B (вручную)**, создайте файл `.env` для примера самостоятельно:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

Отредактируйте `.env`, указав ваш endpoint (без ключа — аутентификация без ключей):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Примечание по безопасности:** API ключ хранить не нужно. Аутентификация происходит с помощью Microsoft Entra ID через `az login` (локально) или управляемую идентичность (в Azure). Файл `.env` содержит только не секретные настройки и уже исключен из контроля версий через `.gitignore`.

## Проверка настройки

Убедитесь, что вы вошли в систему, чтобы аутентификация без ключей могла получить токен, затем запустите пример:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # если вы еще не вошли в систему
mvn clean spring-boot:run
```

Вы должны увидеть ответ от модели `gpt-4o-mini`!

> **Пользователи VS Code:** Нажмите `F5` для запуска. Приложение автоматически загрузит ваш `.env`.

> **Полный пример:** Смотрите [Пример базового чата с Azure AI Foundry](./examples/basic-chat-azure/README.md) для подробностей и устранения неполадок.

## Что дальше?

**Настройка завершена!** Теперь у вас есть:
- Azure AI Foundry с развернутыми `gpt-4o-mini` и `text-embedding-3-small`
- Аутентификация без ключей (Microsoft Entra ID) — никаких ключей для управления
- Локальный `.env` с вашим endpoint и именами развертываний
- Готовая к работе Java-среда разработки

**Продолжайте в** [Глава 3: Основные техники генеративного ИИ](../03-CoreGenerativeAITechniques/README.md), чтобы начать создание AI приложений!

## Ресурсы

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Аутентификация без ключей с Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Документация Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI Documentation](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Дополнительные ресурсы

- [Скачать VS Code](https://code.visualstudio.com/Download)
- [Получить Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Конфигурация Dev Container](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Отказ от ответственности**:
Этот документ был переведен с использованием сервиса машинного перевода [Co-op Translator](https://github.com/Azure/co-op-translator). Несмотря на наши усилия по обеспечению точности, имейте в виду, что автоматический перевод может содержать ошибки или неточности. Оригинальный документ на его исходном языке следует считать авторитетным источником. Для получения критически важной информации рекомендуется обратиться к профессиональному человеческому переводу. Мы не несем ответственности за любые недоразумения или неправильные толкования, возникшие в результате использования этого перевода.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->