==============================
# Installation de WordPress sur Debian avec serveur MySQL distant
===============================================================

## 1. Prérequis sur le serveur web (10.10.120.11)

Installer Apache, PHP et extensions nécessaires :

```bash
sudo apt update
sudo apt install apache2 php php-mysql libapache2-mod-php php-cli php-curl php-xml php-mbstring unzip wget -y
```

## 2. Télécharger et placer WordPress

```bash
cd /var/www/
sudo wget https://wordpress.org/latest.zip
sudo unzip latest.zip
sudo mv wordpress /var/www/html/
```

## 3. Configurer les droits

```bash
sudo chown -R www-data:www-data /var/www/html/wordpress
sudo chmod -R 755 /var/www/html/wordpress
```

## 4. Créer la base de données MySQL (10.10.120.7)

Se connecter à MySQL :

```bash
mysql -u wp_user -p -h 10.10.120.7
```

Puis créer la BDD et l'utilisateur :

```sql
CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'wp_user'@'10.10.120.11' IDENTIFIED BY 'mot_de_passe_fort';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wp_user'@'10.10.120.11';
FLUSH PRIVILEGES;
EXIT;
```

Modifier le fichier MySQL pour autoriser les connexions distantes :

* Éditer `/etc/mysql/mysql.conf.d/mysqld.cnf` et modifier `bind-address = 127.0.0.1` et mettre `bind-address = 10.10.120.11`
* Redémarrer MySQL : `sudo systemctl restart mysql`

## 5. Configurer WordPress

```bash
cd /var/www/html/wordpress
sudo cp wp-config-sample.php wp-config.php
sudo nano wp-config.php
```

Modifier :

```php
define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wp_user' );
define( 'DB_PASSWORD', 'mot_de_passe_fort' );
define( 'DB_HOST', '10.10.120.7' );
```

## 6. Configurer Apache

Créer un VirtualHost :

```bash
sudo nano /etc/apache2/sites-available/wordpress.conf
```

Exemple de configuration :

```apache
<VirtualHost *:80>
    ServerAdmin admin@10.10.120.11
    DocumentRoot /var/www/html/wordpress
    ServerName 10.10.120.11

    <Directory /var/www/html/wordpress>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/wordpress_error.log
    CustomLog ${APACHE_LOG_DIR}/wordpress_access.log combined
</VirtualHost>
```

Activer le site et le module rewrite, désactiver le site par défaut :

```bash
sudo a2ensite wordpress.conf
sudo a2enmod rewrite
sudo a2dissite 000-default.conf
sudo systemctl reload apache2
```

## 7. Installer l’extension MySQL pour PHP

Si page blanche avec message "Your PHP installation appears to be missing the MySQL extension":

```bash
sudo apt install php-mysql -y
sudo systemctl restart apache2
php -m | grep mysqli   # vérifier que mysqli est actif
```

## 8. Activer l’affichage des erreurs PHP (pour debug)

Éditer le fichier php.ini :

```bash
sudo nano /etc/php/*la version du php*/apache2/php.ini
```

Modifier ou ajouter :

```ini
display_errors = On
```

Puis redémarrer Apache :

```bash
sudo systemctl restart apache2
```

## 9. Accéder à WordPress

Dans ton navigateur :

```
http://10.10.120.11/
```

Tu devrais voir l’écran d’installation de WordPress (choix de la langue, création compte admin, etc.).

---

