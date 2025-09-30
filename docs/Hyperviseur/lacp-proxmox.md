# Documentation : Configuration LACP et Trunk via l’interface Web Proxmox VE

## Objectif
Mettre en place une configuration de liens agrégés (LACP) et de trunks VLAN sur Proxmox VE et le switch hyperviseur, afin d’assurer la haute disponibilité et la segmentation réseau via VLANs.

---

## 1. Configuration sur Proxmox VE (Interface Web)

### 1.1. Création du Bond LACP

1. Connectez-vous à l’interface web de Proxmox VE.
2. Allez dans **Datacenter → Node → System → Network**.
3. Cliquez sur **Create → Linux Bond**.
4. Paramétrez le bond comme suit :
   - **Name** : `bond0`
   - **Slaves** : sélectionnez les interfaces physiques (ex : `eno1`, `eno2`)
   - **Mode** : `802.3ad (LACP)`
   - **Transmit Hash Policy** : `layer2+3` (optionnel, selon besoin)
   - **LACP Rate** : `Fast`
   - **Miimon** : `100ms` (surveillance des liens)

5. Cliquez sur **Create** pour valider.

---

### 1.2. Attribution de l’IP au bond via un Linux Bridge

1. Toujours dans l’onglet **Network**, cliquez sur **Create → Linux Bridge**.
2. Paramétrez le bridge pour utiliser le bond comme port physique :
   - **Name** : `vmbr0`
   - **Bridge ports** : `bond0`
   - **STP** : `Off`
   - **Forward Delay** : `0`
   - **IP Address** : `172.28.33.4/24`
   - **Gateway (IPv4)** : `172.28.33.254`

3. Cliquez sur **Create** pour appliquer le bridge.

> Le bridge `vmbr0` permet à vos VM de se connecter au réseau via le bond LACP.

---

### 1.3. Configuration VLANs sur le bridge

1. Cliquez sur le bridge `vmbr0`, puis sur **Create → VLAN**.
2. Créez les VLANs suivants :
   - VLAN natif : pour compatibilité avec certaines cartes physiques ne reconnaissant pas les trames taguées à 4 octets.
   - VLAN serveur : VLAN 221
   - VLANs supplémentaires : 222 à 229 si nécessaire.

3. Pour chaque VLAN :
   - **VLAN tag** : numéro du VLAN (ex : 221)
   - **Bridge ports** : `bond0`
   - **IP configuration** : `Manual` ou `None` selon usage.

> Cette configuration permet de gérer les VLANs directement sur le bridge et d’avoir des VM ou containers connectés aux VLANs via le bridge.

---

## 2. Configuration sur le Switch Hyperviseur

### 2.1. Création du Port-Channel (LACP)

1. Sélectionnez les ports connectés à l’hyperviseur.
2. Créez un **Port-Channel** et activez LACP.

**Exemple Cisco :**

```bash
interface range GigabitEthernet1/0/1-3
    channel-group 1 mode active
    switchport mode trunk
```

* channel-group 1 mode active : active LACP.
* switchport mode trunk : configure le trunk.

### 2.2. Configuration des VLANs sur le trunk

VLAN natif : VLAN par défaut sur le trunk (ex : VLAN 1 ou VLAN spécifique)
VLANs autorisés sur le trunk : 221 à 229

Exemple Cisco :

```bash
interface Port-channel1
    switchport trunk native vlan 1
    switchport trunk allowed vlan 221-229
```

## 3. Vérifications

### 3.1. Sur Proxmox VE

a) Vérifiez que le bond fonctionne :

* Datacenter → Node → System → Network → bond0
* Vérifiez les VLANs sur le bridge :
* Datacenter → Node → System → Network → vmbr0 → VLANs

b) Test de connectivité :

* Ping depuis l’hyperviseur vers la passerelle : ping 172.28.33.254
* Ping vers d’autres VLANs si configurés

### 3.2. Sur le Switch

```bash
show etherchannel summary
show vlan brief
show interfaces trunk
```

* Vérifier que le port-channel est actif et les VLANs correctement propagés.