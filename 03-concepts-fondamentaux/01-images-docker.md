🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 3.1 Images Docker

## Qu'est-ce qu'une image Docker ?

Une image Docker est un **modèle en lecture seule** qui contient tout ce dont vous avez besoin pour exécuter une application. Vous pouvez la comparer à une recette de cuisine ou à un plan de construction.

Imaginez une image Docker comme un **instantané figé** d'un système qui contient :
- Le système d'exploitation de base (une version légère de Linux, par exemple)
- Votre code applicatif
- Les bibliothèques et dépendances nécessaires
- Les fichiers de configuration
- Les commandes à exécuter au démarrage

### Analogie pour mieux comprendre

Pensez à une image Docker comme à un **disque d'installation** ou une **image ISO** d'un système d'exploitation. Tout comme vous pouvez installer Windows à partir d'un DVD sur plusieurs ordinateurs, vous pouvez créer plusieurs conteneurs à partir d'une seule image Docker.

**Image = Le moule**
**Conteneur = L'objet créé à partir du moule**

## Caractéristiques principales des images

### 1. Immutabilité

Une image Docker est **immuable**, ce qui signifie qu'elle ne peut pas être modifiée une fois créée. Si vous voulez apporter des changements, vous devez créer une **nouvelle image**.

Cette immutabilité garantit que :
- Votre application se comportera toujours de la même manière
- Vous pouvez reproduire exactement le même environnement partout
- Les déploiements sont prévisibles et fiables

### 2. Portabilité

Les images Docker sont **portables** : elles fonctionnent de la même manière sur n'importe quel système qui exécute Docker, que ce soit :
- Votre ordinateur portable sous Windows
- Un serveur Linux
- Un Mac
- Le cloud (AWS, Azure, Google Cloud)

C'est ce qui résout le fameux problème "ça marche sur ma machine, mais pas en production !"

### 3. Légèreté

Contrairement aux machines virtuelles qui contiennent un système d'exploitation complet, les images Docker sont **légères** car elles partagent le noyau du système hôte et ne contiennent que le strict nécessaire.

## Architecture en couches (Layers)

L'un des concepts les plus importants à comprendre est que les images Docker sont construites en **couches superposées** (layers).

### Comment ça fonctionne ?

Chaque instruction dans un fichier de construction (Dockerfile) crée une nouvelle couche :

```
┌─────────────────────────────┐
│  Votre application          │  ← Couche 4
├─────────────────────────────┤
│  Dépendances (Node.js, etc) │  ← Couche 3
├─────────────────────────────┤
│  Outils système             │  ← Couche 2
├─────────────────────────────┤
│  Système de base (Ubuntu)   │  ← Couche 1
└─────────────────────────────┘
```

### Avantages du système de couches

1. **Réutilisation** : Si plusieurs images utilisent la même couche de base (par exemple Ubuntu), Docker ne la télécharge qu'une seule fois
2. **Efficacité** : Seules les couches modifiées sont re-téléchargées ou reconstruites
3. **Rapidité** : Le partage de couches accélère considérablement les téléchargements et les constructions

**Exemple concret :**
Si vous avez 10 applications Node.js, toutes partageront les mêmes couches de base (Ubuntu + Node.js), et seule votre application sera différente dans chaque image.

## D'où viennent les images Docker ?

### 1. Docker Hub

**Docker Hub** est le registre public officiel qui héberge des milliers d'images prêtes à l'emploi. C'est comme un "App Store" pour Docker.

Vous y trouverez des images officielles pour :
- Des langages de programmation (Python, Node.js, Java, PHP...)
- Des bases de données (MySQL, PostgreSQL, MongoDB...)
- Des serveurs web (Nginx, Apache...)
- Des systèmes d'exploitation (Ubuntu, Alpine Linux...)

### 2. Images officielles vs images communautaires

**Images officielles** :
- Maintenues par Docker Inc. ou par l'éditeur du logiciel
- Vérifiées et sécurisées
- Mises à jour régulièrement
- Identifiables par un badge "Official Image" sur Docker Hub
- Exemple : `nginx`, `postgres`, `ubuntu`

**Images communautaires** :
- Créées par des utilisateurs
- Préfixées par le nom d'utilisateur : `utilisateur/nom-image`
- À utiliser avec précaution (vérifier la popularité, les téléchargements, les mises à jour)
- Exemple : `johndoe/mon-application`

### 3. Créer vos propres images

Vous pouvez également créer vos propres images personnalisées pour vos applications spécifiques (nous verrons cela en détail dans le chapitre 5).

## Nomenclature et tags des images

Les images Docker suivent une convention de nommage précise :

```
[registry/][utilisateur/]nom:tag
```

### Exemples expliqués

- `nginx` → Image officielle de Nginx, tag "latest" par défaut
- `nginx:1.24` → Image Nginx avec la version spécifique 1.24
- `ubuntu:22.04` → Image Ubuntu version 22.04
- `node:18-alpine` → Image Node.js version 18 basée sur Alpine Linux (version ultra-légère)
- `monnom/mon-app:v1.0` → Image personnelle, version 1.0

### Le tag "latest"

Le tag `latest` est le tag par défaut, mais **attention** : il ne signifie pas toujours "la plus récente". C'est simplement un tag comme un autre qui est appliqué par convention à une version de l'image.

**Bonne pratique** : En production, utilisez toujours des **tags de version spécifiques** plutôt que `latest` pour garantir la stabilité.

## Identifiant unique des images

Chaque image possède un **Image ID** : une empreinte numérique unique (hash SHA256) qui l'identifie de manière certaine.

Exemple : `sha256:abc123def456...`

Cet identifiant garantit l'intégrité de l'image : si un seul bit change, l'ID sera différent.

## Stockage des images

Sur votre machine, les images Docker sont stockées localement par le Docker Engine. Vous n'avez généralement pas besoin de savoir où exactement, car Docker gère cela pour vous.

Quand vous téléchargez une image :
1. Docker vérifie si elle existe déjà localement
2. Si non, il la télécharge depuis le registre (Docker Hub par défaut)
3. L'image est stockée dans le cache local de Docker
4. Vous pouvez maintenant créer des conteneurs à partir de cette image instantanément

## Taille des images

La taille des images est un critère important :

- **Images complètes** : Basées sur Ubuntu, Debian (100-500 MB ou plus)
- **Images slim** : Versions allégées (~50-150 MB)
- **Images Alpine** : Ultra-légères, basées sur Alpine Linux (5-50 MB)

**Exemple :**
- `python:3.11` → environ 920 MB
- `python:3.11-slim` → environ 125 MB
- `python:3.11-alpine` → environ 50 MB

### Quel type choisir ?

- **Images complètes** : Pour le développement, débogage facile
- **Images slim** : Bon compromis pour la production
- **Images Alpine** : Production optimisée, mais peut nécessiter des ajustements

## Images de base communes

Voici quelques images de base fréquemment utilisées :

| Image | Description | Cas d'usage |
|-------|-------------|-------------|
| `alpine` | Distribution Linux ultra-légère (5 MB) | Base pour des images optimisées |
| `ubuntu` | Distribution Linux populaire | Développement, familiarité |
| `debian` | Distribution Linux stable | Applications nécessitant stabilité |
| `scratch` | Image vide (0 MB) | Exécutables statiques, maximum de légèreté |

## Cycle de vie d'une image

```
1. CRÉATION
   (via Dockerfile ou commit)
          ↓
2. STOCKAGE LOCAL
   (dans le cache Docker)
          ↓
3. PUBLICATION (optionnelle)
   (vers Docker Hub ou registre privé)
          ↓
4. TÉLÉCHARGEMENT
   (pull depuis un registre)
          ↓
5. UTILISATION
   (création de conteneurs)
          ↓
6. SUPPRESSION
   (nettoyage des images inutilisées)
```

## Comprendre la différence : Image vs Conteneur

C'est LA distinction fondamentale à bien comprendre :

| Aspect | Image | Conteneur |
|--------|-------|-----------|
| Nature | Modèle statique, template | Instance en cours d'exécution |
| État | Immuable, en lecture seule | Modifiable, possède un état |
| Nombre | Une image peut exister en un seul exemplaire | Une image peut créer des milliers de conteneurs |
| Analogie | La classe en programmation | L'objet/instance en programmation |
| Stockage | Stockée sur disque | S'exécute en mémoire |

**Analogie simple :**
- **Image** = Programme installé (comme Word ou Chrome sur votre disque)
- **Conteneur** = Programme en cours d'exécution (la fenêtre Word ouverte)

Vous pouvez ouvrir plusieurs fenêtres Word (conteneurs) à partir du même programme installé (image).

## Résumé des points clés

✅ Une image Docker est un modèle immuable contenant tout le nécessaire pour exécuter une application

✅ Les images sont construites en couches superposées pour optimiser le stockage et les transferts

✅ Docker Hub est le registre public principal où trouver des milliers d'images prêtes à l'emploi

✅ Privilégiez les images officielles et utilisez des tags de version spécifiques en production

✅ Une image sert de base pour créer un ou plusieurs conteneurs

✅ Les images sont légères comparées aux machines virtuelles et garantissent la portabilité

## Pour aller plus loin

Dans le prochain chapitre (3.2), nous verrons comment **utiliser ces images pour créer et gérer des conteneurs Docker**, c'est-à-dire comment donner vie à ces modèles statiques pour exécuter réellement vos applications.

Vous apprendrez également les commandes essentielles pour :
- Rechercher des images
- Télécharger des images
- Lister vos images locales
- Inspecter les détails d'une image
- Supprimer des images

---

*Maintenant que vous comprenez ce qu'est une image Docker, vous êtes prêt à passer à l'étape suivante : les conteneurs !*

⏭️ [Conteneurs Docker](/03-concepts-fondamentaux/02-conteneurs-docker.md)
