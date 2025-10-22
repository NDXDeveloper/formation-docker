🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 6.1 Comprendre la persistance des données

## Introduction

Lorsque vous travaillez avec Docker, l'une des premières questions qui se pose est : **que deviennent mes données ?** Cette section vous explique pourquoi la gestion des données dans Docker nécessite une attention particulière et comment Docker aborde ce défi.

## Le problème : la nature éphémère des conteneurs

### Qu'est-ce qu'un conteneur éphémère ?

Par défaut, les conteneurs Docker sont **éphémères** (temporaires). Cela signifie qu'ils sont conçus pour être :
- Créés rapidement
- Détruits facilement
- Remplacés sans difficulté

Cette nature éphémère est l'une des forces de Docker, car elle permet une grande flexibilité et une gestion simplifiée de vos applications.

### Le cycle de vie des données dans un conteneur

Imaginez le scénario suivant :

1. Vous lancez un conteneur avec une base de données PostgreSQL
2. Vous créez des tables et ajoutez des données
3. Vous arrêtez le conteneur
4. Vous supprimez le conteneur
5. Vous relancez un nouveau conteneur PostgreSQL

**Que se passe-t-il avec vos données ?** Elles ont disparu !

Cela s'explique par le fonctionnement interne de Docker : toutes les modifications effectuées à l'intérieur d'un conteneur (fichiers créés, données ajoutées, configurations modifiées) sont stockées dans une **couche inscriptible** propre à ce conteneur. Lorsque le conteneur est supprimé, cette couche et tout son contenu sont également supprimés.

## Pourquoi c'est un problème ?

Cette nature éphémère pose plusieurs défis dans des situations réelles :

### 1. Perte de données critiques

Si vous utilisez Docker pour héberger une base de données, un système de fichiers, ou toute application qui génère des données importantes, vous ne pouvez pas vous permettre de perdre ces informations à chaque suppression de conteneur.

**Exemple concret :** Vous développez un blog avec une base de données MySQL. Vous supprimez accidentellement le conteneur pour le recréer avec une nouvelle configuration. Tous les articles, commentaires et utilisateurs sont perdus.

### 2. Difficultés de mise à jour

Lorsque vous mettez à jour une image Docker (par exemple, passer de PostgreSQL 14 à PostgreSQL 15), vous devez généralement supprimer l'ancien conteneur et en créer un nouveau. Sans mécanisme de persistance, vous perdez toutes vos données.

### 3. Partage de données entre conteneurs

Dans une architecture moderne, plusieurs conteneurs doivent parfois accéder aux mêmes données. Par exemple :
- Un conteneur de traitement qui lit des fichiers
- Un conteneur d'analyse qui traite ces mêmes fichiers
- Un conteneur de sauvegarde qui archive les résultats

Sans persistance externe, ces conteneurs ne peuvent pas collaborer efficacement.

### 4. Développement et débogage

Pendant le développement, vous pouvez avoir besoin de redémarrer vos conteneurs fréquemment. Perdre les données de test à chaque redémarrage ralentit considérablement le processus de développement.

## La solution Docker : la séparation des données et du conteneur

Docker propose une philosophie simple mais puissante : **séparer les données de l'application**.

### Le concept de persistance

Au lieu de stocker les données critiques **à l'intérieur** du conteneur, Docker permet de les stocker **à l'extérieur**, dans un espace qui survit à la destruction du conteneur.

```
┌─────────────────────────────────────┐
│         HÔTE DOCKER                 │
│                                     │
│  ┌──────────────┐   ┌────────────┐  │
│  │  Conteneur   │   │  Stockage  │  │
│  │              │◄──┤  persistant│  │
│  │  (éphémère)  │   │  (durable) │  │
│  └──────────────┘   └────────────┘  │
│                                     │
└─────────────────────────────────────┘
```

### Les avantages de cette approche

1. **Durabilité** : Les données survivent à la suppression du conteneur
2. **Flexibilité** : Vous pouvez remplacer, mettre à jour ou recréer des conteneurs sans perdre de données
3. **Portabilité** : Les données peuvent être facilement sauvegardées, déplacées ou partagées
4. **Performance** : Docker peut optimiser le stockage en fonction du système hôte

## Les trois méthodes de persistance

Docker offre trois mécanismes principaux pour persister les données :

### 1. Les volumes Docker (recommandé)

Les **volumes** sont des espaces de stockage gérés entièrement par Docker. Ils constituent la méthode **recommandée** pour la persistance des données.

**Caractéristiques :**
- Gérés par Docker (création, suppression, sauvegarde)
- Stockés dans un emplacement spécifique du système hôte
- Isolés du système de fichiers de l'hôte
- Fonctionnent sur tous les systèmes d'exploitation (Linux, Windows, macOS)

**Cas d'usage typiques :**
- Bases de données (PostgreSQL, MySQL, MongoDB)
- Fichiers de configuration
- Données d'application générées dynamiquement

### 2. Les bind mounts

Les **bind mounts** permettent de monter un dossier ou un fichier spécifique de votre machine hôte directement dans le conteneur.

**Caractéristiques :**
- Vous contrôlez l'emplacement exact sur l'hôte
- Accès direct aux fichiers depuis l'hôte et le conteneur
- Dépendant de la structure du système de fichiers de l'hôte

**Cas d'usage typiques :**
- Code source pendant le développement
- Fichiers de configuration que vous voulez éditer facilement
- Logs que vous voulez consulter en temps réel

### 3. Les tmpfs mounts (mémoire temporaire)

Les **tmpfs mounts** stockent les données en mémoire RAM plutôt que sur le disque.

**Caractéristiques :**
- Très rapides (stockage en RAM)
- Non persistantes (données perdues à l'arrêt du conteneur)
- Uniquement disponibles sur Linux

**Cas d'usage typiques :**
- Données temporaires sensibles (mots de passe, tokens)
- Cache temporaire haute performance

## Comparaison visuelle

```
┌──────────────────────────────────────────────────────────┐
│                    SYSTÈME HÔTE                          │
│                                                          │
│  ┌─────────────────────┐       ┌──────────────────────┐  │
│  │  Espace Docker      │       │  Système de fichiers │  │
│  │                     │       │  de l'hôte           │  │
│  │  ┌──────────────┐   │       │                      │  │
│  │  │   VOLUMES    │   │       │  /home/user/projet/  │  │
│  │  │  (Docker)    │   │       │         │            │  │
│  │  └──────┬───────┘   │       │         │            │  │
│  └─────────┼───────────┘       └─────────┼────────────┘  │
│            │                             │               │
│            │    ┌────────────────────────┘               │
│            │    │                                        │
│       ┌────▼────▼─────────────────────┐                  │
│       │      CONTENEUR DOCKER         │                  │
│       │                               │                  │
│       │  /app/data  ◄── Volume        │                  │
│       │  /app/code  ◄── Bind mount    │                  │
│       └───────────────────────────────┘                  │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

## À retenir

🔑 **Points clés :**

- Par défaut, les données dans un conteneur sont **éphémères** et disparaissent avec le conteneur
- Cette nature éphémère est voulue par design pour faciliter la gestion des conteneurs
- Pour conserver les données importantes, il faut utiliser un mécanisme de **persistance**
- Docker propose trois solutions : **volumes** (recommandé), **bind mounts**, et **tmpfs mounts**
- Le principe fondamental est de **séparer les données de l'application**

## Prochaines étapes

Dans les sections suivantes, nous allons explorer en détail :
- Comment créer et utiliser des volumes Docker
- Comment configurer des bind mounts
- Comment choisir entre volumes et bind mounts selon vos besoins

La compréhension de ces concepts est essentielle pour utiliser Docker de manière professionnelle et éviter les pertes de données catastrophiques.

⏭️ [Les volumes Docker (nommés et anonymes)](/06-gestion-des-donnees/02-les-volumes-docker-nommes-et-anonymes.md)
