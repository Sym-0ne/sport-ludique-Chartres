# Installation et Configuration d’OPNsense sur Nutanix

## 1. Installation

### Déploiement de la VM

* Création d’une VM **OPNsense** sur Nutanix.
* Ressources attribuées :

  * **Stockage** : 50 Go
  * **RAM** : 8 Go
  * **vCPU** : 2

### Réseau et VLAN

* Association de **3 VLAN** :
  - VLAN **Management** : `120`
  - VLAN **DMZ** : `226`
  - VLAN **LAN2DMZ** : `223`

* Attribution d’une **adresse IP de management** (10.10.120.70) dans le VLAN 120 pour accéder à l’interface web et administrer le pare-feu.

---

## 2. Configuration

### Interfaces

* Activation des interfaces **DMZ** et **LAN2DMZ**.
* Attribution d’une **adresse IP** et d’une **passerelle** à chacune : 

    - **DMZ** : 
        * Adresse IP : 172.28.62.253 
        * Gateway : 172.28.62.254
    - **LAN2DMZ** : 
        * Adresse IP : 172.28.63.140
        * Gateway : 172.28.63.254

### Règles de Pare-feu

* Mise en place temporaire d’une règle permissive :

  * **Allow All / Any** (autorisation de toutes les trames, y compris ICMP/ping).
  * Objectif : tester le bon fonctionnement initial du pare-feu.
* Des règles de filtrage plus restrictives seront appliquées ultérieurement.

### Routage

* Configuration des **routes aller/retour**.
* Définition d’une **route par défaut** pour l’accès Internet.

---
