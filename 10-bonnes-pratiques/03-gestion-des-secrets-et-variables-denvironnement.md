🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 10.3 Gestion des secrets et variables d'environnement

## Introduction

Les applications ont souvent besoin d'informations sensibles pour fonctionner : mots de passe de base de données, clés API, tokens d'authentification, certificats SSL, etc. La façon dont vous gérez ces informations sensibles (appelées "secrets") est cruciale pour la sécurité de vos applications.

Dans cette section, nous allons apprendre à gérer correctement les secrets et les variables d'environnement dans Docker, en évitant les erreurs courantes qui pourraient compromettre la sécurité de vos applications.

## Comprendre les concepts de base

### Variables d'environnement

Les variables d'environnement sont des valeurs configurables qui permettent de personnaliser le comportement d'une application sans modifier son code.

**Exemples de variables d'environnement :**
- `NODE_ENV=production` : Mode d'exécution de l'application
- `PORT=8080` : Port d'écoute de l'application
- `LOG_LEVEL=info` : Niveau de journalisation
- `DATABASE_HOST=db.example.com` : Adresse de la base de données

### Secrets

Les secrets sont des informations sensibles qui doivent être protégées et ne jamais être exposées publiquement.

**Exemples de secrets :**
- Mots de passe de base de données
- Clés API et tokens
- Certificats SSL/TLS et clés privées
- Clés de chiffrement
- Credentials d'accès à des services tiers

### La différence fondamentale

**Variables d'environnement :** Peuvent être publiques ou semi-publiques, utilisées pour la configuration
**Secrets :** Doivent toujours rester confidentiels, utilisés pour l'authentification et la sécurité

## Les erreurs à NE JAMAIS commettre

### ❌ Erreur n°1 : Écrire les secrets dans le Dockerfile

**Ne JAMAIS faire cela :**

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .

# DANGEREUX ! Le mot de passe est visible dans l'image
ENV DATABASE_PASSWORD=motdepasse123
ENV API_KEY=sk-1234567890abcdef

CMD ["node", "server.js"]
```

**Pourquoi c'est dangereux :**
- Les secrets restent dans l'image pour toujours
- Toute personne avec accès à l'image peut les voir avec `docker history`
- Ils seront visibles dans votre registre Docker
- Impossible de les changer sans reconstruire l'image

**Vérification :**

```bash
# N'importe qui peut voir les secrets
docker history mon-image:latest
docker inspect mon-image:latest
```

### ❌ Erreur n°2 : Commiter les secrets dans Git

**Ne JAMAIS faire cela :**

```yaml
# docker-compose.yml - NE PAS COMMITER AVEC DES SECRETS
version: '3.8'
services:
  app:
    image: mon-app
    environment:
      DATABASE_PASSWORD: motdepasse123  # DANGEREUX !
      API_KEY: sk-1234567890abcdef      # DANGEREUX !
```

**Pourquoi c'est dangereux :**
- L'historique Git conserve les secrets pour toujours
- Même si vous supprimez le fichier plus tard, il reste dans l'historique
- Accessible à tous ceux qui ont accès au dépôt
- Les dépôts publics exposent vos secrets au monde entier

### ❌ Erreur n°3 : Passer les secrets en arguments de build

**Ne JAMAIS faire cela :**

```dockerfile
FROM node:18-alpine
ARG DATABASE_PASSWORD  # DANGEREUX ! Reste dans l'historique de l'image
WORKDIR /app
COPY . .
CMD ["node", "server.js"]
```

```bash
# Les arguments de build restent visibles
docker build --build-arg DATABASE_PASSWORD=secret123 -t mon-app .
docker history mon-app  # Le secret est visible !
```

### ❌ Erreur n°4 : Logger les secrets

**Ne JAMAIS faire cela :**

```javascript
// Application Node.js - NE JAMAIS LOGGER LES SECRETS
const password = process.env.DATABASE_PASSWORD;
console.log('Connexion avec le mot de passe:', password);  // DANGEREUX !
```

Les logs peuvent être stockés, sauvegardés, ou transmis à des services externes.

## Bonnes pratiques pour les variables d'environnement

### 1. Passer les variables au runtime

Au lieu de les définir dans le Dockerfile, passez-les lors du lancement du conteneur.

```bash
# Passer une variable d'environnement au lancement
docker run -e NODE_ENV=production -e PORT=8080 mon-app

# Passer plusieurs variables
docker run \
  -e NODE_ENV=production \
  -e PORT=8080 \
  -e LOG_LEVEL=info \
  mon-app
```

### 2. Utiliser un fichier d'environnement (.env)

Créez un fichier `.env` pour stocker vos variables :

```bash
# .env
NODE_ENV=production
PORT=8080
LOG_LEVEL=info
DATABASE_HOST=db.example.com
```

**Important :** Ajoutez `.env` à votre `.gitignore` !

```bash
# .gitignore
.env
.env.local
.env.production
*.env
```

Utilisation avec Docker :

```bash
# Charger les variables depuis un fichier
docker run --env-file .env mon-app
```

Utilisation avec Docker Compose :

```yaml
# docker-compose.yml
version: '3.8'
services:
  app:
    image: mon-app
    env_file:
      - .env
```

### 3. Créer un fichier d'exemple

Créez un fichier `.env.example` avec des valeurs fictives à commiter dans Git :

```bash
# .env.example - À commiter dans Git
NODE_ENV=development
PORT=8080
LOG_LEVEL=info
DATABASE_HOST=localhost
DATABASE_PASSWORD=votre_mot_de_passe_ici
API_KEY=votre_cle_api_ici
```

Les développeurs peuvent copier ce fichier et le remplir avec leurs vraies valeurs :

```bash
cp .env.example .env
# Puis éditer .env avec les vraies valeurs
```

### 4. Utiliser des valeurs par défaut

Dans votre Dockerfile, définissez des valeurs par défaut non sensibles :

```dockerfile
FROM node:18-alpine
WORKDIR /app

# Valeurs par défaut pour le développement (non sensibles)
ENV NODE_ENV=development
ENV PORT=3000
ENV LOG_LEVEL=debug

COPY package*.json ./
RUN npm ci --only=production
COPY . .

CMD ["node", "server.js"]
```

Ces valeurs peuvent être surchargées au runtime :

```bash
# Surcharger les valeurs par défaut
docker run -e NODE_ENV=production -e PORT=8080 mon-app
```

## Bonnes pratiques pour les secrets

### 1. Utiliser les variables d'environnement au runtime (solution simple)

Pour les environnements simples ou de développement :

```bash
# Passer un secret au lancement (non visible dans l'image)
docker run -e DATABASE_PASSWORD=secret123 mon-app
```

**Avantages :**
- Simple à mettre en œuvre
- Le secret n'est pas dans l'image

**Inconvénients :**
- Visible dans `docker inspect`
- Visible avec `ps aux` sur l'hôte
- Peut apparaître dans les logs système

### 2. Utiliser Docker Secrets (Docker Swarm)

Docker Secrets est la solution native de Docker pour gérer les secrets de manière sécurisée. Elle nécessite Docker Swarm.

#### Initialiser Docker Swarm

```bash
# Initialiser Swarm (même sur une seule machine)
docker swarm init
```

#### Créer un secret

```bash
# Créer un secret depuis la ligne de commande
echo "motdepasse123" | docker secret create db_password -

# Créer un secret depuis un fichier
echo "motdepasse123" > /tmp/db_password.txt
docker secret create db_password /tmp/db_password.txt
rm /tmp/db_password.txt  # Supprimer le fichier immédiatement

# Lister les secrets
docker secret ls
```

#### Utiliser un secret dans un service

```bash
# Créer un service avec un secret
docker service create \
  --name mon-app \
  --secret db_password \
  mon-image
```

#### Accéder au secret dans l'application

Les secrets sont montés comme des fichiers dans `/run/secrets/` :

```javascript
// Application Node.js
const fs = require('fs');

// Lire le secret depuis le fichier
const dbPassword = fs.readFileSync('/run/secrets/db_password', 'utf8').trim();

// Utiliser le secret
const dbConfig = {
  host: process.env.DATABASE_HOST,
  user: process.env.DATABASE_USER,
  password: dbPassword  // Secret sécurisé
};
```

```python
# Application Python
def read_secret(secret_name):
    try:
        with open(f'/run/secrets/{secret_name}', 'r') as secret_file:
            return secret_file.read().strip()
    except IOError:
        return None

db_password = read_secret('db_password')
```

#### Avec Docker Compose et Swarm

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    image: mon-app
    secrets:
      - db_password
      - api_key
    environment:
      DATABASE_HOST: db
      DATABASE_USER: myuser

secrets:
  db_password:
    external: true
  api_key:
    external: true
```

Déployer avec Swarm :

```bash
# Créer les secrets
echo "motdepasse123" | docker secret create db_password -
echo "sk-1234567890" | docker secret create api_key -

# Déployer le stack
docker stack deploy -c docker-compose.yml mon-stack
```

**Avantages :**
- Secrets chiffrés au repos et en transit
- Accessibles uniquement aux services autorisés
- Gestion centralisée
- Ne passent jamais par les logs

**Inconvénients :**
- Nécessite Docker Swarm
- Plus complexe à mettre en place

### 3. Utiliser Docker Compose avec des fichiers secrets (développement)

Pour le développement local sans Swarm, Docker Compose peut lire les secrets depuis des fichiers.

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    image: mon-app
    secrets:
      - db_password
      - api_key
    environment:
      DATABASE_HOST: db
      DATABASE_USER: myuser

secrets:
  db_password:
    file: ./secrets/db_password.txt
  api_key:
    file: ./secrets/api_key.txt
```

Structure des dossiers :

```
projet/
├── docker-compose.yml
├── secrets/
│   ├── db_password.txt
│   └── api_key.txt
└── .gitignore
```

`.gitignore` :

```
secrets/
.env
```

**Important :** Ne commitez JAMAIS le dossier `secrets/` dans Git !

### 4. Utiliser des gestionnaires de secrets externes

Pour les environnements de production, utilisez des gestionnaires de secrets professionnels.

#### HashiCorp Vault

Vault est un outil populaire pour gérer les secrets.

```javascript
// Exemple avec Node.js et Vault
const vault = require('node-vault')({
  endpoint: process.env.VAULT_ADDR,
  token: process.env.VAULT_TOKEN
});

// Récupérer un secret depuis Vault
const secrets = await vault.read('secret/data/myapp');
const dbPassword = secrets.data.data.db_password;
```

#### AWS Secrets Manager

Pour les applications sur AWS :

```bash
# Dockerfile
FROM node:18-alpine
RUN npm install aws-sdk
COPY . .
CMD ["node", "server.js"]
```

```javascript
// Application
const AWS = require('aws-sdk');
const secretsManager = new AWS.SecretsManager({
  region: process.env.AWS_REGION
});

// Récupérer le secret
const data = await secretsManager.getSecretValue({
  SecretId: 'myapp/database/password'
}).promise();

const dbPassword = JSON.parse(data.SecretString).password;
```

#### Azure Key Vault

Pour les applications sur Azure :

```javascript
// Application
const { DefaultAzureCredential } = require('@azure/identity');
const { SecretClient } = require('@azure/keyvault-secrets');

const credential = new DefaultAzureCredential();
const client = new SecretClient(
  process.env.KEY_VAULT_URL,
  credential
);

// Récupérer le secret
const secret = await client.getSecret('db-password');
const dbPassword = secret.value;
```

## Exemples pratiques complets

### Exemple 1 : Application Node.js avec Express

#### Structure du projet

```
mon-projet/
├── Dockerfile
├── docker-compose.yml
├── .env.example
├── .gitignore
├── package.json
└── server.js
```

#### .env.example (à commiter)

```bash
# Configuration de l'application
NODE_ENV=development
PORT=3000
LOG_LEVEL=debug

# Base de données
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_NAME=myapp
DATABASE_USER=myuser
DATABASE_PASSWORD=changeme

# API externe
API_KEY=your_api_key_here
API_URL=https://api.example.com
```

#### .gitignore

```
node_modules/
.env
.env.local
.env.production
secrets/
npm-debug.log
```

#### Dockerfile

```dockerfile
FROM node:18-alpine

WORKDIR /app

# Pas de secrets dans le Dockerfile !
ENV NODE_ENV=production

COPY package*.json ./
RUN npm ci --only=production

COPY . .

# Créer un utilisateur non-root
RUN adduser -D appuser && chown -R appuser:appuser /app
USER appuser

CMD ["node", "server.js"]
```

#### server.js

```javascript
const express = require('express');
const { Pool } = require('pg');
const fs = require('fs');

const app = express();

// Fonction pour lire un secret (compatible avec Docker Secrets)
function getSecret(secretName, envVar) {
  // Essayer de lire depuis /run/secrets/ (Docker Secrets)
  const secretPath = `/run/secrets/${secretName}`;
  if (fs.existsSync(secretPath)) {
    return fs.readFileSync(secretPath, 'utf8').trim();
  }

  // Sinon, utiliser la variable d'environnement
  return process.env[envVar];
}

// Configuration de la base de données
const dbConfig = {
  host: process.env.DATABASE_HOST || 'localhost',
  port: process.env.DATABASE_PORT || 5432,
  database: process.env.DATABASE_NAME || 'myapp',
  user: process.env.DATABASE_USER || 'myuser',
  password: getSecret('db_password', 'DATABASE_PASSWORD')
};

// NE JAMAIS logger les secrets !
console.log('Configuration de la base de données:');
console.log(`  Host: ${dbConfig.host}`);
console.log(`  Port: ${dbConfig.port}`);
console.log(`  Database: ${dbConfig.database}`);
console.log(`  User: ${dbConfig.user}`);
console.log('  Password: [REDACTED]'); // Ne jamais afficher !

const pool = new Pool(dbConfig);

// Routes
app.get('/', (req, res) => {
  res.send('Application sécurisée !');
});

app.get('/health', async (req, res) => {
  try {
    await pool.query('SELECT 1');
    res.json({ status: 'healthy', database: 'connected' });
  } catch (error) {
    res.status(500).json({ status: 'unhealthy', error: error.message });
  }
});

const port = process.env.PORT || 3000;
app.listen(port, () => {
  console.log(`Serveur démarré sur le port ${port}`);
  console.log(`Environnement: ${process.env.NODE_ENV}`);
});
```

#### docker-compose.yml (développement)

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    env_file:
      - .env
    depends_on:
      - db
    networks:
      - app-network

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: ${DATABASE_NAME}
      POSTGRES_USER: ${DATABASE_USER}
      POSTGRES_PASSWORD: ${DATABASE_PASSWORD}
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - app-network

volumes:
  db-data:

networks:
  app-network:
    driver: bridge
```

#### docker-compose.prod.yml (production avec secrets)

```yaml
version: '3.8'

services:
  app:
    image: mon-app:latest
    ports:
      - "3000:3000"
    secrets:
      - db_password
      - api_key
    environment:
      NODE_ENV: production
      DATABASE_HOST: db
      DATABASE_PORT: 5432
      DATABASE_NAME: myapp
      DATABASE_USER: myuser
      API_URL: https://api.example.com
    depends_on:
      - db
    networks:
      - app-network
    deploy:
      replicas: 3
      restart_policy:
        condition: on-failure

  db:
    image: postgres:15-alpine
    secrets:
      - db_password
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - app-network

secrets:
  db_password:
    external: true
  api_key:
    external: true

volumes:
  db-data:

networks:
  app-network:
    driver: overlay
```

Déploiement :

```bash
# Développement
cp .env.example .env
# Éditer .env avec les vraies valeurs
docker-compose up

# Production avec Swarm
docker swarm init
echo "motdepasse_production" | docker secret create db_password -
echo "cle_api_production" | docker secret create api_key -
docker stack deploy -c docker-compose.prod.yml mon-app
```

### Exemple 2 : Application Python avec Flask

#### Structure du projet

```
mon-projet/
├── Dockerfile
├── docker-compose.yml
├── .env.example
├── .gitignore
├── requirements.txt
├── config.py
└── app.py
```

#### config.py

```python
import os
from pathlib import Path

class Config:
    """Configuration de l'application"""

    @staticmethod
    def get_secret(secret_name, env_var, default=None):
        """
        Récupère un secret depuis Docker Secrets ou les variables d'environnement
        """
        # Essayer de lire depuis /run/secrets/ (Docker Secrets)
        secret_path = Path(f'/run/secrets/{secret_name}')
        if secret_path.exists():
            return secret_path.read_text().strip()

        # Sinon, utiliser la variable d'environnement
        return os.getenv(env_var, default)

    # Configuration générale
    ENV = os.getenv('FLASK_ENV', 'development')
    DEBUG = ENV == 'development'
    PORT = int(os.getenv('PORT', 5000))

    # Base de données
    DATABASE_HOST = os.getenv('DATABASE_HOST', 'localhost')
    DATABASE_PORT = int(os.getenv('DATABASE_PORT', 5432))
    DATABASE_NAME = os.getenv('DATABASE_NAME', 'myapp')
    DATABASE_USER = os.getenv('DATABASE_USER', 'myuser')
    DATABASE_PASSWORD = get_secret.__func__('db_password', 'DATABASE_PASSWORD')

    # API externe
    API_KEY = get_secret.__func__('api_key', 'API_KEY')
    API_URL = os.getenv('API_URL', 'https://api.example.com')

    @classmethod
    def log_config(cls):
        """Affiche la configuration (sans les secrets)"""
        print('Configuration de l\'application:')
        print(f'  Environnement: {cls.ENV}')
        print(f'  Debug: {cls.DEBUG}')
        print(f'  Port: {cls.PORT}')
        print(f'  Database Host: {cls.DATABASE_HOST}')
        print(f'  Database Port: {cls.DATABASE_PORT}')
        print(f'  Database Name: {cls.DATABASE_NAME}')
        print(f'  Database User: {cls.DATABASE_USER}')
        print('  Database Password: [REDACTED]')
        print('  API Key: [REDACTED]')
        print(f'  API URL: {cls.API_URL}')
```

#### app.py

```python
from flask import Flask, jsonify
import psycopg2
from config import Config

app = Flask(__name__)

# Afficher la configuration au démarrage (sans les secrets)
Config.log_config()

def get_db_connection():
    """Créer une connexion à la base de données"""
    return psycopg2.connect(
        host=Config.DATABASE_HOST,
        port=Config.DATABASE_PORT,
        database=Config.DATABASE_NAME,
        user=Config.DATABASE_USER,
        password=Config.DATABASE_PASSWORD
    )

@app.route('/')
def home():
    return jsonify({'message': 'Application sécurisée !'})

@app.route('/health')
def health():
    try:
        conn = get_db_connection()
        cur = conn.cursor()
        cur.execute('SELECT 1')
        cur.close()
        conn.close()
        return jsonify({'status': 'healthy', 'database': 'connected'})
    except Exception as e:
        return jsonify({
            'status': 'unhealthy',
            'error': str(e)
        }), 500

if __name__ == '__main__':
    app.run(
        host='0.0.0.0',
        port=Config.PORT,
        debug=Config.DEBUG
    )
```

## Rotation des secrets

Les secrets doivent être changés régulièrement et immédiatement en cas de compromission.

### Avec Docker Secrets

```bash
# Créer un nouveau secret
echo "nouveau_motdepasse" | docker secret create db_password_v2 -

# Mettre à jour le service pour utiliser le nouveau secret
docker service update \
  --secret-rm db_password \
  --secret-add source=db_password_v2,target=db_password \
  mon-service

# Supprimer l'ancien secret
docker secret rm db_password
```

### Stratégie de rotation

1. **Planifier des rotations régulières** : Tous les 90 jours minimum
2. **Documenter la procédure** : Qui peut changer les secrets ? Comment ?
3. **Tester la procédure** : Simuler une rotation en environnement de test
4. **Automatiser si possible** : Utiliser des outils comme Vault pour la rotation automatique

## Audit et conformité

### Vérifier l'absence de secrets dans les images

```bash
# Vérifier l'historique de l'image
docker history mon-image:latest

# Inspecter l'image
docker inspect mon-image:latest

# Scanner avec des outils spécialisés
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image --severity HIGH,CRITICAL mon-image:latest
```

### Utiliser des outils de détection

**git-secrets** : Empêche de commiter des secrets dans Git

```bash
# Installer git-secrets
brew install git-secrets  # Sur macOS
apt-get install git-secrets  # Sur Ubuntu

# Configurer dans un dépôt
cd mon-projet
git secrets --install
git secrets --register-aws  # Patterns AWS
git secrets --add 'password.*=.*'
git secrets --add 'api[_-]?key.*=.*'
```

**TruffleHog** : Recherche les secrets dans l'historique Git

```bash
# Scanner un dépôt
docker run --rm -v "$PWD:/proj" \
  trufflesecurity/trufflehog:latest \
  git file:///proj --only-verified
```

## Checklist de sécurité

Avant de déployer votre application, vérifiez :

- [ ] Aucun secret dans le Dockerfile
- [ ] Aucun secret dans les fichiers Docker Compose commitées
- [ ] `.env` et `secrets/` dans `.gitignore`
- [ ] `.env.example` présent avec des valeurs fictives
- [ ] Secrets passés au runtime ou via Docker Secrets
- [ ] Aucun log des secrets dans l'application
- [ ] Rotation des secrets planifiée
- [ ] Accès aux secrets limité au strict nécessaire
- [ ] Utilisation d'un gestionnaire de secrets en production
- [ ] Documentation de la gestion des secrets pour l'équipe

## Comparaison des solutions

| Solution | Complexité | Sécurité | Cas d'usage |
|----------|-----------|----------|-------------|
| Variables d'env au runtime | ⭐ Faible | ⭐⭐ Moyenne | Développement |
| Fichiers .env | ⭐ Faible | ⭐⭐ Moyenne | Développement |
| Docker Secrets (Swarm) | ⭐⭐ Moyenne | ⭐⭐⭐⭐ Élevée | Production simple |
| Docker Compose secrets | ⭐⭐ Moyenne | ⭐⭐ Moyenne | Développement |
| Vault / AWS / Azure | ⭐⭐⭐ Élevée | ⭐⭐⭐⭐⭐ Très élevée | Production complexe |

## Conclusion

La gestion des secrets est un aspect critique de la sécurité de vos applications Docker. Les principes clés à retenir :

1. **Ne jamais stocker les secrets dans les images** : Ni dans le Dockerfile, ni dans les layers
2. **Ne jamais commiter les secrets dans Git** : Utilisez `.env.example` et `.gitignore`
3. **Passer les secrets au runtime** : Via variables d'environnement ou Docker Secrets
4. **Utiliser des gestionnaires de secrets en production** : Vault, AWS Secrets Manager, Azure Key Vault
5. **Ne jamais logger les secrets** : Masquer les valeurs sensibles dans les logs
6. **Planifier la rotation des secrets** : Changer régulièrement les mots de passe et clés

Pour le développement, l'utilisation de fichiers `.env` (non commitées) est acceptable. Pour la production, privilégiez Docker Secrets ou un gestionnaire de secrets externe professionnel.

La sécurité est un processus continu : restez vigilant et formez votre équipe aux bonnes pratiques !

## Ressources supplémentaires

- [Docker Secrets Documentation](https://docs.docker.com/engine/swarm/secrets/)
- [OWASP Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)
- [12-Factor App - Config](https://12factor.net/config)
- [HashiCorp Vault](https://www.vaultproject.io/)

⏭️ [Logging et monitoring](/10-bonnes-pratiques/04-logging-et-monitoring.md)
