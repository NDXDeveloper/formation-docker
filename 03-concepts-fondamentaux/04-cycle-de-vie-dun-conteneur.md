🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 3.4 Le cycle de vie d'un conteneur

## Introduction

Le cycle de vie d'un conteneur représente **toutes les étapes** qu'un conteneur traverse depuis sa création jusqu'à sa suppression définitive. Comprendre ce cycle est essentiel pour gérer efficacement vos conteneurs Docker au quotidien.

Tout comme un processus sur votre ordinateur qui peut être démarré, mis en pause, redémarré ou fermé, un conteneur Docker traverse différents **états** tout au long de son existence.

### Analogie : Un programme sur votre ordinateur

Pensez à une application comme Microsoft Word :

1. **Installation** → Image Docker téléchargée
2. **Lancement** → Conteneur créé et démarré
3. **En cours d'utilisation** → Conteneur en exécution (running)
4. **Minimisation** → Conteneur en pause
5. **Fermeture** → Conteneur arrêté (stopped)
6. **Désinstallation** → Conteneur supprimé

## Les états d'un conteneur

Un conteneur Docker peut se trouver dans l'un des états suivants :

### Vue d'ensemble des états

```
┌─────────────────────────────────────────────────────────┐
│                    CYCLE DE VIE                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────┐   docker create                            │
│  │  IMAGE  │ ──────────────────┐                        │
│  └─────────┘                   │                        │
│                                ↓                        │
│                          ┌──────────┐                   │
│                          │ CREATED  │                   │
│                          └────┬─────┘                   │
│                               │ docker start            │
│                               ↓                         │
│    ┌──────────────────────────────────────┐             │
│    │          ┌──────────┐                │             │
│    │          │ RUNNING  │ ←──────────┐   │             │
│    │          └────┬─────┘            │   │             │
│    │               │                  │   │             │
│    │     ┌─────────┼─────────┐        │   │             │
│    │     │         │         │        │   │             │
│    │  docker    docker   docker    restart│             │
│    │   pause     stop     kill        │   │             │
│    │     │         │         │        │   │             │
│    │     ↓         ↓         ↓        │   │             │
│    │ ┌────────┐ ┌────────┐ ┌────────┐ │   │             │
│    │ │ PAUSED │ │ STOPPED│ │ EXITED │ │   │             │
│    │ └────┬───┘ └───┬────┘ └───┬────┘ │   │             │
│    │      │         │          │      │   │             │
│    │   unpause   restart    restart   │   │             │
│    │      └─────────┴──────────┴──────┘   │             │
│    │                                      │             │
│    └──────────────────────────────────────┘             │
│                     │                                   │
│                docker rm                                │
│                     ↓                                   │
│              ┌──────────┐                               │
│              │ REMOVED  │                               │
│              └──────────┘                               │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

## Description détaillée de chaque état

### 1. CREATED (Créé)

**Définition :** Le conteneur a été créé mais n'a pas encore été démarré.

**Caractéristiques :**
- Le conteneur existe en tant qu'entité
- Les ressources sont allouées
- La configuration est en place
- **Aucun processus n'est lancé**
- Le conteneur consomme très peu de ressources

**Commande pour atteindre cet état :**
```bash
docker create <image>
```

**Exemple concret :**
```bash
docker create nginx
# Crée un conteneur nginx mais ne le démarre pas
```

**Quand l'utiliser ?**
- Rarement utilisé directement
- Utile pour préparer un conteneur avant de le démarrer
- Permet de configurer avant l'exécution

**Analogie :** Vous avez installé Word sur votre ordinateur, mais vous ne l'avez pas encore ouvert.

### 2. RUNNING (En cours d'exécution)

**Définition :** Le conteneur est **actif** et son processus principal est en cours d'exécution.

**Caractéristiques :**
- Le processus principal est lancé et s'exécute
- L'application est accessible et fonctionnelle
- Le conteneur consomme des ressources (CPU, mémoire)
- C'est l'état "productif" où le conteneur fait son travail

**Commandes pour atteindre cet état :**
```bash
# Créer ET démarrer en une seule commande
docker run <image>

# Démarrer un conteneur créé ou arrêté
docker start <conteneur>

# Redémarrer un conteneur
docker restart <conteneur>
```

**Exemple concret :**
```bash
docker run -d nginx
# Crée et démarre un serveur nginx en arrière-plan
```

**États internes :**
- Le processus principal (PID 1) est actif
- Les processus enfants peuvent également tourner
- Les ports exposés sont accessibles
- Les logs sont générés en temps réel

**Analogie :** Word est ouvert et vous êtes en train de travailler sur un document.

### 3. PAUSED (En pause)

**Définition :** Le conteneur est **temporairement figé**, tous ses processus sont suspendus.

**Caractéristiques :**
- Les processus sont gelés en mémoire
- Aucune instruction n'est exécutée
- L'état est conservé en RAM
- **Très peu de CPU utilisé**, mais la mémoire reste occupée
- Peut être réactivé instantanément

**Commandes :**
```bash
# Mettre en pause
docker pause <conteneur>

# Reprendre
docker unpause <conteneur>
```

**Exemple concret :**
```bash
docker pause mon-conteneur-web
# Les requêtes HTTP sont mises en attente
# Le processus nginx est figé

docker unpause mon-conteneur-web
# Tout reprend instantanément
```

**Cas d'usage :**
- Libérer temporairement du CPU
- Déboguer un conteneur à un instant précis
- Maintenance système sans perdre l'état

**⚠️ Note :** État rarement utilisé en production, plus utile pour le développement et les tests.

**Analogie :** Vous mettez Word en pause avec Ctrl+Z dans un terminal Linux.

### 4. STOPPED (Arrêté)

**Définition :** Le conteneur a été **arrêté proprement**, son processus principal est terminé.

**Caractéristiques :**
- Le processus principal s'est arrêté
- Les données en mémoire sont perdues
- Le système de fichiers du conteneur est conservé
- Peut être redémarré
- Consomme zéro CPU et zéro mémoire

**Commandes :**
```bash
# Arrêt propre (envoi du signal SIGTERM)
docker stop <conteneur>

# Arrêt forcé immédiat (envoi du signal SIGKILL)
docker kill <conteneur>
```

**Différence stop vs kill :**

| Commande | Comportement | Délai | Usage |
|----------|-------------|-------|-------|
| `docker stop` | Arrêt gracieux (SIGTERM) | 10 secondes par défaut | Normal |
| `docker kill` | Arrêt brutal (SIGKILL) | Immédiat | Conteneur bloqué |

**Exemple concret :**
```bash
docker stop mon-api
# L'API termine ses requêtes en cours (jusqu'à 10s)
# Puis s'arrête proprement

docker kill mon-api
# L'API s'arrête immédiatement
# Les requêtes en cours sont interrompues
```

**Ce qui est conservé :**
- ✅ La couche en lecture/écriture du conteneur
- ✅ Les logs
- ✅ La configuration
- ❌ Les données en mémoire RAM

**Ce qui est perdu :**
- ❌ L'état des processus
- ❌ Les connexions réseau actives
- ❌ Les données en mémoire non sauvegardées

**Analogie :** Vous fermez Word. Votre document est sauvegardé, mais Word n'est plus en mémoire.

### 5. EXITED (Sorti/Terminé)

**Définition :** Le conteneur s'est terminé, soit **normalement** soit suite à une **erreur**.

**Caractéristiques :**
- Le processus principal s'est arrêté (code de sortie disponible)
- État similaire à STOPPED
- Conserve un **code de sortie** (exit code)
  - `0` = Terminaison normale (succès)
  - `≠ 0` = Erreur (échec)

**Comment y arriver :**
- Le processus principal se termine naturellement
- Une erreur fatale se produit
- Commande `docker stop` ou `docker kill`

**Exemple concret :**
```bash
# Script qui se termine après son exécution
docker run ubuntu echo "Hello World"
# Le conteneur passe automatiquement à EXITED après avoir affiché le texte

# Code de sortie 0 (succès)
docker run ubuntu bash -c "exit 0"

# Code de sortie 1 (erreur)
docker run ubuntu bash -c "exit 1"
```

**Inspection du code de sortie :**
```bash
docker ps -a
# La colonne STATUS affiche : Exited (0) ou Exited (1)
```

**Codes de sortie courants :**

| Code | Signification |
|------|---------------|
| 0 | Succès - Tout s'est bien passé |
| 1 | Erreur générale - Quelque chose a échoué |
| 2 | Mauvaise utilisation de la commande shell |
| 126 | Commande invoquée ne peut pas être exécutée |
| 127 | Commande introuvable |
| 130 | Interruption par Ctrl+C |
| 137 | Processus tué (SIGKILL) |
| 143 | Arrêt gracieux (SIGTERM) |

**Analogie :** Vous avez fermé Word et le système a enregistré si la fermeture s'est bien passée ou non.

### 6. REMOVED (Supprimé)

**Définition :** Le conteneur a été **définitivement supprimé** du système.

**Caractéristiques :**
- Le conteneur n'existe plus
- Toutes ses données sont perdues (sauf volumes persistants)
- L'image source reste intacte
- Impossible de le redémarrer
- Pour le recréer, il faut repartir de l'image

**Commandes :**
```bash
# Supprimer un conteneur arrêté
docker rm <conteneur>

# Supprimer un conteneur en cours d'exécution (force)
docker rm -f <conteneur>

# Supprimer tous les conteneurs arrêtés
docker container prune
```

**Ce qui est supprimé :**
- ❌ Le conteneur lui-même
- ❌ La couche en lecture/écriture
- ❌ Les logs
- ❌ Les métadonnées
- ✅ Les volumes nommés persistent (si non explicitement supprimés)
- ✅ L'image source reste disponible

**Exemple concret :**
```bash
# Conteneur arrêté
docker stop mon-conteneur
docker rm mon-conteneur

# Conteneur en cours d'exécution (avec force)
docker rm -f mon-conteneur-actif

# Nettoyer tous les conteneurs arrêtés
docker container prune
# WARNING! This will remove all stopped containers.
```

**⚠️ Attention :** Cette action est **irréversible** !

**Analogie :** Vous désinstallez complètement Word de votre ordinateur.

## Transitions entre les états

### Vue simplifiée des transitions

```
IMAGE
  │
  ├─ docker run ─────────────→ RUNNING
  │
  └─ docker create ──────────→ CREATED
                                  │
                    docker start ─┘
                                  ↓
                               RUNNING
                                  │
                    ┌─────────────┼─────────────┐
                    │             │             │
              docker pause   docker stop   docker kill
                    │             │             │
                    ↓             ↓             ↓
                 PAUSED        STOPPED       EXITED
                    │             │             │
              docker unpause      └──── docker restart
                    │                           │
                    └──────────→ RUNNING ←──────┘
                                  │
                            docker rm -f
                                  ↓
                              REMOVED
```

### Tableau récapitulatif des commandes

| État actuel | Commande | État final | Description |
|-------------|----------|------------|-------------|
| IMAGE | `docker create` | CREATED | Crée le conteneur sans le démarrer |
| IMAGE | `docker run` | RUNNING | Crée et démarre en une fois |
| CREATED | `docker start` | RUNNING | Démarre le conteneur |
| RUNNING | `docker pause` | PAUSED | Fige temporairement |
| RUNNING | `docker stop` | STOPPED | Arrêt propre |
| RUNNING | `docker kill` | EXITED | Arrêt brutal |
| RUNNING | `docker rm -f` | REMOVED | Supprime de force |
| PAUSED | `docker unpause` | RUNNING | Reprend l'exécution |
| STOPPED | `docker start` | RUNNING | Redémarre |
| STOPPED | `docker rm` | REMOVED | Supprime |
| EXITED | `docker start` | RUNNING | Redémarre |
| EXITED | `docker rm` | REMOVED | Supprime |

## Scénarios courants du cycle de vie

### Scénario 1 : Cycle de vie normal

**Workflow typique pour une application web :**

```
1. Téléchargement
   docker pull nginx:latest

2. Création et démarrage
   docker run -d -p 8080:80 --name mon-web nginx

3. Vérification
   docker ps
   # STATUS: Up 2 minutes

4. Application en production
   [Plusieurs heures/jours de fonctionnement]

5. Mise à jour nécessaire - Arrêt propre
   docker stop mon-web
   # STATUS: Exited (0)

6. Démarrage de la nouvelle version
   docker run -d -p 8080:80 --name mon-web-v2 nginx:1.24

7. Nettoyage de l'ancienne version
   docker rm mon-web
```

### Scénario 2 : Développement itératif

**Workflow de développement avec modifications fréquentes :**

```
1. Démarrage pour tests
   docker run -d --name dev-api mon-api:dev

2. Tests et détection d'un bug
   docker logs dev-api

3. Arrêt pour correction
   docker stop dev-api

4. Modification du code source
   [Édition des fichiers]

5. Reconstruction de l'image
   docker build -t mon-api:dev .

6. Suppression de l'ancien conteneur
   docker rm dev-api

7. Nouveau test
   docker run -d --name dev-api mon-api:dev

8. Répéter 2-7 jusqu'à satisfaction
```

### Scénario 3 : Débogage avec pause

**Utile pour inspecter l'état à un moment précis :**

```
1. Application en cours
   docker run -d --name app-debug mon-app

2. Détection d'un comportement étrange
   [Logs montrent une anomalie]

3. Mise en pause pour inspection
   docker pause app-debug

4. Inspection de l'état
   docker inspect app-debug
   docker exec app-debug cat /tmp/debug.log

5. Reprise
   docker unpause app-debug

6. Monitoring
   docker stats app-debug
```

### Scénario 4 : Gestion d'erreur

**Conteneur qui crash à répétition :**

```
1. Démarrage
   docker run -d --name unstable-app mon-app

2. Crash de l'application
   # STATUS: Exited (137)

3. Inspection du problème
   docker logs unstable-app
   docker inspect unstable-app

4. Plusieurs tentatives
   docker start unstable-app
   # Crash à nouveau

5. Décision de supprimer
   docker rm unstable-app

6. Correction et reconstruction
   [Fix du code]
   docker build -t mon-app:fixed .

7. Nouveau test
   docker run -d --name stable-app mon-app:fixed
```

## Politiques de redémarrage

Docker permet de définir des **politiques de redémarrage automatique** pour gérer les conteneurs qui s'arrêtent.

### Les 4 politiques disponibles

**1. no (par défaut)**
```bash
docker run --restart=no nginx
```
- Le conteneur ne redémarre jamais automatiquement
- Vous devez le redémarrer manuellement

**2. on-failure**
```bash
docker run --restart=on-failure nginx
```
- Redémarre uniquement si le conteneur se termine avec un code d'erreur (≠ 0)
- Ne redémarre pas si le conteneur se termine normalement (code 0)
- Utile pour les tâches qui peuvent échouer temporairement

**Variante avec limite :**
```bash
docker run --restart=on-failure:3 nginx
```
- Redémarre maximum 3 fois en cas d'échec
- Après 3 échecs, abandonne

**3. always**
```bash
docker run --restart=always nginx
```
- Redémarre toujours le conteneur s'il s'arrête
- Même après un `docker stop` manuel
- Au démarrage du système Docker, relance automatiquement le conteneur
- **Idéal pour les services en production**

**4. unless-stopped**
```bash
docker run --restart=unless-stopped nginx
```
- Similaire à `always`
- MAIS : ne redémarre pas si vous l'avez arrêté explicitement avec `docker stop`
- Au démarrage du système Docker, relance seulement les conteneurs qui tournaient
- **Recommandé pour la production**

### Tableau comparatif

| Politique | Arrêt normal | Arrêt erreur | docker stop | Redémarrage système |
|-----------|--------------|--------------|-------------|---------------------|
| `no` | Non | Non | Non | Non |
| `on-failure` | Non | Oui | Non | Oui (si en erreur) |
| `always` | Oui | Oui | Oui | Oui |
| `unless-stopped` | Oui | Oui | Non | Oui (si non stoppé) |

### Exemples pratiques

**Service web critique (doit toujours tourner) :**
```bash
docker run -d \
  --name web-prod \
  --restart=unless-stopped \
  -p 80:80 \
  nginx
```

**Tâche de traitement pouvant échouer temporairement :**
```bash
docker run -d \
  --name worker \
  --restart=on-failure:5 \
  mon-worker
```

**Service de développement (pas de redémarrage automatique) :**
```bash
docker run -d \
  --name dev-server \
  --restart=no \
  node-dev-server
```

## Durée de vie et nettoyage

### Conteneurs éphémères

Par défaut, les conteneurs sont **éphémères** :
- Créés pour une tâche spécifique
- Supprimés après utilisation
- Ne laissent pas de traces (sauf volumes)

**Mode éphémère automatique :**
```bash
docker run --rm nginx
# Le conteneur se supprime automatiquement après son arrêt
```

**Avantages :**
- Pas d'accumulation de conteneurs inutiles
- Gestion automatique du cycle de vie
- Parfait pour les tâches ponctuelles

### Nettoyage manuel

**Supprimer un conteneur spécifique :**
```bash
docker rm conteneur-id
```

**Supprimer plusieurs conteneurs :**
```bash
docker rm conteneur1 conteneur2 conteneur3
```

**Supprimer tous les conteneurs arrêtés :**
```bash
docker container prune

# Avec confirmation automatique
docker container prune -f
```

**Supprimer TOUS les conteneurs (même running) :**
```bash
docker rm -f $(docker ps -aq)
```

⚠️ **Attention :** Cette commande est très destructive !

### Bonnes pratiques de nettoyage

✅ **Utilisez `--rm` pour les tâches ponctuelles**
```bash
docker run --rm ubuntu echo "Test"
```

✅ **Nettoyez régulièrement les conteneurs arrêtés**
```bash
# Hebdomadaire ou mensuel
docker container prune
```

✅ **Nommez vos conteneurs importants**
```bash
docker run -d --name prod-database postgres
# Évite la suppression accidentelle
```

✅ **Utilisez des volumes pour les données importantes**
```bash
docker run -v data:/var/lib/postgresql/data postgres
# Les données persistent même après suppression du conteneur
```

❌ **Évitez d'accumuler des dizaines de conteneurs arrêtés**
- Consomme de l'espace disque
- Complique la gestion
- Ralentit certaines opérations Docker

## Surveiller le cycle de vie

### Commandes de surveillance

**Voir les conteneurs en cours d'exécution :**
```bash
docker ps
```

**Voir TOUS les conteneurs (y compris arrêtés) :**
```bash
docker ps -a
```

**Filtrer par état :**
```bash
# Seulement les conteneurs en cours
docker ps --filter "status=running"

# Seulement les conteneurs arrêtés
docker ps --filter "status=exited"

# Seulement les conteneurs en pause
docker ps --filter "status=paused"
```

**Surveiller en temps réel :**
```bash
# Afficher les statistiques d'utilisation
docker stats

# Surveiller un conteneur spécifique
docker stats mon-conteneur

# Suivre les logs en temps réel
docker logs -f mon-conteneur
```

**Historique d'un conteneur :**
```bash
# Voir toutes les modifications
docker diff mon-conteneur

# Voir l'historique des événements
docker events --filter container=mon-conteneur
```

## Résumé des points clés

✅ Le cycle de vie d'un conteneur passe par **plusieurs états** : CREATED, RUNNING, PAUSED, STOPPED, EXITED, REMOVED

✅ Les **transitions** entre états se font via des commandes Docker spécifiques

✅ Un conteneur peut être **redémarré** s'il est arrêté, mais pas s'il est supprimé

✅ Les **politiques de redémarrage** (`--restart`) permettent une gestion automatique

✅ Par défaut, les données dans un conteneur sont **éphémères** et perdues à la suppression

✅ Utilisez `docker ps -a` pour voir **tous les conteneurs**, y compris arrêtés

✅ Nettoyez régulièrement avec `docker container prune` pour économiser l'espace disque

✅ L'option `--rm` supprime **automatiquement** le conteneur après son arrêt

✅ Les **codes de sortie** indiquent si le conteneur s'est terminé correctement (0) ou avec une erreur (≠ 0)

## Pour aller plus loin

Vous avez maintenant une compréhension complète des **concepts fondamentaux de Docker** :
- ✅ Les images (chapitre 3.1)
- ✅ Les conteneurs (chapitre 3.2)
- ✅ Les registres (chapitre 3.3)
- ✅ Le cycle de vie (chapitre 3.4)

Dans le **chapitre 4**, nous allons mettre tout cela en pratique avec les **commandes Docker essentielles** ! Vous apprendrez à manipuler concrètement images et conteneurs, à inspecter leur état, à consulter les logs, et bien plus encore.

C'est là que la théorie rencontre la pratique ! 🚀

---

*Vous maîtrisez maintenant les fondations de Docker. Prêt à passer à l'action avec les commandes pratiques ?*

⏭️ [Commandes Docker essentielles](/04-commandes-docker-essentielles/README.md)
