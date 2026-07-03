# Налаштування середовища розробки для Azure AI Foundry

> Цей посібник налаштовує моделі **Azure AI Foundry** для програм Java AI цього курсу, використовуючи автентифікацію **без ключів** (Microsoft Entra ID) — без необхідності керувати API ключами. Ви новачок у цьому інструментарії? Почніть з [посібника зі створення середовища розробки](./README.md).

Цей посібник налаштовує моделі **Azure AI Foundry** для програм Java AI цього курсу. У вас є два варіанти:

- **Варіант A — Розгорнути за допомогою `azd` + Bicep (рекомендовано):** одна команда розгортає обліковий запис Foundry та моделі як код. Без кліків у порталі.
- **Варіант B — Створити ресурси вручну** у порталі Azure AI Foundry.

Обидва варіанти використовують автентифікацію **без ключів** (Microsoft Entra ID) — немає API ключів для копіювання чи витоку.

## Зміст

- [Що створюється](#що-створюється)
- [Вимоги](#вимоги)
- [Варіант A: Розгортання з azd + Bicep (Рекомендовано)](#option-a-provision-with-azd--bicep-recommended)
- [Варіант B: Створення ресурсів вручну](#варіант-b-створення-ресурсів-вручну)
- [Налаштування середовища](#налаштування-середовища)
- [Перевірка налаштувань](#перевірка-налаштувань)
- [Що далі?](#що-далі)
- [Ресурси](#ресурси)
- [Додаткові ресурси](#додаткові-ресурси)

## Що створюється

Шаблони Bicep з папки [`infra/`](../../../02-SetupDevEnvironment/infra) створюють:

- Обліковий запис **Azure AI Foundry** (`Microsoft.CognitiveServices/accounts`, тип `AIServices`) з проектом
- Розгортання для **чат** — `gpt-4o-mini`
- Розгортання для **векторного уявлення (embedding)** — `text-embedding-3-small` (використовується в пізніших розділах)
- Призначення ролі **без ключів** (`Cognitive Services OpenAI User`), щоб ви могли входити в систему через `az login` замість керування ключами

## Вимоги

- [Підписка Azure](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) та [Maven 3.9+](https://maven.apache.org/download.cgi)

## Варіант A: Розгортання з azd + Bicep (Рекомендовано)

З папки `02-SetupDevEnvironment` виконайте команду:

```bash
cd 02-SetupDevEnvironment

# Увійти (обидва інструменти)
azd auth login
az login

# Надати обліковий запис Foundry + розгортання моделей
azd up
```

`azd` запропонує ввести **назву середовища** (наприклад, `genai-java`) і **регіон**. Оберіть регіон, де доступні `gpt-4o-mini` та `text-embedding-3-small` — наприклад, `eastus2` або `swedencentral`.

Після завершення розгортання `azd`:

1. Розгортає все, визначене у [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep).
2. Виконує postprovision хук, який записує [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) з вашим кінцевим пунктом та іменами розгортань (без секретів).

> **Порада:** повторно виконайте `azd up` у будь-який час для застосування змін. Виконайте `azd down`, щоб видалити все і припинити нарахування витрат.

Щоб побачити згенеровані налаштування:

```bash
azd env get-values
```

Тепер можете перейти до розділу [Перевірка налаштувань](#перевірка-налаштувань).

## Варіант B: Створення ресурсів вручну

Віддаєте перевагу порталу? Створіть ресурси вручну:

1. Перейдіть у [портал Azure AI Foundry](https://ai.azure.com/) та увійдіть.
2. **Створіть проект** (це також створить ресурс AI Foundry). Дайте йому ім’я, наприклад `GenAIJava`.
3. У вашому проекті відкрийте **Models + endpoints** → **Deploy model** → **Deploy base model**.
4. Розгорніть **gpt-4o-mini** (ім’я розгортання `gpt-4o-mini`). Повторіть для **text-embedding-3-small**, якщо хочете використати приклади з embedding.
5. На сторінці **Overview** скопіюйте **endpoint** (наприклад, `https://<resource>.openai.azure.com/`).
6. Надання собі доступу без ключів: на ресурсі відкрийте **Access control (IAM)** → **Add role assignment** → призначте роль **Cognitive Services OpenAI User** для свого акаунта.

> **Ще виникають проблеми?** Перегляньте [документацію Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Налаштування середовища

**Якщо ви використали Варіант A (`azd up`)**, файл налаштувань вже створено — нічого додатково налаштовувати не потрібно. Перейдіть до [Перевірка налаштувань](#перевірка-налаштувань).

**Якщо ви використали Варіант B (вручну)**, створіть файл `.env` прикладу самостійно:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

Відредагуйте `.env` із вашим endpoint (без ключа — автентифікація без ключів):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Примітка з безпеки:** Ключ API зберігати не потрібно. Ви автентифікуєтесь за допомогою Microsoft Entra ID через `az login` (локально) або керовану ідентичність (в Azure). Файл `.env` містить лише несекретні налаштування і вже внесений у `.gitignore`.

## Перевірка налаштувань

Переконайтеся, що ви увійшли в систему, щоб автентифікація без ключів могла отримати токен, потім запустіть приклад:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # якщо ви ще не увійшли в систему
mvn clean spring-boot:run
```

Ви повинні побачити відповідь від моделі `gpt-4o-mini`!

> **Користувачі VS Code:** Натисніть `F5` для запуску. Програма автоматично завантажить ваш `.env`.

> **Повний приклад:** дивіться [Basic Chat with Azure AI Foundry example](./examples/basic-chat-azure/README.md) для деталей і усунення неполадок.

## Що далі?

**Налаштування завершено!** Ви тепер маєте:
- Azure AI Foundry з розгорнутими `gpt-4o-mini` та `text-embedding-3-small`
- Автентифікацію без ключів (Microsoft Entra ID) — без необхідності керувати ключами
- Локальний `.env` з вашим endpoint і назвами розгортань
- Готове до роботи середовище розробки Java

**Продовжуйте до** [Розділу 3: Основні техніки генеративного AI](../03-CoreGenerativeAITechniques/README.md), щоб розпочати створення AI-додатків!

## Ресурси

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Автентифікація без ключів із Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Документація Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)
- [Документація Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Додаткові ресурси

- [Завантажити VS Code](https://code.visualstudio.com/Download)
- [Отримати Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Конфігурація Dev Container](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Відмова від відповідальності**:
Цей документ було перекладено за допомогою сервісу штучного інтелекту для перекладу [Co-op Translator](https://github.com/Azure/co-op-translator). Хоча ми прагнемо до точності, будь ласка, майте на увазі, що автоматичні переклади можуть містити помилки або неточності. Оригінальний документ рідною мовою слід вважати авторитетним джерелом. Для критично важливої інформації рекомендується професійний людський переклад. Ми не несемо відповідальності за будь-які непорозуміння або неправильні тлумачення, що виникли внаслідок використання цього перекладу.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->