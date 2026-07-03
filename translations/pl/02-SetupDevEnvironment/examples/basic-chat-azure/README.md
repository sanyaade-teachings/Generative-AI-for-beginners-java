# Podstawowy czat z Azure AI Foundry - przykład end-to-end

Ten przykład to prosta aplikacja Spring Boot, która łączy się z modelem **Azure AI Foundry** za pomocą **uwierzytelniania bezkluczowego** (Microsoft Entra ID) i testuje Twoją konfigurację. Używa klienta czatu `ChatClient` z biblioteki Spring AI.

## Spis treści

- [Wymagania wstępne](#wymagania-wstępne)
- [Szybki start](#szybki-start)
- [Jak działa uwierzytelnianie](#jak-działa-uwierzytelnianie)
- [Uruchamianie aplikacji](#uruchamianie-aplikacji)
  - [Użycie Maven](#użycie-maven)
  - [Użycie VS Code](#użycie-vs-code)
  - [Oczekiwany rezultat](#oczekiwany-rezultat)
- [Referencja konfiguracji](#referencja-konfiguracji)
  - [Zmienne środowiskowe](#zmienne-środowiskowe)
  - [Konfiguracja Spring](#konfiguracja-spring)
- [Rozwiązywanie problemów](#rozwiązywanie-problemów)
  - [Typowe problemy](#typowe-problemy)
  - [Tryb debugowania](#tryb-debugowania)
- [Kolejne kroki](#kolejne-kroki)
- [Zasoby](#zasoby)

## Wymagania wstępne

Przed uruchomieniem tego przykładu upewnij się, że masz:

- Zasób Azure AI Foundry z wdrożeniem `gpt-4o-mini` — utwórz go za pomocą `azd up` lub ręcznie wg [przewodnika po Azure AI Foundry](../../getting-started-azure-openai.md)
- Rolę **Cognitive Services OpenAI User** na tym zasobie (szablony Bicep przypisują ją automatycznie)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), zalogowany przez `az login`
- Java 21+ i Maven 3.9+

> **Brak wymaganego klucza API** — uwierzytelnianie jest bezkluczowe przez Microsoft Entra ID.

## Szybki start

```bash
# 1. Przejdź do projektu
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Zaloguj się, aby uwierzytelnianie bezkluczowe mogło uzyskać token
az login

# 3. Skonfiguruj punkt końcowy
#    - Jeśli uruchomiłeś `azd up`, plik .env został dla Ciebie utworzony (pomiń ten krok).
#    - W przeciwnym razie skopiuj szablon i ustaw AZURE_OPENAI_ENDPOINT:
cp .env.example .env

# 4. Uruchom aplikację
mvn spring-boot:run
```

## Jak działa uwierzytelnianie

Ten przykład uwierzytelnia się przez **Microsoft Entra ID** — nie używa klucza API.

Kiedy ustawione jest tylko `spring.ai.azure.openai.endpoint` (bez klucza api-key), Spring AI tworzy klienta Azure OpenAI z [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). To poświadczenie automatycznie znajduje token z lokalnej sesji `az login` albo z tożsamości zarządzanej w Azure — więc ten sam kod działa w obu środowiskach bez zmian.

## Uruchamianie aplikacji

### Użycie Maven

```bash
mvn spring-boot:run
```

### Użycie VS Code

1. Otwórz projekt w VS Code
2. Naciśnij `F5` lub użyj panelu "Uruchom i debuguj"
3. Wybierz konfigurację "Spring Boot-BasicChatApplication"

> **Uwaga**: konfiguracja VS Code automatycznie ładuje Twój plik .env

### Oczekiwany rezultat

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

## Referencja konfiguracji

### Zmienne środowiskowe

| Zmienna | Opis | Wymagana | Przykład |
|----------|-------------|----------|---------|
| `AZURE_OPENAI_ENDPOINT` | URL punktu końcowego Foundry (Azure OpenAI) | Tak | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Nazwa wdrożenia modelu czatu | Nie | `gpt-4o-mini` (domyślne) |

> Nie ma zmiennej dla klucza API — uwierzytelnianie jest bezkluczowe (Microsoft Entra ID za pomocą `az login`).

### Konfiguracja Spring

Plik `application.yml` konfiguruje:
- **Endpoint**: `${AZURE_OPENAI_ENDPOINT}` - z zmiennej środowiskowej
- **Wdrożenie**: `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - ze zmiennej środowiskowej z wartością domyślną
- **Auth**: bezkluczowe — nie ustawiono `api-key`, więc Spring AI używa `DefaultAzureCredential`
- **Temperatura**: `0.7` - kontroluje kreatywność (0.0 = deterministyczne, 1.0 = kreatywne)
- **Max Tokens**: `500` - maksymalna długość odpowiedzi

## Rozwiązywanie problemów

### Typowe problemy

<details>
<summary><strong>Błąd: 401 / "PermissionDenied" / błędy tokenu</strong></summary>

- Wykonaj `az login` — uwierzytelnianie bezkluczowe wymaga aktywnego zalogowania, by uzyskać token
- Sprawdź, czy Twój użytkownik ma rolę **Cognitive Services OpenAI User** na zasobie
- Jeśli właśnie przypisałeś rolę, odczekaj chwilę na propagację
- Zweryfikuj, czy jesteś w odpowiednim tenantcie/subskrypcji (`az account show`)
</details>

<details>
<summary><strong>Błąd: "The endpoint is not valid" / błędy połączenia</strong></summary>

- Upewnij się, że `AZURE_OPENAI_ENDPOINT` to pełny adres URL podstawowy (np. `https://your-resource.openai.azure.com/`)
- Sprawdź spójność ukośników na końcu
- Potwierdź, że punkt końcowy odpowiada Twojemu zasobowi (`azd env get-values`)
</details>

<details>
<summary><strong>Błąd: "The deployment was not found"</strong></summary>

- Zweryfikuj, czy `AZURE_OPENAI_DEPLOYMENT` odpowiada nazwie wdrożenia w Azure
- Sprawdź, czy model jest poprawnie wdrożony i aktywny
- Domyślną nazwą wdrożenia jest `gpt-4o-mini`
</details>

<details>
<summary><strong>VS Code: Zmienne środowiskowe się nie ładują</strong></summary>

- Upewnij się, że plik `.env` znajduje się w katalogu głównym projektu (na tym samym poziomie co `pom.xml`)
- Spróbuj uruchomić `mvn spring-boot:run` w zintegrowanym terminalu VS Code
- Sprawdź, czy rozszerzenie Java w VS Code jest poprawnie zainstalowane
</details>

### Tryb debugowania

Aby włączyć szczegółowe logowanie, odkomentuj te linie w `application.yml`:

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Kolejne kroki

**Ustawienia zakończone!** Kontynuuj naukę:

[Rozdział 3: Podstawowe techniki generatywnej AI](../../../03-CoreGenerativeAITechniques/README.md)

## Zasoby

- [Dokumentacja Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Uwierzytelnianie bezkluczowe z Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Portal Azure AI Foundry](https://ai.azure.com/)
- [Dokumentacja Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Zastrzeżenie**:
Niniejszy dokument został przetłumaczony za pomocą usługi tłumaczenia AI [Co-op Translator](https://github.com/Azure/co-op-translator). Choć dążymy do dokładności, prosimy pamiętać, że automatyczne tłumaczenia mogą zawierać błędy lub niedokładności. Oryginalny dokument w jego języku źródłowym należy uznawać za autorytatywne źródło. W przypadku informacji krytycznych zalecane jest skorzystanie z profesjonalnego tłumaczenia wykonanego przez człowieka. Nie ponosimy odpowiedzialności za jakiekolwiek nieporozumienia lub błędne interpretacje wynikające z użycia tego tłumaczenia.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->