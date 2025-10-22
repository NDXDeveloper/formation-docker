🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 5.2 Instructions de base (FROM, RUN, COPY, ADD) et la différence cruciale entre CMD et ENTRYPOINT

## Introduction

Dans cette section, nous allons explorer les **instructions fondamentales** que vous utiliserez dans presque tous vos Dockerfiles. Ces cinq instructions constituent la base de la création d'images :

- **FROM** : Définit l'image de base (obligatoire, toujours en premier)
- **RUN** : Exécute des commandes pendant la construction
- **COPY** : Copie des fichiers depuis votre machine vers l'image
- **ADD** : Alternative à COPY avec des fonctionnalités supplémentaires
- **CMD** et **ENTRYPOINT** : Définissent ce qui s'exécute au démarrage du conteneur

Maîtriser ces instructions vous permettra de créer 90% de vos images Docker !

## L'instruction FROM : choisir l'image de base

### Qu'est-ce que FROM ?

`FROM` est l'instruction **la plus importante** et **obligatoire** dans tout Dockerfile. Elle définit l'**image de base** sur laquelle vous allez construire votre image personnalisée.

💡 **Analogie** : Si vous construisez une maison, `FROM` choisit le type de fondation et de structure de base.

### Syntaxe

```dockerfile
FROM <image>:<tag>
```

### Exemples simples

```dockerfile
# Partir d'Ubuntu 22.04
FROM ubuntu:22.04

# Partir de Python 3.11
FROM python:3.11

# Partir de Node.js 18
FROM node:18

# Partir d'Alpine Linux (très léger)
FROM alpine:latest

# Partir de Nginx
FROM nginx:alpine
```

### Règles importantes

1. **FROM doit être la première instruction** (sauf pour les commentaires et ARG avant FROM)

```dockerfile
# ✅ Correct
FROM python:3.11
RUN pip install flask

# ❌ Incorrect - FROM doit être en premier
RUN apt-get update
FROM python:3.11
```

2. **Spécifiez toujours un tag** pour garantir la reproductibilité

```dockerfile
# ✅ Bon - version spécifique
FROM python:3.11.5

# ⚠️ Acceptable pour le développement
FROM python:3.11

# ❌ À éviter en production - version changeante
FROM python:latest
```

### Choisir la bonne image de base

Docker Hub propose des milliers d'images. Voici comment choisir :

#### Images officielles vs communautaires

**Images officielles** (recommandées) :
```dockerfile
FROM python:3.11      # Officielle
FROM node:18          # Officielle
FROM mysql:8.0        # Officielle
```

**Images communautaires** :
```dockerfile
FROM user/custom-python:3.11  # Créée par un utilisateur
```

#### Variantes d'images : standard, slim, alpine

La plupart des images officielles proposent plusieurs variantes :

```dockerfile
# Standard - image complète (~900 MB pour Python)
FROM python:3.11

# Slim - image allégée (~150 MB)
FROM python:3.11-slim

# Alpine - image ultra-légère (~50 MB)
FROM python:3.11-alpine
```

**Comparaison** :

| Variante | Taille | Outils inclus | Cas d'usage |
|----------|--------|---------------|-------------|
| **Standard** | Grande | Tous | Développement, beaucoup de dépendances |
| **Slim** | Moyenne | L'essentiel | Production, compromis taille/fonctionnalité |
| **Alpine** | Petite | Minimum | Production, optimisation maximale |

**Recommandation pour débutants** : Commencez avec la variante **slim** qui offre le meilleur compromis.

### Images spécialisées

Pour certains besoins, utilisez des images déjà configurées :

```dockerfile
# Pour une application Node.js
FROM node:18-slim

# Pour une application Python avec Jupyter
FROM jupyter/base-notebook

# Pour une base de données PostgreSQL
FROM postgres:15

# Pour un serveur web Nginx
FROM nginx:alpine
```

### FROM scratch : partir de zéro

Pour les experts, il existe une image spéciale `scratch` (vide) :

```dockerfile
FROM scratch
# Image totalement vide, pour binaires statiques uniquement
```

C'est très avancé et rarement utilisé. En tant que débutant, utilisez toujours une image de base établie.

### Exemples complets

**Application Python Flask** :
```dockerfile
FROM python:3.11-slim
# Suite du Dockerfile...
```

**Application Node.js Express** :
```dockerfile
FROM node:18-alpine
# Suite du Dockerfile...
```

**Site web statique** :
```dockerfile
FROM nginx:alpine
# Suite du Dockerfile...
```

## L'instruction RUN : exécuter des commandes

### Qu'est-ce que RUN ?

`RUN` exécute des **commandes shell** pendant la **construction de l'image**. C'est ici que vous installez des packages, configurez des outils, créez des fichiers, etc.

⚠️ **Important** : `RUN` s'exécute au moment du **build**, pas au démarrage du conteneur.

### Syntaxe

Il existe deux formes :

**Forme shell** (la plus courante) :
```dockerfile
RUN <commande>
```

**Forme exec** (pour plus de contrôle) :
```dockerfile
RUN ["executable", "param1", "param2"]
```

### Exemples simples

```dockerfile
# Mettre à jour les packages système
RUN apt-get update

# Installer un package
RUN apt-get install -y curl

# Installer des dépendances Python
RUN pip install flask numpy pandas

# Installer des dépendances Node.js
RUN npm install express

# Créer un répertoire
RUN mkdir -p /app/data

# Créer un fichier
RUN echo "Hello Docker" > /app/hello.txt
```

### Chaîner plusieurs commandes

Vous pouvez chaîner plusieurs commandes avec `&&` :

```dockerfile
# ❌ Crée 3 couches distinctes
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y git

# ✅ Crée 1 seule couche
RUN apt-get update && \
    apt-get install -y curl && \
    apt-get install -y git
```

**Pourquoi c'est mieux ?**
- Moins de couches = image plus légère
- Plus rapide à construire
- Plus facile à maintenir

### Retour à la ligne avec backslash

Pour améliorer la lisibilité, utilisez `\` :

```dockerfile
RUN apt-get update && \
    apt-get install -y \
        curl \
        git \
        vim \
        wget && \
    rm -rf /var/lib/apt/lists/*
```

**Notez** :
- Le `\` à la fin de chaque ligne (sauf la dernière)
- L'indentation pour la lisibilité
- Le nettoyage à la fin (`rm -rf /var/lib/apt/lists/*`) pour réduire la taille

### Bonnes pratiques avec RUN

#### 1. Combiner update et install

```dockerfile
# ✅ Bon - évite les problèmes de cache
RUN apt-get update && apt-get install -y \
    package1 \
    package2

# ❌ Mauvais - peut utiliser un cache obsolète
RUN apt-get update
RUN apt-get install -y package1
```

#### 2. Nettoyer après installation

```dockerfile
# Debian/Ubuntu
RUN apt-get update && apt-get install -y \
    curl \
    git && \
    rm -rf /var/lib/apt/lists/*

# Alpine Linux
RUN apk add --no-cache curl git

# Python
RUN pip install --no-cache-dir flask pandas
```

#### 3. Trier alphabétiquement les packages

```dockerfile
RUN apt-get update && apt-get install -y \
    curl \
    git \
    vim \
    wget
```

Plus facile à lire et à maintenir !

### Cas d'usage courants

**Installer des dépendances système** :
```dockerfile
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y \
    build-essential \
    python3-dev \
    libpq-dev
```

**Installer des dépendances Python** :
```dockerfile
FROM python:3.11-slim
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
```

**Installer des dépendances Node.js** :
```dockerfile
FROM node:18-alpine
COPY package*.json ./
RUN npm ci --only=production
```

**Créer un utilisateur non-root** :
```dockerfile
RUN useradd -m -u 1000 appuser
```

### Erreurs courantes

**Oublier le nettoyage** :
```dockerfile
# ❌ Laisse des fichiers temporaires
RUN apt-get update
RUN apt-get install -y curl

# ✅ Nettoie dans la même couche
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*
```

**Trop de RUN** :
```dockerfile
# ❌ Crée beaucoup de couches
RUN pip install flask
RUN pip install requests
RUN pip install numpy

# ✅ Une seule couche
RUN pip install flask requests numpy
```

## L'instruction COPY : copier des fichiers

### Qu'est-ce que COPY ?

`COPY` copie des **fichiers ou répertoires** depuis votre machine (le contexte de build) vers l'image Docker.

💡 **Analogie** : C'est comme faire un copier-coller de votre disque dur vers l'image.

### Syntaxe

```dockerfile
COPY <source> <destination>
```

- **source** : chemin sur votre machine (relatif au contexte de build)
- **destination** : chemin dans l'image Docker

### Exemples simples

```dockerfile
# Copier un fichier
COPY app.py /app/

# Copier plusieurs fichiers
COPY app.py config.json /app/

# Copier tout un répertoire
COPY ./src /app/src

# Copier tout le contenu du répertoire actuel
COPY . /app

# Copier et renommer
COPY app.py /app/main.py
```

### Chemins relatifs et absolus

**Chemins source** (toujours relatifs au contexte de build) :
```dockerfile
COPY ./app.py /app/          # Relatif explicite
COPY app.py /app/            # Relatif implicite
COPY ../config.json /app/    # ❌ ERREUR - ne peut pas sortir du contexte
```

**Chemins destination** :
```dockerfile
COPY app.py /app/            # Absolu
COPY app.py ./app/           # Relatif au WORKDIR
COPY app.py app/             # Relatif au WORKDIR
```

### Wildcards (jokers)

Utilisez `*` et `?` pour copier plusieurs fichiers :

```dockerfile
# Copier tous les fichiers .py
COPY *.py /app/

# Copier tous les fichiers .json et .yaml
COPY *.json *.yaml /config/

# Copier tous les fichiers commençant par "test"
COPY test* /app/tests/
```

### COPY avec WORKDIR

Si vous avez défini un `WORKDIR`, les chemins de destination deviennent relatifs :

```dockerfile
FROM python:3.11
WORKDIR /app

# Ces deux lignes sont équivalentes
COPY app.py /app/app.py
COPY app.py app.py

# Copier dans le WORKDIR actuel
COPY . .
```

### Bonnes pratiques avec COPY

#### 1. Copier seulement ce qui est nécessaire

```dockerfile
# ❌ Copie tout, y compris node_modules/, .git/, etc.
COPY . /app

# ✅ Utilise .dockerignore pour exclure les fichiers inutiles
# Créer un fichier .dockerignore avec :
# node_modules/
# .git/
# *.log
COPY . /app
```

#### 2. Ordre stratégique pour le cache

```dockerfile
# ✅ Bon - copie les dépendances d'abord
FROM node:18
WORKDIR /app
COPY package*.json ./        # Change rarement
RUN npm install              # Utilise le cache si package.json n'a pas changé
COPY . .                     # Change souvent

# ❌ Moins efficace
FROM node:18
WORKDIR /app
COPY . .                     # Change souvent, invalide tout le cache
RUN npm install              # Réinstalle à chaque fois
```

#### 3. Copier avec la bonne structure

```dockerfile
# Source sur votre machine :
# mon-projet/
# ├── src/
# │   ├── app.py
# │   └── utils.py
# └── config/
#     └── settings.json

FROM python:3.11
WORKDIR /app

# Préserver la structure
COPY src/ ./src/
COPY config/ ./config/

# Résultat dans l'image :
# /app/
# ├── src/
# │   ├── app.py
# │   └── utils.py
# └── config/
#     └── settings.json
```

### Options avancées

#### --chown : définir le propriétaire

```dockerfile
# Copier et définir l'utilisateur propriétaire
COPY --chown=user:group app.py /app/

# Exemple avec UID/GID numériques
COPY --chown=1000:1000 . /app
```

#### --chmod : définir les permissions

```dockerfile
# Copier avec des permissions spécifiques
COPY --chmod=755 script.sh /usr/local/bin/
```

### Exemples complets

**Application Python** :
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY src/ ./src/
COPY config.yaml .
```

**Application Node.js** :
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
```

**Site web statique** :
```dockerfile
FROM nginx:alpine
COPY html/ /usr/share/nginx/html/
COPY nginx.conf /etc/nginx/nginx.conf
```

## L'instruction ADD : alternative à COPY

### Qu'est-ce que ADD ?

`ADD` est similaire à `COPY`, mais avec des **fonctionnalités supplémentaires** :
- Extraction automatique d'archives tar
- Téléchargement de fichiers depuis des URLs

### Syntaxe

```dockerfile
ADD <source> <destination>
```

### Fonctionnalités spéciales

#### 1. Extraction automatique des archives

```dockerfile
# Copie ET extrait automatiquement
ADD app.tar.gz /app/

# Résultat : le contenu de app.tar.gz est extrait dans /app/
```

Formats supportés :
- `.tar`
- `.tar.gz`, `.tgz`
- `.tar.bz2`, `.tbz2`
- `.tar.xz`, `.txz`

#### 2. Téléchargement depuis une URL

```dockerfile
# Télécharge un fichier depuis Internet
ADD https://example.com/config.json /app/
```

⚠️ **Attention** : Les fichiers téléchargés via URL ne sont **pas extraits** automatiquement, même si ce sont des archives.

### COPY vs ADD : que choisir ?

**Règle générale** : Préférez toujours `COPY` sauf si vous avez besoin des fonctionnalités spéciales de `ADD`.

```dockerfile
# ✅ Préféré - plus clair et explicite
COPY app.py /app/

# ⚠️ Fonctionne mais déconseillé sans raison
ADD app.py /app/
```

**Quand utiliser ADD** :

✅ Extraction automatique d'une archive locale :
```dockerfile
ADD my-app.tar.gz /app/
```

✅ (Rare) Téléchargement d'un fichier :
```dockerfile
ADD https://example.com/file.txt /app/
```

**Quand utiliser COPY** :

✅ Tous les autres cas (99% du temps) :
```dockerfile
COPY . /app/
COPY config.json /app/
COPY src/ /app/src/
```

### Pourquoi préférer COPY ?

1. **Plus clair** : indique clairement l'intention (copier)
2. **Plus sûr** : pas de comportement "magique" inattendu
3. **Meilleur pour le cache** : comportement plus prévisible
4. **Recommandation officielle Docker**

### Exemples comparatifs

**Copier un fichier simple** :
```dockerfile
# ✅ Utilisez COPY
COPY app.py /app/

# ❌ ADD n'apporte rien ici
ADD app.py /app/
```

**Copier une archive SANS l'extraire** :
```dockerfile
# ✅ COPY préserve l'archive
COPY backup.tar.gz /backups/

# ❌ ADD extrairait automatiquement
ADD backup.tar.gz /backups/
```

**Extraire une archive** :
```dockerfile
# ✅ ADD extrait automatiquement
ADD app.tar.gz /app/

# Alternative avec COPY (plus verbeux mais plus clair)
COPY app.tar.gz /tmp/
RUN tar -xzf /tmp/app.tar.gz -C /app/ && \
    rm /tmp/app.tar.gz
```

### Tableau récapitulatif

| Action | COPY | ADD |
|--------|------|-----|
| Copier un fichier | ✅ Recommandé | ⚠️ Fonctionne mais déconseillé |
| Copier un répertoire | ✅ Recommandé | ⚠️ Fonctionne mais déconseillé |
| Extraire une archive tar | ❌ Non supporté | ✅ Automatique |
| Télécharger depuis URL | ❌ Non supporté | ✅ Supporté |
| Clarté | ✅ Très clair | ⚠️ Comportement "magique" |
| Recommandation Docker | ✅ Préféré | ⚠️ Cas spéciaux uniquement |

## CMD et ENTRYPOINT : la différence cruciale

### Introduction

`CMD` et `ENTRYPOINT` sont les instructions les plus **mal comprises** par les débutants Docker. Elles définissent toutes deux ce qui s'exécute au **démarrage du conteneur**, mais leur comportement est très différent.

💡 **Point clé** : Comprendre leur différence est essentiel pour créer des images flexibles et réutilisables.

## L'instruction CMD

### Qu'est-ce que CMD ?

`CMD` définit la **commande par défaut** qui s'exécute quand le conteneur démarre. Cette commande peut être **remplacée** facilement depuis la ligne de commande.

### Syntaxe

Il existe trois formes :

**Forme exec** (recommandée) :
```dockerfile
CMD ["executable", "param1", "param2"]
```

**Forme shell** :
```dockerfile
CMD commande param1 param2
```

**Forme paramètres** (utilisée avec ENTRYPOINT) :
```dockerfile
CMD ["param1", "param2"]
```

### Exemples

```dockerfile
# Démarrer une application Python
CMD ["python", "app.py"]

# Démarrer un serveur Node.js
CMD ["node", "server.js"]

# Lancer un shell
CMD ["/bin/bash"]

# Commande avec arguments
CMD ["npm", "start"]
```

### Comportement : facilement remplaçable

La commande `CMD` peut être **remplacée** lors du `docker run` :

**Dockerfile** :
```dockerfile
FROM ubuntu:22.04
CMD ["echo", "Hello from CMD"]
```

**Utilisation** :
```bash
# Utilise le CMD du Dockerfile
docker run mon-image
# Sortie : Hello from CMD

# Remplace le CMD
docker run mon-image echo "Hello World"
# Sortie : Hello World

# Remplace par une autre commande
docker run mon-image ls -la
# Sortie : liste des fichiers
```

### Règle importante

**Un seul CMD par Dockerfile** : Si vous définissez plusieurs `CMD`, seul le dernier est pris en compte.

```dockerfile
FROM ubuntu:22.04
CMD ["echo", "Premier"]
CMD ["echo", "Deuxième"]  # ← Seul celui-ci sera exécuté
CMD ["echo", "Troisième"] # ← Celui-ci remplace les précédents
```

## L'instruction ENTRYPOINT

### Qu'est-ce que ENTRYPOINT ?

`ENTRYPOINT` définit l'**exécutable principal** du conteneur. Contrairement à `CMD`, il est plus **difficile à remplacer** et transforme le conteneur en une **commande exécutable**.

### Syntaxe

**Forme exec** (recommandée) :
```dockerfile
ENTRYPOINT ["executable", "param1"]
```

**Forme shell** :
```dockerfile
ENTRYPOINT commande param1
```

### Exemples

```dockerfile
# Le conteneur devient une commande curl
ENTRYPOINT ["curl"]

# Le conteneur devient une commande personnalisée
ENTRYPOINT ["python", "mon-script.py"]

# Point d'entrée avec chemin complet
ENTRYPOINT ["/usr/local/bin/mon-app"]
```

### Comportement : exécutable fixe

Avec `ENTRYPOINT`, les arguments passés à `docker run` sont ajoutés à la commande, ils ne la remplacent pas :

**Dockerfile** :
```dockerfile
FROM alpine:latest
ENTRYPOINT ["echo", "Message:"]
```

**Utilisation** :
```bash
# Sans argument
docker run mon-image
# Sortie : Message:

# Avec arguments - ils s'ajoutent
docker run mon-image "Hello World"
# Sortie : Message: Hello World

# Même avec plusieurs arguments
docker run mon-image "Hello" "from" "Docker"
# Sortie : Message: Hello from Docker
```

### Remplacer ENTRYPOINT (rare)

Pour remplacer l'ENTRYPOINT, utilisez l'option `--entrypoint` :

```bash
docker run --entrypoint /bin/sh mon-image
```

C'est rare et généralement réservé au débogage.

## La différence cruciale : CMD vs ENTRYPOINT

### Tableau comparatif

| Aspect | CMD | ENTRYPOINT |
|--------|-----|-----------|
| **But** | Commande par défaut | Exécutable principal |
| **Remplacement** | Facile (arguments docker run) | Difficile (--entrypoint requis) |
| **Arguments docker run** | Remplacent CMD | S'ajoutent à ENTRYPOINT |
| **Flexibilité** | Très flexible | Plus rigide |
| **Cas d'usage** | Applications configurables | Outils/commandes |

### Visualisation du comportement

**Avec CMD uniquement** :
```dockerfile
FROM alpine
CMD ["echo", "Hello"]
```
```bash
docker run image              → echo Hello
docker run image ls           → ls (remplace echo Hello)
```

**Avec ENTRYPOINT uniquement** :
```dockerfile
FROM alpine
ENTRYPOINT ["echo"]
```
```bash
docker run image              → echo
docker run image Hello        → echo Hello (ajoute l'argument)
```

**Avec les DEUX (combinaison)** :
```dockerfile
FROM alpine
ENTRYPOINT ["echo"]
CMD ["World"]
```
```bash
docker run image              → echo World
docker run image Hello        → echo Hello (remplace CMD)
```

### Exemples concrets

#### Exemple 1 : Application web flexible (CMD seul)

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .
CMD ["npm", "start"]
```

**Utilisation** :
```bash
# Mode production (par défaut)
docker run mon-app
# Exécute : npm start

# Mode développement (remplace CMD)
docker run mon-app npm run dev
# Exécute : npm run dev

# Tests (remplace CMD)
docker run mon-app npm test
# Exécute : npm test
```

✅ **Bon choix** : L'application a plusieurs modes d'exécution.

#### Exemple 2 : Outil CLI (ENTRYPOINT seul)

```dockerfile
FROM alpine:latest
RUN apk add --no-cache curl
ENTRYPOINT ["curl"]
```

**Utilisation** :
```bash
# Le conteneur devient la commande curl
docker run mon-curl https://api.github.com
# Exécute : curl https://api.github.com

docker run mon-curl -I https://google.com
# Exécute : curl -I https://google.com
```

✅ **Bon choix** : Le conteneur est un outil avec une seule fonction.

#### Exemple 3 : Combinaison CMD + ENTRYPOINT

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY mon-script.py .
ENTRYPOINT ["python", "mon-script.py"]
CMD ["--help"]
```

**Utilisation** :
```bash
# Sans argument - affiche l'aide (CMD par défaut)
docker run mon-script
# Exécute : python mon-script.py --help

# Avec argument - remplace CMD
docker run mon-script --verbose process data.csv
# Exécute : python mon-script.py --verbose process data.csv
```

✅ **Bon choix** : Script avec options par défaut mais personnalisables.

## Quand utiliser quoi ?

### Utilisez CMD quand :

✅ Votre conteneur est une **application flexible** avec plusieurs modes
✅ Vous voulez que les utilisateurs puissent facilement changer le comportement
✅ Vous conteneurisez une application classique (web app, API, etc.)

**Exemples** :
```dockerfile
# Application web
CMD ["npm", "start"]

# API
CMD ["python", "api.py"]

# Base de données
CMD ["mysqld"]
```

### Utilisez ENTRYPOINT quand :

✅ Votre conteneur est un **outil exécutable** ou une commande
✅ Vous voulez que le conteneur se comporte comme une commande système
✅ Vous voulez forcer l'exécutable principal

**Exemples** :
```dockerfile
# Outil CLI
ENTRYPOINT ["aws"]

# Script d'automatisation
ENTRYPOINT ["./backup.sh"]

# Commande personnalisée
ENTRYPOINT ["/usr/local/bin/mon-outil"]
```

### Utilisez les DEUX quand :

✅ Vous voulez un **exécutable fixe** avec des **paramètres par défaut**
✅ Vous créez un outil avec options configurables

**Exemples** :
```dockerfile
# Script avec options par défaut
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "8000"]

# Commande avec mode par défaut
ENTRYPOINT ["./mon-app"]
CMD ["start"]
```

## Forme exec vs forme shell

### Différences importantes

**Forme exec** (avec crochets) :
```dockerfile
CMD ["python", "app.py"]
ENTRYPOINT ["python", "app.py"]
```
- N'utilise pas de shell
- Les variables d'environnement ne sont pas interprétées
- Plus direct et plus performant
- Recommandé

**Forme shell** (sans crochets) :
```dockerfile
CMD python app.py
ENTRYPOINT python app.py
```
- Utilise `/bin/sh -c`
- Les variables d'environnement sont interprétées
- Permet des commandes complexes
- Processus principal a le PID 1 = `/bin/sh` (pas votre app)

### Exemple des différences

**Avec variables d'environnement** :

```dockerfile
FROM alpine
ENV NAME=World

# Forme shell - interprète $NAME
CMD echo "Hello $NAME"
# Sortie : Hello World

# Forme exec - n'interprète pas $NAME
CMD ["echo", "Hello $NAME"]
# Sortie : Hello $NAME (littéral)

# Forme exec avec interprétation
CMD ["/bin/sh", "-c", "echo Hello $NAME"]
# Sortie : Hello World
```

### Recommandation

✅ **Préférez la forme exec** sauf si vous avez besoin de fonctionnalités shell (variables, pipes, etc.)

```dockerfile
# ✅ Recommandé
CMD ["python", "app.py"]

# ⚠️ Acceptable si vous avez besoin du shell
CMD python app.py --config=$CONFIG_FILE
```

## Exemples complets et cas d'usage

### Application web Node.js

```dockerfile
FROM node:18-alpine
WORKDIR /app

# Copier les dépendances
COPY package*.json ./
RUN npm ci --only=production

# Copier le code
COPY . .

# Exposer le port
EXPOSE 3000

# CMD pour flexibilité
CMD ["node", "server.js"]
```

**Utilisation** :
```bash
# Production
docker run -p 3000:3000 mon-app

# Développement
docker run -p 3000:3000 mon-app npm run dev

# Tests
docker run mon-app npm test
```

### Outil CLI personnalisé

```dockerfile
FROM python:3.11-slim
WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY mon-outil.py .

# ENTRYPOINT pour comportement de commande
ENTRYPOINT ["python", "mon-outil.py"]

# CMD pour paramètres par défaut
CMD ["--help"]
```

**Utilisation** :
```bash
# Afficher l'aide
docker run mon-outil

# Utiliser l'outil
docker run mon-outil process --input data.csv --output result.json
```

### Application avec script d'initialisation

```dockerfile
FROM python:3.11-slim
WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

# Script d'entrée qui fait de l'initialisation
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

# ENTRYPOINT pour le script d'init
ENTRYPOINT ["/entrypoint.sh"]

# CMD pour la commande finale
CMD ["python", "app.py"]
```

**entrypoint.sh** :
```bash
#!/bin/bash
# Initialisation
echo "Démarrage de l'application..."
python manage.py migrate

# Exécuter la commande passée (CMD ou arguments)
exec "$@"
```

## Résumé et bonnes pratiques

### Récapitulatif des instructions

| Instruction | Rôle | Quand l'utiliser |
|-------------|------|------------------|
| **FROM** | Image de base | Toujours (obligatoire) |
| **RUN** | Exécuter pendant le build | Installation, configuration |
| **COPY** | Copier des fichiers | Presque toujours |
| **ADD** | Copier avec extras | Extraction tar, URL (rare) |
| **CMD** | Commande par défaut | Applications flexibles |
| **ENTRYPOINT** | Exécutable principal | Outils CLI |

### Bonnes pratiques essentielles

✅ **FROM** :
- Utilisez des images officielles
- Spécifiez toujours un tag (pas `latest` en production)
- Préférez les variantes `slim` ou `alpine`

✅ **RUN** :
- Chaînez les commandes avec `&&`
- Nettoyez dans la même couche
- Triez les packages alphabétiquement

✅ **COPY** :
- Préférez COPY à ADD (sauf besoin spécifique)
- Copiez les fichiers de dépendances avant le code
- Utilisez `.dockerignore`

✅ **CMD/ENTRYPOINT** :
- Utilisez la forme exec `["cmd", "arg"]`
- CMD pour les applications, ENTRYPOINT pour les outils
- Combinez-les pour plus de flexibilité

### Template de base

Voici un template de Dockerfile bien structuré :

```dockerfile
# Image de base
FROM python:3.11-slim

# Métadonnées (optionnel)
LABEL maintainer="vous@example.com"
LABEL version="1.0"

# Variables d'environnement
ENV APP_HOME=/app \
    PYTHONUNBUFFERED=1

# Répertoire de travail
WORKDIR $APP_HOME

# Dépendances système (si nécessaire)
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Dépendances de l'application
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Code de l'application
COPY . .

# Port (documentation)
EXPOSE 8000

# Utilisateur non-root (sécurité)
RUN useradd -m appuser
USER appuser

# Commande par défaut
CMD ["python", "app.py"]
```

## Conclusion

Dans cette section, vous avez maîtrisé les **cinq instructions fondamentales** :

- ✅ **FROM** : choisir votre image de base
- ✅ **RUN** : exécuter des commandes de configuration
- ✅ **COPY** : ajouter vos fichiers
- ✅ **ADD** : copier avec des fonctionnalités spéciales (rarement nécessaire)
- ✅ **CMD vs ENTRYPOINT** : définir le comportement au démarrage

Vous comprenez maintenant la **différence cruciale entre CMD et ENTRYPOINT** et savez quand utiliser l'un, l'autre, ou les deux ensemble.

Ces instructions constituent la fondation de 90% des Dockerfiles que vous créerez. Dans la section suivante, vous découvrirez les instructions avancées qui vous donneront encore plus de contrôle et de flexibilité.

⏭️ [Instructions avancées (ENV, WORKDIR, EXPOSE, USER, VOLUME)](/05-creation-dimages-personnalisees/03-instructions-avancees.md)
