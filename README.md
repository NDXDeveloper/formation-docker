# 🐳 Formation Docker - Débutant à Intermédiaire

![License](https://img.shields.io/badge/License-CC%20BY%204.0-blue.svg)
![Docker Version](https://img.shields.io/badge/Docker-27.x-blue.svg)
![Modules](https://img.shields.io/badge/Chapitres-12%2F12-green.svg)
![Language](https://img.shields.io/badge/Langue-Français-blue.svg)
![Platform](https://img.shields.io/badge/Plateforme-Windows%20%7C%20Ubuntu-lightgrey.svg)

**Un guide progressif pour maîtriser Docker, de la découverte à l'utilisation intermédiaire.**

![Docker Logo](https://www.docker.com/wp-content/uploads/2022/03/horizontal-logo-monochromatic-white.png)

---

## 📖 Sommaire

- [À propos](#-à-propos)
- [Contenu de la formation](#-contenu-de-la-formation)
- [Prérequis](#-prérequis)
- [Démarrage rapide](#-démarrage-rapide)
- [Structure du projet](#-structure-du-projet)
- [Utilisation](#-comment-utiliser-cette-formation)
- [Parcours suggéré](#️-parcours-suggéré)
- [Licence](#-licence)
- [Contact](#-contact)

---

## 📋 À propos

Formation sur Docker couvrant les aspects essentiels pour passer de débutant à un niveau intermédiaire en conteneurisation. Compatible Windows et Ubuntu.

**✨ Points clés :**
- 📚 **12 chapitres progressifs** du niveau débutant à intermédiaire
- 🐳 **Théorie sans exercices** - concepts expliqués clairement
- 🔧 **Docker Engine & Docker Desktop** couverts
- 🌐 **Compatible Windows et Ubuntu** - instructions pour les deux plateformes
- 💾 **Gestion des données** - volumes, bind mounts et persistance
- 🔗 **Réseaux et Docker Compose** - orchestration multi-conteneurs
- 🇫🇷 **En français** et gratuit (CC BY 4.0)

**Durée estimée :** 8-12 heures • **Niveau :** Débutant à Intermédiaire

---

## 📚 Contenu de la formation

### Chapitres

1. **Introduction à Docker** - Conteneurisation, architecture, avantages
2. **Installation et configuration** - Setup Windows/Ubuntu, vérification
3. **Concepts fondamentaux** - Images, conteneurs, registres, cycle de vie
4. **Commandes Docker essentielles** - Pull, run, ps, logs, exec
5. **Création d'images personnalisées** - Dockerfile, CMD vs ENTRYPOINT, optimisation
6. **Gestion des données** - Volumes nommés/anonymes, bind mounts, persistance
7. **Réseaux Docker** - Bridge, host, overlay, communication inter-conteneurs
8. **Docker Compose** - Orchestration multi-services, fichier compose.yaml
9. **Gestion des registres** - Docker Hub, tags, registres privés
10. **Bonnes pratiques** - Sécurité, optimisation, secrets, logging
11. **Sujets intermédiaires** - Multi-stage builds, resources, health checks, VS Code Dev Containers
12. **Conclusion et perspectives** - Récapitulatif, Kubernetes, Docker Swarm

📄 **[Voir la table des matières détaillée](SOMMAIRE.md)**

---

## ✅ Prérequis

**Connaissances requises :**
- Utilisation de base du terminal/ligne de commande
- Notions de développement logiciel (optionnel mais utile)

**Matériel requis :**
- Ordinateur sous Windows 10/11 ou Ubuntu 20.04+
- 4 Go de RAM minimum (8 Go recommandés)
- 20 Go d'espace disque disponible
- Connexion Internet

---

## 🚀 Démarrage rapide

### Installation de Docker

#### 🪟 Windows
```powershell
# Télécharger Docker Desktop
# https://www.docker.com/products/docker-desktop

# Vérifier l'installation
docker --version  
docker compose version  
```

#### 🐧 Ubuntu
```bash
# Mettre à jour les paquets
sudo apt update

# Installer Docker
sudo apt install docker.io -y

# Démarrer Docker
sudo systemctl start docker  
sudo systemctl enable docker  

# Vérifier l'installation
docker --version

# Ajouter votre utilisateur au groupe docker
sudo usermod -aG docker $USER
# Se déconnecter et se reconnecter pour appliquer
```

### Test de l'installation

```bash
# Exécuter le conteneur de test
docker run hello-world
```

### Cloner cette formation

```bash
git clone https://github.com/NDXDeveloper/formation-docker.git  
cd formation-docker  
```

---

## 📁 Structure du projet

```
formation-docker/
├── README.md
├── SOMMAIRE.md
├── LICENSE
├── 01-introduction-a-docker/
│   ├── README.md
│   └── [fichiers .md]
├── 02-installation-et-configuration/
├── 03-concepts-fondamentaux/
├── 04-commandes-docker-essentielles/
├── 05-creation-dimages-personnalisees/
├── 06-gestion-des-donnees/
├── 07-reseaux-docker/
├── 08-docker-compose/
├── 09-gestion-des-registres/
├── 10-bonnes-pratiques/
├── 11-sujets-intermediaires/
└── 12-conclusion-et-perspectives/
```

---

## 🎯 Comment utiliser cette formation

### Débutant complet
👉 Commencez par le [Chapitre 1 : Introduction](01-introduction-a-docker/) et suivez l'ordre séquentiel

### Utilisateur avec bases Linux
👉 Parcourez rapidement les chapitres 1-2, puis approfondissez à partir du [Chapitre 3](03-concepts-fondamentaux/)

### Développeur cherchant des best practices
👉 Consultez directement le [Chapitre 10 : Bonnes pratiques](10-bonnes-pratiques/)

### Besoin d'une référence rapide
👉 Consultez le [SOMMAIRE.md](SOMMAIRE.md) pour naviguer rapidement

**💡 Conseil :** Testez les commandes dans un environnement isolé. Docker est parfait pour expérimenter sans risque !

---

## 🗓️ Parcours suggéré

| Niveau | Chapitres | Durée | Objectif |
|--------|-----------|-------|----------|
| 🌱 **Débutant** | 1-4 | 2-3h | Comprendre Docker et les commandes de base |
| 🌿 **Utilisateur** | 5-8 | 3-4h | Créer des images et orchestrer avec Compose |
| 🌳 **Intermédiaire** | 9-12 | 3-5h | Maîtriser les bonnes pratiques et sujets avancés |

**🎯 Approche recommandée :** 30-45 minutes par jour sur 2-3 semaines

---

## 🔑 Concepts clés abordés

- ✅ Différence entre **conteneurs** et **machines virtuelles**
- ✅ Distinction cruciale entre **CMD** et **ENTRYPOINT**
- ✅ Volumes **nommés** vs **anonymes**
- ✅ **Multi-stage builds** pour optimiser les images
- ✅ Utilisation de **Docker Compose** pour orchestrer des services
- ✅ Intégration avec **VS Code Dev Containers**
- ✅ Bonnes pratiques de **sécurité** et **optimisation**

---

## ❓ Questions fréquentes

**Q : Puis-je suivre cette formation sur macOS ?**
R : La formation cible Windows et Ubuntu, mais les concepts s'appliquent à macOS avec Docker Desktop.

**Q : Y a-t-il des exercices pratiques ?**
R : Cette formation est axée sur la théorie et les concepts. Vous pouvez créer vos propres exercices en testant les commandes présentées.

**Q : Quelle est la différence avec Docker Desktop et Docker Engine ?**
R : Cela est expliqué en détail dans le [Chapitre 2.4](02-installation-et-configuration/04-docker-desktop-vs-docker-engine.md).

**Q : Dois-je connaître Kubernetes pour cette formation ?**
R : Non, Kubernetes est brièvement introduit en fin de formation comme perspective d'évolution.

---

## 📝 Licence

Ce projet est sous licence **CC BY 4.0** (Creative Commons Attribution 4.0 International).

✅ Libre d'utiliser, modifier, partager (même commercialement) avec attribution.

**Attribution requise :**
```
Formation Docker par Nicolas DEOUX  
https://github.com/NDXDeveloper/formation-docker  
Licence CC BY 4.0  
```

📄 Voir le fichier [LICENSE](LICENSE) pour les détails complets.

---

## 👨‍💻 Contact

**Nicolas DEOUX**
- 📧 [NDXDev@gmail.com](mailto:NDXDev@gmail.com)
- 💼 [LinkedIn](https://www.linkedin.com/in/nicolas-deoux-ab295980/)
- 🐙 [GitHub](https://github.com/NDXDeveloper)

---

## 🙏 Remerciements

Merci à la communauté Docker, aux contributeurs open source, et à tous ceux qui partagent leurs connaissances sur la conteneurisation ! 🐳

**Ressources inspirantes :**
[Documentation Docker](https://docs.docker.com/) • [Docker Hub](https://hub.docker.com/) • [Play with Docker](https://labs.play-with-docker.com/)

---

<div align="center">

**🐳 Bon apprentissage avec Docker ! 🐳**

[![Star on GitHub](https://img.shields.io/github/stars/NDXDeveloper/formation-docker?style=social)](https://github.com/NDXDeveloper/formation-docker)
[![Follow](https://img.shields.io/github/followers/NDXDeveloper?style=social)](https://github.com/NDXDeveloper)

**[⬆ Retour en haut](#-formation-docker---débutant-à-intermédiaire)**

*Dernière mise à jour : Mars 2026*

</div>
