🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 11.1 Multi-stage builds

## Introduction

Les multi-stage builds (constructions multi-étapes) sont une fonctionnalité puissante de Docker qui permet de créer des images finales plus petites et plus sécurisées. Cette technique est particulièrement utile lorsque vous développez des applications qui nécessitent des outils de compilation ou de build, mais qui n'ont pas besoin de ces outils pour s'exécuter en production.

## Le problème à résoudre

### Situation classique sans multi-stage builds

Imaginons que vous développez une application en Java, Python, ou Node.js. Pour construire votre application, vous avez besoin de :
- Compilateurs
- Gestionnaires de paquets
- Outils de développement
- Fichiers sources
- Dépendances de développement

**Le problème** : Si vous installez tous ces outils dans votre image Docker, votre image finale sera très volumineuse et contiendra des outils dont vous n'avez pas besoin en production.

**Exemple concret** : Une application Node.js peut nécessiter 500 Mo d'outils de build, mais l'application compilée finale ne pèse que 50 Mo.

### Ancienne solution (avant les multi-stage builds)

Avant l'introduction des multi-stage builds, les développeurs devaient :
1. Créer un Dockerfile pour builder l'application
2. Créer un autre Dockerfile pour l'image de production
3. Utiliser des scripts shell complexes pour copier les fichiers entre les deux

Cette approche était fastidieuse et source d'erreurs.

## Qu'est-ce qu'un multi-stage build ?

Un multi-stage build permet de définir **plusieurs étapes** (stages) dans un seul Dockerfile. Chaque étape :
- Commence par une instruction `FROM`
- Peut avoir son propre environnement et ses propres outils
- Peut être nommée pour être référencée plus tard
- Permet de copier des fichiers d'une étape à une autre

**L'idée principale** : Utiliser une première étape pour compiler/builder l'application avec tous les outils nécessaires, puis copier uniquement le résultat dans une seconde étape plus légère pour l'exécution.

## Syntaxe de base

### Structure générale

```dockerfile
# Première étape : BUILD
FROM image-de-build AS builder
# Instructions pour compiler/builder l'application
...

# Deuxième étape : PRODUCTION
FROM image-legere
# Copie des fichiers depuis l'étape précédente
COPY --from=builder /chemin/source /chemin/destination
# Instructions pour l'exécution
...
```

### Éléments clés

- **`FROM ... AS nom`** : Nomme une étape pour pouvoir la référencer
- **`COPY --from=nom`** : Copie des fichiers depuis une étape nommée
- **Plusieurs FROM** : Chaque `FROM` démarre une nouvelle étape

## Exemple simple : Application Node.js

Prenons un exemple concret d'une application Node.js.

### Sans multi-stage build (❌ Mauvaise pratique)

```dockerfile
FROM node:18

WORKDIR /app

# Copie des fichiers de dépendances
COPY package*.json ./

# Installation de TOUTES les dépendances (dev + prod)
RUN npm install

# Copie du code source
COPY . .

# Build de l'application
RUN npm run build

# Lancement de l'application
CMD ["npm", "start"]
```

**Problèmes** :
- L'image contient tous les `node_modules` (dev + prod)
- Les fichiers sources TypeScript inutiles en production
- Les outils de build restent dans l'image
- **Taille de l'image : ~1.2 GB**

### Avec multi-stage build (✅ Bonne pratique)

```dockerfile
# ========================================
# ÉTAPE 1 : BUILD
# ========================================
FROM node:18 AS builder

WORKDIR /app

# Installation des dépendances
COPY package*.json ./
RUN npm install

# Copie du code source et build
COPY . .
RUN npm run build

# ========================================
# ÉTAPE 2 : PRODUCTION
# ========================================
FROM node:18-alpine

WORKDIR /app

# Copie uniquement des fichiers nécessaires depuis l'étape builder
COPY package*.json ./

# Installation uniquement des dépendances de production
RUN npm install --production

# Copie des fichiers buildés (pas les sources)
COPY --from=builder /app/dist ./dist

# Variables d'environnement
ENV NODE_ENV=production

# Utilisateur non-root pour la sécurité
USER node

# Lancement de l'application
CMD ["node", "dist/index.js"]
```

**Avantages** :
- Image finale basée sur Alpine (plus légère)
- Seulement les dépendances de production
- Pas de code source, seulement les fichiers compilés
- Meilleure sécurité
- **Taille de l'image : ~150 MB** (réduction de 87% !)

## Exemple avec une application Go

Go est un excellent candidat pour les multi-stage builds car il compile en un binaire statique unique.

```dockerfile
# ========================================
# ÉTAPE 1 : BUILD
# ========================================
FROM golang:1.21 AS builder

WORKDIR /app

# Copie des fichiers de dépendances Go
COPY go.mod go.sum ./
RUN go mod download

# Copie du code source
COPY . .

# Compilation de l'application
# CGO_ENABLED=0 : compilation statique sans dépendances C
RUN CGO_ENABLED=0 GOOS=linux go build -o myapp .

# ========================================
# ÉTAPE 2 : PRODUCTION
# ========================================
FROM alpine:latest

# Installation de certificats SSL (nécessaires pour les requêtes HTTPS)
RUN apk --no-cache add ca-certificates

WORKDIR /root/

# Copie uniquement du binaire compilé
COPY --from=builder /app/myapp .

# Exposition du port
EXPOSE 8080

# Lancement de l'application
CMD ["./myapp"]
```

**Résultat impressionnant** :
- Étape de build : ~800 MB (avec compilateur Go)
- Image finale : ~10 MB (Alpine + binaire)
- **Réduction de 98% !**

## Exemple avec Python et Flask

```dockerfile
# ========================================
# ÉTAPE 1 : BUILD
# ========================================
FROM python:3.11 AS builder

WORKDIR /app

# Installation des dépendances de build
RUN pip install --upgrade pip setuptools wheel

# Copie et installation des dépendances Python
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# ========================================
# ÉTAPE 2 : PRODUCTION
# ========================================
FROM python:3.11-slim

WORKDIR /app

# Copie des dépendances installées depuis builder
COPY --from=builder /root/.local /root/.local

# Copie du code de l'application
COPY . .

# Mise à jour du PATH pour utiliser les packages locaux
ENV PATH=/root/.local/bin:$PATH

# Variables d'environnement
ENV PYTHONUNBUFFERED=1
ENV FLASK_APP=app.py

# Exposition du port
EXPOSE 5000

# Lancement de l'application
CMD ["python", "-m", "flask", "run", "--host=0.0.0.0"]
```

## Utilisation avancée : Plus de deux étapes

Vous pouvez avoir autant d'étapes que nécessaire. Voici un exemple avec trois étapes :

```dockerfile
# ========================================
# ÉTAPE 1 : Téléchargement des dépendances
# ========================================
FROM node:18 AS dependencies

WORKDIR /app
COPY package*.json ./
RUN npm install

# ========================================
# ÉTAPE 2 : Build de l'application
# ========================================
FROM node:18 AS builder

WORKDIR /app
COPY --from=dependencies /app/node_modules ./node_modules
COPY . .
RUN npm run build
RUN npm run test

# ========================================
# ÉTAPE 3 : Image de production
# ========================================
FROM node:18-alpine

WORKDIR /app
COPY package*.json ./
RUN npm install --production
COPY --from=builder /app/dist ./dist

CMD ["node", "dist/index.js"]
```

## Copier depuis des étapes spécifiques

Vous pouvez copier depuis n'importe quelle étape nommée :

```dockerfile
FROM node:18 AS dependencies
WORKDIR /app
COPY package*.json ./
RUN npm install

FROM node:18 AS development-dependencies
WORKDIR /app
COPY package*.json ./
RUN npm install --include=dev

FROM node:18 AS builder
WORKDIR /app
COPY --from=development-dependencies /app/node_modules ./node_modules
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

## Cibler une étape spécifique lors du build

Parfois, vous voulez construire seulement jusqu'à une certaine étape (par exemple, pour le développement).

### Construire jusqu'à une étape spécifique

```bash
# Construire seulement l'étape "builder"
docker build --target builder -t mon-app:builder .

# Construire l'image complète (dernière étape)
docker build -t mon-app:latest .
```

### Exemple pratique : Environnements multiples

```dockerfile
# ========================================
# ÉTAPE 1 : Base commune
# ========================================
FROM node:18 AS base
WORKDIR /app
COPY package*.json ./
RUN npm install

# ========================================
# ÉTAPE 2 : Développement
# ========================================
FROM base AS development
COPY . .
CMD ["npm", "run", "dev"]

# ========================================
# ÉTAPE 3 : Build
# ========================================
FROM base AS builder
COPY . .
RUN npm run build

# ========================================
# ÉTAPE 4 : Production
# ========================================
FROM node:18-alpine AS production
WORKDIR /app
COPY package*.json ./
RUN npm install --production
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/index.js"]
```

**Utilisation** :
```bash
# Pour le développement
docker build --target development -t mon-app:dev .

# Pour la production
docker build --target production -t mon-app:prod .
```

## Bonnes pratiques des multi-stage builds

### 1. Nommez vos étapes de manière explicite

```dockerfile
# ✅ Bon
FROM node:18 AS dependencies
FROM node:18 AS builder
FROM node:18-alpine AS production

# ❌ Éviter
FROM node:18
FROM node:18
FROM node:18-alpine
```

### 2. Utilisez des images de base appropriées

```dockerfile
# Pour le build : image complète
FROM node:18 AS builder

# Pour la production : image slim ou alpine
FROM node:18-alpine AS production
```

### 3. Minimisez le nombre de layers dans l'image finale

```dockerfile
# Dans l'étape de build, vous pouvez avoir beaucoup de layers
FROM node:18 AS builder
RUN apt-get update
RUN apt-get install -y python
RUN npm install
RUN npm run build

# Dans l'étape finale, combinez les commandes
FROM node:18-alpine
RUN apk add --no-cache ca-certificates && \
    addgroup -g 1000 node && \
    adduser -u 1000 -G node -s /bin/sh -D node
```

### 4. Ne copiez que le nécessaire

```dockerfile
# ✅ Bon : copie seulement les fichiers buildés
COPY --from=builder /app/dist ./dist

# ❌ Éviter : copie tout le répertoire de build
COPY --from=builder /app .
```

### 5. Ordonnez les étapes logiquement

```dockerfile
# 1. Téléchargement des dépendances
# 2. Build de l'application
# 3. Tests (optionnel)
# 4. Image finale de production
```

### 6. Utilisez le cache de build efficacement

```dockerfile
# Copiez d'abord les fichiers de dépendances
COPY package*.json ./
RUN npm install

# Puis copiez le code source (qui change plus souvent)
COPY . .
RUN npm run build
```

## Comprendre les bénéfices

### Réduction de la taille des images

| Application | Sans multi-stage | Avec multi-stage | Réduction |
|-------------|------------------|------------------|-----------|
| Node.js | 1.2 GB | 150 MB | 87% |
| Go | 800 MB | 10 MB | 98% |
| Python | 900 MB | 200 MB | 78% |
| Java | 600 MB | 150 MB | 75% |

### Amélioration de la sécurité

- **Moins d'outils** : Pas de compilateurs en production
- **Moins de vulnérabilités** : Surface d'attaque réduite
- **Moins de dépendances** : Seulement les packages nécessaires à l'exécution

### Optimisation des performances

- **Démarrage plus rapide** : Images plus petites à télécharger
- **Moins d'espace disque** : Important dans les environnements cloud
- **Pull plus rapide** : Mise à jour des conteneurs accélérée

## Cas d'usage courants

### 1. Applications compilées (Go, Rust, C++)

Compilez dans une étape, exécutez le binaire dans une image minimale.

### 2. Applications avec transpilation (TypeScript, React)

Transpilez dans une étape, servez les fichiers statiques depuis nginx.

### 3. Applications avec tests

Exécutez les tests dans une étape intermédiaire sans impacter l'image finale.

### 4. Applications avec assets à compiler (Sass, Webpack)

Compilez les assets dans une étape, copiez uniquement les fichiers de production.

## Exemple complet : Application React + Node.js

```dockerfile
# ========================================
# ÉTAPE 1 : Build du frontend React
# ========================================
FROM node:18 AS frontend-builder

WORKDIR /app/frontend

COPY frontend/package*.json ./
RUN npm install

COPY frontend/ .
RUN npm run build

# ========================================
# ÉTAPE 2 : Build du backend Node.js
# ========================================
FROM node:18 AS backend-builder

WORKDIR /app/backend

COPY backend/package*.json ./
RUN npm install

COPY backend/ .
RUN npm run build

# ========================================
# ÉTAPE 3 : Image de production
# ========================================
FROM node:18-alpine

WORKDIR /app

# Copie des dépendances de production du backend
COPY backend/package*.json ./
RUN npm install --production

# Copie du backend compilé
COPY --from=backend-builder /app/backend/dist ./dist

# Copie du frontend compilé vers le dossier public
COPY --from=frontend-builder /app/frontend/build ./public

# Variables d'environnement
ENV NODE_ENV=production
ENV PORT=3000

# Exposition du port
EXPOSE 3000

# Utilisateur non-root
USER node

# Démarrage de l'application
CMD ["node", "dist/server.js"]
```

## Débogage des multi-stage builds

### Voir toutes les étapes intermédiaires

```bash
# Afficher toutes les images intermédiaires
docker images -a
```

### Construire et inspecter une étape spécifique

```bash
# Construire jusqu'à l'étape builder
docker build --target builder -t debug:builder .

# Lancer un conteneur avec cette étape
docker run -it debug:builder /bin/sh
```

### Utiliser BuildKit pour de meilleures performances

```bash
# Activer BuildKit (recommandé)
DOCKER_BUILDKIT=1 docker build -t mon-app .
```

## Résumé

Les multi-stage builds sont une technique essentielle pour :

- ✅ **Réduire la taille des images** (souvent de 70% à 95%)
- ✅ **Améliorer la sécurité** (moins d'outils, moins de vulnérabilités)
- ✅ **Simplifier le Dockerfile** (tout dans un seul fichier)
- ✅ **Accélérer les déploiements** (images plus petites)
- ✅ **Séparer les préoccupations** (build vs runtime)

**Principe fondamental** : Construisez dans une étape lourde avec tous les outils nécessaires, puis copiez uniquement le résultat dans une image légère pour l'exécution en production.

Cette approche est devenue une **meilleure pratique standard** dans le développement avec Docker et est utilisée dans la majorité des projets professionnels modernes.

⏭️ [Gestion des ressources (CPU, mémoire)](/11-sujets-intermediaires/02-gestion-des-ressources.md)
