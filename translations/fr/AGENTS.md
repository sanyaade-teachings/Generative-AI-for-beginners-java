# AGENTS.md

## Aperçu du projet

Il s'agit d'un dépôt éducatif pour apprendre le développement en IA générative avec Java. Il propose un cours pratique complet couvrant les grands modèles de langage (LLM), l'ingénierie des prompts, les embeddings, le RAG (Retrieval-Augmented Generation) et le Model Context Protocol (MCP).

**Technologies clés :**
- Java 21
- Spring Boot 3.5.x
- Spring AI 1.1.x
- Maven
- LangChain4j
- Azure AI Foundry, Azure OpenAI, et SDK OpenAI

**Architecture :**
- Plusieurs applications Spring Boot autonomes organisées par chapitres
- Projets exemples démontrant différents patterns IA
- Prêt pour GitHub Codespaces avec des conteneurs de développement préconfigurés

## Commandes d'installation

### Prérequis
- Java 21 ou supérieur
- Maven 3.x
- Abonnement Azure avec un déploiement de modèle Azure AI Foundry (provisionner avec `azd up`)
- Azure CLI (`az`) et Azure Developer CLI (`azd`), connectés pour authentification sans clé

### Configuration de l'environnement

**Option 1 : GitHub Codespaces (recommandé)**
```bash
# Forkez le dépôt et créez un codespace depuis l'interface GitHub
# Le conteneur de développement installera automatiquement toutes les dépendances
# Attendez environ 2 minutes pour la configuration de l'environnement
```

**Option 2 : Conteneur de développement local**
```bash
# Cloner le dépôt
git clone https://github.com/microsoft/Generative-AI-for-beginners-java.git
cd Generative-AI-for-beginners-java

# Ouvrir dans VS Code avec l'extension Dev Containers
# Rouvrir dans le conteneur lorsqu'on vous le demande
```

**Option 3 : Installation locale**
```bash
# Installer les dépendances
sudo apt-get update
sudo apt-get install -y maven openjdk-21-jdk

# Vérifier l'installation
java -version
mvn -version
```

### Configuration

**Configuration Azure AI Foundry (sans clé, recommandée) :**
```bash
# Provisionnez le compte Foundry + les déploiements de modèles en tant que code
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
# azd écrit examples/basic-chat-azure/.env avec votre point de terminaison (sans clé)
```

**Configuration manuelle du point de terminaison :**
```bash
# Si vous n'avez pas utilisé azd, définissez vous-même le point de terminaison (l'authentification reste sans clé via az login)
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
# Modifiez .env : AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
```

## Flux de développement

### Structure du projet
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

### Exécution des applications

**Exécuter une application Spring Boot :**
```bash
cd [project-directory]
mvn spring-boot:run
```

**Compiler un projet :**
```bash
cd [project-directory]
mvn clean install
```

**Démarrer le serveur MCP Calculator :**
```bash
cd 04-PracticalSamples/calculator
mvn spring-boot:run
# Le serveur fonctionne sur http://localhost:8080
```

**Lancer des exemples clients :**
```bash
# Après avoir démarré le serveur dans un autre terminal
cd 04-PracticalSamples/calculator

# Client MCP direct
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.SDKClient"

# Client propulsé par l'IA (nécessite AZURE_OPENAI_ENDPOINT + connexion az)
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.LangChain4jClient"

# Bot interactif
mvn exec:java -Dexec.mainClass="com.microsoft.mcp.sample.client.Bot"
```

### Rechargement à chaud
Spring Boot DevTools est inclus dans les projets qui supportent le rechargement à chaud :
```bash
# Les modifications des fichiers Java seront rechargées automatiquement lorsqu'elles sont enregistrées
mvn spring-boot:run
```

## Instructions de test

### Exécution des tests

**Exécuter tous les tests d'un projet :**
```bash
cd [project-directory]
mvn test
```

**Exécuter les tests avec sortie détaillée :**
```bash
mvn test -X
```

**Exécuter une classe de test spécifique :**
```bash
mvn test -Dtest=CalculatorServiceTest
```

### Structure des tests
- Les fichiers de test utilisent JUnit 5 (Jupiter)
- Les classes de test se trouvent dans `src/test/java/`
- Les exemples clients dans le projet calculator sont situés dans `src/test/java/com/microsoft/mcp/sample/client/`

### Tests manuels
De nombreux exemples sont des applications interactives nécessitant des tests manuels :

1. Démarrer l'application avec `mvn spring-boot:run`
2. Tester les points de terminaison ou interagir avec la CLI
3. Vérifier que le comportement attendu correspond à la documentation dans le README.md de chaque projet

### Tests avec Azure AI Foundry
- Connectez-vous avec `az login` avant d'exécuter les exemples (authentification sans clé)
- Assurez-vous que votre compte possède le rôle Utilisateur Cognitive Services OpenAI sur la ressource
- Testez le filtrage de contenu avec l'exemple d'IA responsable au Chapitre 5

## Directives de style de code

### Conventions Java
- **Version Java :** Java 21 avec fonctionnalités modernes
- **Style :** Suivre les conventions Java standard
- **Nommage :** 
  - Classes : PascalCase
  - Méthodes/variables : camelCase
  - Constantes : UPPER_SNAKE_CASE
  - Noms de packages : minuscules

### Patterns Spring Boot
- Utiliser `@Service` pour la logique métier
- Utiliser `@RestController` pour les points de terminaison REST
- Configuration via `application.yml` ou `application.properties`
- Variables d'environnement préférées aux valeurs codées en dur
- Utiliser l'annotation `@Tool` pour les méthodes exposées par MCP

### Organisation des fichiers
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

### Dépendances
- Gérées via Maven `pom.xml`
- Spring AI BOM pour la gestion des versions
- LangChain4j pour les intégrations IA
- Parent starter Spring Boot pour les dépendances Spring

### Commentaires dans le code
- Ajouter des JavaDoc pour les API publiques
- Inclure des commentaires explicatifs pour les interactions IA complexes
- Documenter clairement les descriptions d'outils MCP

## Compilation et déploiement

### Compilation des projets

**Compiler sans tests :**
```bash
mvn clean install -DskipTests
```

**Compiler avec toutes les vérifications :**
```bash
mvn clean install
```

**Packager l'application :**
```bash
mvn package
# Crée un JAR dans le répertoire target/
```

### Dossiers de sortie
- Classes compilées : `target/classes/`
- Classes de test : `target/test-classes/`
- Fichiers JAR : `target/*.jar`
- Artifacts Maven : `target/`

### Configuration spécifique à l'environnement

**Développement :**
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

**Production :**
- Utiliser une identité managée au lieu de `az login` pour authentification sans clé
- Pointer `AZURE_OPENAI_ENDPOINT` vers votre ressource Foundry de production
- Gérer la configuration via variables d'environnement ou Azure Key Vault

### Considérations déploiement
- Il s'agit d'un dépôt éducatif avec des applications exemples
- Non conçu pour un déploiement en production tel quel
- Les exemples démontrent des patterns à adapter pour une utilisation en production
- Voir les README des projets individuels pour les notes spécifiques de déploiement

## Notes supplémentaires

### Azure AI Foundry
- **Authentification sans clé :** connexion via Microsoft Entra ID — pas de gestion de clés API
- **Provisionné comme code :** Bicep + azd (`azd up`) créent le compte et les déploiements de modèles
- Le même code compatible OpenAI s'exécute localement (`az login`) et sur Azure (identité managée)

### Travail avec plusieurs projets
Chaque projet exemple est autonome :
```bash
# Naviguer vers un projet spécifique
cd 04-PracticalSamples/[project-name]

# Chacun a son propre pom.xml et peut être construit indépendamment
mvn clean install
```

### Problèmes communs

**Incompatibilité de version Java :**
```bash
# Vérifier Java 21
java -version
# Mettre à jour JAVA_HOME si nécessaire
export JAVA_HOME=/usr/lib/jvm/msopenjdk-current
```

**Problèmes de téléchargement des dépendances :**
```bash
# Vider le cache Maven et réessayer
rm -rf ~/.m2/repository
mvn clean install
```

**Point de terminaison ou connexion non trouvée :**
```bash
# Définir le point de terminaison dans la session en cours et se connecter (sans clé)
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
az login

# Ou utiliser un fichier .env dans le répertoire du projet
echo "AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/" > .env
```

**Port déjà utilisé :**
```bash
# Spring Boot utilise par défaut le port 8080
# Modifier dans application.properties :
server.port=8081
```

### Support multilingue
- Documentation disponible en plus de 45 langues via traduction automatique
- Traductions dans le répertoire `translations/`
- Traduction gérée par le workflow GitHub Actions

### Parcours d'apprentissage
1. Commencer par [02-SetupDevEnvironment](02-SetupDevEnvironment/README.md)
2. Suivre les chapitres dans l'ordre (01 → 05)
3. Réaliser les exemples pratiques de chaque chapitre
4. Explorer les projets exemples au Chapitre 4
5. Apprendre les pratiques d'IA responsable au Chapitre 5

### Conteneur de développement
Le fichier `.devcontainer/devcontainer.json` configure :
- Environnement de développement Java 21
- Maven préinstallé
- Extensions VS Code pour Java
- Outils Spring Boot
- Intégration GitHub Copilot
- Support Docker-in-Docker
- Azure CLI

### Considérations performances
- Les déploiements Azure AI Foundry ont des quotas tokens/requêtes par minute
- Utiliser des tailles de lots adaptées pour les embeddings
- Envisager la mise en cache pour les appels API répétitifs
- Surveiller l'utilisation des tokens pour optimiser les coûts

### Notes de sécurité
- Ne jamais commiter de fichiers `.env` (déjà dans `.gitignore`)
- Préférer l'authentification sans clé (Microsoft Entra ID) aux clés API
- Utiliser les identités managées sur Azure ; `az login` pour développement local
- Suivre les directives d'IA responsable au Chapitre 5

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Avertissement** :
Ce document a été traduit à l'aide du service de traduction automatique [Co-op Translator](https://github.com/Azure/co-op-translator). Bien que nous nous efforçions d'assurer l'exactitude, veuillez noter que les traductions automatisées peuvent contenir des erreurs ou des inexactitudes. Le document original dans sa langue native doit être considéré comme la source faisant autorité. Pour les informations critiques, il est recommandé de recourir à une traduction professionnelle réalisée par un humain. Nous ne saurions être tenus responsables des malentendus ou erreurs d'interprétation découlant de l'utilisation de cette traduction.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->