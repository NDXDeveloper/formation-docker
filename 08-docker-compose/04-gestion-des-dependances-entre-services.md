🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 8.4 Gestion des dépendances entre services

## Introduction

Dans une application composée de plusieurs services, certains services dépendent d'autres pour fonctionner. Par exemple, votre application backend ne peut pas démarrer sans que la base de données soit prête à accepter des connexions.

Gérer correctement ces dépendances est crucial pour éviter des erreurs au démarrage et garantir que votre application fonctionne de manière fiable. Dans cette section, nous allons explorer les différentes techniques pour gérer ces dépendances efficacement.

## Le problème : depends_on n'est pas suffisant

### Ce que fait depends_on

Nous avons vu `depends_on` dans les sections précédentes. Rappel :

```yaml
services:
  app:
    image: mon-app
    depends_on:
      - database

  database:
    image: postgres:15
```

Avec cette configuration, Docker Compose garantit que :
1. Le conteneur `database` démarre **avant** le conteneur `app`
2. Le conteneur `app` s'arrête **avant** le conteneur `database` lors d'un `docker compose down`

### Le problème

**depends_on attend seulement que le conteneur démarre, pas qu'il soit prêt !**

Voici ce qui se passe en réalité :

```
Temps 0s  : database démarre (conteneur créé)
Temps 1s  : app démarre (depends_on satisfait)
Temps 2s  : app essaie de se connecter à database
Temps 3s  : ERREUR ! database n'est pas encore prête
Temps 5s  : database est enfin prête à accepter des connexions
```

PostgreSQL, MySQL et autres bases de données ont besoin de temps pour :
- Initialiser leurs fichiers
- Charger leurs configurations
- Créer les bases de données
- Être prêtes à accepter des connexions

Pendant ce temps, votre application peut échouer en essayant de se connecter trop tôt ! 😱

## Solution 1 : Les Health Checks (Recommandé)

Les health checks permettent de vérifier qu'un service est réellement prêt à fonctionner, pas seulement qu'il a démarré.

### Définir un health check

```yaml
version: '3.8'

services:
  database:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5
      start_period: 10s

  app:
    image: mon-app
    depends_on:
      database:
        condition: service_healthy
```

**Explication des paramètres** :

- **test** : La commande à exécuter pour vérifier la santé
  - `pg_isready` : Outil PostgreSQL qui vérifie si la base accepte des connexions
  - `CMD-SHELL` : Exécute la commande dans un shell

- **interval** : Temps entre chaque vérification (ici, toutes les 5 secondes)

- **timeout** : Temps maximum pour que la commande se termine (5 secondes)

- **retries** : Nombre d'échecs consécutifs avant de marquer le service comme "unhealthy" (5 fois)

- **start_period** : Période de grâce au démarrage où les échecs ne comptent pas (10 secondes)

### depends_on avec condition

```yaml
depends_on:
  database:
    condition: service_healthy  # Attend que le healthcheck réussisse
```

Les conditions disponibles :
- `service_started` : Le conteneur a démarré (comportement par défaut)
- `service_healthy` : Le healthcheck réussit
- `service_completed_successfully` : Le conteneur s'est terminé avec succès (code 0)

### Exemples de health checks pour différents services

#### PostgreSQL

```yaml
database:
  image: postgres:15
  healthcheck:
    test: ["CMD-SHELL", "pg_isready -U postgres"]
    interval: 5s
    timeout: 5s
    retries: 5
```

#### MySQL

```yaml
database:
  image: mysql:8
  healthcheck:
    test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
    interval: 5s
    timeout: 5s
    retries: 5
```

#### Redis

```yaml
cache:
  image: redis:7
  healthcheck:
    test: ["CMD", "redis-cli", "ping"]
    interval: 5s
    timeout: 5s
    retries: 5
```

#### MongoDB

```yaml
database:
  image: mongo:6
  healthcheck:
    test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
    interval: 10s
    timeout: 5s
    retries: 5
```

#### Application HTTP

```yaml
api:
  image: mon-api
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
    interval: 10s
    timeout: 5s
    retries: 3
```

#### Application avec wget

Si curl n'est pas disponible dans l'image :

```yaml
api:
  image: mon-api
  healthcheck:
    test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost:8080/health"]
    interval: 10s
    timeout: 5s
    retries: 3
```

## Solution 2 : Scripts d'attente (wait-for)

Une autre approche consiste à faire attendre votre application que le service soit prêt.

### Utiliser wait-for-it

`wait-for-it` est un script bash populaire qui attend qu'un port TCP soit accessible.

```yaml
version: '3.8'

services:
  database:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret

  app:
    image: mon-app
    depends_on:
      - database
    command: >
      sh -c "
        ./wait-for-it.sh database:5432 --timeout=60 --strict --
        node server.js
      "
```

**Ce qui se passe** :
1. Le script `wait-for-it.sh` attend que le port 5432 de `database` soit accessible
2. Timeout après 60 secondes si la connexion échoue
3. Une fois prêt, lance `node server.js`

### Télécharger wait-for-it

Dans votre Dockerfile :

```dockerfile
FROM node:18

# Installer wait-for-it
ADD https://raw.githubusercontent.com/vishnubob/wait-for-it/master/wait-for-it.sh /usr/local/bin/wait-for-it.sh
RUN chmod +x /usr/local/bin/wait-for-it.sh

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .

CMD ["node", "server.js"]
```

### Utiliser dockerize

`dockerize` est un autre outil similaire mais plus puissant :

```yaml
services:
  database:
    image: postgres:15

  app:
    image: mon-app
    depends_on:
      - database
    command: >
      dockerize
        -wait tcp://database:5432
        -timeout 60s
        node server.js
```

## Solution 3 : Retry Logic dans l'application

La meilleure solution est souvent d'implémenter une logique de retry directement dans votre application.

### Exemple en Node.js

```javascript
// config/database.js
const { Client } = require('pg');

async function connectWithRetry(maxRetries = 10, delay = 5000) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const client = new Client({
        host: process.env.DB_HOST,
        port: process.env.DB_PORT,
        user: process.env.DB_USER,
        password: process.env.DB_PASSWORD,
        database: process.env.DB_NAME
      });

      await client.connect();
      console.log('✅ Connecté à la base de données');
      return client;

    } catch (error) {
      console.log(`❌ Tentative ${i + 1}/${maxRetries} échouée`);

      if (i < maxRetries - 1) {
        console.log(`⏳ Nouvelle tentative dans ${delay/1000}s...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw new Error('Impossible de se connecter à la base de données');
      }
    }
  }
}

module.exports = { connectWithRetry };
```

### Exemple en Python

```python
# database.py
import psycopg2
import time
import os

def connect_with_retry(max_retries=10, delay=5):
    for attempt in range(max_retries):
        try:
            connection = psycopg2.connect(
                host=os.getenv('DB_HOST'),
                port=os.getenv('DB_PORT'),
                user=os.getenv('DB_USER'),
                password=os.getenv('DB_PASSWORD'),
                database=os.getenv('DB_NAME')
            )
            print('✅ Connecté à la base de données')
            return connection

        except psycopg2.OperationalError as e:
            print(f'❌ Tentative {attempt + 1}/{max_retries} échouée')

            if attempt < max_retries - 1:
                print(f'⏳ Nouvelle tentative dans {delay}s...')
                time.sleep(delay)
            else:
                raise Exception('Impossible de se connecter à la base de données')
```

### Avec docker-compose

```yaml
services:
  app:
    build: .
    depends_on:
      - database
    environment:
      DB_HOST: database
      DB_PORT: 5432
      DB_USER: postgres
      DB_PASSWORD: secret
    # L'application gère elle-même l'attente

  database:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
```

**Avantages de cette approche** :
- ✅ Fonctionne même si la base de données redémarre pendant l'exécution
- ✅ Pas de dépendance externe (wait-for-it, dockerize)
- ✅ Plus robuste en production
- ✅ Logs clairs pour le debugging

## Solution 4 : restart: on-failure

Une approche simple : laisser le conteneur redémarrer automatiquement s'il échoue :

```yaml
services:
  app:
    image: mon-app
    depends_on:
      - database
    restart: on-failure
    environment:
      DB_HOST: database

  database:
    image: postgres:15
    restart: always
```

**Comment ça fonctionne** :
1. `app` démarre et essaie de se connecter à `database`
2. Si `database` n'est pas prête, `app` crash
3. Docker redémarre automatiquement `app` grâce à `restart: on-failure`
4. Après quelques tentatives, `database` est prête et `app` se connecte

**⚠️ Attention** : Cette approche peut être moins élégante (logs d'erreur, délai au démarrage) mais elle fonctionne !

## Solution 5 : Init containers (avec un service temporaire)

Vous pouvez créer un service temporaire qui attend que tout soit prêt :

```yaml
version: '3.8'

services:
  # Service d'initialisation
  wait-for-db:
    image: postgres:15
    command: >
      sh -c "
        until pg_isready -h database -U postgres; do
          echo 'En attente de la base de données...';
          sleep 2;
        done;
        echo 'Base de données prête !';
      "
    depends_on:
      - database

  # Base de données
  database:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret

  # Application
  app:
    image: mon-app
    depends_on:
      wait-for-db:
        condition: service_completed_successfully
```

## Dépendances multiples et complexes

### Dépendances en chaîne

```yaml
services:
  # Niveau 1 : Base de données
  database:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]
      interval: 5s

  # Niveau 2 : Migrations
  migrations:
    image: flyway/flyway
    depends_on:
      database:
        condition: service_healthy
    command: migrate

  # Niveau 3 : API
  api:
    image: mon-api
    depends_on:
      migrations:
        condition: service_completed_successfully
      database:
        condition: service_healthy

  # Niveau 4 : Worker
  worker:
    image: mon-worker
    depends_on:
      api:
        condition: service_started
```

**Ordre de démarrage** : database → migrations → api → worker

### Dépendances parallèles

```yaml
services:
  # L'application dépend de plusieurs services
  app:
    image: mon-app
    depends_on:
      database:
        condition: service_healthy
      cache:
        condition: service_healthy
      queue:
        condition: service_healthy

  database:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]
      interval: 5s

  cache:
    image: redis:7
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s

  queue:
    image: rabbitmq:3
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "ping"]
      interval: 10s
```

**Ce qui se passe** : database, cache et queue démarrent en parallèle, puis app démarre quand tous sont healthy.

## Exemple complet : Application réelle avec dépendances

Voici un exemple qui combine plusieurs techniques :

```yaml
version: '3.8'

services:
  # Base de données principale
  database:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: production
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5
      start_period: 10s
    restart: always

  # Cache Redis
  cache:
    image: redis:7-alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5
    restart: always

  # Migration de base de données
  migrate:
    image: migrate/migrate
    depends_on:
      database:
        condition: service_healthy
    volumes:
      - ./migrations:/migrations
    command: >
      -path=/migrations
      -database postgres://postgres:${DB_PASSWORD}@database:5432/production?sslmode=disable
      up

  # API Backend
  api:
    build: ./api
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgres://postgres:${DB_PASSWORD}@database:5432/production
      REDIS_URL: redis://cache:6379
      NODE_ENV: production
    depends_on:
      database:
        condition: service_healthy
      cache:
        condition: service_healthy
      migrate:
        condition: service_completed_successfully
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost:3000/health"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 30s
    restart: unless-stopped

  # Worker de tâches en arrière-plan
  worker:
    build: ./worker
    environment:
      DATABASE_URL: postgres://postgres:${DB_PASSWORD}@database:5432/production
      REDIS_URL: redis://cache:6379
    depends_on:
      database:
        condition: service_healthy
      cache:
        condition: service_healthy
      api:
        condition: service_healthy
    restart: unless-stopped

  # Frontend
  frontend:
    build: ./frontend
    ports:
      - "80:80"
    depends_on:
      api:
        condition: service_healthy
    restart: unless-stopped

volumes:
  postgres-data:
```

**Points clés de cet exemple** :
1. Tous les services critiques ont des health checks
2. Les migrations s'exécutent avant l'API
3. L'API démarre après la base de données ET les migrations
4. Le worker attend que l'API soit healthy
5. Le frontend attend que l'API soit healthy
6. Politiques de restart appropriées pour chaque service

## Débogage des problèmes de dépendances

### Voir l'état des services

```bash
# Voir tous les services et leur état
docker compose ps

# Voir les logs d'un service spécifique
docker compose logs database

# Suivre les logs en temps réel
docker compose logs -f api

# Voir les logs de tous les services
docker compose logs -f
```

### Vérifier le health check

```bash
# Voir l'état de santé des conteneurs
docker compose ps

# Inspecter un conteneur pour voir les détails du health check
docker inspect <container-name> | grep -A 10 Health
```

### Tester manuellement la connexion

```bash
# Se connecter à un conteneur
docker compose exec api sh

# Tester la connexion à la base de données depuis l'intérieur
ping database
nc -zv database 5432
```

## Stratégies de démarrage selon l'environnement

### Développement : Simplicité

```yaml
services:
  app:
    image: mon-app
    depends_on:
      - database
    restart: on-failure

  database:
    image: postgres:15
```

Simple et suffisant pour le développement local.

### Staging : Équilibre

```yaml
services:
  app:
    image: mon-app
    depends_on:
      database:
        condition: service_healthy
    restart: unless-stopped

  database:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]
      interval: 5s
```

Plus robuste avec health checks.

### Production : Robustesse maximale

```yaml
services:
  app:
    image: mon-app
    depends_on:
      database:
        condition: service_healthy
      cache:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 30s
    restart: always

  database:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]
      interval: 5s
      timeout: 5s
      retries: 5
    restart: always
```

Health checks partout + retry logic dans le code.

## Bonnes pratiques

### ✅ À faire

1. **Toujours définir des health checks pour les bases de données**
   ```yaml
   database:
     healthcheck:
       test: ["CMD-SHELL", "pg_isready"]
   ```

2. **Implémenter la logique de retry dans votre code**
   - Plus robuste que depends_on seul
   - Gère les redémarrages en production

3. **Utiliser start_period pour les services lents**
   ```yaml
   healthcheck:
     start_period: 30s  # 30s de grâce au démarrage
   ```

4. **Définir des politiques de restart appropriées**
   ```yaml
   restart: unless-stopped  # En production
   restart: on-failure      # En développement
   ```

5. **Logger clairement les tentatives de connexion**
   - Aide au debugging
   - Montre la progression

### ❌ À éviter

1. **Ne pas utiliser sleep avec un délai fixe**
   ```yaml
   # ❌ Mauvais
   command: sh -c "sleep 10 && node app.js"
   # Trop court = échec, trop long = perte de temps
   ```

2. **Ne pas ignorer les dépendances**
   ```yaml
   # ❌ Risqué
   app:
     image: mon-app
     # Pas de depends_on !
   ```

3. **Ne pas utiliser depends_on sans condition pour les services critiques**
   ```yaml
   # ❌ Insuffisant en production
   depends_on:
     - database  # Service started seulement

   # ✅ Mieux
   depends_on:
     database:
       condition: service_healthy
   ```

## Points clés à retenir

✅ `depends_on` seul n'attend que le démarrage du conteneur, pas qu'il soit prêt

✅ Les **health checks** sont la meilleure solution pour vérifier qu'un service est réellement prêt

✅ La **condition: service_healthy** dans depends_on attend que le health check réussisse

✅ Implémenter une **logique de retry** dans votre code est la solution la plus robuste

✅ Les scripts comme **wait-for-it** sont utiles mais pas indispensables

✅ Utilisez **restart: on-failure** ou **restart: unless-stopped** pour plus de résilience

✅ Définissez des health checks pour tous les services critiques (bases de données, APIs)

✅ Adaptez votre stratégie selon l'environnement (dev, staging, production)

✅ Loggez clairement les tentatives de connexion pour faciliter le debugging

✅ Combinez plusieurs techniques pour une solution robuste

---

**Prochaine section** : Nous allons explorer les commandes Docker Compose essentielles pour gérer votre application au quotidien !

⏭️ [Commandes Docker Compose (up, down, logs, ps)](/08-docker-compose/05-commandes-docker-compose.md)
