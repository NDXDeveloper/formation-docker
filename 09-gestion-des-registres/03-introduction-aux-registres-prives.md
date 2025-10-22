🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 9.3 Introduction aux registres privés

## Introduction

Jusqu'à présent, nous avons travaillé avec Docker Hub et ses repositories publics, où vos images sont accessibles à tout le monde. Mais que se passe-t-il lorsque vous développez une application propriétaire, une solution pour un client, ou que vous ne voulez simplement pas partager votre code avec le monde entier ?

C'est là qu'interviennent les **registres privés** ! 🔒

Un registre privé vous permet d'héberger vos images Docker de manière sécurisée, en contrôlant précisément qui peut y accéder. C'est l'équivalent d'un GitHub privé, mais pour vos images Docker.

Dans cette section, nous allons explorer ce que sont les registres privés, pourquoi les utiliser, et quelles sont les différentes options disponibles.

## Qu'est-ce qu'un registre privé ?

### Définition

Un **registre privé** (ou private registry) est un service qui héberge vos images Docker de manière sécurisée, avec contrôle d'accès. Seules les personnes autorisées peuvent :
- Voir vos images
- Télécharger (pull) vos images
- Pousser (push) de nouvelles images

### Registre public vs Registre privé

**Registre public (Docker Hub gratuit)** :
```
🌍 Visible par tout le monde
📥 Téléchargeable par tout le monde
💰 Gratuit et illimité
📖 Open source, partage de connaissances
```

**Registre privé** :
```
🔒 Visible uniquement par les personnes autorisées
🔐 Nécessite une authentification
💼 Pour le code propriétaire
🏢 Pour les entreprises
```

### Analogie simple

Pensez à un registre Docker comme à un entrepôt :
- **Registre public** = Bibliothèque publique : tout le monde peut emprunter les livres
- **Registre privé** = Coffre-fort personnel : seules les personnes avec la clé peuvent accéder

## Pourquoi utiliser un registre privé ?

### 1. Confidentialité et sécurité

Votre code et vos configurations restent privés. Personne ne peut voir ce qu'il y a dans vos images.

```bash
# ❌ Public : tout le monde peut télécharger
docker pull johndoe/mon-app-secrete:latest

# ✅ Privé : authentification requise
docker login private-registry.com
docker pull private-registry.com/johndoe/mon-app-secrete:latest
```

### 2. Propriété intellectuelle

Protégez votre code propriétaire, vos algorithmes secrets, ou vos solutions client.

### 3. Conformité légale

Certaines réglementations (RGPD, HIPAA, etc.) exigent que vos données restent dans certaines juridictions ou sous votre contrôle.

### 4. Contrôle d'accès granulaire

Définissez précisément qui peut faire quoi :
- Alice peut push et pull
- Bob peut seulement pull
- L'équipe A accède aux images frontend
- L'équipe B accède aux images backend

### 5. Performance

Un registre privé auto-hébergé dans votre datacenter ou cloud peut être plus rapide qu'un service public distant.

### 6. Pas de limites de pull

Docker Hub gratuit a des limites de téléchargements (rate limits). Avec votre propre registre, pas de limites !

### 7. Personnalisation

Vous contrôlez entièrement votre infrastructure : stockage, politique de rétention, backups, etc.

## Types de registres privés

Il existe plusieurs options pour héberger un registre privé. Voyons les principales.

### 1. Docker Hub - Repositories privés

**Description** : Docker Hub propose aussi des repositories privés.

**Avantages** :
- ✅ Facile à utiliser (même interface que les repos publics)
- ✅ Pas de serveur à gérer
- ✅ Intégration directe avec Docker
- ✅ 1 repository privé gratuit

**Inconvénients** :
- ❌ Limité à 1 repository privé dans le plan gratuit
- ❌ Données hébergées chez Docker Inc.
- ❌ Coût pour plus de repositories

**Prix** :
- Gratuit : 1 repository privé
- Pro ($5/mois) : Repositories privés illimités
- Team/Business : Plus de fonctionnalités

**Idéal pour** : Petits projets, freelancers, équipes de 1-5 personnes.

### 2. GitHub Container Registry (ghcr.io)

**Description** : Le registre de conteneurs intégré à GitHub.

**Avantages** :
- ✅ Gratuit pour les repositories publics
- ✅ Intégré à GitHub (même authentification)
- ✅ Excellente intégration avec GitHub Actions
- ✅ Packages privés gratuits avec des limites généreuses

**Inconvénients** :
- ❌ Nécessite un compte GitHub
- ❌ Moins de fonctionnalités qu'un registre dédié

**Format des images** :
```
ghcr.io/username/image:tag
ghcr.io/organization/image:tag
```

**Idéal pour** : Projets open source, projets déjà sur GitHub.

### 3. GitLab Container Registry

**Description** : Registre intégré à GitLab.

**Avantages** :
- ✅ Inclus gratuitement avec GitLab (même self-hosted)
- ✅ Intégré au CI/CD GitLab
- ✅ Privé par défaut
- ✅ Pas de limite de repositories

**Inconvénients** :
- ❌ Nécessite GitLab
- ❌ Moins connu que Docker Hub

**Format des images** :
```
registry.gitlab.com/username/project:tag
```

**Idéal pour** : Équipes utilisant déjà GitLab.

### 4. Amazon ECR (Elastic Container Registry)

**Description** : Service de registre Docker d'AWS.

**Avantages** :
- ✅ Haute performance sur AWS
- ✅ Sécurité et conformité AWS
- ✅ Intégration avec ECS, EKS
- ✅ Scalabilité illimitée
- ✅ Scan automatique des vulnérabilités

**Inconvénients** :
- ❌ Payant (stockage + transfert de données)
- ❌ Complexité AWS
- ❌ Nécessite un compte AWS

**Format des images** :
```
123456789012.dkr.ecr.eu-west-1.amazonaws.com/mon-app:tag
```

**Idéal pour** : Applications déployées sur AWS, grandes entreprises.

### 5. Google Container Registry (GCR) / Artifact Registry

**Description** : Service de registre Docker de Google Cloud.

**Avantages** :
- ✅ Haute performance sur GCP
- ✅ Intégration avec GKE
- ✅ Scan de vulnérabilités
- ✅ Geo-réplication

**Inconvénients** :
- ❌ Payant
- ❌ Nécessite un compte Google Cloud

**Format des images** :
```
gcr.io/project-id/image:tag
europe-west1-docker.pkg.dev/project-id/repo/image:tag
```

**Idéal pour** : Applications sur Google Cloud Platform.

### 6. Azure Container Registry (ACR)

**Description** : Service de registre Docker de Microsoft Azure.

**Avantages** :
- ✅ Intégration Azure
- ✅ Support des images multi-architecture
- ✅ Geo-réplication
- ✅ Scan de sécurité

**Inconvénients** :
- ❌ Payant
- ❌ Nécessite un compte Azure

**Format des images** :
```
myregistry.azurecr.io/image:tag
```

**Idéal pour** : Applications sur Microsoft Azure.

### 7. Harbor (Self-hosted)

**Description** : Solution open source de registre Docker auto-hébergé.

**Avantages** :
- ✅ Open source et gratuit
- ✅ Contrôle total
- ✅ Fonctionnalités enterprise
- ✅ Scan de vulnérabilités intégré
- ✅ Réplication entre registres
- ✅ Interface web avancée

**Inconvénients** :
- ❌ Infrastructure à gérer soi-même
- ❌ Maintenance et mises à jour
- ❌ Besoin de compétences système

**Idéal pour** : Grandes entreprises, besoin de contrôle total, on-premise.

### 8. Docker Registry (Self-hosted)

**Description** : Le registre Docker officiel open source (version basique).

**Avantages** :
- ✅ Open source et gratuit
- ✅ Léger et simple
- ✅ Contrôle total

**Inconvénients** :
- ❌ Très basique (pas d'interface web)
- ❌ Pas de scan de sécurité
- ❌ Configuration manuelle
- ❌ Infrastructure à gérer

**Idéal pour** : Petites équipes, projets simples, apprentissage.

### 9. JFrog Artifactory

**Description** : Solution commerciale enterprise avec support Docker.

**Avantages** :
- ✅ Support enterprise
- ✅ Multi-format (Docker, Maven, npm, etc.)
- ✅ Fonctionnalités avancées
- ✅ Haute disponibilité

**Inconvénients** :
- ❌ Coûteux
- ❌ Complexe

**Idéal pour** : Grandes entreprises avec besoins avancés.

## Tableau comparatif des options

| Registre | Coût | Hébergement | Complexité | Idéal pour |
|----------|------|-------------|------------|-----------|
| **Docker Hub Private** | Gratuit (limité) / Payant | Cloud | Faible | Petits projets |
| **GitHub Container Registry** | Gratuit | Cloud | Faible | Projets GitHub |
| **GitLab Container Registry** | Gratuit | Cloud/Self | Faible | Projets GitLab |
| **AWS ECR** | Payant | Cloud | Moyenne | Applications AWS |
| **Google GCR/Artifact** | Payant | Cloud | Moyenne | Applications GCP |
| **Azure ACR** | Payant | Cloud | Moyenne | Applications Azure |
| **Harbor** | Gratuit | Self-hosted | Élevée | Enterprise, on-premise |
| **Docker Registry** | Gratuit | Self-hosted | Moyenne | Petites équipes |
| **JFrog Artifactory** | Payant | Cloud/Self | Élevée | Enterprise |

## Comment fonctionne un registre privé ?

### Architecture basique

```
[Votre machine]
      ↓
   docker login
      ↓
[Registre privé] ← Authentification
      ↓
  docker push/pull
      ↓
[Vos images Docker stockées]
```

### Le processus d'authentification

1. **Vous vous authentifiez** :
   ```bash
   docker login registry.example.com
   ```

2. **Docker stocke vos credentials** :
   - Linux : `~/.docker/config.json`
   - Windows : `%USERPROFILE%\.docker\config.json`
   - Mac : `~/.docker/config.json`

3. **Les commandes Docker utilisent automatiquement ces credentials** :
   ```bash
   docker pull registry.example.com/mon-app:latest
   docker push registry.example.com/mon-app:latest
   ```

### Format des URLs de registre

Chaque registre privé a sa propre URL :

```bash
# Docker Hub (pas besoin de spécifier l'URL)
docker pull johndoe/mon-app:latest

# GitHub Container Registry
docker pull ghcr.io/johndoe/mon-app:latest

# GitLab
docker pull registry.gitlab.com/johndoe/mon-projet/mon-app:latest

# AWS ECR
docker pull 123456789012.dkr.ecr.eu-west-1.amazonaws.com/mon-app:latest

# Registre personnalisé
docker pull registry.mon-entreprise.com/mon-app:latest
```

## Utiliser un registre privé : Exemple avec GitHub Container Registry

Voyons un exemple concret avec GitHub Container Registry (gratuit et facile).

### Étape 1 : Créer un Personal Access Token

1. Allez sur GitHub → Settings → Developer settings → Personal access tokens
2. Générez un nouveau token (classic)
3. Cochez les permissions :
   - `write:packages` (pour push)
   - `read:packages` (pour pull)
   - `delete:packages` (optionnel)
4. Copiez le token (vous ne le reverrez plus !)

### Étape 2 : Se connecter au registre

```bash
# Sauvegardez votre token dans une variable
export CR_PAT=YOUR_TOKEN

# Se connecter
echo $CR_PAT | docker login ghcr.io -u USERNAME --password-stdin
```

**Résultat** :
```
Login Succeeded
```

### Étape 3 : Taguer votre image

```bash
# Format : ghcr.io/username/image:tag
docker tag mon-app ghcr.io/johndoe/mon-app:latest
```

### Étape 4 : Pousser l'image

```bash
docker push ghcr.io/johndoe/mon-app:latest
```

### Étape 5 : Vérifier sur GitHub

1. Allez sur votre profil GitHub
2. Cliquez sur "Packages"
3. Vous verrez votre image !

### Étape 6 : Rendre le package privé (si public par défaut)

1. Cliquez sur votre package
2. Package settings → Change visibility → Private

### Étape 7 : Utiliser l'image ailleurs

```bash
# Sur une autre machine
docker login ghcr.io
docker pull ghcr.io/johndoe/mon-app:latest
docker run ghcr.io/johndoe/mon-app:latest
```

## Sécurité des registres privés

### 1. Authentification forte

**Utilisez toujours** :
- ✅ Tokens d'accès (pas de mots de passe en clair)
- ✅ Authentification à deux facteurs (2FA)
- ✅ Rotation régulière des credentials

**N'utilisez jamais** :
- ❌ Mots de passe dans les scripts
- ❌ Credentials partagés entre personnes
- ❌ Tokens avec permissions excessives

### 2. HTTPS obligatoire

Tous les registres modernes utilisent HTTPS. Ne configurez **jamais** un registre en HTTP en production.

```bash
# ❌ Dangereux (credentials en clair)
docker login http://registry.example.com

# ✅ Sécurisé
docker login https://registry.example.com
```

### 3. Scan de vulnérabilités

Beaucoup de registres proposent un scan automatique des images :

**AWS ECR** :
```bash
# Activer le scan automatique
aws ecr put-image-scanning-configuration \
  --repository-name mon-app \
  --image-scanning-configuration scanOnPush=true
```

**Harbor** : Scan automatique intégré dans l'interface web.

### 4. Contrôle d'accès basé sur les rôles (RBAC)

Définissez des rôles pour vos utilisateurs :

**Exemple dans Harbor** :
- **Admin** : Tout contrôle
- **Developer** : Push et pull
- **Guest** : Pull uniquement

### 5. Politiques de rétention

Supprimez automatiquement les anciennes images :
- Garder les 10 dernières versions
- Supprimer les images de plus de 90 jours
- Garder uniquement les images taguées

### 6. Audit et logs

Activez les logs pour tracer :
- Qui a poussé quelle image
- Qui a téléchargé quoi
- Tentatives d'accès non autorisées

## Registre privé dans Docker Compose

Vous pouvez utiliser des images de registres privés dans vos fichiers `docker-compose.yml` :

```yaml
version: '3.8'

services:
  app:
    image: ghcr.io/johndoe/mon-app:latest
    ports:
      - "8080:8080"

  api:
    image: registry.mon-entreprise.com/api:v1.2.0
    ports:
      - "3000:3000"
```

**Important** : Vous devez être connecté au registre avant de faire `docker compose up` :

```bash
docker login ghcr.io
docker login registry.mon-entreprise.com
docker compose up -d
```

## Registre privé en CI/CD

Les registres privés sont essentiels dans les pipelines CI/CD.

### Exemple avec GitHub Actions

```yaml
name: Build et Push

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Login to GitHub Container Registry
        uses: docker/login-action@v2
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build et Push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}
```

### Exemple avec GitLab CI

```yaml
build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
```

## Choisir le bon registre pour votre projet

### Vous êtes un développeur solo ou freelance ?

**Recommandation** : Docker Hub (1 repo privé gratuit) ou GitHub Container Registry

**Pourquoi** :
- Gratuit
- Facile à utiliser
- Pas de serveur à gérer

### Votre projet est sur GitHub ?

**Recommandation** : GitHub Container Registry (ghcr.io)

**Pourquoi** :
- Intégré à GitHub
- GitHub Actions simple
- Gratuit

### Votre projet est sur GitLab ?

**Recommandation** : GitLab Container Registry

**Pourquoi** :
- Inclus gratuitement
- CI/CD intégré
- Privé par défaut

### Vous déployez sur AWS ?

**Recommandation** : Amazon ECR

**Pourquoi** :
- Performance optimale sur AWS
- Intégration ECS/EKS
- Sécurité AWS

### Vous déployez sur Google Cloud ?

**Recommandation** : Google Artifact Registry

**Pourquoi** :
- Performance sur GCP
- Intégration GKE

### Vous déployez sur Azure ?

**Recommandation** : Azure Container Registry

**Pourquoi** :
- Intégration Azure
- Performance optimale

### Vous avez besoin de contrôle total (on-premise) ?

**Recommandation** : Harbor

**Pourquoi** :
- Open source
- Fonctionnalités enterprise
- Aucune dépendance cloud

### Vous débutez et voulez apprendre ?

**Recommandation** : Docker Registry (self-hosted simple)

**Pourquoi** :
- Comprendre les bases
- Gratuit
- Simple

## Coûts approximatifs

### Solutions gratuites
- ✅ Docker Hub : 1 repo privé
- ✅ GitHub Container Registry : Limites généreuses
- ✅ GitLab Container Registry : Illimité (self-hosted)
- ✅ Harbor : Gratuit (mais coût d'infrastructure)
- ✅ Docker Registry : Gratuit (mais coût d'infrastructure)

### Solutions payantes (ordre de grandeur)
- 💰 Docker Hub Pro : $5/mois
- 💰 AWS ECR : ~$0.10/GB stockage + transfert
- 💰 Google Artifact Registry : ~$0.10/GB stockage + transfert
- 💰 Azure ACR : Dès $5/mois (Basic)
- 💰 JFrog Artifactory : $$$$ (enterprise)

**Note** : Les prix varient selon l'usage. Consultez les sites officiels pour les tarifs actuels.

## Migration vers un registre privé

### Déplacer des images de Docker Hub vers un registre privé

```bash
# 1. Pull depuis Docker Hub
docker pull johndoe/mon-app:latest

# 2. Taguer pour le nouveau registre
docker tag johndoe/mon-app:latest ghcr.io/johndoe/mon-app:latest

# 3. Push vers le nouveau registre
docker login ghcr.io
docker push ghcr.io/johndoe/mon-app:latest

# 4. Mettre à jour vos déploiements
# Ancienne référence : johndoe/mon-app:latest
# Nouvelle référence : ghcr.io/johndoe/mon-app:latest
```

## Registres multiples

Vous pouvez utiliser plusieurs registres simultanément :

```yaml
# docker-compose.yml
version: '3.8'

services:
  frontend:
    image: ghcr.io/myorg/frontend:latest

  backend:
    image: registry.gitlab.com/myorg/backend:latest

  database:
    image: postgres:15  # Docker Hub public
```

## Points clés à retenir

✅ Un **registre privé** héberge vos images Docker de manière sécurisée avec contrôle d'accès

✅ Les registres privés sont essentiels pour le **code propriétaire** et les applications d'entreprise

✅ Plusieurs options existent : **Docker Hub privé**, **GitHub/GitLab**, **cloud providers** (AWS/GCP/Azure), **self-hosted**

✅ Choisissez votre registre selon vos besoins : **coût**, **intégration existante**, **performance**, **contrôle**

✅ L'authentification se fait via **docker login** avec credentials ou tokens

✅ Utilisez des **tokens d'accès** plutôt que des mots de passe pour la sécurité

✅ Les registres cloud (**ECR**, **GCR**, **ACR**) offrent la meilleure performance dans leur écosystème respectif

✅ **Harbor** est la meilleure solution open source self-hosted pour l'enterprise

✅ **GitHub Container Registry** est idéal pour les projets déjà sur GitHub (gratuit et simple)

✅ Activez le **scan de vulnérabilités** et les **politiques de rétention** pour la sécurité

✅ Les registres privés s'intègrent naturellement dans les **pipelines CI/CD**

---

**Vous avez maintenant une vue d'ensemble complète des registres privés ! Dans vos projets, n'hésitez pas à commencer avec une solution gratuite (GitHub ou GitLab) avant de migrer vers une solution plus robuste si nécessaire.** 🚀

⏭️ [Bonnes pratiques](/10-bonnes-pratiques/README.md)
