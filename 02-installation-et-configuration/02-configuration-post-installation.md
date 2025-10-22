🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 2.2 Configuration post-installation

## Introduction

Félicitations ! Docker est maintenant installé sur votre système. Mais avant de vous lancer dans la création de conteneurs, quelques ajustements de configuration vous permettront d'optimiser votre expérience et d'éviter des frustrations futures.

Cette configuration post-installation ne prend que quelques minutes mais fait toute la différence. Vous allez configurer Docker pour qu'il démarre automatiquement, optimiser l'utilisation des ressources, et personnaliser certains paramètres selon vos besoins.

Ne vous inquiétez pas : toutes ces configurations sont optionnelles et réversibles. Vous pouvez les ajuster à tout moment.

## Configuration par système d'exploitation

Les configurations à effectuer varient selon votre système d'exploitation. Rendez-vous directement à la section qui vous concerne :
- [Configuration sur Linux](#configuration-sur-linux)
- [Configuration sur Windows](#configuration-sur-windows)
- [Configuration sur macOS](#configuration-sur-macos)

## Configuration sur Linux

### 1. Démarrage automatique de Docker

Par défaut, le service Docker ne démarre pas toujours automatiquement au démarrage de votre système. Pour activer le démarrage automatique :

```bash
# Activer Docker au démarrage
sudo systemctl enable docker

# Démarrer Docker immédiatement
sudo systemctl start docker

# Vérifier le statut
sudo systemctl status docker
```

Vous devriez voir `active (running)` dans le statut.

**Pourquoi c'est important ?**
Sans cette configuration, vous devrez démarrer Docker manuellement à chaque redémarrage de votre machine, ce qui peut être fastidieux.

### 2. Utiliser Docker sans sudo (Recommandé)

Par défaut sur Linux, toutes les commandes Docker nécessitent `sudo`. C'est pénible à la longue. Voici comment configurer Docker pour l'utiliser sans sudo :

```bash
# Créer le groupe docker (si pas déjà existant)
sudo groupadd docker

# Ajouter votre utilisateur au groupe docker
sudo usermod -aG docker $USER

# Activer les changements de groupe sans déconnexion
newgrp docker
```

**Tester la configuration :**
```bash
# Cette commande devrait fonctionner sans sudo
docker run hello-world
```

Si cela fonctionne, vous n'avez plus besoin de taper `sudo` avant chaque commande Docker !

**Note importante sur la sécurité :**
Ajouter un utilisateur au groupe `docker` lui donne des privilèges équivalents à root. Ne faites cela que pour des utilisateurs de confiance sur votre système. Sur un serveur de production, gardez plutôt l'utilisation de `sudo`.

**Si vous rencontrez des problèmes de permissions :**
```bash
# Corriger les permissions du socket Docker
sudo chown root:docker /var/run/docker.sock
sudo chmod 660 /var/run/docker.sock
```

### 3. Configuration des ressources système

Sur Linux, Docker utilise directement les ressources du système hôte. Cependant, vous pouvez limiter les ressources qu'un conteneur peut consommer.

**Vérifier les ressources disponibles :**
```bash
# Voir les informations système
docker info | grep -i memory
docker info | grep -i cpu
```

Ces limites se configurent généralement au niveau de chaque conteneur (nous verrons cela plus tard) plutôt qu'au niveau global.

### 4. Configurer le stockage Docker

Docker stocke ses données (images, conteneurs, volumes) dans `/var/lib/docker` par défaut. Si vous manquez d'espace ou préférez un autre emplacement :

**Vérifier l'espace disque utilisé :**
```bash
# Voir l'espace utilisé par Docker
docker system df
```

**Changer l'emplacement de stockage (si nécessaire) :**

1. Arrêter Docker :
```bash
sudo systemctl stop docker
```

2. Éditer le fichier de configuration :
```bash
sudo nano /etc/docker/daemon.json
```

3. Ajouter cette configuration :
```json
{
  "data-root": "/nouveau/chemin/docker"
}
```

4. Déplacer les données existantes :
```bash
sudo mv /var/lib/docker /nouveau/chemin/docker
```

5. Redémarrer Docker :
```bash
sudo systemctl start docker
```

**Note :** Cette manipulation n'est généralement nécessaire que si `/var` est sur une partition petite.

### 5. Configurer le logging

Docker génère des logs qui peuvent rapidement occuper de l'espace. Configurons une rotation des logs :

Éditez `/etc/docker/daemon.json` :
```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

Cette configuration limite chaque fichier de log à 10 Mo et garde maximum 3 fichiers.

Redémarrez Docker pour appliquer :
```bash
sudo systemctl restart docker
```

### 6. Configuration réseau (optionnel)

Par défaut, Docker crée un réseau bridge. Pour personnaliser les plages IP :

Éditez `/etc/docker/daemon.json` :
```json
{
  "bip": "192.168.1.1/24",
  "default-address-pools": [
    {
      "base": "192.168.2.0/24",
      "size": 28
    }
  ]
}
```

**Note :** Cette configuration n'est nécessaire que si vous avez des conflits IP avec votre réseau existant.

## Configuration sur Windows

Si vous avez installé Docker Desktop sur Windows, la plupart des configurations se font via l'interface graphique.

### 1. Accéder aux paramètres Docker Desktop

1. Cliquez sur l'icône Docker dans la barre des tâches (zone de notification)
2. Cliquez sur l'icône ⚙️ (Settings)
3. La fenêtre de configuration s'ouvre

### 2. Configuration des ressources

Docker Desktop sur Windows utilise WSL 2, qui gère automatiquement les ressources. Mais vous pouvez définir des limites :

**Dans Docker Desktop :**
1. Allez dans **Settings** → **Resources**
2. Ajustez les paramètres :
   - **CPUs** : Nombre de processeurs alloués (recommandé : 2-4)
   - **Memory** : RAM allouée (recommandé : 4-8 Go)
   - **Swap** : Mémoire swap (recommandé : 1-2 Go)
   - **Disk image size** : Espace disque maximum (recommandé : 64 Go minimum)

**Recommandations selon votre machine :**
- **8 Go de RAM** : Allouez 3-4 Go à Docker
- **16 Go de RAM** : Allouez 6-8 Go à Docker
- **32 Go de RAM ou plus** : Allouez 8-12 Go à Docker

Laissez toujours au moins 4 Go de RAM pour Windows lui-même.

### 3. Configuration WSL 2

Pour une meilleure performance, vous pouvez limiter les ressources WSL 2 globalement.

Créez ou éditez le fichier `C:\Users\VotreNom\.wslconfig` :

```ini
[wsl2]
memory=6GB
processors=4
swap=2GB
```

**Redémarrez WSL 2 pour appliquer :**
```powershell
wsl --shutdown
```

Puis relancez Docker Desktop.

### 4. Démarrage automatique

**Activer le démarrage automatique :**
1. **Settings** → **General**
2. Cochez ✅ **Start Docker Desktop when you log in**

**Accélérer le démarrage (optionnel) :**
1. Décochez **Use the WSL 2 based engine** si vous n'utilisez pas de conteneurs Linux (rare)
2. Décochez **Enable Kubernetes** si vous n'en avez pas besoin (pour débutants : décoché recommandé)

### 5. Configuration du stockage

**Changer l'emplacement des images Docker :**
1. **Settings** → **Resources** → **Advanced**
2. **Disk image location** : Cliquez sur **Browse** pour changer l'emplacement
3. Choisissez un disque avec plus d'espace si nécessaire

**Nettoyer l'espace disque :**
Docker Desktop offre une fonction de nettoyage intégrée :
1. Cliquez sur l'icône **🐞** (Troubleshoot)
2. Cliquez sur **Clean / Purge data**
3. Sélectionnez ce que vous voulez nettoyer

### 6. Configuration réseau

**Dans Docker Desktop :**
1. **Settings** → **Resources** → **Network**
2. Vous pouvez ajuster les DNS si nécessaire
3. Par défaut, Docker utilise les DNS de Windows (recommandé)

### 7. Intégration avec WSL

Si vous utilisez WSL (Windows Subsystem for Linux), activez l'intégration :

1. **Settings** → **Resources** → **WSL Integration**
2. Activez l'intégration pour vos distributions WSL
3. Cochez les distributions où vous voulez utiliser Docker

Cela permet d'utiliser les commandes Docker directement depuis votre terminal Linux dans WSL.

### 8. Configuration des proxys (si applicable)

Si vous êtes derrière un proxy d'entreprise :

1. **Settings** → **Resources** → **Proxies**
2. Activez **Manual proxy configuration**
3. Entrez vos paramètres proxy HTTP/HTTPS

## Configuration sur macOS

Sur macOS, Docker Desktop offre une interface de configuration similaire à Windows.

### 1. Accéder aux paramètres

1. Cliquez sur l'icône Docker dans la barre de menus (en haut)
2. Sélectionnez **Settings** (ou Préférences)

### 2. Configuration des ressources

**Pour Mac Intel :**
1. **Resources** → **Advanced**
2. Ajustez les paramètres :
   - **CPUs** : 2-4 cœurs (selon votre machine)
   - **Memory** : 4-8 Go
   - **Swap** : 1-2 Go
   - **Disk image size** : 64 Go minimum

**Pour Mac Apple Silicon (M1/M2/M3) :**
Docker utilise la technologie de virtualisation native d'Apple (Virtualization.framework). Les ressources sont généralement gérées automatiquement de manière efficace.

Vous pouvez toujours ajuster si nécessaire :
1. **Resources**
2. Les mêmes paramètres sont disponibles

**Recommandations selon votre Mac :**
- **8 Go de RAM** : Allouez 3-4 Go à Docker
- **16 Go de RAM** : Allouez 6-8 Go à Docker
- **32 Go de RAM ou plus** : Allouez 8-12 Go à Docker

### 3. Démarrage automatique

1. **General**
2. Cochez ✅ **Start Docker Desktop when you log in**

Pour les utilisateurs avancés qui veulent contrôler finement le démarrage, vous pouvez décocher cette option et lancer Docker manuellement quand vous en avez besoin.

### 4. Configuration du stockage

**Emplacement des données :**
Docker stocke ses données dans `~/Library/Containers/com.docker.docker/Data`

**Nettoyer l'espace disque :**
1. Icône 🐞 (Troubleshoot) dans le menu Docker
2. **Clean / Purge data**
3. Sélectionnez les éléments à supprimer

**Voir l'utilisation du disque :**
```bash
docker system df
```

### 5. Configuration réseau

**Dans Docker Desktop :**
1. **Resources** → **Network**
2. Docker utilise par défaut les paramètres réseau de macOS

**Si vous avez des VPN ou configurations réseau complexes :**
Vous pouvez ajuster les DNS manuellement si nécessaire.

### 6. Partage de fichiers

Docker Desktop doit avoir accès à certains dossiers pour les bind mounts :

1. **Resources** → **File Sharing**
2. Par défaut : `/Users`, `/Volumes`, `/private`, `/tmp`
3. Ajoutez d'autres dossiers si nécessaire

**Note :** Sur les Mac Apple Silicon récents, cette configuration est plus automatique.

### 7. Optimisation des performances (Apple Silicon)

Sur les Mac M1/M2/M3, Docker est déjà très performant grâce à la virtualisation native.

**Pour maximiser les performances :**
1. Utilisez des images multi-architecture ou natives ARM64 quand disponibles
2. Activez **VirtioFS** (devrait être activé par défaut) pour de meilleures performances de partage de fichiers
3. **Settings** → **General** → Cochez **Use Virtualization framework**

## Configurations communes à tous les systèmes

Certaines configurations sont utiles quel que soit votre système d'exploitation.

### 1. Configuration de Docker Hub

Si vous comptez télécharger beaucoup d'images ou publier vos propres images, créez un compte Docker Hub :

1. Allez sur [hub.docker.com](https://hub.docker.com)
2. Créez un compte gratuit
3. Dans votre terminal :
```bash
docker login
```
4. Entrez vos identifiants

**Avantages d'un compte Docker Hub :**
- Pas de limite de téléchargement d'images (compte anonyme : 100 pulls/6h)
- Possibilité de publier vos propres images
- Images privées (limitées en version gratuite)

### 2. Activer BuildKit

BuildKit est le nouveau moteur de build de Docker, plus rapide et plus efficace.

**Sur Linux, éditez `/etc/docker/daemon.json` :**
```json
{
  "features": {
    "buildkit": true
  }
}
```

**Sur Windows/macOS avec Docker Desktop :**
1. **Settings** → **Docker Engine**
2. Ajoutez dans la configuration JSON :
```json
{
  "features": {
    "buildkit": true
  }
}
```

**Ou activez-le temporairement pour une commande :**
```bash
DOCKER_BUILDKIT=1 docker build .
```

### 3. Configuration des alias (pratique)

Pour gagner du temps, créez des alias pour les commandes Docker fréquentes.

**Sur Linux/macOS, ajoutez à `~/.bashrc` ou `~/.zshrc` :**
```bash
# Alias Docker pratiques
alias dps='docker ps'
alias dpa='docker ps -a'
alias di='docker images'
alias dex='docker exec -it'
alias dlog='docker logs -f'
alias drm='docker rm'
alias drmi='docker rmi'
alias dstop='docker stop $(docker ps -q)'
```

**Sur Windows PowerShell, ajoutez à votre profil :**
```powershell
# Alias Docker
function dps { docker ps $args }
function dpa { docker ps -a $args }
function di { docker images $args }
```

Rechargez votre terminal et testez : `dps` au lieu de `docker ps`.

### 4. Configuration de Docker Compose

Docker Compose est maintenant intégré (plugin `docker compose`), mais vous pouvez le configurer :

**Vérifier la version :**
```bash
docker compose version
```

**Créer un fichier de configuration par défaut :**
Dans vos projets, créez toujours un fichier `.env` pour les variables d'environnement plutôt que de les hardcoder.

## Vérification de votre configuration

Une fois toutes ces configurations effectuées, vérifiez que tout fonctionne correctement.

### Test 1 : Commandes de base

```bash
# Doit fonctionner sans sudo (Linux) ou erreurs
docker version
docker info
docker ps
```

### Test 2 : Exécution d'un conteneur

```bash
# Tester avec un conteneur simple
docker run --rm hello-world
```

Si vous voyez le message de bienvenue, tout est parfait !

### Test 3 : Ressources allouées

```bash
# Voir les ressources système vues par Docker
docker info | grep -i "CPUs\|Memory"
```

Vérifiez que les valeurs correspondent à votre configuration.

### Test 4 : Stockage

```bash
# Voir l'utilisation du disque
docker system df
```

### Test 5 : Réseau

```bash
# Lister les réseaux
docker network ls
```

Vous devriez voir au moins le réseau `bridge` par défaut.

## Maintenance régulière recommandée

Maintenant que Docker est configuré, voici quelques bonnes pratiques de maintenance :

### Nettoyage régulier

Docker peut accumuler des images, conteneurs et volumes inutilisés. Nettoyez régulièrement :

```bash
# Nettoyer tout ce qui est inutilisé
docker system prune

# Version agressive (attention : supprime aussi les volumes !)
docker system prune -a --volumes
```

**Fréquence recommandée :** Une fois par semaine ou quand l'espace disque devient limité.

### Mise à jour de Docker

**Linux :**
```bash
sudo apt update && sudo apt upgrade docker-ce docker-ce-cli containerd.io
```

**Windows/macOS Docker Desktop :**
Docker Desktop se met à jour automatiquement, ou vous pouvez vérifier manuellement :
- Menu Docker → **Check for updates**

**Fréquence recommandée :** Tous les 1-2 mois ou quand une mise à jour importante sort.

### Surveillance de l'espace disque

```bash
# Voir l'espace utilisé en détail
docker system df -v
```

Si Docker utilise plus de 50 Go, c'est peut-être le moment de nettoyer.

## Résolution de problèmes de configuration

### Docker ne démarre pas

**Linux :**
```bash
# Voir les logs d'erreur
sudo journalctl -u docker

# Réinitialiser le service
sudo systemctl restart docker
```

**Windows/macOS :**
- Redémarrez Docker Desktop
- Si ça ne fonctionne pas : Menu Docker → **Troubleshoot** → **Reset to factory defaults**

### Erreur "Cannot connect to Docker daemon"

**Causes possibles :**
1. Docker n'est pas démarré : `sudo systemctl start docker`
2. Problème de permissions (Linux) : Vérifiez que vous êtes dans le groupe docker
3. Socket Docker inaccessible : Vérifiez les permissions de `/var/run/docker.sock`

### Docker est lent

**Solutions :**
1. Allouez plus de ressources (CPU/RAM) dans les paramètres
2. Nettoyez les images inutilisées : `docker system prune -a`
3. Sur Windows : Assurez-vous que WSL 2 est bien activé
4. Désactivez Kubernetes si vous ne l'utilisez pas

### Conflit de ports

Si Docker ne peut pas démarrer à cause de ports déjà utilisés :
```bash
# Voir quel processus utilise le port 80 (exemple)
sudo lsof -i :80  # Linux/macOS
netstat -ano | findstr :80  # Windows
```

## En résumé

La configuration post-installation de Docker est une étape importante qui améliore significativement votre expérience :

**Sur Linux :**
- Activez le démarrage automatique
- Configurez l'utilisation sans sudo
- Ajustez le stockage et les logs si nécessaire

**Sur Windows :**
- Allouez les bonnes ressources dans Docker Desktop
- Configurez WSL 2 correctement
- Activez le démarrage automatique

**Sur macOS :**
- Allouez suffisamment de ressources
- Utilisez la virtualisation native sur Apple Silicon
- Configurez le partage de fichiers

**Pour tous :**
- Créez un compte Docker Hub
- Activez BuildKit pour de meilleures performances
- Créez des alias pour les commandes fréquentes
- Planifiez une maintenance régulière

Votre environnement Docker est maintenant optimisé et prêt pour un usage intensif. Dans la prochaine section, nous allons vérifier en détail que tout fonctionne parfaitement avant de passer aux concepts fondamentaux.

⏭️ [Vérification de l'installation](/02-installation-et-configuration/03-verification-de-linstallation.md)
