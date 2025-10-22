🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 6.2 Les volumes Docker (nommés et anonymes)

## Introduction

Maintenant que vous comprenez pourquoi la persistance des données est essentielle dans Docker, il est temps de découvrir la solution **recommandée** par Docker : **les volumes**.

Les volumes sont le moyen le plus simple, le plus sûr et le plus performant de conserver vos données. Ils sont entièrement gérés par Docker, ce qui signifie que vous n'avez pas à vous soucier de leur emplacement exact sur votre système ou de leur compatibilité entre différents systèmes d'exploitation.

## Qu'est-ce qu'un volume Docker ?

Un volume Docker est un **espace de stockage géré par Docker** qui existe indépendamment des conteneurs. Pensez à un volume comme à un "disque dur virtuel" que Docker peut attacher à un ou plusieurs conteneurs.

### Analogie simple

Imaginez que vous avez :
- Une **clé USB** (le volume)
- Un **ordinateur portable** (le conteneur)

Vous pouvez :
- Brancher la clé USB sur l'ordinateur et y accéder
- Débrancher la clé USB et éteindre l'ordinateur
- Rallumer l'ordinateur (ou même en utiliser un autre)
- Rebrancher la même clé USB et retrouver toutes vos données

C'est exactement ce que font les volumes Docker !

```
┌─────────────────────────────────────────────────────────┐
│                    SYSTÈME HÔTE                         │
│                                                         │
│  ┌───────────────────────────┐                          │
│  │   Espace géré par Docker  │                          │
│  │                           │                          │
│  │   ┌─────────────────┐     │     ┌────────────────┐   │
│  │   │  Volume: db-data│─────────► │  Conteneur DB  │   │
│  │   └─────────────────┘     │     └────────────────┘   │
│  │                           │                          │
│  │   (Survit même si le conteneur est supprimé)         │
│  └───────────────────────────┘                          │
└─────────────────────────────────────────────────────────┘
```

## Les deux types de volumes

Docker propose deux types de volumes : les **volumes nommés** et les **volumes anonymes**. Comprendre la différence est essentiel pour bien utiliser Docker.

### 1. Les volumes nommés (Named Volumes)

Un volume nommé est un volume auquel vous donnez un **nom explicite** et facile à retenir.

**Exemple :** `postgres-data`, `mon-projet-uploads`, `cache-redis`

#### Avantages
✅ Facile à identifier et à gérer
✅ Réutilisable entre plusieurs conteneurs
✅ Peut être sauvegardé et restauré facilement
✅ Persiste même après la suppression du conteneur
✅ **C'est la méthode recommandée pour la production**

#### Quand l'utiliser ?
- Bases de données (PostgreSQL, MySQL, MongoDB)
- Données importantes de votre application
- Fichiers uploadés par les utilisateurs
- Tout ce que vous voulez conserver à long terme

### 2. Les volumes anonymes (Anonymous Volumes)

Un volume anonyme est un volume créé automatiquement par Docker **sans nom explicite**. Docker lui attribue un identifiant unique aléatoire.

**Exemple d'identifiant :** `a3f5b2c1d4e6f7g8h9i0j1k2l3m4n5o6p7q8r9s0t1u2v3w4x5y6z7`

#### Caractéristiques
⚠️ Difficile à identifier (long identifiant aléatoire)
⚠️ Compliqué à réutiliser
⚠️ Peut s'accumuler et occuper de l'espace disque
✅ Supprimé automatiquement avec le conteneur (avec `--rm`)
✅ Utile pour les données temporaires

#### Quand l'utiliser ?
- Cache temporaire
- Données de développement jetables
- Situations où vous ne voulez pas nommer le volume
- Tests rapides

## Créer et gérer des volumes nommés

### Créer un volume

La commande de base pour créer un volume nommé :

```bash
docker volume create nom-du-volume
```

**Exemple concret :**

```bash
# Créer un volume pour une base de données PostgreSQL
docker volume create postgres-data

# Créer un volume pour stocker des fichiers uploadés
docker volume create app-uploads

# Créer un volume pour un cache Redis
docker volume create redis-cache
```

> 💡 **Astuce :** Choisissez des noms descriptifs qui indiquent clairement le contenu et l'usage du volume.

### Lister les volumes

Pour voir tous les volumes existants sur votre système :

```bash
docker volume ls
```

**Sortie typique :**

```
DRIVER    VOLUME NAME
local     postgres-data
local     app-uploads
local     redis-cache
local     a3f5b2c1d4e6...    ← Volume anonyme
```

### Inspecter un volume

Pour obtenir des informations détaillées sur un volume :

```bash
docker volume inspect nom-du-volume
```

**Exemple :**

```bash
docker volume inspect postgres-data
```

**Sortie typique :**

```json
[
    {
        "CreatedAt": "2025-01-15T10:30:00Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/postgres-data/_data",
        "Name": "postgres-data",
        "Options": {},
        "Scope": "local"
    }
]
```

Le champ `Mountpoint` indique où Docker stocke physiquement les données sur votre système hôte. **Important :** Vous ne devez généralement pas accéder directement à cet emplacement. Laissez Docker gérer les volumes.

### Supprimer un volume

Pour supprimer un volume que vous n'utilisez plus :

```bash
docker volume rm nom-du-volume
```

**Exemple :**

```bash
docker volume rm postgres-data
```

⚠️ **Attention :** Cette commande supprime définitivement toutes les données du volume. Assurez-vous d'avoir fait une sauvegarde si nécessaire !

> ℹ️ **Note :** Vous ne pouvez pas supprimer un volume qui est actuellement utilisé par un conteneur. Vous devez d'abord arrêter et supprimer le conteneur.

### Nettoyer les volumes inutilisés

Avec le temps, vous pouvez accumuler des volumes qui ne sont plus utilisés. Pour les supprimer tous en une fois :

```bash
docker volume prune
```

Cette commande supprime tous les volumes qui ne sont attachés à aucun conteneur.

⚠️ **Attention :** Utilisez cette commande avec précaution ! Elle peut supprimer des volumes que vous souhaitez conserver.

## Utiliser des volumes avec des conteneurs

### Attacher un volume nommé à un conteneur

Pour utiliser un volume avec un conteneur, vous utilisez l'option `-v` ou `--volume` lors du lancement :

```bash
docker run -v nom-volume:/chemin/dans/conteneur nom-image
```

**Anatomie de la commande :**
- `nom-volume` : Le nom de votre volume
- `:` : Séparateur
- `/chemin/dans/conteneur` : L'emplacement où le volume sera monté dans le conteneur

### Exemple complet : Base de données PostgreSQL

Créons une base de données PostgreSQL avec un volume pour persister les données :

```bash
# Étape 1 : Créer le volume
docker volume create postgres-data

# Étape 2 : Lancer PostgreSQL avec le volume
docker run -d \
  --name ma-base-postgres \
  -e POSTGRES_PASSWORD=motdepasse123 \
  -v postgres-data:/var/lib/postgresql/data \
  postgres:15
```

**Explication ligne par ligne :**
- `-d` : Lance le conteneur en arrière-plan
- `--name ma-base-postgres` : Donne un nom au conteneur
- `-e POSTGRES_PASSWORD=motdepasse123` : Définit le mot de passe PostgreSQL
- `-v postgres-data:/var/lib/postgresql/data` : Attache le volume au répertoire où PostgreSQL stocke ses données
- `postgres:15` : L'image à utiliser

### Test de persistance

Maintenant, testons que nos données persistent vraiment :

```bash
# 1. Se connecter à PostgreSQL et créer une base de données
docker exec -it ma-base-postgres psql -U postgres
# Dans psql, exécutez :
# CREATE DATABASE mon_app;
# \l  (pour lister les bases de données)
# \q  (pour quitter)

# 2. Arrêter et supprimer le conteneur
docker stop ma-base-postgres
docker rm ma-base-postgres

# 3. Relancer un nouveau conteneur avec le MÊME volume
docker run -d \
  --name ma-base-postgres-v2 \
  -e POSTGRES_PASSWORD=motdepasse123 \
  -v postgres-data:/var/lib/postgresql/data \
  postgres:15

# 4. Vérifier que la base de données existe toujours
docker exec -it ma-base-postgres-v2 psql -U postgres
# Dans psql, exécutez :
# \l
# Vous verrez que "mon_app" existe toujours ! 🎉
```

### Exemple : Application Node.js avec uploads

Imaginons une application qui permet aux utilisateurs de télécharger des images :

```bash
# Créer un volume pour les uploads
docker volume create app-uploads

# Lancer l'application
docker run -d \
  --name mon-app-nodejs \
  -p 3000:3000 \
  -v app-uploads:/app/uploads \
  mon-app:latest
```

Maintenant, tous les fichiers uploadés dans `/app/uploads` seront stockés dans le volume `app-uploads` et survivront même si le conteneur est supprimé ou mis à jour.

## Utiliser des volumes anonymes

Les volumes anonymes sont créés automatiquement lorsque vous spécifiez un chemin de montage sans nom de volume.

### Création automatique

```bash
docker run -d \
  --name test-anonyme \
  -v /app/data \
  nginx
```

Notez l'absence de nom avant les `:`. Docker crée automatiquement un volume anonyme et le monte sur `/app/data`.

### Vérification

```bash
docker volume ls
```

Vous verrez apparaître un volume avec un long identifiant aléatoire.

### Suppression automatique avec --rm

Si vous voulez que le volume anonyme soit supprimé automatiquement avec le conteneur :

```bash
docker run -d \
  --rm \
  --name test-temporaire \
  -v /app/cache \
  nginx
```

Lorsque vous arrêtez ce conteneur, le volume sera automatiquement supprimé.

## Partager un volume entre plusieurs conteneurs

Un des grands avantages des volumes est qu'ils peuvent être partagés entre plusieurs conteneurs.

### Exemple : Application web + Worker

```bash
# Créer un volume partagé
docker volume create fichiers-partages

# Lancer l'application web qui génère des fichiers
docker run -d \
  --name app-web \
  -v fichiers-partages:/app/files \
  mon-app-web:latest

# Lancer un worker qui traite ces fichiers
docker run -d \
  --name worker-traitement \
  -v fichiers-partages:/data \
  mon-worker:latest
```

Les deux conteneurs accèdent au même volume :
- `app-web` écrit dans `/app/files`
- `worker-traitement` lit depuis `/data`
- Mais ils accèdent au **même stockage** !

## Volumes nommés vs anonymes : Tableau comparatif

| Critère | Volumes nommés | Volumes anonymes |
|---------|---------------|------------------|
| **Identification** | Nom explicite choisi par vous | Identifiant aléatoire long |
| **Réutilisabilité** | ✅ Facile à réutiliser | ❌ Difficile (ID compliqué) |
| **Gestion** | ✅ Simple avec `docker volume ls` | ⚠️ S'accumulent facilement |
| **Suppression** | Manuel avec `docker volume rm` | Peut être automatique avec `--rm` |
| **Production** | ✅ Recommandé | ❌ Déconseillé |
| **Développement** | ✅ Recommandé | ⚠️ Possible pour tests rapides |
| **Partage** | ✅ Facile entre conteneurs | ⚠️ Difficile à référencer |

## Bonnes pratiques

### 1. Privilégiez les volumes nommés

Sauf pour des tests très temporaires, utilisez toujours des volumes nommés :

```bash
# ✅ BON
docker run -v postgres-data:/var/lib/postgresql/data postgres

# ❌ À ÉVITER
docker run -v /var/lib/postgresql/data postgres
```

### 2. Utilisez des noms descriptifs

Choisissez des noms qui indiquent clairement le contenu et l'usage :

```bash
# ✅ BON
docker volume create monprojet-postgres-data
docker volume create monprojet-uploads
docker volume create monprojet-redis-cache

# ❌ PAS CLAIR
docker volume create vol1
docker volume create data
docker volume create test
```

### 3. Documentez vos volumes

Dans votre documentation ou README, listez les volumes utilisés par votre application :

```markdown
## Volumes requis

- `monapp-db-data` : Données de la base de données PostgreSQL
- `monapp-uploads` : Fichiers uploadés par les utilisateurs
- `monapp-logs` : Logs de l'application
```

### 4. Sauvegardez régulièrement

Les volumes contiennent vos données critiques. Mettez en place une stratégie de sauvegarde :

```bash
# Exemple simple de sauvegarde d'un volume
docker run --rm \
  -v postgres-data:/data \
  -v $(pwd):/backup \
  ubuntu tar czf /backup/postgres-backup.tar.gz /data
```

### 5. Nettoyez régulièrement

Évitez l'accumulation de volumes inutilisés :

```bash
# Lister les volumes non utilisés
docker volume ls -f dangling=true

# Supprimer les volumes non utilisés (avec confirmation)
docker volume prune
```

### 6. Ne mélangez pas les environnements

Utilisez des volumes différents pour développement, staging et production :

```bash
# Développement
docker volume create monapp-dev-db

# Staging
docker volume create monapp-staging-db

# Production
docker volume create monapp-prod-db
```

## Où Docker stocke-t-il les volumes ?

Par curiosité, vous pouvez vous demander où Docker stocke physiquement les données des volumes. L'emplacement varie selon votre système d'exploitation :

### Linux
```
/var/lib/docker/volumes/
```

### macOS (avec Docker Desktop)
```
~/Library/Containers/com.docker.docker/Data/vms/0/data/docker/volumes/
```

### Windows (avec Docker Desktop)
```
\\wsl$\docker-desktop-data\version-pack-data\community\docker\volumes\
```

> ⚠️ **Important :** Même si vous pouvez voir ces emplacements, **ne modifiez jamais directement les fichiers**. Laissez toujours Docker gérer les volumes. Accéder directement à ces emplacements peut corrompre vos données.

## Commandes récapitulatives

Voici un récapitulatif des commandes essentielles pour gérer les volumes :

```bash
# Créer un volume nommé
docker volume create nom-volume

# Lister tous les volumes
docker volume ls

# Inspecter un volume
docker volume inspect nom-volume

# Supprimer un volume
docker volume rm nom-volume

# Supprimer tous les volumes non utilisés
docker volume prune

# Utiliser un volume avec un conteneur
docker run -v nom-volume:/chemin/dans/conteneur image

# Créer et utiliser un volume anonyme
docker run -v /chemin/dans/conteneur image
```

## À retenir

🔑 **Points clés :**

- Les **volumes** sont la méthode **recommandée** pour persister les données dans Docker
- Les **volumes nommés** ont un nom explicite et sont faciles à gérer (recommandés)
- Les **volumes anonymes** ont un identifiant aléatoire et sont plus difficiles à gérer
- Un volume peut être partagé entre plusieurs conteneurs
- Les volumes survivent à la suppression des conteneurs
- Les volumes sont gérés entièrement par Docker (création, stockage, suppression)
- Utilisez des noms descriptifs pour vos volumes
- Mettez en place des sauvegardes régulières de vos volumes critiques

## Prochaine étape

Maintenant que vous maîtrisez les volumes Docker, nous allons explorer une méthode alternative : les **bind mounts** (section 6.3). Bien que les volumes soient recommandés pour la production, les bind mounts sont extrêmement utiles pendant le développement pour modifier vos fichiers en temps réel.

⏭️ [Les bind mounts](/06-gestion-des-donnees/03-les-bind-mounts.md)
