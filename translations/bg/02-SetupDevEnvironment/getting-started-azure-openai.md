# Настройване на средата за разработка за Azure AI Foundry

> Това ръководство настройва модели на **Azure AI Foundry** за Java AI приложенията в този курс, използвайки **автентикация без ключове** (Microsoft Entra ID) — без нужда от управление на API ключове. Нови сте с инструментите? Започнете с [ръководството за среда за разработка](./README.md).

Това ръководство настройва модели на **Azure AI Foundry** за Java AI приложенията в този курс. Имате два варианта:

- **Опция A — Provision с `azd` + Bicep (препоръчително):** една команда разгръща Foundry акаунта и моделите като код. Без кликане в портала.
- **Опция B — Създаване на ресурси ръчно** в портала Azure AI Foundry.

И двата варианта използват **автентикация без ключове** (Microsoft Entra ID) — няма нужда да копирате или излагате API ключове.

## Съдържание

- [Какво се създава](#какво-се-създава)
- [Изисквания](#изисквания)
- [Опция A: Provision с azd + Bicep (Препоръчително)](#option-a-provision-with-azd--bicep-recommended)
- [Опция B: Създаване на ресурси ръчно](#опция-b-създаване-на-ресурси-ръчно)
- [Конфигуриране на вашата среда](#конфигуриране-на-вашата-среда)
- [Тестване на вашата настройка](#тестване-на-вашата-настройка)
- [Какво следва?](#какво-следва)
- [Ресурси](#ресурси)
- [Допълнителни ресурси](#допълнителни-ресурси)

## Какво се създава

Bicep шаблоните в [`infra/`](../../../02-SetupDevEnvironment/infra) предоставят:

- **Azure AI Foundry** акаунт (`Microsoft.CognitiveServices/accounts`, тип `AIServices`) с проект
- Разгръщане на **чат** — `gpt-4o-mini`
- Разгръщане на **embedding** — `text-embedding-3-small` (използва се в по-късните глави)
- **Назначение на роля без ключ** (`Cognitive Services OpenAI User`), за да влизате с `az login` вместо да управлявате ключове

## Изисквания

- [Абонамент за Azure](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) и [Maven 3.9+](https://maven.apache.org/download.cgi)

## Опция A: Provision с azd + Bicep (Препоръчително)

От папката `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# Вход (и за двата инструмента)
azd auth login
az login

# Осигуряване на Foundry акаунт + разгръщане на модели
azd up
```

`azd` ще ви попита за **име на среда** (например `genai-java`) и **регион**. Изберете регион, където са налични `gpt-4o-mini` и `text-embedding-3-small` — например `eastus2` или `swedencentral`.

След като разгръщането приключи, azd:

1. Разгръща всичко, дефинирано в [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep).
2. Изпълнява постразгръщащ hook, който записва файла [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) с вашия endpoint и имена на разгръщания (без тайни).

> **Съвет:** Пускайте `azd up` всеки път, когато искате да приложите промени. Използвайте `azd down`, за да изтриете всичко и спрете натрупването на разходи.

За да видите генерираните настройки:

```bash
azd env get-values
```

Сега прескочете към [Тестване на вашата настройка](#тестване-на-вашата-настройка).

## Опция B: Създаване на ресурси ръчно

Предпочитате портала? Създайте ресурсите ръчно:

1. Отидете в [Azure AI Foundry портала](https://ai.azure.com/) и влезте.
2. **Създайте проект** (то също създава ресурс AI Foundry). Дайте му име, например `GenAIJava`.
3. В проекта си отворете **Модели + крайни точки** → **Deploy model** → **Deploy base model**.
4. Разгърнете **gpt-4o-mini** (име на разгръщане `gpt-4o-mini`). Повторете за **text-embedding-3-small**, ако искате embedding примерите.
5. От **Overview** копирайте **крайната точка** (например `https://<resource>.openai.azure.com/`).
6. Дайте си безключов достъп: на ресурса отворете **Access control (IAM)** → **Add role assignment** → назначете **Cognitive Services OpenAI User** на вашия акаунт.

> **Все още имате проблем?** Разгледайте [документацията на Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Конфигуриране на вашата среда

**Ако сте използвали Опция A (`azd up`)**, файлът с настройки вече е записан — няма какво да конфигурирате. Продължете към [Тестване на вашата настройка](#тестване-на-вашата-настройка).

**Ако сте използвали Опция B (ръчно)**, създайте `.env` файла за примера сами:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

Редактирайте `.env` с вашия endpoint (без ключ — автентикацията е без ключ):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Забележка за сигурността:** Няма API ключ, който да се съхранява. Автентикирате се с Microsoft Entra ID чрез `az login` (локално) или управляван идентификатор (в Azure). Файлът `.env` съдържа само настройки без тайни и вече е защитен от `.gitignore`.

## Тестване на вашата настройка

Уверете се, че сте влезли, за да автентикацията без ключове може да получи токен, след това стартирайте примера:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # ако вече не сте влезли в системата
mvn clean spring-boot:run
```

Трябва да видите отговор от модела `gpt-4o-mini`!

> **Потребители на VS Code:** Натиснете `F5`, за да стартирате. Приложението зарежда `.env` автоматично.

> **Пълен пример:** Вижте [Basic Chat с Azure AI Foundry пример](./examples/basic-chat-azure/README.md) за подробности и отстраняване на проблеми.

## Какво следва?

**Настройката е завършена!** Вече имате:
- Azure AI Foundry с разгърнати `gpt-4o-mini` и `text-embedding-3-small`
- Автентикация без ключове (Microsoft Entra ID) — без ключове за управление
- Локален `.env` с вашия endpoint и имена на разгръщания
- Среда за разработка на Java, готова за работа

**Продължете към** [Глава 3: Основни техники за генеративен AI](../03-CoreGenerativeAITechniques/README.md), за да започнете да изграждате AI приложения!

## Ресурси

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Автентикация без ключове с Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Документация за Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI документация](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Допълнителни ресурси

- [Изтеглете VS Code](https://code.visualstudio.com/Download)
- [Вземете Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Конфигурация на Dev Container](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Отказ от отговорност**:
Този документ е преведен с помощта на AI преводачески услуга [Co-op Translator](https://github.com/Azure/co-op-translator). Въпреки че се стремим към точност, моля имайте предвид, че автоматизираните преводи могат да съдържат грешки или неточности. Оригиналният документ на неговия роден език трябва да се счита за авторитетен източник. За критична информация се препоръчва професионален човешки превод. Ние не носим отговорност за каквито и да е недоразумения или неправилни тълкувания, произтичащи от използването на този превод.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->