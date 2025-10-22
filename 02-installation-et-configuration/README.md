🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 2. Installation et configuration

## Bienvenue dans la partie pratique !

Vous avez terminé le premier chapitre et vous comprenez maintenant ce qu'est Docker, comment il fonctionne, et pourquoi il est devenu incontournable. Il est temps de passer à l'action ! Dans ce chapitre, nous allons installer Docker sur votre système et le configurer correctement pour que vous puissiez commencer à créer et gérer vos premiers conteneurs.

L'installation de Docker est beaucoup plus simple que vous ne l'imaginez peut-être. En quelques minutes, vous aurez un environnement Docker pleinement opérationnel, prêt à exécuter vos premières commandes.

## Pourquoi ce chapitre est crucial

L'installation et la configuration sont les fondations de votre expérience Docker. Une installation correcte vous évitera des frustrations plus tard et vous permettra de vous concentrer sur l'apprentissage plutôt que sur le dépannage.

### Ce que vous allez accomplir

À la fin de ce chapitre, vous aurez :
- Docker installé et fonctionnel sur votre machine
- Compris les différences entre les options d'installation
- Configuré Docker selon les meilleures pratiques
- Vérifié que tout fonctionne correctement
- Un environnement prêt pour vos premiers projets

### La réalité de l'installation moderne

Il y a quelques années, installer Docker pouvait être laborieux, avec de nombreuses étapes manuelles et des problèmes de compatibilité. Aujourd'hui, grâce aux efforts de Docker Inc. et de la communauté, l'installation est devenue remarquablement simple et fiable.

Que vous soyez sur Windows, macOS ou Linux, vous bénéficierez d'installateurs modernes qui gèrent automatiquement la plupart des configurations complexes.

## Vue d'ensemble du chapitre

### 2.1 Installation de Docker selon votre système d'exploitation

La première étape : installer Docker ! Cette section vous guidera à travers le processus d'installation spécifique à votre système d'exploitation.

**Ce que vous apprendrez :**
- Les différentes options disponibles (Docker Desktop vs Docker Engine)
- Les prérequis système pour chaque plateforme
- Les instructions détaillées pour Windows, macOS et Linux
- Comment vérifier que l'installation a réussi
- Les problèmes courants et leurs solutions

Nous couvrirons toutes les distributions Linux populaires (Ubuntu, Debian, Fedora, Arch, CentOS) ainsi que les spécificités de Windows 10/11 et macOS (Intel et Apple Silicon).

### 2.2 Configuration post-installation

Une fois Docker installé, quelques ajustements permettront d'optimiser votre expérience.

**Ce que vous apprendrez :**
- Configurer Docker pour démarrer automatiquement
- Sur Linux : utiliser Docker sans sudo
- Allouer les ressources appropriées (CPU, RAM, disque)
- Configurer les paramètres réseau de base
- Paramétrer Docker Desktop (si applicable)
- Optimiser les performances selon votre utilisation

Cette configuration initiale vous fera gagner du temps et évitera des problèmes par la suite.

### 2.3 Vérification de l'installation

Avant de vous lancer dans la création de conteneurs, il est important de s'assurer que tout fonctionne parfaitement.

**Ce que vous apprendrez :**
- Tester la connexion au Docker Daemon
- Vérifier les composants installés
- Exécuter vos premiers conteneurs de test
- Comprendre les messages de sortie
- Diagnostiquer les problèmes éventuels
- Valider que votre configuration est optimale

Ces tests de validation sont rapides mais essentiels pour partir sur de bonnes bases.

### 2.4 Docker Desktop vs Docker Engine

Cette section vous aidera à comprendre les différences entre ces deux options et à choisir la meilleure pour votre cas d'usage.

**Ce que vous apprendrez :**
- Les caractéristiques de Docker Desktop
- Les caractéristiques de Docker Engine
- Quand utiliser l'un ou l'autre
- Les avantages et inconvénients de chaque solution
- Comment passer de l'un à l'autre si nécessaire
- Les fonctionnalités spécifiques de Docker Desktop

Cette compréhension vous permettra de faire un choix éclairé ou de mieux utiliser la solution que vous avez installée.

## À quoi s'attendre pendant l'installation

### Durée estimée

L'installation complète de Docker et sa configuration prennent généralement :
- **Windows** : 15-20 minutes (incluant WSL 2)
- **macOS** : 10-15 minutes
- **Linux** : 10-20 minutes selon la distribution

Ces durées incluent les téléchargements, l'installation, la configuration et les vérifications.

### Niveau de difficulté

**Ne vous inquiétez pas !** Même si vous n'êtes pas très à l'aise avec la ligne de commande, les instructions sont détaillées et expliquées pas à pas. Nous avons conçu ce chapitre pour que même un débutant complet puisse suivre sans difficulté.

Pour chaque système d'exploitation, vous trouverez :
- Des explications claires de chaque étape
- Les commandes exactes à exécuter
- Des captures d'écran conceptuelles (descriptions)
- Des solutions aux problèmes courants

### Ce dont vous aurez besoin

Avant de commencer, assurez-vous d'avoir :
- **Un ordinateur** avec Windows 10/11, macOS ou Linux
- **Une connexion internet** pour télécharger Docker
- **Des droits administrateur** sur votre machine
- **Environ 5 Go d'espace disque** disponible
- **Au moins 4 Go de RAM** (8 Go recommandés)

Si votre machine ne remplit pas ces critères, Docker pourrait fonctionner au ralenti ou rencontrer des problèmes.

## Approche pédagogique de ce chapitre

### Progressivité

Nous commençons par l'installation basique, puis nous ajoutons progressivement des configurations et optimisations. Vous pouvez vous arrêter après l'installation minimale si vous voulez rapidement passer aux chapitres suivants, puis revenir à la configuration plus tard.

### Adaptabilité

Les instructions sont adaptées à votre niveau :
- Si vous êtes débutant, suivez chaque étape dans l'ordre
- Si vous êtes plus expérimenté, vous pouvez sauter certaines explications
- Des encadrés "Pour aller plus loin" proposent des informations avancées optionnelles

### Pragmatisme

Ce chapitre se concentre sur ce qui fonctionne. Les options et configurations proposées sont celles qui ont fait leurs preuves auprès de milliers d'utilisateurs.

## Conseils avant de commencer

### 1. Lisez d'abord, agissez ensuite

Avant d'exécuter des commandes, lisez toute la section concernée pour comprendre ce que vous allez faire. Cela vous évitera des erreurs et vous donnera une vision d'ensemble du processus.

### 2. Ne paniquez pas en cas d'erreur

Les erreurs d'installation sont normales et généralement faciles à résoudre. Pour chaque étape critique, nous fournissons une section de dépannage avec les solutions aux problèmes les plus fréquents.

### 3. Gardez une fenêtre de documentation ouverte

Ayez ce tutoriel sous les yeux pendant que vous effectuez l'installation. Vous pouvez l'afficher sur un deuxième écran ou l'imprimer si cela vous aide.

### 4. Notez les messages d'erreur

Si vous rencontrez un problème, notez le message d'erreur exact. Cela facilitera grandement la recherche de solutions.

### 5. Prenez votre temps

L'installation n'est pas une course. Mieux vaut prendre 30 minutes et tout faire correctement que de se précipiter et devoir tout recommencer.

## Environnements spécifiques

### Entreprise ou environnement géré

Si vous installez Docker sur un ordinateur d'entreprise, vous pourriez avoir besoin :
- D'autorisations spéciales de votre service informatique
- De passer par un proxy d'entreprise
- De respecter des politiques de sécurité spécifiques

Contactez votre service IT avant de commencer si vous avez des doutes.

### Machine virtuelle

Vous pouvez installer Docker dans une machine virtuelle, mais :
- Activez la virtualisation imbriquée (nested virtualization) si possible
- Allouez suffisamment de ressources (au moins 4 Go de RAM)
- Les performances seront légèrement réduites

Pour l'apprentissage sur une VM, c'est tout à fait acceptable.

### Machines anciennes ou limitées

Docker fonctionne sur des machines modestes, mais :
- Avec 4 Go de RAM, limitez le nombre de conteneurs simultanés
- Sur des disques lents, l'expérience sera moins fluide
- Les processeurs anciens peuvent ralentir certaines opérations

Docker restera fonctionnel, simplement moins rapide.

## Ce qui vous attend après ce chapitre

Une fois Docker installé et configuré, vous serez prêt à :
- Découvrir les concepts fondamentaux de Docker (Chapitre 3)
- Exécuter vos premières commandes Docker (Chapitre 4)
- Créer vos propres images personnalisées (Chapitre 5)
- Construire des applications multi-conteneurs (Chapitre 8)

L'installation est le point de départ de votre voyage Docker. Après cette étape, les choses deviennent vraiment intéressantes !

## Prêt à installer ?

Vous avez maintenant une vision claire de ce qui vous attend dans ce chapitre. L'installation de Docker est une étape excitante : c'est le moment où vous passez de la théorie à la pratique.

Choisissez la section correspondant à votre système d'exploitation et commençons l'installation. Dans quelques minutes, vous exécuterez votre premier conteneur Docker !

Allons-y...

⏭️ [Installation de Docker selon votre système d'exploitation](/02-installation-et-configuration/01-installation-de-docker-selon-votre-systeme.md)
