# Configuration FreeRADIUS — WPA2-Enterprise (PEAP/MSCHAPv2 + OpenLDAP)

Cette documentation détaille l'installation et la configuration du serveur **FreeRADIUS** `CHA-RADIUS` afin de fournir une authentification **WPA2-Enterprise** via **PEAP/MSCHAPv2**, en s'appuyant sur un annuaire **OpenLDAP** installé localement sur la même VM.

L'ordre d'installation est le suivant :

1. Installation et configuration d'**OpenLDAP**
2. Création de l'**OU WiFi** et des **comptes utilisateurs**
3. Installation et configuration de **FreeRADIUS**

---

## 1. Prérequis

- Un serveur Debian/Ubuntu opérationnel (ici `CHA-RADIUS`)
- Des certificats TLS signés par l'AC interne (clé privée + certificat serveur + certificat AC)

---

## 2. Installation d'OpenLDAP

### 2.1 Installation des paquets

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install slapd ldap-utils -y
```

Durant l'installation, un mot de passe administrateur LDAP est demandé. Le noter soigneusement.

Reconfigurer ensuite le paquet pour définir le domaine de base :

```bash
sudo dpkg-reconfigure slapd
```

Répondre aux questions comme suit :

| Question | Réponse |
|---|---|
| Omettre la configuration d'OpenLDAP ? | **Non** |
| Nom de domaine DNS | `chartres.sportludique.fr` |
| Nom de l'organisation | `sportludique` |
| Mot de passe administrateur | *(choisir un mot de passe fort)* |
| Supprimer la base lors de `purge` ? | Non |
| Déplacer l'ancienne base ? | Oui |

### 2.2 Démarrage du service

```bash
sudo systemctl enable slapd
sudo systemctl start slapd
sudo systemctl status slapd
```

### 2.3 Vérification de la base de données

```bash
sudo ldapsearch -Y EXTERNAL -H ldapi:/// -b cn=config -LLL \
  "(objectClass=olcDatabaseConfig)" dn olcSuffix olcRootDN olcDbDirectory 2>/dev/null
```

Résultat attendu :

```
dn: olcDatabase={1}mdb,cn=config
olcDbDirectory: /var/lib/ldap
olcSuffix: dc=chartres,dc=sportludique,dc=fr
olcRootDN: cn=admin,dc=chartres,dc=sportludique,dc=fr
```

---

## 3. Création de l'OU WiFi et des comptes utilisateurs

### 3.1 Créer l'OU WiFi

Créer le fichier LDIF :

```bash
sudo nano /tmp/ou_wifi.ldif
```

```ldif
dn: ou=WiFi,dc=chartres,dc=sportludique,dc=fr
objectClass: organizationalUnit
ou: WiFi
```

Injecter dans l'annuaire :

```bash
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /tmp/ou_wifi.ldif
```

Vérifier la création :

```bash
sudo ldapsearch -Y EXTERNAL -H ldapi:/// \
  -b "dc=chartres,dc=sportludique,dc=fr" \
  -LLL "(objectClass=organizationalUnit)" dn ou 2>/dev/null
```

Résultat attendu :

```
dn: ou=WiFi,dc=chartres,dc=sportludique,dc=fr
ou: WiFi
```

### 3.2 Créer un compte de service RADIUS

Ce compte est utilisé par FreeRADIUS pour s'authentifier auprès de l'annuaire et lire les entrées utilisateurs.

Générer un hash du mot de passe :

```bash
slappasswd -s 'ADMINLDAPRADIUS255!'
```

La commande retourne un hash de la forme `{SSHA}...`. Le copier pour l'étape suivante.

```bash
sudo nano /tmp/admin_radius.ldif
```

```ldif
dn: cn=admin.radius,ou=WiFi,dc=chartres,dc=sportludique,dc=fr
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
cn: admin.radius
sn: radius
userPassword: {SSHA}HASH_GENERE_CI_DESSUS
```

```bash
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /tmp/admin_radius.ldif
```

### 3.3 Créer les comptes utilisateurs WiFi

Chaque utilisateur autorisé à se connecter au WiFi doit avoir une entrée dans l'OU `WiFi`. L'attribut `uid` correspond au login saisi lors de la connexion.

Générer les hash des mots de passe :

```bash
slappasswd -s 'MotDePasseUtilisateur'
```

Créer le fichier LDIF avec l'ensemble des utilisateurs :

```bash
sudo nano /tmp/users_wifi.ldif
```

```ldif
dn: uid=y.saintmarc,ou=WiFi,dc=chartres,dc=sportludique,dc=fr
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
cn: Y. Saintmarc
sn: Saintmarc
uid: y.saintmarc
userPassword: {SSHA}HASH_y.saintmarc

dn: uid=m.henault,ou=WiFi,dc=chartres,dc=sportludique,dc=fr
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
cn: M. Henault
sn: Henault
uid: m.henault
userPassword: {SSHA}HASH_m.henault

dn: uid=e.bernier,ou=WiFi,dc=chartres,dc=sportludique,dc=fr
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
cn: E. Bernier
sn: Bernier
uid: e.bernier
userPassword: {SSHA}HASH_e.bernier

dn: uid=p.redler,ou=WiFi,dc=chartres,dc=sportludique,dc=fr
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
cn: P. Redler
sn: Redler
uid: p.redler
userPassword: {SSHA}HASH_p.redler

dn: uid=admin.sio,ou=WiFi,dc=chartres,dc=sportludique,dc=fr
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: person
cn: Admin SIO
sn: SIO
uid: admin.sio
userPassword: {SSHA}HASH_admin.sio
```

```bash
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /tmp/users_wifi.ldif
```

!!! tip
    Pour modifier le mot de passe d'un utilisateur existant :
    ```bash
    sudo ldappasswd -Y EXTERNAL -H ldapi:/// \
      "uid=y.saintmarc,ou=WiFi,dc=chartres,dc=sportludique,dc=fr" \
      -s 'NouveauMotDePasse'
    ```
    Pour supprimer un utilisateur :
    ```bash
    sudo ldapdelete -Y EXTERNAL -H ldapi:/// \
      "uid=y.saintmarc,ou=WiFi,dc=chartres,dc=sportludique,dc=fr"
    ```

### 3.4 Vérification de l'annuaire complet

```bash
sudo ldapsearch -Y EXTERNAL -H ldapi:/// \
  -b "dc=chartres,dc=sportludique,dc=fr" \
  -LLL "(objectClass=*)" dn 2>/dev/null
```

Résultat attendu :

```
dn: dc=chartres,dc=sportludique,dc=fr
dn: ou=WiFi,dc=chartres,dc=sportludique,dc=fr
dn: cn=admin.radius,ou=WiFi,dc=chartres,dc=sportludique,dc=fr
dn: uid=y.saintmarc,ou=WiFi,dc=chartres,dc=sportludique,dc=fr
dn: uid=m.henault,ou=WiFi,dc=chartres,dc=sportludique,dc=fr
dn: uid=e.bernier,ou=WiFi,dc=chartres,dc=sportludique,dc=fr
dn: uid=p.redler,ou=WiFi,dc=chartres,dc=sportludique,dc=fr
dn: uid=admin.sio,ou=WiFi,dc=chartres,dc=sportludique,dc=fr
```

---

## 4. Installation de FreeRADIUS

```bash
sudo apt install freeradius freeradius-ldap -y
```

!!! note
    Le paquet `freeradius-ldap` est indispensable pour activer le module `ldap` dans FreeRADIUS.

Vérifier que le service démarre correctement :

```bash
sudo systemctl enable freeradius
sudo systemctl start freeradius
sudo systemctl status freeradius
```

---

## 5. Structure des fichiers de configuration FreeRADIUS

Les fichiers de configuration se trouvent dans `/etc/freeradius/3.0/` :

| Fichier / Dossier | Rôle |
|---|---|
| `clients.conf` | Déclaration des équipements NAS (bornes AP) |
| `mods-available/eap` | Configuration EAP/PEAP/TLS |
| `mods-available/ldap` | Connexion à l'annuaire LDAP |
| `mods-enabled/` | Liens symboliques vers les modules actifs |
| `sites-available/default` | Serveur virtuel principal (pipeline d'authentification) |
| `certs/` | Certificats TLS utilisés par FreeRADIUS |

---

## 6. Certificats TLS

### 6.1 Contenu du dossier certs

```bash
sudo ls /etc/freeradius/3.0/certs
```

Les fichiers nécessaires sont :

| Fichier | Rôle |
|---|---|
| `radius.key` | Clé privée du serveur RADIUS |
| `radius.pem` | Certificat serveur signé par l'AC interne |
| `ca.pem` | Certificat de l'AC interne |
| `dh` | Paramètres Diffie-Hellman |

### 6.2 Génération des fichiers (si absents)

```bash
# Clé privée
openssl genrsa -out /etc/freeradius/3.0/certs/radius.key 2048

# CSR — à faire signer par l'AC interne
openssl req -new -key /etc/freeradius/3.0/certs/radius.key \
  -out /etc/freeradius/3.0/certs/radius.csr

# Paramètres DH — une seule fois, peut prendre quelques minutes
openssl dhparam -out /etc/freeradius/3.0/certs/dh 2048
```

!!! warning
    Le CSR doit être signé par l'AC interne. Les clients WiFi devront faire confiance à cette AC (via GPO ou profil MDM).

---

## 7. Configuration des clients RADIUS

Fichier : `/etc/freeradius/3.0/clients.conf`

Ce fichier déclare les équipements autorisés à envoyer des requêtes au serveur RADIUS (bornes AP, NAS).

```bash
sudo nano /etc/freeradius/3.0/clients.conf
```

```ini
# ---------------------------------------------------------------------------
# BORNE AP — à dupliquer pour chaque borne
# ---------------------------------------------------------------------------
client ARUBA-AP-103 {
    # IP de la borne AP sur le réseau
    ipaddr       = 192.168.99.1

    # Secret partagé : identique ici et dans la conf RADIUS de la borne
    # Générer avec : openssl rand -base64 32
    # Ne jamais réutiliser le même secret sur deux bornes différentes
    secret       = P@sswordRADIUS548!

    shortname    = ARUBA-AP-103
    require_message_authenticator = yes
    nas_type     = other
}

# ---------------------------------------------------------------------------
# localhost — pour tester avec radtest en local
# ---------------------------------------------------------------------------
client localhost {
    ipaddr       = 127.0.0.1
    secret       = testing123
    shortname    = localhost
    nas_type     = other
}

client localhost_eth {
    ipaddr       = 172.28.33.7
    secret       = testing123
    shortname    = localhost-eth
    nas_type     = other
}
```

!!! tip
    Pour ajouter une borne supplémentaire, dupliquer le bloc `client ARUBA-AP-103` en modifiant `ipaddr`, `secret` et `shortname`.

---

## 8. Module LDAP

Fichier : `/etc/freeradius/3.0/mods-available/ldap`

```bash
sudo nano /etc/freeradius/3.0/mods-available/ldap
```

```ini
ldap {
    server   = 'ldap://127.0.0.1'
    # port = 389  (pas de TLS en local)
    identity = 'cn=admin,dc=chartres,dc=sportludique,dc=fr'
    password = 'ADMINLDAPRADIUS255!'
    base_dn  = 'dc=chartres,dc=sportludique,dc=fr'

    user {
        # OU contenant les comptes utilisateurs WiFi
        base_dn = 'ou=WiFi,DC=chartres,DC=sportludique,DC=fr'

        # Filtre : recherche par uid — seuls les comptes de l'OU WiFi sont acceptés
        filter  = "(uid=%{User-Name})"
        scope   = 'sub'
    }

    update {
        control:Password-With-Header := 'userPassword'
    }
    password_attribute = userPassword

    # ------------------------------------------------------------------
    # TLS — connexion chiffrée (commenté : LDAP local en clair)
    # ------------------------------------------------------------------
    #  tls {
    #      ca_file        = '/etc/freeradius/3.0/certs/ca.pem'
    #      check_cert_cn  = no
    #      require_cert   = 'demand'
    #  }

    # ------------------------------------------------------------------
    # POOL DE CONNEXIONS
    # ------------------------------------------------------------------
    pool {
        start        = 2
        min          = 1
        max          = 20
        spare        = 2
        idle_timeout = 60
        retry_delay  = 10
    }

    timeout     = 5
    timelimit   = 3
    net_timeout = 10
}
```

Activer le module :

```bash
sudo ln -s /etc/freeradius/3.0/mods-available/ldap \
           /etc/freeradius/3.0/mods-enabled/ldap
```

---

## 9. Module EAP

Fichier : `/etc/freeradius/3.0/mods-available/eap`

Seule la méthode **PEAP/MSCHAPv2** est activée. Toutes les autres méthodes EAP sont volontairement désactivées.

```bash
sudo nano /etc/freeradius/3.0/mods-available/eap
```

```ini
eap {
    default_eap_type         = peap
    timer_expire             = 60
    ignore_unknown_eap_types = yes

    # ------------------------------------------------------------------
    # PEAP — tunnel TLS dans lequel transite MSCHAPv2
    # Compatible WPA2-Enterprise : Windows / Android / iOS
    # ------------------------------------------------------------------
    peap {
        default_eap_type = mschapv2
        tls              = tls-peap
        virtual_server   = inner-tunnel

        tls_session_resumption {
            enable   = yes
            lifetime = 86400    # 24h en secondes
        }
    }

    # ------------------------------------------------------------------
    # TLS — certificat présenté par le serveur RADIUS aux clients WiFi
    # Ce certificat doit être signé par l'AC interne
    # ------------------------------------------------------------------
    tls-config tls-peap {
        private_key_file  = /etc/freeradius/3.0/certs/radius.key
        certificate_file  = /etc/freeradius/3.0/certs/radius.pem
        ca_file           = /etc/freeradius/3.0/certs/ca.pem
        dh_file           = /etc/freeradius/3.0/certs/dh

        tls_min_version   = "1.2"
        cipher_list       = "ECDHE+AESGCM:ECDHE+CHACHA20"
    }

    # MSCHAPv2 — renvoyer le message d'erreur au client en cas d'échec
    mschapv2 {
        send_error = yes
    }
}
```

Le module EAP est activé par défaut. Vérifier le lien symbolique :

```bash
ls -la /etc/freeradius/3.0/mods-enabled/eap
```

---

## 10. Serveur virtuel — sites-available/default

Fichier : `/etc/freeradius/3.0/sites-available/default`

Ce fichier définit le pipeline d'authentification. Seule l'authentification WPA2-Enterprise est traitée ici, sans SQL ni logique VLAN.

```bash
sudo nano /etc/freeradius/3.0/sites-available/default
```

```ini
server default {

    # ------------------------------------------------------------------
    # ÉCOUTE — UDP 1812 (auth) uniquement
    # ------------------------------------------------------------------
    listen {
        type   = auth
        ipaddr = *
        port   = 1812
    }

    # ------------------------------------------------------------------
    # AUTHORIZE
    # Détermine comment authentifier la requête entrante
    # Flux : client envoie EAP -> on détecte le type -> on cherche dans le LDAP
    # ------------------------------------------------------------------
    authorize {

        # Nettoyer les attributs malformés
        preprocess

        # Détection MS-CHAPv2 (dans le tunnel PEAP)
        mschap

        # Détection EAP — renvoie vers la section authenticate/eap
        # "ok = return" : si EAP détecté, on passe directement à l'auth
        # sans chercher dans d'autres modules (fichier users, sql...)
        eap {
            ok = return
        }

        # Recherche de l'utilisateur dans l'OU WiFi du LDAP local
        # Le filtre dans mods-available/ldap gère le rejet si hors-OU
        -ldap

        pap
    }

    # ------------------------------------------------------------------
    # AUTHENTICATE
    # Effectue l'authentification selon le type détecté dans authorize
    # ------------------------------------------------------------------
    authenticate {
        Auth-Type PAP {
            pap
        }

        # EAP-PEAP : tout passe ici, le tunnel est géré par le module eap
        eap

        # MSCHAPv2 à l'intérieur du tunnel PEAP
        Auth-Type MS-CHAP {
            mschap
        }
    }

    # ------------------------------------------------------------------
    # POST-AUTH
    # Actions après authentification — uniquement la gestion des rejets
    # ------------------------------------------------------------------
    post-auth {

        # Si Access-Accept : ne rien faire de particulier (pas de VLAN)
        update {
            &reply: += &session-state:
        }

        # Si Access-Reject : filtrer les attributs de réponse
        Post-Auth-Type REJECT {
            attr_filter.access_reject
            eap
            remove_reply_message_if_eap
        }
    }
}
```

Vérifier que le site est activé :

```bash
ls -la /etc/freeradius/3.0/sites-enabled/default
```

---

## 11. Activation des modules

S'assurer que tous les modules nécessaires sont bien activés (liens symboliques dans `mods-enabled`) :

```bash
# ldap — à créer manuellement
sudo ln -s /etc/freeradius/3.0/mods-available/ldap \
           /etc/freeradius/3.0/mods-enabled/ldap

# mschap et eap sont généralement activés par défaut
ls /etc/freeradius/3.0/mods-enabled/
```

---

## 12. Tests et vérification

### 12.1 Test de syntaxe

```bash
sudo freeradius -XC
```

Cette commande affiche les erreurs de configuration sans démarrer le service. Elle ne doit retourner aucune erreur critique.

### 12.2 Test en mode debug

```bash
sudo systemctl stop freeradius
sudo freeradius -X
```

Lancer un test depuis un second terminal :

```bash
radtest y.saintmarc MonMotDePasse 127.0.0.1 0 testing123
```

Résultat attendu en cas de succès :

```
Received Access-Accept Id 0 from 127.0.0.1:1812
```

### 12.3 Test de connexion LDAP

Vérifier que FreeRADIUS peut interroger l'annuaire :

```bash
sudo ldapsearch -x -H ldap://127.0.0.1 \
  -D "cn=admin,dc=chartres,dc=sportludique,dc=fr" \
  -w 'ADMINLDAPRADIUS255!' \
  -b "ou=WiFi,dc=chartres,dc=sportludique,dc=fr" \
  "(uid=y.saintmarc)"
```

### 12.4 Redémarrage du service

Une fois les tests validés :

```bash
sudo systemctl start freeradius
sudo systemctl status freeradius
```

---

## 13. Récapitulatif de l'architecture

```
Client WiFi (PC / Mobile)
        |
        | WPA2-Enterprise (PEAP/MSCHAPv2)
        |
Borne AP (ARUBA-AP-103 — 192.168.99.1)
        |
        | RADIUS UDP/1812 — secret partagé
        |
CHA-RADIUS (FreeRADIUS + OpenLDAP — même VM)
        |
        | LDAP ldap://127.0.0.1:389 (local)
        |
OpenLDAP (ou=WiFi,dc=chartres,dc=sportludique,dc=fr)
```

Le flux d'authentification est le suivant :

1. Le client envoie ses identifiants via PEAP/MSCHAPv2 à la borne AP
2. La borne transmet la requête RADIUS au serveur `CHA-RADIUS`
3. FreeRADIUS interroge l'annuaire OpenLDAP local pour vérifier l'existence de l'utilisateur dans l'OU `WiFi`
4. Si l'utilisateur existe et que le mot de passe est correct → **Access-Accept**
5. Sinon → **Access-Reject**
