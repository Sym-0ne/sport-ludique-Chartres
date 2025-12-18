# Configuration DNS autorité BIND9

Cette documentation sert a détailler l'installation et la configuration de notre serveur DNS d'autorité grâce au service BIND9 basé sur Debian 13.

---

## 1. Préparation 

Avant de commencer il faut se poser les bonnes questions, nous aurons donc besoin : 

1.1 Serveur d'autorité UNIQUEMENT :arrow_right: aucune récursivité sur celui-ci

1.2 Deux zones de recherche DNS, une locale et une externe, explication sur le [mkdoc de Mr Merry](https://lmeryfulbert.github.io/SportLudique2025-2026/cours/05-Services/dns/dns/#dns-partage-split-horizon-dns)

1.3 Un PAT sur le routeur pour les utilisateurs externes

1.4 Des routes statiques sur le DNS d'autorité à cause de l'IP Forwarding bloqué par l'IPS du StormShield

---

## 2. Installation de BIND9 

```
sudo apt update 
sudo apt upgrade 
sudo apt install bind9 bind9utils bind9-doc dnsutils -y
```
<div class="annotate" markdown>

:bulb:(1)

</div>

1. A noter que ``` bind9utils bind9-doc dnsutils ``` ne sont que des outils supplémentaires pour le test du DNS et du BIND9

---

## 3. Création des dossier dédiés

Création de dossiers où l'on va stocker nos fichiers de zones et attribution des droits au service BIND9

```
sudo mkdir -p /etc/bind/zones
sudo chown bind:bind /etc/bind/zones
```

---

## 4. Configuration des "views"
Les "view" sont les différentes zones que notre DNS va utiliser en fonction des adresses que ces mêmes zones desservent. 

```
sudo nano /etc/bind/name.conf.local
```

```
// === VUE INTERNE ===
view "internal" {
    match-clients { 172.28.0.0/16; 127.0.0.1; };
    recursion no;

    zone "chartres.sportludique.fr" {
        type master;
        file "/etc/bind/zones/db.chartres.sportludique.fr.internal";
    };
};

// === VUE EXTERNE ===
view "external" {
    match-clients { any; };
    recursion no;

    zone "chartres.sportludique.fr" {
        type master;
        file "/etc/bind/zones/db.chartres.sportludique.fr.external";
    };
};
```

---

## 5. Création des fichiers de zones

### 5.1 Création du fichier "internal"
Ce fichier servira a traduire les noms de domaines en IP pour toutes les adresses du LAN.
```
sudo nano /etc/bind/zones/db.chartres.sportludique.fr.internal
```

```
$TTL 86400
@   IN  SOA ns1.chartres.sportludique.fr. admin.chartres.sportludique.fr. (
        2025100701 ; Numéros de série au format YYYY/MM/DD/01
        3600       ; Intervalle de rafraîchissement
        1800       ; Intervalle de réessai
        1209600    ; Intervalle d'expiration
        86400 )    ; Durée minimale de cache

; Serveurs de nom
@   IN  NS  ns1.chartres.sportludique.fr.

; Enregistrements
ns1 IN  A   172.28.62.1   ; Serveur DNS d'autorité (nous même)
www IN  A   172.28.62.5   ; Reverse Proxy
www.cimmob IN A 172.28.62.5
cha IN  A   172.28.33.2   ; Serveur DNS de l'AD
```

### 5.2 Création du fichier "external"
Ce fichier servira a traduire les noms de domaines en IP pour toutes les adresses externes à notre infrastructure. 
```
sudo nano /etc/bind/zones/db.chartres.sportludique.fr.external
```

```
$TTL 86400
@   IN  SOA ns1.chartres.sportludique.fr. admin.chartres.sportludique.fr. (
        2025100701 ; Numéros de série au format YYYY/MM/DD/01
        3600       ; Intervalle de rafraîchissement
        1800       ; Intervalle de réessai
        1209600    ; Intervalle d'expiration
        86400 )    ; Durée minimale de cache

; Serveurs de nom
@   IN  NS  ns1.chartres.sportludique.fr.

; Enregistrements
ns1 IN  A   183.44.28.1   ; Serveur DNS d'autorité (nous même)
www IN  A   183.44.28.1   ; Serveur Web
www.cimmob IN A 183.44.28.1
```
<div class="annotate" markdown>

:bulb:(1)

</div>
1. Tout les enregistrements pointent vers l'IP publique de R1 qui feras ensuite le lien avec le LAN grâce au PAT, la haute disponibilité seras implémenter plus tard

---

**⚠️⚠️⚠️ Tout changement dans la configuration des zones requier l'incrémentation du "Numéros de série" pour prendre effet ⚠️⚠️⚠️**

## 6. Désactivation des 13 serveurs racines

Les 13 serveurs DNS racines servent à TOUS les serveurs récursifs à se diriger vers les bons serveurs DNS pour accomplir leur résolution, nous allons les désactivées car ils rentrent en conflit avec nos views, de plus il est inutile de les avoirs étant donné que ce serveur ne fait AUCUNE récursion. 

```
sudo nano /etc/bind/named.conf.root-hints 
```

Il suffit ensuite de mettre toutes les lignes en commentaires

```
// prime the server with knowledge of the root servers
//zone "." {
//      type hint;
//      file "/usr/share/dns/root.hints";
//};
```

---

## 7. Tests et vérification :

Pour finir il ne nous reste plus qu'à exécuter ces 3 commandes afin de vérifier le bon fonctionnement de notre DNS

### 7.1 Fichier de configuration 

```
sudo named-checkconf
```
Cette commande ne nous retourne rien si le fichier de configuration est bon

### 7.2 Zones 

```
sudo named-checkzone chartres.sportludique.fr /etc/bind/zones/db.chartres.sportludique.fr.internal
sudo named-checkzone chartres.sportludique.fr /etc/bind/zones/db.chartres.sportludique.fr.external
```

Ces deux commandes nous retourneront la valeur du numéro de série de la zone si elle est bien configurée

---

## 8. Pare-feux local (UFW)
<div class="annotate" markdown>

Nous allons mettre en place un firewall local grâce a UFW (1) afin de bloquer toutes les connexion non necessaire au fonctionnement du DNS pour plus de sécurité.

</div>
1. Toute la documentation concernant UFW se trouve [ici](Pare-feux/UFW/)

### 8.1 Besoins

<div class="annotate" markdown > 

Après avoir installer UFW (1) il nous suffit de mettre en place 3 règles :

</div>
1. :warning: Attention, l'activation d'UFW dois être effectuer a la fin pour ne pas perdre la connexion SSH avant d'avoir mis les règles qui l'autorise en place.

- Une règle qui autorise les conexion SSH depuis le réseau de management sur mon interface de management
- Une règle qui autorise les connexions DNS en UDP sur le port 53
- Une règle qui autorise les connexions DNS en TCP sur le port 53

Ces règles ont été décider grâce a l'étude des connexion qui auront lieu sur notre réseau et leur sens.

### 8.2 Configuration

#### 8.2.1 Règles par défaut
Par defaut nous allons refuser toutes les connexion afin d'autoriser uniquement les connexion DNS par la suite.

```
sudo ufw default deny incoming 
sudo ufw default deny outgoing
```
#### 8.2.2 Règles de filtrage
Voici donc les règles de filtrages a appliquer a nos interfaces :
```
sudo ufw allow in on ens4 from 10.10.120.0/24 to 10.10.120.8 port 22 proto tcp
sudo ufw allow in on ens3 from all to 172.28.62.1 port 53 proto tcp
sudo ufw allow in on ens3 from all to 172.28.62.1 port 53 proto udp
```
#### 8.2.3 Activation d'UFW
Une fois les règles qui laissent passer les trafics voulues il ne nous reste plus qu'à activer UFW afin que nos règles prennent effet.
```
sudo ufw enable
```

---

## 9. Route statique

### 9.1 Pourquoi
L'ajout de routes statiques au sein de notre DNS est obligatoire à cause de notre pare-feu Stormshield (PFW) et de la conception de notre réseau. En effet, comme[expliqué ici](https://sym-0ne.github.io/sport-ludique-Chartres/Pare-feux/stormshield/#7-statefull-inspection) le Stormshield et son Statefull Inspection bloquent le flux TCP, car le handshake ne s'effectue pas correctement..

### 9.2 Ajout des routes
Il nous faut donc ajouter manuellement des routes statiques afin de passer directement par le VFW pour rejoindre notre LAN.
```
sudo nano /etc/network/interfaces
```

Ajouter ensuite ces lignes :

```
# Exemple de route : up ip route add 'réseau à joindre' via 'passerelle' dev 'interface sortante'
up ip route add 172.28.32.0/24 via 172.28.62.253 dev ens3
up ip route add 172.28.33.0/24 via 172.28.62.253 dev ens3
up ip route add 172.28.35.0/24 via 172.28.62.253 dev ens3
```

Grâce à ces lignes, notre DNS passera directement par le VFW et non par le PFW (Stormshield).

---

## 10. NS1 et NS2 

<div class="annotate"markdown>
Voici la phrase corrigée :

Afin de **garantir la haute disponibilité** de nos services, nous avons doublé notre PFW ainsi que notre DNS d'autorité (1). Dans notre cas, le cheminement réseau change en fonction des équipements en service ou hors service.

</div>
1. À noter que c'est à cause de la [Stateful Inspection](https://sym-0ne.github.io/sport-ludique-Chartres/Pare-feux/stormshield/#7-statefull-inspection) du StormShield que nous devons doubler ces équipements.

### 10.1 Besoins

<div class="annotate" markdown>
Les zones **internes** ne changeant pas, NS1 et NS2 seront donc synchronisées, NS1 étant **master** et NS2 **slave** (1).

Les zones externes, elles, changent, notamment au niveau de la résolution, où **NS1** renvoie vers **R1** et **NS2** vers **R2**.
</div>
1. **Master** est le maître et **Slave** l'esclave ; le maître envoie la configuration et les esclaves la copient.

### 10.2 Configuration

Dans le fichier `/etc/bind/name.conf.local` de <ins>**NS1**</ins>, il faut ajouter ces lignes :
```python hl_lines="9 10"
// === VUE INTERNE ===
view "internal" {
    match-clients { 172.28.0.0/16; 127.0.0.1; };
    recursion no;

    zone "chartres.sportludique.fr" {
        type master;
        file "/etc/bind/zones/db.chartres.sportludique.fr.internal";
        allow-transfer {172.28.62.11;};
		also-notify {172.28.62.11;};
    };
};

// === VUE EXTERNE ===---
view "external" {
    match-clients { any; };
    recursion no;

    zone "chartres.sportludique.fr" {
        type master;
        file "/etc/bind/zones/db.chartres.sportludique.fr.external";
    };
};
```
Ensuite, dans le fichier `/etc/bind/name.conf.local` de <ins>**NS2**</ins>, il suffit de rajouter ces lignes :
```python hl_lines="7 8 9"
// Vue interne
view "internal" {
    match-clients { 127.0.0.1; 172.28.0.0/16; };
    recursion no;

    zone "chartres.sportludique.fr" {
        type slave;
        masters { 172.28.62.1; };  // IP du maître
        file "/var/cache/bind/db.chartres.sportludique.fr.internal";
    };
};

// Vue externe
view "external" {
    match-clients { any; };
    recursion no;

    zone "chartres.sportludique.fr" {
        type master;
        file "/etc/bind/zones/db.chartres.sportludique.fr.external";
    };---
};
```

Pour finir, il suffit de modifier le fichier `/etc/bind/zones/db.chartres.sportludique.fr.external` de <ins>**NS2**</ins> pour qu'il pointe vers **R2** et non **R1**. 

```
$TTL 86400
@       IN SOA  ns1.chartres.sportludique.fr. admin.chartres.sportludique.fr. (
                2025100704 ; Serial
                3600        ; Refresh
                1800        ; Retry
                1209600     ; Expire
                86400 )     ; Negative Cache TTL

; ----- Serveurs DNS -----
@       IN NS   ns1.chartres.sportludique.fr.
@       IN NS   ns2.chartres.sportludique.fr.

; Exemple (à compléter si tu veux) :
ns1    IN A 183.44.28.1
ns2    IN A 221.87.128.2  
www    IN A 221.87.128.2
cimmob IN A 221.87.128.2
```

Ne pas oublier de mettre à jour le fichier **External** sur les deux DNS, puisqu'ils ne sont plus synchronisés.

---