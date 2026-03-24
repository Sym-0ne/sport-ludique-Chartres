# Mise en place - DHCP SNOOPING

## 1. Objectif 
---
Le DHCP Snooping est une fonctionnalité de sécurité
qui permet de protéger le réseau contre les serveurs
DHCP non autorisés (Rogue DHCP Server).

Le switch analyse les messages DHCP et autorise
uniquement les réponses provenant des ports
déclarés comme "trusted".

Il crée également une table contenant :

* Adresse IP
* Adresse MAC
* VLAN
* Port du switch


Cette table est appelée "DHCP Snooping Binding Table".

---

## 2. Princpie de fonctionnement 
---

Deux types de ports existent :

+----------------+--------------------------------------------------------------+
| Type de Port   | Description                                                  |
+----------------+--------------------------------------------------------------+
| Trusted Port   | - Port autorisé à envoyer des réponses DHCP                 |
|                | - Généralement connecté au serveur DHCP                     |
|                | - Peut envoyer les messages DHCP OFFER et DHCP ACK          |
+----------------+--------------------------------------------------------------+
| Untrusted Port | - Ports utilisés par les utilisateurs                       |
|                | - Ne peuvent pas envoyer de réponses DHCP                   |
|                | - Seules les requêtes DHCP sont autorisées                  |
+----------------+--------------------------------------------------------------+

---

## 3. Topologie 
---

             DHCP SERVER
                 |
                 |
           fa1/0/1 (TRUSTED)
               SWITCH
            HP 5500
             /   \
            /     \
      fa1/0/2    fa1/0/3
      CLIENT      CLIENT

---

## 4. Activation du DHCP SNOOPING 
---

Entrer en mode configuration :

```
system-view
```

Activer DHCP Snooping :

```
dhcp-snooping
```

---

## 5. Configuration du port TRUSTED
---

Le port connecté au serveur DHCP doit être configuré
comme trusted.

Exemple :

```
interface Bridge-Aggregation1    # le port sur lequel se trouve le serveur DHCP ou le relai DHCP (souvent un port d'interco 802.1Q)
dhcp snooping trust   # autorise a voir des réponses DHCP
```

## 6. Vérification de la configuration
---

Afficher l'état du DHCP Snooping :

```
display dhcp-snooping
```

Afficher les ports trusted :

```
display dhcp-snooping trust
```

Afficher la table des clients :

```
display dhcp-snooping binding
```

---

## 7. Configuration complète
---

```
system-view

dhcp-snooping

interface Bridge-Aggregation1    # le port sur lequel se trouve le serveur DHCP ou le relai DHCP (souvent un port d'interco 802.1Q)
dhcp-snooping trust   # autorise a voir des réponses DHCP

interface GigaEthernet2/0/25   # on limite le nombre de requetes clients DHCP venant sur ce port
dhcp snooping rate-limit 64 #Evite les attaques DHCP Starvation (Sur HP on ne peut pas limiter les requetes DHCP en dessous de 64/sec)
```

---

## 8. Test Complet 

- On désactive le service DHCP de notre serveur (AD) pour simuler une déni de service causé par l'attaquant (DOS).
- On desactive le dhcp-snooping sur notre SWCore.
- On lance une requete DHCP sur notre machine cliente, elle récupere belle et bien une ip depuis le DHCP ROG.
- On réactive le service DHCP sur notre serveur (AD), la machine récupere encore une ip depuis le DHCP ROG.
- On active alors le service dhcp-snooping sur notre SWCore, la machine récupere maitenant l'ip depuis notre DHCP Serveur (AD).


- Conclusion le service fonctionne correctement. 

---