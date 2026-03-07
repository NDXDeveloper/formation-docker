🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 9.2 Gestion des tags et versions

## Introduction

Les tags sont un élément fondamental de Docker. Ils permettent de gérer différentes versions de vos images, de déployer la bonne version au bon endroit, et de maintenir un historique clair de vos modifications.

Une bonne gestion des tags est essentielle pour :
- 🎯 **Déployer** la bonne version en production
- 🔄 **Revenir** à une version antérieure en cas de problème
- 🧪 **Tester** différentes versions en parallèle
- 📊 **Suivre** l'évolution de votre application
- 🤝 **Communiquer** clairement avec votre équipe

Dans cette section, nous allons explorer les stratégies de tagging, les bonnes pratiques, et comment gérer efficacement les versions de vos images Docker. 🚀

## Qu'est-ce qu'un tag Docker ?

### Définition simple

Un tag est un **label** attaché à une image Docker. C'est comme une étiquette qui identifie une version spécifique de votre image.

### Anatomie d'un nom d'image complet

```
registry/namespace/repository:tag
```

**Exemples** :
```
docker.io/library/nginx:1.27.3  
docker.io/johndoe/mon-app:v1.0.0  
ghcr.io/myorg/api:latest  
```

**Décortiquons** :
- **registry** : Le registre (docker.io par défaut, peut être omis)
- **namespace** : Utilisateur ou organisation (library pour les images officielles)
- **repository** : Nom de l'image
- **tag** : La version ou variante

### Format simplifié pour Docker Hub

Sur Docker Hub, le format simplifié est :
```
username/image:tag
```

**Exemples** :
```
johndoe/mon-app:latest  
johndoe/mon-app:1.0.0  
postgres:15  
nginx:alpine  
```

### Le tag par défaut : "latest"

Si vous ne spécifiez pas de tag, Docker utilise automatiquement `latest` :

```bash
# Ces deux commandes sont équivalentes
docker pull nginx  
docker pull nginx:latest  

# Ces deux commandes aussi
docker run nginx  
docker run nginx:latest  
```

**⚠️ Important** : `latest` ne signifie pas toujours "la toute dernière version" ! C'est juste un tag par défaut. C'est à vous de le maintenir à jour.

## Pourquoi les tags sont-ils importants ?

### 1. Reproductibilité

Sans tags spécifiques, votre déploiement peut changer sans que vous le sachiez :

```bash
# ❌ Dangereux en production
docker pull mon-app:latest
# Vous ne savez pas quelle version vous obtenez !

# ✅ Sûr et reproductible
docker pull mon-app:1.2.5
# Vous obtenez toujours exactement la même version
```

### 2. Déploiements contrôlés

Les tags permettent de déployer différentes versions dans différents environnements :

```bash
# Développement : dernière version
docker run mon-app:dev

# Staging : version candidate
docker run mon-app:1.3.0-rc1

# Production : version stable
docker run mon-app:1.2.5
```

### 3. Rollback facile

En cas de problème, vous pouvez revenir rapidement à une version antérieure :

```bash
# Version actuelle pose problème
docker stop mon-app-prod  
docker rm mon-app-prod  

# Retour à la version précédente
docker run -d --name mon-app-prod mon-app:1.2.4
```

### 4. Tests A/B

Vous pouvez tester plusieurs versions simultanément :

```bash
# Version actuelle
docker run -p 8080:80 mon-app:1.0.0

# Nouvelle version en test
docker run -p 8081:80 mon-app:1.1.0
```

## Stratégies de tagging

Il existe plusieurs stratégies pour taguer vos images. Voyons les plus courantes.

### 1. Versionnement sémantique (Recommandé)

Le versionnement sémantique (SemVer) utilise le format `MAJEUR.MINEUR.PATCH` :

```
MAJEUR.MINEUR.PATCH
```

**Règles** :
- **MAJEUR** : Changements incompatibles avec les versions précédentes
- **MINEUR** : Nouvelles fonctionnalités compatibles
- **PATCH** : Corrections de bugs compatibles

**Exemples** :
```
mon-app:1.0.0    # Première version stable  
mon-app:1.0.1    # Correction de bug  
mon-app:1.1.0    # Nouvelle fonctionnalité  
mon-app:2.0.0    # Changement majeur (breaking change)  
```

**Illustration** :
```
1.0.0 → 1.0.1 → 1.0.2 (Patches)
  ↓
1.1.0 → 1.1.1 (Nouvelle fonctionnalité + patch)
  ↓
2.0.0 (Changement majeur)
```

### 2. Tags avec préfixe "v"

Certains préfèrent ajouter un "v" devant :

```
mon-app:v1.0.0  
mon-app:v1.0.1  
mon-app:v2.0.0  
```

**Les deux approches sont valides**, choisissez celle que vous préférez et restez cohérent.

### 3. Tags multiples pour la même image

Une bonne pratique consiste à créer plusieurs tags pour la même version :

```bash
# Image avec plusieurs tags
docker tag mon-app johndoe/mon-app:1.2.5  
docker tag mon-app johndoe/mon-app:1.2  
docker tag mon-app johndoe/mon-app:1  
docker tag mon-app johndoe/mon-app:latest  

# Pousser tous les tags
docker push johndoe/mon-app:1.2.5  
docker push johndoe/mon-app:1.2  
docker push johndoe/mon-app:1  
docker push johndoe/mon-app:latest  
```

**Avantage** : Les utilisateurs peuvent choisir leur niveau de précision :
- `1.2.5` : Version exacte (production)
- `1.2` : Dernière patch de la version 1.2 (staging)
- `1` : Dernière version mineure de la version 1 (dev)
- `latest` : Toute dernière version (tests)

### 4. Tags d'environnement

Pour différencier les builds selon l'environnement :

```
mon-app:dev  
mon-app:staging  
mon-app:prod  
```

Ou combinés avec la version :

```
mon-app:1.2.5-dev  
mon-app:1.2.5-staging  
mon-app:1.2.5-prod  
```

### 5. Tags de release candidate

Pour les versions en cours de test :

```
mon-app:1.3.0-rc1      # Release Candidate 1  
mon-app:1.3.0-rc2      # Release Candidate 2  
mon-app:1.3.0          # Version finale  
```

### 6. Tags avec SHA Git

Pour lier directement à un commit :

```bash
# Récupérer le SHA du commit
GIT_SHA=$(git rev-parse --short HEAD)

# Créer un tag avec le SHA
docker tag mon-app johndoe/mon-app:${GIT_SHA}  
docker tag mon-app johndoe/mon-app:1.2.5-${GIT_SHA}  
```

**Résultat** :
```
mon-app:abc1234  
mon-app:1.2.5-abc1234  
```

**Avantage** : Traçabilité parfaite entre votre code et votre image Docker.

### 7. Tags avec horodatage

Pour les builds nocturnes ou de développement :

```bash
# Format : YYYYMMDD
docker tag mon-app johndoe/mon-app:20251022

# Format : YYYYMMDD-HHMM
docker tag mon-app johndoe/mon-app:20251022-1430

# Combiné avec version
docker tag mon-app johndoe/mon-app:1.2.5-20251022
```

### 8. Tags de variantes

Pour différentes configurations de la même version :

```
nginx:1.27.3-alpine       # Version légère avec Alpine Linux  
nginx:1.27.3-perl         # Avec support Perl  
python:3.11-slim          # Version minimale  
python:3.11-alpine        # Avec Alpine  
python:3.11-bullseye      # Avec Debian Bullseye  
```

## Stratégie de tagging complète : Exemple réel

Voici une stratégie complète pour un projet réel :

```bash
# Build de l'image
docker build -t mon-app .

# Version complète (production)
docker tag mon-app johndoe/mon-app:1.2.5

# Version majeure.mineure (staging)
docker tag mon-app johndoe/mon-app:1.2

# Version majeure (développement)
docker tag mon-app johndoe/mon-app:1

# Latest (dernière stable)
docker tag mon-app johndoe/mon-app:latest

# SHA Git (traçabilité)
docker tag mon-app johndoe/mon-app:1.2.5-abc1234

# Horodatage (archivage)
docker tag mon-app johndoe/mon-app:1.2.5-20251022

# Pousser tous les tags
docker push johndoe/mon-app --all-tags
```

## Le tag "latest" : Mythes et réalités

### Mythe 1 : "latest" est toujours la dernière version

**Faux !** `latest` est juste un tag comme un autre. Si vous ne le mettez pas à jour, il pointera vers une vieille version.

```bash
# Vous publiez la version 1.0.0
docker push johndoe/mon-app:1.0.0  
docker push johndoe/mon-app:latest  # latest = 1.0.0  

# Vous publiez la version 2.0.0
docker push johndoe/mon-app:2.0.0
# Mais vous oubliez de mettre à jour latest !

# latest pointe toujours vers 1.0.0 !
```

### Mythe 2 : Il ne faut jamais utiliser "latest" en production

**Nuancé !** Il est vrai que `latest` peut être imprévisible, mais il a son utilité :

**❌ Évitez `latest` pour** :
- Les déploiements de production
- Les environnements critiques
- Quand vous avez besoin de reproductibilité

**✅ Utilisez `latest` pour** :
- Le développement local rapide
- Les tests automatisés (avec prudence)
- Les démos et prototypes
- Comme raccourci pour "dernière version stable"

### Bonne pratique avec "latest"

Maintenez `latest` comme pointeur vers la dernière version **stable** :

```bash
# Nouvelle version stable publiée
docker push johndoe/mon-app:1.2.5  
docker push johndoe/mon-app:latest  # Met à jour latest  

# Version candidate (pas stable)
docker push johndoe/mon-app:1.3.0-rc1
# Ne pas mettre à jour latest !
```

## Gestion pratique des tags

### Créer plusieurs tags d'un coup

```bash
# Variables
VERSION="1.2.5"  
MAJOR="1"  
MINOR="1.2"  

# Créer tous les tags
docker tag mon-app johndoe/mon-app:${VERSION}  
docker tag mon-app johndoe/mon-app:${MINOR}  
docker tag mon-app johndoe/mon-app:${MAJOR}  
docker tag mon-app johndoe/mon-app:latest  
```

### Script de tagging automatisé

**Fichier `tag-and-push.sh`** :
```bash
#!/bin/bash

IMAGE_NAME="johndoe/mon-app"  
VERSION=$1  

if [ -z "$VERSION" ]; then
    echo "Usage: ./tag-and-push.sh VERSION"
    echo "Exemple: ./tag-and-push.sh 1.2.5"
    exit 1
fi

# Extraire majeur et mineur
MAJOR=$(echo $VERSION | cut -d. -f1)  
MINOR=$(echo $VERSION | cut -d. -f1,2)  

# Taguer
echo "Tagging ${IMAGE_NAME}:${VERSION}..."  
docker tag mon-app ${IMAGE_NAME}:${VERSION}  
docker tag mon-app ${IMAGE_NAME}:${MINOR}  
docker tag mon-app ${IMAGE_NAME}:${MAJOR}  
docker tag mon-app ${IMAGE_NAME}:latest  

# Pousser
echo "Pushing tags to Docker Hub..."  
docker push ${IMAGE_NAME}:${VERSION}  
docker push ${IMAGE_NAME}:${MINOR}  
docker push ${IMAGE_NAME}:${MAJOR}  
docker push ${IMAGE_NAME}:latest  

echo "✅ Done!"
```

**Utilisation** :
```bash
chmod +x tag-and-push.sh
./tag-and-push.sh 1.2.5
```

### Lister tous les tags d'une image locale

```bash
# Voir tous les tags d'une image
docker images johndoe/mon-app

# Format personnalisé
docker images --format "table {{.Repository}}:{{.Tag}}\t{{.ID}}\t{{.Size}}" johndoe/mon-app
```

### Supprimer un tag local

```bash
# Supprimer un tag spécifique
docker rmi johndoe/mon-app:old-version

# Supprimer tous les tags d'une image (garde l'image si d'autres tags existent)
docker rmi johndoe/mon-app:tag1 johndoe/mon-app:tag2
```

## Gestion des tags sur Docker Hub

### Voir tous les tags

Sur Docker Hub :
1. Allez dans votre repository
2. Cliquez sur l'onglet **Tags**
3. Vous verrez tous les tags publiés avec leurs détails

### Informations disponibles

Pour chaque tag, vous pouvez voir :
- La date de publication
- La taille du tag
- Le digest (hash unique)
- Le statut (actif, obsolète)
- Les layers

### Supprimer des tags obsolètes

**Sur Docker Hub** :
1. Allez dans **Tags**
2. Cochez les tags à supprimer
3. Cliquez sur **Delete**
4. Confirmez

**Via l'API Docker Hub** (avancé) :
```bash
# Nécessite un token d'authentification
curl -X DELETE \
  -H "Authorization: Bearer $TOKEN" \
  https://hub.docker.com/v2/repositories/johndoe/mon-app/tags/old-tag/
```

### Nettoyer les anciens tags automatiquement

Vous pouvez mettre en place une stratégie de rétention :
- Garder les 10 dernières versions
- Supprimer les tags de plus de 6 mois
- Garder uniquement les versions majeures anciennes

Malheureusement, Docker Hub gratuit ne propose pas de nettoyage automatique. Vous devrez utiliser un script ou un service tiers.

## Bonnes pratiques de versionnement

### ✅ À faire

#### 1. Utilisez le versionnement sémantique

```
✅ 1.0.0, 1.0.1, 1.1.0, 2.0.0
❌ version1, new, old, final
```

#### 2. Maintenez plusieurs niveaux de tags

```bash
docker push johndoe/mon-app:1.2.5  # Précis  
docker push johndoe/mon-app:1.2    # Mineur  
docker push johndoe/mon-app:1      # Majeur  
docker push johndoe/mon-app:latest # Dernier  
```

#### 3. Documentez votre stratégie de tagging

Ajoutez un fichier `VERSIONING.md` dans votre projet :

```markdown
# Stratégie de versionnement

## Tags disponibles

- `1.2.5` : Version exacte (production)
- `1.2` : Dernière patch de 1.2.x (staging)
- `1` : Dernière version 1.x.x (développement)
- `latest` : Dernière version stable

## Versionnement sémantique

- MAJEUR : Changements incompatibles
- MINEUR : Nouvelles fonctionnalités compatibles
- PATCH : Corrections de bugs
```

#### 4. Ne réécrivez jamais un tag en production

```bash
# ❌ Dangereux : réécrire un tag
docker push johndoe/mon-app:1.0.0  # contenu A
# quelques jours plus tard...
docker push johndoe/mon-app:1.0.0  # contenu B différent !

# ✅ Créez un nouveau tag
docker push johndoe/mon-app:1.0.1
```

#### 5. Incluez des métadonnées dans vos images

Utilisez les labels dans votre Dockerfile :

```dockerfile
FROM node:18

LABEL version="1.2.5"  
LABEL build-date="2025-10-22"  
LABEL git-commit="abc1234"  
LABEL maintainer="johndoe@example.com"  

COPY . /app  
WORKDIR /app  
CMD ["node", "server.js"]  
```

### ❌ À éviter

#### 1. Évitez les tags ambigus

```bash
❌ mon-app:new
❌ mon-app:old
❌ mon-app:final
❌ mon-app:final-final
❌ mon-app:prod-v2
```

#### 2. N'utilisez pas de caractères spéciaux

```bash
❌ mon-app:v1.0.0@special
❌ mon-app:version#1
✅ mon-app:1.0.0
✅ mon-app:v1.0.0
```

#### 3. Ne mélangez pas les stratégies

```bash
❌ Incohérent :
mon-app:1.0.0  
mon-app:version-2  
mon-app:v3  
mon-app:latest-stable  

✅ Cohérent :
mon-app:1.0.0  
mon-app:2.0.0  
mon-app:3.0.0  
mon-app:latest  
```

## Exemples de stratégies par type de projet

### Projet open source

```
mon-lib:1.0.0  
mon-lib:1.0.1  
mon-lib:1.1.0  
mon-lib:2.0.0  
mon-lib:latest  
```

Simple et clair pour les utilisateurs.

### Application web en microservices

```
api-users:1.2.5-abc1234  
api-products:3.1.2-def5678  
frontend:2.0.0-ghi9012  
```

Avec SHA Git pour la traçabilité.

### Application avec variants

```
mon-app:1.2.5-alpine  
mon-app:1.2.5-debian  
mon-app:1.2.5-ubuntu  
mon-app:latest-alpine  
```

Pour différentes images de base.

### Application avec environnements

```
mon-app:1.2.5-dev  
mon-app:1.2.5-staging  
mon-app:1.2.5-prod  
```

Tags spécifiques par environnement.

## Versionner avec Git et Docker ensemble

### Synchroniser les versions Git et Docker

```bash
# Dans Git
git tag v1.2.5  
git push origin v1.2.5  

# Dans Docker (même version)
docker tag mon-app johndoe/mon-app:1.2.5  
docker push johndoe/mon-app:1.2.5  
```

### Pipeline automatisé

**GitHub Actions** :
```yaml
name: Build et Push

on:
  push:
    tags:
      - 'v*'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Extraire la version
        id: version
        run: echo "VERSION=${GITHUB_REF#refs/tags/v}" >> $GITHUB_OUTPUT

      - name: Build
        run: docker build -t mon-app .

      - name: Login Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Tag et Push
        run: |
          VERSION=${{ steps.version.outputs.VERSION }}
          MAJOR=$(echo $VERSION | cut -d. -f1)
          MINOR=$(echo $VERSION | cut -d. -f1,2)

          docker tag mon-app johndoe/mon-app:$VERSION
          docker tag mon-app johndoe/mon-app:$MINOR
          docker tag mon-app johndoe/mon-app:$MAJOR
          docker tag mon-app johndoe/mon-app:latest

          docker push johndoe/mon-app --all-tags
```

Quand vous créez un tag Git `v1.2.5`, l'image Docker `1.2.5` est automatiquement construite et publiée !

## Audit et traçabilité

### Voir l'historique d'une image

```bash
# Voir les layers d'une image
docker history johndoe/mon-app:1.2.5

# Voir les métadonnées
docker inspect johndoe/mon-app:1.2.5

# Voir les labels
docker inspect johndoe/mon-app:1.2.5 --format='{{json .Config.Labels}}' | jq
```

### Comparer deux versions

```bash
# Comparer les tailles
docker images johndoe/mon-app

# Comparer les layers
docker history johndoe/mon-app:1.2.4  
docker history johndoe/mon-app:1.2.5  
```

### Tracer une image jusqu'au code source

Avec des labels appropriés :

```dockerfile
LABEL git-commit="abc1234"  
LABEL git-branch="main"  
LABEL build-date="2025-10-22T10:30:00Z"  
```

Vous pouvez retrouver exactement quel code a été utilisé pour construire l'image.

## Migration de stratégie de tagging

### Si vous voulez changer votre stratégie

**Ne pas** :
- ❌ Supprimer tous les anciens tags d'un coup
- ❌ Réécrire les tags existants

**À faire** :
- ✅ Commencer la nouvelle stratégie pour les nouvelles versions
- ✅ Garder les anciens tags pour la compatibilité
- ✅ Documenter le changement
- ✅ Donner un préavis aux utilisateurs

**Exemple** :
```
Anciennes versions :  
mon-app:v1.0  
mon-app:v1.1  

Nouvelles versions (nouvelle stratégie):  
mon-app:2.0.0  
mon-app:2.0.1  
```

## Points clés à retenir

✅ Les **tags** permettent d'identifier différentes versions de vos images

✅ Le **versionnement sémantique** (MAJEUR.MINEUR.PATCH) est la stratégie recommandée

✅ `latest` ne signifie pas automatiquement "la plus récente" - vous devez le maintenir

✅ Créez **plusieurs tags** pour la même image (1.2.5, 1.2, 1, latest)

✅ **Ne réécrivez jamais** un tag utilisé en production

✅ Utilisez des tags **spécifiques** en production, pas `latest`

✅ Documentez votre **stratégie de tagging** pour votre équipe

✅ Incluez des **métadonnées** (labels) dans vos images pour la traçabilité

✅ Synchronisez vos tags **Git et Docker** pour une meilleure cohérence

✅ **Nettoyez régulièrement** les tags obsolètes sur Docker Hub

✅ Utilisez des **outils d'automatisation** (GitHub Actions, GitLab CI) pour le tagging

---

**Prochaine section** : Nous allons explorer les registres privés pour héberger vos propres images en toute sécurité !

⏭️ [Introduction aux registres privés](/09-gestion-des-registres/03-introduction-aux-registres-prives.md)
