🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 5.5 Optimisation des Dockerfiles et layers

## Introduction

Vous savez maintenant créer des images Docker fonctionnelles. Mais une image fonctionnelle n'est pas nécessairement une image **optimale**. Dans cette section finale du chapitre 5, nous allons apprendre à créer des images :

- **Plus légères** : qui occupent moins d'espace disque
- **Plus rapides** : qui se construisent et se déploient rapidement
- **Plus sécurisées** : avec moins de composants inutiles
- **Plus maintenables** : faciles à comprendre et à modifier

💡 **Pourquoi optimiser ?** Une image de 1 GB qui prend 10 minutes à construire coûte cher en :
- Temps de développement (attente des builds)
- Bande passante (transfert vers les serveurs)
- Stockage (dans les registres)
- Argent (coûts d'infrastructure)

## Comprendre le système de couches et le cache

### Le principe des couches (layers)

Rappel : chaque instruction Docker crée une **nouvelle couche** dans l'image finale.

```dockerfile
FROM ubuntu:22.04          # Couche 1
RUN apt-get update         # Couche 2
RUN apt-get install -y curl # Couche 3
COPY app.py /app/          # Couche 4
CMD ["python", "/app/app.py"] # Couche 5
```

**Visualisation** :
```
┌─────────────────────────────┐
│ CMD ["python", "app.py"]    │ ← 5. Métadonnées (0 KB)
├─────────────────────────────┤
│ COPY app.py                 │ ← 4. Fichier app (15 KB)
├─────────────────────────────┤
│ RUN apt-get install curl    │ ← 3. Packages (50 MB)
├─────────────────────────────┤
│ RUN apt-get update          │ ← 2. Listes de packages (30 MB)
├─────────────────────────────┤
│ FROM ubuntu:22.04           │ ← 1. Image de base (80 MB)
└─────────────────────────────┘
Total : ~180 MB
```

### Le système de cache Docker

Docker met en **cache chaque couche**. Si une instruction n'a pas changé, Docker réutilise la couche en cache.

**Invalidation du cache** : Si une couche est reconstruite, **toutes les couches suivantes** sont aussi reconstruites, même si elles n'ont pas changé.

**Exemple** :
```dockerfile
FROM python:3.11-slim
COPY requirements.txt .     # Couche A (change rarement)
RUN pip install -r requirements.txt  # Couche B (change rarement)
COPY . .                    # Couche C (change souvent)
CMD ["python", "app.py"]    # Couche D
```

**Scénario 1** : Modification du code (pas de requirements.txt)
```
Couche A : ✅ Cache utilisé
Couche B : ✅ Cache utilisé (pip install n'est pas refait !)
Couche C : ⚠️ Reconstruite (fichiers changés)
Couche D : ⚠️ Reconstruite (couche précédente changée)
```

**Scénario 2** : Mauvais ordre des instructions
```dockerfile
FROM python:3.11-slim
COPY . .                    # ❌ Copie tout en premier
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

À chaque modification du code :
```
Couche 1 : ⚠️ Reconstruite (code changé)
Couche 2 : ⚠️ Reconstruite (pip install refait à chaque fois !)
Couche 3 : ⚠️ Reconstruite
```

💰 **Temps perdu** : Si `pip install` prend 2 minutes, vous perdez 2 minutes à chaque build !

## Optimisation 1 : Ordre des instructions

### Règle d'or : du moins changeant au plus changeant

Placez les instructions par ordre de **fréquence de modification** :

```dockerfile
# 1. Image de base (change presque jamais)
FROM python:3.11-slim

# 2. Dépendances système (changent rarement)
RUN apt-get update && apt-get install -y curl

# 3. Fichiers de dépendances (changent occasionnellement)
COPY requirements.txt .

# 4. Installation des dépendances (dépend de #3)
RUN pip install -r requirements.txt

# 5. Code source (change fréquemment)
COPY . .

# 6. Commande de démarrage (change rarement)
CMD ["python", "app.py"]
```

### Exemple avant/après : Application Node.js

**❌ AVANT (non optimisé)** :
```dockerfile
FROM node:18-alpine
WORKDIR /app

# Copie TOUT le code dès le début
COPY . .

# Installation (refaite à chaque changement de code)
RUN npm install

EXPOSE 3000
CMD ["npm", "start"]
```

**Problème** : Chaque modification du code (même un commentaire !) invalide le cache et force `npm install` à se réexécuter.

**✅ APRÈS (optimisé)** :
```dockerfile
FROM node:18-alpine
WORKDIR /app

# 1. Copier uniquement les fichiers de dépendances
COPY package*.json ./

# 2. Installer (mis en cache tant que package.json ne change pas)
RUN npm ci --only=production

# 3. Copier le code (en dernier)
COPY . .

EXPOSE 3000
CMD ["npm", "start"]
```

**Gain** : `npm ci` n'est réexécuté que quand `package.json` change (rare).

### Exemple : Application Python

**❌ AVANT** :
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

**✅ APRÈS** :
```dockerfile
FROM python:3.11-slim
WORKDIR /app

# Dépendances d'abord
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Code ensuite
COPY . .
CMD ["python", "app.py"]
```

### Séparer les dépendances par fréquence de changement

Si vous avez des dépendances qui changent à des rythmes différents :

```dockerfile
FROM python:3.11-slim
WORKDIR /app

# Dépendances de base (changent rarement)
COPY requirements-base.txt .
RUN pip install -r requirements-base.txt

# Dépendances de développement (changent plus souvent)
COPY requirements-dev.txt .
RUN pip install -r requirements-dev.txt

# Code (change tout le temps)
COPY . .
```

## Optimisation 2 : Réduire le nombre de couches

### Principe : fusionner les commandes

Chaque instruction `RUN`, `COPY`, ou `ADD` crée une couche. Moins de couches = image plus légère.

**❌ AVANT (beaucoup de couches)** :
```dockerfile
FROM ubuntu:22.04
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y git
RUN apt-get install -y vim
RUN apt-get clean
```

5 couches RUN = image plus grosse !

**✅ APRÈS (une seule couche)** :
```dockerfile
FROM ubuntu:22.04
RUN apt-get update && \
    apt-get install -y \
        curl \
        git \
        vim && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

1 seule couche = image plus légère !

### Pourquoi ça marche ?

Chaque couche contient les **changements** par rapport à la couche précédente. Si vous créez un fichier puis le supprimez dans deux couches séparées :

**❌ Non optimisé** :
```dockerfile
RUN dd if=/dev/zero of=/tmp/big-file bs=1M count=100  # Crée 100 MB
RUN rm /tmp/big-file                                   # Supprime le fichier
```

Résultat : L'image contient encore 100 MB (dans la première couche) !

**✅ Optimisé** :
```dockerfile
RUN dd if=/dev/zero of=/tmp/big-file bs=1M count=100 && \
    rm /tmp/big-file
```

Résultat : 0 MB ajouté (création et suppression dans la même couche).

### Nettoyage dans la même couche

**❌ AVANT** :
```dockerfile
FROM ubuntu:22.04
RUN apt-get update
RUN apt-get install -y build-essential
RUN apt-get clean
```

Les caches apt sont dans la couche 2, pas supprimés vraiment !

**✅ APRÈS** :
```dockerfile
FROM ubuntu:22.04
RUN apt-get update && \
    apt-get install -y build-essential && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

Nettoyage efficace !

### Exception : lisibilité vs optimisation

Parfois, séparer aide à la compréhension :

```dockerfile
# Acceptable pour la clarté
RUN apt-get update && apt-get install -y curl
RUN useradd -m appuser
RUN mkdir -p /app/data && chown appuser:appuser /app/data
```

vs

```dockerfile
# Plus optimisé mais moins lisible
RUN apt-get update && apt-get install -y curl && \
    useradd -m appuser && \
    mkdir -p /app/data && chown appuser:appuser /app/data
```

**Compromis** : Groupez logiquement, mais gardez lisible.

## Optimisation 3 : Choisir la bonne image de base

### Impact de l'image de base sur la taille

**Comparaison Python** :
```dockerfile
# Standard - 920 MB
FROM python:3.11
# → Image complète avec tous les outils

# Slim - 130 MB
FROM python:3.11-slim
# → Image allégée sans packages inutiles

# Alpine - 50 MB
FROM python:3.11-alpine
# → Image ultra-légère basée sur Alpine Linux
```

**Économie** : 870 MB entre standard et alpine !

### Variantes d'images courantes

| Image de base | Taille typique | Avantages | Inconvénients |
|--------------|----------------|-----------|---------------|
| `xxx:standard` | 500-1000 MB | Tout inclus, compatible | Très grosse |
| `xxx:slim` | 100-200 MB | Bonne compatibilité, raisonnable | Manque certains outils |
| `xxx:alpine` | 20-100 MB | Ultra-légère | Peut nécessiter des adaptations |
| `scratch` | 0 MB | Vide totale | Seulement pour binaires statiques |

### Choisir selon le cas d'usage

**Développement** : Image standard
```dockerfile
# Tous les outils de debug disponibles
FROM node:18
```

**Production** : Image slim ou alpine
```dockerfile
# Optimisé pour la production
FROM node:18-alpine
```

**Microservice simple** : Alpine
```dockerfile
FROM python:3.11-alpine
```

**Binaire Go** : Scratch
```dockerfile
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN go build -o app

FROM scratch
COPY --from=builder /app/app /app
CMD ["/app"]
```

### Adapter le Dockerfile pour Alpine

Alpine utilise `apk` au lieu de `apt-get` :

**Debian/Ubuntu** :
```dockerfile
FROM python:3.11-slim
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*
```

**Alpine** :
```dockerfile
FROM python:3.11-alpine
RUN apk add --no-cache gcc musl-dev
```

**Note** : `--no-cache` évite de stocker le cache des packages.

## Optimisation 4 : Techniques de nettoyage

### Nettoyer les caches de packages

**Debian/Ubuntu** :
```dockerfile
RUN apt-get update && \
    apt-get install -y curl git && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

**Alpine** :
```dockerfile
RUN apk add --no-cache curl git
```

Alpine nettoie automatiquement avec `--no-cache` !

**CentOS/RedHat** :
```dockerfile
RUN yum install -y curl git && \
    yum clean all && \
    rm -rf /var/cache/yum
```

### Nettoyer les caches de langages

**Python** :
```dockerfile
# Sans cache pip
RUN pip install --no-cache-dir -r requirements.txt

# Supprimer les fichiers .pyc
ENV PYTHONDONTWRITEBYTECODE=1

# Nettoyer après installation
RUN pip install -r requirements.txt && \
    find /usr/local -type d -name __pycache__ -exec rm -rf {} + && \
    find /usr/local -type f -name '*.pyc' -delete
```

**Node.js** :
```dockerfile
# Utiliser npm ci au lieu de npm install
RUN npm ci --only=production

# Supprimer les caches npm
RUN npm cache clean --force

# Supprimer les fichiers inutiles
RUN rm -rf /root/.npm /tmp/*
```

**Composer (PHP)** :
```dockerfile
RUN composer install --no-dev --optimize-autoloader && \
    composer clear-cache
```

### Supprimer les outils de build temporaires

**Pattern courant** : Installer, compiler, nettoyer
```dockerfile
FROM python:3.11-slim

# Installer build tools, build, puis nettoyer
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        gcc \
        build-essential && \
    pip install -r requirements.txt && \
    apt-get purge -y --auto-remove \
        gcc \
        build-essential && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### Supprimer les fichiers temporaires

```dockerfile
RUN command-qui-cree-temp && \
    rm -rf /tmp/* /var/tmp/* && \
    rm -rf /root/.cache
```

## Optimisation 5 : Multi-stage builds

### Principe des builds multi-étapes

Séparer la **construction** de l'**exécution** en utilisant plusieurs images :

```dockerfile
# Étape 1 : Build (image grosse avec tous les outils)
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Étape 2 : Production (image légère)
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
```

**Résultat** : L'image finale ne contient que les fichiers nécessaires, pas les outils de build !

### Exemple complet : Application React

**❌ Sans multi-stage (image de 1.2 GB)** :
```dockerfile
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
# L'image contient Node.js + node_modules + sources + build
EXPOSE 80
CMD ["npx", "serve", "-s", "build"]
```

**✅ Avec multi-stage (image de 25 MB)** :
```dockerfile
# Stage 1: Build
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production
FROM nginx:alpine
COPY --from=builder /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**Gain** : ~1175 MB (98% de réduction) !

### Exemple : Application Go

**Multi-stage avec scratch** :
```dockerfile
# Build
FROM golang:1.21 AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

# Production (image de ~10 MB)
FROM scratch
COPY --from=builder /app/app /app
EXPOSE 8080
CMD ["/app"]
```

### Exemple : Application Python

```dockerfile
# Build avec dépendances de compilation
FROM python:3.11 AS builder
WORKDIR /app
RUN apt-get update && apt-get install -y gcc
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Production légère
FROM python:3.11-slim
WORKDIR /app
# Copier seulement les packages installés
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
CMD ["python", "app.py"]
```

### Builds multi-stages avec plusieurs artefacts

```dockerfile
# Stage 1: Build frontend
FROM node:18 AS frontend-builder
WORKDIR /frontend
COPY frontend/package*.json ./
RUN npm ci
COPY frontend/ .
RUN npm run build

# Stage 2: Build backend
FROM golang:1.21 AS backend-builder
WORKDIR /backend
COPY backend/go.* ./
RUN go mod download
COPY backend/ .
RUN go build -o server

# Stage 3: Production finale
FROM alpine:latest
RUN apk add --no-cache ca-certificates
COPY --from=backend-builder /backend/server /server
COPY --from=frontend-builder /frontend/dist /static
EXPOSE 8080
CMD ["/server"]
```

### Construire jusqu'à une étape spécifique

```bash
# Builder uniquement le stage "builder"
docker build --target builder -t mon-app:builder .

# Build complet (toutes les étapes)
docker build -t mon-app:prod .
```

## Optimisation 6 : Analyser et réduire la taille

### Commandes d'analyse

**Voir la taille d'une image** :
```bash
docker images mon-app
```

**Voir l'historique des couches** :
```bash
docker history mon-app:latest
```

Sortie :
```
IMAGE          CREATED        CREATED BY                                      SIZE
a1b2c3d4e5f6   2 hours ago    CMD ["python" "app.py"]                        0B
f6e5d4c3b2a1   2 hours ago    COPY . . # buildkit                            15.2MB
...
```

**Voir la taille détaillée** :
```bash
docker history mon-app:latest --human --no-trunc
```

### Outil : dive (analyse détaillée)

**Installation** :
```bash
# macOS
brew install dive

# Linux
wget https://github.com/wagoodman/dive/releases/download/v0.11.0/dive_0.11.0_linux_amd64.deb
sudo dpkg -i dive_0.11.0_linux_amd64.deb
```

**Utilisation** :
```bash
dive mon-app:latest
```

Dive affiche :
- Chaque couche avec sa taille
- Les fichiers ajoutés/modifiés/supprimés
- Le "waste" (espace gaspillé)
- Un score d'efficacité

### Analyser les grosses couches

**Script d'analyse** :
```bash
#!/bin/bash
# analyze-image.sh

IMAGE=$1

echo "=== Taille totale ==="
docker images $IMAGE --format "{{.Size}}"

echo ""
echo "=== Top 5 couches les plus grosses ==="
docker history $IMAGE --human --format "{{.Size}}\t{{.CreatedBy}}" | \
  sort -hr | head -5

echo ""
echo "=== Analyse détaillée avec dive ==="
dive $IMAGE
```

### Identifier les fichiers inutiles

**Entrer dans l'image** :
```bash
docker run -it mon-app:latest sh

# Explorer
du -sh /* | sort -hr
find / -type f -size +10M
```

## Optimisation 7 : Astuces avancées

### Utiliser COPY au lieu d'ADD

```dockerfile
# ✅ Préféré
COPY app.py .

# ⚠️ Éviter (sauf si besoin d'extraction tar)
ADD app.py .
```

`COPY` est plus simple et plus prévisible.

### Copier seulement ce qui est nécessaire

```dockerfile
# ❌ Copie tout
COPY . .

# ✅ Copie seulement le nécessaire
COPY src/ ./src/
COPY package.json package-lock.json ./
COPY config/ ./config/
```

### Utiliser .dockerignore efficacement

**.dockerignore** :
```
node_modules/
.git/
*.log
.env
tests/
docs/
*.md
.vscode/
.idea/
```

**Impact** :
- Sans .dockerignore : contexte de 500 MB
- Avec .dockerignore : contexte de 15 MB

### Éviter les commandes interactives

```dockerfile
# ❌ Peut bloquer le build
RUN apt-get install -y package

# ✅ Forcer non-interactif
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y --no-install-recommends package
```

### Utiliser des variables pour éviter la duplication

```dockerfile
FROM python:3.11-slim

# Variable pour réutilisation
ARG PYTHON_DEPS="flask requests pandas"

RUN pip install --no-cache-dir $PYTHON_DEPS
```

## Exemple complet : Application optimisée

### Avant optimisation

**Dockerfile non optimisé** :
```dockerfile
FROM python:3.11
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
RUN apt-get update
RUN apt-get install -y curl
CMD ["python", "app.py"]
```

**Résultat** :
- Taille : 980 MB
- Build time : 3 min
- Reconstruction : 3 min (cache invalide souvent)

### Après optimisation

**Dockerfile optimisé** :
```dockerfile
# Multi-stage build
FROM python:3.11-slim AS builder

# Variables
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

WORKDIR /app

# Dépendances système (une seule couche)
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        gcc \
        curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Installation Python (ordre optimisé)
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Image finale
FROM python:3.11-slim

# Utilisateur non-root
RUN useradd -m -u 1000 appuser

WORKDIR /app

# Copier seulement les packages installés
COPY --from=builder /root/.local /home/appuser/.local
COPY --chown=appuser:appuser . .

# Variables d'environnement
ENV PATH=/home/appuser/.local/bin:$PATH \
    PYTHONUNBUFFERED=1

USER appuser

EXPOSE 8000
CMD ["python", "app.py"]
```

**Résultat** :
- Taille : 185 MB (81% de réduction)
- Build time : 1 min (cache efficace)
- Reconstruction : 10s (code seul)

## Checklist d'optimisation

### Avant de commencer

- [ ] Créer `.dockerignore` avec fichiers inutiles
- [ ] Vérifier la taille de l'image de base
- [ ] Identifier les dépendances nécessaires

### Structure du Dockerfile

- [ ] Instructions ordonnées du moins au plus changeant
- [ ] Fichiers de dépendances copiés avant le code
- [ ] Commandes RUN groupées et nettoyées
- [ ] Multi-stage build si applicable

### Nettoyage

- [ ] Caches de packages supprimés
- [ ] Outils de build supprimés après usage
- [ ] Fichiers temporaires nettoyés
- [ ] `--no-cache` ou équivalent utilisé

### Sécurité et bonnes pratiques

- [ ] Utilisateur non-root défini
- [ ] Image de base avec tag spécifique
- [ ] Variables d'environnement documentées
- [ ] Ports EXPOSE documentés

### Validation

- [ ] Taille < objectif (ex: < 200 MB)
- [ ] Build time < 2 minutes
- [ ] Reconstruction < 30 secondes
- [ ] Score dive > 80%

## Benchmarks et objectifs

### Tailles cibles par type d'application

| Type d'application | Taille cible | Acceptable | Trop gros |
|-------------------|--------------|------------|-----------|
| Microservice Go | < 20 MB | < 50 MB | > 100 MB |
| API Python | < 150 MB | < 300 MB | > 500 MB |
| Application Node.js | < 200 MB | < 400 MB | > 600 MB |
| Application Java | < 300 MB | < 500 MB | > 800 MB |
| Frontend (SPA) | < 30 MB | < 50 MB | > 100 MB |

### Temps de build cibles

| Opération | Idéal | Acceptable | Problème |
|-----------|-------|------------|----------|
| Build complet (sans cache) | < 2 min | < 5 min | > 10 min |
| Rebuild (changement code) | < 10 s | < 30 s | > 1 min |
| Rebuild (changement dépendances) | < 1 min | < 3 min | > 5 min |

## Outils et ressources

### Outils d'analyse

- **dive** : Analyse détaillée des couches
- **docker-slim** : Optimisation automatique
- **hadolint** : Linter pour Dockerfiles
- **docker history** : Historique des couches

### Scripts utiles

**Script de comparaison** :
```bash
#!/bin/bash
# compare-images.sh

echo "Image 1: $1"
docker images $1 --format "Size: {{.Size}}"
echo ""

echo "Image 2: $2"
docker images $2 --format "Size: {{.Size}}"
echo ""

echo "Top 5 couches Image 1:"
docker history $1 --human | head -6
echo ""

echo "Top 5 couches Image 2:"
docker history $2 --human | head -6
```

## Résumé

Dans cette section, vous avez appris à **optimiser vos images Docker** :

- ✅ **Ordre des instructions** : Du moins au plus changeant pour le cache
- ✅ **Réduction des couches** : Fusionner les commandes, nettoyer dans la même couche
- ✅ **Image de base** : Choisir slim ou alpine selon les besoins
- ✅ **Techniques de nettoyage** : Supprimer caches et fichiers temporaires
- ✅ **Multi-stage builds** : Séparer build et production
- ✅ **Analyse** : Utiliser dive et docker history
- ✅ **Astuces avancées** : .dockerignore, COPY vs ADD, variables

**Points clés à retenir** :

🔑 **L'ordre compte** : Fichiers de dépendances avant le code
🔑 **Nettoyer dans la même couche** : Suppression dans le même RUN
🔑 **Alpine = léger** : Mais peut nécessiter des adaptations
🔑 **Multi-stage = puissant** : Image finale ultra-légère
🔑 **Mesurer** : docker history et dive pour analyser

### Exemple de transformation

**Avant** : 1200 MB, 5 min de build
**Après** : 150 MB, 30s de rebuild
**Gain** : 92% d'espace, 10x plus rapide

Vous savez maintenant créer des images Docker **professionnelles, optimisées et efficaces** ! Cette compétence est essentielle pour des déploiements rapides et des coûts d'infrastructure réduits.

Dans les sections suivantes du tutoriel, vous découvrirez comment gérer les données persistantes avec les volumes, configurer les réseaux, et orchestrer plusieurs conteneurs avec Docker Compose.

⏭️ [Gestion des données](/06-gestion-des-donnees/README.md)
