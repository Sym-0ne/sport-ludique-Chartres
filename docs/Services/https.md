# Mise en place HTTPS 

## Objectif :
----------

Mettre en place un site web sécurisé avec Apache2 utilisant
une autorité de certification interne (CA).

Le domaine utilisé est : www.chartres.sportludique.fr
La localité : Chartres
L'organisation : SportLudique

| Serveur | Rôle                      | OS                  |
| ------- | ------------------------- | ------------------- |
| CHA-CA  | Autorité de certification | Ubuntu Server 22.04 |
| CHA-WEB | Serveur web Apache        | Ubuntu Server 22.04 |

---

## 1. Préparation des serveurs

**Sur CHA-WEB**

Mise à jour du système :
```
sudo apt update && sudo apt upgrade -y
```

Installation d’Apache et des modules nécessaires :
```
sudo apt install apache2 -y
sudo a2enmod ssl
sudo a2enmod rewrite
sudo systemctl restart apache2
```

---

## 2. Création du certificat serveur

À réaliser sur CHA-WEB uniquement.

### 2.1 Création du répertoire des certificats

```
sudo mkdir -p /etc/ssl/localcerts
```

### 2.2 Génération de la clé privée du serveur

```
sudo openssl genrsa -out /etc/ssl/localcerts/www.chartres.sportludique.fr.key.pem 2048

sudo chmod 400 /etc/ssl/localcerts/www.chartres.sportludique.fr.key.pem
```

### 2.3 Génération de la CSR (Certificate Signing Request)

```
sudo openssl req -new \
  -key /etc/ssl/localcerts/www.chartres.sportludique.fr.key.pem \
  -out /etc/ssl/localcerts/www.chartres.sportludique.fr.csr.pem \
  -subj "/C=FR/ST=Centre-Val de Loire/L=Chartres/O=SportLudique/CN=www.chartres.sportludique.fr"
```

**La CSR sera ensuite envoyée à la CA interne afin d’être signée.**

### 2.4 Transfert de la CSR vers la CA

```
scp /etc/ssl/localcerts/www.chartres.sportludique.fr.csr.pem \
    certificat@IP_CHA-CA:/tmp/
```

---

## 3. Installation du certificat signé

Une fois le certificat signé sur CHA-CA, copier les fichiers suivants sur CHA-WEB :

- certificat serveur
- certificat de la CA

Exemple :
```
scp certificat@IP_CHA-CA:/home/certificat/www.chartres.sportludique.fr.cert.pem /etc/ssl/localcerts/
scp certificat@IP_CHA-CA:/home/certificat/ca.cert.pem /etc/ssl/localcerts/
```

---

## 4. Configuration Apache

### 4.1 Création du VirtualHost

```
sudo nano /etc/apache2/sites-available/www.chartres.sportludique.fr.conf
```

Configuration :

```
<VirtualHost *:80>
    ServerName www.chartres.sportludique.fr
    ServerAlias chartres.sportludique.fr
    Redirect permanent / https://www.chartres.sportludique.fr/
</VirtualHost>

<VirtualHost *:443>
    ServerName www.chartres.sportludique.fr
    DocumentRoot /var/www/www.chartres.sportludique.fr

    SSLEngine on
    SSLCertificateFile /etc/ssl/localcerts/www.chartres.sportludique.fr.cert.pem
    SSLCertificateKeyFile /etc/ssl/localcerts/www.chartres.sportludique.fr.key.pem
    SSLCACertificateFile /etc/ssl/localcerts/ca.cert.pem

    <Directory /var/www/www.chartres.sportludique.fr>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/www-chartres-error.log
    CustomLog ${APACHE_LOG_DIR}/www-chartres-access.log combined
</VirtualHost>
```

---

### 5. Organisation du site web

Création du répertoire du site :
```
sudo mkdir -p /var/www/www.chartres.sportludique.fr
```
Création d’une page d’accueil :
```
sudo nano /var/www/www.chartres.sportludique.fr/index.html
```
Permissions :
```
sudo chown -R www-data:www-data /var/www/www.chartres.sportludique.fr
sudo chmod -R 755 /var/www/www.chartres.sportludique.fr
```

---

### 6. Activation du site

Activation du VirtualHost :
```
sudo a2ensite www.chartres.sportludique.fr.conf
sudo a2dissite 000-default.conf
```

Vérification de la configuration :
```
sudo apachectl configtest
```

Si Syntax OK, redémarrer Apache :
```
sudo systemctl restart apache2
```

---

## 7. Test du certificat SSL

```
curl -Iv https://www.chartres.sportludique.fr --cacert /etc/ssl/localcerts/ca.cert.pem
```

Résultat attendu :
```
SSL certificate verify ok.
HTTP/1.1 200 OK
```

---

## 8. Astuce : Redirection automatique HTTP → HTTPS

Déjà incluse dans le VirtualHost *:80 avec la ligne :
Redirect permanent / https://www.chartres.sportludique.fr/


---

## 8. Vérification du SAN (Subject Alternative Name)

```
openssl x509 -in /etc/ssl/localcerts/www.chartres.sportludique.fr.cert.pem \
    -noout -text | grep -A2 "Subject Alternative Name"
```

Si le SAN ne s’affiche pas :
- Vérifier le fichier d’extension : `www.chartres.sportludique.fr.ext`
- Regénérer le certificat avec l’option : `-extfile`

---

## 9. Déploiement du certificat sur les clients

activer rdp dans windows 10

Pour activer le RDP (Remote Desktop / Bureau à distance) dans Windows 10, voici les méthodes les plus simples.
1️⃣ Méthode via les Paramètres (la plus facile)

    Ouvrez Paramètres (Win + I)

    Cliquez sur Système

    Cliquez sur Bureau à distance

    Activez Activer le Bureau à distance

    Confirmez avec Activer

💡 Windows activera automatiquement le pare-feu pour permettre la connexion.
2️⃣ Vérifier l’utilisateur autorisé

    Dans la même page Bureau à distance

    Cliquez sur Sélectionner les utilisateurs pouvant accéder à distance

    Ajoutez l’utilisateur si nécessaire.

3️⃣ Connexion depuis un autre PC

Depuis un autre ordinateur :

    Ouvrez Connexion Bureau à distance

    Tapez l’adresse IP ou le nom du PC

    Entrez les identifiants Windows

L’outil s’appelle :
➡️ Remote Desktop Connection
4️⃣ Vérifier l’édition de Windows

⚠️ Important : le serveur RDP fonctionne uniquement sur :

    Windows 10 Pro

    Windows 10 Enterprise

    Windows 10 Education

❌ Windows 10 Home ne permet pas d’accepter les connexions RDP.

✅ Si vous voulez, je peux aussi vous montrer :

    la commande PowerShell pour activer RDP en ligne de commande

    comment activer RDP à distance sur un PC

    comment ouvrir le port 3389 sur le firewall / routeur.


bureau a disrtand activé, je peux ping la vm windows, jai win 10 pro mais remmina se connecte pas en rdp dpeuis mon poste linux 

Si le Bureau à distance est activé, que le ping fonctionne, mais que Remmina ne se connecte pas en RDP vers ta VM Windows 10, voici les vérifications les plus courantes 👇
1️⃣ Vérifier que le port RDP est ouvert (3389)

Depuis ton poste Linux :

nc -zv IP_DE_LA_VM 3389

ou

telnet IP_DE_LA_VM 3389

Résultat attendu :

succeeded

❌ Si le port est fermé → problème firewall Windows.
2️⃣ Vérifier le firewall Windows

Dans la VM Windows :

    Ouvre Pare-feu Windows Defender

    Clique Autoriser une application via le pare-feu

    Vérifie que Bureau à distance est coché sur :

        Privé

        Public

Ou via PowerShell :

Enable-NetFirewallRule -DisplayGroup "Remote Desktop"

3️⃣ Vérifier que le service RDP tourne

Dans Windows :

services.msc

Service à vérifier :

Remote Desktop Services

Il doit être :

    Running

    Automatic

4️⃣ Vérifier le type de réseau de la VM

Si Windows pense être sur un réseau public, parfois ça bloque.

Vérifie dans :

Paramètres → Réseau → Propriétés

Mettre le réseau en Privé.
5️⃣ Vérifier le mode réseau de la VM

Très fréquent en VM 👇

Si ta VM est sur :

    NAT → RDP peut ne pas marcher directement

    Bridged / Bridge → fonctionne normalement

Selon ton hyperviseur :

    VirtualBox

    VMware Workstation

    Proxmox VE

    KVM

6️⃣ Tester avec un client RDP Linux

Pour vérifier si le problème vient de Remmina :

xfreerdp /v:IP_DE_LA_VM /u:utilisateur

Client utilisé :
➡️ FreeRDP
7️⃣ Désactiver temporairement NLA

Parfois Remmina bloque avec Network Level Authentication.

Dans Windows :

SystemPropertiesRemote

Décoche :

Autoriser uniquement les connexions utilisant NLA

✅ Si tu veux, donne-moi :

    message d’erreur exact dans Remmina

    hyperviseur utilisé (VirtualBox / KVM / VMware / Proxmox)

    IP de la VM

    résultat de nc -zv IP 3389

et je peux te diagnostiquer le problème en 1-2 minutes.

 comment verif sur windows si les port 3389 est bien ouvert sur tt les infterfaces 

Pour vérifier sur Windows 10 si le port RDP 3389 écoute bien sur toutes les interfaces, voici les méthodes les plus fiables 👇
1️⃣ Vérifier les interfaces d’écoute (méthode principale)

Ouvre PowerShell ou Invite de commandes et tape :

netstat -ano | findstr :3389

Résultat possible :

TCP    0.0.0.0:3389     0.0.0.0:0     LISTENING

✔️ 0.0.0.0:3389 = écoute sur toutes les interfaces réseau.

Autres cas :

TCP    192.168.1.50:3389     0.0.0.0:0     LISTENING

➡️ écoute uniquement sur cette IP.

TCP    [::]:3389

➡️ écoute sur toutes les interfaces IPv6.
2️⃣ Vérifier que le service RDP tourne

Le port est ouvert seulement si ce service tourne :

sc query TermService

Service :

Remote Desktop Services

Statut attendu :

STATE: RUNNING

3️⃣ Vérifier la configuration dans le registre

Commande :

reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp"

Regarde la valeur :

PortNumber

Normalement :

3389

4️⃣ Vérifier le firewall Windows

Dans PowerShell :

Get-NetFirewallRule -DisplayGroup "Remote Desktop"

Les règles doivent être Enabled = True.
5️⃣ Tester depuis la machine Windows elle-même

Toujours dans PowerShell :

Test-NetConnection localhost -Port 3389

Résultat attendu :

TcpTestSucceeded : True

✅ Diagnostic rapide
Résultat	Signification
0.0.0.0:3389 LISTENING	RDP accessible sur toutes les interfaces
LISTENING sur IP spécifique	limité à une interface
rien	RDP pas actif

💡 Si tu veux, je peux aussi te montrer la commande qui révèle 90 % des problèmes RDP entre Linux et Windows (pare-feu, NLA, interface, port).

c quoi le service en francais sur win 1°0 pr terminal service

Sur Windows 10, le service Terminal Services / TermService s’appelle en français :

➡️ Services Bureau à distance

Nom du service interne (le même dans toutes les langues) :

TermService

Comment le voir dans Windows

    Appuie sur Win + R

    Tape :

services.msc

    Cherche le service :

Services Bureau à distance

État attendu :

    En cours d’exécution

    Type de démarrage : Automatique

Vérifier en ligne de commande

Dans PowerShell ou cmd :

sc query TermService

Résultat attendu :

STATE : RUNNING

Autres services liés au RDP

Tu peux aussi voir :
Nom français	Nom service
Services Bureau à distance	TermService
Configuration des Services Bureau à distance	SessionEnv
Redirection de ports du mode utilisateur des Services Bureau à distance	UmRdpService

✅ Si tu veux, je peux aussi te donner la commande unique pour diagnostiquer pourquoi RDP refuse une connexion (ça révèle souvent le problème firewall ou NLA en 5 secondes).

ya pas de service, Services Bureau à distance

Si “Services Bureau à distance” n’apparaît pas dans services.msc sur Windows 10, ce n’est pas normal sur Windows 10 Pro. Le service interne TermService doit toujours exister. On va vérifier plusieurs choses.
1️⃣ Vérifier directement le service par commande

Ouvre PowerShell en administrateur :

Get-Service TermService

Résultat possible :

    ✅ Running ou Stopped → le service existe

    ❌ service not found → problème système

Tu peux aussi tester :

sc query TermService

2️⃣ Vérifier l’édition exacte de Windows

Même si tu penses avoir Pro :

winver

ou :

systeminfo | findstr /B /C:"OS Name"

Éditions possibles :

    ✔️ Windows 10 Pro → RDP serveur disponible

    ❌ Windows 10 Home → pas de service RDP serveur

3️⃣ Vérifier dans le registre si RDP est désactivé

Commande :

reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections

Résultat :

fDenyTSConnections    REG_DWORD    0x0

    0 → RDP autorisé

    1 → RDP bloqué

Pour activer :

reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f

4️⃣ Vérifier si le port 3389 écoute

Commande :

netstat -ano | findstr 3389

Si rien apparaît → le service RDP ne tourne pas.
5️⃣ Vérifier si Windows est une version modifiée

Certaines VM ou ISO custom suppriment RDP :

    Tiny10

    Windows Lite

    images entreprise modifiées

Dans ce cas TermService peut être supprimé.

✅ Test rapide à faire maintenant :

Dans PowerShell :

Get-Service *term*

et dis-moi ce que ça affiche.

💡 Et aussi dis-moi avec quoi tourne ta VM :

    VirtualBox

    VMware Workstation

    Proxmox VE

    KVM

Parce qu’il existe un cas très fréquent en VM où RDP est activé mais aucune connexion n’arrive, et c’est souvent le réseau NAT.

non il est running mais dans le logiciel service ya pas 

Si TermService est RUNNING mais que tu ne vois pas “Services Bureau à distance” dans services.msc sur Windows 10, il y a plusieurs raisons possibles — la plus fréquente est simplement le nom différent ou l’affichage trié.

Voici comment vérifier 👇
1️⃣ Chercher par nom du service (plus fiable)

Dans services.msc :

    Clique dans la liste

    Appuie sur T

    Cherche :

Services Bureau à distance

Nom interne :

TermService

Windows affiche parfois seulement le nom localisé, pas le nom technique.
2️⃣ Ouvrir directement le service

Appuie sur Win + R et tape :

services.msc /service=TermService

Cela ouvre directement le service.
3️⃣ Vérifier les infos du service

Dans PowerShell :

Get-Service TermService | Format-List *

Tu verras :

    DisplayName

    Status

    StartType

4️⃣ Vérifier que RDP écoute bien

Toujours dans PowerShell :

netstat -ano | findstr 3389

Résultat attendu :

0.0.0.0:3389 LISTENING

5️⃣ Test local sur Windows

Test-NetConnection localhost -Port 3389

Résultat attendu :

TcpTestSucceeded : True

✅ Important :

Si :

    TermService = running

    3389 = LISTENING

    ping OK

alors le problème vient presque toujours de :

    firewall Windows

    NLA (Network Level Authentication)

    configuration réseau de la VM

    client RDP Linux (ex : Remmina)

💡 Si tu veux, dis-moi juste :

    résultat de

    netstat -ano | findstr 3389

    message exact d’erreur dans Remmina

et je peux identifier le problème RDP en général en 1 commande (il y a un test très rapide).

ya tout qui est bon mais le netstat ya pas le port 3389 qi saffich e

Ah, là on a trouvé le problème clé 😅

Si TermService est RUNNING mais que netstat ne montre pas le port 3389, ça veut dire que le serveur RDP ne s’est pas lié à un port réseau. Même si RDP est activé dans les paramètres, Windows ne l’écoute pas pour une raison précise. Sur Windows 10 Pro, les causes les plus fréquentes sont :
1️⃣ Vérifier le paramètre “Autoriser les connexions à distance”

Ouvre PowerShell en admin et tape :

reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections

    0x0 → RDP activé

    0x1 → RDP bloqué

Si ce n’est pas 0, active-le :

reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f

2️⃣ Vérifier que NLA ne bloque pas le bind

Si “Autoriser uniquement les connexions avec NLA” est activé, certains clients (ou VM) peuvent empêcher l’écoute du port.

Pour désactiver temporairement NLA :

reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v UserAuthentication /t REG_DWORD /d 0 /f

Puis redémarre le service RDP :

net stop TermService /y
net start TermService

3️⃣ Vérifier si un autre logiciel bloque le port

    Antivirus ou solution de sécurité (ex: Sophos, ESET, Avast)

    Pare-feu tiers

    Certaines VM peuvent filtrer le port même si le firewall Windows est ouvert

4️⃣ Vérifier si la clé de registre du port RDP est correcte

reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v PortNumber

    Valeur attendue : 3389

    Si autre chose, remets 3389 :

reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v PortNumber /t REG_DWORD /d 3389 /f

Redémarre ensuite le service TermService.

💡 Après ça, refais un :

netstat -ano | findstr 3389

Tu devrais voir maintenant quelque chose comme :

TCP    0.0.0.0:3389     0.0.0.0:0     LISTENING

Si tu veux, je peux te donner la commande PowerShell unique qui corrige tous les paramètres RDP et NLA pour que le port 3389 écoute immédiatement sur Windows 10 Pro.

Veux‑tu que je fasse ça ?

jai 0x0 → RDP activé

Parfait, ça confirme que RDP est activé, donc le problème vient probablement de l’étape “bind du port 3389”.

Si netstat ne montre rien sur le port 3389, même avec TermService en RUNNING et fDenyTSConnections = 0, les causes typiques sont :
1️⃣ NLA (Network Level Authentication) bloque le bind

Même si RDP est activé, Windows ne démarre parfois pas l’écoute si NLA est activé mais pas supporté par la configuration réseau ou la VM.

Pour désactiver temporairement NLA :

    Ouvre PowerShell en administrateur

    Tape :

reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v UserAuthentication /t REG_DWORD /d 0 /f

    Redémarre le service RDP :

net stop TermService /y
net start TermService

    Vérifie ensuite :

netstat -ano | findstr 3389

2️⃣ Vérifier que le port RDP est correct

Dans PowerShell :

reg query "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v PortNumber

    Doit être 3389.

    Si autre chose, remettre 3389 (comme expliqué avant) et redémarrer TermService.

3️⃣ Vérifier les conflits logiciels

Certains antivirus ou suites de sécurité peuvent empêcher TermService de lier le port 3389.

    Antivirus (ESET, Avast, Sophos)

    Pare-feu tiers

Désactive temporairement pour tester.

💡 Prochain test rapide :

Après avoir désactivé NLA et redémarré TermService, fais sur la VM :

netstat -ano | findstr 3389

Si tu vois 0.0.0.0:3389 LISTENING, alors le problème avec Remmina va disparaître.

Si tu veux, je peux te donner une commande PowerShell unique qui force RDP à écouter sur toutes les interfaces et désactive NLA, c’est très rapide.

Veux‑tu que je fasse ça ?

dé crit rapidement l'utilité de Ansible et son fonctionnement pr que je mette ça dans ma doc technique 

Voici une version courte, claire et exploitable en doc technique 👇
🔧 Ansible – Utilité et fonctionnement
📌 Présentation

Ansible est un outil d’automatisation IT permettant de gérer la configuration, le déploiement d’applications et l’orchestration de systèmes à grande échelle.
🎯 Utilité

Ansible permet de :

    Automatiser l’installation de logiciels et la configuration des systèmes

    Déployer des applications de manière reproductible

    Gérer des infrastructures (serveurs, VM, cloud)

    Éviter les erreurs humaines grâce à des configurations déclaratives

👉 Il est souvent utilisé pour du DevOps, de la gestion de parc ou du provisionnement de machines.
⚙️ Fonctionnement

Ansible fonctionne selon un modèle agentless :

    Pas d’agent à installer sur les machines cibles

    Communication via SSH (Linux) ou WinRM (Windows)

Éléments clés :

    Inventory : liste des machines à gérer

    Playbooks : fichiers YAML décrivant les actions à exécuter

    Modules : scripts exécutant les tâches (ex: installer un paquet, copier un fichier)

🔄 Principe

    L’utilisateur lance un playbook depuis un poste de contrôle

    Ansible se connecte aux machines cibles

    Il exécute les tâches dans l’ordre défini

    Chaque tâche est idempotente (ne change rien si l’état est déjà correct)

✅ Avantages

    Simple (YAML lisible)

    Sans agent

    Idempotent

    Rapide à déployer

Si tu veux, je peux te faire une version encore plus “projet / rapport de stage” ou ajouter un schéma simple 👍

un ptit pavé redigé 

Ansible est un outil d’automatisation IT utilisé pour gérer la configuration des systèmes, déployer des applications et orchestrer des infrastructures de manière centralisée. Il permet d’automatiser des tâches répétitives sur un ensemble de machines, tout en garantissant une configuration cohérente et reproductible. Son fonctionnement repose sur un modèle sans agent (agentless) : aucune installation n’est nécessaire sur les machines cibles, car la communication s’effectue via des protocoles standards comme SSH pour les systèmes Linux ou WinRM pour Windows.

L’utilisateur définit les actions à exécuter dans des fichiers appelés playbooks, rédigés en YAML, qui décrivent l’état souhaité du système (installation de paquets, configuration de services, copie de fichiers, etc.). Ansible exécute ensuite ces tâches dans l’ordre défini sur les machines listées dans un inventaire. Grâce à son principe d’idempotence, il vérifie l’état actuel du système et n’applique des modifications que si nécessaire, ce qui évite les erreurs et garantit la stabilité des déploiements. Ainsi, Ansible est largement utilisé dans les environnements DevOps pour automatiser et fiabiliser la gestion des infrastructures.

installer le module ssl sur une vm linux

Pour installer le support SSL sur une VM Linux, ça dépend surtout du service que tu utilises (souvent un serveur web comme Apache HTTP Server ou Nginx). Voici une version claire pour ta doc 👇
🔐 Installation du support SSL sur Linux

L’installation du SSL consiste à activer le chiffrement HTTPS sur un service, généralement un serveur web.
📦 1. Installer les paquets nécessaires
Sur Debian / Ubuntu :

sudo apt update
sudo apt install openssl

Pour Apache :

sudo apt install apache2
sudo a2enmod ssl

Sur CentOS / RHEL :

sudo yum install openssl mod_ssl

⚙️ 2. Générer un certificat SSL (auto-signé)

openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
-keyout /etc/ssl/private/server.key \
-out /etc/ssl/certs/server.crt

🌐 3. Configurer le serveur web
Apache :

Activer le site SSL :

sudo a2ensite default-ssl
sudo systemctl restart apache2

Vérifier dans le fichier :

/etc/apache2/sites-available/default-ssl.conf

Chemins :

SSLCertificateFile
SSLCertificateKeyFile

🔍 4. Vérification

Tester :

curl -k https://localhost

Ou via navigateur :

https://IP_DE_LA_VM

✅ Résultat

    Le serveur écoute sur le port 443 (HTTPS)

    Les communications sont chiffrées en SSL/TLS

💡 À savoir :

    Un certificat auto-signé provoque une alerte navigateur (normal)

    En production, on utilise Let's Encrypt pour des certificats gratuits et valides

Si tu veux, je peux te faire une version ultra courte (5 lignes) ou adaptée à ton cas précis (Apache, Nginx, ou autre service).

corrige  : # Mise en place d'une Autorité de Certification Interne

## Objectif :
----------

bla bla bla

## 1 Mise à jour et installation

Sur CHA-CA :

sudo apt update && sudo apt upgrade -y
sudo apt install openssl



--

## 2. Création de l'autorité de certification (CA interne)

Sur CHA-CA uniquement :

### 2.1 Créer la structure de la CA dans /etc/ssl/ca

sudo mkdir -p /home/certificat
sudo chmod 700 /home/certificat



### 2.2 Génération de la clé privée de la CA

sudo openssl genrsa -out /home/certificat/ca.key.pem 4096
sudo chmod 400 /home/certificat/ca.key.pem



⚠️ **Il y a une PassPhrase : le mdp de la VM à voir dans le Keepass**

### 2.3 Création du certificat racine auto-signé

sudo openssl req -x509 -new -nodes \
  -key /home/certificat/ca.key.pem \
  -sha256 -days 3650 \
  -out /home/certificat/ca.cert.pem \
  -subj "/C=FR/ST=Centre-Val de Loire/L=Chartres/O=SportLudique/CN=CA-CHARTRES"

sudo chmod 444 /home/certificat/ca.cert.pem



---

## 4. Signature du certificat côté CA

Sur CHA-CA uniquement :

### 4.1 Copier la CSR dans le dossier CA

sudo mv /tmp/www.chartres.sportludique.fr.csr.pem /home/certificat

Voici une version corrigée, propre et cohérente pour ta doc technique 👇
Mise en place d'une Autorité de Certification Interne
Objectif

Mettre en place une autorité de certification interne (CA) afin de signer des certificats SSL pour les services internes (sites web, applications, etc.).
1. Mise à jour et installation

Sur CHA-CA :

sudo apt update && sudo apt upgrade -y
sudo apt install openssl -y

2. Création de l'autorité de certification (CA interne)

À réaliser uniquement sur CHA-CA.
2.1 Création de la structure de la CA

sudo mkdir -p /home/certificat
sudo chmod 700 /home/certificat

2.2 Génération de la clé privée de la CA

sudo openssl genrsa -out /home/certificat/ca.key.pem 4096
sudo chmod 400 /home/certificat/ca.key.pem

⚠️ Une passphrase est demandée : utiliser le mot de passe stocké dans Keepass.
2.3 Création du certificat racine auto-signé

sudo openssl req -x509 -new -nodes \
  -key /home/certificat/ca.key.pem \
  -sha256 -days 3650 \
  -out /home/certificat/ca.cert.pem \
  -subj "/C=FR/ST=Centre-Val de Loire/L=Chartres/O=SportLudique/CN=CA-CHARTRES"

sudo chmod 444 /home/certificat/ca.cert.pem

3. Signature d’un certificat côté CA

À réaliser uniquement sur CHA-CA.
3.1 Copier la CSR dans le dossier de la CA

sudo mv /tmp/www.chartres.sportludique.fr.csr.pem /home/certificat/

3.2 Signature du certificat

sudo openssl x509 -req \
  -in /home/certificat/www.chartres.sportludique.fr.csr.pem \
  -CA /home/certificat/ca.cert.pem \
  -CAkey /home/certificat/ca.key.pem \
  -CAcreateserial \
  -out /home/certificat/www.chartres.sportludique.fr.cert.pem \
  -days 365 -sha256

Résultat

    Certificat racine : /home/certificat/ca.cert.pem

    Clé privée CA : /home/certificat/ca.key.pem

    Certificat signé : /home/certificat/www.chartres.sportludique.fr.cert.pem

💡 Remarques :

    Le certificat racine doit être installé sur les machines clientes pour éviter les alertes de sécurité

    La clé privée de la CA doit rester strictement confidentielle

Si tu veux, je peux aussi te corriger la partie côté serveur web (Apache/Nginx) pour utiliser le certificat 👍

c quoi ca : 
### 3.3 Création du fichier d’extension SAN pour le serveur

sudo nano /home/certificat/www.chartres.sportludique.fr.ext

Contenu :
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = www.chartres.sportludique.fr
DNS.2 = chartres.sportludique.fr


C’est une très bonne question, et c’est une étape importante 👍
📌 À quoi sert ce fichier ?

Ce fichier est un fichier d’extensions OpenSSL utilisé lors de la signature du certificat.

Il permet d’ajouter des informations supplémentaires au certificat, notamment les SAN (Subject Alternative Names).
🔐 Pourquoi c’est nécessaire ?

Avant, seul le CN (Common Name) était utilisé :

CN=www.chartres.sportludique.fr

❌ Aujourd’hui, ce n’est plus suffisant
👉 Les navigateurs modernes exigent des SAN

Sans ça :

    certificat considéré comme invalide

    erreur HTTPS

🧩 Détail du fichier
Section principale

[ v3_req ]

➡️ Définit les extensions utilisées lors de la génération du certificat
Contraintes

basicConstraints = CA:FALSE

➡️ Indique que ce certificat est un certificat serveur, pas une autorité de certification
Usages de la clé

keyUsage = nonRepudiation, digitalSignature, keyEncipherment

➡️ Définit à quoi sert la clé :

    signature

    chiffrement

SAN (le plus important)

subjectAltName = @alt_names

➡️ Dit à OpenSSL d’aller lire la section [ alt_names ]
Noms de domaine autorisés

[ alt_names ]
DNS.1 = www.chartres.sportludique.fr
DNS.2 = chartres.sportludique.fr

➡️ Le certificat sera valide pour :

    www.chartres.sportludique.fr

    chartres.sportludique.fr

⚠️ Sans ce fichier

    Le certificat peut être généré

    MAIS :

        ❌ navigateur = erreur

        ❌ Remmina / clients stricts peuvent refuser

        ❌ HTTPS non conforme

🧠 Résumé simple (pour ta doc)

👉 Ce fichier permet d’ajouter les noms de domaine (SAN) au certificat afin qu’il soit reconnu comme valide par les navigateurs et les clients modernes.
💡 Bonus (important)

Quand tu signes ton certificat, il faut utiliser ce fichier :

openssl x509 -req \
-in fichier.csr \
-CA ca.cert.pem \
-CAkey ca.key.pem \
-CAcreateserial \
-out cert.pem \
-days 365 \
-sha256 \
-extfile www.chartres.sportludique.fr.ext \
-extensions v3_req

Si tu veux, je peux te faire une version ultra courte (2 lignes) pour ton rapport 👍

refait ca bien : # Mise en place HTTPS 

## Objectif :
----------

Mettre en place un site web sécurisé avec Apache2 utilisant
une autorité de certification interne (CA).

Le domaine utilisé est : www.chartres.sportludique.fr
La localité : Chartres
L'organisation : SportLudique

Serveurs utilisés :
------------------
1. CHA-CA  : VM Ubuntu Server 22.04 (Autorité de certification)
2. CHA-WEB : VM Ubuntu Server 22.04 (Serveur Apache2)

---

## 1. Préparation des serveurs

Sur CHA-WEB :

sudo apt update && sudo apt upgrade -y



Installer Apache2 et modules SSL :

sudo apt install apache2 -y
sudo a2enmod ssl
sudo a2enmod rewrite
sudo systemctl restart apache2



---

## 3. Création du certificat serveur web

Sur CHA-WEB uniquement :

### 3.1 Préparer le répertoire pour les certificats

sudo mkdir -p /etc/ssl/localcerts



### 3.2 Génération de la clé privée du serveur web

sudo openssl genrsa -out /etc/ssl/localcerts/www.chartres.sportludique.fr.key.pem 2048
sudo chmod 400 /etc/ssl/localcerts/www.chartres.sportludique.fr.key.pem



### 3.3 Génération de la CSR (Certificate Signing Request)

sudo openssl req -new \
  -key /etc/ssl/localcerts/www.chartres.sportludique.fr.key.pem \
  -out /etc/ssl/localcerts/www.chartres.sportludique.fr.csr.pem \
  -subj "/C=FR/ST=Centre-Val de Loire/L=Chartres/O=SportLudique/CN=www.chartres.sportludique.fr"



### 3.4 Transfert de la CSR vers la CA

scp /etc/ssl/localcerts/www.chartres.sportludique.fr.csr.pem \
    certificat@IP_CHA-CA:/tmp/



---

## 5. Configuration Apache sur CHA-WEB

### 5.1 Création du VirtualHost

sudo nano /etc/apache2/sites-available/www.chartres.sportludique.fr.conf



<VirtualHost *:80>
    ServerName www.chartres.sportludique.fr
    ServerAlias chartres.sportludique.fr
    Redirect permanent / https://www.chartres.sportludique.fr/
</VirtualHost>

<VirtualHost *:443>
    ServerName www.chartres.sportludique.fr
    DocumentRoot /var/www/www.chartres.sportludique.fr

    SSLEngine on
    SSLCertificateFile /etc/ssl/localcerts/www.chartres.sportludique.fr.cert.pem
    SSLCertificateKeyFile /etc/ssl/localcerts/www.chartres.sportludique.fr.key.pem
    SSLCACertificateFile /etc/ssl/localcerts/ca.cert.pem

    <Directory /var/www/www.chartres.sportludique.fr>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/www-chartres-error.log
    CustomLog ${APACHE_LOG_DIR}/www-chartres-access.log combined
</VirtualHost>



### 5.2 Organisation du site web

sudo mkdir -p /var/www/www.chartres.sportludique.fr
sudo nano /var/www/www.chartres.sportludique.fr/index.html
sudo chown -R www-data:www-data /var/www/www.chartres.sportludique.fr
sudo chmod -R 755 /var/www/www.chartres.sportludique.fr



### 5.3 Activation du site et vérification

sudo a2ensite www.chartres.sportludique.fr.conf
sudo a2dissite 000-default.conf
sudo apachectl configtest
Syntax OK → relancer Apache
sudo systemctl restart apache2



---

## 6. Test du certificat SSL

curl -Iv https://www.chartres.sportludique.fr --cacert /etc/ssl/localcerts/ca.cert.pem



Résultat attendu :

SSL certificate verify ok.
HTTP/1.1 200 OK



---

## 7. Astuce : Redirection automatique HTTP → HTTPS

Déjà incluse dans le VirtualHost *:80 avec la ligne :
Redirect permanent / https://www.chartres.sportludique.fr/


---

## 8. Vérification SAN (Subject Alternative Name)

openssl x509 -in /etc/ssl/localcerts/www.chartres.sportludique.fr.cert.pem \
    -noout -text | grep -A2 "Subject Alternative Name"



Si le SAN ne s’affiche pas :
- Vérifier le fichier d’extension : www.chartres.sportludique.fr.ext
- Regénérer le certificat avec l’option : -extfile

---

## 9. Déploiement sur client

- Copier le certificat CA ca.cert.pem sur le poste client
- L’ajouter comme certificat de confiance pour supprimer l’avertissement navigateur
- Exemple Linux : 
sudo cp ca.cert.pem /usr/local/share/ca-certificates/mon-ca.crt
sudo update-ca-certificates

---

Voici une version corrigée, structurée et propre pour une documentation technique. J’ai corrigé la numérotation, amélioré certaines formulations et uniformisé les sections.
Mise en place d’un site HTTPS avec Apache
Objectif

Mettre en place un site web sécurisé en HTTPS avec Apache HTTP Server en utilisant une autorité de certification interne (CA).

Paramètres utilisés :

    Domaine : www.chartres.sportludique.fr

    Localité : Chartres

    Organisation : SportLudique

Serveurs utilisés
Serveur	Rôle	OS
CHA-CA	Autorité de certification	Ubuntu Server 22.04
CHA-WEB	Serveur web Apache	Ubuntu Server 22.04
1. Préparation du serveur web

Sur CHA-WEB :

Mise à jour du système :

sudo apt update && sudo apt upgrade -y

Installation d’Apache et des modules nécessaires :

sudo apt install apache2 -y
sudo a2enmod ssl
sudo a2enmod rewrite
sudo systemctl restart apache2

2. Création du certificat serveur

À réaliser sur CHA-WEB uniquement.
2.1 Création du répertoire des certificats

sudo mkdir -p /etc/ssl/localcerts

2.2 Génération de la clé privée du serveur

sudo openssl genrsa -out /etc/ssl/localcerts/www.chartres.sportludique.fr.key.pem 2048
sudo chmod 400 /etc/ssl/localcerts/www.chartres.sportludique.fr.key.pem

2.3 Génération de la CSR (Certificate Signing Request)

sudo openssl req -new \
-key /etc/ssl/localcerts/www.chartres.sportludique.fr.key.pem \
-out /etc/ssl/localcerts/www.chartres.sportludique.fr.csr.pem \
-subj "/C=FR/ST=Centre-Val de Loire/L=Chartres/O=SportLudique/CN=www.chartres.sportludique.fr"

La CSR sera ensuite envoyée à la CA interne afin d’être signée.
2.4 Transfert de la CSR vers la CA

scp /etc/ssl/localcerts/www.chartres.sportludique.fr.csr.pem \
certificat@IP_CHA-CA:/tmp/

3. Installation du certificat signé

Une fois le certificat signé sur CHA-CA, copier les fichiers suivants sur CHA-WEB :

    certificat serveur

    certificat de la CA

Exemple :

scp certificat@IP_CHA-CA:/home/certificat/www.chartres.sportludique.fr.cert.pem /etc/ssl/localcerts/
scp certificat@IP_CHA-CA:/home/certificat/ca.cert.pem /etc/ssl/localcerts/

4. Configuration Apache
4.1 Création du VirtualHost

sudo nano /etc/apache2/sites-available/www.chartres.sportludique.fr.conf

Configuration :

<VirtualHost *:80>
    ServerName www.chartres.sportludique.fr
    ServerAlias chartres.sportludique.fr
    Redirect permanent / https://www.chartres.sportludique.fr/
</VirtualHost>

<VirtualHost *:443>

    ServerName www.chartres.sportludique.fr
    DocumentRoot /var/www/www.chartres.sportludique.fr

    SSLEngine on
    SSLCertificateFile /etc/ssl/localcerts/www.chartres.sportludique.fr.cert.pem
    SSLCertificateKeyFile /etc/ssl/localcerts/www.chartres.sportludique.fr.key.pem
    SSLCACertificateFile /etc/ssl/localcerts/ca.cert.pem

    <Directory /var/www/www.chartres.sportludique.fr>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/www-chartres-error.log
    CustomLog ${APACHE_LOG_DIR}/www-chartres-access.log combined

</VirtualHost>

5. Organisation du site web

Création du répertoire du site :

sudo mkdir -p /var/www/www.chartres.sportludique.fr

Création d’une page d’accueil :

sudo nano /var/www/www.chartres.sportludique.fr/index.html

Permissions :

sudo chown -R www-data:www-data /var/www/www.chartres.sportludique.fr
sudo chmod -R 755 /var/www/www.chartres.sportludique.fr

6. Activation du site

Activation du VirtualHost :

sudo a2ensite www.chartres.sportludique.fr.conf
sudo a2dissite 000-default.conf

Vérification de la configuration :

sudo apachectl configtest

Si Syntax OK, redémarrer Apache :

sudo systemctl restart apache2

7. Test du certificat SSL

Tester la connexion HTTPS :

curl -Iv https://www.chartres.sportludique.fr --cacert /etc/ssl/localcerts/ca.cert.pem

Résultat attendu :

SSL certificate verify ok
HTTP/1.1 200 OK

8. Vérification du SAN (Subject Alternative Name)

openssl x509 -in /etc/ssl/localcerts/www.chartres.sportludique.fr.cert.pem \
-noout -text | grep -A2 "Subject Alternative Name"

Si le SAN n’apparaît pas, vérifier :

    le fichier d’extension utilisé lors de la signature

    l’option -extfile dans la commande OpenSSL

9. Déploiement du certificat sur les clients

Pour éviter les alertes de sécurité dans les navigateurs, il faut installer le certificat racine de la CA sur les machines clientes.

- Copier le certificat CA `ca.cert.pem` sur le poste client
- L’ajouter comme certificat de confiance pour supprimer l’avertissement navigateur

- Exemple Linux : 
```
sudo cp ca.cert.pem /usr/local/share/ca-certificates/mon-ca.crt
sudo update-ca-certificates
```

---

✅ Le site est maintenant accessible en HTTPS sécurisé via :

https://www.chartres.sportludique.fr

---