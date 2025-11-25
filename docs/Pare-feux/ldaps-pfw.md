# Configuration LDAPS Stormshield SN210


## 1. CONFIGURATION DES ANNUAIRES DISTANTS

Chemin interface web : Configuration > Utilisateurs > Configuration Annuaires > Ajouter un annuaire

Annuaire Distant :

| Paramètre                    | Valeur / Description                                          |
| ---------------------------- | ------------------------------------------------------------- |
| **Nom de domaine**           | cha.chartres.sportludique.fr                                  |
| **Serveur**                  | *Créer l'objet ActiveDirectory* avec l'adresse IP 10.10.120.2 |
| **Port**                     | LDAPS (636)                                                   |
| **Domaine Racine (Base DN)** | DC=cha,DC=chartres,DC=sportludique,DC=fr                      |
| **Identifiant**              | CN=admin.ldap,OU=UserServices                                 |
| **Mot de Passe**             | *Mot de passe de l'utilisateur de liaison : admin.ldap*       |


Connexion Sécurisée (SSL) :

| Paramètre                                                      | Valeur / Description        |
| -------------------------------------------------------------- | --------------------------- |
| **Activer l'accès en SSL**                                     | Oui *Cocher la case*        |
| **Vérifier le certificat selon une Autorité de certification** | Non *Ne pas cocher la case* |

Configuration Avancée :

| Paramètre                                                            | Valeur / Description                                                    |
| -------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| **Serveur de secours**                                               | *Créer l'objet ActiveDirectorySecondaire* avec l'adresse IP 10.10.120.3 |
| **Port serveur de secours**                                          | LDAPS (636)                                                             |
| **Utiliser le compte du firewall pour vérifier l’authentification**  | Non *Ne pas cocher la case*                                             |
| **Ne pas compléter l'identifiant (ID) par le Domain Name (Base DN)** | Non *Ne pas cocher la case*                                             |


---

## 2. AJOUT DES UTILISATEURS ADMINISTRATEURS TECHNICIENS DSI

Chemin interface web : Configuration > Système > Administrateurs > Ajouter un administrateur

- Rechercher les utilisateurs de la DSI concernés : wassim.dsi, simon.dsi, david.dsi
- Ajouter les utilisateurs un par un
- Accorder tous les droits administratifs à ces utilisateurs
- Sauvegarder en bas de page

---

## 3. PROBLÈMES RENCONTRÉS LORS DE LA CONFIGURATION

- Stormshield garde en mémoire le premier ajout d'annuaire, même si celui-ci n'était plus visible dans l'interface web.
- Dans les fichiers de configuration LDAP, l'annuaire externe peut rester présent, ce qui empêche de réinitialiser la configuration correctement.

---

:eight_pointed_black_star: **PROBLÈME RENCONTRÉ**

## RÉINITIALISER LES COMPTEURS LDAP (CLI)

Chemin interface web : Configuration > Système > Console CLI

- Vérifier les compteurs LDAP :
    ```
    config ldap count
    ```
    Si `External=1` ou `Internal=1`, procéder aux étapes suivantes.

- Supprimer l'annuaire problématique :
    ```
    config ldap remove domainname=cha.chartres.sportludique.fr force=on
    ```
    *Remplacer le nom de domaine par celui de votre annuaire concerné.*

- Mettre à jour la configuration :
    ```
    config ldap update
    ```

- Vérifier à nouveau les compteurs LDAP :
    ```
    config ldap count
    ```
    - Vérifier que `External=0` et `Internal=0` avant de refaire l’ajout correct des annuaires.

---

