🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 2.3 Vérification de l'installation

## Introduction

Docker est installé et configuré. Maintenant, avant de commencer à créer des conteneurs et à développer avec Docker, il est essentiel de vérifier que tout fonctionne correctement. Cette étape de vérification peut vous faire gagner beaucoup de temps en détectant d'éventuels problèmes maintenant plutôt que plus tard.

Cette section vous guide à travers une série de tests progressifs qui valideront votre installation. Chaque test vérifie un aspect différent de Docker, et nous expliquerons ce que signifie chaque résultat.

Ne sautez pas cette étape ! Même si l'installation s'est déroulée sans erreur apparente, ces vérifications garantissent que tout est réellement opérationnel.

## Pourquoi vérifier l'installation ?

### Détecter les problèmes tôt

Un problème de configuration détecté maintenant sera beaucoup plus facile à résoudre qu'un bug mystérieux qui apparaîtra plus tard dans un projet complexe.

### Comprendre votre environnement

Ces tests vous permettent de voir concrètement comment Docker fonctionne et vous familiariser avec les commandes de base avant d'aller plus loin.

### Gagner en confiance

Une fois tous les tests passés, vous saurez avec certitude que votre environnement Docker est prêt. Vous pourrez avancer sereinement dans votre apprentissage.

## Préparation aux tests

Avant de commencer, assurez-vous que :

1. **Docker est démarré**
   - Sur Linux : `sudo systemctl status docker` doit montrer "active (running)"
   - Sur Windows/macOS : Docker Desktop doit afficher "Docker is running"

2. **Vous avez un terminal ouvert**
   - Linux/macOS : Terminal ou votre shell préféré
   - Windows : PowerShell, Command Prompt, ou WSL

3. **Vous avez une connexion internet**
   - Certains tests téléchargeront de petites images depuis Docker Hub

Si vous êtes prêt, commençons les vérifications !

## Test 1 : Vérification de la version Docker

### Objectif
Ce premier test confirme que la commande `docker` est accessible et affiche la version installée.

### Commande à exécuter

```bash
docker --version
```

### Résultat attendu

Vous devriez voir quelque chose comme :
```
Docker version 24.0.6, build ed223bc
```

Le numéro de version peut varier (24.x, 25.x, etc.), c'est normal. L'important est que la commande s'exécute sans erreur.

### Interprétation

**✅ Si vous voyez la version :** Docker CLI (client) est correctement installé et accessible depuis votre terminal.

**❌ Si vous obtenez "command not found" ou "docker is not recognized" :**
- Sur Linux : Docker n'est pas installé ou pas dans le PATH
- Sur Windows : Docker Desktop n'est peut-être pas démarré, ou vous devez redémarrer votre terminal
- Solution : Vérifiez l'installation et redémarrez votre terminal

### Aller plus loin

Pour voir des informations plus détaillées sur votre installation :

```bash
docker version
```

Cette commande affiche :
- La version du **Client** (docker CLI)
- La version du **Server** (Docker Daemon)
- La version de l'**API**
- La version de **Go** utilisée pour compiler Docker

Exemple de sortie :
```
Client:
 Version:           24.0.6
 API version:       1.43
 Go version:        go1.20.7
 OS/Arch:           linux/amd64

Server: Docker Engine
 Version:          24.0.6
 API version:      1.43 (minimum version 1.12)
 Go version:       go1.20.7
 OS/Arch:          linux/amd64
```

Si vous voyez à la fois Client et Server, c'est parfait !

## Test 2 : Vérification de la connexion au Docker Daemon

### Objectif
Confirmer que le Docker Client peut communiquer avec le Docker Daemon.

### Commande à exécuter

```bash
docker info
```

### Résultat attendu

Cette commande affiche une longue liste d'informations système. Voici les éléments clés à vérifier :

```
Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 24.0.6
 Storage Driver: overlay2
 Kernel Version: 5.15.0
 Operating System: Ubuntu 22.04.3 LTS
 OSType: linux
 Architecture: x86_64
 CPUs: 4
 Total Memory: 7.76GiB
 Docker Root Dir: /var/lib/docker
```

### Interprétation

**✅ Si vous voyez toutes ces informations :** Excellent ! Le Docker Daemon fonctionne et le Client peut communiquer avec lui.

**Points importants à noter :**
- **Containers** : Le nombre de conteneurs actuels (probablement 0 pour une installation neuve)
- **Images** : Le nombre d'images locales (probablement 0 aussi)
- **Storage Driver** : Généralement `overlay2` (le plus courant et performant)
- **CPUs** et **Total Memory** : Les ressources disponibles pour Docker

**❌ Si vous obtenez "Cannot connect to the Docker daemon" :**
- **Linux** : Le service Docker n'est pas démarré
  ```bash
  sudo systemctl start docker
  ```
- **Windows/macOS** : Docker Desktop n'est pas démarré
  - Lancez Docker Desktop depuis vos applications
- **Linux - Problème de permissions** : Vous n'êtes peut-être pas dans le groupe docker
  ```bash
  sudo usermod -aG docker $USER
  newgrp docker
  ```

## Test 3 : Exécution du conteneur Hello World

### Objectif
Ce test est le plus important. Il vérifie que Docker peut télécharger une image, créer un conteneur, et l'exécuter correctement.

### Commande à exécuter

```bash
docker run hello-world
```

### Ce qui se passe en coulisses

Lorsque vous exécutez cette commande, Docker effectue plusieurs opérations :

1. **Recherche locale** : Docker cherche l'image `hello-world` sur votre machine
2. **Téléchargement** : Comme l'image n'existe pas localement, Docker la télécharge depuis Docker Hub
3. **Création** : Docker crée un conteneur à partir de cette image
4. **Exécution** : Le conteneur démarre et exécute son programme
5. **Affichage** : Le message est affiché dans votre terminal
6. **Arrêt** : Le conteneur s'arrête automatiquement

### Résultat attendu

```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
c1ec31eb5944: Pull complete
Digest: sha256:d211f485f2dd1dee407a80973c8f129f00d54604d2c90732e8e320e5038a0348
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

### Interprétation

**✅ Si vous voyez "Hello from Docker!" :** Parfait ! Votre installation Docker est totalement fonctionnelle. Vous avez :
- Téléchargé votre première image ✓
- Créé votre premier conteneur ✓
- Exécuté votre premier conteneur ✓
- Reçu la sortie du conteneur ✓

C'est un moment important : vous venez d'exécuter votre premier conteneur Docker !

**❌ Erreurs possibles :**

**"Cannot connect to the Docker daemon"**
- Docker n'est pas démarré (voir Test 2)

**"Error response from daemon: Get https://registry-1.docker.io/v2/: dial tcp: lookup registry-1.docker.io"**
- Problème de connexion internet ou de DNS
- Vérifiez votre connexion
- Sur certains réseaux d'entreprise, vous devrez peut-être configurer un proxy

**"permission denied while trying to connect to the Docker daemon socket"**
- Problème de permissions sur Linux
- Solution temporaire : `sudo docker run hello-world`
- Solution permanente : Ajoutez-vous au groupe docker (voir section 2.2)

## Test 4 : Vérification de la liste des conteneurs

### Objectif
Vérifier que Docker garde bien trace des conteneurs créés.

### Commandes à exécuter

```bash
# Voir les conteneurs en cours d'exécution
docker ps

# Voir tous les conteneurs (y compris arrêtés)
docker ps -a
```

### Résultat attendu

**Pour `docker ps` :**
```
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

Une liste vide (juste l'en-tête) est normal : le conteneur `hello-world` s'est arrêté automatiquement après avoir affiché son message.

**Pour `docker ps -a` :**
```
CONTAINER ID   IMAGE         COMMAND    CREATED         STATUS                     PORTS     NAMES
a1b2c3d4e5f6   hello-world   "/hello"   2 minutes ago   Exited (0) 2 minutes ago             zealous_tesla
```

### Interprétation

Vous devriez voir votre conteneur `hello-world` avec :
- **CONTAINER ID** : Un identifiant unique (aléatoire)
- **IMAGE** : `hello-world`
- **STATUS** : `Exited (0)` (0 signifie que tout s'est bien passé)
- **NAMES** : Un nom aléatoire généré par Docker (par exemple `zealous_tesla`)

**✅ Si vous voyez votre conteneur :** Docker enregistre bien tous les conteneurs créés, même ceux qui sont arrêtés.

**Note :** Le conteneur reste sur votre système même s'il est arrêté. Pour le supprimer :
```bash
docker rm <CONTAINER_ID ou NAME>
```

Ou pour tout supprimer d'un coup :
```bash
docker container prune
```

## Test 5 : Vérification des images

### Objectif
Confirmer que l'image téléchargée est bien stockée localement.

### Commande à exécuter

```bash
docker images
```

Ou la forme équivalente :
```bash
docker image ls
```

### Résultat attendu

```
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    d2c94e258dcb   7 months ago   13.3kB
```

### Interprétation

Vous devriez voir l'image `hello-world` avec :
- **REPOSITORY** : Le nom de l'image (`hello-world`)
- **TAG** : La version (`latest` par défaut)
- **IMAGE ID** : Un identifiant unique
- **SIZE** : La taille de l'image (13.3 kB, elle est minuscule !)

**✅ Si vous voyez l'image :** Docker a bien téléchargé et stocké l'image localement. Les prochaines fois que vous exécuterez `docker run hello-world`, ce sera instantané car l'image est déjà présente.

**Points à comprendre :**
- Les images sont stockées localement et réutilisables
- Vous ne téléchargez une image qu'une seule fois
- Plusieurs conteneurs peuvent être créés à partir de la même image

## Test 6 : Exécution d'un conteneur interactif

### Objectif
Tester un conteneur plus complexe qui permet d'interagir avec un shell.

### Commande à exécuter

```bash
docker run -it ubuntu bash
```

**Décomposition de la commande :**
- `docker run` : Exécute un conteneur
- `-it` : Mode interactif avec un terminal
  - `-i` : Keep STDIN open (interactif)
  - `-t` : Allocate a pseudo-TTY (terminal)
- `ubuntu` : Nom de l'image
- `bash` : Commande à exécuter dans le conteneur

### Résultat attendu

Docker va d'abord télécharger l'image Ubuntu (environ 70 Mo), puis vous verrez un prompt de shell :

```
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
3153aa388d02: Pull complete
Digest: sha256:0bced47fffa3361afa981854fcabcd4577cd43cebbb808cea2b1f33a3dd7f508
Status: Downloaded newer image for ubuntu:latest
root@a1b2c3d4e5f6:/#
```

### Interprétation

**✅ Si vous voyez le prompt `root@...:/#` :** Vous êtes maintenant **à l'intérieur** du conteneur Ubuntu !

C'est un système Ubuntu complet, isolé, qui tourne dans un conteneur.

**Testez quelques commandes :**

```bash
# Voir le système d'exploitation
cat /etc/os-release

# Lister les fichiers
ls /

# Voir les processus en cours
ps aux

# Créer un fichier
echo "Hello from inside the container" > test.txt
cat test.txt
```

**Pour sortir du conteneur :**
```bash
exit
```

Ou tapez `Ctrl+D`.

Une fois sorti, le conteneur s'arrête automatiquement.

**Ce que ce test démontre :**
- Docker peut exécuter des systèmes complets (Ubuntu)
- Vous pouvez interagir avec un conteneur en temps réel
- Les conteneurs sont isolés (ce que vous faites dedans n'affecte pas votre système)

## Test 7 : Vérification des réseaux Docker

### Objectif
Confirmer que Docker a créé les réseaux par défaut.

### Commande à exécuter

```bash
docker network ls
```

### Résultat attendu

```
NETWORK ID     NAME      DRIVER    SCOPE
a1b2c3d4e5f6   bridge    bridge    local
g7h8i9j0k1l2   host      host      local
m3n4o5p6q7r8   none      null      local
```

### Interprétation

**✅ Si vous voyez ces trois réseaux :** Docker a correctement créé ses réseaux par défaut.

**Explication des réseaux :**
- **bridge** : Le réseau par défaut pour les conteneurs. Ils peuvent communiquer entre eux via ce réseau.
- **host** : Le conteneur partage la pile réseau de l'hôte (utilisation avancée).
- **none** : Aucun réseau. Le conteneur est totalement isolé.

Nous explorerons les réseaux Docker en détail dans le chapitre 7.

## Test 8 : Vérification des volumes Docker

### Objectif
S'assurer que Docker peut gérer des volumes (espaces de stockage persistants).

### Commande à exécuter

```bash
docker volume ls
```

### Résultat attendu

Pour une installation neuve :
```
DRIVER    VOLUME NAME
```

Une liste vide est normal. Aucun volume n'a été créé pour le moment.

**Créons un volume de test :**

```bash
docker volume create test-volume
```

Résultat :
```
test-volume
```

**Vérifions à nouveau :**
```bash
docker volume ls
```

Résultat :
```
DRIVER    VOLUME NAME
local     test-volume
```

**Inspectons le volume :**
```bash
docker volume inspect test-volume
```

Résultat :
```json
[
    {
        "CreatedAt": "2024-10-21T10:30:00Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/test-volume/_data",
        "Name": "test-volume",
        "Options": {},
        "Scope": "local"
    }
]
```

**Nettoyons le volume de test :**
```bash
docker volume rm test-volume
```

### Interprétation

**✅ Si vous avez pu créer et supprimer un volume :** Docker peut gérer le stockage persistant. C'est essentiel pour les bases de données et les applications qui ont besoin de conserver des données.

## Test 9 : Vérification de Docker Compose

### Objectif
Confirmer que Docker Compose est installé et fonctionnel.

### Commande à exécuter

```bash
docker compose version
```

**Note :** Sur les anciennes versions, c'était `docker-compose` (avec un tiret). Maintenant c'est `docker compose` (sans tiret), car Compose est un plugin intégré à Docker.

### Résultat attendu

```
Docker Compose version v2.23.0
```

Le numéro de version peut varier. L'important est que la commande fonctionne.

### Interprétation

**✅ Si vous voyez la version :** Docker Compose est installé. Vous pourrez orchestrer plusieurs conteneurs facilement (nous verrons cela au chapitre 8).

**❌ Si la commande échoue :**
- Sur Docker Desktop : Compose est normalement inclus, vérifiez votre installation
- Sur Linux avec Docker Engine : Installez le plugin compose
  ```bash
  sudo apt-get install docker-compose-plugin
  ```

## Test 10 : Test de performance basique

### Objectif
Vérifier que Docker peut gérer plusieurs conteneurs simultanément.

### Commandes à exécuter

**Lancer plusieurs conteneurs :**
```bash
docker run -d nginx
docker run -d nginx
docker run -d nginx
```

L'option `-d` (detached) lance les conteneurs en arrière-plan.

**Vérifier qu'ils tournent :**
```bash
docker ps
```

Vous devriez voir 3 conteneurs nginx en cours d'exécution.

**Stopper et supprimer tous les conteneurs :**
```bash
# Stopper tous les conteneurs en cours
docker stop $(docker ps -q)

# Supprimer tous les conteneurs arrêtés
docker container prune -f
```

### Interprétation

**✅ Si vous avez pu lancer et gérer 3 conteneurs :** Docker peut gérer plusieurs conteneurs simultanément. Votre système a suffisamment de ressources.

**Note :** Si vous avez des erreurs, c'est peut-être dû à un manque de ressources (RAM). Vérifiez votre configuration dans la section 2.2.

## Récapitulatif des tests

Voici un tableau récapitulatif de tous les tests :

| Test | Commande | Ce qui est vérifié | Statut |
|------|----------|-------------------|--------|
| 1 | `docker --version` | Installation du CLI | □ |
| 2 | `docker info` | Communication Client-Daemon | □ |
| 3 | `docker run hello-world` | Cycle complet d'utilisation | □ |
| 4 | `docker ps -a` | Gestion des conteneurs | □ |
| 5 | `docker images` | Gestion des images | □ |
| 6 | `docker run -it ubuntu bash` | Conteneurs interactifs | □ |
| 7 | `docker network ls` | Gestion des réseaux | □ |
| 8 | `docker volume create` | Gestion des volumes | □ |
| 9 | `docker compose version` | Docker Compose | □ |
| 10 | Multiple containers | Performance basique | □ |

Cochez mentalement chaque test réussi. Si tous sont validés, votre installation est parfaite !

## Que faire si des tests échouent ?

### Stratégie de dépannage

1. **Ne paniquez pas** : La plupart des problèmes ont des solutions simples
2. **Lisez les messages d'erreur** : Ils contiennent souvent la solution
3. **Vérifiez les prérequis** : Service démarré, permissions, connexion internet
4. **Consultez la section dépannage** ci-dessous
5. **Redémarrez** : Parfois, un simple redémarrage de Docker ou de votre machine résout le problème

### Problèmes courants et solutions

**Problème : "Cannot connect to the Docker daemon"**
- **Cause** : Docker Daemon n'est pas démarré
- **Solution** :
  - Linux : `sudo systemctl start docker`
  - Windows/macOS : Démarrez Docker Desktop

**Problème : "permission denied"**
- **Cause** : Problème de permissions (Linux)
- **Solution** : Ajoutez-vous au groupe docker (voir section 2.2)

**Problème : Téléchargements très lents**
- **Cause** : Connexion internet lente ou problème de DNS
- **Solution** : Vérifiez votre connexion, essayez de changer les DNS

**Problème : "no space left on device"**
- **Cause** : Disque plein
- **Solution** :
  ```bash
  docker system prune -a
  ```
  Ou allouez plus d'espace dans les paramètres Docker

**Problème : Docker est très lent**
- **Cause** : Ressources insuffisantes
- **Solution** : Augmentez CPU/RAM dans les paramètres (voir section 2.2)

### Réinstallation complète (dernier recours)

Si vraiment rien ne fonctionne, vous pouvez faire une réinstallation propre :

**Linux :**
```bash
# Désinstaller complètement
sudo apt-get purge docker-ce docker-ce-cli containerd.io

# Supprimer les données
sudo rm -rf /var/lib/docker

# Réinstaller (voir section 2.1)
```

**Windows/macOS :**
- Docker Desktop → Troubleshoot → Reset to factory defaults
- Ou désinstallez complètement et réinstallez

## Commandes utiles pour le diagnostic

Voici quelques commandes qui peuvent aider à diagnostiquer les problèmes :

```bash
# Voir les logs du Docker Daemon (Linux)
sudo journalctl -u docker -f

# Voir les événements Docker en temps réel
docker events

# Voir l'utilisation des ressources
docker stats

# Informations détaillées sur un conteneur
docker inspect <container_id>

# Logs d'un conteneur
docker logs <container_id>
```

## Confirmation finale

Si vous avez réussi au moins les 5 premiers tests (les plus importants), vous êtes prêt à continuer !

**Vous pouvez maintenant :**
- ✅ Exécuter des conteneurs Docker
- ✅ Télécharger des images depuis Docker Hub
- ✅ Gérer des conteneurs (créer, lister, supprimer)
- ✅ Comprendre les bases de l'architecture Docker

**Points de validation finale :**

1. `docker run hello-world` fonctionne sans erreur ? ✓
2. `docker ps` affiche quelque chose (même vide) ? ✓
3. `docker images` liste vos images ? ✓
4. Vous avez pu entrer dans un conteneur Ubuntu ? ✓

Si vous répondez oui à ces 4 questions, **votre installation Docker est validée** !

## En résumé

La vérification de l'installation est une étape cruciale qui garantit que votre environnement Docker est totalement opérationnel. Nous avons testé :

**Les composants de base :**
- Docker CLI (Client)
- Docker Daemon (Serveur)
- Communication Client-Daemon

**Les fonctionnalités essentielles :**
- Téléchargement d'images depuis Docker Hub
- Création et exécution de conteneurs
- Gestion des conteneurs et images
- Conteneurs interactifs
- Réseaux et volumes
- Docker Compose

**Les performances :**
- Gestion de plusieurs conteneurs simultanés
- Utilisation des ressources

Si tous vos tests sont passés avec succès, félicitations ! Votre environnement Docker est parfaitement configuré. Vous êtes maintenant prêt à plonger dans les concepts fondamentaux de Docker et à commencer à créer vos propres conteneurs.

Dans le prochain chapitre, nous explorerons en profondeur les concepts d'images et de conteneurs, les deux piliers de Docker.

⏭️ [Docker Desktop vs Docker Engine](/02-installation-et-configuration/04-docker-desktop-vs-docker-engine.md)
