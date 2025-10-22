🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 7.3 Créer et gérer des réseaux personnalisés

## Introduction

Vous connaissez maintenant les principes des réseaux Docker et les différents types disponibles. Il est temps de passer à la pratique : **créer et gérer vos propres réseaux personnalisés**.

Bien que Docker fournisse des réseaux par défaut, créer vos propres réseaux personnalisés offre de nombreux avantages :

✅ **Meilleur contrôle** - Configuration adaptée à vos besoins
✅ **Isolation** - Séparation logique entre projets ou environnements
✅ **Résolution DNS automatique** - Les conteneurs se trouvent par leur nom
✅ **Sécurité renforcée** - Contrôle précis de qui peut communiquer avec qui
✅ **Organisation** - Structure claire de votre infrastructure

Cette section vous apprendra à maîtriser la création et la gestion de réseaux personnalisés de manière professionnelle.

## Pourquoi créer des réseaux personnalisés ?

### Le problème avec le réseau par défaut

Lorsque vous lancez des conteneurs sans spécifier de réseau, ils utilisent le réseau `bridge` par défaut. Ce réseau a des limitations :

```bash
# Ces conteneurs utilisent le réseau bridge par défaut
docker run -d --name app1 nginx
docker run -d --name app2 nginx

# ❌ Problème : app1 ne peut pas trouver app2 par son nom
docker exec app1 ping app2
# Erreur: ping: bad address 'app2'
```

### La solution : Réseaux personnalisés

Avec un réseau personnalisé, tout devient plus simple :

```bash
# Créer un réseau personnalisé
docker network create mon-reseau

# Lancer les conteneurs sur ce réseau
docker run -d --name app1 --network mon-reseau nginx
docker run -d --name app2 --network mon-reseau nginx

# ✅ Succès : app1 trouve app2 par son nom
docker exec app1 ping app2
# PING app2 (172.18.0.3): 56 data bytes
# 64 bytes from 172.18.0.3: icmp_seq=0 ttl=64 time=0.123 ms
```

## Créer un réseau personnalisé

### Commande de base

La commande la plus simple pour créer un réseau :

```bash
docker network create nom-du-reseau
```

**Exemple :**

```bash
docker network create app-network
```

Cette commande crée un réseau bridge personnalisé avec des paramètres par défaut.

### Vérifier la création

```bash
# Lister tous les réseaux
docker network ls

# Sortie typique
NETWORK ID     NAME           DRIVER    SCOPE
abc123def456   bridge         bridge    local
def456ghi789   host           host      local
ghi789jkl012   app-network    bridge    local  ← Notre réseau !
```

### Inspecter le réseau créé

```bash
docker network inspect app-network
```

**Sortie (simplifiée) :**

```json
[
    {
        "Name": "app-network",
        "Driver": "bridge",
        "Scope": "local",
        "IPAM": {
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Containers": {}
    }
]
```

Docker a automatiquement :
- Attribué un sous-réseau (`172.18.0.0/16`)
- Configuré une passerelle (`172.18.0.1`)
- Créé un réseau isolé et prêt à l'emploi

## Options de configuration avancées

Vous pouvez personnaliser votre réseau lors de sa création avec plusieurs options.

### Spécifier le driver

```bash
docker network create --driver bridge mon-reseau
```

Les drivers disponibles : `bridge`, `overlay`, `macvlan`, `host`, `none`

> 💡 **Note :** Par défaut, c'est `bridge` qui est utilisé.

### Définir un sous-réseau personnalisé

```bash
docker network create \
  --subnet=192.168.100.0/24 \
  mon-reseau
```

**Pourquoi ?** Contrôler la plage d'adresses IP pour éviter les conflits avec d'autres réseaux.

### Définir une passerelle

```bash
docker network create \
  --subnet=192.168.100.0/24 \
  --gateway=192.168.100.1 \
  mon-reseau
```

La passerelle est l'adresse IP du routeur virtuel pour ce réseau.

### Définir une plage d'adresses IP

```bash
docker network create \
  --subnet=192.168.100.0/24 \
  --ip-range=192.168.100.128/25 \
  mon-reseau
```

**Explication :**
- `--subnet` : Plage complète disponible (192.168.100.0 à 192.168.100.255)
- `--ip-range` : Portion que Docker peut utiliser (192.168.100.128 à 192.168.100.255)

### Ajouter des labels

Les labels sont des métadonnées qui aident à organiser vos ressources :

```bash
docker network create \
  --label projet=ecommerce \
  --label environnement=production \
  app-prod-network
```

Vous pouvez ensuite filtrer par labels :

```bash
docker network ls --filter label=environnement=production
```

### Configuration IPv6

Si vous avez besoin d'IPv6 :

```bash
docker network create \
  --ipv6 \
  --subnet=2001:db8:1::/64 \
  mon-reseau-ipv6
```

### Options combinées - Configuration complète

Voici un exemple avec plusieurs options :

```bash
docker network create \
  --driver bridge \
  --subnet=172.28.0.0/16 \
  --ip-range=172.28.5.0/24 \
  --gateway=172.28.5.254 \
  --label projet=monapp \
  --label environnement=dev \
  app-dev-network
```

Cette commande crée un réseau :
- De type bridge
- Avec un sous-réseau spécifique
- Docker peut attribuer des IPs dans une plage limitée
- Avec une passerelle personnalisée
- Étiqueté pour faciliter l'organisation

## Connecter des conteneurs à un réseau

### Au moment de la création du conteneur

La méthode la plus simple est de spécifier le réseau lors du lancement :

```bash
docker run -d \
  --name mon-app \
  --network app-network \
  nginx
```

### Connecter un conteneur existant

Vous pouvez connecter un conteneur déjà en cours d'exécution à un réseau :

```bash
# Lancer un conteneur (utilise le réseau par défaut)
docker run -d --name app nginx

# Le connecter à un réseau personnalisé
docker network connect app-network app
```

Maintenant, le conteneur `app` est connecté à **deux** réseaux : le bridge par défaut ET `app-network`.

### Spécifier une adresse IP fixe

Par défaut, Docker attribue automatiquement les adresses IP. Mais vous pouvez en forcer une :

```bash
docker run -d \
  --name mon-app \
  --network app-network \
  --ip 172.28.5.10 \
  nginx
```

⚠️ **Attention :** L'adresse IP doit être dans la plage du sous-réseau du réseau.

### Connecter un conteneur à plusieurs réseaux

Un conteneur peut appartenir à plusieurs réseaux simultanément :

```bash
# Créer deux réseaux
docker network create frontend-network
docker network create backend-network

# Lancer un conteneur sur le premier réseau
docker run -d --name api --network frontend-network mon-api

# Le connecter aussi au second réseau
docker network connect backend-network api
```

**Cas d'usage :** Un conteneur API qui doit communiquer à la fois avec le frontend et la base de données sur des réseaux séparés.

```
┌─────────────────────────────────────────────────┐
│                                                 │
│   Frontend Network          Backend Network     │
│   ┌─────────┐               ┌─────────┐         │
│   │   Web   │               │   DB    │         │
│   └────┬────┘               └────┬────┘         │
│        │                         │              │
│        │    ┌──────────────┐     │              │
│        └───►│     API      │◄────┘              │
│             │ (sur les 2)  │                    │
│             └──────────────┘                    │
│                                                 │
└─────────────────────────────────────────────────┘
```

## Déconnecter un conteneur d'un réseau

Pour retirer un conteneur d'un réseau :

```bash
docker network disconnect app-network mon-app
```

⚠️ **Important :** Un conteneur doit toujours être connecté à au moins un réseau. Vous ne pouvez pas déconnecter le dernier réseau.

## Inspecter et monitorer les réseaux

### Inspecter un réseau

La commande `inspect` donne des détails complets :

```bash
docker network inspect app-network
```

**Informations fournies :**
- Configuration réseau (subnet, gateway)
- Conteneurs connectés et leurs IPs
- Options et labels
- État du réseau

### Exemple de sortie utile

```bash
docker network inspect app-network --format '{{range .Containers}}{{.Name}} - {{.IPv4Address}}{{println}}{{end}}'
```

**Sortie :**
```
web-server - 172.28.0.2/16
api-backend - 172.28.0.3/16
postgres-db - 172.28.0.4/16
```

### Lister les réseaux avec filtres

#### Filtrer par driver

```bash
docker network ls --filter driver=bridge
```

#### Filtrer par label

```bash
docker network ls --filter label=environnement=production
```

#### Filtrer par nom

```bash
docker network ls --filter name=app
```

### Voir les réseaux d'un conteneur

```bash
docker inspect mon-conteneur --format '{{range $net, $conf := .NetworkSettings.Networks}}{{$net}} {{end}}'
```

Ou plus simplement :

```bash
docker inspect mon-conteneur | grep NetworkMode
```

## Gestion de plusieurs réseaux

### Organiser par projet

Une bonne pratique est de créer un réseau par projet :

```bash
# Projet e-commerce
docker network create ecommerce-network

# Projet blog
docker network create blog-network

# Projet API interne
docker network create api-internal-network
```

### Organiser par environnement

Créez des réseaux séparés pour chaque environnement :

```bash
docker network create app-dev-network
docker network create app-staging-network
docker network create app-prod-network
```

### Organiser par couche applicative

Séparez les différentes couches de votre architecture :

```bash
# Couche présentation
docker network create frontend-network

# Couche logique métier
docker network create backend-network

# Couche données
docker network create database-network
```

**Architecture typique :**

```
┌──────────────────────────────────────────────────────────┐
│  Frontend Network                                        │
│  ┌────────┐    ┌────────┐                                │
│  │ Nginx  │    │ React  │                                │
│  └───┬────┘    └───┬────┘                                │
│      │             │                                     │
└──────┼─────────────┼─────────────────────────────────────┘
       │             │
┌──────┼─────────────┼─────────────────────────────────────┐
│      │   Backend Network                                 │
│      │             │                                     │
│  ┌───▼─────┐  ┌────▼─────┐                               │
│  │   API   │  │  Worker  │                               │
│  └───┬─────┘  └───┬──────┘                               │
│      │            │                                      │
└──────┼────────────┼──────────────────────────────────────┘
       │            │
┌──────┼────────────┼──────────────────────────────────────┐
│      │   Database Network                                │
│  ┌───▼──────┐  ┌──▼────┐                                 │
│  │PostgreSQL│  │ Redis │                                 │
│  └──────────┘  └───────┘                                 │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### Exemple complet : Application multi-tiers

```bash
# 1. Créer les réseaux
docker network create frontend-net
docker network create backend-net
docker network create database-net

# 2. Lancer la base de données (uniquement sur database-net)
docker run -d \
  --name postgres-db \
  --network database-net \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

# 3. Lancer l'API (sur backend-net et database-net)
docker run -d \
  --name api \
  --network backend-net \
  -e DB_HOST=postgres-db \
  mon-api

docker network connect database-net api

# 4. Lancer le serveur web (sur frontend-net et backend-net)
docker run -d \
  --name nginx \
  --network frontend-net \
  -p 80:80 \
  nginx

docker network connect backend-net nginx
```

**Avantages de cette architecture :**
- La base de données n'est accessible que par l'API
- Le frontend ne peut pas accéder directement à la BDD
- Sécurité renforcée par isolation réseau
- Chaque composant est sur le réseau minimum nécessaire

## Supprimer des réseaux

### Supprimer un réseau spécifique

```bash
docker network rm nom-du-reseau
```

⚠️ **Conditions :** Vous ne pouvez supprimer un réseau que si :
- Aucun conteneur n'y est connecté
- Ce n'est pas un réseau système (bridge, host, none)

### Déconnecter avant de supprimer

Si des conteneurs sont encore connectés :

```bash
# Lister les conteneurs sur le réseau
docker network inspect mon-reseau --format '{{range .Containers}}{{.Name}} {{end}}'

# Déconnecter chaque conteneur
docker network disconnect mon-reseau conteneur1
docker network disconnect mon-reseau conteneur2

# Puis supprimer le réseau
docker network rm mon-reseau
```

### Supprimer plusieurs réseaux

```bash
docker network rm reseau1 reseau2 reseau3
```

### Nettoyer les réseaux inutilisés

Docker peut accumuler des réseaux inutilisés. Pour les supprimer tous :

```bash
docker network prune
```

**Avertissement :**
```
WARNING! This will remove all custom networks not used by at least one container.
Are you sure you want to continue? [y/N]
```

Tapez `y` pour confirmer.

### Nettoyage forcé (sans confirmation)

```bash
docker network prune -f
```

### Nettoyer avec filtre

Supprimer uniquement les réseaux avec un certain label :

```bash
docker network prune --filter "label=environnement=test"
```

## Cas d'usage pratiques

### Cas 1 : Environnement de développement isolé

Chaque développeur a son propre réseau pour éviter les conflits :

```bash
# Alice
docker network create alice-dev-network
docker run -d --network alice-dev-network --name alice-db postgres
docker run -d --network alice-dev-network --name alice-app mon-app

# Bob
docker network create bob-dev-network
docker run -d --network bob-dev-network --name bob-db postgres
docker run -d --network bob-dev-network --name bob-app mon-app
```

Les conteneurs d'Alice et Bob sont complètement isolés.

### Cas 2 : Tests automatisés isolés

Créez un réseau temporaire pour chaque série de tests :

```bash
# Avant les tests
TEST_NETWORK="test-run-$(date +%s)"
docker network create $TEST_NETWORK

# Lancer les conteneurs de test
docker run -d --network $TEST_NETWORK --name test-db postgres
docker run --network $TEST_NETWORK --name test-runner mon-test-suite

# Après les tests
docker network rm $TEST_NETWORK
```

### Cas 3 : Séparation staging/production

```bash
# Réseau de staging
docker network create staging-network

docker run -d --network staging-network --name staging-db postgres
docker run -d --network staging-network --name staging-api mon-api
docker run -d --network staging-network -p 8080:80 --name staging-web nginx

# Réseau de production
docker network create production-network

docker run -d --network production-network --name prod-db postgres
docker run -d --network production-network --name prod-api mon-api
docker run -d --network production-network -p 80:80 --name prod-web nginx
```

**Avantage :** Aucun risque qu'un conteneur de staging communique accidentellement avec un conteneur de production.

### Cas 4 : Architecture microservices

```bash
# Réseau public (accessible depuis l'extérieur)
docker network create public-network

# Réseau privé (interne uniquement)
docker network create private-network

# API Gateway (sur les deux réseaux)
docker run -d \
  --name api-gateway \
  --network public-network \
  -p 80:80 \
  mon-api-gateway

docker network connect private-network api-gateway

# Microservices (uniquement sur private-network)
docker run -d --network private-network --name user-service user-api
docker run -d --network private-network --name order-service order-api
docker run -d --network private-network --name payment-service payment-api
```

**Résultat :** Les microservices ne sont pas directement accessibles depuis l'extérieur, seulement via l'API Gateway.

### Cas 5 : Réseau avec configuration réseau spécifique

Pour un projet nécessitant une configuration réseau précise :

```bash
docker network create \
  --driver bridge \
  --subnet 10.0.0.0/24 \
  --gateway 10.0.0.1 \
  --ip-range 10.0.0.128/25 \
  --label projet=infra \
  --label criticite=haute \
  infrastructure-network

# Lancer des services avec IPs fixes
docker run -d --network infrastructure-network --ip 10.0.0.10 --name dns-server mon-dns
docker run -d --network infrastructure-network --ip 10.0.0.11 --name dhcp-server mon-dhcp
```

## Options avancées de gestion

### Définir des options de driver personnalisées

Certains drivers acceptent des options spécifiques :

```bash
docker network create \
  --opt "com.docker.network.bridge.name=my-bridge" \
  --opt "com.docker.network.bridge.enable_icc=true" \
  mon-reseau
```

**Options bridge courantes :**
- `com.docker.network.bridge.name` : Nom de l'interface bridge
- `com.docker.network.bridge.enable_icc` : Activer la communication inter-conteneurs (true/false)
- `com.docker.network.bridge.enable_ip_masquerade` : Activer le NAT (true/false)
- `com.docker.network.bridge.host_binding_ipv4` : IP de liaison sur l'hôte

### Désactiver la communication inter-conteneurs

Par défaut, tous les conteneurs sur un réseau peuvent communiquer. Pour désactiver :

```bash
docker network create \
  --opt "com.docker.network.bridge.enable_icc=false" \
  reseau-isole
```

Maintenant, les conteneurs sur ce réseau ne peuvent communiquer qu'avec l'extérieur, pas entre eux.

### Configurer le MTU (Maximum Transmission Unit)

Pour des environnements spécifiques (VPN, overlay) :

```bash
docker network create \
  --opt "com.docker.network.driver.mtu=1450" \
  mon-reseau
```

Le MTU par défaut est 1500, mais peut nécessiter un ajustement selon votre infrastructure.

## Troubleshooting et debugging

### Problème : Impossible de créer un réseau

**Erreur :**
```
Error response from daemon: could not find an available, non-overlapping IPv4 address pool
```

**Solution :** Docker manque d'espace d'adressage. Spécifiez un sous-réseau :

```bash
docker network create --subnet=172.25.0.0/16 mon-reseau
```

### Problème : Les conteneurs ne se trouvent pas par leur nom

**Vérifications :**

1. Sont-ils sur le même réseau ?
```bash
docker network inspect mon-reseau
```

2. Utilisez-vous un réseau personnalisé (pas le bridge par défaut) ?
```bash
# Créer un réseau personnalisé si nécessaire
docker network create mon-reseau-custom
docker network connect mon-reseau-custom conteneur1
docker network connect mon-reseau-custom conteneur2
```

3. Test de résolution DNS :
```bash
docker exec conteneur1 nslookup conteneur2
```

### Problème : Conflit de sous-réseaux

**Erreur :**
```
Error response from daemon: Pool overlaps with other one on this address space
```

**Solution :** Choisissez un sous-réseau différent :

```bash
# Lister les sous-réseaux utilisés
docker network inspect $(docker network ls -q) | grep Subnet

# Créer un réseau avec un sous-réseau non utilisé
docker network create --subnet=172.30.0.0/16 mon-nouveau-reseau
```

### Problème : Impossible de supprimer un réseau

**Erreur :**
```
Error response from daemon: network mon-reseau has active endpoints
```

**Solution :** Des conteneurs sont encore connectés. Les identifier et déconnecter :

```bash
# Voir les conteneurs connectés
docker network inspect mon-reseau --format '{{range .Containers}}{{.Name}} {{end}}'

# Déconnecter ou arrêter les conteneurs
docker stop $(docker network inspect mon-reseau --format '{{range .Containers}}{{.Name}} {{end}}')

# Puis supprimer le réseau
docker network rm mon-reseau
```

### Déboguer la connectivité réseau

#### Tester la connectivité entre conteneurs

```bash
# Lancer un conteneur de test avec des outils réseau
docker run -it --network mon-reseau --name debug-container nicolaka/netshoot

# À l'intérieur du conteneur
ping autre-conteneur
nslookup autre-conteneur
traceroute autre-conteneur
curl http://autre-conteneur:port
```

#### Analyser le trafic réseau

```bash
# Capturer le trafic réseau sur un conteneur
docker exec -it mon-conteneur tcpdump -i eth0

# Ou avec netshoot
docker run --network container:mon-conteneur nicolaka/netshoot tcpdump
```

## Bonnes pratiques

### 1. Nommez vos réseaux de manière descriptive

❌ **Mauvais :**
```bash
docker network create net1
docker network create net2
docker network create test
```

✅ **Bon :**
```bash
docker network create ecommerce-frontend-network
docker network create ecommerce-backend-network
docker network create ecommerce-database-network
```

### 2. Utilisez des labels pour l'organisation

```bash
docker network create \
  --label projet=ecommerce \
  --label environnement=production \
  --label equipe=backend \
  --label contact=admin@example.com \
  ecommerce-prod-network
```

### 3. Documentez votre architecture réseau

Créez un document qui décrit :
- Quels réseaux existent
- Leur but
- Quels conteneurs/services sont sur quels réseaux
- Les règles de communication

**Exemple de documentation :**

```markdown
## Architecture Réseau - Projet E-commerce

### Réseaux

1. **ecommerce-frontend-network** (172.20.0.0/16)
   - Web server (Nginx)
   - React App

2. **ecommerce-backend-network** (172.21.0.0/16)
   - API Gateway
   - User Service
   - Order Service
   - Payment Service

3. **ecommerce-database-network** (172.22.0.0/16)
   - PostgreSQL
   - Redis Cache

### Règles de communication
- Frontend → Backend : Oui (via API Gateway)
- Backend → Database : Oui
- Frontend → Database : Non (isolation)
```

### 4. Un réseau par environnement

Séparez clairement dev, staging et production :

```bash
docker network create app-dev-network
docker network create app-staging-network
docker network create app-production-network
```

### 5. Nettoyez régulièrement

Mettez en place un processus de nettoyage périodique :

```bash
# Créer un script de nettoyage
cat > cleanup-networks.sh << 'EOF'
#!/bin/bash
echo "Nettoyage des réseaux Docker inutilisés..."
docker network prune -f
echo "Réseaux restants:"
docker network ls
EOF

chmod +x cleanup-networks.sh
```

### 6. Limitez l'exposition des services

N'exposez (avec `-p`) que les services qui doivent être accessibles depuis l'extérieur :

```bash
# ✅ BON - Seul le frontend est exposé
docker run -d --network frontend-net -p 80:80 web-frontend
docker run -d --network backend-net api-backend  # Pas de -p
docker run -d --network db-net postgres-db       # Pas de -p

# ❌ MAUVAIS - Tout est exposé
docker run -d -p 80:80 web-frontend
docker run -d -p 3000:3000 api-backend
docker run -d -p 5432:5432 postgres-db
```

### 7. Utilisez des sous-réseaux cohérents

Définissez un schéma d'adressage cohérent :

```bash
# Développement : 172.20.x.0/16
docker network create --subnet=172.20.1.0/24 app-dev-frontend
docker network create --subnet=172.20.2.0/24 app-dev-backend
docker network create --subnet=172.20.3.0/24 app-dev-database

# Production : 172.21.x.0/16
docker network create --subnet=172.21.1.0/24 app-prod-frontend
docker network create --subnet=172.21.2.0/24 app-prod-backend
docker network create --subnet=172.21.3.0/24 app-prod-database
```

### 8. Testez avant de déployer en production

Testez votre configuration réseau dans un environnement de staging :

```bash
# Créer un environnement de test identique
docker network create --subnet=172.28.0.0/16 test-network

# Lancer vos conteneurs
docker run -d --network test-network --name test-app mon-app
docker run -d --network test-network --name test-db postgres

# Tester la connectivité
docker exec test-app ping test-db
```

## Automatisation avec scripts

### Script de création de réseaux

```bash
#!/bin/bash
# create-networks.sh

PROJECT="monapp"
ENVIRONMENTS=("dev" "staging" "prod")
LAYERS=("frontend" "backend" "database")

for ENV in "${ENVIRONMENTS[@]}"; do
    for LAYER in "${LAYERS[@]}"; do
        NETWORK_NAME="${PROJECT}-${ENV}-${LAYER}-network"

        echo "Création du réseau: $NETWORK_NAME"

        docker network create \
            --label projet=$PROJECT \
            --label environnement=$ENV \
            --label couche=$LAYER \
            $NETWORK_NAME
    done
done

echo "Réseaux créés avec succès!"
docker network ls --filter label=projet=$PROJECT
```

### Script de nettoyage intelligent

```bash
#!/bin/bash
# cleanup-networks.sh

echo "Réseaux inutilisés:"
docker network ls --filter "dangling=true"

echo ""
read -p "Voulez-vous supprimer ces réseaux? (y/n) " -n 1 -r
echo

if [[ $REPLY =~ ^[Yy]$ ]]; then
    docker network prune -f
    echo "Nettoyage terminé!"
else
    echo "Annulé."
fi
```

## Intégration avec Docker Compose

Dans Docker Compose, vous pouvez facilement gérer des réseaux personnalisés :

```yaml
version: '3.8'

services:
  web:
    image: nginx
    networks:
      - frontend
    ports:
      - "80:80"

  api:
    image: mon-api
    networks:
      - frontend
      - backend

  database:
    image: postgres
    networks:
      - backend
    environment:
      POSTGRES_PASSWORD: secret

networks:
  frontend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.25.0.0/16

  backend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.26.0.0/16
```

Cette configuration crée automatiquement les réseaux et connecte les services correctement.

## À retenir

🔑 **Points clés :**

### Création
- Utilisez `docker network create` pour créer des réseaux personnalisés
- Les réseaux personnalisés offrent la **résolution DNS automatique**
- Personnalisez avec `--subnet`, `--gateway`, `--ip-range`, etc.

### Gestion
- Connectez les conteneurs avec `--network` ou `docker network connect`
- Un conteneur peut être sur **plusieurs réseaux** simultanément
- Inspectez avec `docker network inspect` pour déboguer

### Organisation
- **Un réseau par projet** ou par environnement
- Séparez les couches applicatives (frontend, backend, database)
- Utilisez des **labels** pour l'organisation

### Sécurité
- N'exposez que ce qui est nécessaire
- Isolez les services sensibles (bases de données)
- Limitez la communication inter-conteneurs si besoin

### Maintenance
- Nettoyez régulièrement avec `docker network prune`
- Documentez votre architecture réseau
- Automatisez avec des scripts

## Prochaine étape

Maintenant que vous savez créer et gérer des réseaux personnalisés, la section suivante (7.4) se concentrera sur la **communication entre conteneurs** : comment optimiser les échanges, gérer les ports, implémenter des patterns de communication, et résoudre les problèmes de connectivité avancés.

⏭️ [Communication entre conteneurs](/07-reseaux-docker/04-communication-entre-conteneurs.md)
