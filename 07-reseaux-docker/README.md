🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 7. Réseaux Docker

## Introduction au chapitre

Bravo pour avoir complété le chapitre sur la gestion des données ! Vous savez maintenant créer des conteneurs, construire des images, et persister vos données de manière professionnelle. Mais il reste une pièce essentielle du puzzle Docker : **les réseaux**.

Jusqu'à présent, vous avez principalement travaillé avec des conteneurs isolés. Dans le monde réel, les applications modernes sont rarement composées d'un seul conteneur. Elles sont constituées de **multiples services** qui doivent **communiquer entre eux** :

- Une application web qui se connecte à une base de données
- Un frontend qui appelle une API backend
- Des microservices qui échangent des données
- Un serveur de cache qui accélère les performances
- Des workers qui traitent des tâches en arrière-plan

**Comment tous ces conteneurs peuvent-ils se parler ?** C'est précisément ce que vous allez découvrir dans ce chapitre.

## Pourquoi les réseaux sont-ils cruciaux ?

### Le problème de l'isolation

Docker, par sa nature, **isole** chaque conteneur. C'est excellent pour la sécurité et la stabilité, mais cela crée un défi : comment faire communiquer des conteneurs qui sont, par design, isolés les uns des autres ?

Imaginez ces scénarios courants :

**Scénario 1 : Application web classique**
```
┌──────────┐      ?      ┌──────────┐      ?      ┌──────────┐
│  Nginx   │────────────►│   API    │────────────►│   DB     │
│(Frontend)│             │(Backend) │             │(Postgres)│
└──────────┘             └──────────┘             └──────────┘
```
Comment le frontend trouve-t-il l'API ? Comment l'API se connecte-t-elle à la base de données ?

**Scénario 2 : Microservices**
```
                    ┌──────────────┐
                    │ API Gateway  │
                    └───────┬──────┘
              ┌─────────────┼─────────────┐
              │             │             │
        ┌─────▼─────┐ ┌─────▼─────┐ ┌─────▼─────┐
        │  Users    │ │  Orders   │ │  Payment  │
        │  Service  │ │  Service  │ │  Service  │
        └───────────┘ └───────────┘ └───────────┘
```
Comment l'API Gateway route-t-elle les requêtes vers les bons services ?

**Scénario 3 : Accès depuis votre navigateur**
```
    Votre ordinateur
           │
           │ Comment accéder ?
           ↓
    ┌──────────────┐
    │  Conteneur   │
    │  Nginx:80    │
    └──────────────┘
```
Comment accédez-vous à votre application conteneurisée depuis `localhost` ?

Sans une compréhension solide des réseaux Docker, ces questions restent des mystères frustrants.

### Ce que permettent les réseaux Docker

Les réseaux Docker résolvent tous ces défis en offrant :

✅ **Communication entre conteneurs** - Vos services peuvent se parler facilement  
✅ **Isolation et sécurité** - Contrôlez qui peut communiquer avec qui  
✅ **Découverte de services** - Trouvez automatiquement les autres conteneurs  
✅ **Accès depuis l'extérieur** - Exposez vos applications au monde  
✅ **Architectures complexes** - Construisez des systèmes multi-services robustes

## Les concepts que vous allez maîtriser

Ce chapitre est structuré de manière progressive, du plus simple au plus avancé :

### 7.1 Principes des réseaux dans Docker

Vous commencerez par comprendre les **fondamentaux** :
- Comment Docker gère l'isolation réseau
- Les différents niveaux de communication (conteneur↔conteneur, conteneur↔hôte, conteneur↔Internet)
- La résolution DNS automatique
- Le mappage de ports pour exposer vos services

**Ce que vous saurez faire :**
- Comprendre pourquoi les conteneurs ne peuvent pas communiquer par défaut
- Faire dialoguer deux conteneurs simplement
- Accéder à vos applications depuis votre navigateur

### 7.2 Types de réseaux (bridge, host, none, overlay)

Vous explorerez les **quatre types de réseaux** que Docker propose :
- **Bridge** : Le réseau standard pour vos applications (le plus utilisé)
- **Host** : Partage direct du réseau de la machine hôte (performance maximale)
- **None** : Isolation complète (sécurité maximale)
- **Overlay** : Communication entre plusieurs machines (clusters)

**Ce que vous saurez faire :**
- Choisir le bon type de réseau selon votre besoin
- Comprendre les avantages et limites de chaque type
- Optimiser les performances réseau de vos applications

### 7.3 Créer et gérer des réseaux personnalisés

Vous apprendrez à **créer vos propres réseaux** :
- Créer des réseaux isolés pour chaque projet
- Configurer des sous-réseaux et des adresses IP
- Connecter et déconnecter des conteneurs dynamiquement
- Organiser votre infrastructure réseau

**Ce que vous saurez faire :**
- Créer des architectures réseau professionnelles
- Isoler différents environnements (dev, staging, prod)
- Gérer plusieurs projets Docker sur la même machine
- Déboguer les problèmes de réseau

### 7.4 Communication entre conteneurs

Enfin, vous maîtriserez la **communication pratique** :
- Les meilleures pratiques de communication (DNS, variables d'environnement)
- Les patterns d'architecture courants (3-tier, microservices, load balancing)
- La sécurité réseau et l'isolation
- Le debugging et le dépannage

**Ce que vous saurez faire :**
- Construire des applications multi-conteneurs robustes
- Implémenter des architectures de microservices
- Sécuriser vos communications
- Résoudre les problèmes de connectivité

## Un exemple motivant : Du simple au complexe

Pour illustrer l'évolution de vos compétences, voici comment vous progresserez :

### Niveau débutant : Conteneur unique
```bash
docker run -d -p 80:80 nginx
# Accessible sur http://localhost:80
```
Simple, mais limité à un seul service.

### Niveau intermédiaire : Deux conteneurs qui communiquent
```bash
docker network create app-network  
docker run -d --name db --network app-network postgres  
docker run -d --name api --network app-network -p 3000:3000 mon-api  
# L'API peut maintenant se connecter à "db"
```
Vos services peuvent dialoguer !

### Niveau avancé : Architecture multi-tiers complète
```bash
docker network create frontend-net  
docker network create backend-net  
docker network create database-net  

# Base de données isolée
docker run -d --name postgres --network database-net postgres

# API avec accès frontend et database
docker run -d --name api --network backend-net mon-api  
docker network connect database-net api  
docker network connect frontend-net api  

# Frontend exposé publiquement
docker run -d --name web --network frontend-net -p 80:80 nginx
```
Architecture professionnelle avec isolation et sécurité !

## Ce que vous construirez

À la fin de ce chapitre, vous serez capable de construire des architectures comme celle-ci :

```
┌─────────────────────────────────────────────────────────────┐
│                     INTERNET                                │
└───────────────────────────┬─────────────────────────────────┘
                            │ Port 80
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                  Réseau Public                              │
│                                                             │
│                    ┌──────────────┐                         │
│                    │  API Gateway │                         │
│                    │   (Nginx)    │                         │
│                    └───────┬──────┘                         │
│                            │                                │
└────────────────────────────┼────────────────────────────────┘
                             │
┌────────────────────────────┼────────────────────────────────┐
│                  Réseau Backend                             │
│                            │                                │
│         ┌──────────────────┼──────────────────┐             │
│         │                  │                  │             │
│    ┌────▼────┐       ┌────▼────┐       ┌──────▼────┐        │
│    │ Users   │       │ Orders  │       │ Payment   │        │
│    │ Service │       │ Service │       │ Service   │        │
│    └────┬────┘       └────┬────┘       └─────┬─────┘        │
│         │                 │                  │              │
└─────────┼─────────────────┼──────────────────┼──────────────┘
          │                 │                  │
┌─────────┼─────────────────┼──────────────────┼──────────────┐
│         │      Réseau Database               │              │
│    ┌────▼────┐       ┌────▼────┐       ┌─────▼───┐          │
│    │  User   │       │  Order  │       │ Payment │          │
│    │   DB    │       │   DB    │       │   DB    │          │
│    └─────────┘       └─────────┘       └─────────┘          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

Chaque couche est isolée, chaque service communique uniquement avec ce dont il a besoin, et vous contrôlez tout !

## Les questions auxquelles vous saurez répondre

Après ce chapitre, vous maîtriserez des concepts comme :

✅ Pourquoi mes conteneurs ne peuvent-ils pas se parler ?  
✅ Comment exposer mon application web sur le port 80 ?  
✅ Comment faire communiquer une API avec sa base de données ?  
✅ Quel type de réseau choisir pour mon projet ?  
✅ Comment isoler mon environnement de dev de la production ?  
✅ Comment déboguer une erreur "Connection refused" ?  
✅ Comment sécuriser mes communications entre services ?  
✅ Comment construire une architecture microservices ?

## L'importance des réseaux dans le monde réel

### En développement

Les réseaux Docker simplifient drastiquement le développement :

**Avant Docker :**
- Installer PostgreSQL localement
- Configurer les ports
- Gérer les versions
- Conflits avec d'autres projets
- Configuration différente selon les développeurs

**Avec Docker et les réseaux :**
```bash
docker network create monprojet-dev  
docker run -d --name db --network monprojet-dev postgres  
docker run -d --name api --network monprojet-dev mon-api  
```
Tout fonctionne, isolé, reproductible, et identique pour toute l'équipe !

### En production

Les réseaux Docker sont la fondation d'architectures modernes :

- **Microservices** : Chaque service communique via le réseau
- **Scalabilité** : Ajoutez des instances facilement
- **Sécurité** : Isolez les composants sensibles
- **Monitoring** : Contrôlez et observez le trafic
- **Déploiement** : Zéro downtime avec le bon setup réseau

## Ce que ce chapitre N'est PAS

Pour être transparent, ce chapitre se concentre sur Docker en mode **single-host** (une seule machine). Nous aborderons brièvement les réseaux overlay pour le multi-host, mais l'orchestration avancée (Docker Swarm, Kubernetes) est un sujet distinct qui sera introduit dans le chapitre 12.

Ce chapitre vous donne les **fondations solides** nécessaires pour comprendre ensuite les systèmes d'orchestration plus complexes.

## Prérequis et état d'esprit

### Ce que vous devez déjà savoir

Pour tirer le meilleur parti de ce chapitre, assurez-vous de maîtriser :

- ✅ Les bases de Docker (images, conteneurs)
- ✅ La création d'images avec Dockerfile
- ✅ La gestion des volumes et de la persistance des données
- ✅ Les commandes Docker essentielles

Si vous avez suivi les chapitres précédents, vous êtes prêt !

### L'état d'esprit à adopter

Les réseaux peuvent sembler abstraits au début, mais pensez-y comme à des "ponts" ou des "routes" entre vos conteneurs. Chaque section s'appuie sur la précédente, alors prenez votre temps et expérimentez avec les exemples.

> 💡 **Conseil :** Les réseaux Docker sont un domaine où la **pratique** est essentielle. Essayez chaque commande, créez des réseaux de test, expérimentez. C'est en faisant que vous comprendrez vraiment !

## Outils et ressources

### Outils que vous utiliserez

Vous découvrirez plusieurs outils pour travailler avec les réseaux :

- `docker network` : Commandes de gestion des réseaux
- `docker inspect` : Inspection détaillée des configurations
- `ping`, `curl`, `telnet` : Tests de connectivité
- `docker logs` : Débogage des connexions
- Des conteneurs de debugging comme `nicolaka/netshoot`

### Commandes que vous maîtriserez

```bash
# Gestion des réseaux
docker network ls  
docker network create  
docker network inspect  
docker network connect  
docker network disconnect  
docker network rm  

# Lancement avec réseau
docker run --network  
docker run -p  

# Debugging
docker exec conteneur ping autre-conteneur  
docker logs conteneur  
```

## Structure d'apprentissage

Le chapitre suit une progression logique :

1. **Comprendre** les principes (7.1) → Pourquoi et comment ?
2. **Explorer** les options (7.2) → Quels outils sont disponibles ?
3. **Pratiquer** la création (7.3) → Comment construire ?
4. **Maîtriser** la communication (7.4) → Comment optimiser ?

Chaque section s'appuie sur la précédente, créant une compréhension complète et pratique.

## Un dernier mot avant de commencer

Les réseaux Docker sont l'un des aspects les plus puissants de Docker. Ils transforment des conteneurs isolés en **systèmes distribués cohérents**. Ils sont la clé pour passer de "faire tourner un conteneur" à "construire des architectures complètes".

Ce chapitre va changer votre façon de penser Docker. Vous passerez de conteneurs individuels à des **écosystèmes d'applications** où tout communique harmonieusement.

Les concepts peuvent sembler nouveaux, mais avec de la pratique, ils deviendront une seconde nature. Et une fois maîtrisés, vous vous demanderez comment vous avez pu travailler sans !

## Prêt à plonger ?

Attachez votre ceinture ! Dans la section suivante (7.1), nous allons explorer les **principes fondamentaux** des réseaux Docker. Vous découvrirez comment Docker gère l'isolation, comment les conteneurs peuvent communiquer, et comment vous pouvez accéder à vos applications depuis votre navigateur.

Les réseaux Docker n'auront plus de secrets pour vous. Allons-y ! 🚀

---

**Note :** N'oubliez pas que Docker fournit une excellente documentation officielle sur les réseaux. Ce cours vous donne une base solide et pratique, mais n'hésitez pas à consulter la documentation pour des cas d'usage très spécifiques.

⏭️ [Principes des réseaux dans Docker](/07-reseaux-docker/01-principes-des-reseaux-dans-docker.md)
