🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 11.5 Introduction au débogage de conteneurs

## Introduction

Le débogage est une compétence essentielle pour tout développeur. Lorsque quelque chose ne fonctionne pas, vous devez être capable de comprendre **pourquoi** et **comment** le résoudre. Avec Docker, le débogage peut sembler différent au début, car votre application s'exécute dans un environnement isolé : le conteneur.

Imaginez que vous avez lancé un conteneur, mais :
- 🔴 L'application ne démarre pas
- 🔴 Le conteneur s'arrête immédiatement
- 🔴 L'application ne répond pas aux requêtes
- 🔴 Vous obtenez des erreurs mystérieuses

Comment investiguer ? C'est exactement ce que nous allons apprendre dans ce chapitre.

## Pourquoi le débogage de conteneurs est différent

### Différences avec le développement local

**Développement local** :
```bash
# Voir directement les erreurs
python app.py
# Erreur affichée immédiatement dans le terminal

# Inspecter les fichiers
ls -la  
cat config.json  

# Installer des outils
apt install vim
```

**Avec Docker** :
```bash
# Le conteneur démarre et s'arrête
docker run my-app
# Rien ne s'affiche... Que s'est-il passé ?

# Les fichiers sont dans le conteneur
# Comment les voir ?

# Pas d'accès direct au système de fichiers
```

### Défis spécifiques

1. **Environnement isolé** : L'application est dans une "boîte noire"
2. **Conteneurs éphémères** : Ils peuvent disparaître avec leurs logs
3. **Réseau virtuel** : Les problèmes de connexion sont moins évidents
4. **Layers et cache** : Les erreurs de build peuvent être cachées
5. **Permissions** : Problèmes d'utilisateurs et de droits

## Les outils de débogage essentiels

### Vue d'ensemble

Docker fournit plusieurs commandes pour inspecter et déboguer vos conteneurs :

| Commande | Utilité | Quand l'utiliser |
|----------|---------|------------------|
| `docker logs` | Voir les sorties du conteneur | Erreurs d'exécution |
| `docker exec` | Exécuter des commandes dans un conteneur | Explorer l'environnement |
| `docker inspect` | Voir la configuration détaillée | Problèmes de configuration |
| `docker ps` | Lister les conteneurs | Vérifier l'état |
| `docker stats` | Voir l'utilisation des ressources | Problèmes de performance |
| `docker top` | Voir les processus en cours | Identifier ce qui tourne |

## Technique 1 : Analyser les logs

Les logs sont votre **première ligne de défense** pour le débogage.

### Voir les logs d'un conteneur

```bash
# Logs d'un conteneur en cours d'exécution
docker logs mon-conteneur

# Logs d'un conteneur arrêté
docker logs mon-conteneur

# Suivre les logs en temps réel (comme tail -f)
docker logs -f mon-conteneur

# Afficher les 50 dernières lignes
docker logs --tail 50 mon-conteneur

# Afficher avec les timestamps
docker logs -t mon-conteneur

# Logs depuis les 10 dernières minutes
docker logs --since 10m mon-conteneur
```

### Exemple pratique : Application qui crash

**Scénario** : Vous lancez votre application, mais le conteneur s'arrête immédiatement.

```bash
# Lancer le conteneur
docker run -d --name my-app python-app

# Vérifier l'état
docker ps -a
# CONTAINER ID   NAME      STATUS
# abc123         my-app    Exited (1) 2 seconds ago

# Le conteneur est arrêté ! Pourquoi ?
# Regarder les logs
docker logs my-app
```

**Output des logs** :
```
Traceback (most recent call last):
  File "app.py", line 3, in <module>
    import flask
ModuleNotFoundError: No module named 'flask'
```

**Diagnostic** : Flask n'est pas installé ! Il faut l'ajouter au Dockerfile.

### Interpréter les logs courants

**Erreur de connexion à la base de données** :
```
Error: could not connect to database  
Connection refused at localhost:5432  
```
→ La base de données n'est pas accessible (mauvais host, port, ou DB non démarrée)

**Erreur de permission** :
```
PermissionError: [Errno 13] Permission denied: '/data/output.txt'
```
→ Problème de permissions sur les fichiers/volumes

**Port déjà utilisé** :
```
Error: bind: address already in use
```
→ Un autre conteneur ou processus utilise déjà ce port

### Logs avec Docker Compose

```bash
# Logs de tous les services
docker compose logs

# Logs d'un service spécifique
docker compose logs web

# Suivre les logs en temps réel
docker compose logs -f

# Logs des 3 derniers conteneurs
docker compose logs --tail 100 web api database
```

## Technique 2 : Entrer dans un conteneur

Parfois, vous avez besoin d'**explorer l'intérieur** du conteneur pour comprendre ce qui se passe.

### Commande docker exec

```bash
# Ouvrir un shell interactif dans un conteneur en cours d'exécution
docker exec -it mon-conteneur bash

# Si bash n'est pas disponible, utiliser sh
docker exec -it mon-conteneur sh

# Exécuter une commande simple
docker exec mon-conteneur ls -la /app

# Exécuter une commande en tant que root
docker exec -u root -it mon-conteneur bash
```

### Cas d'usage pratiques

**Vérifier les fichiers** :
```bash
docker exec -it mon-conteneur bash  
ls -la /app  
cat /app/config.json  
```

**Tester la connectivité** :
```bash
docker exec mon-conteneur ping database  
docker exec mon-conteneur curl http://api:5000/health  
```

**Vérifier les variables d'environnement** :
```bash
docker exec mon-conteneur env  
docker exec mon-conteneur printenv DATABASE_URL  
```

**Inspecter les processus** :
```bash
docker exec mon-conteneur ps aux
```

### Exemple : Déboguer une connexion à la base de données

```bash
# Entrer dans le conteneur de l'application
docker exec -it my-app bash

# Tester la connexion PostgreSQL
psql -h database -U postgres -d mydb
# Si ça ne fonctionne pas, vérifier :

# 1. Est-ce que le host est correct ?
ping database

# 2. Est-ce que le port est ouvert ?
nc -zv database 5432

# 3. Les variables d'environnement sont-elles correctes ?
echo $DATABASE_URL
```

### Que faire si le conteneur s'arrête immédiatement ?

Si le conteneur ne reste pas actif, vous ne pouvez pas utiliser `docker exec`. Solution :

```bash
# Méthode 1 : Lancer avec une commande qui ne termine pas
docker run -it --entrypoint /bin/bash mon-image

# Méthode 2 : Override la commande
docker run -it mon-image bash

# Méthode 3 : sleep infini pour garder le conteneur actif
docker run -d --name debug-container mon-image sleep infinity  
docker exec -it debug-container bash  
```

## Technique 3 : Inspecter la configuration

La commande `docker inspect` affiche **toutes les informations** sur un conteneur.

### Utilisation de base

```bash
# Inspecter un conteneur
docker inspect mon-conteneur

# Afficher uniquement certaines informations
docker inspect --format='{{.State.Status}}' mon-conteneur  
docker inspect --format='{{.NetworkSettings.IPAddress}}' mon-conteneur  
docker inspect --format='{{json .Config.Env}}' mon-conteneur  
```

### Informations utiles pour le débogage

**État du conteneur** :
```bash
docker inspect --format='{{.State.Status}}' mon-conteneur
# Output: running, exited, paused, etc.

docker inspect --format='{{.State.ExitCode}}' mon-conteneur
# Output: 0 (succès), 1 (erreur), 137 (OOMKilled), etc.
```

**Variables d'environnement** :
```bash
docker inspect --format='{{json .Config.Env}}' mon-conteneur | jq
```

**Volumes montés** :
```bash
docker inspect --format='{{json .Mounts}}' mon-conteneur | jq
```

**Configuration réseau** :
```bash
docker inspect --format='{{json .NetworkSettings.Networks}}' mon-conteneur | jq
```

**Ports exposés** :
```bash
docker inspect --format='{{json .NetworkSettings.Ports}}' mon-conteneur | jq
```

### Exemple : Diagnostic complet

```bash
#!/bin/bash
# Script de diagnostic automatique

CONTAINER=$1

echo "=== État du conteneur ==="  
docker inspect --format='Status: {{.State.Status}}' $CONTAINER  
docker inspect --format='Exit Code: {{.State.ExitCode}}' $CONTAINER  
docker inspect --format='Started At: {{.State.StartedAt}}' $CONTAINER  

echo -e "\n=== Configuration réseau ==="  
docker inspect --format='IP Address: {{.NetworkSettings.IPAddress}}' $CONTAINER  
docker inspect --format='Ports: {{json .NetworkSettings.Ports}}' $CONTAINER | jq  

echo -e "\n=== Volumes ==="  
docker inspect --format='{{json .Mounts}}' $CONTAINER | jq  

echo -e "\n=== Variables d'environnement ==="  
docker inspect --format='{{json .Config.Env}}' $CONTAINER | jq  

echo -e "\n=== Commande de démarrage ==="  
docker inspect --format='{{.Config.Cmd}}' $CONTAINER  
```

**Utilisation** :
```bash
./diagnose.sh mon-conteneur
```

## Technique 4 : Analyser les ressources

### Surveillance en temps réel

```bash
# Voir l'utilisation CPU, mémoire, réseau, disque
docker stats

# Stats d'un conteneur spécifique
docker stats mon-conteneur

# Afficher une seule fois (pas en continu)
docker stats --no-stream
```

**Output** :
```
CONTAINER ID   NAME         CPU %    MEM USAGE / LIMIT    MEM %    NET I/O  
abc123         my-app       0.5%     50MiB / 512MiB      9.77%    1.2kB / 0B  
def456         database     2.1%     200MiB / 2GiB       9.77%    5kB / 2kB  
```

### Interpréter les statistiques

**CPU à 100%** :
- Boucle infinie dans le code ?
- Traitement intensif normal ?
- Processus qui ne termine pas ?

```bash
# Voir les processus dans le conteneur
docker top mon-conteneur

# Entrer pour investiguer
docker exec -it mon-conteneur bash  
ps aux  
top  
```

**Mémoire proche de la limite** :
- Fuite mémoire ?
- Pas assez de RAM allouée ?
- Charge trop importante ?

```bash
# Augmenter temporairement la limite
docker update --memory="2g" mon-conteneur

# Ou redémarrer avec plus de mémoire
docker run --memory="2g" mon-image
```

**Trafic réseau excessif** :
- Problème de boucle de requêtes ?
- Transfert de gros fichiers ?
- Attaque en cours ?

## Technique 5 : Déboguer le build

Les erreurs peuvent survenir lors de la **construction** de l'image.

### Build avec sortie détaillée

```bash
# Build normal
docker build -t mon-app .

# Build avec plus de détails
docker build --progress=plain -t mon-app .

# Build sans utiliser le cache (pour déboguer les problèmes de cache)
docker build --no-cache -t mon-app .
```

### Déboguer une étape spécifique

Si le build échoue à une étape :

```dockerfile
FROM python:3.11

WORKDIR /app

# Étape 1 : OK
COPY requirements.txt .

# Étape 2 : ÉCHOUE ICI
RUN pip install -r requirements.txt

# Étape 3 : Jamais atteinte
COPY . .  
CMD ["python", "app.py"]  
```

**Technique** : Builder jusqu'à l'étape problématique

```bash
# Construire uniquement jusqu'avant l'étape qui échoue
docker build --target builder -t debug-image .

# Ou créer un Dockerfile de debug temporaire
```

**Dockerfile.debug** :
```dockerfile
FROM python:3.11

WORKDIR /app

COPY requirements.txt .

# Commenter la ligne qui échoue
# RUN pip install -r requirements.txt

# Garder le conteneur actif
CMD ["sleep", "infinity"]
```

```bash
# Builder l'image de debug
docker build -f Dockerfile.debug -t debug-image .

# Lancer le conteneur
docker run -d --name debug debug-image

# Entrer et tester manuellement
docker exec -it debug bash  
pip install -r requirements.txt  
# Voir exactement quelle dépendance pose problème
```

### Erreurs de build courantes

**Erreur : fichier non trouvé** :
```
COPY app.py /app/  
ERROR: failed to compute cache key: "/app.py" not found  
```
→ Le fichier n'existe pas ou le chemin est incorrect

**Erreur : permission denied** :
```
RUN ./install.sh
/bin/sh: ./install.sh: Permission denied
```
→ Le fichier n'est pas exécutable

**Solution** :
```dockerfile
COPY install.sh /tmp/  
RUN chmod +x /tmp/install.sh && /tmp/install.sh  
```

**Erreur : contexte de build trop large** :
```
Sending build context to Docker daemon  5.2GB
```
→ Créer/améliorer le `.dockerignore`

## Problèmes courants et solutions

### Problème 1 : Le conteneur s'arrête immédiatement

**Symptômes** :
```bash
docker run -d --name test nginx  
docker ps  
# Le conteneur n'apparaît pas

docker ps -a
# STATUS: Exited (0) 1 second ago
```

**Causes possibles** :
1. La commande principale se termine
2. Erreur dans l'application
3. Mauvaise commande CMD/ENTRYPOINT

**Diagnostic** :
```bash
# Voir les logs
docker logs test

# Voir le code de sortie
docker inspect --format='{{.State.ExitCode}}' test
```

**Codes de sortie courants** :
- `0` : Sortie normale (mais pourquoi si rapide ?)
- `1` : Erreur générale
- `126` : Commande non exécutable
- `127` : Commande non trouvée
- `137` : Tué par SIGKILL (souvent OOM)

**Solutions** :
```bash
# Lancer en mode interactif pour voir ce qui se passe
docker run -it --name test mon-image

# Override la commande pour garder le conteneur actif
docker run -d --name test mon-image tail -f /dev/null
```

### Problème 2 : Application inaccessible

**Symptômes** :
```bash
docker run -d -p 8080:80 --name web nginx  
curl http://localhost:8080  
# curl: (7) Failed to connect to localhost port 8080
```

**Diagnostic** :

**1. Le port est-il bien mappé ?**
```bash
docker ps
# Vérifier la colonne PORTS: 0.0.0.0:8080->80/tcp

docker port web
# 80/tcp -> 0.0.0.0:8080
```

**2. L'application écoute-t-elle sur le bon port ?**
```bash
docker exec web netstat -tlnp
# Doit montrer :80 en LISTEN

# Ou avec ss
docker exec web ss -tlnp
```

**3. L'application écoute-t-elle sur 0.0.0.0 ou localhost ?**
```python
# ❌ Mauvais : écoute seulement sur localhost
app.run(host='127.0.0.1', port=5000)

# ✅ Bon : écoute sur toutes les interfaces
app.run(host='0.0.0.0', port=5000)
```

**4. Y a-t-il un firewall ?**
```bash
# Vérifier les règles iptables
sudo iptables -L -n
```

**5. Tester depuis l'intérieur du conteneur**
```bash
docker exec web curl http://localhost:80
# Si ça marche, le problème est le mapping de port
```

### Problème 3 : Connexion entre conteneurs impossible

**Symptômes** :
```bash
# Depuis le conteneur web
docker exec web curl http://database:5432
# curl: (6) Could not resolve host: database
```

**Diagnostic** :

**1. Les conteneurs sont-ils sur le même réseau ?**
```bash
# Voir les réseaux de chaque conteneur
docker inspect --format='{{json .NetworkSettings.Networks}}' web | jq  
docker inspect --format='{{json .NetworkSettings.Networks}}' database | jq  
```

**2. Le nom du service est-il correct ?**
```bash
# Lister tous les conteneurs
docker ps

# Tester avec l'IP directement
docker inspect --format='{{.NetworkSettings.IPAddress}}' database
# Essayer avec cette IP
docker exec web curl http://172.17.0.3:5432
```

**3. Le port est-il ouvert ?**
```bash
docker exec web nc -zv database 5432
```

**Solution** :
```bash
# Créer un réseau personnalisé
docker network create my-network

# Lancer les conteneurs sur ce réseau
docker run -d --name database --network my-network postgres  
docker run -d --name web --network my-network nginx  

# Tester
docker exec web ping database
```

### Problème 4 : Variables d'environnement non définies

**Symptômes** :
```bash
docker logs app
# KeyError: 'DATABASE_URL'
```

**Diagnostic** :
```bash
# Vérifier les variables d'environnement
docker exec app env  
docker exec app printenv DATABASE_URL  
```

**Solutions** :

**Avec docker run** :
```bash
docker run -e DATABASE_URL=postgres://... app
```

**Avec Docker Compose** :
```yaml
services:
  app:
    environment:
      - DATABASE_URL=postgres://user:pass@db:5432/mydb
    # Ou depuis un fichier
    env_file:
      - .env
```

**.env** :
```
DATABASE_URL=postgres://user:pass@db:5432/mydb  
API_KEY=secret123  
```

### Problème 5 : Volumes vides ou données manquantes

**Symptômes** :
```bash
docker exec app ls /data
# Vide alors que vous avez copié des fichiers
```

**Diagnostic** :
```bash
# Voir les volumes montés
docker inspect --format='{{json .Mounts}}' app | jq

# Vérifier le volume
docker volume inspect my-volume
```

**Causes courantes** :
1. Volume monté après COPY dans le Dockerfile
2. Mauvais chemin de montage
3. Permissions incorrectes

**Solution** :

**Ordre correct dans Dockerfile** :
```dockerfile
# ❌ Mauvais ordre
VOLUME /data  
COPY data/ /data/  
# Le VOLUME efface ce qui a été copié !

# ✅ Bon ordre
COPY data/ /data/  
VOLUME /data  
```

**Vérifier le montage** :
```bash
docker run -v $(pwd)/data:/data app ls /data
```

### Problème 6 : Permissions denied sur les fichiers

**Symptômes** :
```bash
docker logs app
# PermissionError: [Errno 13] Permission denied: '/data/output.txt'
```

**Diagnostic** :
```bash
# Vérifier les permissions
docker exec app ls -la /data

# Voir l'utilisateur actuel
docker exec app whoami  
docker exec app id  
```

**Solutions** :

**Méthode 1 : Changer les permissions** :
```dockerfile
RUN mkdir -p /data && chmod 777 /data
```

**Méthode 2 : Exécuter en tant que root** :
```bash
docker run --user root app
```

**Méthode 3 : Créer l'utilisateur avec le bon UID** :
```dockerfile
ARG USER_ID=1000  
ARG GROUP_ID=1000  

RUN groupadd -g ${GROUP_ID} appuser && \
    useradd -u ${USER_ID} -g appuser -m appuser

USER appuser
```

```bash
docker build --build-arg USER_ID=$(id -u) --build-arg GROUP_ID=$(id -g) -t app .
```

## Outils avancés de débogage

### 1. Docker logs avec filtres

```bash
# Logs depuis une date/heure spécifique
docker logs --since "2024-10-22T10:00:00" mon-conteneur

# Logs jusqu'à une date/heure
docker logs --until "2024-10-22T12:00:00" mon-conteneur

# Logs entre deux dates
docker logs --since "10:00" --until "11:00" mon-conteneur
```

### 2. Copier des fichiers depuis/vers un conteneur

```bash
# Copier un fichier depuis le conteneur
docker cp mon-conteneur:/app/logs/error.log ./local-error.log

# Copier un fichier vers le conteneur
docker cp config.json mon-conteneur:/app/config.json

# Copier un dossier complet
docker cp mon-conteneur:/var/log/ ./logs/
```

**Cas d'usage** : Récupérer des logs ou fichiers de configuration pour analyse.

### 3. Voir les événements Docker

```bash
# Surveiller tous les événements en temps réel
docker events

# Filtrer par type
docker events --filter type=container

# Filtrer par événement spécifique
docker events --filter event=start  
docker events --filter event=die  

# Filtrer par conteneur
docker events --filter container=mon-conteneur
```

**Utilité** : Comprendre ce qui se passe au niveau de Docker (démarrages, arrêts, erreurs).

### 4. Commande docker diff

```bash
# Voir les modifications dans le système de fichiers
docker diff mon-conteneur
```

**Output** :
```
A /app/logs/app.log      # Ajouté  
C /app/config.json       # Modifié  
D /tmp/cache.tmp         # Supprimé  
```

**Utilité** : Identifier quels fichiers ont changé depuis le démarrage.

### 5. Docker commit pour investigation

```bash
# Sauvegarder l'état actuel d'un conteneur en tant qu'image
docker commit mon-conteneur debug-snapshot

# Lancer un nouveau conteneur depuis ce snapshot
docker run -it debug-snapshot bash

# Investiguer tranquillement
```

**Cas d'usage** : Le conteneur est dans un état bizarre ? Sauvegardez-le pour investigation.

### 6. Utiliser strace pour tracer les appels système

```bash
# Installer strace dans le conteneur
docker exec -u root mon-conteneur apt-get update && apt-get install -y strace

# Tracer un processus
docker exec mon-conteneur strace -p <PID>

# Ou tracer une nouvelle commande
docker exec mon-conteneur strace curl http://api:5000
```

**Utilité** : Voir exactement quels appels système l'application fait (ouverture de fichiers, connexions réseau, etc.).

### 7. tcpdump pour analyser le trafic réseau

```bash
# Installer tcpdump
docker exec -u root mon-conteneur apt-get update && apt-get install -y tcpdump

# Capturer le trafic
docker exec mon-conteneur tcpdump -i any -n

# Capturer seulement le trafic HTTP
docker exec mon-conteneur tcpdump -i any -n port 80

# Sauvegarder dans un fichier pour analyse avec Wireshark
docker exec mon-conteneur tcpdump -i any -w /tmp/capture.pcap  
docker cp mon-conteneur:/tmp/capture.pcap ./  
```

## Stratégies de débogage

### Approche méthodique

**1. Identifier le symptôme**
- Le conteneur ne démarre pas ?
- Il démarre mais crash ?
- Il tourne mais ne répond pas ?
- Il répond mais avec des erreurs ?

**2. Collecter les informations**
```bash
docker ps -a              # État  
docker logs mon-conteneur # Logs  
docker inspect mon-conteneur # Configuration  
```

**3. Former des hypothèses**
- Configuration incorrecte ?
- Problème de connectivité ?
- Ressources insuffisantes ?
- Bug dans le code ?

**4. Tester les hypothèses**
```bash
# Tester la connectivité
docker exec mon-conteneur ping database

# Tester les ressources
docker stats mon-conteneur

# Tester la configuration
docker inspect mon-conteneur
```

**5. Appliquer la solution**

**6. Vérifier que c'est résolu**

### Checklist de débogage rapide

```
□ Vérifier docker ps -a pour l'état du conteneur
□ Lire les logs avec docker logs
□ Vérifier le code de sortie
□ Inspecter la configuration réseau
□ Vérifier les variables d'environnement
□ Tester la connectivité entre conteneurs
□ Vérifier les ports exposés et mappés
□ Inspecter les volumes montés
□ Vérifier les ressources (CPU, mémoire)
□ Entrer dans le conteneur et tester manuellement
```

## Debugging en environnement de développement vs production

### Développement

**Approche** : Maximum de verbosité et d'outils

```yaml
# docker-compose.dev.yml
services:
  app:
    build:
      context: .
      target: development
    volumes:
      - .:/app  # Code monté en direct
    environment:
      - DEBUG=true
      - LOG_LEVEL=debug
    command: python app.py --debug --reload
```

**Avantages** :
- Rechargement automatique du code
- Logs détaillés
- Accès facile au conteneur
- Outils de développement installés

### Production

**Approche** : Minimale et sécurisée

```yaml
# docker-compose.prod.yml
services:
  app:
    image: mon-app:latest
    environment:
      - LOG_LEVEL=info
    restart: unless-stopped
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

**Pratiques** :
- Logs structurés et centralisés
- Monitoring externe (Prometheus, Grafana)
- Pas de volumes de code
- Logs avec rotation
- Pas de shell interactif

## Bonnes pratiques pour faciliter le débogage

### 1. Logs structurés

```python
# ❌ Mauvais
print("Error happened")

# ✅ Bon
import logging  
logging.basicConfig(level=logging.INFO)  
logger = logging.getLogger(__name__)  

logger.info("Application started", extra={"version": "1.0.0"})  
logger.error("Database connection failed", extra={"host": "db", "port": 5432})  
```

### 2. Health checks

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost/health || exit 1
```

Permet de savoir si l'application est vraiment fonctionnelle.

### 3. Labels pour documentation

```dockerfile
LABEL maintainer="dev@example.com"  
LABEL version="1.0.0"  
LABEL description="API Backend"  
LABEL debug.instructions="Use 'docker exec -it <container> bash' to debug"  
```

```bash
docker inspect --format='{{json .Config.Labels}}' mon-conteneur | jq
```

### 4. Installer des outils de debug en développement

```dockerfile
# Multi-stage build
FROM python:3.11 AS base  
WORKDIR /app  
COPY requirements.txt .  
RUN pip install -r requirements.txt  

# Stage de développement avec outils
FROM base AS development  
RUN apt-get update && apt-get install -y \  
    vim \
    curl \
    telnet \
    net-tools \
    iputils-ping \
    && rm -rf /var/lib/apt/lists/*

# Stage de production sans outils
FROM base AS production  
COPY . .  
CMD ["python", "app.py"]  
```

```bash
# Développement
docker build --target development -t app:dev .

# Production
docker build --target production -t app:prod .
```

### 5. Variables d'environnement pour le debug

```dockerfile
ENV DEBUG=false  
ENV LOG_LEVEL=info  
```

```bash
# Activer le mode debug
docker run -e DEBUG=true -e LOG_LEVEL=debug app
```

### 6. Ne pas supprimer les conteneurs arrêtés immédiatement

```bash
# ❌ Supprime le conteneur
docker run --rm app

# ✅ Garde le conteneur pour investigation
docker run --name app-test app

# Si échec, on peut toujours voir les logs
docker logs app-test  
docker inspect app-test  
```

### 7. Nommer les conteneurs explicitement

```bash
# ❌ Nom aléatoire
docker run -d nginx

# ✅ Nom explicite
docker run -d --name frontend-prod nginx
```

Plus facile à identifier lors du débogage.

### 8. Utiliser des tags de version

```bash
# ❌ Utiliser latest (version inconnue)
docker run app:latest

# ✅ Version explicite
docker run app:1.2.3
```

Si un bug apparaît, vous savez exactement quelle version investiguer.

## Ressources et outils externes

### Portainer

Interface web pour gérer et déboguer Docker :

```bash
docker run -d \
  -p 9000:9000 \
  --name portainer \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce
```

Accès : http://localhost:9000

**Fonctionnalités** :
- Vue graphique des conteneurs
- Logs en direct
- Console interactive
- Statistiques en temps réel

### Lazydocker

Terminal UI pour Docker :

```bash
# Installation
docker run --rm -it -v /var/run/docker.sock:/var/run/docker.sock lazyteam/lazydocker
```

**Avantages** :
- Interface TUI (Terminal User Interface)
- Navigation rapide
- Logs, stats, et shell en un coup d'œil

### ctop

Monitoring des conteneurs style `top` :

```bash
docker run --rm -ti \
  --name ctop \
  -v /var/run/docker.sock:/var/run/docker.sock \
  quay.io/vektorlab/ctop
```

## Résumé

Le débogage de conteneurs Docker nécessite une approche méthodique et des outils spécifiques.

### Commandes essentielles à retenir

```bash
# 1. Voir les logs
docker logs -f mon-conteneur

# 2. Entrer dans le conteneur
docker exec -it mon-conteneur bash

# 3. Inspecter la configuration
docker inspect mon-conteneur

# 4. Voir les statistiques
docker stats mon-conteneur

# 5. Voir les processus
docker top mon-conteneur

# 6. Copier des fichiers
docker cp mon-conteneur:/app/log.txt ./
```

### Workflow de débogage

1. **Observer** : `docker ps -a`, `docker logs`
2. **Investiguer** : `docker exec`, `docker inspect`
3. **Tester** : Commandes manuelles dans le conteneur
4. **Corriger** : Modifier Dockerfile, configuration, ou code
5. **Valider** : Reconstruire et relancer

### Points clés

✅ **Logs d'abord** : 90% des problèmes sont visibles dans les logs  
✅ **Inspection** : `docker inspect` donne toute la configuration  
✅ **Shell interactif** : `docker exec` pour explorer l'environnement  
✅ **Approche systématique** : Ne pas deviner, vérifier méthodiquement  
✅ **Documentation** : Documenter les solutions trouvées  
✅ **Prevention** : Health checks, logs structurés, nommage clair

### Erreurs courantes à éviter

❌ Supprimer les conteneurs avant d'avoir investigué  
❌ Ne pas lire les logs complets  
❌ Deviner au lieu de vérifier  
❌ Ne pas documenter les solutions  
❌ Oublier de vérifier les variables d'environnement

Le débogage devient plus facile avec l'expérience. Plus vous pratiquez, plus vous reconnaîtrez rapidement les symptômes et saurez où chercher. N'hésitez pas à utiliser les outils et techniques présentés ici pour résoudre efficacement vos problèmes Docker !

⏭️ [Conclusion et perspectives](/12-conclusion-et-perspectives/README.md)
