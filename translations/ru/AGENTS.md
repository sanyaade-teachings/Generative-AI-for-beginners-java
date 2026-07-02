# AGENTS.md

## Обзор проекта

Это учебный репозиторий для изучения разработки Generative AI на Java. Он предоставляет всесторонний практический курс, охватывающий большие языковые модели (LLM), проектирование подсказок, эмбеддинги, RAG (генерация с поддержкой поиска) и протокол контекста модели (MCP).

**Ключевые технологии:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI и OpenAI SDK

**Архитектура:**
- Несколько автономных приложений Spring Boot, организованных по главам
- Примеры проектов, демонстрирующие разные AI-паттерны
- Готово для GitHub Codespaces с предварительно настроенными dev-контейнерами

## Команды для настройки

### Требования
- Java 21 или выше
- Maven 3.x
- Подписка Azure с развертыванием модели Azure AI Foundry (создается с помощью `azd up`)
- Azure CLI (`az`) и Azure Developer CLI (`azd`), выполнен вход для аутентификации без ключей

### Настройка окружения

**Вариант 1: GitHub Codespaces (рекомендуется)**
```bash
# Форкните репозиторий и создайте codespace через интерфейс GitHub
# Контейнер разработки автоматически установит все зависимости
# Подождите около 2 минут для настройки окружения
```

**Вариант 2: Локальный dev-контейнер**
```bash
# Клонировать репозиторий
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Открыть в VS Code с расширением Dev Containers
# Перезапустить в контейнере при появлении запроса
```

**Вариант 3: Локальная установка**
```bash
# Установить зависимости
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Проверить установку
java -version
mvn -version
```

### Конфигурация

**Настройка Azure AI Foundry (без ключей, рекомендуется):**
```bash
# Обеспечить развертывание учетной записи Foundry и моделей как код
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd создает examples/basic-chat-azure/.env с вашим конечным URL (без ключа)
```

**Ручная конфигурация endpoint:**
```bash
# Если вы не использовали azd, установите конечную точку самостоятельно (аутентификация остаётся без ключа через az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Редактируйте .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Рабочий процесс разработки

### Структура проекта
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

### Запуск приложений

**Запуск Spring Boot приложения:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Сборка проекта:**
```bash
cd [project-directory]
mvn clean install
```

**Запуск MCP Calculator Server:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# Сервер работает на http://localhost:8080
```

**Запуск клиентских примеров:**
```bash
# После запуска сервера в другом терминале
cd 04-PracticalSamples/calculator

# Прямой клиент MCP
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# Клиент с поддержкой ИИ (требуется AZURE_OPENAI_ENDPOINT + az login)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Интерактивный бот
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Горячая перезагрузка
Spring Boot DevTools включен в проекты с поддержкой горячей перезагрузки:
```bash
# Изменения в файлах Java будут автоматически перезагружаться при сохранении
mvn spring-boot:run
```

## Инструкции по тестированию

### Запуск тестов

**Запустить все тесты в проекте:**
```bash
cd [project-directory]
mvn test
```

**Запуск тестов с подробным выводом:**
```bash
mvn test -X
```

**Запуск конкретного класса тестов:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Структура тестов
- Тестовые файлы используют JUnit 5 (Jupiter)
- Классы тестов находятся в `src/test/java/`
- Клиентские примеры в проекте калькулятора расположены в `src/test/java/com/microsoft/mcp/sample/client/`

### Ручное тестирование
Многие примеры являются интерактивными приложениями, требующими ручного тестирования:

1. Запустите приложение командой `mvn spring-boot:run`
2. Тестируйте конечные точки или взаимодействуйте с CLI
3. Проверьте, что поведение соответствует документации в README.md каждого проекта

### Тестирование с Azure AI Foundry
- Выполните вход с помощью `az login` перед запуском примеров (аутентификация без ключей)
- Убедитесь, что у вашей учетной записи есть роль Cognitive Services OpenAI User на ресурсе
- Проверьте фильтрацию контента с примером ответственного AI в Главе 5

## Руководство по стилю кода

### Конвенции Java
- **Версия Java:** Java 21 с современными возможностями
- **Стиль:** Соблюдайте стандартные конвенции Java
- **Именование:** 
  - Классы: PascalCase
  - Методы/переменные: camelCase
  - Константы: UPPER_SNAKE_CASE
  - Названия пакетов: строчные буквы

### Паттерны Spring Boot
- Используйте `@Service` для бизнес-логики
- Используйте `@RestController` для REST-эндпоинтов
- Конфигурация через `application.yml` или `application.properties`
- Предпочтение переменным окружения вместо захардкоженных значений
- Используйте аннотацию `@Tool` для методов, доступных через MCP

### Организация файлов
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
- Управление через Maven `pom.xml`
- Spring AI BOM для управления версиями
- LangChain4j для интеграций AI
- Spring Boot starter parent для зависимостей Spring

### Комментарии в коде
- Добавляйте JavaDoc для публичных API
- Включайте поясняющие комментарии для сложных AI-взаимодействий
- Четко документируйте описания инструментов MCP

## Сборка и деплой

### Сборка проектов

**Сборка без тестов:**
```bash
mvn clean install -DskipTests
```

**Сборка со всеми проверками:**
```bash
mvn clean install
```

**Упаковка приложения:**
```bash
mvn package
# Создает JAR в каталоге target/
```

### Каталоги вывода
- Скомпилированные классы: `target/classes/`
- Классы тестов: `target/test-classes/`
- JAR-файлы: `target/*.jar`
- Maven артефакты: `target/`

### Конфигурация для разных окружений

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

**Продакшен:**
- Используйте управляемую идентичность вместо `az login` для аутентификации без ключей
- Укажите `AZURE_OPENAI_ENDPOINT` на ваш продакшен-ресурс Foundry
- Управляйте конфигурацией через переменные окружения или Azure Key Vault

### Особенности деплоя
- Это учебный репозиторий с примерами приложений
- Не предназначен для деплоя в продакшен в исходном виде
- Примеры демонстрируют паттерны для адаптации под продакшен
- Смотрите README отдельных проектов для примечаний по деплою

## Дополнительные заметки

### Azure AI Foundry
- **Аутентификация без ключей:** подключение через Microsoft Entra ID — не нужно управлять API-ключами
- **Provisioning как код:** Bicep + azd (`azd up`) создают аккаунт и развертывания моделей
- Один и тот же код, совместимый с OpenAI, работает локально (`az login`) и в Azure (управляемая идентичность)

### Работа с несколькими проектами
Каждый пример проекта является автономным:
```bash
# Перейти к конкретному проекту
cd 04-PracticalSamples/[project-name]

# Каждый имеет свой собственный pom.xml и может быть собран независимо
mvn clean install
```

### Распространенные проблемы

**Несоответствие версии Java:**
```bash
# Проверить Java 21
java -version
# Обновить JAVA_HOME, если необходимо
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Проблемы с загрузкой зависимостей:**
```bash
# Очистить кэш Maven и повторить попытку
rm -rf ~/.m2/repository
mvn clean install
```

**Не найден endpoint или вход в систему:**
```bash
# Установите конечную точку в текущей сессии и выполните вход (без ключа)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Или используйте файл .env в директории проекта
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Порт уже используется:**
```bash
# Spring Boot по умолчанию использует порт 8080
# Измените в application.properties:
server.port=8081
```

### Поддержка нескольких языков
- Документация доступна на более чем 45 языках через автоматический перевод
- Переводы находятся в каталоге `translations/`
- Перевод управляется рабочим процессом GitHub Actions

### Путь обучения
1. Начните с [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. Следуйте главам по порядку (01 → 05)
3. Выполняйте практические примеры в каждой главе
4. Изучайте примеры проектов в Главе 4
5. Ознакомьтесь с практиками ответственного AI в Главе 5

### Контейнер разработки
`.devcontainer/devcontainer.json` настраивает:
- Среду разработки Java 21
- Maven предустановлен
- Расширения для Java в VS Code
- Инструменты Spring Boot
- Интеграция с GitHub Copilot
- Поддержка Docker-in-Docker
- Azure CLI

### Особенности производительности
- Развертывания Azure AI Foundry имеют квоты по токенам/запросам в минуту
- Используйте подходящий размер батча для эмбеддингов
- Рассмотрите кэширование при повторных API вызовах
- Мониторьте использование токенов для оптимизации затрат

### Заметки по безопасности
- Никогда не коммитьте файлы `.env` (они уже в `.gitignore`)
- Предпочитайте аутентификацию без ключей (Microsoft Entra ID) вместо API-ключей
- Используйте управляемые идентичности в Azure; `az login` для локальной разработки
- Следуйте рекомендациям по ответственному AI из Главы 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Отказ от ответственности**:
Этот документ был переведен с использованием сервиса машинного перевода [Co-op Translator](https://github.com/Azure/co-op-translator). Несмотря на наши усилия по обеспечению точности, имейте в виду, что автоматический перевод может содержать ошибки или неточности. Оригинальный документ на его исходном языке следует считать авторитетным источником. Для получения критически важной информации рекомендуется обратиться к профессиональному человеческому переводу. Мы не несем ответственности за любые недоразумения или неправильные толкования, возникшие в результате использования этого перевода.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->