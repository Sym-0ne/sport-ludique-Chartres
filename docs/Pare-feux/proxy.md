# Mise en place et configuration d'un Proxy Filtrant via Stormshield (SN210)

---

## Contexte : qu'est ce qu'un proxy filtrant ?

Un **proxy filtrant** est un composant de sécurité réseau qui se place entre les utilisateurs et Internet.<br>
Son rôle est de **contrôler**, **analyser** et éventuellement **bloquer** ou **modifier** le trafic web avant qu’il n’atteigne les sites visités.

Il sert notamment à :

* Bloquer les sites dangereux ou non conformes aux règles de l’entreprise,
* Analyser le trafic HTTPS (inspection SSL),
* Protéger les utilisateurs contre les malwares,
* Appliquer des politiques d’accès (catégories, filtrage, blocage…).

En résumé, c’est un intermédiaire sécurisé qui vérifie tout ce qui sort et tout ce qui rentre.

---

## Stormshield et son proxy filtrant

Les firewalls **Stormshield (SNS)** intègrent un **proxy filtrant natif**, capable de gérer :

* Le filtrage URL,
* L’analyse HTTPS (déchiffrement SSL),
* Les contrôles de certificats,
* L’application de règles de sécurité par utilisateur, catégorie ou site.

Ce proxy fait partie des modules de protection applicative du firewall, et il permet d’aller beaucoup plus loin qu’un simple filtrage IP classique.

---

## 1. Configurer la politique de filtrage SSL

### 1.1 Se rendre dans la configuration :
Configuration → Politique de sécurité → Filtrage SSL.

### 1.2 Mettre en place le filtrage SSL
Créez une nouvelle politique SSL en cliquant sur ```Ajouter```

### 1.3 Configurez la règle :
    -   Déchiffrer → inspection SSL,
    -   Passer sans déchiffrer,
    -   Bloquer sans déchiffrer.
### 1.4 Odres des réglès
Assurez-vous du bon ordre des règles selon les autorisations et interdictions mises en place.
---
### 1.5 Catégories d'objets :

```URL-CN``` est un objet (catégorie d’URL) regroupant plusieurs sites. Par exemple :

* URL-CN : **Online** → sites de paris en ligne, réseaux sociaux, plateformes interactives…
* URL-CN : **News** → journaux en ligne, sites de radiodiffusion, magazines, médias d’actualité…

---

## 2. Configurer la politique de Filtrage et NAT 

### 2.1 Se rendre dans la configuration :
Configuration → Politique de sécurité → Filtrage & NAT.

### 2.2 Mettre en place le filtrage
Créez une nouvelle politique de filtrage en cliquant sur ```Nouvelle règle```

### 2.3 Configurez la règle :
    -   État : Activé
    -   Action : Déchiffrer
    -   Sources : Network_Internals
    -   Destination : In
    -   Port Destiation : SSL_srv (Objet incluant les protocoles : https, imaps, ftps...)
    -   Protocole : Laisser cette case vide
    -   Inspection de Sécurité : Filtrage SSL : *Nom de la règle de filtrage SSL créér précédemment*

### 2.4 Odres des réglès
Assurez-vous que cette règle se trouve en tête de liste.

---

## 4. Tester le déchiffrement SSL
Depuis un poste client, accédez à un site HTTPS.

### 4.1  Vérifiez que :
a) Le site se charge.<br>
b) Le site chargé est bloqué avec une page violette qui est le Proxy avec le message suivant : 

```Your administrator reject the connection to this SSL Server 
```

Cela signifique que le Proxy est bel et bien fonctionnel !

---

## 5. Accéder aux objects :
    Objets → Certificat / PKI → Ajouter → Importer un fichier :
        - Importer le CA (de tout les sites sportludique.fr)
        - Selectionner le format **PEM**
        - Ne pas mettre de mot de passe
        - Élements à importer : CA
    Puis valider l'importation !

### 5.1 Accéder à la configuration : 
    - Configuration → Protection applicative → Protocoles.
    - Sélectionnez *SSL*.
    - Cliquez sur *Accéder à la configuration globale*.
    - Autorités de certification personnalisées → Ajouter → Séléctionner le CA

---