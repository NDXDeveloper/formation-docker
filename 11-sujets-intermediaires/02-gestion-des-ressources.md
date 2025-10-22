🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 11.2 Gestion des ressources (CPU, mémoire)

## Introduction

Par défaut, un conteneur Docker peut utiliser **toutes les ressources disponibles** sur la machine hôte : processeur (CPU), mémoire (RAM), disque, etc. Cette situation peut causer des problèmes sérieux si un conteneur consomme trop de ressources et affecte les autres conteneurs ou le système hôte lui-même.

La gestion des ressources permet de **limiter et contrôler** l'utilisation du CPU et de la mémoire par vos conteneurs pour garantir la stabilité, la performance et l'équité entre les applications.

## Pourquoi limiter les ressources ?

### Problèmes sans limitation de ressources

Imaginez plusieurs scénarios problématiques :

**Scénario 1 : Le conteneur gourmand**
- Vous lancez une application qui a une fuite mémoire (memory leak)
- Le conteneur consomme progressivement toute la RAM disponible
- Les autres conteneurs et le système hôte deviennent lents ou plantent
- Le serveur peut devenir complètement inaccessible

**Scénario 2 : La charge CPU excessive**
- Un conteneur exécute un traitement intensif (calcul, compression, etc.)
- Il utilise 100% de tous les cœurs CPU
- Les autres applications deviennent extrêmement lentes
- Vos services critiques ne peuvent plus répondre aux requêtes

**Scénario 3 : L'effet domino**
- Un conteneur mal configuré consomme toutes les ressources
- Les autres conteneurs ne peuvent plus fonctionner correctement
- Votre environnement de production devient instable
- Les utilisateurs finaux subissent des temps de réponse dégradés

### Avantages de la limitation de ressources

✅ **Stabilité** : Évite qu'un conteneur ne monopolise toutes les ressources
✅ **Prévisibilité** : Garantit des performances cohérentes
✅ **Isolation** : Protège les autres conteneurs des conteneurs gourmands
✅ **Planification** : Permet de déterminer combien de conteneurs peuvent tourner sur un serveur
✅ **Coûts** : Optimise l'utilisation des ressources et donc les coûts cloud

## Comprendre les ressources système

### CPU (Processeur)

Le CPU est la ressource qui exécute les calculs. Concepts importants :

- **Cœurs CPU** : Votre machine peut avoir 2, 4, 8 cœurs ou plus
- **Part de CPU** : Pourcentage ou fraction de CPU allouée
- **Priorité** : Certains conteneurs peuvent avoir la priorité sur d'autres

**Exemple** : Si votre machine a 4 cœurs CPU et que vous limitez un conteneur à 2 cœurs, il ne pourra utiliser que 50% de la puissance totale.

### Mémoire (RAM)

La mémoire est l'espace de travail où les applications stockent leurs données temporaires.

- **Mémoire physique** : La RAM installée sur votre machine
- **Limite dure** : Maximum absolu de mémoire qu'un conteneur peut utiliser
- **OOM Killer** : Mécanisme qui tue un processus si la mémoire est dépassée

**Important** : Contrairement au CPU (partageable), la mémoire est exclusive. Une fois allouée, elle n'est pas disponible pour d'autres processus.

## Limitation de la mémoire

### Définir une limite mémoire

La syntaxe de base pour limiter la mémoire est l'option `--memory` (ou `-m`) :

```bash
docker run --memory="500m" nginx
```

### Unités de mesure

Docker accepte différentes unités pour la mémoire :

| Unité | Signification | Exemple |
|-------|---------------|---------|
| `b` | Bytes (octets) | `1024b` |
| `k` | Kilobytes | `512k` |
| `m` | Megabytes | `500m` |
| `g` | Gigabytes | `2g` |

### Exemples pratiques

```bash
# Limiter à 256 mégaoctets
docker run --memory="256m" --name app1 nginx

# Limiter à 1 gigaoctet
docker run --memory="1g" --name app2 postgres

# Limiter à 512 mégaoctets
docker run -m 512m --name app3 redis
```

### Que se passe-t-il si la limite est atteinte ?

Quand un conteneur atteint sa limite mémoire :

1. Le système tente de libérer de la mémoire (garbage collection)
2. Si impossible, le conteneur est **tué** par l'OOM Killer (Out Of Memory)
3. Le conteneur s'arrête avec un code d'erreur 137
4. Vous verrez un message d'erreur dans les logs

**Exemple** :
```bash
# Lancer un conteneur avec très peu de mémoire
docker run --memory="50m" --name test-mem nginx

# Si nginx nécessite plus de 50m, le conteneur sera tué
docker logs test-mem
# Possible output: OOMKilled
```

### Réservation de mémoire (Memory Reservation)

En plus de la limite dure, vous pouvez définir une **réservation souple** avec `--memory-reservation` :

```bash
docker run --memory="1g" --memory-reservation="750m" nginx
```

**Fonctionnement** :
- Le conteneur peut utiliser normalement jusqu'à 750m
- Il peut dépasser 750m et aller jusqu'à 1g si nécessaire
- Si le système manque de mémoire, Docker essaie de ramener le conteneur à 750m

**Utilité** : Permet de la flexibilité tout en gardant une limite de sécurité.

### Swap Memory

Le swap permet d'utiliser le disque dur comme extension de la RAM (plus lent).

```bash
# Limite mémoire : 500m, swap : 1g
docker run --memory="500m" --memory-swap="1g" nginx
```

**Attention** : La valeur de `--memory-swap` est le **total** (mémoire + swap).

Dans l'exemple ci-dessus :
- Mémoire RAM : 500m
- Swap disponible : 1g - 500m = 500m

```bash
# Désactiver complètement le swap
docker run --memory="500m" --memory-swap="500m" nginx

# Swap illimité (non recommandé)
docker run --memory="500m" --memory-swap="-1" nginx
```

## Limitation du CPU

### Comprendre les parts de CPU

Docker utilise un système de **parts relatives** (shares) plutôt que des pourcentages absolus par défaut.

### Limiter avec --cpus

La façon la plus simple est d'utiliser `--cpus` :

```bash
# Limiter à 1.5 cœur CPU
docker run --cpus="1.5" nginx
```

**Exemples** :
```bash
# Utiliser maximum 1 cœur entier
docker run --cpus="1" redis

# Utiliser maximum 0.5 cœur (50% d'un cœur)
docker run --cpus="0.5" nginx

# Utiliser 2.5 cœurs
docker run --cpus="2.5" postgres
```

**Signification** :
- `--cpus="1"` : Le conteneur peut utiliser 100% d'un seul cœur CPU
- `--cpus="2"` : Le conteneur peut utiliser 100% de deux cœurs CPU
- `--cpus="0.5"` : Le conteneur peut utiliser 50% d'un cœur CPU

### CPU Shares (parts de CPU)

Le système de shares définit la **priorité relative** entre conteneurs.

```bash
# Conteneur avec priorité normale (1024 shares par défaut)
docker run --cpu-shares=1024 --name app1 nginx

# Conteneur avec priorité double
docker run --cpu-shares=2048 --name app2 redis

# Conteneur avec priorité basse
docker run --cpu-shares=512 --name app3 postgres
```

**Fonctionnement** :
- Si app1 et app2 sont tous les deux actifs et demandent du CPU :
  - app2 obtiendra 2x plus de CPU que app1
  - Ratio : app2 (66%) vs app1 (33%)

**Important** : Les shares n'ont d'effet que quand il y a **contention** (plusieurs conteneurs veulent utiliser le CPU simultanément).

### Exemple de répartition avec CPU Shares

Machine avec 2 cœurs CPU (200% de capacité totale) :

```bash
docker run -d --cpu-shares=1024 --name container1 stress --cpu 2
docker run -d --cpu-shares=512 --name container2 stress --cpu 2
docker run -d --cpu-shares=512 --name container3 stress --cpu 2
```

**Répartition** :
- Total shares : 1024 + 512 + 512 = 2048
- container1 : (1024/2048) × 200% = 100% (1 cœur)
- container2 : (512/2048) × 200% = 50% (0.5 cœur)
- container3 : (512/2048) × 200% = 50% (0.5 cœur)

### Épingler sur des cœurs spécifiques (CPU Pinning)

Vous pouvez forcer un conteneur à n'utiliser que certains cœurs CPU :

```bash
# Utiliser uniquement les cœurs 0 et 1
docker run --cpuset-cpus="0,1" nginx

# Utiliser uniquement le cœur 3
docker run --cpuset-cpus="3" redis

# Utiliser les cœurs 0 à 3
docker run --cpuset-cpus="0-3" postgres
```

**Cas d'usage** :
- Isolation stricte entre applications
- Optimisation NUMA (Non-Uniform Memory Access)
- Performance prévisible pour applications critiques

### Période et Quota CPU

Pour un contrôle plus fin, vous pouvez utiliser `--cpu-period` et `--cpu-quota` :

```bash
# Limiter à 50% d'un cœur
docker run --cpu-period=100000 --cpu-quota=50000 nginx
```

**Explication** :
- `--cpu-period` : Période en microsecondes (généralement 100000 = 100ms)
- `--cpu-quota` : Quota en microsecondes durant cette période

**Formule** : Pourcentage CPU = (quota / period) × 100

**Exemples** :
```bash
# 25% d'un cœur
docker run --cpu-period=100000 --cpu-quota=25000 nginx

# 150% (1.5 cœur)
docker run --cpu-period=100000 --cpu-quota=150000 nginx
```

**Note** : L'option `--cpus` est plus simple et fait la même chose en arrière-plan !

## Combinaison de limitations

Vous pouvez combiner les limitations CPU et mémoire :

```bash
docker run \
  --name my-app \
  --memory="1g" \
  --memory-reservation="750m" \
  --cpus="1.5" \
  nginx
```

### Exemple : Application web

```bash
docker run -d \
  --name frontend \
  --memory="512m" \
  --cpus="0.5" \
  --restart=unless-stopped \
  nginx:alpine
```

### Exemple : Base de données

```bash
docker run -d \
  --name database \
  --memory="2g" \
  --memory-reservation="1.5g" \
  --cpus="2" \
  --cpuset-cpus="0-1" \
  postgres:15
```

### Exemple : Worker de traitement

```bash
docker run -d \
  --name worker \
  --memory="1g" \
  --cpus="1" \
  --cpu-shares=512 \
  my-worker-image
```

## Avec Docker Compose

Vous pouvez définir les ressources dans votre fichier `docker-compose.yml`.

### Version moderne (v3 et +)

```yaml
version: '3.8'

services:
  web:
    image: nginx
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M

  database:
    image: postgres
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '1'
          memory: 1G

  cache:
    image: redis
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 256M
```

**Explications** :
- `limits` : Limites maximales (dures)
- `reservations` : Ressources garanties (souples)

### Version v2 (ancienne syntaxe)

```yaml
version: '2.4'

services:
  web:
    image: nginx
    mem_limit: 512m
    mem_reservation: 256m
    cpus: 0.5
    cpu_shares: 512
```

## Surveiller l'utilisation des ressources

### Commande docker stats

La commande `docker stats` affiche l'utilisation en temps réel :

```bash
# Afficher les statistiques de tous les conteneurs
docker stats

# Afficher les stats d'un conteneur spécifique
docker stats my-container

# Afficher une seule fois (pas en temps réel)
docker stats --no-stream
```

**Exemple de sortie** :
```
CONTAINER ID   NAME      CPU %    MEM USAGE / LIMIT    MEM %    NET I/O
abc123         nginx     0.5%     50MiB / 512MiB      9.77%     1.2kB / 0B
def456         redis     1.2%     100MiB / 1GiB       9.77%     5kB / 2kB
```

### Inspecter les limites configurées

```bash
# Voir toutes les configurations d'un conteneur
docker inspect my-container

# Voir uniquement les ressources
docker inspect my-container --format='{{.HostConfig.Memory}}'
docker inspect my-container --format='{{.HostConfig.NanoCpus}}'
```

### Format personnalisé

```bash
# Afficher seulement nom, CPU et mémoire
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

## Bonnes pratiques

### 1. Toujours définir des limites en production

```bash
# ❌ Mauvais : pas de limite
docker run -d nginx

# ✅ Bon : limites définies
docker run -d --memory="512m" --cpus="1" nginx
```

**Raison** : Protège votre infrastructure contre les applications défaillantes.

### 2. Utilisez des réservations pour les services critiques

```bash
docker run -d \
  --memory="2g" \
  --memory-reservation="1.5g" \
  --cpus="2" \
  postgres
```

**Raison** : Garantit un minimum de ressources pour les services importants.

### 3. Testez les limites en développement

Ne découvrez pas en production que votre application nécessite plus de ressources !

```bash
# Testez avec différentes limites
docker run --memory="256m" my-app  # Trop peu ?
docker run --memory="512m" my-app  # Suffisant ?
docker run --memory="1g" my-app    # Confortable ?
```

### 4. Surveillez régulièrement avec docker stats

```bash
# Surveillez vos conteneurs
docker stats --no-stream > stats.log

# Automatisez avec cron
*/5 * * * * docker stats --no-stream >> /var/log/docker-stats.log
```

### 5. Ajustez progressivement

Commencez avec des limites conservatrices, puis ajustez selon les besoins réels :

```bash
# Étape 1 : Limites larges
docker run --memory="2g" --cpus="2" my-app

# Observer pendant quelques jours...

# Étape 2 : Ajustement
docker run --memory="1g" --cpus="1" my-app
```

### 6. Documentez vos choix

Créez un fichier README ou un commentaire dans docker-compose.yml :

```yaml
services:
  api:
    image: my-api
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 1G
    # Ressources basées sur les tests de charge du 2024-10-22
    # Pic observé : 0.7 CPU, 750MB RAM
    # Marge de 30% ajoutée pour les pics occasionnels
```

### 7. Prévoyez de la marge

```bash
# ❌ Limite trop juste
docker run --memory="500m" my-app  # Si l'app utilise 490m normalement

# ✅ Marge de sécurité de 30-50%
docker run --memory="750m" my-app
```

### 8. Utilisez des profils pour différents environnements

```yaml
# docker-compose.yml
services:
  app:
    image: my-app

# docker-compose.prod.yml
services:
  app:
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G

# docker-compose.dev.yml
services:
  app:
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
```

**Utilisation** :
```bash
# Développement
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up

# Production
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up
```

## Calcul des ressources nécessaires

### Méthode pour dimensionner vos conteneurs

**Étape 1 : Mesurer sans limite**
```bash
docker run -d --name test-app my-app
docker stats test-app --no-stream
```

**Étape 2 : Observer pendant la charge**
```bash
# Simulez une charge réaliste
# Puis observez les pics
docker stats test-app
```

**Étape 3 : Appliquer la formule**
```
Limite = Pic_observé × 1.3 à 1.5
```

**Exemple** :
- Pic observé : 600 MB de RAM, 0.8 CPU
- Limite recommandée : 900 MB de RAM, 1.2 CPU

### Exemple de planification de serveur

Serveur avec **16 GB RAM** et **8 cœurs CPU** :

```yaml
services:
  # 3 frontends
  frontend-1:
    deploy:
      resources:
        limits: { memory: 512M, cpus: '0.5' }
  frontend-2:
    deploy:
      resources:
        limits: { memory: 512M, cpus: '0.5' }
  frontend-3:
    deploy:
      resources:
        limits: { memory: 512M, cpus: '0.5' }

  # 2 APIs
  api-1:
    deploy:
      resources:
        limits: { memory: 1G, cpus: '1' }
  api-2:
    deploy:
      resources:
        limits: { memory: 1G, cpus: '1' }

  # 1 base de données
  database:
    deploy:
      resources:
        limits: { memory: 4G, cpus: '2' }

  # 1 cache
  redis:
    deploy:
      resources:
        limits: { memory: 1G, cpus: '0.5' }
```

**Total** :
- RAM : (3×0.5) + (2×1) + 4 + 1 = 8.5 GB (53% du serveur)
- CPU : (3×0.5) + (2×1) + 2 + 0.5 = 6 cœurs (75% du serveur)

**Marge disponible** : Suffisante pour absorber les pics et le système hôte.

## Problèmes courants et solutions

### Problème 1 : Conteneur tué (OOMKilled)

**Symptômes** :
```bash
docker ps -a
# STATUS: Exited (137)

docker inspect my-container --format='{{.State.OOMKilled}}'
# true
```

**Solutions** :
1. Augmenter la limite mémoire
2. Optimiser l'application (fuites mémoire ?)
3. Ajouter du swap (temporairement)

```bash
# Augmenter la limite
docker run --memory="2g" my-app

# Ajouter du swap
docker run --memory="1g" --memory-swap="2g" my-app
```

### Problème 2 : Application lente avec CPU limité

**Symptômes** :
- Application qui répond lentement
- docker stats montre CPU à 100% de la limite

**Solutions** :
```bash
# Augmenter la limite CPU
docker run --cpus="2" my-app

# Ou réduire la priorité d'autres conteneurs
docker update --cpu-shares=512 other-container
```

### Problème 3 : Impossible de démarrer (mémoire insuffisante)

**Symptômes** :
```bash
docker run --memory="100m" postgres
# Error: cannot allocate memory
```

**Solution** :
```bash
# Vérifier les besoins minimums de l'image
docker run --memory="512m" postgres  # Postgres nécessite au moins 256m
```

### Problème 4 : Ressources non libérées après arrêt

**Solution** :
```bash
# Supprimer les conteneurs arrêtés
docker container prune

# Libérer les volumes non utilisés
docker volume prune

# Nettoyage complet
docker system prune -a
```

## Outils de monitoring avancés

### cAdvisor (Container Advisor)

```bash
docker run -d \
  --name=cadvisor \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --publish=8080:8080 \
  google/cadvisor:latest
```

Accessible sur http://localhost:8080

### Prometheus + Grafana

Pour un monitoring professionnel, combinez :
- **Prometheus** : Collecte des métriques
- **Grafana** : Visualisation
- **cAdvisor** : Exporte les métriques Docker

## Mise à jour dynamique des ressources

Vous pouvez modifier les limites d'un conteneur en cours d'exécution :

```bash
# Augmenter la mémoire
docker update --memory="2g" my-container

# Modifier le CPU
docker update --cpus="2" my-container

# Modifier les CPU shares
docker update --cpu-shares=1024 my-container

# Plusieurs modifications simultanées
docker update --memory="2g" --cpus="2" my-container
```

**Attention** : Certaines limitations ne peuvent pas être mises à jour dynamiquement (ex: cpuset-cpus).

## Résumé

La gestion des ressources dans Docker est **essentielle** pour :

### Limitations mémoire
- `--memory` : Limite dure de RAM
- `--memory-reservation` : Réservation souple
- `--memory-swap` : Contrôle du swap

### Limitations CPU
- `--cpus` : Nombre de cœurs (méthode simple)
- `--cpu-shares` : Priorité relative entre conteneurs
- `--cpuset-cpus` : Épinglage sur des cœurs spécifiques

### Commandes clés
```bash
# Lancer avec limites
docker run --memory="1g" --cpus="1" my-app

# Surveiller
docker stats

# Mettre à jour
docker update --memory="2g" my-container
```

### Règles d'or

1. ⚠️ **Toujours définir des limites en production**
2. 📊 **Mesurer avant de limiter**
3. 🔄 **Ajuster progressivement selon les observations**
4. 📈 **Surveiller régulièrement**
5. 🎯 **Prévoir une marge de 30-50%**

La gestion appropriée des ressources garantit la **stabilité**, la **performance** et la **prévisibilité** de vos environnements Docker.

⏭️ [Health checks](/11-sujets-intermediaires/03-health-checks.md)
