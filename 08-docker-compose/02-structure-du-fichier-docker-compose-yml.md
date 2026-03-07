🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 8.2 Structure du fichier docker-compose.yml

## Introduction

Le fichier `docker-compose.yml` est le cœur de Docker Compose. C'est dans ce fichier que vous allez décrire toute votre application : quels conteneurs lancer, comment ils communiquent entre eux, quelles ressources ils utilisent, etc.

Dans cette section, nous allons décortiquer la structure de ce fichier pour que vous puissiez créer vos propres configurations en toute confiance.

## Le format YAML : Les bases à connaître

Avant de plonger dans Docker Compose, il est important de comprendre quelques règles du format YAML :

### 1. L'indentation est cruciale

YAML utilise l'indentation (les espaces en début de ligne) pour définir la structure. **Utilisez toujours des espaces, jamais des tabulations.**

```yaml
# ✅ Correct
services:
  web:
    image: nginx

# ❌ Incorrect (mauvaise indentation)
services:  
web:  
  image: nginx
```

### 2. Les deux-points pour les paires clé-valeur

```yaml
clé: valeur  
nom: Docker  
port: 8080  
```

### 3. Les tirets pour les listes

```yaml
ports:
  - "8080:80"
  - "443:443"

volumes:
  - ./data:/app/data
  - ./config:/app/config
```

### 4. Les commentaires commencent par #

```yaml
# Ceci est un commentaire
services:
  web:  # Commentaire en fin de ligne
    image: nginx
```

C'est tout ce que vous devez savoir sur YAML pour commencer ! 🎓

## Structure globale d'un fichier docker-compose.yml

Voici le squelette de base d'un fichier Docker Compose :

```yaml
services:
  # Vos conteneurs ici

volumes:
  # Vos volumes nommés ici (optionnel)

networks:
  # Vos réseaux personnalisés ici (optionnel)
```

Les trois sections principales sont :
- **services** : La définition de vos conteneurs (obligatoire)
- **volumes** : La définition des volumes nommés (optionnel)
- **networks** : La définition des réseaux personnalisés (optionnel)

Examinons chaque section en détail.

> 💡 **Note historique** : Dans les anciennes versions de Docker Compose (V1), il était nécessaire d'ajouter une ligne `version: '3.8'` en haut du fichier. Avec Docker Compose V2 (intégré comme plugin `docker compose`), cette ligne n'est plus nécessaire et est ignorée. Vous pouvez simplement l'omettre. Si vous rencontrez d'anciens fichiers avec cette ligne, sachez qu'elle n'a plus d'effet.

## La section services

C'est la section la plus importante ! C'est ici que vous définissez tous vos conteneurs.

### Structure de base d'un service

```yaml
services:
  nom-du-service:
    image: nom-de-l-image:tag
    # autres configurations...
```

Le nom du service (`nom-du-service`) est important car :
- C'est le nom par lequel les autres services pourront le contacter sur le réseau
- Il apparaîtra dans les logs et commandes Docker Compose

### Exemple avec plusieurs services

```yaml
services:
  frontend:
    image: nginx:alpine
    ports:
      - "80:80"

  backend:
    image: node:18
    ports:
      - "3000:3000"

  database:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: motdepasse
```

Dans cet exemple, nous avons trois services : `frontend`, `backend` et `database`.

## Les propriétés principales d'un service

Voyons maintenant les propriétés les plus courantes que vous pouvez définir pour chaque service.

### image

Spécifie l'image Docker à utiliser pour ce service.

```yaml
services:
  web:
    image: nginx:latest

  database:
    image: postgres:15-alpine

  app:
    image: mon-utilisateur/mon-app:v1.0
```

### build

Si vous voulez construire une image à partir d'un Dockerfile au lieu d'utiliser une image existante :

```yaml
services:
  app:
    build: .  # Construit à partir du Dockerfile dans le répertoire courant

  # Ou avec plus de détails :
  api:
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
```

**Note** : Vous ne pouvez pas utiliser `image` et `build` en même temps de manière conflictuelle, mais vous pouvez donner un nom à l'image construite :

```yaml
services:
  app:
    build: .
    image: mon-app:dev  # Nom de l'image une fois construite
```

### ports

Expose les ports du conteneur sur votre machine hôte.

```yaml
services:
  web:
    image: nginx
    ports:
      - "8080:80"      # Format: "port-hôte:port-conteneur"
      - "443:443"
      - "3000:3000"
```

**Explication** : `"8080:80"` signifie "le port 80 du conteneur sera accessible sur le port 8080 de ma machine".

### environment

Définit les variables d'environnement pour le conteneur.

```yaml
services:
  database:
    image: postgres:15
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret123
      POSTGRES_DB: ma_base

  # Ou avec une autre syntaxe :
  app:
    image: mon-app
    environment:
      - NODE_ENV=production
      - API_KEY=abc123
      - DEBUG=false
```

### volumes

Monte des volumes ou des répertoires dans le conteneur.

```yaml
services:
  web:
    image: nginx
    volumes:
      # Bind mount (lier un dossier local)
      - ./mon-site:/usr/share/nginx/html

      # Volume nommé
      - mes-donnees:/var/lib/data

      # Volume anonyme
      - /var/log
```

### depends_on

Indique qu'un service dépend d'un autre et doit démarrer après lui.

```yaml
services:
  app:
    image: mon-app
    depends_on:
      - database
      - cache

  database:
    image: postgres:15

  cache:
    image: redis:alpine
```

**Important** : `depends_on` attend seulement que le conteneur démarre, pas qu'il soit prêt à accepter des connexions. Pour des dépendances plus sophistiquées, il existe des solutions que nous verrons plus tard.

### networks

Connecte le service à un ou plusieurs réseaux.

```yaml
services:
  web:
    image: nginx
    networks:
      - frontend

  api:
    image: mon-api
    networks:
      - frontend
      - backend

  database:
    image: postgres:15
    networks:
      - backend

networks:
  frontend:
  backend:
```

### restart

Définit la politique de redémarrage du conteneur.

```yaml
services:
  app:
    image: mon-app
    restart: always  # Redémarre toujours si le conteneur s'arrête

  worker:
    image: mon-worker
    restart: unless-stopped  # Redémarre sauf si arrêté manuellement

  test:
    image: mon-test
    restart: no  # Ne redémarre jamais (par défaut)

  critical:
    image: critique
    restart: on-failure  # Redémarre uniquement en cas d'erreur
```

### container_name

Permet de donner un nom spécifique au conteneur (optionnel).

```yaml
services:
  database:
    image: postgres:15
    container_name: ma-base-postgres
```

**Note** : Sans cette option, Docker Compose génère automatiquement un nom comme `monprojet_database_1`.

### command

Remplace la commande par défaut du conteneur.

```yaml
services:
  app:
    image: node:18
    command: npm run dev

  database:
    image: postgres:15
    command: postgres -c max_connections=200
```

### working_dir

Définit le répertoire de travail dans le conteneur.

```yaml
services:
  app:
    image: node:18
    working_dir: /app
    volumes:
      - ./src:/app
    command: npm start
```

## La section volumes

Cette section permet de définir des volumes nommés qui peuvent être réutilisés par plusieurs services.

```yaml
services:
  app:
    image: mon-app
    volumes:
      - donnees-app:/data

  backup:
    image: mon-backup
    volumes:
      - donnees-app:/data:ro  # ro = read-only (lecture seule)

volumes:
  donnees-app:  # Déclaration du volume nommé
```

Les volumes définis ici sont créés par Docker et gérés automatiquement. Ils persistent même si vous supprimez les conteneurs.

### Options avancées pour les volumes

```yaml
volumes:
  mes-donnees:
    driver: local
    driver_opts:
      type: none
      device: /chemin/sur/l-hote
      o: bind
```

Pour les débutants, la déclaration simple suffit largement !

## La section networks

Cette section permet de définir des réseaux personnalisés.

```yaml
services:
  web:
    image: nginx
    networks:
      - frontend

  api:
    image: mon-api
    networks:
      - frontend
      - backend

  database:
    image: postgres:15
    networks:
      - backend

networks:
  frontend:
  backend:
```

**Par défaut**, si vous ne définissez pas de réseaux, Docker Compose crée automatiquement un réseau par défaut où tous les services peuvent communiquer.

### Options des réseaux

```yaml
networks:
  frontend:
    driver: bridge

  backend:
    driver: bridge
    internal: true  # Réseau sans accès à Internet
```

## Exemple complet et commenté

Voici un exemple complet qui combine tous ces éléments :

```yaml
services:
  # Service web (frontend)
  web:
    image: nginx:alpine
    container_name: mon-site-web
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./site:/usr/share/nginx/html:ro
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    networks:
      - frontend
    depends_on:
      - api
    restart: unless-stopped

  # Service API (backend)
  api:
    build: ./backend
    container_name: mon-api
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: production
      DATABASE_URL: postgresql://database:5432/madb
      REDIS_URL: redis://cache:6379
    volumes:
      - ./backend:/app
      - /app/node_modules
    networks:
      - frontend
      - backend
    depends_on:
      - database
      - cache
    restart: unless-stopped

  # Base de données
  database:
    image: postgres:15-alpine
    container_name: ma-base-donnees
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: ${DB_PASSWORD}  # Utilise une variable d'environnement
      POSTGRES_DB: madb
    volumes:
      - donnees-postgres:/var/lib/postgresql/data
    networks:
      - backend
    restart: unless-stopped

  # Cache Redis
  cache:
    image: redis:alpine
    container_name: mon-cache
    networks:
      - backend
    restart: unless-stopped

# Déclaration des volumes nommés
volumes:
  donnees-postgres:

# Déclaration des réseaux
networks:
  frontend:
  backend:
```

## Bonnes pratiques pour organiser votre fichier

### 1. Commentez votre code

```yaml
services:
  # Service principal de l'application
  app:
    image: mon-app
    # Exposition sur le port 8080 pour le développement
    ports:
      - "8080:8000"
```

### 2. Utilisez des noms explicites

```yaml
# ✅ Bon
services:
  api-utilisateurs:
    image: mon-api

  database-principale:
    image: postgres

# ❌ Moins clair
services:
  svc1:
    image: mon-api

  db:
    image: postgres
```

### 3. Groupez les services par fonction

```yaml
services:
  # --- Frontend ---
  web:
    image: nginx

  interface-admin:
    image: admin-ui

  # --- Backend ---
  api:
    image: api-service

  worker:
    image: worker-service

  # --- Data ---
  database:
    image: postgres

  cache:
    image: redis
```

### 4. Utilisez des variables d'environnement pour les données sensibles

Créez un fichier `.env` à côté de votre `docker-compose.yml` :

```env
DB_PASSWORD=supersecret  
API_KEY=abc123xyz  
```

Puis utilisez-les dans votre fichier :

```yaml
services:
  database:
    image: postgres
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}

  api:
    image: mon-api
    environment:
      API_KEY: ${API_KEY}
```

**Important** : N'oubliez pas d'ajouter `.env` dans votre `.gitignore` pour ne pas commiter les secrets !

## Les erreurs courantes à éviter

### 1. Problèmes d'indentation

```yaml
# ❌ Erreur : mauvaise indentation
services:  
web:  
  image: nginx

# ✅ Correct
services:
  web:
    image: nginx
```

### 2. Oublier les guillemets pour les ports

```yaml
# ❌ Peut causer des problèmes
ports:
  - 8080:80

# ✅ Recommandé
ports:
  - "8080:80"
```

### 3. Utiliser des tabulations au lieu d'espaces

YAML n'accepte que les espaces. Si vous avez une erreur de parsing, vérifiez que vous n'avez pas utilisé des tabulations.

### 4. Oublier les deux-points

```yaml
# ❌ Erreur : manque les deux-points
services:
  web
    image: nginx

# ✅ Correct
services:
  web:
    image: nginx
```

## Valider votre fichier docker-compose.yml

Pour vérifier que votre fichier est correctement formaté, utilisez la commande :

```bash
docker compose config
```

Cette commande affiche la configuration finale (avec toutes les valeurs par défaut) et vous indique s'il y a des erreurs de syntaxe.

## Points clés à retenir

✅ Le fichier `docker-compose.yml` utilise le format YAML avec une indentation stricte (espaces uniquement)

✅ La section `services` est obligatoire et contient la définition de vos conteneurs

✅ Les sections `volumes` et `networks` sont optionnelles mais utiles pour organiser votre application

✅ Les propriétés principales d'un service sont : `image`, `build`, `ports`, `environment`, `volumes`, `depends_on`

✅ Utilisez des commentaires et des noms explicites pour rendre votre fichier lisible

✅ Utilisez des variables d'environnement (fichier `.env`) pour les données sensibles

✅ Validez votre fichier avec `docker compose config` avant de l'utiliser

---

**Prochaine section** : Nous allons voir comment définir plusieurs services et les faire communiquer entre eux dans des applications réelles !

⏭️ [Définir des services multiples](/08-docker-compose/03-definir-des-services-multiples.md)
