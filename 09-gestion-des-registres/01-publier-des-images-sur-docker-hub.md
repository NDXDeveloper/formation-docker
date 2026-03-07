🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 9.1 Publier des images sur Docker Hub

## Introduction

Vous avez créé une image Docker personnalisée et vous souhaitez la partager avec d'autres développeurs, la déployer sur un serveur, ou simplement la sauvegarder dans le cloud ? C'est exactement ce que permet Docker Hub !

Docker Hub est le registre d'images Docker officiel et gratuit, un peu comme GitHub pour le code source. C'est l'endroit où vous pouvez :
- **Publier** vos images Docker
- **Télécharger** des images créées par d'autres
- **Versionner** vos images avec des tags
- **Collaborer** avec votre équipe
- **Automatiser** vos workflows de build

Dans cette section, nous allons apprendre à publier vos propres images sur Docker Hub ! 🚀

## Qu'est-ce que Docker Hub ?

### Un registre d'images centralisé

Docker Hub est un service cloud qui héberge des images Docker. C'est le registre par défaut utilisé par Docker : quand vous tapez `docker pull nginx`, Docker télécharge automatiquement l'image depuis Docker Hub.

### Les chiffres de Docker Hub

- Plus de **100 millions d'images** téléchargées
- Des **millions d'images publiques** disponibles
- Des **images officielles** vérifiées (nginx, postgres, redis, etc.)
- **Gratuit** pour les images publiques
- Des **plans payants** pour les images privées et fonctionnalités avancées

### Registre public vs privé

**Images publiques** :
- Visibles et téléchargeables par tout le monde
- Idéales pour l'open source
- Gratuites et illimitées

**Images privées** :
- Visibles uniquement par vous et les personnes autorisées
- Idéales pour le code propriétaire
- Limitées dans le plan gratuit (1 repository privé)

## Pourquoi publier vos images sur Docker Hub ?

### 1. Partage facile
Partagez vos applications avec le monde entier ou avec votre équipe, sans avoir à envoyer des fichiers volumineux.

### 2. Déploiement simplifié
Déployez vos applications sur n'importe quel serveur en une commande : `docker pull votre-image`.

### 3. Versionning
Gérez différentes versions de votre application avec des tags (v1.0, v1.1, latest...).

### 4. Sauvegarde
Vos images sont sauvegardées dans le cloud, même si vous perdez votre machine locale.

### 5. CI/CD
Intégrez facilement Docker Hub dans vos pipelines d'intégration continue.

### 6. Collaboration
Travaillez en équipe sur les mêmes images.

## Prérequis

Avant de commencer, vous devez avoir :
- ✅ Docker installé sur votre machine
- ✅ Une image Docker que vous voulez publier (ou nous en créerons une ensemble)
- ✅ Une connexion Internet

## Étape 1 : Créer un compte Docker Hub

### Inscription gratuite

1. Allez sur [https://hub.docker.com](https://hub.docker.com)
2. Cliquez sur **Sign Up** (S'inscrire)
3. Remplissez le formulaire :
   - **Docker ID** : Votre nom d'utilisateur unique (exemple : `johndoe`)
   - **Email** : Votre adresse email
   - **Password** : Un mot de passe sécurisé
4. Vérifiez votre email
5. Connectez-vous à Docker Hub

**Votre Docker ID** est très important ! C'est lui que vous utiliserez pour nommer vos images : `docker-id/nom-image`

### Exemple
Si votre Docker ID est `johndoe` et que vous voulez publier une image appelée `mon-app`, le nom complet sera : `johndoe/mon-app`

## Étape 2 : Se connecter depuis le terminal

Avant de pouvoir pousser des images, vous devez vous authentifier depuis votre terminal.

### Commande docker login

```bash
docker login
```

**Ce qui se passe** :
```bash
$ docker login
Login with your Docker ID to push and pull images from Docker Hub.  
Username: johndoe  
Password:  
Login Succeeded  
```

Entrez votre Docker ID (pas votre email) et votre mot de passe.

### Connexion réussie

Une fois connecté, vous verrez le message **"Login Succeeded"**. Vos identifiants sont maintenant sauvegardés localement et vous pouvez pousser des images.

### Se déconnecter

Pour vous déconnecter :
```bash
docker logout
```

## Étape 3 : Préparer votre image

Pour publier une image, vous devez d'abord l'avoir localement. Créons une image simple pour l'exemple.

### Créer une application simple

**Fichier `index.html`** :
```html
<!DOCTYPE html>
<html>
<head>
    <title>Mon Application Docker</title>
</head>
<body>
    <h1>Bienvenue sur mon application ! 🚀</h1>
    <p>Cette application a été conteneurisée et publiée sur Docker Hub.</p>
</body>
</html>
```

**Fichier `Dockerfile`** :
```dockerfile
FROM nginx:alpine

# Copier notre page HTML
COPY index.html /usr/share/nginx/html/index.html

# Exposer le port 80
EXPOSE 80

# nginx démarre automatiquement (défini dans l'image de base)
```

### Construire l'image localement

```bash
docker build -t mon-app .
```

**Vérifier que l'image existe** :
```bash
docker images
```

Vous devriez voir votre image `mon-app` dans la liste.

### Tester l'image localement

Avant de publier, testons que l'image fonctionne :

```bash
docker run -d -p 8080:80 mon-app
```

Visitez `http://localhost:8080` dans votre navigateur. Vous devriez voir votre page HTML !

Une fois vérifié, arrêtez le conteneur :
```bash
docker stop $(docker ps -q --filter ancestor=mon-app)
```

## Étape 4 : Taguer votre image correctement

Pour publier une image sur Docker Hub, elle doit suivre le format de nommage :
```
docker-id/nom-image:tag
```

### Comprendre le format

- **docker-id** : Votre identifiant Docker Hub (exemple : `johndoe`)
- **nom-image** : Le nom de votre application (exemple : `mon-app`)
- **tag** : La version ou variante (exemple : `v1.0`, `latest`, `dev`)

### Exemples de noms valides

```
johndoe/mon-app:latest  
johndoe/mon-app:v1.0  
johndoe/mon-site-web:production  
johndoe/api-users:2.3.1  
```

### Taguer votre image

Si votre image s'appelle actuellement `mon-app` et que votre Docker ID est `johndoe` :

```bash
docker tag mon-app johndoe/mon-app:latest
```

**Syntaxe générale** :
```bash
docker tag nom-actuel docker-id/nouveau-nom:tag
```

### Vérifier les tags

```bash
docker images
```

Vous verrez maintenant deux entrées (qui pointent vers la même image) :
```
REPOSITORY           TAG       IMAGE ID       CREATED         SIZE  
mon-app              latest    abc123def456   5 minutes ago   25MB  
johndoe/mon-app      latest    abc123def456   5 minutes ago   25MB  
```

**Note** : Elles ont le même IMAGE ID car c'est la même image, juste avec deux noms différents.

### Tags multiples

Vous pouvez créer plusieurs tags pour la même image :

```bash
# Tag latest (dernière version)
docker tag mon-app johndoe/mon-app:latest

# Tag avec numéro de version
docker tag mon-app johndoe/mon-app:1.0.0

# Tag pour l'environnement
docker tag mon-app johndoe/mon-app:prod
```

## Étape 5 : Pousser l'image sur Docker Hub

Maintenant que votre image est correctement taguée, vous pouvez la pousser sur Docker Hub !

### Commande docker push

```bash
docker push johndoe/mon-app:latest
```

**Ce qui se passe** :
```bash
$ docker push johndoe/mon-app:latest
The push refers to repository [docker.io/johndoe/mon-app]
5f70bf18a086: Pushed
e2eb06d8af82: Pushed  
latest: digest: sha256:abc123... size: 1234  
```

Docker envoie les layers de votre image sur Docker Hub. Les layers déjà présents (comme ceux de nginx:alpine) ne sont pas re-uploadés.

### Pousser plusieurs tags

Pour pousser tous les tags d'une image :

```bash
docker push johndoe/mon-app:latest  
docker push johndoe/mon-app:1.0.0  
docker push johndoe/mon-app:prod  
```

Ou en une seule commande (pousse tous les tags) :

```bash
docker push johndoe/mon-app --all-tags
```

### Temps de push

Le premier push peut prendre du temps selon :
- La taille de votre image
- Votre connexion Internet
- Les layers déjà présents sur Docker Hub

Les pushs suivants sont beaucoup plus rapides car seuls les layers modifiés sont envoyés.

## Étape 6 : Vérifier sur Docker Hub

### Accéder à votre repository

1. Connectez-vous sur [https://hub.docker.com](https://hub.docker.com)
2. Allez dans **Repositories**
3. Vous devriez voir votre image : `johndoe/mon-app`

### Informations visibles

Sur la page de votre repository, vous verrez :
- Le nom complet de l'image
- La description (que vous pouvez modifier)
- Les tags disponibles (latest, 1.0.0, etc.)
- La date de dernière mise à jour
- Le nombre de pulls (téléchargements)
- Les statistiques d'utilisation

### Ajouter une description

1. Cliquez sur votre repository
2. Allez dans l'onglet **Description**
3. Cliquez sur **Edit**
4. Ajoutez une description en Markdown :

````markdown
# Mon Application Docker

Cette application est un exemple de site web conteneurisé avec Nginx.

## Utilisation

```bash
docker run -d -p 8080:80 johndoe/mon-app:latest
```

Puis visitez http://localhost:8080

## Tags disponibles

- `latest` : Dernière version stable
- `1.0.0` : Version 1.0.0
- `dev` : Version de développement
````

Cette description apparaîtra sur Docker Hub et aidera les autres développeurs à utiliser votre image.

## Étape 7 : Télécharger et utiliser votre image

Maintenant que votre image est sur Docker Hub, n'importe qui peut la télécharger !

### Sur une autre machine

Sur n'importe quelle machine avec Docker installé :

```bash
# Télécharger l'image
docker pull johndoe/mon-app:latest

# Lancer un conteneur
docker run -d -p 8080:80 johndoe/mon-app:latest
```

**C'est tout !** Votre application fonctionne sans avoir à reconstruire l'image.

### Depuis Docker Compose

Vous pouvez aussi utiliser votre image dans un `docker-compose.yml` :

```yaml
services:
  web:
    image: johndoe/mon-app:latest
    ports:
      - "8080:80"
```

Puis :
```bash
docker compose up -d
```

Docker téléchargera automatiquement votre image depuis Docker Hub.

## Gestion des images sur Docker Hub

### Voir toutes vos images

Sur Docker Hub, dans la section **Repositories**, vous verrez la liste de toutes vos images publiées.

### Gérer les tags

Pour chaque repository, vous pouvez :
- Voir tous les tags disponibles
- Supprimer des tags obsolètes
- Voir la taille de chaque tag

### Supprimer un tag

1. Allez dans votre repository
2. Cliquez sur l'onglet **Tags**
3. Sélectionnez le(s) tag(s) à supprimer
4. Cliquez sur **Delete**

**⚠️ Attention** : Cette action est irréversible !

### Supprimer un repository entier

1. Allez dans **Settings** de votre repository
2. Scrollez jusqu'en bas
3. Cliquez sur **Delete repository**
4. Confirmez en tapant le nom du repository

**⚠️ DANGER** : Tous les tags seront supprimés !

## Images publiques vs images privées

### Créer un repository public (par défaut)

Les images publiques sont :
- ✅ Gratuites et illimitées
- ✅ Visibles par tout le monde
- ✅ Téléchargeables par tout le monde
- ✅ Idéales pour l'open source

Quand vous poussez une image, elle est publique par défaut.

### Créer un repository privé

Les images privées sont :
- 🔒 Visibles uniquement par vous et les personnes autorisées
- 🔒 Téléchargeables uniquement après authentification
- 💰 Limitées dans le plan gratuit (1 repository privé)
- 💰 Illimitées avec un plan payant

Pour créer un repository privé :

1. Sur Docker Hub, cliquez sur **Create Repository**
2. Donnez un nom à votre repository
3. Cochez **Private**
4. Cliquez sur **Create**

Puis poussez votre image normalement :
```bash
docker push johndoe/mon-app-privee:latest
```

### Qui peut voir mes images privées ?

Par défaut, uniquement vous. Pour autoriser d'autres personnes :

1. Allez dans **Settings** → **Collaborators**
2. Ajoutez des utilisateurs Docker Hub
3. Ils pourront pull votre image après `docker login`

## Workflow complet de publication

Voici le processus complet du développement à la publication :

### 1. Développer localement

```bash
# Créer votre Dockerfile et votre application
nano Dockerfile  
nano index.html  
```

### 2. Construire l'image

```bash
docker build -t mon-app .
```

### 3. Tester localement

```bash
docker run -d -p 8080:80 mon-app
# Tester dans le navigateur
docker stop <container-id>
```

### 4. Se connecter à Docker Hub

```bash
docker login
```

### 5. Taguer l'image

```bash
docker tag mon-app johndoe/mon-app:1.0.0  
docker tag mon-app johndoe/mon-app:latest  
```

### 6. Pousser sur Docker Hub

```bash
docker push johndoe/mon-app:1.0.0  
docker push johndoe/mon-app:latest  
```

### 7. Utiliser l'image ailleurs

```bash
# Sur un serveur de production
docker pull johndoe/mon-app:latest  
docker run -d -p 80:80 johndoe/mon-app:latest  
```

## Bonnes pratiques pour les tags

### 1. Utilisez des versions sémantiques

```bash
# ✅ Bon : versions explicites
docker push johndoe/mon-app:1.0.0  
docker push johndoe/mon-app:1.0.1  
docker push johndoe/mon-app:2.0.0  

# ❌ Moins clair
docker push johndoe/mon-app:v1  
docker push johndoe/mon-app:version2  
```

### 2. Maintenez un tag "latest"

```bash
# Toujours pousser latest avec la dernière version stable
docker tag mon-app johndoe/mon-app:2.0.0  
docker tag mon-app johndoe/mon-app:latest  
docker push johndoe/mon-app:2.0.0  
docker push johndoe/mon-app:latest  
```

Le tag `latest` doit pointer vers la dernière version stable.

### 3. Utilisez des tags d'environnement

```bash
# Pour différents environnements
johndoe/mon-app:dev  
johndoe/mon-app:staging  
johndoe/mon-app:prod  
```

### 4. Incluez le SHA Git si pertinent

```bash
# Lier à un commit spécifique
johndoe/mon-app:1.0.0-abc123
```

### 5. Ne supprimez jamais un tag en production

Une fois qu'un tag est utilisé en production, ne le supprimez pas et ne le réécrivez pas. Créez un nouveau tag à la place.

```bash
# ❌ Mauvais : réécrire un tag existant
docker push johndoe/mon-app:1.0.0  # contenu A
# ... plus tard ...
docker push johndoe/mon-app:1.0.0  # contenu B différent !

# ✅ Bon : créer un nouveau tag
docker push johndoe/mon-app:1.0.0  
docker push johndoe/mon-app:1.0.1  # nouvelle version  
```

## Automatisation avec GitHub Actions

Vous pouvez automatiser la publication sur Docker Hub avec GitHub Actions.

**Fichier `.github/workflows/docker-publish.yml`** :
```yaml
name: Publier sur Docker Hub

on:
  push:
    branches: [ main ]
    tags: [ 'v*' ]

jobs:
  push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Se connecter à Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build et Push
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: |
            johndoe/mon-app:latest
            johndoe/mon-app:${{ github.sha }}
```

À chaque push sur `main`, votre image est automatiquement construite et publiée !

## Sécurité et tokens d'accès

### Ne partagez jamais votre mot de passe

Pour les scripts ou la CI/CD, utilisez des **Access Tokens** au lieu de votre mot de passe.

### Créer un Access Token

1. Sur Docker Hub, allez dans **Account Settings** → **Security**
2. Cliquez sur **New Access Token**
3. Donnez un nom descriptif (ex: "CI/CD GitHub Actions")
4. Définissez les permissions (Read, Write, Delete)
5. Cliquez sur **Generate**
6. **Copiez le token** (vous ne pourrez plus le voir !)

### Utiliser le token

```bash
# Se connecter avec un token
docker login -u johndoe -p <votre-token>
```

Ou dans GitHub Actions, ajoutez-le dans les secrets du repository.

### Révoquer un token

Si un token est compromis, révoquez-le immédiatement depuis Docker Hub → Security.

## Limites et quotas

### Plan gratuit

- ✅ Images publiques illimitées
- ✅ 1 repository privé
- ✅ Téléchargements illimités pour vos propres images
- ⚠️ Rate limits sur les pulls d'images publiques (100 pulls / 6h pour les utilisateurs anonymes, 200 pulls / 6h pour les utilisateurs connectés)

### Plans payants

Pour plus de repositories privés et sans rate limits, consultez les [plans Docker Hub](https://www.docker.com/pricing).

## Alternatives à Docker Hub

Docker Hub n'est pas le seul registre d'images Docker :

### GitHub Container Registry (ghcr.io)
- Intégré à GitHub
- Gratuit pour les images publiques
- Excellente intégration avec GitHub Actions

### GitLab Container Registry
- Inclus avec GitLab
- Privé par défaut
- Intégré dans les pipelines GitLab CI

### Amazon ECR
- Service AWS
- Hautement évolutif
- Idéal pour les déploiements AWS

### Google Artifact Registry
- Service Google Cloud
- Intégration avec GKE
- Performant

### Azure Container Registry
- Service Microsoft Azure
- Intégration avec AKS
- Geo-réplication

**Docker Hub reste néanmoins le registre le plus populaire et le plus simple pour débuter ! 🎯**

## Dépannage des problèmes courants

### "denied: requested access to the resource is denied"

**Cause** : Vous n'êtes pas connecté ou vous n'avez pas les permissions.

**Solution** :
```bash
docker logout  
docker login  
# Vérifiez votre Docker ID et mot de passe
```

### "tag does not exist"

**Cause** : Le tag n'est pas correctement formaté.

**Solution** : Vérifiez le format `docker-id/image:tag`
```bash
# ❌ Incorrect
docker push mon-app:latest

# ✅ Correct
docker push johndoe/mon-app:latest
```

### "no basic auth credentials"

**Cause** : Vous devez vous connecter d'abord.

**Solution** :
```bash
docker login
```

### "unauthorized: incorrect username or password"

**Cause** : Identifiants incorrects.

**Solution** :
- Vérifiez que vous utilisez votre **Docker ID** (pas votre email)
- Vérifiez votre mot de passe
- Si vous utilisez un token, vérifiez qu'il est encore valide

### Problèmes de rate limit

**Cause** : Trop de pulls en peu de temps.

**Solution** :
- Connectez-vous : `docker login` (double la limite)
- Attendez quelques heures
- Considérez un plan payant pour les usages intensifs

## Points clés à retenir

✅ **Docker Hub** est le registre d'images Docker officiel et gratuit

✅ Créez un compte sur hub.docker.com et connectez-vous avec `docker login`

✅ Les images doivent être taguées au format `docker-id/nom-image:tag`

✅ Utilisez `docker tag` pour renommer vos images correctement

✅ Utilisez `docker push` pour publier vos images sur Docker Hub

✅ Les images **publiques** sont gratuites et illimitées

✅ Les images **privées** sont limitées dans le plan gratuit (1 repository)

✅ Utilisez des **versions sémantiques** pour vos tags (1.0.0, 1.1.0, 2.0.0)

✅ Maintenez toujours un tag **latest** pointant vers la version stable

✅ Utilisez des **Access Tokens** pour la CI/CD plutôt que votre mot de passe

✅ Documentez vos images avec une bonne **description** sur Docker Hub

---

**Prochaine section** : Nous allons explorer la gestion des tags et versions pour organiser efficacement vos images Docker !

⏭️ [Gestion des tags et versions](/09-gestion-des-registres/02-gestion-des-tags-et-versions.md)
