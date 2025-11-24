# Mise en place du Multi Site sur WordPress.        

Site (Wassim) WordPress : ```www.cimmob.chartres.sportludique.fr```
Site (David) WordPress : ```www.david.cimmob.chartres.sportludique.fr```
Site (Simon) WordPress : ```www.simon.cimmob.chartres.sportludique.fr```

## Prérequis

- Serveur WordPress : ```172.28.62.3```
- Serveur Base de données : ```192.168.28.10```
- Reverse Proxy : ```172.28.62.5 (Nginx)```
- DNS interne : ```Bind9 (serveur autorité interne)```
- WordPress installé dans : ```/var/www/html/wordpress```

---

## 1. Choix du type de multisite
-----------------------------

WordPress permet deux modes de multisite :

A) Sous-domaines  
   Exemple : site1.domain.fr, site2.domain.fr  
   → Nécessite la configuration DNS (wildcard ou enregistrements individuels).

B) Sous-répertoires  
   Exemple : domain.fr/site1, domain.fr/site2  
   → Ne nécessite aucune modification DNS.

Ici, nous choisirons la **méthode A** à savoir les *Sous-domaines*.

---

## 2. Activer le mode multisite
-----------------------------

Modifier le fichier wp-config.php (à la racine de WordPress).

Ajouter avant la ligne : **/* That's all, stop editing! */** :

```define('WP_ALLOW_MULTISITE', true);```

Enregistrer et recharger l’interface d’administration WordPress.

---

## 3. Installer le réseau multisite
---------------------------------

Dans le tableau de bord WordPress :

Outils > Configuration du Réseau

Choisir :
- Sous-domaines

Puis cliquer sur “Installer”.

---

## 4. Ajouter les lignes demandées par WordPress
---------------------------------------------

Une fois l’installation lancée, WordPress fournit deux blocs de code à ajouter :

A) Dans wp-config.php :

Exemple de lignes (varient selon configuration) :

```define('MULTISITE', true);
define('SUBDOMAIN_INSTALL', true);
define('DOMAIN_CURRENT_SITE', 'chartres.sportludique.fr');
define('PATH_CURRENT_SITE', '/');
define('SITE_ID_CURRENT_SITE', 1);
define('BLOG_ID_CURRENT_SITE', 1);
```

B) Dans .htaccess :

Remplacer toute la partie WordPress par le bloc fourni, exemple :

```RewriteEngine On
RewriteBase /
RewriteRule ^index\.php$ - [L]

RewriteCond %{REQUEST_FILENAME} -f [OR]
RewriteCond %{REQUEST_FILENAME} -d
RewriteRule ^ - [L]
RewriteRule . index.php [L]
```

(WordPress fournit exactement les lignes adaptées, il faut respecter ce qu’il montre.)

---

## 5. Reconnexion
--------------

Après les modifications, WordPress demande de se reconnecter.  
Vous êtes maintenant en mode multisite.

---

## 6. Créer les sites pour chaque utilisateur
------------------------------------------

Aller dans :

Mes sites > Réseau admin > Sites > Ajouter

Créer un site par personne, exemple :

- Domaine : cimmob.chartres.sportludique.fr
- Domaine : david.cimmob.chartres.sportludique.fr
- Domaine : simon.cimmob.chartres.sportludique.fr

Chaque site est totalement indépendant.

---

## 7. Configuration DNS & Proxy (si vous utilisez les sous-domaines)
----------------------------------------------------------

Mettre en place la meme configuration pour tout les noms de domaines qui ont été créer via le Multisite grace aux explciations présentes sur les pages suivantes : 

Vous pouvez aller voir la documention pour le [Reverse Proxy & DNS](https://sym-0ne.github.io/sport-ludique-Chartres/Services/wordpress2/).

---

## 8. Gestion des utilisateurs
---------------------------

Dans :

Réseau admin > Utilisateurs > Ajouter

Créer ou associer un utilisateur à un site spécifique en lui donnant le rôle “Administrateur” uniquement sur son site (et non sur le réseau entier).

---