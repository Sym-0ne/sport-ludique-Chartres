# Configuration des Switchs HP A5500 (Coeur de Réseau)

Cette documentation décrit pas à pas la configuration initiale d’un switch HP A5500 sous Comware.  
Elle couvre les points suivants :  

- Connexion au switch via le port console depuis Linux Mint  
- Réinitialisation de la configuration (factory reset)  
- Configuration d’un stack IRF (Intelligent Resilient Framework)
- Mise en place d’un VLAN de management (VLAN 120)  
- Activation de l’accès SSH pour l’administration  

---

## 🔌 Connexion au switch via port console (Linux Mint)

### Matériel requis
- Câble console (RJ45-DB9 ou adaptateur USB-RJ45 selon le modèle)
- PC sous Linux Mint avec droits administrateur

### Étapes
1. Brancher le câble console entre le PC et le switch.  

   ⚠️ Linux Mint ne supporte pas les Switchs HP dû à un problème d'OS, donc il faut utiliser le Laptot Windows XP afin de configurer les équipements HP sans problèmes.

   Utiliser Putty sur le laptop et la connexion fonctionne directement.

## 🔄 Réinitialisation de la configuration (Factory Reset)

1. Une fois démarré, supprimer la configuration existante :
    
   ```
   <HP> reset saved-configuration
   ```

   Si cette commande ne supprime pas correctement la conf, il faut les supprimés manuellement un par un : 
   
   ```
   <HP> delete "nom du fichier de conf à supprimer" 
   ```

2. Enregistrer : 

   ```
   <HP> save force 
   ```

3. Redémarrer :

   ```
   <HP> reboot
   ```

👉 Le switch redémarre avec la configuration d’usine.

## 🖇 Configuration d’un stack IRF (Intelligent Resilient Framework)

1. Vérifier que les deux switchs ont la même version logicielle.
2. Configurer l’ID IRF sur chaque switch :
   
   ```
   <HP> system-view
   [HP] irf member 1 renumber 1   ← premier switch
   [HP] irf member 1 renumber 2   ← deuxième switch
   ```

3. Sauvegarder et redémarrer :

    ```
    [HP] save
    ```

4. Configurer les ports IRF :<br>

   *** Rack4sw1 (Switch Master) IRF port configuration ***<br>

   ```
   [Rack4sw1]interface Ten-GigabitEthernet 1/1/1
   [Rack4sw1-Ten-GigabitEthernet1/1/1]shutdown
   [Rack4sw1-Ten-GigabitEthernet1/1/1]quit
   [Rack4sw1]irf-port 1/1
   [Rack4sw1-irf-port1/1]port group interface Ten-GigabitEthernet 1/1/1
   [Rack4sw1-irf-port1/1]quit
   [Rack4sw1]interface Ten-GigabitEthernet 1/1/1
   [Rack4sw1-Ten-GigabitEthernet1/1/1]undo shutdown
   [Rack4sw1]irf-port-configuration active
   [Rack4sw1]save force 
   ```

   *** Rack4sw2 (Switch Slave) port configuration ***<br>
   
   ```
   [Rack6sw2]interface Ten-GigabitEthernet 2/1/1
   [Rack6sw2-Ten-GigabitEthernet2/1/1]shutdown
   [Rack6sw2-Ten-GigabitEthernet2/1/1]quit
   ```

   ```
   [Rack6sw2]irf-port 2/2
   [Rack6sw2-irf-port2/2]port group interface Ten-GigabitEthernet 2/1/1
   [Rack6sw2-irf-port2/2]quit
   ```

   ```
   [Rack6sw2]interface Ten-GigabitEthernet 2/1/1
   [Rack6sw2-Ten-GigabitEthernet2/1/1]undo shutdown
   [Rack6sw2-Ten-GigabitEthernet2/1/1]quit
   ```

   Note: Save the configuation before activating the port.

   ```
   [Rack6sw2]save force 
   ```

5. Activer la configuration IRF et redémarrer :

   ```
   [HP] irf-port-configuration active
   [HP] save
   [HP] reboot
   ```

   *** IRF verification ***

   <Rack4sw1>display irf<br>

   Switch  Role   Priority  CPU-Mac         Description<br>
   *+1   Master  1         b8af-xxxx-xxxx  -----<br>
     2   Slave   1         b8af-xxxx-xxxx  -----

## 🌐 Étape 4 : Création d’un VLAN de management (VLAN 120)

1. Créer le VLAN 120 :<br>

   ```
   [HP] vlan 120
   [HP-vlan120] quit
   ```
2. Créer l’interface VLAN et attribuer une adresse IP libre :<br>

   ```
   [HP] interface Vlan-interface 120
   [HP-Vlan-interface120] ip address "ip" "masque sous réseau"
   [HP-Vlan-interface120] quit
   ```
3. Associer un port physique au VLAN 120 :<br>

   ```
   [HP] interface GigabitEthernet1/0/1
   [HP-GigabitEthernet1/0/1] port link-type access
   [HP-GigabitEthernet1/0/1] port access vlan 120
   ```

👉 Ce VLAN servira exclusivement pour l’administration.

## 🔐 Étape 5 : Activer et sécuriser l’accès SSH

1. Générer les clés RSA pour SSH :

   ```
   [HP] public-key local create rsa
   ```

2. Activer le service SSH (stelnet) :

   ```
   [HPSwitch] ssh server enable
   ```

3. Créer un utilisateur administrateur :

   ```
   [HP] local-user admin
   [HP-luser-admin] password simple MonMotDePasseFort
   [HP-luser-admin] service-type ssh
   [HP-luser-admin] authorization-attribute level 3
   ```
   
4. Configurer les sessions VTY pour n’autoriser que SSH :

   ```
   [HP] user-interface vty 0 4
   [HP-ui-vty0-4] authentication-mode scheme
   [HP-ui-vty0-4] protocol inbound ssh
   [HP-ui-vty0-4] quit
   ```

👉 Ainsi, Telnet est désactivé et seul SSH est autorisé.

## ✅ Étape 6 : Vérifications et tests

1. Vérifier l’état du stack IRF :

   ```
   <HP> display irf
   ```

2. Vérifier l’adresse IP du VLAN de management :

   ```
   <HP> display ip interface brief
   ```

3. Depuis un poste client, tester l’accès SSH :

   ```
   ssh admin@ip.vlan.management
   ```

👉 Si tout est correct, la connexion doit s’établir en SSH avec l’utilisateur admin.

4. Une fois SSH fonctionnel, il faut test un PING des machines physiques au Coeur de réseau, des différentes VM Nutanix Coeur de réseau.

⚠️ Si le ping depuis la machine physique vers la VM Windows ne fonctionne pas : activer une règle dans les paramètres du Pare-Feu de Windows afin d'autoriser les pings entrants car il les bloque par défaut.