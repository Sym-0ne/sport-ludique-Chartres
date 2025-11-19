# Configuration du site WordPress & Nom de domaine.

Site WordPress : ```www.cimmob.chartres.sportludique.fr```


## 1. Prérequis

- Serveur WordPress : ```172.28.62.3```
- Serveur Base de données : ```192.168.28.10```
- Reverse Proxy : ```172.28.62.5 (Nginx)```
- DNS interne : ```Bind9 (serveur autorité interne)```
- WordPress installé dans : ```/var/www/html/wordpress```

## 2. Configuration DNS (Bind9)

### 2.1 Zone **interne**: chartres.sportludique.fr
- Fichier : /etc/bind/zones/db.chartres.sportludique.fr.internal<br>
- Ajouter les enregistrements :
```
www      IN  A   172.28.62.5
cimmob   IN  A   172.28.62.5   
```

- Vérifier la syntaxe:
```
sudo named-checkzone chartres.sportludique.fr /etc/bind/zones/db.chartres.sportludique.fr.internal
```

- Recharger Bind9:
```
sudo systemctl reload bind9
```

---

### 2.1 Zone **externe** : chartres.sportludique.fr
- Fichier : /etc/bind/zones/db.chartres.sportludique.fr.external<br>
- Ajouter les enregistrements :
```
www      IN  A   183.44.28.1; IP de la FAI
cimmob   IN  A   183.44.28.1   ; IP de la FAI
```

- Vérifier la syntaxe :
```
sudo named-checkzone chartres.sportludique.fr /etc/bind/zones/db.chartres.sportludique.fr.external
```

- Recharger Bind9 :
```
sudo systemctl reload bind9
```

## 3. Configuration Reverse Proxy (Nginx)

### 3.1 Site client (déjà existant)
- Domaine : ```www.cimmob.chartres.sportludique.fr```
- Redirige le trafic vers WordPress

### 3.2 Site admin pour réseau management
- Fichier : /etc/nginx/sites-enabled/cimmob.chartres.sportludique.fr 
- Contenu :

```
server {
    listen 80;
    server_name www.cimmob.chartres.sportludique.fr;

    location / {
        proxy_pass http://192.168.28.20;   
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

```

- Activer le site:
```
sudo ln -s /etc/nginx/sites-enabled/cimmob.chartres.sportludique.fr  /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## 4. Configuration Apache sur serveur WordPress

- VirtualHost pour WordPress (/etc/apache2/sites-available/cimmob.conf):

```
<VirtualHost *:80>
    ServerName www.cimmob.chartres.sportludique.fr
    ServerAlias www.cimmob.chartres.sportludique.fr
    DocumentRoot /var/www/html/wordpress

    <Directory /var/www/html/wordpress>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/cimmob_error.log
    CustomLog ${APACHE_LOG_DIR}/cimmob_access.log combined
</VirtualHost>
```

- Redémarrer Apache:
```
sudo systemctl restart apache2
```

---

## 5. WordPress Configuration (wp-config.php)

- Fichier: /var/www/html/wordpress/wp-config.php
- Vérifier les lignes DB :

```
define( 'DB_NAME', 'wordpress_db' );
define( 'DB_USER', 'wp_user' );
define( 'DB_PASSWORD', 'CHABDD083' );
define( 'DB_HOST', '10.10.120.7' );
```

---

## 6. Vérifications

### 6.1 Depuis réseau client :
- URL : ```http://www.cimmob.chartres.sportludique.fr```
- Résultat: site accessible via reverse proxy

---

