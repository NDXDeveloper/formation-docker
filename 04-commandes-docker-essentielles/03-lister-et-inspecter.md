🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 4.3 Lister et inspecter (ps, images, inspect)

## Introduction

Une fois que vous avez créé des conteneurs et téléchargé des images, vous devez pouvoir **voir ce qui existe sur votre système** et **obtenir des informations détaillées** sur ces ressources. C'est exactement ce que permettent les commandes de cette section.

Pensez à ces commandes comme à des **outils de diagnostic** qui vous donnent une vue complète de votre environnement Docker. Elles sont essentielles pour :
- Savoir quels conteneurs sont en cours d'exécution
- Vérifier quelles images sont disponibles localement
- Obtenir des informations techniques détaillées
- Déboguer des problèmes

## La commande `docker ps` : lister les conteneurs

### Qu'est-ce que `docker ps` ?

La commande `docker ps` (pour "process status") affiche la liste des conteneurs Docker. Par défaut, elle montre **uniquement les conteneurs en cours d'exécution**.

💡 **Origine du nom** : `ps` vient du monde Unix/Linux où cette commande liste les processus en cours.

### Syntaxe de base

```bash
docker ps
```

### Exemple de sortie

```
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                  NAMES
f4d5e8a9b2c1   nginx     "/docker-entrypoint.…"   5 minutes ago   Up 5 minutes   0.0.0.0:8080->80/tcp   mon-nginx
a1b2c3d4e5f6   mysql     "docker-entrypoint.s…"   2 hours ago     Up 2 hours     3306/tcp, 33060/tcp    ma-base
```

### Comprendre les colonnes

Analysons chaque colonne en détail :

| Colonne | Description | Exemple |
|---------|-------------|---------|
| **CONTAINER ID** | Identifiant unique court du conteneur (12 premiers caractères) | `f4d5e8a9b2c1` |
| **IMAGE** | Nom de l'image utilisée pour créer le conteneur | `nginx` |
| **COMMAND** | Commande exécutée au démarrage du conteneur | `"/docker-entrypoint.…"` |
| **CREATED** | Quand le conteneur a été créé | `5 minutes ago` |
| **STATUS** | État actuel du conteneur | `Up 5 minutes` |
| **PORTS** | Mappages de ports entre l'hôte et le conteneur | `0.0.0.0:8080->80/tcp` |
| **NAMES** | Nom du conteneur (auto-généré ou défini par l'utilisateur) | `mon-nginx` |

### Afficher TOUS les conteneurs avec `-a`

Par défaut, `docker ps` n'affiche que les conteneurs actifs. Pour voir **tous les conteneurs** (y compris ceux qui sont arrêtés) :

```bash
docker ps -a
```

Ou la version longue :

```bash
docker ps --all
```

Exemple de sortie incluant des conteneurs arrêtés :

```
CONTAINER ID   IMAGE         STATUS                     NAMES
f4d5e8a9b2c1   nginx         Up 5 minutes               mon-nginx
a1b2c3d4e5f6   mysql         Up 2 hours                 ma-base
9e8d7c6b5a4f   ubuntu        Exited (0) 10 minutes ago  test-ubuntu
3b2a1c0d9e8f   hello-world   Exited (0) 3 days ago      modest_tesla
```

Notez la colonne STATUS :
- `Up X minutes/hours` = conteneur en cours d'exécution
- `Exited (0)` = conteneur arrêté normalement (code de sortie 0)
- `Exited (1)` = conteneur arrêté avec une erreur (code de sortie différent de 0)

### Afficher uniquement les IDs avec `-q`

L'option `-q` (quiet) affiche uniquement les IDs des conteneurs, sans les autres informations :

```bash
docker ps -q
```

Sortie :
```
f4d5e8a9b2c1
a1b2c3d4e5f6
```

C'est très utile pour les scripts et pour combiner avec d'autres commandes :

```bash
# Arrêter tous les conteneurs en cours
docker stop $(docker ps -q)

# Supprimer tous les conteneurs arrêtés
docker rm $(docker ps -aq -f status=exited)
```

### Filtrer les conteneurs avec `-f`

L'option `--filter` (ou `-f`) permet de filtrer les résultats selon différents critères :

**Filtrer par statut** :
```bash
# Conteneurs en cours d'exécution
docker ps -f status=running

# Conteneurs arrêtés
docker ps -af status=exited

# Conteneurs en pause
docker ps -f status=paused
```

**Filtrer par nom** :
```bash
docker ps -f name=nginx
```

**Filtrer par image** :
```bash
docker ps -f ancestor=nginx
```

**Combiner plusieurs filtres** :
```bash
docker ps -f status=running -f name=web
```

### Personnaliser l'affichage avec `--format`

Vous pouvez personnaliser les colonnes affichées avec l'option `--format` :

```bash
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}"
```

Sortie :
```
CONTAINER ID   NAMES         STATUS
f4d5e8a9b2c1   mon-nginx     Up 5 minutes
a1b2c3d4e5f6   ma-base       Up 2 hours
```

Variables disponibles :
- `.ID` : ID du conteneur
- `.Names` : Nom du conteneur
- `.Image` : Nom de l'image
- `.Status` : Statut
- `.Ports` : Ports mappés
- `.Command` : Commande exécutée
- `.CreatedAt` : Date de création

### Afficher la taille des conteneurs avec `-s`

Pour voir l'espace disque utilisé par chaque conteneur :

```bash
docker ps -s
```

Cette option ajoute une colonne SIZE qui montre :
- La taille de la couche inscriptible du conteneur
- La taille totale (image + couche inscriptible)

### Afficher les derniers conteneurs créés avec `-n`

Pour voir les N derniers conteneurs créés :

```bash
docker ps -n 5
```

Affiche les 5 derniers conteneurs créés, qu'ils soient actifs ou arrêtés.

### Afficher uniquement les derniers conteneurs avec `-l`

Pour voir uniquement le dernier conteneur créé :

```bash
docker ps -l
```

## La commande `docker images` : lister les images

### Qu'est-ce que `docker images` ?

La commande `docker images` (ou `docker image ls`) affiche la liste de toutes les images Docker présentes **localement** sur votre machine.

### Syntaxe de base

```bash
docker images
```

Ou la syntaxe alternative :

```bash
docker image ls
```

### Exemple de sortie

```
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
nginx         latest    605c77e624dd   2 weeks ago    141MB
nginx         alpine    8e75cbc5b25c   2 weeks ago    41MB
mysql         8.0       3218b38490ce   3 weeks ago    517MB
python        3.11      a7a6f5d0f4e6   1 month ago    917MB
ubuntu        22.04     5a81c4b8502e   2 months ago   77.8MB
```

### Comprendre les colonnes

| Colonne | Description | Exemple |
|---------|-------------|---------|
| **REPOSITORY** | Nom de l'image (dépôt) | `nginx` |
| **TAG** | Version ou variante de l'image | `latest`, `alpine`, `1.25` |
| **IMAGE ID** | Identifiant unique de l'image (12 premiers caractères) | `605c77e624dd` |
| **CREATED** | Quand l'image a été créée (par le créateur) | `2 weeks ago` |
| **SIZE** | Taille de l'image sur le disque | `141MB` |

### Comprendre REPOSITORY et TAG

Une image complète est identifiée par `REPOSITORY:TAG` :

```
nginx:latest
nginx:1.25
nginx:alpine
python:3.11-slim
```

- Si vous ne spécifiez pas de tag, Docker utilise automatiquement `latest`
- Plusieurs tags peuvent pointer vers la même image (même IMAGE ID)

### Afficher toutes les images avec `-a`

Par défaut, `docker images` masque les images intermédiaires. Pour tout voir :

```bash
docker images -a
```

Les images intermédiaires sont créées pendant la construction d'images et apparaissent avec `<none>` dans REPOSITORY et TAG.

### Afficher uniquement les IDs avec `-q`

Comme pour `docker ps`, l'option `-q` n'affiche que les IDs :

```bash
docker images -q
```

Sortie :
```
605c77e624dd
8e75cbc5b25c
3218b38490ce
```

Utile pour des opérations en masse :

```bash
# Supprimer toutes les images (attention !)
docker rmi $(docker images -q)
```

### Filtrer les images avec `-f`

**Filtrer par nom** :
```bash
docker images -f reference=nginx
```

**Filtrer les images "pendantes" (dangling)** :

Les images "dangling" sont des images qui n'ont plus de tag (souvent après reconstruction) :

```bash
docker images -f dangling=true
```

Sortie :
```
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
<none>       <none>    4a5e9c7d8b3f   1 hour ago     150MB
```

Pour les supprimer :
```bash
docker image prune
```

**Filtrer par date de création** :
```bash
# Images créées avant nginx:latest
docker images -f before=nginx:latest

# Images créées après python:3.11
docker images -f since=python:3.11
```

### Personnaliser l'affichage

```bash
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

Sortie :
```
REPOSITORY   TAG       SIZE
nginx        latest    141MB
mysql        8.0       517MB
python       3.11      917MB
```

### Afficher les digest avec `--digests`

Le digest est une empreinte cryptographique unique de l'image :

```bash
docker images --digests
```

Sortie :
```
REPOSITORY   TAG      DIGEST                                                                    IMAGE ID       SIZE
nginx        latest   sha256:0d17b565c37bcbd895e9d92315a05c1c3c9a29f762b011a10c54a66cd53c9b31   605c77e624dd   141MB
```

## La commande `docker inspect` : inspection détaillée

### Qu'est-ce que `docker inspect` ?

`docker inspect` est la commande la plus puissante pour obtenir des **informations détaillées** sur n'importe quelle ressource Docker (conteneurs, images, volumes, réseaux). Elle retourne un JSON complet avec toutes les métadonnées.

### Syntaxe de base

```bash
docker inspect <nom_ou_id_conteneur>
```

Ou pour une image :

```bash
docker inspect <nom_image>
```

### Inspecter un conteneur

```bash
docker inspect mon-nginx
```

Cette commande retourne un JSON volumineux contenant toutes les informations :

```json
[
    {
        "Id": "f4d5e8a9b2c1d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8",
        "Created": "2025-10-22T10:30:45.123456789Z",
        "Path": "/docker-entrypoint.sh",
        "Args": ["nginx", "-g", "daemon off;"],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 12345,
            "ExitCode": 0,
            "StartedAt": "2025-10-22T10:30:46.123456789Z"
        },
        "Image": "sha256:605c77e624dd...",
        "Name": "/mon-nginx",
        "HostConfig": {
            "PortBindings": {
                "80/tcp": [{"HostIp": "", "HostPort": "8080"}]
            }
        },
        "NetworkSettings": {
            "IPAddress": "172.17.0.2",
            "Ports": {
                "80/tcp": [{"HostIp": "0.0.0.0", "HostPort": "8080"}]
            }
        },
        "Mounts": []
    }
]
```

### Extraire une information spécifique avec `--format`

Le JSON complet est souvent trop verbeux. Utilisez `--format` pour extraire uniquement ce qui vous intéresse :

**Obtenir l'adresse IP d'un conteneur** :
```bash
docker inspect --format='{{.NetworkSettings.IPAddress}}' mon-nginx
```

Sortie :
```
172.17.0.2
```

**Obtenir le statut d'un conteneur** :
```bash
docker inspect --format='{{.State.Status}}' mon-nginx
```

**Obtenir les variables d'environnement** :
```bash
docker inspect --format='{{.Config.Env}}' mon-nginx
```

**Obtenir les ports mappés** :
```bash
docker inspect --format='{{.NetworkSettings.Ports}}' mon-nginx
```

**Obtenir le chemin des logs** :
```bash
docker inspect --format='{{.LogPath}}' mon-nginx
```

### Templates Go utiles

Docker utilise les templates Go pour l'option `--format`. Voici quelques exemples pratiques :

**Afficher plusieurs champs** :
```bash
docker inspect --format='Nom: {{.Name}} | IP: {{.NetworkSettings.IPAddress}} | Status: {{.State.Status}}' mon-nginx
```

**Parcourir les variables d'environnement** :
```bash
docker inspect --format='{{range .Config.Env}}{{println .}}{{end}}' mon-nginx
```

**Vérifier si un conteneur est en cours d'exécution** :
```bash
docker inspect --format='{{.State.Running}}' mon-nginx
```

Retourne `true` ou `false`.

### Inspecter une image

```bash
docker inspect nginx:latest
```

Les informations incluent :
- L'historique des layers
- Les variables d'environnement par défaut
- Les ports exposés
- Le point d'entrée et la commande par défaut
- La taille de l'image

**Exemples utiles pour les images** :

Obtenir la commande par défaut :
```bash
docker inspect --format='{{.Config.Cmd}}' nginx
```

Obtenir les ports exposés :
```bash
docker inspect --format='{{.Config.ExposedPorts}}' nginx
```

Obtenir la taille :
```bash
docker inspect --format='{{.Size}}' nginx
```

### Inspecter plusieurs ressources

Vous pouvez inspecter plusieurs conteneurs ou images en une seule commande :

```bash
docker inspect conteneur1 conteneur2 image1
```

## Commandes alternatives et raccourcis

### `docker container ls` vs `docker ps`

Ces deux commandes sont équivalentes :

```bash
docker ps
docker container ls
```

La syntaxe `docker container ls` fait partie de la nouvelle syntaxe Docker qui regroupe les commandes par type de ressource.

### `docker image ls` vs `docker images`

Pareil pour les images :

```bash
docker images
docker image ls
```

### Nouvelle syntaxe recommandée

Docker encourage désormais une syntaxe plus structurée :

| Ancienne syntaxe | Nouvelle syntaxe |
|------------------|------------------|
| `docker ps` | `docker container ls` |
| `docker images` | `docker image ls` |
| `docker rm` | `docker container rm` |
| `docker rmi` | `docker image rm` |

Les deux fonctionnent, mais la nouvelle syntaxe est plus explicite et cohérente.

## Cas d'usage pratiques

### Trouver l'IP d'un conteneur

```bash
docker inspect --format='{{.NetworkSettings.IPAddress}}' mon-conteneur
```

### Vérifier les ports d'un conteneur

```bash
docker inspect --format='{{json .NetworkSettings.Ports}}' mon-conteneur | python -m json.tool
```

### Lister tous les conteneurs avec leur IP

```bash
docker ps -q | xargs docker inspect --format='{{.Name}} - {{.NetworkSettings.IPAddress}}'
```

### Afficher l'utilisation de mémoire des conteneurs

```bash
docker ps --format="table {{.Names}}\t{{.Status}}" && docker stats --no-stream
```

### Trouver les conteneurs qui utilisent une image spécifique

```bash
docker ps -a --filter ancestor=nginx
```

### Compter le nombre total d'images

```bash
docker images | wc -l
```

## Comprendre les sorties

### États possibles d'un conteneur

| État | Description |
|------|-------------|
| **created** | Conteneur créé mais jamais démarré |
| **running** | Conteneur en cours d'exécution |
| **paused** | Conteneur mis en pause |
| **restarting** | Conteneur en cours de redémarrage |
| **exited** | Conteneur arrêté |
| **dead** | Conteneur dans un état non récupérable |

### Codes de sortie importants

Quand un conteneur s'arrête avec `Exited (code)`, le code indique pourquoi :

| Code | Signification |
|------|---------------|
| **0** | Arrêt normal, sans erreur |
| **1** | Erreur d'application |
| **137** | Le conteneur a été tué (SIGKILL) - souvent manque de mémoire |
| **139** | Segmentation fault |
| **143** | Arrêt gracieux (SIGTERM) |

## Bonnes pratiques

### 1. Utilisez `-f` pour filtrer plutôt que grep

```bash
# ✅ Bon - utilise les filtres Docker
docker ps -f name=nginx

# ❌ Moins efficace
docker ps | grep nginx
```

### 2. Utilisez `--format` pour des scripts

```bash
# ✅ Bon - sortie structurée
docker ps --format '{{.Names}}'

# ❌ Difficile à parser
docker ps | awk '{print $NF}'
```

### 3. Combinez `-q` avec d'autres commandes

```bash
# Arrêter tous les conteneurs nginx
docker stop $(docker ps -q -f name=nginx)
```

### 4. Inspectez avant de déboguer

Avant de chercher un problème, inspectez d'abord le conteneur :

```bash
docker inspect mon-conteneur
docker logs mon-conteneur
```

### 5. Nettoyez régulièrement

Vérifiez l'espace utilisé :

```bash
docker system df
```

Supprimez les ressources inutilisées :

```bash
docker system prune
```

## Résumé

Dans cette section, vous avez appris à :

- ✅ **Lister les conteneurs** avec `docker ps` et ses nombreuses options
- ✅ **Lister les images** avec `docker images`
- ✅ **Filtrer les résultats** avec `-f` pour trouver rapidement ce que vous cherchez
- ✅ **Inspecter en détail** avec `docker inspect` pour obtenir toutes les métadonnées
- ✅ **Extraire des informations spécifiques** avec `--format`
- ✅ **Comprendre les états** des conteneurs et leur signification
- ✅ Utiliser les **commandes de manière efficace** dans vos workflows

Ces commandes de diagnostic sont essentielles pour comprendre l'état de votre environnement Docker. Vous les utiliserez constamment, que ce soit pour vérifier qu'un conteneur fonctionne correctement, trouver une adresse IP, ou déboguer un problème.

Dans la section suivante, vous apprendrez à supprimer proprement les conteneurs et les images pour maintenir un environnement Docker sain et organisé.

⏭️ [Supprimer conteneurs et images (rm, rmi, prune)](/04-commandes-docker-essentielles/04-supprimer-conteneurs-et-images.md)
