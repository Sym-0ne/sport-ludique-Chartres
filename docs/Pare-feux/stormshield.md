# Configuration du Pare Feu Stormshield

## 🔄 1.Reset du pare feu 

 **Sur les boîtiers physiques:** un appui sur le bouton reset (attendre que les led devant clignotent) pour les boîtiers physiques permet de restaurer la configuration d'usine et redémarrer en bridge sur toutes les interfaces.

### Schema du pare feu après reset
 
 ![schema](PF/schema-pare-feu-apres-reset.png)

## 🖥️ 2.Connexion après reset 

 Pour configurer le pare-feu, il faut se brancher sur l'interface IN et mettre son poste en DHCP.

 En configuration usine sur un boîtier physique, toutes les interfaces sont incluses dans un **bridge dont l'adresse est 10.0.0.254/8**.Un serveur DHCP est actif sur toutes les interfaces du bridge et il distribue des adresses IP comprises entre 10.0.0.10 et 10.0.0.100. **L'accès à l'interface web** de configuration du pare-feu se fait avec l'url : **https://10.0.0.254/admin**.

 Par défaut, seul le compte système **admin (mot de passe par défaut admin)**, disposant de tous les privilèges sur le boîtier.

 ![page d'accueil](PF/page-d'accueil.png)

## 🔧 3.Configuration générale 

### Modification du mot de passe de l'administrateur

 La modification du mot de passe admin se fait dans le menu **Configuration/Système/Administrateurs puis onglet Compte ADMIN**.

 ![mdp admin](PF/mdp-admin.png)

 Puis cliquer sur **Appliquer**.

### Nom

 Sélectionner dans le menu à gauche **Configuration / Système puis Configuration Générale**.

 Commencer par donner un nom à votre boîtier et changer la langue de la console.

 ![nom](PF/nom.png)

 Puis cliquer sur **Appliquer**.

### Fuseau horaire

 La zone « Paramètres de date et d'heure » permet de modifier le fuseau horaire dans la zone Fuseau horaire, sélectionnez **Europe/Paris**.

 ![heure](PF/heure.png)

 Après le redémarrage (au bout d'environ 3 minutes), revenir au menu Configuration / Système puis Configuration et dans la zone Paramètres de date et d'heure cliquer sur **Maintenir le pare-feu à l'heure (NTP) pour que les mises à jour d'heure d'été/heure d'hiver soient également effectives**.

 Puis cliquer sur **Appliquer**.

## 🔧 4.Configuration du réseau 

 Toute les interfaces sont dans le **bridge**.

 ![bridge](PF/bridge.png)

 Choisir une interface (par exemple IN), pour la **sortir du bridge et la configurer avec une IP fixe**.

 ![IP](PF/IP.png)

 Puis cliquer sur **Appliquer**.

 Faire pareil avec les autres interfaces (WAN,DMZ).

 ![WAN/DMZ](PF/interfaces.png)

## 🛣️ 5.Routage 

### Route par défaut 

 Cliquer **Configuration / Réseau / Routage / Routes statiques IPv4**.

 ![route](PF/route.png)

 Cliquer sur l'icône ![icone](PF/icone.png) pour ajouter un objet réseau, choisir **Machine et renseigner les champs Nom et Adresse IPv4** du pare-feu puis cliquer sur le bouton Créer.

 ![passerelle](PF/gateway.png)

 Puis cliquer sur **Appliquer**.

### Route de retour

 ![route de retour](PF/route-retour.png)

## 🧱 6.Filtrage 

 Allez dans **Configuration / Politique de sécurité / Filtrage et NAT**.

 Choisir la régle "**Pass all**" et modifier la colonne "**Inspection de sécurité**" en mettant "**Ne pas inspecter**".

 ![filtrage](PF/filtrage.png)

 ![régle](PF/regle.png)

 Puis cliquer sur **Appliquer**.

## ⚠️ 7. Statefull Inspection

Le stormshield a par défaut un inspection des paquets qui n'est pas désactivable, cette inspection peut rejeter silencieusement des paquets sans même l'enregistrer dans les logs, dans notre cas cette inspection pose problème avec le TCP handcheck.

Ce principe est la base du protocole TCP, il sagit d'une liaison des états SYN, SYN-ACK et ACK, le client envoie un SYN, le serveur envoie le paquet avec un SYN-ACK au client puis le client renvoie un ACK au serveur, cela permet au client de savoir que le paquet a été reçus par le seruveur grâce au SYN-ACK puis au serveur de savoir que le client a bien reçus le paquet grâce au ACK. Ce principe permet la vérification des "étapes" de transition d'un paquet.

Ce qui veux dire que Stormshield refuse les paquets TCP dont il n'a aucune trace d'initialisation en mémoire ou qu'il n'a pas initialiser lui même, de ce fait, si le Stormshield reçois un SYN-ACK sans avoir reçus de SYN sur la même session, il ne considère pas avoir démaré la session TCP et ne répond donc pas a celle ci.