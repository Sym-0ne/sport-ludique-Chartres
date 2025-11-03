# Configuration DNS Resolveur UNBOUND

On met en place **Unbound comme résolveur DNS** (non validateur DNSSEC) sur une VM Debian, en l’intégrant à un DNS autoritaire.

## 📦 1. Installation de UNBOUND

```
sudo apt update
sudo apt install unbound -y
```

Unbound se lancera automatiquement après l'installation.

Vérifier si c'est bien le cas :

```
sudo systemctl status unbound
```

## 🔧 2. Configuration du DNS

Editer le fichier suivant :

```
sudo nano /etc/unbound/unbound.conf
```

Puis ajouter votre conf :

```
server:
    #######################################################################
    #  PARAMÈTRES GÉNÉRAUX DU SERVEUR UNBOUND
    #######################################################################

    # Interfaces d'écoute
    interface: 0.0.0.0
    interface: ::0
    port: 53

    # Autorise IPv4, désactive IPv6 si inutile
    do-ip4: yes
    do-ip6: no
    do-udp: yes
    do-tcp: yes

    # Réseaux autorisés à interroger le résolveur
    access-control: 127.0.0.0/8 allow        # localhost
    access-control: 172.28.33.0/24 allow     # LAN
    access-control: 172.28.62.0/24 allow     # DMZ
    access-control: 172.28.35.0/24 allow          # CLIENTS
    access-control: 0.0.0.0/0 refuse         # tout le reste bloqué

    # Désactive DNSSEC
    module-config: "iterator"
    val-permissive-mode: no
    auto-trust-anchor-file: ""
    trust-anchor-signaling: no
    harden-dnssec-stripped: no

    # Répertoires de travail
    directory: "/etc/unbound"
    logfile: "/var/log/unbound.log"
    verbosity: 1

    # Cache
    cache-min-ttl: 60
    cache-max-ttl: 86400

#######################################################################
#  DÉCLARATION DES ZONES AVEC SERVEURS FORWARDERS SPÉCIFIQUES
#######################################################################

# Zone locale -> DNS de la DMZ
stub-zone:
    name: "chartres.sportludique.fr"
    stub-addr: 172.28.62.1     # ← IP du DNS autoritaire de la DMZ

# Zone de l’entreprise -> DNS de l’enseignant
stub-zone:
    name: "sportludique.fr"
    stub-addr: 121.183.90.205  # ← IP du DNS de l’enseignant
```

Vérifier la syntaxe :

```
sudo unbound-checkconf
```

Si il y a des erreurs, le **DNS ne fonctionnerra pas** car vous ne pourrais pas redémarrer le service donc **il faut absolument résoudre les erreurs**.

Sinon redémarrer le service :

```
sudo systemctl restart unbound
sudo systemctl enable unbound
```

Vérifier que le DNS redirige bien vers l'autorité :

```
sudo cat /etc/resolve.conf
```

Le résultat attendus est :

```
nameserver 172.28.62.1
search chartres.sportludique.fr
```

## ✅ 3. Vérification

Tester depuis le client avec **"nslookup ns1.chartres.sportludique.fr"**.

Il devrait renvoyer **l'adresse IP du DNS autorité**.

Et ensuite tester depuis un poste d'un autre groupe avec la même commande.

Il devrait renvoyer **l'adresse IP de la FAI (183.44.28.1)**.