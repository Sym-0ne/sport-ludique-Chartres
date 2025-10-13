# Script PowerShell — Création d'une Structure Active Directory

## 1. Objectif

Ce script PowerShell automatise la **création d’une arborescence Active Directory** pour le domaine  
**cha.chartres.sportludique.fr**.  

Le script permet de créer automatiquement :<br>
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

## 5. Attribution des droits Administrateurs aux utilisateurs de la DSI

### 1. Le groupe “Admins du domaine”

#### 1.1 Définition

Le groupe **“Admins du domaine”** est un groupe de sécurité intégré créé automatiquement lors de l’installation d’un domaine Active Directory (équivalent anglais : Domain Admins).
Il est situé dans le conteneur :

```
CN=Users,DC=cha,DC=chartres,DC=sportludique,DC=fr
```

#### 1.2 Rôle

Les membres du groupe Admins du domaine disposent de privilèges complets sur :

    - Tous les objets Active Directory (utilisateurs, ordinateurs, groupes, OU)
    - Les stratégies de groupe (GPO)
    - Les serveurs et postes membres du domaine
    - Les droits d’administration locale sur chaque machine du domaine

C’est donc le plus haut niveau d’administration au sein d’un domaine AD.

### 2. Attribution des droits aux techniciens DSI

L’attribution s’est faite à l’aide de la commande PowerShell suivante, exécutée depuis une session administrateur sur un contrôleur de domaine :

```
Add-ADGroupMember -Identity "Admins du domaine" -Members "david.dsi","wassim.dsi","simon.dsi"
```

### 3. Vérification de l’appartenance au groupe

Après l’ajout, la commande suivante a été utilisée pour vérifier que les utilisateurs ont bien été intégrés :

```
Get-ADGroupMember "Admins du domaine" | Select Name, SamAccountName
```

📋 Résultat obtenu :

```
Name           SamAccountName
----           ---------------
Administrator  administrator
David          david.dsi
Wassim         wassim.dsi
Simon          simon.dsi
```

## 6. Script création des UO, Utilisateurs et Groupes du Pôle Chartre. 

🧾 **Contexte**

Nous disposons d’un fichier CSV nommé chartres.csv, contenant la liste des utilisateurs à créer dans Active Directory.<br>
Chaque ligne du fichier indique :

- Le nom et prénom de l’utilisateur,
- Le service auquel il appartient,
- Et éventuellement le sous-service.

L’objectif est d’automatiser la création de :

- Les Unités d’Organisation (OU) correspondant aux services,
- Les sous-OU correspondant aux sous-services,
- Les groupes de sécurité associés à chaque niveau (service et sous-service),
- Les utilisateurs, placés dans la bonne OU selon leur service/sous-service.

👥 **Groupes de sécurité**

Pour chaque service et sous-service, des groupes AD sont créés afin de faciliter la gestion des droits.<br>
Les groupes suivent la convention de nommage suivante :

| Niveau           | Exemple de nom de groupe                                              |
|------------------|-----------------------------------------------------------------------|
| **Service global** | `GRP_<Service>` (exemple : `GRP_SAV`)                                |
| **Sous-service**   | `GRP_<Service>_<SousService>` (exemples : `GRP_SAV_Responsable`, `GRP_Marketing_A`) |


👤 **Utilisateurs**

Chaque utilisateur est créé avec :

| Champ                                     | Description                                                                                             | Exemple             |
|-------------------------------------------|---------------------------------------------------------------------------------------------------------|---------------------|
| **Nom d’affichage**                       | Combinaison du nom et du prénom de l’utilisateur.                                                       | `Dupont Jean`       |
| **Identifiant de connexion (samAccountName)** | Format standardisé : `nomdefamille.prenom`.                                                         | `dupont.jean`       |
| **Mot de passe initial**                  | Mot de passe générique attribué à la création du compte.                                                | `Password`          |
| **Changement de mot de passe obligatoire**| L’utilisateur devra définir un nouveau mot de passe à sa première connexion.                            | *(Automatique)*     |
| **Gestion des doublons (collision)**      | Si un samAccountName existe déjà, un numéro est ajouté en suffixe pour garantir l’unicité.              | `dupont.jean2`      |


📁 Exemple de structure finale dans l’AD : 

```
OU=Users
├── OU=DIRECTION
│   └── GRP_Direction
├── OU=DSI
│   └── GRP_DSI
├── OU=RH
│   └── GRP_RH
├── OU=COMPTABILITE
│   └── GRP_Comptabilite
├── OU=SAV
│   ├── OU=Responsable
│   └── OU=Operateur
│   ├── GRP_SAV
│   ├── GRP_SAV_Responsable
│   └── GRP_SAV_Operateur
├── OU=Marketing
│   ├── OU=A
│   ├── OU=B
│   ├── OU=X
│   └── OU=Y
│   ├── GRP_Marketing
│   ├── GRP_Marketing_A
│   ├── GRP_Marketing_B
│   ├── GRP_Marketing_X
│   └── GRP_Marketing_Y
└── OU=Juridique
    └── GRP_Juridique
```

### Script Powershell afin de mettre en place les consignes ci-dessus.

```
# === Script création des UO, Utilisateurs et Groupes du Pôle Chartres ===

Import-Module ActiveDirectory

$csvPath = "C:\Users\Administrateur\Documents\chartres.csv"
$baseDN  = "DC=cha,DC=chartres,DC=sportludique,DC=fr"
$domain  = "cha.chartres.sportludique.fr"
$pw = ConvertTo-SecureString "Password" -AsPlainText -Force

$subOUs = @{
    "SAV"        = @("Responsable","Operateur")
    "Marketing"  = @("A","B","X","Y")
}

function Remove-Accents($text){
    $n = $text.Normalize([Text.NormalizationForm]::FormD)
    -join ($n.ToCharArray() | Where-Object{[Globalization.CharUnicodeInfo]::GetUnicodeCategory($_) -ne "NonSpacingMark"})
}

function Unique-Sam($last,$first){
    $b = ((Remove-Accents $last).ToLower() -replace "\s+") + "." + ((Remove-Accents $first).ToLower() -replace "\s+")
    if($b.Length -gt 20){$b=$b.Substring(0,20)}
    $s=$b;$i=1
    while(Get-ADUser -Filter "SamAccountName -eq '$s'" -ErrorAction SilentlyContinue){$s="$b$i";$i++}
    return $s
}

$users = Import-Csv $csvPath -Delimiter ';' -Encoding Default

foreach($u in $users){
    $service = $u.service.Trim().ToUpper()
    $sous = $u.'sous service'.Trim()
    $nom = $u.NOM; $prenom = $u.PRENOM
    if(!$nom -or !$prenom -or !$service){continue}

    $ouService = "OU=$service,$baseDN"
    if(-not (Get-ADOrganizationalUnit -Filter "DistinguishedName -eq '$ouService'" -ErrorAction SilentlyContinue)){
        New-ADOrganizationalUnit -Name $service -Path $baseDN -ProtectedFromAccidentalDeletion $false
    }

    if($subOUs.ContainsKey($service)){
        foreach($s in $subOUs[$service]){
            $ouSous = "OU=$s,OU=$service,$baseDN"
            if(-not (Get-ADOrganizationalUnit -Filter "DistinguishedName -eq '$ouSous'" -ErrorAction SilentlyContinue)){
                New-ADOrganizationalUnit -Name $s -Path "OU=$service,$baseDN" -ProtectedFromAccidentalDeletion $false
            }
        }
    }

    $grpService = "GRP_$service"
    if(-not (Get-ADGroup -Filter "Name -eq '$grpService'" -ErrorAction SilentlyContinue)){
        New-ADGroup -Name $grpService -GroupScope Global -Path $ouService
    }

    if($sous -and $subOUs[$service] -contains $sous){
        $grpSous = "GRP_${service}_$sous"
        if(-not (Get-ADGroup -Filter "Name -eq '$grpSous'" -ErrorAction SilentlyContinue)){
            New-ADGroup -Name $grpSous -GroupScope Global -Path "OU=$sous,OU=$service,$baseDN"
        }
        $targetOU = "OU=$sous,OU=$service,$baseDN"
    } else {
        $targetOU = $ouService
    }

    $sam = Unique-Sam $nom $prenom
    $upn = "$sam@$domain"
    $display = "$nom $prenom"

    if(-not (Get-ADUser -Filter "SamAccountName -eq '$sam'" -ErrorAction SilentlyContinue)){
        New-ADUser -Name $display -GivenName $prenom -Surname $nom -SamAccountName $sam -UserPrincipalName $upn -AccountPassword $pw -Enabled $true -ChangePasswordAtLogon $true -Path $targetOU
    }

    Add-ADGroupMember -Identity $grpService -Members $sam -ErrorAction SilentlyContinue
    if($grpSous){Add-ADGroupMember -Identity $grpSous -Members $sam -ErrorAction SilentlyContinue}
}
```