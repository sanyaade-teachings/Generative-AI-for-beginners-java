# AGENTS.md

## Преглед на проекта

Това е образователно хранилище за обучение на разработка на Генеративен AI с Java. То предоставя изчерпателен практически курс, обхващащ Големи езикови модели (LLMs), инженеринг на подсказки, вграждания, RAG (генерация с подпомагане от извличане) и протокола за контекст на модела (MCP).

**Основни технологии:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI и OpenAI SDK-та

**Архитектура:**
- Несвързани Spring Boot приложения, организирани по глави
- Примерни проекти, демонстриращи различни AI модели
- Готови за GitHub Codespaces с предварително конфигурирани контейнери за разработка

## Команди за настройка

### Предварителни условия
- Java 21 или по-нова версия
- Maven 3.x
- Абонамент за Azure с разгръщане на модел в Azure AI Foundry (осигурява се с `azd up`)
- Azure CLI (`az`) и Azure Developer CLI (`azd`), вписани за автентикация без ключ

### Настройка на средата

**Опция 1: GitHub Codespaces (препоръчително)**
```bash
# Форкнете репозитория и създайте кодово пространство от GitHub UI
# Контейнерът за разработка автоматично ще инсталира всички зависимости
# Изчакайте около 2 минути за настройка на средата
```

**Опция 2: Локален контейнер за разработка**
```bash
# Клониране на хранилище
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Отворете във VS Code с разширението Dev Containers
# Отворете отново в контейнер при подкана
```

**Опция 3: Локална настройка**
```bash
# Инсталиране на зависимости
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Проверка на инсталацията
java -version
mvn -version
```

### Конфигурация

**Настройка на Azure AI Foundry (без ключ, препоръчително):**
```bash
# Осигурете Foundry акаунт + разгръщания на модели като код
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd записва examples/basic-chat-azure/.env с вашия крайна точка (без ключ)
```

**Ръчна конфигурация на крайна точка:**
```bash
# Ако не използвате azd, задайте крайна точка сами (автентикацията остава без ключ чрез az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Редактирайте .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Работен процес на разработка

### Структура на проекта
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

### Стартиране на приложения

**Стартиране на Spring Boot приложение:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Създаване на проект:**
```bash
cd [project-directory]
mvn clean install
```

**Стартиране на MCP Calculator Server:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# Сървърът работи на http://localhost:8080
```

**Изпълнение на клиентски примери:**
```bash
# След стартиране на сървъра в друг терминал
cd 04-PracticalSamples/calculator

# Директен MCP клиент
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# Клиент с изкуствен интелект (изисква AZURE_OPENAI_ENDPOINT + az login)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Интерактивен бот
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Горещо презареждане
Spring Boot DevTools е включен в проекти, които поддържат горещо презареждане:
```bash
# Промените в Java файловете ще се презареждат автоматично при запазване
mvn spring-boot:run
```

## Инструкции за тестване

### Стартиране на тестове

**Стартиране на всички тестове в проект:**
```bash
cd [project-directory]
mvn test
```

**Стартиране на тестове с разширен изход:**
```bash
mvn test -X
```

**Стартиране на конкретен тестов клас:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Структура на тестовете
- Тестовите файлове използват JUnit 5 (Jupiter)
- Тестовите класове са разположени в `src/test/java/`
- Клиентските примери в проекта calculator са в `src/test/java/com/microsoft/mcp/sample/client/`

### Ръчно тестване
Много примери са интерактивни приложения, които изискват ръчно тестване:

1. Стартирайте приложението с `mvn spring-boot:run`
2. Тествайте крайни точки или взаимодействайте с CLI
3. Проверете дали очакваното поведение съответства на документацията във всеки README.md на проекта

### Тестване с Azure AI Foundry
- Впишете се с `az login` преди да стартирате примерите (автентикация без ключ)
- Уверете се, че акаунтът ви има роля Cognitive Services OpenAI User върху ресурса
- Тестване на филтриране на съдържание с примера за отговорен AI в Глава 5

## Насоки за стил на кода

### Java конвенции
- **Версия на Java:** Java 21 с модерни функции
- **Стил:** Следвайте стандартните Java конвенции
- **Именуване:** 
  - Класове: PascalCase
  - Методи/променливи: camelCase
  - Константи: UPPER_SNAKE_CASE
  - Имена на пакети: с малки букви

### Шаблони на Spring Boot
- Използвайте `@Service` за бизнес логика
- Използвайте `@RestController` за REST крайни точки
- Конфигурация чрез `application.yml` или `application.properties`
- Предпочитат се променливи на средата над твърдо кодирани стойности
- Използвайте анотацията `@Tool` за методите, изложени чрез MCP

### Организация на файловете
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

### Зависимости
- Управляват се чрез Maven `pom.xml`
- Spring AI BOM за управление на версиите
- LangChain4j за AI интеграции
- Spring Boot стартиращ родител за Spring зависимости

### Коментари в кода
- Добавяйте JavaDoc за публични API-та
- Включвайте обяснителни коментари за сложни AI взаимодействия
- Документирайте ясно описанията на MCP инструментите

## Създаване и разгръщане

### Създаване на проекти

**Създаване без тестове:**
```bash
mvn clean install -DskipTests
```

**Създаване с всички проверки:**
```bash
mvn clean install
```

**Опаковане на приложение:**
```bash
mvn package
# Създава JAR в директория target/
```

### Папки за изходните файлове
- Компилирани класове: `target/classes/`
- Тестови класове: `target/test-classes/`
- JAR файлове: `target/*.jar`
- Maven артефакти: `target/`

### Конфигурация специфична за средата

**Разработка:**
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

**Продукция:**
- Използвайте управлявана идентичност вместо `az login` за автентикация без ключ
- Насочете `AZURE_OPENAI_ENDPOINT` към вашия продукционен ресурс на Foundry
- Управлявайте конфигурацията чрез променливи на средата или Azure Key Vault

### Бележки за разгръщане
- Това е образователно хранилище с примерни приложения
- Не е предназначено за директно производство
- Примерите демонстрират модели за адаптиране в продукция
- Вижте README файловете на отделните проекти за специфични бележки за разгръщане

## Допълнителни бележки

### Azure AI Foundry
- **Автентикация без ключ:** свързване с Microsoft Entra ID — няма нужда от ключове за API
- **Осигурена като код:** Bicep + azd (`azd up`) създават акаунта и разгръщането на модели
- Същият OpenAI-съвместим код работи локално (`az login`) и в Azure (управлявана идентичност)

### Работа с множество проекти
Всеки примерен проект е самостоятелен:
```bash
# Навигирайте до конкретен проект
cd 04-PracticalSamples/[project-name]

# Всеки има собствен pom.xml и може да се изгражда независимо
mvn clean install
```

### Чести проблеми

**Несъвпадение на версията на Java:**
```bash
# Проверете Java 21
java -version
# Актуализирайте JAVA_HOME ако е необходимо
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Проблеми при изтегляне на зависимости:**
```bash
# Изчистете кеша на Maven и опитайте отново
rm -rf ~/.m2/repository
mvn clean install
```

**Крайна точка или вход не открити:**
```bash
# Задайте крайна точка в текущата сесия и влезте (без ключ)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Или използвайте .env файл в директорията на проекта
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Портът е вече зает:**
```bash
# Spring Boot използва порт 8080 по подразбиране
# Промяна в application.properties:
server.port=8081
```

### Поддръжка на множество езици
- Документацията е налична на повече от 45 езика чрез автоматичен превод
- Преводите са в директория `translations/`
- Преводът се управлява чрез GitHub Actions workflow

### Път на обучение
1. Започнете с [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. Следвайте главите в ред (01 → 05)
3. Завършете практическите примери във всяка глава
4. Изследвайте примерните проекти в Глава 4
5. Научете за отговорния AI в Глава 5

### Контейнер за разработка
Файлът `.devcontainer/devcontainer.json` конфигурира:
- Среда за разработка с Java 21
- Предварително инсталиран Maven
- VS Code Java разширения
- Spring Boot инструменти
- Интеграция с GitHub Copilot
- Поддръжка на Docker-in-Docker
- Azure CLI

### Съображения за производителността
- Разгръщанията на Azure AI Foundry имат квоти на токени/заявки на минута
- Използвайте подходящи размери на пакети за вграждания
- Помислете за кеширане при повтарящи се API повиквания
- Следете използването на токени за оптимизиране на разходите

### Бележки за сигурността
- Никога не качвайте `.env` файлове (вече са в `.gitignore`)
- Предпочитайте автентикация без ключ (Microsoft Entra ID) пред API ключове
- Използвайте управлявани идентичности в Azure; `az login` за локална разработка
- Спазвайте насоките за отговорен AI в Глава 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Отказ от отговорност**:
Този документ е преведен с помощта на AI преводачески услуга [Co-op Translator](https://github.com/Azure/co-op-translator). Въпреки че се стремим към точност, моля имайте предвид, че автоматизираните преводи могат да съдържат грешки или неточности. Оригиналният документ на неговия роден език трябва да се счита за авторитетен източник. За критична информация се препоръчва професионален човешки превод. Ние не носим отговорност за каквито и да е недоразумения или неправилни тълкувания, произтичащи от използването на този превод.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->