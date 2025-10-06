# Script PowerShell — Création d'une Structure Active Directory

## 1. Objectif

Ce script PowerShell automatise la **création d’une arborescence Active Directory** pour le domaine  
**cha.chartres.sportludique.fr**.  

Le script permet de créer automatiquement :
- Les **Unités Organisationnelles (OU)** des différents services.  
- Les **utilisateurs** correspondant aux rôles du projet.  
- Les **groupes de sécurité** associés.  
- Et une **vérification finale** pour s’assurer du bon déploiement.

---

## 2. Présentation générale

| Élément                     | Détails                                                                 |
|-----------------------------|--------------------------------------------------------------------------|
| **Langage**                 | PowerShell                                                              |
| **Module requis**           | `ActiveDirectory`                                                       |
| **Domaine**                 | `cha.chartres.sportludique.fr`                                          |
| **Unité de base (root)**    | `DC=cha,DC=chartres,DC=sportludique,DC=fr`                              |
| **OU créées**               | Direction, Marketing, Finances, Ressources Humaines, DSI                |
| **Groupes créés**           | `G_DSI`, `G_RH`                                                         |
| **Utilisateurs créés**      | 5 (DSI + RH)                                                            |
| **Mot de passe par défaut** | `P@ssword123` *(modifiable dans la variable `$Password`)*               |

---

## 3. Contexte

- Les utilisateurs **Claude Postic** (Directeur DSI) et **Helen Paisley-Le Bihan** (Directrice RH) ont été créés **car le scénario du projet l’exigeait**, représentant la direction de deux pôles essentiels de l’entreprise.  
- Les utilisateurs **David**, **Wassim** et **Simon** représentent les **techniciens de la DSI**.  
  > Ces trois comptes correspondent en réalité aux **administrateurs techniques et réalisateurs du projet BTS SIO SISR**.  
  Ils disposent de **droits administratifs étendus** sur l’infrastructure et servent à la gestion et à la maintenance du système.

Ainsi, la structure reflète à la fois le **besoin fonctionnel** et le **besoin technique** (accès administrateur pour les concepteurs du projet).

---

## 4. Contenu du script

```
powershell
Import-Module ActiveDirectory

# ============================
# 1. Création des OU
# ============================
New-ADOrganizationalUnit -Name "Direction" -Path "DC=cha,DC=chartres,DC=sportludique,DC=fr"
New-ADOrganizationalUnit -Name "Marketing" -Path "DC=cha,DC=chartres,DC=sportludique,DC=fr"
New-ADOrganizationalUnit -Name "Finances" -Path "DC=cha,DC=chartres,DC=sportludique,DC=fr"
New-ADOrganizationalUnit -Name "Ressources Humaines" -Path "DC=cha,DC=chartres,DC=sportludique,DC=fr"
New-ADOrganizationalUnit -Name "DSI" -Path "DC=cha,DC=chartres,DC=sportludique,DC=fr"

# ============================
# 2. Variables générales
# ============================
$Domaine = "cha.chartres.sportludique.fr"
$Password = ConvertTo-SecureString "P@ssword123" -AsPlainText -Force

# ============================
# 3. Création des utilisateurs
# ============================

# --- DSI ---
New-ADUser -Name "Claude Postic" -GivenName "Claude" -Surname "Postic" `
-DisplayName "Claude Postic" -SamAccountName "claude.dsi" `
-UserPrincipalName "claude.dsi@$Domaine" `
-Path "OU=DSI,DC=cha,DC=chartres,DC=sportludique,DC=fr" `
-AccountPassword $Password -Enabled $true -Title "Directeur DSI"

New-ADUser -Name "David" -GivenName "David" -Surname "Technicien" `
-DisplayName "David" -SamAccountName "david.dsi" `
-UserPrincipalName "david.dsi@$Domaine" `
-Path "OU=DSI,DC=cha,DC=chartres,DC=sportludique,DC=fr" `
-AccountPassword $Password -Enabled $true -Title "Technicien DSI"

New-ADUser -Name "Wassim" -GivenName "Wassim" -Surname "Technicien" `
-DisplayName "Wassim" -SamAccountName "wassim.dsi" `
-UserPrincipalName "wassim.dsi@$Domaine" `
-Path "OU=DSI,DC=cha,DC=chartres,DC=sportludique,DC=fr" `
-AccountPassword $Password -Enabled $true -Title "Technicien DSI"

New-ADUser -Name "Simon" -GivenName "Simon" -Surname "Technicien" `
-DisplayName "Simon" -SamAccountName "simon.dsi" `
-UserPrincipalName "simon.dsi@$Domaine" `
-Path "OU=DSI,DC=cha,DC=chartres,DC=sportludique,DC=fr" `
-AccountPassword $Password -Enabled $true -Title "Technicien DSI"

# --- Ressources Humaines ---
New-ADUser -Name "Helen Paisley-Le Bihan" -GivenName "Helen" -Surname "Paisley-Le Bihan" `
-DisplayName "Helen Paisley-Le Bihan" -SamAccountName "helen.rh" `
-UserPrincipalName "helen.rh@$Domaine" `
-Path "OU=Ressources Humaines,DC=cha,DC=chartres,DC=sportludique,DC=fr" `
-AccountPassword $Password -Enabled $true -Title "Directrice RH"

# ============================
# 4. (Optionnel) Création des groupes
# ============================
New-ADGroup -Name "G_DSI" -GroupScope Global -Path "OU=DSI,DC=cha,DC=chartres,DC=sportludique,DC=fr"
New-ADGroup -Name "G_RH" -GroupScope Global -Path "OU=Ressources Humaines,DC=cha,DC=chartres,DC=sportludique,DC=fr"

Add-ADGroupMember -Identity "G_DSI" -Members "claude.dsi","david.dsi","wassim.dsi","simon.dsi"
Add-ADGroupMember -Identity "G_RH" -Members "helen.rh"

# ============================
# 5. Vérification
# ============================
Write-Host "`n✅ Structure AD créée avec succès !`n"
Get-ADOrganizationalUnit -Filter * | Select-Object Name
Get-ADUser -Filter * | Select-Object Name, SamAccountName, DistinguishedName
```
