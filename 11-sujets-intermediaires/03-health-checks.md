🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 11.3 Health checks

## Introduction

Imaginez que vous avez lancé un conteneur Docker. La commande `docker ps` vous indique qu'il est en cours d'exécution. Mais est-ce que l'application à l'intérieur **fonctionne vraiment correctement** ?

Un conteneur peut être "démarré" techniquement, mais l'application qu'il contient peut :
- Être bloquée dans une boucle infinie
- Ne plus répondre aux requêtes
- Avoir perdu sa connexion à la base de données
- Être dans un état incohérent

C'est exactement le problème que les **health checks** (vérifications de santé) permettent de résoudre.

## Qu'est-ce qu'un Health Check ?

Un health check est un **test automatique** que Docker exécute régulièrement pour vérifier si votre application fonctionne correctement. C'est comme prendre le pouls d'un patient pour vérifier qu'il est en bonne santé.

### Analogie simple

Pensez à un restaurant :
- 🏪 **Sans health check** : Vous voyez que le restaurant est ouvert (lumières allumées, porte déverrouillée) → Le conteneur est "running"
- ✅ **Avec health check** : Vous entrez et demandez "Puis-je commander ?" et le serveur répond "Oui, bien sûr !" → L'application est vraiment opérationnelle

### Problème sans health checks

```bash
# Lancer un conteneur
docker run -d --name my-app nginx

# Vérifier le statut
docker ps
# STATUS: Up 2 minutes

# Mais l'application peut être cassée !
curl http://localhost
# Error: Connection refused
```

Docker pense que le conteneur va bien (processus en cours), mais l'application ne répond pas.

## Comment fonctionne un Health Check ?

Docker exécute périodiquement une commande de test dans le conteneur :

1. **Test réussi** (code de sortie 0) → Le conteneur est "healthy" (en bonne santé)
2. **Test échoué** (code de sortie 1) → Le conteneur est "unhealthy" (malsain)
3. **En cours de démarrage** → Le conteneur est "starting" (les premiers tests)

### Cycle de vie avec health checks

```
Container démarré
       ↓
   [starting] ← Tests initiaux en cours
       ↓
   [healthy] ← Application répond correctement
       ↓
   (problème détecté)
       ↓
   [unhealthy] ← Application ne répond plus
```

## Définir un Health Check dans un Dockerfile

### Syntaxe de base

```dockerfile
HEALTHCHECK [OPTIONS] CMD commande
```

### Exemple simple : Application web

```dockerfile
FROM nginx:alpine

# Copie de votre application
COPY ./html /usr/share/nginx/html

# Health check : vérifier que nginx répond
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost/ || exit 1
```

**Explication ligne par ligne** :
- `--interval=30s` : Exécuter le test toutes les 30 secondes
- `--timeout=3s` : Le test doit réussir en moins de 3 secondes
- `--start-period=5s` : Attendre 5 secondes avant le premier test (temps de démarrage)
- `--retries=3` : Considérer unhealthy après 3 échecs consécutifs
- `CMD curl -f http://localhost/` : Commande de test (faire une requête HTTP)
- `|| exit 1` : Si la commande échoue, retourner le code d'erreur 1

### Les options disponibles

| Option | Description | Valeur par défaut | Exemple |
|--------|-------------|-------------------|---------|
| `--interval` | Délai entre deux tests | 30s | `--interval=1m` |
| `--timeout` | Temps maximum pour un test | 30s | `--timeout=10s` |
| `--start-period` | Temps avant le premier test | 0s | `--start-period=30s` |
| `--retries` | Nombre d'échecs avant unhealthy | 3 | `--retries=5` |

## Exemples pratiques par type d'application

### Application web (HTTP)

```dockerfile
FROM node:18-alpine

WORKDIR /app
COPY package*.json ./
RUN npm install --production
COPY . .

# Health check HTTP avec curl
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s \
  CMD curl -f http://localhost:3000/health || exit 1

CMD ["node", "server.js"]
```

**Alternative avec wget** (si curl n'est pas disponible) :
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget --quiet --tries=1 --spider http://localhost:3000/health || exit 1
```

### Base de données PostgreSQL

```dockerfile
FROM postgres:15

# Health check : vérifier que PostgreSQL accepte les connexions
HEALTHCHECK --interval=10s --timeout=5s --start-period=30s --retries=3 \
  CMD pg_isready -U postgres || exit 1
```

**Explication** :
- `pg_isready` : Commande PostgreSQL pour vérifier la disponibilité
- `-U postgres` : Nom d'utilisateur
- `--start-period=30s` : PostgreSQL a besoin de temps pour démarrer

### Base de données MySQL

```dockerfile
FROM mysql:8

# Health check MySQL
HEALTHCHECK --interval=10s --timeout=5s --start-period=30s \
  CMD mysqladmin ping -h localhost || exit 1
```

### Redis

```dockerfile
FROM redis:alpine

# Health check Redis
HEALTHCHECK --interval=5s --timeout=3s --start-period=5s \
  CMD redis-cli ping || exit 1
```

**Explication** :
- `redis-cli ping` : Commande Redis qui retourne "PONG" si le serveur répond
- Interval court (5s) car Redis est rapide à démarrer

### Application Python Flask

```dockerfile
FROM python:3.11-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .

# Health check avec curl
HEALTHCHECK --interval=30s --timeout=5s --start-period=15s \
  CMD curl -f http://localhost:5000/api/health || exit 1

CMD ["python", "app.py"]
```

### API avec endpoint dédié

Créez un endpoint spécifique pour les health checks :

**app.js (Node.js/Express)** :
```javascript
app.get('/health', (req, res) => {
  // Vérifier la connexion à la base de données
  if (database.isConnected()) {
    res.status(200).json({ status: 'healthy' });
  } else {
    res.status(503).json({ status: 'unhealthy' });
  }
});
```

**Dockerfile** :
```dockerfile
FROM node:18-alpine

WORKDIR /app
COPY package*.json ./
RUN npm install --production
COPY . .

# Health check sur l'endpoint dédié
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s \
  CMD curl -f http://localhost:3000/health || exit 1

CMD ["node", "app.js"]
```

## Health Checks via docker run

Vous pouvez définir un health check directement lors du lancement d'un conteneur :

```bash
docker run -d \
  --name my-app \
  --health-cmd="curl -f http://localhost/ || exit 1" \
  --health-interval=30s \
  --health-timeout=3s \
  --health-start-period=5s \
  --health-retries=3 \
  nginx
```

**Avantage** : Pratique pour tester sans modifier le Dockerfile.

### Exemple avec PostgreSQL

```bash
docker run -d \
  --name postgres-db \
  --health-cmd="pg_isready -U postgres" \
  --health-interval=10s \
  --health-timeout=5s \
  --health-start-period=30s \
  --health-retries=3 \
  -e POSTGRES_PASSWORD=secret \
  postgres:15
```

## Health Checks avec Docker Compose

C'est la méthode la plus courante en pratique.

### Syntaxe Docker Compose

```yaml
version: '3.8'

services:
  web:
    image: nginx:alpine
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 3s
      start_period: 5s
      retries: 3
```

### Exemple complet : Application multi-services

```yaml
version: '3.8'

services:
  # Frontend
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 5s
      start_period: 10s
      retries: 3
    depends_on:
      api:
        condition: service_healthy

  # API Backend
  api:
    build: ./api
    ports:
      - "5000:5000"
    environment:
      - DATABASE_URL=postgres://user:pass@database:5432/mydb
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/api/health"]
      interval: 20s
      timeout: 5s
      start_period: 15s
      retries: 3
    depends_on:
      database:
        condition: service_healthy
      redis:
        condition: service_healthy

  # Base de données
  database:
    image: postgres:15
    environment:
      - POSTGRES_PASSWORD=secret
      - POSTGRES_USER=user
      - POSTGRES_DB=mydb
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      timeout: 5s
      start_period: 30s
      retries: 5
    volumes:
      - postgres_data:/var/lib/postgresql/data

  # Cache Redis
  redis:
    image: redis:alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      start_period: 5s
      retries: 3

volumes:
  postgres_data:
```

**Points importants** :
- `depends_on` avec `condition: service_healthy` : Attend que les services dépendants soient healthy
- Chaque service a son health check adapté
- Les start_period plus longs pour les services qui démarrent lentement (PostgreSQL)

## Vérifier le statut de santé

### Commande docker ps

```bash
docker ps
```

**Output** :
```
CONTAINER ID   IMAGE     STATUS                    PORTS
abc123         nginx     Up 2 minutes (healthy)    80/tcp
def456         redis     Up 1 minute (healthy)     6379/tcp
ghi789         postgres  Up 30 seconds (starting)  5432/tcp
```

### Commande docker inspect

```bash
# Voir tous les détails du health check
docker inspect my-container

# Voir uniquement le statut de santé
docker inspect --format='{{.State.Health.Status}}' my-container
```

**Output** : `healthy`, `unhealthy`, ou `starting`

### Voir l'historique des health checks

```bash
docker inspect --format='{{json .State.Health}}' my-container | jq
```

**Output** :
```json
{
  "Status": "healthy",
  "FailingStreak": 0,
  "Log": [
    {
      "Start": "2024-10-22T10:30:00Z",
      "End": "2024-10-22T10:30:01Z",
      "ExitCode": 0,
      "Output": ""
    }
  ]
}
```

## Syntaxes de commandes

Il existe trois syntaxes pour définir la commande de test :

### 1. Syntaxe CMD (Dockerfile)

```dockerfile
HEALTHCHECK CMD curl -f http://localhost || exit 1
```

La commande est exécutée via le shell (`/bin/sh -c`).

### 2. Syntaxe exec form (Dockerfile)

```dockerfile
HEALTHCHECK CMD ["curl", "-f", "http://localhost"]
```

La commande est exécutée directement sans shell.

### 3. CMD-SHELL (Docker Compose)

```yaml
healthcheck:
  test: ["CMD-SHELL", "curl -f http://localhost || exit 1"]
```

Équivalent à la syntaxe CMD dans Dockerfile.

### Quelle syntaxe choisir ?

**Recommandation** : Utilisez la syntaxe exec form (tableau) pour éviter les problèmes avec le shell.

```yaml
# ✅ Recommandé
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]

# ⚠️ Fonctionne mais moins portable
healthcheck:
  test: ["CMD-SHELL", "curl -f http://localhost || exit 1"]
```

## Health Checks avancés

### Vérifier plusieurs conditions

```dockerfile
FROM node:18-alpine

# Installation de curl
RUN apk add --no-cache curl

WORKDIR /app
COPY . .

# Health check complexe : vérifier l'API et la connexion DB
HEALTHCHECK --interval=30s --timeout=10s --start-period=15s \
  CMD curl -f http://localhost:3000/health && \
      curl -f http://localhost:3000/api/db/ping || exit 1

CMD ["node", "server.js"]
```

### Script de health check personnalisé

**healthcheck.sh** :
```bash
#!/bin/sh

# Vérifier que l'API répond
if ! curl -f http://localhost:3000/health > /dev/null 2>&1; then
  echo "API health check failed"
  exit 1
fi

# Vérifier la connexion à la base de données
if ! curl -f http://localhost:3000/api/db/ping > /dev/null 2>&1; then
  echo "Database connection failed"
  exit 1
fi

# Vérifier l'utilisation mémoire (optionnel)
MEMORY_USAGE=$(free | grep Mem | awk '{print ($3/$2) * 100.0}')
if [ $(echo "$MEMORY_USAGE > 90" | bc) -eq 1 ]; then
  echo "Memory usage too high: ${MEMORY_USAGE}%"
  exit 1
fi

echo "All checks passed"
exit 0
```

**Dockerfile** :
```dockerfile
FROM node:18-alpine

RUN apk add --no-cache curl bc

WORKDIR /app
COPY healthcheck.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/healthcheck.sh
COPY . .

HEALTHCHECK --interval=30s --timeout=10s --start-period=15s \
  CMD /usr/local/bin/healthcheck.sh

CMD ["node", "server.js"]
```

### Health check avec authentification

```yaml
services:
  api:
    build: ./api
    healthcheck:
      test: ["CMD", "curl", "-f", "-H", "Authorization: Bearer ${HEALTH_TOKEN}", "http://localhost:5000/health"]
      interval: 30s
      timeout: 5s
```

### Health check pour applications HTTPS

```dockerfile
# Health check avec certificat auto-signé (insecure)
HEALTHCHECK CMD curl -f -k https://localhost || exit 1

# Health check avec certificat valide
HEALTHCHECK CMD curl -f https://localhost || exit 1
```

## Désactiver les Health Checks

Parfois, vous voulez désactiver un health check défini dans l'image :

### Dans Dockerfile

```dockerfile
HEALTHCHECK NONE
```

### Avec docker run

```bash
docker run -d --no-healthcheck nginx
```

### Dans Docker Compose

```yaml
services:
  web:
    image: nginx
    healthcheck:
      disable: true
```

## Utilisation avec les orchestrateurs

### Docker Swarm

Docker Swarm utilise les health checks pour :
- Redémarrer automatiquement les conteneurs unhealthy
- Router le trafic uniquement vers les conteneurs healthy

```yaml
version: '3.8'

services:
  web:
    image: my-app
    deploy:
      replicas: 3
      update_config:
        order: start-first
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 5s
      retries: 3
```

Swarm va :
1. Attendre que le nouveau conteneur soit healthy avant de tuer l'ancien
2. Ne pas router le trafic vers les conteneurs unhealthy

### Préparation pour Kubernetes

Les concepts de health checks Docker se traduisent en Kubernetes :

**Docker** :
```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost/health"]
```

**Kubernetes** :
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 30
```

## Bonnes pratiques

### 1. Créez toujours un endpoint /health dédié

```javascript
// app.js
app.get('/health', (req, res) => {
  // Vérifications légères et rapides
  if (server.isRunning()) {
    res.status(200).send('OK');
  } else {
    res.status(503).send('Service Unavailable');
  }
});
```

**Pourquoi ?** :
- ✅ Plus facile à tester
- ✅ Ne dépend pas de la logique métier
- ✅ Rapide à exécuter

### 2. Gardez les health checks simples et rapides

```dockerfile
# ❌ Mauvais : trop complexe et lent
HEALTHCHECK CMD curl -f http://localhost/api/run-full-tests

# ✅ Bon : simple et rapide
HEALTHCHECK CMD curl -f http://localhost/health
```

**Règle** : Un health check doit s'exécuter en moins de 1 seconde.

### 3. Adaptez start-period au temps de démarrage

```dockerfile
# Application légère (nginx)
HEALTHCHECK --start-period=5s CMD curl -f http://localhost

# Application Java (démarrage lent)
HEALTHCHECK --start-period=60s CMD curl -f http://localhost:8080/health

# Base de données
HEALTHCHECK --start-period=30s CMD pg_isready
```

### 4. N'oubliez pas d'installer les outils nécessaires

```dockerfile
# ❌ Erreur courante
FROM alpine
HEALTHCHECK CMD curl -f http://localhost
# curl n'est pas installé !

# ✅ Correct
FROM alpine
RUN apk add --no-cache curl
HEALTHCHECK CMD curl -f http://localhost
```

### 5. Utilisez des codes de sortie corrects

```bash
# ✅ Bon
curl -f http://localhost/health || exit 1

# ❌ Mauvais (ne retourne pas toujours 1 en cas d'échec)
curl http://localhost/health
```

### 6. Testez vos health checks

```bash
# Construire l'image
docker build -t my-app .

# Lancer le conteneur
docker run -d --name test-app my-app

# Observer le démarrage
watch -n 1 'docker ps --format "table {{.Names}}\t{{.Status}}"'

# Vérifier les logs du health check
docker inspect --format='{{json .State.Health}}' test-app | jq
```

### 7. Logguez les échecs de health checks

```javascript
// app.js
app.get('/health', (req, res) => {
  try {
    // Vérifications
    const dbOk = database.isConnected();
    const cacheOk = redis.isConnected();

    if (dbOk && cacheOk) {
      res.status(200).json({ status: 'healthy', checks: { db: 'ok', cache: 'ok' } });
    } else {
      console.error('Health check failed', { db: dbOk, cache: cacheOk });
      res.status(503).json({ status: 'unhealthy', checks: { db: dbOk, cache: cacheOk } });
    }
  } catch (error) {
    console.error('Health check error:', error);
    res.status(503).json({ status: 'error', message: error.message });
  }
});
```

### 8. Utilisez depends_on avec condition

```yaml
services:
  api:
    depends_on:
      database:
        condition: service_healthy
      redis:
        condition: service_healthy
```

**Avantage** : L'API ne démarre que quand les dépendances sont ready.

### 9. Ajustez les intervalles selon le contexte

```yaml
# Développement : checks fréquents pour détecter rapidement les problèmes
healthcheck:
  interval: 10s
  timeout: 3s

# Production : checks moins fréquents pour économiser les ressources
healthcheck:
  interval: 60s
  timeout: 5s
```

### 10. Documentez vos health checks

```yaml
services:
  api:
    build: ./api
    healthcheck:
      # Vérifie que l'API répond et que la connexion DB est active
      # Endpoint /health vérifie : DB, Redis, file system
      # Timeout de 5s car l'API peut être lente sous charge
      test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
      interval: 30s
      timeout: 5s
      start_period: 15s
      retries: 3
```

## Problèmes courants et solutions

### Problème 1 : Conteneur toujours "starting"

**Symptômes** :
```bash
docker ps
# STATUS: Up 5 minutes (starting)
```

**Causes possibles** :
1. Le health check échoue constamment
2. Le start-period est trop court
3. L'application met trop de temps à démarrer

**Solutions** :
```dockerfile
# Augmenter le start-period
HEALTHCHECK --start-period=60s CMD curl -f http://localhost

# Vérifier les logs
docker logs my-container

# Tester le health check manuellement
docker exec my-container curl -f http://localhost
```

### Problème 2 : Health check passe mais l'application ne fonctionne pas

**Cause** : Health check trop simple qui ne vérifie pas vraiment l'application.

**Solution** :
```dockerfile
# ❌ Trop simple
HEALTHCHECK CMD curl http://localhost

# ✅ Vérifie vraiment l'application
HEALTHCHECK CMD curl -f http://localhost/api/health
```

### Problème 3 : Conteneur devient unhealthy aléatoirement

**Causes** :
- Timeout trop court
- Pics de charge
- Problèmes réseau temporaires

**Solutions** :
```yaml
healthcheck:
  # Augmenter le timeout
  timeout: 10s
  # Augmenter les retries
  retries: 5
  # Vérifier les logs
```

### Problème 4 : "curl: command not found"

**Solution** :
```dockerfile
# Pour Alpine
RUN apk add --no-cache curl

# Pour Debian/Ubuntu
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*

# Ou utiliser wget
HEALTHCHECK CMD wget --quiet --tries=1 --spider http://localhost
```

### Problème 5 : Health check ralentit le système

**Cause** : Trop de health checks ou checks trop lourds.

**Solution** :
```yaml
# Réduire la fréquence
healthcheck:
  interval: 2m  # Au lieu de 30s

# Simplifier le test
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost/ping"]  # Endpoint léger
```

## Alternatives aux Health Checks HTTP

### 1. Vérification TCP

```dockerfile
# Vérifier qu'un port écoute
HEALTHCHECK CMD nc -z localhost 8080 || exit 1
```

### 2. Vérification de processus

```dockerfile
# Vérifier qu'un processus tourne
HEALTHCHECK CMD pgrep -f "java.*myapp" || exit 1
```

### 3. Vérification de fichier

```dockerfile
# Vérifier qu'un fichier existe (l'app le crée quand elle est ready)
HEALTHCHECK CMD test -f /tmp/app-ready || exit 1
```

### 4. Script Python

```dockerfile
FROM python:3.11-slim

COPY healthcheck.py /usr/local/bin/
RUN chmod +x /usr/local/bin/healthcheck.py

HEALTHCHECK CMD python /usr/local/bin/healthcheck.py
```

**healthcheck.py** :
```python
#!/usr/bin/env python3
import sys
import requests

try:
    response = requests.get('http://localhost:5000/health', timeout=3)
    if response.status_code == 200:
        sys.exit(0)
    else:
        sys.exit(1)
except:
    sys.exit(1)
```

## Monitoring et alertes

### Intégration avec des outils de monitoring

**Prometheus exporter** :
```python
from prometheus_client import Gauge

health_status = Gauge('app_health_status', 'Application health status')

@app.route('/health')
def health():
    if check_health():
        health_status.set(1)  # Healthy
        return 'OK', 200
    else:
        health_status.set(0)  # Unhealthy
        return 'Error', 503
```

### Script de surveillance

```bash
#!/bin/bash
# check-containers-health.sh

for container in $(docker ps --format '{{.Names}}'); do
    status=$(docker inspect --format='{{.State.Health.Status}}' $container 2>/dev/null)

    if [ "$status" = "unhealthy" ]; then
        echo "⚠️  Container $container is UNHEALTHY!"
        # Envoyer une alerte (email, Slack, etc.)
        curl -X POST https://hooks.slack.com/services/YOUR/WEBHOOK/URL \
          -d "{\"text\": \"Container $container is unhealthy!\"}"
    fi
done
```

## Résumé

Les health checks sont **essentiels** pour garantir la fiabilité de vos applications conteneurisées.

### Points clés à retenir

✅ **Toujours définir des health checks** en production
✅ **Créer un endpoint /health dédié** dans vos applications
✅ **Adapter le start-period** au temps de démarrage réel
✅ **Garder les checks simples et rapides** (< 1 seconde)
✅ **Utiliser depends_on avec condition** dans Docker Compose
✅ **Surveiller les statuts** avec `docker ps` et `docker inspect`
✅ **Tester les health checks** avant la mise en production

### Commandes essentielles

```bash
# Voir le statut
docker ps

# Détails du health check
docker inspect --format='{{json .State.Health}}' container

# Tester manuellement
docker exec container curl -f http://localhost/health

# Logs du conteneur
docker logs container
```

### Template de base

```yaml
version: '3.8'

services:
  app:
    build: .
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 5s
      start_period: 10s
      retries: 3
```

Les health checks transforment vos conteneurs de "processus en cours d'exécution" en **services fiables et auto-surveillés**, une pratique indispensable pour des déploiements modernes et résilients.

⏭️ [Docker dans un environnement de développement (ex: utilisation avec VS Code Dev Containers)](/11-sujets-intermediaires/04-docker-dans-un-environnement-de-developpement.md)
