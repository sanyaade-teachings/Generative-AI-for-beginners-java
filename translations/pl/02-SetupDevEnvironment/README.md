# Konfiguracja środowiska programistycznego dla Generatywnej AI dla Javy

> **Szybki start:** Dostarcz swoje modele AI na **Azure AI Foundry** jako kod przy użyciu Bicep + `azd` w kilka minut — zobacz [Przewodnik konfiguracji Azure AI Foundry](getting-started-azure-openai.md). Uwierzytelnianie jest **bezkluczowe** (Microsoft Entra ID), więc nie musisz zarządzać kluczami API.

## Czego się nauczysz

- Konfiguracji środowiska programistycznego Java dla aplikacji AI
- Wybierania i konfigurowania preferowanego środowiska programistycznego (chmurowego z Codespaces, lokalnego kontenera deweloperskiego lub pełnej lokalnej instalacji)
- Testowania konfiguracji przez połączenie z modelem Azure AI Foundry

## Spis treści

- [Czego się nauczysz](#czego-się-nauczysz)
- [Wprowadzenie](#wprowadzenie)
- [Krok 1: Skonfiguruj środowisko programistyczne](#krok-1-skonfiguruj-środowisko-programistyczne)
  - [Opcja A: GitHub Codespaces (zalecane)](#opcja-a-github-codespaces-zalecane)
  - [Opcja B: Lokalny kontener deweloperski](#opcja-b-lokalny-kontener-deweloperski)
  - [Opcja C: Użyj istniejącej lokalnej instalacji](#opcja-c-użyj-istniejącej-lokalnej-instalacji)
- [Krok 2: Provision Azure AI Foundry](#krok-2-provision-azure-ai-foundry)
- [Krok 3: Przetestuj konfigurację](#krok-3-przetestuj-konfigurację)
- [Rozwiązywanie problemów](#rozwiązywanie-problemów)
- [Podsumowanie](#podsumowanie)
- [Kolejne kroki](#kolejne-kroki)

## Wprowadzenie

Ten rozdział poprowadzi Cię przez konfigurację środowiska programistycznego. W całym kursie używamy **Azure AI Foundry** dla modeli. Modele udostępniasz jako kod za pomocą Bicep i Azure Developer CLI (`azd`), następnie łączysz się z **uwierzytelnianiem bezkluczowym** (Microsoft Entra ID) — bez kopiowania czy wycieków kluczy API.

**Nie wymaga lokalnej konfiguracji!** Możesz użyć GitHub Codespaces, które zapewnia pełne środowisko programistyczne w przeglądarce i tam też zrealizować provisioning Foundry.

Wybieramy **Azure AI Foundry** na potrzeby kursu, ponieważ:
- **Provisioning jako kod** — jedno polecenie `azd up` wdraża konto i wdrożenia modeli
- **Bezkluczowe** — uwierzytelnianie poprzez logowanie do Azure lub zarządzaną tożsamość
- **Gotowe do produkcji** — ten sam kod działa lokalnie i w Azure
- **Elastyczne** — wymieniaj modele zmieniając nazwę wdrożenia, nie swój kod

> **Uwaga**: Wdrożenia Azure AI Foundry są rozliczane za token (płać za zużycie). Zobacz [przewodnik konfiguracji Azure AI Foundry](getting-started-azure-openai.md) dla informacji o provisioning, regionach i kosztach.

## Krok 1: Skonfiguruj środowisko programistyczne

<a name="quick-start-cloud"></a>

Stworzyliśmy wstępnie skonfigurowany kontener deweloperski, aby zminimalizować czas konfiguracji i zapewnić Ci wszystkie niezbędne narzędzia do tego kursu Generatywnej AI dla Javy. Wybierz preferowaną metodę:

### Opcje konfiguracji środowiska:

#### Opcja A: GitHub Codespaces (zalecane)

**Zacznij pisać kod w 2 minuty — bez konieczności konfiguracji lokalnej!**

1. Sforkuj to repozytorium na swoje konto GitHub
   > **Uwaga**: Jeśli chcesz edytować podstawową konfigurację, zobacz [Dev Container Configuration](../../../.devcontainer/devcontainer.json)
2. Kliknij **Code** → zakładka **Codespaces** → **...** → **New with options...**
3. Użyj domyślnych ustawień – wybierze to konfigurację **Dev container**: **Generative AI Java Development Environment** stworzony na potrzeby tego kursu
4. Kliknij **Create codespace**
5. Poczekaj około 2 minut na gotowość środowiska
6. Przejdź do [Kroku 2: Provision Azure AI Foundry](#krok-2-provision-azure-ai-foundry)

<img src="../../../translated_images/pl/codespaces.9945ded8ceb431a5.webp" alt="Screenshot: Codespaces submenu" width="50%">

<img src="../../../translated_images/pl/image.833552b62eee7766.webp" alt="Screenshot: New with options" width="50%">

<img src="../../../translated_images/pl/codespaces-create.b44a36f728660ab7.webp" alt="Screenshot: Create codespace options" width="50%">

> **Zalety Codespaces**:
> - Nie wymaga instalacji lokalnej
> - Działa na dowolnym urządzeniu z przeglądarką
> - Wstępnie skonfigurowany ze wszystkimi narzędziami i zależnościami
> - Darmowe 60 godzin miesięcznie dla kont osobistych
> - Spójne środowisko dla wszystkich uczestników kursu

#### Opcja B: Lokalny kontener deweloperski

**Dla programistów preferujących lokalny rozwój z Dockerem**

1. Sforkuj i sklonuj to repozytorium na swój lokalny komputer
   > **Uwaga**: Jeśli chcesz edytować podstawową konfigurację, zobacz [Dev Container Configuration](../../../.devcontainer/devcontainer.json)
2. Zainstaluj [Docker Desktop](https://www.docker.com/products/docker-desktop/) i [VS Code](https://code.visualstudio.com/)
3. Zainstaluj rozszerzenie [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) w VS Code
4. Otwórz folder repozytorium w VS Code
5. Gdy pojawi się monit, kliknij **Reopen in Container** (lub użyj `Ctrl+Shift+P` → "Dev Containers: Reopen in Container")
6. Poczekaj na zbudowanie i uruchomienie kontenera
7. Przejdź do [Kroku 2: Provision Azure AI Foundry](#krok-2-provision-azure-ai-foundry)

<img src="../../../translated_images/pl/devcontainer.21126c9d6de64494.webp" alt="Screenshot: Dev container setup" width="50%">

<img src="../../../translated_images/pl/image-3.bf93d533bbc84268.webp" alt="Screenshot: Dev container build complete" width="50%">

#### Opcja C: Użyj istniejącej lokalnej instalacji

**Dla programistów mających już środowisko Java**

Wymagania:
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html) 
- [Maven 3.9+](https://maven.apache.org/download.cgi)
- [VS Code](https://code.visualstudio.com) lub ulubione IDE

Kroki:
1. Sklonuj to repozytorium lokalnie
2. Otwórz projekt w swoim IDE
3. Przejdź do [Kroku 2: Provision Azure AI Foundry](#krok-2-provision-azure-ai-foundry)

> **Porada**: Jeśli masz słabszy komputer, a chcesz używać VS Code lokalnie, skorzystaj z GitHub Codespaces! Możesz połączyć lokalny VS Code z chmurowym Codespace, łącząc to, co najlepsze z obu światów.

<img src="../../../translated_images/pl/image-2.fc0da29a6e4d2aff.webp" alt="Screenshot: created local devcontainer instance" width="50%">

## Krok 2: Provision Azure AI Foundry

Wdróż modele AI kursu do Azure AI Foundry jako kod. Z katalogu głównego repozytorium:

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```

`azd` poprosi o nazwę środowiska i region, utworzy konto Azure AI Foundry z wdrożeniami `gpt-4o-mini` i `text-embedding-3-small`, a następnie zapisze endpoint w pliku `.env` przykładu — wszystko z **uwierzytelnianiem bezkluczowym** (bez kluczy API).

> **Szczegółowy przewodnik:** Zobacz [Przewodnik konfiguracji Azure AI Foundry](getting-started-azure-openai.md) dla wymagań wstępnych, alternatywy manualnej (portal), wskazówek dot. regionów oraz informacji o kosztach i czyszczeniu.

## Krok 3: Przetestuj konfigurację

Gdy modele Foundry są wdrożone, przetestuj połączenie za pomocą aplikacji przykładowej w [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure).

1. Otwórz terminal w swoim środowisku programistycznym.
2. Przejdź do przykładu:
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
3. Upewnij się, że jesteś zalogowany (uwierzytelnianie bezkluczowe wymaga tokena):
   ```bash
   az login
   ```
   > Jeśli uruchomiłeś `azd up`, plik `.env` z endpointem został już dla Ciebie zapisany.
4. Uruchom aplikację:
   ```bash
   mvn clean spring-boot:run
   ```

Powinieneś zobaczyć odpowiedź z modelu `gpt-4o-mini`.

### Zrozumienie przykładowego kodu

Przykład w `examples/basic-chat-azure` to aplikacja Spring Boot, która używa **Spring AI** do łączenia się z Azure AI Foundry z uwierzytelnianiem bezkluczowym.

**Co robi ten kod:**
- **Łączy się** z Azure AI Foundry używając Twojego logowania do Azure (Microsoft Entra ID) — bez klucza API
- **Wysyła** zapytanie do modelu `gpt-4o-mini`
- **Odbiera** i wyświetla odpowiedź AI
- **Weryfikuje** poprawność konfiguracji

**Główna zależność** (w `pom.xml`):
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```

**Konfiguracja** (`application.yml`):
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

## Podsumowanie

Świetnie! Masz teraz wszystko skonfigurowane:

- Zdalnie wdrożyłeś modele Azure AI Foundry jako kod za pomocą Bicep + `azd`
- Uruchomiłeś środowisko programistyczne Java (czy to Codespaces, kontener, czy lokalne)
- Połączyłeś się z Azure AI Foundry z uwierzytelnianiem bezkluczowym (Microsoft Entra ID) — bez kluczy API
- Przetestowałeś, że wszystko działa na prostym przykładzie komunikującym się z modelem

## Kolejne kroki

[Rozdział 3: Podstawowe techniki generatywnej AI](../03-CoreGenerativeAITechniques/README.md)

## Rozwiązywanie problemów

Masz problemy? Oto najczęstsze błędy i rozwiązania:

- **Błąd uwierzytelniania (401/403)?**
  - Uruchom `az login` — uwierzytelnianie jest bezkluczowe, musisz być zalogowany
  - Sprawdź, czy Twoje konto ma rolę **Cognitive Services OpenAI User** na zasobie
  - Jeśli właśnie provisionowałeś, poczekaj minutę na propagację przypisania roli

- **Nie znaleziono Maven?**
  - W kontenerach deweloperskich/Codespaces Maven jest zainstalowany domyślnie
  - W lokalnej konfiguracji upewnij się, że masz Java 21+ i Maven 3.9+
  - Sprawdź instalację poleceniem `mvn --version`

- **Nie znaleziono `azd` lub provisioning się nie powiódł?**
  - Zainstaluj [Azure Developer CLI](https://aka.ms/azure-dev/install) i użyj `azd auth login`
  - Wybierz region, gdzie jest dostępny `gpt-4o-mini` (np. `eastus2`)
  - Sprawdź [przewodnik konfiguracji Azure AI Foundry](getting-started-azure-openai.md) po szczegóły

- **Kontener deweloperski się nie uruchamia?**
  - Upewnij się, że Docker Desktop działa (do lokalnego rozwoju)
  - Spróbuj przebudować kontener: `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **Błędy kompilacji aplikacji?**
  - Upewnij się, że jesteś w odpowiednim folderze: `02-SetupDevEnvironment/examples/basic-chat-azure`
  - Spróbuj wyczyścić i przebudować projekt: `mvn clean compile`

> **Potrzebujesz pomocy?**: Jeśli nadal masz problemy, otwórz zgłoszenie w repozytorium, a pomożemy Ci.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Zastrzeżenie**:
Niniejszy dokument został przetłumaczony za pomocą usługi tłumaczenia AI [Co-op Translator](https://github.com/Azure/co-op-translator). Choć dążymy do dokładności, prosimy pamiętać, że automatyczne tłumaczenia mogą zawierać błędy lub niedokładności. Oryginalny dokument w jego języku źródłowym należy uznawać za autorytatywne źródło. W przypadku informacji krytycznych zalecane jest skorzystanie z profesjonalnego tłumaczenia wykonanego przez człowieka. Nie ponosimy odpowiedzialności za jakiekolwiek nieporozumienia lub błędne interpretacje wynikające z użycia tego tłumaczenia.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->