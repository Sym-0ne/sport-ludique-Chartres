# Docker

## 1. Principe du conteneur

Un conteneur est un espace isolé de la machine qui sers a faire tourner des applications,il partage le même noyau d'OS que l'hôte tout en ne fesant tourner uniquement les insatnces necessaires au bon fonctionnement de l'application conteneurisé. 

Au lieu d'avoir plusieurs application qui utilisent la même instance d'un programme exemple : plusisuers services Web qui utilisent la même instance d'apache, la contenairisation permet d'isoler chaque service Web dans un conteneur dédier, chaque conteneur a sa propre instance d'Apache et sa propre arborescence de fichier limitant les bugs. La conteneurisation evite donc de devoir crée une machine virtuel par application.

Toute la Documentation de docker est disponible sur [le github de mr Mery](https://lmeryfulbert.github.io/SportLudique2025-2026/cours/05-Services/containers/11-docker/#introduction).

## 2. Pourquoi utiliser Docker ? 

1. Isolation : chaque application fonctionne dans son conteneur, séparée des autres, ce qui réduit les conflits de dépendances et les bugs.

2. Portabilité : un conteneur fonctionne de la même manière partout (PC, serveur, cloud), pas de “ça marche chez moi mais pas ailleurs”.

3. Légèreté : contrairement aux machines virtuelles, les conteneurs partagent le noyau de l’OS, donc moins de ressources utilisées.

4. Déploiement rapide : créer, lancer ou arrêter un conteneur est beaucoup plus rapide qu’une VM.

5. Scalabilité : facile de lancer plusieurs instances du même conteneur pour gérer la charge.

6. Cohérence des environnements : développeurs, testeurs et production utilisent exactement le même environnement.

7. Gestion simplifiée : Docker et ses outils permettent d’automatiser le déploiement et la mise à jour des applications.

## 3. Mise en place de Docker

### 3.1 Instalation et configuration primaire

L'installation de docker est légèrement différente des installation de paquets habituels sur linux, en effet docker necessite de récupérer une clef GPG et plussuers autres manipulations pour que l'installation se passe au mieux.

Ce [tutoriel](https://www.it-connect.fr/installation-pas-a-pas-de-docker-sur-debian-11/#II_Installer_Docker_sur_Debian_13) vous montreras pas à pas comment installer docker sur un environement linux Debian 13. 

### 3.2 Création d'un conteneur

Chaque conteneur travaille dans son propre dossier, pour cet exemple nous allons crée le conteneur de notre [serveur GLPI](). 

Il nous faut donc trvavailler dans un dossier a part `mkdir glpi`, il faut ensuite crée le fichier Docker Compose, fichier docker qui permet de gérer des applications multi-conteneurs.

`nano docker-compose.yml`

```

services:
  glpi:
    image: diouxx/glpi:latest
    container_name: glpi_app 
    restart: always 
    ports: 
      - "172.28.33.8:2000:80"   # LAN
      - "10.10.120.15:2000:80"  # MANA
    environment:
      GLPI_DB_HOST: ${DB_HOST}
      GLPI_DB_NAME: ${DB_NAME}
      GLPI_DB_USER: ${DB_USER}
      GLPI_DB_PASSWORD: ${DB_PASSWORD}
    volumes:
      - ./glpi:/var/www/html/glpi
    networks:
      - glpi_net

networks:
  glpi_net:
    driver: bridge
```
Les informations propres a ce fichier Docker sont disponibles sur [la page dédiée a glpi]().

### 3.3 Lancement d'un conteneur

La commande `docker compose up -d` à éxecuter dans le dossier de votre conteneur permet de lancer les instances présentes dnas votre fichier docker-compose.yml en arrière plan 

Il est possible de vérifier les les services lancées grâce a la commande `docker compose ps`

Enfin pour arreter un conteneur il suffit d'executer la commande `docker compose down` toujours dans le répertoire de votre conteneur.
