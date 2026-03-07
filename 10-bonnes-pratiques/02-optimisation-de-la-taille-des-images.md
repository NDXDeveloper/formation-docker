🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 10.2 Optimisation de la taille des images

## Introduction

La taille des images Docker a un impact direct sur les performances de vos applications et sur vos coûts d'infrastructure. Des images plus petites se téléchargent plus rapidement, se déploient plus vite et consomment moins d'espace disque et de bande passante. Dans cette section, nous allons explorer les techniques pour créer des images Docker optimisées sans compromettre leur fonctionnalité.

## Pourquoi optimiser la taille des images ?

### Les avantages des images légères

1. **Déploiement plus rapide** : Une image de 50 MB se télécharge beaucoup plus vite qu'une image de 1 GB
2. **Économies de stockage** : Moins d'espace nécessaire sur vos serveurs et dans vos registres
3. **Réduction des coûts** : Moins de bande passante utilisée, notamment avec les registres cloud payants
4. **Sécurité améliorée** : Moins de packages installés signifie moins de vulnérabilités potentielles
5. **Temps de build réduit** : Les layers en cache sont plus petits et plus rapides à traiter

### Exemple concret

Imaginez que vous devez déployer votre application sur 100 serveurs :
- Avec une image de 1 GB : 100 GB de données à transférer
- Avec une image de 100 MB : 10 GB de données à transférer

C'est une différence de 90 GB, ce qui représente un gain de temps et d'argent considérable !

## Comprendre les layers Docker

Avant d'optimiser, il est important de comprendre comment Docker construit les images.

### Le système de layers

Chaque instruction dans un Dockerfile crée un nouveau layer (couche). Ces layers sont empilés les uns sur les autres pour former l'image finale.

```dockerfile
FROM ubuntu:22.04        # Layer 1  
RUN apt-get update       # Layer 2  
RUN apt-get install -y python3  # Layer 3  
COPY app.py /app/        # Layer 4  
```

**Important :** Chaque layer conserve TOUTES les modifications par rapport au layer précédent, y compris les fichiers supprimés !

### Exemple du problème

```dockerfile
# Ce Dockerfile crée une image inutilement grande
FROM ubuntu:22.04  
RUN apt-get update && apt-get install -y wget  
RUN wget https://example.com/fichier-enorme.zip  
RUN unzip fichier-enorme.zip  
RUN rm fichier-enorme.zip  # Le fichier est supprimé mais toujours dans le layer !  
```

Même si le fichier ZIP est supprimé, il reste présent dans le layer où il a été créé, augmentant la taille totale de l'image.

## Techniques d'optimisation

### 1. Choisir la bonne image de base

Le choix de l'image de base est la décision la plus importante pour la taille finale.

#### Comparaison des tailles d'images de base

```bash
# Images Ubuntu
ubuntu:22.04           → ~77 MB

# Images Debian
debian:12-slim         → ~74 MB

# Images Alpine (très légères)
alpine:3.18            → ~7 MB

# Images Node.js
node:18                → ~1 GB  
node:18-slim           → ~240 MB  
node:18-alpine         → ~170 MB  

# Images Python
python:3.11            → ~1 GB  
python:3.11-slim       → ~125 MB  
python:3.11-alpine     → ~50 MB  
```

**Recommandations :**

- **Alpine Linux** : Idéal pour la production, très légère mais peut avoir des incompatibilités
- **Debian Slim / Ubuntu** : Bon compromis entre compatibilité et taille
- **Images complètes** : À réserver au développement uniquement

#### Exemple avec différentes bases

```dockerfile
# Version lourde (1+ GB)
FROM python:3.11  
COPY app.py .  
RUN pip install flask  
CMD ["python", "app.py"]  

# Version optimisée (125 MB)
FROM python:3.11-slim  
COPY app.py .  
RUN pip install flask  
CMD ["python", "app.py"]  

# Version ultra-légère (60 MB)
FROM python:3.11-alpine  
COPY app.py .  
RUN pip install flask  
CMD ["python", "app.py"]  
```

### 2. Combiner les commandes RUN

Chaque instruction RUN crée un nouveau layer. En combinant les commandes, vous réduisez le nombre de layers et optimisez le cache.

#### ❌ Mauvaise pratique

```dockerfile
FROM ubuntu:22.04  
RUN apt-get update  
RUN apt-get install -y curl  
RUN apt-get install -y git  
RUN apt-get install -y vim  
RUN rm -rf /var/lib/apt/lists/*  
```

Problèmes :
- 5 layers créés
- Les fichiers temporaires persistent dans les layers intermédiaires
- Taille totale plus grande

#### ✅ Bonne pratique

```dockerfile
FROM ubuntu:22.04  
RUN apt-get update && \  
    apt-get install -y \
        curl \
        git \
        vim && \
    rm -rf /var/lib/apt/lists/*
```

Avantages :
- Un seul layer créé
- Les fichiers temporaires sont nettoyés dans le même layer
- Taille réduite

### 3. Nettoyer dans la même commande

Il est crucial de nettoyer les fichiers temporaires et les caches dans la MÊME instruction RUN qui les a créés.

#### Exemple avec apt (Debian/Ubuntu)

```dockerfile
FROM ubuntu:22.04

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        python3 \
        python3-pip && \
    # Nettoyer dans la même instruction
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
```

**Note :** L'option `--no-install-recommends` évite d'installer les packages recommandés mais non essentiels.

#### Exemple avec apk (Alpine)

```dockerfile
FROM alpine:3.18

RUN apk add --no-cache \
    python3 \
    py3-pip
```

**Note :** L'option `--no-cache` d'Alpine évite automatiquement de stocker le cache.

#### Exemple avec pip (Python)

```dockerfile
FROM python:3.11-slim

COPY requirements.txt .  
RUN pip install --no-cache-dir -r requirements.txt  
```

**Note :** L'option `--no-cache-dir` évite de stocker le cache pip.

### 4. Utiliser .dockerignore

Le fichier `.dockerignore` fonctionne comme `.gitignore` : il empêche certains fichiers d'être copiés dans l'image.

#### Créer un fichier .dockerignore

```
# Fichiers de développement
.git
.gitignore
README.md
*.md

# Dépendances
node_modules
__pycache__
*.pyc
.pytest_cache

# Fichiers de build
dist  
build  
*.egg-info

# Environnements virtuels
venv  
env  
.env

# IDE
.vscode
.idea
*.swp
*.swo

# Logs et données temporaires
*.log
tmp  
temp  

# Fichiers de test
tests  
test  
*.test.js
```

#### Impact du .dockerignore

Sans `.dockerignore` :
```dockerfile
FROM node:18-alpine  
WORKDIR /app  
COPY . .                    # Copie TOUT, y compris node_modules (100+ MB)  
RUN npm install  
CMD ["node", "server.js"]  
```

Avec `.dockerignore` :
```dockerfile
FROM node:18-alpine  
WORKDIR /app  
COPY package*.json ./  
RUN npm install  
COPY . .                    # Copie seulement les fichiers nécessaires  
CMD ["node", "server.js"]  
```

Gain : Plusieurs centaines de Mo, plus un build plus rapide !

### 5. Multi-stage builds

Les multi-stage builds permettent de séparer la phase de compilation de la phase d'exécution. C'est l'une des techniques les plus puissantes pour réduire la taille des images.

#### Concept

Vous utilisez une première image pour compiler votre application (avec tous les outils nécessaires), puis vous copiez uniquement le résultat final dans une image plus légère.

#### Exemple avec une application Go

**Sans multi-stage (image lourde) :**

```dockerfile
FROM golang:1.23  
WORKDIR /app  
COPY . .  
RUN go build -o myapp  
CMD ["./myapp"]  
```

Résultat : ~1 GB (contient le compilateur Go et toutes les dépendances de build)

**Avec multi-stage (image optimisée) :**

```dockerfile
# Stage 1: Build
FROM golang:1.23 AS builder  
WORKDIR /app  
COPY . .  
RUN go build -o myapp  

# Stage 2: Runtime
FROM alpine:3.18  
WORKDIR /app  
COPY --from=builder /app/myapp .  
CMD ["./myapp"]  
```

Résultat : ~15 MB (contient seulement l'exécutable compilé)

#### Exemple avec une application Node.js

```dockerfile
# Stage 1: Build
FROM node:18 AS builder  
WORKDIR /app  
COPY package*.json ./  
RUN npm install  
COPY . .  
RUN npm run build  

# Stage 2: Runtime
FROM node:18-alpine  
WORKDIR /app  
COPY package*.json ./  
RUN npm ci --omit=dev  
COPY --from=builder /app/dist ./dist  
CMD ["node", "dist/server.js"]  
```

Avantages :
- Les devDependencies ne sont pas dans l'image finale
- Les fichiers sources TypeScript ne sont pas inclus
- Image finale beaucoup plus légère

#### Exemple avec une application Python

```dockerfile
# Stage 1: Build
FROM python:3.11 AS builder  
WORKDIR /app  
COPY requirements.txt .  
RUN pip install --user --no-cache-dir -r requirements.txt  

# Stage 2: Runtime
FROM python:3.11-slim  
WORKDIR /app  
# Copier les packages installés depuis le builder
COPY --from=builder /root/.local /root/.local  
COPY . .  
# Ajouter les binaires au PATH
ENV PATH=/root/.local/bin:$PATH  
CMD ["python", "app.py"]  
```

### 6. Optimiser l'ordre des instructions

Docker met en cache chaque layer. En organisant intelligemment vos instructions, vous pouvez réutiliser le cache et accélérer les builds.

#### ❌ Ordre non optimisé

```dockerfile
FROM node:18-alpine  
WORKDIR /app  
COPY . .                    # Copie tout (invalide le cache à chaque modification)  
RUN npm install             # Réinstalle TOUT à chaque fois  
CMD ["node", "server.js"]  
```

Problème : Chaque modification du code invalide le cache et force une réinstallation complète des dépendances.

#### ✅ Ordre optimisé

```dockerfile
FROM node:18-alpine  
WORKDIR /app  
COPY package*.json ./       # Copie d'abord les fichiers de dépendances  
RUN npm install             # S'exécute seulement si package.json change  
COPY . .                    # Copie le code (changements fréquents en dernier)  
CMD ["node", "server.js"]  
```

Avantages :
- Les dépendances ne sont réinstallées que si `package.json` change
- Les modifications du code n'invalident pas le cache des dépendances
- Builds beaucoup plus rapides

### 7. Utiliser des images distroless

Les images "distroless" sont des images ultra-minimalistes qui contiennent uniquement votre application et ses dépendances runtime, sans système d'exploitation complet.

```dockerfile
# Stage 1: Build
FROM node:18 AS builder  
WORKDIR /app  
COPY package*.json ./  
RUN npm install  
COPY . .  
RUN npm run build  

# Stage 2: Runtime avec distroless
FROM gcr.io/distroless/nodejs18-debian11  
WORKDIR /app  
COPY --from=builder /app/dist ./dist  
COPY --from=builder /app/node_modules ./node_modules  
CMD ["dist/server.js"]  
```

Avantages :
- Taille très réduite
- Pas de shell, pas de package manager → sécurité renforcée
- Surface d'attaque minimale

Inconvénients :
- Pas de shell → débogage plus difficile
- Compatibilité à vérifier

### 8. Supprimer les fichiers inutiles

Supprimez systématiquement les fichiers qui ne sont pas nécessaires à l'exécution.

```dockerfile
FROM python:3.11-slim

WORKDIR /app  
COPY requirements.txt .  

RUN pip install --no-cache-dir -r requirements.txt && \
    # Supprimer les fichiers .pyc (bytecode)
    find /usr/local/lib/python3.11 -name '*.pyc' -delete && \
    # Supprimer les tests et la documentation des packages
    find /usr/local/lib/python3.11 -name 'tests' -type d -exec rm -rf {} + && \
    find /usr/local/lib/python3.11 -name '*.md' -delete

COPY . .  
CMD ["python", "app.py"]  
```

### 9. Compresser les fichiers statiques

Si votre application contient des fichiers statiques volumineux, pensez à les compresser.

```dockerfile
FROM nginx:alpine

# Installer gzip
RUN apk add --no-cache gzip

# Copier et compresser les fichiers statiques
COPY static /usr/share/nginx/html/static  
RUN find /usr/share/nginx/html/static -type f \  
    \( -name '*.html' -o -name '*.css' -o -name '*.js' \) \
    -exec gzip -k {} \;

CMD ["nginx", "-g", "daemon off;"]
```

## Exemples pratiques complets

### Exemple 1 : Application Node.js

#### Avant optimisation (1.2 GB)

```dockerfile
FROM node:18  
WORKDIR /app  
COPY . .  
RUN npm install  
EXPOSE 3000  
CMD ["node", "server.js"]  
```

#### Après optimisation (150 MB)

```dockerfile
# Utiliser Alpine pour réduire la taille
FROM node:18-alpine

# Définir le répertoire de travail
WORKDIR /app

# Copier les fichiers de dépendances en premier pour optimiser le cache
COPY package*.json ./

# Installer seulement les dépendances de production sans cache
RUN npm ci --omit=dev --no-cache && \
    npm cache clean --force

# Copier le reste de l'application
COPY . .

# Utiliser un utilisateur non-root
USER node

# Exposer le port
EXPOSE 3000

# Démarrer l'application
CMD ["node", "server.js"]
```

**Gain : 1050 MB (87% de réduction)**

### Exemple 2 : Application Python avec multi-stage

#### Avant optimisation (1.1 GB)

```dockerfile
FROM python:3.11  
WORKDIR /app  
COPY . .  
RUN pip install -r requirements.txt  
CMD ["python", "app.py"]  
```

#### Après optimisation (80 MB)

```dockerfile
# Stage 1: Installation des dépendances
FROM python:3.11-slim AS builder

WORKDIR /app

# Installer les dépendances de build si nécessaire
RUN apt-get update && \
    apt-get install -y --no-install-recommends gcc && \
    rm -rf /var/lib/apt/lists/*

# Copier et installer les dépendances
COPY requirements.txt .  
RUN pip install --user --no-cache-dir -r requirements.txt  

# Stage 2: Image finale
FROM python:3.11-slim

WORKDIR /app

# Copier les packages Python installés
COPY --from=builder /root/.local /root/.local

# Copier le code de l'application
COPY . .

# Ajouter les binaires Python au PATH
ENV PATH=/root/.local/bin:$PATH

# Créer un utilisateur non-root
RUN useradd -m appuser && chown -R appuser:appuser /app  
USER appuser  

CMD ["python", "app.py"]
```

**Gain : 1020 MB (93% de réduction)**

### Exemple 3 : Application React

#### Avant optimisation (1.8 GB)

```dockerfile
FROM node:18  
WORKDIR /app  
COPY . .  
RUN npm install  
RUN npm run build  
RUN npm install -g serve  
CMD ["serve", "-s", "build"]  
```

#### Après optimisation (25 MB)

```dockerfile
# Stage 1: Build
FROM node:18-alpine AS builder  
WORKDIR /app  
COPY package*.json ./  
RUN npm ci --omit=dev  
COPY . .  
RUN npm run build  

# Stage 2: Serveur web
FROM nginx:alpine  
COPY --from=builder /app/build /usr/share/nginx/html  
COPY nginx.conf /etc/nginx/conf.d/default.conf  
EXPOSE 80  
CMD ["nginx", "-g", "daemon off;"]  
```

**Gain : 1775 MB (98% de réduction)**

## Vérifier la taille des images

### Commandes utiles

```bash
# Voir la taille de toutes les images
docker images

# Voir la taille d'une image spécifique
docker images mon-image

# Voir l'historique et la taille de chaque layer
docker history mon-image:latest

# Analyser en détail l'image (avec Docker Desktop)
docker scout quickview mon-image:latest
```

### Utiliser dive pour analyser les layers

L'outil `dive` permet d'explorer en détail les layers d'une image.

```bash
# Installer dive
# Sur Mac avec Homebrew
brew install dive

# Sur Linux
wget https://github.com/wagoodman/dive/releases/download/v0.11.0/dive_0.11.0_linux_amd64.deb  
sudo apt install ./dive_0.11.0_linux_amd64.deb  

# Analyser une image
dive mon-image:latest
```

## Checklist d'optimisation

Avant de construire votre image, vérifiez ces points :

- [ ] Utiliser une image de base légère (alpine, slim, ou distroless)
- [ ] Créer un fichier `.dockerignore` complet
- [ ] Combiner les commandes RUN pour réduire les layers
- [ ] Nettoyer les caches et fichiers temporaires dans la même instruction RUN
- [ ] Utiliser `--no-cache` ou équivalent pour les gestionnaires de packages
- [ ] Ordonner les instructions du moins au plus changeant
- [ ] Utiliser multi-stage builds si compilation nécessaire
- [ ] Installer uniquement les dépendances de production
- [ ] Supprimer les outils de build dans l'image finale
- [ ] Vérifier qu'aucun fichier secret n'est copié dans l'image

## Outils et ressources

### Outils d'analyse

- **docker history** : Voir l'historique des layers
- **dive** : Explorer les layers en détail
- **docker scout** : Scanner les vulnérabilités et la taille
- **container-diff** : Comparer deux images

### Ressources utiles

- [Docker Official Images](https://hub.docker.com/search?q=&type=image&image_filter=official) : Images de référence optimisées
- [Distroless Images](https://github.com/GoogleContainerTools/distroless) : Images Google minimalistes
- [Alpine Linux packages](https://pkgs.alpinelinux.org/packages) : Rechercher des packages Alpine

## Conclusion

L'optimisation de la taille des images Docker est un processus itératif qui combine plusieurs techniques. Voici les principes clés à retenir :

1. **Choisissez la bonne base** : Alpine ou slim pour la production
2. **Utilisez multi-stage builds** : Séparez build et runtime
3. **Nettoyez intelligemment** : Supprimez dans la même instruction RUN
4. **Optimisez le cache** : Ordonnez les instructions correctement
5. **Excluez l'inutile** : Utilisez .dockerignore

En appliquant ces techniques, vous pouvez généralement réduire la taille de vos images de 70% à 95%, ce qui améliore significativement les performances, la sécurité et réduit les coûts.

N'oubliez pas : une image optimisée est une image qui contient exactement ce dont elle a besoin, ni plus, ni moins !

⏭️ [Gestion des secrets et variables d'environnement](/10-bonnes-pratiques/03-gestion-des-secrets-et-variables-denvironnement.md)
