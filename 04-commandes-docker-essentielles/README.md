🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 4. Commandes Docker essentielles

## Introduction

Maintenant que vous comprenez les concepts fondamentaux de Docker (images, conteneurs, registres), il est temps de passer à la pratique ! Cette section est cruciale car elle vous permettra de maîtriser les commandes de base pour travailler efficacement avec Docker au quotidien.

## Pourquoi maîtriser les commandes Docker ?

Docker s'utilise principalement via la **ligne de commande** (terminal ou invite de commandes). Bien que des interfaces graphiques existent (comme Docker Desktop), la maîtrise des commandes CLI (Command Line Interface) est essentielle car :

- **Universalité** : les commandes fonctionnent de la même manière sur tous les systèmes d'exploitation
- **Automatisation** : vous pourrez créer des scripts pour automatiser vos tâches
- **Précision** : vous aurez un contrôle total sur vos actions
- **Documentation** : la majorité des tutoriels et de la documentation utilisent les commandes CLI
- **Environnements de production** : les serveurs distants n'ont généralement pas d'interface graphique

## Structure des commandes Docker

Toutes les commandes Docker suivent une structure similaire et prévisible :

```bash
docker <commande> [options] <arguments>
```

Par exemple :
```bash
docker run -d nginx
docker stop mon-conteneur
docker images --all
```

### Obtenir de l'aide

Docker intègre une aide complète directement dans la ligne de commande. Vous n'êtes jamais seul !

Pour voir toutes les commandes disponibles :
```bash
docker --help
```

Pour obtenir de l'aide sur une commande spécifique :
```bash
docker run --help
docker build --help
```

**Conseil** : N'hésitez pas à consulter l'aide régulièrement, même les utilisateurs expérimentés le font !

## Les catégories de commandes

Les commandes Docker peuvent être regroupées en plusieurs catégories selon leur fonction. Dans cette section, nous allons explorer les plus importantes :

### 1. Gestion des images
- Rechercher des images disponibles
- Télécharger des images depuis un registre
- Lister les images locales
- Supprimer des images
- Construire ses propres images

### 2. Gestion des conteneurs
- Créer et démarrer des conteneurs
- Arrêter et redémarrer des conteneurs
- Lister les conteneurs actifs et inactifs
- Supprimer des conteneurs
- Inspecter l'état des conteneurs

### 3. Interaction avec les conteneurs
- Exécuter des commandes dans un conteneur en cours d'exécution
- Consulter les logs d'un conteneur
- Se connecter à un conteneur

### 4. Nettoyage et maintenance
- Supprimer les ressources inutilisées
- Libérer de l'espace disque
- Maintenir un environnement propre

## Ce que vous allez apprendre

Dans les sous-sections qui suivent, vous allez découvrir en détail :

**4.1 Rechercher et télécharger des images**
Comment trouver les images Docker dont vous avez besoin et les télécharger sur votre machine.

**4.2 Lancer et gérer des conteneurs**
Les commandes pour créer, démarrer, arrêter et redémarrer vos conteneurs - le cœur de l'utilisation de Docker.

**4.3 Lister et inspecter**
Comment obtenir des informations sur vos conteneurs et images, vérifier leur état et examiner leur configuration.

**4.4 Supprimer conteneurs et images**
Gérer l'espace disque en supprimant proprement les ressources dont vous n'avez plus besoin.

**4.5 Interagir avec les conteneurs**
Techniques avancées pour exécuter des commandes, consulter les logs et accéder à l'intérieur de vos conteneurs.

## Approche pédagogique

Pour chaque commande, nous suivrons cette structure :

1. **À quoi sert cette commande ?** - Comprendre l'objectif
2. **Syntaxe de base** - La forme la plus simple
3. **Exemples pratiques** - Des cas concrets d'utilisation
4. **Options importantes** - Les paramètres les plus utiles
5. **Cas d'usage courants** - Quand et pourquoi utiliser cette commande

## Conseils avant de commencer

### 1. Pratiquez dans un environnement de test
N'hésitez pas à expérimenter ! Docker est conçu pour être sûr : vous ne risquez pas d'endommager votre système en testant des commandes.

### 2. Lisez les messages d'erreur
Les messages d'erreur de Docker sont généralement clairs et vous indiquent exactement ce qui ne va pas. Prenez le temps de les lire.

### 3. Utilisez l'auto-completion
Sur Linux et macOS, activez l'auto-complétion pour Docker. Elle vous fera gagner du temps et évitera les erreurs de frappe.

### 4. Documentez vos commandes
Gardez une trace des commandes que vous utilisez fréquemment. Vous pouvez créer un fichier avec vos "snippets" favoris.

### 5. Commencez simple
Ne vous sentez pas obligé de mémoriser toutes les options de chaque commande. Commencez par les syntaxes de base, le reste viendra avec la pratique.

## Conventions utilisées dans cette section

Dans les exemples de code :

- Les éléments entre `<chevrons>` doivent être remplacés par vos propres valeurs
  ```bash
  docker stop <nom_du_conteneur>
  ```

- Les éléments entre `[crochets]` sont optionnels
  ```bash
  docker run [options] <image>
  ```

- Le symbole `$` ou `>` en début de ligne indique une commande à taper (ne le tapez pas)
  ```bash
  $ docker ps
  ```

- Le symbole `#` indique un commentaire explicatif
  ```bash
  docker run nginx  # Ceci est un commentaire
  ```

## Prêt à commencer ?

Vous avez maintenant une vue d'ensemble de ce qui vous attend dans cette section. Les commandes Docker peuvent sembler nombreuses au début, mais vous verrez qu'avec la pratique, elles deviennent rapidement naturelles.

Chaque commande que vous allez apprendre est un outil dans votre boîte à outils Docker. Vous n'utiliserez pas toutes les commandes tous les jours, mais savoir qu'elles existent et comment les utiliser vous rendra autonome et efficace.

**Place à la pratique !** Passons maintenant à la première sous-section : rechercher et télécharger des images Docker.

⏭️ [Rechercher et télécharger des images (pull, search)](/04-commandes-docker-essentielles/01-rechercher-et-telecharger-des-images.md)
