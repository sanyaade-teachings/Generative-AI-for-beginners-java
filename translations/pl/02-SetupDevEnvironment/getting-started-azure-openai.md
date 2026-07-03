# Konfigurowanie środowiska deweloperskiego dla Azure AI Foundry

> Ten przewodnik konfiguruje modele **Azure AI Foundry** dla aplikacji AI w Javie w tym kursie, używając uwierzytelniania **bezkluczowego** (Microsoft Entra ID) — bez konieczności zarządzania kluczami API. Nowy w narzędziach? Zacznij od [przewodnika po środowisku deweloperskim](./README.md).

Ten przewodnik konfiguruje modele **Azure AI Foundry** dla aplikacji AI w Javie w tym kursie. Masz dwie ścieżki:

- **Opcja A — Provisioning za pomocą `azd` + Bicep (zalecane):** jeden polecenie wdraża konto Foundry i modele jako kod. Bez klikania w portalu.
- **Opcja B — Ręczne tworzenie zasobów** w portalu Azure AI Foundry.

Obie ścieżki korzystają z uwierzytelniania **bezkluczowego** (Microsoft Entra ID) — nie ma potrzeby kopiowania ani wycieku kluczy API.

## Spis treści

- [Co zostaje utworzone](#co-zostaje-utworzone)
- [Wymagania wstępne](#wymagania-wstępne)
- [Opcja A: Provisioning za pomocą azd + Bicep (zalecane)](#option-a-provision-with-azd--bicep-recommended)
- [Opcja B: Ręczne tworzenie zasobów](#opcja-b-ręczne-tworzenie-zasobów)
- [Konfiguracja środowiska](#konfiguracja-środowiska)
- [Testowanie konfiguracji](#testowanie-konfiguracji)
- [Co dalej?](#co-dalej)
- [Zasoby](#zasoby)
- [Dodatkowe zasoby](#dodatkowe-zasoby)

## Co zostaje utworzone

Szablony Bicep w [`infra/`](../../../02-SetupDevEnvironment/infra) tworzą:

- Konto **Azure AI Foundry** (`Microsoft.CognitiveServices/accounts`, typ `AIServices`) z projektem
- Deployment **czatu** — `gpt-4o-mini`
- Deployment **embeddingu** — `text-embedding-3-small` (używany w późniejszych rozdziałach)
- **Bezkluczowe przypisanie roli** (`Cognitive Services OpenAI User`), abyś mógł się zalogować przez `az login` zamiast zarządzać kluczami

## Wymagania wstępne

- [Subskrypcja Azure](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) i [Maven 3.9+](https://maven.apache.org/download.cgi)

## Opcja A: Provisioning za pomocą azd + Bicep (zalecane)

Z folderu `02-SetupDevEnvironment`:

```bash
cd 02-SetupDevEnvironment

# Zaloguj się (oba narzędzia)
azd auth login
az login

# Udostępnij konto Foundry + wdrożenia modeli
azd up
```

`azd` poprosi o **nazwę środowiska** (np. `genai-java`) oraz **region**. Wybierz region, w którym dostępne są `gpt-4o-mini` i `text-embedding-3-small` — na przykład `eastus2` lub `swedencentral`.

Po zakończeniu provisioning `azd`:

1. Wdraża wszystko zdefiniowane w [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep).
2. Uruchamia hook postprovision, który zapisuje [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) z twoim endpointem i nazwami deploymentów (bez sekretów).

> **Wskazówka:** Uruchom ponownie `azd up` w dowolnym momencie, aby zastosować zmiany. Użyj `azd down`, aby usunąć wszystko i zakończyć generowanie kosztów.

Aby zobaczyć wygenerowane ustawienia:

```bash
azd env get-values
```

Teraz przejdź do [Testowania konfiguracji](#testowanie-konfiguracji).

## Opcja B: Ręczne tworzenie zasobów

Wolisz portal? Utwórz zasoby ręcznie:

1. Wejdź na [portal Azure AI Foundry](https://ai.azure.com/) i zaloguj się.
2. **Utwórz projekt** (to także tworzy zasób AI Foundry). Nadaj mu nazwę np. `GenAIJava`.
3. W projekcie otwórz **Modele + endpointy** → **Wdróż model** → **Wdróż model bazowy**.
4. Wdróż **gpt-4o-mini** (nazwa deploymentu `gpt-4o-mini`). Powtórz dla **text-embedding-3-small**, jeśli chcesz przykłady embeddingu.
5. Z **Przeglądu** skopiuj **endpoint** (np. `https://<resource>.openai.azure.com/`).
6. Przyznaj sobie dostęp bezkluczowy: na zasobie otwórz **Kontrola dostępu (IAM)** → **Dodaj przypisanie roli** → przypisz rolę **Cognitive Services OpenAI User** do swojego konta.

> **Wciąż masz problemy?** Sprawdź [dokumentację Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Konfiguracja środowiska

**Jeśli użyłeś Opcji A (`azd up`)**, plik ustawień jest już zapisany — nic nie musisz konfigurować. Przejdź do [Testowania konfiguracji](#testowanie-konfiguracji).

**Jeśli użyłeś Opcji B (ręczna), utwórz samodzielnie plik `.env` przykładu:**

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```

Edytuj `.env`, wpisując swój endpoint (bez klucza — uwierzytelnianie jest bezkluczowe):

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```

> **Uwaga dotycząca bezpieczeństwa:** Nie ma potrzeby przechowywania klucza API. Uwierzytelniasz się za pomocą Microsoft Entra ID przez `az login` (lokalnie) lub zarządzaną tożsamość (w Azure). Plik `.env` zawiera tylko ustawienia niebędące sekretami i jest już objęty `.gitignore`.

## Testowanie konfiguracji

Upewnij się, że jesteś zalogowany, aby uwierzytelnianie bezkluczowe mogło pobrać token, następnie uruchom przykład:

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # jeśli nie jesteś jeszcze zalogowany
mvn clean spring-boot:run
```

Powinieneś zobaczyć odpowiedź z modelu `gpt-4o-mini`!

> **Użytkownicy VS Code:** Naciśnij `F5`, aby uruchomić. Aplikacja automatycznie wczytuje twój plik `.env`.

> **Pełny przykład:** Zobacz [Basic Chat z Azure AI Foundry](./examples/basic-chat-azure/README.md) dla szczegółów i rozwiązywania problemów.

## Co dalej?

**Konfiguracja zakończona!** Masz teraz:
- Azure AI Foundry z wdrożonymi modelami `gpt-4o-mini` i `text-embedding-3-small`
- Uwierzytelnianie bezkluczowe (Microsoft Entra ID) — bez zarządzania kluczami
- Lokalny plik `.env` z twoim endpointem i nazwami deploymentów
- Gotowe środowisko deweloperskie w Javie

**Kontynuuj do** [Rozdział 3: Podstawowe techniki generatywnej AI](../03-CoreGenerativeAITechniques/README.md), aby zacząć budować aplikacje AI!

## Zasoby

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)
- [Uwierzytelnianie bezkluczowe z Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Dokumentacja Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)
- [Spring AI Azure OpenAI Dokumentacja](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Dodatkowe zasoby

- [Pobierz VS Code](https://code.visualstudio.com/Download)
- [Pobierz Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Konfiguracja środowiska deweloperskiego kontenera](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Zastrzeżenie**:
Niniejszy dokument został przetłumaczony za pomocą usługi tłumaczenia AI [Co-op Translator](https://github.com/Azure/co-op-translator). Choć dążymy do dokładności, prosimy pamiętać, że automatyczne tłumaczenia mogą zawierać błędy lub niedokładności. Oryginalny dokument w jego języku źródłowym należy uznawać za autorytatywne źródło. W przypadku informacji krytycznych zalecane jest skorzystanie z profesjonalnego tłumaczenia wykonanego przez człowieka. Nie ponosimy odpowiedzialności za jakiekolwiek nieporozumienia lub błędne interpretacje wynikające z użycia tego tłumaczenia.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->