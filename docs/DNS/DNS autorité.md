# Configuration DNS autorité BIND9

Cette documentation sert a détailler l'installation et la configuration de notre serveur DNS d'autorité grâce au service BIND9 basé sur Debian 13.

## Préparation :memo:

Avant de commencer il faut se poser les bonnes questions, nous aurons donc besoin : 

1. Serveur d'autorité UNIQUEMENT :arrow_right: aucune récursivité sur celui-ci

2. Deux zones de recherche DNS, une locale et une externe, explication sur le [mkdoc de Mr Merry](https://lmeryfulbert.github.io/SportLudique2025-2026/cours/05-Services/dns/dns/#dns-partage-split-horizon-dns)

3. Un PAT sur le routeur pour les utilisateurs externes

4. Des routes statiques sur le DNS d'autorité à cause de l'IP Forwarding bloqué par l'IPS du StormShield

## Installation de BIND9 

```
sudo apt update 
sudo apt upgrade 
sudo apt install bind9 bind9utils bind9-doc dnsutils -y
```
<div class="annotate" markdown>

:bulb:(1)

</div>

1. A noter que ``` bind9utils bind9-doc dnsutils ``` ne sont que des outils supplémentaires pour le test du DNS et du BIND9

## Création des dossier dédiés

Création de dossiers où l'on va stocker nos fichiers de zones et attribution des droits au service BIND9

```
sudo mkdir -p /etc/bind/zones
sudo chown bind:bind /etc/bind/zones
```
## Configuration des "views"
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

## Création des fichiers de zones

### Création du fichier "internal"
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
www IN  A   172.28.62.2   ; Serveur Web
cha IN  A   172.28.33.2   ; Serveur DNS de l'AD
```
### Création du fichier "external"
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
```
<div class="annotate" markdown>

:bulb:(1)

</div>
1. Tout les enregistrements pointent vers l'IP publique de R1 qui feras ensuite le lien avec le LAN grâce au PAT, la haute disponibilité seras implémenter plus tard

## :warning: Désactivation des 13 serveurs DNS racines

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
## Tests et vérification :white_check_mark:

Pour finir il ne nous reste plus qu'à exécuter ces 3 commandes afin de vérifier le bon fonctionnement de notre DNS

### Fichier de configuration 

```
sudo named-checkconf
```
Cette commande ne nous retourne rien si le fichier de configuration est bon

 ### Zones 

```
sudo named-checkzone chartres.sportludique.fr /etc/bind/zones/db.chartres.sportludique.fr.internal
sudo named-checkzone chartres.sportludique.fr /etc/bind/zones/db.chartres.sportludique.fr.external
```

Ces deux commandes nous retourneront la valeur du numéro de série de la zone si elle est bien configurée
