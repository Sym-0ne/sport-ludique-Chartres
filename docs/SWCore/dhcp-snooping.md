# Mise en place - DHCP SNOOPING

## 1. OBJECTIF
---
Le DHCP Snooping est une fonctionnalité de sécurité
qui permet de protéger le réseau contre les serveurs
DHCP non autorisés (Rogue DHCP Server).

Le switch analyse les messages DHCP et autorise
uniquement les réponses provenant des ports
déclarés comme "trusted".

Il crée également une table contenant :
- Adresse IP
- Adresse MAC
- VLAN
- Port du switch

Cette table est appelée "DHCP Snooping Binding Table".

---

## 2. PRINCIPE DE FONCTIONNEMENT
---

Deux types de ports existent :

Trusted Port
- Port autorisé à envoyer des réponses DHCP
- Généralement connecté au serveur DHCP
- Peut envoyer DHCP OFFER / ACK

Untrusted Port
- Ports des utilisateurs
- Ne peuvent pas envoyer de réponses DHCP
- Seules les requêtes DHCP sont autorisées

---

## 3. TOPOLOGIE EXEMPLE
---

             DHCP SERVER
                 |
                 |
           GE1/0/1 (TRUSTED)
               SWITCH
            HP 5500
             /   \
            /     \
      GE1/0/2    GE1/0/3
      CLIENT      CLIENT

---

## 4. ACTIVATION DU DHCP SNOOPING
---

Entrer en mode configuration :

system-view

Activer DHCP Snooping :

dhcp-snooping

---

## 5. CONFIGURATION DU PORT TRUSTED
---

Le port connecté au serveur DHCP doit être configuré
comme trusted.

Exemple :

interface GigabitEthernet1/0/1
 dhcp-snooping trust
 quit

---

## 6. PORTS CLIENTS
---

Les ports utilisateurs restent en mode untrusted
(par défaut).

Exemple :

interface GigabitEthernet1/0/2
 quit

interface GigabitEthernet1/0/3
 quit

---

## 7. VERIFICATION DE LA CONFIGURATION
---

Afficher l'état du DHCP Snooping :

display dhcp-snooping

Afficher les ports trusted :

display dhcp-snooping trust

Afficher la table des clients :

display dhcp-snooping binding

---

## 8. CONFIGURATION COMPLETE EXEMPLE
---

system-view

dhcp-snooping

interface GigabitEthernet1/0/1
 dhcp-snooping trust
 quit

interface GigabitEthernet1/0/2
 quit

interface GigabitEthernet1/0/3
 quit

---

## 9. OPTION 82 (OPTIONNEL)
---

L'option 82 permet d'ajouter des informations
sur le port et le VLAN dans les requêtes DHCP.

Activation :

dhcp-snooping information enable

Cela permet au serveur DHCP d'identifier
le port exact d'où provient la requête.

---

## 10. BONNES PRATIQUES
---

- Activer DHCP Snooping sur les switches d'accès
- Configurer uniquement le port du serveur DHCP
  comme TRUSTED
- Laisser tous les ports utilisateurs en UNTRUSTED
- Vérifier régulièrement la table DHCP Snooping
- Combiner avec Dynamic ARP Inspection pour
  une meilleure sécurité

---
