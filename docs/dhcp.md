# Installer et configurer un serveur DHCP sur Windows Server 2025.

## Etape 1. Installation du rôle DHCP
Ouvrir le Gestionnaire de serveur.
Cliquer sur Gérer, puis sur Ajouter des rôles et fonctionnalités.

Choisir : « Installation basée sur un rôle ou une fonctionnalité ».

![Installation du role DHCP](images/DHCP/1.png)

Sélectionner le serveur local.

![Selection serveur local](images/DHCP/2.png)

Cocher **Serveur DHCP**, ajouter les fonctionnalités proposées.

![Ajouter fonctionnalités dhcp](images/DHCP/3.png)

**Nous n’allons pas ajouté de fonctionnalité**, nous pouvons faire suivant : 

![Selectionner dhcp](images/DHCP/4.png)

Dérouler la petite flèche pour accéder aux options :

![Menu DHCP](images/DHCP/5.png)

Faire une nouvelle étendue : 

![Nvl etendue dhcp](images/DHCP/6.png)

Cliquer sur suivant pour passer à la prochaine étape : 

![Suivant](images/DHCP/7.png)

Nommer l'étendue et une description pour préciser sa signification **(La description est facultative)**

![Nommer etendue](images/DHCP/8.png)

Remplir l'entendue avec les adresses IP correspondantes aux besoins.

![Remplir IP](images/DHCP/9.png)

Remplir les adresses IP exclu des étendues DHCP.

![Exclure IP](images/DHCP/10.png)

Ne pas toucher au bail, cliquer juste sur Suivant.

![Suivant](images/DHCP/11.png)

Confirmer la mise en place de l'etendue DHCP.

![Nommer etendue](images/DHCP/12.png)

Ajouter une passerelle par défaut pour le routeur qui permettra l'accès à Internet.

![Gateway](images/DHCP/13.png)

Ajouter un nom de Domaine ainsi que l'adresse correspondante au serveur DNS.

![Domaine et IP DNS](images/DHCP/14.png)

On ne mettra pas de serveur WINS, cliquer sur Suivant.

![Serv WINS](images/DHCP/15.png)

Confirmer l'activation de l'etendue.

![Activation etendue](images/DHCP/16.png)

Cliquer sur Suivant.

![Suivant](images/DHCP/17.png)

Dérouler les flèches afin de vérifier si les extensions apparaissent bien dans le menu DHCP.

![Verif](images/DHCP/18.png)



