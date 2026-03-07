🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 10.5 Nommage et organisation des ressources

## Introduction

Imaginez un projet avec des dizaines de conteneurs, d'images, de volumes et de réseaux. Sans une stratégie de nommage et d'organisation cohérente, il devient rapidement difficile de s'y retrouver. Vous vous demandez : "Quel conteneur fait quoi ?", "Cette image est-elle pour le développement ou la production ?", "Puis-je supprimer ce volume ?".

Une bonne organisation et des conventions de nommage claires sont essentielles pour :
- **Comprendre rapidement** ce que fait chaque ressource
- **Collaborer efficacement** avec votre équipe
- **Maintenir facilement** vos projets sur le long terme
- **Éviter les erreurs** en production

Dans cette section, nous allons explorer les meilleures pratiques pour nommer et organiser vos ressources Docker.

## Pourquoi le nommage est important ?

### Sans convention de nommage

```bash
docker ps
# CONTAINER ID   NAME               IMAGE
# abc123         hungry_darwin      node:18
# def456         amazing_tesla      postgres
# ghi789         clever_euler       redis
```

**Problèmes :**
- Noms aléatoires générés par Docker (hungry_darwin, amazing_tesla...)
- Impossible de savoir ce que fait chaque conteneur
- Difficile de retrouver un conteneur spécifique
- Risque de supprimer le mauvais conteneur

### Avec convention de nommage

```bash
docker ps
# CONTAINER ID   NAME                    IMAGE
# abc123         myapp-web-prod          myapp:1.2.3
# def456         myapp-db-prod           postgres:15
# ghi789         myapp-cache-prod        redis:7-alpine
```

**Avantages :**
- Noms explicites et descriptifs
- Identification immédiate du rôle de chaque conteneur
- Facilité de maintenance et de débogage
- Communication claire en équipe

## Conventions de nommage pour les conteneurs

### Structure recommandée

```
[projet]-[service]-[environnement]-[instance]
```

**Composants :**
- **projet** : Nom du projet ou de l'application
- **service** : Rôle du conteneur (web, api, db, cache, worker...)
- **environnement** : dev, staging, prod
- **instance** (optionnel) : Numéro d'instance pour le scaling (1, 2, 3...)

### Exemples de bons noms

```bash
# Application e-commerce
ecommerce-web-prod  
ecommerce-api-prod  
ecommerce-db-prod  
ecommerce-cache-prod  
ecommerce-worker-prod-1  
ecommerce-worker-prod-2  

# Blog personnel
blog-frontend-dev  
blog-backend-dev  
blog-database-dev  

# Projet d'analyse de données
analytics-jupyter-dev  
analytics-postgres-dev  
analytics-redis-dev  
```

### Règles de nommage

1. **Utiliser des minuscules** : `myapp-web` ✅ pas `MyApp-Web` ❌
2. **Utiliser des tirets** : `myapp-web` ✅ pas `myapp_web` ou `myappweb` ❌
3. **Être descriptif** : `payment-api` ✅ pas `api1` ❌
4. **Rester concis** : `auth-service` ✅ pas `authentication-microservice-backend` ❌
5. **Éviter les caractères spéciaux** : Seulement lettres, chiffres et tirets

### Application avec Docker

```bash
# Lancer un conteneur avec un nom explicite
docker run -d --name ecommerce-web-prod nginx

# Avec Docker Compose (les noms sont générés automatiquement)
# Mais vous pouvez les personnaliser avec container_name
```

## Conventions de nommage pour les images

### Structure des tags d'images

```
[registry]/[organisation]/[nom-image]:[tag]
```

**Exemples :**
```
docker.io/library/nginx:1.27-alpine  
ghcr.io/mycompany/myapp:1.2.3  
registry.example.com/frontend/web-app:latest  
```

### Stratégies de tagging

#### 1. Versioning sémantique (recommandé pour la production)

```bash
# Format: major.minor.patch
myapp:1.0.0  
myapp:1.0.1  
myapp:1.2.0  
myapp:2.0.0  

# Avec tags multiples
docker tag myapp:1.2.3 myapp:1.2  
docker tag myapp:1.2.3 myapp:1  
docker tag myapp:1.2.3 myapp:latest  
```

**Avantages :**
- Traçabilité des versions
- Rollback facile vers version précédente
- Compatible avec les outils d'orchestration

#### 2. Tags par environnement

```bash
myapp:dev  
myapp:staging  
myapp:prod  
```

**Utilisation :**
- Développement rapide
- Tests multiples
- Déploiement par environnement

#### 3. Tags avec commit SHA (CI/CD)

```bash
myapp:abc123f  
myapp:1.2.3-abc123f  
myapp:main-abc123f  
```

**Avantages :**
- Traçabilité exacte du code source
- Facilite le débogage
- Idéal pour les pipelines CI/CD

#### 4. Tags avec date

```bash
myapp:2024-01-15  
myapp:20240115-142030  
```

**Utilisation :**
- Builds nocturnes (nightly builds)
- Snapshots de développement

### Bonnes pratiques pour les tags

```bash
# ✅ BON : Tag spécifique en production
docker pull myapp:1.2.3

# ⚠️ ÉVITER : latest en production (version non garantie)
docker pull myapp:latest

# ✅ BON : Multiples tags pour la même image
docker tag myapp:1.2.3 myapp:1.2  
docker tag myapp:1.2.3 myapp:1  
docker tag myapp:1.2.3 myapp:latest  

# ✅ BON : Nom d'image descriptif
company/payment-service:1.2.3

# ❌ MAUVAIS : Nom générique
company/app1:v1
```

### Exemples de stratégie complète

```bash
# Image de développement
myapp:dev-20240115  
myapp:dev-latest  

# Image de staging
myapp:staging-1.2.3-rc1  
myapp:staging-latest  

# Image de production
myapp:1.2.3  
myapp:1.2  
myapp:1  
myapp:latest  
myapp:stable  
```

## Conventions de nommage pour les volumes

### Structure recommandée

```
[projet]-[service]-[type]-[environnement]
```

**Types courants :**
- `data` : Données persistantes
- `config` : Fichiers de configuration
- `logs` : Fichiers de logs
- `uploads` : Fichiers téléchargés
- `cache` : Données en cache

### Exemples de bons noms

```bash
# Volumes pour une application e-commerce
ecommerce-db-data-prod  
ecommerce-web-uploads-prod  
ecommerce-api-logs-prod  
ecommerce-redis-data-prod  

# Volumes pour un blog
blog-postgres-data-dev  
blog-media-uploads-dev  
blog-nginx-config-dev  
```

### Application avec Docker

```bash
# Créer un volume avec un nom explicite
docker volume create ecommerce-db-data-prod

# Lister les volumes
docker volume ls

# Inspecter un volume
docker volume inspect ecommerce-db-data-prod
```

### Avec Docker Compose

```yaml
services:
  db:
    image: postgres:15
    volumes:
      - ecommerce-db-data-prod:/var/lib/postgresql/data

  web:
    image: myapp:latest
    volumes:
      - ecommerce-web-uploads-prod:/app/uploads
      - ecommerce-web-logs-prod:/app/logs

volumes:
  ecommerce-db-data-prod:
  ecommerce-web-uploads-prod:
  ecommerce-web-logs-prod:
```

## Conventions de nommage pour les réseaux

### Structure recommandée

```
[projet]-[type]-[environnement]
```

**Types courants :**
- `frontend` : Réseau pour les services frontend
- `backend` : Réseau pour les services backend
- `internal` : Réseau interne isolé
- `external` : Réseau exposé à l'extérieur
- `database` : Réseau pour les bases de données

### Exemples de bons noms

```bash
# Réseaux pour une application
myapp-frontend-prod  
myapp-backend-prod  
myapp-database-prod  

# Architecture microservices
services-api-gateway-prod  
services-internal-prod  
services-monitoring-prod  
```

### Application avec Docker

```bash
# Créer un réseau avec un nom explicite
docker network create myapp-backend-prod

# Lister les réseaux
docker network ls

# Connecter un conteneur à un réseau
docker network connect myapp-backend-prod myapp-api-prod
```

### Avec Docker Compose

```yaml
services:
  web:
    image: myapp-frontend:latest
    networks:
      - myapp-frontend-prod
      - myapp-backend-prod

  api:
    image: myapp-api:latest
    networks:
      - myapp-backend-prod
      - myapp-database-prod

  db:
    image: postgres:15
    networks:
      - myapp-database-prod

networks:
  myapp-frontend-prod:
    driver: bridge
  myapp-backend-prod:
    driver: bridge
  myapp-database-prod:
    driver: bridge
    internal: true  # Isolé de l'extérieur
```

## Organisation des fichiers de projet

### Structure de projet recommandée

```
mon-projet/
├── .dockerignore
├── .gitignore
├── .env.example
├── README.md
├── docker-compose.yml
├── docker-compose.dev.yml
├── docker-compose.prod.yml
├── docker/
│   ├── nginx/
│   │   ├── Dockerfile
│   │   └── nginx.conf
│   ├── backend/
│   │   ├── Dockerfile
│   │   ├── Dockerfile.dev
│   │   └── requirements.txt
│   └── frontend/
│       ├── Dockerfile
│       ├── Dockerfile.dev
│       └── package.json
├── scripts/
│   ├── build.sh
│   ├── deploy.sh
│   └── cleanup.sh
├── config/
│   ├── nginx/
│   ├── postgres/
│   └── redis/
├── backend/
│   ├── src/
│   ├── tests/
│   └── ...
└── frontend/
    ├── src/
    ├── public/
    └── ...
```

### Alternative pour projets simples

```
mon-projet-simple/
├── Dockerfile
├── docker-compose.yml
├── .dockerignore
├── .gitignore
├── .env.example
├── README.md
├── src/
├── tests/
└── config/
```

### Fichiers essentiels

#### README.md

````markdown
# Mon Projet

## Description
Application web avec architecture microservices

## Prérequis
- Docker 24.0+
- Docker Compose 2.20+

## Installation

### Développement
```bash
cp .env.example .env  
docker compose -f docker-compose.dev.yml up  
```

### Production
```bash
docker compose -f docker-compose.prod.yml up -d
```

## Services
- **web** : Frontend React (port 3000)
- **api** : Backend Node.js (port 8000)
- **db** : PostgreSQL 15 (port 5432)
- **cache** : Redis 7 (port 6379)

## Commandes utiles
```bash
# Voir les logs
docker compose logs -f

# Arrêter tous les services
docker compose down

# Reconstruire les images
docker compose build --no-cache
```

## Variables d'environnement
Voir `.env.example` pour la liste complète
````

#### .dockerignore

```
# Git
.git
.gitignore
.gitattributes

# Documentation
README.md  
CHANGELOG.md  
docs/  

# CI/CD
.github/
.gitlab-ci.yml
.circleci/

# Dépendances
node_modules/  
venv/  
__pycache__/
*.pyc

# Environnement
.env
.env.local
.env.*.local

# IDE
.vscode/
.idea/
*.swp
*.swo

# Build
dist/  
build/  
target/  
*.log

# Tests
coverage/
.pytest_cache/
*.test

# OS
.DS_Store
Thumbs.db
```

#### .gitignore

```
# Environnement
.env
.env.local
.env.*.local

# Secrets
secrets/
*.pem
*.key

# Docker
.docker/

# Dépendances
node_modules/  
venv/  
__pycache__/

# Build
dist/  
build/  

# Logs
logs/
*.log

# IDE
.vscode/
.idea/

# OS
.DS_Store
```

## Organisation avec Docker Compose

### Fichier docker-compose.yml de base

```yaml
# Définir les services dans un ordre logique
services:
  # 1. Base de données
  database:
    image: postgres:15-alpine
    container_name: myapp-db-prod
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - myapp-db-data:/var/lib/postgresql/data
    networks:
      - myapp-backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  # 2. Cache
  cache:
    image: redis:7-alpine
    container_name: myapp-cache-prod
    restart: unless-stopped
    volumes:
      - myapp-cache-data:/data
    networks:
      - myapp-backend
    command: redis-server --appendonly yes

  # 3. Backend API
  api:
    build:
      context: ./backend
      dockerfile: Dockerfile
    image: myapp-api:${VERSION:-latest}
    container_name: myapp-api-prod
    restart: unless-stopped
    depends_on:
      database:
        condition: service_healthy
      cache:
        condition: service_started
    environment:
      NODE_ENV: production
      DATABASE_URL: postgresql://${DB_USER}:${DB_PASSWORD}@database:5432/${DB_NAME}
      REDIS_URL: redis://cache:6379
    networks:
      - myapp-backend
      - myapp-frontend
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:8000/health"]
      interval: 30s
      timeout: 5s
      retries: 3

  # 4. Frontend
  web:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    image: myapp-web:${VERSION:-latest}
    container_name: myapp-web-prod
    restart: unless-stopped
    depends_on:
      - api
    ports:
      - "80:80"
    networks:
      - myapp-frontend

# Définir les volumes
volumes:
  myapp-db-data:
    name: myapp-db-data-prod
  myapp-cache-data:
    name: myapp-cache-data-prod

# Définir les réseaux
networks:
  myapp-backend:
    name: myapp-backend-prod
    driver: bridge
  myapp-frontend:
    name: myapp-frontend-prod
    driver: bridge
```

### Séparer les environnements

#### docker-compose.dev.yml

```yaml
services:
  database:
    ports:
      - "5432:5432"  # Exposer pour accès direct
    environment:
      POSTGRES_DB: myapp_dev

  cache:
    ports:
      - "6379:6379"  # Exposer pour accès direct

  api:
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
    volumes:
      - ./backend:/app  # Hot reload
      - /app/node_modules
    environment:
      NODE_ENV: development
      LOG_LEVEL: debug
    command: npm run dev

  web:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    volumes:
      - ./frontend:/app  # Hot reload
      - /app/node_modules
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: development
```

#### docker-compose.prod.yml

```yaml
services:
  api:
    image: registry.example.com/myapp-api:${VERSION}
    deploy:
      replicas: 3
      restart_policy:
        condition: on-failure
        max_attempts: 3
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M

  web:
    image: registry.example.com/myapp-web:${VERSION}
    deploy:
      replicas: 2
```

### Utilisation

```bash
# Développement
docker compose -f docker-compose.yml -f docker-compose.dev.yml up

# Production
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

## Utilisation des labels

Les labels permettent d'ajouter des métadonnées à vos ressources Docker.

### Labels dans un Dockerfile

```dockerfile
FROM node:18-alpine

LABEL maintainer="dev@example.com"  
LABEL version="1.2.3"  
LABEL description="API Backend pour Mon Application"  
LABEL project="myapp"  
LABEL environment="production"  
LABEL com.example.vendor="My Company"  
LABEL com.example.release-date="2024-01-15"  
LABEL com.example.version.major="1"  
LABEL com.example.version.minor="2"  
LABEL com.example.version.patch="3"  

WORKDIR /app  
COPY package*.json ./  
RUN npm ci --omit=dev  
COPY . .  

CMD ["node", "server.js"]
```

### Labels avec Docker Compose

```yaml
services:
  api:
    image: myapp-api:latest
    labels:
      - "com.example.project=myapp"
      - "com.example.service=api"
      - "com.example.environment=production"
      - "com.example.team=backend"
      - "com.example.version=1.2.3"
    networks:
      - backend

  web:
    image: myapp-web:latest
    labels:
      - "com.example.project=myapp"
      - "com.example.service=web"
      - "com.example.environment=production"
      - "com.example.team=frontend"
      - "traefik.enable=true"
      - "traefik.http.routers.web.rule=Host(`example.com`)"
```

### Utiliser les labels pour filtrer

```bash
# Lister tous les conteneurs d'un projet
docker ps --filter "label=com.example.project=myapp"

# Lister tous les conteneurs d'un environnement
docker ps --filter "label=com.example.environment=production"

# Arrêter tous les conteneurs d'un service
docker stop $(docker ps -q --filter "label=com.example.service=api")

# Supprimer toutes les images de développement
docker rmi $(docker images -q --filter "label=com.example.environment=dev")
```

## Scripts d'automatisation

### build.sh

```bash
#!/bin/bash

# Script de build des images Docker

set -e

VERSION=${1:-latest}  
REGISTRY="registry.example.com"  
PROJECT="myapp"  

echo "Building images version: $VERSION"

# Build backend
echo "Building backend..."  
docker build \  
  -t ${REGISTRY}/${PROJECT}-api:${VERSION} \
  -t ${REGISTRY}/${PROJECT}-api:latest \
  --label "version=${VERSION}" \
  --label "build-date=$(date -u +'%Y-%m-%dT%H:%M:%SZ')" \
  ./backend

# Build frontend
echo "Building frontend..."  
docker build \  
  -t ${REGISTRY}/${PROJECT}-web:${VERSION} \
  -t ${REGISTRY}/${PROJECT}-web:latest \
  --label "version=${VERSION}" \
  --label "build-date=$(date -u +'%Y-%m-%dT%H:%M:%SZ')" \
  ./frontend

echo "Build completed successfully!"
```

### deploy.sh

```bash
#!/bin/bash

# Script de déploiement

set -e

ENVIRONMENT=${1:-staging}  
VERSION=${2:-latest}  

echo "Deploying to ${ENVIRONMENT} with version ${VERSION}"

# Exporter la version
export VERSION=${VERSION}

# Déployer selon l'environnement
case ${ENVIRONMENT} in
  dev)
    docker compose -f docker-compose.yml -f docker-compose.dev.yml up -d
    ;;
  staging)
    docker compose -f docker-compose.yml -f docker-compose.staging.yml up -d
    ;;
  prod)
    docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
    ;;
  *)
    echo "Environment inconnu: ${ENVIRONMENT}"
    exit 1
    ;;
esac

echo "Deployment completed!"
```

### cleanup.sh

```bash
#!/bin/bash

# Script de nettoyage des ressources Docker

set -e

echo "Nettoyage des ressources Docker..."

# Arrêter tous les conteneurs du projet
echo "Arrêt des conteneurs..."  
docker compose down  

# Supprimer les images non utilisées
echo "Suppression des images non utilisées..."  
docker image prune -f --filter "label=com.example.project=myapp"  

# Supprimer les volumes orphelins
echo "Suppression des volumes orphelins..."  
docker volume prune -f  

# Supprimer les réseaux non utilisés
echo "Suppression des réseaux non utilisés..."  
docker network prune -f  

# Afficher l'espace libéré
echo "Espace disque libéré!"  
docker system df  

echo "Nettoyage terminé!"
```

Rendre les scripts exécutables :

```bash
chmod +x scripts/*.sh
```

## Documentation dans le code

### Commenter les Dockerfiles

```dockerfile
# Image de base officielle Node.js LTS sur Alpine Linux
FROM node:18-alpine

# Métadonnées de l'image
LABEL maintainer="dev@example.com"  
LABEL version="1.2.3"  
LABEL description="Backend API pour l'application MyApp"  

# Installer les dépendances système nécessaires
# - python3 : requis pour certains packages npm
# - make, g++ : requis pour compiler les dépendances natives
RUN apk add --no-cache python3 make g++

# Définir le répertoire de travail
WORKDIR /app

# Copier les fichiers de dépendances en premier
# Cela permet de tirer parti du cache Docker lors des builds
COPY package*.json ./

# Installer les dépendances de production uniquement
# --no-cache-dir : évite de stocker le cache npm
RUN npm ci --omit=dev && \
    npm cache clean --force

# Copier le code source de l'application
COPY . .

# Créer un utilisateur non-root pour des raisons de sécurité
RUN adduser -D appuser && \
    chown -R appuser:appuser /app

# Changer vers l'utilisateur non-root
USER appuser

# Exposer le port de l'application (documentation uniquement)
EXPOSE 8000

# Health check pour vérifier que l'application fonctionne
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node healthcheck.js || exit 1

# Commande de démarrage de l'application
CMD ["node", "server.js"]
```

### Commenter Docker Compose

```yaml
# Configuration de l'application MyApp
# Architecture: Frontend React + Backend Node.js + PostgreSQL + Redis

services:
  # Base de données PostgreSQL
  # Port interne : 5432
  # Données persistantes dans le volume myapp-db-data
  database:
    image: postgres:15-alpine
    container_name: myapp-db-prod
    restart: unless-stopped
    environment:
      # Variables de configuration de la base de données
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      # Volume pour la persistance des données
      - myapp-db-data:/var/lib/postgresql/data
      # Scripts d'initialisation (optionnel)
      - ./config/postgres/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    networks:
      - backend
    healthcheck:
      # Vérifier que PostgreSQL est prêt à accepter des connexions
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Cache Redis pour les sessions et le cache applicatif
  cache:
    image: redis:7-alpine
    container_name: myapp-cache-prod
    restart: unless-stopped
    volumes:
      - myapp-cache-data:/data
    networks:
      - backend
    command: redis-server --appendonly yes  # Activer la persistance

# Volumes nommés pour la persistance des données
volumes:
  myapp-db-data:
    name: myapp-db-data-prod
    # Ce volume contient toutes les données de PostgreSQL
  myapp-cache-data:
    name: myapp-cache-data-prod
    # Ce volume contient les données Redis persistées

# Réseaux pour isoler les services
networks:
  backend:
    name: myapp-backend-prod
    driver: bridge
    # Réseau interne pour la communication backend
  frontend:
    name: myapp-frontend-prod
    driver: bridge
    # Réseau pour exposer les services frontend
```

## Checklist d'organisation

Avant de partager ou déployer votre projet, vérifiez :

### Nommage
- [ ] Noms de conteneurs explicites et cohérents
- [ ] Tags d'images avec versioning sémantique
- [ ] Noms de volumes descriptifs
- [ ] Noms de réseaux clairs
- [ ] Conventions de nommage documentées

### Structure de fichiers
- [ ] README.md complet et à jour
- [ ] .dockerignore configuré
- [ ] .gitignore configuré
- [ ] .env.example fourni (sans secrets)
- [ ] Dockerfiles organisés dans des dossiers dédiés
- [ ] Séparation dev/staging/prod

### Docker Compose
- [ ] Services organisés dans un ordre logique
- [ ] Labels ajoutés pour le filtrage
- [ ] Health checks configurés
- [ ] Dépendances entre services définies
- [ ] Limites de ressources définies (production)
- [ ] Commentaires pour expliquer les choix

### Documentation
- [ ] Instructions de démarrage claires
- [ ] Variables d'environnement documentées
- [ ] Architecture décrite
- [ ] Commandes utiles listées
- [ ] Dockerfiles commentés
- [ ] Docker Compose commenté

### Automatisation
- [ ] Scripts de build
- [ ] Scripts de déploiement
- [ ] Scripts de nettoyage
- [ ] Scripts de sauvegarde (si nécessaire)

## Exemples de projets bien organisés

### Projet simple - Blog

```
blog/
├── README.md
├── .dockerignore
├── .gitignore
├── .env.example
├── Dockerfile
├── docker-compose.yml
├── docker-compose.dev.yml
├── package.json
├── src/
│   ├── server.js
│   ├── routes/
│   └── models/
└── public/
    └── ...
```

### Projet microservices

```
ecommerce-platform/
├── README.md
├── .gitignore
├── docker-compose.yml
├── docker-compose.dev.yml
├── docker-compose.prod.yml
├── .env.example
│
├── services/
│   ├── frontend/
│   │   ├── Dockerfile
│   │   ├── Dockerfile.dev
│   │   ├── package.json
│   │   └── src/
│   │
│   ├── api-gateway/
│   │   ├── Dockerfile
│   │   ├── package.json
│   │   └── src/
│   │
│   ├── auth-service/
│   │   ├── Dockerfile
│   │   ├── requirements.txt
│   │   └── app/
│   │
│   ├── product-service/
│   │   ├── Dockerfile
│   │   └── ...
│   │
│   └── payment-service/
│       ├── Dockerfile
│       └── ...
│
├── infrastructure/
│   ├── nginx/
│   │   ├── Dockerfile
│   │   └── nginx.conf
│   ├── postgres/
│   │   └── init.sql
│   └── monitoring/
│       ├── prometheus.yml
│       └── grafana/
│
└── scripts/
    ├── build-all.sh
    ├── deploy.sh
    ├── test.sh
    └── cleanup.sh
```

## Outils pour faciliter l'organisation

### Docker Compose Override

Permet de surcharger des configurations sans dupliquer :

```yaml
# docker-compose.yml (base)
services:
  app:
    image: myapp:latest
    environment:
      NODE_ENV: production
```

```yaml
# docker-compose.override.yml (automatiquement chargé en dev)
services:
  app:
    build: .
    volumes:
      - .:/app
    environment:
      NODE_ENV: development
```

### Make pour simplifier les commandes

```makefile
# Makefile
.PHONY: help build up down logs

help:
	@echo "Commandes disponibles:"
	@echo "  make build    - Construire les images"
	@echo "  make up       - Démarrer les services"
	@echo "  make down     - Arrêter les services"
	@echo "  make logs     - Voir les logs"

build:
	docker compose build

up:
	docker compose up -d

down:
	docker compose down

logs:
	docker compose logs -f

clean:
	docker compose down -v
	docker system prune -f
```

Utilisation :
```bash
make build  
make up  
make logs  
```

## Conclusion

Une bonne organisation et des conventions de nommage cohérentes sont essentielles pour :

1. **La clarté** : Comprendre rapidement l'architecture et le rôle de chaque composant
2. **La maintenance** : Faciliter les modifications et les mises à jour
3. **La collaboration** : Permettre à l'équipe de travailler efficacement ensemble
4. **La fiabilité** : Réduire les erreurs en production
5. **L'évolutivité** : Faciliter l'ajout de nouveaux services

**Principes clés à retenir :**
- Noms explicites et descriptifs pour toutes les ressources
- Structure de fichiers logique et cohérente
- Documentation complète et à jour
- Séparation claire des environnements
- Utilisation de labels pour les métadonnées
- Scripts d'automatisation pour les tâches répétitives

En appliquant ces bonnes pratiques dès le début de votre projet, vous gagnerez un temps considérable lors de la maintenance et de l'évolution de vos applications Docker.

## Ressources supplémentaires

- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Semantic Versioning](https://semver.org/)
- [12-Factor App](https://12factor.net/)
- [Docker Compose Best Practices](https://docs.docker.com/compose/compose-file/)

⏭️ [Sujets intermédiaires](/11-sujets-intermediaires/README.md)
