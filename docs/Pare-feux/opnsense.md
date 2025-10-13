# Installation et Configuration d’OPNsense sur Nutanix

## 1. Installation 📦

### Déploiement de la VM 🚀

* Création d’une VM **OPNsense** sur Nutanix.
* Ressources attribuées :

  * **Stockage** : 50 Go
  * **RAM** : 8 Go
  * **vCPU** : 2

### Réseau et VLAN 🌐

* Association de **3 VLAN** :
  - VLAN **Management** : `120`
  - VLAN **DMZ** : `226`
  - VLAN **LAN2DMZ** : `223`

* Attribution d’une **adresse IP de management** (10.10.120.70) dans le VLAN 120 pour accéder à l’interface web et administrer le pare-feu.

---

## 2. Configuration 🔧

### Accès Web & comptes 🖥️

- **Utilisateur administrateur** : `root` (nom d’utilisateur vérifié).  
- **Mot de passe** : mot de passe initial remplacé — un mot de passe administrateur personnalisé a été défini pour l’accès à l’interface web.  
- Recommandation : stocker les identifiants de manière sécurisée (gestionnaire de mots de passe).


### Interfaces 🖧

* Activation des interfaces **DMZ** et **LAN2DMZ**.
* Attribution d’une **adresse IP** et d’une **passerelle** à chacune : 

    - **DMZ** : 
        * Adresse IP : 172.28.62.253 
        * Gateway : 172.28.62.254
    - **LAN2DMZ** : 
        * Adresse IP : 172.28.63.140
        * Gateway : 172.28.63.254

### Règles de Pare-feu 🧱

* Mise en place temporaire d’une règle permissive :

  * **Allow All / Any** (autorisation de toutes les trames, y compris ICMP/ping).
  * Objectif : tester le bon fonctionnement initial du pare-feu.
* Des règles de filtrage plus restrictives seront appliquées ultérieurement.

### Routage 🛣️

* Configuration des **routes aller/retour**.

    * Adresse IP : 172.28.35.0/24 | Gateway : 172.28.63.130 (SWCORE) 
    * Adresse IP : 172.28.32.0/24 | Gateway : 172.28.63.130 (SWCORE) 
    * Adresse IP : 172.28.33.0/24 | Gateway : 172.28.63.130 (SWCORE)  

---
