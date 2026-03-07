🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 7.4 Communication entre conteneurs

## Introduction

Vous savez maintenant créer des réseaux personnalisés et y connecter des conteneurs. Mais comment ces conteneurs **communiquent-ils réellement** entre eux ? Comment une application web se connecte-t-elle à sa base de données ? Comment un service API appelle-t-il un autre service ?

Cette section explore en profondeur les mécanismes de communication entre conteneurs, les patterns courants, et les meilleures pratiques pour créer des architectures robustes et efficaces.

## Les fondamentaux de la communication

### Trois méthodes principales

Docker offre plusieurs façons de faire communiquer les conteneurs :

1. **Par nom de conteneur** (recommandé) - Via la résolution DNS
2. **Par adresse IP** (déconseillé) - Directement avec l'IP du conteneur
3. **Via l'hôte** - En passant par la machine hôte (pour des cas spécifiques)

### Analogie : Appeler un ami

**Méthode 1 (Par nom)** : "Appelle Alice" - Votre téléphone trouve automatiquement le numéro  
**Méthode 2 (Par IP)** : "Appelle le 06 12 34 56 78" - Vous devez connaître le numéro exact  
**Méthode 3 (Via l'hôte)** : "Appelle maman, elle transmettra à Alice" - Passage par un intermédiaire  

La première méthode est clairement la plus pratique !

## Méthode 1 : Communication par nom (DNS)

### Le principe de la résolution DNS

Sur un réseau Docker personnalisé, Docker fournit un **serveur DNS intégré** qui fait automatiquement correspondre les noms de conteneurs à leurs adresses IP.

```
┌─────────────────────────────────────────────────────┐
│            Réseau Docker : app-network              │
│                                                     │
│  ┌─────────────────┐              ┌──────────────┐  │
│  │  web-server     │              │  db-server   │  │
│  │                 │              │              │  │
│  │ "Connecte-toi à │              │              │  │
│  │  db-server"     │              │ PostgreSQL   │  │
│  │                 │─────────────►│              │  │
│  │                 │  Docker DNS  │ 172.18.0.3   │  │
│  │ 172.18.0.2      │  résout      │              │  │
│  └─────────────────┘  "db-server" └──────────────┘  │
│                       en                            │
│                       172.18.0.3                    │
└─────────────────────────────────────────────────────┘
```

### Exemple pratique : Application web et base de données

#### Étape 1 : Créer le réseau

```bash
docker network create app-network
```

#### Étape 2 : Lancer la base de données

```bash
docker run -d \
  --name postgres-db \
  --network app-network \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=myapp \
  postgres:15
```

#### Étape 3 : Lancer l'application web

```bash
docker run -d \
  --name web-app \
  --network app-network \
  -p 8080:3000 \
  -e DATABASE_HOST=postgres-db \
  -e DATABASE_PORT=5432 \
  -e DATABASE_NAME=myapp \
  -e DATABASE_USER=postgres \
  -e DATABASE_PASSWORD=secret \
  mon-application
```

**Points clés :**
- `DATABASE_HOST=postgres-db` : L'application utilise le **nom du conteneur**
- Docker résout automatiquement `postgres-db` vers l'adresse IP correcte
- Aucun besoin de connaître l'IP réelle de la base de données

#### Code applicatif (exemple Node.js)

```javascript
const { Client } = require('pg');

const client = new Client({
  host: process.env.DATABASE_HOST,     // 'postgres-db'
  port: process.env.DATABASE_PORT,     // 5432
  database: process.env.DATABASE_NAME, // 'myapp'
  user: process.env.DATABASE_USER,     // 'postgres'
  password: process.env.DATABASE_PASSWORD
});

client.connect();
// Connexion réussie grâce à la résolution DNS !
```

### Vérifier la résolution DNS

Vous pouvez tester la résolution DNS depuis un conteneur :

```bash
# Entrer dans le conteneur web-app
docker exec -it web-app sh

# Tester la résolution DNS
ping postgres-db
# PING postgres-db (172.18.0.3): 56 data bytes
# 64 bytes from 172.18.0.3: icmp_seq=0 ttl=64 time=0.123 ms

# Avec nslookup (si disponible)
nslookup postgres-db
# Server:    127.0.0.11
# Address:   127.0.0.11:53
# Name:      postgres-db
# Address:   172.18.0.3
```

### Alias réseau

Vous pouvez donner plusieurs noms à un même conteneur sur un réseau :

```bash
docker network connect app-network postgres-db --alias database --alias db
```

Maintenant, ces trois noms pointent vers le même conteneur :
- `postgres-db`
- `database`
- `db`

**Utilité :** Faciliter la migration ou supporter plusieurs configurations.

### Important : Réseau personnalisé requis

⚠️ **Attention :** La résolution DNS automatique par nom de conteneur fonctionne **uniquement sur les réseaux personnalisés**, pas sur le réseau bridge par défaut.

```bash
# ❌ Ne fonctionne PAS sur le bridge par défaut
docker run -d --name db postgres  
docker run -d --name app mon-app  
docker exec app ping db  # Erreur !  

# ✅ Fonctionne sur un réseau personnalisé
docker network create my-network  
docker run -d --name db --network my-network postgres  
docker run -d --name app --network my-network mon-app  
docker exec app ping db  # Succès !  
```

## Méthode 2 : Communication par IP

### Obtenir l'adresse IP d'un conteneur

```bash
docker inspect postgres-db --format '{{.NetworkSettings.Networks.app-network.IPAddress}}'
# 172.18.0.3
```

### Se connecter par IP (déconseillé)

```bash
docker run -d \
  --name web-app \
  --network app-network \
  -e DATABASE_HOST=172.18.0.3 \
  mon-application
```

### Pourquoi c'est déconseillé ?

❌ **Problèmes avec les IPs :**

1. **IPs dynamiques** - L'IP peut changer si le conteneur est recréé
2. **Peu lisible** - Difficile de savoir quel service est à quelle IP
3. **Maintenance** - Modifications complexes lors de changements d'architecture
4. **Erreurs** - Facile de se tromper ou de référencer un ancien conteneur

✅ **Avec les noms :**

1. **Stable** - Le nom reste le même même si le conteneur est recréé
2. **Lisible** - `postgres-db` est plus clair que `172.18.0.3`
3. **Flexible** - Changements d'infrastructure transparents
4. **Sûr** - Moins de risques d'erreur

### Quand utiliser les IPs ?

Il existe de rares cas où vous pourriez vouloir utiliser des IPs :

- Configuration de pare-feu très spécifique
- Debugging et troubleshooting temporaire
- Intégration avec des systèmes legacy qui nécessitent des IPs fixes

**Mais même dans ces cas, préférez les alias réseau aux IPs directes.**

## Méthode 3 : Communication via l'hôte

### Scénario : Conteneur vers service sur l'hôte

Parfois, un conteneur doit accéder à un service qui tourne sur la machine hôte (pas dans un conteneur).

#### Sur Linux

Utilisez l'IP de la passerelle du réseau Docker :

```bash
# Trouver l'IP de la passerelle
docker network inspect app-network --format '{{.IPAM.Config}}'
# [{172.18.0.0/16  172.18.0.1 map[]}]
# La passerelle est 172.18.0.1
```

Dans votre conteneur, connectez-vous à `172.18.0.1`.

#### Sur Windows et macOS

Utilisez le nom d'hôte spécial `host.docker.internal` :

```bash
docker run -d \
  --name mon-app \
  -e API_HOST=host.docker.internal \
  -e API_PORT=8080 \
  mon-application
```

Cela pointe vers la machine hôte.

### Scénario : Partager le réseau de l'hôte

Avec le mode `--network host`, le conteneur partage directement le réseau de l'hôte :

```bash
docker run -d --network host mon-app
```

**Avantage :** Accès direct à tous les services de l'hôte  
**Inconvénient :** Aucune isolation réseau (moins sécurisé)  

## Patterns de communication

### Pattern 1 : Application web classique (3-tier)

Architecture typique : Frontend → Backend → Database

```bash
# Créer le réseau
docker network create webapp-network

# Base de données (backend only)
docker run -d \
  --name postgres-db \
  --network webapp-network \
  postgres:15

# API Backend
docker run -d \
  --name api-server \
  --network webapp-network \
  -e DB_HOST=postgres-db \
  mon-api:latest

# Frontend (Nginx)
docker run -d \
  --name web-frontend \
  --network webapp-network \
  -p 80:80 \
  -e API_URL=http://api-server:3000 \
  nginx
```

**Communication :**
- Frontend → API : `http://api-server:3000`
- API → Database : `postgres-db:5432`
- Utilisateur → Frontend : `http://localhost:80`

```
┌─────────────────────────────────────────────────┐
│         Réseau : webapp-network                 │
│                                                 │
│  Utilisateur                                    │
│      ↓ (port 80)                                │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐   │
│  │ Frontend │───►│   API    │───►│   DB     │   │
│  │  (Nginx) │    │ (Node.js)│    │(Postgres)│   │
│  └──────────┘    └──────────┘    └──────────┘   │
│   Port 80:80     Interne         Interne        │
│                                                 │
└─────────────────────────────────────────────────┘
```

### Pattern 2 : Microservices avec API Gateway

Architecture : API Gateway → Multiple Services → Databases

```bash
# Réseaux séparés pour sécurité
docker network create frontend-network  
docker network create backend-network  

# Bases de données
docker run -d --name users-db --network backend-network postgres  
docker run -d --name orders-db --network backend-network postgres  

# Microservices
docker run -d --name users-service --network backend-network \
  -e DB_HOST=users-db user-service:latest
docker run -d --name orders-service --network backend-network \
  -e DB_HOST=orders-db order-service:latest

# API Gateway (sur les deux réseaux)
docker run -d --name api-gateway --network frontend-network \
  -p 80:80 api-gateway:latest

docker network connect backend-network api-gateway
```

**Communication :**
- Client → API Gateway : via port 80
- API Gateway → Users Service : `http://users-service:3001`
- API Gateway → Orders Service : `http://orders-service:3002`
- Services → Databases : par nom

```
┌──────────────────────────────────────────────────────────┐
│  Frontend Network                                        │
│                                                          │
│  Client ───► API Gateway ─┐                              │
│              (Port 80)     │                             │
└────────────────────────────┼─────────────────────────────┘
                             │
┌────────────────────────────┼─────────────────────────────┐
│  Backend Network           │                             │
│                            │                             │
│                    ┌───────▼────────┐                    │
│                    │  API Gateway   │                    │
│                    └───┬────────┬───┘                    │
│                        │        │                        │
│              ┌─────────▼──┐  ┌──▼─────────┐              │
│              │   Users    │  │   Orders   │              │
│              │  Service   │  │  Service   │              │
│              └─────┬──────┘  └─────┬──────┘              │
│                    │               │                     │
│              ┌─────▼──────┐  ┌─────▼──────┐              │
│              │  Users DB  │  │ Orders DB  │              │
│              └────────────┘  └────────────┘              │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Pattern 3 : Load Balancing (équilibrage de charge)

Plusieurs instances du même service derrière un load balancer :

```bash
docker network create loadbalanced-network

# Lancer plusieurs instances du service
docker run -d --name app1 --network loadbalanced-network mon-app:latest  
docker run -d --name app2 --network loadbalanced-network mon-app:latest  
docker run -d --name app3 --network loadbalanced-network mon-app:latest  

# Load balancer (Nginx ou HAProxy)
docker run -d --name loadbalancer --network loadbalanced-network \
  -p 80:80 \
  -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf:ro \
  nginx
```

**Configuration Nginx (nginx.conf) :**

```nginx
upstream backend {
    server app1:3000;
    server app2:3000;
    server app3:3000;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
    }
}
```

**Communication :**
- Client → Load Balancer : port 80
- Load Balancer distribue entre app1, app2, app3

### Pattern 4 : Worker Queue (file d'attente)

Architecture avec producteur, queue, et consumers :

```bash
docker network create queue-network

# Message broker (Redis ou RabbitMQ)
docker run -d --name redis-queue --network queue-network redis:7

# Producer (génère des tâches)
docker run -d --name producer --network queue-network \
  -e REDIS_HOST=redis-queue producer-app:latest

# Workers (consomment des tâches)
docker run -d --name worker1 --network queue-network \
  -e REDIS_HOST=redis-queue worker-app:latest
docker run -d --name worker2 --network queue-network \
  -e REDIS_HOST=redis-queue worker-app:latest
```

**Communication :**
- Producer → Redis : `redis-queue:6379`
- Workers → Redis : `redis-queue:6379`

```
┌─────────────────────────────────────────────────┐
│         Réseau : queue-network                  │
│                                                 │
│  ┌──────────┐                                   │
│  │ Producer │                                   │
│  └─────┬────┘                                   │
│        │                                        │
│        ↓                                        │
│  ┌─────────────┐                                │
│  │    Redis    │                                │
│  │   (Queue)   │                                │
│  └──┬────┬─────┘                                │
│     │    │                                      │
│     ↓    ↓                                      │
│  ┌────┐ ┌────┐                                  │
│  │ W1 │ │ W2 │  (Workers)                       │
│  └────┘ └────┘                                  │
│                                                 │
└─────────────────────────────────────────────────┘
```

### Pattern 5 : Service Mesh (avancé)

Pour des architectures complexes, chaque service a un proxy sidecar :

```bash
# Service A avec son proxy
docker run -d --name service-a --network mesh-network app-a:latest  
docker run -d --name proxy-a --network mesh-network \  
  -e SERVICE_HOST=service-a envoy-proxy:latest

# Service B avec son proxy
docker run -d --name service-b --network mesh-network app-b:latest  
docker run -d --name proxy-b --network mesh-network \  
  -e SERVICE_HOST=service-b envoy-proxy:latest
```

Les services communiquent via leurs proxies, ce qui permet d'ajouter :
- Chiffrement
- Retry logic
- Circuit breaking
- Observabilité

> 💡 **Note :** C'est un pattern avancé, souvent implémenté avec des outils comme Istio ou Linkerd dans Kubernetes.

## Communication bidirectionnelle vs unidirectionnelle

### Unidirectionnelle (One-way)

Un service appelle l'autre, mais pas l'inverse :

```bash
docker network create oneway-network

# Service B (appelé)
docker run -d --name service-b --network oneway-network service-b:latest

# Service A (appelant)
docker run -d --name service-a --network oneway-network \
  -e SERVICE_B_URL=http://service-b:3000 \
  service-a:latest
```

**Flux :** A → B (B ne connaît pas A)

### Bidirectionnelle (Two-way)

Les deux services se connaissent et peuvent s'appeler mutuellement :

```bash
docker network create twoway-network

docker run -d --name service-a --network twoway-network \
  -e SERVICE_B_URL=http://service-b:3000 \
  service-a:latest

docker run -d --name service-b --network twoway-network \
  -e SERVICE_A_URL=http://service-a:3000 \
  service-b:latest
```

**Flux :** A ⇄ B

⚠️ **Attention :** La communication bidirectionnelle peut créer des dépendances circulaires. À éviter si possible.

## Protocoles de communication

### HTTP/HTTPS (le plus courant)

Communication via API REST :

```bash
docker run -d --name api --network app-network mon-api:latest

docker run -d --name client --network app-network \
  -e API_ENDPOINT=http://api:3000/api/v1 \
  mon-client:latest
```

**Dans le code :**

```javascript
// Appel HTTP vers l'API
const response = await fetch('http://api:3000/api/v1/users');  
const users = await response.json();  
```

### TCP/UDP brut

Pour des protocoles custom ou des services comme Redis :

```bash
# Redis (TCP port 6379)
docker run -d --name redis --network app-network redis:7

# Application se connectant à Redis
docker run -d --name app --network app-network \
  -e REDIS_HOST=redis \
  -e REDIS_PORT=6379 \
  mon-app:latest
```

**Dans le code :**

```javascript
const redis = require('redis');  
const client = redis.createClient({  
  host: 'redis',  // Nom du conteneur
  port: 6379
});
```

### gRPC

Communication haute performance entre microservices :

```bash
docker run -d --name grpc-server --network app-network \
  -e GRPC_PORT=50051 \
  mon-grpc-server:latest

docker run -d --name grpc-client --network app-network \
  -e GRPC_SERVER=grpc-server:50051 \
  mon-grpc-client:latest
```

### WebSockets

Communication en temps réel :

```bash
docker run -d --name websocket-server --network app-network \
  -p 8080:8080 \
  websocket-server:latest

docker run -d --name websocket-client --network app-network \
  -e WS_URL=ws://websocket-server:8080 \
  websocket-client:latest
```

## Variables d'environnement pour la configuration

### Passer les informations de connexion

Les variables d'environnement sont la méthode standard pour configurer les connexions :

```bash
docker run -d \
  --name mon-app \
  --network app-network \
  -e DATABASE_HOST=postgres-db \
  -e DATABASE_PORT=5432 \
  -e DATABASE_NAME=myapp \
  -e DATABASE_USER=admin \
  -e DATABASE_PASSWORD=secret \
  -e REDIS_HOST=redis-cache \
  -e REDIS_PORT=6379 \
  -e API_BACKEND_URL=http://api-server:3000 \
  mon-app:latest
```

### Fichier .env avec Docker Compose

Dans Docker Compose, utilisez un fichier `.env` :

```yaml
# docker-compose.yml
services:
  app:
    image: mon-app:latest
    environment:
      - DATABASE_HOST=postgres-db
      - DATABASE_PORT=5432
      - DATABASE_NAME=${DB_NAME}
      - DATABASE_USER=${DB_USER}
      - DATABASE_PASSWORD=${DB_PASSWORD}
    networks:
      - app-network

  postgres-db:
    image: postgres:15
    environment:
      - POSTGRES_DB=${DB_NAME}
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    networks:
      - app-network

networks:
  app-network:
```

```bash
# .env
DB_NAME=myapp  
DB_USER=admin  
DB_PASSWORD=supersecret123  
```

## Docker Links (méthode legacy - à éviter)

### Qu'est-ce que c'était ?

Avant les réseaux personnalisés, Docker utilisait `--link` pour connecter les conteneurs :

```bash
docker run -d --name db postgres  
docker run -d --name app --link db:database mon-app  
```

### Pourquoi ne plus l'utiliser ?

❌ **Problèmes avec --link :**
- Déprécié depuis plusieurs versions de Docker
- Ne fonctionne que sur le réseau bridge par défaut
- Moins flexible que les réseaux personnalisés
- Ordre de création important (le conteneur lié doit exister)
- Pas de résolution DNS bidirectionnelle

✅ **Utilisez plutôt les réseaux personnalisés !**

## Sécurité de la communication

### Principe du moindre privilège réseau

Ne connectez les conteneurs qu'aux réseaux strictement nécessaires :

```bash
# ✅ BON - Séparation claire
docker network create frontend-net  
docker network create backend-net  
docker network create database-net  

# Frontend n'a PAS accès à la base de données
docker run -d --name frontend --network frontend-net nginx

# Backend a accès au frontend et à la database
docker run -d --name backend --network backend-net api  
docker network connect frontend-net backend  
docker network connect database-net backend  

# Database est isolée
docker run -d --name database --network database-net postgres
```

### Chiffrement des communications

#### TLS/SSL entre conteneurs

Pour des données sensibles, utilisez TLS même en interne :

```bash
# Serveur avec certificat
docker run -d --name api-server --network secure-network \
  -v $(pwd)/certs:/certs:ro \
  -e TLS_CERT=/certs/server.crt \
  -e TLS_KEY=/certs/server.key \
  api-secure:latest

# Client configuré pour HTTPS
docker run -d --name client --network secure-network \
  -e API_URL=https://api-server:443 \
  -v $(pwd)/certs/ca.crt:/ca.crt:ro \
  client-secure:latest
```

#### Réseau overlay chiffré

Pour les réseaux overlay (multi-hôtes), activez le chiffrement :

```bash
docker network create \
  --driver overlay \
  --opt encrypted \
  secure-overlay-network
```

### Firewall et isolation

Docker intègre un pare-feu basé sur iptables. Vous pouvez désactiver la communication inter-conteneurs :

```bash
docker network create \
  --opt "com.docker.network.bridge.enable_icc=false" \
  isolated-network
```

Les conteneurs sur ce réseau ne peuvent communiquer qu'avec l'extérieur, pas entre eux.

## Debugging de la communication

### Vérifier la connectivité réseau

#### Tester avec ping

```bash
docker exec conteneur1 ping conteneur2
```

Si ça fonctionne, la connectivité réseau de base est OK.

#### Tester avec curl

```bash
docker exec conteneur1 curl http://conteneur2:3000/health
```

Teste la connectivité HTTP et le service lui-même.

#### Tester avec telnet

```bash
docker exec conteneur1 telnet conteneur2 5432
```

Vérifie si le port est ouvert et accessible.

### Utiliser un conteneur de debugging

Lancez un conteneur avec des outils réseau :

```bash
docker run -it --network app-network nicolaka/netshoot

# À l'intérieur, utilisez:
nslookup postgres-db  
ping api-server  
curl http://web-frontend  
traceroute database  
tcpdump -i eth0  
```

### Inspecter les logs

Les logs révèlent souvent les problèmes de connexion :

```bash
docker logs mon-app

# Rechercher des erreurs de connexion
docker logs mon-app 2>&1 | grep -i "connection"  
docker logs mon-app 2>&1 | grep -i "refused"  
docker logs mon-app 2>&1 | grep -i "timeout"  
```

### Vérifier la configuration réseau

```bash
# Conteneurs sur le réseau
docker network inspect app-network

# Configuration réseau d'un conteneur
docker inspect mon-app --format '{{json .NetworkSettings.Networks}}' | jq
```

### Analyser le trafic réseau

Capturer les paquets pour analyse approfondie :

```bash
# Depuis l'hôte
docker exec mon-app tcpdump -i eth0 -w /tmp/capture.pcap

# Ou avec netshoot
docker run --network container:mon-app nicolaka/netshoot \
  tcpdump -i eth0 -w /tmp/capture.pcap
```

## Problèmes courants et solutions

### Problème 1 : "Connection refused"

**Symptômes :**
```
Error: connect ECONNREFUSED 172.18.0.3:3000
```

**Causes possibles :**

1. Le service n'écoute pas sur le bon port
2. Le service n'est pas démarré
3. Le conteneur utilise `localhost` au lieu de `0.0.0.0`

**Solutions :**

```bash
# Vérifier que le service écoute
docker exec api-server netstat -tulpn | grep 3000

# Vérifier les logs
docker logs api-server

# Le service doit écouter sur 0.0.0.0, pas 127.0.0.1
# Mauvais: app.listen(3000, 'localhost')
# Bon: app.listen(3000, '0.0.0.0')
```

### Problème 2 : "Unknown host" ou "Could not resolve"

**Symptômes :**
```
Error: getaddrinfo ENOTFOUND postgres-db
```

**Causes possibles :**

1. Les conteneurs ne sont pas sur le même réseau
2. Utilisation du réseau bridge par défaut (pas de DNS)
3. Faute de frappe dans le nom

**Solutions :**

```bash
# Vérifier les réseaux
docker network inspect app-network

# Créer un réseau personnalisé si nécessaire
docker network create app-network  
docker network connect app-network conteneur1  
docker network connect app-network conteneur2  

# Vérifier le nom exact
docker ps --format "{{.Names}}"
```

### Problème 3 : "Connection timeout"

**Symptômes :**
```
Error: connect ETIMEDOUT
```

**Causes possibles :**

1. Le service est trop lent à démarrer
2. Pare-feu ou règles réseau bloquantes
3. Mauvaise configuration réseau

**Solutions :**

```bash
# Augmenter le timeout dans le code
# Ajouter des healthchecks
# Vérifier les règles iptables

# Tester la connectivité basique
docker exec conteneur1 ping conteneur2  
docker exec conteneur1 telnet conteneur2 3000  
```

### Problème 4 : Ordre de démarrage

**Symptôme :** L'application démarre avant la base de données.

**Solution :** Implémenter une logique de retry ou utiliser `depends_on` avec healthcheck dans Docker Compose :

```yaml
services:
  app:
    image: mon-app
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
```

### Problème 5 : Communication intermittente

**Causes possibles :**

1. Load balancing avec sessions non persistantes
2. DNS cache obsolète
3. Conteneurs qui redémarrent

**Solutions :**

```bash
# Vérifier les redémarrages
docker ps -a

# Surveiller les logs en temps réel
docker logs -f mon-app

# Implémenter des healthchecks
# Utiliser des retry mechanisms dans le code
```

## Bonnes pratiques

### 1. Toujours utiliser des noms, jamais des IPs

```bash
# ✅ BON
-e DATABASE_HOST=postgres-db

# ❌ MAUVAIS
-e DATABASE_HOST=172.18.0.3
```

### 2. Utiliser des variables d'environnement

Externalisez toute la configuration :

```bash
# ✅ BON - Configuration flexible
docker run -d \
  -e DB_HOST=postgres \
  -e DB_PORT=5432 \
  mon-app

# ❌ MAUVAIS - Configuration codée en dur
```

### 3. Implémenter des healthchecks

Vérifiez que les services sont réellement prêts :

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:3000/health || exit 1
```

### 4. Utiliser des retry mechanisms

Gérez les échecs temporaires :

```javascript
// Exemple avec retry
async function connectWithRetry(maxRetries = 5) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      await connectToDatabase('postgres-db');
      return;
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await sleep(2000 * (i + 1)); // Backoff exponentiel
    }
  }
}
```

### 5. Documenter l'architecture

Créez un schéma de communication :

```
Services et leurs dépendances:

web-frontend (port 80)
  ↓ appelle
api-backend (port 3000)
  ↓ appelle
postgres-db (port 5432)  
redis-cache (port 6379)  
```

### 6. Limiter l'exposition des ports

```bash
# ✅ BON - Seul le frontend est exposé publiquement
docker run -d -p 80:80 frontend  
docker run -d api-backend  # Pas de -p  
docker run -d database     # Pas de -p  

# ❌ MAUVAIS - Tout est exposé
docker run -d -p 80:80 frontend  
docker run -d -p 3000:3000 api-backend  
docker run -d -p 5432:5432 database  
```

### 7. Séparer les réseaux par fonction

```bash
docker network create public-network  
docker network create private-network  

# Services publics
docker run -d --network public-network frontend

# Services internes uniquement
docker run -d --network private-network database
```

### 8. Tester la communication en développement

Avant de déployer, testez :

```bash
# Script de test de connectivité
#!/bin/bash
docker exec api-server curl -f http://database:5432 || echo "DB non accessible"  
docker exec frontend curl -f http://api-server:3000/health || echo "API non accessible"  
```

## À retenir

🔑 **Points clés :**

### Communication
- Utilisez les **noms de conteneurs** pour la communication (DNS automatique)
- Les **réseaux personnalisés** sont essentiels pour le DNS
- Évitez les **adresses IP directes** (sauf cas très spécifiques)

### Configuration
- Passez la configuration via **variables d'environnement**
- N'utilisez **jamais** `--link` (deprecated)
- Externalisez les secrets et configurations sensibles

### Architecture
- Séparez les réseaux par **fonction** (frontend, backend, database)
- Appliquez le principe du **moindre privilège**
- Documentez les **flux de communication**

### Debugging
- Utilisez `ping`, `curl`, `telnet` pour tester
- Analysez les **logs** en premier
- Utilisez des **conteneurs de debugging** (netshoot)

### Sécurité
- **Isolez** les services sensibles (databases)
- N'exposez publiquement que le **strict nécessaire**
- Considérez le **chiffrement** pour les données sensibles

## Conclusion

La communication entre conteneurs est un aspect fondamental de Docker. En maîtrisant ces concepts, vous pouvez créer des architectures robustes, scalables et sécurisées.

La clé du succès : **simplicité** (noms plutôt qu'IPs), **isolation** (réseaux séparés), et **documentation** (architecture claire).

## Prochaine étape

Avec vos connaissances sur les réseaux et la communication, vous êtes prêt pour le **chapitre 8 : Docker Compose**, où vous apprendrez à orchestrer facilement des applications multi-conteneurs avec toutes ces bonnes pratiques de communication intégrées dans un seul fichier de configuration.

⏭️ [Docker Compose](/08-docker-compose/README.md)
