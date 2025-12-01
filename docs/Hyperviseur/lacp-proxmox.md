# Documentation : Configuration LACP et Trunk via l’interface Web Proxmox VE

---

## Objectif :
----------

Mettre en place une configuration de liens agrégés (LACP) et de trunks VLAN sur Proxmox VE et le switch hyperviseur, afin d’assurer la haute disponibilité et la segmentation réseau via VLANs.

---

## 1. Configuration sur Proxmox VE (Interface Web) 

### 1.1 Création du Bond LACP 

1. Connectez-vous à l’interface web de Proxmox VE.
2. Allez dans **Datacenter → Node → System → Network**.
3. Cliquez sur **Create → Linux Bond**.
4. Paramétrez le bond comme suit :
    - **Name** : `bond0`
    - **Slaves** : sélectionnez les interfaces (ex : `eno1`, `ens3f0`, `ens3f1`) 
    - **Mode** : `802.3ad (LACP)`
    - **Transmit Hash Policy** : `layer2+3` 
5. Cliquez sur **Create** pour valider.

⚠️ Ne mettre aucune IP ni Gateway sur le **Linux Bond**

---

### 1.2 Attribution de l’IP au bond via un Linux Bridge 

1. Toujours dans l’onglet **Network**, cliquez sur **Create → Linux Bridge**.
2. Paramétrez le bridge pour utiliser le bond comme port physique :
    - **Name** : `vmbr2`
    - **Bridge ports** : `bond0`
    - **Manage Vlan's** : `On`
    - **STP** : `Off`
    - **IP Address** : `172.28.33.4/24`
    - **Gateway (IPv4)** : `172.28.33.254`
    - **Vlan ID** : `221-229`

3. Cliquez sur **Create** pour appliquer le bridge.

> Le bridge `vmbr2` permet à vos VM de se connecter au réseau via le bond LACP.

---

### 1.3 VLANs sur le bridge 

Contraites VLANs with Trunk :<br>

   * VLAN serveur : VLAN 221<br>
   * VLANs supplémentaires : 222 à 229 si nécessaire.<br>

> Cette configuration permet de gérer les VLANs directement sur le bridge et d’avoir des VM ou containers connectés aux VLANs via le bridge.

---

## 2. Configuration sur le Switch Hyperviseur 

### 2.1 Création du Port-Channel (LACP)

1. Sélectionnez les ports connectés à l’hyperviseur.
2. Créez un **Port-Channel** et activez LACP.

**Configuration Cisco :**

```bash
interface range GigabitEthernet1/0/21-23
    channel-group 2 mode active
    switchport mode trunk
```

* channel-group 2 mode active : active LACP.
* switchport mode trunk : configure le trunk.

### 2.2 Configuration des VLANs sur le trunk

VLANs autorisés sur le trunk : 221 à 229

**Configuration Cisco :**

```bash
interface Port-channel2
    switchport trunk allowed vlan 221-229
```

---

## 3. Vérifications 

### 3.1 Sur Proxmox VE 

a) Vérifiez que le bond fonctionne :

* Datacenter → Node → System → Network → bond0
* Vérifiez les VLANs sur le bridge :
* Datacenter → Node → System → Network → vmbr2 → VLANs

b) Test de connectivité :

* Ping depuis l’hyperviseur vers la passerelle : ping 172.28.33.254
* Ping vers d’autres VLANs si configurés

### 3.2 Sur le Switch

```bash
show etherchannel summary
show vlan brief
show interfaces trunk
```

* Vérifier que le port-channel est actif et les VLANs correctement propagés.

---