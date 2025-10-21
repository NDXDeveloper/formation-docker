🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 1.3 Architecture de Docker (Docker Engine, Docker Client, Docker Daemon)

## Introduction

Maintenant que vous comprenez ce qu'est Docker et en quoi il diffère des machines virtuelles, il est temps de découvrir comment Docker fonctionne sous le capot. Docker n'est pas un simple programme monolithique, mais plutôt un écosystème composé de plusieurs éléments qui travaillent ensemble de manière harmonieuse.

Comprendre cette architecture vous aidera à :
- Mieux utiliser Docker au quotidien
- Diagnostiquer les problèmes quand ils surviennent
- Comprendre les messages d'erreur
- Optimiser vos configurations

Ne vous inquiétez pas si certains concepts semblent complexes au premier abord. Nous allons les décortiquer progressivement avec des analogies simples.

## Vue d'ensemble de l'architecture Docker

Docker utilise une **architecture client-serveur**. Cette architecture comprend trois composants principaux :

1. **Le Docker Client** : L'interface avec laquelle vous interagissez
2. **Le Docker Daemon** : Le moteur qui fait le travail réel
3. **Le Docker Registry** : L'entrepôt où sont stockées les images

Ces trois éléments communiquent entre eux pour créer, gérer et exécuter vos conteneurs.

## L'analogie du restaurant

Pour bien comprendre l'architecture Docker, imaginez un restaurant :

- **Le client (vous)** : Vous consultez le menu et passez commande
- **Le serveur (Docker Client)** : Il prend votre commande et la transmet en cuisine
- **La cuisine (Docker Daemon)** : C'est là que le plat est réellement préparé
- **Le garde-manger (Docker Registry)** : L'endroit où sont stockés tous les ingrédients (images)

Vous ne rentrez jamais directement en cuisine pour préparer votre plat. Vous passez par le serveur qui communique avec la cuisine. La cuisine prend les ingrédients du garde-manger et prépare votre commande.

## Le Docker Client : Votre interface de commande

### Qu'est-ce que le Docker Client ?

Le **Docker Client** est l'outil que vous utilisez pour interagir avec Docker. C'est votre point d'entrée, votre interface de commande.

Quand vous tapez une commande comme :
```bash
docker run hello-world
```

C'est le Docker Client qui reçoit cette commande. Mais il ne fait pas le travail lui-même. Son rôle est de transmettre vos instructions au Docker Daemon.

### Où se trouve le Docker Client ?

Le Docker Client est généralement installé sur votre machine locale. C'est la commande `docker` que vous utilisez dans votre terminal.

### Modes de communication

Le Docker Client peut communiquer avec le Docker Daemon de différentes manières :
- **Localement** : Sur la même machine (le cas le plus courant)
- **À distance** : Sur un serveur distant via le réseau

C'est très pratique : vous pouvez contrôler Docker sur un serveur distant depuis votre ordinateur local.

### Interface graphique

Bien que la ligne de commande soit le moyen principal d'utiliser Docker, il existe aussi des interfaces graphiques comme **Docker Desktop** qui facilitent l'utilisation pour les débutants.

## Le Docker Daemon : Le cerveau de l'opération

### Qu'est-ce que le Docker Daemon ?

Le **Docker Daemon** (aussi appelé `dockerd`) est le véritable moteur de Docker. C'est un processus qui tourne en arrière-plan sur votre système et qui fait tout le travail réel :

- Construire des images
- Créer des conteneurs
- Gérer les conteneurs (démarrer, arrêter, supprimer)
- Gérer les réseaux et les volumes
- Télécharger des images depuis les registres

### Le Daemon est un service

Le Docker Daemon est un **service système** (ou daemon dans le jargon Unix). Cela signifie :
- Il démarre automatiquement au démarrage de votre ordinateur
- Il tourne constamment en arrière-plan
- Il attend les commandes du Docker Client
- Il gère toutes les ressources Docker sur votre système

### Comment fonctionne le Daemon ?

Lorsque le Docker Daemon reçoit une commande du Client, voici ce qui se passe :

1. **Réception** : Le Daemon reçoit la requête via une API REST
2. **Validation** : Il vérifie que la commande est valide
3. **Exécution** : Il effectue l'action demandée
4. **Réponse** : Il renvoie le résultat au Client

### Gestion des objets Docker

Le Daemon gère tous les **objets Docker** :
- **Images** : Les modèles pour créer des conteneurs
- **Conteneurs** : Les instances en cours d'exécution
- **Networks** : Les réseaux permettant aux conteneurs de communiquer
- **Volumes** : Les espaces de stockage persistants

## Le Docker Engine : Le cœur du système

### Qu'est-ce que le Docker Engine ?

Vous avez peut-être remarqué que nous parlons de Docker Client, Docker Daemon, mais aussi de **Docker Engine**. Alors, qu'est-ce que c'est exactement ?

Le **Docker Engine** est le terme qui englobe l'ensemble de la plateforme Docker. C'est une application client-serveur qui comprend :

1. **Le serveur** : Le Docker Daemon (`dockerd`)
2. **Une API REST** : L'interface qui permet au Client et au Daemon de communiquer
3. **Le client** : L'interface en ligne de commande (`docker`)

En d'autres termes, quand on parle de "Docker Engine", on parle de l'ensemble du système Docker installé sur votre machine.

### Les composants sous-jacents

Le Docker Engine repose lui-même sur plusieurs technologies :

- **containerd** : Un daemon qui gère le cycle de vie des conteneurs
- **runc** : L'outil qui crée et exécute réellement les conteneurs
- **Les namespaces Linux** : Pour l'isolation des conteneurs
- **Les cgroups** : Pour la gestion des ressources

Vous n'avez pas besoin de connaître ces détails pour utiliser Docker, mais sachez qu'ils existent et font le travail en coulisses.

## Le Docker Registry : L'entrepôt d'images

### Qu'est-ce qu'un Registry ?

Un **Docker Registry** est un système de stockage et de distribution d'images Docker. C'est comme une bibliothèque géante où sont stockées toutes les images disponibles.

### Docker Hub : Le registre public principal

**Docker Hub** (hub.docker.com) est le registre public par défaut. Il contient :
- Des millions d'images publiques prêtes à l'emploi
- Des images officielles maintenues par Docker et les éditeurs
- Des images créées par la communauté
- Vos propres images (publiques ou privées)

Exemples d'images populaires sur Docker Hub :
- `nginx` : Un serveur web
- `postgres` : Une base de données PostgreSQL
- `python` : Un environnement Python
- `node` : Un environnement Node.js

### Registres privés

Vous pouvez aussi créer vos propres registres privés :
- Pour garder vos images confidentielles
- Pour un contrôle total sur vos images
- Pour des raisons de sécurité ou de conformité

Des solutions comme **Harbor**, **Amazon ECR**, **Google Container Registry** ou **Azure Container Registry** offrent des registres privés hébergés.

## Architecture complète : Comment tout s'articule

Voici un schéma complet de l'architecture Docker :

```
┌─────────────────────────────────────────────────────────────┐
│                      DOCKER HOST                            │
│                                                             │
│  ┌──────────────────┐          ┌─────────────────────────┐  │
│  │  Docker Client   │          │    Docker Daemon        │  │
│  │    (docker)      │◄────────►│     (dockerd)           │  │
│  │                  │ API REST │                         │  │
│  │  Lignes de       │          │  ┌───────────────────┐  │  │
│  │  commande        │          │  │   Images          │  │  │
│  └──────────────────┘          │  ├───────────────────┤  │  │
│                                │  │   Conteneurs      │  │  │
│                                │  ├───────────────────┤  │  │
│                                │  │   Networks        │  │  │
│                                │  ├───────────────────┤  │  │
│                                │  │   Volumes         │  │  │
│                                │  └───────────────────┘  │  │
│                                └─────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                              │
                              │ pull / push
                              ▼
                    ┌──────────────────┐
                    │  Docker Registry │
                    │   (Docker Hub)   │
                    │                  │
                    │   📦 Images      │
                    └──────────────────┘
```

## Le flux de travail typique

Voyons concrètement ce qui se passe quand vous exécutez une commande Docker. Prenons l'exemple de `docker run nginx` :

### Étape 1 : Vous tapez la commande
```bash
docker run nginx
```

### Étape 2 : Le Docker Client réceptionne
Le Docker Client intercepte cette commande et la convertit en une requête API.

### Étape 3 : Envoi au Daemon
Le Client envoie la requête au Docker Daemon via l'API REST (généralement via un socket Unix local : `/var/run/docker.sock`).

### Étape 4 : Le Daemon vérifie les images locales
Le Daemon regarde si l'image `nginx` existe déjà sur votre machine.

### Étape 5 : Téléchargement si nécessaire
Si l'image n'existe pas localement, le Daemon contacte Docker Hub (le registry par défaut) et télécharge l'image `nginx`.

### Étape 6 : Création du conteneur
Une fois l'image disponible, le Daemon crée un nouveau conteneur à partir de cette image.

### Étape 7 : Démarrage du conteneur
Le Daemon démarre le conteneur. Nginx s'exécute maintenant dans un environnement isolé.

### Étape 8 : Réponse au Client
Le Daemon renvoie l'information au Client, qui vous affiche le résultat dans votre terminal.

Tout cela se passe en quelques secondes !

## Communication via l'API REST

### Qu'est-ce qu'une API REST ?

L'**API REST** (Representational State Transfer) est le langage de communication entre le Docker Client et le Docker Daemon. C'est un standard web qui permet aux applications de communiquer via HTTP.

### Pourquoi c'est important ?

Grâce à cette API :
- N'importe quel programme peut contrôler Docker (pas seulement le client officiel)
- Vous pouvez créer vos propres outils qui interagissent avec Docker
- Docker peut être contrôlé à distance via le réseau
- Des interfaces graphiques peuvent être développées

### Exemple concret

Quand vous tapez `docker ps`, voici ce qui se passe réellement :

1. Le Client envoie une requête HTTP GET à l'API : `GET /containers/json`
2. Le Daemon répond avec une liste JSON de tous les conteneurs
3. Le Client formate cette réponse et vous l'affiche joliment dans le terminal

## Docker Desktop : Une surcouche pratique

### Qu'est-ce que Docker Desktop ?

**Docker Desktop** est une application complète qui inclut :
- Le Docker Engine (Client + Daemon)
- Une interface graphique
- Docker Compose
- Kubernetes (optionnel)
- Des outils de gestion simplifiés

### Pourquoi utiliser Docker Desktop ?

Docker Desktop est particulièrement utile pour :
- **Windows et macOS** : Sur ces systèmes, Docker nécessite une VM Linux sous-jacente. Docker Desktop gère tout cela automatiquement
- **Les débutants** : L'interface graphique facilite la prise en main
- **Le développement** : Configuration simplifiée pour les développeurs

### Docker Desktop vs Docker Engine

Sur **Linux**, vous pouvez installer directement Docker Engine sans Docker Desktop.

Sur **Windows et macOS**, Docker Desktop est la solution recommandée car il inclut la VM Linux nécessaire et configure tout automatiquement.

## Les différents modes de déploiement

### Mode local (le plus courant)

```
[Votre Machine]
  ├─ Docker Client
  └─ Docker Daemon
      └─ Conteneurs
```

Le Client et le Daemon sont sur la même machine. C'est le cas typique pour le développement.

### Mode distant

```
[Votre Machine]          [Serveur Distant]
  └─ Docker Client  ──────►  Docker Daemon
                                └─ Conteneurs
```

Vous contrôlez Docker sur un serveur distant depuis votre machine locale. Pratique pour gérer des environnements de production.

### Mode distribué (avancé)

```
[Manager]                [Worker 1]     [Worker 2]     [Worker 3]
  └─ Daemon  ──────────►  Daemon        Daemon         Daemon
                           └─ Conteneurs └─ Conteneurs └─ Conteneurs
```

Docker Swarm ou Kubernetes permettent de gérer des clusters de machines Docker. C'est le mode utilisé en production pour les applications à grande échelle.

## Résumé visuel simplifié

Voici la version la plus simple à retenir :

```
Vous → [Docker Client] → [Docker Daemon] ⇄ [Docker Registry]
                              ↓
                         [Conteneurs]
```

1. **Vous** donnez des ordres au **Docker Client**
2. Le **Client** transmet au **Daemon**
3. Le **Daemon** télécharge des images depuis le **Registry** si nécessaire
4. Le **Daemon** crée et gère vos **Conteneurs**

## En résumé

L'architecture de Docker repose sur trois piliers :

**Le Docker Client** est votre interface. C'est avec lui que vous interagissez via des commandes. Il ne fait pas le travail lui-même, mais transmet vos demandes au Daemon.

**Le Docker Daemon** est le cerveau qui fait tout le travail réel : construire des images, créer des conteneurs, gérer les ressources. Il tourne en permanence en arrière-plan.

**Le Docker Engine** est le terme global qui englobe le Client, le Daemon et l'API qui les fait communiquer. Quand vous installez Docker, vous installez le Docker Engine complet.

**Le Docker Registry** (comme Docker Hub) est l'entrepôt où sont stockées et partagées les images Docker.

Cette architecture client-serveur rend Docker flexible, puissant et facile à utiliser, que ce soit localement pour du développement ou à distance pour gérer des infrastructures en production.

Maintenant que vous comprenez comment Docker est architecturé, nous allons explorer les cas d'usage concrets où Docker brille particulièrement.

⏭️ [Cas d'usage et avantages de Docker](/01-introduction-a-docker/04-cas-dusage-et-avantages-de-docker.md)
