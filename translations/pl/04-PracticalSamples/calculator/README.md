# Samouczek kalkulatora MCP dla początkujących

## Spis treści

- [Czego się nauczysz](#czego-si%C4%99-nauczysz)
- [Wymagania wstępne](#wymagania-wst%C4%99pne)
- [Zrozumienie struktury projektu](#zrozumienie-struktury-projektu)
- [Wyjaśnienie kluczowych komponentów](#wyja%C5%9Bnienie-kluczowych-komponent%C3%B3w)
  - [1. Główna aplikacja](#1-g%C5%82%C3%B3wna-aplikacja)
  - [2. Serwis kalkulatora](#2-serwis-kalkulatora)
  - [3. Bezpośredni klient MCP](#3-bezpo%C5%9Bredni-klient-mcp)
  - [4. Klient zasilany SI](#4-klient-zasilany-si)
- [Uruchamianie przykładów](#uruchamianie-przyk%C5%82ad%C3%B3w)
- [Jak to wszystko działa razem](#jak-to-wszystko-dzia%C5%82a-razem)
- [Kolejne kroki](#kolejne-kroki)

## Czego się nauczysz

Ten samouczek wyjaśnia, jak zbudować serwis kalkulatora przy użyciu Model Context Protocol (MCP). Zrozumiesz:

- Jak stworzyć serwis, którego SI może używać jako narzędzia
- Jak skonfigurować bezpośrednią komunikację z serwisami MCP
- Jak modele SI mogą automatycznie wybierać, których narzędzi używać
- Różnicę między bezpośrednimi wywołaniami protokołu a interakcjami wspieranymi przez SI

## Wymagania wstępne

Przed rozpoczęciem upewnij się, że masz:
- Zainstalowaną Javę 21 lub nowszą
- Maven do zarządzania zależnościami
- Wdrożenie modelu Azure AI Foundry (załóż je za pomocą `azd up` — zobacz [Rozdział 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli), zalogowane poleceniem `az login` (uwierzytelnianie bezkluczowe)
- Podstawową znajomość Javy i Spring Boot

## Zrozumienie struktury projektu

Projekt kalkulatora zawiera kilka ważnych plików:

```
calculator/
├── src/main/java/com/microsoft/mcp/sample/server/
│   ├── McpServerApplication.java          # Main Spring Boot app
│   └── service/CalculatorService.java     # Calculator operations
└── src/test/java/com/microsoft/mcp/sample/client/
    ├── SDKClient.java                     # Direct MCP communication
    ├── LangChain4jClient.java            # AI-powered client
    └── Bot.java                          # Simple chat interface
```

## Wyjaśnienie kluczowych komponentów

### 1. Główna aplikacja

**Plik:** `McpServerApplication.java`

Jest to punkt startowy naszego serwisu kalkulatora. To standardowa aplikacja Spring Boot z jednym specjalnym dodatkiem:

```java
@SpringBootApplication
public class McpServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(McpServerApplication.class, args);
    }
    
    @Bean
    public ToolCallbackProvider calculatorTools(CalculatorService calculator) {
        return MethodToolCallbackProvider.builder().toolObjects(calculator).build();
    }
}
```

**Co to robi:**
- Uruchamia serwer webowy Spring Boot na porcie 8080
- Tworzy `ToolCallbackProvider`, który udostępnia nasze metody kalkulatora jako narzędzia MCP
- Adnotacja `@Bean` mówi Springowi, aby zarządzał tym jako komponentem, którego mogą używać inne części

### 2. Serwis kalkulatora

**Plik:** `CalculatorService.java`

Tu odbywa się cała matematyka. Każda metoda oznaczona jest adnotacją `@Tool`, aby była dostępna przez MCP:

```java
@Service
public class CalculatorService {

    @Tool(description = "Add two numbers together")
    public String add(double a, double b) {
        double result = a + b;
        return formatResult(a, "+", b, result);
    }

    @Tool(description = "Subtract the second number from the first number")
    public String subtract(double a, double b) {
        double result = a - b;
        return formatResult(a, "-", b, result);
    }
    
    // Więcej operacji kalkulatora...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Kluczowe cechy:**

1. **Adnotacja `@Tool`**: Informuje MCP, że metoda może być wywoływana przez zewnętrznych klientów
2. **Jasne opisy**: Każde narzędzie ma opis, który pomaga modelom SI zrozumieć, kiedy go użyć
3. **Spójny format zwrotu**: Wszystkie operacje zwracają czytelne dla człowieka ciągi znaków, np. "5.00 + 3.00 = 8.00"
4. **Obsługa błędów**: Dzielenie przez zero i pierwiastek z liczby ujemnej zwracają komunikaty o błędach

**Dostępne operacje:**
- `add(a, b)` - Dodaje dwie liczby
- `subtract(a, b)` - Odejmuje drugą od pierwszej
- `multiply(a, b)` - Mnoży dwie liczby
- `divide(a, b)` - Dzieli pierwszą przez drugą (ze sprawdzeniem dzielenia przez zero)
- `power(base, exponent)` - Potęguje podstawę do wykładnika
- `squareRoot(number)` - Oblicza pierwiastek kwadratowy (ze sprawdzeniem ujemnej wartości)
- `modulus(a, b)` - Zwraca resztę z dzielenia
- `absolute(number)` - Zwraca wartość bezwzględną
- `help()` - Zwraca informacje o wszystkich operacjach

### 3. Bezpośredni klient MCP

**Plik:** `SDKClient.java`

Ten klient komunikuje się bezpośrednio z serwerem MCP, nie używając SI. Ręcznie wywołuje konkretne funkcje kalkulatora:

```java
public class SDKClient {
    
    public static void main(String[] args) {
        McpClientTransport transport = WebFluxSseClientTransport.builder(
            WebClient.builder().baseUrl("http://localhost:8080")
        ).build();
        new SDKClient(transport).run();
    }
    
    public void run() {
        var client = McpClient.sync(this.transport).build();
        client.initialize();
        
        // Wyświetl dostępne narzędzia
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Wywołaj określone funkcje kalkulatora
        CallToolResult resultAdd = client.callTool(
            new CallToolRequest("add", Map.of("a", 5.0, "b", 3.0))
        );
        System.out.println("Add Result = " + resultAdd);
        
        CallToolResult resultSqrt = client.callTool(
            new CallToolRequest("squareRoot", Map.of("number", 16.0))
        );
        System.out.println("Square Root Result = " + resultSqrt);
        
        client.closeGracefully();
    }
}
```

**Co to robi:**
1. **Łączy się** z serwerem kalkulatora pod adresem `http://localhost:8080` używając wzorca buildera
2. **Wyświetla** listę wszystkich dostępnych narzędzi (naszych funkcji kalkulatora)
3. **Wywołuje** konkretne funkcje z dokładnymi parametrami
4. **Drukuje** wyniki bezpośrednio

**Uwaga:** Ten przykład używa zależności Spring AI 1.1.0-SNAPSHOT, która wprowadziła wzorzec buildera dla `WebFluxSseClientTransport`. Jeżeli używasz starszej stabilnej wersji, być może trzeba będzie użyć bezpośredniego konstruktora.

**Kiedy to używać:** Gdy dokładnie wiesz, jakie obliczenie chcesz wykonać i chcesz wywołać je programowo.

### 4. Klient zasilany SI

**Plik:** `LangChain4jClient.java`

Ten klient używa modelu SI (GPT-4o-mini), który automatycznie decyduje, których narzędzi kalkulatora użyć:

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Skonfiguruj model AI (Azure AI Foundry, uwierzytelnianie bezkluczowe przez Microsoft Entra ID)
        String endpoint = System.getenv("AZURE_OPENAI_ENDPOINT");
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1";
        String token = new DefaultAzureCredentialBuilder().build()
                .getToken(new TokenRequestContext().addScopes("https://ai.azure.com/.default"))
                .block().getToken();
        ChatLanguageModel model = OpenAiOfficialChatModel.builder()
                .baseUrl(baseUrl)
                .apiKey(token)
                .modelName("gpt-4o-mini")
                .build();

        // Połącz się z naszym serwerem kalkulatora MCP
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Pokazuje, co robi AI
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Daj AI dostęp do naszych narzędzi kalkulatora
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Utwórz bota AI, który może korzystać z naszego kalkulatora
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Teraz możemy prosić AI o wykonywanie obliczeń w języku naturalnym
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Co to robi:**
1. **Tworzy** połączenie z modelem SI z uwierzytelnianiem bezkluczowym (Microsoft Entra ID)
2. **Łączy** SI z naszym serwerem MCP kalkulatora
3. **Daje** SI dostęp do wszystkich naszych narzędzi kalkulatora
4. **Pozwala** na zapytania w języku naturalnym, np. "Oblicz sumę 24.5 i 17.3"

**SI automatycznie:**
- Rozumie, że chcesz dodać liczby
- Wybiera narzędzie `add`
- Wywołuje `add(24.5, 17.3)`
- Zwraca wynik w naturalnym formacie odpowiedzi

## Uruchamianie przykładów

### Krok 1: Uruchom serwer kalkulatora

Najpierw zaloguj się i ustaw swój punkt końcowy Azure AI Foundry (potrzebne dla klienta SI — uwierzytelnianie bezkluczowe, bez klucza API):

**Windows:**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**Linux/macOS:**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

Uruchom serwer:
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

Serwer uruchomi się pod adresem `http://localhost:8080`. Powinieneś zobaczyć:
```
Started McpServerApplication in X.XXX seconds
```

### Krok 2: Przetestuj klienta bezpośredniego

W **NOWYM** terminalu z włączonym serwerem uruchom bezpośredniego klienta MCP:
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Zobaczysz output taki jak:
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Krok 3: Przetestuj klienta z SI

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Zobaczysz, jak SI automatycznie używa narzędzi:
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Krok 4: Zamknij serwer MCP

Po zakończeniu testów możesz zatrzymać klienta SI, naciskając `Ctrl+C` w jego terminalu. Serwer MCP będzie działał dalej, dopóki go nie zatrzymasz.
Aby zatrzymać serwer, naciśnij `Ctrl+C` w terminalu, w którym jest uruchomiony.

## Jak to wszystko działa razem

Oto pełny przebieg, gdy zapytasz SI „Ile to 5 + 3?”:

1. **Ty** zadajesz SI pytanie w języku naturalnym
2. **SI** analizuje prośbę i rozpoznaje, że chcesz wykonac dodawanie
3. **SI** wywołuje serwer MCP: `add(5.0, 3.0)`
4. **Serwis kalkulatora** wykonuje: `5.0 + 3.0 = 8.0`
5. **Serwis kalkulatora** zwraca: `"5.00 + 3.00 = 8.00"`
6. **SI** odbiera wynik i formatuje odpowiedź naturalnym językiem
7. **Ty** otrzymujesz: "Suma 5 i 3 to 8"

## Kolejne kroki

Więcej przykładów znajdziesz w [Rozdziale 04: Praktyczne przykłady](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Zastrzeżenie**:
Niniejszy dokument został przetłumaczony za pomocą usługi tłumaczenia AI [Co-op Translator](https://github.com/Azure/co-op-translator). Choć dążymy do dokładności, prosimy pamiętać, że automatyczne tłumaczenia mogą zawierać błędy lub niedokładności. Oryginalny dokument w jego języku źródłowym należy uznawać za autorytatywne źródło. W przypadku informacji krytycznych zalecane jest skorzystanie z profesjonalnego tłumaczenia wykonanego przez człowieka. Nie ponosimy odpowiedzialności za jakiekolwiek nieporozumienia lub błędne interpretacje wynikające z użycia tego tłumaczenia.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->