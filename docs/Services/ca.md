# Mise en place d'une Autorité de Certification Interne

## Objectif :
----------

Mettre en place une autorité de certification interne (CA) afin de générer et signer des certificats numériques utilisés pour sécuriser les communications au sein de l’infrastructure.

Cette autorité permet de délivrer des certificats SSL/TLS pour des services internes (serveurs web, applications, RDP, etc.) sans dépendre d’une autorité publique. Elle garantit l’authenticité des serveurs, le chiffrement des échanges et l’intégrité des données.

---

## 1 Mise à jour et installation

Sur CHA-CA :
```
sudo apt update && sudo apt upgrade -y
sudo apt install openssl -y
```

---

## 2. Création de l'autorité de certification (CA interne)

À réaliser uniquement sur CHA-CA.

### 2.1 Création de la structure de la CA

```
sudo mkdir -p /home/certificat
sudo chmod 700 /home/certificat
```

### 2.2 Génération de la clé privée de la CA

```
sudo openssl genrsa -out /home/certificat/ca.key.pem 4096
sudo chmod 400 /home/certificat/ca.key.pem
```

⚠️ Une **passphrase** est demandée : utiliser le mot de passe de la vm stocké dans Keepass.

### 2.3 Création du certificat racine auto-signé

```
sudo openssl req -x509 -new -nodes \
  -key /home/certificat/ca.key.pem \
  -sha256 -days 3650 \
  -out /home/certificat/ca.cert.pem \
  -subj "/C=FR/ST=Centre-Val de Loire/L=Chartres/O=SportLudique/CN=CA-CHARTRES"

sudo chmod 444 /home/certificat/ca.cert.pem
```

---

## 3. Signature d’un certificat côté CA

À réaliser uniquement sur CHA-CA.

### 3.1 Copier la CSR dans le dossier de la CA

```
sudo mv /tmp/www.chartres.sportludique.fr.csr.pem /home/certificat
```

### 3.2 Signature du certificat
✅ Le site est maintenant accessible en HTTPS sécurisé via :
```
sudo openssl x509 -req \
  -in /home/certificat/www.chartres.sportludique.fr.csr.pem \
  -CA /home/certificat/ca.cert.pem \
  -CAkey /home/certificat/ca.key.pem \
  -CAcreateserial \
  -out /home/certificat/www.chartres.sportludique.fr.cert.pem \
  -days 365 -sha256
```

### 3.3 Création du fichier d’extension SAN pour le serveur (Dans le cadre du certificat pour HTTPs)

```
sudo nano /home/certificat/www.chartres.sportludique.fr.ext

Contenu :
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = www.chartres.sportludique.fr
DNS.2 = chartres.sportludique.fr
```

### 4.3 Signature du certificat serveur

```
sudo openssl x509 -req \
  -in /home/certificat/www.chartres.sportludique.fr.csr.pem \
  -CA /home/certificat/ca.cert.pem \
  -CAkey /home/certificat/ca.key.pem \
  -CAcreateserial \
  -out /home/certificat/www.chartres.sportludique.fr.cert.pem \
  -days 825 -sha256 \
  -extfile /home/certificat/www.chartres.sportludique.fr.ext
```

### 4.4 Transfert des certificats vers CHA-WEB

```
scp /home/certificat/www.chartres.sportludique.fr.cert.pem \
    user@CHA-WEB:/etc/ssl/certs/
scp /home/certificat/ca.cert.pem \
    user@CHA-WEB:/etc/ssl/certs/
```

---