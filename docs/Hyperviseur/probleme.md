# Incident – Proxmox Crash After Power Outage

## Problème rencontré

Suite à une **coupure de courant**, le serveur a subi un **crash disque** au redémarrage.

Constats lors du redémarrage :

* Le serveur a **booté sur le CD-ROM au lieu du HDD**.
* Le **boot order dans le BIOS avait été réinitialisé**.
* Le démarrage a été **forcé manuellement sur le disque dur**.

Après démarrage du système :

* Les **services Proxmox ne démarraient plus**
* Le **cluster Proxmox était arrêté**
* Le **filesystem était monté en lecture seule**

Message observé :

```
errors=remount-ro
```

Cela signifie que le système de fichiers a détecté une erreur et s’est **verrouillé en lecture seule**.

---

# Cause du problème

Le fichier :

```
/etc/hosts
```

était **corrompu ou incorrect**.

Conséquences :

* le **hostname ne pouvait plus être résolu**
* le service **pve-cluster refusait de démarrer**
* le filesystem cluster **pmxcfs ne se montait pas**
* le dossier **/etc/pve restait vide**

Le service affichait l’état suivant :

```
pve-cluster.service
Active: inactive (dead)
```

Comme **/etc/pve est un filesystem virtuel**, s'il ne se monte pas, **le dossier apparaît vide**.

---

# Diagnostic réalisé

### Vérification du montage du filesystem

```
mount | grep " / "
```

Résultat observé :

```
errors=remount-ro
```

Le système était donc monté en **lecture seule**.

---

### Vérification de la base de configuration du cluster

```
ls -lah /var/lib/pve-cluster/
```

Présence du fichier attendu :

```
config.db
```

---

### Vérification du service cluster

```
systemctl status pve-cluster
```

Résultat :

```
Active: inactive (dead)
```

---

# Solution appliquée

## 1. Correction du fichier hosts

Modification du fichier :

```
/etc/hosts
```

Ajout de la résolution du hostname vers l’IP du serveur.

Exemple :

```
127.0.0.1 localhost
10.10.120.X CHA-PROXMOX
```

Cela permet au système de **résoudre correctement le nom du node Proxmox**.

---

## 2. Réparation du système de fichiers

Exécution d'une réparation du filesystem :

```
fsck
```

afin de corriger les erreurs provoquées par le crash disque.

---

## 3. Redémarrage du cluster

```
systemctl restart pve-cluster
```

Vérification :

```
systemctl status pve-cluster
```

---

## 4. Vérification du montage du filesystem Proxmox

```
ls /etc/pve
```

Le dossier doit maintenant contenir les configurations :

```
nodes
local
storage.cfg
```

---

## 5. Redémarrage des services Proxmox

```
systemctl restart pvedaemon
systemctl restart pveproxy
```

---

## 6. Redémarrage du serveur

```
reboot
```

---

## 7. Correction du boot order dans le BIOS

Modification du **boot order dans le Setup BIOS** :

1. HDD en premier
2. CD-ROM en second

Après modification :

* le serveur **boote automatiquement sur Proxmox**
* plus besoin de forcer le boot manuellement

---

# Conclusion

L’incident a été causé par :

* une **coupure de courant**
* une **corruption du système de fichiers**
* une **corruption du fichier /etc/hosts empêchant la résolution du hostname**

Cela a provoqué :

* l’arrêt du service **pve-cluster**
* l’absence de montage du filesystem **/etc/pve**
* l’arrêt des services Proxmox.

La correction du fichier **/etc/hosts**, la **réparation du filesystem**, puis le **redémarrage du cluster et des services Proxmox** ont permis de restaurer le fonctionnement normal du serveur.

---