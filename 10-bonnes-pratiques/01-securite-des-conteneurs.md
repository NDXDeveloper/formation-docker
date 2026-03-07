🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 10.1 Sécurité des conteneurs

## Introduction

La sécurité est un aspect fondamental lors de l'utilisation de Docker. Même si les conteneurs offrent une certaine isolation, ils ne sont pas invulnérables. Comprendre les principes de base de la sécurité des conteneurs vous permettra de déployer vos applications de manière plus sûre et de protéger vos données sensibles.

Dans cette section, nous allons explorer les concepts essentiels et les bonnes pratiques de sécurité que tout utilisateur de Docker devrait connaître.

## Pourquoi la sécurité des conteneurs est importante ?

Les conteneurs partagent le noyau du système d'exploitation hôte, ce qui signifie qu'une faille de sécurité dans un conteneur peut potentiellement affecter l'hôte et les autres conteneurs. De plus, les conteneurs sont souvent connectés à Internet et manipulent des données sensibles, ce qui en fait des cibles potentielles pour les attaques.

## Principes de base de la sécurité

### 1. Utiliser des images de confiance

**Le problème :** N'importe qui peut publier des images sur Docker Hub. Certaines images peuvent contenir du code malveillant ou des vulnérabilités connues.

**Les bonnes pratiques :**

- **Privilégier les images officielles** : Elles sont identifiées par un badge "Official Image" sur Docker Hub et sont maintenues par Docker ou l'éditeur du logiciel.

```bash
# Image officielle - recommandée
docker pull nginx

# Image non officielle - à éviter si possible
docker pull utilisateur-inconnu/nginx
```

- **Utiliser des images vérifiées** : Recherchez le badge "Verified Publisher" qui indique que l'éditeur a été vérifié par Docker.

- **Vérifier la popularité** : Les images avec beaucoup de téléchargements et d'étoiles sont généralement plus fiables.

- **Consulter le Dockerfile** : Avant d'utiliser une image, examinez son Dockerfile sur GitHub pour comprendre ce qu'elle contient.

### 2. Maintenir les images à jour

Les vulnérabilités de sécurité sont découvertes régulièrement dans les logiciels. Il est crucial de maintenir vos images à jour.

**Les bonnes pratiques :**

- **Reconstruire régulièrement vos images** : Ne laissez pas vos images devenir obsolètes.

```bash
# Télécharger la dernière version d'une image
docker pull ubuntu:latest

# Reconstruire votre image personnalisée
docker build -t mon-app:latest .
```

- **Utiliser des tags spécifiques** : Plutôt que d'utiliser uniquement `latest`, utilisez des versions spécifiques pour plus de contrôle.

```dockerfile
# Moins recommandé - version non spécifiée
FROM node:latest

# Recommandé - version spécifique
FROM node:18.17.0
```

- **Scanner vos images** : Utilisez des outils pour détecter les vulnérabilités connues.

```bash
# Docker Scout (intégré à Docker Desktop)
docker scout cves mon-image:latest
```

### 3. Ne pas exécuter les conteneurs en tant que root

Par défaut, les processus dans un conteneur s'exécutent en tant qu'utilisateur root, ce qui pose un risque de sécurité important.

**Le problème :** Si un attaquant parvient à s'échapper du conteneur, il aura les privilèges root sur le système hôte.

**La solution :** Créer et utiliser un utilisateur non privilégié dans vos Dockerfiles.

```dockerfile
FROM ubuntu:22.04

# Installer les dépendances
RUN apt-get update && apt-get install -y python3

# Créer un utilisateur non-root
RUN useradd -m -u 1000 appuser

# Créer un répertoire pour l'application
RUN mkdir /app && chown appuser:appuser /app

# Changer vers l'utilisateur non-root
USER appuser

# Définir le répertoire de travail
WORKDIR /app

# Copier et exécuter l'application
COPY --chown=appuser:appuser app.py .  
CMD ["python3", "app.py"]  
```

**Vérification :** Vous pouvez vérifier sous quel utilisateur s'exécute votre conteneur :

```bash
docker exec mon-conteneur whoami
```

### 4. Limiter les capacités des conteneurs

Docker attribue par défaut certaines capacités (capabilities) aux conteneurs. Réduire ces capacités limite ce qu'un attaquant peut faire s'il compromet un conteneur.

**Supprimer toutes les capacités et n'ajouter que celles nécessaires :**

```bash
# Supprimer toutes les capacités par défaut
docker run --cap-drop=ALL \
           --cap-add=NET_BIND_SERVICE \
           mon-image
```

**Les capacités courantes :**
- `NET_BIND_SERVICE` : Permet de se lier à des ports inférieurs à 1024
- `CHOWN` : Permet de changer la propriété des fichiers
- `DAC_OVERRIDE` : Permet de contourner les permissions de fichiers

### 5. Utiliser le mode read-only

Rendre le système de fichiers du conteneur en lecture seule empêche les modifications non autorisées.

```bash
# Lancer un conteneur en lecture seule
docker run --read-only \
           --tmpfs /tmp \
           mon-image
```

**Note :** L'option `--tmpfs /tmp` crée un système de fichiers temporaire en mémoire pour `/tmp`, car de nombreuses applications ont besoin d'écrire des fichiers temporaires.

### 6. Limiter les ressources

Empêcher un conteneur de consommer toutes les ressources du système est important pour éviter les attaques par déni de service.

```bash
# Limiter la mémoire et le CPU
docker run --memory="512m" \
           --cpus="1.0" \
           mon-image
```

**Avec Docker Compose :**

```yaml
services:
  mon-service:
    image: mon-image
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M
```

### 7. Ne pas exposer le socket Docker

Le socket Docker (`/var/run/docker.sock`) permet de contrôler entièrement Docker. Ne le montez jamais dans un conteneur sauf si c'est absolument nécessaire.

```bash
# À ÉVITER sauf nécessité absolue
docker run -v /var/run/docker.sock:/var/run/docker.sock mon-image
```

**Pourquoi c'est dangereux ?** Un conteneur avec accès au socket Docker peut créer, modifier ou supprimer n'importe quel conteneur, y compris avec des privilèges élevés.

### 8. Gérer les secrets de manière sécurisée

Ne jamais stocker des mots de passe, clés API ou autres secrets directement dans les images Docker.

**Mauvaise pratique :**

```dockerfile
# NE JAMAIS FAIRE ÇA
ENV DATABASE_PASSWORD=monmotdepasse123
```

**Bonnes pratiques :**

**a) Utiliser des variables d'environnement au runtime :**

```bash
docker run -e DATABASE_PASSWORD=monmotdepasse mon-image
```

**b) Utiliser des fichiers de secrets (Docker Swarm) :**

```bash
echo "monmotdepasse" | docker secret create db_password -  
docker service create --secret db_password mon-image  
```

**c) Utiliser un fichier .env avec Docker Compose :**

```yaml
# docker-compose.yml
services:
  app:
    image: mon-image
    environment:
      - DATABASE_PASSWORD=${DATABASE_PASSWORD}
```

```bash
# .env (ne jamais commiter ce fichier)
DATABASE_PASSWORD=monmotdepasse
```

**d) Utiliser des gestionnaires de secrets externes :**
- HashiCorp Vault
- AWS Secrets Manager
- Azure Key Vault

### 9. Minimiser la surface d'attaque

**Utiliser des images minimales :**

Plus une image contient de logiciels, plus elle a de chances de contenir des vulnérabilités.

```dockerfile
# Image complète (plus grande surface d'attaque)
FROM ubuntu:22.04

# Image minimale Alpine (surface d'attaque réduite)
FROM alpine:3.18

# Image "distroless" (encore plus minimale - sans shell)
FROM gcr.io/distroless/base
```

**Avantages des images minimales :**
- Moins de vulnérabilités potentielles
- Taille d'image réduite
- Temps de démarrage plus rapide
- Moins de packages à maintenir

**Ne pas installer d'outils de débogage en production :**

```dockerfile
# Pour le développement
FROM node:18  
RUN apt-get update && apt-get install -y vim curl  

# Pour la production - image minimale
FROM node:18-alpine  
COPY package*.json ./  
RUN npm ci --omit=dev  
COPY . .  
CMD ["node", "app.js"]  
```

### 10. Isoler les réseaux

Par défaut, les conteneurs peuvent communiquer entre eux. Il est préférable de créer des réseaux isolés pour limiter les communications.

```bash
# Créer des réseaux séparés
docker network create frontend-network  
docker network create backend-network  

# Connecter les conteneurs aux réseaux appropriés
docker run --network=frontend-network web-app  
docker run --network=backend-network --network=frontend-network api  
docker run --network=backend-network database  
```

Dans cet exemple :
- L'application web ne peut pas accéder directement à la base de données
- L'API peut communiquer avec les deux réseaux
- La base de données est isolée du frontend

### 11. Activer Content Trust

Docker Content Trust utilise des signatures numériques pour vérifier l'intégrité et l'authenticité des images.

```bash
# Activer Content Trust
export DOCKER_CONTENT_TRUST=1

# Maintenant, seules les images signées peuvent être téléchargées
docker pull mon-image:latest
```

### 12. Scanner régulièrement vos images

Utilisez des outils d'analyse de vulnérabilités pour identifier les problèmes de sécurité.

**Outils disponibles :**

- **Docker Scout** (intégré à Docker Desktop)
- **Trivy** (outil open-source)
- **Clair** (outil open-source)
- **Snyk** (service commercial)

```bash
# Exemple avec Trivy
docker run aquasec/trivy image mon-image:latest
```

### 13. Journaliser et surveiller

Configurez la journalisation pour détecter les comportements suspects.

```bash
# Configurer le driver de journalisation
docker run --log-driver=syslog \
           --log-opt syslog-address=tcp://192.168.1.100:514 \
           mon-image
```

**Surveiller :**
- Les tentatives d'accès non autorisées
- Les modifications de fichiers inattendues
- Les pics d'utilisation des ressources
- Les connexions réseau inhabituelles

## Récapitulatif des bonnes pratiques

1. ✅ Utiliser des images officielles ou vérifiées
2. ✅ Maintenir les images à jour
3. ✅ Ne pas exécuter en tant que root
4. ✅ Limiter les capacités des conteneurs
5. ✅ Utiliser le mode read-only quand possible
6. ✅ Limiter les ressources (CPU, mémoire)
7. ✅ Ne pas exposer le socket Docker
8. ✅ Gérer les secrets de manière sécurisée
9. ✅ Utiliser des images minimales
10. ✅ Isoler les réseaux
11. ✅ Activer Content Trust
12. ✅ Scanner régulièrement les vulnérabilités
13. ✅ Journaliser et surveiller les conteneurs

## Exemple de Dockerfile sécurisé

Voici un exemple complet intégrant plusieurs bonnes pratiques de sécurité :

```dockerfile
# Utiliser une image de base spécifique et minimale
FROM node:18-alpine3.18

# Créer un utilisateur non-root
RUN addgroup -g 1000 appgroup && \
    adduser -D -u 1000 -G appgroup appuser

# Définir le répertoire de travail
WORKDIR /app

# Copier les fichiers de dépendances
COPY package*.json ./

# Installer les dépendances en tant que root
RUN npm ci --omit=dev && \
    npm cache clean --force

# Copier le code de l'application
COPY --chown=appuser:appgroup . .

# Changer vers l'utilisateur non-root
USER appuser

# Exposer le port (documentation uniquement)
EXPOSE 3000

# Ajouter un health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node healthcheck.js || exit 1

# Commande de démarrage
CMD ["node", "server.js"]
```

## Conclusion

La sécurité des conteneurs est un processus continu, pas une action ponctuelle. En appliquant ces principes de base, vous réduisez considérablement les risques liés à l'utilisation de Docker. N'oubliez pas que la sécurité repose sur plusieurs couches de protection : plus vous en mettez en place, mieux votre système sera protégé.

Dans les sections suivantes, nous verrons d'autres bonnes pratiques concernant l'optimisation des images, la gestion des secrets et le monitoring, qui contribuent également à la sécurité globale de vos déploiements Docker.

## Ressources supplémentaires

- [Documentation officielle Docker - Security](https://docs.docker.com/engine/security/)
- [OWASP Docker Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)
- [CIS Docker Benchmark](https://www.cisecurity.org/benchmark/docker)

⏭️ [Optimisation de la taille des images](/10-bonnes-pratiques/02-optimisation-de-la-taille-des-images.md)
