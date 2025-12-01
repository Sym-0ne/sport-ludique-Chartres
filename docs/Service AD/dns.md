# Installer et configurer un serveur DNS sur Windows Server.

Dans l’assistant, sélectionnez **Installation basée sur un rôle ou une fonctionnalité** :  
![Assistant d’installation](DNS/1.png)

Choisissez votre serveur dans la liste proposée :  
![Choix du serveur](DNS/2.png)

Dans la section des rôles, sélectionnez le rôle **DNS** :  
![Sélection du rôle DNS](DNS/3.png)

Passez ensuite à la section **Confirmation** et cliquez sur **Installer** pour lancer l’installation.

----------------------------------------------------------

## 1. Configuration du service DNS 

Dans l’onglet **Outils**, sélectionnez le service **DNS** :  
![Ouverture du service DNS](DNS/4.png)

Effectuez un clic droit sur votre serveur puis cliquez sur **Configurer un serveur DNS** :  
![Configuration du serveur DNS](DNS/5.png)

Créez une **zone de recherche directe** :  
![Création de la zone](DNS/6.png)

Notre serveur gère la zone entière et délègue les requêtes inconnues au DNS de notre FAI :  
![Gestion de la zone](DNS/7.png)

Ajoutez le nom de la zone de recherche, dans notre cas **chartres** :  
![Nom de la zone](DNS/8.png)

Par mesure de sécurité, seules les **mises à jour dynamiques sécurisées** seront autorisées :  
![Mises à jour dynamiques](DNS/9.png)

Ajoutez l’adresse IP du DNS public de votre choix. Dans notre cas, nous utilisons le serveur du réseau Internet fictif :  
![Ajout du DNS public](DNS/10.png)

Enfin, enregistrez votre nouvelle **zone de recherche DNS** :  
![Enregistrement de la zone](DNS/11.png)
