🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 5. Création d'images personnalisées

## Introduction

Jusqu'à présent, vous avez utilisé des images Docker **préexistantes** téléchargées depuis Docker Hub : nginx, ubuntu, python, mysql, etc. Ces images sont excellentes pour démarrer rapidement, mais dans un contexte réel, vous aurez besoin de créer vos **propres images personnalisées** qui contiennent votre application, vos configurations spécifiques et vos dépendances exactes.

Cette section marque un **tournant important** dans votre apprentissage de Docker : vous allez passer du rôle de **consommateur** d'images à celui de **créateur** d'images. C'est ici que Docker devient vraiment puissant et adapté à vos besoins spécifiques.

## Pourquoi créer ses propres images ?

### 1. Conteneuriser vos propres applications

Les images publiques sont génériques. Votre application a des besoins uniques :
- Votre code source spécifique
- Vos fichiers de configuration
- Vos dépendances précises (versions exactes de bibliothèques)
- Votre environnement d'exécution particulier

**Exemple** : Vous développez une application e-commerce en Python avec Flask. Aucune image sur Docker Hub ne contient votre code ! Vous devez créer une image personnalisée qui :
- Part d'une image Python de base
- Installe Flask et vos dépendances
- Copie votre code
- Configure l'application

### 2. Reproductibilité parfaite

Une image personnalisée garantit que votre application fonctionnera **exactement de la même manière** sur :
- Votre machine de développement
- Les machines de vos collègues
- Les serveurs de test
- Les serveurs de production
- Dans 6 mois, 1 an, 5 ans...

**Le problème classique sans Docker** :
```
Développeur 1: "Ça marche sur ma machine !"  
Développeur 2: "Chez moi ça ne fonctionne pas..."  
Admin système: "En production, ça plante..."  
```

**Avec Docker** :
```
Tous: "On utilise la même image, donc ça fonctionne partout de la même façon !"
```

### 3. Contrôle total

Créer vos propres images vous donne un **contrôle complet** sur :
- Les versions exactes de tous les composants
- Les optimisations de taille et de performance
- Les configurations de sécurité
- L'ajout ou le retrait d'outils spécifiques

### 4. Portabilité et distribution

Une fois votre image créée, vous pouvez :
- La partager avec votre équipe
- La déployer sur n'importe quel serveur avec Docker
- La publier sur Docker Hub ou un registre privé
- La versionner comme du code

### 5. Automatisation et intégration continue

Les images personnalisées s'intègrent parfaitement dans vos pipelines CI/CD :
```
Code modifié → Tests → Build image → Tests sur l'image → Déploiement
```

## Les bénéfices concrets

### Pour les développeurs

✅ **Environnement de développement instantané**
- Un nouveau développeur peut démarrer en quelques minutes
- Plus besoin d'installer manuellement Python, Node.js, Java, etc.
- Configuration identique pour toute l'équipe

✅ **Développement local identique à la production**
- Fini le "ça marche chez moi mais pas en prod"
- Tests en conditions réelles sur votre machine
- Débug facilité

✅ **Versions multiples sans conflit**
- Python 3.9 pour un projet, Python 3.12 pour un autre
- Node 18 et Node 22 sur la même machine
- Aucun conflit de dépendances

### Pour les équipes

✅ **Onboarding accéléré**
```bash
# Au lieu de 2 jours d'installation...
git clone projet  
cd projet  
docker compose up  
# ... 5 minutes !
```

✅ **Documentation vivante**
- Le Dockerfile documente l'environnement nécessaire
- Plus de "README.md" obsolète
- La configuration est dans le code

✅ **Collaboration simplifiée**
- Tous travaillent dans le même environnement
- Pas de "je ne peux pas reproduire ton bug"

### Pour l'infrastructure

✅ **Déploiement simplifié**
- Une seule commande pour déployer
- Rollback instantané vers une version précédente
- Mise à l'échelle facilitée

✅ **Isolation et sécurité**
- Chaque application dans son conteneur
- Limitation des ressources par conteneur
- Isolation des dépendances

## De l'image de base à l'image personnalisée

### Le concept de "couches"

Créer une image personnalisée, c'est **ajouter des couches** par-dessus une image de base :

```
┌──────────────────────────────┐
│  Votre application           │  ← Votre code
├──────────────────────────────┤
│  Dépendances de l'app        │  ← npm install, pip install...
├──────────────────────────────┤
│  Configuration système       │  ← Packages, outils
├──────────────────────────────┤
│  Image de base               │  ← ubuntu, node, python...
└──────────────────────────────┘
```

### Exemple de transformation

**Point de départ** : Image Node.js de base
```bash
docker run node:18
# Contient : Node.js, npm, système d'exploitation de base
```

**Point d'arrivée** : Votre application e-commerce
```
Image personnalisée "mon-ecommerce:v1.0"
└─ Contient :
   ├─ Node.js 18 (de l'image de base)
   ├─ Express.js et toutes vos dépendances
   ├─ Votre code source
   ├─ Vos fichiers de configuration
   └─ Vos assets (images, CSS, JS)
```

## Le workflow de création d'images

Voici le processus typique que vous allez apprendre :

### 1. Écrire un Dockerfile

Un fichier texte qui décrit comment construire votre image :

```dockerfile
FROM node:18  
WORKDIR /app  
COPY package.json .  
RUN npm install  
COPY . .  
CMD ["npm", "start"]  
```

### 2. Construire l'image

```bash
docker build -t mon-app:v1 .
```

Docker lit le Dockerfile et crée l'image automatiquement.

### 3. Tester l'image

```bash
docker run -p 3000:3000 mon-app:v1
```

Vérifier que tout fonctionne comme prévu.

### 4. Itérer et améliorer

Modifier le Dockerfile, reconstruire, tester... jusqu'à obtenir l'image parfaite.

### 5. Partager et déployer

```bash
docker push mon-registre/mon-app:v1
```

L'image est maintenant disponible pour toute votre équipe ou pour la production.

## Ce que vous allez apprendre dans cette section

Cette section complète vous guidera à travers tous les aspects de la création d'images personnalisées :

### 5.1 Comprendre le Dockerfile
Vous découvrirez le **fichier Dockerfile**, le point de départ de toute création d'image. Vous apprendrez :
- Ce qu'est un Dockerfile et comment il fonctionne
- La structure et la syntaxe de base
- Le concept fondamental de couches (layers)
- Le système de cache qui accélère vos builds
- Les conventions et bonnes pratiques

### 5.2 Instructions de base (FROM, RUN, COPY, ADD)
Vous maîtriserez les **quatre instructions essentielles** pour créer vos premières images :
- **FROM** : choisir et utiliser une image de base
- **RUN** : exécuter des commandes pendant la construction
- **COPY** : copier vos fichiers dans l'image
- **ADD** : une alternative à COPY avec des fonctionnalités supplémentaires
- La différence cruciale entre **CMD** et **ENTRYPOINT**

### 5.3 Instructions avancées
Vous approfondirez avec des instructions pour des configurations plus sophistiquées :
- **ENV** : gérer les variables d'environnement
- **WORKDIR** : organiser la structure de fichiers
- **EXPOSE** : documenter les ports réseau
- **USER** : gérer la sécurité et les permissions
- **VOLUME** : configurer la persistance des données

### 5.4 Construction d'images (build, tag)
Vous apprendrez à **construire et gérer** vos images :
- Utiliser `docker build` avec toutes ses options
- Système de tags pour versionner vos images
- Comprendre le contexte de build
- Utiliser `.dockerignore` efficacement
- Déboguer les erreurs de construction

### 5.5 Optimisation des Dockerfiles et layers
Vous découvrirez comment créer des images **performantes et légères** :
- Optimiser l'ordre des instructions pour le cache
- Réduire le nombre et la taille des couches
- Choisir la bonne image de base
- Techniques de nettoyage et de minimisation
- Analyse de la taille des images

## Les outils que vous utiliserez

### Le Dockerfile

Le **fichier de recette** qui décrit votre image. C'est un simple fichier texte que vous éditez avec votre éditeur favori :

```
mon-projet/
├── Dockerfile       ← Le fichier magique !
├── src/
│   └── app.py
├── requirements.txt
└── README.md
```

### La commande docker build

L'outil qui **transforme** votre Dockerfile en image :

```bash
docker build -t mon-image:v1.0 .
```

### Les registres d'images

Des **bibliothèques** où stocker et partager vos images :
- **Docker Hub** : registre public gratuit
- **Registres privés** : pour les images internes d'entreprise
- **GitHub Container Registry**, **Amazon ECR**, **Google Artifact Registry**...

## Analogies pour bien comprendre

### Analogie 1 : La recette de cuisine

**Image de base** = Pâte à pizza du commerce  
**Dockerfile** = Votre recette personnalisée  
**docker build** = Le four qui cuit la pizza  
**Image finale** = Votre pizza personnalisée prête à servir  
**Conteneur** = Une assiette avec une part de cette pizza  

### Analogie 2 : La construction d'une maison

**Image de base** = Fondations et structure de base  
**Instructions du Dockerfile** = Plans de construction  
**Couches (layers)** = Étages successifs  
**docker build** = L'équipe de construction  
**Image finale** = Maison terminée et habitable  

### Analogie 3 : Le modèle et les copies

**Image** = Un moule à gaufres  
**Conteneur** = Une gaufre faite avec ce moule  
- Le moule (image) reste intact
- On peut faire autant de gaufres (conteneurs) qu'on veut
- Chaque gaufre est identique aux autres

## Prérequis pour cette section

Pour tirer le meilleur parti de cette section, assurez-vous de :

✅ Avoir Docker installé et fonctionnel  
✅ Connaître les commandes de base (sections 1-4)  
✅ Comprendre ce qu'est une image et un conteneur  
✅ Savoir utiliser `docker run`, `docker ps`, `docker images`  
✅ Avoir un éditeur de texte (VS Code, Sublime, Vim, etc.)

## État d'esprit et approche

### Soyez expérimental

N'ayez pas peur d'expérimenter ! Les erreurs lors de la création d'images sont :
- **Sans danger** : vous ne risquez pas d'endommager votre système
- **Instructives** : les messages d'erreur vous guident
- **Faciles à corriger** : modifiez le Dockerfile et reconstruisez

### Itérez progressivement

Commencez simple et ajoutez de la complexité petit à petit :

1. **Image minimaliste** : fait juste fonctionner l'app
2. **Image fonctionnelle** : ajoute la configuration nécessaire
3. **Image optimisée** : réduit la taille et améliore les performances
4. **Image production-ready** : ajoute sécurité et monitoring

### Apprenez par la pratique

La meilleure façon d'apprendre est de **créer vos propres Dockerfiles**. Essayez de conteneuriser :
- Un script Python simple
- Une application web statique
- Un petit serveur API
- Une de vos applications existantes

## Exemples de ce que vous pourrez créer

À la fin de cette section, vous serez capable de créer des images pour :

### Applications web
- Site statique avec Nginx
- Application Node.js/Express
- Application Python/Flask ou Django
- Application PHP/Laravel

### APIs et microservices
- API REST en Python
- API GraphQL en Node.js
- Microservice en Go

### Applications de données
- Scripts d'analyse de données
- Applications de traitement batch
- ETL (Extract, Transform, Load)

### Outils et utilitaires
- Scripts d'automatisation
- Outils CLI personnalisés
- Services de monitoring

## Philosophie Docker : Infrastructure as Code

En créant des Dockerfiles, vous adoptez une philosophie moderne :

**Avant Docker** :
```
Configuration manuelle du serveur  
Documentation dans un wiki (souvent obsolète)  
"On a installé ça il y a 2 ans, je ne sais plus comment..."
```

**Avec Docker** :
```
Configuration dans le Dockerfile (versionnée avec Git)  
Documentation vivante et toujours à jour  
Reproductible à l'infini  
```

Votre infrastructure devient du **code** :
- Versionné avec Git
- Testé automatiquement
- Revu par l'équipe (code review)
- Documenté par nature

## Prêt à commencer ?

Vous êtes maintenant prêt à franchir cette étape importante ! La création d'images personnalisées peut sembler complexe au premier abord, mais vous verrez qu'avec de la pratique, cela devient naturel et même agréable.

Chaque section vous guidera pas à pas, avec :
- Des explications claires et accessibles
- De nombreux exemples concrets
- Des cas d'usage réels
- Des bonnes pratiques
- Des pièges à éviter

**N'oubliez pas** : même les experts Docker ont commencé par leur premier Dockerfile simple. L'important est de commencer, d'expérimenter, et d'apprendre de vos erreurs.

**Commençons par comprendre le Dockerfile, l'outil fondamental de la création d'images !**

⏭️ [Comprendre le Dockerfile](/05-creation-dimages-personnalisees/01-comprendre-le-dockerfile.md)
