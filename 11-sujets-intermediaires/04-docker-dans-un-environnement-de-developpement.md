🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 11.4 Docker dans un environnement de développement

## Introduction

Vous est-il déjà arrivé d'entendre : **"Ça marche sur ma machine !"** ? C'est l'une des phrases les plus frustrantes du développement logiciel. Un développeur a un projet qui fonctionne parfaitement sur son ordinateur, mais quand un collègue essaie de l'exécuter, c'est la catastrophe : erreurs de dépendances, versions incompatibles, configurations manquantes...

Docker peut transformer votre environnement de développement pour résoudre ces problèmes une fois pour toutes.

## Le problème des environnements de développement traditionnels

### Scénario typique sans Docker

**Jour 1** : Un nouveau développeur rejoint l'équipe
```
1. Cloner le dépôt Git ✅
2. Installer Python 3.9 (mais il a Python 3.11) 😰
3. Installer PostgreSQL (quelle version déjà ?) 😰
4. Installer Redis 😰
5. Configurer les variables d'environnement 😰
6. Installer les dépendances npm (erreurs de compilation) 😰
7. Lancer l'application... ça ne marche pas 😭
8. Déboguer pendant des heures 😭😭
9. Enfin ça fonctionne ! 🎉
10. Total : 1 journée perdue
```

### Scénario idéal avec Docker

**Jour 1** : Un nouveau développeur rejoint l'équipe
```
1. Cloner le dépôt Git ✅
2. Ouvrir dans VS Code
3. Cliquer sur "Reopen in Container"
4. Attendre 2 minutes
5. Commencer à coder ✅
Total : 5 minutes
```

## Qu'est-ce qu'un environnement de développement conteneurisé ?

Un environnement de développement conteneurisé signifie que **tout ce dont vous avez besoin pour développer** (langages, outils, dépendances, bases de données) tourne dans un ou plusieurs conteneurs Docker.

### Avantages majeurs

✅ **Environnement identique pour tous** : Tous les développeurs ont exactement le même setup  
✅ **Onboarding rapide** : Un nouveau développeur est opérationnel en minutes  
✅ **Isolation** : Chaque projet a son propre environnement sans conflits  
✅ **Reproductibilité** : "Ça marche sur ma machine" devient "Ça marche partout"  
✅ **Nettoyage facile** : Supprimer le conteneur = environnement propre  
✅ **Pas de pollution** : Votre machine reste propre

### Exemple concret

**Projet A** : Application Python 3.9 + PostgreSQL 13  
**Projet B** : Application Python 3.11 + MongoDB  
**Projet C** : Application Node.js 18 + Redis  

Sans Docker : Installation et gestion de tous ces outils en local, risques de conflits.  
Avec Docker : Chaque projet dans son conteneur, isolation totale.  

## Introduction à VS Code Dev Containers

VS Code Dev Containers est une extension de Visual Studio Code qui permet de développer **à l'intérieur** d'un conteneur Docker comme si vous développiez localement.

### Comment ça fonctionne ?

1. Vous ouvrez un projet dans VS Code
2. VS Code détecte la configuration Dev Container
3. VS Code construit et démarre le conteneur
4. VS Code connecte votre éditeur au conteneur
5. Vous développez normalement, mais tout s'exécute dans le conteneur

**Schéma simplifié** :
```
Votre Machine
  ├─ VS Code (interface)
  │    ↕️ (connexion)
  └─ Docker
       └─ Conteneur de développement
            ├─ Code source (volume monté)
            ├─ Outils de développement
            ├─ Extensions VS Code
            └─ Terminal
```

### Installation

**Prérequis** :
1. Docker installé et en cours d'exécution
2. Visual Studio Code installé

**Installation de l'extension** :
1. Ouvrir VS Code
2. Aller dans Extensions (Ctrl+Shift+X / Cmd+Shift+X)
3. Rechercher "Dev Containers"
4. Installer l'extension "Dev Containers" par Microsoft

## Votre premier Dev Container

### Méthode 1 : Configuration depuis un template

**Étape 1** : Créer un nouveau projet ou ouvrir un existant

**Étape 2** : Ouvrir la palette de commandes
- Windows/Linux : `Ctrl+Shift+P`
- macOS : `Cmd+Shift+P`

**Étape 3** : Chercher et sélectionner
```
Dev Containers: Add Dev Container Configuration Files...
```

**Étape 4** : Choisir un template
- Python 3
- Node.js
- Go
- Java
- PHP
- Ruby
- etc.

**Étape 5** : VS Code crée automatiquement :
```
.devcontainer/
  └── devcontainer.json
```

**Étape 6** : Rouvrir dans le conteneur
- Cliquer sur le bouton vert en bas à gauche de VS Code
- Sélectionner "Reopen in Container"

### Structure du fichier devcontainer.json

```json
{
  "name": "Mon Projet Python",
  "image": "mcr.microsoft.com/devcontainers/python:3.11",
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-python.python",
        "ms-python.vscode-pylance"
      ]
    }
  }
}
```

**Explications** :
- `name` : Nom de votre environnement
- `image` : Image Docker à utiliser
- `extensions` : Extensions VS Code à installer automatiquement

## Exemples de configurations par langage

### Projet Python avec Flask

**.devcontainer/devcontainer.json** :
```json
{
  "name": "Flask App",
  "image": "mcr.microsoft.com/devcontainers/python:3.11",

  "customizations": {
    "vscode": {
      "extensions": [
        "ms-python.python",
        "ms-python.vscode-pylance",
        "ms-python.black-formatter",
        "njpwerner.autodocstring"
      ],
      "settings": {
        "python.defaultInterpreterPath": "/usr/local/bin/python",
        "python.linting.enabled": true,
        "python.formatting.provider": "black"
      }
    }
  },

  "postCreateCommand": "pip install -r requirements.txt",

  "forwardPorts": [5000],

  "remoteUser": "vscode"
}
```

**Explications** :
- `postCreateCommand` : Commande exécutée après la création du conteneur
- `forwardPorts` : Ports à exposer automatiquement
- `remoteUser` : Utilisateur dans le conteneur

### Projet Node.js avec Express

**.devcontainer/devcontainer.json** :
```json
{
  "name": "Node.js App",
  "image": "mcr.microsoft.com/devcontainers/javascript-node:18",

  "customizations": {
    "vscode": {
      "extensions": [
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode",
        "christian-kohler.npm-intellisense"
      ],
      "settings": {
        "editor.formatOnSave": true,
        "editor.defaultFormatter": "esbenp.prettier-vscode"
      }
    }
  },

  "postCreateCommand": "npm install",

  "forwardPorts": [3000],

  "features": {
    "ghcr.io/devcontainers/features/node:1": {
      "version": "18"
    }
  }
}
```

### Projet avec plusieurs services (Full Stack)

**.devcontainer/docker-compose.yml** :
```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      - ..:/workspace:cached
    command: sleep infinity
    environment:
      - DATABASE_URL=postgresql://postgres:password@db:5432/myapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis

  db:
    image: postgres:15
    restart: unless-stopped
    environment:
      POSTGRES_PASSWORD: password
      POSTGRES_DB: myapp
    volumes:
      - postgres-data:/var/lib/postgresql/data

  redis:
    image: redis:alpine
    restart: unless-stopped

volumes:
  postgres-data:
```

**.devcontainer/devcontainer.json** :
```json
{
  "name": "Full Stack App",
  "dockerComposeFile": "docker-compose.yml",
  "service": "app",
  "workspaceFolder": "/workspace",

  "customizations": {
    "vscode": {
      "extensions": [
        "ms-python.python",
        "ms-azuretools.vscode-docker",
        "mtxr.sqltools",
        "mtxr.sqltools-driver-pg"
      ]
    }
  },

  "postCreateCommand": "pip install -r requirements.txt",

  "forwardPorts": [5000, 5432, 6379]
}
```

**.devcontainer/Dockerfile** :
```dockerfile
FROM python:3.11-slim

# Installation des dépendances système
RUN apt-get update && apt-get install -y \
    git \
    curl \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*

# Utilisateur non-root
ARG USERNAME=vscode  
ARG USER_UID=1000  
ARG USER_GID=$USER_UID  

RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \
    && apt-get update \
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

USER $USERNAME

WORKDIR /workspace
```

## Configuration avancée avec devcontainer.json

### Options de base

```json
{
  "name": "Mon Projet",
  "image": "ubuntu:22.04",

  // OU utiliser un Dockerfile
  "build": {
    "dockerfile": "Dockerfile",
    "context": "..",
    "args": {
      "NODE_VERSION": "18"
    }
  },

  // OU utiliser docker-compose
  "dockerComposeFile": "docker-compose.yml",
  "service": "app",
  "workspaceFolder": "/workspace"
}
```

### Commandes de cycle de vie

```json
{
  "onCreateCommand": "echo 'Conteneur créé'",
  "updateContentCommand": "git pull",
  "postCreateCommand": "npm install && npm run build",
  "postStartCommand": "npm run dev",
  "postAttachCommand": "echo 'Attaché au conteneur'"
}
```

**Quand sont exécutées ces commandes ?**
- `onCreateCommand` : Première création du conteneur
- `updateContentCommand` : Quand le contenu change
- `postCreateCommand` : Après la création (installation des dépendances)
- `postStartCommand` : À chaque démarrage du conteneur
- `postAttachCommand` : Quand VS Code se connecte au conteneur

### Variables d'environnement

```json
{
  "containerEnv": {
    "NODE_ENV": "development",
    "API_KEY": "dev-key-123",
    "DATABASE_URL": "postgresql://postgres:password@db:5432/myapp"
  },

  // Variables pour la machine hôte (pas le conteneur)
  "remoteEnv": {
    "PATH": "${containerEnv:PATH}:/custom/path"
  }
}
```

### Montage de volumes

```json
{
  "mounts": [
    // Monter le socket Docker (pour utiliser Docker dans Docker)
    "source=/var/run/docker.sock,target=/var/run/docker.sock,type=bind",

    // Monter un cache pour npm
    "source=npm-cache,target=/root/.npm,type=volume",

    // Monter un dossier local
    "source=${localEnv:HOME}/.ssh,target=/home/vscode/.ssh,readonly,type=bind"
  ]
}
```

### Ports et redirection

```json
{
  // Ports à rediriger automatiquement
  "forwardPorts": [3000, 5000, 8080],

  // Configuration détaillée des ports
  "portsAttributes": {
    "3000": {
      "label": "Frontend",
      "onAutoForward": "notify"
    },
    "5000": {
      "label": "API",
      "protocol": "https"
    }
  },

  // Ouvrir automatiquement dans le navigateur
  "otherPortsAttributes": {
    "onAutoForward": "openBrowser"
  }
}
```

### Features (fonctionnalités pré-packagées)

Les features sont des scripts réutilisables pour ajouter des outils :

```json
{
  "features": {
    // Installer Git
    "ghcr.io/devcontainers/features/git:1": {},

    // Installer Docker CLI
    "ghcr.io/devcontainers/features/docker-in-docker:2": {
      "version": "latest"
    },

    // Installer Node.js
    "ghcr.io/devcontainers/features/node:1": {
      "version": "18",
      "nodeGypDependencies": true
    },

    // Installer Python
    "ghcr.io/devcontainers/features/python:1": {
      "version": "3.11",
      "installTools": true
    },

    // Installer AWS CLI
    "ghcr.io/devcontainers/features/aws-cli:1": {},

    // Installer kubectl
    "ghcr.io/devcontainers/features/kubectl-helm-minikube:1": {}
  }
}
```

## Exemples de projets complets

### Application React + Node.js + PostgreSQL

**Structure du projet** :
```
mon-projet/
├── .devcontainer/
│   ├── devcontainer.json
│   └── docker-compose.yml
├── frontend/
│   ├── package.json
│   └── src/
├── backend/
│   ├── package.json
│   └── src/
└── docker-compose.yml (pour la production)
```

**.devcontainer/devcontainer.json** :
```json
{
  "name": "React + Node + PostgreSQL",
  "dockerComposeFile": "docker-compose.yml",
  "service": "workspace",
  "workspaceFolder": "/workspace",

  "customizations": {
    "vscode": {
      "extensions": [
        // React
        "dsznajder.es7-react-js-snippets",
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode",

        // Node.js
        "christian-kohler.npm-intellisense",

        // Database
        "mtxr.sqltools",
        "mtxr.sqltools-driver-pg",

        // Docker
        "ms-azuretools.vscode-docker"
      ],
      "settings": {
        "editor.formatOnSave": true,
        "editor.defaultFormatter": "esbenp.prettier-vscode"
      }
    }
  },

  "postCreateCommand": "cd frontend && npm install && cd ../backend && npm install",

  "forwardPorts": [3000, 5000, 5432],

  "portsAttributes": {
    "3000": {
      "label": "Frontend React"
    },
    "5000": {
      "label": "Backend API"
    },
    "5432": {
      "label": "PostgreSQL"
    }
  }
}
```

**.devcontainer/docker-compose.yml** :
```yaml
services:
  workspace:
    image: mcr.microsoft.com/devcontainers/javascript-node:18
    volumes:
      - ..:/workspace:cached
    command: sleep infinity
    network_mode: service:db
    environment:
      - DATABASE_URL=postgresql://postgres:password@localhost:5432/myapp
    depends_on:
      - db

  db:
    image: postgres:15
    restart: unless-stopped
    environment:
      POSTGRES_PASSWORD: password
      POSTGRES_DB: myapp
    volumes:
      - postgres-data:/var/lib/postgresql/data

volumes:
  postgres-data:
```

### Application Django + Redis + Celery

**.devcontainer/devcontainer.json** :
```json
{
  "name": "Django + Redis + Celery",
  "dockerComposeFile": "docker-compose.yml",
  "service": "app",
  "workspaceFolder": "/workspace",

  "customizations": {
    "vscode": {
      "extensions": [
        "ms-python.python",
        "ms-python.vscode-pylance",
        "batisteo.vscode-django",
        "wholroyd.jinja",
        "mtxr.sqltools",
        "mtxr.sqltools-driver-pg"
      ],
      "settings": {
        "python.defaultInterpreterPath": "/usr/local/bin/python",
        "python.linting.pylintEnabled": true,
        "python.formatting.provider": "black",
        "python.languageServer": "Pylance"
      }
    }
  },

  "postCreateCommand": "pip install -r requirements.txt && python manage.py migrate",

  "forwardPorts": [8000, 5432, 6379, 5555],

  "portsAttributes": {
    "8000": {
      "label": "Django"
    },
    "5555": {
      "label": "Celery Flower"
    }
  }
}
```

## Commandes et raccourcis utiles

### Commandes de la palette

Ouvrir la palette : `Ctrl+Shift+P` (Windows/Linux) ou `Cmd+Shift+P` (macOS)

**Commandes Dev Containers** :
- `Dev Containers: Reopen in Container` : Ouvrir dans le conteneur
- `Dev Containers: Rebuild Container` : Reconstruire le conteneur
- `Dev Containers: Rebuild Without Cache` : Reconstruire sans cache
- `Dev Containers: Reopen Locally` : Revenir à l'environnement local
- `Dev Containers: Show Container Log` : Voir les logs du conteneur
- `Dev Containers: Open Container Configuration` : Ouvrir devcontainer.json

### Indicateur de statut

En bas à gauche de VS Code, vous verrez :
- 🟦 **Dev Container: Nom** → Vous êtes dans le conteneur
- 🔵 **WSL: Ubuntu** → Vous êtes sur WSL
- Rien → Vous êtes en local

Cliquer dessus ouvre un menu rapide avec les actions disponibles.

## Fonctionnalités pratiques

### 1. Partage de configuration

Commitez le dossier `.devcontainer/` dans Git :
```bash
git add .devcontainer/  
git commit -m "Add dev container configuration"  
git push  
```

Maintenant, toute votre équipe peut utiliser le même environnement !

### 2. Terminal intégré

Le terminal dans VS Code s'exécute **dans le conteneur** :
```bash
# Dans le terminal VS Code (dans le conteneur)
python --version  # Version du conteneur  
npm --version     # Version du conteneur  
psql -U postgres -h db  # Connexion à la DB  
```

### 3. Debugging

Le débogueur VS Code fonctionne normalement dans le conteneur :

**launch.json** (pour Python) :
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Python: Flask",
      "type": "python",
      "request": "launch",
      "module": "flask",
      "env": {
        "FLASK_APP": "app.py",
        "FLASK_DEBUG": "1"
      },
      "args": ["run", "--host=0.0.0.0"],
      "jinja": true
    }
  ]
}
```

### 4. Extensions automatiques

Les extensions définies dans `devcontainer.json` sont installées automatiquement dans le conteneur.

### 5. Git fonctionne normalement

Git utilise vos credentials de la machine hôte :
```bash
git pull  
git add .  
git commit -m "Update"  
git push  
```

## Personnalisation pour l'équipe

### Fichiers de configuration partagés

**.devcontainer/devcontainer.json** (commité dans Git) :
```json
{
  "name": "Mon Projet",
  "image": "mcr.microsoft.com/devcontainers/python:3.11",

  "customizations": {
    "vscode": {
      // Extensions obligatoires pour tous
      "extensions": [
        "ms-python.python",
        "ms-python.vscode-pylance"
      ]
    }
  },

  "postCreateCommand": "pip install -r requirements.txt"
}
```

**.devcontainer/devcontainer.local.json** (non commité, .gitignore) :
```json
{
  "customizations": {
    "vscode": {
      // Extensions personnelles
      "extensions": [
        "vscode-icons-team.vscode-icons",
        "eamodio.gitlens"
      ]
    }
  }
}
```

## Bonnes pratiques

### 1. Utilisez des images officielles Microsoft

```json
{
  "image": "mcr.microsoft.com/devcontainers/python:3.11"
}
```

Ces images sont optimisées pour Dev Containers et incluent des outils utiles.

### 2. Créez un Dockerfile pour des besoins spécifiques

```dockerfile
# .devcontainer/Dockerfile
FROM mcr.microsoft.com/devcontainers/python:3.11

# Installation d'outils supplémentaires
RUN apt-get update && apt-get install -y \
    vim \
    tmux \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*

# Configuration personnalisée
COPY .bashrc /home/vscode/.bashrc
```

### 3. Utilisez des volumes pour le cache

```json
{
  "mounts": [
    "source=pip-cache,target=/home/vscode/.cache/pip,type=volume",
    "source=npm-cache,target=/home/vscode/.npm,type=volume"
  ]
}
```

Cela accélère les installations après un rebuild.

### 4. Documentez votre configuration

**README.md** :
```markdown
## Environnement de développement

Ce projet utilise VS Code Dev Containers.

### Prérequis
- Docker
- VS Code
- Extension "Dev Containers"

### Démarrage rapide
1. Cloner le projet
2. Ouvrir dans VS Code
3. Cliquer sur "Reopen in Container"
4. Attendre la construction (première fois : ~5 minutes)
5. Lancer l'application : `npm run dev`

### Services disponibles
- Frontend : http://localhost:3000
- API : http://localhost:5000
- PostgreSQL : localhost:5432
```

### 5. Versions explicites

```json
{
  "features": {
    "ghcr.io/devcontainers/features/node:1": {
      "version": "18.17.0"  // Version précise
    }
  }
}
```

Évite les surprises avec des mises à jour.

### 6. Testez avant de commiter

```bash
# Reconstruire complètement
Ctrl+Shift+P → Dev Containers: Rebuild Without Cache

# Vérifier que tout fonctionne
npm install  
npm run build  
npm test  
```

### 7. Séparez dev et production

**Pour le développement** : `.devcontainer/`  
**Pour la production** : `Dockerfile` à la racine  

Ne mélangez pas les deux !

## Alternatives et outils complémentaires

### GitHub Codespaces

GitHub Codespaces utilise la même technologie que Dev Containers, mais dans le cloud :

1. Allez sur votre dépôt GitHub
2. Cliquez sur "Code" → "Codespaces" → "New codespace"
3. Un environnement VS Code s'ouvre dans le navigateur
4. Utilise votre fichier `.devcontainer/devcontainer.json`

**Avantages** :
- Pas besoin d'installer Docker localement
- Machine puissante dans le cloud
- Accessible depuis n'importe où

### JetBrains (IntelliJ, PyCharm, etc.)

JetBrains supporte également les Dev Containers :

1. Ouvrir le projet
2. Aller dans "Settings" → "Docker"
3. Configurer Docker
4. Créer une configuration "Docker"

### Docker Compose standalone

Sans VS Code, vous pouvez utiliser Docker Compose directement :

```bash
# Lancer les services
docker compose -f .devcontainer/docker-compose.yml up -d

# Entrer dans le conteneur
docker exec -it <container_name> bash

# Développer avec votre éditeur préféré
vim app.py
```

## Problèmes courants et solutions

### Problème 1 : "Cannot connect to Docker daemon"

**Cause** : Docker n'est pas démarré.

**Solution** :
```bash
# Vérifier que Docker tourne
docker ps

# Sous Linux
sudo systemctl start docker

# Sous macOS/Windows
Lancer Docker Desktop
```

### Problème 2 : Construction très lente

**Causes** :
- Première construction (normal)
- Connexion internet lente
- Trop de fichiers copiés

**Solutions** :
```
# .dockerignore
node_modules/
__pycache__/
.git/
*.pyc
.venv/
dist/  
build/  
```

```json
{
  "mounts": [
    // Utiliser des volumes pour le cache
    "source=npm-cache,target=/root/.npm,type=volume"
  ]
}
```

### Problème 3 : Les changements de code ne se reflètent pas

**Cause** : Le code n'est pas monté correctement.

**Solution** :
```json
{
  "workspaceFolder": "/workspace",
  "workspaceMount": "source=${localWorkspaceFolder},target=/workspace,type=bind"
}
```

Vérifier que le volume est bien monté :
```bash
docker exec -it <container> ls /workspace
```

### Problème 4 : Extensions VS Code ne s'installent pas

**Solution** :
```bash
# Reconstruire le conteneur
Ctrl+Shift+P → Dev Containers: Rebuild Container
```

Ou installer manuellement :
```bash
code --install-extension ms-python.python
```

### Problème 5 : Permission denied sur les fichiers

**Cause** : Problème d'utilisateur/groupe.

**Solution dans le Dockerfile** :
```dockerfile
ARG USERNAME=vscode  
ARG USER_UID=1000  
ARG USER_GID=$USER_UID  

RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME

USER $USERNAME
```

**Dans devcontainer.json** :
```json
{
  "remoteUser": "vscode"
}
```

### Problème 6 : Port déjà utilisé

**Cause** : Un autre processus utilise le port.

**Solution** :
```bash
# Trouver le processus
lsof -i :3000

# Tuer le processus
kill <PID>
```

Ou changer le port dans `devcontainer.json` :
```json
{
  "forwardPorts": [3001, 5001]  // Ports différents
}
```

### Problème 7 : Conteneur trop volumineux

**Solution** :
```dockerfile
# Utiliser des images slim/alpine
FROM python:3.11-slim  # Au lieu de python:3.11

# Nettoyer après installation
RUN apt-get update && apt-get install -y git \
    && rm -rf /var/lib/apt/lists/*

# Multi-stage build si nécessaire
```

## Workflow quotidien

### Démarrage de la journée

```
1. Ouvrir VS Code
2. Ouvrir le projet
3. VS Code se reconnecte automatiquement au conteneur
4. Commencer à coder
```

### Nouveau développeur

```
1. git clone <repository>
2. Ouvrir dans VS Code
3. Cliquer "Reopen in Container"
4. ☕ Attendre 2-5 minutes (première fois)
5. Commencer à coder
```

### Mise à jour des dépendances

```bash
# Dans le terminal du conteneur
npm install <nouvelle-dépendance>

# Mettre à jour requirements.txt ou package.json
git add package.json  
git commit -m "Add new dependency"  
git push  

# Autres développeurs
git pull
# VS Code détecte le changement et propose de rebuild
```

### Changement de configuration

```bash
# Modifier .devcontainer/devcontainer.json
# Sauvegarder

# Rebuild
Ctrl+Shift+P → Dev Containers: Rebuild Container
```

## Astuces avancées

### 1. Utiliser dotfiles personnels

```json
{
  "dotfiles": {
    "repository": "https://github.com/username/dotfiles",
    "targetPath": "/home/vscode",
    "installCommand": "./install.sh"
  }
}
```

Vos configurations personnelles (`.bashrc`, `.vimrc`, etc.) sont automatiquement installées.

### 2. Scripts d'initialisation

**.devcontainer/post-create.sh** :
```bash
#!/bin/bash

# Installation des dépendances
pip install -r requirements.txt

# Création de la base de données
python manage.py migrate

# Chargement des données de test
python manage.py loaddata fixtures/test_data.json

# Message de bienvenue
echo "✅ Environnement prêt ! Lancez 'python manage.py runserver'"
```

```json
{
  "postCreateCommand": "bash .devcontainer/post-create.sh"
}
```

### 3. Profils multiples

**devcontainer-backend.json** : Configuration pour le backend  
**devcontainer-frontend.json** : Configuration pour le frontend  
**devcontainer-fullstack.json** : Configuration complète  

Choisir le profil au démarrage.

### 4. Variables secrets

**Ne jamais commiter de secrets !**

**.env** (gitignored) :
```
DATABASE_PASSWORD=secret123  
API_KEY=my-secret-key  
```

```json
{
  "runArgs": ["--env-file", ".env"]
}
```

## Comparaison avec d'autres approches

| Approche | Avantages | Inconvénients |
|----------|-----------|---------------|
| **Installation locale** | Simple, rapide | Conflits, "marche sur ma machine" |
| **Machines virtuelles** | Isolation complète | Lourdes, lentes, gourmandes en ressources |
| **Dev Containers** | Léger, rapide, reproductible | Nécessite Docker |
| **GitHub Codespaces** | Cloud, puissant, accessible partout | Coût, connexion internet requise |
| **GitPod** | Cloud, gratuit (limité) | Moins flexible |

## Résumé

Les Dev Containers transforment votre workflow de développement :

### Configuration initiale

1. **Installer** Docker et VS Code + extension Dev Containers
2. **Créer** `.devcontainer/devcontainer.json`
3. **Configurer** l'image, extensions, commandes
4. **Commiter** la configuration dans Git

### Utilisation quotidienne

1. **Ouvrir** le projet dans VS Code
2. **Cliquer** "Reopen in Container"
3. **Développer** normalement
4. **Tout s'exécute** dans le conteneur

### Bénéfices

✅ **Onboarding rapide** : 5 minutes au lieu de plusieurs heures  
✅ **Environnement identique** : Plus de "ça marche sur ma machine"  
✅ **Isolation** : Chaque projet dans son conteneur  
✅ **Reproductibilité** : Partage facile avec l'équipe  
✅ **Nettoyage simple** : Supprimer le conteneur = environnement propre

### Template minimal

```json
{
  "name": "Mon Projet",
  "image": "mcr.microsoft.com/devcontainers/python:3.11",
  "customizations": {
    "vscode": {
      "extensions": ["ms-python.python"]
    }
  },
  "postCreateCommand": "pip install -r requirements.txt",
  "forwardPorts": [5000]
}
```

Les Dev Containers sont devenus un **standard de l'industrie** pour le développement moderne. Ils éliminent les problèmes d'environnement et permettent aux équipes de se concentrer sur le code, pas sur la configuration.

🚀 **Commencez avec Dev Containers dès aujourd'hui** et dites adieu aux problèmes d'"environnement qui marche sur ma machine" !

⏭️ [Introduction au débogage de conteneurs](/11-sujets-intermediaires/05-introduction-au-debogage-de-conteneurs.md)
