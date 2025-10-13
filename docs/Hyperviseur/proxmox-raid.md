# Configuration de Proxmox VE & RAID

## 1. Proxmox VE

### Installation 📦

* Mise en place de **Proxmox VE** sur le serveur physique.
* Définition d’un **mot de passe administrateur** avec l'utilisateur par défaut **root** pour accéder à l’interface web et à l’OS.

### Configuration de l’hyperviseur 🔧

* Attribution d’un **nom d’hôte** :
  `cha.chartres.sportludiques.fr`
  
* Association des **VLAN** :
  - VLAN **Mana** : `120` → accès à l’interface web d’administration.
    - Adrresse IP du **Mana** : `10.10.120.50`
  - VLAN **Serveur** : `221` → configuration car l’hyperviseur est utilisé en tant que serveur.
    - Adrresse IP du **Serveur** : `172.28.33.4`
---

## 2. RAID 1+0

### Contrainte matérielle ⚙️

* Le serveur (génération 8) ne prend en charge que les niveaux de RAID suivants :

  * RAID 0
  * RAID 1
  * RAID 1+0

### Choix effectué 🟢

* Mise en place d’un **RAID 1+0** (au lieu du RAID 5 initialement prévu).
* Conséquence : perte de **50 % de la capacité totale de stockage**, mais gain en **fiabilité** et en **performances** en lecture/écriture.

---
