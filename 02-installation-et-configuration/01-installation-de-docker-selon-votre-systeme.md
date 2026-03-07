🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 2.1 Installation de Docker selon votre système d'exploitation

## Introduction

L'installation de Docker est la première étape pratique de votre apprentissage. La bonne nouvelle, c'est que Docker a énormément simplifié ce processus au fil des années. Que vous soyez sur Windows, macOS ou Linux, vous aurez Docker opérationnel en quelques minutes.

Cette section vous guidera pas à pas dans l'installation de Docker sur votre système d'exploitation. Nous couvrirons les prérequis, les différentes options disponibles, et les étapes détaillées pour chaque plateforme.

## Choix entre Docker Desktop et Docker Engine

Avant de commencer l'installation, il est important de comprendre les deux principales options :

### Docker Desktop

**Qu'est-ce que c'est ?**
Docker Desktop est une application complète avec interface graphique qui inclut :
- Docker Engine (le moteur)
- Docker CLI (ligne de commande)
- Docker Compose
- Une interface graphique (GUI)
- Kubernetes (optionnel)
- Des outils de développement intégrés

**Pour qui ?**
- **Windows** : Fortement recommandé (gère automatiquement la VM Linux nécessaire)
- **macOS** : Fortement recommandé (gère automatiquement la VM Linux nécessaire)
- **Linux** : Optionnel (vous pouvez utiliser Docker Engine directement)

**Avantages :**
- Installation simple "tout-en-un"
- Interface graphique conviviale
- Mises à jour automatiques
- Parfait pour les débutants

### Docker Engine

**Qu'est-ce que c'est ?**
Docker Engine est le moteur Docker de base, sans interface graphique. Utilisation uniquement en ligne de commande.

**Pour qui ?**
- **Linux** : Option native et performante
- Serveurs et environnements de production
- Utilisateurs avancés qui préfèrent la ligne de commande

**Avantages :**
- Plus léger (pas de GUI)
- Performances optimales sur Linux
- Contrôle total

### Quelle option choisir ?

| Système | Recommandation | Raison |
|---------|----------------|--------|
| **Windows** | Docker Desktop | Nécessaire pour gérer la virtualisation |
| **macOS** | Docker Desktop | Nécessaire pour gérer la virtualisation |
| **Linux (débutant)** | Docker Desktop | Interface graphique pratique |
| **Linux (avancé)** | Docker Engine | Performance optimale |
| **Serveurs** | Docker Engine | Pas besoin de GUI |

## Installation sur Windows

### Prérequis système

Avant d'installer Docker Desktop sur Windows, vérifiez que votre système remplit ces conditions :

**Configuration minimale :**
- Windows 10 64-bit : version 21H2 ou ultérieure (Home, Pro, Enterprise, Education)
- Windows 11 64-bit
- 4 Go de RAM minimum (8 Go recommandés)
- Virtualisation matérielle activée dans le BIOS

**Technologies requises :**
- WSL 2 (Windows Subsystem for Linux 2)
- Hyper-V et Containers (activés automatiquement par l'installateur)

### Vérifier la virtualisation

La virtualisation doit être activée dans votre BIOS. Pour vérifier :

1. Ouvrez le **Gestionnaire des tâches** (Ctrl + Shift + Échap)
2. Allez dans l'onglet **Performance**
3. Cliquez sur **Processeur**
4. Regardez en bas à droite : "Virtualisation" doit indiquer **Activé**

Si elle est désactivée, vous devrez l'activer dans le BIOS de votre ordinateur (la procédure varie selon le fabricant).

### Étapes d'installation

#### Étape 1 : Télécharger Docker Desktop

1. Rendez-vous sur le site officiel : [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
2. Cliquez sur **Download for Windows**
3. Le fichier `Docker Desktop Installer.exe` sera téléchargé (environ 500 Mo)

#### Étape 2 : Installer WSL 2

Docker Desktop nécessite WSL 2. Si vous ne l'avez pas déjà :

1. Ouvrez **PowerShell en tant qu'administrateur** (clic droit sur le menu Démarrer)
2. Exécutez cette commande :
```powershell
wsl --install
```
3. Redémarrez votre ordinateur si demandé

#### Étape 3 : Exécuter l'installateur Docker

1. Double-cliquez sur `Docker Desktop Installer.exe`
2. L'installateur se lance
3. Laissez les options par défaut cochées :
   - ✅ Use WSL 2 instead of Hyper-V (recommandé)
   - ✅ Add shortcut to desktop
4. Cliquez sur **OK**
5. L'installation prend 2-5 minutes
6. Cliquez sur **Close and restart** à la fin

#### Étape 4 : Premier démarrage

1. Après le redémarrage, Docker Desktop se lance automatiquement
2. Acceptez les conditions d'utilisation
3. Vous pouvez créer un compte Docker Hub (optionnel) ou cliquer sur **Skip**
4. Docker Desktop se configure (cela prend quelques instants)
5. Quand vous voyez "Docker Desktop is running", c'est prêt !

### Vérification de l'installation sur Windows

Ouvrez **PowerShell** ou **Command Prompt** et tapez :

```bash
docker --version
```

Vous devriez voir quelque chose comme :
```
Docker version 27.x.x, build xxxxxxx
```

Testez également :
```bash
docker run hello-world
```

Ce test télécharge une petite image et exécute un conteneur qui affiche un message de confirmation.

### Problèmes courants sur Windows

**"WSL 2 installation is incomplete"**
- Solution : Ouvrez PowerShell en admin et exécutez `wsl --update`

**"Hardware assisted virtualization is not available"**
- Solution : Activez la virtualisation dans votre BIOS

**Docker Desktop ne démarre pas**
- Vérifiez que Hyper-V est activé (Panneau de configuration → Programmes → Activer ou désactiver des fonctionnalités Windows)
- Redémarrez votre ordinateur

## Installation sur macOS

### Prérequis système

**Pour Mac avec processeur Intel :**
- macOS 11 (Big Sur) ou ultérieur
- Au moins 4 Go de RAM (8 Go recommandés)
- Processeur Intel à 64 bits

**Pour Mac avec processeur Apple Silicon (M1, M2, M3) :**
- macOS 11 (Big Sur) ou ultérieur
- Au moins 4 Go de RAM (8 Go recommandés)

Docker Desktop existe en deux versions : une pour Intel et une pour Apple Silicon. Choisissez la bonne version !

### Étapes d'installation

#### Étape 1 : Télécharger Docker Desktop

1. Allez sur [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
2. Cliquez sur **Download for Mac**
3. Choisissez la version appropriée :
   - **Mac with Intel chip** : pour les Mac Intel
   - **Mac with Apple silicon** : pour M1/M2/M3

Le fichier `Docker.dmg` sera téléchargé (environ 500-600 Mo).

#### Étape 2 : Installer Docker Desktop

1. Double-cliquez sur `Docker.dmg`
2. Une fenêtre s'ouvre avec l'icône Docker
3. **Glissez-déposez** l'icône Docker vers le dossier **Applications**
4. Attendez que la copie se termine
5. Éjectez le volume Docker.dmg

#### Étape 3 : Lancer Docker Desktop

1. Ouvrez le dossier **Applications**
2. Double-cliquez sur **Docker**
3. macOS peut afficher un avertissement de sécurité :
   - Cliquez sur **Ouvrir**
   - Si nécessaire, allez dans Préférences Système → Sécurité et cliquez sur **Ouvrir quand même**
4. Docker Desktop peut demander votre mot de passe administrateur (normal)
5. Acceptez les conditions d'utilisation
6. Le tutoriel d'introduction s'affiche (vous pouvez le faire ou le passer)

#### Étape 4 : Première utilisation

Docker Desktop apparaît dans votre barre de menus (en haut de l'écran) avec une icône de baleine.

Quand l'icône est stable (pas de mouvement), Docker est prêt !

### Vérification de l'installation sur macOS

Ouvrez le **Terminal** (Applications → Utilitaires → Terminal) et tapez :

```bash
docker --version
```

Vous devriez voir :
```
Docker version 27.x.x, build xxxxxxx
```

Testez ensuite :
```bash
docker run hello-world
```

Si vous voyez un message de bienvenue, tout fonctionne parfaitement !

### Problèmes courants sur macOS

**"Docker Desktop requires a newer version of macOS"**
- Solution : Mettez à jour macOS vers la dernière version

**Docker Desktop est très lent**
- Sur les Mac Intel avec peu de RAM, augmentez les ressources allouées :
  - Docker Desktop → Settings → Resources → Ajustez la RAM et CPU

**Erreur "Cannot connect to the Docker daemon"**
- Docker Desktop n'est pas démarré, lancez-le depuis Applications

## Installation sur Linux

Sur Linux, vous avez le choix entre Docker Desktop (avec interface graphique) et Docker Engine (ligne de commande uniquement). Nous allons couvrir les deux approches pour les distributions les plus populaires.

### Option 1 : Docker Desktop pour Linux

#### Distributions supportées

- Ubuntu 22.04 LTS ou ultérieur
- Debian 11 ou ultérieur
- Fedora 37 ou ultérieur

#### Installation sur Ubuntu/Debian

```bash
# Téléchargez le package DEB depuis le site Docker
# https://docs.docker.com/desktop/install/linux-install/

# Pour Ubuntu (exemple, remplacez le numéro de version par le dernier disponible)
wget https://desktop.docker.com/linux/main/amd64/docker-desktop-amd64.deb

# Installez le package
sudo apt-get update  
sudo apt-get install ./docker-desktop-amd64.deb  
```

> **Note :** Vérifiez toujours la dernière version disponible sur la [page de téléchargement officielle](https://docs.docker.com/desktop/install/linux-install/) avant de télécharger.

#### Lancement de Docker Desktop

```bash
systemctl --user start docker-desktop
```

Docker Desktop apparaîtra dans vos applications.

### Option 2 : Docker Engine pour Linux (Recommandé)

Cette méthode installe Docker Engine nativement, sans interface graphique. C'est l'approche préférée sur Linux pour sa performance et sa légèreté.

#### Installation sur Ubuntu

**Étape 1 : Nettoyer les anciennes versions**

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

**Étape 2 : Configurer le dépôt Docker**

```bash
# Mettre à jour les paquets et installer les prérequis
sudo apt-get update  
sudo apt-get install ca-certificates curl  

# Créer le répertoire pour les clés et ajouter la clé GPG officielle de Docker
sudo install -m 0755 -d /etc/apt/keyrings  
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc  
sudo chmod a+r /etc/apt/keyrings/docker.asc  

# Configurer le dépôt
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

**Étape 3 : Installer Docker Engine**

```bash
# Mettre à jour l'index des paquets
sudo apt-get update

# Installer Docker Engine, containerd et Docker Compose
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

**Étape 4 : Vérifier l'installation**

```bash
sudo docker run hello-world
```

#### Installation sur Debian

Les étapes sont identiques à Ubuntu, mais utilisez :

```bash
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc  
sudo chmod a+r /etc/apt/keyrings/docker.asc  

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Puis continuez avec `apt-get install docker-ce...`

#### Installation sur Fedora

```bash
# Installer les prérequis
sudo dnf -y install dnf-plugins-core

# Ajouter le dépôt Docker
sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo

# Installer Docker
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Démarrer Docker
sudo systemctl start docker

# Activer Docker au démarrage
sudo systemctl enable docker
```

#### Installation sur Arch Linux

```bash
# Docker est disponible dans les dépôts officiels
sudo pacman -S docker

# Démarrer et activer Docker
sudo systemctl start docker.service  
sudo systemctl enable docker.service  
```

> **Note :** Sur Arch Linux, Docker Compose est inclus en tant que plugin via le paquet `docker`. La commande `docker compose` (sans tiret) est disponible directement.

#### Installation sur CentOS/RHEL

```bash
# Installer les prérequis
sudo yum install -y yum-utils

# Ajouter le dépôt Docker
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# Installer Docker
sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Démarrer Docker
sudo systemctl start docker  
sudo systemctl enable docker  
```

### Vérification de l'installation sur Linux

Pour vérifier que Docker est correctement installé :

```bash
# Vérifier la version
docker --version

# Vérifier que le service tourne
sudo systemctl status docker

# Tester avec un conteneur
sudo docker run hello-world
```

### Utiliser Docker sans sudo (Linux uniquement)

Par défaut sur Linux, vous devez utiliser `sudo` pour exécuter les commandes Docker. Pour éviter cela :

```bash
# Créer le groupe docker (s'il n'existe pas)
sudo groupadd docker

# Ajouter votre utilisateur au groupe docker
sudo usermod -aG docker $USER

# Appliquer les changements (ou déconnectez-vous et reconnectez-vous)
newgrp docker

# Tester sans sudo
docker run hello-world
```

**Note de sécurité :** Ajouter un utilisateur au groupe docker lui donne des privilèges équivalents à root. Faites-le uniquement pour des utilisateurs de confiance.

## Tableau récapitulatif des installations

| Système | Méthode recommandée | Temps d'installation | Difficulté |
|---------|-------------------|---------------------|-----------|
| **Windows 10/11** | Docker Desktop | 10-15 min | Facile |
| **macOS Intel** | Docker Desktop | 5-10 min | Très facile |
| **macOS Apple Silicon** | Docker Desktop | 5-10 min | Très facile |
| **Ubuntu/Debian** | Docker Engine | 10-15 min | Moyenne |
| **Fedora** | Docker Engine | 10-15 min | Moyenne |
| **Arch Linux** | Docker Engine | 5 min | Facile |
| **CentOS/RHEL** | Docker Engine | 10-15 min | Moyenne |

## Après l'installation : Premières vérifications

Une fois Docker installé, effectuez ces vérifications de base :

### 1. Version de Docker

```bash
docker --version
```

Devrait afficher quelque chose comme : `Docker version 27.x.x, build xxxxxxx`

### 2. Informations système

```bash
docker info
```

Cette commande affiche des informations détaillées sur votre installation Docker.

### 3. Test avec hello-world

```bash
docker run hello-world
```

Ce conteneur de test confirme que Docker peut :
- Télécharger des images depuis Docker Hub
- Créer et exécuter un conteneur
- Afficher la sortie

Si vous voyez le message "Hello from Docker!", votre installation est parfaite !

### 4. Vérifier Docker Compose

Docker Compose est maintenant inclus avec Docker. Vérifiez :

```bash
docker compose version
```

## Ressources supplémentaires

### Documentation officielle

Pour aller plus loin ou résoudre des problèmes spécifiques :

- **Documentation Docker** : [https://docs.docker.com/](https://docs.docker.com/)
- **Installation Windows** : [https://docs.docker.com/desktop/install/windows-install/](https://docs.docker.com/desktop/install/windows-install/)
- **Installation Mac** : [https://docs.docker.com/desktop/install/mac-install/](https://docs.docker.com/desktop/install/mac-install/)
- **Installation Linux** : [https://docs.docker.com/engine/install/](https://docs.docker.com/engine/install/)

### Support communautaire

- **Docker Community Forums** : [https://forums.docker.com/](https://forums.docker.com/)
- **Docker Discord** : Communauté active pour poser des questions
- **Stack Overflow** : Tag `docker` pour des questions techniques

## En résumé

L'installation de Docker varie selon votre système d'exploitation, mais reste simple dans tous les cas :

**Windows et macOS** : Docker Desktop est la solution recommandée. Installation en quelques clics avec une interface graphique conviviale.

**Linux** : Vous avez le choix entre Docker Desktop (avec GUI) ou Docker Engine (performances optimales). Docker Engine est généralement préféré sur Linux.

**Points clés à retenir :**
- Sur Windows, WSL 2 est requis
- Sur macOS, choisissez la bonne version (Intel vs Apple Silicon)
- Sur Linux, Docker Engine offre les meilleures performances
- Le test `docker run hello-world` confirme que tout fonctionne

Maintenant que Docker est installé sur votre système, nous allons passer à la configuration post-installation pour optimiser votre environnement de travail.

⏭️ [Configuration post-installation](/02-installation-et-configuration/02-configuration-post-installation.md)
