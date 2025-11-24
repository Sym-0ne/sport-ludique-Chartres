# Incident Utilisateurs AD : Expiration massive des mots de passe utilisateurs dans Active Directory après coupure de courant

---

## Contexte

* Environnement : Active Directory avec plusieurs Domain Controllers (DC), un principal et un secondaire (AD CS).
* Proxmox lié à l'AD pour authentification LDAP.
* Les utilisateurs du DSI (wassim.dsi, simon.dsi, david.dsi) étaient configurés avec des mots de passe actifs et ne devaient pas expirer.
* Une coupure de courant a provoqué l'arrêt brutal du DC Secondaire.

---

## Symptômes observés

* Les utilisateurs ont tous un mot de passe expirés donc la connexion aux sessions est impossible.
* Logs Proxmox indiquant :

```
authentication failure; user=<utilisateur>@ad msg=80090308: LdapErr: DSID-0C090549, comment: AcceptSecurityContext error, data 532
```

* `data 532` signifie : **Password expired**.

---

## Analyse technique

1. Chaque compte utilisateur AD possède les attributs :

   * `pwdLastSet` (date du dernier changement de mot de passe)
   * `userAccountControl` (flags, dont PASSWORD_EXPIRED)
2. La GPO du domaine indique :

```
MaxPasswordAge : 42 jours
MinPasswordAge : 1 jour
ComplexityEnabled : True
```

3. Après la coupure de courant :

- Les DC ont redémarré brutalement, certains journaux NTDS non flushés.
- L'heure système sur les DC a pu être désynchronisée.
- Les DC ont réévalué `pwdLastSet` pour tous les comptes.
- Comme l'heure ou les USN étaient incohérents, AD a marqué tous les mots de passe comme expirés.

4. Même sans expiration forcée dans la GPO pour certains comptes, AD peut appliquer `PASSWORD_EXPIRED` si :

- `pwdLastSet + MaxPasswordAge` > heure système (calcul erroné)
- ou si le DC considère que le compte n'a jamais eu son mot de passe défini correctement (corruption partielle NTDS)

---

## Conséquences

* Tous les utilisateurs apparaissent comme ayant un mot de passe expiré.
* Proxmox renvoie des erreurs LDAP 532.

---

## Solution appliquée

1. Réinitialiser les mots de passe expirés pour tous les utilisateurs du domaine (sauf DSI) :

```powershell
$ExcludeUsers = @("wassim.dsi","simon.dsi","david.dsi")
$TempPassword = "TempP@ssword123!"
Get-ADUser -Filter * -SearchBase "DC=cha,DC=chartres,DC=sportludique,DC=fr" | Where-Object {
    $ExcludeUsers -notcontains $_.SamAccountName
} | ForEach-Object {
    Set-ADAccountPassword -Identity $_.DistinguishedName -Reset -NewPassword (ConvertTo-SecureString $TempPassword -AsPlainText -Force)
    Set-ADUser -Identity $_.DistinguishedName -ChangePasswordAtLogon $true
}
```

---

2. Vérifier les utilisateurs avec mots de passe expirés :

```powershell
Search-ADAccount -PasswordExpired | Select-Object Name, SamAccountName, DistinguishedName
```

---

3. Pour éviter que le problème se reproduise :

   * Désactiver l'expiration globale si souhaité : `MaximumPasswordAge = 0` dans la GPO `Default Domain Policy`.
   * Ou mettre des comptes spécifiques comme "Password never expires".

```
PS C:\WINDOWS\system32> Get-ADUser -Filter * | Where-Object {$Exclude -notcontains $_.SamAccountName} | Set-ADUser -PasswordNeverExpires $true
```

---

## Recommandations

* Surveiller la synchronisation horaire des DC (`w32tm /query /status`).
* Vérifier régulièrement l'état des journaux NTDS après redémarrage brutal.
* Maintenir des scripts pour réinitialiser les mots de passe expirés rapidement après un incident.
* Pour les comptes critiques (Proxmox, service admin), utiliser "Password never expires" pour éviter les blocages inattendus.

---

## Références Microsoft

* [Search-ADAccount](https://learn.microsoft.com/en-us/powershell/module/activedirectory/search-adaccount)
* [Get-ADUser](https://learn.microsoft.com/en-us/powershell/module/activedirectory/get-aduser)
* [GPO Password Policy](https://learn.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/password-policy)
* [Troubleshoot password expiration](https://learn.microsoft.com/en-us/troubleshoot/windows-server/identity/password-expiration-issues)
