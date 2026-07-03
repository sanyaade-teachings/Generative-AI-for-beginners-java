# Configuration de l’environnement de développement pour Generative AI for Java

> **Démarrage rapide :** Provisionnez vos modèles d’IA sur **Azure AI Foundry** en tant que code avec Bicep + `azd` en quelques minutes — voir le [Guide de configuration Azure AI Foundry](getting-started-azure-openai.md). L’authentification est **sans clés** (Microsoft Entra ID), il n’y a donc pas de clés API à gérer.

## Ce que vous allez apprendre

- Configurer un environnement de développement Java pour les applications d’IA
- Choisir et configurer votre environnement de développement préféré (cloud-first avec Codespaces, conteneur de développement local ou configuration locale complète)
- Tester votre configuration en vous connectant à un modèle Azure AI Foundry

## Table des matières

- [Ce que vous allez apprendre](#ce-que-vous-allez-apprendre)
- [Introduction](#introduction)
- [Étape 1 : Configurez votre environnement de développement](#étape-1-configurez-votre-environnement-de-développement)
  - [Option A : GitHub Codespaces (Recommandé)](#option-a-github-codespaces-recommandé)
  - [Option B : Conteneur de développement local](#option-b-conteneur-de-développement-local)
  - [Option C : Utilisez votre installation locale existante](#option-c-utilisez-votre-installation-locale-existante)
- [Étape 2 : Provisionnez Azure AI Foundry](#étape-2-provisionnez-azure-ai-foundry)
- [Étape 3 : Testez votre configuration](#étape-3-testez-votre-configuration)
- [Dépannage](#dépannage)
- [Résumé](#résumé)
- [Étapes suivantes](#étapes-suivantes)

## Introduction

Ce chapitre vous guidera à travers la configuration d’un environnement de développement. Nous utiliserons **Azure AI Foundry** pour les modèles tout au long de ce cours. Vous provisionnez les modèles en tant que code avec Bicep et l’Azure Developer CLI (`azd`), puis vous vous connectez avec une **authentification sans clés** (Microsoft Entra ID) — aucune clé d’API à copier ou à divulguer.

**Aucune configuration locale requise !** Vous pouvez utiliser GitHub Codespaces, qui fournit un environnement de développement complet dans votre navigateur, et provisionner Foundry à partir de là.

Nous utilisons **Azure AI Foundry** pour ce cours parce qu’il est :
- **Provisionné en tant que code** — un `azd up` déploie le compte et les déploiements de modèles
- **Sans clés** — authentifiez-vous avec votre connexion Azure ou une identité gérée
- **Prêt pour la production** — le même code fonctionne localement et dans Azure
- **Flexible** — changez de modèle en modifiant un nom de déploiement, pas votre code

> **Note** : Les déploiements Azure AI Foundry sont facturés à la token (paiement à l’usage). Voir le [guide de configuration Azure AI Foundry](getting-started-azure-openai.md) pour les détails concernant le provisioning, la région et le coût.


## Étape 1 : Configurez votre environnement de développement

<a name="quick-start-cloud"></a>

Nous avons créé un conteneur de développement pré-configuré pour minimiser le temps de configuration et vous assurer de disposer de tous les outils nécessaires pour ce cours Generative AI for Java. Choisissez votre méthode de développement préférée :

### Options de configuration de l’environnement :

#### Option A : GitHub Codespaces (Recommandé)

**Commencez à coder en 2 minutes - aucune installation locale requise !**

1. Forkez ce dépôt sur votre compte GitHub  
   > **Note** : Si vous souhaitez modifier la configuration de base, veuillez consulter la [Configuration du conteneur de développement](../../../.devcontainer/devcontainer.json)
2. Cliquez sur **Code** → onglet **Codespaces** → **...** → **Nouveau avec options...**
3. Utilisez les paramètres par défaut – cela sélectionnera la **configuration du conteneur de développement** : **Environnement de développement Java Generative AI** conteneur dev personnalisé créé pour ce cours
4. Cliquez sur **Créer un codespace**
5. Attendez ~2 minutes que l’environnement soit prêt
6. Passez à l’[Étape 2 : Provisionner Azure AI Foundry](#étape-2-provisionnez-azure-ai-foundry)

<img src="../../../translated_images/fr/codespaces.9945ded8ceb431a5.webp" alt="Capture d’écran : Sous-menu Codespaces" width="50%">

<img src="../../../translated_images/fr/image.833552b62eee7766.webp" alt="Capture d’écran : Nouveau avec options" width="50%">

<img src="../../../translated_images/fr/codespaces-create.b44a36f728660ab7.webp" alt="Capture d’écran : Options de création de codespace" width="50%">

> **Avantages des Codespaces** :
> - Aucune installation locale requise
> - Fonctionne sur n’importe quel appareil avec un navigateur
> - Préconfiguré avec tous les outils et dépendances
> - 60 heures gratuites par mois pour les comptes personnels
> - Environnement cohérent pour tous les apprenants

#### Option B : Conteneur de développement local

**Pour les développeurs qui préfèrent un développement local avec Docker**

1. Forkez et clonez ce dépôt sur votre machine locale  
   > **Note** : Si vous souhaitez modifier la configuration de base, veuillez consulter la [Configuration du conteneur de développement](../../../.devcontainer/devcontainer.json)
2. Installez [Docker Desktop](https://www.docker.com/products/docker-desktop/) et [VS Code](https://code.visualstudio.com/)
3. Installez l’[extension Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) dans VS Code
4. Ouvrez le dossier du dépôt dans VS Code
5. Quand on vous le demande, cliquez sur **Rouvrir dans le conteneur** (ou utilisez `Ctrl+Shift+P` → "Dev Containers: Reopen in Container")
6. Attendez que le conteneur se construise et démarre
7. Passez à l’[Étape 2 : Provisionner Azure AI Foundry](#étape-2-provisionnez-azure-ai-foundry)

<img src="../../../translated_images/fr/devcontainer.21126c9d6de64494.webp" alt="Capture d’écran : Configuration du conteneur de développement" width="50%">

<img src="../../../translated_images/fr/image-3.bf93d533bbc84268.webp" alt="Capture d’écran : Construction du conteneur de développement terminée" width="50%">

#### Option C : Utilisez votre installation locale existante

**Pour les développeurs disposant déjà d’un environnement Java**

Prérequis :  
- [Java 21+](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html)  
- [Maven 3.9+](https://maven.apache.org/download.cgi)  
- [VS Code](https://code.visualstudio.com) ou votre IDE préféré

Étapes :  
1. Clonez ce dépôt sur votre machine locale  
2. Ouvrez le projet dans votre IDE  
3. Passez à l’[Étape 2 : Provisionner Azure AI Foundry](#étape-2-provisionnez-azure-ai-foundry)

> **Astuce pro** : Si vous avez une machine peu puissante mais souhaitez VS Code local, utilisez GitHub Codespaces ! Vous pouvez connecter votre VS Code local à un Codespace hébergé dans le cloud pour profiter du meilleur des deux mondes.

<img src="../../../translated_images/fr/image-2.fc0da29a6e4d2aff.webp" alt="Capture d’écran : Instance locale du conteneur de développement créée" width="50%">


## Étape 2 : Provisionnez Azure AI Foundry

Déployez les modèles d’IA du cours dans Azure AI Foundry en tant que code. Depuis la racine du dépôt :

```bash
cd 02-SetupDevEnvironment
azd auth login
az login
azd up
```
  
`azd` vous demande un nom d’environnement et une région, provisionne un compte Azure AI Foundry avec les déploiements `gpt-4o-mini` et `text-embedding-3-small`, et écrit le point de terminaison dans le fichier `.env` de l’exemple — le tout avec une authentification **sans clés** (pas de clés API).

> **Guide complet :** Voir le [Guide de configuration Azure AI Foundry](getting-started-azure-openai.md) pour les prérequis, une alternative manuelle (portail), des conseils sur la région, ainsi que les notes sur les coûts et le nettoyage.

## Étape 3 : Testez votre configuration

Une fois vos modèles Foundry provisionnés, testez la connexion avec l’application exemple dans [`02-SetupDevEnvironment/examples/basic-chat-azure`](../../../02-SetupDevEnvironment/examples/basic-chat-azure).

1. Ouvrez le terminal dans votre environnement de développement.  
2. Naviguez vers l’exemple :  
   ```bash
   cd 02-SetupDevEnvironment/examples/basic-chat-azure
   ```
3. Assurez-vous d’être connecté (l’authentification sans clés nécessite un jeton) :  
   ```bash
   az login
   ```
   > Si vous avez lancé `azd up`, le fichier `.env` avec votre point de terminaison a déjà été écrit pour vous.  
4. Lancez l’application :  
   ```bash
   mvn clean spring-boot:run
   ```
  
Vous devriez voir une réponse du modèle `gpt-4o-mini`.

### Comprendre le code exemple

L’exemple sous `examples/basic-chat-azure` est une application Spring Boot qui utilise **Spring AI** pour se connecter à Azure AI Foundry avec authentification sans clés.

**Ce que fait ce code :**  
- **Se connecte** à Azure AI Foundry en utilisant votre connexion Azure (Microsoft Entra ID) — pas de clé API  
- **Envoie** un prompt au modèle `gpt-4o-mini`  
- **Reçoit** et affiche la réponse de l’IA  
- **Valide** que votre configuration fonctionne correctement

**Dépendance clé** (dans `pom.xml`) :  
```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-model-azure-openai</artifactId>
</dependency>
```
  
**Configuration** (`application.yml`) :  
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
  
## Résumé

Super ! Vous avez maintenant tout configuré :

- Modèles Azure AI Foundry provisionnés en tant que code avec Bicep + `azd`  
- Votre environnement de développement Java opérationnel (que ce soit Codespaces, conteneurs de développement ou local)  
- Connexion à Azure AI Foundry avec authentification sans clés (Microsoft Entra ID) — sans clés API  
- Testé avec un exemple simple qui interagit avec votre modèle

## Étapes suivantes

[Chapitre 3 : Techniques de base d’IA générative](../03-CoreGenerativeAITechniques/README.md)

## Dépannage

Vous rencontrez des problèmes ? Voici les problèmes courants et leurs solutions :

- **Échec d’authentification (401/403) ?**  
  - Exécutez `az login` — l’authentification est sans clés, vous devez donc être connecté  
  - Vérifiez que votre compte dispose du rôle **Cognitive Services OpenAI User** sur la ressource  
  - Si vous venez de provisionner, attendez une minute que l’affectation de rôle soit propagée

- **Maven introuvable ?**  
  - Si vous utilisez les conteneurs dev/Codespaces, Maven devrait être préinstallé  
  - Pour une installation locale, assurez-vous que Java 21+ et Maven 3.9+ sont installés  
  - Essayez `mvn --version` pour vérifier l’installation

- **`azd` introuvable ou le provisioning échoue ?**  
  - Installez l’[Azure Developer CLI](https://aka.ms/azure-dev/install) et exécutez `azd auth login`  
  - Choisissez une région où `gpt-4o-mini` est disponible (ex. `eastus2`)  
  - Voir le [guide de configuration Azure AI Foundry](getting-started-azure-openai.md) pour plus de détails

- **Le conteneur de développement ne démarre pas ?**  
  - Assurez-vous que Docker Desktop est en cours d’exécution (pour le développement local)  
  - Essayez de reconstruire le conteneur : `Ctrl+Shift+P` → "Dev Containers: Rebuild Container"

- **Erreurs de compilation de l’application ?**  
  - Assurez-vous d’être dans le bon répertoire : `02-SetupDevEnvironment/examples/basic-chat-azure`  
  - Essayez de nettoyer et reconstruire : `mvn clean compile`

> **Besoin d’aide ?** : Toujours des problèmes ? Ouvrez un ticket dans le dépôt et nous vous aiderons.

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Avertissement** :
Ce document a été traduit à l'aide du service de traduction automatique [Co-op Translator](https://github.com/Azure/co-op-translator). Bien que nous nous efforçions d'assurer l'exactitude, veuillez noter que les traductions automatisées peuvent contenir des erreurs ou des inexactitudes. Le document original dans sa langue native doit être considéré comme la source faisant autorité. Pour les informations critiques, il est recommandé de recourir à une traduction professionnelle réalisée par un humain. Nous ne saurions être tenus responsables des malentendus ou erreurs d'interprétation découlant de l'utilisation de cette traduction.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->