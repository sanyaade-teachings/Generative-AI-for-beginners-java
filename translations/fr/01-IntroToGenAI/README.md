# Introduction à l'IA générative - Édition Java

## Ce que vous apprendrez

- **Les fondamentaux de l'IA générative** incluant les LLM, l'ingénierie des prompts, les tokens, les embeddings et les bases de données vectorielles
- **Comparer les outils de développement IA Java** comprenant Azure OpenAI SDK, Spring AI, et OpenAI Java SDK
- **Découvrir le Model Context Protocol** et son rôle dans la communication des agents IA

## Table des matières

- [Introduction](#introduction)
- [Un rappel rapide sur les concepts de l’IA générative](#un-rappel-rapide-sur-les-concepts-de-l’ia-générative)
- [Revue de l’ingénierie des prompts](#revue-de-l’ingénierie-des-prompts)
- [Tokens, embeddings, et agents](#tokens-embeddings-et-agents)
- [Outils et bibliothèques de développement IA pour Java](#outils-et-bibliothèques-de-développement-ia-pour-java)
  - [OpenAI Java SDK](#openai-java-sdk)
  - [Spring AI](#spring-ai)
  - [Azure OpenAI Java SDK](#azure-openai-java-sdk)
- [Résumé](#résumé)
- [Étapes suivantes](#étapes-suivantes)

## Introduction

Bienvenue au premier chapitre de l’IA générative pour débutants - Édition Java ! Cette leçon fondamentale vous présente les concepts clés de l’IA générative et comment travailler avec eux en Java. Vous apprendrez les éléments essentiels des applications IA, y compris les modèles de langage larges (LLM), les tokens, embeddings et agents IA. Nous explorerons aussi les principaux outils Java que vous utiliserez tout au long de ce cours.

### Un rappel rapide sur les concepts de l’IA générative

L’IA générative est un type d’intelligence artificielle qui crée du contenu nouveau, comme du texte, des images ou du code, basé sur des motifs et relations appris à partir des données. Les modèles d’IA générative peuvent produire des réponses semblables à celles des humains, comprendre le contexte, et parfois même créer du contenu très humain.

En développant vos applications IA Java, vous travaillerez avec des **modèles d’IA générative** pour créer du contenu. Certaines capacités des modèles IA générative incluent :

- **Génération de texte** : Produire du texte humain pour chatbots, contenus, et complétion de texte.
- **Génération et analyse d’images** : Créer des images réalistes, améliorer des photos, et détecter des objets.
- **Génération de code** : Écrire des extraits de code ou scripts.

Il existe des types spécifiques de modèles optimisés pour différentes tâches. Par exemple, les **Petits Modèles de Langage (SLM)** et les **Modèles de Langage Larges (LLM)** peuvent gérer la génération de texte, avec les LLM offrant généralement de meilleures performances pour les tâches complexes. Pour les tâches liées aux images, vous utiliserez des modèles spécialisés en vision ou des modèles multi-modaux.

![Figure: Types de modèles d’IA générative et cas d’utilisation.](../../../translated_images/fr/llms.225ca2b8a0d34473.webp)

Évidemment, les réponses de ces modèles ne sont pas parfaites tout le temps. Vous avez sûrement entendu parler de modèles qui "hallucinent" ou génèrent des informations incorrectes de manière autoritaire. Mais vous pouvez aider à guider le modèle pour générer de meilleures réponses en lui fournissant des instructions claires et un contexte. C’est ici que l’**ingénierie des prompts** intervient.

#### Revue de l’ingénierie des prompts

L’ingénierie des prompts consiste à concevoir des entrées efficaces pour guider les modèles IA vers les sorties désirées. Cela implique :

- **Clarté** : Formuler des instructions claires et sans ambiguïté.
- **Contexte** : Fournir les informations contextuelles nécessaires.
- **Contraintes** : Spécifier les éventuelles limitations ou formats.

Quelques bonnes pratiques incluent la conception du prompt, des instructions claires, le découpage des tâches, l’apprentissage one-shot et few-shot, ainsi que l’ajustement des prompts. Tester différents prompts est essentiel pour trouver ce qui fonctionne le mieux pour votre cas d’usage spécifique.

Lors du développement d’applications, vous travaillerez avec différents types de prompts :
- **Prompts système** : Posent les règles de base et le contexte du comportement du modèle
- **Prompts utilisateur** : Les données d’entrée provenant des utilisateurs de votre application
- **Prompts assistant** : Les réponses du modèle basées sur les prompts système et utilisateur

> **En savoir plus** : En savoir plus sur l’ingénierie des prompts dans le [chapitre d’ingénierie des prompts du cours GenAI pour débutants](https://github.com/microsoft/generative-ai-for-beginners/tree/main/04-prompt-engineering-fundamentals)

#### Tokens, embeddings, et agents

Lorsque vous travaillez avec les modèles IA générative, vous rencontrerez des termes comme **tokens**, **embeddings**, **agents**, et **Model Context Protocol (MCP)**. Voici un aperçu détaillé de ces concepts :

- **Tokens** : Les tokens sont la plus petite unité de texte dans un modèle. Ils peuvent être des mots, des caractères ou des sous-mots. Ils représentent les données textuelles dans un format compréhensible par le modèle. Par exemple, la phrase "The quick brown fox jumped over the lazy dog" peut être tokenisée en ["The", " quick", " brown", " fox", " jumped", " over", " the", " lazy", " dog"] ou ["The", " qu", "ick", " br", "own", " fox", " jump", "ed", " over", " the", " la", "zy", " dog"] selon la stratégie de tokenisation.

![Figure: Exemple de tokens en IA générative segmentant les mots en tokens](../../../translated_images/fr/tokens.6283ed277a2ffff4.webp)

La tokenisation est le processus de division du texte en ces unités plus petites. C’est crucial car les modèles fonctionnent sur des tokens plutôt que sur du texte brut. Le nombre de tokens dans un prompt influence la longueur et la qualité de la réponse, car les modèles ont des limites de tokens dans leur fenêtre de contexte (par exemple, 128K tokens pour le contexte total de GPT-4o, incluant entrée et sortie).

  En Java, vous pouvez utiliser des bibliothèques comme l’OpenAI SDK pour gérer automatiquement la tokenisation lors de l’envoi de requêtes aux modèles IA.

- **Embeddings** : Les embeddings sont des représentations vectorielles des tokens qui capturent le sens sémantique. Ce sont des représentations numériques (typiquement des tableaux de nombres à virgule flottante) permettant aux modèles de comprendre les relations entre les mots et de générer des réponses contextuellement pertinentes. Des mots similaires possèdent des embeddings similaires, ce qui permet au modèle de saisir les synonymes et relations sémantiques.

![Figure: Embeddings](../../../translated_images/fr/embedding.398e50802c0037f9.webp)

  En Java, vous pouvez générer des embeddings avec l’OpenAI SDK ou d’autres bibliothèques supportant la génération d’embeddings. Ces embeddings sont essentiels pour des tâches comme la recherche sémantique, où vous souhaitez trouver du contenu similaire basé sur le sens plutôt que sur une correspondance textuelle exacte.

- **Bases de données vectorielles** : Les bases de données vectorielles sont des systèmes de stockage spécialisés optimisés pour les embeddings. Elles permettent une recherche efficace par similarité et sont cruciales pour les patrons de génération augmentée par recherche (RAG), où il faut retrouver des informations pertinentes dans de grands jeux de données basées sur la similarité sémantique plus que sur la correspondance exacte.

![Figure: Architecture d’une base de données vectorielle montrant comment les embeddings sont stockés et récupérés pour une recherche par similarité.](../../../translated_images/fr/vector.f12f114934e223df.webp)

> **Note** : Dans ce cours, nous n’aborderons pas les bases de données vectorielles, mais nous les mentionnons car elles sont fréquemment utilisées dans les applications réelles.

- **Agents & MCP** : Composants IA qui interagissent de manière autonome avec les modèles, outils, et systèmes externes. Le Model Context Protocol (MCP) fournit un moyen standardisé pour les agents d’accéder de façon sécurisée aux sources de données et outils externes. Apprenez-en plus dans notre cours [MCP pour débutants](https://github.com/microsoft/mcp-for-beginners).

Dans les applications IA Java, vous utiliserez les tokens pour le traitement du texte, les embeddings pour la recherche sémantique et RAG, les bases de données vectorielles pour la récupération de données, et les agents avec MCP pour construire des systèmes intelligents utilisant des outils.

![Figure: comment un prompt devient une réponse—tokens, vecteurs, recherche RAG optionnelle, réflexion LLM, et un agent MCP en un flux rapide.](../../../translated_images/fr/flow.f4ef62c3052d12a8.webp)

### Outils et bibliothèques de développement IA pour Java

Java offre d’excellents outils pour le développement IA. Il y a trois bibliothèques principales que nous explorerons dans ce cours - OpenAI Java SDK, Azure OpenAI SDK, et Spring AI.

Voici un tableau récapitulatif indiquant quelle SDK est utilisée dans les exemples de chaque chapitre :

| Chapitre | Extrait | SDK |
|---------|--------|-----|
| 02-SetupDevEnvironment | basic-chat-azure | Spring AI Azure OpenAI |
| 03-CoreGenerativeAITechniques | examples | Azure OpenAI SDK |
| 04-PracticalSamples | petstory | OpenAI Java SDK |
| 04-PracticalSamples | foundrylocal | OpenAI Java SDK |
| 04-PracticalSamples | calculator | Spring AI MCP SDK + LangChain4j |

**Liens vers la documentation des SDK :**
- [Azure OpenAI Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai_1.0.0-beta.16/sdk/openai/azure-ai-openai)
- [Spring AI](https://docs.spring.io/spring-ai/reference/)
- [OpenAI Java SDK](https://github.com/openai/openai-java)
- [LangChain4j](https://docs.langchain4j.dev/)

#### OpenAI Java SDK

OpenAI SDK est la bibliothèque officielle Java pour l’API OpenAI. Elle offre une interface simple et cohérente pour interagir avec les modèles OpenAI, facilitant l’intégration des capacités IA dans les applications Java. L’application Pet Story et l’exemple Foundry Local du chapitre 4 illustrent l’approche OpenAI SDK avec Azure AI Foundry.

#### Spring AI

Spring AI est un framework complet qui apporte des capacités IA aux applications Spring, fournissant une couche d’abstraction cohérente à travers différents fournisseurs IA. Il s’intègre parfaitement dans l’écosystème Spring, ce qui en fait le choix idéal pour les applications Java d’entreprise nécessitant des capacités IA.

La force de Spring AI réside dans son intégration fluide avec l’écosystème Spring, facilitant la construction d’applications IA prêtes pour la production avec des patterns Spring familiers comme l’injection de dépendances, la gestion de configuration, et les frameworks de tests. Vous utiliserez Spring AI dans les chapitres 2 et 4 pour créer des applications exploitées à la fois avec OpenAI et les bibliothèques MCP Spring AI.

##### Model Context Protocol (MCP)

Le [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) est une norme émergente permettant aux applications IA d’interagir de manière sécurisée avec des sources de données et outils externes. MCP fournit un moyen standardisé pour les modèles IA d’accéder à l’information contextuelle et d’exécuter des actions dans vos applications.

Dans le chapitre 4, vous construirez un simple service calculatrice MCP démontrant les fondamentaux du Model Context Protocol avec Spring AI, montrant comment créer des intégrations d’outils basiques et des architectures de service.

#### Azure OpenAI Java SDK

La bibliothèque cliente Azure OpenAI pour Java est une adaptation des API REST OpenAI qui fournit une interface idiomatique et une intégration avec le reste de l’écosystème Azure SDK. Dans le chapitre 3, vous développerez des applications utilisant l’Azure OpenAI SDK, y compris des applications chat, appel de fonctions, et patrons RAG (Retrieval-Augmented Generation).

> Note : Azure OpenAI SDK est en retard par rapport à OpenAI Java SDK en termes de fonctionnalités, donc pour vos projets futurs, considérez l’utilisation de l’OpenAI Java SDK.

## Résumé

Voilà pour les bases ! Vous comprenez désormais :

- Les concepts clés de l’IA générative - des LLM et de l’ingénierie des prompts aux tokens, embeddings et bases de données vectorielles
- Vos options d’outils pour le développement IA Java : Azure OpenAI SDK, Spring AI, et OpenAI Java SDK
- Ce qu’est le Model Context Protocol et comment il permet aux agents IA d’utiliser des outils externes

## Étapes suivantes

[Chapitre 2 : Configuration de l’environnement de développement](../02-SetupDevEnvironment/README.md)

---

<!-- CO-OP TRANSLATOR DISCLAIMER START -->
**Avertissement** :
Ce document a été traduit à l'aide du service de traduction automatique [Co-op Translator](https://github.com/Azure/co-op-translator). Bien que nous nous efforçions d'assurer l'exactitude, veuillez noter que les traductions automatisées peuvent contenir des erreurs ou des inexactitudes. Le document original dans sa langue native doit être considéré comme la source faisant autorité. Pour les informations critiques, il est recommandé de recourir à une traduction professionnelle réalisée par un humain. Nous ne saurions être tenus responsables des malentendus ou erreurs d'interprétation découlant de l'utilisation de cette traduction.
<!-- CO-OP TRANSLATOR DISCLAIMER END -->