🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 4.4 Supprimer conteneurs et images (rm, rmi, prune)

## Introduction

Au fil du temps, votre système Docker accumule des **conteneurs arrêtés** et des **images inutilisées** qui occupent de l'espace disque. Savoir supprimer proprement ces ressources est essentiel pour :

- **Libérer de l'espace disque** : les images Docker peuvent être volumineuses (plusieurs centaines de Mo à plusieurs Go)
- **Maintenir un environnement propre** : éviter la confusion avec des conteneurs obsolètes
- **Améliorer les performances** : moins de ressources à gérer pour Docker
- **Faciliter la maintenance** : un environnement organisé est plus facile à gérer

⚠️ **Important** : La suppression est **définitive** et **irréversible**. Soyez toujours certain de ce que vous supprimez, surtout en production !

## Vérifier l'espace disque utilisé

Avant de commencer à supprimer, il est utile de voir combien d'espace Docker utilise :

```bash
docker system df
```

Sortie exemple :
```
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          10        5         2.5GB     1.2GB (48%)
Containers      15        3         150MB     145MB (96%)
Local Volumes   5         2         500MB     300MB (60%)
Build Cache     0         0         0B        0B
```

Cette commande vous montre :
- Combien de ressources existent
- Combien sont actuellement utilisées
- L'espace total occupé
- L'espace récupérable

## La commande `docker rm` : supprimer des conteneurs

### Qu'est-ce que `docker rm` ?

`docker rm` (remove) supprime un ou plusieurs conteneurs. Cette commande ne fonctionne que sur des conteneurs **arrêtés**. Vous ne pouvez pas supprimer un conteneur en cours d'exécution (sauf avec l'option `-f`).

### Syntaxe de base

```bash
docker rm <nom_ou_id_conteneur>
```

### Supprimer un conteneur arrêté

```bash
docker rm mon-conteneur
```

Si le conteneur est bien arrêté, Docker affiche son nom ou son ID confirmant la suppression :
```
mon-conteneur
```

### Supprimer un conteneur en cours d'exécution avec `-f`

Si vous essayez de supprimer un conteneur en cours d'exécution, vous obtiendrez une erreur :

```bash
docker rm mon-nginx
```

Erreur :
```
Error response from daemon: You cannot remove a running container. Stop the container before attempting removal or force remove
```

Pour forcer la suppression (arrêt puis suppression) :

```bash
docker rm -f mon-nginx
```

Ou la version longue :
```bash
docker rm --force mon-nginx
```

⚠️ **Attention** : L'option `-f` arrête brutalement le conteneur sans lui laisser le temps de se fermer proprement. Préférez toujours `docker stop` puis `docker rm`.

### Supprimer plusieurs conteneurs

Vous pouvez supprimer plusieurs conteneurs en une seule commande :

```bash
docker rm conteneur1 conteneur2 conteneur3
```

### Supprimer tous les conteneurs arrêtés

**Méthode 1** : Utiliser `docker ps` avec `-q` et `-f`

```bash
docker rm $(docker ps -aq -f status=exited)
```

Explication :
- `docker ps -aq -f status=exited` : liste les IDs de tous les conteneurs arrêtés
- `docker rm` reçoit cette liste et supprime tous ces conteneurs

**Méthode 2** : Utiliser `docker container prune` (voir plus loin)

### Supprimer les volumes associés avec `-v`

Quand vous supprimez un conteneur, ses volumes anonymes restent par défaut. Pour les supprimer également :

```bash
docker rm -v mon-conteneur
```

⚠️ **Attention** : Cette option supprime uniquement les volumes anonymes, pas les volumes nommés qui restent préservés.

### Supprimer avec un filtre

Supprimer tous les conteneurs créés il y a plus de 24 heures (arrêtés) :

```bash
docker container prune --filter "until=24h"
```

## La commande `docker rmi` : supprimer des images

### Qu'est-ce que `docker rmi` ?

`docker rmi` (remove image) supprime une ou plusieurs images Docker de votre système local. Une image ne peut être supprimée que si **aucun conteneur** (même arrêté) ne l'utilise.

### Syntaxe de base

```bash
docker rmi <nom_ou_id_image>
```

Ou la nouvelle syntaxe :
```bash
docker image rm <nom_ou_id_image>
```

### Supprimer une image

```bash
docker rmi nginx:alpine
```

Si l'image n'est utilisée par aucun conteneur, Docker la supprime et affiche :
```
Untagged: nginx:alpine
Untagged: nginx:alpine@sha256:...
Deleted: sha256:8e75cbc5b25c...
Deleted: sha256:a2c3f4e5b6d7...
```

### Erreur courante : image en cours d'utilisation

Si vous essayez de supprimer une image utilisée par un conteneur :

```bash
docker rmi nginx
```

Erreur :
```
Error response from daemon: conflict: unable to remove repository reference "nginx" (must force) - container f4d5e8a9 is using its referenced image 605c77e6
```

**Solutions** :
1. Supprimer d'abord le conteneur qui utilise l'image :
   ```bash
   docker rm f4d5e8a9
   docker rmi nginx
   ```

2. Forcer la suppression avec `-f` (déconseillé) :
   ```bash
   docker rmi -f nginx
   ```

⚠️ **Note** : Forcer la suppression ne supprime que le tag, pas nécessairement l'image complète si elle est encore référencée par un conteneur.

### Supprimer plusieurs images

```bash
docker rmi nginx:latest nginx:alpine python:3.11
```

### Supprimer toutes les images non utilisées

**Attention** : Cette commande supprime toutes les images qui ne sont pas utilisées par au moins un conteneur !

```bash
docker rmi $(docker images -q)
```

Cette commande est **très destructive**. Utilisez-la avec précaution !

Une alternative plus sûre est d'utiliser `docker image prune` (voir plus loin).

### Supprimer les images "dangling" (pendantes)

Les images dangling sont des images sans tag, souvent créées lors de reconstructions :

```
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
<none>       <none>    4a5e9c7d8b3f   1 hour ago     150MB
```

Pour les lister :
```bash
docker images -f dangling=true
```

Pour les supprimer :
```bash
docker rmi $(docker images -f dangling=true -q)
```

Ou plus simplement :
```bash
docker image prune
```

### Supprimer une image par son ID

Vous pouvez utiliser l'ID complet ou juste les premiers caractères (tant qu'ils sont uniques) :

```bash
docker rmi 605c77e6
```

### Supprimer toutes les images d'un repository

Pour supprimer toutes les versions d'une image :

```bash
docker rmi $(docker images nginx -q)
```

Cela supprime toutes les images nginx (latest, alpine, 1.25, etc.).

## La commande `docker prune` : nettoyage automatisé

### Qu'est-ce que `docker prune` ?

Les commandes `prune` sont des **outils de nettoyage automatique** qui suppriment les ressources inutilisées. C'est la méthode recommandée pour maintenir un environnement Docker propre.

Il existe plusieurs variantes de prune selon le type de ressource :

### `docker container prune` : nettoyer les conteneurs

Supprime tous les conteneurs **arrêtés** :

```bash
docker container prune
```

Docker vous demandera confirmation :
```
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N]
```

Tapez `y` pour confirmer.

**Pour ignorer la confirmation** :
```bash
docker container prune -f
```

Ou :
```bash
docker container prune --force
```

**Exemple de sortie** :
```
Deleted Containers:
f4d5e8a9b2c1d3e4f5a6b7c8d9e0f1a2
a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6
9e8d7c6b5a4f3e2d1c0b9a8e7d6c5b4

Total reclaimed space: 145MB
```

**Avec filtre temporel** :

Supprimer les conteneurs arrêtés depuis plus de 24 heures :
```bash
docker container prune --filter "until=24h"
```

### `docker image prune` : nettoyer les images

Par défaut, `docker image prune` supprime uniquement les images **dangling** (sans tag) :

```bash
docker image prune
```

**Pour supprimer toutes les images non utilisées** (y compris celles avec tags) :

```bash
docker image prune -a
```

Ou :
```bash
docker image prune --all
```

⚠️ **Attention** : `-a` supprime TOUTES les images qui ne sont pas utilisées par au moins un conteneur, même celles que vous venez de télécharger !

**Exemple de sortie** :
```
WARNING! This will remove all images without at least one container associated to them.
Are you sure you want to continue? [y/N] y

Deleted Images:
untagged: nginx:alpine
deleted: sha256:8e75cbc5b25c...
deleted: sha256:a2c3f4e5b6d7...

Total reclaimed space: 1.2GB
```

**Avec filtre** :

Supprimer les images créées il y a plus de 7 jours :
```bash
docker image prune -a --filter "until=168h"
```

### `docker volume prune` : nettoyer les volumes

Supprime tous les volumes **non utilisés** par au moins un conteneur :

```bash
docker volume prune
```

⚠️ **Attention** : Les données dans les volumes seront perdues définitivement !

### `docker network prune` : nettoyer les réseaux

Supprime tous les réseaux personnalisés **non utilisés** :

```bash
docker network prune
```

Les réseaux par défaut (bridge, host, none) ne sont jamais supprimés.

### `docker system prune` : nettoyage complet

Cette commande combine plusieurs prune pour un **nettoyage global** :

```bash
docker system prune
```

Par défaut, elle supprime :
- Tous les conteneurs arrêtés
- Tous les réseaux non utilisés
- Toutes les images dangling
- Tout le cache de build

**Pour un nettoyage complet incluant toutes les images** :

```bash
docker system prune -a
```

Cette commande supprime également :
- Toutes les images non utilisées (pas seulement les dangling)

**Pour inclure les volumes** :

```bash
docker system prune -a --volumes
```

⚠️ **Attention extrême** : Cette commande supprime presque tout ! N'utilisez jamais cette commande en production sans être absolument certain !

**Exemple de sortie** :
```
WARNING! This will remove:
  - all stopped containers
  - all networks not used by at least one container
  - all images without at least one container associated to them
  - all build cache

Are you sure you want to continue? [y/N] y

Deleted Containers:
...

Deleted Images:
...

Deleted Networks:
...

Total reclaimed space: 3.5GB
```

### Filtres temporels utiles

Les commandes prune acceptent des filtres temporels :

```bash
# Conteneurs arrêtés depuis plus de 1 heure
docker container prune --filter "until=1h"

# Images créées il y a plus de 7 jours
docker image prune -a --filter "until=168h"

# Nettoyage système de ressources de plus de 24 heures
docker system prune --filter "until=24h"
```

## Comparaison des commandes de suppression

| Commande | Ce qu'elle supprime | Confirmation requise |
|----------|---------------------|----------------------|
| `docker rm` | Conteneur(s) spécifique(s) | Non |
| `docker rmi` | Image(s) spécifique(s) | Non |
| `docker container prune` | Tous les conteneurs arrêtés | Oui (sauf avec `-f`) |
| `docker image prune` | Images dangling | Oui (sauf avec `-f`) |
| `docker image prune -a` | Toutes les images non utilisées | Oui (sauf avec `-f`) |
| `docker system prune` | Conteneurs, réseaux, images dangling, cache | Oui (sauf avec `-f`) |
| `docker system prune -a` | Presque tout | Oui (sauf avec `-f`) |

## Stratégies de nettoyage

### 1. Nettoyage quotidien léger

Pour un environnement de développement :

```bash
# Supprimer les conteneurs arrêtés
docker container prune -f

# Supprimer les images dangling
docker image prune -f
```

### 2. Nettoyage hebdomadaire moyen

```bash
# Supprimer les ressources inutilisées depuis plus de 7 jours
docker system prune --filter "until=168h" -f
```

### 3. Nettoyage mensuel complet

```bash
# Arrêter tous les conteneurs
docker stop $(docker ps -q)

# Nettoyage complet (sauf volumes)
docker system prune -a -f

# Vérifier l'espace récupéré
docker system df
```

### 4. Reset complet (développement seulement)

Pour repartir de zéro (⚠️ **destructif !**) :

```bash
# Arrêter et supprimer tous les conteneurs
docker stop $(docker ps -aq)
docker rm $(docker ps -aq)

# Supprimer toutes les images
docker rmi $(docker images -q)

# Supprimer tous les volumes
docker volume rm $(docker volume ls -q)

# Supprimer tous les réseaux personnalisés
docker network rm $(docker network ls -q)
```

## Automatiser le nettoyage

### Script de nettoyage hebdomadaire

Créez un fichier `docker-cleanup.sh` :

```bash
#!/bin/bash
echo "🧹 Nettoyage Docker hebdomadaire"
echo "================================"

echo "📊 Espace avant nettoyage :"
docker system df

echo ""
echo "🗑️  Suppression des conteneurs arrêtés..."
docker container prune -f

echo "🗑️  Suppression des images dangling..."
docker image prune -f

echo "🗑️  Suppression des volumes non utilisés..."
docker volume prune -f

echo "🗑️  Suppression des réseaux non utilisés..."
docker network prune -f

echo ""
echo "📊 Espace après nettoyage :"
docker system df

echo ""
echo "✅ Nettoyage terminé !"
```

Rendez-le exécutable :
```bash
chmod +x docker-cleanup.sh
```

Exécutez-le :
```bash
./docker-cleanup.sh
```

### Planifier avec cron (Linux/Mac)

Pour exécuter automatiquement chaque dimanche à 2h du matin :

```bash
# Ouvrir crontab
crontab -e

# Ajouter cette ligne
0 2 * * 0 /chemin/vers/docker-cleanup.sh >> /var/log/docker-cleanup.log 2>&1
```

## Bonnes pratiques

### 1. Utilisez `--rm` lors du développement

Pour les conteneurs de test, utilisez toujours `--rm` :

```bash
docker run --rm -it ubuntu bash
```

Le conteneur sera automatiquement supprimé à l'arrêt.

### 2. Vérifiez avant de supprimer

Toujours lister ce qui sera supprimé avant d'exécuter la commande :

```bash
# Voir les conteneurs qui seront supprimés
docker ps -a -f status=exited

# Puis supprimer
docker container prune -f
```

### 3. Utilisez des filtres temporels

Ne supprimez pas tout d'un coup, utilisez des filtres :

```bash
# Supprimer uniquement les anciennes ressources
docker system prune --filter "until=168h"
```

### 4. Sauvegardez les images importantes

Avant un nettoyage majeur, exportez vos images importantes :

```bash
docker save mon-image:latest -o mon-image-backup.tar
```

Pour la restaurer plus tard :
```bash
docker load -i mon-image-backup.tar
```

### 5. Documentez vos suppressions

En production, documentez toujours vos actions :

```bash
# Créer un log
echo "$(date): docker system prune exécuté" >> /var/log/docker-maintenance.log
docker system prune -f >> /var/log/docker-maintenance.log 2>&1
```

### 6. Attention en production

⚠️ En production, **JAMAIS** :
- `docker system prune -a` sans vérification préalable
- Supprimer des volumes sans backup
- Forcer la suppression de conteneurs critiques

✅ En production, **TOUJOURS** :
- Faire des sauvegardes avant
- Vérifier les ressources utilisées
- Utiliser des filtres précis
- Tester dans un environnement de staging

### 7. Surveillez l'espace disque

Configurez des alertes quand l'espace disque devient critique :

```bash
# Ajouter à votre monitoring
docker system df -v
```

## Cas d'usage courants

### Libérer rapidement de l'espace

```bash
docker system prune -a -f --volumes
```

### Supprimer toutes les images d'un projet spécifique

```bash
docker rmi $(docker images "mon-projet*" -q)
```

### Supprimer les conteneurs d'une image spécifique

```bash
docker rm $(docker ps -aq --filter ancestor=nginx)
```

### Nettoyer après des tests

```bash
# Supprimer tous les conteneurs créés dans les dernières 2 heures
docker container prune --filter "until=2h" -f
```

## Vérifier l'impact du nettoyage

### Avant le nettoyage

```bash
docker system df -v
```

Notez la taille utilisée.

### Après le nettoyage

```bash
docker system df -v
```

Comparez l'espace récupéré.

## Récupérer une image ou un conteneur supprimé par erreur

⚠️ **Important** : Une fois qu'un conteneur ou une image est supprimé, il est **impossible** de le récupérer localement. Cependant :

**Pour les images** :
- Vous pouvez les retélécharger depuis Docker Hub (si elles proviennent de là)
  ```bash
  docker pull nginx:latest
  ```

**Pour les conteneurs** :
- Les données dans les volumes nommés sont généralement préservées
- Vous devrez recréer le conteneur avec `docker run`

**Solution** : Faites toujours des sauvegardes des configurations et volumes importants !

## Résumé

Dans cette section, vous avez appris à :

- ✅ **Supprimer des conteneurs** avec `docker rm`
- ✅ **Supprimer des images** avec `docker rmi`
- ✅ **Utiliser les commandes prune** pour un nettoyage automatisé
- ✅ **Comprendre la différence** entre les différentes commandes de nettoyage
- ✅ **Appliquer des filtres temporels** pour des suppressions ciblées
- ✅ **Mettre en place des stratégies** de nettoyage régulier
- ✅ **Suivre les bonnes pratiques** pour éviter les suppressions accidentelles
- ✅ **Surveiller l'espace disque** utilisé par Docker

La gestion de l'espace et le nettoyage régulier sont essentiels pour maintenir un environnement Docker performant et organisé. Avec ces commandes, vous avez tous les outils nécessaires pour garder votre système propre.

Dans la section suivante, vous découvrirez comment interagir avec vos conteneurs en cours d'exécution : exécuter des commandes, consulter les logs, et accéder à l'intérieur des conteneurs.

⏭️ [Interagir avec les conteneurs (exec, logs, attach)](/04-commandes-docker-essentielles/05-interagir-avec-les-conteneurs.md)
