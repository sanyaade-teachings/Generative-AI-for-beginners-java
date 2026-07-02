# Basic Chat with Azure AI Foundry - Кінцева демонстрація

Цей приклад — проста програма на Spring Boot, яка підключається до моделі **Azure AI Foundry** за допомогою **автентифікації без ключа** (Microsoft Entra ID) і перевіряє вашу конфігурацію. Використовується `ChatClient` зі Spring AI.

## Зміст

- [Передумови](#передумови)
- [Швидкий старт](#швидкий-старт)
- [Як працює автентифікація](#як-працює-автентифікація)
- [Запуск програми](#запуск-програми)
  - [Використання Maven](#використання-maven)
  - [Використання VS Code](#використання-vs-code)
  - [Очікуваний результат](#очікуваний-результат)
- [Довідка з конфігурації](#довідка-з-конфігурації)
  - [Змінні середовища](#змінні-середовища)
  - [Налаштування Spring](#налаштування-spring)
- [Усунення неполадок](#усунення-неполадок)
  - [Поширені проблеми](#поширені-проблеми)
  - [Режим налагодження](#режим-налагодження)
- [Наступні кроки](#наступні-кроки)
- [Ресурси](#ресурси)

## Передумови

Перш ніж запускати цей приклад, переконайтесь, що у вас є:

- Ресурс Azure AI Foundry з розгортанням `gpt-4o-mini` — створіть його за допомогою `azd up` або вручну, слідуючи [керівництву по налаштуванню Azure AI Foundry](../../getting-started-azure-openai.md)
- Роль **Користувача Cognitive Services OpenAI** для цього ресурсу (шаблони Bicep надають її автоматично)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), увійдені через `az login`
- Java 21+ та Maven 3.9+

> **Ключ API не потрібен** — автентифікація безключова через Microsoft Entra ID.

## Швидкий старт

```bash
# 1. Перейдіть до проекту
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Увійдіть, щоб автентифікація без ключа могла отримати токен
az login

# 3. Налаштуйте кінцеву точку
#    - Якщо ви запускали `azd up`, файл .env було створено для вас (пропустіть цей крок).
#    - Інакше скопіюйте шаблон і встановіть AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. Запустіть застосунок
mvn spring-boot:run
```

## Як працює автентифікація

Цей приклад автентифікується за допомогою **Microsoft Entra ID** — ключ API не використовується.

Якщо встановлено лише `spring.ai.azure.openai.endpoint` (і немає api-key), Spring AI створює клієнт Azure OpenAI із [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). Ці облікові дані автоматично отримують токен із вашої локальної сесії `az login` або з керованої ідентичності при запуску в Azure — таким чином однаковий код працює у двох середовищах без змін.

## Запуск програми

### Використання Maven

```bash
mvn spring-boot:run
```

### Використання VS Code

1. Відкрийте проект у VS Code  
2. Натисніть `F5` або використайте панель "Run and Debug"  
3. Виберіть конфігурацію "Spring Boot-BasicChatApplication"

> **Примітка**: Конфігурація VS Code автоматично завантажує ваш файл .env

### Очікуваний результат

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

## Довідка з конфігурації

### Змінні середовища

| Змінна | Опис | Обов’язково | Приклад |
|--------|-------|------------|---------|
| `AZURE_OPENAI_ENDPOINT` | URL кінцевої точки Foundry (Azure OpenAI) | Так | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Назва розгортання моделі чату | Ні | `gpt-4o-mini` (за замовчуванням) |

> Змінна для ключа API **відсутня** — автентифікація безключова (Microsoft Entra ID через `az login`).

### Налаштування Spring

Файл `application.yml` налаштовує:  
- **Endpoint**: `${AZURE_OPENAI_ENDPOINT}` - з змінної середовища  
- **Deployment**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - зі змінної середовища з резервним значенням  
- **Auth**: безключова — ключ API не заданий, тому Spring AI використовує `DefaultAzureCredential`  
- **Temperature**: `0.7` - контролює креативність (0.0 = детермінований, 1.0 = креативний)  
- **Max Tokens**: `500` - максимальна довжина відповіді  

## Усунення неполадок

### Поширені проблеми

<details>
<summary><strong>Помилка: 401 / "PermissionDenied" / проблеми з токеном</strong></summary>

- Виконайте `az login` — для безключової автентифікації потрібен активний вхід, щоб отримати токен  
- Перевірте, що ваша облікова запис має роль **Користувача Cognitive Services OpenAI** для ресурсу  
- Якщо роль щойно надана, зачекайте хвилину для поширення змін  
- Переконайтесь, що ви у правильному орендарі/підписці (`az account show`)  
</details>

<details>
<summary><strong>Помилка: "Кінцева точка недійсна" / проблеми з підключенням</strong></summary>

- Переконайтесь, що `AZURE_OPENAI_ENDPOINT` — це повний базовий URL (наприклад, `https://your-resource.openai.azure.com/`)  
- Зверніть увагу на наявність чи відсутність кінцевого слеша  
- Перевірте, чи кінцева точка співпадає з вашим створеним ресурсом (`azd env get-values`)  
</details>

<details>
<summary><strong>Помилка: "Розгортання не знайдено"</strong></summary>

- Перевірте, чи `AZURE_OPENAI_DEPLOYMENT` відповідає імені розгортання в Azure  
- Переконайтесь, що модель успішно розгорнута і активна  
- За замовчуванням ім'я розгортання — `gpt-4o-mini`  
</details>

<details>
<summary><strong>VS Code: Змінні середовища не завантажуються</strong></summary>

- Переконайтесь, що файл `.env` знаходиться у кореневому каталозі проекту (на одному рівні з `pom.xml`)  
- Спробуйте запустити `mvn spring-boot:run` у вбудованому терміналі VS Code  
- Переконайтесь, що розширення Java для VS Code встановлене правильно  
</details>

### Режим налагодження

Щоб увімкнути детальне логування, розкоментуйте ці рядки у `application.yml`:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Наступні кроки

**Налаштування завершено!** Продовжуйте навчання:

[Розділ 3: Основні методи генеративного ШІ](../../../03-CoreGenerativeAITechniques/README.md)

## Ресурси

- [Документація Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Безключова автентифікація з Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Портал Azure AI Foundry](https://ai.azure.com/)
- [Документація Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Відмова від відповідальності**:
Цей документ було перекладено за допомогою сервісу штучного інтелекту для перекладу [Co-op Translator](https://github.com/Azure/co-op-translator). Хоча ми прагнемо до точності, будь ласка, майте на увазі, що автоматичні переклади можуть містити помилки або неточності. Оригінальний документ рідною мовою слід вважати авторитетним джерелом. Для критично важливої інформації рекомендується професійний людський переклад. Ми не несемо відповідальності за будь-які непорозуміння або неправильні тлумачення, що виникли внаслідок використання цього перекладу.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->