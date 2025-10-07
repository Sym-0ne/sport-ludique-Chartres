# Documentation — Intégration de Proxmox avec Active Directory (AD)

## 📋 Objectif

Ce document détaille les étapes permettant de :

1. Créer un utilisateur et un groupe d’administration Proxmox dans Active Directory (AD).  
2. Intégrer le domaine Active Directory dans Proxmox VE pour l’authentification centralisée.  
3. Configurer les permissions dans Proxmox.  
4. Gérer le mode d’authentification LDAP / LDAPS.

---

## ⚙️ Étape 1 — Création des utilisateurs et groupes AD

### 🧩 Description

Le script PowerShell suivant permet de :

- Créer l’OU `UserServices` si elle n’existe pas.
- Créer l’utilisateur `proxmox.admin`.
- Créer le groupe `G_Proxmox_Admins`.
- Ajouter l’utilisateur au groupe.
- Vérifier la création et l’appartenance.

---

### 💻 Script PowerShell

```
Import-Module ActiveDirectory

# ============================
# 1. Variables générales
# ============================
$Domaine = "cha.chartres.sportludique.fr"
$OUPath = "OU=UserServices,DC=cha,DC=chartres,DC=sportludique,DC=fr"
$NomUtilisateur = "proxmox.admin"
$NomComplet = "proxmox"
$MotDePasse = ConvertTo-SecureString "CHAPVE2526!" -AsPlainText -Force
$NomGroupe = "G_Proxmox_Admins"

# ============================
# 2. Vérification / Création de l'OU
# ============================
if (-not (Get-ADOrganizationalUnit -Filter "Name -eq 'UserServices'" -ErrorAction SilentlyContinue)) {
    Write-Host "📁 Création de l'OU 'UserServices'..."
    New-ADOrganizationalUnit -Name "UserServices" -Path "DC=cha,DC=chartres,DC=sportludique,DC=fr"
} else {
    Write-Host "✅ L'OU 'UserServices' existe déjà."
}

# ============================
# 3. Création de l'utilisateur Proxmox
# ============================
if (-not (Get-ADUser -Filter "SamAccountName -eq '$NomUtilisateur'" -ErrorAction SilentlyContinue)) {
    Write-Host "👤 Création de l'utilisateur '$NomUtilisateur'..."
    New-ADUser `
        -Name $NomComplet `
        -GivenName "Proxmox" `
        -Surname "Admin" `
        -SamAccountName $NomUtilisateur `
        -UserPrincipalName "$NomUtilisateur@$Domaine" `
        -Path $OUPath `
        -AccountPassword $MotDePasse `
        -Enabled $true `
        -ChangePasswordAtLogon $false `
        -PasswordNeverExpires $true `
        -Description "Compte d'administration Proxmox lié à l'AD" `
        -Title "Service Infrastructure"
    Write-Host "✅ Utilisateur '$NomUtilisateur' créé avec succès."
} else {
    Write-Host "ℹ️ L'utilisateur '$NomUtilisateur' existe déjà, aucune action nécessaire."
}

# ============================
# 4. Création du groupe G_Proxmox_Admins
# ============================
if (-not (Get-ADGroup -Filter "Name -eq '$NomGroupe'" -ErrorAction SilentlyContinue)) {
    Write-Host "👥 Création du groupe '$NomGroupe'..."
    New-ADGroup `
        -Name $NomGroupe `
        -GroupScope Global `
        -GroupCategory Security `
        -Path $OUPath `
        -Description "Groupe des administrateurs Proxmox"
    Write-Host "✅ Groupe '$NomGroupe' créé avec succès."
} else {🖥️ Étape 2 — Intégration du domaine AD dans Proxmox
1️⃣ Création du Realm (Royaume)

Chemin d’accès :
Datacenter → Permissions → Realms → Add → Active Directory
Champ	Valeur
Realm	ad
Default Domain	cha.chartres.sportludique.fr
Port	389 (par défaut)
Mode	LDAP (par défaut)
Server1 / Server2	10.10.120.2 et 10.10.120.3
Commentaire	Authentification AD
    Write-Host "✅ Le groupe '$NomGroupe' existe déjà."
}

# ============================
# 5. Ajout de l'utilisateur au groupe
# ============================
if (-not (Get-ADGroupMember -Identity $NomGroupe | Where-Object {$_.SamAccountName -eq $NomUtilisateur})) {
    Write-Host "➕ Ajout de l'utilisateur '$NomUtilisateur' au groupe '$NomGroupe'..."
    Add-ADGroupMember -Identity $NomGroupe -Members $NomUtilisateur
    Write-Host "✅ Utilisateur ajouté au groupe avec succès."
} else {
    Write-Host "ℹ️ L'utilisateur '$NomUtilisateur' est déjà membre du groupe '$NomGroupe'."
}

# ============================
# 6. Vérification finale
# ============================
Write-Host "`n🎯 Vérification finale :"
Get-ADUser -Identity $NomUtilisateur -Properties Name,SamAccountName,DistinguishedName,PasswordNeverExpires | 
Select-Object Name, SamAccountName, DistinguishedName, PasswordNeverExpires

Get-ADGroupMember -Identity $NomGroupe | 
Select-Object Name, SamAccountName

Write-Host "`n✅ Script exécuté avec succès."
```

## Étape 2 — Intégration du domaine AD dans Proxmox

### 1. Création du Realm (Royaume)

Chemin d’accès :

```
Datacenter → Permissions → Realms → Add → Active Directory
```

| Champ                 | Valeur                       |
| --------------------- | ---------------------------- |
| **Realm**             | ad                           |
| **Default Domain**    | cha.chartres.sportludique.fr |
| **Port**              | 389 *(par défaut)*           |
| **Mode**              | LDAP *(par défaut)*          |
| **Server1 / Server2** | 10.10.120.2 et 10.10.120.3   |
| **Commentaire**       | Authentification AD          |


### 2. Création de l’utilisateur Proxmox dans l’interface

Chemin d’accès :

```
Datacenter → Permissions → Utilisateurs → Add
```

| Champ               | Valeur              |
| ------------------- | ------------------- |
| **Nom utilisateur** | proxmox.admin       |
| **Royaume**         | Authentification AD |
Appliquer la GPO puis forcer la mise à jour :

### 3. Attribution des permissiation LDAP / LDAPS.
ons

Chemin d’accès :

```
Datacenter → Permissions → Add → Permissions de l'utilisateur
```

| Champ              | Valeur              |
| ------------------ | ------------------- |
| **Chemin d'accès** | `/` *(accès total)* |
| **Utilisateur**    | proxmox.admin@ad    |
| **Rôle**           | Administrator       |
| **Propager**       | ✅ Activé            |


💡 Note : **Le mode LDAP est temporaire**. Une migration vers **LDAPS** sera effectuée pour sécuriser les échanges.

## Etape 3 — Gestion du protocole LDAP / LDAPS
Appliquer la GPO puis forcer la mise à jour :
⚠️ Problématique

Depuis Windows Server 2019 (patch 2020) et renforcé dans Windows Server 2022 / 2025,
les connexions **LDAP simples non chiffrées (port 389)** sont désormais **bloquées ou limitées.**

Erreur typique :

```
can't contact LDAP server
```

**Raison :**
Les identifiants sont envoyés en clair sur le réseau.

**Recommandation :**
Utiliser LDAPS (port 636) pour chiffrer les échanges entre Proxmox et Active Directory.


**Méthode : via la Stratégie de groupe (GPO)**

- Ouvrir la console de gestion des stratégies de groupe (GPMC).
- Éditer la GPO appliquée aux contrôleurs de domaine (ou créer une dédiée).

**Naviguer vers :**

```
Configuration ordinateur
   → Stratégies
      → Paramètres Windows
         → Paramètres de sécurité
            → Stratégies locales
               → Options de sécurité
```

Modifier les paramètres suivants :

| Paramètre                                                                                       | Valeur                       |
| ----------------------------------------------------------------------------------------------- | ---------------------------- |
| Contrôleur de domaine : conditions requises pour la signature de serveur LDAP                   | Aucun                        |
| Contrôleur de domaine : application des conditions requises pour la signature de serveur LDAP   | Désactivé                    |
| Contrôleur de domaine : configuration requise pour le jeton de liaison du canal du serveur LDAP | Lorsqu’il est pris en charge |

Appliquer la GPO puis forcer la mise à jour :

```
gpupdate /force
```