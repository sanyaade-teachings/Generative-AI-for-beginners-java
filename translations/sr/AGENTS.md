# AGENTS.md

## Преглед пројекта

Ово је образовни репозиторијум за учење развоја генеративне вештачке интелигенције уз Јава. Пружа свеобухватан практичан курс који покрива велике језичке моделе (LLMs), инжењеринг упита, уграђене представке (embeddings), РАГ (Retrieval-Augmented Generation) и протокол контекста модела (Model Context Protocol, MCP).

**Кључне технологије:**
- Јава 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI и OpenAI SDK-ови

**Архитектура:**
- Више самосталних Spring Boot апликација организованих по поглављима
- Примерни пројекти који демонстрирају различите обрасце у вештачкој интелигенцији
- Припремљено за GitHub Codespaces са предконфигурисаним развојним контејнерима

## Команде за подешавање

### Захтеви
- Јава 21 или новија
- Maven 3.x
- Azure претплата са развојем Azure AI Foundry модела (постављање са `azd up`)
- Azure CLI (`az`) и Azure Developer CLI (`azd`), пријављени за аутентификацију без кључа

### Подешавање окружења

**Опција 1: GitHub Codespaces (препоручено)**
```bash
# Направите форк репозиторијума и креирајте codespace преко GitHub корисничког интерфејса
# Дев контејнер ће аутоматски инсталирати све зависности
# Сачекајте ~2 минута за подешавање окружења
```

**Опција 2: Локални развојни контејнер**
```bash
# Клонирај репозиторијум
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Отвори у VS Code-у са Dev Containers екстензијом
# Поново отвори у контејнеру када буде затражено
```

**Опција 3: Локално подешавање**
```bash
# Инсталирајте зависности
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Проверите инсталацију
java -version
mvn -version
```

### Конфигурација

**Подешавање Azure AI Foundry (безкључна, препоручена):**
```bash
# Обезбедите налог Foundry и распоређивања модела као код
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd пише examples/basic-chat-azure/.env са вашим крајњим тј. тачком (без кључа)
```

**Ручна конфигурација крајње тачке:**
```bash
# Ако нисте користили azd, сами подесите крајњу тачку (аутентификација остаје без кључа преко az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Уредите .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Развојни ток рада

### Структура пројекта
```
/
├── 01-IntroToGenAI/              # Chapter 1: Introduction
├── 02-SetupDevEnvironment/       # Chapter 2: Environment setup
│   └── examples/                 # Working examples
├── 03-CoreGenerativeAITechniques/ # Chapter 3: Core techniques
├── 04-PracticalSamples/          # Chapter 4: Sample projects
│   ├── calculator/               # MCP service example
│   ├── foundrylocal/            # Local model integration
│   └── petstory/                # Multi-modal app
├── 05-ResponsibleGenAI/         # Chapter 5: Responsible AI
└── translations/                # Multi-language support
```

### Покретање апликација

**Покретање Spring Boot апликације:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Изградња пројекта:**
```bash
cd [project-directory]
mvn clean install
```

**Покретање MCP Calculator сервера:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# Сервер ради на http://localhost:8080
```

**Покретање примера клијената:**
```bash
# Након покретања сервера у другом терминалу
cd 04-PracticalSamples/calculator

# Директни MCP клијент
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# Клијент са вештачком интелигенцијом (захтева AZURE_OPENAI_ENDPOINT + az login)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Интерктивни бот
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Хот Релоад
Spring Boot DevTools је укључен у пројекте који подржавају хот релоад:
```bash
# Промене у Java фајловима ће се аутоматски поново учитати када се сачувају
mvn spring-boot:run
```

## Упутства за тестирање

### Покретање тестова

**Покрени све тестове у пројекту:**
```bash
cd [project-directory]
mvn test
```

**Покрени тестове са детаљним излазом:**
```bash
mvn test -X
```

**Покрени одређену класу тестова:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Структура тестова
- Тест датотеке користе JUnit 5 (Jupiter)
- Тест класе се налазе у `src/test/java/`
- Клијентски примери у калькулатор пројекту су у `src/test/java/com/microsoft/mcp/sample/client/`

### Ручно тестирање
Многи примери су интерактивне апликације које захтевају ручно тестирање:

1. Покрените апликацију помоћу `mvn spring-boot:run`
2. Тестирајте крајње тачке или интерагујте са CLI-јем
3. Потврдите да је очекивано понашање у складу са документацијом у README.md сваког пројекта

### Тестирање са Azure AI Foundry
- Пријавите се са `az login` пре покретања примера (безкључна аутентификација)
- Уверите се да ваш налог има улогу Cognitive Services OpenAI User на ресурсу
- Тестирајте филтрирање садржаја помоћу примера одговорне вештачке интелигенције у Поглављу 5

## Водич за стил кода

### Јава конвенције
- **Верзија Јаве:** Јава 21 са модерним функцијама
- **Стил:** Пратите стандардне Јава конвенције
- **Намена:** 
  - Класе: PascalCase
  - Методи/променљиве: camelCase
  - Константе: UPPER_SNAKE_CASE
  - Имена пакета: мала слова

### Обрасци у Spring Boot-у
- Користите `@Service` за бизнис логику
- Користите `@RestController` за REST крајње тачке
- Конфигурација преко `application.yml` или `application.properties`
- Преферирајте променљиве окружења уместо фиксних вредности
- Користите `@Tool` анотацију за методе изложене преко MCP-а

### Организација фајлова
```
src/
├── main/
│   ├── java/
│   │   └── com/microsoft/[component]/
│   │       ├── [Component]Application.java
│   │       ├── config/
│   │       ├── controller/
│   │       ├── service/
│   │       └── exception/
│   └── resources/
│       ├── application.yml
│       └── static/
└── test/
    └── java/
        └── com/microsoft/[component]/
```

### Зависности
- Управљање преко Maven `pom.xml`
- Spring AI BOM за управљање верзијама
- LangChain4j за AI интеграције
- Spring Boot стартер родитељ за Spring зависности

### Коментари у коду
- Додајте JavaDoc за јавне API-је
- Укључите објашњавајуће коментаре за сложене AI интеракције
- Јасно документајте описе MCP алата

## Изградња и имплементација

### Изградња пројеката

**Изградња без тестова:**
```bash
mvn clean install -DskipTests
```

**Изградња са свим проверницама:**
```bash
mvn clean install
```

**Паковање апликације:**
```bash
mvn package
# Креира JAR у директоријуму target/
```

### Директоријуми за излаз

- Скомпилиране класе: `target/classes/`
- Тест класе: `target/test-classes/`
- JAR фајлови: `target/*.jar`
- Maven артефакти: `target/`

### Конфигурација по окружењима

**Развој:**
```yaml
# application.yml (keyless - no api-key; auth via DefaultAzureCredential)
spring:
  ai:
    azure:
      openai:
        endpoint: ${AZURE_OPENAI_ENDPOINT}
        chat:
          options:
            deployment-name: ${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}
```

**Производња:**
- Користите управљани идентитет уместо `az login` за безкључну аутентификацију
- Поставите `AZURE_OPENAI_ENDPOINT` на ваш продукцијски Foundry ресурс
- Управљајте конфигурацијом преко променљивих окружења или Azure Key Vault-а

### Разматрања при имплементацији
- Ово је образовни репозиторijум са пример апликацијама
- Нису намењени директно за продукцијску имплементацију
- Примери показују обрасце које треба прилагодити за продукцијску употребу
- Погледајте README сваког појединачног пројекта за специфичне белешке о имплементацији

## Додатне напомене

### Azure AI Foundry
- **Безкључна аутентификација:** повезује се са Microsoft Entra ID — нема потребе за управљањем API кључевима
- **Постављено као код:** Bicep + azd (`azd up`) креира налог и развоје модела
- Исти OpenAI-компатибилан код ради локално (`az login`) и у Azure-у (управљани идентитет)

### Рад са више пројеката
Сваки примерни пројекат је самосталан:
```bash
# Идите до конкретног пројекта
cd 04-PracticalSamples/[project-name]

# Сваки има свој pom.xml и може се независно градити
mvn clean install
```

### Чести проблеми

**Неусклађеност верзије Јаве:**
```bash
# Проверите Java 21
java -version
# Ажурирајте JAVA_HOME ако је потребно
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Проблеми са преузимањем зависности:**
```bash
# Очистите Maven кеш и покушајте поново
rm -rf ~/.m2/repository
mvn clean install
```

**Крајња тачка или пријава нису пронађени:**
```bash
# Подеси крајњу тачку у тренутној сесији и пријави се (без кључа)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Или користи .env фајл у директоријуму пројекта
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Порт је већ у употреби:**
```bash
# Spring Boot подразумевано користи порт 8080
# Промена у application.properties:
server.port=8081
```

### Подршка за више језика
- Документација доступна на 45+ језика преко аутоматског превођења
- Преводи у директоријуму `translations/`
- Превод се управља преко GitHub Actions радног тока

### Пут учења
1. Почните са [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. Пратите поглавља узастопно (01 → 05)
3. Завршите практичне примере у сваком поглављу
4. Истражите примерне пројекте у Поглављу 4
5. Научите праксе одговорне вештачке интелигенције у Поглављу 5

### Развојни контејнер
Конфигурација `.devcontainer/devcontainer.json` укључује:
- Развојно окружење са Јава 21
- Прединсталирани Maven
- VS Code Java екстензије
- Spring Boot алате
- GitHub Copilot интеграцију
- Подршку за Docker-in-Docker
- Azure CLI

### Разматрања перформанси
- Azure AI Foundry развоји имају квоте за токене/захтеве по минути
- Користите одговарајуће величине пакета за embeddings
- Размислите о кеширању за поновљене API позиве
- Пратите употребу токена ради оптимизације трошкова

### Напомене о безбедности
- Никада не комитујте `.env` фајлове (већ у `.gitignore`)
- Преферирајте безкључну аутентификацију (Microsoft Entra ID) уместо API кључева
- Користите управљане идентитете у Azure-у; `az login` за локални развој
- Пратите смернице одговорне вештачке интелигенције у Поглављу 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Изјава о одрицању одговорности**:
Овај документ је преведен коришћењем услуге за аутоматски превод [Co-op Translator](https://github.com/Azure/co-op-translator). Иако тежимо тачности, имајте у виду да аутоматски преводи могу садржати грешке или нетачности. Оригинални документ на његовом изворном језику треба сматрати ауторитативним извором. За критичне информације препоручује се професионални људски превод. Нисмо одговорни за било каква неспоразума или погрешна тумачења која произилазе из коришћења овог превода.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->