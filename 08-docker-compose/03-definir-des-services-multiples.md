🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 8.3 Définir des services multiples

## Introduction

La vraie puissance de Docker Compose se révèle lorsque vous travaillez avec plusieurs services qui doivent communiquer entre eux. Dans cette section, nous allons apprendre à orchestrer plusieurs conteneurs pour créer des applications complètes et fonctionnelles.

Une application moderne se compose rarement d'un seul composant. Généralement, vous aurez besoin de :
- Un frontend (interface utilisateur)
- Un backend (API, logique métier)
- Une base de données
- Peut-être un système de cache, une file de messages, etc.

Docker Compose rend la gestion de tous ces composants simple et élégante ! 🚀

## Comprendre la communication entre services

### Le réseau par défaut

Quand vous définissez plusieurs services dans un fichier `docker-compose.yml`, Docker Compose crée automatiquement un réseau virtuel privé. Tous les services sur ce réseau peuvent se parler en utilisant leur nom de service comme nom d'hôte.

**Exemple simple** :

```yaml
services:
  frontend:
    image: nginx

  backend:
    image: node:18

  database:
    image: postgres:15
```

Dans cet exemple :
- `frontend` peut contacter `backend` en utilisant l'URL `http://backend:port`
- `backend` peut contacter `database` en utilisant le nom d'hôte `database`
- Tout cela fonctionne automatiquement, sans configuration supplémentaire !

### La résolution DNS automatique

Docker Compose agit comme un serveur DNS interne. Quand un service veut contacter un autre service nommé `database`, Docker traduit automatiquement ce nom en l'adresse IP du conteneur correspondant.

```yaml
services:
  app:
    image: mon-app
    environment:
      # Utilisez le nom du service comme nom d'hôte
      DATABASE_HOST: database
      DATABASE_PORT: 5432

  database:
    image: postgres:15
```

## Exemple 1 : Application web simple avec base de données

Commençons par un exemple classique : une application web qui se connecte à une base de données PostgreSQL.

```yaml
services:
  # Base de données PostgreSQL
  database:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: motdepasse123
      POSTGRES_DB: ma_base
    volumes:
      - donnees-postgres:/var/lib/postgresql/data
    # Pas besoin d'exposer les ports si seulement l'app y accède

  # Application web
  web:
    image: mon-application:latest
    ports:
      - "8080:8080"
    environment:
      # Connexion à la base de données en utilisant le nom du service
      DATABASE_HOST: database
      DATABASE_PORT: 5432
      DATABASE_NAME: ma_base
      DATABASE_USER: admin
      DATABASE_PASSWORD: motdepasse123
    depends_on:
      - database

volumes:
  donnees-postgres:
```

**Points clés à noter** :
- Le service `web` utilise `database` comme nom d'hôte pour se connecter
- La base de données n'expose pas de ports à l'extérieur (plus sécurisé)
- Seul le service `web` expose le port 8080 vers l'extérieur
- `depends_on` s'assure que la base démarre avant l'application

## Exemple 2 : Stack complète (Frontend, Backend, Base de données)

Construisons une application plus complexe avec une séparation frontend/backend :

```yaml
services:
  # Frontend React
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      # L'URL de l'API backend
      REACT_APP_API_URL: http://localhost:4000/api
    depends_on:
      - backend

  # Backend API Node.js
  backend:
    build: ./backend
    ports:
      - "4000:4000"
    environment:
      NODE_ENV: development
      PORT: 4000
      # Connexion à la base de données
      DATABASE_URL: postgresql://postgres:secret@database:5432/myapp
      # Connexion au cache Redis
      REDIS_URL: redis://cache:6379
    depends_on:
      - database
      - cache

  # Base de données PostgreSQL
  database:
    image: postgres:15-alpine
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: myapp
    volumes:
      - postgres-data:/var/lib/postgresql/data

  # Cache Redis
  cache:
    image: redis:7-alpine
    volumes:
      - redis-data:/data

volumes:
  postgres-data:
  redis-data:
```

**Architecture de cette application** :
```
[Utilisateur]
    ↓
[Frontend :3000] → [Backend :4000] → [Database]
                        ↓
                    [Cache Redis]
```

## Exemple 3 : Application WordPress complète

Un cas d'usage très populaire : déployer WordPress avec sa base de données MySQL.

```yaml
services:
  # Base de données MySQL pour WordPress
  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppassword
    volumes:
      - db-data:/var/lib/mysql
    restart: always

  # WordPress
  wordpress:
    image: wordpress:latest
    ports:
      - "8080:80"
    environment:
      # Configuration de la connexion à MySQL
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppassword
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wordpress-data:/var/www/html
    depends_on:
      - db
    restart: always

volumes:
  db-data:
  wordpress-data:
```

**Ce qui se passe** :
1. MySQL démarre et crée la base de données
2. WordPress démarre ensuite et se connecte à MySQL via le nom d'hôte `db`
3. Vous pouvez accéder à WordPress sur `http://localhost:8080`

## Exemple 4 : Application avec plusieurs bases de données

Parfois, vous avez besoin de différents types de stockage :

```yaml
services:
  # Application principale
  app:
    build: .
    ports:
      - "5000:5000"
    environment:
      # Base relationnelle
      POSTGRES_URL: postgresql://postgres:secret@postgres:5432/maindb
      # Base NoSQL
      MONGO_URL: mongodb://mongo:27017/documents
      # Cache
      REDIS_URL: redis://redis:6379
    depends_on:
      - postgres
      - mongo
      - redis

  # Base de données relationnelle
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: maindb
    volumes:
      - postgres-data:/var/lib/postgresql/data

  # Base de données NoSQL
  mongo:
    image: mongo:6
    volumes:
      - mongo-data:/data/db

  # Cache Redis
  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data

volumes:
  postgres-data:
  mongo-data:
  redis-data:
```

## Communication entre services : Les différents scénarios

### Scénario 1 : Backend vers Base de données

C'est le cas le plus simple. Le backend utilise directement le nom du service :

```yaml
services:
  api:
    image: mon-api
    environment:
      DB_HOST: database  # Nom du service
      DB_PORT: 5432

  database:
    image: postgres:15
```

### Scénario 2 : Frontend vers Backend

Le frontend (qui s'exécute dans le navigateur) ne peut pas utiliser directement les noms de services Docker. Il doit passer par les ports exposés :

```yaml
services:
  frontend:
    image: mon-frontend
    environment:
      # Utilise localhost car le navigateur est en dehors de Docker
      API_URL: http://localhost:4000/api

  backend:
    image: mon-backend
    ports:
      - "4000:4000"  # Expose le port pour le navigateur
```

### Scénario 3 : Service vers service via HTTP

```yaml
services:
  service-a:
    image: service-a
    environment:
      SERVICE_B_URL: http://service-b:8080

  service-b:
    image: service-b
    # Pas besoin d'exposer le port si seulement service-a y accède
```

## Organiser des services avec des réseaux personnalisés

Pour une meilleure organisation et sécurité, vous pouvez séparer vos services en plusieurs réseaux :

```yaml
services:
  # Frontend - réseau public
  nginx:
    image: nginx
    ports:
      - "80:80"
    networks:
      - frontend

  # API - réseaux public et privé
  api:
    image: mon-api
    networks:
      - frontend  # Pour communiquer avec nginx
      - backend   # Pour communiquer avec la DB

  # Base de données - réseau privé uniquement
  database:
    image: postgres:15
    networks:
      - backend  # Pas accessible depuis frontend

  # Worker - réseau privé uniquement
  worker:
    image: mon-worker
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
```

**Architecture avec réseaux** :
```
[Internet]
    ↓
[nginx] -------- frontend network -------- [api]
                                              ↓
                                         backend network
                                              ↓
                                     [database] [worker]
```

Cette approche améliore la sécurité : la base de données n'est accessible que par les services qui en ont vraiment besoin.

## Exemple 5 : Stack de développement complète

Voici un exemple réaliste d'environnement de développement :

```yaml
services:
  # Proxy inverse Nginx
  proxy:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - frontend
      - api
    networks:
      - frontend-network

  # Application React
  frontend:
    build: ./frontend
    volumes:
      - ./frontend:/app
      - /app/node_modules
    environment:
      CHOKIDAR_USEPOLLING: "true"  # Pour le hot reload
    networks:
      - frontend-network

  # API REST
  api:
    build: ./api
    volumes:
      - ./api:/app
      - /app/node_modules
    environment:
      DATABASE_URL: postgresql://postgres:secret@db:5432/devdb
      REDIS_URL: redis://redis:6379
      JWT_SECRET: dev-secret-key
    depends_on:
      - db
      - redis
    networks:
      - frontend-network
      - backend-network

  # Base de données PostgreSQL
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: devdb
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - backend-network

  # Redis pour le cache et les sessions
  redis:
    image: redis:7-alpine
    networks:
      - backend-network

  # Outil d'administration de base de données
  adminer:
    image: adminer
    ports:
      - "8080:8080"
    depends_on:
      - db
    networks:
      - backend-network

  # Service de mail pour le développement
  mailpit:
    image: axllent/mailpit
    ports:
      - "1025:1025"  # SMTP
      - "8025:8025"  # Interface web
    networks:
      - backend-network

volumes:
  postgres-data:

networks:
  frontend-network:
  backend-network:
```

**Ce qui rend cette stack puissante** :
- Interface d'administration de base de données (Adminer)
- Serveur de mail de test (Mailpit)
- Séparation réseau frontend/backend
- Hot reload pour le développement
- Volumes pour la persistance

## Partager des volumes entre services

Plusieurs services peuvent utiliser le même volume pour partager des fichiers :

```yaml
services:
  # Application qui génère des fichiers
  producer:
    image: mon-producteur
    volumes:
      - fichiers-partages:/data/output

  # Application qui traite ces fichiers
  consumer:
    image: mon-consommateur
    volumes:
      - fichiers-partages:/data/input

  # Backup des fichiers
  backup:
    image: mon-backup
    volumes:
      - fichiers-partages:/data:ro  # En lecture seule

volumes:
  fichiers-partages:
```

## Variables d'environnement partagées

Pour éviter la duplication, vous pouvez utiliser le fichier `.env` :

**Fichier `.env`** :
```env
# Base de données
DB_USER=admin  
DB_PASSWORD=supersecret  
DB_NAME=production  

# Configuration générale
APP_ENV=production  
LOG_LEVEL=info  
```

**Fichier `docker-compose.yml`** :
```yaml
services:
  api:
    image: mon-api
    environment:
      DATABASE_USER: ${DB_USER}
      DATABASE_PASSWORD: ${DB_PASSWORD}
      DATABASE_NAME: ${DB_NAME}
      APP_ENV: ${APP_ENV}

  worker:
    image: mon-worker
    environment:
      DATABASE_USER: ${DB_USER}
      DATABASE_PASSWORD: ${DB_PASSWORD}
      DATABASE_NAME: ${DB_NAME}
      LOG_LEVEL: ${LOG_LEVEL}

  database:
    image: postgres:15
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
```

## Bonnes pratiques pour les services multiples

### 1. Nommage cohérent des services

```yaml
# ✅ Bon : noms descriptifs et cohérents
services:
  api-users:
    image: api-users

  api-products:
    image: api-products

  database-users:
    image: postgres

  database-products:
    image: postgres

# ❌ Moins clair
services:
  svc1:
    image: api-users

  db:
    image: postgres
```

### 2. Séparer les préoccupations avec les réseaux

Ne mettez pas tous les services sur le même réseau. Isolez ce qui doit l'être :

```yaml
services:
  public-api:
    networks:
      - public

  internal-service:
    networks:
      - internal  # Pas accessible depuis l'extérieur

  database:
    networks:
      - internal  # Isolé du réseau public

networks:
  public:
  internal:
    internal: true  # Pas d'accès Internet
```

### 3. Utiliser des profils pour différents environnements

Les profils permettent de démarrer seulement certains services :

```yaml
services:
  app:
    image: mon-app
    # Toujours démarré

  database:
    image: postgres
    # Toujours démarré

  debug-tools:
    image: debug-tools
    profiles:
      - debug  # Démarré uniquement avec --profile debug

  performance-monitor:
    image: grafana
    profiles:
      - monitoring
```

Utilisation :
```bash
# Démarre app et database uniquement
docker compose up

# Démarre tout y compris debug-tools
docker compose --profile debug up

# Démarre avec monitoring
docker compose --profile monitoring up
```

### 4. Documenter les dépendances

```yaml
services:
  # Service principal
  # Dépend de : database, cache
  # Expose : API REST sur le port 3000
  api:
    image: mon-api
    depends_on:
      - database
      - cache

  # Base de données principale
  # Utilisée par : api, worker
  database:
    image: postgres:15

  # Cache Redis
  # Utilisé par : api (sessions), worker (queues)
  cache:
    image: redis:7
```

### 5. Ordonner les services logiquement

Organisez votre fichier par couches ou par fonction :

```yaml
services:
  # === Couche présentation ===
  nginx:
    image: nginx

  frontend:
    image: mon-frontend

  # === Couche application ===
  api:
    image: mon-api

  worker:
    image: mon-worker

  # === Couche données ===
  database:
    image: postgres

  cache:
    image: redis

  # === Outils ===
  monitoring:
    image: prometheus
```

## Démarrage conditionnel avec healthchecks

Pour attendre qu'un service soit vraiment prêt (et pas juste démarré) :

```yaml
services:
  database:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

  api:
    image: mon-api
    depends_on:
      database:
        condition: service_healthy  # Attend que la DB soit saine
```

## Scaling de services

Vous pouvez démarrer plusieurs instances d'un service :

```bash
# Démarre 3 instances du worker
docker compose up --scale worker=3
```

```yaml
services:
  # Load balancer
  nginx:
    image: nginx
    ports:
      - "80:80"

  # Application (scalable)
  app:
    image: mon-app
    # Pas de ports exposés directement
    # Nginx fera le load balancing

  # Base de données (pas scalable comme ça)
  database:
    image: postgres
```

## Points clés à retenir

✅ Les services communiquent entre eux en utilisant leur nom comme nom d'hôte

✅ Docker Compose crée automatiquement un réseau privé pour tous vos services

✅ Utilisez `depends_on` pour gérer l'ordre de démarrage

✅ Exposez uniquement les ports nécessaires vers l'extérieur

✅ Utilisez des réseaux personnalisés pour isoler les services sensibles

✅ Un même volume peut être partagé entre plusieurs services

✅ Le fichier `.env` permet de centraliser les variables d'environnement

✅ Documentez vos services et leurs dépendances

✅ Organisez votre fichier logiquement (par couche ou par fonction)

✅ Utilisez les healthchecks pour les dépendances critiques

---

**Prochaine section** : Nous allons voir comment gérer les dépendances entre services de manière plus sophistiquée et garantir un démarrage fiable de votre application !

⏭️ [Gestion des dépendances entre services](/08-docker-compose/04-gestion-des-dependances-entre-services.md)
