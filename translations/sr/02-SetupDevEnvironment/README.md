# Постављање развојног окружења за генеративну вештачку интелигенцију за Јаву

> **Брзи почетак:** Обезбедите своје AI моделе на **Azure AI Foundry** као код помоћу Bicep + `azd` за неколико минута — погледајте [Azure AI Foundry Setup Guide](getting-started-azure-openai.md). Аутентификација је **без кључа** (Microsoft Entra ID), тако да нема потребе за управљањем API кључевима.

## Шта ћете научити

- Постављање развојног окружења за AI апликације у Јави
- Избор и конфигурација жељеног развојног окружења (cloud-first са Codespaces, локални dev контејнер или потпуна локална конфигурација)
- Тестирање поставке повезивањем са Azure AI Foundry моделом

## Садржај

- [Шта ћете научити](#шта-ћете-научити)
- [Увод](#увод)
- [Корак 1: Поставите развојно окружење](#корак-1-поставите-развојно-окружење)
  - [Опција А: GitHub Codespaces (пожељно)](#опција-а-github-codespaces-пожељно)
  - [Опција Б: Локални Dev контејнер](#опција-б-локални-dev-контејнер)
  - [Опција Ц: Користите своју постојећу локалну инсталацију](#опција-ц-користите-своју-постојећу-локалну-инсталацију)
- [Корак 2: Постављање Azure AI Foundry](#корак-2-постављање-azure-ai-foundry)
- [Корак 3: Тестирање](#корак-3-тестирајте-поставку)
- [Решавање проблема](#решавање-проблема)
- [Сажетак](#сажетак)
- [Следећи кораци](#следећи-кораци)

## Увод

Ово поглавље ће вас провести кроз постављање развојног окружења. Током курса користићемо **Azure AI Foundry** за моделе. Моделе обезбеђујете као код помоћу Bicep-a и Azure Developer CLI-а (`azd`), затим се повезујете са **аутентификацијом без кључа** (Microsoft Entra ID) — нема потребе да копирате или откривате API кључеве.

**Локална конфигурација није потребна!** Можете користити GitHub Codespaces, који пружа потпуно развојно окружење у вашем прегледачу и поставити Foundry одатле.

Користимо **Azure AI Foundry** у овом курсу јер је:
- **Обезбеђен као код** — један `azd up` покреће налог и моделско разврставање
- **Без кључа** — аутентификујете се путем Azure пријаве или управљаног идентитета
- **Спреман за продукцију** — исти код ради локално и у Azure-у
- **Флексибилан** — мењајте моделе тако што мењате име разврставања, а не свој код

> **Напомена**: Azure AI Foundry разврставања се наплаћују по броју токена (плаћање према коришћењу). Погледајте [Azure AI Foundry setup guide](getting-started-azure-openai.md) за детаље о постављању, регионима и трошковима.


## Корак 1: Поставите развојно окружење

<a name="quick-start-cloud"></a>

Креирали смо предконфигурисани развојни контејнер како бисмо смањили време поставке и обезбедили да имате све неопходне алате за овај курс генеративне AI за Јаву. Изаберите своју жељену развојну опцију:

### Опције за постављање окружења:

#### Опција А: GitHub Codespaces (пожељно)

**Почните са кодирањем за 2 минута – локална инсталација није потребна!**

1. Направите fork овог репозиторијума на свом GitHub налогу  
   > **Напомена**: Ако желите да измените основну конфигурацију, погледајте [Dev Container Configuration](../../../.devcontainer/devcontainer.json)
2. Кликните **Code** → таб **Codespaces** → **...** → **New with options...**
3. Користите подразумеване поставке – одабраће се **Dev container конфигурација**: **Generative AI Java Development Environment** прилагођени devcontainer направљен за овај курс
4. Кликните **Create codespace**
5. Сачекајте око 2 минута да се окружење припреми
6. Наставите на [Корак 2: Постављање Azure AI Foundry](#корак-2-постављање-azure-ai-foundry)

<img src="../../../translated_images/sr/codespaces.9945ded8ceb431a5.webp" alt="Снимак екрана: Подмени за Codespaces" width="50%">

<img src="../../../translated_images/sr/image.833552b62eee7766.webp" alt="Снимак екрана: New with options" width="50%">

<img src="../../../translated_images/sr/codespaces-create.b44a36f728660ab7.webp" alt="Снимак екрана: Опције за креирање codespace-а" width="50%">


> **Предности Codespaces-а**:  
> - Нема потребе за локалном инсталацијом  
> - Ради на било ком уређају са прегледачем  
> - Предконфигурисано са свим алатима и зависностима  
> - Бесплатно 60 сати месечно за личне налоге  
> - Конзистентно окружење за све ученике

#### Опција Б: Локални Dev контејнер

**За програмере који више воле локални развој помоћу Docker-а**

1. Направите fork и клонирајте овај репозиторијум на свој локални рачунар  
   > **Напомена**: Ако желите да измените основну конфигурацију, погледајте [Dev Container Configuration](../../../.devcontainer/devcontainer.json)
2. Инсталирајте [Docker Desktop](https://www.docker.com/products/docker-desktop/) и [VS Code](https://code.visualstudio.com/)
3. Инсталирајте [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) у VS Code-у
4. Отворите фасциклу репозиторијума у VS Code-у
5. Када се појави упит, кликните **Reopen in Container** (или користите `Ctrl+Shift+P` → "Dev Containers: Reopen in Container")
6. Сачекајте да се контејнер изгради и покрене
7. Наставите на [Корак 2: Постављање Azure AI Foundry](#корак-2-постављање-azure-ai-foundry)

<img src="../../../translated_images/sr/devcontainer.21126c9d6de64494.webp" alt="Снимак екрана: Подешавање Dev контејнера" width="50%">

<img src="../../../translated_images/sr/image-3.bf93d533bbc84268.webp" alt="Снимак екрана: Завршена изградња Dev контејнера" width="50%">

#### Опција Ц: Користите своју постојећу локалну инсталацију

**За програмере са постојећим Java окружењем**

Услови:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html)  
- [Maven 3.9+](https://maven.apache.org/download.cgi)  
- [VS Code](https://code.visualstudio.com) или ваш омиљени IDE

Кораци:  
1. Клонирајте овај репозиторијум на свој локални рачунар  
2. Отворите пројекат у свом IDE-ју  
3. Наставите на [Корак 2: Постављање Azure AI Foundry](#корак-2-постављање-azure-ai-foundry)

> **Професионални савет**: Ако имате рачунар са слабим спецификацијама, али желите VS Code локално, користите GitHub Codespaces! Можете повезати локални VS Code са cloud-hosted Codespace-ом за најбоље из оба света.

<img src="../../../translated_images/sr/image-2.fc0da29a6e4d2aff.webp" alt="Снимак екрана: креирани локални devcontainer примерак" width="50%">


## Корак 2: Постављање Azure AI Foundry

Деплојујте AI моделе курса на Azure AI Foundry као код. Из корена репозиторијума:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```
  
`azd` ће тражити име окружења и регион, поставиће Azure AI Foundry налог са deployment-има `gpt-4o-mini` и `text-embedding-3-small`, и уписати endpoint у пример `.env` фајл — све са **аутентификацијом без кључа** (без API кључева).

> **Потпун водич:** Погледајте [Azure AI Foundry Setup Guide](getting-started-azure-openai.md) за предуслове, алтернативу кроз портал, препоруке о региону и напомене о трошковима/чишћењу.

## Корак 3: Тестирајте поставку

Када ваши Foundry модели буду постављени, тестирате везу са пример апликацијом у [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure).

1. Отворите терминал у свом развојном окружењу.  
2. Идите у пример:  
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
  
3. Проверите да ли сте пријављени (аутентификација без кључа захтева токен):  
   ```bash
   az login
   ```
  
   > Ако сте покренули `azd up`, `.env` фајл са вашим endpoint-ом је већ уписан за вас.  
4. Покрените апликацију:  
   ```bash
   mvn clean spring-boot:run
   ```
  
Требало би да видите одговор од `gpt-4o-mini` модела.

### Разумевање пример кода

Пример у `examples/basic-chat-azure` је Spring Boot апликација која користи **Spring AI** за повезивање са Azure AI Foundry уз аутентификацију без кључа.

**Овај код ради следеће:**  
- **Повезује се** са Azure AI Foundry користећи вашу Azure пријаву (Microsoft Entra ID) — без API кључа  
- **Шаље** упит `gpt-4o-mini` моделу  
- **Прима** и приказује одговор AI-а  
- **Верификује** да је ваша поставка исправна

**Кључна зависност** (у `pom.xml`):  
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```
  
**Конфигурација** (`application.yml`):  
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
  
## Сажетак

Одлично! Сада имате све постављено:

- Обезбеђени Azure AI Foundry модели као код помоћу Bicep + `azd`  
- Покренуто ваше развојно окружење за Јаву (било да је то Codespaces, dev контејнери или локално)  
- Повезано са Azure AI Foundry уз аутентификацију без кључа (Microsoft Entra ID) — без API кључева  
- Тестирано све да ради са једноставним примером који комуницира са вашим моделом

## Следећи кораци

[Поглавље 3: Основне технике генеративне AI](../03-CoreGenerativeAITechniques/README.md)

## Решавање проблема

Имате проблема? Ево уобичајених проблема и решења:

- **Аутентификација не успева (401/403)?**  
  - Покрените `az login` — аутентификација је без кључа, па морате бити пријављени  
  - Проверите да ваш налог има **Cognitive Services OpenAI User** улогу на ресурсу  
  - Ако сте управо поставили, сачекајте минут да улога буде примењена

- **Мaven није пронађен?**  
  - Ако користите dev контејнере/Codespaces, Maven је већ инсталиран  
  - За локалну поставку, обезбедите да имате инсталиран Java 21+ и Maven 3.9+  
  - Покушајте `mvn --version` да проверите инсталацију

- **`azd` није пронађен или поставка не успева?**  
  - Инсталирајте [Azure Developer CLI](https://aka.ms/azure-dev/install) и покрените `azd auth login`  
  - Одаберите регион где је `gpt-4o-mini` доступан (нпр. `eastus2`)  
  - Погледајте [Azure AI Foundry setup guide](getting-started-azure-openai.md) за детаље

- **Dev контејнер се не покреће?**  
  - Уверите се да је Docker Desktop покренут (за локални развој)  
  - Покушајте поново изградити контејнер: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **Грешке приликом компилације апликације?**  
  - Обавезно сте у исправном директоријуму: `02-SetupDevEnvironment/examples/basic-chat-azure`  
  - Покушајте очистити и поново компајлирати: `mvn clean compile`

> **Потребна помоћ?**: И даље имате проблема? Отворите issue у репозиторијуму и помоћи ћемо вам.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Изјава о одрицању одговорности**:
Овај документ је преведен коришћењем услуге за аутоматски превод [Co-op Translator](https://github.com/Azure/co-op-translator). Иако тежимо тачности, имајте у виду да аутоматски преводи могу садржати грешке или нетачности. Оригинални документ на његовом изворном језику треба сматрати ауторитативним извором. За критичне информације препоручује се професионални људски превод. Нисмо одговорни за било каква неспоразума или погрешна тумачења која произилазе из коришћења овог превода.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->