# Tutoriel du Calculateur MCP pour Débutants

## Table des Matières

- [Ce que vous allez apprendre](#ce-que-vous-allez-apprendre)
- [Prérequis](#prérequis)
- [Comprendre la Structure du Projet](#comprendre-la-structure-du-projet)
- [Explication des Composants Clés](#explication-des-composants-clés)
  - [1. Application Principale](#1-application-principale)
  - [2. Service de Calculatrice](#2-service-de-calculatrice)
  - [3. Client MCP Direct](#3-client-mcp-direct)
  - [4. Client Propulsé par l’IA](#4-client-propulsé-par-lia)
- [Exécution des Exemples](#exécution-des-exemples)
- [Comment Tout Fonctionne Ensemble](#comment-tout-fonctionne-ensemble)
- [Étapes Suivantes](#étapes-suivantes)

## Ce que vous allez apprendre

Ce tutoriel explique comment construire un service de calculatrice en utilisant le Model Context Protocol (MCP). Vous comprendrez :

- Comment créer un service que l’IA peut utiliser comme outil
- Comment configurer une communication directe avec des services MCP
- Comment les modèles d’IA peuvent choisir automatiquement les outils à utiliser
- La différence entre les appels directs au protocole et les interactions assistées par IA

## Prérequis

Avant de commencer, assurez-vous d’avoir :
- Java 21 ou version supérieure installé
- Maven pour la gestion des dépendances
- Un déploiement de modèle Azure AI Foundry (provisionnez-le avec `azd up` — voir [Chapitre 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md))
- Le [CLI Azure](https://learn.microsoft.com/cli/azure/install-azure-cli), connecté avec `az login` (authentification sans clé)
- Une compréhension de base de Java et Spring Boot

## Comprendre la Structure du Projet

Le projet calculatrice contient plusieurs fichiers importants :

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

## Explication des Composants Clés

### 1. Application Principale

**Fichier :** `McpServerApplication.java`

C’est le point d’entrée de notre service calculatrice. C’est une application Spring Boot standard avec une addition spéciale :

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

**Ce que cela fait :**
- Démarre un serveur web Spring Boot sur le port 8080
- Crée un `ToolCallbackProvider` qui rend nos méthodes de calculatrice disponibles comme outils MCP
- L’annotation `@Bean` indique à Spring de gérer ce composant pour que d’autres parties puissent l’utiliser

### 2. Service de Calculatrice

**Fichier :** `CalculatorService.java`

C’est ici que tous les calculs ont lieu. Chaque méthode est marquée avec `@Tool` pour la rendre accessible via MCP :

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
    
    // Plus d'opérations de calculatrice...
    
    private String formatResult(double a, String operator, double b, double result) {
        return String.format("%.2f %s %.2f = %.2f", a, operator, b, result);
    }
}
```

**Caractéristiques clés :**

1. **Annotation `@Tool`** : cela indique à MCP que cette méthode peut être appelée par des clients externes
2. **Descriptions Claires** : chaque outil a une description qui aide les modèles IA à comprendre quand l’utiliser
3. **Format de Retour Cohérent** : toutes les opérations renvoient des chaînes lisibles par l’humain comme « 5.00 + 3.00 = 8.00 »
4. **Gestion des Erreurs** : la division par zéro et les racines carrées négatives renvoient des messages d’erreur

**Opérations Disponibles :**
- `add(a, b)` — Additionne deux nombres
- `subtract(a, b)` — Soustrait le second du premier
- `multiply(a, b)` — Multiplie deux nombres
- `divide(a, b)` — Divise le premier par le second (contrôle de zéro)
- `power(base, exponent)` — Élève la base à la puissance de l’exposant
- `squareRoot(number)` — Calcule la racine carrée (contrôle de négatif)
- `modulus(a, b)` — Renvoie le reste de la division
- `absolute(number)` — Renvoie la valeur absolue
- `help()` — Fournit des informations sur toutes les opérations

### 3. Client MCP Direct

**Fichier :** `SDKClient.java`

Ce client communique directement avec le serveur MCP sans utiliser l’IA. Il appelle manuellement des fonctions spécifiques de la calculatrice :

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
        
        // Lister les outils disponibles
        ListToolsResult toolsList = client.listTools();
        System.out.println("Available Tools = " + toolsList);
        
        // Appeler des fonctions spécifiques de la calculatrice
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

**Ce que cela fait :**
1. **Se connecte** au serveur calculatrice à `http://localhost:8080` en utilisant le pattern builder
2. **Liste** tous les outils disponibles (nos fonctions calculatrice)
3. **Appelle** des fonctions spécifiques avec des paramètres exacts
4. **Affiche** directement les résultats

**Note :** Cet exemple utilise la dépendance Spring AI 1.1.0-SNAPSHOT, qui a introduit un pattern builder pour `WebFluxSseClientTransport`. Si vous utilisez une version stable plus ancienne, vous devrez peut-être utiliser le constructeur direct.

**Quand l’utiliser :** quand vous savez exactement quel calcul effectuer et voulez l’appeler de façon programmée.

### 4. Client Propulsé par l’IA

**Fichier :** `LangChain4jClient.java`

Ce client utilise un modèle IA (GPT-4o-mini) qui peut automatiquement décider quels outils de calculatrice utiliser :

```java
public class LangChain4jClient {
    
    public static void main(String[] args) throws Exception {
        // Configurez le modèle IA (Azure AI Foundry, authentification sans clé via Microsoft Entra ID)
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

        // Connectez-vous à notre serveur MCP de calculatrice
        McpTransport transport = new HttpMcpTransport.Builder()
                .sseUrl("http://localhost:8080/sse")
                .logRequests(true)  // Montre ce que fait l'IA
                .logResponses(true)
                .build();

        McpClient mcpClient = new DefaultMcpClient.Builder()
                .transport(transport)
                .build();

        // Donnez à l'IA l'accès à nos outils de calculatrice
        ToolProvider toolProvider = McpToolProvider.builder()
                .mcpClients(List.of(mcpClient))
                .build();

        // Créez un bot IA qui peut utiliser notre calculatrice
        Bot bot = AiServices.builder(Bot.class)
                .chatLanguageModel(model)
                .toolProvider(toolProvider)
                .build();

        // Maintenant, nous pouvons demander à l'IA d'effectuer des calculs en langage naturel
        String response = bot.chat("Calculate the sum of 24.5 and 17.3 using the calculator service");
        System.out.println(response);

        response = bot.chat("What's the square root of 144?");
        System.out.println(response);
    }
}
```

**Ce que cela fait :**
1. **Crée** une connexion au modèle IA avec votre token GitHub
2. **Connecte** l’IA à notre serveur MCP calculatrice
3. **Donne** à l’IA accès à tous nos outils calculatrice
4. **Permet** des requêtes en langage naturel comme « Calcule la somme de 24.5 et 17.3 »

**L’IA fait automatiquement :**
- Comprend que vous voulez additionner des nombres
- Choisit l’outil `add`
- Appelle `add(24.5, 17.3)`
- Renvoie le résultat dans une réponse naturelle

## Exécution des Exemples

### Étape 1 : Démarrer le Serveur Calculatrice

D’abord, connectez-vous et configurez votre point de terminaison Azure AI Foundry (nécessaire pour le client IA — authentification sans clé, pas besoin de clé API) :

**Windows :**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**Linux/macOS :**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

Démarrez le serveur :
```bash
cd 04-PracticalSamples/calculator
mvn clean spring-boot:run
```

Le serveur démarrera sur `http://localhost:8080`. Vous devriez voir :
```
Started McpServerApplication in X.XXX seconds
```

### Étape 2 : Tester avec le Client Direct

Dans un terminal **NOUVEAU** en gardant le Serveur en marche, lancez le client MCP direct :
```bash
cd 04-PracticalSamples/calculator
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient" -Dexec.classpathScope=test
```

Vous verrez une sortie comme :
```
Available Tools = [add, subtract, multiply, divide, power, squareRoot, modulus, absolute, help]
Add Result = 5.00 + 3.00 = 8.00
Square Root Result = √16.00 = 4.00
```

### Étape 3 : Tester avec le Client IA

```bash
mvn test-compile exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient" -Dexec.classpathScope=test
```

Vous verrez l’IA utiliser automatiquement les outils :
```
The sum of 24.5 and 17.3 is 41.8.
The square root of 144 is 12.
```

### Étape 4 : Fermer le Serveur MCP

Quand vous avez fini les tests, vous pouvez arrêter le client IA avec `Ctrl+C` dans son terminal. Le serveur MCP continuera à tourner jusqu’à ce que vous l’arrêtiez.
Pour arrêter le serveur, pressez `Ctrl+C` dans le terminal où il tourne.

## Comment Tout Fonctionne Ensemble

Voici le flux complet lorsque vous demandez à l’IA « Quel est le résultat de 5 + 3 ? » :

1. **Vous** posez la question à l’IA en langage naturel
2. **L’IA** analyse votre demande et comprend que vous voulez additionner
3. **L’IA** appelle le serveur MCP : `add(5.0, 3.0)`
4. **Le Service Calculatrice** effectue : `5.0 + 3.0 = 8.0`
5. **Le Service Calculatrice** renvoie : `"5.00 + 3.00 = 8.00"`
6. **L’IA** reçoit le résultat et formate une réponse naturelle
7. **Vous** obtenez : « La somme de 5 et 3 est 8 »

## Étapes Suivantes

Pour plus d’exemples, voir [Chapitre 04 : Exemples pratiques](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Avertissement** :
Ce document a été traduit à l'aide du service de traduction automatique [Co-op Translator](https://github.com/Azure/co-op-translator). Bien que nous nous efforçions d'assurer l'exactitude, veuillez noter que les traductions automatisées peuvent contenir des erreurs ou des inexactitudes. Le document original dans sa langue native doit être considéré comme la source faisant autorité. Pour les informations critiques, il est recommandé de recourir à une traduction professionnelle réalisée par un humain. Nous ne saurions être tenus responsables des malentendus ou erreurs d'interprétation découlant de l'utilisation de cette traduction.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->