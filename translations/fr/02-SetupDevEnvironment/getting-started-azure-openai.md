# Configuration de l'environnement de développement pour Azure AI Foundry

> Ce guide configure les modèles **Azure AI Foundry** pour les applications Java AI de ce cours, en utilisant une authentification **sans clé** (Microsoft Entra ID) — aucune clé API à gérer. Nouveau dans les outils ? Commencez par le [guide de l'environnement de développement](./README.md).

Ce guide configure les modèles **Azure AI Foundry** pour les applications Java AI de ce cours. Vous avez deux options :

- **Option A — Provisionner avec `azd` + Bicep (recommandé) :** une commande déploie le compte Foundry et les modèles en tant que code. Pas de clics dans le portail.
- **Option B — Créer les ressources manuellement** dans le portail Azure AI Foundry.

Les deux options utilisent une **authentification sans clé** (Microsoft Entra ID) — il n’y a pas de clés API à copier ou à divulguer.

## Table des matières

- [Ce qui est créé](#ce-qui-est-créé)
- [Prérequis](#prérequis)
- [Option A : Provisionner avec azd + Bicep (Recommandé)](#option-a-provision-with-azd--bicep-recommended)
- [Option B : Créer les ressources manuellement](#option-b-créer-les-ressources-manuellement)
- [Configurer votre environnement](#configurer-votre-environnement)
- [Tester votre configuration](#tester-votre-configuration)
- [Et après ?](#et-après)
- [Ressources](#ressources)
- [Ressources supplémentaires](#ressources-supplémentaires)

## Ce qui est créé

Les modèles Bicep dans [`infra/`](../../../02-SetupDevEnvironment/infra) provisionnent :

- Un compte **Azure AI Foundry** (`Microsoft.CognitiveServices/accounts`, type `AIServices`) avec un projet
- Un déploiement **chat** — `gpt-4o-mini`
- Un déploiement **embedding** — `text-embedding-3-small` (utilisé dans les chapitres suivants)
- Un **attribution de rôle sans clé** (`Cognitive Services OpenAI User`) pour que vous puissiez vous connecter avec `az login` au lieu de gérer des clés

## Prérequis

- Un [abonnement Azure](https://azure.microsoft.com/free/)
- [Azure Developer CLI (`azd`)](https://aka.ms/azure-dev/install)
- [Azure CLI (`az`)](https://learn.microsoft.com/cli/azure/install-azure-cli)
- [Java 21+](https://learn.microsoft.com/java/openjdk/download) et [Maven 3.9+](https://maven.apache.org/download.cgi)

## Option A : Provisionner avec azd + Bicep (Recommandé)

Depuis le dossier `02-SetupDevEnvironment` :

```bash
cd 02-SetupDevEnvironment

# Se connecter (les deux outils)
azd auth login
az login

# Approvisionner le compte Foundry + déploiements des modèles
azd up
```
  
`azd` vous invite à saisir un **nom d’environnement** (par exemple `genai-java`) et une **région**. Choisissez une région où `gpt-4o-mini` et `text-embedding-3-small` sont disponibles — par exemple `eastus2` ou `swedencentral`.

Lorsque le provisionnement est terminé, azd :

1. Déploie tout ce qui est défini dans [`infra/main.bicep`](../../../02-SetupDevEnvironment/infra/main.bicep).
2. Exécute un hook post-provisionnement qui écrit [`examples/basic-chat-azure/.env`](../../../02-SetupDevEnvironment/examples/basic-chat-azure) avec votre point de terminaison et les noms de déploiement (sans secrets).

> **Astuce :** Relancez `azd up` à tout moment pour appliquer des modifications. Lancez `azd down` pour tout supprimer et arrêter d’encourir des coûts.

Pour voir les paramètres générés :

```bash
azd env get-values
```
  
Passez maintenant à [Tester votre configuration](#tester-votre-configuration).

## Option B : Créer les ressources manuellement

Vous préférez le portail ? Créez les ressources à la main :

1. Allez sur le [portail Azure AI Foundry](https://ai.azure.com/) et connectez-vous.
2. **Créez un projet** (cela crée aussi une ressource AI Foundry). Donnez-lui un nom comme `GenAIJava`.
3. Dans votre projet, ouvrez **Modèles + points de terminaison** → **Déployer un modèle** → **Déployer un modèle de base**.
4. Déployez **gpt-4o-mini** (nom de déploiement `gpt-4o-mini`). Répétez pour **text-embedding-3-small** si vous souhaitez les exemples d’embedding.
5. Depuis **Vue d’ensemble**, copiez le **point de terminaison** (par exemple `https://<ressource>.openai.azure.com/`).
6. Accordez-vous un accès sans clé : dans la ressource, ouvrez **Contrôle d’accès (IAM)** → **Ajouter une attribution de rôle** → attribuez **Cognitive Services OpenAI User** à votre compte.

> **Toujours des difficultés ?** Consultez la [documentation Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/create-projects).

## Configurer votre environnement

**Si vous avez utilisé l’option A (`azd up`)**, votre fichier de paramètres est déjà écrit — rien à configurer. Passez à [Tester votre configuration](#tester-votre-configuration).

**Si vous avez utilisé l’option B (manuel)**, créez vous-même le fichier `.env` de l’exemple :

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure
cp .env.example .env
```
  
Modifiez `.env` avec votre point de terminaison (pas de clé — l’authentification est sans clé) :

```bash
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_DEPLOYMENT=gpt-4o-mini
```
  
> **Note de sécurité :** Il n’y a pas de clé API à stocker. Vous vous authentifiez avec Microsoft Entra ID via `az login` (localement) ou une identité managée (dans Azure). Le fichier `.env` contient seulement des paramètres non secrets et est déjà couvert par `.gitignore`.

## Tester votre configuration

Assurez-vous d’être connecté pour que l’authentification sans clé puisse obtenir un jeton, puis exécutez l’exemple :

```bash
cd 02-SetupDevEnvironment/examples/basic-chat-azure

az login          # si vous n'êtes pas déjà connecté
mvn clean spring-boot:run
```
  
Vous devriez voir une réponse du modèle `gpt-4o-mini` !

> **Utilisateurs VS Code :** Appuyez sur `F5` pour exécuter. L’application charge automatiquement votre `.env`.

> **Exemple complet :** Consultez l’[exemple Basic Chat avec Azure AI Foundry](./examples/basic-chat-azure/README.md) pour les détails et la résolution des problèmes.

## Et après ?

**Configuration terminée !** Vous avez maintenant :  
- Azure AI Foundry avec `gpt-4o-mini` et `text-embedding-3-small` déployés  
- Authentification sans clé (Microsoft Entra ID) — pas de clés à gérer  
- Un `.env` local avec votre point de terminaison et les noms de déploiement  
- Un environnement de développement Java prêt à l’emploi  

**Continuez vers** [Chapitre 3 : Techniques de base de l’IA générative](../03-CoreGenerativeAITechniques/README.md) pour commencer à créer des applications IA !

## Ressources

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install)  
- [Authentification sans clé avec Microsoft Entra ID](https://learn.microsoft.com/azure/ai-foundry/foundry-models/how-to/configure-entra-id)  
- [Documentation Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/)  
- [Documentation Spring AI Azure OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/azure-openai-chat.html)  
- [Azure OpenAI Java SDK](https://learn.microsoft.com/java/api/overview/azure/ai-openai-readme)

## Ressources supplémentaires

- [Télécharger VS Code](https://code.visualstudio.com/Download)  
- [Obtenir Docker Desktop](https://www.docker.com/products/docker-desktop)  
- [Configuration du conteneur de développement](../../../.devcontainer/devcontainer.json)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Avertissement** :
Ce document a été traduit à l'aide du service de traduction automatique [Co-op Translator](https://github.com/Azure/co-op-translator). Bien que nous nous efforçions d'assurer l'exactitude, veuillez noter que les traductions automatisées peuvent contenir des erreurs ou des inexactitudes. Le document original dans sa langue native doit être considéré comme la source faisant autorité. Pour les informations critiques, il est recommandé de recourir à une traduction professionnelle réalisée par un humain. Nous ne saurions être tenus responsables des malentendus ou erreurs d'interprétation découlant de l'utilisation de cette traduction.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->