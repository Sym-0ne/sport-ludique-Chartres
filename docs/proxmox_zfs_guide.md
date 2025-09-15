# Proxmox + ZFS RAID-Z1

---

# 1. Introduction
Ce guide fournit une documentation complète et détaillée sur l'utilisation de **Proxmox VE** avec **ZFS RAID-Z1**. Il explique les concepts, le fonctionnement, les avantages, et propose des instructions pratiques pour la mise en place et la gestion de l’infrastructure.

---

# 2. Concepts fondamentaux

## 2.1 Proxmox VE
Proxmox VE est une solution open-source de virtualisation, permettant de gérer :
- **Machines virtuelles (VM)** via KVM.
- **Containers Linux (LXC)** pour des services légers.
- **Clusters multi-nœuds** pour la haute disponibilité.
- **Stockages avancés**, incluant ZFS, Ceph, NFS, iSCSI.

**Objectif pédagogique** : Proxmox permet de comprendre le fonctionnement d’une infrastructure virtuelle complète, de l’hébergement de services à la gestion des ressources réseau et stockage.

## 2.2 ZFS (Zettabyte File System)
ZFS combine un **système de fichiers** et un **gestionnaire de volumes logiques**, offrant :
- **Protection des données** via des checksums pour détecter et corriger la corruption (bit rot).
- **RAID-Z** : tolérance aux pannes intégrée.
- **Snapshots** : copies instantanées pour sauvegarde ou test.
- **Pools de stockage** : plusieurs disques regroupés en un volume unique et flexible.
- **Compression et déduplication** pour optimiser l’espace disque.

### Concepts importants de ZFS
- **Pool** : volume logique regroupant plusieurs disques.
- **Vdev** : unité de stockage à l’intérieur d’un pool (ex : RAID-Z1, miroir).
- **Dataset** : sous-volume ZFS avec quotas, snapshots et options spécifiques.

## 2.3 RAID-Z1
RAID-Z1 est l’équivalent de RAID 5 :
- **Tolérance** : un disque peut tomber sans perte de données.
- **Capacité utilisable** : N-1 disques (ex. 4x1To = 3To utilisables).
- **Parité distribuée** : répartie sur tous les disques pour reconstruction automatique.
- **Performance** : lecture rapide grâce au striping ; écriture légèrement plus lente que RAID 10 à cause du calcul de parité.

---. Il explique les concepts, le f

# 3. Fonctionnement détaillé de ZFS RAID-Z1

### 3.1 Création et organisation
1. **Pool ZFS** : regroupe tous les disques sélectionnés.
2. **Distribution des données** : chaque bloc de données est écrit sur plusieurs disques avec un bloc de parité.
3. **Tolérance aux pannes** : si un disque tombe, ZFS reconstruit automatiquement ses données depuis la parité.

### 3.2 Lecture/Écriture
- **Lecture** : parallélisée sur tous les disques du pool → très bonnes performances.
- **Écriture** : nécessite le calcul de la parité → légèrement moins rapide que RAID 10.
- **Reconstruction après panne** : lecture et calcul de parité sur les disques restants → plus sûr et rapide que RAID 5 classique grâce à ZFS.

### 3.3 Snapshots et clones
- **Snapshots** : instantanés qui ne consomment que l’espace des modifications.
- **Clones** : copies de datasets ou VMs dérivées d’un snapshot, idéales pour TP ou tests.

### 3.4 Autres fonctionnalités
- **Compression automatique** : `lz4` recommandée, gain d’espace sans perte de performances.
- **Checksums** : détecte la corruption et corrige automatiquement si possible.
- **Gestion des quotas** : limiter l’espace disque par VM ou dataset.

---

# 4. Pré-requis matériels 
- **Disques** : 4 disques identiques pour éviter déséquilibre et erreurs.
- **RAM** : minimum 8Go, 1Go conseillé par To de stockage pour ZFS.
- **Processeur** : supporte la virtualisation (Intel VT-x ou AMD-V).
- **Réseau** : carte réseau compatible pour la gestion Web et clusters.

---

# 5. Installation de Proxmox VE

## 5.1 Téléchargement et préparation
1. Télécharger l’ISO officiel depuis [proxmox.com](https://www.proxmox.com/en/downloads).
2. Créer une clé USB bootable.
3. Installer sur le serveur physique.

## 5.2 Configuration initiale
- Définir l’adresse IP fixe.
- Configurer le mot de passe root.
- Vérifier l’accès à l’interface Web (`https://<ip-du-serveur>:8006`).

---

# 6. Création d’un pool ZFS RAID-Z1

### 6.1 Via l’interface Web
1. **Datacenter > Stockage > Ajouter > ZFS**
2. Nommer le pool (`pool_zfs`).
3. Sélectionner les 4 disques.
4. Choisir **RAID-Z1**.
5. Cliquer sur **Créer**.

### 6.2 Via la ligne de commande
```bash
# Lister les disques disponibles
lsblk

# Créer le pool ZFS RAID-Z1
zpool create pool_zfs raidz1 /dev/sd[b-e]

# Vérifier l’état du pool
zpool status
```

### 6.3 Création d’un dataset
```bash
# Créer un dataset pour les VM
zfs create pool_zfs/vmdata

# Activer la compression
zfs set compression=lz4 pool_zfs/vmdata

# Vérifier le dataset
zfs list
```

### 6.4 Gestion des snapshots
```bash
# Créer un snapshot
zfs snapshot pool_zfs/vmdata@snapshot1

# Restaurer un snapshot
zfs rollback pool_zfs/vmdata@snapshot1
```

---

# 7. Déploiement de VM sur ZFS
1. Créer une VM dans Proxmox Web.
2. Choisir `pool_zfs` comme stockage pour le disque principal.
3. Activer les snapshots et la compression si nécessaire.
4. Profiter de la tolérance aux pannes et des performances optimisées.

---

# 8. Surveillance et maintenance
- Vérifier l’état du pool : `zpool status`
- Surveiller les alertes SMART des disques.
- Faire des snapshots réguliers avant les TP critiques.
- Prévoir une sauvegarde externe avec **Proxmox Backup Server** pour plus de sécurité.

---

# 9. Bonnes pratiques pédagogiques
- Toujours tester les snapshots avant de modifier les VMs.
- Documenter chaque configuration pour TP et projets.
- Expérimenter le clustering de nœuds pour comprendre la haute disponibilité.
- Comparer les performances RAID-Z1 avec RAID10 pour comprendre les compromis.

---

# 10. Conclusion
L’association **Proxmox VE + ZFS RAID-Z1** offre un environnement complet et pédagogique pour :
- Comprendre la virtualisation (VM et containers).
- Gérer un stockage sécurisé et tolérant aux pannes.
- Expérimenter avec des fonctionnalités avancées (snapshots, compression, pools).


---

# Annexes
- Commandes utiles :
```bash
zpool list
zpool status
zfs list
zfs snapshot pool_zfs/vmdata@snapshot1
zfs rollback pool_zfs/vmdata@snapshot1
```
- Liens :
  - [Documentation Proxmox VE](https://pve.proxmox.com/pve-docs/)
  - [Documentation ZFS](https://openzfs.org/wiki/Documentation)

