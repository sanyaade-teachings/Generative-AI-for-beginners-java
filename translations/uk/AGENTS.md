# AGENTS.md

## Огляд проекту

Це освітній репозиторій для вивчення розробки генеративного ШІ на Java. Він надає комплексний практичний курс, що охоплює великі мовні моделі (LLMs), інженерію запитів, embeddings, RAG (генерація з використанням пошуку), та Протокол контексту моделі (MCP).

**Ключові технології:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI та OpenAI SDK

**Архітектура:**
- Кілька автономних застосунків Spring Boot, організованих по розділах
- Зразкові проекти, що демонструють різні шаблони ШІ
- Готовність до GitHub Codespaces з попередньо налаштованими dev-контейнерами

## Команди для налаштування

### Попередні вимоги
- Java 21 або вище
- Maven 3.x
- Підписка Azure з розгортанням моделі Azure AI Foundry (провізія за допомогою `azd up`)
- Azure CLI (`az`) та Azure Developer CLI (`azd`), підключені для автентифікації без ключів

### Налаштування середовища

**Варіант 1: GitHub Codespaces (рекомендується)**
```bash
# Форкніть репозиторій і створіть codespace через інтерфейс GitHub
# Контейнер розробника автоматично встановить всі залежності
# Зачекайте приблизно 2 хвилини для налаштування середовища
```

**Варіант 2: Локальний dev-контейнер**
```bash
# Клонувати репозиторій
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Відкрити у VS Code з розширенням Dev Containers
# Повторно відкрити в контейнері, коли з'явиться запит
```

**Варіант 3: Локальна установка**
```bash
# Встановити залежності
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Перевірити встановлення
java -version
mvn -version
```

### Конфігурація

**Налаштування Azure AI Foundry (без ключів, рекомендується):**
```bash
# Забезпечити обліковий запис Foundry + розгортання моделей як код
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd записує examples/basic-chat-azure/.env з вашим кінцевим пунктом (без ключа)
```

**Ручне налаштування endpoint:**
```bash
# Якщо ви не використовували azd, встановіть кінцеву точку самостійно (авторизація залишається без ключа через az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Відредагуйте .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Робочий процес розробки

### Структура проекту
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

### Запуск застосунків

**Запуск Spring Boot застосунку:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Збірка проекту:**
```bash
cd [project-directory]
mvn clean install
```

**Запуск MCP Calculator Server:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# Сервер працює на http://localhost:8080
```

**Запуск прикладів клієнтів:**
```bash
# Після запуску сервера в іншому терміналі
cd 04-PracticalSamples/calculator

# Прямий MCP клієнт
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# Клієнт на базі штучного інтелекту (потрібно AZURE_OPENAI_ENDPOINT + az login)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Інтерактивний бот
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Гаряче перезавантаження
Spring Boot DevTools включено у проекти, які підтримують гаряче перезавантаження:
```bash
# Зміни у файлах Java будуть автоматично перезавантажені при збереженні
mvn spring-boot:run
```

## Інструкції для тестування

### Запуск тестів

**Запуск усіх тестів у проекті:**
```bash
cd [project-directory]
mvn test
```

**Запуск тестів з детальним виводом:**
```bash
mvn test -X
```

**Запуск конкретного тестового класу:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Структура тестування
- Тестові файли використовують JUnit 5 (Jupiter)
- Тестові класи розміщені у `src/test/java/`
- Приклади клієнтів у проекті калькулятора знаходяться у `src/test/java/com/microsoft/mcp/sample/client/`

### Ручне тестування
Багато прикладів — інтерактивні застосунки, що потребують ручного тестування:

1. Запустіть застосунок командою `mvn spring-boot:run`
2. Тестуйте кінцеві точки або взаємодійте з CLI
3. Перевіряйте, чи збігається очікувана поведінка з документацією у README.md кожного проєкту

### Тестування з Azure AI Foundry
- Увійдіть через `az login` перед запуском прикладів (автентифікація без ключів)
- Переконайтеся, що у вашого акаунту є роль Cognitive Services OpenAI User на ресурсі
- Перевірте фільтрування контенту за допомогою прикладу Responsible AI у розділі 5

## Стандарти стилю коду

### Конвенції Java
- **Версія Java:** Java 21 з сучасними можливостями
- **Стиль:** Дотримуйтесь стандартних конвенцій Java
- **Іменування:** 
  - Класи: PascalCase
  - Методи/змінні: camelCase
  - Константи: UPPER_SNAKE_CASE
  - Назви пакетів: всі в нижньому регістрі

### Шаблони Spring Boot
- Використовуйте `@Service` для бізнес-логіки
- Використовуйте `@RestController` для REST-ендпоінтів
- Конфігурація через `application.yml` або `application.properties`
- Бажано використовувати змінні середовища замість хардкодів
- Використовуйте анотацію `@Tool` для методів, що виставляються MCP

### Організація файлів
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

### Залежності
- Керується через Maven `pom.xml`
- Spring AI BOM для управління версіями
- LangChain4j для інтеграцій ШІ
- Spring Boot starter parent для залежностей Spring

### Коментарі в коді
- Додавайте JavaDoc для публічних API
- Включайте пояснювальні коментарі для складних взаємодій із ШІ
- Чітко документуйте описи інструментів MCP

## Збірка і розгортання

### Збірка проектів

**Збірка без тестів:**
```bash
mvn clean install -DskipTests
```

**Збірка з усіма перевірками:**
```bash
mvn clean install
```

**Пакування застосунку:**
```bash
mvn package
# Створює JAR у директорії target/
```

### Каталоги з виводом
- Скомпільовані класи: `target/classes/`
- Тестові класи: `target/test-classes/`
- JAR-файли: `target/*.jar`
- Артефакти Maven: `target/`

### Конфігурація для різних середовищ

**Розробка:**
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

**Продакшн:**
- Використовуйте керовану ідентичність замість `az login` для аутентифікації без ключів
- Вказуйте `AZURE_OPENAI_ENDPOINT` на ваш продакшн-ресурс Foundry
- Керуйте конфігурацією через змінні середовища або Azure Key Vault

### Обмеження розгортання
- Це освітній репозиторій із зразковими застосунками
- Не призначений для розгортання в продакшн без змін
- Зразки демонструють шаблони, які можна адаптувати для продакшну
- Див. README окремих проектів для конкретних нотаток щодо розгортання

## Додаткові примітки

### Azure AI Foundry
- **Автентифікація без ключів:** підключення через Microsoft Entra ID — нема API-ключів для керування
- **Провізія як код:** Bicep + azd (`azd up`) створюють акаунт і розгортання моделей
- Один і той самий OpenAI-сумісний код працює локально (`az login`) та в Azure (керована ідентичність)

### Робота з кількома проєктами
Кожен зразковий проект є автономним:
```bash
# Перейти до конкретного проєкту
cd 04-PracticalSamples/[project-name]

# Кожен має свій власний pom.xml і може будуватися незалежно
mvn clean install
```

### Поширені проблеми

**Несумісність версії Java:**
```bash
# Перевірте Java 21
java -version
# Оновіть JAVA_HOME, якщо потрібно
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Проблеми з завантаженням залежностей:**
```bash
# Очистіть кеш Maven і спробуйте знову
rm -rf ~/.m2/repository
mvn clean install
```

**Кінцева точка або вхід не знайдено:**
```bash
# Встановіть кінцеву точку в поточній сесії та увійдіть (без ключа)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Або використовуйте файл .env у каталозі проекту
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Порт вже використовується:**
```bash
# Spring Boot за замовчуванням використовує порт 8080
# Зміна в application.properties:
server.port=8081
```

### Підтримка кількох мов
- Документація доступна на 45+ мовах через автоматичний переклад
- Переклади у каталозі `translations/`
- Керування перекладами здійснюється через GitHub Actions

### Навчальна програма
1. Почніть з [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. Слідуйте розділам послідовно (01 → 05)
3. Виконуйте практичні приклади в кожному розділі
4. Огляньте зразкові проекти в розділі 4
5. Вивчіть принципи відповідального ШІ в розділі 5

### Контейнер розробника
Файл `.devcontainer/devcontainer.json` налаштовує:
- Розробку на Java 21
- Попередньо встановлений Maven
- Розширення VS Code для Java
- Spring Boot інструменти
- Інтеграцію GitHub Copilot
- Підтримку Docker-in-Docker
- Azure CLI

### Продуктивність
- Розгортання Azure AI Foundry мають квоти на токени/запити за хвилину
- Використовуйте оптимальний розмір пакетів для embeddings
- Розглядайте кешування для повторних викликів API
- Моніторте використання токенів для оптимізації витрат

### Примітки з безпеки
- Ніколи не комітьте `.env` файли (вже в `.gitignore`)
- Віддавайте перевагу автентифікації без ключів (Microsoft Entra ID) замість API-ключів
- Використовуйте керовані ідентичності в Azure; `az login` для локальної розробки
- Дотримуйтесь принципів відповідального ШІ, описаних у розділі 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Відмова від відповідальності**:
Цей документ було перекладено за допомогою сервісу штучного інтелекту для перекладу [Co-op Translator](https://github.com/Azure/co-op-translator). Хоча ми прагнемо до точності, будь ласка, майте на увазі, що автоматичні переклади можуть містити помилки або неточності. Оригінальний документ рідною мовою слід вважати авторитетним джерелом. Для критично важливої інформації рекомендується професійний людський переклад. Ми не несемо відповідальності за будь-які непорозуміння або неправильні тлумачення, що виникли внаслідок використання цього перекладу.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->