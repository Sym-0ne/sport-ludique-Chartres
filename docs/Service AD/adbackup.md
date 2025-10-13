## Installation de L'AD

Suivre cette documentation pour configuer l'AD Backup en adaptant pour lui : [Installe AD](https://sym-0ne.github.io/sport-ludique-Chartres/Service%20AD/ad/)

## Redondance AD1 et AD2 

- AD1 : 172.28.33.2<br>nom : CHA-DC-01.cha.chartres.sportludique.fr
- AD2 : 172.28.33.3<br>nom : CHA_DC_02.cha.chartres.sportludique.fr
- Domaine : cha.chartres.sportludique.fr

Les deux hébergent :

- Active Directory Domain Services (AD DS)
- DNS intégré à Active Directory

Ainsi, si l'AD1 tombe, les utilisateurs et ordinateurs peuvent encore :

- s'authentifer sur le domaine
- résoudre les noms via DNS

### Mettre l'AD2 en contrôleur de domaine secondaire

1. Installer les rôles nécessaires

Dessus en PowerShell :
```
Install-WindowsFeature AD-Domain-Services, DNS -IncludeManagementTools
```

2. Promouvoir en contrôleur de domaine

```
Install-ADDSDomainController `
    -DomainName "cha.chartres.sportludique.fr" `
    -InstallDns:$true `
    -Credential (Get-Credential) `
    -SiteName "Default-First-Site-Name" `
    -ReplicationSourceDC "CHA_DC_01.cha.chartres.sportludique.fr"
```

💡 Une fenêtre d’authentification s’ouvre → entre un compte du domaine ayant les droits d’administrateur et redémarrer à la fin.

### Vérification de la réplication AD

Sur Powershell :
```
repadmin /replsummary
```

Si les deux AD s'affiche et qu'il n'y a aucune erreur alors la réplication fonctionne.

### Configuration DNS croisée

Sur **AD1**, configure :
```
DNS préféré : 172.28.33.2
DNS secondaire : 172.28.33.3
```
Sur **AD2** :
```
DNS préféré : 172.28.33.3
DNS secondaire : 172.28.33.2
```

### Vérification de la réplication DNS

Sur AD2, ouvre :

- Gestionnaire DNS → Zones de recherche directe → cha.chartres.sportludique.fr

Tu devrais y voir **exactement les mêmes enregistrements que sur AD1**.<br>
Si oui, la réplication DNS fonctionne (grâce à AD).

## Vérification finale 

1. Éteins AD1.

2. Sur le client, vérifie la résolution DNS :
```
nslookup cha.chartres.sportludique.fr
```
Si il y a une réponse sa veut dire que le **DNS est bien redondant**.

3. Connecte-toi à ton domaine :

Si l'authentification fonctionne alors **l'AD est redondant aussi**.

Si ces deux test fonctionne alors notre **AD2 et bien en backup de l'AD1**.