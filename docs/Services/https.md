==========================================================
# Tutoriel : Mise en place HTTPS avec CA interne
==========================================================

## Objectif :
----------
Mettre en place un site web sécurisé avec Apache2 utilisant
une autorité de certification interne (CA) sur Ubuntu Server
22.04. Le domaine utilisé est : www.chartres.sportludique.fr
La localité : Chartres
L'organisation : SportLudique

Serveurs utilisés :
------------------
1. CHA-CA  : VM Ubuntu Server 22.04 (Autorité de certification)
2. CHA-WEB : VM Ubuntu Server 22.04 (Serveur Apache2)

----------------------------------------------------------
## 1. Préparation des serveurs
---------------------------
Sur CHA-CA et CHA-WEB :
```
sudo apt update && sudo apt upgrade -y
```

Sur CHA-WEB uniquement, installer Apache2 et modules SSL :
```
sudo apt install apache2 -y
sudo a2enmod ssl
sudo a2enmod rewrite
sudo systemctl restart apache2
```

----------------------------------------------------------
## 2. Création de l'autorité de certification (CA interne)
----------------------------------------------------------
Sur CHA-CA uniquement :

### 1. Créer la structure de la CA dans /etc/ssl/ca
-------------------------------------------------
```
sudo mkdir -p /etc/ssl/ca/{certs,crl,newcerts,private}
sudo chmod 700 /etc/ssl/ca/private
sudo touch /etc/ssl/ca/index.txt
echo 1000 | sudo tee /etc/ssl/ca/serial
```

### 2. Génération de la clé privée de la CA
----------------------------------------
```
sudo openssl genrsa -out /etc/ssl/ca/private/ca.key.pem 4096
sudo chmod 400 /etc/ssl/ca/private/ca.key.pem
```

### 3. Création du certificat racine auto-signé
--------------------------------------------
```
sudo openssl req -x509 -new -nodes \
  -key /etc/ssl/ca/private/ca.key.pem \
  -sha256 -days 3650 \
  -out /etc/ssl/ca/certs/ca.cert.pem \
  -subj "/C=FR/ST=Centre-Val de Loire/L=Chartres/O=SportLudique/CN=CA-CHARTRES"

sudo chmod 444 /etc/ssl/ca/certs/ca.cert.pem
```

----------------------------------------------------------
## 3. Création du certificat serveur web
----------------------------------------------------------
Sur CHA-WEB uniquement :

### 1. Préparer le répertoire pour les certificats
------------------------------------------------
```
sudo mkdir -p /etc/ssl/localcerts
```

### 2. Génération de la clé privée du serveur web
-----------------------------------------------
```
sudo openssl genrsa -out /etc/ssl/localcerts/www.chartres.sportludique.fr.key.pem 2048
sudo chmod 400 /etc/ssl/localcerts/www.chartres.sportludique.fr.key.pem
```

### 3. Génération de la CSR (Certificate Signing Request)
------------------------------------------------------
```
sudo openssl req -new \
  -key /etc/ssl/localcerts/www.chartres.sportludique.fr.key.pem \
  -out /etc/ssl/localcerts/www.chartres.sportludique.fr.csr.pem \
  -subj "/C=FR/ST=Centre-Val de Loire/L=Chartres/O=SportLudique/CN=www.chartres.sportludique.fr"
```

### 4. Transfert de la CSR vers la CA
---------------------------------
```
scp /etc/ssl/localcerts/www.chartres.sportludique.fr.csr.pem \
    certificat@IP_CHA-CA:/tmp/
```

----------------------------------------------------------
## 4. Signature du certificat côté CA
----------------------------------------------------------
Sur CHA-CA uniquement :

### 1. Copier la CSR dans le dossier CA
------------------------------------
sudo mv /tmp/www.chartres.sportludique.fr.csr.pem /etc/ssl/ca/

### 2. Création du fichier d’extension SAN pour le serveur
------------------------------------------------------
```
sudo nano /etc/ssl/ca/www.chartres.sportludique.fr.ext

Contenu :
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = www.chartres.sportludique.fr
DNS.2 = chartres.sportludique.fr
```

### 3. Signature du certificat serveur
-----------------------------------
```
sudo openssl x509 -req \
  -in /etc/ssl/ca/www.chartres.sportludique.fr.csr.pem \
  -CA /etc/ssl/ca/certs/ca.cert.pem \
  -CAkey /etc/ssl/ca/private/ca.key.pem \
  -CAcreateserial \
  -out /etc/ssl/ca/certs/www.chartres.sportludique.fr.cert.pem \
  -days 825 -sha256 \
  -extfile /etc/ssl/ca/www.chartres.sportludique.fr.ext
```

### 4. Transfert des certificats vers CHA-WEB
-----------------------------------------
```
scp /etc/ssl/ca/certs/www.chartres.sportludique.fr.cert.pem \
    user@CHA-WEB:/etc/ssl/localcerts/
scp /etc/ssl/ca/certs/ca.cert.pem \
    user@CHA-WEB:/etc/ssl/localcerts/
```

----------------------------------------------------------
## 5. Configuration Apache sur CHA-WEB
----------------------------------------------------------
### 1. Création du VirtualHost
---------------------------
```
sudo nano /etc/apache2/sites-available/www.chartres.sportludique.fr.conf
```

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

### 2. Organisation du site web
----------------------------
```
sudo mkdir -p /var/www/www.chartres.sportludique.fr
sudo nano /var/www/www.chartres.sportludique.fr/index.html
sudo chown -R www-data:www-data /var/www/www.chartres.sportludique.fr
sudo chmod -R 755 /var/www/www.chartres.sportludique.fr
```

### 3. Activation du site et vérification
--------------------------------------
```
sudo a2ensite www.chartres.sportludique.fr.conf
sudo a2dissite 000-default.conf
sudo apachectl configtest
Syntax OK → relancer Apache
sudo systemctl restart apache2
```

----------------------------------------------------------
## 6. Test du certificat SSL
--------------------------
```
curl -Iv https://www.chartres.sportludique.fr --cacert /etc/ssl/localcerts/ca.cert.pem
```

Résultat attendu :
```
SSL certificate verify ok.
HTTP/1.1 200 OK
```

----------------------------------------------------------
## 7. Astuce : Redirection automatique HTTP → HTTPS
----------------------------------------------------------
Déjà incluse dans le VirtualHost *:80 avec la ligne :
Redirect permanent / https://www.chartres.sportludique.fr/

----------------------------------------------------------
## 8. Vérification SAN (Subject Alternative Name)
----------------------------------------------------------
```
openssl x509 -in /etc/ssl/localcerts/www.chartres.sportludique.fr.cert.pem \
    -noout -text | grep -A2 "Subject Alternative Name"
```

Si le SAN ne s’affiche pas :
- Vérifier le fichier d’extension : `www.chartres.sportludique.fr.ext`
- Regénérer le certificat avec l’option : `-extfile`

----------------------------------------------------------
## 9. Déploiement sur client
--------------------------
- Copier le certificat CA `ca.cert.pem` sur le poste client
- L’ajouter comme certificat de confiance pour supprimer l’avertissement navigateur
- Exemple Linux : 
sudo cp ca.cert.pem /usr/local/share/ca-certificates/mon-ca.crt
sudo update-ca-certificates

----------------------------------------------------------
## 10. Gestion des certificats et suppression
-------------------------------------------
Pour repartir proprement sur CHA-CA ou CHA-WEB :
sudo rm -rf /etc/ssl/ca /etc/ssl/localcerts
sudo mkdir -p /etc/ssl/ca /etc/ssl/localcerts

