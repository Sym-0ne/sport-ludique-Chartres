# Installation et configuration d’un serveur de fichiers sur Windows Server

## Introduction

Un serveur de fichiers permet de centraliser, partager et sécuriser l’accès aux données dans une infrastructure réseau. C’est l’un des rôles les plus courants et utiles sur un serveur Windows. 

Dans cet article, nous allons voir comment installer et configurer un serveur de fichiers sur Windows Server, avec une explication des différentes options disponibles pour adapter la configuration à vos besoins.

## Prérequis

Un Windows Server installé (et de préférence rejoint à un domaine). Un disque ou volume disponible pour le stockage des données. Une connexion à l’interface d’administration(Gestionnaire de serveur).


1. Installer le rôle « Serveur de fichiers »

Ouvrir le Gestionnaire de serveur.
Cliquer sur Gérer, puis sur Ajouter des rôles et fonctionnalités :

![alt text](docs/images/srvdefichier/image.png)

Dans l’assistant, sélectionner Installation basée sur un rôle ou une fonctionnalité :

![alt text](docs/images/srvdefichier/image-1.png)

Choisir votre serveur dans la liste proposée :

![alt text](docs/images/srvdefichier/image-2.png)

Dans la section des rôles, cocher Services de fichiers et de stockage, puis Services de fichiers et iSCSI puis cochez Serveur de fichiers dans les sous-rôles proposés :

![alt text](docs/images/srvdefichier/image-3.png)

Pour une configuration standard, le Serveur de fichiers suffit.
Passez à la section Confirmation et cliquez sur Installer pour lancer l’installation.
Vous pouvez ensuite cliquer sur Installer :

![alt text](docs/images/srvdefichier/image-4.png)

Finaliser l’installation en suivant l’assistant jusqu’au bout :

![alt text](docs/images/srvdefichier/image-5.png)

2. Créer un dossier partagé

Ouvrir l’outil Gestionnaire de serveur, puis aller dans Services de fichiers et de stockage > Partages :

![alt text](docs/images/srvdefichier/image-6.png)

Cliquer sur Tâches, puis sélectionner Nouveau partage :

![alt text](docs/images/srvdefichier/image-7.png)

Choisir le type de partage selon le besoin : SMB – Partage rapide pour un usage courant, ou SMB – Avancé pour configurer plus en détail les autorisations :

![alt text](docs/images/srvdefichier/image-8.png)

Sélectionner le serveur et le disque sur lequel sera créé le dossier à partager, ou en créer un nouveau via l’interface :

![alt text](docs/images/srvdefichier/image-9.png)

Définir les paramètres comme le nom du partage, le chemin d’accès :

![alt text](docs/images/srvdefichier/image-10.png)

Configurer les autorisations d’accès selon les besoins (voir section suivante pour les détails) :

![alt text](docs/images/srvdefichier/image-11.png)