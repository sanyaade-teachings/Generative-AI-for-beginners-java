# AGENTS.md

## Przegląd projektu

To jest edukacyjne repozytorium do nauki tworzenia generatywnej sztucznej inteligencji w Javie. Zapewnia kompleksowy kurs praktyczny obejmujący duże modele językowe (LLM), inżynierię promptów, embeddingi, RAG (Retrieval-Augmented Generation) oraz protokół kontekstu modelu (MCP).

**Kluczowe technologie:**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI oraz SDK OpenAI

**Architektura:**
- Wiele samodzielnych aplikacji Spring Boot zorganizowanych wg rozdziałów
- Przykładowe projekty demonstrujące różne wzorce AI
- Gotowe do uruchomienia w GitHub Codespaces z wstępnie skonfigurowanymi kontenerami deweloperskimi

## Komendy instalacyjne

### Wymagania wstępne
- Java 21 lub nowsza
- Maven 3.x
- Subskrypcja Azure z wdrożeniem modelu Azure AI Foundry (prowizja przez `azd up`)
- Azure CLI (`az`) i Azure Developer CLI (`azd`), zalogowane dla autoryzacji bezkluczowej

### Konfiguracja środowiska

**Opcja 1: GitHub Codespaces (zalecane)**
```bash
# Utwórz fork repozytorium i stwórz codespace z interfejsu GitHub
# Kontener deweloperski automatycznie zainstaluje wszystkie zależności
# Poczekaj około 2 minuty na przygotowanie środowiska
```

**Opcja 2: Lokalny kontener deweloperski**
```bash
# Sklonuj repozytorium
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Otwórz w VS Code z rozszerzeniem Dev Containers
# Ponownie otwórz w kontenerze, gdy pojawi się monit
```

**Opcja 3: Lokalne uruchomienie**
```bash
# Zainstaluj zależności
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Zweryfikuj instalację
java -version
mvn -version
```

### Konfiguracja

**Azure AI Foundry (bezkluczowa, zalecana):**
```bash
# Przygotuj konto Foundry + wdrożenia modeli jako kod
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd zapisuje examples/basic-chat-azure/.env z twoim punktem końcowym (bez klucza)
```

**Ręczna konfiguracja punktu końcowego:**
```bash
# Jeśli nie używałeś azd, ustaw punkt końcowy samodzielnie (uwierzytelnianie pozostaje bezkluczowe przez az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Edytuj .env: AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Przebieg prac rozwojowych

### Struktura projektu
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

### Uruchamianie aplikacji

**Uruchamianie aplikacji Spring Boot:**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Budowanie projektu:**
```bash
cd [project-directory]
mvn clean install
```

**Uruchamianie serwera kalkulatora MCP:**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# Serwer działa na http://localhost:8080
```

**Uruchamianie przykładów klienta:**
```bash
# Po uruchomieniu serwera w innym terminalu
cd 04-PracticalSamples/calculator

# Bezpośredni klient MCP
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# Klient zasilany przez AI (wymaga AZURE_OPENAI_ENDPOINT + az login)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Interaktywny bot
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Hot Reload
Spring Boot DevTools jest dołączony w projektach obsługujących hot reload:
```bash
# Zmiany w plikach Java zostaną automatycznie przeładowane po zapisaniu
mvn spring-boot:run
```

## Instrukcje testowania

### Uruchamianie testów

**Uruchom wszystkie testy w projekcie:**
```bash
cd [project-directory]
mvn test
```

**Uruchom testy z wyjściem szczegółowym:**
```bash
mvn test -X
```

**Uruchom konkretną klasę testową:**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Struktura testów
- Pliki testowe korzystają z JUnit 5 (Jupiter)
- Klasy testowe znajdują się w `src/test/java/`
- Przykłady klienta w projekcie kalkulatora są w `src/test/java/com/microsoft/mcp/sample/client/`

### Testowanie ręczne
Wiele przykładów to aplikacje interaktywne wymagające testów manualnych:

1. Uruchom aplikację za pomocą `mvn spring-boot:run`
2. Testuj endpointy lub korzystaj z CLI
3. Sprawdź, czy oczekiwane zachowanie zgadza się z dokumentacją w README.md każdego projektu

### Testowanie z Azure AI Foundry
- Zaloguj się za pomocą `az login` przed uruchomieniem przykładów (autoryzacja bezkluczowa)
- Upewnij się, że Twoje konto ma rolę Cognitive Services OpenAI User dla zasobu
- Przetestuj filtrowanie treści za pomocą przykładu odpowiedzialnej AI z rozdziału 5

## Wytyczne dotyczące stylu kodu

### Konwencje Java
- **Wersja Javy:** Java 21 z nowoczesnymi funkcjami
- **Styl:** Zgodny ze standardowymi konwencjami Javy
- **Nazewnictwo:** 
  - Klasy: PascalCase
  - Metody/zmienne: camelCase
  - Stałe: UPPER_SNAKE_CASE
  - Nazwy pakietów: małe litery

### Wzorce Spring Boot
- Używaj `@Service` do logiki biznesowej
- Używaj `@RestController` do endpointów REST
- Konfiguracja przez `application.yml` lub `application.properties`
- Preferuj zmienne środowiskowe zamiast wartości na sztywno
- Używaj adnotacji `@Tool` dla metod udostępnianych przez MCP

### Organizacja plików
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

### Zależności
- Zarządzane przez Maven `pom.xml`
- Spring AI BOM do zarządzania wersjami
- LangChain4j dla integracji AI
- Spring Boot starter parent dla zależności Spring

### Komentarze w kodzie
- Dodawaj JavaDoc do publicznych API
- Zawieraj wyjaśniające komentarze przy złożonych interakcjach AI
- Jasno dokumentuj opisy narzędzi MCP

## Budowanie i wdrażanie

### Budowanie projektów

**Budowanie bez testów:**
```bash
mvn clean install -DskipTests
```

**Budowanie ze wszystkimi kontrolami:**
```bash
mvn clean install
```

**Pakowanie aplikacji:**
```bash
mvn package
# Tworzy plik JAR w katalogu target/
```

### Katalogi wyjściowe
- Skompilowane klasy: `target/classes/`
- Klasy testowe: `target/test-classes/`
- Pliki JAR: `target/*.jar`
- Artefakty Maven: `target/`

### Konfiguracja specyficzna dla środowiska

**Deweloperskie:**
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

**Produkcja:**
- Używaj tożsamości zarządzanej zamiast `az login` dla autoryzacji bezkluczowej
- Ustaw `AZURE_OPENAI_ENDPOINT` na zasób Foundry produkcji
- Zarządzaj konfiguracją przez zmienne środowiskowe lub Azure Key Vault

### Uwagi do wdrożenia
- To repozytorium edukacyjne z przykładowymi aplikacjami
- Nie jest przeznaczone do produkcyjnego wdrożenia bez modyfikacji
- Przykłady pokazują wzorce, które można zaadaptować produkcyjnie
- Zobacz README poszczególnych projektów dla konkretnych wskazówek wdrożeniowych

## Dodatkowe informacje

### Azure AI Foundry
- **Autoryzacja bezkluczowa:** łącz się za pomocą Microsoft Entra ID — brak konieczności zarządzania kluczami API
- **Prowizja jako kod:** Bicep + azd (`azd up`) tworzą konto i wdrożenia modeli
- Ten sam kod kompatybilny z OpenAI działa lokalnie (`az login`) i w Azure (tożsamość zarządzana)

### Praca z wieloma projektami
Każdy projekt przykładowy jest samodzielny:
```bash
# Przejdź do konkretnego projektu
cd 04-PracticalSamples/[project-name]

# Każdy ma własny plik pom.xml i może być budowany niezależnie
mvn clean install
```

### Typowe problemy

**Niezgodność wersji Javy:**
```bash
# Zweryfikuj Java 21
java -version
# Zaktualizuj JAVA_HOME w razie potrzeby
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Problemy z pobraniem zależności:**
```bash
# Wyczyść pamięć podręczną Maven i spróbuj ponownie
rm -rf ~/.m2/repository
mvn clean install
```

**Brak punktu końcowego lub logowania:**
```bash
# Ustaw endpoint w bieżącej sesji i zaloguj się (bez klucza)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Lub użyj pliku .env w katalogu projektu
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Port już zajęty:**
```bash
# Spring Boot domyślnie używa portu 8080
# Zmiana w application.properties:
server.port=8081
```

### Wsparcie wielojęzyczne
- Dokumentacja dostępna w ponad 45 językach dzięki automatycznemu tłumaczeniu
- Tłumaczenia w katalogu `translations/`
- Zarządzanie tłumaczeniami przez workflow GitHub Actions

### Ścieżka nauki
1. Zacznij od [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. Stosuj się do kolejności rozdziałów (01 → 05)
3. Wykonaj praktyczne przykłady w każdym rozdziale
4. Zbadaj projekty przykładowe w rozdziale 4
5. Poznaj praktyki odpowiedzialnej AI w rozdziale 5

### Kontener deweloperski
Plik `.devcontainer/devcontainer.json` konfiguruje:
- Środowisko deweloperskie Java 21
- Preinstalowany Maven
- Rozszerzenia Java do VS Code
- Narzędzia Spring Boot
- Integrację GitHub Copilot
- Obsługę Docker-in-Docker
- Azure CLI

### Wskazówki dotyczące wydajności
- Wdrożenia Azure AI Foundry mają limity tokenów/żądań na minutę
- Używaj odpowiednich rozmiarów pakietów w embeddingach
- Rozważ cache’owanie dla powtarzających się wywołań API
- Monitoruj zużycie tokenów w celu optymalizacji kosztów

### Uwagi dotyczące bezpieczeństwa
- Nigdy nie commituj plików `.env` (są już w `.gitignore`)
- Preferuj autoryzację bezkluczową (Microsoft Entra ID) zamiast kluczy API
- Używaj tożsamości zarządzanych w Azure; `az login` do środowiska lokalnego
- Stosuj się do wytycznych odpowiedzialnej AI z rozdziału 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Zastrzeżenie**:
Niniejszy dokument został przetłumaczony za pomocą usługi tłumaczenia AI [Co-op Translator](https://github.com/Azure/co-op-translator). Choć dążymy do dokładności, prosimy pamiętać, że automatyczne tłumaczenia mogą zawierać błędy lub niedokładności. Oryginalny dokument w jego języku źródłowym należy uznawać za autorytatywne źródło. W przypadku informacji krytycznych zalecane jest skorzystanie z profesjonalnego tłumaczenia wykonanego przez człowieka. Nie ponosimy odpowiedzialności za jakiekolwiek nieporozumienia lub błędne interpretacje wynikające z użycia tego tłumaczenia.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->