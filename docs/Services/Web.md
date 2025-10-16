# Serveur WEB 

## 🎯 1. Objectifs 
Héberger un site web fais partie des services "basiques" d'une entreprise, dans notre cas nous allons implémenter un premier site web sur un serveur Apache2 heberger sur une machine Debian 13.

## 📦 2. Installation d'Apache2 
Mettez a jour votre système :
```
sudo apt update 
sudo apt full-upgrade
```
Installez les packages Apache2 
```
sudo apt install apache2
```

## 🔧 3. Configuration d'Apache2 

### Besoins 
<div class="annotate" markdown>
Dans notre cas ce serveur vas a terme heberger plusieurs sites internets, nous aurons donc besoin des VirtualsHosts. :light_bulb:(1)
</div>
1. Toute la documentation sur les VirtualsHosts est contenue sur [it-connect](https://www.it-connect.fr/les-vhosts-sous-apache2/)

### Création de la page

Nous allons tout dabord crée un nouveau répertoire et lui donner les bonnes permissions dans le dossier `/var/www/` afin que le VHost puisse avoir un Symlink qui pointe vers un site.
```
sudo mkdir /var/www/chartres
sudo chown -R www-data:www-data /var/www/ville
sudo chmod -R 755 /var/www/ville
```

Ensuite il nous faut crée le fichier index.html qui seras la page de base de notre site.
```
sudo nano /var/www/ville/index.html
```

Dans ce fichier nous allons crée une page des plus basiques juste pour pouvoir tester notre VHost par la suite. 
```
<html>
<head>
    <title>Bienvenue sur Chartres.sportludique</title>
</head>
<body>
    <h1>Bienvenue sur chartres.sportludique</h1>
</body>
</html>
```
### Création du fichier VHost

Nous allons crée le fichier de configuration VHost dans le dossier `/etc/apache2/sites-available` :
```
sudo nano /etc/apache2/sites-available/www.chartres.sportludique.fr.conf
```

Dans ce fichier de configuration nous avons copier et adapter la configuration du fichier 000-default.conf

```python hl_lines="1 11 12"
<VirtualHost 172.28.62.2:80>
    # The ServerName directive sets the request scheme, hostname and port that
    # the server uses to identify itself. This is used when creating
    # redirection URLs. In the context of virtual hosts, the ServerName
    # specifies what hostname must appear in the request's Host: header to
    # match this virtual host. For the default virtual host (this file) this
    # value is not decisive as it is used as a last resort host regardless.
    # However, you must set it for any further virtual host explicitly.
    #ServerName www.example.com

    ServerAdmin admin@chartres.sportludique.fr
    DocumentRoot /var/www/chartres

    # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
    # error, crit, alert, emerg.
    # It is also possible to configure the loglevel for particular
    # modules, e.g.
    #LogLevel info ssl:warn

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    # For most configuration files from conf-available/, which are
    # enabled or disabled at a global level, it is possible to
    # include a line for only one particular virtual host. For example the
    # following line enables the CGI configuration for this host only
    # after it has been globally disabled with "a2disconf".
    #Include conf-available/serve-cgi-bin.conf
</VirtualHost>
```

Dans notre cas nous modifions l'adresse sur laquel notre service écoute sois l'interface DMZ, l'adresse mail du responsable du site et enfin le chemin vers le site. 

### Ajout du lien Symlink 

Grâce a ce lien qui est l'equivalent d'un racourcis vers le site répertorier dans le fichier de configuration du VHost, le VHost iras directement cherche le site dna sle bon répertoire. 

```
sudo a2ensite www.chartres.sportludique.fr.conf
sudo systemctl reload apache2
```

Le VHost est maintenant configurer et prêt à fonctionner