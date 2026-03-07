🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 5.1 Comprendre le Dockerfile

## Introduction

Jusqu'à présent, vous avez travaillé avec des images Docker **préexistantes**, créées par d'autres et téléchargées depuis Docker Hub. Mais la vraie puissance de Docker réside dans la capacité de **créer vos propres images personnalisées** adaptées à vos besoins spécifiques.

Le **Dockerfile** est le point de départ de cette création : c'est un fichier texte contenant les **instructions** pour construire une image Docker. C'est en quelque sorte la "recette" qui décrit comment assembler votre image.

## Qu'est-ce qu'un Dockerfile ?

### Définition

Un **Dockerfile** est un fichier texte simple qui contient une série d'instructions que Docker exécute automatiquement pour construire une image. Chaque instruction ajoute une nouvelle couche (layer) à l'image.

### Analogie

Pensez au Dockerfile comme à une **recette de cuisine** :

```
Recette de cuisine          →    Dockerfile
================================
Ingrédients                 →    Image de base (FROM)
Étape 1: Préchauffer        →    Configuration (ENV)
Étape 2: Mélanger           →    Copie des fichiers (COPY)
Étape 3: Cuire              →    Installation (RUN)
Plat final                  →    Image Docker
```

Tout comme une recette décrit **comment préparer** un plat (et non le plat lui-même), un Dockerfile décrit **comment construire** une image (et non l'image elle-même).

### Caractéristiques principales

- **Nom du fichier** : `Dockerfile` (sans extension, avec un "D" majuscule par convention)
- **Format** : Fichier texte simple
- **Syntaxe** : Une instruction par ligne
- **Encodage** : UTF-8
- **Commentaires** : Lignes commençant par `#`

## Structure d'un Dockerfile

### Exemple minimal

Voici le Dockerfile le plus simple possible :

```dockerfile
FROM ubuntu:22.04  
CMD ["echo", "Hello Docker!"]  
```

Ce Dockerfile fait deux choses :
1. Part d'une image Ubuntu 22.04
2. Configure la commande qui s'exécutera au démarrage du conteneur

### Exemple plus complet

Voici un exemple plus réaliste pour une application Node.js :

```dockerfile
# Utiliser Node.js version 18 comme image de base
FROM node:18

# Définir le répertoire de travail
WORKDIR /app

# Copier les fichiers de dépendances
COPY package*.json ./

# Installer les dépendances
RUN npm install

# Copier le reste du code de l'application
COPY . .

# Exposer le port 3000
EXPOSE 3000

# Démarrer l'application
CMD ["node", "server.js"]
```

### Anatomie d'une instruction

Chaque ligne d'un Dockerfile suit ce format :

```dockerfile
INSTRUCTION arguments
```

- **INSTRUCTION** : mot-clé en MAJUSCULES (par convention)
- **arguments** : paramètres de l'instruction

Exemples :
```dockerfile
FROM node:18  
RUN apt-get update  
COPY . /app  
```

## Principe fondamental : les couches (layers)

### Qu'est-ce qu'une couche ?

Chaque instruction dans un Dockerfile crée une **nouvelle couche** (layer) dans l'image finale. Les couches sont **empilées** les unes sur les autres, comme des couches de gâteau.

### Visualisation

```
┌─────────────────────────┐
│   CMD ["node", "app.js"]│  ← Couche 5: Commande de démarrage
├─────────────────────────┤
│   COPY . /app           │  ← Couche 4: Code de l'application
├─────────────────────────┤
│   RUN npm install       │  ← Couche 3: Dépendances installées
├─────────────────────────┤
│   COPY package.json /app│  ← Couche 2: Fichier de config
├─────────────────────────┤
│   FROM node:18          │  ← Couche 1: Image de base
└─────────────────────────┘
```

### Pourquoi les couches sont importantes ?

1. **Mise en cache** : Docker met en cache chaque couche. Si une couche n'a pas changé, Docker la réutilise au lieu de la reconstruire.

2. **Efficacité** : Les images peuvent partager des couches communes, économisant de l'espace disque.

3. **Rapidité** : Seules les couches modifiées sont reconstruites lors d'un rebuild.

### Exemple de mise en cache

**Première construction** :
```bash
docker build -t mon-app .
```
```
[+] Building 45.2s (9/9) FINISHED
 => [1/5] FROM node:18
 => [2/5] COPY package.json /app
 => [3/5] RUN npm install
 => [4/5] COPY . /app
 => [5/5] CMD ["node", "app.js"]
 => exporting to image
```

**Deuxième construction** (sans modifications) :
```bash
docker build -t mon-app .
```
```
[+] Building 0.5s (9/9) FINISHED
 => CACHED [1/5] FROM node:18
 => CACHED [2/5] COPY package.json /app
 => CACHED [3/5] RUN npm install
 => CACHED [4/5] COPY . /app
 => CACHED [5/5] CMD ["node", "app.js"]
 => exporting to image
```

Tout est pris du cache ! Construction quasi instantanée.

**Troisième construction** (après modification du code) :
```bash
docker build -t mon-app .
```
```
[+] Building 2.1s (9/9) FINISHED
 => CACHED [1/5] FROM node:18
 => CACHED [2/5] COPY package.json /app
 => CACHED [3/5] RUN npm install
 => [4/5] COPY . /app                    ← Reconstruite
 => [5/5] CMD ["node", "app.js"]         ← Reconstruite
 => exporting to image
```

Seules les étapes 4 et 5 sont reconstruites !

### Invalidation du cache

Une couche est reconstruite si :
- Son instruction a changé
- Ses fichiers sources ont changé
- Une couche précédente a été reconstruite

💡 **Règle d'or** : Placez les instructions qui changent rarement en **haut** du Dockerfile, et celles qui changent souvent en **bas**.

## Conventions et bonnes pratiques de base

### 1. Nom et emplacement

**Convention** : Le fichier s'appelle `Dockerfile` (sans extension)

```
mon-projet/
├── Dockerfile          ← Nom standard
├── src/
├── package.json
└── README.md
```

**Alternative** : Vous pouvez utiliser d'autres noms, mais devez le spécifier lors du build :

```bash
docker build -f Dockerfile.dev -t mon-app .
```

### 2. Commentaires

Utilisez `#` pour ajouter des commentaires :

```dockerfile
# Ceci est un commentaire
FROM node:18

# Installation des dépendances système
RUN apt-get update && apt-get install -y \
    curl \
    git
```

### 3. Une instruction par ligne

```dockerfile
# ✅ Bon - lisible
FROM node:18  
WORKDIR /app  
COPY . .  

# ❌ Invalide - une seule instruction par ligne
FROM node:18 WORKDIR /app COPY . .
```

### 4. Instructions en majuscules

Par convention, les instructions sont en MAJUSCULES :

```dockerfile
# ✅ Bon
FROM node:18  
RUN npm install  

# ❌ Éviter (fonctionne mais non conventionnel)
from node:18  
run npm install  
```

### 5. Ordre logique des instructions

```dockerfile
# 1. Image de base
FROM python:3.11

# 2. Métadonnées (optionnel)
LABEL maintainer="vous@example.com"

# 3. Configuration de l'environnement
ENV APP_HOME=/app  
WORKDIR $APP_HOME  

# 4. Installation des dépendances système
RUN apt-get update && apt-get install -y curl

# 5. Copie des fichiers de dépendances
COPY requirements.txt .

# 6. Installation des dépendances de l'application
RUN pip install -r requirements.txt

# 7. Copie du code de l'application
COPY . .

# 8. Configuration du réseau
EXPOSE 8000

# 9. Commande de démarrage
CMD ["python", "app.py"]
```

## Le processus de construction (build)

### Commande de base

Pour construire une image à partir d'un Dockerfile :

```bash
docker build -t nom-image:tag .
```

Décomposition :
- `docker build` : commande de construction
- `-t nom-image:tag` : donne un nom et un tag à l'image
- `.` : contexte de build (répertoire actuel)

### Exemple complet

Créez un fichier `Dockerfile` :

```dockerfile
FROM alpine:latest  
RUN echo "Construction de l'image..."  
CMD ["echo", "Image construite avec succès!"]  
```

Construisez l'image :

```bash
docker build -t mon-test:v1 .
```

Sortie :

```
[+] Building 2.3s (6/6) FINISHED
 => [internal] load build definition from Dockerfile
 => => transferring dockerfile: 150B
 => [internal] load .dockerignore
 => => transferring context: 2B
 => [internal] load metadata for docker.io/library/alpine:latest
 => [1/2] FROM docker.io/library/alpine:latest
 => [2/2] RUN echo "Construction de l'image..."
 => exporting to image
 => => exporting layers
 => => writing image sha256:abc123...
 => => naming to docker.io/library/mon-test:v1
```

Testez l'image :

```bash
docker run --rm mon-test:v1
```

Sortie :
```
Image construite avec succès!
```

### Le contexte de build

Le **contexte de build** est l'ensemble des fichiers que Docker peut utiliser pendant la construction. C'est le répertoire spécifié à la fin de la commande `docker build`.

```bash
docker build -t mon-image .
                          ↑
                    contexte de build
```

**Exemple de structure** :

```
mon-projet/              ← Contexte de build
├── Dockerfile
├── src/
│   ├── app.py
│   └── utils.py
├── requirements.txt
└── README.md
```

Tout ce qui est dans `mon-projet/` peut être copié dans l'image avec `COPY` ou `ADD`.

### Le fichier .dockerignore

Certains fichiers ne doivent pas être inclus dans le contexte de build (fichiers temporaires, secrets, etc.). Créez un fichier `.dockerignore` :

```
# Fichier .dockerignore
node_modules/
.git/
.env
*.log
.DS_Store
__pycache__/
*.pyc
```

Cela fonctionne comme `.gitignore` : les fichiers listés ne seront pas envoyés à Docker pendant le build.

**Pourquoi c'est important** :
- Réduit la taille du contexte de build
- Accélère la construction
- Évite de copier des fichiers sensibles dans l'image

## Comprendre le flux de construction

### Étapes du processus

1. **Lecture du Dockerfile** : Docker lit le fichier ligne par ligne

2. **Chargement du contexte** : Docker charge tous les fichiers du répertoire (sauf ceux dans .dockerignore)

3. **Exécution des instructions** : Chaque instruction crée un conteneur temporaire, exécute l'instruction, puis crée une couche

4. **Création de l'image finale** : Toutes les couches sont empilées pour former l'image

### Visualisation du processus

```
Dockerfile               Processus de build              Image finale
──────────               ──────────────────              ────────────

FROM ubuntu:22.04   →    Télécharge ubuntu:22.04    →   [Couche 0: Ubuntu]
                                                              ↓
RUN apt-get update  →    Crée conteneur temporaire  →   [Couche 1: Packages MAJ]
                         Exécute apt-get update          ↓
                         Sauvegarde les changements

COPY app.py /app/   →    Copie le fichier           →   [Couche 2: app.py]
                                                              ↓
CMD ["python", ...]  →    Configure la commande      →   [Couche 3: CMD]
                                                              ↓
                                                         Image complète
```

### Build cache intelligent

Docker est intelligent : il compare l'instruction actuelle avec l'instruction précédente du build précédent.

**Exemple** :

```dockerfile
FROM node:18                    # Couche 1: rarement modifiée → cache  
COPY package.json /app/         # Couche 2: rarement modifiée → cache  
RUN npm install                 # Couche 3: dépend de couche 2 → cache  
COPY . /app/                    # Couche 4: code change souvent → REBUILD  
CMD ["node", "app.js"]          # Couche 5: dépend de couche 4 → REBUILD  
```

Si seul le code change (pas `package.json`), seules les couches 4 et 5 sont reconstruites.

## Types d'instructions (aperçu)

Nous verrons en détail chaque instruction dans les sections suivantes, mais voici un aperçu :

### Instructions de configuration

- **FROM** : Définit l'image de base
- **LABEL** : Ajoute des métadonnées
- **ENV** : Définit des variables d'environnement
- **WORKDIR** : Définit le répertoire de travail
- **USER** : Définit l'utilisateur
- **EXPOSE** : Documente les ports utilisés

### Instructions d'exécution

- **RUN** : Exécute une commande pendant la construction
- **COPY** : Copie des fichiers depuis le contexte
- **ADD** : Copie des fichiers (avec fonctionnalités supplémentaires)

### Instructions de démarrage

- **CMD** : Commande par défaut au démarrage du conteneur
- **ENTRYPOINT** : Point d'entrée du conteneur

### Autres instructions

- **VOLUME** : Déclare un point de montage
- **ARG** : Définit une variable de build
- **ONBUILD** : Instructions pour les images enfants
- **HEALTHCHECK** : Configure les vérifications de santé
- **SHELL** : Change le shell par défaut

## Exemple pratique : application Python simple

### Structure du projet

```
mon-app-python/
├── Dockerfile
├── requirements.txt
└── app.py
```

### app.py

```python
print("Hello from Docker!")  
print("Application Python en cours d'exécution...")  
```

### requirements.txt

```
# Aucune dépendance pour cet exemple simple
```

### Dockerfile

```dockerfile
# Utiliser Python 3.11 comme image de base
FROM python:3.11-slim

# Définir le répertoire de travail dans le conteneur
WORKDIR /app

# Copier le fichier requirements.txt
COPY requirements.txt .

# Installer les dépendances (même si vide dans cet exemple)
RUN pip install --no-cache-dir -r requirements.txt

# Copier le code de l'application
COPY app.py .

# Définir la commande par défaut
CMD ["python", "app.py"]
```

### Construction et exécution

```bash
# Construire l'image
docker build -t mon-app-python:v1 .

# Exécuter le conteneur
docker run --rm mon-app-python:v1
```

Sortie :
```
Hello from Docker!  
Application Python en cours d'exécution...  
```

## Exemple pratique : serveur web simple

### Structure du projet

```
mon-site-web/
├── Dockerfile
└── index.html
```

### index.html

```html
<!DOCTYPE html>
<html>
<head>
    <title>Mon site Docker</title>
</head>
<body>
    <h1>Bienvenue sur mon site conteneurisé !</h1>
    <p>Ce site fonctionne dans un conteneur Docker.</p>
</body>
</html>
```

### Dockerfile

```dockerfile
# Utiliser Nginx comme serveur web
FROM nginx:alpine

# Copier notre page HTML dans le répertoire par défaut de Nginx
COPY index.html /usr/share/nginx/html/

# Nginx expose déjà le port 80, mais on le documente
EXPOSE 80

# Nginx démarre automatiquement, pas besoin de CMD
```

### Construction et exécution

```bash
# Construire l'image
docker build -t mon-site:v1 .

# Exécuter le conteneur et mapper le port
docker run -d -p 8080:80 --name mon-site mon-site:v1

# Accéder au site dans votre navigateur
# http://localhost:8080
```

## Déboguer un Dockerfile

### Voir les couches d'une image

```bash
docker history mon-image:v1
```

Cela affiche toutes les couches et les commandes qui les ont créées.

### Construire jusqu'à une étape spécifique

```bash
docker build --target nom-etape -t mon-image:debug .
```

(Nécessite des builds multi-étapes, voir section 11.1)

### Inspecter une image

```bash
docker inspect mon-image:v1
```

### Tester interactivement

Pour tester vos commandes avant de les mettre dans le Dockerfile :

```bash
# Démarrer un conteneur de l'image de base
docker run -it python:3.11 bash

# Tester vos commandes
pip install flask  
python --version  

# Une fois satisfait, ajoutez-les au Dockerfile
```

## Erreurs courantes de débutants

### 1. Oublier le contexte de build

```bash
# ❌ Erreur
docker build -t mon-image

# ✅ Correct - spécifier le contexte
docker build -t mon-image .
```

### 2. Mauvais ordre des instructions

```dockerfile
# ❌ Inefficace - invalide le cache à chaque changement de code
FROM node:18  
COPY . /app  
RUN npm install  

# ✅ Efficace - cache optimisé
FROM node:18  
COPY package.json /app/  
RUN npm install  
COPY . /app  
```

### 3. Ne pas utiliser .dockerignore

Sans `.dockerignore`, tout est copié, y compris `node_modules/`, `.git/`, etc., ralentissant le build.

### 4. Plusieurs instructions RUN inutiles

```dockerfile
# ❌ Crée 3 couches
RUN apt-get update  
RUN apt-get install -y curl  
RUN apt-get install -y git  

# ✅ Crée 1 couche
RUN apt-get update && \
    apt-get install -y curl git
```

### 5. Oublier de nettoyer après installation

```dockerfile
# ❌ Garde les caches apt
RUN apt-get update && apt-get install -y curl

# ✅ Nettoie pour réduire la taille
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*
```

## Ressources et documentation

### Image de base

Choisissez une image de base adaptée :
- **Alpine** : ultra-légère (~5 MB) mais peut manquer d'outils
- **Slim** : légère (~100-150 MB) avec l'essentiel
- **Standard** : complète (~300-900 MB) avec tous les outils

Exemples :
```dockerfile
FROM python:3.11-alpine   # Le plus léger  
FROM python:3.11-slim     # Compromis  
FROM python:3.11          # Le plus complet  
```

### Documentation officielle

- [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)
- [Best practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

## Résumé

Dans cette section, vous avez appris :

- ✅ **Ce qu'est un Dockerfile** : un fichier texte avec des instructions pour construire une image
- ✅ **La structure de base** : une instruction par ligne, format `INSTRUCTION arguments`
- ✅ **Le concept de couches** : chaque instruction crée une couche, empilées pour former l'image
- ✅ **Le système de cache** : Docker réutilise les couches non modifiées pour accélérer les builds
- ✅ **Les conventions** : nom du fichier, commentaires, ordre des instructions
- ✅ **Le processus de build** : comment Docker construit une image étape par étape
- ✅ **Le contexte de build** : l'ensemble des fichiers disponibles pendant la construction
- ✅ **Le fichier .dockerignore** : pour exclure certains fichiers du contexte
- ✅ **Des exemples pratiques** : applications Python et serveur web

Vous avez maintenant une compréhension solide de ce qu'est un Dockerfile et comment il fonctionne. Dans les sections suivantes, vous découvrirez en détail chaque instruction disponible et comment les utiliser efficacement pour créer vos propres images Docker optimisées.

⏭️ [Instructions de base (FROM, RUN, COPY, ADD) et la différence cruciale entre CMD et ENTRYPOINT](/05-creation-dimages-personnalisees/02-instructions-de-base-et-cmd-vs-entrypoint.md)
