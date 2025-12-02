# Installation & Configuration du Serveur Mail

## Contexte

Comme dans toute entreprise, la communication est vitale, qu'elle soit interne au sein du site de Chartres ou externe entre les différents sites. Dans ce but, nous avons configuré un serveur de messagerie afin d'envoyer des mails qui utilise les protocoles SMTP et IMAP.

---

## 1. Installation 

Nous avons utilisé le service "Hmail" sur Windows, nous l'utiliserons sur une VM Windows 10 hébergée sur notre hyperviseur Proxmox.

### 1.1 Hmail

La configuration d'installation de HMail est relativement simple : il suffit de lancer le programme d'installation du service disponible [ICI](https://www.hmailserver.com/download), de créer un mot de passe pour le service, éventuellement de modifier les dossiers de l'application, puis de terminer l'installation.

---

## 2. Configuration

### 2.1 Domaine et Utilisateurs
La première étape consiste à définir le nom de domaine, dans notre cas : `chartres.sportludique.fr`.

Concernant les utilisateurs, HMail ne dispose pas d’un module de synchronisation LDAP. Deux options sont donc possibles :
- Créer chaque utilisateur manuellement.
- Créer automatiquement les utilisateurs via un script PowerShell utilisant LDAP.

Nous avons naturellement choisi d’automatiser la création via un script PowerShell.  
Pour cela, il faut créer dans l’AD un compte disposant des droits de lecture sur les objets “utilisateurs” du domaine.

Voici le script PowerShell utilisé :

```
# Script de synchronisation Active Directory vers hMailServer
# Support AD distant avec authentification

# ========== CONFIGURATION ==========
# Configuration hMailServer
$hMailAdminPassword = "ADMMAIL873!"
$defaultDomain = "chartres.sportludique.fr"
$defaultPassword = "azerty" # Mot de passe par défaut pour nouveaux comptes

# Configuration Active Directory DISTANT
$adServer = "10.10.120.2"  # Nom ou IP du serveur AD
$adDomain = "cha.chatres.sportludique.fr"  # Domaine AD
$adUsername = "ldap@cha.chartres.sportludique.fr"  # Utilisateur avec droits de lecture AD
$adPassword = "adm123!"  # Mot de passe pour l'AD

# Construction du chemin LDAP
$ldapPath = "LDAP://$adServer/DC=cha,DC=chartres,DC=sportludique,DC=fr"

# Options
$createNewAccounts = $true
$updateExistingAccounts = $true
$accountMaxSize = 1024  # Taille max en MB
$enableAccountsByDefault = $true

Write-Host "========================================" -ForegroundColor Cyan
Write-Host "Synchronisation AD -> hMailServer" -ForegroundColor Cyan
Write-Host "========================================" -ForegroundColor Cyan

# ========== TEST CONNECTIVITÉ AD ==========
Write-Host "`n[Préparation] Test de connectivité AD..." -ForegroundColor Yellow

try {
    Write-Host "  → Test ping vers $adServer..." -ForegroundColor Gray
    $pingResult = Test-Connection -ComputerName $adServer -Count 1 -Quiet -ErrorAction SilentlyContinue
    
    if ($pingResult) {
        Write-Host "  ✓ Serveur AD accessible" -ForegroundColor Green
    } else {
        Write-Host "  ⚠ Serveur AD non pingable (peut être normal si ICMP bloqué)" -ForegroundColor Yellow
    }
    
    Write-Host "  → Test port LDAP 389..." -ForegroundColor Gray
    $tcpClient = New-Object System.Net.Sockets.TcpClient
    $asyncResult = $tcpClient.BeginConnect($adServer, 389, $null, $null)
    $wait = $asyncResult.AsyncWaitHandle.WaitOne(3000, $false)
    
    if ($wait) {
        $tcpClient.EndConnect($asyncResult)
        $tcpClient.Close()
        Write-Host "  ✓ Port LDAP 389 accessible" -ForegroundColor Green
    } else {
        Write-Host "  ✗ Port LDAP 389 non accessible" -ForegroundColor Red
        Write-Host "    Vérifiez le pare-feu et la connectivité réseau" -ForegroundColor Yellow
    }
}
catch {
    Write-Host "  ⚠ Erreur de test réseau: $_" -ForegroundColor Yellow
}

# ========== CONNEXION HMAILSERVER ==========
Write-Host "`n[1/4] Connexion à hMailServer..." -ForegroundColor Yellow

$hMailApp = $null
$domain = $null

try {
    Write-Host "  → Création de l'objet COM hMailServer.Application..." -ForegroundColor Gray
    $hMailApp = New-Object -ComObject hMailServer.Application
    
    if ($hMailApp -eq $null) {
        throw "L'objet hMailServer.Application est null"
    }
    
    Write-Host "  → Authentification..." -ForegroundColor Gray
    $authResult = $hMailApp.Authenticate("Administrator", $hMailAdminPassword)
    
    if (-not $authResult) {
        throw "Échec de l'authentification hMailServer. Vérifiez le mot de passe."
    }
    
    Write-Host "  → Récupération du domaine $defaultDomain..." -ForegroundColor Gray
    $domain = $hMailApp.Domains.ItemByName($defaultDomain)
    
    if ($domain -eq $null) {
        Write-Host "`n✗ Domaine '$defaultDomain' non trouvé!" -ForegroundColor Red
        Write-Host "`nDomaines disponibles:" -ForegroundColor Yellow
        for ($i = 0; $i -lt $hMailApp.Domains.Count; $i++) {
            $d = $hMailApp.Domains.Item($i)
            Write-Host "  - $($d.Name)" -ForegroundColor Cyan
        }
        exit
    }
    
    Write-Host "✓ Connecté à hMailServer - Domaine: $defaultDomain" -ForegroundColor Green
}
catch {
    Write-Host "`n✗ ERREUR de connexion hMailServer:" -ForegroundColor Red
    Write-Host $_.Exception.Message -ForegroundColor Red
    Write-Host "`nVérifications:" -ForegroundColor Yellow
    Write-Host "1. Service hMailServer démarré ?" -ForegroundColor White
    Write-Host "2. PowerShell exécuté en Administrateur ?" -ForegroundColor White
    Write-Host "3. Mot de passe administrateur correct ?" -ForegroundColor White
}

# ========== RÉCUPÉRATION DES COMPTES HMAILSERVER ==========
Write-Host "`n[2/4] Récupération des comptes hMailServer existants..." -ForegroundColor Yellow

$hmailAccounts = @{}
try {
    for ($i = 0; $i -lt $domain.Accounts.Count; $i++) {
        $account = $domain.Accounts.Item($i)
        $hmailAccounts[$account.Address.ToLower()] = $account
    }
    Write-Host "✓ $($hmailAccounts.Count) comptes trouvés" -ForegroundColor Green
}
catch {
    Write-Host "✗ Erreur: $_" -ForegroundColor Red
}

# ========== CONNEXION À L'AD DISTANT ==========
Write-Host "`n[3/4] Connexion à l'Active Directory distant..." -ForegroundColor Yellow
Write-Host "  Serveur: $adServer" -ForegroundColor Gray
Write-Host "  Chemin LDAP: $ldapPath" -ForegroundColor Gray

try {
    # Créer des credentials pour l'AD distant
    $secPassword = ConvertTo-SecureString $adPassword -AsPlainText -Force
    $credential = New-Object System.Management.Automation.PSCredential($adUsername, $secPassword)
    
    # Créer la connexion LDAP avec authentification
    Write-Host "  → Authentification sur l'AD..." -ForegroundColor Gray
    
    $directoryEntry = New-Object System.DirectoryServices.DirectoryEntry(
        $ldapPath,
        $adUsername,
        $adPassword,
        [System.DirectoryServices.AuthenticationTypes]::Secure
    )
    
    # Test de la connexion
    $testBind = $directoryEntry.Name
    if ([string]::IsNullOrEmpty($testBind) -and $directoryEntry.NativeObject -eq $null) {
        throw "Impossible de se connecter à l'AD. Vérifiez les credentials et le chemin LDAP."
    }
    
    Write-Host "  ✓ Authentifié sur l'AD" -ForegroundColor Green
    
    # Créer le searcher
    Write-Host "  → Recherche des utilisateurs..." -ForegroundColor Gray
    $searcher = New-Object System.DirectoryServices.DirectorySearcher
    $searcher.SearchRoot = $directoryEntry
    
    # Filtre pour les utilisateurs avec email
    $searcher.Filter = "(&(objectClass=user)(objectCategory=person)(mail=*))"
    
    # Propriétés à récupérer
    $searcher.PropertiesToLoad.AddRange(@(
        "sAMAccountName",
        "mail",
        "givenName",
        "sn",
        "displayName",
        "userAccountControl"
    ))
    
    $searcher.PageSize = 1000
    $searcher.SearchScope = [System.DirectoryServices.SearchScope]::Subtree
    
    $results = $searcher.FindAll()
    
    Write-Host "✓ $($results.Count) utilisateurs trouvés dans AD" -ForegroundColor Green
}
catch {
    Write-Host "`n✗ ERREUR de connexion AD:" -ForegroundColor Red
    Write-Host $_.Exception.Message -ForegroundColor Red
    Write-Host "`nVérifications:" -ForegroundColor Yellow
    Write-Host "1. Le serveur AD '$adServer' est-il accessible ?" -ForegroundColor White
    Write-Host "2. Les credentials AD sont-ils corrects ?" -ForegroundColor White
    Write-Host "   Username: $adUsername" -ForegroundColor Gray
    Write-Host "3. Le chemin LDAP est-il correct ?" -ForegroundColor White
    Write-Host "   $ldapPath" -ForegroundColor Gray
    Write-Host "4. Le port 389 (LDAP) est-il ouvert ?" -ForegroundColor White
    Write-Host "5. Essayez avec LDAPS (port 636) si disponible" -ForegroundColor White
}

# ========== SYNCHRONISATION ==========
Write-Host "`n[4/4] Synchronisation en cours..." -ForegroundColor Yellow
Write-Host "────────────────────────────────────────" -ForegroundColor Gray

$stats = @{
    Created = 0
    Updated = 0
    Skipped = 0
    Errors = 0
}

foreach ($result in $results) {
    $props = $result.Properties
    
    # Récupérer les informations
    $samAccount = if ($props["samaccountname"].Count -gt 0) { $props["samaccountname"][0] } else { "Inconnu" }
    $email = if ($props["mail"].Count -gt 0) { $props["mail"][0] } else { $null }
    $firstName = if ($props["givenname"].Count -gt 0) { $props["givenname"][0] } else { "" }
    $lastName = if ($props["sn"].Count -gt 0) { $props["sn"][0] } else { "" }
    $displayName = if ($props["displayname"].Count -gt 0) { $props["displayname"][0] } else { $samAccount }
    
    # Vérifier si le compte est actif dans AD
    $userAccountControl = if ($props["useraccountcontrol"].Count -gt 0) { $props["useraccountcontrol"][0] } else { 0 }
    $isEnabled = -not ($userAccountControl -band 0x2)  # 0x2 = ACCOUNTDISABLE
    
    # Vérifier l'adresse email
    if ([string]::IsNullOrWhiteSpace($email)) {
        Write-Host "⊘ $displayName - Pas d'email" -ForegroundColor DarkGray
        $stats.Skipped++
        continue
    }
    
    # Forcer le domaine mail si l'email AD ne correspond pas
    if (-not $email.EndsWith("@$defaultDomain")) {
        $emailParts = $email.Split('@')
        $email = "$($emailParts[0])@$defaultDomain"
        Write-Host "ℹ $displayName - Email modifié: $email" -ForegroundColor Cyan
    }
    
    $emailLower = $email.ToLower()
    
    try {
        # Vérifier si le compte existe dans hMailServer
        if ($hmailAccounts.ContainsKey($emailLower)) {
            # MISE À JOUR
            if ($updateExistingAccounts) {
                $account = $hmailAccounts[$emailLower]
                
                $account.PersonF---irstName = $firstName
                $account.PersonLastName = $lastName
                $account.Active = $isEnabled
                $account.Save()
                
                Write-Host "↻ $email - Mis à jour" -ForegroundColor Cyan
                $stats.Updated++
            }
            else {
                Write-Host "→ $email - Existe déjà" -ForegroundColor Gray
                $stats.Skipped++
            }
        }
        else {
            # CRÉATION
            if ($createNewAccounts) {
                $newAccount = $domain.Accounts.Add()
                $newAccount.Address = $email
                $newAccount.Password = $defaultPassword
                $newAccount.Active = if ($enableAccountsByDefault) { $isEnabled } else { $false }
                $newAccount.PersonFirstName = $firstName
                $newAccount.PersonLastName = $lastName
                $newAccount.MaxSize = $accountMaxSize
                $newAccount.Save()
                
                Write-Host "✓ $email - Créé" -ForegroundColor Green
                $stats.Created++
            }
            else {
                Write-Host "⊘ $email - Nouveau (création désactivée)" -ForegroundColor DarkGray
                $stats.Skipped++
            }
        }
    }
    catch {
        Write-Host "✗ $email - ERREUR: $_" -ForegroundColor Red
        $stats.Errors++
    }
}

# ========== RÉSULTATS ==========
Write-Host "`n========================================" -ForegroundColor Cyan
Write-Host "RÉSUMÉ DE LA SYNCHRONISATION" -ForegroundColor Cyan
Write-Host "========================================" -ForegroundColor Cyan
Write-Host "Comptes créés    : $($stats.Created)" -ForegroundColor Green
Write-Host "Comptes mis à jour: $($stats.Updated)" -ForegroundColor Cyan
Write-Host "Comptes ignorés  : $($stats.Skipped)" -ForegroundColor Gray
Write-Host "Erreurs          : $($stats.Errors)" -ForegroundColor Red
Write-Host "────────────────────────────────────────" -ForegroundColor Gray
Write-Host "Total hMailServer: $($domain.Accounts.Count) comptes" -ForegroundColor White
Write-Host "========================================`n" -ForegroundColor Cyan

# ========== NETTOYAGE ==========
if ($results) { $results.Dispose() }
if ($searcher) { $searcher.Dispose() }
if ($directoryEntry) { $directoryEntry.Dispose() }
if ($domain -ne $null) {
    [System.Runtime.Interopservices.Marshal]::ReleaseComObject($domain) | Out-Null
}
if ($hMailApp -ne $null) {
    [System.Runtime.Interopservices.Marshal]::ReleaseComObject($hMailApp) | Out-Null
}
[System.GC]::Collect()
[System.GC]::WaitForPendingFinalizers()

Write-Host "Synchronisation terminée!" -ForegroundColor Green
```

<div class="annotate" markdown>

Ce script se connecte à l’AD en LDAP en utilisant le compte `ldap`.  
Il lit tous les utilisateurs ainsi que leur adresse mail, puis crée les comptes correspondants sur le serveur HMail.  
Le script supprime également les comptes qui ne sont plus présents dans l’AD.  
Il peut être intégré au Planificateur de tâches Windows pour une exécution automatique à intervalles réguliers, afin de maintenir la liste des utilisateurs à jour (1).

</div>
1. À noter que ce script attribue le mot de passe "azerty" à tous les comptes mail. Ce mot de passe peut évidemment être synchronisé avec les mots de passe de l’Active Directory.

---

### 2.2 Paramétrage réseau

Afin que le serveur SMTP et IMAP puissent joindre et répondre aux connexions des autres réseaux ainsi que des connexions Internet, il faut mettre en place plusieurs choses :
- Routes de retour sur le serveur HMail
- PAT sur les routeurs R1 et R2
- Enregistrements DNS imap et smtp
- Ajout des réseaux dans la configuration HMail

---

#### 2.2.1 Routes de retour sur le serveur Hmail 

Toujours à cause du [stateful inspection](https://sym-0ne.github.io/sport-ludique-Chartres/Pare-feux/stormshield/#7-statefull-inspection), nous devons ajouter des routes de retour sur tout équipement ayant besoin de joindre le LAN.


```
===========================================================================
Itinéraires persistants :
  Adresse réseau    Masque réseau  Adresse passerelle Métrique
      172.28.35.0    255.255.255.0    172.28.62.253       1
    172.28.63.128  255.255.255.128    172.28.62.253       1
      172.28.33.0    255.255.255.0    172.28.62.253       1
          0.0.0.0          0.0.0.0    172.28.62.254  Par défaut
===========================================================================
```

---

#### 2.2.2 PAT sur R1 & R2 

```
ip nat inside source static tcp 172.28.62.2 25 183.44.28.1 25 #Règle SMTP
ip nat inside source static tcp 172.28.62.2 143 183.44.28.1 143 #Règle IMAP
```

---

#### 2.2.3 Enregistrements DNS
<div class="annotate" markdown>
Sur la zone **externe** de notre DNS autorité (1)
</div>
1. À savoir que les enregistrements sont les mêmes sur le DNS d’autorité secondaire, car nous n’avons pas de serveur mail secondaire.

```
smtp IN A 183.44.28.1  #Redirection requêtes SMTP
imap IN A 183.44.28.1 #Redirection requêtes IMAP
chartres.sportludique.fr. IN MX 10 smtp.chartres.sportludique.fr. #Redirection des requêtes SMTP depuis l'extérieur
```

Sur la zone **interne** de notre DNS d’autorité (les zones internes sont synchronisées entre le primaire et le secondaire).

```
#Services
helpdesk IN A 172.28.62.5
#Mail
smtp IN A 172.28.62.2
imap IN A 172.28.62.2
chartres.sportludique.fr. IN MX 10 smtp.chartres.sportludique.fr.
```

---

#### 2.2.4 Ajout des réseaux dans Hmail 

Il faut ajouter les réseaux autorisés à communiquer avec le serveur directement dans la configuration de celui-ci :

`Settings > Advanced > IP Ranges > Add`

Il suffit ensuite d’ajouter le réseau client : 172.28.35.0 → 172.28.35.254 et d’augmenter la priorité pour que cette étendue passe au-dessus des autres. Pensez bien à désactiver le SSL/TLS s’il n’est pas utilisé.

---

## 3. Utilisation 

Côté utilisateur, il suffit de se connecter depuis un client de messagerie, de s’authentifier avec ses identifiants du domaine et d’envoyer des mails !

---