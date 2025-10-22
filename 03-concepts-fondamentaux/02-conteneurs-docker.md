🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 3.2 Conteneurs Docker

## Qu'est-ce qu'un conteneur Docker ?

Un conteneur Docker est une **instance en cours d'exécution** créée à partir d'une image Docker. C'est votre application qui tourne réellement, isolée dans son propre environnement.

Si l'image est le **plan** ou le **moule**, le conteneur est **l'objet construit** à partir de ce plan, **vivant et actif**.

### Analogies pour bien comprendre

**Analogie 1 - Le programme informatique :**
- **Image** = Le fichier .exe installé sur votre disque dur
- **Conteneur** = Le programme ouvert et en cours d'exécution

**Analogie 2 - La recette de cuisine :**
- **Image** = La recette écrite dans un livre
- **Conteneur** = Le plat que vous cuisinez en suivant cette recette

**Analogie 3 - La programmation orientée objet :**
- **Image** = La classe (le modèle, le blueprint)
- **Conteneur** = L'instance de cette classe (l'objet créé)

## La relation Image → Conteneur

Voici comment fonctionnent les images et les conteneurs ensemble :

```
     IMAGE (ubuntu)
         │
         │  docker run
         │
         ├──────┬──────┬──────┐
         │      │      │      │
         ↓      ↓      ↓      ↓
    CONTENEUR CONTENEUR CONTENEUR CONTENEUR
       #1       #2       #3       #4
    (actif)  (arrêté) (actif)  (actif)
```

**Point clé :** Une seule image peut générer **plusieurs conteneurs** simultanément, chacun fonctionnant de manière totalement indépendante.

### Exemple concret

Imaginons que vous ayez téléchargé l'image `nginx` (un serveur web) :

- **L'image nginx** reste stockée sur votre disque, immuable
- Vous pouvez créer **conteneur-web-1** qui écoute sur le port 8080
- Vous pouvez créer **conteneur-web-2** qui écoute sur le port 8081
- Vous pouvez créer **conteneur-web-test** pour vos tests
- Tous ces conteneurs partagent la même image de base, mais sont totalement indépendants

## Caractéristiques principales des conteneurs

### 1. Isolation

Chaque conteneur est **isolé** des autres conteneurs et du système hôte :

- Possède son propre système de fichiers
- A ses propres processus
- Dispose de sa propre interface réseau
- Utilise ses propres ressources (CPU, mémoire)

**Avantage :** Si un conteneur plante ou est compromis, les autres continuent de fonctionner normalement.

```
┌───────────────────────────────────────────┐
│         SYSTÈME HÔTE (votre PC)           │
│                                           │
│  ┌──────────┐  ┌───────────┐  ┌─────────┐ │
│  │Conteneur │  │Conteneur  │  │Conteneur│ │
│  │   Web    │  │   BDD     │  │  Cache  │ │
│  │  (nginx) │  │ (postgres)│  │ (redis) │ │
│  └──────────┘  └───────────┘  └─────────┘ │
│       ↑             ↑            ↑        │
│       └─────────────┴────────────┘        │
│            Isolés entre eux               │
└───────────────────────────────────────────┘
```

### 2. Éphémérité

Par défaut, les conteneurs sont **éphémères** : toutes les modifications faites à l'intérieur d'un conteneur sont **perdues** lorsque le conteneur est supprimé.

**Exemple :**
1. Vous lancez un conteneur Ubuntu
2. Vous créez un fichier `test.txt` à l'intérieur
3. Vous supprimez le conteneur
4. Vous relancez un nouveau conteneur depuis la même image
5. Le fichier `test.txt` n'existe plus !

**Pourquoi ?** Parce que l'image (le modèle) n'a pas changé, et chaque nouveau conteneur repart d'un état propre.

**Note importante :** Il existe des moyens de persister les données (volumes et bind mounts), que nous verrons au chapitre 6.

### 3. Légèreté et rapidité

Les conteneurs sont **incroyablement légers et rapides** :

- **Démarrage** : Quelques secondes, voire millisecondes
- **Utilisation mémoire** : Seulement ce dont l'application a besoin
- **Espace disque** : Partage des couches avec l'image source

**Comparaison avec une VM :**

| Aspect | Machine Virtuelle | Conteneur Docker |
|--------|------------------|------------------|
| Démarrage | 30 secondes à plusieurs minutes | < 5 secondes |
| Taille | Plusieurs GB | Quelques MB à centaines de MB |
| Système d'exploitation | OS complet | Partage le noyau de l'hôte |
| Isolation | Très forte (hyperviseur) | Forte (namespaces et cgroups) |
| Performances | Overhead significatif | Quasi-natif |

### 4. Portabilité

Un conteneur qui fonctionne sur votre machine de développement fonctionnera **exactement de la même manière** :
- Sur le serveur de production
- Sur la machine d'un collègue
- Dans le cloud (AWS, Azure, Google Cloud)
- Sur n'importe quel système supportant Docker

C'est la solution au fameux problème **"Ça marche sur ma machine !"** 🎉

## Les états d'un conteneur

Un conteneur peut se trouver dans différents états au cours de son cycle de vie :

```
     CRÉATION
         │
         ↓
    ┌─────────┐
    │ CREATED │ ← Conteneur créé mais pas encore démarré
    └────┬────┘
         │ start
         ↓
    ┌─────────┐
    │ RUNNING │ ← Conteneur en cours d'exécution
    └────┬────┘
         │
    ┌────┴────┬──────────┐
    │         │          │
   stop      pause     crash
    │         │          │
    ↓         ↓          ↓
┌─────────┐ ┌────────┐ ┌─────────┐
│ STOPPED │ │ PAUSED │ │ EXITED  │
└────┬────┘ └───┬────┘ └────┬────┘
     │          │           │
     │ restart  │ unpause   │ restart
     └──────────┴───────────┘
              │
              ↓
         ┌─────────┐
         │ REMOVED │ ← Conteneur supprimé (définitif)
         └─────────┘
```

### Description des états

**CREATED** (Créé)
- Le conteneur a été créé mais n'est pas encore démarré
- Les ressources sont allouées mais aucun processus n'est lancé

**RUNNING** (En cours d'exécution)
- Le conteneur est actif et ses processus tournent
- L'application est accessible et fonctionnelle
- C'est l'état "productif" du conteneur

**PAUSED** (En pause)
- Le conteneur est figé temporairement
- Les processus sont suspendus mais restent en mémoire
- Permet de libérer temporairement des ressources CPU

**STOPPED** (Arrêté)
- Le conteneur est arrêté proprement
- Les processus sont terminés
- Les données en mémoire sont perdues, mais le conteneur peut redémarrer

**EXITED** (Sorti/Terminé)
- Le conteneur s'est arrêté (normalement ou suite à une erreur)
- Peut être relancé
- Les logs restent disponibles

**REMOVED** (Supprimé)
- Le conteneur est définitivement supprimé
- Impossible à redémarrer
- Pour le recréer, il faut repartir de l'image

## L'anatomie d'un conteneur

Qu'y a-t-il à l'intérieur d'un conteneur en cours d'exécution ?

```
┌────────────────────────────────────────┐
│         CONTENEUR DOCKER               │
├────────────────────────────────────────┤
│  Couche en lecture/écriture            │ ← Modifications temporaires
├────────────────────────────────────────┤
│  Processus de l'application            │ ← Votre application qui tourne
├────────────────────────────────────────┤
│  Variables d'environnement             │ ← Configuration
├────────────────────────────────────────┤
│  Système de fichiers isolé             │ ← Copie from image + modifs
├────────────────────────────────────────┤
│  Interface réseau virtuelle            │ ← Connexion réseau
├────────────────────────────────────────┤
│  Couches de l'image (lecture seule)    │ ← Hérité de l'image
└────────────────────────────────────────┘
```

### La couche en lecture/écriture

C'est la grande différence entre une image et un conteneur :

- **L'image** : Toutes les couches sont en **lecture seule** (immuables)
- **Le conteneur** : Ajoute une **couche en lecture/écriture** au-dessus de l'image

Toutes les modifications (création de fichiers, modifications, suppressions) sont écrites dans cette couche temporaire. Lorsque le conteneur est supprimé, cette couche disparaît.

```
CONTENEUR
┌────────────────────────────┐
│ Couche R/W (temporaire)    │ ← Vos modifications
├────────────────────────────┤
│ Couche 3 (lecture seule)   │ ┐
├────────────────────────────┤ │
│ Couche 2 (lecture seule)   │ ├─ Provenant de l'IMAGE
├────────────────────────────┤ │
│ Couche 1 (lecture seule)   │ ┘
└────────────────────────────┘
```

## Les ressources d'un conteneur

### Processus

Chaque conteneur exécute un ou plusieurs **processus**. En général, il y a un **processus principal** qui, lorsqu'il se termine, entraîne l'arrêt du conteneur.

**Exemple :**
- Conteneur nginx → Processus principal = serveur nginx
- Conteneur MySQL → Processus principal = serveur MySQL
- Conteneur de script → Processus principal = votre script

**Philosophie Docker :** Un conteneur = Un processus principal = Une responsabilité unique

### Réseau

Chaque conteneur dispose d'une **interface réseau virtuelle** qui lui permet de :
- Communiquer avec d'autres conteneurs
- Communiquer avec le monde extérieur (Internet)
- Exposer des services sur des ports

**Par défaut :**
- Le conteneur obtient une adresse IP privée
- Il peut accéder à Internet
- Il est isolé des autres conteneurs (sauf configuration contraire)

### Système de fichiers

Le conteneur voit un **système de fichiers complet et isolé** qui combine :
- Les couches de l'image (lecture seule)
- Sa propre couche en lecture/écriture
- Éventuellement des volumes montés

**Exemple de structure :**
```
/                    ← Racine du système de fichiers du conteneur
├── bin/
├── etc/
├── home/
├── usr/
├── var/
└── app/             ← Votre application
    └── index.js
```

## Identifiant et nommage des conteneurs

### Container ID

Chaque conteneur reçoit un **identifiant unique** généré automatiquement :

**Format complet :** `abc123def456789...` (64 caractères hexadécimaux)
**Format court :** `abc123def456` (12 premiers caractères, suffisant dans la plupart des cas)

### Container Name

Vous pouvez également donner un **nom explicite** à vos conteneurs :

```bash
# Docker génère un nom aléatoire si vous n'en spécifiez pas
→ elegant_einstein, nervous_tesla, quirky_shannon...

# Vous pouvez spécifier votre propre nom
→ web-server, database-prod, redis-cache...
```

**Bonne pratique :** Utilisez des noms descriptifs pour identifier facilement vos conteneurs, surtout dans des environnements avec de nombreux conteneurs.

## Gestion des ressources

Docker permet de **limiter les ressources** qu'un conteneur peut utiliser :

### CPU
- Limiter le pourcentage de CPU disponible
- Définir la priorité des conteneurs
- Empêcher qu'un conteneur monopolise tout le CPU

### Mémoire
- Définir une limite maximale de RAM
- Configurer le swap
- Protéger le système hôte contre les fuites mémoire

**Pourquoi c'est important ?**
- Éviter qu'un conteneur défaillant consomme toutes les ressources
- Garantir des performances prévisibles
- Optimiser la densité de conteneurs sur un serveur

## Les logs des conteneurs

Chaque conteneur capture automatiquement les **sorties standard** (stdout) et les **erreurs** (stderr) de votre application.

**Avantage :** Pas besoin de configurer des fichiers de logs complexes. Écrivez simplement dans stdout/stderr et Docker s'occupe du reste.

```
Application dans le conteneur
         ↓
  stdout / stderr
         ↓
   Logs Docker
         ↓
  docker logs <conteneur>
```

## Les métadonnées d'un conteneur

Docker conserve de nombreuses **informations** sur chaque conteneur :

- Date de création
- Image source
- Commande exécutée
- Variables d'environnement
- Configuration réseau
- Volumes montés
- État actuel
- Statistiques d'utilisation
- Historique des états

Ces métadonnées sont accessibles via les commandes d'inspection (que nous verrons au chapitre 4).

## Différences clés : Image vs Conteneur

Récapitulons les différences essentielles :

| Caractéristique | Image | Conteneur |
|-----------------|-------|-----------|
| **Nature** | Modèle statique, template | Instance dynamique, en exécution |
| **État** | Immuable, figée | Modifiable, possède un état |
| **Couches** | Toutes en lecture seule | Ajoute une couche R/W |
| **Stockage** | Sur le disque | En mémoire + disque |
| **Nombre** | Une image existe en un exemplaire | Plusieurs conteneurs depuis une image |
| **Processus** | Aucun processus | Exécute des processus |
| **Durée de vie** | Permanente (jusqu'à suppression manuelle) | Éphémère (données perdues à la suppression) |
| **Commande** | `docker images` | `docker ps` |
| **Action** | Se build, se pull, se push | Se run, se start, se stop |

## Cas d'usage typiques

### Développement local
```
Image Python + votre code
        ↓
Conteneur de développement
        ↓
Vous codez, testez, débogguez
```

### Tests
```
Image de votre application
        ↓
Plusieurs conteneurs de test en parallèle
        ↓
Tests automatisés isolés
```

### Production
```
Image versionnée et testée
        ↓
Multiples conteneurs identiques
        ↓
Application scalable et résiliente
```

### Microservices
```
Image service-auth    Image service-api    Image service-data
       ↓                     ↓                      ↓
 Conteneur auth       Conteneur api         Conteneur data
       └──────────────────────┴─────────────────────┘
                    Communication réseau
```

## Isolation et sécurité

### Namespaces

Docker utilise les **namespaces** Linux pour isoler les conteneurs :

- **PID** : Chaque conteneur a son propre arbre de processus
- **Network** : Interface réseau isolée
- **Mount** : Système de fichiers isolé
- **UTS** : Nom d'hôte distinct
- **IPC** : Communication inter-processus isolée
- **User** : Espace utilisateur séparé

### Control Groups (cgroups)

Les **cgroups** limitent et mesurent les ressources utilisées :
- CPU
- Mémoire
- I/O disque
- Réseau

### Sécurité

**Important :** Les conteneurs partagent le noyau du système hôte. Ils sont moins isolés que des VMs, mais suffisamment pour la plupart des usages.

**Bonnes pratiques :**
- Ne pas exécuter les conteneurs en tant que root
- Limiter les capacités (capabilities)
- Utiliser des images de confiance
- Maintenir Docker et les images à jour

## Quand utiliser des conteneurs ?

✅ **Parfait pour :**
- Applications web et APIs
- Microservices
- Environnements de développement reproductibles
- Tests automatisés
- Déploiements rapides et fréquents
- Applications nécessitant une scalabilité horizontale

❌ **Moins adapté pour :**
- Applications graphiques complexes (GUI)
- Applications nécessitant un accès direct au matériel
- Applications avec des exigences d'isolation extrême
- Systèmes legacy complexes et monolithiques

## Résumé des points clés

✅ Un conteneur est une **instance en cours d'exécution** créée à partir d'une image

✅ Les conteneurs sont **isolés**, **légers**, **éphémères** et **portables**

✅ Un conteneur ajoute une **couche en lecture/écriture** au-dessus d'une image immuable

✅ Les modifications dans un conteneur sont **perdues** lorsqu'il est supprimé (sauf utilisation de volumes)

✅ Un conteneur traverse différents **états** : created, running, stopped, paused, exited, removed

✅ Plusieurs conteneurs peuvent être créés à partir d'une **seule image**, fonctionnant indépendamment

✅ La philosophie Docker : **un conteneur = un processus principal = une responsabilité**

## Pour aller plus loin

Dans le prochain chapitre (3.3), nous découvrirons les **registres Docker**, ces entrepôts où sont stockées et partagées les images, avec un focus particulier sur Docker Hub.

Ensuite, au chapitre 4, nous mettrons tout cela en pratique avec les **commandes essentielles** pour créer, gérer, et interagir avec vos conteneurs !

---

*Maintenant que vous comprenez la différence entre images et conteneurs, vous êtes prêt à découvrir où trouver ces images et comment les partager !*

⏭️ [Registres Docker (Docker Hub)](/03-concepts-fondamentaux/03-registres-docker.md)
