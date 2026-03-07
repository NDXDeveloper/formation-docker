🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 8.1 Introduction à Docker Compose

## Qu'est-ce que Docker Compose ?

Docker Compose est un outil qui permet de définir et gérer des applications Docker multi-conteneurs. Plutôt que de lancer vos conteneurs un par un avec des commandes `docker run` longues et complexes, Docker Compose vous permet de décrire toute votre application dans un seul fichier de configuration, puis de la démarrer avec une seule commande.

## Le problème que Docker Compose résout

Imaginons que vous développez une application web composée de :
- Un serveur web (nginx ou Apache)
- Une application backend (Node.js, Python, etc.)
- Une base de données (MySQL, PostgreSQL, MongoDB)
- Un système de cache (Redis)

Sans Docker Compose, vous devriez :
1. Lancer chaque conteneur manuellement avec des commandes `docker run` séparées
2. Configurer les réseaux pour que les conteneurs puissent communiquer entre eux
3. Créer et attacher les volumes pour la persistance des données
4. Gérer les dépendances (par exemple, s'assurer que la base de données démarre avant l'application)
5. Répéter toutes ces étapes à chaque fois que vous voulez démarrer votre application

Cela peut rapidement devenir fastidieux et source d'erreurs !

## La solution Docker Compose

Avec Docker Compose, vous définissez tous ces services dans un fichier unique appelé `docker-compose.yml`. Ce fichier décrit :
- Quels conteneurs doivent être lancés
- À partir de quelles images
- Comment ils sont connectés entre eux
- Quels volumes utiliser
- Quelles variables d'environnement configurer
- Et bien plus encore...

Une fois ce fichier créé, vous pouvez :
- Démarrer toute l'application avec `docker compose up`
- L'arrêter avec `docker compose down`
- Voir les logs de tous les services avec `docker compose logs`

## Exemple simple pour comprendre

Voici à quoi ressemble un fichier `docker-compose.yml` basique pour une application web avec une base de données :

```yaml
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"

  database:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: monmotdepasse
```

Ce simple fichier définit deux services qui vont tourner dans des conteneurs séparés. Avec une seule commande, vous pouvez les démarrer tous les deux !

## Les avantages principaux de Docker Compose

### 1. Simplicité et gain de temps
Plutôt que de taper de longues commandes Docker à chaque fois, vous écrivez la configuration une fois et la réutilisez indéfiniment.

### 2. Reproductibilité
Votre fichier `docker-compose.yml` peut être partagé avec votre équipe. Tout le monde aura exactement la même configuration d'environnement de développement.

### 3. Gestion centralisée
Tous vos services sont définis au même endroit, ce qui facilite la compréhension de l'architecture de votre application.

### 4. Environnements isolés
Docker Compose crée automatiquement un réseau isolé pour votre application, ce qui permet d'éviter les conflits avec d'autres projets.

### 5. Versionning
Votre fichier `docker-compose.yml` peut être versé dans Git avec votre code, ce qui permet de suivre l'évolution de votre infrastructure.

## Quand utiliser Docker Compose ?

Docker Compose est particulièrement utile pour :
- **Le développement local** : Reproduire rapidement un environnement de développement complet
- **Les tests** : Créer des environnements de test isolés et reproductibles
- **Les démonstrations** : Déployer rapidement une application complète pour une démo
- **Les petits déploiements** : Pour des applications simples en production (pour des besoins plus avancés, on se tournera vers Kubernetes ou Docker Swarm)

## Installation de Docker Compose

La bonne nouvelle : si vous avez installé Docker Desktop (sur Windows ou macOS), Docker Compose est déjà inclus ! Vous n'avez rien d'autre à faire.

Sur Linux, Docker Compose est généralement installé avec Docker Engine dans les versions récentes. Pour vérifier si vous avez Docker Compose, tapez :

```bash
docker compose version
```

Si vous voyez un numéro de version s'afficher, vous êtes prêt à l'utiliser !

**Note importante** : Les versions récentes utilisent la commande `docker compose` (avec un espace), tandis que les anciennes versions utilisaient `docker-compose` (avec un tiret). Les deux fonctionnent de manière similaire, mais la nouvelle syntaxe est recommandée.

## La structure d'un fichier docker-compose.yml

Un fichier Docker Compose est écrit en format YAML (Yet Another Markup Language), un format de configuration lisible par les humains. Voici les éléments de base :

```yaml
services:  # Liste des conteneurs à créer
  nom-du-service:
    image: nom-de-l-image
    # autres configurations...

  autre-service:
    image: autre-image
    # autres configurations...

volumes:  # Définition des volumes (optionnel)
  nom-du-volume:

networks:  # Définition des réseaux personnalisés (optionnel)
  nom-du-reseau:
```

Ne vous inquiétez pas si cela semble abstrait pour le moment. Nous allons détailler chaque élément dans les sections suivantes du tutoriel !

## Workflow typique avec Docker Compose

Voici comment vous allez généralement utiliser Docker Compose dans vos projets :

1. **Créer** un fichier `docker-compose.yml` à la racine de votre projet
2. **Définir** tous les services dont votre application a besoin
3. **Démarrer** l'application avec `docker compose up`
4. **Développer** votre code (les conteneurs tournent en arrière-plan)
5. **Consulter** les logs avec `docker compose logs` si nécessaire
6. **Arrêter** l'application avec `docker compose down` quand vous avez terminé

## Comparaison : Docker vs Docker Compose

Pour bien comprendre la différence, voici un exemple concret :

### Avec Docker seul :
```bash
# Créer un réseau
docker network create mon-reseau

# Créer un volume
docker volume create mes-donnees

# Lancer la base de données
docker run -d \
  --name ma-bdd \
  --network mon-reseau \
  -v mes-donnees:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

# Lancer l'application
docker run -d \
  --name mon-app \
  --network mon-reseau \
  -p 3000:3000 \
  -e DATABASE_URL=postgresql://ma-bdd:5432 \
  mon-image-app

# ... et il faut se souvenir de tout ça !
```

### Avec Docker Compose :
```yaml
# fichier docker-compose.yml
services:
  database:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - mes-donnees:/var/lib/postgresql/data

  app:
    image: mon-image-app
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgresql://database:5432
    depends_on:
      - database

volumes:
  mes-donnees:
```

Puis simplement :
```bash
docker compose up
```

La différence est frappante, n'est-ce pas ? 🎯

## Ce que vous allez apprendre dans les sections suivantes

Maintenant que vous comprenez ce qu'est Docker Compose et pourquoi c'est utile, dans les sections suivantes, nous allons approfondir :

- La structure détaillée du fichier `docker-compose.yml`
- Comment définir plusieurs services et les faire communiquer
- Comment gérer les dépendances entre services
- Les commandes Docker Compose les plus utiles
- Des exemples pratiques d'applications complètes

Docker Compose va considérablement simplifier votre travail avec Docker. C'est un outil indispensable pour tout développeur travaillant avec des applications conteneurisées !

## Points clés à retenir

✅ Docker Compose permet de gérer des applications multi-conteneurs facilement

✅ Tout est défini dans un fichier `docker-compose.yml` au format YAML

✅ Une seule commande (`docker compose up`) démarre toute votre application

✅ C'est idéal pour le développement local et les environnements de test

✅ Le fichier de configuration peut être versionné avec votre code

✅ Docker Compose est déjà inclus dans Docker Desktop

---

**Prochaine section** : Nous allons explorer en détail la structure du fichier `docker-compose.yml` et apprendre à configurer nos premiers services !

⏭️ [Structure du fichier docker-compose.yml](/08-docker-compose/02-structure-du-fichier-docker-compose-yml.md)
