# Wprowadzenie do Generatywnej AI - Edycja Java

## Czego się nauczysz

- **Podstawy generatywnej AI**, w tym LLM, inżynierię promptów, tokeny, embeddingi oraz bazy danych wektorowych
- **Porównanie narzędzi do tworzenia AI w Javie**, w tym Azure OpenAI SDK, Spring AI oraz OpenAI Java SDK
- **Poznasz Model Context Protocol** oraz jego rolę w komunikacji agentów AI

## Spis treści

- [Wprowadzenie](#wprowadzenie)
- [Szybkie przypomnienie pojęć generatywnej AI](#szybkie-przypomnienie-pojęć-generatywnej-ai)
- [Przegląd inżynierii promptów](#przegląd-inżynierii-promptów)
- [Tokeny, embeddingi i agenci](#tokeny-embeddingi-i-agenci)
- [Narzędzia i biblioteki do tworzenia AI w Javie](#narzędzia-i-biblioteki-do-tworzenia-ai-w-javie)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [Podsumowanie](#podsumowanie)
- [Kolejne kroki](#kolejne-kroki)

## Wprowadzenie

Witamy w pierwszym rozdziale Kursu Generatywnej AI dla Początkujących - Edycja Java! Ta lekcja wprowadzająca przedstawi Ci podstawowe pojęcia dotyczące generatywnej AI oraz jak z nimi pracować za pomocą Javy. Nauczysz się o kluczowych elementach aplikacji AI, w tym dużych modelach językowych (LLM), tokenach, embeddingach oraz agentach AI. Poznamy również podstawowe narzędzia Java, które będziesz używać w trakcie tego kursu.

### Szybkie przypomnienie pojęć generatywnej AI

Generatywna AI to rodzaj sztucznej inteligencji, która tworzy nowe treści, takie jak tekst, obrazy lub kod, na podstawie wzorców i zależności wyuczonych z danych. Modele generatywnej AI potrafią generować odpowiedzi przypominające ludzkie, rozumieć kontekst, a czasem nawet tworzyć treści wyglądające na napisane przez człowieka.

Podczas tworzenia aplikacji AI w Javie, będziesz pracować z **modelami generatywnej AI**, aby generować treści. Niektóre możliwości modeli generatywnej AI to:

- **Generowanie tekstu**: tworzenie tekstów przypominających ludzkie dla chatbotów, treści i uzupełniania tekstu.
- **Generowanie i analiza obrazów**: tworzenie realistycznych obrazów, poprawa zdjęć i wykrywanie obiektów.
- **Generowanie kodu**: pisanie fragmentów kodu lub skryptów.

Istnieją różne typy modeli zoptymalizowanych pod kątem konkretnych zadań. Na przykład zarówno **Małe Modele Językowe (SLM)**, jak i **Duże Modele Językowe (LLM)** mogą radzić sobie z generowaniem tekstu, przy czym LLM zwykle oferują lepszą wydajność w przypadku zadań skomplikowanych. Do zadań związanych z obrazami używa się specjalistycznych modeli wizji lub modeli multimodalnych.

![Figure: Generative AI model types and use cases.](../../../translated_images/pl/llms.225ca2b8a0d34473.webp)

Oczywiście, odpowiedzi z tych modeli nie są zawsze doskonałe. Prawdopodobnie słyszałeś o modelach "halucynujących" lub generujących nieprawdziwe informacje w sposób autorytatywny. Możesz jednak pomóc modelowi w generowaniu lepszych odpowiedzi, dostarczając mu jasne instrukcje i kontekst. Właśnie tutaj pojawia się **inżynieria promptów**.

#### Przegląd inżynierii promptów

Inżynieria promptów to praktyka projektowania efektywnych wejść, które kierują modele AI ku pożądanym wyjściom. Obejmuje ona:

- **Jasność**: formułowanie instrukcji w sposób przejrzysty i jednoznaczny.
- **Kontekst**: dostarczanie niezbędnych informacji w tle.
- **Ograniczenia**: określanie wszelkich limitów lub formatów.

Do najlepszych praktyk inżynierii promptów należą projektowanie promptów, jasne instrukcje, podział zadania, nauka jednorazowa i wielokrotna oraz dostrajanie promptów. Testowanie różnych promptów jest niezbędne, aby znaleźć najlepsze rozwiązanie dla konkretnego zastosowania.

Podczas tworzenia aplikacji będziesz pracować z różnymi typami promptów:
- **Prompty systemowe**: ustalają podstawowe zasady i kontekst zachowania modelu
- **Prompty użytkownika**: dane wejściowe od użytkowników Twojej aplikacji
- **Prompty asystenta**: odpowiedzi modelu oparte o prompty systemowe i użytkownika

> **Dowiedz się więcej**: Dowiedz się więcej o inżynierii promptów w [rozdziale Inżynieria promptów kursu GenAI dla początkujących](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### Tokeny, embeddingi i agenci

Pracując z modelami generatywnej AI, napotkasz terminy takie jak **tokeny**, **embeddingi**, **agenci** oraz **Model Context Protocol (MCP)**. Oto szczegółowy przegląd tych pojęć:

- **Tokeny**: Tokeny to najmniejsza jednostka tekstu w modelu. Mogą to być słowa, znaki lub podwyrazy. Tokeny służą do reprezentowania danych tekstowych w formacie zrozumiałym dla modelu. Na przykład zdanie "The quick brown fox jumped over the lazy dog" może zostać rozbite na tokeny jako ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] lub ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"], w zależności od strategii tokenizacji.

![Figure: Generative AI tokens example of breaking words into tokens](../../../translated_images/pl/tokens.6283ed277a2ffff4.webp)

Tokenizacja to proces rozbijania tekstu na te mniejsze jednostki. To kluczowe, ponieważ modele operują na tokenach, a nie na surowym tekście. Liczba tokenów w prompt wpływa na długość i jakość odpowiedzi modelu, ponieważ modele mają limity tokenów dla okna kontekstowego (np. 128K tokenów dla całkowitego kontekstu GPT-4o, łącznie z wejściem i wyjściem).

  W Javie możesz używać bibliotek, takich jak OpenAI SDK, które automatycznie obsługują tokenizację podczas wysyłania zapytań do modeli AI.

- **Embeddingi**: Embeddingi to reprezentacje wektorowe tokenów, które uchwytują znaczenie semantyczne. Są to reprezentacje liczbowe (zwykle tablice liczb zmiennoprzecinkowych), które pozwalają modelom rozumieć relacje między słowami i generować odpowiedzi kontekstowo odpowiednie. Podobne słowa mają podobne embeddingi, co umożliwia modelowi rozumienie koncepcji takich jak synonimy oraz relacje semantyczne.

![Figure: Embeddings](../../../translated_images/pl/embedding.398e50802c0037f9.webp)

  W Javie możesz generować embeddingi za pomocą OpenAI SDK lub innych bibliotek wspierających ich tworzenie. Embeddingi są niezbędne do zadań takich jak wyszukiwanie semantyczne, gdy chcesz znaleźć podobne treści na bazie znaczenia, a nie dokładnych dopasowań tekstu.

- **Bazy danych wektorowych**: Bazy danych wektorowych to wyspecjalizowane systemy przechowywania zoptymalizowane pod kątem embeddingów. Umożliwiają efektywne wyszukiwanie podobieństw i są kluczowe dla wzorców Generowania Wspomaganego Odzyskiwaniem (RAG), gdzie trzeba znaleźć odpowiednie informacje z dużych zbiorów na podstawie podobieństwa semantycznego, a nie dokładnego dopasowania.

![Figure: Vector database architecture showing how embeddings are stored and retrieved for similarity search.](../../../translated_images/pl/vector.f12f114934e223df.webp)

> **Uwaga**: W tym kursie nie omówimy baz wektorowych, ale warto o nich wspomnieć, ponieważ są powszechnie używane w aplikacjach produkcyjnych.

- **Agenci i MCP**: Komponenty AI, które autonomicznie wchodzą w interakcje z modelami, narzędziami i systemami zewnętrznymi. Model Context Protocol (MCP) zapewnia ustandaryzowany sposób, w jaki agenci mogą bezpiecznie uzyskiwać dostęp do zewnętrznych źródeł danych i narzędzi. Więcej informacji znajdziesz w naszym kursie [MCP dla początkujących](https://github.com/microsoft/mcp-for-beginners).

W aplikacjach AI w Javie użyjesz tokenów do przetwarzania tekstu, embeddingów do wyszukiwania semantycznego i RAG, baz wektorowych do pobierania danych oraz agentów z MCP do budowy inteligentnych systemów korzystających z narzędzi.

![Figure: how a prompt becomes a reply—tokens, vectors, optional RAG lookup, LLM thinking, and an MCP agent all in one quick flow..](../../../translated_images/pl/flow.f4ef62c3052d12a8.webp)

### Narzędzia i biblioteki do tworzenia AI w Javie

Java oferuje doskonałe narzędzia do tworzenia AI. Istnieją trzy główne biblioteki, które omówimy w trakcie kursu - OpenAI Java SDK, Azure OpenAI SDK oraz Spring AI.

Poniżej szybka tabela pokazująca, które SDK jest używane w przykładach na poszczególnych rozdziałach:

| Rozdział | Przykład | SDK |
|---------|--------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**Linki do dokumentacji SDK:**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

OpenAI SDK to oficjalna biblioteka Java dla API OpenAI. Zapewnia prosty i spójny interfejs do interakcji z modelami OpenAI, ułatwiając integrację funkcji AI w aplikacjach Java. Aplikacje z rozdziału 4, Pet Story oraz Foundry Local, demonstrują podejście OpenAI SDK w połączeniu z Azure AI Foundry.

#### Spring AI

Spring AI to kompleksowy framework, który dostarcza możliwości AI do aplikacji Spring, zapewniając spójną warstwę abstrakcji między różnymi dostawcami AI. Integruje się bezproblemowo z ekosystemem Springa, co czyni go idealnym rozwiązaniem dla aplikacji korporacyjnych w Javie potrzebujących funkcji AI.

Siła Spring AI polega na jego płynnej integracji z ekosystemem Spring, dzięki czemu łatwo budować produkcyjne aplikacje AI z użyciem znanych wzorców Spring, takich jak wstrzykiwanie zależności, zarządzanie konfiguracją oraz frameworki testowe. Użyjesz Spring AI w rozdziałach 2 i 4 do budowy aplikacji korzystających zarówno z OpenAI, jak i Model Context Protocol (MCP) w bibliotekach Spring AI.

##### Model Context Protocol (MCP)

[Model Context Protocol (MCP)](https://modelcontextprotocol.io/) to rozwijający się standard umożliwiający aplikacjom AI bezpieczną interakcję z zewnętrznymi źródłami danych i narzędziami. MCP dostarcza ustandaryzowany sposób, w jaki modele AI uzyskują dostęp do informacji kontekstowych i wykonują działania w Twoich aplikacjach.

W rozdziale 4 zbudujesz prostą usługę kalkulatora MCP, która demonstruje podstawy Model Context Protocol ze Spring AI, pokazując jak tworzyć podstawowe integracje narzędzi i architekturę usług.

#### Azure OpenAI Java SDK

Biblioteka klienta Azure OpenAI dla Javy to adaptacja REST API OpenAI, która zapewnia idiomatyczny interfejs i integrację z resztą ekosystemu Azure SDK. W rozdziale 3 zbudujesz aplikacje używając Azure OpenAI SDK, w tym aplikacje czatowe, wywoływanie funkcji oraz wzorce RAG (Retrieval-Augmented Generation).

> Uwaga: Azure OpenAI SDK pozostaje w tyle za OpenAI Java SDK pod względem funkcji, dlatego do przyszłych projektów rozważ użycie OpenAI Java SDK.

## Podsumowanie

To by było na tyle jeśli chodzi o podstawy! Teraz rozumiesz:

- Kluczowe pojęcia generatywnej AI – od LLM i inżynierii promptów po tokeny, embeddingi i bazy wektorowe
- Twoje opcje narzędziowe do tworzenia AI w Javie: Azure OpenAI SDK, Spring AI i OpenAI Java SDK
- Czym jest Model Context Protocol i jak umożliwia agentom AI współpracę z zewnętrznymi narzędziami

## Kolejne kroki

[Rozdział 2: Konfiguracja środowiska programistycznego](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Zastrzeżenie**:
Niniejszy dokument został przetłumaczony za pomocą usługi tłumaczenia AI [Co-op Translator](https://github.com/Azure/co-op-translator). Choć dążymy do dokładności, prosimy pamiętać, że automatyczne tłumaczenia mogą zawierać błędy lub niedokładności. Oryginalny dokument w jego języku źródłowym należy uznawać za autorytatywne źródło. W przypadku informacji krytycznych zalecane jest skorzystanie z profesjonalnego tłumaczenia wykonanego przez człowieka. Nie ponosimy odpowiedzialności za jakiekolwiek nieporozumienia lub błędne interpretacje wynikające z użycia tego tłumaczenia.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->