# Mise en place de LACP sur HP A5500

## 🔹 1. Introduction
**LACP (Link Aggregation Control Protocol)** est défini dans la norme IEEE 802.3ad.  
Il permet de regrouper plusieurs interfaces physiques en une seule interface logique (**Eth-Trunk**) afin de :  
- augmenter la bande passante disponible,  
- assurer une redondance en cas de panne de lien,  
- répartir le trafic réseau entre plusieurs ports.  

Dans une stack de switchs HP A5500, l’utilisation de **LACP** est recommandée pour les liaisons :  
- vers un autre switch (uplink inter-switch),  
- vers un serveur ou une infrastructure (ex. Nutanix, VMware).  

---

## 🔹 2. Prérequis
- Deux équipements **compatibles LACP** (ex. switchs HP, serveur avec carte réseau supportant IEEE 802.3ad).  
- Même configuration de vitesse et duplex sur les interfaces physiques.  
- VLAN et trunk configurés de manière cohérente de part et d’autre du lien.  
- Accès en **CLI** sur le switch (console/SSH).

---

## 🔹 3. Étapes de configuration

### 3.1. Création du port logique (Eth-Trunk)
Depuis le mode configuration système :
```bash
[HP] system-view
[HP] interface Eth-Trunk 1
[HP-Eth-Trunk1] mode lacp
```
- `Eth-Trunk 1` : création d’un port agrégé logique numéro 1.  
- `mode lacp` : activation du protocole LACP.  

---

### 3.2. Ajout des interfaces physiques dans l’Eth-Trunk
Exemple avec **GigabitEthernet1/0/1** et **GigabitEthernet1/0/2** :
```bash
[HP] interface GigabitEthernet 1/0/1
[HP-GigabitEthernet1/0/1] port link-aggregation group 1

[HP] interface GigabitEthernet 1/0/2
[HP-GigabitEthernet1/0/2] port link-aggregation group 1
```
- Les interfaces physiques sont associées au groupe `Eth-Trunk 1`.  
- Elles deviennent des membres du port agrégé.  

---

### 3.3. Configuration du type de lien (trunk/access)
Si le lien doit transporter plusieurs VLANs (trunk) :
```bash
[HP] interface Eth-Trunk 1
[HP-Eth-Trunk1] port link-type trunk
[HP-Eth-Trunk1] port trunk permit vlan all
```
➡ Autorise tous les VLANs.  
*(adapter selon ton plan de VLANs)*  

Si le lien est pour un seul VLAN (access) :
```bash
[HP] interface Eth-Trunk 1
[HP-Eth-Trunk1] port link-type access
[HP-Eth-Trunk1] port access vlan 10
```

---

### 3.4. Vérification de l’état LACP
Commande de diagnostic :
```bash
[HP] display link-aggregation verbose
```

Exemple de sortie attendue :
```
Aggregation Group 1:
  Aggregation Mode: Dynamic
  Selected Ports: GigabitEthernet1/0/1, GigabitEthernet1/0/2
  Operational Key: 1
```

---

## 🔹 4. Points importants
- Tous les ports d’un même Eth-Trunk doivent être configurés **de manière identique** (même VLAN, même mode trunk/access).  
- Sur l’équipement distant (autre switch, serveur), la configuration doit correspondre (même agrégation et VLANs).  
- LACP est dynamique : seuls les liens valides et actifs seront utilisés dans l’agrégat.  
- En cas de panne d’un lien, le trafic continue sur les liens restants.  

---

## 🔹 5. Exemple complet (stack de switch HP A5500)
Objectif : agrégation de **4 ports (2 par switch dans la stack)** vers un serveur Nutanix, en trunk VLAN 10 et 20.  

```bash
[HP] system-view
[HP] interface Eth-Trunk 1
[HP-Eth-Trunk1] mode lacp
[HP-Eth-Trunk1] port link-type trunk
[HP-Eth-Trunk1] port trunk permit vlan 10 20

[HP] interface GigabitEthernet 1/0/1
[HP-GigabitEthernet1/0/1] port link-aggregation group 1

[HP] interface GigabitEthernet 1/0/2
[HP-GigabitEthernet1/0/2] port link-aggregation group 1

[HP] interface GigabitEthernet 2/0/1
[HP-GigabitEthernet2/0/1] port link-aggregation group 1

[HP] interface GigabitEthernet 2/0/2
[HP-GigabitEthernet2/0/2] port link-aggregation group 1
```

Vérification :
```bash
[HP] display link-aggregation summary
```

---

## 🔹 6. Conclusion
Avec LACP, tu disposes d’un **lien logique robuste et performant** entre ton stack HP A5500 et un serveur ou autre switch.  
Cette configuration est standard et réutilisable dans différents contextes (uplink, serveur, cluster).  
