# Configuration des VLAN et Zones dans la SDN 🔧

## 1. Contexte
Cette procédure décrit la configuration des **VLAN** et **Zones** au sein de la **SDN (Software Defined Network)** utilisée dans l’infrastructure.  
L’objectif est de structurer le réseau en différentes zones logiques et de garantir une gestion cohérente des interfaces et des connexions entre les machines virtuelles.

---

## 2. Création de la Zone “Service” ➕

1. Accéder à la console de gestion SDN sur l'interface Web de Proxmox.
2. Créer une **nouvelle zone** nommée :  
   **`Service`**  
3. Cette zone sera utilisée pour l’ensemble des VLAN listés ci-dessous.

---

## 3. Création des VLAN dans les Vnets ➕

### VLAN concernés 
| Nom du VLAN     | ID VLAN | Zone associée |
|-----------------|----------|---------------|
| Gestion Actif   | 222      | Service       |
| Client          | 224      | Service       |
| DMZ             | 226      | Service       |
| Serveur         | 221      | Service       |

### Étapes de création 📝
1. Dans la section **Vnets**, créer un VLAN pour chaque identifiant mentionné ci-dessus.  
2. Affecter à chaque VLAN :
    - Un **nom explicite** (ex. : `VLAN_GestionActif`, `VLAN_Client`, etc.).  
    - La **zone “Service”**.  
3. **Ne pas cocher** :
    - *Gestion des VLANs*  
    - *Pare-feu*  
4. **Ne pas ajouter d’étiquette VLAN (VLAN Tag)** lors de la configuration.

---

## 4. Association des VLAN aux Machines Virtuelles (VM) 🔗

Lors de la création d’une nouvelle VM :

1. Dans les paramètres de la **carte réseau** de la VM, section **Pont (Bridge)** :
    - Sélectionner la **Zone “Service”**.  
2. **Ne pas cocher** :
    - *Gestion des VLANs*  
    - *Pare-feu*   
3. **Ne pas renseigner de Tag VLAN**.  

---

## 5. Problème de détection de la carte réseau 🔴

Si, après la création de la VM, **la carte réseau n’apparaît pas dans l’invite de commande (CMD) de la VM** :

1. Ouvrir les paramètres de la VM sur Proxmox.  
2. Aller dans **Matériel → Carte Réseau → Modèle**.  
3. Changer le modèle de la carte réseau en **Intel E1000**.  
4. Redémarrer la VM si nécessaire.  

---

## 6. VLAN indépendant

Le **VLAN 120 (Management)** est **indépendant** de la zone *Service* et de la SDN.  
Il dispose d’une interface réseau **dédiée** :  
- **Interface** : `vmbr0`  
- **Adresse IP** : `10.10.120.50/24`  

### Cas d’utilisation 🛈
Si une **VM** doit disposer d’un accès au VLAN Management :

1. Ajouter une **seconde carte réseau** à la VM.  
2. Dans la rubrique **Pont (Bridge)**, sélectionner **`vmbr0`**.  
3. **Ne pas cocher** :
    - *Gestion des VLANs*  
    - *Pare-feu*  
4. **Ne pas mettre d’étiquette VLAN**.  

---

## 7. Résumé rapide 📌

| Élément                  | Paramètre / Valeur                     |
|--------------------------|----------------------------------------|
| Zone principale          | Service                                |
| VLANs utilisés           | 221, 222, 224, 226                     |
| VLAN Management          | 120 (indépendant)                      |
| Interface Management     | vmbr0 – IP : 10.10.120.50/24           |
| Modèle de carte réseau   | Intel E1000                            |
| Options à ne pas activer | Gestion des VLANs / Pare-feu / Tag VLAN |

---
