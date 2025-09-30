# Configuration du Pare Feu Stormshield

## 1.Reset du pare feu

**Sur les boîtiers physiques :** Un appui sur le bouton reset (attendre que les Led de la facade clignotent) pour les boîtiers physiques permet de restaurer la configuration d'usine et redémarrer en bridge sur toutes les interfaces.

### Schema du pare feu après reset
 
 ![schema](PF/schema-pare-feu-apres-reset.png)

## 2.Connexion après reset

Pour configurer le pare-feu, il faut se brancher sur l'interface "IN" et mettre son poste en **DHCP**.

En configuration usine sur un boîtier physique, toutes les interfaces sont incluses dans un **bridge dont l'adresse est 10.0.0.254/8**. 
Un serveur DHCP est actif sur toutes les interfaces du bridge et il distribue des adresses IP comprises entre 10.0.0.10 et 10.0.0.100. 
**L'accès à l'interface web** de configuration du pare-feu SNS se fait avec l'url : **https://10.0.0.254/admin**.

Par défaut, seul le compte système **admin (mot de passe par défaut admin)**, disposant de tous les privilèges sur le boîtier.

