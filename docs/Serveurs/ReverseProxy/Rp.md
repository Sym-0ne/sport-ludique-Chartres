# Installation & Configuration d'un Reverse Proxy

## Objectif :
----------

Un reverse proxy est un serveur intermédiaire positionné devant un ou plusieurs serveurs d'application. Il reçoit les requêtes des clients (navigateurs web) et les transmet au serveur interne approprié, puis retourne la réponse du serveur au client, agissant comme le point d'entrée unique.

---

## 1. Installation de Nginx

Sur Debian/Ubuntu :

```
sudo apt update
sudo apt install nginx
```

---

## 2. Activation de la configuration

Créez le fichier :

```
sudo nano /etc/nginx/sites-available/chartres.sportludique.fr
```

Mettre cette conf pour un accès en http :

```
server {
    listen 80;
    server_name www.chartres.sportludique.fr;

    location / {
        proxy_pass http://172.28.62.3;   
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Activez-le :

```
sudo ln -s /etc/nginx/sites-available/chartres.sportludique.fr /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```
Puis vérifier si depuis l'interne et l'externe le site est accessible.

---

## 3. Mise en place des certificats pour HTTPS

Il faut importer **le certificat autorité, le certificat auto-signé et la clé privée qui sont lié au nom de domaine du site**. Les certificats se situe sur la vm Openssl :

```
scp www.chartres.sportludique.fr.cert.pem user@10.10.120.80:/tmp/
scp ca.cert.pem user@10.10.120.80:/tmp/
scp www.chartres.sportludique.fr.key.pem user@10.10.120.80:/tmp/
```
Puis il faut les **déplacer au bon endroit dans la vm** et leur accorder des droits :

```
sudo mv /tmp/www.chartres.sportludique.fr.cert.pem /etc/ssl/certs/
sudo mv /tmp/ca.cert.pem /etc/ssl/certs/
sudo mv /tmp/www.chartres.sportludique.fr.key.pem /etc/ssl/private/

sudo chmod 644 /etc/ssl/certs/www.chartres.sportludique.fr.cert.pem
sudo chmod 644 /etc/ssl/certs/ca.cert.pem
sudo chmod 600 /etc/ssl/private/www.chartres.sportludique.fr.key.pem
sudo chown root:root /etc/ssl/certs/www.chartres.sportludique.fr.cert.pem
sudo chown root:root /etc/ssl/certs/ca.cert.pem
sudo chown root:root /etc/ssl/private/www.chartres.sportludique.fr.key.pem
```

Ensuite il faut vérifier que les fichier "www.chartres.sportludique.fr.cert.pem" et "www.chartres.sportludique.fr.key.pem" soit **bien liée** :

```
openssl x509 -noout -modulus -in /etc/ssl/certs/chartres.sportludique.fr.crt.pem | openssl md5
openssl rsa -noout -modulus -in /etc/ssl/private/chartres.sportludique.fr.key.pem | openssl md5
```

Si ils ont le même hash alors les fichier sont bien liée, sinon il faut recommencer toute l'étape car il faut **absolument qu'ils soit liée sinon le HTTPS ne fonctionnera jamais**.

---

## 4. Combiner certificat autorité + auto-signé dans le même fichier

Nginx ne permet pas dans sa configuration d'aller **chercher 2 certificats**, il faut donc assembler les deux dans un seul et même fichier:

```
sudo -i

cat /etc/ssl/certs/www.chartres.sportludique.fr.cert.pem /etc/ssl/certs/ca.cert.pem > /etc/ssl/certs/www.chartres.sportludique.fr.fullchain.pem
```

---

## 5. Configuration HTTPS

Modifié la conf pour que le site fonctionne en HTTPS:

```
sudo nano /etc/nginx/sites-available/chartres.sportludique.fr
```

```
server {
    listen 80;
    server_name www.chartres.sportludique.fr;

    # Redirection HTTP → HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name www.chartres.sportludique.fr;

    # Certificat SSL du reverse proxy (public)
    ssl_certificate /etc/ssl/certs/www.chartres.sportludique.fr.fullchain.pem;
    ssl_certificate_key /etc/ssl/private/www.chartres.sportludique.fr.key.pem;

    # (Optionnel) paramètres SSL recommandés
    proxy_ssl_protocols TLSv1.2 TLSv1.3;
    proxy_ssl_ciphers HIGH:!aNULL:!MD5;

    location / {
        # Redirection vers ton site interne HTTPS (ta VM)
        proxy_pass https://172.28.62.3;

        proxy_ssl_server_name on;
        proxy_ssl_verify off;  # si ton certificat interne n’est pas signé par une autorité publique

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
    }
}
```

---

## 6. Vérification

Vérifier qu'il n'y a pas d'erreur et redémarrer le service:

```
sudo nginx -t
sudo systemctl restart nginx
```

Puis vérifier si depuis l'interne et l'externe le site est accessible.

---