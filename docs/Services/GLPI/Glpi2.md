# Mise en place du LDAP entre l'AD & GLPI 

## Objectif :
----------

La mise en place de la liaison LDAP entre l'AD & GLPI va permettre à tous les utilisateurs du domaine **chartres.sportludique** de pouvoir se connecter au GLPI afin de faire des tickets, demandes et autres.<br>
Ainsi qu'aux utilisateurs de la DSI de gérer les configurations, tickets, demandes faites par les clients grâce aux habilitations *Super-Admin* qui leur seront attribués.

--- 

## 1. Prérequis 

-   Serveur GLPI fonctionnel (mis en place sur un Docker dans le cas présent).
-   Serveur AD fonctionnel avec les utilistaeurs.

--- 

## 2. Configuration annuaire LDAP

Connectez-vous à l'interface GLPI avec un compte administrateur, puis dans le menu : Configuration → Authentification → Annuaire LDAP → Ajouter.

Vous devez être rendu sur cette interface : 

![annuaire](ldap2-glpi.png)

Voici les valeurs à remplir dans notre cas : 

| **Paramètre**                | **Valeur**                                                                                    |
| ---------------------------- | --------------------------------------------------------------------------------------------- |
| **Nom**                      | Active Directory                                                                              |
| **Serveur par défaut**       | Oui                                                                                           |
| **Actif**                    | Oui                                                                                           |
| **Serveur**                  | 10.10.120.2                                                                                   |
| **Port**                     | 389                                                                                           |
| **Filtre de connexion**      | *Laisser le champ vide*                                                                       |
| **BaseDN**                   | DC=cha,DC=chartres,DC=sportludique,DC=fr                                                      |
| **Utiliser bind**            | Oui                                                                                           |
| **DN du compte**             | admin.ldap                                                                                    |
| **Mot de passe du compte**   | *Mot de passe du compte - voir keepass*                                                       |
| **Champ de l'identifiant**   | SamAccountName                                                                             |
| **Champ de synchronisation** | *Laisser le champ vide*                                                                       |

Dans la foulée, GLPI va effectuer un test de connexion LDAP et vous indiquer s'il est parvenu, ou non, à se connecter à votre annuaire. Si ce n'est pas le cas , cliquez sur le nom de votre annuaire, vérifiez la configuration, puis retournez dans **"Tester"** sur la gauche afin de lancer un nouveau test. 

---

## 3. Importation des utilisateurs de l'AD

A partir de GLPI, vous pouvez forcer une synchronisation LDAP de façon à mettre à jour les comptes dans GLPI "liés" à des comptes Active Directory, mais aussi pour importer en masse tous les comptes des utilisateurs Active Directory. Ceci vous évite d'attendre la première connexion et vous permet de préparer le compte : attribution du bon rôle, etc.

Cliquez sur "Administration" dans le menu, puis "Utilisateurs". Ici, vous avez accès au bouton "Liaison annuaire LDAP".

![annuaire-ldap](ldap1-glpi.png)

Si vous cliquez sur **"Importation de nouveaux utilisateurs"**, vous pourrez importer en masse les comptes dans l'Active Directory. Il vous suffit de lancer une recherche, de sélectionner les comptes à importer et de lancer l'import grâce au bouton **"Actions"**.

---

## 4. Attribution des habilitations aux utilisateurs DSI (Admin)

Rendez-vous dans le menu : Administration → Utilisateurs → *Cliquer sur le profil souhaité* → Habilitations.

Une fois rendu ici, remplissez les champs suivants avant de cliquer sur **Ajouter**: 

| **Entité**                | **Profil**                                                                   | **Récursif** |
| --------------------------| -----------------------------------------------------------------------------|--------------|
| **Entrée Racine**         | *Selectionner le profil avec les droits souhaités : Super-Admin pour la DSI* | Non          |

Dès lors que l'entité apparaît dans la liste, l'utilisateur a donc les habilitations sélectionnées au préalable.

---