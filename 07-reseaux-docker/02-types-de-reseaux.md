🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 7.2 Types de réseaux (bridge, host, none, overlay)

## Introduction

Dans la section précédente, vous avez découvert les principes fondamentaux des réseaux Docker. Maintenant, plongeons dans les **quatre types de réseaux** que Docker propose. Chacun a ses propres caractéristiques, avantages et cas d'usage spécifiques.

Les quatre types de réseaux Docker sont :

1. **Bridge** - Le réseau par défaut (le plus utilisé)
2. **Host** - Partage le réseau de l'hôte
3. **None** - Isolation réseau complète
4. **Overlay** - Pour les clusters multi-machines (avancé)

Comprendre ces différents types vous permettra de choisir la meilleure configuration pour vos besoins.

## Vue d'ensemble comparative

Avant d'entrer dans les détails, voici un aperçu rapide :

| Type | Usage principal | Isolation | Complexité | Cas d'usage |
|------|----------------|-----------|------------|-------------|
| **Bridge** | Applications standard | Moyenne | ⭐ Facile | Développement, production |
| **Host** | Performance maximale | Aucune | ⭐⭐ Moyenne | Services réseau intensifs |
| **None** | Isolation totale | Maximale | ⭐ Facile | Traitement isolé, batch |
| **Overlay** | Clusters Docker Swarm | Élevée | ⭐⭐⭐ Avancée | Production distribuée |

## 1. Le réseau Bridge

### Qu'est-ce que le réseau Bridge ?

Le réseau **bridge** est le type de réseau **par défaut** dans Docker. C'est le plus courant et celui que vous utiliserez dans la majorité des cas.

Un réseau bridge crée un **pont virtuel** entre vos conteneurs et votre machine hôte. C'est comme un switch réseau virtuel auquel tous vos conteneurs se connectent.

### Analogie : L'immeuble avec un réseau local

Imaginez un immeuble (votre machine) avec un réseau WiFi interne (le bridge). Chaque appartement (conteneur) se connecte à ce WiFi pour communiquer avec les autres appartements et accéder à Internet via la connexion principale de l'immeuble.

```
┌─────────────────────────────────────────────────────────┐
│                  MACHINE HÔTE                           │
│                                                         │
│               Interface eth0 (Internet)                 │
│                        │                                │
│                        ↓                                │
│   ┌───────────────────────────────────────────────┐     │
│   │       Réseau Bridge (docker0)                 │     │
│   │                                               │     │
│   │   ┌──────────┐   ┌──────────┐   ┌──────────┐  │     │
│   │   │Container │   │Container │   │Container │  │     │
│   │   │    A     │   │    B     │   │    C     │  │     │
│   │   │172.17.0.2│   │172.17.0.3│   │172.17.0.4│  │     │
│   │   └──────────┘   └──────────┘   └──────────┘  │     │
│   │                                               │     │
│   └───────────────────────────────────────────────┘     │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Le bridge par défaut vs les bridges personnalisés

Docker crée automatiquement un réseau bridge nommé `bridge` (c'est le bridge par défaut). Cependant, vous pouvez aussi créer vos propres réseaux bridge personnalisés.

#### Bridge par défaut

```bash
# Lancer un conteneur sans spécifier de réseau
docker run -d --name conteneur1 nginx
# Utilise automatiquement le bridge par défaut
```

**Limitations :**
- ❌ Pas de résolution DNS automatique entre conteneurs
- ❌ Les conteneurs doivent utiliser des adresses IP pour communiquer
- ❌ Configuration limitée

#### Bridge personnalisé (recommandé)

```bash
# Créer un bridge personnalisé
docker network create mon-bridge

# Lancer des conteneurs sur ce bridge
docker run -d --name web --network mon-bridge nginx
docker run -d --name api --network mon-bridge node:18
```

**Avantages :**
- ✅ Résolution DNS automatique (les conteneurs se trouvent par leur nom)
- ✅ Meilleure isolation entre groupes de conteneurs
- ✅ Configuration personnalisable
- ✅ Possibilité de connecter/déconnecter des conteneurs à chaud

### Exemple pratique : Application web avec base de données

```bash
# 1. Créer un réseau bridge personnalisé
docker network create app-network

# 2. Lancer PostgreSQL
docker run -d \
  --name postgres-db \
  --network app-network \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

# 3. Lancer l'application
docker run -d \
  --name web-app \
  --network app-network \
  -p 8080:80 \
  -e DATABASE_URL=postgres://postgres:secret@postgres-db:5432/mydb \
  mon-app
```

**Points clés :**
- Les deux conteneurs sont sur `app-network`
- `web-app` peut se connecter à `postgres-db` par son nom
- Seul `web-app` expose un port (8080) vers l'extérieur

### Configuration avancée du bridge

Vous pouvez personnaliser un réseau bridge lors de sa création :

```bash
docker network create \
  --driver bridge \
  --subnet=172.28.0.0/16 \
  --ip-range=172.28.5.0/24 \
  --gateway=172.28.5.254 \
  mon-bridge-custom
```

**Options expliquées :**
- `--subnet` : La plage complète d'adresses IP disponibles
- `--ip-range` : La portion de la plage que Docker peut utiliser
- `--gateway` : L'adresse de la passerelle

### Inspecter un réseau bridge

```bash
docker network inspect mon-bridge
```

Cela affiche :
- Les conteneurs connectés
- Leurs adresses IP
- La configuration du réseau
- Les options définies

### Quand utiliser le réseau Bridge ?

✅ **Utilisez Bridge quand :**
- Vous développez des applications multi-conteneurs sur une seule machine
- Vos conteneurs doivent communiquer entre eux
- Vous avez besoin d'isolation entre différents projets
- C'est un environnement de développement ou de production sur un seul hôte

✅ **Cas d'usage typiques :**
- Stack web classique (frontend + backend + base de données)
- Microservices sur une seule machine
- Environnements de développement local
- Serveurs de production simples (non-distribués)

## 2. Le réseau Host

### Qu'est-ce que le réseau Host ?

Le réseau **host** supprime l'isolation réseau entre le conteneur et la machine hôte. Le conteneur utilise **directement** le réseau de la machine hôte, comme si l'application tournait nativement sur la machine.

### Analogie : Vivre directement dans la maison principale

Au lieu d'avoir un appartement séparé (conteneur avec son propre réseau), vous vivez directement dans la maison principale (machine hôte) et partagez toutes ses ressources réseau.

```
┌─────────────────────────────────────────────────────────┐
│              MACHINE HÔTE                               │
│                                                         │
│   Interface réseau : eth0                               │
│                  │                                      │
│                  │  (Partagé directement)               │
│                  ↓                                      │
│          ┌────────────────┐                             │
│          │   Conteneur    │                             │
│          │  (mode host)   │                             │
│          │                │                             │
│          │  Pas d'isolation réseau !                    │
│          │  Utilise directement eth0                    │
│          └────────────────┘                             │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Comment utiliser le réseau Host

```bash
docker run -d --network host --name mon-conteneur nginx
```

**Important :** Avec le mode host, vous **ne devez pas** utiliser `-p` pour mapper les ports. Le conteneur utilise directement les ports de l'hôte.

### Exemple pratique

```bash
# Lancer Nginx en mode host
docker run -d --network host nginx

# Nginx écoute directement sur le port 80 de la machine hôte
# Accessible via http://localhost:80 (ou l'IP de la machine)
```

### Caractéristiques du mode Host

#### Avantages
✅ **Performances réseau maximales** - Pas de traduction d'adresse (NAT)
✅ **Configuration simple** pour certains services
✅ **Accès direct aux interfaces réseau de l'hôte**
✅ **Idéal pour les outils de monitoring réseau**

#### Inconvénients
❌ **Aucune isolation réseau** - Moins sécurisé
❌ **Conflits de ports** possibles avec des services de l'hôte
❌ **Moins portable** - Dépend de la configuration réseau de l'hôte
❌ **Fonctionne uniquement sur Linux** (limitation Docker Desktop sur Windows/macOS)

### Cas d'usage du réseau Host

⚠️ **Note importante :** Le mode host est **principalement utilisé sur Linux**. Sur Windows et macOS avec Docker Desktop, il ne fonctionne pas comme attendu en raison de l'architecture basée sur une VM.

#### Quand utiliser Host ?

✅ **Utilisez Host quand :**
- Vous avez besoin de performances réseau maximales
- Vous développez des outils de monitoring ou d'analyse réseau
- Vous devez accéder à des interfaces réseau spécifiques de l'hôte
- Vous déployez sur Linux et la sécurité moindre est acceptable

✅ **Cas d'usage typiques :**
- Serveurs de jeux à haute performance
- Proxies inverses comme HAProxy ou Nginx (en edge computing)
- Outils de monitoring réseau (Prometheus, Grafana en certaines configs)
- Applications nécessitant un accès direct au réseau physique

❌ **N'utilisez PAS Host quand :**
- Vous avez besoin d'isolation et de sécurité
- Vous voulez faire tourner plusieurs conteneurs utilisant les mêmes ports
- Vous développez sur Windows ou macOS
- Vous prévoyez de déployer sur plusieurs machines

### Exemple comparatif : Bridge vs Host

#### Avec Bridge (standard)

```bash
# Port mapping nécessaire
docker run -d -p 8080:80 --name nginx-bridge nginx
# Accessible sur localhost:8080
```

#### Avec Host

```bash
# Pas de port mapping
docker run -d --network host --name nginx-host nginx
# Accessible directement sur localhost:80
```

## 3. Le réseau None

### Qu'est-ce que le réseau None ?

Le réseau **none** désactive complètement le réseau pour un conteneur. Le conteneur n'a **aucune connectivité réseau** - ni avec d'autres conteneurs, ni avec l'hôte, ni avec Internet.

### Analogie : Une chambre forte isolée

C'est comme mettre quelque chose dans une chambre forte complètement isolée, sans téléphone, sans Internet, sans aucun moyen de communication avec l'extérieur.

```
┌─────────────────────────────────────────┐
│         MACHINE HÔTE                    │
│                                         │
│                                         │
│        ┌────────────────────────────┐   │
│        │   Conteneur                │   │
│        │  (mode none)               │   │
│        │                            │   │
│        │  ❌ Pas de réseau          │   │
│        │  ❌ Isolation totale       │   │
│        │  ❌ Aucune communication   │   │
│        └────────────────────────────┘   │
│                                         │
└─────────────────────────────────────────┘
```

### Comment utiliser le réseau None

```bash
docker run -d --network none --name conteneur-isole alpine sleep 3600
```

### Vérifier l'isolation

```bash
# Entrer dans le conteneur
docker exec -it conteneur-isole sh

# Essayer de pinger quelque chose
ping google.com
# Erreur : Network is unreachable

# Vérifier les interfaces réseau
ip addr
# Seulement l'interface loopback (lo), pas d'interface réseau
```

### Caractéristiques du mode None

#### Avantages
✅ **Sécurité maximale** - Aucune fuite réseau possible
✅ **Isolation totale** - Parfait pour le traitement de données sensibles
✅ **Pas de risque d'attaque réseau**
✅ **Performance** - Pas de surcharge réseau

#### Inconvénients
❌ **Aucune communication** - Très limitatif
❌ **Pas d'accès Internet** - Impossible de télécharger quoi que ce soit
❌ **Échange de données complexe** - Doit se faire via volumes ou stdin/stdout

### Quand utiliser le réseau None ?

✅ **Utilisez None quand :**
- Vous traitez des données extrêmement sensibles
- Vous exécutez des calculs isolés (batch processing)
- Vous voulez garantir qu'aucune donnée ne peut fuir par le réseau
- Vous testez la sécurité d'une application

✅ **Cas d'usage typiques :**
- Traitement de données confidentielles (données médicales, financières)
- Conversion de fichiers sans risque de fuite
- Exécution de code non fiable de manière sécurisée
- Tests de sécurité et sandboxing
- Batch jobs qui ne nécessitent aucune communication réseau

### Exemple pratique : Conversion de fichiers

Imaginons que vous devez convertir des PDF confidentiels en images :

```bash
# Créer un volume pour échanger des fichiers
docker volume create fichiers-confidentiels

# Copier le PDF dans le volume
docker run --rm \
  -v fichiers-confidentiels:/data \
  -v $(pwd):/source \
  alpine cp /source/document.pdf /data/

# Convertir le PDF en images (sans accès réseau)
docker run --rm \
  --network none \
  -v fichiers-confidentiels:/data \
  image-converter \
  convert /data/document.pdf /data/output.png

# Récupérer les images converties
docker run --rm \
  -v fichiers-confidentiels:/data \
  -v $(pwd):/destination \
  alpine cp /data/output.png /destination/
```

**Avantage :** Même si le conteneur `image-converter` contenait du code malveillant, il ne pourrait pas envoyer vos données sur Internet.

## 4. Le réseau Overlay

### Qu'est-ce que le réseau Overlay ?

Le réseau **overlay** est un réseau qui s'étend sur **plusieurs machines Docker**. Il permet à des conteneurs tournant sur différentes machines physiques de communiquer comme s'ils étaient sur le même réseau local.

### Analogie : Un VPN d'entreprise

C'est comme un VPN qui connecte plusieurs bureaux d'une entreprise. Les employés dans différentes villes peuvent communiquer comme s'ils étaient dans le même bureau.

```
┌────────────────────────────────────────────────────────────┐
│                  RÉSEAU OVERLAY                            │
│                                                            │
│   ┌──────────────────────┐       ┌──────────────────────┐  │
│   │   Machine 1 (Paris)  │       │  Machine 2 (Londres) │  │
│   │                      │       │                      │  │
│   │  ┌────────────────┐  │       │  ┌────────────────┐  │  │
│   │  │  Conteneur A   │◄─┼───────┼─►│  Conteneur B   │  │  │
│   │  │  (web)         │  │       │  │  (db)          │  │  │
│   │  └────────────────┘  │       │  └────────────────┘  │  │
│   │                      │       │                      │  │
│   └──────────────────────┘       └──────────────────────┘  │
│                                                            │
│   Communication comme si sur la même machine !             │
└────────────────────────────────────────────────────────────┘
```

### Prérequis : Docker Swarm ou Kubernetes

Les réseaux overlay nécessitent un système d'orchestration comme :
- **Docker Swarm** (intégré à Docker)
- **Kubernetes** (système d'orchestration externe)

> 💡 **Note pour débutants :** Les réseaux overlay sont un sujet **avancé**. Si vous débutez, vous pouvez ignorer cette partie pour le moment et y revenir plus tard quand vous apprendrez l'orchestration de conteneurs.

### Docker Swarm en bref

Docker Swarm transforme plusieurs machines Docker en un **cluster** (groupe de machines qui travaillent ensemble).

```bash
# Sur la machine manager (master)
docker swarm init

# Cette commande retourne un token pour rejoindre le swarm
# Sur les autres machines (workers)
docker swarm join --token SWMTKN-1-xxxxx manager-ip:2377
```

### Créer un réseau Overlay

```bash
# Créer un réseau overlay (nécessite un swarm actif)
docker network create \
  --driver overlay \
  --attachable \
  mon-reseau-overlay
```

L'option `--attachable` permet aux conteneurs standalone (non-services) de se connecter au réseau.

### Exemple pratique : Application distribuée

Imaginons une application répartie sur 3 serveurs :

```bash
# Sur le nœud manager
docker network create --driver overlay --attachable app-overlay

# Déployer une base de données sur le serveur 1
docker service create \
  --name postgres-db \
  --network app-overlay \
  postgres:15

# Déployer l'API sur le serveur 2
docker service create \
  --name api \
  --network app-overlay \
  -e DB_HOST=postgres-db \
  mon-api

# Déployer le frontend sur le serveur 3
docker service create \
  --name web \
  --network app-overlay \
  --publish 80:80 \
  mon-frontend
```

Magiquement, `api` peut se connecter à `postgres-db` même s'ils sont sur des machines différentes !

### Caractéristiques du réseau Overlay

#### Avantages
✅ **Communication multi-machines** - Cluster de conteneurs
✅ **Scalabilité horizontale** - Ajoutez des machines facilement
✅ **Haute disponibilité** - Redondance et failover
✅ **Résolution DNS** entre tous les nœuds
✅ **Chiffrement** optionnel du trafic entre machines

#### Inconvénients
❌ **Complexité élevée** - Configuration et gestion
❌ **Nécessite Docker Swarm ou Kubernetes**
❌ **Performances** légèrement inférieures (encapsulation réseau)
❌ **Debugging plus difficile**

### Configuration avancée : Chiffrement

Vous pouvez chiffrer le trafic réseau entre machines :

```bash
docker network create \
  --driver overlay \
  --opt encrypted \
  mon-reseau-securise
```

Cela chiffre toutes les communications entre conteneurs sur ce réseau.

### Quand utiliser le réseau Overlay ?

✅ **Utilisez Overlay quand :**
- Vous déployez une application sur plusieurs serveurs
- Vous avez besoin de scalabilité horizontale
- Vous utilisez Docker Swarm ou Kubernetes
- Vous construisez une architecture distribuée

✅ **Cas d'usage typiques :**
- Applications en production à grande échelle
- Microservices distribués sur plusieurs machines
- Systèmes nécessitant haute disponibilité
- Clusters de bases de données (réplication)

❌ **N'utilisez PAS Overlay quand :**
- Vous travaillez sur une seule machine (utilisez bridge)
- Vous débutez avec Docker (apprenez d'abord bridge)
- Vous n'avez pas besoin de distribuer vos conteneurs

### Limitations importantes

⚠️ **Points à retenir :**
- Overlay nécessite Docker Swarm mode activé
- Configuration réseau entre machines (ports ouverts : 2377, 7946, 4789)
- Légère surcharge de performance due à l'encapsulation
- Nécessite une compréhension approfondie des réseaux

## Tableau comparatif complet

| Critère | Bridge | Host | None | Overlay |
|---------|--------|------|------|---------|
| **Isolation** | Moyenne | Aucune | Maximale | Moyenne |
| **Performance** | Bonne | Excellente | Excellente | Moyenne |
| **DNS automatique** | Oui (custom) | N/A | Non | Oui |
| **Multi-machine** | Non | Non | Non | Oui |
| **Port mapping** | Oui | Non (inutile) | Non | Oui |
| **Sécurité** | Bonne | Faible | Maximale | Bonne |
| **Complexité** | Faible | Faible | Faible | Élevée |
| **Production simple** | ✅ | ⚠️ | ❌ | ❌ |
| **Production distribuée** | ❌ | ❌ | ❌ | ✅ |
| **Développement** | ✅ | ⚠️ | ❌ | ❌ |
| **Windows/macOS** | ✅ | ❌ | ✅ | ✅ |

## Choisir le bon type de réseau

### Arbre de décision

```
Quel type de réseau choisir ?
│
├─ Avez-vous besoin de communication réseau ?
│  │
│  ├─ NON (isolation totale)
│  │  └─ 🎯 NONE
│  │
│  └─ OUI → Continuez...
│
├─ Déployez-vous sur une seule machine ou plusieurs ?
│  │
│  ├─ PLUSIEURS MACHINES
│  │  └─ 🎯 OVERLAY (avec Docker Swarm/Kubernetes)
│  │
│  └─ UNE SEULE MACHINE → Continuez...
│
├─ Avez-vous besoin de performance réseau maximale ?
│  │
│  ├─ OUI (et sur Linux)
│  │  └─ 🎯 HOST
│  │
│  └─ NON → Continuez...
│
└─ Configuration standard multi-conteneurs ?
   │
   └─ OUI
      └─ 🎯 BRIDGE (personnalisé de préférence)
```

### Guide rapide par scénario

#### Scénario 1 : Développement local (stack web)
**Recommandation :** Bridge personnalisé

```bash
docker network create dev-network
docker run -d --network dev-network --name db postgres
docker run -d --network dev-network --name api mon-api
docker run -d --network dev-network -p 80:80 --name web mon-web
```

#### Scénario 2 : Proxy inverse haute performance (Linux)
**Recommandation :** Host

```bash
docker run -d --network host nginx
```

#### Scénario 3 : Traitement de données sensibles
**Recommandation :** None

```bash
docker run --network none -v data:/data traitement-batch
```

#### Scénario 4 : Cluster de production multi-serveurs
**Recommandation :** Overlay

```bash
docker network create --driver overlay prod-network
docker service create --network prod-network mon-service
```

## Commandes utiles par type de réseau

### Bridge

```bash
# Créer un bridge personnalisé
docker network create mon-bridge

# Avec options personnalisées
docker network create \
  --driver bridge \
  --subnet 172.20.0.0/16 \
  mon-bridge

# Lancer un conteneur sur ce bridge
docker run -d --network mon-bridge --name app mon-image
```

### Host

```bash
# Lancer en mode host (pas de -p nécessaire)
docker run -d --network host --name mon-app nginx
```

### None

```bash
# Lancer sans réseau
docker run -d --network none --name isole alpine sleep 3600
```

### Overlay

```bash
# Initialiser swarm (si pas déjà fait)
docker swarm init

# Créer un réseau overlay
docker network create --driver overlay --attachable mon-overlay

# Déployer un service sur ce réseau
docker service create --network mon-overlay --name web nginx
```

## Inspecter les réseaux

Quelle que soit le type, vous pouvez inspecter les réseaux :

```bash
# Lister tous les réseaux
docker network ls

# Inspecter un réseau spécifique
docker network inspect nom-reseau

# Voir les réseaux utilisés par un conteneur
docker inspect conteneur | grep NetworkMode
```

## Bonnes pratiques par type

### Bridge
✅ Créez des bridges personnalisés (pas le bridge par défaut)
✅ Un bridge par projet ou application
✅ Nommez clairement vos réseaux (`app-prod`, `app-dev`, etc.)
✅ Documentez quels conteneurs sont sur quels réseaux

### Host
⚠️ Utilisez uniquement quand nécessaire (performance critique)
⚠️ Vérifiez les conflits de ports avant déploiement
⚠️ Considérez les implications de sécurité
⚠️ Testez sur l'OS cible (Linux uniquement)

### None
✅ Parfait pour les traitements batch sensibles
✅ Utilisez des volumes pour l'échange de données
✅ Vérifiez que l'application n'a vraiment pas besoin de réseau
✅ Documentez pourquoi le réseau est désactivé

### Overlay
✅ Planifiez votre architecture de cluster en amont
✅ Configurez correctement les pare-feu
✅ Surveillez les performances réseau
✅ Utilisez le chiffrement pour les données sensibles
✅ Commencez petit, scalez progressivement

## Limitations et considérations

### Limitations de performance

**Bridge :**
- Légère surcharge due au NAT
- Généralement acceptable pour la plupart des applications

**Host :**
- Performances natives excellentes
- Mais pas d'isolation = risques de sécurité

**Overlay :**
- Encapsulation VXLAN = surcharge ~10-15%
- Acceptable pour la plupart des cas, mais à considérer pour applications très intensives

### Compatibilité OS

| Type | Linux | Windows | macOS |
|------|-------|---------|-------|
| Bridge | ✅ | ✅ | ✅ |
| Host | ✅ | ❌ | ❌ |
| None | ✅ | ✅ | ✅ |
| Overlay | ✅ | ✅ | ✅ |

> **Note :** Sur Windows et macOS, Docker fonctionne dans une VM Linux, donc host ne fonctionne pas comme attendu.

## Dépannage par type de réseau

### Bridge : Les conteneurs ne se voient pas

```bash
# Vérifier qu'ils sont sur le même réseau
docker network inspect mon-bridge

# Si sur le bridge par défaut, créer un bridge personnalisé
docker network create mon-bridge
docker network connect mon-bridge conteneur1
docker network connect mon-bridge conteneur2
```

### Host : Conflit de port

```bash
# Vérifier les ports utilisés sur l'hôte
netstat -tulpn | grep :80

# Arrêter le service conflictuel ou changer de port
```

### None : Besoin d'installer des packages

```bash
# Impossible d'installer des packages sans réseau
# Solution : Créer une image avec tout pré-installé
# Ou monter les binaires via volumes
```

### Overlay : Conteneurs sur différentes machines ne communiquent pas

```bash
# Vérifier que le swarm est actif
docker node ls

# Vérifier que les ports sont ouverts
# 2377 (management), 7946 (communication), 4789 (overlay network)

# Vérifier les pare-feu
sudo ufw status
```

## À retenir

🔑 **Points clés :**

### Bridge
- Type de réseau **par défaut** et le plus utilisé
- Créez des **bridges personnalisés** pour le DNS automatique
- Idéal pour applications sur **une seule machine**

### Host
- **Performance maximale** mais **aucune isolation**
- Fonctionne **uniquement sur Linux**
- À utiliser avec **précaution** (implications de sécurité)

### None
- **Isolation réseau complète**
- Parfait pour traitement de **données sensibles**
- Échange de données via **volumes uniquement**

### Overlay
- Pour **clusters multi-machines** (Docker Swarm/Kubernetes)
- Nécessite configuration **avancée**
- À utiliser en **production distribuée**

## Prochaine étape

Maintenant que vous connaissez les différents types de réseaux, la section suivante (7.3) vous apprendra à **créer et gérer des réseaux personnalisés** de manière avancée, avec des configurations spécifiques et des cas d'usage pratiques. Vous découvrirez comment optimiser vos réseaux pour vos besoins spécifiques.

⏭️ [Créer et gérer des réseaux personnalisés](/07-reseaux-docker/03-creer-et-gerer-des-reseaux-personnalises.md)
