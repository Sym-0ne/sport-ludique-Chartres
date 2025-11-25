# Mise en place et configuration d'un Proxy Filtrant via Stormshield (SN210)

*Prérequis :*
- Connectez-vous à l’interface Web du SN210 avec un compte administrateur.

# Contexte : c’est quoi un proxy filtrant ?

Un **proxy filtrant** est un composant de sécurité réseau qui se place entre les utilisateurs et Internet.<br>
Son rôle est de **contrôler**, **analyser** et éventuellement **bloquer** ou **modifier** le trafic web avant qu’il n’atteigne les sites visités.

Il sert notamment à :

* Bloquer les sites dangereux ou non conformes aux règles de l’entreprise,
* Analyser le trafic HTTPS (inspection SSL),
* Protéger les utilisateurs contre les malwares,
* Appliquer des politiques d’accès (catégories, filtrage, blocage…).

En résumé, c’est un intermédiaire sécurisé qui vérifie tout ce qui sort et tout ce qui rentre.

## 🛡️ Stormshield et son proxy filtrant

Les firewalls **Stormshield (SNS)** intègrent un **proxy filtrant natif, capable de gérer :

* Le filtrage URL,
* L’analyse HTTPS (déchiffrement SSL),
* Les contrôles de certificats,
* L’application de règles de sécurité par utilisateur, catégorie ou site.

Ce proxy fait partie des modules de protection applicative du firewall, et il permet d’aller beaucoup plus loin qu’un simple filtrage IP classique.

## 1. Configurer la politique de filtrage SSL

### 1.1  Allez dans : Configuration → Politique de sécurité → Filtrage SSL.
### 1.2  Créez une politique SSL.
### 1.3  Configurez les règles par catégories d’URL ou CN :
    -   Déchiffrer → inspection SSL,
    -   Passer sans déchiffrer,
    -   Bloquer sans déchiffrer.
### 1.4  Assurez-vous du bon ordre des règles.

<div class="annotate">
Note : (1)
</div>
URL-CN est sous forme d'objet (catégorie d'url) qui regroupe plusieurs sites, exemple :

* URL-CN : **Online** = Sites de paris en ligne, réseaux sociaux...
* URL-CN : **News** = Journaux en ligne, sites de radiodiffusion, magazines..

## 2.  Exportez la CA publique depuis l’interface Web du SN210.
    Object → Certificat / PKI 
            - CLique droit sur le cerfiticat : SSL Proxy Default Authority
            - Le télécharger 

## 3. Installer la CA sur les postes clients

Sur chaque machine client, installez la CA interne dans les autorités
de certification racine de confiance : 
    - Windows
    - Linux 
    - macOS
    - Navigateurs si nécessaire (Firefox, Edge...)

## 4. Tester le déchiffrement SSL

### 4.1  Depuis un poste client, accédez à un site HTTPS.
### 4.2  Vérifiez que :
    -   Le site se charge.
    -   Le site chargé est bloqué avec une page violette qui est le Proxy avec le message suivant : 

    ```
    Your administrator reject the connection to this SSL Server 
    ```

    - Cela signifique que le Proxy est bel et bien fonctionnel !


## 5. Accéder à la configuration :
    Object → Certificat / PKI → Ajouter → Importer un fichier :
        - Importer le CA
        - Selectionner le format **PEM**
        - Ne pas mettre de mot de passe
        - Élements à importer : CA
    Puis valider l'importation !

### 5.1  Allez dans : 
    - Configuration → Protection applicative → Protocoles.
    - Sélectionnez *SSL*.
    - Cliquez sur *Accéder à la configuration globale*.
    - Autorités de certification personnalisées → Ajouter → Séléctionner le CA


4.1 Ajouter des CA personnalisées

-   Dans Autorités de certification personnalisées, ajoutez les CA
    internes / privées à considérer comme fiables.

4.3 Certificats de confiance

-   Dans Certificats de confiance, ajoutez les certificats serveurs
    explicitement approuvés.

5. Appliquer la CA signataire au proxy SSL

1.  Assurez-vous que la CA interne est bien sélectionnée comme CA
    signataire.
2.  Cliquez sur Appliquer pour sauvegarder.
