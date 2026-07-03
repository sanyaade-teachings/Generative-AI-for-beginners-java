# Tutoriel Générateur d'Histoires d'Animaux pour Débutants

## Table des Matières

- [Prérequis](#prérequis)
- [Comprendre la Structure du Projet](#comprendre-la-structure-du-projet)
- [Composants Principaux Expliqués](#composants-principaux-expliqués)
  - [1. Application Principale](#1-application-principale)
  - [2. Contrôleur Web](#2-contrôleur-web)
  - [3. Service d'Histoires](#3-service-dhistoires)
  - [4. Modèles Web](#4-modèles-web)
  - [5. Configuration](#5-configuration)
- [Exécution de l’Application](#exécution-de-lapplication)
- [Comment Tout Fonctionne Ensemble](#comment-tout-fonctionne-ensemble)
- [Comprendre l’Intégration de l’IA](#comprendre-lintégration-de-lia)
- [Étapes Suivantes](#étapes-suivantes)

## Prérequis

Avant de commencer, assurez-vous d’avoir :
- Java 21 ou version ultérieure installé
- Maven pour la gestion des dépendances
- Un déploiement de modèle Azure AI Foundry (provisionnez-le avec `azd up` — voir [Chapitre 2](../../02-SetupDevEnvironment/getting-started-azure-openai.md)), connecté avec `az login` (authentification sans clé)
- Connaissances de base en Java, Spring Boot, et développement web

## Comprendre la Structure du Projet

Le projet d’histoire d’animaux comporte plusieurs fichiers importants :

```
petstory/
├── src/main/java/com/example/petstory/
│   ├── PetStoryApplication.java       # Main Spring Boot application
│   ├── PetController.java             # Web request handler
│   ├── StoryService.java              # AI story generation service
│   └── SecurityConfig.java            # Security configuration
├── src/main/resources/
│   ├── application.properties         # App configuration
│   └── templates/
│       ├── index.html                 # Upload form page
│       └── result.html               # Story display page
└── pom.xml                           # Maven dependencies
```

## Composants Principaux Expliqués

### 1. Application Principale

**Fichier :** `PetStoryApplication.java`

C’est le point d’entrée de notre application Spring Boot :

```java
@SpringBootApplication
public class PetStoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(PetStoryApplication.class, args);
    }
}
```

**Ce que cela fait :**
- L’annotation `@SpringBootApplication` active la configuration automatique et la recherche des composants
- Démarre un serveur web intégré (Tomcat) sur le port 8080
- Crée automatiquement tous les beans et services Spring nécessaires

### 2. Contrôleur Web

**Fichier :** `PetController.java`

Gère toutes les requêtes web et interactions utilisateur :

```java
@Controller
public class PetController {
    
    private final StoryService storyService;
    
    public PetController(StoryService storyService) {
        this.storyService = storyService;
    }
    
    @GetMapping("/")
    public String index() {
        return "index";  // Retourne le template index.html
    }
    
    @PostMapping("/generate-story")
    public String generateStory(@RequestParam("description") String description, 
                               Model model, 
                               RedirectAttributes redirectAttributes) {
        
        // Validation des entrées
        if (description.trim().isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please provide a description.");
            return "redirect:/";
        }
        
        // Nettoyer l'entrée pour la sécurité
        String sanitizedDescription = sanitizeInput(description);
        
        // Générer l'histoire avec gestion des erreurs
        try {
            String story = storyService.generateStory(sanitizedDescription);
            model.addAttribute("caption", sanitizedDescription);
            model.addAttribute("story", story);
            return "result";  // Retourne le template result.html
            
        } catch (Exception e) {
            // Utiliser une histoire de secours si l'IA échoue
            String fallbackStory = generateFallbackStory(sanitizedDescription);
            model.addAttribute("story", fallbackStory);
            return "result";
        }
    }
    
    private String sanitizeInput(String input) {
        return input.replaceAll("[<>\"'&]", "")  // Remove dangerous characters
                   .trim()
                   .substring(0, Math.min(input.length(), 500));  // Limiter la longueur
    }
}
```

**Fonctionnalités clés :**

1. **Gestion des routes** : `@GetMapping("/")` affiche le formulaire d’upload, `@PostMapping("/generate-story")` traite les soumissions
2. **Validation des entrées** : Vérifie les descriptions vides et les limites de longueur
3. **Sécurité** : Assainit les entrées utilisateur pour éviter les attaques XSS
4. **Gestion des erreurs** : Fournit des histoires de secours quand le service d’IA échoue
5. **Binding de modèle** : Passe les données aux templates HTML avec Spring `Model`

**Système de secours :**
Le contrôleur inclut des modèles d’histoires pré-écrits utilisés lorsque le service d’IA n’est pas disponible :

```java
private String generateFallbackStory(String description) {
    String[] storyTemplates = {
        "Meet the most wonderful pet in the world – a furry ball of energy...",
        "Once upon a time, there lived a remarkable pet whose heart was as big...",
        "In a cozy home filled with love, there lived an extraordinary pet..."
    };
    
    // Utilisez le hachage de description pour des réponses cohérentes
    int index = Math.abs(description.hashCode() % storyTemplates.length);
    return storyTemplates[index];
}
```

### 3. Service d'Histoires

**Fichier :** `StoryService.java`

Ce service communique avec Azure AI Foundry pour générer des histoires en utilisant une authentification sans clé :

```java
@Service
public class StoryService {
    
    private final OpenAIClient openAIClient;
    private final String modelName;
    
    public StoryService(@Value("${azure.openai.endpoint:}") String endpoint,
                       @Value("${azure.openai.deployment:gpt-4o-mini}") String modelName) {
        this.modelName = modelName;
        if (endpoint == null || endpoint.isBlank()) {
            endpoint = System.getenv("AZURE_OPENAI_ENDPOINT");
        }
        
        // Le point de terminaison compatible OpenAI de Foundry se trouve sous /openai/v1/
        String baseUrl = (endpoint.endsWith("/") ? endpoint : endpoint + "/") + "openai/v1/";
        
        // Authentification sans clé avec Microsoft Entra ID (pas de clé API)
        DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
        this.openAIClient = OpenAIOkHttpClient.builder()
                .baseUrl(baseUrl)
                .credential(BearerTokenCredential.create(
                        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
                .build();
    }
    
    public String generateStory(String description) {
        String systemPrompt = "You are a creative storyteller who writes fun, " +
                             "family-friendly short stories about pets. " +
                             "Keep stories under 500 words and appropriate for all ages.";
        
        String userPrompt = "Write a fun short story about a pet described as: " + description;
        
        // Configurer la requête AI
        ChatCompletionCreateParams params = ChatCompletionCreateParams.builder()
                .model(modelName)
                .addSystemMessage(systemPrompt)
                .addUserMessage(userPrompt)
                .maxCompletionTokens(500)  // Limiter la longueur de la réponse
                .temperature(0.8)          // Contrôler la créativité (0.0-1.0)
                .build();
        
        // Envoyer la requête et obtenir la réponse
        ChatCompletion response = openAIClient.chat().completions().create(params);
        
        return response.choices().get(0).message().content().orElse("");
    }
}
```

**Composants clés :**

1. **Client OpenAI** : Utilise le SDK Java officiel OpenAI configuré pour Azure AI Foundry (sans clé)
2. **Prompt système** : Définit le comportement de l’IA pour écrire des histoires d’animaux adaptées aux familles
3. **Prompt utilisateur** : Indique exactement à l’IA quelle histoire écrire selon la description
4. **Paramètres** : Contrôle la longueur de l’histoire et le niveau de créativité
5. **Gestion des erreurs** : Lance des exceptions capturées et gérées par le contrôleur

### 4. Modèles Web

**Fichier :** `index.html` (Formulaire d’Upload)

La page principale où les utilisateurs décrivent leurs animaux :

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Pet Story Generator</title>
    <!-- CSS styling -->
</head>
<body>
    <div class="container">
        <h1>Pet Story Generator</h1>
        <p>Describe your pet and we'll create a fun story about them!</p>
        
        <!-- Error message display -->
        <div th:if="${error}" class="error" th:text="${error}"></div>
        
        <!-- Story generation form -->
        <form action="/generate-story" method="post">
            <div class="form-group">
                <label for="description">Describe your pet:</label>
                <textarea id="description" name="description" 
                         placeholder="Tell us about your pet - what they look like, their personality, favorite activities..."
                         maxlength="1000" required></textarea>
            </div>
            <button type="submit" class="btn btn-primary">Generate Story</button>
        </form>
        
        <!-- Image upload section with client-side processing -->
        <div class="upload-section">
            <h2>Or Upload a Photo</h2>
            <input type="file" id="imageInput" accept="image/*" />
            <button onclick="analyzeImage()" class="upload-btn">Analyze Image</button>
        </div>
        
        <script>
            // Client-side image analysis using Transformers.js
            async function analyzeImage() {
                // Image processing code here
                // Generates description automatically from uploaded image
            }
        </script>
    </div>
</body>
</html>
```

**Fichier :** `result.html` (Affichage de l’Histoire)

Affiche l’histoire générée :

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Pet Story Result</title>
</head>
<body>
    <div class="container">
        <h1>Your Pet's Story</h1>
        
        <div class="result-section">
            <div class="result-label">Pet Description:</div>
            <div class="result-content" th:text="${caption}"></div>
        </div>
        
        <div class="result-section">
            <div class="result-label">Generated Story:</div>
            <div class="result-content" th:text="${story}"></div>
        </div>
        
        <div class="result-section" th:if="${analysisType}">
            <div class="result-label">Analysis Type:</div>
            <div class="result-content" th:text="${analysisType}"></div>
        </div>
        
        <a href="/" class="back-link">Generate Another Story</a>
    </div>
</body>
</html>
```

**Fonctionnalités du modèle :**

1. **Intégration Thymeleaf** : Utilise les attributs `th:` pour le contenu dynamique
2. **Design Responsive** : Styles CSS pour mobiles et desktop
3. **Gestion des erreurs** : Affiche les erreurs de validation aux utilisateurs
4. **Traitement côté client** : JavaScript pour l’analyse d’image (utilisant Transformers.js)

### 5. Configuration

**Fichier :** `application.properties`

Paramètres de configuration de l’application :

```properties
spring.application.name=pet-story-app

# File upload limits
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=10MB

# Logging configuration
logging.level.com.example.petstory=INFO

# Azure AI Foundry (keyless) configuration
azure.openai.endpoint=${AZURE_OPENAI_ENDPOINT:}
azure.openai.deployment=${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}
```

**Explications de la configuration :**

1. **Téléversement de fichiers** : Permet des images jusqu’à 10Mo
2. **Journalisation** : Contrôle les informations affichées dans les logs pendant l’exécution
3. **Azure AI Foundry** : Spécifie l’endpoint et le déploiement du modèle à utiliser (authentification sans clé)
4. **Sécurité** : Configuration de gestion des erreurs pour éviter d’exposer des informations sensibles

## Exécution de l’Application

### Étape 1 : Connexion et Configuration de l’Endpoint

L’authentification est sans clé (Microsoft Entra ID), donc pas de clé API. Connectez-vous et définissez votre endpoint Foundry :

**Windows (Invite de commandes) :**
```cmd
az login
set AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**Windows (PowerShell) :**
```powershell
az login
$env:AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
```

**Linux/macOS :**
```bash
az login
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

**Pourquoi c’est nécessaire :**
- Azure AI Foundry utilise Microsoft Entra ID pour authentifier les requêtes d'inférence
- L’authentification sans clé signifie qu’aucun secret n’est stocké dans votre code source ou environnement
- Votre compte doit avoir le rôle **Cognitive Services OpenAI User** sur la ressource

### Étape 2 : Compiler et Lancer

Allez dans le répertoire du projet :
```bash
cd 04-PracticalSamples/petstory
```

Compilez l’application :
```bash
mvn clean compile
```

Démarrez le serveur :
```bash
mvn spring-boot:run
```

L’application démarrera sur `http://localhost:8080`.

### Étape 3 : Tester l’Application

1. **Ouvrez** `http://localhost:8080` dans votre navigateur
2. **Décrivez** votre animal dans la zone de texte (par exemple, "Un golden retriever joueur qui adore rapporter")
3. **Cliquez** sur "Generate Story" pour obtenir une histoire générée par l’IA
4. **Sinon**, téléversez une image d’animal pour générer automatiquement une description
5. **Voyez** l’histoire créative basée sur la description de votre animal

## Comment Tout Fonctionne Ensemble

Voici le flux complet lors de la génération d’une histoire d’animal :

1. **Entrée utilisateur** : Vous décrivez votre animal dans le formulaire web
2. **Soumission du formulaire** : Le navigateur envoie une requête POST à `/generate-story`
3. **Traitement par le contrôleur** : `PetController` valide et assainit l’entrée
4. **Appel du service IA** : `StoryService` envoie la requête au modèle Azure AI Foundry
5. **Génération d’histoire** : L’IA crée une histoire créative basée sur la description
6. **Gestion de la réponse** : Le contrôleur reçoit l’histoire et l’ajoute au modèle
7. **Rendu du template** : Thymeleaf affiche `result.html` avec l’histoire
8. **Affichage** : L’utilisateur voit l’histoire générée dans son navigateur

**Gestion des erreurs :**
Si le service IA échoue :
1. Le contrôleur attrape l’exception
2. Génère une histoire de secours à partir de modèles pré-écrits
3. Affiche l’histoire de secours avec une note sur l’indisponibilité de l’IA
4. L’utilisateur reçoit toujours une histoire, garantissant une bonne expérience utilisateur

## Comprendre l’Intégration de l’IA

### Azure AI Foundry (sans clé)
L’application utilise Azure AI Foundry avec une authentification sans clé (Microsoft Entra ID) :

```java
// Authentification sans clé - pas de clé API
DefaultAzureCredential credential = new DefaultAzureCredentialBuilder().build();
this.openAIClient = OpenAIOkHttpClient.builder()
    .baseUrl(endpoint + "openai/v1/")
    .credential(BearerTokenCredential.create(
        AuthenticationUtil.getBearerTokenSupplier(credential, "https://ai.azure.com/.default")))
    .build();
```

### Prompt Engineering
Le service utilise des prompts soigneusement conçus pour obtenir de bons résultats :

```java
String systemPrompt = "You are a creative storyteller who writes fun, " +
                     "family-friendly short stories about pets. " +
                     "Keep stories under 500 words and appropriate for all ages.";
```

### Traitement des Réponses
La réponse de l’IA est extraite et validée :

```java
ChatCompletion response = openAIClient.chat().completions().create(params);
String story = response.choices().get(0).message().content().orElse("");
```

## Étapes Suivantes

Pour plus d’exemples, voir [Chapitre 04 : Exemples pratiques](../README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Avertissement** :
Ce document a été traduit à l'aide du service de traduction automatique [Co-op Translator](https://github.com/Azure/co-op-translator). Bien que nous nous efforçions d'assurer l'exactitude, veuillez noter que les traductions automatisées peuvent contenir des erreurs ou des inexactitudes. Le document original dans sa langue native doit être considéré comme la source faisant autorité. Pour les informations critiques, il est recommandé de recourir à une traduction professionnelle réalisée par un humain. Nous ne saurions être tenus responsables des malentendus ou erreurs d'interprétation découlant de l'utilisation de cette traduction.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->