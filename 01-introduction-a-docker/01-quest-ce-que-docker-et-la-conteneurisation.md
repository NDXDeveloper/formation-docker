🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 1.1 Qu'est-ce que Docker et la conteneurisation

## Introduction

Imaginez que vous développez une application sur votre ordinateur. Elle fonctionne parfaitement. Mais lorsque vous essayez de la déployer sur un serveur ou que votre collègue tente de l'exécuter sur son ordinateur, elle ne fonctionne plus. Vous entendez alors la phrase redoutée : **"Pourtant, ça marche sur ma machine !"**

Ce problème classique du développement logiciel est précisément ce que Docker vient résoudre.

## Le problème traditionnel

Quand vous développez une application, celle-ci dépend de nombreux éléments :
- Une version spécifique d'un langage de programmation (Python 3.9, Node.js 18, etc.)
- Des bibliothèques et dépendances particulières
- Des configurations système spécifiques
- Des variables d'environnement
- Des fichiers de configuration

Sur votre machine de développement, tout est configuré exactement comme il faut. Mais sur une autre machine, ces éléments peuvent être différents ou absents, ce qui provoque des erreurs.

## Qu'est-ce que la conteneurisation ?

La **conteneurisation** est une approche qui consiste à empaqueter une application avec toutes ses dépendances, ses bibliothèques et ses fichiers de configuration dans une unité isolée appelée **conteneur**.

### L'analogie du conteneur maritime

Le terme "conteneur" vient du monde du transport maritime. Avant l'invention des conteneurs standardisés dans les années 1950, le chargement des marchandises sur les navires était chaotique : chaque type de marchandise nécessitait une manipulation différente.

Les conteneurs maritimes ont révolutionné le transport en créant une unité standardisée. Peu importe ce qu'il y a à l'intérieur (vêtements, électronique, nourriture), le conteneur lui-même est toujours de la même taille et se manipule de la même manière.

**La conteneurisation logicielle fonctionne sur le même principe** : peu importe l'application à l'intérieur (Python, Java, Node.js), le conteneur se manipule toujours de la même façon et s'exécute de manière prévisible sur n'importe quel système compatible.

## Qu'est-ce que Docker ?

**Docker** est la plateforme de conteneurisation la plus populaire au monde. C'est un outil qui permet de créer, déployer et exécuter des applications dans des conteneurs.

Docker n'a pas inventé la conteneurisation. Des technologies comme **LXC** (Linux Containers), les **cgroups** et les **namespaces** Linux existaient déjà avant Docker. Mais Docker a rendu la conteneurisation accessible, simple et standardisée pour tous les développeurs, grâce à une interface conviviale et un écosystème complet (images, registres, outils).

### Les composants principaux de Docker

Docker repose sur trois éléments fondamentaux :

1. **Les images Docker** : Ce sont des modèles en lecture seule qui contiennent tout ce dont votre application a besoin pour fonctionner (code, dépendances, configuration). Pensez-y comme à une "recette" ou un "plan" de votre application.

2. **Les conteneurs Docker** : Ce sont des instances en cours d'exécution créées à partir d'images. Si l'image est le "plan", le conteneur est la "construction réelle". Vous pouvez créer plusieurs conteneurs à partir de la même image.

3. **Le Docker Engine** : C'est le moteur qui fait fonctionner tout l'écosystème Docker. Il gère la création, l'exécution et la suppression des conteneurs.

## Les caractéristiques clés d'un conteneur Docker

### Isolation
Chaque conteneur s'exécute dans son propre environnement isolé. Les conteneurs ne peuvent pas interférer les uns avec les autres, ni avec le système hôte, sauf si vous le configurez explicitement.

### Portabilité
Un conteneur qui fonctionne sur votre ordinateur portable fonctionnera exactement de la même manière sur un serveur de production, qu'il soit sous Linux, Windows ou macOS (avec Docker installé).

> **Note :** Sur Windows et macOS, Docker exécute les conteneurs Linux à l'intérieur d'une machine virtuelle légère (gérée automatiquement par Docker Desktop). Cela est transparent pour l'utilisateur, mais signifie que les conteneurs Linux ne tournent pas directement sur le noyau de l'hôte dans ces cas-là.

### Légèreté
Contrairement aux machines virtuelles, les conteneurs partagent le noyau du système d'exploitation hôte, ce qui les rend beaucoup plus légers et rapides à démarrer (quelques secondes au lieu de plusieurs minutes).

### Reproductibilité
Grâce aux images Docker, vous pouvez garantir que votre application s'exécutera toujours avec exactement les mêmes dépendances et configurations, peu importe où elle est déployée.

## Un exemple concret

Prenons l'exemple d'une application web Python utilisant Flask :

**Sans Docker** :
- Vous devez installer Python 3.9 sur chaque machine
- Installer Flask et toutes les bibliothèques nécessaires
- Configurer les variables d'environnement
- S'assurer que les versions correspondent exactement
- Répéter ce processus sur chaque environnement (développement, test, production)

**Avec Docker** :
- Vous créez une image qui contient Python 3.9, Flask et toutes les dépendances
- Vous exécutez cette image comme un conteneur sur n'importe quelle machine avec Docker
- L'application fonctionne de manière identique partout
- Pour mettre à jour, vous créez simplement une nouvelle image

## Pourquoi Docker est devenu incontournable

Depuis sa création en 2013, Docker est devenu un standard de l'industrie pour plusieurs raisons :

- **Simplification du déploiement** : Déployer une application devient aussi simple que de lancer un conteneur
- **Environnements cohérents** : Finies les différences entre développement et production
- **Efficacité des ressources** : Vous pouvez exécuter des dizaines de conteneurs sur une seule machine
- **Développement accéléré** : Les développeurs peuvent démarrer rapidement sans passer des heures à configurer leur environnement
- **Intégration avec les outils modernes** : Docker s'intègre parfaitement avec les pratiques DevOps, CI/CD, et l'orchestration de conteneurs (Kubernetes)

## En résumé

Docker et la conteneurisation résolvent le problème fondamental de "ça marche sur ma machine" en empaquetant les applications avec tout ce dont elles ont besoin dans des unités portables et isolées. Cette approche a transformé la manière dont les applications sont développées, testées et déployées dans le monde moderne du développement logiciel.

Dans les prochaines sections, nous verrons comment Docker se distingue des machines virtuelles et comment son architecture fonctionne en détail.

⏭️ [Différences entre conteneurs et machines virtuelles](/01-introduction-a-docker/02-differences-entre-conteneurs-et-machines-virtuelles.md)
