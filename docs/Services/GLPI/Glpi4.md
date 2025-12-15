# Deploiement de l'agent GLPI sur toutes les VM Linux via Ansible

############################################################
# CONFIGURATION – NOMS EXACTS DES SERVEURS
############################################################

$servers = @(
    "CHA-DC-01",    # AD Principal
    "CHA_DC_02",    # AD Secondaire
    "CHA-HMAIL"     # Serveur Mail
)

# Nom de l'OU
$ouName = "SERVEURS"

# Domaine actuel
$domain = (Get-ADDomain).DistinguishedName

# Dossier de déploiement et fichier MSI
$deployPath = "C:\Deploy\GLPI-Agent"
$msiDest = "$deployPath\glpi-agent.msi"

# Nom du partage réseau
$shareName = "Deploy"

# Nom de la GPO
$gpoName = "Déploiement Agent GLPI"


############################################################
# 1️⃣ CRÉATION DE L’OU SERVEURS (si non existante)
############################################################

Write-Host "`n[1] Vérification / création OU 'SERVEURS'..." -ForegroundColor Cyan

if (-not (Get-ADOrganizationalUnit -Filter "Name -eq '$ouName'" -ErrorAction SilentlyContinue)) {
    New-ADOrganizationalUnit -Name $ouName -Path $domain
    Write-Host "   → OU SERVEURS créée." -ForegroundColor Green
} else {
    Write-Host "   → OU SERVEURS existe déjà." -ForegroundColor Yellow
}


############################################################
# 2️⃣ DÉPLACEMENT DES SERVEURS DANS L’OU
############################################################

Write-Host "`n[2] Déplacement des serveurs dans l’OU SERVEURS..." -ForegroundColor Cyan

foreach ($srv in $servers) {
    $comp = Get-ADComputer -Filter "Name -eq '$srv'" -ErrorAction SilentlyContinue
    if ($comp) {
        Move-ADObject -Identity $comp.DistinguishedName -TargetPath "OU=$ouName,$domain"
        Write-Host "   → $srv déplacé." -ForegroundColor Green
    } else {
        Write-Host "   ⚠ Serveur introuvable dans l’AD : $srv" -ForegroundColor Yellow
    }
}


############################################################
# 3️⃣ VÉRIFICATION DU FICHIER MSI
############################################################

Write-Host "`n[3] Vérification du fichier glpi-agent.msi..." -ForegroundColor Cyan

if (Test-Path $msiDest) {
    Write-Host "   → Fichier trouvé : $msiDest" -ForegroundColor Green
} else {
    Write-Host "❌ Le fichier glpi-agent.msi n'existe pas dans $deployPath !" -ForegroundColor Red
    Write-Host "   Mets-le dans ce dossier et relance le script."
    Read-Host "Appuyez sur Entrée pour quitter..."
    exit
}


############################################################
# 4️⃣ CRÉATION DU PARTAGE RÉSEAU
############################################################

Write-Host "`n[4] Création du partage réseau..." -ForegroundColor Cyan

if (Get-SmbShare -Name $shareName -ErrorAction SilentlyContinue) {
    Remove-SmbShare -Name $shareName -Force
}

New-SmbShare -Name $shareName -Path "C:\Deploy" -ReadAccess "Authenticated Users" -FullAccess "Administrators"
Write-Host "   → Partage créé : \\$env:COMPUTERNAME\$shareName" -ForegroundColor Green


############################################################
# 5️⃣ CRÉATION DE LA GPO
############################################################

Write-Host "`n[5] Création de la GPO 'Déploiement Agent GLPI'..." -ForegroundColor Cyan

$gpo = Get-GPO -Name $gpoName -ErrorAction SilentlyContinue
if (!$gpo) {
    $gpo = New-GPO -Name $gpoName
    Write-Host "   → GPO créée." -ForegroundColor Green
} else {
    Write-Host "   → GPO déjà existante." -ForegroundColor Yellow
}


############################################################
# 6️⃣ LIAISON DE LA GPO À L’OU SERVEURS
############################################################

Write-Host "`n[6] Liaison de la GPO à l’OU SERVEURS..." -ForegroundColor Cyan

New-GPLink -Name $gpoName -Target "OU=$ouName,$domain"

Write-Host "   → GPO liée à l’OU SERVEURS." -ForegroundColor Green


############################################################
# 7️⃣ INFORMATIONS À L’UTILISATEUR
############################################################

Write-Host "`n🎉 SCRIPT TERMINÉ !" -ForegroundColor Green
Write-Host "Les serveurs CHA-DC-01, CHA_DC_02 et CHA-HMAIL sont maintenant dans l'OU SERVEURS." -ForegroundColor Green
Write-Host "Partage réseau disponible : \\$env:COMPUTERNAME\$shareName" -ForegroundColor Green
Write-Host "N'oubliez pas d'ajouter le package MSI glpi-agent.msi dans la GPO via la console GPMC." -ForegroundColor Yellow


############################################################
# 8️⃣ ATTENDRE QUE L’UTILISATEUR APPUIE SUR ENTRÉE
############################################################

Read-Host "Appuyez sur Entrée pour fermer cette fenêtre..."





gpupdate /force

1️⃣ Ouvrir la console de gestion des stratégies de groupe (GPMC)

Sur ton serveur AD principal (CHA-DC-01) :

Win + R → tape gpmc.msc → Entrée

Ou Menu Démarrer → Outils d’administration → Gestion des stratégies de groupe

2️⃣ Localiser ta GPO

Dans le panneau de gauche :

Forêt : ton-domaine
    Domaines
        ton-domaine
            Objets de stratégie de groupe


Clique sur la GPO “Déploiement Agent GLPI”.

3️⃣ Ajouter le package MSI

Clique droit sur la GPO → Modifier

Dans l’éditeur de stratégie de groupe, va à :

Configuration ordinateur
    Stratégies
        Paramètres logiciels
            Installation de logiciels


Clique droit sur Installation de logiciels → Nouveau → Package

Dans la fenêtre qui s’ouvre :

Chemin UNC du MSI :

\\CHA-DC-01\Deploy\GLPI-Agent\glpi-agent.msi


⚠️ Il faut utiliser le chemin UNC, pas un chemin local (C:\...)

Choisis Attribué (Assigned) → OK

4️⃣ Vérifier

Le package doit maintenant apparaître sous Installation de logiciels.

Assure-toi que “Attribué” est bien sélectionné et non “Publié”.

5️⃣ Résultat

Les serveurs de l’OU SERVEURS (CHA-DC-01, CHA_DC_02, CHA-HMAIL) recevront automatiquement l’agent GLPI au prochain redémarrage.