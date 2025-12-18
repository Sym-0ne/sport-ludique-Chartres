# Mise en place de LACP sur HP A5500

## 1. Introduction

**LACP (Link Aggregation Control Protocol)** est défini dans la norme IEEE 802.3ad.  
Il permet de regrouper plusieurs interfaces physiques en une seule interface logique (**Eth-Trunk**) afin de :  
- augmenter la bande passante disponible,  
- assurer une redondance en cas de panne de lien,  
- répartir le trafic réseau entre plusieurs ports.  

Dans une stack de switchs HP A5500, l’utilisation de **LACP** est recommandée pour les liaisons :  
- vers un autre switch. 
- vers un serveur (Nutanix & Proxmox dans notre cas).  

---

## 2. Étapes de configuration

### 2.1 Création du port logique 
Depuis le mode configuration système :
```bash
[SW-A5500] system-view
[SW-A5500] int bridge-aggregation 1
[SW-A5500-Bridge-aggregation1] link-aggregation mode dynamic
[SW-A5500-Bridge-aggregation1] quit
```

---

### 2.2 Ajout des interfaces physiques dans le port link-aggregation
Exemple avec **GigabitEthernet1/0/32** et **GigabitEthernet2/0/32** :
```
[SW-A5500] int gig 1/0/32
[SW-A5500-GigabitEthernet1/0/32] port link-aggregation group 1

[SW-A5500-GigabitEthernet2/0/32] int gig 2/0/32
[SW-A5500-GigabitEthernet2/0/32] port link-aggregation group 1

[SW-A5500-GigabitEthernet2/0/32] quit
```

- Les interfaces physiques sont associées au groupe `Bridge-aggregation1`.  
- Elles deviennent des membres du port agrégé.  

---

### 2.3 Configuration du type de lien (trunk/access)
Si le lien doit transporter plusieurs VLANs (trunk) :
```
[SW-A5500] int bridge-aggregation 1
[SW-A5500-Bridge-aggregation1] port link-type trunk

[SW-A5500-Bridge-aggregation1] port trunk permit vlan *num vlan* to *num vlan* 

[SW-A5500-Bridge-aggregation1] quit
```
⚠️ ➡ Le **PERMIT VLAN ALL** ne ***fonctionne PAS*** sur le Swtich HP A5000 JD374A Comware version 5.20 (Raison Inconnu). ⚠️

Si le lien est pour un seul VLAN (access) :
```bash
[SW-A5500] int bridge-aggregation 1
[SW-A5500-Bridge-aggregation1] port link-type access
[SW-A5500-Bridge-aggregation1] port access vlan 10
```

```
[SW-A5500] save
```

---

### 2.4 Vérification de l’état LACP
Commande de diagnostic :
```bash
[HP] display link-aggregation verbose
```

Exemple de sortie attendue :
```
Aggregation Group 1:
  Aggregation Mode: Dynamic
  Selected Ports: GigabitEthernet1/0/32, GigabitEthernet2/0/32
  Operational Key: 1
```

---

## 3. Points importants
- Tous les ports d’un même Bridge-Aggregaton doivent être configurés **de manière identique** (même VLAN, même mode trunk/access).  
- Sur l’équipement distant (autre switch, serveur), la configuration doit correspondre (même agrégation et VLANs).  
- LACP est dynamique : seuls les liens valides et actifs seront utilisés dans l’agrégat.  
- En cas de panne d’un lien, le trafic continue sur les liens restants.  

---

## 4. Conclusion
Avec LACP, nous disposons d’un **lien logique robuste et performant** entre notre stack HP A5500 et un serveur ou autre switch.  
Cette configuration est standard et réutilisable dans différents contextes (uplink, serveur, cluster).  

---