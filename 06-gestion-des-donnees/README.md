🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 6. Gestion des données

## Introduction au chapitre

Félicitations ! Vous avez maintenant une solide compréhension des concepts fondamentaux de Docker : vous savez créer des conteneurs, construire des images personnalisées, et les manipuler avec les commandes essentielles.

Mais il manque encore un élément crucial pour utiliser Docker dans des situations réelles : **la gestion des données**.

## Pourquoi ce chapitre est-il crucial ?

Imaginez que vous développez une application web avec Docker. Vous avez :
- Un conteneur pour votre application (Node.js, Python, PHP...)
- Un conteneur pour votre base de données (PostgreSQL, MySQL, MongoDB...)
- Un conteneur pour votre serveur web (Nginx, Apache...)

Tout fonctionne parfaitement... jusqu'au jour où vous devez :
- Mettre à jour votre application
- Redémarrer un conteneur qui a planté
- Migrer votre infrastructure vers un autre serveur
- Sauvegarder vos données avant une maintenance

**Et là, catastrophe** : toutes vos données ont disparu !

Les inscriptions de vos utilisateurs, les articles de votre blog, les photos téléchargées, les fichiers de configuration personnalisés... tout est perdu.

Ce scénario cauchemardesque est malheureusement la réalité si vous ne comprenez pas comment Docker gère les données.

## Le défi de la persistance dans Docker

Docker a été conçu avec un principe fondamental : **l'immutabilité et l'éphémère**. Les conteneurs sont pensés pour être :
- Lancés rapidement
- Arrêtés sans conséquence
- Supprimés et recréés facilement
- Remplaçables à tout moment

Cette philosophie apporte énormément de flexibilité et de simplicité dans la gestion des applications. Cependant, elle entre en conflit direct avec un besoin essentiel : **conserver les données dans le temps**.

### Le paradoxe Docker

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│    Conteneurs = ÉPHÉMÈRES     ⚡️     Données = DURABLES │
│                                                         │
│    Comment réconcilier ces deux exigences ?             │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

Docker doit donc répondre à ce paradoxe : comment offrir des conteneurs jetables et remplaçables tout en préservant les données critiques ?

## Ce que vous allez apprendre

Ce chapitre va transformer votre compréhension de Docker en vous montrant comment maîtriser la gestion des données. Vous allez découvrir :

### 6.1 Comprendre la persistance des données
Vous apprendrez pourquoi les données disparaissent par défaut dans Docker et comment ce système a été conçu. Vous comprendrez les concepts fondamentaux qui sous-tendent toute la gestion des données dans Docker.

### 6.2 Les volumes Docker
Vous découvrirez la méthode **recommandée** par Docker pour persister vos données. Les volumes sont puissants, simples à utiliser, et gérés automatiquement par Docker. Vous apprendrez à créer, utiliser et gérer des volumes nommés et anonymes.

### 6.3 Les bind mounts
Vous explorerez une méthode alternative qui vous donne un contrôle direct sur où sont stockées vos données sur votre machine hôte. Particulièrement utile en développement, les bind mounts vous permettent de modifier vos fichiers en temps réel.

### 6.4 Comparaison et choix entre volumes et bind mounts
Enfin, vous saurez **quand utiliser quoi**. Chaque méthode a ses avantages et ses cas d'usage spécifiques. Vous apprendrez à faire le bon choix selon votre situation.

## Les questions auxquelles vous saurez répondre

À la fin de ce chapitre, vous serez capable de répondre à ces questions essentielles :

✅ Pourquoi mes données disparaissent quand je supprime un conteneur ?  
✅ Comment sauvegarder ma base de données dans Docker ?  
✅ Comment partager des fichiers entre mon ordinateur et un conteneur ?  
✅ Comment plusieurs conteneurs peuvent-ils accéder aux mêmes données ?  
✅ Où Docker stocke-t-il réellement mes données ?  
✅ Quelle méthode choisir pour mon projet : volumes ou bind mounts ?  
✅ Comment éviter de perdre des données lors d'une mise à jour ?

## Un concept, plusieurs solutions

Docker vous offre plusieurs outils pour gérer les données, chacun adapté à des besoins différents :

| Méthode | Géré par | Cas d'usage principal | Niveau |
|---------|----------|----------------------|--------|
| **Volumes** | Docker | Production, bases de données | ⭐⭐⭐ Recommandé |
| **Bind mounts** | Vous | Développement, code source | ⭐⭐ |
| **tmpfs** | Mémoire RAM | Données temporaires, cache | ⭐ |

> 💡 **Note :** Dans ce chapitre, nous nous concentrerons principalement sur les volumes et les bind mounts, car ce sont les deux méthodes les plus utilisées au quotidien.

## Un exemple concret pour commencer

Pour illustrer l'importance de ce chapitre, prenons un exemple simple :

```bash
# Vous lancez un conteneur PostgreSQL
docker run -d --name ma-db postgres

# Vous créez une base de données et des tables
# Vous ajoutez des données importantes
# Tout fonctionne parfaitement !

# Puis vous supprimez le conteneur (erreur de manipulation, mise à jour...)
docker rm -f ma-db

# Vous relancez un nouveau conteneur
docker run -d --name ma-db postgres

# 🚨 PROBLÈME : Toutes vos données ont disparu !
```

Après ce chapitre, vous saurez comment éviter ce scénario et garantir que vos données survivent, quoi qu'il arrive à vos conteneurs.

## L'état d'esprit à adopter

Avant de plonger dans les détails techniques, adoptez cet état d'esprit fondamental :

> **Les conteneurs sont temporaires, les données sont permanentes.**
>
> Ne stockez jamais de données critiques directement dans un conteneur. Utilisez toujours un mécanisme de persistance externe.

Cette règle d'or vous évitera 99% des problèmes liés à la perte de données dans Docker.

## Prêt à commencer ?

La gestion des données peut sembler intimidante au début, mais une fois que vous aurez compris les concepts de base, elle deviendra une seconde nature. Docker a fait en sorte que persister des données soit simple et élégant.

Dans la section suivante (6.1), nous allons plonger dans le cœur du sujet en explorant pourquoi et comment la persistance des données fonctionne dans Docker.

Allons-y ! 🚀

⏭️ [Comprendre la persistance des données](/06-gestion-des-donnees/01-comprendre-la-persistance-des-donnees.md)
