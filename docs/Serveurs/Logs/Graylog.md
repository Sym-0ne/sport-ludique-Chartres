# Installation et Configuration de Graylog

## Objectif :
----------

Le serveur permet de centraliser la collecte et la visualisation des logs, d’identifier rapidement les problèmes et de suivre les activités de toutes les VMs, switch, routeur et pare-feu.<br>

---

## 1. Installation de MongoDB

### 1.1 Installer gnupg et curl :

```
sudo apt-get install gnupg curl
```

### 1.2 Importer la clé publique MongoDB :

```
curl -fsSL https://www.mongodb.org/static/pgp/server-8.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg \
   --dearmor
```

### 1.3 Créer un fichier de liste pour MongoDB :

```
echo "deb [ signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] http://repo.mongodb.org/apt/debian bookworm/mongodb-org/8.0 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
```

### 1.4 Recharger la base de données de paquets local :

```
sudo apt-get update
```

### 1.5 Installez la dernière version stable de MongoDB :

```
sudo apt-get install -y mongodb-org
```

### 1.6 Maintenez la version actuellement installée du package MongoDB pour éviter qu'il ne soit automatiquement mis à niveau vers une version plus récente lorsque les mises à jour sont installées :

```
sudo apt-mark hold mongodb-org
```

### 1.7 Ouvrez le fichier de configuration MongoDB :

```
sudo nano /etc/mongod.conf
```

Puis modifier le bindIp pour se lier à une interface spécifique, comme 192.168.50.71:

```
net:
  port: 27017
  bindIp: 192.168.50.71
```

### 1.8 Enfin activez MongoDB et démarrez le service :

```
sudo systemctl daemon-reload
sudo systemctl enable mongod.service
sudo systemctl start mongod.service
```

# Important

Avant de passer à l'étape 2 il faut installer le serveur [Datanode](https://sym-0ne.github.io/sport-ludique-Chartres/Serveurs/Logs/Datanode/).


## 2. Installation de Graylog

### 2.1 Sur votre nœud Graylog/MongoDB, installez le package Graylog :

```
sudo apt-get install graylog-server
```

### 2.2 Utilisez la commande suivante pour créer votre root_password_sha2. Ceci est le mot de passe du compte administrateur root pour l'interface Graylog :

```
echo -n "Enter Password: " && head -1 </dev/stdin | tr -d '\n' | sha256sum | cut -d" " -f1
```

**Attention** : N'essayez pas de vous connecter à Graylog pour la première fois en utilisant le mot de passe que vous avez généré dans les étapes ci-dessus! Vous devez remplir la connexion prévol avec les informations d'identification trouvées dans le fichier journal après avoir démarré le service Graylog.

### 2.3 Ouvrez le fichier de configuration du serveur Graylog :

```
sudo nano /etc/graylog/server/server.conf
```

Ensuite entrez la valeur que vous avez créée pour le **root_password_sha2** à l'étape précédente dans le fichier de configuration Graylog.

Après récupérez le secret du mot de passe à partir du fichier de configuration de **nœud de données** comme indiqué aux étapes 3 à 5 de **Installer le nœud de données** et l'ajouter au **password_secret** propriété dans le fichier de configuration Graylog.

### 2.4 Définir le http_bind_address valeur dans le fichier de configuration Graylog au nom de l'hôte public ou à une adresse IP publique sur laquelle le serveur Web écoute les requêtes HTTP entrantes :

```
http_bind_address = 0.0.0.0:9000
```

### 2.5 Ajustez les paramètres de votre journal :

```
message_journal_max_age = 72h
message_journal_max_size = 90gb
```

### 2.6 Ouvrir le graylog-server fichier trouvé par défaut à /etc/default/ :

```
sudo nano /etc/default/graylog-server
```

### 2.7 Dans le graylog-server fichier, ajustez vos paramètres de tas à la moitié de la mémoire système jusqu'à un maximum de 16 Go pour le service Graylog :

```
GRAYLOG_SERVER_JAVA_OPTS="-Xms2g -Xmx2g -server -XX:+UseG1GC -XX:-OmitStackTraceInFastThrow"
```

### 2.8 Activez Graylog et démarrez le service :

```
sudo systemctl daemon-reload
sudo systemctl enable graylog-server.service
sudo systemctl start graylog-server.service
```

## 3. Connexion à l'interface Web

Une fois l'installation terminée, passez immédiatement à [l'interface Web](https://sym-0ne.github.io/sport-ludique-Chartres/Serveurs/Logs/InterfaceWebLog/) pour plus d'informations sur la réalisation du processus de prévol et la connexion à Graylog pour la première fois.