# Installation d'un serveur GLPI **(Version VIA DOCKER)**

⚠️ L'installation du GLPI via Docker est différente qu'une installation classique !
Pour voir la mise en place du Docker lié à celle de notre GLPI voir [la documentation Docker](https://sym-0ne.github.io/sport-ludique-Chartres/Serveurs/docker.md/)

## Objectif :
----------

Notre serveur GLPI va nous permettre de centraliser la gestion du parc informatique et du support technique de l'organisation **chartres.sportludique**.

Il offre une vue globale sur les équipements (postes, serveurs, réseaux, logiciels), facilite le suivi des incidents grâce à un système de tickets, automatise l’inventaire des machines et améliore la traçabilité des interventions. 

En résumé, GLPI est un outil essentiel pour organiser, superviser et optimiser l’ensemble des activités du service informatique.

---

## 1. Installation

```image: diouxx/glpi:latest``` → correspond à l’image Docker officielle GLPI que Docker va télécharger automatiquement depuis Docker Hub si elle n’est pas déjà présente sur ta machine.<br>
Cette image contient déjà Apache, PHP et GLPI prêt à l’emploi.

```
glpi:
    image: diouxx/glpi:latest
    container_name: glpi_app
```

---

## 2. Configuration 

Fichier de conf à remplir & modifier dans le serveur Docker afin de faire la liaison avec la Base de Données GLPI.

```
version: "3.9"
 
services:
  glpi:
    image: diouxx/glpi:latest
    container_name: glpi_app
    restart: always
    ports:
      - "172.28.33.8(RESEAU CLIENT AUTORISÉ):2000:80"   # Port 2000 : Port externe à DOCKER & Local à la machine
      - "10.10.120.15(RESEAU MANAGEMENT AUTORISÉ):2000:80"  # Port 80 : Port interne à DOCKER
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

---