# Mise en place du Protocole LDAPS sur Proxmox.

## Objectif :
----------

Configurer une authentification **Active Directory sécurisée (LDAPS)** sur un cluster **Proxmox**, à l’aide d’un certificat généré par le service **ADCS (Active Directory Certificate Services)**, et assurer la communication sécurisée entre le serveur **Proxmox** et les **contrôleurs de domaine**.

---

## 1. Création d’un dossier partagé pour les certificats. 

Sur le contrôleur de domaine principal (AD principal), créer un dossier partagé qui servira à **stocker le certificat généré** par le service ADCS :
```
C:\Backup\Certificat
```

---

## 2. Génération et exportation du certificat via ADCS. 

- Installer le rôle ADCS (Active Directory Certificate Services) sur le **contrôleur de domaine secondaire**.
- Générer un certificat pour le serveur LDAP sécurisé (LDAPS).
- Exporter le certificat au format .crt (ou .cer).
- Copier ce fichier dans le dossier partagé créé précédemment.

```
\\CHA-DC-01\Backup\Certificat\certificat.crt
```

Pour toutes ces étapes voir : [Tuto ADCS et Certificat](https://sym-0ne.github.io/sport-ludique-Chartres/Service%20AD/adcs/)

---

## 3. Transfert du certificat vers le serveur Proxmox. 
Sur le poste où se trouve le certificat, exécuter la commande suivante pour le transférer sur le serveur Proxmox :
```
scp Documents/certificat.crt root@10.10.120.50:/usr/local/share/ca-certificates
```

💡 10.10.120.50 correspond à l’adresse IP du serveur Proxmox.

---

## 4. Installation du certificat sur Proxmox 

Connectez-vous au serveur Proxmox via SSH, puis exécutez :
```
cd /usr/local/share/ca-certificates
ls
update-ca-certificates
```

💡 Cette commande mettra à jour la base des autorités de certification de Proxmox pour reconnaître le certificat ADCS comme fiable.

---

## 5. Normalisation du nom du serveur Active Directory.

Proxmox n’accepte pas les caractères “_” (underscore) dans les noms de serveurs.
```
Ancien nom : CHA_DC_01
Nouveau nom : CHA-DC-01
```

Ce changement permet une compatibilité totale avec Proxmox et le protocole LDAPS.

---

## 6. Utilisation du FQDN (Nom de domaine complet). 

Le nom complet (FQDN) du serveur Active Directory principal est :
```     
CHA-DC-01.cha.chartres.sportludique.fr
```

C’est ce FQDN qui sera utilisé comme identifiant de serveur AD dans la configuration de Proxmox et non l'ip du serveur.

---

## 7. Configuration du Realm (Royaume) dans Proxmox. 

Chemin d’accès :

```
Datacenter → Permissions → Realms → Add → Active Directory
```

| Champ                      | Valeur                                 |
| -------------------------- | -------------------------------------- |
| **Realm**                  | ad                                     |
| **Default Domain**         | cha.chartres.sportludique.fr           |
| **Port**                   | 636                                    |
| **Mode**                   | LDAPS                                  |
| **Server1**                | CHA-DC-01.cha.chartres.sportludique.fr |
| **Server2**                | 10.10.120.3 *(serveur secondaire AD)*  |
| **Vérifier le certificat** | ✅ Oui                                 |
| **Commentaire**            | Authentification AD                    |

🧩 Le **port 636** correspond au protocole LDAPS (LDAP sécurisé via SSL/TLS).

---