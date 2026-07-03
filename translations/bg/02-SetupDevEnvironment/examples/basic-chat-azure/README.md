# Основен чат с Azure AI Foundry - пример от край до край

Това пример е проста Spring Boot апликация, която се свързва с модел на **Azure AI Foundry** чрез **автентикация без ключ** (Microsoft Entra ID) и тества вашата настройка. Използва `ChatClient` на Spring AI.

## Съдържание

- [Изисквания](#изисквания)
- [Бърз старт](#бърз-старт)
- [Как работи автентикацията](#как-работи-автентикацията)
- [Стартиране на приложението](#стартиране-на-приложението)
  - [Използване на Maven](#използване-на-maven)
  - [Използване на VS Code](#използване-на-vs-code)
  - [Очакван изход](#очакван-изход)
- [Референтна конфигурация](#референтна-конфигурация)
  - [Променливи на средата](#променливи-на-средата)
  - [Spring конфигурация](#spring-конфигурация)
- [Отстраняване на проблеми](#отстраняване-на-проблеми)
  - [Чести проблеми](#чести-проблеми)
  - [Режим на дебъг](#режим-на-дебъг)
- [Следващи стъпки](#следващи-стъпки)
- [Ресурси](#ресурси)

## Изисквания

Преди да стартирате този пример, уверете се, че имате:

- Ресурс Azure AI Foundry с разгръщане `gpt-4o-mini` — осигурете го с `azd up` или ръчно чрез [Ръководството за настройка на Azure AI Foundry](../../getting-started-azure-openai.md)
- Ролята **Cognitive Services OpenAI User** на този ресурс (Bicep шаблоните я присвояват автоматично)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), влезли с `az login`
- Java 21+ и Maven 3.9+

> **Не е необходим API ключ** — автентикацията е безключова чрез Microsoft Entra ID.

## Бърз старт

```bash
# 1. Навигирайте до проекта
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Впишете се, за да може безключовата автентикация да получи токен
az login

# 3. Конфигурирайте крайния пункт
#    - Ако сте изпълнили `azd up`, .env файлът е създаден за вас (пропуснете това).
#    - В противен случай копирайте шаблона и задайте AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. Стартирайте приложението
mvn spring-boot:run
```

## Как работи автентикацията

Този пример автентикира чрез **Microsoft Entra ID** — няма API ключ.

Когато е зададен само `spring.ai.azure.openai.endpoint` (и няма api-key), Spring AI изгражда клиента на Azure OpenAI с [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). Този credential автоматично намира токен от вашата сесия `az login` локално или от управлявана идентичност при стартиране в Azure — така същият код работи и на двете места без промени.

## Стартиране на приложението

### Използване на Maven

```bash
mvn spring-boot:run
```

### Използване на VS Code

1. Отворете проекта във VS Code
2. Натиснете `F5` или използвайте панела "Run and Debug"
3. Изберете конфигурацията "Spring Boot-BasicChatApplication"

> **Забележка**: Конфигурацията на VS Code автоматично зарежда вашия .env файл

### Очакван изход

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

## Референтна конфигурация

### Променливи на средата

| Променлива | Описание | Задължително | Пример |
|------------|-----------|------------|--------|
| `AZURE_OPENAI_ENDPOINT` | URL на крайна точка Foundry (Azure OpenAI) | Да | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Име на разгръщане на чат модел | Не | `gpt-4o-mini` (по подразбиране) |

> Няма променлива за API ключ — автентикацията е безключова (Microsoft Entra ID чрез `az login`).

### Spring конфигурация

Файлът `application.yml` конфигурира:
- **Endpoint**: `${AZURE_OPENAI_ENDPOINT}` - От променлива на средата
- **Deployment**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - От променлива на средата с резервна стойност
- **Auth**: безключова — няма зададен `api-key`, така че Spring AI използва `DefaultAzureCredential`
- **Temperature**: `0.7` - Контролира креативността (0.0 = детерминиран, 1.0 = креативен)
- **Max Tokens**: `500` - Максимална дължина на отговора

## Отстраняване на проблеми

### Чести проблеми

<details>
<summary><strong>Грешка: 401 / "PermissionDenied" / грешки с токен</strong></summary>

- Изпълнете `az login` — безключовата автентикация изисква активен вход за получаване на токен
- Проверкете дали акаунтът ви има ролята **Cognitive Services OpenAI User** на ресурса
- Ако току-що сте присвоили ролята, изчакайте минута за разпространение
- Потвърдете, че сте в правилния tenant/абонамент (`az account show`)
</details>

<details>
<summary><strong>Грешка: "The endpoint is not valid" / грешки при връзка</strong></summary>

- Уверете се, че `AZURE_OPENAI_ENDPOINT` е пълният базов URL (напр. `https://your-resource.openai.azure.com/`)
- Проверете консистентност на завършващия наклонена черта
- Потвърдете, че крайният адрес съвпада с вашия ресурс (`azd env get-values`)
</details>

<details>
<summary><strong>Грешка: "The deployment was not found"</strong></summary>

- Проверете дали `AZURE_OPENAI_DEPLOYMENT` съвпада с име на разгръщане в Azure
- Проверете, че моделът е успешно разположен и активен
- По подразбиране името на разгръщането е `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: Променливите на средата не се зареждат</strong></summary>

- Уверете се, че вашият `.env` файл е в кореновата директория на проекта (на същото ниво като `pom.xml`)
- Опитайте да стартирате `mvn spring-boot:run` във вградения терминал на VS Code
- Проверете дали разширението Java за VS Code е инсталирано правилно
</details>

### Режим на дебъг

За да активирате подробен лог, разкоментирайте тези редове във `application.yml`:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Следващи стъпки

**Настройката е завършена!** Продължете своето обучение:

[Глава 3: Основни техники за генеративен AI](../../../03-CoreGenerativeAITechniques/README.md)

## Ресурси

- [Spring AI документация за Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Безключова автентикация с Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Портал Azure AI Foundry](https://ai.azure.com/)
- [Документация Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Отказ от отговорност**:
Този документ е преведен с помощта на AI преводачески услуга [Co-op Translator](https://github.com/Azure/co-op-translator). Въпреки че се стремим към точност, моля имайте предвид, че автоматизираните преводи могат да съдържат грешки или неточности. Оригиналният документ на неговия роден език трябва да се счита за авторитетен източник. За критична информация се препоръчва професионален човешки превод. Ние не носим отговорност за каквито и да е недоразумения или неправилни тълкувания, произтичащи от използването на този превод.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->