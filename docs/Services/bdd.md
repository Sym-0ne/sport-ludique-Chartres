# Base De Données

---

## 1. Installation

Pour commencer, mettez à jour votre système :
```
sudo apt update
sudo apt full-upgrade
```
Installez le service MariaDB :
```
sudo apt install mariadb-server mariadb-client 
```
Vérifiez que le service est actif : 
```
sytemctl status mariadb
```
<div class="annotate" markdown>

Si le service n'es pas actif (1)

</div>

1. ```
systemctl enable mariadb 
systemctl start mariadb```

---

## 2. Sécurisation basique

Nous allons sécuriser l'acces a la base de donnée grâce aux système intégrées a mariaDB
```
sudo mysql_secure_installation 
    NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB SERVERS IN PRODUCTION USE! PLEASE READ EACH STEP CAREFULLYI
 
In order to log into MariaDB to secure it, we'll need the current password for the root user. If you've just installed MariaDB, and haven't set the root password yet, you should just press enter here,
 
Enter current password for root (enter for none):
```
Choisiez si vous voulez changer le socket d'authentification pour unix :
```
Enter current password for root (enter for none); OK, successfully used password, moving on.
 
Setting the root password or using the unix_socket ensures that nobody can log into the MariaDB root user without the proper authorisation,
 
You already have your root account protected, so you can safely answer
 
Switch to unix socket authentication [Y/n] : n
```
<div class="annotate" markdown>

Info (1)

</div>

1. Dans notre cas, comme indiqué sur la ligne de commande, le compte root est déja sécurisé, nul besoin d'activer ce socket d'authentification

Changer le mot de passe root si besoin : 
```
you already have your root account protected, so you can safely answer 'n'
 
Change the root password? [Y/n]: n
```

Suprimmer les utilisateurs anonymes : 

```
By default, a MariaDB installation has an anonymous user, allowing anyone to log into MariaDB without having to have a user account created for them. This is intended only for testing, and to make the installation go a bit smoother. You should remove them before moving into a
 
production environment.
 
Remove anonymous users? [Y/n]: y
```

<div class="annotate" markdown>

Désactivez le login a distance via le compte root si besoin (1)

</div>

1. Dans notre cas, tout les accès se feront via SSH donc nous laisserons l'option active

Suprimez la base de donnée test : 

```
By default, MariaDB comes with a database named 'test' that anyone can access, This is also intended only for testing, and should be removed before moving into a production environment,
 
Remove test database and access to it? [Y/n] y
```
 
Pour finir, metez la table des privileges a jour pour appliquer vos modifications : 
```
Reloading the privilege tables will ensure that all changes made so far will take effect immediately,
 
Reload privilege tables now? [Y/n] y ... Success!
 
Cleaning up...
 
All done! If you've completed all of the above steps, your MariaDB installation should now be secure.
 
Thanks for using MariaDBI bdd@CHA-BDD:"$
```

---