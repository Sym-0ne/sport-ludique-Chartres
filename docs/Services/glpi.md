# Installation d'un serveur GLPI


## Objectif :
----------

Notre serveur GLPI va nous permettre de centraliser la gestion du parc informatique et du support technique de l'organisation **chartres.sportludique**.

Il offre une vue globale sur les équipements (postes, serveurs, réseaux, logiciels), facilite le suivi des incidents grâce à un système de tickets, automatise l’inventaire des machines et améliore la traçabilité des interventions. 

En résumé, GLPI est un outil essentiel pour organiser, superviser et optimiser l’ensemble des activités du service informatique.

---

## 1. Prérequis

-   Serveur Linux (Debian/Ubuntu/Rocky)
-   Accès root ou sudo
-   Accès réseau vers la base de données externe
-   Apache/Nginx + PHP 8.1+
-   MariaDB/MySQL externe

---

## 2. Installation des dépendances

Sur VM Linux :

```sudo apt update
sudo apt upgrade
sudo apt install apache2 mariadb-client php php-cli php-common php-mysql php-xml php-gd php-curl php-intl php-zip php-ldap php-mbstring php-apcu php-bz2 php-imap
```

---

## 3. Téléchargement de GLPI

```wget https://github.com/glpi-project/glpi/releases/latest/download/glpi.tgz
tar -xzf glpi.tgz
sudo mv glpi /var/www/html/
sudo chown -R www-data:www-data /var/www/html/glpi
```

---

## 4. Configuration Apache

Créer /etc/apache2/sites-available/glpi.conf :

```<VirtualHost *:80>
    ServerName glpi.local
    DocumentRoot ```/var/www/html/glpi
    <Directory /var/www/html/glpi>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

Activer :

```sudo a2ensite glpi
sudo a2enmod rewrite
sudo systemctl restart apache2
```

---

## 5. Configuration base de données (serveur externe)

Connexion au serveur SQL de la BDD :

```CREATE DATABASE glpi CHARACTER SET utf8mb4;
CREATE USER 'glpi_user'@'...' IDENTIFIED BY 'motdepasse';
GRANT ALL PRIVILEGES ON glpidb.* TO 'glpi_user'@'...';
FLUSH PRIVILEGES;
```

---

## 6. Installation web

Accéder via navigateur : http://IP_SERVEUR_GLPI/glpi

Puis suivre les étapes : - Sélection langue - Acceptation licence -
Choix installation - Connexion à la base externe - Création du schéma -
Finalisation

---

## 7. Sécurisation

```sudo rm -rf /var/www/html/glpi/install
```

---

## 8. Identifiants par défaut

-   glpi / glpi
-   tech / tech
-   normal / normal
-   post-only / post-only

---