🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 5.4 Construction d'images (build, tag)

## Introduction

Vous avez maintenant appris à écrire des Dockerfiles avec toutes les instructions nécessaires. Il est temps de découvrir comment **transformer** ce Dockerfile en une image Docker utilisable. C'est le moment où votre recette devient un plat réel !

Dans cette section, nous allons explorer :
- La commande `docker build` et toutes ses options
- Le système de **tags** pour versionner vos images
- Le **contexte de build** et son importance
- Le fichier `.dockerignore` pour optimiser vos builds
- Le **débogage** des erreurs de construction

## La commande docker build : créer une image

### Qu'est-ce que docker build ?

`docker build` est la commande qui **lit votre Dockerfile** et **construit une image** à partir de celui-ci. C'est le "compilateur" de vos images Docker.

💡 **Analogie** : Si le Dockerfile est une recette de cuisine, `docker build` est le chef qui suit cette recette étape par étape pour créer le plat final.

### Syntaxe de base

```bash
docker build [OPTIONS] PATH
```

- **OPTIONS** : paramètres de construction (tag, arguments, etc.)
- **PATH** : chemin vers le contexte de build (souvent `.` pour le répertoire actuel)

### Premier build simple

**Structure du projet** :
```
mon-projet/
├── Dockerfile
└── app.py
```

**Dockerfile** :
```dockerfile
FROM python:3.11-slim
COPY app.py .
CMD ["python", "app.py"]
```

**Construction** :
```bash
docker build .
```

Sortie typique :
```
[+] Building 5.2s (7/7) FINISHED
 => [internal] load build definition from Dockerfile
 => => transferring dockerfile: 98B
 => [internal] load .dockerignore
 => => transferring context: 2B
 => [internal] load metadata for docker.io/library/python:3.11-slim
 => [1/2] FROM docker.io/library/python:3.11-slim
 => [internal] load build context
 => => transferring context: 123B
 => [2/2] COPY app.py .
 => exporting to image
 => => exporting layers
 => => writing image sha256:a1b2c3d4e5f6...
```

✅ **Image créée avec succès** !

Mais... comment l'utiliser ? Elle n'a pas de nom ! D'où l'importance du tag.

## Les tags : nommer et versionner les images

### Qu'est-ce qu'un tag ?

Un **tag** est un **nom lisible** associé à une image Docker. Sans tag, votre image n'a qu'un ID (comme `a1b2c3d4e5f6`), difficile à retenir et à utiliser.

💡 **Analogie** : Un tag est comme le nom d'un produit sur une étagère. Au lieu de dire "le produit avec le code-barres 123456789", vous dites "le lait demi-écrémé".

### Format d'un tag complet

```
[registry/][namespace/]repository:version
```

Exemples :
```
nginx:latest                           # Simple
mon-app:1.0                           # Avec version
username/mon-app:v2.3                 # Avec namespace
registry.example.com/app:prod         # Avec registry privé
```

Décomposition :
- **registry** : où l'image est stockée (par défaut: docker.io)
- **namespace** : votre nom d'utilisateur ou organisation
- **repository** : nom de l'image
- **version** : tag de version (par défaut: latest)

### Construire avec un tag : option -t

```bash
docker build -t nom-image:tag .
```

**Exemples** :
```bash
# Tag simple
docker build -t mon-app .
# Équivaut à: mon-app:latest

# Tag avec version
docker build -t mon-app:1.0 .

# Tag avec namespace
docker build -t username/mon-app:1.0 .

# Tag complet avec registry
docker build -t registry.example.com/mon-app:prod .
```

### Plusieurs tags pour la même image

Vous pouvez associer **plusieurs tags** à la même image :

```bash
# Construire avec plusieurs tags
docker build -t mon-app:latest -t mon-app:1.0 -t mon-app:1.0.5 .
```

Ou taguer après construction :
```bash
# Construire avec un tag
docker build -t mon-app:1.0 .

# Ajouter d'autres tags
docker tag mon-app:1.0 mon-app:latest
docker tag mon-app:1.0 mon-app:stable
```

### Convention de nommage des tags

#### Versions sémantiques

```bash
# Version majeure.mineure.patch
docker build -t mon-app:1.2.3 .

# Avec préfixe 'v'
docker build -t mon-app:v1.2.3 .

# Version majeure.mineure
docker build -t mon-app:1.2 .

# Version majeure
docker build -t mon-app:1 .
```

#### Tags d'environnement

```bash
# Environnements
docker build -t mon-app:dev .
docker build -t mon-app:staging .
docker build -t mon-app:prod .

# Avec version
docker build -t mon-app:1.0-prod .
```

#### Tags descriptifs

```bash
# Variantes de build
docker build -t mon-app:alpine .
docker build -t mon-app:debian .

# Fonctionnalités
docker build -t mon-app:with-redis .
docker build -t mon-app:gpu .

# Date
docker build -t mon-app:2025-01-15 .
```

### Le tag "latest" : utilisation et pièges

**latest** est le tag **par défaut** si aucun tag n'est spécifié :

```bash
docker build -t mon-app .
# Équivaut à
docker build -t mon-app:latest .
```

⚠️ **Attention** : `latest` ne signifie PAS "la dernière version" ! C'est juste le tag par défaut.

**Problèmes avec latest** :
```bash
# Construction v1.0
docker build -t mon-app:latest .

# 2 mois plus tard, construction v2.0
docker build -t mon-app:latest .
# ❌ L'ancien latest est "écrasé"

# Quelqu'un fait docker pull mon-app:latest
# Quelle version obtient-il ? Impossible à savoir !
```

**Recommandations** :

```bash
# ✅ Bon - version explicite en production
docker build -t mon-app:1.2.3 .

# ✅ Bon - latest + version
docker build -t mon-app:1.2.3 -t mon-app:latest .

# ⚠️ Développement uniquement
docker build -t mon-app:latest .

# ❌ En production
# Ne faites jamais docker run mon-app:latest en prod
```

### Stratégie de tagging recommandée

Pour chaque build, créez **plusieurs tags** :

```bash
# Version 1.2.3
docker build \
  -t mon-app:1.2.3 \      # Version complète (immuable)
  -t mon-app:1.2 \         # Version mineure
  -t mon-app:1 \           # Version majeure
  -t mon-app:latest \      # Dernière version
  .
```

**Avantages** :
- `1.2.3` : version exacte, ne change jamais
- `1.2` : reçoit les patches de sécurité
- `1` : reçoit les mises à jour mineures
- `latest` : toujours la dernière version

## Les options importantes de docker build

### -t, --tag : nommer l'image

```bash
docker build -t mon-app:1.0 .
```

Peut être utilisé plusieurs fois :
```bash
docker build -t mon-app:1.0 -t mon-app:latest .
```

### -f, --file : spécifier un Dockerfile différent

Par défaut, Docker cherche `Dockerfile`. Pour utiliser un autre nom :

```bash
# Utiliser Dockerfile.dev
docker build -f Dockerfile.dev -t mon-app:dev .

# Dockerfile dans un autre répertoire
docker build -f docker/Dockerfile.prod -t mon-app:prod .
```

**Structure avec plusieurs Dockerfiles** :
```
mon-projet/
├── Dockerfile           # Production
├── Dockerfile.dev       # Développement
├── Dockerfile.test      # Tests
└── src/
```

### --no-cache : reconstruire sans cache

Force Docker à reconstruire toutes les couches :

```bash
docker build --no-cache -t mon-app:1.0 .
```

**Quand l'utiliser** :
- Après modification de ressources externes (dépendances)
- Pour forcer une mise à jour complète
- Pour déboguer des problèmes de cache

```bash
# Exemple : forcer la mise à jour des packages
docker build --no-cache -t mon-app:fresh .
```

### --build-arg : passer des variables de build

Passer des valeurs aux instructions `ARG` du Dockerfile :

**Dockerfile** :
```dockerfile
FROM python:3.11-slim

# Variable avec valeur par défaut
ARG APP_VERSION=1.0
ARG BUILD_DATE

RUN echo "Building version $APP_VERSION on $BUILD_DATE"

LABEL version=$APP_VERSION
```

**Build** :
```bash
# Utiliser les valeurs par défaut
docker build -t mon-app .

# Passer des valeurs personnalisées
docker build \
  --build-arg APP_VERSION=2.0 \
  --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
  -t mon-app:2.0 .
```

**Cas d'usage courants** :

```bash
# Environnement de build
docker build --build-arg ENV=production -t mon-app:prod .

# Version de dépendances
docker build --build-arg NODE_VERSION=18 -t mon-app .

# Proxy pour le build
docker build \
  --build-arg HTTP_PROXY=http://proxy.example.com:8080 \
  -t mon-app .
```

### --target : builds multi-étapes

Pour les Dockerfiles multi-étapes, construire jusqu'à une étape spécifique :

**Dockerfile** :
```dockerfile
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine AS production
COPY --from=builder /app/dist /usr/share/nginx/html
```

**Build** :
```bash
# Builder uniquement (pour tests)
docker build --target builder -t mon-app:builder .

# Production (par défaut, dernière étape)
docker build -t mon-app:prod .
```

### --pull : forcer le téléchargement de l'image de base

Force Docker à télécharger la dernière version de l'image de base :

```bash
docker build --pull -t mon-app:1.0 .
```

**Utile pour** :
- S'assurer d'avoir les dernières mises à jour de sécurité
- Forcer une reconstruction complète

### --quiet, -q : mode silencieux

Affiche uniquement l'ID de l'image créée :

```bash
docker build -q -t mon-app:1.0 .
# Sortie : sha256:a1b2c3d4e5f6...
```

**Utile dans les scripts** :
```bash
IMAGE_ID=$(docker build -q -t mon-app .)
echo "Image créée : $IMAGE_ID"
```

### --progress : contrôler l'affichage

```bash
# Affichage auto (par défaut)
docker build -t mon-app .

# Affichage simple (une ligne par étape)
docker build --progress=plain -t mon-app .

# Désactiver l'affichage
docker build --progress=false -t mon-app .
```

### Exemples de commandes complètes

**Build de développement** :
```bash
docker build \
  -f Dockerfile.dev \
  -t mon-app:dev \
  --build-arg ENV=development \
  .
```

**Build de production** :
```bash
docker build \
  -f Dockerfile.prod \
  -t mon-app:1.2.3 \
  -t mon-app:latest \
  --build-arg ENV=production \
  --build-arg VERSION=1.2.3 \
  --pull \
  .
```

**Build optimisé** :
```bash
docker build \
  -t registry.example.com/mon-app:$(git rev-parse --short HEAD) \
  --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
  --build-arg GIT_COMMIT=$(git rev-parse HEAD) \
  --no-cache \
  .
```

## Le contexte de build : comprendre et optimiser

### Qu'est-ce que le contexte de build ?

Le **contexte de build** est l'ensemble des fichiers et répertoires que Docker peut utiliser pendant la construction. C'est le chemin spécifié à la fin de `docker build`.

```bash
docker build -t mon-app .
                        ↑
                   contexte = répertoire actuel
```

### Comment ça fonctionne

1. Docker **copie** tout le contexte vers le daemon Docker
2. Seuls les fichiers du contexte peuvent être utilisés avec `COPY` et `ADD`
3. Plus le contexte est gros, plus le build est lent

**Visualisation** :
```
Votre machine                Docker daemon
─────────────                ─────────────
mon-projet/          →       /tmp/docker-build-123/
├── Dockerfile       →       ├── Dockerfile
├── src/             →       ├── src/
├── package.json     →       ├── package.json
└── node_modules/    →       ├── node_modules/  ❌ Inutile !
```

### Optimiser le contexte

**Problème** : Contexte trop large
```bash
# ❌ Mauvais - envoie tout, y compris .git, node_modules, etc.
cd mon-projet/
docker build -t mon-app .
# Sending build context to Docker daemon  2.5GB  ← Trop !
```

**Solution 1** : Utiliser `.dockerignore`
```bash
# Créer .dockerignore (voir section suivante)
docker build -t mon-app .
# Sending build context to Docker daemon  15MB  ← Mieux !
```

**Solution 2** : Contexte spécifique
```bash
# Utiliser un sous-répertoire comme contexte
docker build -f docker/Dockerfile -t mon-app ./src
```

### Le fichier .dockerignore : filtrer le contexte

### Qu'est-ce que .dockerignore ?

`.dockerignore` fonctionne comme `.gitignore` : il liste les fichiers et répertoires à **exclure** du contexte de build.

💡 **Analogie** : C'est comme une liste de "ne pas déranger" pour Docker. Vous dites "n'envoie pas ces fichiers au daemon Docker".

### Créer un .dockerignore

**Fichier `.dockerignore`** à la racine du projet :
```
# Contrôle de version
.git
.gitignore
.github

# Dépendances
node_modules/
__pycache__/
*.pyc
venv/
env/

# Build artifacts
dist/
build/
*.egg-info

# IDE
.vscode/
.idea/
*.swp
*.swo

# Logs
*.log
logs/

# OS
.DS_Store
Thumbs.db

# Documentation
README.md
docs/

# Tests
tests/
*.test.js
coverage/

# Environnement
.env
.env.local
*.pem
```

### Syntaxe de .dockerignore

**Patterns de base** :
```
# Commentaire
node_modules/          # Répertoire exact
*.log                  # Tous les .log
temp*                  # Commence par "temp"
**/*.tmp               # Tous les .tmp dans tous les sous-répertoires
```

**Exceptions** :
```
# Ignorer tous les markdown
*.md

# SAUF README.md
!README.md
```

**Répertoires spécifiques** :
```
# Ignorer tests/ à la racine uniquement
/tests/

# Ignorer tous les répertoires tests/
tests/
```

### Exemples par type d'application

**Application Node.js** :
```
node_modules/
npm-debug.log
.npmrc
.env
.git
README.md
Dockerfile*
docker-compose*.yml
.dockerignore
```

**Application Python** :
```
__pycache__/
*.pyc
*.pyo
*.pyd
.Python
venv/
env/
.env
.git
*.log
.pytest_cache/
.coverage
htmlcov/
```

**Application Java/Maven** :
```
target/
.mvn/
.gradle/
build/
.classpath
.project
.settings/
.git
*.log
```

### Impact sur les performances

**Sans .dockerignore** :
```bash
docker build -t mon-app .
# Sending build context to Docker daemon  2.5GB
# Step 1/5 : FROM node:18-alpine
# [Build takes 2 minutes]
```

**Avec .dockerignore** :
```bash
docker build -t mon-app .
# Sending build context to Docker daemon  15MB
# Step 1/5 : FROM node:18-alpine
# [Build takes 10 seconds]
```

✅ **150x plus petit, 12x plus rapide !**

### Template .dockerignore universel

```
# Contrôle de version
.git/
.gitignore
.gitattributes
.github/
.gitlab-ci.yml

# CI/CD
.travis.yml
.circleci/
Jenkinsfile

# Docker
Dockerfile*
docker-compose*.yml
.dockerignore

# Documentation
README*.md
LICENSE
docs/
*.md

# Dépendances (à adapter selon le langage)
node_modules/
__pycache__/
venv/
vendor/

# Build & test
dist/
build/
coverage/
.pytest_cache/
.tox/

# IDE
.vscode/
.idea/
*.swp
*.sublime-*

# OS
.DS_Store
Thumbs.db
desktop.ini

# Logs et données temporaires
*.log
logs/
tmp/
temp/

# Secrets (NE JAMAIS commiter)
.env
.env.*
*.pem
*.key
secrets/
```

## Déboguer les erreurs de build

### Comprendre les messages d'erreur

Docker affiche des messages d'erreur clairs qui indiquent :
- **Quelle étape** a échoué
- **Pourquoi** elle a échoué
- **Le contexte** de l'erreur

**Exemple d'erreur** :
```
Step 5/8 : RUN npm install
 ---> Running in abc123def456
npm ERR! code ENOENT
npm ERR! syscall open
npm ERR! path /app/package.json
npm ERR! errno -2
npm ERR! enoent ENOENT: no such file or directory, open '/app/package.json'

The command '/bin/sh -c npm install' returned a non-zero code: 254
```

### Erreurs courantes et solutions

#### 1. Fichier introuvable lors du COPY

**Erreur** :
```
Step 3/6 : COPY app.py /app/
COPY failed: file not found in build context
```

**Causes** :
- Le fichier n'existe pas
- Le fichier est en dehors du contexte de build
- Le fichier est dans `.dockerignore`

**Solutions** :
```bash
# Vérifier que le fichier existe
ls app.py

# Vérifier le contexte de build
docker build -t mon-app .  # Contexte = répertoire actuel

# Vérifier .dockerignore
cat .dockerignore
```

#### 2. Commande introuvable dans RUN

**Erreur** :
```
Step 4/8 : RUN npm install
/bin/sh: npm: not found
```

**Cause** : L'outil n'est pas installé dans l'image de base.

**Solution** :
```dockerfile
# ❌ Mauvais
FROM alpine:latest
RUN npm install

# ✅ Bon - utiliser une image avec Node.js
FROM node:18-alpine
RUN npm install
```

#### 3. Permissions refusées

**Erreur** :
```
Step 5/8 : RUN mkdir /app/data
mkdir: cannot create directory '/app/data': Permission denied
```

**Cause** : Utilisateur sans permissions suffisantes.

**Solution** :
```dockerfile
# Opérations nécessitant root AVANT USER
RUN mkdir -p /app/data && chown appuser:appuser /app/data

# Puis basculer vers USER
USER appuser
```

#### 4. Dépendances manquantes

**Erreur** :
```
Step 6/10 : RUN pip install -r requirements.txt
ERROR: Could not find a version that satisfies the requirement package==x.y.z
```

**Solutions** :
```dockerfile
# Mettre à jour pip
RUN pip install --upgrade pip

# Installer les dépendances système nécessaires
RUN apt-get update && apt-get install -y build-essential

# Vérifier requirements.txt
COPY requirements.txt .
RUN cat requirements.txt  # Debug
RUN pip install -r requirements.txt
```

#### 5. Out of memory (OOM)

**Erreur** :
```
The command '/bin/sh -c npm install' returned a non-zero code: 137
```

Code 137 = tué par le système (manque de mémoire).

**Solutions** :
```bash
# Augmenter la mémoire de Docker Desktop (GUI)
# Ou au niveau système

# Optimiser les commandes
RUN npm ci --only=production  # Au lieu de npm install
```

### Techniques de débogage

#### 1. Construire étape par étape

Commentez les instructions problématiques :

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
# COPY . .               ← Commenter temporairement
# CMD ["python", "app.py"]  ← Commenter temporairement
```

Construisez et testez :
```bash
docker build -t debug-image .
docker run -it debug-image bash
# Testez manuellement les commandes
```

#### 2. Utiliser --progress=plain

Affiche plus de détails :
```bash
docker build --progress=plain -t mon-app . 2>&1 | tee build.log
```

#### 3. Inspecter les couches intermédiaires

Docker garde les couches même en cas d'échec :

```bash
# Trouver l'ID de la dernière couche réussie
docker images -a

# Démarrer un conteneur depuis cette couche
docker run -it <image-id> bash

# Explorer et tester
```

#### 4. Ajouter des RUN echo pour déboguer

```dockerfile
FROM python:3.11-slim
WORKDIR /app

RUN echo "=== Current directory ===" && pwd
RUN echo "=== Listing files ===" && ls -la

COPY requirements.txt .

RUN echo "=== requirements.txt content ===" && cat requirements.txt
RUN pip install -r requirements.txt
```

#### 5. Utiliser shellcheck pour les scripts

Si vous avez des scripts complexes :
```bash
# Installer shellcheck
apt-get install shellcheck

# Vérifier votre script
shellcheck mon-script.sh
```

### Valider le Dockerfile

#### Hadolint : linter pour Dockerfiles

**Installation** :
```bash
# macOS
brew install hadolint

# Linux
wget -O hadolint https://github.com/hadolint/hadolint/releases/download/v2.12.0/hadolint-Linux-x86_64
chmod +x hadolint
```

**Utilisation** :
```bash
hadolint Dockerfile
```

**Exemple de sortie** :
```
Dockerfile:5 DL3008 warning: Pin versions in apt get install
Dockerfile:10 DL3059 info: Multiple consecutive RUN instructions
Dockerfile:15 DL3025 warning: Use arguments JSON notation for CMD
```

## Workflow de build complet

### Développement local

```bash
#!/bin/bash
# build-dev.sh

# Build avec cache pour rapidité
docker build \
  -f Dockerfile.dev \
  -t mon-app:dev \
  --build-arg ENV=development \
  .

# Ou avec docker-compose
docker-compose build
```

### CI/CD automatisé

```bash
#!/bin/bash
# build-ci.sh

# Variables
VERSION=$(git describe --tags --always)
BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ')
GIT_COMMIT=$(git rev-parse HEAD)

# Build sans cache pour garantir la fraîcheur
docker build \
  -t mon-app:${VERSION} \
  -t mon-app:latest \
  --build-arg VERSION=${VERSION} \
  --build-arg BUILD_DATE=${BUILD_DATE} \
  --build-arg GIT_COMMIT=${GIT_COMMIT} \
  --no-cache \
  --pull \
  .

# Tests de l'image
docker run --rm mon-app:${VERSION} npm test

# Push vers le registry
docker push mon-app:${VERSION}
docker push mon-app:latest
```

### Production

```bash
#!/bin/bash
# build-prod.sh

# Validation préalable
hadolint Dockerfile

# Build multi-architecture (si nécessaire)
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t registry.example.com/mon-app:1.0.0 \
  --push \
  .
```

## Bonnes pratiques de build

### 1. Toujours utiliser .dockerignore

```bash
# ✅ Obligatoire
echo "node_modules/" > .dockerignore
echo ".git/" >> .dockerignore
```

### 2. Tagger de manière cohérente

```bash
# ✅ Bon - stratégie claire
docker build \
  -t mon-app:1.2.3 \
  -t mon-app:1.2 \
  -t mon-app:1 \
  -t mon-app:latest \
  .
```

### 3. Utiliser des ARG pour la flexibilité

```dockerfile
ARG NODE_VERSION=18
FROM node:${NODE_VERSION}-alpine
```

### 4. Valider avant de construire

```bash
# Linter
hadolint Dockerfile

# Vérifier la syntaxe
docker build --check .
```

### 5. Documenter vos builds

```bash
# Ajouter des métadonnées
docker build \
  --label "version=1.0.0" \
  --label "maintainer=team@example.com" \
  --label "description=Application web production" \
  -t mon-app:1.0.0 \
  .
```

### 6. Automatiser avec Makefile

```makefile
# Makefile
VERSION := $(shell git describe --tags --always)

.PHONY: build
build:
	docker build -t mon-app:$(VERSION) .

.PHONY: build-no-cache
build-no-cache:
	docker build --no-cache -t mon-app:$(VERSION) .

.PHONY: build-dev
build-dev:
	docker build -f Dockerfile.dev -t mon-app:dev .

.PHONY: build-prod
build-prod:
	docker build \
		-f Dockerfile.prod \
		-t mon-app:$(VERSION) \
		-t mon-app:latest \
		--build-arg VERSION=$(VERSION) \
		.

.PHONY: test
test: build
	docker run --rm mon-app:$(VERSION) npm test
```

**Utilisation** :
```bash
make build              # Build standard
make build-no-cache     # Build sans cache
make build-dev          # Build développement
make build-prod         # Build production
make test               # Build + tests
```

## Commandes utiles après le build

### Lister les images

```bash
# Toutes les images
docker images

# Filtrer par nom
docker images mon-app

# Voir la taille
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

### Inspecter une image

```bash
# Détails complets
docker inspect mon-app:1.0

# Historique des couches
docker history mon-app:1.0

# Variables d'environnement
docker inspect mon-app:1.0 --format='{{.Config.Env}}'
```

### Exporter/Importer une image

```bash
# Sauvegarder
docker save mon-app:1.0 -o mon-app-1.0.tar

# Charger
docker load -i mon-app-1.0.tar
```

### Nettoyer après builds

```bash
# Supprimer les images <none> (dangling)
docker image prune

# Supprimer toutes les images inutilisées
docker image prune -a

# Forcer (sans confirmation)
docker image prune -af
```

## Résumé

Dans cette section, vous avez maîtrisé la **construction d'images Docker** :

- ✅ **docker build** : transformer un Dockerfile en image
- ✅ **Tags** : nommer et versionner vos images efficacement
- ✅ **Options de build** : -t, -f, --no-cache, --build-arg, --target, etc.
- ✅ **Contexte de build** : optimiser ce qui est envoyé au daemon
- ✅ **.dockerignore** : exclure les fichiers inutiles (essentiel !)
- ✅ **Débogage** : résoudre les erreurs courantes de build
- ✅ **Bonnes pratiques** : workflow professionnel de construction

**Points clés à retenir** :

🔑 **Toujours utiliser .dockerignore** : réduit drastiquement le temps de build
🔑 **Stratégie de tagging claire** : versions sémantiques + latest
🔑 **Build-args pour flexibilité** : configuration sans modifier le Dockerfile
🔑 **Valider avant de construire** : hadolint, shellcheck
🔑 **Automatiser** : scripts ou Makefile pour reproductibilité

Vous savez maintenant construire, tagger et déboguer vos images Docker comme un professionnel ! Dans la section suivante, vous découvrirez comment optimiser vos Dockerfiles pour créer des images plus légères et plus rapides.

⏭️ [Optimisation des Dockerfiles et layers](/05-creation-dimages-personnalisees/05-optimisation-des-dockerfiles-et-layers.md)
