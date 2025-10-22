🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 3.3 Registres Docker (Docker Hub)

## Qu'est-ce qu'un registre Docker ?

Un registre Docker est un **service de stockage et de distribution d'images Docker**. C'est l'équivalent d'un "magasin d'applications" ou d'une bibliothèque centralisée où vous pouvez :
- **Télécharger** des images créées par d'autres
- **Publier** vos propres images
- **Partager** des images avec votre équipe ou la communauté

### Analogies pour bien comprendre

**Analogie 1 - L'App Store/Play Store :**
- **Registre Docker** = App Store pour applications mobiles
- **Images** = Applications disponibles au téléchargement
- **Pull** = Télécharger une application
- **Push** = Publier votre application sur le store

**Analogie 2 - GitHub :**
- **Registre Docker** = GitHub (pour le code)
- **Images** = Repositories de code
- **Docker Hub** = GitHub.com (la plateforme principale)
- **Pull/Push** = Clone/Push de repositories

**Analogie 3 - Une bibliothèque :**
- **Registre Docker** = La bibliothèque municipale
- **Images** = Les livres disponibles
- **Pull** = Emprunter un livre
- **Push** = Déposer votre propre livre

## Le workflow avec un registre

Voici comment les images circulent entre votre machine, le registre et d'autres machines :

```
     DÉVELOPPEUR A                    DÉVELOPPEUR B
    (Votre machine)                  (Machine collègue)
          │                                  │
          │ docker push                      │ docker pull
          ↓                                  ↓
    ┌─────────────────────────────────────────────┐
    │           REGISTRE DOCKER                   │
    │            (Docker Hub)                     │
    │                                             │
    │  ┌────────┐  ┌────────┐  ┌────────┐         │
    │  │ nginx  │  │ python │  │ mon-app│         │
    │  │:latest │  │:3.11   │  │:v1.0   │         │
    │  └────────┘  └────────┘  └────────┘         │
    └─────────────────────────────────────────────┘
          ↑                                  │
          │ docker pull                      │ docker push
          │                                  │
    SERVEUR PRODUCTION              SERVEUR DE TEST
```

### Les opérations principales

**Pull (Télécharger)** : Récupérer une image du registre vers votre machine locale
```
Registre → Votre machine
```

**Push (Publier)** : Envoyer une image de votre machine vers le registre
```
Votre machine → Registre
```

**Search (Rechercher)** : Trouver des images disponibles dans le registre
```
Votre recherche → Résultats du registre
```

## Docker Hub : Le registre officiel

**Docker Hub** (https://hub.docker.com) est le registre public officiel maintenu par Docker Inc. C'est le plus grand et le plus populaire registre d'images Docker.

### Les chiffres

- Plus de **100 000** images publiques disponibles
- Des **milliards** de téléchargements d'images
- Utilisé par des **millions** de développeurs dans le monde
- **Gratuit** pour les images publiques (avec limitations)

### Ce que vous y trouvez

1. **Images officielles** : Maintenues par Docker et les éditeurs (nginx, mysql, python...)
2. **Images vérifiées** : Images de qualité provenant d'éditeurs reconnus
3. **Images communautaires** : Créées et partagées par des utilisateurs
4. **Vos propres images** : Vos créations personnelles ou d'entreprise

### Compte gratuit vs payant

**Compte gratuit :**
- ✅ Téléchargement illimité d'images publiques
- ✅ Publication d'images publiques illimitées
- ✅ 1 registre privé gratuit
- ⚠️ Limitations sur les pull rates (nombre de téléchargements par période)

**Compte payant (Pro, Team, Business) :**
- ✅ Plus de registres privés
- ✅ Pull rates plus élevés
- ✅ Support prioritaire
- ✅ Fonctionnalités avancées (scan de sécurité, webhooks...)

## Types d'images sur Docker Hub

### 1. Images Officielles (Official Images)

Ce sont les images **les plus fiables et recommandées**.

**Caractéristiques :**
- Badge "Official Image" sur Docker Hub
- Maintenues par Docker Inc. ou l'éditeur officiel du logiciel
- Mises à jour régulières et correctifs de sécurité
- Documentation complète et détaillée
- Testées et optimisées
- Nom simple sans préfixe : `nginx`, `ubuntu`, `postgres`

**Exemples populaires :**

| Image | Description | Utilisation |
|-------|-------------|-------------|
| `nginx` | Serveur web haute performance | Servir des sites web statiques ou reverse proxy |
| `node` | Runtime JavaScript | Applications Node.js |
| `python` | Langage Python | Scripts et applications Python |
| `mysql` | Base de données relationnelle | Stockage de données structurées |
| `postgres` | Base de données relationnelle | Alternative à MySQL, plus riche en fonctionnalités |
| `redis` | Cache en mémoire | Système de cache, sessions, files de messages |
| `ubuntu` | Distribution Linux | Base pour créer vos propres images |
| `alpine` | Distribution Linux ultra-légère | Images optimisées en taille |
| `mongo` | Base de données NoSQL | Stockage de documents JSON |

### 2. Images Vérifiées (Verified Publishers)

**Caractéristiques :**
- Badge "Verified Publisher"
- Proviennent d'entreprises reconnues et vérifiées
- Qualité contrôlée
- Support commercial souvent disponible
- Exemple : `datadog/agent`, `elastic/elasticsearch`

### 3. Images Communautaires (Community Images)

**Caractéristiques :**
- Créées par des utilisateurs individuels ou organisations
- Préfixées par le nom d'utilisateur : `utilisateur/nom-image`
- Qualité variable
- À utiliser avec discernement

**Comment évaluer une image communautaire :**

✅ **Nombre de téléchargements** : Plus c'est élevé, plus elle est populaire et testée
✅ **Date de dernière mise à jour** : Image maintenue récemment ?
✅ **Nombre d'étoiles** : Popularité et satisfaction des utilisateurs
✅ **Documentation** : README complet et clair ?
✅ **Dockerfile disponible** : Transparence sur le contenu ?

⚠️ **Signaux d'alerte :**
- Peu ou pas de téléchargements
- Pas mise à jour depuis longtemps
- Pas de documentation
- Pas de Dockerfile source visible

## Nomenclature complète des images

### Structure d'un nom d'image complet

```
[registre/][organisation ou utilisateur/]nom[:tag]
```

### Exemples décortiqués

**Exemple 1 : Image officielle simple**
```
nginx
```
- Registre : Docker Hub (par défaut)
- Organisation : (aucune, c'est une image officielle)
- Nom : nginx
- Tag : latest (par défaut)

**Nom complet implicite :** `docker.io/library/nginx:latest`

**Exemple 2 : Image officielle avec tag spécifique**
```
nginx:1.24-alpine
```
- Registre : Docker Hub (par défaut)
- Organisation : (aucune, c'est une image officielle)
- Nom : nginx
- Tag : 1.24-alpine (version 1.24 basée sur Alpine Linux)

**Exemple 3 : Image communautaire**
```
johndoe/mon-application:v2.1
```
- Registre : Docker Hub (par défaut)
- Organisation/Utilisateur : johndoe
- Nom : mon-application
- Tag : v2.1

**Exemple 4 : Image depuis un registre privé**
```
registry.monentreprise.com/equipe-backend/api:production
```
- Registre : registry.monentreprise.com
- Organisation : equipe-backend
- Nom : api
- Tag : production

### Comprendre les tags

Les tags sont essentiels pour gérer les **versions** de vos images.

**Conventions courantes :**

| Tag | Signification | Exemple |
|-----|---------------|---------|
| `latest` | Dernière version (par défaut) | `nginx:latest` |
| `X.Y.Z` | Version sémantique précise | `node:18.17.0` |
| `X.Y` | Version majeure.mineure | `python:3.11` |
| `X` | Version majeure | `redis:7` |
| `slim` | Version allégée | `python:3.11-slim` |
| `alpine` | Basée sur Alpine Linux (ultra-légère) | `node:18-alpine` |
| `buster`, `bullseye` | Basée sur Debian spécifique | `python:3.11-bullseye` |
| `dev`, `prod` | Environnement | `mon-app:prod` |
| `v1.0`, `v2.0` | Version personnalisée | `mon-api:v2.0` |

**⚠️ Important :** Le tag `latest` ne signifie pas toujours "la plus récente version". C'est simplement le tag par défaut appliqué par convention.

**Bonne pratique :**
- **Développement** : Utilisez `latest` pour avoir les dernières fonctionnalités
- **Production** : Utilisez des tags de version spécifiques (`3.11`, `1.24.0`) pour la stabilité

## Architecture des images multi-plateformes

Certaines images supportent **plusieurs architectures** (processeurs différents) :

- **amd64 / x86_64** : Processeurs Intel/AMD (PC standard, serveurs)
- **arm64 / aarch64** : Processeurs ARM 64 bits (Mac M1/M2, Raspberry Pi 4)
- **armv7** : Processeurs ARM 32 bits (Raspberry Pi 3)

**Avantage :** Docker télécharge automatiquement la version compatible avec votre processeur !

**Exemple :**
```bash
# Sur un Mac M1 (ARM)
docker pull python:3.11
# → Télécharge automatiquement python:3.11 pour arm64

# Sur un PC Windows (Intel)
docker pull python:3.11
# → Télécharge automatiquement python:3.11 pour amd64
```

## Rechercher des images sur Docker Hub

### Via le site web

1. Allez sur https://hub.docker.com
2. Utilisez la barre de recherche
3. Filtrez par :
   - Images officielles
   - Images vérifiées
   - Tri par popularité ou pertinence

### Via la ligne de commande

```bash
# Rechercher des images nginx
docker search nginx

# Rechercher avec filtre (images officielles uniquement)
docker search --filter "is-official=true" nginx

# Limiter les résultats
docker search --limit 5 python
```

**Résultat typique :**
```
NAME                DESCRIPTION                    STARS    OFFICIAL
nginx               Official build of Nginx        19000    [OK]
jwilder/nginx-proxy Automated nginx proxy          2500
bitnami/nginx       Bitnami nginx Docker Image     250
```

## Registres alternatifs

Docker Hub n'est pas le seul registre disponible. Il en existe d'autres :

### Registres publics

**1. GitHub Container Registry (ghcr.io)**
- Intégré à GitHub
- Gratuit pour les projets publics
- Exemple : `ghcr.io/utilisateur/mon-image:latest`

**2. GitLab Container Registry**
- Intégré à GitLab
- CI/CD natif
- Exemple : `registry.gitlab.com/utilisateur/projet/image:tag`

**3. Google Container Registry (GCR)**
- Service Google Cloud
- Exemple : `gcr.io/mon-projet/mon-image:tag`

**4. Amazon Elastic Container Registry (ECR)**
- Service AWS
- Haute intégration avec l'écosystème AWS
- Exemple : `123456789.dkr.ecr.us-east-1.amazonaws.com/mon-image:tag`

**5. Azure Container Registry**
- Service Microsoft Azure
- Exemple : `monregistre.azurecr.io/mon-image:tag`

**6. Quay.io**
- Par Red Hat
- Scan de sécurité avancé
- Exemple : `quay.io/utilisateur/mon-image:tag`

### Registres privés

Vous pouvez aussi héberger votre **propre registre Docker** :

**Docker Registry (Open Source)**
- Solution officielle de Docker
- Gratuite et auto-hébergée
- Idéale pour les environnements d'entreprise

**Harbor (CNCF Project)**
- Registre open source avec interface web
- Gestion avancée des permissions
- Scan de vulnérabilités intégré
- Réplication entre registres

**Artifactory / Nexus**
- Solutions commerciales
- Supportent multiples formats (Docker, npm, Maven...)
- Fonctionnalités enterprise

## Sécurité et confiance

### Scan de vulnérabilités

Docker Hub propose un **scan automatique** des images pour détecter les vulnérabilités de sécurité connues.

**Ce qui est scanné :**
- Vulnérabilités dans les paquets système
- Bibliothèques avec failles connues (CVE)
- Configurations à risque

**Résultat :**
```
Critique   : 2 vulnérabilités
Élevé      : 5 vulnérabilités
Moyen      : 12 vulnérabilités
Faible     : 8 vulnérabilités
```

### Docker Content Trust

**Docker Content Trust (DCT)** garantit l'**authenticité et l'intégrité** des images :
- Vérification que l'image provient bien de l'éditeur
- Garantie que l'image n'a pas été modifiée
- Signatures cryptographiques

**Activation :**
```bash
export DOCKER_CONTENT_TRUST=1
```

Une fois activé, seules les images signées peuvent être téléchargées.

## Rate Limiting (Limitations de téléchargement)

Docker Hub impose des **limites de téléchargement** pour éviter les abus :

**Utilisateurs anonymes :**
- 100 pulls par 6 heures par adresse IP

**Utilisateurs authentifiés (gratuit) :**
- 200 pulls par 6 heures

**Utilisateurs payants :**
- Limites beaucoup plus élevées

**Conseil :** Authentifiez-vous toujours pour éviter les limitations :
```bash
docker login
```

## Bonnes pratiques avec les registres

### 1. Privilégiez les images officielles

✅ Fiables, maintenues, sécurisées
✅ Documentation complète
✅ Communauté large en cas de problème

### 2. Utilisez des tags de version spécifiques

✅ Production : `python:3.11.5` plutôt que `python:latest`
✅ Reproductibilité garantie
✅ Évite les surprises lors des mises à jour

### 3. Vérifiez les images communautaires

✅ Nombre de téléchargements élevé
✅ Mise à jour récente
✅ Documentation claire
✅ Dockerfile source disponible

### 4. Scannez régulièrement vos images

✅ Détectez les vulnérabilités
✅ Mettez à jour les images obsolètes
✅ Utilisez les outils de scan intégrés

### 5. Utilisez un registre privé pour le code propriétaire

✅ Ne publiez jamais de code propriétaire sur un registre public
✅ Configurez un registre privé (Docker Hub, Harbor, ECR...)
✅ Gérez les accès avec des permissions appropriées

### 6. Organisez vos images avec des tags cohérents

✅ Convention de nommage claire : `v1.0.0`, `prod-2024-01-15`
✅ Tags sémantiques : `latest`, `stable`, `dev`
✅ Documentation des tags dans le README

### 7. Nettoyez les anciennes versions

✅ Supprimez les tags obsolètes
✅ Économisez de l'espace de stockage
✅ Réduisez la confusion sur les versions à utiliser

## Workflow typique avec Docker Hub

### 1. Recherche et téléchargement

```
Développeur → Recherche "postgresql" sur Docker Hub
           → Trouve l'image officielle postgres
           → Lit la documentation
           → Télécharge l'image : docker pull postgres:15
```

### 2. Développement local

```
Image locale postgres:15
           → Création d'un conteneur
           → Tests et développement
           → Modifications du code applicatif
```

### 3. Création d'une image personnalisée

```
Création d'un Dockerfile
           → Build de l'image personnalisée
           → Tag : mon-app:v1.0
           → Tests locaux
```

### 4. Publication

```
docker tag mon-app:v1.0 utilisateur/mon-app:v1.0
           → docker push utilisateur/mon-app:v1.0
           → Image disponible sur Docker Hub
```

### 5. Déploiement

```
Serveur de production → docker pull utilisateur/mon-app:v1.0
                      → docker run utilisateur/mon-app:v1.0
                      → Application en production
```

## Comprendre la page d'une image sur Docker Hub

Lorsque vous consultez une image sur Docker Hub, vous trouvez :

### Onglet "Overview" (Vue d'ensemble)
- Description de l'image
- Readme détaillé
- Exemples d'utilisation
- Documentation complète

### Onglet "Tags"
- Liste de tous les tags disponibles
- Taille de chaque variante
- Architecture supportée (amd64, arm64...)
- Date de dernière mise à jour

### Onglet "Vulnerabilities" (Vulnérabilités)
- Scan de sécurité
- Liste des CVE détectées
- Niveau de criticité

### Onglet "Builds" (Constructions)
- Historique des builds
- Logs de construction
- Statut (succès/échec)

## Authentification sur Docker Hub

### Se connecter

```bash
docker login
# Entrez votre nom d'utilisateur Docker Hub
# Entrez votre mot de passe
```

**Résultat :**
```
Login Succeeded
```

Vos identifiants sont stockés de manière sécurisée localement.

### Se déconnecter

```bash
docker logout
```

### Utiliser un token d'accès (Recommandé)

Au lieu d'un mot de passe, utilisez un **Personal Access Token** :
1. Générez un token sur Docker Hub (Settings → Security)
2. Utilisez-le comme mot de passe lors du login

**Avantages :**
- Plus sécurisé qu'un mot de passe
- Peut être révoqué sans changer le mot de passe
- Permissions granulaires

## Résumé des points clés

✅ Un registre Docker est un **entrepôt centralisé** pour stocker et partager des images

✅ **Docker Hub** est le registre public officiel, avec des centaines de milliers d'images

✅ Privilégiez les **images officielles** (badge "Official Image") pour la fiabilité et la sécurité

✅ Les images communautaires peuvent être utiles, mais **vérifiez leur qualité** avant utilisation

✅ Utilisez des **tags de version spécifiques** en production pour la stabilité

✅ La nomenclature complète : `[registre/][utilisateur/]nom:tag`

✅ Il existe de nombreux **registres alternatifs** (GitHub, GitLab, AWS ECR, Azure ACR...)

✅ **Scannez** régulièrement vos images pour détecter les vulnérabilités

✅ **Authentifiez-vous** sur Docker Hub pour éviter les limitations de téléchargement

## Pour aller plus loin

Dans le prochain chapitre (3.4), nous découvrirons le **cycle de vie d'un conteneur** : tous les états par lesquels passe un conteneur depuis sa création jusqu'à sa suppression, et comment gérer ces transitions.

Ensuite, au chapitre 4, nous mettrons tout en pratique avec les **commandes Docker essentielles** pour rechercher, télécharger, et gérer vos images et conteneurs !

---

*Vous savez maintenant où trouver des images de qualité et comment naviguer dans l'écosystème des registres Docker. Prêt à comprendre le cycle de vie complet d'un conteneur ?*

⏭️ [Le cycle de vie d'un conteneur](/03-concepts-fondamentaux/04-cycle-de-vie-dun-conteneur.md)
