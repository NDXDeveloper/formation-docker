🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 6.3 Les bind mounts

## Introduction

Vous maîtrisez maintenant les volumes Docker, la solution recommandée pour persister les données en production. Mais Docker offre une autre méthode de gestion des données, particulièrement utile en développement : les **bind mounts**.

Les bind mounts permettent de monter un dossier ou un fichier **spécifique de votre machine** directement dans un conteneur. C'est comme créer un portail entre votre ordinateur et le conteneur.

## Qu'est-ce qu'un bind mount ?

Un bind mount est une **liaison directe** entre un chemin sur votre machine hôte et un chemin dans le conteneur. Contrairement aux volumes qui sont gérés par Docker dans un emplacement caché, avec les bind mounts, **vous contrôlez exactement où sont stockées les données** sur votre machine.

### Analogie simple

Imaginez que vous avez :
- Un **dossier sur votre bureau** contenant votre code source
- Un **conteneur Docker** qui exécute votre application

Avec un bind mount, c'est comme si vous donniez au conteneur un **accès direct** à votre dossier. Tout changement que vous faites dans le dossier sur votre bureau est **immédiatement visible** dans le conteneur, et vice versa.

```
┌─────────────────────────────────────────────────────────┐
│                 VOTRE ORDINATEUR                        │
│                                                         │
│  /home/utilisateur/mon-projet/                          │
│  ├── app.js              ◄──────────────────┐           │
│  ├── package.json                           │           │
│  └── node_modules/                          │           │
│                                             │           │
│                                    Bind mount           │
│                                             │           │
│  ┌──────────────────────────────────────────┼────────┐  │
│  │        CONTENEUR DOCKER                  │        │  │
│  │                                          │        │  │
│  │        /app/  ◄──────────────────────────┘        │  │
│  │        ├── app.js                                 │  │
│  │        ├── package.json                           │  │
│  │        └── node_modules/                          │  │
│  │                                                   │  │
│  │  (Accès direct aux fichiers de l'hôte)            │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

## Différence fondamentale : Volumes vs Bind Mounts

Avant d'aller plus loin, clarifions la différence essentielle :

| Aspect | Volumes | Bind Mounts |
|--------|---------|-------------|
| **Géré par** | Docker | Vous (utilisateur) |
| **Emplacement** | Caché par Docker | Chemin que vous choisissez |
| **Création** | `docker volume create` | Aucune commande nécessaire |
| **Chemin complet** | Non requis | **Requis** (chemin absolu) |
| **Portabilité** | Excellente (fonctionne partout) | Dépend de la structure de fichiers de l'hôte |
| **Utilisation typique** | Production, données critiques | Développement, code source |
| **Performances** | Optimisées par Docker | Peuvent varier selon l'OS |
| **Visibilité** | Via commandes Docker | Directement sur votre disque |

### Résumé visuel

```
VOLUMES                          BIND MOUNTS
├── Géré par Docker              ├── Géré par vous
├── Emplacement caché            ├── Emplacement visible
├── docker volume create         ├── Juste un chemin
└── Production ✅                 └── Développement ✅
```

## Syntaxe des bind mounts

Il existe deux syntaxes pour créer un bind mount. Les deux fonctionnent, mais la syntaxe moderne est recommandée.

### Syntaxe classique (avec -v)

```bash
docker run -v /chemin/sur/hôte:/chemin/dans/conteneur image
```

### Syntaxe moderne (avec --mount)

```bash
docker run --mount type=bind,source=/chemin/sur/hôte,target=/chemin/dans/conteneur image
```

**Recommandation :** Utilisez la syntaxe `--mount` car elle est plus explicite et moins sujette aux erreurs.

> ⚠️ **Important :** Avec les bind mounts, vous **devez** utiliser des **chemins absolus**, pas des chemins relatifs !

## Créer un bind mount : Premier exemple

Créons un exemple simple avec un serveur web Nginx qui affiche une page HTML.

### Étape 1 : Préparer le dossier local

```bash
# Créer un dossier pour notre site web
mkdir ~/mon-site-web
cd ~/mon-site-web

# Créer une page HTML simple
echo '<h1>Bonjour depuis mon bind mount !</h1>' > index.html
```

### Étape 2 : Lancer Nginx avec un bind mount

```bash
docker run -d \
  --name mon-nginx \
  -p 8080:80 \
  -v ~/mon-site-web:/usr/share/nginx/html \
  nginx
```

**Explication :**
- `-v ~/mon-site-web:/usr/share/nginx/html` : Monte le dossier local dans le conteneur
- Nginx cherche les fichiers HTML dans `/usr/share/nginx/html`
- Notre dossier local est maintenant accessible à cet emplacement

### Étape 3 : Tester

Ouvrez votre navigateur et allez sur `http://localhost:8080`. Vous verrez votre page HTML !

### Étape 4 : Modifier en temps réel

Maintenant, modifions le fichier **sans redémarrer le conteneur** :

```bash
echo '<h1>Page modifiée en temps réel !</h1>' > ~/mon-site-web/index.html
```

Rechargez votre navigateur. La page a changé **instantanément** ! C'est la magie des bind mounts.

## Exemple pratique : Développement d'une application Node.js

Les bind mounts sont particulièrement puissants pour le développement. Voici un exemple concret.

### Structure du projet

```
mon-app-nodejs/
├── app.js
├── package.json
└── node_modules/
```

### Fichier app.js

```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Bonjour depuis Node.js !');
});

app.listen(3000, () => {
  console.log('Serveur démarré sur le port 3000');
});
```

### Lancer avec bind mount

```bash
docker run -d \
  --name dev-nodejs \
  -p 3000:3000 \
  -v "$(pwd)":/app \
  -w /app \
  node:18 \
  node app.js
```

**Explication ligne par ligne :**
- `-v "$(pwd)":/app` : Monte le répertoire courant dans `/app` du conteneur
- `-w /app` : Définit `/app` comme répertoire de travail
- `node app.js` : Lance l'application

> 💡 **Astuce :** `$(pwd)` est remplacé par le chemin absolu du répertoire courant.

### Modifier le code en temps réel

Avec certaines configurations (comme `nodemon` pour Node.js), vous pouvez modifier votre code et voir les changements immédiatement sans redémarrer le conteneur.

```bash
# Installer nodemon dans le projet
npm install --save-dev nodemon

# Relancer avec nodemon
docker run -d \
  --name dev-nodejs-hot \
  -p 3000:3000 \
  -v "$(pwd)":/app \
  -w /app \
  node:18 \
  npx nodemon app.js
```

Maintenant, chaque modification de `app.js` relancera automatiquement l'application !

## Bind mount en lecture seule

Parfois, vous voulez qu'un conteneur puisse **lire** des fichiers mais **pas les modifier**. C'est utile pour des fichiers de configuration sensibles.

### Syntaxe avec -v

```bash
docker run -v /chemin/hôte:/chemin/conteneur:ro image
```

Le `:ro` signifie **read-only** (lecture seule).

### Syntaxe avec --mount

```bash
docker run --mount type=bind,source=/chemin/hôte,target=/chemin/conteneur,readonly image
```

### Exemple : Configuration en lecture seule

```bash
# Créer un fichier de configuration
echo "debug=false" > ~/config.txt

# Monter en lecture seule
docker run -d \
  --name app-avec-config \
  -v ~/config.txt:/app/config.txt:ro \
  mon-application
```

Si le conteneur tente de modifier `/app/config.txt`, il recevra une erreur de permission.

## Bind mount d'un fichier unique

Vous n'êtes pas limité aux dossiers. Vous pouvez monter un fichier spécifique.

### Exemple : Fichier de configuration Nginx

```bash
# Créer une configuration personnalisée
cat > ~/nginx.conf << EOF
server {
    listen 80;
    location / {
        return 200 "Configuration personnalisée!";
    }
}
EOF

# Monter uniquement ce fichier
docker run -d \
  --name nginx-custom \
  -p 8080:80 \
  -v ~/nginx.conf:/etc/nginx/conf.d/default.conf:ro \
  nginx
```

## Cas d'usage courants des bind mounts

### 1. Développement de code source

**Scénario :** Vous développez une application et voulez voir les modifications instantanément.

```bash
docker run -d \
  --name dev-env \
  -v ~/mon-projet:/workspace \
  -w /workspace \
  mon-environnement-dev
```

**Avantages :**
- Modifications instantanées
- Utilisation de votre éditeur préféré sur l'hôte
- Pas besoin de reconstruire l'image à chaque changement

### 2. Partage de fichiers de configuration

**Scénario :** Configuration d'une base de données ou d'un serveur web.

```bash
docker run -d \
  --name postgres-custom \
  -v ~/postgresql.conf:/etc/postgresql/postgresql.conf:ro \
  postgres
```

### 3. Accès aux logs en temps réel

**Scénario :** Vous voulez consulter les logs d'une application directement sur votre machine.

```bash
docker run -d \
  --name app-avec-logs \
  -v ~/logs:/var/log/app \
  mon-application
```

Les logs apparaissent en temps réel dans `~/logs` sur votre machine.

### 4. Tests avec données locales

**Scénario :** Vous testez une application avec un jeu de données spécifique.

```bash
docker run -d \
  --name test-data \
  -v ~/donnees-test:/data \
  mon-application-test
```

## Pièges et erreurs courantes

### 1. Oublier le chemin absolu

❌ **Erreur courante :**

```bash
docker run -v mon-dossier:/app nginx
```

Docker va chercher un volume nommé "mon-dossier" au lieu d'utiliser un bind mount.

✅ **Correct :**

```bash
docker run -v /home/user/mon-dossier:/app nginx
# ou
docker run -v $(pwd)/mon-dossier:/app nginx
# ou
docker run -v ~/mon-dossier:/app nginx
```

### 2. Permissions de fichiers

Sur Linux, les permissions peuvent causer des problèmes si l'utilisateur dans le conteneur n'a pas les droits d'accès.

**Symptôme :** Erreurs "Permission denied" dans le conteneur.

**Solution :** Ajuster les permissions du dossier sur l'hôte ou configurer l'utilisateur du conteneur.

```bash
# Donner les permissions nécessaires
chmod -R 755 ~/mon-dossier

# Ou spécifier l'utilisateur dans Docker
docker run --user $(id -u):$(id -g) -v ~/mon-dossier:/app image
```

### 3. Dossier n'existe pas sur l'hôte

Si le chemin source n'existe pas, Docker **créera un dossier vide** automatiquement, ce qui peut ne pas être votre intention.

✅ **Bonne pratique :** Créez toujours le dossier avant de faire le bind mount.

```bash
mkdir -p ~/mon-dossier
docker run -v ~/mon-dossier:/app nginx
```

### 4. Conflits avec des fichiers existants

Si le dossier de destination dans le conteneur existe déjà et contient des fichiers, le bind mount **masquera** ces fichiers.

**Exemple :**

```bash
# Le conteneur a des fichiers dans /app
# Le bind mount va "cacher" ces fichiers
docker run -v ~/dossier-vide:/app image
```

Les fichiers originaux du conteneur ne seront plus accessibles tant que le bind mount est actif.

### 5. Performances sur Windows et macOS

Sur Windows et macOS, Docker Desktop utilise une machine virtuelle, ce qui peut ralentir les bind mounts, surtout avec beaucoup de fichiers.

**Solutions :**
- Utilisez des volumes Docker pour de meilleures performances
- Montez uniquement les dossiers nécessaires
- Considérez Docker Desktop avec WSL2 (Windows) pour de meilleures performances

## Bind mounts avec Docker Compose

Les bind mounts sont très utilisés dans les fichiers `docker-compose.yml` pour le développement.

### Exemple : Application web complète

```yaml
version: '3.8'

services:
  web:
    image: nginx
    ports:
      - "8080:80"
    volumes:
      - ./site-web:/usr/share/nginx/html
      - ./nginx.conf:/etc/nginx/nginx.conf:ro

  app:
    image: node:18
    working_dir: /app
    command: npm start
    ports:
      - "3000:3000"
    volumes:
      - ./mon-app:/app
      - /app/node_modules  # Volume anonyme pour éviter de monter node_modules
    environment:
      - NODE_ENV=development
```

**Points importants :**
- Les chemins relatifs fonctionnent dans `docker-compose.yml` (relatifs au fichier)
- Vous pouvez mélanger bind mounts et volumes
- Le volume anonyme `/app/node_modules` évite de monter ce dossier depuis l'hôte

## Sécurité et bind mounts

Les bind mounts donnent au conteneur un accès direct à votre système de fichiers. Voici quelques précautions :

### 1. Montez uniquement ce qui est nécessaire

❌ **Dangereux :**

```bash
docker run -v /:/host ubuntu
```

Cela donne accès à **tout votre système** au conteneur !

✅ **Sécurisé :**

```bash
docker run -v ~/mon-projet:/app ubuntu
```

Montez uniquement le dossier spécifique nécessaire.

### 2. Utilisez la lecture seule quand possible

```bash
docker run -v ~/config:/app/config:ro ubuntu
```

### 3. Attention aux conteneurs non fiables

Ne montez jamais de données sensibles dans des conteneurs provenant de sources non fiables.

### 4. Évitez les bind mounts en production

En production, privilégiez les volumes Docker qui sont plus sécurisés et gérés par Docker.

## Commandes utiles avec bind mounts

### Inspecter les montages d'un conteneur

```bash
docker inspect mon-conteneur
```

Cherchez la section `"Mounts"` dans la sortie JSON :

```json
"Mounts": [
    {
        "Type": "bind",
        "Source": "/home/user/mon-projet",
        "Destination": "/app",
        "Mode": "",
        "RW": true,
        "Propagation": "rprivate"
    }
]
```

### Vérifier les montages actifs

```bash
docker inspect -f '{{ .Mounts }}' mon-conteneur
```

## Quand utiliser les bind mounts ?

### ✅ Utilisez les bind mounts pour :

1. **Développement local**
   - Code source que vous modifiez fréquemment
   - Configuration en cours d'ajustement
   - Tests rapides

2. **Accès aux fichiers de l'hôte**
   - Logs que vous voulez consulter
   - Données de test spécifiques
   - Scripts de build

3. **Intégration avec l'environnement local**
   - Outils de développement (IDE, debuggers)
   - Fichiers de configuration système
   - Certificats SSL locaux

### ❌ N'utilisez PAS les bind mounts pour :

1. **Production**
   - Préférez les volumes Docker
   - Meilleure portabilité
   - Meilleures performances

2. **Données critiques**
   - Bases de données en production
   - Données utilisateur permanentes
   - Fichiers système importants

3. **Conteneurs portables**
   - Images qui doivent fonctionner partout
   - CI/CD
   - Environnements partagés

## Bonnes pratiques

### 1. Structure de projet claire

Organisez vos fichiers pour faciliter les bind mounts :

```
mon-projet/
├── docker-compose.yml
├── app/
│   ├── src/
│   └── config/
├── data/
└── logs/
```

### 2. Documentez vos bind mounts

Dans votre README ou documentation :

```markdown
## Développement local

Bind mounts utilisés :
- `./app/src` → `/app/src` : Code source de l'application
- `./logs` → `/var/log/app` : Logs de l'application
- `./data` → `/data` : Données de test (non versionnées)
```

### 3. Utilisez .dockerignore

Créez un fichier `.dockerignore` pour éviter de copier des fichiers inutiles :

```
node_modules
.git
.env
*.log
```

### 4. Combinez avec les volumes

N'hésitez pas à mélanger bind mounts et volumes :

```bash
docker run -d \
  -v $(pwd)/src:/app/src \      # Bind mount pour le code
  -v app-data:/app/data \        # Volume pour les données
  mon-app
```

### 5. Testez sur différents OS

Si votre équipe utilise différents systèmes d'exploitation, testez que vos bind mounts fonctionnent partout :

- Chemins différents (Windows vs Linux/macOS)
- Permissions de fichiers
- Performances

## À retenir

🔑 **Points clés :**

- Les **bind mounts** créent un lien direct entre un dossier de l'hôte et le conteneur
- Vous **contrôlez l'emplacement** exact des fichiers sur votre machine
- Nécessite des **chemins absolus** (ou utiliser `$(pwd)`)
- Parfait pour le **développement** et les modifications en temps réel
- Peut monter des **dossiers ou fichiers** individuels
- Option `:ro` pour le **read-only** (lecture seule)
- Attention aux **permissions** et à la **sécurité**
- **Déconseillé en production** (préférez les volumes)
- Performances variables selon l'OS (moins bon sur Windows/macOS)

## Prochaine étape

Maintenant que vous connaissez à la fois les volumes et les bind mounts, la section suivante (6.4) vous aidera à **choisir la bonne solution** selon votre situation. Nous comparerons en détail ces deux approches pour que vous puissiez prendre des décisions éclairées dans vos projets.

⏭️ [Comparaison et choix entre volumes et bind mounts](/06-gestion-des-donnees/04-comparaison-et-choix-entre-volumes-et-bind-mounts.md)
