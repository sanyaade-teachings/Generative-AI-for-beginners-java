# Основни разговор са Azure AI Foundry - пример од почетка до краја

Овај пример је једноставна Spring Boot апликација која се повезује са **Azure AI Foundry** моделом користећи **аутоентификацију без кључа** (Microsoft Entra ID) и тестира вашу конфигурацију. Користи Spring AI-јев `ChatClient`.

## Садржај

- [Предуслови](#предуслови)
- [Брзи почетак](#брзи-почетак)
- [Како функционише аутентификација](#како-функционише-аутентификација)
- [Покретање апликације](#покретање-апликације)
  - [Коришћење maven-а](#коришћење-maven-а)
  - [Коришћење VS Code-а](#коришћење-vs-code-а)
  - [Очекивани резултат](#очекивани-резултат)
- [Референца конфигурације](#референца-конфигурације)
  - [Променљиве окружења](#променљиве-окружења)
  - [Spring конфигурација](#spring-конфигурација)
- [Решавање проблема](#решавање-проблема)
  - [Чести проблеми](#чести-проблеми)
  - [Режим за отклањање грешака](#режим-за-отклањање-грешака)
- [Следећи кораци](#следећи-кораци)
- [Ресурси](#ресурси)

## Предуслови

Пре покретања овог примера, обезбедите да имате:

- Azure AI Foundry ресурс са `gpt-4o-mini` распоређивањем — поставите га помоћу `azd up` или ручно преко [Azure AI Foundry упутства за покретање](../../getting-started-azure-openai.md)
- Ролa **Cognitive Services OpenAI User** на том ресурсу (Bicep шаблони то аутоматски додељују)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), пријављен корисником преко `az login`
- Јава 21+ и Maven 3.9+

> **Није потребан API кључ** — аутентификација је без кључа преко Microsoft Entra ID-а.

## Брзи почетак

```bash
# 1. Идите до пројекта
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Пријавите се да би безкључна аутентикација добила токен
az login

# 3. Конфигуришите крајњу тачку
#    - Ако сте покренули `azd up`, .env је написан за вас (прескочите ово).
#    - У супротном копирајте шаблон и подесите AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. Покрените апликацију
mvn spring-boot:run
```

## Како функционише аутентификација

Овај пример се аутентификује са **Microsoft Entra ID** — нема API кључ.

Када је постављен само `spring.ai.azure.openai.endpoint` (без api-key), Spring AI гради Azure OpenAI клијента са [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). Тај креденцијал аутоматски проналази токен из ваше локалне `az login` сесије или из managed identity када се извршава у Azure-у — тако да исти код ради на оба места без промена.

## Покретање апликације

### Коришћење Maven-а

```bash
mvn spring-boot:run
```

### Коришћење VS Code-а

1. Отворите пројекат у VS Code-у
2. Притисните `F5` или користите панел "Run and Debug"
3. Изаберите конфигурацију "Spring Boot-BasicChatApplication"

> **Напомена**: VS Code конфигурација аутоматски учитава ваш .env фајл

### Очекивани резултат

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

## Референца конфигурације

### Променљиве окружења

| Променљива | Опис | Обавезна | Пример |
|------------|-------|----------|--------|
| `AZURE_OPENAI_ENDPOINT` | Foundry (Azure OpenAI) URL крајње тачке | Да | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Назив распоређеног chat модела | Не | `gpt-4o-mini` (подразумевано) |

> Не постоји промењива API кључа — аутентификација је без кључа (Microsoft Entra ID преко `az login`).

### Spring конфигурација

Фајл `application.yml` конфигурише:
- **Endpoint**: `${AZURE_OPENAI_ENDPOINT}` - Из променљиве окружења
- **Deployment**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - Из променљиве окружења са резервном вредношћу
- **Auth**: без кључа — није постављен `api-key`, тако да Spring AI користи `DefaultAzureCredential`
- **Температура**: `0.7` - Контролише креативност (0.0 = детерминистички, 1.0 = креативан)
- **Макс токена**: `500` - Максимална дужина одговора

## Решавање проблема

### Чести проблеми

<details>
<summary><strong>Грешка: 401 / "PermissionDenied" / проблеми са токеном</strong></summary>

- Покрените `az login` — аутентификација без кључа захтева активну пријаву за добијање токена
- Проверите да ваш налог има ролу **Cognitive Services OpenAI User** на ресурсу
- Ако сте управо додели ролу, сачекајте минуту да се промена примени
- Потврдите да сте у исправном tenant-у/претплати (`az account show`)
</details>

<details>
<summary><strong>Грешка: "The endpoint is not valid" / проблеми са везом</strong></summary>

- Проверите да је `AZURE_OPENAI_ENDPOINT` пуни базни URL (нпр. `https://your-resource.openai.azure.com/`)
- Проверите доследност завршног слеша
- Потврдите да крајња тачка одговара вашем распоређеном ресурсу (`azd env get-values`)
</details>

<details>
<summary><strong>Грешка: "The deployment was not found"</strong></summary>

- Проверите да `AZURE_OPENAI_DEPLOYMENT` одговара називу распоређеног модела у Azure-у
- Потврдите да је модел успешно распоређен и активан
- Подразумевани назив распоређивања је `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: Променљиве окружења се не учитавају</strong></summary>

- Проверите да је ваш `.env` фајл у коренском директоријуму пројекта (на истом нивоу као `pom.xml`)
- Покушајте да покренете `mvn spring-boot:run` у уграђеном терминалу VS Code-а
- Проверите да је Java додатак у VS Code-у правилно инсталиран
</details>

### Режим за отклањање грешака

Да бисте омогућили детаљније логовање, уклоните коментаре са ових линија у `application.yml`:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Следећи кораци

**Постављање завршено!** Наставите са учењем:

[Поглавље 3: Основне технике генеративне вештачке интелигенције](../../../03-CoreGenerativeAITechniques/README.md)

## Ресурси

- [Spring AI Azure OpenAI документација](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Аутентификација без кључа помоћу Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Azure AI Foundry портал](https://ai.azure.com/)
- [Azure AI Foundry документација](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Изјава о одрицању одговорности**:
Овај документ је преведен коришћењем услуге за аутоматски превод [Co-op Translator](https://github.com/Azure/co-op-translator). Иако тежимо тачности, имајте у виду да аутоматски преводи могу садржати грешке или нетачности. Оригинални документ на његовом изворном језику треба сматрати ауторитативним извором. За критичне информације препоручује се професионални људски превод. Нисмо одговорни за било каква неспоразума или погрешна тумачења која произилазе из коришћења овог превода.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->