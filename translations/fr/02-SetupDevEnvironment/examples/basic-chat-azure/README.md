# Chat de base avec Azure AI Foundry - Exemple de bout en bout

Cet exemple est une application Spring Boot simple qui se connecte à un modèle **Azure AI Foundry** en utilisant une **authentification sans clé** (Microsoft Entra ID) et teste votre configuration. Elle utilise le `ChatClient` de Spring AI.

## Table des matières

- [Prérequis](#prérequis)
- [Démarrage rapide](#démarrage-rapide)
- [Comment fonctionne l’authentification](#comment-fonctionne-lauthentification)
- [Exécution de l’application](#exécution-de-lapplication)
  - [Utilisation de Maven](#utilisation-de-maven)
  - [Utilisation de VS Code](#utilisation-de-vs-code)
  - [Sortie attendue](#sortie-attendue)
- [Référence de configuration](#référence-de-configuration)
  - [Variables d’environnement](#variables-denvironnement)
  - [Configuration Spring](#configuration-spring)
- [Dépannage](#dépannage)
  - [Problèmes courants](#problèmes-courants)
  - [Mode debug](#mode-debug)
- [Étapes suivantes](#étapes-suivantes)
- [Ressources](#ressources)

## Prérequis

Avant d’exécuter cet exemple, assurez-vous de disposer de :

- Une ressource Azure AI Foundry avec un déploiement `gpt-4o-mini` — provisionnez-la avec `azd up` ou manuellement via le [guide de configuration Azure AI Foundry](../../getting-started-azure-openai.md)
- Le rôle **Utilisateur des services cognitifs OpenAI** sur cette ressource (les templates Bicep l’assignent automatiquement)
- Le [CLI Azure (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli), connecté avec `az login`
- Java 21+ et Maven 3.9+

> **Aucune clé API requise** — l’authentification est sans clé via Microsoft Entra ID.

## Démarrage rapide

```bash
# 1. Naviguez vers le projet
cd 02-SetupDevEnvironment/examples/basic-chat-azure

# 2. Connectez-vous pour que l'authentification sans clé puisse obtenir un jeton
az login

# 3. Configurez le point de terminaison
#    - Si vous avez exécuté `azd up`, .env a été écrit pour vous (sauter cette étape).
#    - Sinon, copiez le modèle et définissez AZURE_OPENAI_ENDPOINT :
cp .env.example .env

# 4. Exécutez l'application
mvn spring-boot:run
```

## Comment fonctionne l’authentification

Cet exemple s’authentifie avec **Microsoft Entra ID** — il n’y a pas de clé API.

Lorsque seule la propriété `spring.ai.azure.openai.endpoint` est définie (sans clé API), Spring AI construit le client Azure OpenAI avec [`DefaultAzureCredential`](https://learn.microsoft.com/java/api/com.azure.identity.defaultazurecredential). Cette crédential trouve automatiquement un jeton depuis votre session locale `az login`, ou depuis une identité managée lors de l’exécution dans Azure — ce qui permet que le même code fonctionne dans les deux environnements sans modification.

## Exécution de l’application

### Utilisation de Maven

```bash
mvn spring-boot:run
```

### Utilisation de VS Code

1. Ouvrez le projet dans VS Code  
2. Appuyez sur `F5` ou utilisez le panneau "Run and Debug"  
3. Sélectionnez la configuration "Spring Boot-BasicChatApplication"  

> **Note** : La configuration VS Code charge automatiquement votre fichier .env

### Sortie attendue

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

## Référence de configuration

### Variables d’environnement

| Variable | Description | Obligatoire | Exemple |
|----------|-------------|-------------|---------|
| `AZURE_OPENAI_ENDPOINT` | URL de l’endpoint Foundry (Azure OpenAI) | Oui | `https://my-resource.openai.azure.com/` |
| `AZURE_OPENAI_DEPLOYMENT` | Nom du déploiement du modèle de chat | Non | `gpt-4o-mini` (par défaut) |

> Il n’y a **pas** de variable clé API — l’authentification est sans clé (Microsoft Entra ID via `az login`).

### Configuration Spring

Le fichier `application.yml` configure :  
- **Endpoint** : `${AZURE_OPENAI_ENDPOINT}` - depuis la variable d’environnement  
- **Déploiement** : `${AZURE_OPENAI_DEPLOYMENT:gpt-4o-mini}` - depuis la variable d’environnement avec valeur par défaut  
- **Auth** : sans clé — aucune `api-key` n’est définie, donc Spring AI utilise `DefaultAzureCredential`  
- **Température** : `0.7` - Contrôle la créativité (0.0 = déterministe, 1.0 = créatif)  
- **Max Tokens** : `500` - Longueur maximale de la réponse  

## Dépannage

### Problèmes courants

<details>
<summary><strong>Erreur : 401 / "PermissionDenied" / erreurs de jeton</strong></summary>

- Exécutez `az login` — l’authentification sans clé nécessite une connexion active pour obtenir un jeton  
- Vérifiez que votre compte possède le rôle **Utilisateur des services cognitifs OpenAI** sur la ressource  
- Si vous venez d’assigner ce rôle, attendez une minute pour la propagation  
- Confirmez que vous êtes dans le bon locataire/abonnement (`az account show`)  
</details>

<details>
<summary><strong>Erreur : "L’endpoint n’est pas valide" / erreurs de connexion</strong></summary>

- Assurez-vous que `AZURE_OPENAI_ENDPOINT` est l’URL complète de base (ex. : `https://your-resource.openai.azure.com/`)  
- Vérifiez la cohérence du slash final  
- Confirmez que l’endpoint correspond à la ressource provisionnée (`azd env get-values`)  
</details>

<details>
<summary><strong>Erreur : "Le déploiement n’a pas été trouvé"</strong></summary>

- Vérifiez que `AZURE_OPENAI_DEPLOYMENT` correspond à un nom de déploiement dans Azure  
- Assurez-vous que le modèle est bien déployé et actif  
- Le nom de déploiement par défaut est `gpt-4o-mini`  
</details>

<details>
<summary><strong>VS Code : variables d’environnement non chargées</strong></summary>

- Assurez-vous que votre fichier `.env` est à la racine du projet (au même niveau que `pom.xml`)  
- Essayez d’exécuter `mvn spring-boot:run` dans le terminal intégré de VS Code  
- Vérifiez que l’extension Java de VS Code est correctement installée  
</details>

### Mode debug

Pour activer la journalisation détaillée, décommentez ces lignes dans `application.yml` :

```yaml
logging:
  level:
    org.springframework.ai: DEBUG
    com.azure: DEBUG
```

## Étapes suivantes

**Installation terminée !** Continuez votre parcours d’apprentissage :

[Chapitre 3 : Techniques principales d’IA générative](../../../03-CoreGenerativeAITechniques/README.md)

## Ressources

- [Documentation Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)
- [Authentification sans clé avec Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)
- [Portail Azure AI Foundry](https://ai.azure.com/)
- [Documentation Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Avertissement** :
Ce document a été traduit à l'aide du service de traduction automatique [Co-op Translator](https://github.com/Azure/co-op-translator). Bien que nous nous efforçions d'assurer l'exactitude, veuillez noter que les traductions automatisées peuvent contenir des erreurs ou des inexactitudes. Le document original dans sa langue native doit être considéré comme la source faisant autorité. Pour les informations critiques, il est recommandé de recourir à une traduction professionnelle réalisée par un humain. Nous ne saurions être tenus responsables des malentendus ou erreurs d'interprétation découlant de l'utilisation de cette traduction.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->