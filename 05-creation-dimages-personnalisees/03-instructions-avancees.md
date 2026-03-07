🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 5.3 Instructions avancées (ENV, WORKDIR, EXPOSE, USER, VOLUME)

## Introduction

Maintenant que vous maîtrisez les instructions de base (FROM, RUN, COPY, CMD, ENTRYPOINT), il est temps d'explorer les **instructions avancées** qui vous donneront plus de contrôle et de flexibilité dans la création de vos images.

Ces cinq instructions sont essentielles pour créer des images **professionnelles et production-ready** :

- **ENV** : Gérer les variables d'environnement
- **WORKDIR** : Organiser la structure des fichiers
- **EXPOSE** : Documenter les ports réseau
- **USER** : Améliorer la sécurité
- **VOLUME** : Gérer la persistance des données

Bien que qualifiées d'"avancées", ces instructions sont en réalité assez simples à comprendre et vous les utiliserez dans presque tous vos Dockerfiles !

## L'instruction ENV : variables d'environnement

### Qu'est-ce que ENV ?

`ENV` définit des **variables d'environnement** qui seront disponibles :
- Pendant la construction de l'image (dans les instructions suivantes)
- Dans les conteneurs créés à partir de l'image

💡 **Analogie** : Les variables d'environnement sont comme des **paramètres de configuration** pour votre application. C'est un moyen de dire "utilise cette valeur pour ce paramètre".

### Pourquoi utiliser ENV ?

✅ **Configuration flexible** : Changer le comportement sans reconstruire l'image  
✅ **Éviter la duplication** : Définir une valeur une fois, l'utiliser partout  
✅ **Faciliter le déploiement** : Adapter l'application à différents environnements  
✅ **Best practice** : Séparer la configuration du code

### Syntaxe

Il existe deux formes :

**Forme simple** (une variable à la fois) :
```dockerfile
ENV KEY value
```

**Forme multiple** (plusieurs variables en une ligne) :
```dockerfile
ENV KEY1=value1 KEY2=value2 KEY3=value3
```

### Exemples simples

```dockerfile
# Définir une variable simple
ENV NODE_ENV production

# Définir le port d'écoute
ENV PORT 3000

# Définir plusieurs variables
ENV DATABASE_HOST=localhost \
    DATABASE_PORT=5432 \
    DATABASE_NAME=myapp

# Définir un chemin
ENV APP_HOME /app
```

### Utiliser les variables définies

Les variables ENV peuvent être utilisées dans les instructions suivantes du Dockerfile :

```dockerfile
FROM node:18-alpine

# Définir la variable
ENV APP_HOME=/app

# L'utiliser dans WORKDIR
WORKDIR $APP_HOME

# L'utiliser dans COPY
COPY package.json $APP_HOME/

# L'utiliser dans RUN
RUN echo "Application directory: $APP_HOME"
```

### Variables d'environnement courantes

#### Pour les applications

```dockerfile
# Mode de l'application
ENV NODE_ENV=production  
ENV FLASK_DEBUG=0  
ENV RAILS_ENV=production  

# Configuration réseau
ENV HOST=0.0.0.0  
ENV PORT=8000  

# Niveau de log
ENV LOG_LEVEL=info  
ENV DEBUG=false  
```

#### Pour Python

```dockerfile
# Désactiver le buffering de sortie
ENV PYTHONUNBUFFERED=1

# Désactiver l'écriture de fichiers .pyc
ENV PYTHONDONTWRITEBYTECODE=1

# Définir le chemin Python
ENV PYTHONPATH=/app
```

#### Pour Node.js

```dockerfile
# Mode production
ENV NODE_ENV=production

# Désactiver les avertissements
ENV NODE_OPTIONS="--max-old-space-size=2048"
```

#### Pour Java

```dockerfile
# Options JVM
ENV JAVA_OPTS="-Xmx512m -Xms256m"

# Timezone
ENV TZ=Europe/Paris
```

### Valeurs par défaut modifiables

Une bonne pratique est de définir des valeurs par défaut dans le Dockerfile, qui peuvent être remplacées au runtime :

**Dockerfile** :
```dockerfile
FROM python:3.11-slim  
ENV PORT=8000  
ENV DEBUG=false  
CMD ["python", "app.py"]  
```

**Utilisation** :
```bash
# Utilise les valeurs par défaut (PORT=8000, DEBUG=false)
docker run mon-app

# Remplace les valeurs par défaut
docker run -e PORT=9000 -e DEBUG=true mon-app
```

### Variables avec chemins

Pour définir des chemins complexes :

```dockerfile
FROM python:3.11-slim

# Variables de base
ENV APP_HOME=/app \
    APP_USER=appuser

# Variables dérivées (utilisent les précédentes)
ENV APP_CONFIG=$APP_HOME/config \
    APP_DATA=$APP_HOME/data \
    APP_LOGS=$APP_HOME/logs

# Utilisation
WORKDIR $APP_HOME  
RUN mkdir -p $APP_CONFIG $APP_DATA $APP_LOGS  
```

### ENV vs ARG : quelle différence ?

**ENV** : Disponible pendant le build ET dans le conteneur  
**ARG** : Disponible SEULEMENT pendant le build  

```dockerfile
# ARG : valeur de build uniquement
ARG BUILD_VERSION=1.0.0  
RUN echo "Building version: $BUILD_VERSION"  
# ❌ Non disponible dans le conteneur

# ENV : valeur persistante
ENV APP_VERSION=1.0.0  
RUN echo "App version: $APP_VERSION"  
# ✅ Disponible dans le conteneur
```

**Combinaison ARG + ENV** :
```dockerfile
# Valeur par défaut modifiable au build
ARG NODE_ENV=production

# Convertir en ENV pour le runtime
ENV NODE_ENV=$NODE_ENV
```

Utilisation :
```bash
# Build avec valeur par défaut
docker build -t mon-app .

# Build avec valeur personnalisée
docker build --build-arg NODE_ENV=development -t mon-app .
```

### Bonnes pratiques avec ENV

#### 1. Grouper les variables liées

```dockerfile
# ✅ Bon - variables groupées logiquement
ENV DATABASE_HOST=localhost \
    DATABASE_PORT=5432 \
    DATABASE_NAME=myapp \
    DATABASE_USER=user

# ❌ Moins lisible
ENV DATABASE_HOST=localhost  
ENV DATABASE_PORT=5432  
ENV DATABASE_NAME=myapp  
```

#### 2. Utiliser des noms descriptifs en MAJUSCULES

```dockerfile
# ✅ Bon - convention standard
ENV DATABASE_URL=postgresql://localhost/myapp  
ENV API_KEY_SECRET=changeme  
ENV MAX_CONNECTIONS=100  

# ❌ Éviter
ENV db_url=postgresql://localhost/myapp  
ENV ApiKey=changeme  
```

#### 3. Ne pas mettre de secrets dans ENV

```dockerfile
# ❌ DANGEREUX - le mot de passe est visible dans l'image
ENV DATABASE_PASSWORD=supersecret

# ✅ Bon - passer les secrets au runtime
# docker run -e DATABASE_PASSWORD=secret mon-app
```

⚠️ **Important** : Les variables ENV sont visibles dans l'image avec `docker inspect`. Ne mettez JAMAIS de mots de passe ou tokens dedans !

#### 4. Documenter les variables importantes

```dockerfile
# Variables de configuration
# PORT: Port d'écoute de l'application (défaut: 8000)
# DEBUG: Active le mode debug (défaut: false)
# LOG_LEVEL: Niveau de log (debug|info|warning|error)
ENV PORT=8000 \
    DEBUG=false \
    LOG_LEVEL=info
```

## L'instruction WORKDIR : répertoire de travail

### Qu'est-ce que WORKDIR ?

`WORKDIR` définit le **répertoire de travail** pour toutes les instructions suivantes (RUN, CMD, ENTRYPOINT, COPY, ADD). C'est l'équivalent de la commande `cd` dans un shell.

💡 **Analogie** : C'est comme choisir le dossier dans lequel vous allez travailler. Au lieu de toujours taper le chemin complet, vous vous placez dans le bon dossier.

### Syntaxe

```dockerfile
WORKDIR /chemin/vers/repertoire
```

### Comportement important

- Si le répertoire n'existe pas, Docker le **crée automatiquement**
- Les chemins relatifs suivants seront relatifs à ce WORKDIR
- Vous pouvez utiliser WORKDIR plusieurs fois dans un Dockerfile

### Exemples simples

```dockerfile
FROM node:18-alpine

# Définir le répertoire de travail
WORKDIR /app

# Maintenant tous les chemins sont relatifs à /app
COPY package.json .        # Copie dans /app/package.json  
RUN npm install            # Exécute dans /app  
COPY . .                   # Copie dans /app  
CMD ["node", "server.js"]  # Exécute depuis /app  
```

### Sans WORKDIR vs avec WORKDIR

**Sans WORKDIR** (difficile à lire et à maintenir) :
```dockerfile
FROM node:18-alpine  
RUN mkdir -p /app  
COPY package.json /app/package.json  
RUN cd /app && npm install  
COPY . /app/  
CMD cd /app && node server.js  
```

**Avec WORKDIR** (clair et concis) :
```dockerfile
FROM node:18-alpine  
WORKDIR /app  
COPY package.json .  
RUN npm install  
COPY . .  
CMD ["node", "server.js"]  
```

✅ Beaucoup plus lisible !

### Chemins relatifs et absolus

```dockerfile
FROM ubuntu:22.04

# Chemin absolu
WORKDIR /app

# Chemin relatif (par rapport au WORKDIR actuel)
WORKDIR config
# On est maintenant dans /app/config

# Retour à un chemin absolu
WORKDIR /data
# On est maintenant dans /data

# Encore un chemin relatif
WORKDIR logs
# On est maintenant dans /data/logs
```

### Utiliser des variables ENV

```dockerfile
FROM python:3.11-slim

# Définir une variable
ENV APP_HOME=/app

# L'utiliser dans WORKDIR
WORKDIR $APP_HOME

# Tout se passe maintenant dans /app
COPY . .  
RUN pip install -r requirements.txt  
```

### Structure d'application typique

```dockerfile
FROM python:3.11-slim

# Répertoire principal
WORKDIR /app

# Créer la structure (WORKDIR les crée automatiquement)
WORKDIR /app/src  
WORKDIR /app/config  
WORKDIR /app/logs  
WORKDIR /app/data  

# Retour au répertoire principal
WORKDIR /app

# Copier les fichiers
COPY requirements.txt .  
RUN pip install -r requirements.txt  
COPY src/ ./src/  
COPY config/ ./config/  

# L'application démarre depuis /app
CMD ["python", "src/main.py"]
```

### WORKDIR pour différentes étapes

```dockerfile
FROM node:18 AS builder

# Environnement de build
WORKDIR /build  
COPY package*.json ./  
RUN npm ci  
COPY . .  
RUN npm run build  

# Image finale
FROM nginx:alpine  
WORKDIR /usr/share/nginx/html  

# Copier depuis le builder
COPY --from=builder /build/dist .
```

### Bonnes pratiques avec WORKDIR

#### 1. Toujours utiliser WORKDIR plutôt que RUN cd

```dockerfile
# ❌ Mauvais - ne persiste pas entre les instructions
RUN cd /app  
RUN npm install  # N'est PAS dans /app !  

# ✅ Bon - persiste
WORKDIR /app  
RUN npm install  # Est bien dans /app  
```

#### 2. Utiliser des chemins absolus pour plus de clarté

```dockerfile
# ✅ Bon - clair et explicite
WORKDIR /app

# ⚠️ Peut prêter à confusion
WORKDIR app
```

#### 3. Définir WORKDIR tôt dans le Dockerfile

```dockerfile
FROM python:3.11-slim

# Définir rapidement le WORKDIR
WORKDIR /app

# Puis toutes les opérations
COPY requirements.txt .  
RUN pip install -r requirements.txt  
COPY . .  
```

#### 4. Documenter la structure des répertoires

```dockerfile
FROM node:18-alpine

# Structure de l'application :
# /app              - répertoire principal
# /app/src          - code source
# /app/public       - fichiers statiques
# /app/config       - configurations
# /app/logs         - logs de l'application

WORKDIR /app
```

## L'instruction EXPOSE : documenter les ports

### Qu'est-ce que EXPOSE ?

`EXPOSE` **documente** les ports réseau que votre application utilise. C'est une forme de **documentation** pour les utilisateurs de votre image.

⚠️ **IMPORTANT** : `EXPOSE` ne publie PAS réellement les ports ! C'est uniquement informatif.

💡 **Analogie** : C'est comme mettre une étiquette sur une porte : "Cette pièce est accessible par la porte 80". Mais la porte reste fermée jusqu'à ce que vous l'ouvriez explicitement.

### Syntaxe

```dockerfile
EXPOSE <port>  
EXPOSE <port>/<protocol>  
```

### Exemples simples

```dockerfile
# Port HTTP standard
EXPOSE 80

# Port personnalisé
EXPOSE 8080

# Spécifier le protocole (par défaut: TCP)
EXPOSE 3000/tcp  
EXPOSE 53/udp  

# Plusieurs ports
EXPOSE 80 443
```

### Ce que EXPOSE fait (et ne fait pas)

**Ce que EXPOSE fait** :
✅ Documente quels ports l'application utilise  
✅ Permet `docker run -P` de publier automatiquement ces ports  
✅ Aide les outils à comprendre quels ports sont utilisés  
✅ Améliore la documentation de l'image

**Ce que EXPOSE ne fait PAS** :
❌ Ne publie pas automatiquement le port  
❌ Ne rend pas le port accessible depuis l'hôte  
❌ N'a aucun effet de sécurité

### Publier les ports au runtime

Pour rendre le port accessible, utilisez `-p` avec `docker run` :

**Dockerfile** :
```dockerfile
FROM nginx:alpine  
EXPOSE 80  
```

**Utilisation** :
```bash
# ❌ Le port n'est PAS accessible (EXPOSE seul ne suffit pas)
docker run mon-image

# ✅ Publier le port manuellement
docker run -p 8080:80 mon-image
# Accessible sur http://localhost:8080

# ✅ Publier automatiquement sur un port aléatoire
docker run -P mon-image
# Docker choisit un port libre et mappe vers 80
```

### Exemples d'applications courantes

**Application web Node.js** :
```dockerfile
FROM node:18-alpine  
WORKDIR /app  
COPY . .  
RUN npm install  
EXPOSE 3000  
CMD ["npm", "start"]  
```

**API Python Flask** :
```dockerfile
FROM python:3.11-slim  
WORKDIR /app  
COPY requirements.txt .  
RUN pip install -r requirements.txt  
COPY . .  
EXPOSE 5000  
CMD ["python", "app.py"]  
```

**Application avec plusieurs ports** :
```dockerfile
FROM custom-image
# Port web
EXPOSE 80
# Port API
EXPOSE 8080
# Port metrics
EXPOSE 9090  
CMD ["./start.sh"]  
```

### Protocoles TCP et UDP

Par défaut, EXPOSE utilise TCP. Spécifiez UDP si nécessaire :

```dockerfile
# DNS server
EXPOSE 53/tcp  
EXPOSE 53/udp  

# Web + monitoring UDP
EXPOSE 80/tcp  
EXPOSE 8125/udp  
```

### EXPOSE avec des variables

Vous pouvez utiliser des variables, mais c'est déconseillé car peu pratique :

```dockerfile
# ⚠️ Fonctionne mais peu recommandé
ARG PORT=8000  
EXPOSE $PORT  

# ✅ Préférable : valeur fixe + ENV pour la config
EXPOSE 8000  
ENV PORT=8000  
```

### Bonnes pratiques avec EXPOSE

#### 1. Documenter tous les ports utilisés

```dockerfile
FROM my-app

# Web interface
EXPOSE 8080

# API
EXPOSE 8081

# Admin panel
EXPOSE 9000

# Health check endpoint
EXPOSE 9001
```

#### 2. Ajouter des commentaires explicatifs

```dockerfile
# Port principal de l'application web
EXPOSE 80

# Port HTTPS (si SSL activé)
EXPOSE 443

# Port de metrics Prometheus
EXPOSE 9090
```

#### 3. Utiliser les ports standards quand c'est possible

```dockerfile
# ✅ Ports standards reconnus
EXPOSE 80    # HTTP  
EXPOSE 443   # HTTPS  
EXPOSE 3306  # MySQL  
EXPOSE 5432  # PostgreSQL  
EXPOSE 6379  # Redis  

# ⚠️ Ports non standards (documenter pourquoi)
EXPOSE 8765  # Port personnalisé pour l'API legacy
```

#### 4. Cohérence avec la configuration de l'application

```dockerfile
FROM node:18-alpine  
WORKDIR /app  

# L'application écoute sur le port 3000
ENV PORT=3000

COPY . .  
RUN npm install  

# Documenter le port utilisé
EXPOSE 3000

CMD ["npm", "start"]
```

## L'instruction USER : gérer les utilisateurs

### Qu'est-ce que USER ?

`USER` définit l'**utilisateur** (et optionnellement le groupe) qui exécutera les instructions RUN, CMD et ENTRYPOINT suivantes.

💡 **Pourquoi c'est important** : Par défaut, les conteneurs s'exécutent en tant que **root**, ce qui pose des **problèmes de sécurité**. Utiliser un utilisateur non-privilégié est une **best practice essentielle**.

### Syntaxe

```dockerfile
USER <user>  
USER <user>:<group>  
USER <uid>:<gid>  
```

### Le problème de root

**Par défaut** (sans USER) :
```dockerfile
FROM ubuntu:22.04  
RUN whoami  # Affiche: root  
CMD ["whoami"]  
```

```bash
docker run mon-image  # Sortie: root
```

⚠️ **Risques de sécurité** :
- Si un attaquant compromet l'application, il a les droits root dans le conteneur
- Les fichiers créés appartiennent à root
- Violation du principe du moindre privilège

### Solution : créer et utiliser un utilisateur non-privilégié

```dockerfile
FROM ubuntu:22.04

# Créer un utilisateur et un groupe
RUN groupadd -r appgroup && \
    useradd -r -g appgroup appuser

# Basculer vers cet utilisateur
USER appuser

# Maintenant tout s'exécute en tant que appuser
RUN whoami  # Affiche: appuser  
CMD ["whoami"]  
```

### Exemple complet avec application

```dockerfile
FROM python:3.11-slim

# Créer l'utilisateur tôt
RUN useradd -m -u 1000 appuser

# Créer le répertoire et définir les permissions
WORKDIR /app  
RUN chown appuser:appuser /app  

# Basculer vers l'utilisateur non-privilégié
USER appuser

# Installation locale (dans le home de l'utilisateur)
COPY --chown=appuser:appuser requirements.txt .  
RUN pip install --user -r requirements.txt  

# Copier le code avec les bonnes permissions
COPY --chown=appuser:appuser . .

# L'application démarre en tant que appuser
CMD ["python", "app.py"]
```

### Quand basculer vers USER

**Ordre recommandé** :

1. Opérations nécessitant root (installation de packages)
2. Création de l'utilisateur
3. Configuration des permissions
4. Basculer vers USER
5. Opérations de l'utilisateur

```dockerfile
FROM node:18-alpine

# 1. Installation système (nécessite root)
RUN apk add --no-cache python3

# 2. Créer l'utilisateur
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

# 3. Configuration des permissions
RUN chown -R appuser:appgroup /app

# 4. Basculer vers l'utilisateur
USER appuser

# 5. Opérations utilisateur
COPY --chown=appuser:appgroup package*.json ./  
RUN npm ci  
COPY --chown=appuser:appgroup . .  

CMD ["node", "server.js"]
```

### USER avec UID/GID numériques

Pour éviter les conflits entre les systèmes :

```dockerfile
FROM ubuntu:22.04

# Créer avec UID/GID spécifiques
RUN groupadd -g 1000 appgroup && \
    useradd -u 1000 -g 1000 -m appuser

# Utiliser l'UID numérique
USER 1000:1000

CMD ["whoami"]
```

### Images avec utilisateur préconfiguré

Certaines images ont déjà un utilisateur non-root :

```dockerfile
# Node.js a un utilisateur "node"
FROM node:18-alpine  
USER node  
WORKDIR /home/node/app  

# PostgreSQL a un utilisateur "postgres"
FROM postgres:15  
USER postgres  
```

### Retour temporaire à root

Si vous devez redevenir root temporairement :

```dockerfile
FROM node:18-alpine

# Utiliser l'utilisateur node
USER node  
WORKDIR /app  

COPY --chown=node:node . .  
RUN npm install  

# Retour temporaire à root pour une installation
USER root  
RUN apk add --no-cache curl  

# Retour à l'utilisateur non-privilégié
USER node

CMD ["npm", "start"]
```

### Bonnes pratiques avec USER

#### 1. Toujours utiliser un utilisateur non-root en production

```dockerfile
# ❌ Dangereux - root par défaut
FROM ubuntu:22.04  
COPY app.py /app/  
CMD ["python", "/app/app.py"]  

# ✅ Sécurisé
FROM ubuntu:22.04  
RUN useradd -m appuser  
USER appuser  
COPY --chown=appuser:appuser app.py /home/appuser/  
CMD ["python", "/home/appuser/app.py"]  
```

#### 2. Créer l'utilisateur avec des UID/GID cohérents

```dockerfile
# ✅ UID/GID standard (1000) pour éviter les conflits
RUN groupadd -g 1000 appgroup && \
    useradd -m -u 1000 -g 1000 appuser
USER 1000:1000  
```

#### 3. Définir les permissions correctement

```dockerfile
# ✅ Bon - permissions explicites
WORKDIR /app  
RUN chown -R appuser:appuser /app  
USER appuser  

# ❌ Problème - l'utilisateur ne peut pas écrire
USER appuser  
WORKDIR /app  
COPY . .  # Appartient à root !  
```

#### 4. Utiliser --chown avec COPY

```dockerfile
# ✅ Bon - définit le propriétaire directement
USER appuser  
COPY --chown=appuser:appuser . /app  

# ❌ Nécessite étape supplémentaire
COPY . /app  
RUN chown -R appuser:appuser /app  
USER appuser  
```

## L'instruction VOLUME : points de montage

### Qu'est-ce que VOLUME ?

`VOLUME` crée un **point de montage** dans le conteneur et indique que ce répertoire devrait contenir des **données persistantes** gérées par Docker.

💡 **Analogie** : C'est comme réserver un emplacement de parking. L'emplacement existe, mais c'est au moment du démarrage qu'on décide quelle voiture (quelles données) y stationner.

### Pourquoi VOLUME ?

Par défaut, toutes les données dans un conteneur sont **éphémères** : elles disparaissent quand le conteneur est supprimé. Les volumes permettent de :

✅ **Persister les données** au-delà de la vie du conteneur  
✅ **Partager des données** entre conteneurs  
✅ **Sauvegarder** facilement les données importantes  
✅ **Améliorer les performances** pour les I/O intensifs

### Syntaxe

```dockerfile
VOLUME /chemin/dans/conteneur  
VOLUME ["/chemin1", "/chemin2"]  
```

### Exemples simples

```dockerfile
# Déclarer un volume pour les données
VOLUME /data

# Déclarer plusieurs volumes
VOLUME /var/log /var/cache

# Forme JSON
VOLUME ["/app/uploads", "/app/data"]
```

### Ce que VOLUME fait

Quand vous déclarez un VOLUME :

1. Docker **crée un répertoire** dans le conteneur
2. Docker **marque** ce répertoire comme point de montage
3. Au runtime, Docker peut **créer un volume** pour ce point de montage
4. Les données dans ce volume **persistent** après la suppression du conteneur

### Exemple avec base de données

```dockerfile
FROM postgres:15

# PostgreSQL stocke ses données dans /var/lib/postgresql/data
VOLUME /var/lib/postgresql/data

CMD ["postgres"]
```

**Utilisation** :
```bash
# Créer un conteneur avec volume anonyme (géré par Docker)
docker run --name ma-db postgres:15

# Ou avec volume nommé (recommandé)
docker run --name ma-db -v pg-data:/var/lib/postgresql/data postgres:15

# Les données persistent même si le conteneur est supprimé
docker rm ma-db  
docker run --name ma-db-2 -v pg-data:/var/lib/postgresql/data postgres:15  
# Les données sont toujours là !
```

### VOLUME vs docker run -v

**VOLUME dans le Dockerfile** :
- Déclare qu'un répertoire devrait être un volume
- Docker crée un volume anonyme si non spécifié au runtime
- **Documentation** : indique aux utilisateurs quels répertoires sont importants

**-v au runtime** :
- **Contrôle** explicite du volume utilisé
- Permet les volumes nommés ou bind mounts
- **Plus flexible**

**Recommandation** : Utilisez VOLUME dans le Dockerfile pour documenter, et `-v` au runtime pour le contrôle précis.

### Exemples d'applications

**Application avec uploads** :
```dockerfile
FROM node:18-alpine  
WORKDIR /app  

COPY package*.json ./  
RUN npm install  

COPY . .

# Les uploads des utilisateurs persistent
VOLUME /app/uploads

EXPOSE 3000  
CMD ["npm", "start"]  
```

**Application avec cache** :
```dockerfile
FROM python:3.11-slim  
WORKDIR /app  

COPY requirements.txt .  
RUN pip install -r requirements.txt  

COPY . .

# Cache de l'application
VOLUME /app/cache

# Logs de l'application
VOLUME /app/logs

CMD ["python", "app.py"]
```

**Application avec plusieurs volumes** :
```dockerfile
FROM custom-app

# Configuration (peut être montée en lecture seule)
VOLUME /etc/app

# Données utilisateur
VOLUME /var/app/data

# Logs
VOLUME /var/log/app

# Cache temporaire
VOLUME /tmp/app-cache
```

### Volumes anonymes vs volumes nommés

**Volume anonyme** (créé par Docker) :
```bash
docker run mon-app
# Docker crée un volume avec un nom aléatoire
# Exemple: a1b2c3d4e5f6...
```

**Volume nommé** (recommandé) :
```bash
docker run -v app-data:/app/data mon-app
# Volume nommé "app-data", facile à identifier
```

**Bind mount** (développement) :
```bash
docker run -v $(pwd)/data:/app/data mon-app
# Monte un répertoire de votre machine directement
```

### Limitations de VOLUME

⚠️ **Important** : Une fois qu'un volume est déclaré dans le Dockerfile, vous **ne pouvez pas** :
- Le "dé-déclarer" dans une image enfant
- Changer son chemin
- Empêcher sa création

```dockerfile
FROM parent-image
# Si parent-image a "VOLUME /data"
# Vous ne pouvez PAS annuler cette déclaration
```

### Cas d'usage courants

**Bases de données** :
```dockerfile
FROM mysql:8.0  
VOLUME /var/lib/mysql  
```

**Applications web avec uploads** :
```dockerfile
FROM nginx:alpine  
VOLUME /usr/share/nginx/html/uploads  
```

**Logs d'application** :
```dockerfile
FROM app-image  
VOLUME /var/log/app  
```

**Configuration dynamique** :
```dockerfile
FROM app-image  
VOLUME /etc/app/config  
```

### Bonnes pratiques avec VOLUME

#### 1. Documenter les volumes importants

```dockerfile
# Répertoires avec données persistantes :
# /app/data    - Données de l'application
# /app/uploads - Fichiers uploadés par les utilisateurs
# /app/logs    - Logs de l'application

VOLUME /app/data  
VOLUME /app/uploads  
VOLUME /app/logs  
```

#### 2. Ne pas utiliser VOLUME pour le code

```dockerfile
# ❌ Mauvais - le code devrait être dans l'image
VOLUME /app/src

# ✅ Bon - seulement les données qui changent
VOLUME /app/data
```

#### 3. Utiliser des volumes nommés en production

```bash
# ✅ Production - volume nommé
docker run -v prod-data:/app/data mon-app

# ⚠️ Développement - bind mount
docker run -v $(pwd)/data:/app/data mon-app
```

#### 4. Combiner VOLUME avec USER

```dockerfile
FROM node:18-alpine

RUN adduser -D appuser

WORKDIR /app  
RUN chown appuser:appuser /app  

# Créer le répertoire avec les bonnes permissions
RUN mkdir -p /app/data && chown appuser:appuser /app/data

USER appuser

VOLUME /app/data

CMD ["npm", "start"]
```

## Exemple complet : Application production-ready

Voici un Dockerfile qui utilise toutes les instructions avancées :

```dockerfile
# Image de base
FROM python:3.11-slim

# Variables d'environnement
ENV APP_HOME=/app \
    APP_USER=appuser \
    APP_PORT=8000 \
    PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1

# Métadonnées
LABEL maintainer="votre-email@example.com" \
      version="1.0" \
      description="Application Python production-ready"

# Installation des dépendances système (en tant que root)
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Créer l'utilisateur non-root
RUN useradd -m -u 1000 -s /bin/bash $APP_USER

# Définir le répertoire de travail
WORKDIR $APP_HOME

# Créer les répertoires nécessaires avec les bonnes permissions
RUN mkdir -p logs data uploads && \
    chown -R $APP_USER:$APP_USER $APP_HOME

# Basculer vers l'utilisateur non-root
USER $APP_USER

# Copier et installer les dépendances Python
COPY --chown=$APP_USER:$APP_USER requirements.txt .  
RUN pip install --user --no-cache-dir -r requirements.txt  

# Copier le code de l'application
COPY --chown=$APP_USER:$APP_USER . .

# Déclarer les volumes pour les données persistantes
VOLUME $APP_HOME/data  
VOLUME $APP_HOME/logs  
VOLUME $APP_HOME/uploads  

# Exposer le port de l'application
EXPOSE $APP_PORT

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
    CMD curl -f http://localhost:$APP_PORT/health || exit 1

# Commande de démarrage
CMD ["python", "app.py"]
```

**Utilisation** :
```bash
# Build
docker build -t mon-app:1.0 .

# Run avec volumes nommés
docker run -d \
  --name mon-app \
  -p 8000:8000 \
  -v app-data:/app/data \
  -v app-logs:/app/logs \
  -v app-uploads:/app/uploads \
  -e DATABASE_URL=postgresql://... \
  mon-app:1.0
```

## Récapitulatif et comparaison

### Tableau récapitulatif

| Instruction | Rôle | Quand l'utiliser | Persistance |
|-------------|------|------------------|-------------|
| **ENV** | Variables d'environnement | Configuration, paramètres | ✅ Build + Runtime |
| **WORKDIR** | Répertoire de travail | Organisation, chemins | ✅ Build + Runtime |
| **EXPOSE** | Documentation ports | Toute application réseau | 📝 Documentation |
| **USER** | Utilisateur d'exécution | **Toujours** (sécurité) | ✅ Build + Runtime |
| **VOLUME** | Point de montage | Données persistantes | 📝 Documentation + 🔧 Hint |

### Ordre recommandé dans un Dockerfile

```dockerfile
# 1. Image de base
FROM ...

# 2. Métadonnées (optionnel)
LABEL ...

# 3. Variables d'environnement
ENV ...

# 4. Installation packages (root)
RUN apt-get update && ...

# 5. Création utilisateur
RUN useradd ...

# 6. Répertoire de travail
WORKDIR ...

# 7. Permissions
RUN chown ...

# 8. Basculer vers USER
USER ...

# 9. Copier et installer dépendances
COPY requirements.txt .  
RUN pip install ...  

# 10. Copier le code
COPY . .

# 11. Volumes
VOLUME ...

# 12. Ports
EXPOSE ...

# 13. Health check (optionnel)
HEALTHCHECK ...

# 14. Commande de démarrage
CMD [...]
```

## Résumé

Dans cette section, vous avez maîtrisé les **cinq instructions avancées** :

- ✅ **ENV** : Gérer la configuration via variables d'environnement
- ✅ **WORKDIR** : Organiser l'arborescence des fichiers
- ✅ **EXPOSE** : Documenter les ports réseau utilisés
- ✅ **USER** : Sécuriser vos conteneurs avec un utilisateur non-root
- ✅ **VOLUME** : Gérer la persistance des données

**Points clés à retenir** :

🔑 **ENV** : Configuration flexible sans rebuild  
🔑 **WORKDIR** : Toujours préférer à `RUN cd`  
🔑 **EXPOSE** : Documentation, ne publie pas automatiquement  
🔑 **USER** : Essentiel pour la sécurité en production  
🔑 **VOLUME** : Indique les données qui doivent persister

Avec ces instructions, vous êtes maintenant capable de créer des images Docker **professionnelles, sécurisées et production-ready** !

Dans la section suivante, vous apprendrez à construire ces images efficacement et à gérer leur versionnement avec les tags.

⏭️ [Construction d'images (build, tag)](/05-creation-dimages-personnalisees/04-construction-dimages.md)
