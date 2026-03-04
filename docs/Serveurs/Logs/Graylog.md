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

### 2.2 Utilisez la commande suivante pour créer votre "root_password_sha2". Ceci est le mot de passe du compte administrateur root pour l'interface Graylog :

```
echo -n "Enter Password: " && head -1 </dev/stdin | tr -d '\n' | sha256sum | cut -d" " -f1
```

"Attention" : N'essayez pas de vous connecter à Graylog pour la première fois en utilisant le mot de passe que vous avez généré dans les étapes ci-dessus! Vous devez remplir la connexion prévol avec les informations d'identification trouvées dans le fichier journal après avoir démarré le service Graylog.

### 2.3 Ouvrez le fichier de configuration du serveur Graylog :

```
sudo nano /etc/graylog/server/server.conf
```


