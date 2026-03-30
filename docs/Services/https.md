# Configuration Certificat HTTPS

---

## Objectif :
----------

Mettre en place un site web sécurisé avec Apache2 utilisant une autorité de certification interne (CA).

Le domaine utilisé est : www.chartres.sportludique.fr
La localité : Chartres
L'organisation : SportLudique

Serveurs utilisés :
------------------
1. CHA-CA  : VM Ubuntu Server 22.04 (Autorité de certification)
2. CHA-WEB : VM Ubuntu Server 22.04 (Serveur Apache2)

---

## 1. Préparation du serveur web

Sur CHA-WEB :

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

***La CSR sera ensuite envoyée à la CA interne afin d’être signée.***

### 2.4 Transfert de la CSR vers la CA
```
scp /etc/ssl/localcerts/www.chartres.sportludique.fr.csr.pem \
certificat@IP_CHA-CA:/tmp/
```

---

## 3. Installation du certificat signé

Une fois le certificat signé sur CHA-CA, copier les fichiers suivants sur CHA-WEB :

- Certificat serveur
- Certificat de la CA

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

## 5. Organisation du site web

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

## 6. Activation du site

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

Tester la connexion HTTPS :

```
curl -Iv https://www.chartres.sportludique.fr --cacert /etc/ssl/localcerts/ca.cert.pem
```

Résultat attendu :

```
SSL certificate verify ok
HTTP/1.1 200 OK
```

---

## 8. Vérification du SAN (Subject Alternative Name)

```
openssl x509 -in /etc/ssl/localcerts/www.chartres.sportludique.fr.cert.pem \
-noout -text | grep -A2 "Subject Alternative Name"
```

Si le SAN n’apparaît pas, vérifier :

- Le fichier d’extension utilisé lors de la signature
- L’option -extfile dans la commande OpenSSL

---

## 9. Déploiement du certificat sur les clients

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