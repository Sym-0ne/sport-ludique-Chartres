# Déploiement Automatique des Agents GLPI


Objectif :
---------
Mettre en place une installation automatique de l’agent GLPI (GLPI-Agent) 
sur l’ensemble des serveurs Linux et Windows de l’infrastructure.
L’agent remonte automatiquement l’inventaire matériel et logiciel 
vers le serveur GLPI.

Prérequis :
-----------
1. Un serveur GLPI fonctionnel
   Adresse : http://10.10.120.15:2000

2. Base de données externe
   Adresse : 192.168.28.10 & 10.10.120.7

3. Adresse d’inventaire du serveur GLPI-Agent :
   http://172.28.33.7/glpi/front/inventory.php

4. Ports nécessaires :
   - HTTP : 80 ou 443 selon configuration GLPI
   - Agent GLPI : 62354 (si mode push activé)

------------------------------------------------------------
1) INSTALLATION AUTOMATIQUE AGENT GLPI - SERVEURS LINUX
------------------------------------------------------------

Compatible : Debian, Ubuntu, CentOS, RHEL, Rocky Linux, AlmaLinux

A) Script d’installation automatique (Linux)
--------------------------------------------
Créer un script "install_glpi_agent.sh" :

#!/bin/bash
GLPI_SERVER="http://10.10.120.15:2000"

echo "Téléchargement de l’agent GLPI..."
wget -O /tmp/glpi-agent.deb https://github.com/glpi-project/glpi-agent/releases/latest/download/glpi-agent-linux-installer.deb

echo "Installation..."
apt install -y /tmp/glpi-agent.deb

echo "Configuration du serveur GLPI..."
sed -i "s|server =.*|server = $GLPI_SERVER|" /etc/glpi-agent/glpi-agent.cfg

echo "Activation du service..."
systemctl enable glpi-agent
systemctl start glpi-agent

echo "Envoi immédiat de l’inventaire..."
glpi-agent --debug --force

echo "Installation terminée."

--------------------------------------------
B) Exécution automatique du script
--------------------------------------------
Donner les droits :

chmod +x install_glpi_agent.sh

Exécuter :

sudo ./install_glpi_agent.sh


------------------------------------------------------------
2) INSTALLATION AUTOMATIQUE AGENT GLPI - SERVEURS WINDOWS
------------------------------------------------------------

A) Téléchargement de l’agent Windows
------------------------------------
Télécharger le dernier agent :
https://github.com/glpi-project/glpi-agent/releases/latest

Exemple : glpi-agent-x64.msi

B) Script PowerShell d’installation automatique
-----------------------------------------------
Créer un script "install_glpi_agent.ps1" :

$Installer = "C:\Temp\glpi-agent-x64.msi"
$GlpiServer = "http://172.28.33.7/glpi"

Write-Output "Installation de GLPI-Agent..."
Start-Process msiexec.exe -ArgumentList "/i $Installer SERVER=$GlpiServer /qn" -Wait

Write-Output "Démarrage du service..."
Start-Service "GLPI-Agent"

Write-Output "Envoi immédiat de l’inventaire..."
& "C:\Program Files\GLPI-Agent\glpi-agent.exe" --force

Write-Output "Installation terminée."

C) Exécuter le script
---------------------
PowerShell administrateur :

Set-ExecutionPolicy Bypass -Force
.\install_glpi_agent.ps1


------------------------------------------------------------
3) DEPLOIEMENT AUTOMATIQUE EN MASSE
------------------------------------------------------------

A) Pour Linux via SSH (Ansible)
-------------------------------
Créer un playbook Ansible simple :

- hosts: linux_servers
  become: yes
  tasks:
    - name: Copier le script
      copy: src=install_glpi_agent.sh dest=/tmp/install_glpi_agent.sh mode=0755

    - name: Installer l’agent
      command: /tmp/install_glpi_agent.sh

Exécuter :
ansible-playbook deploy_glpi_agent.yml


B) Pour Windows via GPO
-----------------------
1. Copier "glpi-agent-x64.msi" dans un dossier partagé (ex : \\DC01\packages)
2. Créer une GPO : "Déploiement GLPI-Agent"
3. Modifier : 
   Configuration ordinateur → Stratégies → Paramètres du logiciel → Installation de logiciels
4. Ajouter → Nouveau package
5. Sélectionner : \\DC01\packages\glpi-agent-x64.msi
6. Mode : Assigné
7. Redémarrer les serveurs → installation automatique

C) Pour Windows sans GPO : via PsExec
-------------------------------------
Exemple de commande :

psexec \\SERVER01 -u administrateur -p MDP powershell.exe -File C:\Temp\install_glpi_agent.ps1


------------------------------------------------------------
4) CONFIGURATION DU SERVEUR GLPI POUR L'INVENTAIRE
------------------------------------------------------------

A) Activer l’inventaire natif GLPI
----------------------------------
GLPI → Administration → Entités  
Sélectionner l’entité (ex : Racine)
Aller dans :  
Inventaire → Activer : ✔️

B) Vérifier la réception des inventaires
----------------------------------------
GLPI → Inventaire → Ordinateurs

Les serveurs Linux / Windows doivent apparaître automatiquement.


------------------------------------------------------------
5) TESTS ET VERIFICATIONS
------------------------------------------------------------

A) Côté Linux
-------------
systemctl status glpi-agent
glpi-agent --force

B) Côté Windows
---------------
Services.msc → GLPI-Agent  
État : En cours d’exécution

Inventaire manuel :
"C:\Program Files\GLPI-Agent\glpi-agent.exe" --force

C) Côté GLPI
------------
GLPI → Inventaire → Dernier contact = OK  
Informations matériel/logiciels = OK


------------------------------------------------------------
6) DEPLOIEMENT SILENCIEUX AVEC CONFIGURATION AUTOMATIQUE
------------------------------------------------------------

A) Linux : Arguments utiles
---------------------------
glpi-agent --server http://172.28.33.7/glpi --daemon

B) Windows MSI : Installation silencieuse
-----------------------------------------
msiexec /i glpi-agent-x64.msi SERVER=http://172.28.33.7/glpi /qn
