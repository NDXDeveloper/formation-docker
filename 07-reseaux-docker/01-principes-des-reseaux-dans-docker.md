🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 7.1 Principes des réseaux dans Docker

## Introduction

Vous savez maintenant créer des conteneurs, gérer des images et persister des données. Mais il manque encore une pièce essentielle du puzzle : **comment les conteneurs communiquent-ils entre eux et avec le monde extérieur ?**

Imaginez ces situations courantes :
- Une application web qui doit se connecter à une base de données
- Un serveur Nginx qui transmet les requêtes à une application Node.js
- Plusieurs microservices qui doivent échanger des informations
- Accéder à votre application conteneurisée depuis votre navigateur

Toutes ces situations nécessitent une compréhension des **réseaux Docker**.

## Le problème de l'isolation

### Les conteneurs sont isolés par défaut

L'une des forces de Docker est l'**isolation** : chaque conteneur fonctionne dans son propre environnement, séparé des autres conteneurs et du système hôte. C'est excellent pour la sécurité et la stabilité, mais cela crée un défi :

> **Comment faire communiquer des conteneurs isolés ?**

Par défaut, un conteneur est comme une maison avec toutes les portes et fenêtres fermées. Il est sécurisé, mais coupé du reste du monde.

```
┌─────────────────────────────────────────────────────┐
│              SYSTÈME HÔTE                           │
│                                                     │
│  ┌──────────────┐         ┌──────────────┐          │
│  │ Conteneur A  │    ❌   │ Conteneur B  │          │
│  │   (isolé)    │         │   (isolé)    │          │
│  └──────────────┘         └──────────────┘          │
│                                                     │
│  Sans réseau : impossible de communiquer            │
└─────────────────────────────────────────────────────┘
```

### Exemple concret du problème

Lançons deux conteneurs simples pour illustrer :

```bash
# Conteneur 1 : Une base de données PostgreSQL
docker run -d --name ma-db postgres:15

# Conteneur 2 : Une application qui a besoin de cette base
docker run -d --name mon-app mon-application
```

**Problème :** L'application dans `mon-app` ne peut pas se connecter à la base de données dans `ma-db`. Les conteneurs ne se "voient" pas !

C'est là qu'interviennent les réseaux Docker.

## Qu'est-ce qu'un réseau Docker ?

Un réseau Docker est un **canal de communication virtuel** qui permet aux conteneurs de se parler entre eux et avec le monde extérieur.

### Analogie simple : Le réseau comme un téléphone

Pensez aux réseaux Docker comme à un système téléphonique :

- **Conteneurs** = Personnes avec des téléphones
- **Réseau Docker** = Le réseau téléphonique qui les connecte
- **Nom de conteneur** = Numéro de téléphone
- **Ports** = Extensions téléphoniques

Sans le réseau téléphonique, personne ne peut communiquer. Avec le réseau, les appels sont possibles.

```
┌─────────────────────────────────────────────────────┐
│           RÉSEAU DOCKER : "mon-reseau"              │
│                                                     │
│  ┌──────────────┐    ✅    ┌──────────────┐         │
│  │ Conteneur A  │◄────────►│ Conteneur B  │         │
│  │   ma-db      │  Réseau  │   mon-app    │         │
│  └──────────────┘          └──────────────┘         │
│                                                     │
│  Avec réseau : communication possible !             │
└─────────────────────────────────────────────────────┘
```

## Les trois niveaux de communication

Docker gère trois types de communication réseau :

### 1. Conteneur ↔ Conteneur (interne)

Des conteneurs sur le **même réseau Docker** peuvent communiquer entre eux.

```
┌────────────────────────────────────┐
│       Réseau Docker "app"          │
│                                    │
│   ┌──────┐       ┌──────┐          │
│   │ Web  │──────►│  DB  │          │
│   └──────┘       └──────┘          │
│                                    │
└────────────────────────────────────┘
```

**Exemple :** Une application web communique avec sa base de données.

### 2. Hôte ↔ Conteneur (accès local)

Votre machine peut accéder aux services dans les conteneurs via le **mappage de ports**.

```
┌─────────────────────────────────────┐
│         VOTRE ORDINATEUR            │
│                                     │
│   Navigateur                        │
│      │                              │
│      │ localhost:8080               │
│      ↓                              │
│   ┌──────────────────────┐          │
│   │     Conteneur        │          │
│   │   Port 80 ──► 8080   │          │
│   └──────────────────────┘          │
│                                     │
└─────────────────────────────────────┘
```

**Exemple :** Vous accédez à votre application web sur `http://localhost:8080`.

### 3. Conteneur ↔ Internet (externe)

Les conteneurs peuvent accéder à Internet pour télécharger des ressources, appeler des APIs externes, etc.

```
┌─────────────────────────────────────┐
│       Conteneur                     │
│          │                          │
│          ↓                          │
│    [Internet] ───► API externe      │
│                                     │
└─────────────────────────────────────┘
```

**Exemple :** Votre application récupère des données depuis une API tierce.

## Concepts fondamentaux

### 1. Les réseaux Docker par défaut

Lorsque vous installez Docker, trois réseaux sont créés automatiquement :

```bash
docker network ls
```

**Sortie typique :**

```
NETWORK ID     NAME      DRIVER    SCOPE
abc123def456   bridge    bridge    local
def456ghi789   host      host      local
ghi789jkl012   none      null      local
```

- **bridge** : Le réseau par défaut pour les conteneurs
- **host** : Partage le réseau de l'hôte directement
- **none** : Aucun réseau (isolation totale)

Nous explorerons ces types en détail dans la section 7.2.

### 2. Le réseau bridge par défaut

Quand vous lancez un conteneur sans spécifier de réseau, il est automatiquement connecté au réseau **bridge** par défaut.

```bash
docker run -d --name mon-conteneur nginx
# Automatiquement connecté au réseau bridge
```

**Limitation importante :** Sur le réseau bridge par défaut, les conteneurs **ne peuvent pas** se trouver par leur nom. Ils doivent utiliser des adresses IP (ce qui est peu pratique).

### 3. Les réseaux personnalisés (recommandé)

Vous pouvez créer vos propres réseaux. C'est la méthode **recommandée** car elle offre des fonctionnalités supplémentaires, notamment la **résolution DNS automatique**.

```bash
# Créer un réseau personnalisé
docker network create mon-reseau

# Lancer des conteneurs sur ce réseau
docker run -d --name ma-db --network mon-reseau postgres
docker run -d --name mon-app --network mon-reseau mon-application
```

**Avantage :** `mon-app` peut maintenant se connecter à `ma-db` simplement en utilisant le nom `ma-db` comme nom d'hôte !

## La résolution DNS dans Docker

### Qu'est-ce que le DNS ?

Le DNS (Domain Name System) est comme un annuaire téléphonique : il traduit des noms faciles à retenir en adresses IP.

**Exemple dans la vraie vie :**
- Vous tapez `google.com` dans votre navigateur
- Le DNS traduit cela en `142.250.185.46`
- Votre navigateur se connecte à cette adresse IP

### DNS dans les réseaux Docker personnalisés

Sur un réseau Docker personnalisé, Docker fournit un **serveur DNS intégré** qui fait correspondre les noms de conteneurs à leurs adresses IP.

```
┌────────────────────────────────────────┐
│     Réseau Docker : "app-network"      │
│                                        │
│   ┌───────────┐        ┌────────────┐  │
│   │   web     │        │    db      │  │
│   │           │        │            │  │
│   │ "Connecte-toi à    │            │  │
│   │   db:5432"         │  PostgreSQL│  │
│   │           │───────►│            │  │
│   │           │  DNS   │ 172.18.0.2 │  │
│   └───────────┘ résout └────────────┘  │
│                 "db" en                │
│                 172.18.0.2             │
└────────────────────────────────────────┘
```

### Exemple pratique

```bash
# Créer un réseau
docker network create app-network

# Lancer PostgreSQL
docker run -d \
  --name postgres-db \
  --network app-network \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

# Lancer une application Node.js
docker run -d \
  --name nodejs-app \
  --network app-network \
  -e DB_HOST=postgres-db \
  -e DB_PORT=5432 \
  mon-app-nodejs
```

Dans le code de `nodejs-app`, vous pouvez simplement vous connecter à :
```javascript
host: 'postgres-db',  // Le nom du conteneur !
port: 5432
```

Docker résout automatiquement `postgres-db` vers l'adresse IP correcte.

## Le mappage de ports (Port Mapping)

### Le problème

Les conteneurs ont leurs propres ports internes (comme le port 80 pour un serveur web). Mais comment y accéder depuis votre ordinateur ?

### La solution : Le port mapping

Le **port mapping** (ou publication de ports) crée un lien entre un port de votre machine hôte et un port du conteneur.

```
┌─────────────────────────────────────────┐
│         VOTRE ORDINATEUR                │
│                                         │
│   localhost:8080 (port hôte)            │
│        ↓                                │
│        │ Mappage : 8080 → 80            │
│        ↓                                │
│   ┌─────────────────────┐               │
│   │    Conteneur Nginx  │               │
│   │    Port 80          │               │
│   └─────────────────────┘               │
│                                         │
└─────────────────────────────────────────┘
```

### Syntaxe du port mapping

```bash
docker run -p PORT_HOTE:PORT_CONTENEUR image
```

### Exemples concrets

#### Exemple 1 : Serveur web Nginx

```bash
docker run -d -p 8080:80 --name mon-nginx nginx
```

- **Port 80** : Port interne du conteneur (Nginx écoute sur 80)
- **Port 8080** : Port sur votre machine
- Accès : `http://localhost:8080`

#### Exemple 2 : Base de données PostgreSQL

```bash
docker run -d \
  -p 5432:5432 \
  --name ma-db \
  -e POSTGRES_PASSWORD=secret \
  postgres:15
```

- Vous pouvez maintenant vous connecter avec un client PostgreSQL sur `localhost:5432`

#### Exemple 3 : Application sur un port custom

```bash
docker run -d -p 3000:8080 mon-app
```

- L'application écoute sur le port 8080 dans le conteneur
- Mais vous y accédez via `localhost:3000` sur votre machine

### Mapper plusieurs ports

Vous pouvez mapper plusieurs ports en même temps :

```bash
docker run -d \
  -p 8080:80 \
  -p 8443:443 \
  --name web-server \
  nginx
```

### Port aléatoire

Docker peut choisir un port libre automatiquement :

```bash
docker run -d -p 80 nginx
```

Docker choisira un port aléatoire sur l'hôte. Pour le découvrir :

```bash
docker port mon-conteneur
# Sortie : 80/tcp -> 0.0.0.0:32768
```

### Important : Conflits de ports

⚠️ **Attention :** Vous ne pouvez pas utiliser le même port hôte pour deux conteneurs différents.

```bash
# ✅ OK
docker run -d -p 8080:80 nginx
docker run -d -p 8081:80 nginx

# ❌ ERREUR - Port 8080 déjà utilisé
docker run -d -p 8080:80 nginx
docker run -d -p 8080:80 apache
```

## Adresses IP et sous-réseaux

### Chaque conteneur a une adresse IP

Quand un conteneur rejoint un réseau, Docker lui attribue automatiquement une adresse IP unique.

```bash
# Inspecter le réseau d'un conteneur
docker inspect mon-conteneur | grep IPAddress
```

**Sortie typique :**
```json
"IPAddress": "172.17.0.2"
```

### Les sous-réseaux Docker

Chaque réseau Docker a son propre sous-réseau (plage d'adresses IP).

**Exemple :**
- Réseau `bridge` : `172.17.0.0/16`
- Réseau `mon-reseau` : `172.18.0.0/16`

Docker gère automatiquement ces adresses. Vous n'avez généralement pas besoin de vous en soucier, sauf pour des configurations avancées.

## Isolation et sécurité réseau

### Les conteneurs sur des réseaux différents ne peuvent pas communiquer

C'est une fonctionnalité de sécurité importante.

```
┌─────────────────────────────────────────────┐
│                                             │
│   Réseau A                 Réseau B         │
│   ┌─────────┐              ┌─────────┐      │
│   │ App 1   │    ❌        │ App 2   │      │
│   └─────────┘              └─────────┘      │
│                                             │
│   Communication impossible par défaut       │
└─────────────────────────────────────────────┘
```

### Connecter un conteneur à plusieurs réseaux

Si nécessaire, un conteneur peut appartenir à plusieurs réseaux :

```bash
# Créer deux réseaux
docker network create frontend
docker network create backend

# Lancer un conteneur
docker run -d --name app --network frontend mon-app

# Connecter le même conteneur au réseau backend
docker network connect backend app
```

Maintenant, `app` peut communiquer avec des conteneurs sur les deux réseaux.

## Scénario pratique : Application web complète

Mettons en pratique ces concepts avec une architecture typique.

### Architecture

```
┌─────────────────────────────────────────────────────┐
│                  SYSTÈME HÔTE                       │
│                                                     │
│   Navigateur (localhost:80)                         │
│        │                                            │
│        ↓ Port mapping 80:80                         │
│   ┌─────────────────────────────────────┐           │
│   │     Réseau Docker : "app-network"   │           │
│   │                                     │           │
│   │   ┌────────┐      ┌────────┐        │           │
│   │   │ Nginx  │─────►│  API   │        │           │
│   │   │  :80   │      │ :3000  │        │           │
│   │   └────────┘      └───┬────┘        │           │
│   │                       │             │           │
│   │                       ↓             │           │
│   │                  ┌────────┐         │           │
│   │                  │   DB   │         │           │
│   │                  │ :5432  │         │           │
│   │                  └────────┘         │           │
│   │                                     │           │
│   └─────────────────────────────────────┘           │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### Commandes pour créer cette architecture

```bash
# 1. Créer le réseau
docker network create app-network

# 2. Lancer la base de données (non exposée publiquement)
docker run -d \
  --name postgres-db \
  --network app-network \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

# 3. Lancer l'API (non exposée publiquement)
docker run -d \
  --name api-backend \
  --network app-network \
  -e DB_HOST=postgres-db \
  mon-api:latest

# 4. Lancer Nginx (exposé sur le port 80)
docker run -d \
  --name nginx-frontend \
  --network app-network \
  -p 80:80 \
  nginx
```

### Points importants de cette configuration

1. **Un seul réseau** : Tous les conteneurs sont sur `app-network`
2. **DNS automatique** : L'API peut se connecter à `postgres-db` par son nom
3. **Sécurité** : Seul Nginx est exposé publiquement (port 80)
4. **Isolation** : La base de données n'est pas accessible depuis l'extérieur

## Commandes de base pour les réseaux

Voici les commandes essentielles pour gérer les réseaux Docker :

### Lister les réseaux

```bash
docker network ls
```

### Créer un réseau

```bash
docker network create nom-du-reseau
```

### Inspecter un réseau

```bash
docker network inspect nom-du-reseau
```

Cela affiche tous les conteneurs connectés, les adresses IP, la configuration, etc.

### Connecter un conteneur à un réseau

```bash
docker network connect nom-du-reseau nom-du-conteneur
```

### Déconnecter un conteneur d'un réseau

```bash
docker network disconnect nom-du-reseau nom-du-conteneur
```

### Supprimer un réseau

```bash
docker network rm nom-du-reseau
```

⚠️ **Note :** Vous ne pouvez pas supprimer un réseau qui a encore des conteneurs connectés.

### Nettoyer les réseaux inutilisés

```bash
docker network prune
```

## Bonnes pratiques

### 1. Utilisez des réseaux personnalisés

❌ **À éviter :**
```bash
docker run -d --name app1 mon-app
docker run -d --name app2 mon-app
# Utilise le réseau bridge par défaut (pas de DNS)
```

✅ **Recommandé :**
```bash
docker network create mon-reseau
docker run -d --name app1 --network mon-reseau mon-app
docker run -d --name app2 --network mon-reseau mon-app
# DNS automatique, isolation, meilleure gestion
```

### 2. Séparez vos services par réseau

Créez différents réseaux selon les besoins de communication :

```bash
docker network create frontend   # Pour les services web
docker network create backend    # Pour les APIs
docker network create database   # Pour les BDD
```

### 3. N'exposez que ce qui est nécessaire

Seuls les services qui doivent être accessibles depuis l'extérieur doivent avoir des ports mappés.

```bash
# ✅ BON - Seul Nginx est exposé
docker run -d --network app -p 80:80 nginx
docker run -d --network app mon-api        # Pas de -p
docker run -d --network app postgres       # Pas de -p

# ❌ MAUVAIS - Tout est exposé
docker run -d -p 80:80 nginx
docker run -d -p 3000:3000 mon-api
docker run -d -p 5432:5432 postgres
```

### 4. Nommez vos conteneurs de manière significative

Les noms deviennent des noms d'hôte dans le réseau :

```bash
# ✅ BON - Noms clairs
docker run -d --name api-users --network app mon-api
docker run -d --name db-users --network app postgres

# ❌ PAS CLAIR
docker run -d --name cont1 --network app mon-api
docker run -d --name cont2 --network app postgres
```

### 5. Documentez votre architecture réseau

Créez un diagramme ou une documentation qui explique :
- Quels réseaux existent
- Quels conteneurs sont sur quels réseaux
- Quels ports sont exposés et pourquoi

## Déboguer les problèmes de réseau

### Problème : Les conteneurs ne peuvent pas communiquer

**Vérifications :**

1. Sont-ils sur le même réseau ?
```bash
docker network inspect mon-reseau
```

2. Les conteneurs sont-ils en cours d'exécution ?
```bash
docker ps
```

3. Utilisez-vous le bon nom d'hôte ?
```bash
# À l'intérieur d'un conteneur
docker exec mon-conteneur ping autre-conteneur
```

### Problème : Impossible d'accéder au conteneur depuis l'hôte

**Vérifications :**

1. Le port est-il bien mappé ?
```bash
docker port mon-conteneur
```

2. Le service écoute-t-il sur le bon port ?
```bash
docker logs mon-conteneur
```

3. Y a-t-il un pare-feu qui bloque ?
```bash
# Tester avec curl
curl localhost:8080
```

## À retenir

🔑 **Points clés :**

- Les conteneurs sont **isolés par défaut** et ont besoin de réseaux pour communiquer
- Docker crée trois réseaux par défaut : **bridge**, **host**, et **none**
- Les **réseaux personnalisés** offrent la résolution DNS automatique (recommandé)
- Le **mappage de ports** (`-p`) permet d'accéder aux conteneurs depuis l'hôte
- Sur un réseau personnalisé, les conteneurs se trouvent par leur **nom** (DNS)
- Un conteneur peut appartenir à **plusieurs réseaux**
- Seul ce qui doit être accessible publiquement devrait avoir des ports exposés

### Analogie récapitulative

Pensez aux réseaux Docker comme à un immeuble :
- **Chaque appartement** = Un conteneur (isolé)
- **Le couloir d'étage** = Un réseau Docker (permet la communication)
- **L'interphone** = Le mappage de ports (accès depuis l'extérieur)
- **Les noms sur les boîtes aux lettres** = DNS (trouve qui habite où)

## Prochaine étape

Maintenant que vous comprenez les principes fondamentaux des réseaux Docker, la section suivante (7.2) explorera en détail les **différents types de réseaux** disponibles : bridge, host, none, et overlay. Vous apprendrez quand et comment utiliser chacun d'eux.

⏭️ [Types de réseaux (bridge, host, none, overlay)](/07-reseaux-docker/02-types-de-reseaux.md)
