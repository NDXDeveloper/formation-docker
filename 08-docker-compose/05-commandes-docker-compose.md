🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 8.5 Commandes Docker Compose (up, down, logs, ps)

## Introduction

Docker Compose s'utilise principalement via la ligne de commande. Une fois que vous avez créé votre fichier `docker-compose.yml`, vous allez utiliser la commande `docker compose` suivie de différentes sous-commandes pour gérer votre application.

Dans cette section, nous allons explorer les commandes les plus importantes que vous utiliserez au quotidien. Vous verrez qu'elles sont intuitives et puissantes ! 🚀

## Note importante : docker compose vs docker-compose

Il existe deux syntaxes :
- **`docker compose`** (avec un espace) : Version moderne, intégrée à Docker CLI depuis 2020
- **`docker-compose`** (avec un tiret) : Ancienne version, installée séparément

Dans ce tutoriel, nous utilisons la syntaxe moderne `docker compose`. Si vous avez une ancienne version, remplacez simplement l'espace par un tiret.

```bash
# Moderne (recommandé)
docker compose up

# Ancienne version (fonctionne encore)
docker-compose up
```

## Commande de base : docker compose

Pour voir toutes les commandes disponibles :

```bash
docker compose --help
```

Pour voir l'aide d'une commande spécifique :

```bash
docker compose up --help
```

## 1. docker compose up - Démarrer l'application

C'est LA commande que vous utiliserez le plus souvent. Elle démarre tous les services définis dans votre `docker-compose.yml`.

### Utilisation de base

```bash
docker compose up
```

**Ce qui se passe** :
1. Télécharge les images nécessaires (si elles n'existent pas)
2. Crée les réseaux définis
3. Crée les volumes définis
4. Démarre tous les conteneurs
5. Affiche les logs de tous les services dans le terminal

Les logs de tous les services apparaissent dans votre terminal, avec des couleurs différentes pour chaque service.

### Mode détaché (arrière-plan)

Pour démarrer les services en arrière-plan et récupérer votre terminal :

```bash
docker compose up -d
```

**-d** signifie "detached" (détaché). Vos services tournent en arrière-plan.

```bash
# Exemple de sortie
[+] Running 3/3
 ✔ Container mon-projet-database-1  Started
 ✔ Container mon-projet-cache-1     Started
 ✔ Container mon-projet-app-1       Started
```

### Reconstruire les images

Si vous avez modifié votre code ou votre Dockerfile, vous devez reconstruire les images :

```bash
docker compose up --build
```

**--build** force la reconstruction des images avant de démarrer les services.

```bash
# En mode détaché avec reconstruction
docker compose up -d --build
```

### Démarrer des services spécifiques

Pour démarrer seulement certains services :

```bash
# Démarre uniquement la base de données
docker compose up database

# Démarre la base de données et le cache
docker compose up database cache
```

**Attention** : Les dépendances (définies avec `depends_on`) sont aussi démarrées automatiquement.

### Forcer la recréation des conteneurs

```bash
docker compose up --force-recreate
```

Utile quand vous voulez être sûr que les conteneurs sont recréés, même s'il n'y a pas eu de changements.

### Ne pas recréer les conteneurs existants

```bash
docker compose up --no-recreate
```

Démarre seulement les services qui n'existent pas, sans toucher aux autres.

### Options utiles combinées

```bash
# Démarrer en arrière-plan avec reconstruction
docker compose up -d --build

# Forcer la recréation en mode détaché
docker compose up -d --force-recreate

# Démarrer avec un timeout personnalisé
docker compose up -d --timeout 30
```

## 2. docker compose down - Arrêter l'application

Cette commande arrête et supprime tous les conteneurs, réseaux créés par Docker Compose.

### Utilisation de base

```bash
docker compose down
```

**Ce qui se passe** :
1. Arrête tous les conteneurs
2. Supprime tous les conteneurs
3. Supprime les réseaux créés par Compose
4. **NE supprime PAS** les volumes par défaut

```bash
# Exemple de sortie
[+] Running 4/4
 ✔ Container mon-projet-app-1       Removed
 ✔ Container mon-projet-cache-1     Removed
 ✔ Container mon-projet-database-1  Removed
 ✔ Network mon-projet_default       Removed
```

### Supprimer aussi les volumes

Pour supprimer également les volumes (⚠️ ATTENTION : vous perdrez vos données) :

```bash
docker compose down -v
```

**-v** ou **--volumes** supprime les volumes nommés définis dans la section `volumes` du fichier.

**⚠️ DANGER** : Cette commande supprime vos données de base de données ! À utiliser avec précaution.

### Supprimer les images

Pour supprimer aussi les images créées :

```bash
docker compose down --rmi all
```

Options pour **--rmi** :
- **all** : Supprime toutes les images utilisées par les services
- **local** : Supprime seulement les images sans tag personnalisé

### Supprimer les conteneurs orphelins

```bash
docker compose down --remove-orphans
```

Les "orphelins" sont des conteneurs d'une ancienne version de votre `docker-compose.yml` qui ne sont plus définis.

### Combinaison puissante (nettoyage complet)

```bash
# ATTENTION : Supprime TOUT (conteneurs, réseaux, volumes, images)
docker compose down -v --rmi all --remove-orphans
```

Utilisez cette commande quand vous voulez vraiment repartir de zéro !

## 3. docker compose logs - Voir les logs

Les logs sont essentiels pour comprendre ce qui se passe dans vos conteneurs et pour déboguer.

### Voir tous les logs

```bash
docker compose logs
```

Affiche tous les logs de tous les services (historique complet).

### Suivre les logs en temps réel

```bash
docker compose logs -f
```

**-f** signifie "follow" (suivre). Les nouveaux logs apparaissent en temps réel. Appuyez sur `Ctrl+C` pour arrêter.

### Logs d'un service spécifique

```bash
# Logs d'un seul service
docker compose logs api

# Logs de plusieurs services
docker compose logs api database
```

### Suivre les logs d'un service

```bash
docker compose logs -f api
```

Très utile pour déboguer un service spécifique !

### Limiter le nombre de lignes

```bash
# Affiche les 100 dernières lignes
docker compose logs --tail=100

# Affiche les 50 dernières lignes et suit ensuite
docker compose logs -f --tail=50
```

Sans l'option `--tail`, tous les logs depuis le démarrage sont affichés.

### Ajouter les timestamps

```bash
docker compose logs -t
```

**-t** ou **--timestamps** ajoute l'horodatage à chaque ligne de log.

```bash
# Exemple de sortie avec timestamps
api-1  | 2025-10-22T10:30:45.123456Z Server started on port 3000
```

### Afficher depuis un moment précis

```bash
# Logs depuis il y a 1 heure
docker compose logs --since 1h

# Logs depuis il y a 30 minutes
docker compose logs --since 30m

# Logs depuis une date précise
docker compose logs --since 2025-10-22T10:00:00
```

### Logs jusqu'à un moment précis

```bash
docker compose logs --until 2h
```

### Combinaison d'options utiles

```bash
# Les 100 dernières lignes en temps réel avec timestamps
docker compose logs -f --tail=100 -t

# Logs d'un service depuis 1h avec timestamps
docker compose logs api --since 1h -t

# Suivre les logs de plusieurs services
docker compose logs -f api worker
```

## 4. docker compose ps - Lister les conteneurs

Cette commande affiche l'état des services/conteneurs de votre application.

### Utilisation de base

```bash
docker compose ps
```

**Exemple de sortie** :
```
NAME                    IMAGE           COMMAND                  SERVICE     STATUS        PORTS  
mon-projet-database-1   postgres:15     "docker-entrypoint.s…"   database    Up 2 hours    5432/tcp  
mon-projet-cache-1      redis:7         "docker-entrypoint.s…"   cache       Up 2 hours    6379/tcp  
mon-projet-app-1        mon-app:latest  "node server.js"         app         Up 2 hours    0.0.0.0:3000->3000/tcp  
```

**Informations affichées** :
- **NAME** : Nom du conteneur
- **IMAGE** : Image utilisée
- **COMMAND** : Commande exécutée
- **SERVICE** : Nom du service (dans docker-compose.yml)
- **STATUS** : État (Up, Exited, etc.) et depuis combien de temps
- **PORTS** : Ports exposés

### Afficher tous les conteneurs (même arrêtés)

```bash
docker compose ps -a
```

**-a** ou **--all** montre aussi les conteneurs arrêtés.

### Format compact

```bash
docker compose ps --format table
```

### Filtrer les services

```bash
# Afficher seulement un service
docker compose ps app

# Afficher plusieurs services
docker compose ps app database
```

### Voir uniquement les IDs

```bash
docker compose ps -q
```

**-q** ou **--quiet** affiche seulement les IDs des conteneurs (utile pour les scripts).

### Afficher les services avec leurs états

```bash
docker compose ps --services
```

Liste uniquement les noms des services (sans les détails).

## 5. Autres commandes essentielles

### docker compose start - Démarrer des conteneurs existants

```bash
# Démarre tous les conteneurs (qui ont été arrêtés)
docker compose start

# Démarre un service spécifique
docker compose start api
```

**Différence avec up** :
- **start** : Démarre des conteneurs qui existent déjà (créés auparavant)
- **up** : Crée ET démarre les conteneurs

### docker compose stop - Arrêter les conteneurs

```bash
# Arrête tous les conteneurs (sans les supprimer)
docker compose stop

# Arrête un service spécifique
docker compose stop database

# Arrête avec un timeout personnalisé
docker compose stop -t 30
```

**Différence avec down** :
- **stop** : Arrête les conteneurs mais les garde (vous pouvez les redémarrer avec `start`)
- **down** : Arrête ET supprime les conteneurs

### docker compose restart - Redémarrer les services

```bash
# Redémarre tous les services
docker compose restart

# Redémarre un service spécifique
docker compose restart api

# Redémarre avec un timeout
docker compose restart -t 10
```

Équivalent à `stop` puis `start`.

### docker compose pause / unpause - Mettre en pause

```bash
# Met en pause tous les conteneurs (gel des processus)
docker compose pause

# Remet en route
docker compose unpause

# Pour un service spécifique
docker compose pause database  
docker compose unpause database  
```

**Différence avec stop** :
- **pause** : Gèle les processus (utilise moins de ressources, redémarrage instantané)
- **stop** : Arrête complètement les conteneurs

### docker compose exec - Exécuter une commande dans un conteneur

```bash
# Ouvrir un shell dans un conteneur
docker compose exec api sh  
docker compose exec api bash  

# Exécuter une commande spécifique
docker compose exec database psql -U postgres

# Exécuter en tant qu'un utilisateur spécifique
docker compose exec -u root api sh
```

**Options utiles** :
- **-T** : Désactive l'allocation de pseudo-TTY (utile pour les scripts)
- **-e** : Définit des variables d'environnement
- **--index** : Spécifie l'instance si plusieurs conteneurs du même service

```bash
# Exemples pratiques
docker compose exec api npm install  
docker compose exec database pg_dump -U postgres mydb > backup.sql  
docker compose exec -e DEBUG=true api node debug.js  
```

### docker compose run - Créer et exécuter un conteneur temporaire

```bash
# Exécuter une commande dans un nouveau conteneur
docker compose run api npm test

# Avec suppression automatique après exécution
docker compose run --rm api npm test

# Sans démarrer les services liés
docker compose run --no-deps api npm install
```

**Différence avec exec** :
- **exec** : Exécute dans un conteneur existant qui tourne
- **run** : Crée un nouveau conteneur temporaire

### docker compose build - Construire les images

```bash
# Construit toutes les images
docker compose build

# Construit une image spécifique
docker compose build api

# Construit sans utiliser le cache
docker compose build --no-cache

# Construit avec des arguments
docker compose build --build-arg NODE_ENV=production
```

### docker compose pull - Télécharger les images

```bash
# Télécharge toutes les images
docker compose pull

# Télécharge une image spécifique
docker compose pull database

# Télécharge en silence (sans afficher les détails)
docker compose pull -q
```

### docker compose push - Pousser les images

```bash
# Pousse toutes les images vers le registry
docker compose push

# Pousse une image spécifique
docker compose push api
```

Utile si vous avez construit des images que vous voulez partager sur Docker Hub ou un registry privé.

### docker compose config - Valider et afficher la configuration

```bash
# Affiche la configuration finale (avec valeurs par défaut)
docker compose config

# Affiche uniquement les services
docker compose config --services

# Affiche uniquement les volumes
docker compose config --volumes

# Valide sans afficher
docker compose config -q
```

Très utile pour déboguer votre fichier `docker-compose.yml` !

### docker compose top - Voir les processus

```bash
# Affiche les processus de tous les conteneurs
docker compose top

# Pour un service spécifique
docker compose top api
```

### docker compose images - Lister les images

```bash
docker compose images
```

Affiche toutes les images utilisées par les services.

### docker compose version - Voir la version

```bash
docker compose version
```

Affiche la version de Docker Compose installée.

## Workflows courants

### Workflow de développement quotidien

```bash
# Matin : Démarrer l'environnement
docker compose up -d

# Voir les logs si besoin
docker compose logs -f api

# Redémarrer un service après modification
docker compose restart api

# Accéder au conteneur pour déboguer
docker compose exec api sh

# Soir : Arrêter l'environnement
docker compose stop
```

### Déployer une nouvelle version

```bash
# 1. Télécharger les nouvelles images
docker compose pull

# 2. Arrêter et supprimer les anciens conteneurs
docker compose down

# 3. Démarrer avec les nouvelles images
docker compose up -d

# 4. Vérifier que tout fonctionne
docker compose ps  
docker compose logs -f  
```

### Reconstruction complète

```bash
# 1. Tout arrêter et nettoyer
docker compose down -v

# 2. Reconstruire les images
docker compose build --no-cache

# 3. Démarrer
docker compose up -d

# 4. Vérifier
docker compose logs -f
```

### Debugging d'un problème

```bash
# 1. Voir l'état des services
docker compose ps

# 2. Voir les logs récents
docker compose logs --tail=100 -t

# 3. Suivre les logs en temps réel
docker compose logs -f api

# 4. Accéder au conteneur
docker compose exec api sh

# 5. Redémarrer le service problématique
docker compose restart api
```

### Backup de la base de données

```bash
# Exporter la base de données
docker compose exec -T database pg_dump -U postgres mydb > backup.sql

# Restaurer la base de données
cat backup.sql | docker compose exec -T database psql -U postgres mydb
```

## Commandes avec fichiers spécifiques

Par défaut, Docker Compose cherche un fichier `docker-compose.yml` dans le répertoire courant. Vous pouvez spécifier un autre fichier :

```bash
# Utiliser un fichier spécifique
docker compose -f docker-compose.prod.yml up -d

# Utiliser plusieurs fichiers (ils sont combinés)
docker compose -f docker-compose.yml -f docker-compose.override.yml up -d

# Avec un nom de projet personnalisé
docker compose -p mon-projet up -d
```

**Exemples d'organisation** :
```
projet/
├── docker-compose.yml           # Configuration de base
├── docker-compose.dev.yml       # Overrides pour le dev
├── docker-compose.prod.yml      # Overrides pour la prod
└── docker-compose.test.yml      # Pour les tests
```

Utilisation :
```bash
# Développement
docker compose -f docker-compose.yml -f docker-compose.dev.yml up

# Production
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# Tests
docker compose -f docker-compose.yml -f docker-compose.test.yml run tests
```

## Commandes pour surveiller les ressources

### Voir la consommation de ressources

```bash
# Utiliser Docker stats (pas Compose, mais utile)
docker stats

# Pour un conteneur spécifique
docker stats mon-projet-api-1
```

Affiche CPU, mémoire, réseau, I/O en temps réel.

### Voir les événements

```bash
# Suivre les événements Docker Compose
docker compose events

# Avec filtre JSON
docker compose events --json
```

## Raccourcis et astuces

### Aliases utiles

Ajoutez ces aliases à votre `.bashrc` ou `.zshrc` :

```bash
# Aliases pour Docker Compose
alias dc='docker compose'  
alias dcu='docker compose up'  
alias dcud='docker compose up -d'  
alias dcd='docker compose down'  
alias dcl='docker compose logs'  
alias dclf='docker compose logs -f'  
alias dcp='docker compose ps'  
alias dcr='docker compose restart'  
alias dce='docker compose exec'  
```

Utilisation :
```bash
dcu          # au lieu de docker compose up  
dclf api     # au lieu de docker compose logs -f api  
dce api sh   # au lieu de docker compose exec api sh  
```

### Commandes combinées

```bash
# Arrêter, reconstruire et redémarrer
docker compose down && docker compose build && docker compose up -d

# Nettoyer et redémarrer proprement
docker compose down -v && docker compose up -d --build

# Voir les logs des 5 dernières minutes
docker compose logs --since 5m
```

## Gestion des erreurs courantes

### "Cannot connect to Docker daemon"

```bash
# Vérifier que Docker est lancé
docker version

# Sur Linux, vérifier les permissions
sudo usermod -aG docker $USER
# Puis se déconnecter/reconnecter
```

### "No such service"

```bash
# Vérifier que vous êtes dans le bon répertoire
pwd  
ls -la docker-compose.yml  

# Vérifier les noms des services
docker compose config --services
```

### "Port is already in use"

```bash
# Trouver quel processus utilise le port
sudo lsof -i :8080

# Ou changer le port dans docker-compose.yml
```

### "Service not running"

```bash
# Vérifier l'état
docker compose ps

# Voir les logs pour comprendre pourquoi
docker compose logs service-name

# Redémarrer
docker compose restart service-name
```

## Points clés à retenir

✅ **docker compose up** démarre tous les services (utilisez `-d` pour l'arrière-plan)

✅ **docker compose down** arrête et supprime les conteneurs (ajoutez `-v` pour supprimer les volumes)

✅ **docker compose logs** affiche les logs (utilisez `-f` pour suivre en temps réel)

✅ **docker compose ps** liste les conteneurs et leur état

✅ **docker compose exec** permet d'exécuter des commandes dans un conteneur en cours d'exécution

✅ **docker compose restart** redémarre les services rapidement

✅ **docker compose build** reconstruit les images

✅ Utilisez **--help** sur n'importe quelle commande pour voir toutes les options

✅ Les commandes peuvent être combinées avec des options pour plus de contrôle

✅ Créez des aliases pour gagner du temps avec les commandes fréquentes

---

**Félicitations !** Vous maîtrisez maintenant les commandes essentielles de Docker Compose. Avec ces outils, vous pouvez gérer efficacement vos applications conteneurisées au quotidien ! 🎉

**Section suivante** : Nous allons explorer d'autres aspects avancés de Docker Compose et des cas d'usage pratiques !

⏭️ [Gestion des registres](/09-gestion-des-registres/README.md)
