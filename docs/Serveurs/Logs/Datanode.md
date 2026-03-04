# Installation et Configuration de Datanode

## Objectif :
----------

Data Node est un composant de l'architecture de Graylog conçue pour gérer l'ingestion, le traitement et l'indexation des journaux, pour stocker et interroger les journaux efficacement. Comme indiqué dans l'architecture de base, le service de nœud de données doit être maintenu sur son propre nœud, séparé de Graylog et MongoDB.<br>

---

## 1. Installation de Datanode

### 1.1 Installez le package Data Node :

```
wget https://packages.graylog2.org/repo/packages/graylog-7.0-repository_latest.deb
sudo dpkg -i graylog-7.0-repository_latest.deb
sudo apt-get update
sudo apt-get install graylog-datanode
```

### 1.2 Assurez-vous que le paramètre Linux vm.max_map_count est fixé à au moins 262144 :

Pour vérifier la valeur actuelle :

```
cat /proc/sys/vm/max_map_count
```

Pour augmenter la valeur :

```
echo 'vm.max_map_count=262144' | sudo tee -a /etc/sysctl.d/99-graylog-datanode.conf
sudo sysctl --system
cat /proc/sys/vm/max_map_count
```

### 1.3 Créez votre password_secret. Il s'agit d'une chaîne sécurisée générée aléatoirement utilisée pour chiffrer les données sensibles dans le système :

```
openssl rand -hex 32
```

### 1.4 Ouvrez le fichier de configuration du nœud de données :

```
sudo nano /etc/graylog/datanode/datanode.conf
```

Ensuite ajouter le **password_secret** valeur du fichier de configuration de Data Node. Notez que vous devez ajouter cette même valeur au **fichier de configuration du serveur Graylog** dans une étape ultérieure, car il est crucial que cette valeur soit **la même pour le nœud Graylog**.

### 1.5 Configurez la chaîne de connexion MongoDB dans le fichier de configuration de nœud de données. Dans l'exemple suivant de commande, graylog01 représente le nom d'hôte du nœud MongoDB :

```
mongodb_uri = mongodb://graylog01:27017/graylog
```

### 1.6 Activez le nœud de données et démarrez le service :

```
sudo systemctl daemon-reload
sudo systemctl enable graylog-datanode.service
sudo systemctl start graylog-datanode
```

## 2. Passer à la suite

Une fois que le Datanode est installer vous pouvez continuer l'installation de [Graylog](https://sym-0ne.github.io/sport-ludique-Chartres/Serveurs/Logs/Graylog/) à l'étape 2.