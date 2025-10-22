🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 3. Concepts fondamentaux

## Bienvenue dans le cœur de Docker !

Maintenant que vous avez installé Docker et compris ce qu'est la conteneurisation, il est temps de plonger dans les **concepts fondamentaux** qui constituent la base de tout ce que vous ferez avec Docker.

Ce chapitre est probablement **le plus important** de toute la formation. En maîtrisant ces quatre concepts essentiels, vous aurez les clés pour comprendre et utiliser Docker efficacement dans n'importe quelle situation.

## Pourquoi ces concepts sont-ils si importants ?

Imaginez que vous apprenez à conduire une voiture. Avant de prendre la route, vous devez comprendre :
- Ce qu'est le moteur (comment la voiture avance)
- Comment fonctionne le volant (comment diriger)
- À quoi servent les pédales (comment contrôler la vitesse)
- Où se trouve le réservoir (comment faire le plein)

Avec Docker, c'est pareil ! Avant de créer vos propres applications conteneurisées, vous devez maîtriser les concepts de base qui font fonctionner l'écosystème Docker.

## Les 4 piliers de Docker

Dans ce chapitre, nous allons explorer les **quatre concepts fondamentaux** qui forment le socle de Docker :

### 🖼️ Les Images Docker
Le **modèle** ou le **plan de construction** de votre application. C'est le package immuable qui contient tout ce dont votre application a besoin pour fonctionner.

**Analogie** : Une image, c'est comme le disque d'installation d'un logiciel ou une recette de cuisine.

### 📦 Les Conteneurs Docker
L'**instance en cours d'exécution** créée à partir d'une image. C'est votre application qui tourne réellement, isolée dans son propre environnement.

**Analogie** : Un conteneur, c'est comme un logiciel ouvert et en cours d'utilisation, ou un plat préparé à partir d'une recette.

### 🏪 Les Registres Docker
L'**entrepôt** où sont stockées et partagées les images Docker. Le plus connu est Docker Hub, mais il en existe d'autres (publics ou privés).

**Analogie** : Un registre, c'est comme un App Store ou une bibliothèque où vous pouvez télécharger des images toutes prêtes.

### 🔄 Le Cycle de vie d'un conteneur
Le **processus complet** qu'un conteneur traverse depuis sa création jusqu'à sa suppression, en passant par ses différents états (démarré, arrêté, en pause...).

**Analogie** : Le cycle de vie, c'est comme les étapes par lesquelles passe un programme : lancement, exécution, mise en pause, fermeture.

## La relation entre ces concepts

Pour bien comprendre Docker, il est essentiel de saisir comment ces concepts s'articulent entre eux :

```
┌─────────────────┐
│  REGISTRE       │  (Docker Hub)
│  (Entrepôt)     │
└────────┬────────┘
         │ ① téléchargement (pull)
         ↓
┌─────────────────┐
│  IMAGE          │  (Modèle statique)
│  (Template)     │
└────────┬────────┘
         │ ② création (run)
         ↓
┌─────────────────┐
│  CONTENEUR      │  (Instance active)
│  (Exécution)    │
└────────┬────────┘
         │ ③ cycle de vie
         ↓
    [Start → Stop → Restart → Remove]
```

### Le workflow typique

Voici le parcours classique lorsque vous travaillez avec Docker :

1. **Vous cherchez** une image sur Docker Hub (par exemple, une image nginx pour un serveur web)
2. **Vous téléchargez** cette image depuis le registre vers votre machine locale
3. **Vous créez** un ou plusieurs conteneurs à partir de cette image
4. **Vous gérez** le cycle de vie de ces conteneurs (démarrage, arrêt, redémarrage)
5. **Vous interagissez** avec vos conteneurs en cours d'exécution
6. **Vous nettoyez** en supprimant les conteneurs et images dont vous n'avez plus besoin

## Ce que vous allez apprendre

À la fin de ce chapitre, vous serez capable de :

✅ **Expliquer clairement** la différence entre une image et un conteneur

✅ **Comprendre** comment les images sont structurées en couches

✅ **Identifier** d'où viennent les images et comment les trouver

✅ **Reconnaître** les différents états d'un conteneur

✅ **Visualiser mentalement** le flux de données entre registres, images et conteneurs

✅ **Utiliser** le vocabulaire technique de Docker avec confiance

## L'importance de la distinction Image vs Conteneur

La confusion la plus fréquente chez les débutants est de mélanger **images** et **conteneurs**. C'est normal, car les deux concepts sont intimement liés, mais ils sont fondamentalement différents.

Retenez cette règle simple :

> **Une IMAGE est un modèle figé.**
> **Un CONTENEUR est une instance vivante créée depuis ce modèle.**

Pensez à la programmation orientée objet :
- **Image** = La CLASSE (le modèle)
- **Conteneur** = L'OBJET ou l'INSTANCE (une occurrence concrète)

Ou encore :
- **Image** = Le fichier .exe d'un programme
- **Conteneur** = Le programme en cours d'exécution dans votre gestionnaire de tâches

Cette distinction sera reprise et approfondie dans chacune des sections suivantes.

## Une approche progressive

Nous allons aborder ces concepts dans un ordre logique :

1. D'abord les **images** (le point de départ : qu'est-ce qu'on va exécuter ?)
2. Ensuite les **conteneurs** (comment donner vie aux images ?)
3. Puis les **registres** (où trouver ou stocker nos images ?)
4. Enfin le **cycle de vie** (comment gérer nos conteneurs au quotidien ?)

Chaque section s'appuie sur la précédente, construisant progressivement votre compréhension de l'écosystème Docker.

## Conseils pour tirer le meilleur parti de ce chapitre

💡 **Prenez votre temps** : Ces concepts peuvent sembler abstraits au début. N'hésitez pas à relire et à réfléchir aux analogies proposées.

💡 **Visualisez mentalement** : Essayez de vous représenter les images comme des boîtes fermées et les conteneurs comme ces boîtes ouvertes et actives.

💡 **Posez-vous des questions** : À chaque section, demandez-vous "Quelle est la différence avec le concept précédent ?"

💡 **Ne sautez pas d'étapes** : Même si vous êtes pressé d'arriver aux commandes pratiques, ces fondations sont indispensables.

💡 **Revenez-y si besoin** : Ce chapitre servira de référence. N'hésitez pas à y revenir lorsque vous aurez un doute sur un concept.

## Un dernier mot avant de commencer

Docker peut sembler intimidant au début, mais une fois que vous aurez assimilé ces concepts fondamentaux, tout le reste coulera naturellement. Ces notions sont le socle sur lequel repose toute l'utilisation de Docker, du développement local au déploiement en production.

Vous allez découvrir un outil puissant qui va transformer votre façon de développer et de déployer des applications. Les développeurs du monde entier utilisent Docker quotidiennement pour résoudre des problèmes complexes de manière élégante et simple.

**Êtes-vous prêt ?** Commençons par le premier concept essentiel : les Images Docker.

---

*Dans la section suivante (3.1), nous explorerons en détail ce que sont les images Docker, comment elles sont structurées, et pourquoi elles sont au cœur de l'écosystème Docker.*

⏭️ [Images Docker](/03-concepts-fondamentaux/01-images-docker.md)
