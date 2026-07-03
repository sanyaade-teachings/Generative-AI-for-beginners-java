# Подешавање развојног окружења за Azure AI Foundry

> Овај водич подешава **Azure AI Foundry** моделе за Java AI апликације у овом курсу, користећи **аутентификацију без кључа** (Microsoft Entra ID) — нема потребе за управљањем API кључевима. Новац у алатима? Почните са [водичем за развојно окружење](./README.md).

Овај водич подешава **Azure AI Foundry** моделе за Java AI апликације у овом курсу. Имате два пута:

- **Опција А — Прoвизија са `azd` + Bicep (препоручено):** једна команда приступа Foundry налог и моделе као код. Без кликтања у порталу.
- **Опција Б — Креирајте ресурсе ручно** у Azure AI Foundry порталу.

Оба пута користе **аутентификацију без кључа** (Microsoft Entra ID) — нема API кључева које треба копирати или пропуштати.

## Садржај

- [Шта се креира](#шта-се-креира)
- [Претпоставке](#претпоставке)
- [Опција А: Прoвизија са azd + Bicep (Препоручено)](#опција-а-прoвизија-са-azd--bicep-препоручено)
- [Опција Б: Креирање ресурса ручно](#опција-б-креирање-ресурса-ручно)
- [Подешавање вашег окружења](#подешавање-вашег-окружења)
- [Тестирање вашег подешавања](#тестирање-вашег-подешавања)
- [Шта следи?](#шта-следи)
- [Ресурси](#ресурси)
- [Додатни ресурси](#додатни-ресурси)

## Шта се креира

Bicep шаблони у [`infra/`](../../../02-SetupDevEnvironment/infra) креирају:

- **Azure AI Foundry** налог (`Microsoft.CognitiveServices/accounts`, тип `AIServices`) са пројектом
- **chat** деплојмент — `gpt-4o-mini`
- **embedding** деплојмент — `text-embedding-3-small` (користи се у каснијим поглављима)
- **додељивање улоге без кључа** (`Cognitive Services OpenAI User`) тако да се улазите помоћу `az login` уместо управљања кључевима

## Претпоставке

- [Azure претплата](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) и [Maven 3.9+](https://maven.apache.org/download.cgi)

## Опција А: Прoвизија са azd + Bicep (Препоручено)

Из фолдера `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# Пријавите се (за оба алата)
azd auth login
az login

# Обезбедите Foundry налог + распоређивања модела
azd up
```

`azd` ће тражити **име окружења** (на пример `genai-java`) и **регион**. Изаберите регион где су `gpt-4o-mini` и `text-embedding-3-small` доступни — на пример `eastus2` или `swedencentral`.

Када завршите провизију, azd:

1. Деплојује све дефинисано у [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep).
2. Покреће postprovision hook који пише [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) са вашим endpoint и именима деплојмента (без тајни).

> **Савет:** Покрените `azd up` кад год желите да примените промене. Покрените `azd down` да избришете све и зауставите трошкове.

Да бисте видели генерисане поставке:

```bash
azd env get-values
```

Сада прескочите на [Тестирање вашег подешавања](#тестирање-вашег-подешавања).

## Опција Б: Креирање ресурса ручно

Преферирате портал? Креирајте ресурсе ручно:

1. Идите на [Azure AI Foundry портал](https://ai.azure.com/) и пријавите се.
2. **Креирајте пројекат** (ово такође креира AI Foundry ресурс). Дајте му име као што је `GenAIJava`.
3. У вашем пројекту отворите **Models + endpoints** → **Deploy model** → **Deploy base model**.
4. Деплојујте **gpt-4o-mini** (име деплојмента `gpt-4o-mini`). Поновите за **text-embedding-3-small** ако желите примере укључивања.
5. Из **Overview**, копирајте **endpoint** (на пример `https://<resource>.openai.azure.com/`).
6. Дајте себи приступ без кључа: на ресурсу отворите **Access control (IAM)** → **Add role assignment** → доделите **Cognitive Services OpenAI User** вашем налогу.

> **И даље имате проблема?** Погледајте [документацију за Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Подешавање вашег окружења

**Ако сте користили Опцију А (`azd up`)**, ваш фајл са поставкама је већ написан — није потребно ништа да подешавате. Прескочите на [Тестирање вашег подешавања](#тестирање-вашег-подешавања).

**Ако сте користили Опцију Б (ручно)**, креирајте пример `.env` фајл сами:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

Уредите `.env` са вашим endpoint-ом (без кључа — аутентификација је без кључа):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Безбедносна напомена:** Нема API кључа за чување. Аутентикујете се преко Microsoft Entra ID помоћу `az login` (локално) или управљеног идентитета (у Azure). `.env` фајл садржи само неповерљиве поставке и већ је додат у `.gitignore`.

## Тестирање вашег подешавања

Уверите се да сте пријављени како би аутентификација без кључа добила токен, затим покрените пример:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # ако већ нисте пријављени
mvn clean spring-boot:run
```

Требало би да видите одговор од модела `gpt-4o-mini`!

> **Корисници VS Code:** Притисните `F5` да покренете. Апликација аутоматски учитава ваш `.env`.

> **Комплетан пример:** Погледајте [Basic Chat са Azure AI Foundry пример](./examples/basic-chat-azure/README.md) за детаље и решавање проблема.

## Шта следи?

**Подешавање је завршено!** Сада имате:
- Azure AI Foundry са деплојованим `gpt-4o-mini` и `text-embedding-3-small`
- Аутентификацију без кључа (Microsoft Entra ID) — нема кључева за управљање
- Локални `.env` са вашим endpoint и именима деплојмента
- Спремно Java развојно окружење

**Наставите даље у** [Поглавље 3: Основне технике генеративне вештачке интелигенције](../03-CoreGenerativeAITechniques/README.md) да бисте почели са израдом AI апликација!

## Ресурси

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Аутентификација без кључа са Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry документација](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI документација](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Додатни ресурси

- [Преузмите VS Code](https://code.visualstudio.com/Download)
- [Преузмите Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Конфигурација Dev Container-а](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Изјава о одрицању одговорности**:
Овај документ је преведен коришћењем услуге за аутоматски превод [Co-op Translator](https://github.com/Azure/co-op-translator). Иако тежимо тачности, имајте у виду да аутоматски преводи могу садржати грешке или нетачности. Оригинални документ на његовом изворном језику треба сматрати ауторитативним извором. За критичне информације препоручује се професионални људски превод. Нисмо одговорни за било каква неспоразума или погрешна тумачења која произилазе из коришћења овог превода.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->