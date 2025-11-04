
Mise en place d’un serveur Apache sécurisé avec une Autorité de Certification interne (CA)
🧩 Objectif

Mettre en place un site web sécurisé en HTTPS avec Apache2 en utilisant une autorité de certification interne (CA) hébergée sur une machine virtuelle Ubuntu Server 22.04 LTS.
L’objectif est de pouvoir accéder localement au site via https://www.chartres.sportludique.fr

1. Préparation des serveurs
✅ Mise à jour du système

Sur les deux serveurs (CHA-CA et CHA-WEB) :
```
sudo apt update && sudo apt upgrade -y
```

Installation d’Apache sur le serveur web uniquement

Sur CHA-WEB :
```
sudo apt install apache2 -y
sudo a2enmod ssl
sudo a2enmod rewrite
sudo systemctl restart apache2
```

2. Création de l’autorité de certification (CA interne)

⚙️ À faire uniquement sur la VM “CHA-CA” (Ubuntu Server 22.04).

📁 Création de la structure CA
```
mkdir -p /root/ca/{certs,crl,newcerts,private}
chmod 700 /root/ca/private
touch /root/ca/index.txt
echo 1000 > /root/ca/serial
```

🔑 Génération de la clé privée de la CA
```
openssl genrsa -out /root/ca/private/ca.key.pem 4096
chmod 400 /root/ca/private/ca.key.pem
```

🪪 Création du certificat racine auto-signé
```
openssl req -x509 -new -nodes -key /root/ca/private/ca.key.pem \
  -sha256 -days 3650 -out /root/ca/certs/ca.cert.pem \
  -subj "/C=FR/ST=Centre-Val de Loire/L=Chartres/O=SportLudique/CN=CA-Interne"
```

```
chmod 444 /root/ca/certs/ca.cert.pem
```

✅ Ce certificat ca.cert.pem sera utilisé pour signer les certificats des serveurs web.

📜 3. Génération du certificat pour le serveur web
⚙️ Cette partie se fait sur le serveur web CHA-WEB.

🔑 Création de la clé privée
```
sudo mkdir -p /etc/ssl/localcerts
sudo openssl genrsa -out /etc/ssl/localcerts/www.chartres.sportludique.fr.key.pem 2048
sudo chmod 400 /etc/ssl/localcerts/www.chartres.sportludique.fr.key.pem
```

🧠 Création de la demande de signature (CSR)
```
sudo openssl req -new -key /etc/ssl/localcerts/www.chartres.sportludique.fr.key.pem \
  -out /etc/ssl/localcerts/www.chartres.sportludique.fr.csr.pem \
  -subj "/C=FR/ST=Centre-Val de Loire/L=Chartres/O=SportLudique/CN=www.chartres.sportludique.fr"

```

📦 Envoi du CSR vers la CA interne
```
scp /etc/ssl/localcerts/www.chartres.sportludique.fr.csr.pem certificat@10.10.120.12:/root/ca/
```

🪙 4. Signature du certificat sur la CA interne
```
openssl ca -in /root/ca/www.chartres.sportludique.fr.csr.pem \
  -out /root/ca/certs/www.chartres.sportludique.fr.cert.pem \
  -days 825 -notext -md sha256

```

Puis transfère les certificats vers le serveur web :

```
scp /root/ca/certs/www.chartres.sportludique.fr.cert.pem user@10.10.120.11:/etc/ssl/localcerts/
scp /root/ca/certs/ca.cert.pem user@10.10.120.11:/etc/ssl/localcerts/
```

🌐 5. Configuration du site Apache web.local
Sur CHA-WEB :

📝 Création du VirtualHost
```
sudo nano /etc/apache2/sites-available/www.chartres.sportludique.fr.conf
```

Contenu :
```
<VirtualHost *:80>
    ServerName www.chartres.sportludique.fr
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

    ErrorLog ${APACHE_LOG_DIR}/www.chartres-error.log
    CustomLog ${APACHE_LOG_DIR}/www.chartres-access.log combined
</VirtualHost>

```

📂 6. Organisation du site web
```
sudo mkdir -p /var/www/www.chartres.sportludique.fr
sudo mv /var/www/html/index.html /var/www/www.chartres.sportludique.fr/
sudo chown -R www-data:www-data /var/www/www.chartres.sportludique.fr
sudo chmod -R 755 /var/www/www.chartres.sportludique.fr

```

⚙️ 7. Activation du site et vérification
```
sudo a2ensite www.chartres.sportludique.fr.conf
sudo a2dissite 000-default.conf
sudo apachectl configtest
```

✅ Si le message est :
```
Syntax OK
```

Alors relance Apache :
```
sudo systemctl reload apache2
```

🧪 8. Test du certificat SSL
🔍 Test depuis le serveur web
```
curl -Iv https://www.chartres.sportludique.fr --cacert /etc/ssl/localcerts/ca.cert.pem
```

Résultat attendu :
```
SSL certificate verify ok.
HTTP/1.1 200 OK
```
