🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 4.1 Rechercher et télécharger des images (pull, search)

## Introduction

Avant de pouvoir créer et exécuter des conteneurs Docker, vous devez disposer d'**images Docker**. Rappelons qu'une image est un modèle en lecture seule qui contient tout le nécessaire pour exécuter une application : le code, les bibliothèques, les dépendances et les fichiers de configuration.

Dans cette section, nous allons apprendre à rechercher des images disponibles et à les télécharger sur votre machine locale.

## Les registres Docker

Les images Docker sont stockées dans des **registres** (registries). Le registre public le plus populaire est **Docker Hub** (https://hub.docker.com), qui contient des milliers d'images officielles et communautaires.

Parmi les images les plus courantes, on trouve :
- `nginx` : serveur web
- `mysql` : base de données MySQL
- `node` : environnement d'exécution Node.js
- `python` : environnement Python
- `ubuntu` : système d'exploitation Ubuntu

## Rechercher des images avec `docker search`

### Syntaxe de base

Pour rechercher une image directement depuis votre terminal, utilisez la commande :

```bash
docker search <terme_de_recherche>
```

### Exemple pratique

Imaginons que vous recherchez une image pour un serveur web Nginx :

```bash
docker search nginx
```

Cette commande retournera une liste d'images correspondantes avec plusieurs colonnes d'information :

```
NAME                      DESCRIPTION                                     STARS     OFFICIAL  
nginx                     Official build of Nginx.                        19000     [OK]  
bitnami/nginx             Bitnami nginx Docker Image                      180  
nginx/nginx-ingress       NGINX and NGINX Plus Ingress Controllers...     89  
...
```

### Comprendre les résultats

- **NAME** : le nom de l'image (celui que vous utiliserez pour la télécharger)
- **DESCRIPTION** : une courte description de l'image
- **STARS** : le nombre d'étoiles attribuées par les utilisateurs (indicateur de popularité)
- **OFFICIAL** : indique si l'image est officielle (maintenue par Docker ou l'éditeur du logiciel)

### Limiter les résultats

Par défaut, `docker search` affiche 25 résultats. Vous pouvez modifier ce nombre :

```bash
docker search --limit 5 nginx
```

### Filtrer les résultats

Vous pouvez filtrer pour n'afficher que les images officielles :

```bash
docker search --filter "is-official=true" nginx
```

Ou filtrer par nombre minimum d'étoiles :

```bash
docker search --filter "stars=100" nginx
```

## Télécharger des images avec `docker pull`

Une fois que vous avez identifié l'image dont vous avez besoin, vous devez la télécharger sur votre machine locale avec la commande `docker pull`.

### Syntaxe de base

```bash
docker pull <nom_de_l_image>
```

### Télécharger une image simple

Pour télécharger l'image officielle de Nginx :

```bash
docker pull nginx
```

Docker va alors :
1. Se connecter au registre Docker Hub
2. Télécharger les différentes couches (layers) de l'image
3. Stocker l'image localement sur votre machine

Vous verrez une sortie similaire à :

```
Using default tag: latest  
latest: Pulling from library/nginx  
a2abf6c4d29d: Pull complete  
a9edb18cadd1: Pull complete  
589b7251471a: Pull complete
...
Digest: sha256:0d17b565c37bcbd895e9d92315a05c1c3c9a29f762b011a10c54a66cd53c9b31  
Status: Downloaded newer image for nginx:latest  
docker.io/library/nginx:latest  
```

## Comprendre les tags et les versions

Les images Docker peuvent avoir plusieurs versions, identifiées par des **tags** (étiquettes).

### Le tag par défaut : `latest`

Quand vous ne spécifiez pas de tag, Docker utilise automatiquement le tag `latest` :

```bash
docker pull nginx
# Équivaut à
docker pull nginx:latest
```

⚠️ **Important** : `latest` ne signifie pas toujours "la dernière version stable". C'est simplement le tag par défaut. Il est recommandé de spécifier un tag précis en production.

### Spécifier un tag spécifique

Pour télécharger une version particulière, ajoutez le tag après le nom de l'image, séparé par deux points :

```bash
docker pull nginx:1.27
```

Ou une version encore plus précise :

```bash
docker pull nginx:1.27.3
```

### Variantes d'images

Certaines images proposent différentes variantes avec des tags descriptifs :

```bash
# Version basée sur Alpine Linux (plus légère)
docker pull nginx:alpine

# Version basée sur une version spécifique d'Alpine
docker pull nginx:1.27-alpine

# Version basée sur Debian
docker pull python:3.11-slim
```

### Comment trouver les tags disponibles ?

Pour connaître tous les tags disponibles pour une image, consultez :
- La page Docker Hub de l'image : https://hub.docker.com/_/nginx
- L'onglet "Tags" qui liste toutes les versions disponibles

## Vérifier les images téléchargées

Pour voir toutes les images présentes sur votre machine locale :

```bash
docker images
```

Ou :

```bash
docker image ls
```

Cette commande affiche :

```
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE  
nginx         latest    605c77e624dd   2 weeks ago    141MB  
nginx         alpine    8e75cbc5b25c   2 weeks ago    41MB  
python        3.11      a7a6f5d0f4e6   3 weeks ago    917MB  
```

## Télécharger depuis un registre privé

Par défaut, Docker recherche les images sur Docker Hub. Pour télécharger depuis un registre privé, spécifiez l'URL complète :

```bash
docker pull mon-registre.com:5000/mon-image:v1.0
```

## Bonnes pratiques

1. **Privilégiez les images officielles** : elles sont maintenues, sécurisées et bien documentées
2. **Spécifiez toujours un tag en production** : évitez `latest` pour garantir la reproductibilité
3. **Vérifiez le nombre d'étoiles et la date de mise à jour** : cela indique la fiabilité de l'image
4. **Préférez les images légères** : les variantes `alpine` ou `slim` sont plus petites et se téléchargent plus rapidement
5. **Lisez la documentation** : chaque image officielle sur Docker Hub a une page avec des instructions d'utilisation

## Résumé

Dans cette section, vous avez appris à :

- ✅ Rechercher des images Docker avec `docker search`
- ✅ Comprendre les résultats de recherche (images officielles, étoiles, descriptions)
- ✅ Télécharger des images avec `docker pull`
- ✅ Utiliser les tags pour spécifier des versions précises
- ✅ Vérifier les images disponibles localement avec `docker images`

Vous êtes maintenant prêt à télécharger les images dont vous avez besoin pour créer vos premiers conteneurs Docker !

⏭️ [Lancer et gérer des conteneurs (run, start, stop, restart)](/04-commandes-docker-essentielles/02-lancer-et-gerer-des-conteneurs.md)
