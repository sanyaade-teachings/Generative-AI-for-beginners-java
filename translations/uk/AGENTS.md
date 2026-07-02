# AGENTS.md

## Огляд проекту

Це навчальний репозиторій для вивчення розробки генеративного ШІ на Java. Він надає всебічний практичний курс, який охоплює великі мовні моделі (LLMs), інженерію підказок, embedding-и, RAG (генерація з доповненням за допомогою пошуку) та протокол контексту моделі (MCP).

**Ключові технології:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI та OpenAI SDK

**Архітектура:**
- Декілька автономних Spring Boot додатків, організованих по главах
- Прикладні проекти, що демонструють різні AI патерни
- Підготовлено для GitHub Codespaces з попередньо налаштованими dev контейнерами

## Команди налаштування

### Попередні вимоги
- Java 21 або вище
- Maven 3.x
- Підписка Azure з розгортанням моделі Azure AI Foundry (налаштування через `azd up`)
- Azure CLI (`az`) та Azure Developer CLI (`azd`), увійшли в систему для автентифікації без ключа

### Налаштування середовища

**Варіант 1: GitHub Codespaces (рекомендовано)**
```bash
# Форкніть репозиторій і створіть codespace з інтерфейсу GitHub
# Розробницький контейнер автоматично встановить всі залежності
# Зачекайте приблизно 2 хвилини для налаштування середовища
```

**Варіант 2: Локальний Dev Container**
```bash
# Клонувати репозиторій
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Відкрити у VS Code з розширенням Dev Containers
# Перевідкрити в контейнері, коли буде запропоновано
```

**Варіант 3: Локальна установка**
```bash
# Встановіть залежності
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Перевірте встановлення
java -version
mvn -version
```


### Конфігурація

**Налаштування Azure AI Foundry (без ключів, рекомендовано):**
```bash
# Забезпечте обліковий запис Foundry та розгортання моделей як код
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd записує examples/basic-chat-azure/.env з вашою кінцевою точкою (без ключа)
```

**Ручна конфігурація кінцевої точки:**
```bash
# Якщо ви не використовували azd, встановіть кінцеву точку самостійно (авторизація залишається без ключа через az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Змініть .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
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


### Запуск додатків

**Запуск Spring Boot додатку:**
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

**Запуск клієнтських прикладів:**
```bash
# Після запуску сервера в іншому терміналі
cd 04-PracticalSamples/calculator

# Прямий клієнт MCP
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# Клієнт на базі ШІ (вимагає AZURE_OPENAI_ENDPOINT + az login)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Інтерактивний бот
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```


### Гаряче оновлення
Spring Boot DevTools включено в проекти, які підтримують гаряче оновлення:
```bash
# Зміни у файлах Java автоматично перезавантажуватимуться при збереженні
mvn spring-boot:run
```


## Інструкції для тестування

### Запуск тестів

**Запустити всі тести в проекті:**
```bash
cd [project-directory]
mvn test
```

**Запуск тестів з подробним виводом:**
```bash
mvn test -X
```

**Запуск конкретного класу тестів:**
```bash
mvn test -Dtest=CalculatorServiceTest
```


### Структура тестів
- Тестові файли використовують JUnit 5 (Jupiter)
- Тестові класи розміщені в `src/test/java/`
- Клієнтські приклади в проекті калькулятора — в `src/test/java/com/microsoft/mcp/sample/client/`

### Ручне тестування
Багато прикладів є інтерактивними додатками, які потребують ручного тестування:

1. Запустіть додаток за допомогою `mvn spring-boot:run`
2. Перевірте кінцеві точки або взаємодійте з CLI
3. Переконайтеся, що очікувана поведінка відповідає документації в README.md кожного проекту

### Тестування з Azure AI Foundry
- Увійдіть за допомогою `az login` перед запуском прикладів (автентифікація без ключа)
- Переконайтеся, що ваш обліковий запис має роль Cognitive Services OpenAI User для ресурсу
- Перевірте фільтрацію контенту з прикладом відповідального ШІ у Розділі 5

## Керівництво зі стилю коду

### Конвенції Java
- **Версія Java:** Java 21 з сучасними можливостями
- **Стиль:** Дотримуйтесь стандартних Java-конвенцій
- **Іменування:** 
  - Класи: PascalCase
  - Методи/змінні: camelCase
  - Константи: UPPER_SNAKE_CASE
  - Імена пакетів: малими літерами

### Патерни Spring Boot
- Використовуйте `@Service` для бізнес-логіки
- Використовуйте `@RestController` для REST кінцевих точок
- Конфігурація через `application.yml` або `application.properties`
- Перевага змінним середовища над жорстко закодованими значеннями
- Використовуйте анотацію `@Tool` для методів, які експонуються через MCP

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
- Spring AI BOM для керування версіями
- LangChain4j для AI інтеграцій
- Spring Boot starter parent для залежностей Spring

### Коментарі в коді
- Додавайте JavaDoc для публічних API
- Включайте пояснювальні коментарі для складних AI взаємодій
- Чітко документуйте описи MCP інструментів

## Збірка та розгортання

### Збірка проектів

**Збірка без тестів:**
```bash
mvn clean install -DskipTests
```

**Збірка з усіма перевірками:**
```bash
mvn clean install
```

**Пакування додатку:**
```bash
mvn package
# Створює JAR у директорії target/
```


### Каталоги виводу
- Скомпільовані класи: `target/classes/`
- Класи тестів: `target/test-classes/`
- JAR-файли: `target/*.jar`
- Maven артефакти: `target/`

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

**Виробництво:**
- Використовуйте керовану ідентичність замість `az login` для аутентифікації без ключів
- Вказуйте `AZURE_OPENAI_ENDPOINT` на ваш продакшн Foundry ресурс
- Керуйте конфігурацією через змінні середовища або Azure Key Vault

### Особливості розгортання
- Це навчальний репозиторій з прикладами додатків
- Не призначено для прямого розгортання у виробництво
- Приклади демонструють патерни, які можна адаптувати для виробничого використання
- Дивіться README окремих проектів для конкретних нотаток щодо розгортання

## Додаткові нотатки

### Azure AI Foundry
- **Автентифікація без ключів:** підключення через Microsoft Entra ID — немає необхідності керувати API ключами
- **Пропонується як код:** Bicep + azd (`azd up`) створюють обліковий запис та розгортання моделей
- Той самий OpenAI сумісний код працює локально (`az login`) та в Azure (керована ідентичність)

### Робота з кількома проектами
Кожен прикладний проект є автономним:
```bash
# Перейти до конкретного проекту
cd 04-PracticalSamples/[project-name]

# Кожен має власний pom.xml і може бути побудований незалежно
mvn clean install
```


### Поширені проблеми

**Несумісність версії Java:**
```bash
# Перевірити Java 21
java -version
# Оновити JAVA_HOME, якщо потрібно
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Проблеми завантаження залежностей:**
```bash
# Очистіть кеш Maven і повторіть спробу
rm -rf ~/.m2/repository
mvn clean install
```

**Не знайшли кінцеву точку або вхід:**
```bash
# Встановіть кінцеву точку в поточній сесії та увійдіть (без ключа)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Або використовуйте файл .env у директорії проєкту
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Порт вже використовується:**
```bash
# Spring Boot за замовчуванням використовує порт 8080
# Зміна в application.properties:
server.port=8081
```


### Підтримка кількох мов
- Документація доступна понад 45 мовами через автоматичний переклад
- Переклади зберігаються в каталозі `translations/`
- Переклад контролюється за допомогою GitHub Actions workflow

### Шлях навчання
1. Починайте з [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. Слідуйте за главами у порядку (01 → 05)
3. Закінчіть практичні приклади у кожній главі
4. Ознайомтесь із проектами-прикладами у Главі 4
5. Вивчайте відповідальні практики AI у Главі 5

### Контейнер для розробки
Файл `.devcontainer/devcontainer.json` налаштовує:
- Розробницьке середовище Java 21
- Попередньо встановлений Maven
- Розширення Java для VS Code
- Інструменти Spring Boot
- Інтеграцію GitHub Copilot
- Підтримку Docker-in-Docker
- Azure CLI

### Особливості продуктивності
- Розгортання Azure AI Foundry мають квоти токенів/запитів за хвилину
- Використовуйте відповідний розмір пакетної обробки для embedding-ів
- Розгляньте кешування для повторних API викликів
- Контролюйте використання токенів для оптимізації вартості

### Примітки з безпеки
- Ніколи не додавайте `.env` файли у репозиторій (вже є у `.gitignore`)
- Використовуйте автентифікацію без ключів (Microsoft Entra ID) замість API ключів
- Використовуйте керовані ідентичності в Azure; `az login` для локальної розробки
- Дотримуйтесь рекомендацій з відповідального AI у Главі 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Відмова від відповідальності**:
Цей документ було перекладено за допомогою сервісу штучного інтелекту для перекладу [Co-op Translator](https://github.com/Azure/co-op-translator). Хоча ми прагнемо до точності, будь ласка, майте на увазі, що автоматичні переклади можуть містити помилки або неточності. Оригінальний документ рідною мовою слід вважати авторитетним джерелом. Для критично важливої інформації рекомендується професійний людський переклад. Ми не несемо відповідальності за будь-які непорозуміння або неправильні тлумачення, що виникли внаслідок використання цього перекладу.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->