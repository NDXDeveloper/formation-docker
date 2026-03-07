🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 4.2 Lancer et gérer des conteneurs (run, start, stop, restart)

## Introduction

Créer et gérer des conteneurs est **l'essence même de Docker**. Dans cette section, vous allez apprendre les commandes fondamentales qui vous permettront de contrôler le cycle de vie de vos conteneurs : les créer, les démarrer, les arrêter et les redémarrer.

Ces quatre commandes (`run`, `start`, `stop`, `restart`) sont celles que vous utiliserez le plus souvent dans votre travail quotidien avec Docker.

## La commande `docker run` : créer et démarrer un conteneur

### Qu'est-ce que `docker run` ?

`docker run` est la commande la plus importante de Docker. Elle effectue **deux actions en une seule commande** :
1. Elle **crée** un nouveau conteneur à partir d'une image
2. Elle **démarre** immédiatement ce conteneur

### Syntaxe de base

```bash
docker run <nom_de_l_image>
```

### Premier exemple : Hello World

Testons avec l'image la plus simple de Docker :

```bash
docker run hello-world
```

Cette commande va :
1. Chercher l'image `hello-world` localement
2. Si elle n'existe pas, la télécharger depuis Docker Hub
3. Créer un conteneur à partir de cette image
4. Exécuter le conteneur (qui affiche un message de bienvenue)
5. Arrêter automatiquement le conteneur une fois la tâche terminée

### Exemple pratique : serveur web Nginx

Lançons un serveur web Nginx :

```bash
docker run nginx
```

Vous remarquerez que votre terminal semble "bloqué". C'est normal ! Le conteneur s'exécute en **mode attaché** (au premier plan) et affiche ses logs en temps réel. Pour arrêter le conteneur, appuyez sur `Ctrl+C`.

### Le mode détaché avec `-d`

Pour que le conteneur s'exécute en **arrière-plan** (mode détaché) sans bloquer votre terminal :

```bash
docker run -d nginx
```

Docker affichera alors l'**ID du conteneur** (une longue chaîne de caractères) :
```
f4d5e8a9b2c1d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8
```

Votre terminal redevient disponible, et le conteneur continue de fonctionner en arrière-plan.

### Nommer un conteneur avec `--name`

Par défaut, Docker attribue un nom aléatoire à chaque conteneur (comme "clever_einstein" ou "hungry_tesla"). Pour faciliter la gestion, donnez un nom explicite à vos conteneurs :

```bash
docker run -d --name mon-serveur-web nginx
```

Vous pourrez ensuite référencer ce conteneur facilement par son nom au lieu de son ID.

### Publier des ports avec `-p`

Les conteneurs sont isolés du réseau de votre machine. Pour accéder à un service dans le conteneur (comme un serveur web), vous devez **publier ses ports** :

```bash
docker run -d -p 8080:80 --name mon-nginx nginx
```

Syntaxe : `-p <port_hôte>:<port_conteneur>`

- `8080` : le port de votre machine (hôte)
- `80` : le port à l'intérieur du conteneur

Vous pouvez maintenant accéder à Nginx via votre navigateur à l'adresse `http://localhost:8080`

### Supprimer automatiquement avec `--rm`

Par défaut, un conteneur arrêté reste présent sur votre système. L'option `--rm` supprime automatiquement le conteneur une fois qu'il s'arrête :

```bash
docker run --rm nginx
```

C'est utile pour les tests rapides et éviter l'accumulation de conteneurs inutilisés.

### Mode interactif avec `-it`

Pour interagir directement avec un conteneur (par exemple, ouvrir un terminal dans un conteneur Ubuntu) :

```bash
docker run -it ubuntu bash
```

- `-i` : mode interactif (garde l'entrée standard ouverte)
- `-t` : alloue un pseudo-terminal

Vous vous retrouvez alors à l'intérieur du conteneur Ubuntu avec un shell bash. Tapez `exit` pour en sortir.

### Variables d'environnement avec `-e`

Vous pouvez passer des variables d'environnement au conteneur :

```bash
docker run -d -e MYSQL_ROOT_PASSWORD=monsecret -e MYSQL_DATABASE=mabase mysql
```

### Exemple complet

Voici une commande `docker run` complète qui combine plusieurs options :

```bash
docker run -d \
  --name mon-application \
  -p 3000:3000 \
  -e NODE_ENV=production \
  -e API_KEY=ma-cle-secrete \
  --rm \
  node-app:latest
```

Cette commande :
- Lance le conteneur en arrière-plan (`-d`)
- Le nomme "mon-application" (`--name`)
- Mappe le port 3000 (`-p`)
- Définit des variables d'environnement (`-e`)
- Le supprimera automatiquement à l'arrêt (`--rm`)
- Utilise l'image `node-app` avec le tag `latest`

## La commande `docker stop` : arrêter un conteneur

### Qu'est-ce que `docker stop` ?

`docker stop` arrête **proprement** un conteneur en cours d'exécution. Docker envoie d'abord un signal SIGTERM (demande d'arrêt gracieux), puis attend 10 secondes avant d'envoyer un SIGKILL (arrêt forcé) si nécessaire.

### Syntaxe

```bash
docker stop <nom_ou_id_conteneur>
```

### Exemples

Arrêter un conteneur par son nom :
```bash
docker stop mon-nginx
```

Arrêter un conteneur par son ID (les premiers caractères suffisent) :
```bash
docker stop f4d5e8a9
```

Arrêter plusieurs conteneurs en une seule commande :
```bash
docker stop conteneur1 conteneur2 conteneur3
```

### Modifier le délai d'attente

Si votre application nécessite plus de temps pour s'arrêter proprement :

```bash
docker stop -t 30 mon-conteneur
```

L'option `-t` spécifie le délai en secondes (ici 30 secondes au lieu de 10).

## La commande `docker start` : redémarrer un conteneur arrêté

### Qu'est-ce que `docker start` ?

`docker start` démarre un conteneur qui a été **préalablement créé mais qui est arrêté**. Contrairement à `docker run`, cette commande ne crée pas de nouveau conteneur.

### Syntaxe

```bash
docker start <nom_ou_id_conteneur>
```

### Exemples

Démarrer un conteneur arrêté :
```bash
docker start mon-nginx
```

Démarrer et attacher le terminal aux logs (mode non détaché) :
```bash
docker start -a mon-nginx
```

### Différence cruciale entre `run` et `start`

C'est une source de confusion fréquente pour les débutants :

| Commande | Action | Quand l'utiliser |
|----------|--------|------------------|
| `docker run` | **Crée** un nouveau conteneur et le démarre | Première utilisation d'une image |
| `docker start` | **Démarre** un conteneur existant qui est arrêté | Redémarrer un conteneur que vous avez déjà créé |

**Analogie** :
- `docker run` = acheter une nouvelle voiture et la démarrer
- `docker start` = démarrer votre voiture existante qui était garée

### Exemple pratique

```bash
# Première fois : création et démarrage
docker run -d --name mon-app nginx

# Arrêt du conteneur
docker stop mon-app

# Redémarrage du conteneur existant
docker start mon-app

# Si vous faites "docker run" à nouveau, cela créerait un NOUVEAU conteneur
# et échouerait car le nom "mon-app" est déjà pris
```

## La commande `docker restart` : redémarrer un conteneur

### Qu'est-ce que `docker restart` ?

`docker restart` est un raccourci qui combine `docker stop` puis `docker start`. Elle redémarre un conteneur en cours d'exécution.

### Syntaxe

```bash
docker restart <nom_ou_id_conteneur>
```

### Exemples

Redémarrer un conteneur :
```bash
docker restart mon-nginx
```

Redémarrer avec un délai personnalisé :
```bash
docker restart -t 20 mon-conteneur
```

### Quand utiliser `restart` ?

- Après avoir modifié un fichier de configuration
- Quand une application est instable et nécessite un redémarrage
- Pour appliquer certains changements d'environnement

## Résumé des commandes et de leur cycle de vie

Voici le cycle de vie typique d'un conteneur avec les commandes associées :

```
┌─────────────────────────────────────────┐
│  1. Créer et démarrer : docker run      │
└────────────────┬────────────────────────┘
                 │
                 ▼
         ┌───────────────┐
         │   RUNNING     │  ◄──┐
         │  (en cours)   │     │
         └───────┬───────┘     │
                 │             │
                 │             │
  docker stop    │             │ docker start
                 │             │ docker restart
                 ▼             │
         ┌───────────────┐     │
         │   STOPPED     │  ───┘
         │   (arrêté)    │
         └───────────────┘
                 │
                 │ docker rm
                 ▼
         ┌───────────────┐
         │   DELETED     │
         │  (supprimé)   │
         └───────────────┘
```

## Options importantes à connaître

Voici un tableau récapitulatif des options les plus utiles pour `docker run` :

| Option | Description | Exemple |
|--------|-------------|---------|
| `-d` | Mode détaché (arrière-plan) | `docker run -d nginx` |
| `--name` | Donner un nom au conteneur | `docker run --name web nginx` |
| `-p` | Publier un port | `docker run -p 8080:80 nginx` |
| `-e` | Variable d'environnement | `docker run -e VAR=valeur nginx` |
| `-it` | Mode interactif avec terminal | `docker run -it ubuntu bash` |
| `--rm` | Supprimer automatiquement après l'arrêt | `docker run --rm nginx` |
| `-v` | Monter un volume | `docker run -v /data:/app/data nginx` |
| `--network` | Connecter à un réseau | `docker run --network mon-reseau nginx` |

## Commandes supplémentaires utiles

### Arrêter tous les conteneurs en cours

```bash
docker stop $(docker ps -q)
```

Explication :
- `docker ps -q` liste les IDs de tous les conteneurs en cours
- Ces IDs sont passés à `docker stop`

### Redémarrer tous les conteneurs arrêtés

```bash
docker start $(docker ps -aq -f status=exited)
```

## Bonnes pratiques

### 1. Toujours nommer vos conteneurs importants

```bash
# ✅ Bon
docker run -d --name mon-app-prod nginx

# ❌ Éviter (nom aléatoire difficile à retrouver)
docker run -d nginx
```

### 2. Utiliser `--rm` pour les tests

```bash
# Pour les tests rapides
docker run --rm -p 8080:80 nginx
```

### 3. Publier les ports explicitement

```bash
# ✅ Clair et contrôlé
docker run -d -p 8080:80 nginx

# ❌ Éviter de mapper tous les ports automatiquement
docker run -d -P nginx
```

### 4. Utiliser des tags spécifiques en production

```bash
# ✅ Bon (version fixée)
docker run -d nginx:1.27.3

# ❌ Éviter en production (version changeante)
docker run -d nginx:latest
```

### 5. Grouper les options pour la lisibilité

```bash
docker run -d \
  --name mon-application \
  -p 3000:3000 \
  -e NODE_ENV=production \
  --restart unless-stopped \
  mon-image:v1.0
```

## Dépannage courant

### "Conflict. The container name is already in use"

**Problème** : Vous essayez de créer un conteneur avec un nom déjà utilisé.

**Solution** :
```bash
# Supprimer l'ancien conteneur
docker rm mon-conteneur

# Ou utiliser un nom différent
docker run --name mon-conteneur-v2 nginx
```

### "Cannot connect to the Docker daemon"

**Problème** : Docker n'est pas démarré ou vous n'avez pas les permissions.

**Solution** :
- Vérifiez que Docker Desktop est lancé (Windows/Mac)
- Sur Linux, démarrez le service : `sudo systemctl start docker`
- Vérifiez vos permissions : ajoutez votre utilisateur au groupe docker

### "Port is already allocated"

**Problème** : Le port que vous essayez d'utiliser est déjà occupé.

**Solution** :
```bash
# Utiliser un autre port
docker run -d -p 8081:80 nginx

# Ou arrêter le conteneur qui utilise ce port
docker stop <conteneur_utilisant_le_port>
```

## Résumé

Dans cette section, vous avez appris à :

- ✅ **Créer et démarrer** des conteneurs avec `docker run`
- ✅ **Arrêter** des conteneurs proprement avec `docker stop`
- ✅ **Redémarrer** des conteneurs existants avec `docker start`
- ✅ **Redémarrer** rapidement avec `docker restart`
- ✅ Comprendre la **différence entre run et start**
- ✅ Utiliser les options essentielles : `-d`, `--name`, `-p`, `-e`, `-it`, `--rm`
- ✅ Appliquer les **bonnes pratiques** pour une gestion efficace

Vous maîtrisez maintenant les commandes fondamentales pour contrôler vos conteneurs Docker. Dans la prochaine section, vous apprendrez à lister et inspecter vos conteneurs pour obtenir des informations détaillées sur leur état.

⏭️ [Lister et inspecter (ps, images, inspect)](/04-commandes-docker-essentielles/03-lister-et-inspecter.md)
