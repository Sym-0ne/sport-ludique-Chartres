# Reverse Proxy Secondaire

## 1. Installation de Nginx

Vous pouvez aller voir la documention pour l'[installation](https://sym-0ne.github.io/sport-ludique-Chartres/Services/reverse-proxy).

## 2. Synchronisation des 2 Reverse Proxy

### 2.1 Création de la clé SSH sur le Reverse Proxy Primaire

```
ssh-keygen -t ed25519
```

La clé se créee dans le repertoire **/root/.ssh/id_ed25519**.

Elle servira pour la connexion automatique vers B.

### 2.2 Copie de la clé publique vers le Reverse Proxy Secondaire

```
ssh-copy-id -i /root/.ssh/id_ed25519.pub root@172.28.62.10
```

Comme sa le Reverse Proxy Primaire pourra se connecter en **SSH sans mot de passe** vers le Reverse Proxy Secondaire.

### 2.3 Installation de rsync et lsyncd

Il faut les installer sur les deux Reverse Proxy.

```
sudo apt install rsync lsyncd
```

### 2.4 Configuration de lsyncd

```
settings {
    logfile    = "/var/log/lsyncd.log",
    statusFile = "/var/log/lsyncd.status",
    statusInterval = 20,
}

sync {
    default.rsync,
    source = "/etc/nginx/",
    target = "root@172.28.62.10:/etc/nginx/",
    rsync = {
        archive = true,
        compress = false,
        verbose = true,
    }
}
```
Puis redémarrer le service :

```
systemctl restart lsyncd
systemctl enable lsyncd
```

### 2.5 Vérification

Vérifié les logs pour voir si il n'y a pas d'erreur :

```
tail -f /var/log/lsyncd.log
```

Si il n'y a rien et qu'il y a se message "Normal: Startup of /etc/nginx -> root@172.28.62.10:/etc/nginx/ finished.", alors la synchronisation est opérationnelle.

Sinon il faut regarder le message d'erreur et le résoudre.

## 3. Test 

Sur le Reverse Proxy Primaire :

```
sudo touch /etc/nginx/test-sync.txt
```

Puis sur le Reverse Proxy Secondaire :

```
ls -l /etc/nginx/
```

Si le fichier "test-sync.txt" y est alors la synchronisation est fonctionnelle.

## 4. Important

Si il y a des sites en https il ne faudra pas oublier d'importer les certificats du site car la synchronisation ne le gère pas.

Pour savoir comment faire voir [ici](https://sym-0ne.github.io/sport-ludique-Chartres/Services/reverse-proxy).

