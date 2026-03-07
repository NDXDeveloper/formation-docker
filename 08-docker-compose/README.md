🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 8. Docker Compose

## Introduction au chapitre

Bienvenue dans le chapitre consacré à Docker Compose ! 🎉

Jusqu'à présent, vous avez appris à travailler avec des conteneurs Docker individuels : créer des images, lancer des conteneurs, gérer les volumes et les réseaux. Mais que se passe-t-il lorsque votre application devient plus complexe et nécessite plusieurs conteneurs qui doivent travailler ensemble ?

Imaginez une application web moderne typique :
- Un serveur web (Nginx) pour servir le frontend
- Une application backend (Node.js, Python, Java...)
- Une base de données (PostgreSQL, MySQL, MongoDB...)
- Un système de cache (Redis)
- Peut-être une file de messages (RabbitMQ)

Gérer tous ces conteneurs manuellement avec des commandes `docker run` devient rapidement un cauchemar :
- Des commandes longues et complexes à mémoriser
- La difficulté de gérer les dépendances entre les services
- Le risque d'erreurs à chaque démarrage
- L'impossibilité de partager facilement votre configuration avec votre équipe

**C'est exactement le problème que Docker Compose résout ! ✨**

## Qu'est-ce que Docker Compose ?

Docker Compose est un outil qui vous permet de définir et gérer des applications Docker multi-conteneurs. Au lieu de lancer vos conteneurs un par un avec des commandes complexes, vous décrivez toute votre application dans un simple fichier de configuration (`docker-compose.yml`), puis vous la démarrez avec une seule commande.

**Un seul fichier. Une seule commande. Toute votre application.**

### De ceci...

```bash
# Créer un réseau
docker network create mon-app-network

# Lancer la base de données
docker run -d \
  --name ma-base \
  --network mon-app-network \
  -v donnees-db:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

# Lancer Redis
docker run -d \
  --name mon-cache \
  --network mon-app-network \
  redis:7

# Lancer l'application
docker run -d \
  --name mon-api \
  --network mon-app-network \
  -p 3000:3000 \
  -e DATABASE_URL=postgresql://ma-base:5432 \
  -e REDIS_URL=redis://mon-cache:6379 \
  mon-image-api

# Et il faut se souvenir de tout ça !
```

### ...à cela !

```yaml
# docker-compose.yml
services:
  database:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - donnees-db:/var/lib/postgresql/data

  cache:
    image: redis:7

  api:
    image: mon-image-api
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgresql://database:5432
      REDIS_URL: redis://cache:6379
    depends_on:
      - database
      - cache

volumes:
  donnees-db:
```

Puis simplement :
```bash
docker compose up
```

**Magique, n'est-ce pas ? 🪄**

## Pourquoi Docker Compose est indispensable

### 1. Simplicité

Une seule commande pour démarrer toute votre application, au lieu de gérer chaque conteneur individuellement.

### 2. Reproductibilité

Votre fichier `docker-compose.yml` peut être versionné avec Git et partagé avec votre équipe. Tout le monde aura exactement le même environnement de développement.

### 3. Documentation vivante

Le fichier `docker-compose.yml` documente votre architecture. Un nouveau développeur peut comprendre immédiatement de quels services l'application est composée.

### 4. Isolation

Chaque projet peut avoir son propre environnement Docker Compose, évitant les conflits entre projets.

### 5. Portabilité

Que vous soyez sur Windows, macOS ou Linux, votre application fonctionnera de la même manière.

## Ce que vous allez apprendre dans ce chapitre

Ce chapitre est structuré pour vous emmener progressivement de débutant à utilisateur confiant de Docker Compose :

### 📚 Section 8.1 : Introduction à Docker Compose
Vous découvrirez en détail ce qu'est Docker Compose, comment il fonctionne, et pourquoi c'est un outil essentiel pour tout développeur travaillant avec Docker.

### 📝 Section 8.2 : Structure du fichier docker-compose.yml
Vous apprendrez à écrire et comprendre un fichier `docker-compose.yml`. Nous explorerons toutes les sections importantes et les options de configuration disponibles.

### 🔗 Section 8.3 : Définir des services multiples
Vous verrez comment créer des applications complètes avec plusieurs services qui communiquent entre eux : bases de données, APIs, workers, caches, et plus encore.

### ⚙️ Section 8.4 : Gestion des dépendances entre services
Vous maîtriserez les techniques pour gérer l'ordre de démarrage et vous assurer que vos services démarrent de manière fiable, en particulier avec les bases de données.

### 💻 Section 8.5 : Commandes Docker Compose
Vous découvrirez toutes les commandes essentielles pour gérer votre application au quotidien : `up`, `down`, `logs`, `ps`, et bien d'autres.

## Cas d'usage concrets

Docker Compose est utilisé dans de nombreux scénarios :

### Développement local
```
Frontend React + Backend Node.js + PostgreSQL + Redis
```
Démarrez tout votre environnement de développement en une commande, avec hot-reload et debugging.

### Tests automatisés
```
Application + Base de données de test + Services externes mockés
```
Créez des environnements de test isolés et reproductibles pour votre CI/CD.

### Prototypes et démonstrations
```
Stack complète : Nginx + Application + Base de données + Monitoring
```
Déployez rapidement une démo complète de votre application pour un client ou une présentation.

### Microservices en développement
```
Service Auth + Service Users + Service Products + API Gateway + Base de données
```
Orchestrez plusieurs microservices localement pour le développement et les tests.

## Prérequis pour ce chapitre

Avant de commencer ce chapitre, assurez-vous d'être à l'aise avec :

✅ Les concepts de base de Docker (images, conteneurs)  
✅ Les commandes Docker essentielles (`docker run`, `docker build`)  
✅ Les volumes Docker pour la persistance des données  
✅ Les réseaux Docker pour la communication entre conteneurs

Si vous avez suivi les chapitres précédents de cette formation, vous avez déjà toutes les connaissances nécessaires !

## Installation de Docker Compose

**Bonne nouvelle !** Si vous avez installé Docker Desktop (Windows ou macOS), Docker Compose est déjà inclus. Vous n'avez rien d'autre à installer.

Sur Linux, Docker Compose est généralement inclus avec Docker Engine dans les versions récentes.

Pour vérifier que Docker Compose est installé :

```bash
docker compose version
```

Si vous voyez un numéro de version (par exemple `Docker Compose version v2.23.0`), vous êtes prêt à commencer ! 🚀

## Votre premier aperçu

Pour vous donner un avant-goût, voici à quoi ressemble un fichier Docker Compose complet pour une application WordPress :

```yaml
services:
  # Base de données MySQL
  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppass
    volumes:
      - db_data:/var/lib/mysql

  # Application WordPress
  wordpress:
    image: wordpress:latest
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppass
      WORDPRESS_DB_NAME: wordpress
    depends_on:
      - db
    volumes:
      - wp_data:/var/www/html

volumes:
  db_data:
  wp_data:
```

Avec ces quelques lignes et la commande `docker compose up -d`, vous avez un site WordPress complet qui tourne !

## Comment aborder ce chapitre

### Pour les débutants complets
Lisez les sections dans l'ordre. Chaque section s'appuie sur les précédentes et introduit progressivement de nouveaux concepts.

### Pour ceux qui ont déjà utilisé Docker Compose
Vous pouvez sauter directement aux sections qui vous intéressent, comme la gestion des dépendances (8.4) ou les commandes avancées (8.5).

### Approche pratique
Bien que ce tutoriel n'inclue pas d'exercices formels, nous vous encourageons vivement à :
- Créer vos propres fichiers `docker-compose.yml` pendant votre lecture
- Tester les exemples donnés
- Expérimenter avec différentes configurations
- Casser des choses (c'est comme ça qu'on apprend !)

**La meilleure façon d'apprendre Docker Compose est de l'utiliser ! 💪**

## Conventions utilisées dans ce chapitre

### Exemples de code
Les exemples de code YAML seront formatés ainsi :
```yaml
services:
  mon-service:
    image: nginx
```

### Commandes shell
Les commandes à taper dans votre terminal seront formatées ainsi :
```bash
docker compose up -d
```

### Notes importantes
Les informations critiques seront mises en évidence :
⚠️ **Attention** : Points importants à ne pas manquer  
✅ **Conseil** : Bonnes pratiques recommandées  
💡 **Astuce** : Astuces pour gagner du temps

## Mindset pour réussir

Docker Compose peut sembler intimidant au début, mais gardez à l'esprit :

1. **C'est normal de ne pas tout comprendre immédiatement** - Docker Compose a beaucoup d'options, mais vous n'avez besoin que d'un sous-ensemble pour commencer.

2. **Les erreurs font partie de l'apprentissage** - Vous allez faire des erreurs de syntaxe YAML, oublier des deux-points, mal indenter votre fichier. C'est normal et c'est comme ça qu'on apprend !

3. **Commencez simple** - Ne créez pas immédiatement une architecture de 15 services. Commencez par 2-3 services et augmentez progressivement la complexité.

4. **La documentation est votre amie** - N'hésitez pas à consulter la [documentation officielle de Docker Compose](https://docs.docker.com/compose/) pour des détails supplémentaires.

5. **La communauté est là** - Des millions de développeurs utilisent Docker Compose. Si vous avez un problème, quelqu'un l'a probablement déjà résolu !

## À la fin de ce chapitre, vous saurez...

✅ Créer des fichiers `docker-compose.yml` pour vos projets  
✅ Orchestrer plusieurs services qui communiquent entre eux  
✅ Gérer les dépendances et l'ordre de démarrage  
✅ Utiliser les commandes essentielles pour gérer vos applications  
✅ Déboguer les problèmes courants  
✅ Appliquer les bonnes pratiques pour des configurations robustes  
✅ Créer des environnements de développement reproductibles

## Prêt à commencer ?

Docker Compose va transformer votre façon de travailler avec Docker. Au lieu de jongler avec des dizaines de commandes, vous allez pouvoir gérer des applications complexes avec élégance et simplicité.

**Tournons la page et plongeons dans le vif du sujet ! 🏊‍♂️**

---

**Prochaine section** : [8.1 Introduction à Docker Compose](/08-docker-compose/01-introduction-a-docker-compose.md) - Découvrons en détail cet outil puissant !

⏭️ [Introduction à Docker Compose](/08-docker-compose/01-introduction-a-docker-compose.md)
