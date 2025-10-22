🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 10.4 Logging et monitoring

## Introduction

Imaginez que vous avez déployé votre application Docker en production et qu'un utilisateur vous signale un problème. Comment savoir ce qui s'est passé ? Comment identifier la cause du problème ? C'est là qu'interviennent le logging (journalisation) et le monitoring (surveillance).

Le **logging** consiste à enregistrer les événements qui se produisent dans vos applications et conteneurs, tandis que le **monitoring** permet de surveiller en temps réel les performances et la santé de vos systèmes.

Dans cette section, nous allons apprendre à mettre en place une stratégie efficace de logging et monitoring pour vos conteneurs Docker.

## Pourquoi le logging et le monitoring sont essentiels ?

### Sans logging et monitoring

Vous êtes "aveugle" :
- Impossible de savoir pourquoi une application a planté
- Pas de trace des erreurs passées
- Difficile de détecter les problèmes de performance
- Aucun historique pour l'analyse

### Avec logging et monitoring

Vous avez de la visibilité :
- Historique complet des événements
- Détection proactive des problèmes
- Analyse des tendances et des performances
- Débogage facilité
- Conformité et audit

## Comprendre les logs Docker

### Les trois types de logs

Docker gère trois types de logs principaux :

1. **Logs des conteneurs (stdout/stderr)** : Tout ce que votre application écrit sur la sortie standard
2. **Logs du daemon Docker** : Logs du moteur Docker lui-même
3. **Logs des événements Docker** : Événements système (création, arrêt de conteneurs, etc.)

### Le fonctionnement de base

Par défaut, Docker capture tout ce que votre application écrit sur `stdout` (sortie standard) et `stderr` (sortie d'erreur) et les stocke.

```javascript
// Application Node.js
console.log('Message normal');        // → stdout → logs Docker
console.error('Message d\'erreur');   // → stderr → logs Docker
```

```python
# Application Python
print('Message normal')                # → stdout → logs Docker
import sys
print('Erreur', file=sys.stderr)      # → stderr → logs Docker
```

## Consulter les logs : commandes de base

### docker logs

La commande la plus simple pour voir les logs d'un conteneur :

```bash
# Voir tous les logs d'un conteneur
docker logs mon-conteneur

# Voir les dernières lignes (comme tail)
docker logs --tail 50 mon-conteneur

# Suivre les logs en temps réel (comme tail -f)
docker logs -f mon-conteneur

# Afficher les timestamps
docker logs -t mon-conteneur

# Logs depuis une date/heure spécifique
docker logs --since 2024-01-01T10:00:00 mon-conteneur

# Logs depuis les 10 dernières minutes
docker logs --since 10m mon-conteneur

# Logs jusqu'à une certaine date
docker logs --until 2024-01-01T12:00:00 mon-conteneur

# Combiner plusieurs options
docker logs -f --tail 100 -t mon-conteneur
```

### Exemples pratiques

```bash
# Déboguer une application qui ne démarre pas
docker logs mon-app

# Surveiller les logs en temps réel pendant un test
docker logs -f mon-app

# Voir uniquement les derniers logs d'erreur
docker logs --tail 20 mon-app 2>&1 | grep -i error

# Exporter les logs dans un fichier
docker logs mon-app > logs.txt 2>&1
```

### Avec Docker Compose

```bash
# Voir les logs de tous les services
docker-compose logs

# Voir les logs d'un service spécifique
docker-compose logs app

# Suivre les logs en temps réel
docker-compose logs -f

# Logs des 100 dernières lignes de tous les services
docker-compose logs --tail 100
```

## Configurer le logging

### Drivers de logging

Docker supporte plusieurs drivers de logging qui déterminent où et comment les logs sont stockés.

#### Drivers disponibles

| Driver | Description | Cas d'usage |
|--------|-------------|-------------|
| `json-file` | Stockage local en JSON (par défaut) | Développement, petites applications |
| `syslog` | Envoie vers syslog | Intégration avec systèmes existants |
| `journald` | Envoie vers systemd journal | Systèmes Linux avec systemd |
| `gelf` | Format Graylog Extended Log | Graylog, ELK Stack |
| `fluentd` | Envoie vers Fluentd | Architectures microservices |
| `awslogs` | AWS CloudWatch Logs | Applications sur AWS |
| `gcplogs` | Google Cloud Logging | Applications sur GCP |
| `splunk` | Envoie vers Splunk | Grandes entreprises |
| `none` | Désactive le logging | Conteneurs temporaires |

#### Vérifier le driver actuel

```bash
# Voir le driver de logging d'un conteneur
docker inspect -f '{{.HostConfig.LogConfig.Type}}' mon-conteneur
```

### Configurer le driver json-file (par défaut)

Le driver par défaut stocke les logs en JSON. Il est important de limiter la taille pour éviter de remplir le disque.

#### Configuration globale (daemon Docker)

```json
// /etc/docker/daemon.json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3",
    "labels": "production_status",
    "env": "os,customer"
  }
}
```

Après modification, redémarrer Docker :

```bash
sudo systemctl restart docker
```

#### Configuration par conteneur

```bash
# Lancer un conteneur avec des options de logging spécifiques
docker run -d \
  --name mon-app \
  --log-driver json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  mon-image
```

#### Avec Docker Compose

```yaml
version: '3.8'

services:
  app:
    image: mon-app
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
        labels: "production"
```

**Paramètres importants :**
- `max-size` : Taille maximale d'un fichier de log avant rotation (10m = 10 mégaoctets)
- `max-file` : Nombre de fichiers de log à conserver (3 = 3 rotations)
- **Total stockage = max-size × max-file** (ex: 10m × 3 = 30m maximum par conteneur)

### Configurer syslog

Syslog est un protocole standard pour la gestion des logs sous Linux.

```bash
# Envoyer les logs vers le syslog local
docker run -d \
  --log-driver syslog \
  --log-opt syslog-address=unix:///dev/log \
  --log-opt tag="mon-app" \
  mon-image

# Envoyer vers un serveur syslog distant
docker run -d \
  --log-driver syslog \
  --log-opt syslog-address=tcp://192.168.1.100:514 \
  --log-opt tag="mon-app" \
  mon-image
```

Avec Docker Compose :

```yaml
version: '3.8'

services:
  app:
    image: mon-app
    logging:
      driver: syslog
      options:
        syslog-address: "tcp://192.168.1.100:514"
        tag: "mon-app"
```

### Configurer pour AWS CloudWatch

```bash
docker run -d \
  --log-driver awslogs \
  --log-opt awslogs-region=eu-west-1 \
  --log-opt awslogs-group=mon-groupe \
  --log-opt awslogs-stream=mon-stream \
  mon-image
```

Avec Docker Compose :

```yaml
version: '3.8'

services:
  app:
    image: mon-app
    logging:
      driver: awslogs
      options:
        awslogs-region: eu-west-1
        awslogs-group: mon-groupe-logs
        awslogs-stream: mon-app
        awslogs-create-group: "true"
```

## Structurer les logs dans vos applications

### Format des logs

Utilisez un format structuré (JSON) pour faciliter le parsing et l'analyse.

#### ❌ Logs non structurés

```javascript
// Difficile à parser et analyser
console.log('User john logged in from 192.168.1.1');
console.log('Request took 150ms');
```

#### ✅ Logs structurés (JSON)

```javascript
// Facile à parser et analyser
console.log(JSON.stringify({
  timestamp: new Date().toISOString(),
  level: 'info',
  message: 'User logged in',
  user: 'john',
  ip: '192.168.1.1'
}));

console.log(JSON.stringify({
  timestamp: new Date().toISOString(),
  level: 'info',
  message: 'Request completed',
  duration_ms: 150,
  endpoint: '/api/users'
}));
```

### Utiliser des bibliothèques de logging

#### Node.js avec Winston

```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.Console()
  ]
});

// Utilisation
logger.info('User logged in', {
  user: 'john',
  ip: '192.168.1.1'
});

logger.error('Database connection failed', {
  error: err.message,
  database: 'postgresql'
});
```

#### Python avec structlog

```python
import structlog

# Configuration
structlog.configure(
    processors=[
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.JSONRenderer()
    ]
)

logger = structlog.get_logger()

# Utilisation
logger.info('user_logged_in', user='john', ip='192.168.1.1')
logger.error('db_connection_failed',
             error=str(err),
             database='postgresql')
```

### Niveaux de logs

Utilisez les bons niveaux de log selon la situation :

| Niveau | Description | Exemple |
|--------|-------------|---------|
| **DEBUG** | Informations détaillées pour le débogage | Variables, états internes |
| **INFO** | Événements normaux de l'application | Démarrages, requêtes HTTP |
| **WARNING** | Situations anormales mais gérées | Retry, deprecated features |
| **ERROR** | Erreurs qui n'empêchent pas l'application | Échec d'une requête API |
| **CRITICAL** | Erreurs graves, application instable | Base de données inaccessible |

```javascript
// Exemple avec différents niveaux
logger.debug('Connexion database tentée', { attempt: 1 });
logger.info('Utilisateur connecté', { user: 'john' });
logger.warn('Taux limite API atteint, retry dans 60s');
logger.error('Échec envoi email', { error: err.message });
logger.critical('Base de données inaccessible!');
```

## Monitoring des conteneurs

### Commandes de base

#### docker stats

Affiche les statistiques en temps réel :

```bash
# Voir les stats de tous les conteneurs en cours d'exécution
docker stats

# Voir les stats d'un conteneur spécifique
docker stats mon-conteneur

# Afficher les stats une seule fois (sans rafraîchissement)
docker stats --no-stream

# Format personnalisé
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

Sortie typique :
```
CONTAINER ID   NAME        CPU %   MEM USAGE / LIMIT     MEM %   NET I/O       BLOCK I/O
abc123         mon-app     2.5%    150MiB / 512MiB      29.3%   1.2MB / 800kB 5MB / 2MB
```

#### docker inspect

Obtenir des informations détaillées :

```bash
# Informations complètes
docker inspect mon-conteneur

# État du conteneur
docker inspect -f '{{.State.Status}}' mon-conteneur

# Utilisation mémoire
docker inspect -f '{{.HostConfig.Memory}}' mon-conteneur

# Variables d'environnement
docker inspect -f '{{.Config.Env}}' mon-conteneur
```

### Monitoring avec Docker Compose

```bash
# Voir le statut de tous les services
docker-compose ps

# Voir les ressources utilisées
docker-compose top
```

### Health checks

Ajoutez des health checks pour surveiller l'état de santé de vos conteneurs.

#### Dans un Dockerfile

```dockerfile
FROM node:18-alpine

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .

# Health check : vérifie que l'app répond sur le port 3000
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node healthcheck.js || exit 1

CMD ["node", "server.js"]
```

```javascript
// healthcheck.js
const http = require('http');

const options = {
  host: 'localhost',
  port: 3000,
  path: '/health',
  timeout: 2000
};

const request = http.request(options, (res) => {
  if (res.statusCode == 200) {
    process.exit(0);
  } else {
    process.exit(1);
  }
});

request.on('error', () => {
  process.exit(1);
});

request.end();
```

#### Avec Docker Compose

```yaml
version: '3.8'

services:
  app:
    image: mon-app
    healthcheck:
      test: ["CMD", "node", "healthcheck.js"]
      interval: 30s
      timeout: 3s
      retries: 3
      start_period: 40s
```

Vérifier le statut :

```bash
# Voir l'état de santé
docker ps

# Détails du health check
docker inspect --format='{{json .State.Health}}' mon-conteneur | jq
```

## Solutions de monitoring avancées

### 1. Prometheus + Grafana

**Prometheus** collecte les métriques et **Grafana** les visualise.

#### docker-compose.yml

```yaml
version: '3.8'

services:
  # Votre application
  app:
    image: mon-app
    ports:
      - "3000:3000"
    networks:
      - monitoring

  # Prometheus pour collecter les métriques
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    ports:
      - "9090:9090"
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
    networks:
      - monitoring

  # Grafana pour visualiser
  grafana:
    image: grafana/grafana:latest
    volumes:
      - grafana-data:/var/lib/grafana
    ports:
      - "3001:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    networks:
      - monitoring

  # cAdvisor pour les métriques des conteneurs
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    ports:
      - "8080:8080"
    networks:
      - monitoring

volumes:
  prometheus-data:
  grafana-data:

networks:
  monitoring:
    driver: bridge
```

#### prometheus.yml

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  # Métriques de Prometheus lui-même
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # Métriques des conteneurs via cAdvisor
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']

  # Métriques de votre application (si elle expose /metrics)
  - job_name: 'mon-app'
    static_configs:
      - targets: ['app:3000']
```

Accès :
- Prometheus : http://localhost:9090
- Grafana : http://localhost:3001 (admin/admin)
- cAdvisor : http://localhost:8080

### 2. ELK Stack (Elasticsearch, Logstash, Kibana)

Pour centraliser et analyser les logs.

```yaml
version: '3.8'

services:
  # Votre application
  app:
    image: mon-app
    logging:
      driver: gelf
      options:
        gelf-address: "udp://localhost:12201"
        tag: "mon-app"

  # Elasticsearch pour stocker les logs
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - xpack.security.enabled=false
    volumes:
      - es-data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"

  # Logstash pour traiter les logs
  logstash:
    image: docker.elastic.co/logstash/logstash:8.11.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    ports:
      - "12201:12201/udp"
    depends_on:
      - elasticsearch

  # Kibana pour visualiser
  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch

volumes:
  es-data:
```

### 3. Solution simple : Portainer

Portainer offre une interface web pour gérer et monitorer Docker.

```bash
# Créer un volume pour les données Portainer
docker volume create portainer_data

# Lancer Portainer
docker run -d \
  -p 9000:9000 \
  --name portainer \
  --restart always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

Accès : http://localhost:9000

Fonctionnalités :
- Vue d'ensemble de tous les conteneurs
- Logs en temps réel
- Statistiques de ressources
- Gestion des conteneurs via interface web

## Centralisation des logs

### Pourquoi centraliser ?

Avec plusieurs conteneurs et services, il devient difficile de consulter les logs individuellement. La centralisation permet :

- **Vue unifiée** : Tous les logs au même endroit
- **Recherche facilitée** : Recherche globale dans tous les logs
- **Corrélation** : Relier les événements entre services
- **Rétention** : Conserver les logs longtemps
- **Analyse** : Détecter des patterns et anomalies

### Solution simple : Fluentd

Fluentd est un collecteur de logs léger et flexible.

```yaml
version: '3.8'

services:
  # Votre application
  app:
    image: mon-app
    logging:
      driver: fluentd
      options:
        fluentd-address: localhost:24224
        tag: mon-app

  # Fluentd pour collecter les logs
  fluentd:
    image: fluent/fluentd:v1.16-1
    volumes:
      - ./fluentd/fluent.conf:/fluentd/etc/fluent.conf
      - ./logs:/fluentd/log
    ports:
      - "24224:24224"
      - "24224:24224/udp"
```

Configuration Fluentd (fluent.conf) :

```
# Recevoir les logs depuis Docker
<source>
  @type forward
  port 24224
  bind 0.0.0.0
</source>

# Filtrer et enrichir les logs
<filter mon-app>
  @type record_transformer
  <record>
    hostname "#{Socket.gethostname}"
    environment "production"
  </record>
</filter>

# Sauvegarder dans un fichier
<match mon-app>
  @type file
  path /fluentd/log/mon-app.log
  <buffer>
    timekey 1d
    timekey_use_utc true
    timekey_wait 10m
  </buffer>
</match>
```

## Exemples pratiques complets

### Application Node.js avec logging structuré

```javascript
// logger.js
const winston = require('winston');

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: {
    service: 'mon-app',
    environment: process.env.NODE_ENV
  },
  transports: [
    new winston.transports.Console()
  ]
});

module.exports = logger;
```

```javascript
// server.js
const express = require('express');
const logger = require('./logger');

const app = express();
const port = process.env.PORT || 3000;

// Middleware pour logger toutes les requêtes
app.use((req, res, next) => {
  const start = Date.now();

  res.on('finish', () => {
    const duration = Date.now() - start;
    logger.info('HTTP Request', {
      method: req.method,
      url: req.url,
      status: res.statusCode,
      duration_ms: duration,
      ip: req.ip
    });
  });

  next();
});

// Routes
app.get('/', (req, res) => {
  logger.debug('Home page accessed');
  res.send('Hello World!');
});

app.get('/health', (req, res) => {
  res.json({ status: 'healthy', timestamp: new Date().toISOString() });
});

app.get('/error', (req, res) => {
  logger.error('Test error endpoint accessed', {
    route: '/error',
    user: req.query.user
  });
  res.status(500).json({ error: 'Test error' });
});

// Gestion des erreurs
app.use((err, req, res, next) => {
  logger.error('Unhandled error', {
    error: err.message,
    stack: err.stack,
    url: req.url,
    method: req.method
  });
  res.status(500).json({ error: 'Internal server error' });
});

// Démarrage
app.listen(port, () => {
  logger.info('Server started', {
    port: port,
    environment: process.env.NODE_ENV
  });
});

// Gestion de l'arrêt propre
process.on('SIGTERM', () => {
  logger.info('SIGTERM received, shutting down gracefully');
  process.exit(0);
});
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: production
      LOG_LEVEL: info
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:3000/health"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 10s
```

### Application Python avec monitoring

```python
# logger.py
import logging
import structlog
import sys

def configure_logging(level='INFO'):
    """Configure le logging structuré"""
    logging.basicConfig(
        format="%(message)s",
        stream=sys.stdout,
        level=level
    )

    structlog.configure(
        processors=[
            structlog.processors.TimeStamper(fmt="iso"),
            structlog.stdlib.add_log_level,
            structlog.processors.StackInfoRenderer(),
            structlog.processors.format_exc_info,
            structlog.processors.JSONRenderer()
        ],
        wrapper_class=structlog.stdlib.BoundLogger,
        context_class=dict,
        logger_factory=structlog.stdlib.LoggerFactory(),
        cache_logger_on_first_use=True,
    )

    return structlog.get_logger()
```

```python
# app.py
from flask import Flask, jsonify, request
from logger import configure_logging
import os
import time

app = Flask(__name__)
logger = configure_logging(level=os.getenv('LOG_LEVEL', 'INFO'))

@app.before_request
def log_request():
    """Log toutes les requêtes entrantes"""
    request.start_time = time.time()

@app.after_request
def log_response(response):
    """Log toutes les réponses"""
    duration = (time.time() - request.start_time) * 1000
    logger.info(
        'http_request',
        method=request.method,
        path=request.path,
        status=response.status_code,
        duration_ms=round(duration, 2),
        ip=request.remote_addr
    )
    return response

@app.route('/')
def home():
    logger.debug('home_accessed')
    return jsonify({'message': 'Hello World!'})

@app.route('/health')
def health():
    return jsonify({
        'status': 'healthy',
        'service': 'mon-app',
        'version': '1.0.0'
    })

@app.errorhandler(Exception)
def handle_error(error):
    logger.error(
        'unhandled_error',
        error=str(error),
        error_type=type(error).__name__,
        path=request.path
    )
    return jsonify({'error': 'Internal server error'}), 500

if __name__ == '__main__':
    logger.info(
        'starting_server',
        port=int(os.getenv('PORT', 5000)),
        environment=os.getenv('FLASK_ENV', 'production')
    )
    app.run(
        host='0.0.0.0',
        port=int(os.getenv('PORT', 5000))
    )
```

## Bonnes pratiques

### 1. Toujours logger vers stdout/stderr

```dockerfile
# ✅ Bon : logs vers stdout
CMD ["python", "app.py"]

# ❌ Éviter : logs vers fichier dans le conteneur
CMD ["python", "app.py", ">", "/var/log/app.log"]
```

**Pourquoi ?** Docker capture automatiquement stdout/stderr. Écrire dans des fichiers complexifie la récupération des logs.

### 2. Utiliser des logs structurés (JSON)

```javascript
// ❌ Difficile à parser
console.log('User john logged in at 2024-01-15 10:30:00');

// ✅ Facile à parser et analyser
logger.info('user_login', {
  user: 'john',
  timestamp: new Date().toISOString()
});
```

### 3. Inclure du contexte utile

```javascript
// ❌ Pas assez d'information
logger.error('Database error');

// ✅ Contexte complet
logger.error('Database connection failed', {
  error: err.message,
  database: 'postgresql',
  host: dbHost,
  retry_count: retryCount,
  operation: 'user_fetch'
});
```

### 4. Limiter la taille des logs

```yaml
# Toujours définir des limites
logging:
  driver: json-file
  options:
    max-size: "10m"    # Taille max par fichier
    max-file: "3"      # Nombre de fichiers
```

### 5. Ne jamais logger de secrets

```javascript
// ❌ DANGEREUX !
logger.info('Database connection', { password: dbPassword });

// ✅ Sécurisé
logger.info('Database connection', {
  host: dbHost,
  database: dbName,
  password: '[REDACTED]'
});
```

### 6. Utiliser les bons niveaux de log

```javascript
logger.debug('Variable x =', x);           // Développement uniquement
logger.info('User logged in');             // Événements normaux
logger.warn('Rate limit approaching');     // Avertissements
logger.error('Failed to send email');      // Erreurs récupérables
logger.critical('Database unreachable');   // Erreurs critiques
```

### 7. Ajouter des corrélation IDs

Pour tracer une requête à travers plusieurs services :

```javascript
const { v4: uuidv4 } = require('uuid');

app.use((req, res, next) => {
  req.correlationId = req.headers['x-correlation-id'] || uuidv4();
  res.setHeader('x-correlation-id', req.correlationId);

  logger.info('Request received', {
    correlation_id: req.correlationId,
    method: req.method,
    url: req.url
  });

  next();
});
```

### 8. Monitorer les métriques importantes

**Métriques à surveiller :**
- Utilisation CPU et mémoire
- Nombre de requêtes par seconde
- Temps de réponse moyen
- Taux d'erreur
- Nombre de conteneurs en cours d'exécution
- Échecs de health checks

## Alerting

Configurez des alertes pour être prévenu des problèmes.

### Avec Prometheus Alertmanager

```yaml
# prometheus-rules.yml
groups:
  - name: container_alerts
    interval: 30s
    rules:
      - alert: HighCPUUsage
        expr: rate(container_cpu_usage_seconds_total[5m]) > 0.8
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "CPU usage is high"
          description: "Container {{ $labels.name }} CPU usage is above 80%"

      - alert: HighMemoryUsage
        expr: container_memory_usage_bytes / container_spec_memory_limit_bytes > 0.9
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Memory usage is high"
          description: "Container {{ $labels.name }} memory usage is above 90%"

      - alert: ContainerDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Container is down"
          description: "Container {{ $labels.instance }} is down"
```

## Checklist

Avant de déployer en production, vérifiez :

- [ ] Logs configurés pour écrire vers stdout/stderr
- [ ] Logs structurés (JSON) pour faciliter l'analyse
- [ ] Limites de taille configurées (max-size, max-file)
- [ ] Health checks configurés sur tous les conteneurs
- [ ] Niveaux de log appropriés (INFO en production, pas DEBUG)
- [ ] Aucun secret dans les logs
- [ ] Contexte suffisant dans les messages de log
- [ ] Correlation IDs pour tracer les requêtes
- [ ] Solution de centralisation des logs en place
- [ ] Dashboards de monitoring configurés
- [ ] Alertes configurées pour les métriques critiques
- [ ] Documentation de la stratégie de logging

## Outils recommandés

### Logging
- **Développement** : docker logs, Docker Desktop
- **Production simple** : Fluentd + Fichiers ou Syslog
- **Production avancée** : ELK Stack, Loki + Grafana, Cloud solutions (AWS CloudWatch, GCP Logging)

### Monitoring
- **Simple** : docker stats, Portainer
- **Avancé** : Prometheus + Grafana, Datadog, New Relic

### Bibliothèques de logging
- **Node.js** : Winston, Pino, Bunyan
- **Python** : structlog, python-json-logger
- **Java** : Logback, Log4j2
- **Go** : Zap, Logrus

## Conclusion

Le logging et le monitoring sont essentiels pour maintenir des applications Docker en production. Les points clés à retenir :

1. **Toujours logger vers stdout/stderr** : Docker les capture automatiquement
2. **Utiliser des logs structurés** : JSON pour faciliter l'analyse
3. **Limiter la taille des logs** : Éviter de remplir le disque
4. **Centraliser les logs** : Vue unifiée de tous vos services
5. **Monitorer les métriques** : CPU, mémoire, santé des conteneurs
6. **Configurer des alertes** : Détection proactive des problèmes
7. **Ne jamais logger de secrets** : Sécurité avant tout

Avec une bonne stratégie de logging et monitoring, vous aurez la visibilité nécessaire pour maintenir et améliorer vos applications Docker en production.

## Ressources supplémentaires

- [Docker Logging Documentation](https://docs.docker.com/config/containers/logging/)
- [12-Factor App - Logs](https://12factor.net/logs)
- [Prometheus Best Practices](https://prometheus.io/docs/practices/)
- [Grafana Documentation](https://grafana.com/docs/)
- [ELK Stack Guide](https://www.elastic.co/guide/index.html)

⏭️ [Nommage et organisation des ressources](/10-bonnes-pratiques/05-nommage-et-organisation-des-ressources.md)
