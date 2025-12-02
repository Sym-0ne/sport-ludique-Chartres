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
Comme précédemment cité, nous allons utiliser Docker pour installer notre service GLPI. Pour ce faire, après avoir créé le conteneur et le fichier docker-compose.yml, nous allons lui indiquer cette configuration :


```
version: "3.9"
 
services:
  glpi:
    image: diouxx/glpi:latest # Installe la dèrnière versionde GLPI depuis les dépôts Docker
    container_name: glpi_app # Nom du conteneur
    restart: always
    ports:
      - "172.28.33.8:2000:80"   # Mappage des ports (expliqué en dessous)
      - "10.10.120.15:2000:80"  
    environment:
      GLPI_DB_HOST: ${DB_HOST} # Appel de variables, présentes dans le fichier .env
      GLPI_DB_NAME: ${DB_NAME}
      GLPI_DB_USER: ${DB_USER}
      GLPI_DB_PASSWORD: ${DB_PASSWORD}
    volumes: 
      - ./glpi:/var/www/html/glpi # Mappage des volumes (expliqué en dessous)
    networks:
      - glpi_net #Isolation des réseaux
 
networks:
  glpi_net:
    driver: bridge
```

## 2. Différences

Avec notre installation Docker, plusieurs différences existent entre une installation Docker et une installation classique.


### 2.1 Instalation 

GLPI est installé depuis les dépôts Docker. Étant donné que GLPI est dans un conteneur, la version utilisée est une version spéciale Docker qui fonctionnera sur tous les OS.


### 2.2 Mappage des ports

Compte tenu de l'existence de plusieurs services grâce à Docker, il est possible que plusieurs services aient besoin du même port de sortie (80, 443, etc.). Le mappage des ports permet de résoudre cette problématique : on attribue un port local de la machine à un port virtuel du conteneur, ce qui évite de modifier les fichiers de configuration de nos applications. 

Dans notre cas, nous avons attribué et délimité ces ports :

```
  - "172.28.33.8:2000:80"
  - "10.10.120.15:2000:80"  
```
Nous autorisons le réseau `172.28.33.8` et le réseau `10.10.120.15` sur ce conteneur. Le port 2000 est le port local de notre machine (port à contacter pour joindre le conteneur Docker) et il est mappé sur le port 80 du conteneur. Cela signifie que toutes les requêtes adressées au port 2000 des deux réseaux autorisés de notre VM arriveront sur le port 80 du conteneur GLPI.


### 2.3 Mappage des volumes

Par défaut, les fichiers d'un conteneur sont internes à celui-ci. Quand le conteneur est stoppé, tous ses fichiers sont supprimés. Il est donc important de mapper un volume. Ce mappage permet de sauvegarder les fichiers d'un conteneur dans un dossier local de la machine, rendant possible la sauvegarde de ceux-ci ainsi que la persistance des configurations. Dans notre cas, nous avons mappé le dossier `/glpi` de notre machine locale vers le dossier `/var/www/html/glpi` de notre conteneur.

Nous retrouvons bien les fichiers de GLPI dans le dossier `glpi` de notre machine locale.

```
user@user:~/glpi/glpi$ ls
ajax        bin           config           css                   files  inc         install     lib      locales      plugins  README.md  routes       src         templates  version
apirest.md  CHANGELOG.md  CONTRIBUTING.md  dependency_injection  front  index.html  INSTALL.md  LICENSE  marketplace  public   resources  SECURITY.md  SUPPORT.md  vendor
```

### Isolation des réseaux

L'isolation des réseaux est la création de cartes virtuelles propres à chaque conteneur, qui ne s'activent que lors du lancement du conteneur associé. Cela permet d'éviter toute communication indésirable entre les conteneurs.
