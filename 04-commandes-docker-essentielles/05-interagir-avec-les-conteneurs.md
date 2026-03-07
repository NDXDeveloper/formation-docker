🔝 Retour au [Sommaire](/SOMMAIRE.md)

# 4.5 Interagir avec les conteneurs (exec, logs, attach)

## Introduction

Une fois vos conteneurs en cours d'exécution, vous aurez souvent besoin d'**interagir avec eux** : exécuter des commandes à l'intérieur, consulter leurs journaux, ou vous y connecter directement. C'est exactement ce que permettent les commandes de cette section.

Ces outils sont essentiels pour :
- **Déboguer** des problèmes dans un conteneur
- **Consulter les logs** pour comprendre ce qui s'est passé
- **Exécuter des commandes** de maintenance ou de diagnostic
- **Accéder au shell** d'un conteneur pour explorer son environnement
- **Surveiller** l'activité en temps réel

Ces commandes transforment vos conteneurs de "boîtes noires" en environnements accessibles et inspectables.

## La commande `docker exec` : exécuter des commandes dans un conteneur

### Qu'est-ce que `docker exec` ?

`docker exec` (execute) permet d'**exécuter une commande** dans un conteneur **en cours d'exécution**. C'est la commande la plus utilisée pour interagir avec les conteneurs en production.

⚠️ **Important** : Le conteneur doit être en cours d'exécution (`status: running`). Vous ne pouvez pas utiliser `exec` sur un conteneur arrêté.

### Syntaxe de base

```bash
docker exec <nom_ou_id_conteneur> <commande>
```

### Exemples simples

**Lister les fichiers dans un conteneur** :
```bash
docker exec mon-nginx ls -la /usr/share/nginx/html
```

**Afficher la date du conteneur** :
```bash
docker exec mon-nginx date
```

**Vérifier la version d'un logiciel** :
```bash
docker exec mon-conteneur python --version
```

**Afficher les processus en cours** :
```bash
docker exec mon-nginx ps aux
```

Ces commandes s'exécutent, affichent le résultat, puis se terminent immédiatement.

### Mode interactif avec `-it`

Pour ouvrir un **shell interactif** et travailler à l'intérieur du conteneur, utilisez les options `-it` :

```bash
docker exec -it mon-nginx bash
```

Explication des options :
- `-i` (--interactive) : garde l'entrée standard (stdin) ouverte
- `-t` (--tty) : alloue un pseudo-terminal

Vous vous retrouvez alors avec un shell bash à l'intérieur du conteneur :

```bash
root@f4d5e8a9b2c1:/#
```

Vous pouvez maintenant exécuter n'importe quelle commande comme si vous étiez directement dans le conteneur :

```bash
# Naviguer dans les dossiers
cd /usr/share/nginx/html

# Lire un fichier
cat index.html

# Installer un outil (si le conteneur le permet)
apt-get update && apt-get install -y vim

# Quitter le shell
exit
```

💡 **Astuce** : Tapez `exit` ou appuyez sur `Ctrl+D` pour quitter le shell interactif.

### Shell alternatifs

Tous les conteneurs n'ont pas `bash` installé. Essayez `sh` si `bash` n'est pas disponible :

```bash
# Pour les conteneurs basés sur Alpine Linux
docker exec -it mon-conteneur sh
```

Si vous ne savez pas quel shell est disponible :

```bash
# Essayer bash d'abord
docker exec -it mon-conteneur bash

# Si ça échoue, essayer sh
docker exec -it mon-conteneur sh

# En dernier recours
docker exec -it mon-conteneur /bin/sh
```

### Exécuter en tant qu'utilisateur spécifique avec `-u`

Par défaut, les commandes s'exécutent en tant qu'utilisateur root. Pour utiliser un autre utilisateur :

```bash
docker exec -u www-data mon-nginx whoami
```

Sortie :
```
www-data
```

Ceci est important pour les questions de permissions et de sécurité.

### Définir le répertoire de travail avec `-w`

Pour exécuter une commande dans un répertoire spécifique :

```bash
docker exec -w /app mon-conteneur ls -la
```

### Définir des variables d'environnement avec `-e`

```bash
docker exec -e MY_VAR=value mon-conteneur env
```

### Exécuter en arrière-plan avec `-d`

Pour exécuter une commande longue en arrière-plan :

```bash
docker exec -d mon-conteneur /usr/local/bin/script-long.sh
```

### Cas d'usage courants

**1. Inspecter les fichiers de configuration** :
```bash
docker exec mon-nginx cat /etc/nginx/nginx.conf
```

**2. Tester la connectivité réseau** :
```bash
docker exec mon-conteneur ping -c 3 google.com
```

**3. Vérifier les variables d'environnement** :
```bash
docker exec mon-conteneur env
```

**4. Accéder à une base de données** :
```bash
docker exec -it ma-base mysql -u root -p
```

**5. Exécuter des tests** :
```bash
docker exec mon-app npm test
```

**6. Vider le cache d'une application** :
```bash
docker exec mon-conteneur php artisan cache:clear
```

**7. Créer un backup** :
```bash
docker exec ma-base mysqldump -u root -p ma_base > backup.sql
```

## La commande `docker logs` : consulter les journaux

### Qu'est-ce que `docker logs` ?

`docker logs` affiche les **logs** (journaux) d'un conteneur, c'est-à-dire tout ce qui a été écrit sur la sortie standard (stdout) et la sortie d'erreur (stderr) du processus principal du conteneur.

C'est la **première commande à utiliser** quand quelque chose ne fonctionne pas comme prévu !

### Syntaxe de base

```bash
docker logs <nom_ou_id_conteneur>
```

### Exemple simple

```bash
docker logs mon-nginx
```

Sortie exemple :
```
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
...
2025/10/22 10:30:46 [notice] 1#1: nginx/1.27.3
2025/10/22 10:30:46 [notice] 1#1: start worker processes
```

### Suivre les logs en temps réel avec `-f`

Pour afficher les logs en continu (comme `tail -f`) :

```bash
docker logs -f mon-nginx
```

Les nouveaux logs apparaîtront au fur et à mesure. Appuyez sur `Ctrl+C` pour arrêter de suivre.

💡 **Astuce** : C'est extrêmement utile pendant le développement pour voir les erreurs en temps réel !

### Afficher les horodatages avec `-t`

Pour voir la date et l'heure de chaque ligne de log :

```bash
docker logs -t mon-nginx
```

Sortie :
```
2025-10-22T10:30:45.123456789Z /docker-entrypoint.sh: Configuration starting
2025-10-22T10:30:46.234567890Z [notice] 1#1: nginx/1.27.3
```

### Limiter le nombre de lignes avec `--tail`

Pour voir uniquement les N dernières lignes :

```bash
docker logs --tail 50 mon-nginx
```

Pour voir les 10 dernières lignes et suivre en temps réel :

```bash
docker logs -f --tail 10 mon-nginx
```

### Filtrer par période avec `--since` et `--until`

**Afficher les logs depuis un certain temps** :

```bash
# Logs des 5 dernières minutes
docker logs --since 5m mon-nginx

# Logs de la dernière heure
docker logs --since 1h mon-nginx

# Logs depuis une date précise (format RFC3339)
docker logs --since 2025-10-22T10:00:00Z mon-nginx
```

**Afficher les logs jusqu'à un certain moment** :

```bash
docker logs --until 2025-10-22T12:00:00Z mon-nginx
```

**Combiner les deux** :

```bash
docker logs --since 1h --until 30m mon-nginx
```

Cela affiche les logs entre il y a 1 heure et il y a 30 minutes.

### Afficher uniquement stderr ou stdout

Par défaut, `docker logs` affiche les deux flux (stdout et stderr). Certains outils permettent de filtrer, mais Docker n'a pas d'option native pour cela. Vous pouvez rediriger dans un script shell :

```bash
docker logs mon-nginx 2>&1 | grep "error"
```

### Cas d'usage courants

**1. Déboguer un conteneur qui ne démarre pas** :
```bash
docker logs mon-conteneur
```

**2. Surveiller les erreurs en temps réel** :
```bash
docker logs -f mon-conteneur | grep -i error
```

**3. Analyser les logs d'une application web** :
```bash
docker logs --tail 100 mon-webapp | grep "POST /api"
```

**4. Vérifier les logs d'une base de données** :
```bash
docker logs ma-base | grep -i "ready for connections"
```

**5. Exporter les logs dans un fichier** :
```bash
docker logs mon-conteneur > conteneur-logs-$(date +%Y%m%d).log
```

**6. Surveiller plusieurs conteneurs** :
```bash
docker logs -f conteneur1 &  
docker logs -f conteneur2 &  
wait  
```

### Comprendre la rotation des logs

Docker stocke les logs localement. Par défaut, il n'y a pas de limite de taille, ce qui peut remplir votre disque !

**Vérifier la taille des logs** :
```bash
docker inspect --format='{{.LogPath}}' mon-conteneur
```

Puis :
```bash
ls -lh /var/lib/docker/containers/[ID]/[ID]-json.log
```

**Configurer la rotation des logs** (dans `/etc/docker/daemon.json`) :

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

Cela limite chaque fichier de log à 10 Mo et conserve 3 fichiers au maximum.

## La commande `docker attach` : se connecter au processus principal

### Qu'est-ce que `docker attach` ?

`docker attach` vous connecte **directement au processus principal** du conteneur. Vous voyez ce que le processus écrit sur stdout/stderr et pouvez interagir avec stdin.

⚠️ **Attention** : Cette commande est moins utilisée que `docker exec` car elle présente plusieurs inconvénients (voir la comparaison plus bas).

### Syntaxe de base

```bash
docker attach <nom_ou_id_conteneur>
```

### Exemple

```bash
docker attach mon-nginx
```

Vous êtes maintenant attaché au processus Nginx. Vous verrez tous les logs en temps réel, mais vous ne pouvez pas exécuter de commandes.

### Se détacher sans arrêter le conteneur

Par défaut, si vous appuyez sur `Ctrl+C` après un `attach`, le conteneur s'arrête ! Pour vous détacher sans l'arrêter, utilisez la séquence de touches :

```
Ctrl+P puis Ctrl+Q
```

💡 **Astuce importante** : Appuyez d'abord sur `Ctrl+P`, maintenez, puis appuyez sur `Ctrl+Q`. Relâchez tout.

### Mode read-only avec `--no-stdin`

Pour attacher en lecture seule (ne pas envoyer d'entrée au processus) :

```bash
docker attach --no-stdin mon-conteneur
```

### Utiliser un proxy de détachement avec `--detach-keys`

Vous pouvez personnaliser la séquence de touches pour vous détacher :

```bash
docker attach --detach-keys="ctrl-x,x" mon-conteneur
```

Maintenant, `Ctrl+X` suivi de `X` vous détache du conteneur.

### Cas d'usage

`docker attach` est utile principalement pour :

1. **Déboguer un conteneur interactif** lancé avec `-it` :
   ```bash
   docker run -dit ubuntu bash
   docker attach <id>
   ```

2. **Observer la sortie d'un processus** qui écrit beaucoup sur stdout :
   ```bash
   docker attach mon-processus-verbeux
   ```

3. **Interagir avec une application console** dans le conteneur :
   ```bash
   docker attach mon-jeu-console
   ```

## Comparaison : `exec` vs `attach` vs `logs`

Voici un tableau pour comprendre quand utiliser chaque commande :

| Critère | `docker exec` | `docker attach` | `docker logs` |
|---------|---------------|-----------------|---------------|
| **But principal** | Exécuter de nouvelles commandes | Se connecter au processus principal | Consulter les journaux |
| **Processus** | Crée un nouveau processus | Se connecte au processus existant | Aucun (lecture seule) |
| **Interactivité** | ✅ Complète (avec -it) | ⚠️ Limitée | ❌ Aucune |
| **Shell** | ✅ Oui (bash, sh, etc.) | ❌ Non | ❌ Non |
| **Impact sur le conteneur** | ❌ Aucun impact | ⚠️ Ctrl+C arrête le conteneur | ❌ Aucun impact |
| **Sécurité** | ✅ Plus sûr | ⚠️ Risqué | ✅ Très sûr |
| **Utilisation courante** | ✅ Débug, maintenance | ⚠️ Rare | ✅✅✅ Débug, monitoring |
| **Conteneur arrêté** | ❌ Ne fonctionne pas | ❌ Ne fonctionne pas | ✅ Fonctionne |

### Recommandations

- **Pour le debug et la maintenance** : Utilisez `docker exec -it`
- **Pour consulter les logs** : Utilisez `docker logs -f`
- **Pour attach** : Utilisez-le rarement et avec précaution

### Pourquoi privilégier `exec` plutôt qu'`attach` ?

1. **Sécurité** : `exec` ne risque pas d'arrêter le conteneur par erreur
2. **Flexibilité** : Vous pouvez exécuter n'importe quelle commande
3. **Isolation** : Chaque `exec` est un processus séparé
4. **Simplicité** : Plus facile à utiliser pour les débutants

**Exemple concret** :

❌ **Mauvaise pratique** :
```bash
docker attach mon-serveur-web
# Si vous appuyez sur Ctrl+C par erreur, le serveur s'arrête !
```

✅ **Bonne pratique** :
```bash
docker exec -it mon-serveur-web bash
# Vous êtes dans un shell séparé, Ctrl+C ne fait que quitter le shell
```

## Combiner les commandes

### Suivre les logs tout en exécutant des commandes

Ouvrez deux terminaux :

**Terminal 1** :
```bash
docker logs -f mon-application
```

**Terminal 2** :
```bash
docker exec -it mon-application bash
```

Vous pouvez ainsi voir les logs en temps réel pendant que vous effectuez des actions dans le conteneur.

### Déboguer un problème de démarrage

```bash
# 1. Vérifier les logs pour voir l'erreur
docker logs mon-conteneur

# 2. Si le conteneur est arrêté, le redémarrer
docker start mon-conteneur

# 3. Accéder au conteneur pour corriger
docker exec -it mon-conteneur bash

# 4. Suivre les logs après correction
docker logs -f mon-conteneur
```

### Script de monitoring

```bash
#!/bin/bash
# Surveiller plusieurs conteneurs
echo "=== Monitoring des conteneurs ==="

for container in $(docker ps --format '{{.Names}}'); do
    echo ""
    echo "📦 Conteneur: $container"
    echo "📊 Dernières lignes de log:"
    docker logs --tail 5 $container
    echo "---"
done
```

## Bonnes pratiques

### 1. Utilisez `exec` plutôt qu'`attach` pour la maintenance

```bash
# ✅ Bon
docker exec -it mon-conteneur bash

# ❌ Risqué
docker attach mon-conteneur
```

### 2. Limitez l'accès aux logs avec `--tail`

```bash
# ✅ Bon - charge uniquement les dernières lignes
docker logs --tail 100 -f mon-conteneur

# ❌ Peut être lent pour de gros logs
docker logs -f mon-conteneur
```

### 3. Utilisez des filtres temporels pour les logs

```bash
# ✅ Bon - analyse ciblée
docker logs --since 30m mon-conteneur | grep ERROR

# ❌ Analyse tout l'historique
docker logs mon-conteneur | grep ERROR
```

### 4. Configurez la rotation des logs

Évitez que les logs ne remplissent votre disque en configurant la rotation dans `/etc/docker/daemon.json`.

### 5. N'utilisez pas `exec` pour modifier des fichiers de configuration

Les modifications faites avec `exec` dans un conteneur sont **éphémères** : elles disparaissent si le conteneur est supprimé ou recréé.

```bash
# ❌ Mauvaise pratique
docker exec mon-nginx vim /etc/nginx/nginx.conf

# ✅ Bonne pratique : utiliser des volumes ou reconstruire l'image
```

### 6. Utilisez `-u` pour respecter les permissions

```bash
# ✅ Bon - respecte les permissions de l'application
docker exec -u www-data mon-conteneur php artisan migrate

# ❌ Peut créer des problèmes de permissions
docker exec mon-conteneur php artisan migrate
```

### 7. Documentez vos commandes de debug

Créez un fichier `DEBUG.md` dans votre projet avec les commandes utiles :

````markdown
## Commandes de debug

### Voir les logs de l'application
```bash
docker logs -f --tail 100 mon-app
```

### Accéder au shell
```bash
docker exec -it mon-app bash
```

### Vérifier la configuration
```bash
docker exec mon-app cat /app/config/app.php
```
````

### 8. Automatisez les tâches courantes

Créez des alias pour gagner du temps :

```bash
# Dans votre ~/.bashrc ou ~/.zshrc
alias dlogs='docker logs -f --tail 100'  
alias dexec='docker exec -it'  
alias dsh='docker exec -it'  
```

Utilisation :
```bash
dlogs mon-conteneur  
dexec mon-conteneur bash  
```

## Dépannage courant

### "Cannot exec in a stopped container"

**Problème** : Vous essayez d'utiliser `exec` sur un conteneur arrêté.

**Solution** :
```bash
# Démarrer le conteneur d'abord
docker start mon-conteneur

# Puis exécuter la commande
docker exec -it mon-conteneur bash
```

### "OCI runtime exec failed: exec failed"

**Problème** : La commande que vous essayez d'exécuter n'existe pas dans le conteneur.

**Solution** :
```bash
# Vérifier quelle commande est disponible
docker exec mon-conteneur which bash  
docker exec mon-conteneur which sh  

# Utiliser la commande disponible
docker exec -it mon-conteneur sh
```

### Les logs ne s'affichent pas

**Problème** : `docker logs` ne retourne rien ou très peu.

**Causes possibles** :
1. L'application n'écrit pas sur stdout/stderr
2. L'application utilise un fichier de log à la place
3. Le conteneur vient juste de démarrer

**Solution** :
```bash
# Vérifier que le conteneur est en cours d'exécution
docker ps

# Vérifier les logs avec horodatage
docker logs -t mon-conteneur

# Si l'application écrit dans un fichier
docker exec mon-conteneur cat /var/log/application.log
```

### "The input device is not a TTY"

**Problème** : Vous essayez d'utiliser `-it` dans un script ou un pipeline.

**Solution** :
```bash
# Sans -it si non interactif
docker exec mon-conteneur ls -la

# Ou vérifier si c'est un TTY
if [ -t 0 ]; then
    docker exec -it mon-conteneur bash
else
    docker exec mon-conteneur bash
fi
```

## Outils complémentaires

### Docker Desktop Dashboard

Docker Desktop offre une interface graphique pour :
- Consulter les logs
- Ouvrir un terminal dans le conteneur
- Inspecter les fichiers

### Extensions VS Code

**Dev Containers** permet de :
- Développer directement dans un conteneur
- Déboguer avec des breakpoints
- Accéder aux fichiers du conteneur

### Portainer

Interface web pour gérer Docker, incluant :
- Console interactive
- Visualisation des logs en temps réel
- Exécution de commandes

## Résumé

Dans cette section, vous avez appris à :

- ✅ **Exécuter des commandes** dans un conteneur avec `docker exec`
- ✅ **Ouvrir un shell interactif** avec `docker exec -it`
- ✅ **Consulter les logs** avec `docker logs` et ses options
- ✅ **Suivre les logs en temps réel** avec `docker logs -f`
- ✅ **Filtrer les logs** par période avec `--since` et `--until`
- ✅ **Se connecter au processus principal** avec `docker attach`
- ✅ **Comprendre les différences** entre exec, attach et logs
- ✅ **Appliquer les bonnes pratiques** pour un debugging efficace

Ces commandes d'interaction sont vos meilleurs alliés pour le debugging et la maintenance de vos conteneurs. Maîtriser `docker exec` et `docker logs` vous rendra autonome dans la résolution de problèmes et l'exploration de vos environnements conteneurisés.

Vous avez maintenant couvert toutes les commandes Docker essentielles ! Dans les sections suivantes du tutoriel, vous apprendrez à créer vos propres images personnalisées avec les Dockerfiles.

⏭️ [Création d'images personnalisées](/05-creation-dimages-personnalisees/README.md)
