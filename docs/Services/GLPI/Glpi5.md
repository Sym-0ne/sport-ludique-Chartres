# Déploiement de l’agent GLPI sur toutes les VM Windows via GPO


## Objectif : 
-------------
Centraliser l’inventaire matériel et logiciel des machines Windows du domaine via l’agent GLPI.
L’objectif est de déployer l’agent automatiquement sur toutes les machines de l’infrastructure grâce à une GPO et la configuration du registre.

---

## 1. Configuration des serveurs et préparation

### 1.1 Définir les serveurs concernés et le chemin du déploiement :

```
$servers = @(
    "CHA-DC-01",    # AD Principal
    "CHA_DC_02",    # AD Secondaire
    "CHA-HMAIL"     # Serveur Mail
)

$deployPath = "C:\Deploy\GLPI-Agent"
$batDest = "$deployPath\glpi-agent.bat"
$shareName = "Deploy"
$gpoName = "Déploiement Agent GLPI"
```

---

### 1.2 Vérifier le fichier `glpi-agent.bat` :

```
if (-not (Test-Path $batDest)) {
    Write-Host "❌ Le fichier glpi-agent.bat n'existe pas dans $deployPath !"
    exit
}
```

---

### 1.3 Créer le partage réseau pour le déploiement :
```
if (Get-SmbShare -Name $shareName -ErrorAction SilentlyContinue) {
    Remove-SmbShare -Name $shareName -Force
}

New-SmbShare -Name $shareName -Path "C:\Deploy" -ReadAccess "Authenticated Users" -FullAccess "Administrators"

Chemin du partage : \\CHA-DC-01\Deploy
```

---

## 2. Création de la GPO pour déploiement

### 2.1 Créer ou récupérer la GPO :
```
$gpo = Get-GPO -Name $gpoName -ErrorAction SilentlyConti(optionnel)nue
if (!$gpo) {
    $gpo = New-GPO -Name $gpoName
}
```

### 2.2 Lier la GPO à **toutes les machines de l’infrastructure** :

**Exemple : lier à la racine du domaine pour toucher toutes les machines**

```
New-GPLink -Name $gpoName -Target "DC=mondomaine,DC=local"
```

---

## 3. Déploiement de l’agent GLPI via la GPO

### 3.1 Dans la console GPMC (sur CHA-DC-01) :

- Win + R → `gpmc.msc` → Entrée  
OU Menu Démarrer → Outils d’administration → Gestion des stratégies de groupe

---

### 3.2 Localiser la GPO “Déploiement Agent GLPI” et modifier :
```
Configuration ordinateur
    → Stratégies
        → Paramètres logiciels
            → Installation de logiciels
```

---

### 4. Configuration du registre via GPO 

Pour que toutes les machines pointent vers le serveur GLPI correct :

| Élément            | Valeur                                         |
|--------------------|------------------------------------------------|
| Chemin du registre | HKEY_LOCAL_MACHINE\SOFTWARE\GLPI-Agent        |
| Valeur             | server                                         |
| Type               | REG_SZ                                         |
| Donnée             | http://10.10.120.15/front/inventory.php       |
| Action             | Mettre à jour                                  |


Cela permet de configurer automatiquement l’adresse du serveur GLPI pour toutes les machines.

---

### 5. Installation manuelle (DC principal)

- Installer manuellement l’agent sur CHA-DC-01 :  
  `"C:\Program Files\GLPI-Agent\glpi-agent.bat" --server http://10.10.120.15/front/inventory.php --force --debug --logger=stderr`

- Vérifier que l’agent remonte dans GLPI.

---

