# Régle des Pare-Feu

## Stormshield n°1

| Catégorie              | Interface | N° | Source         | Port Src | Destination | Port Dest | Protocole | Action |
| ---------------------- | --------- | -- | -------------- | -------- | ----------- | --------- | --------- | ------ |
| DNS (entrée)           | WAN       | 1  | *              | *        | 172.28.62.1 | 53        | TCP/UDP   | Permit |
| Reverse Proxy HTTP     | WAN       | 2  | *              | *        | 172.28.62.5 | 80        | TCP       | Permit |
| Reverse Proxy HTTPS    | WAN       | 3  | *              | *        | 172.28.62.5 | 443       | TCP       | Permit |
| Mail SMTP (entrée)     | WAN       | 4  | *              | *        | 172.28.62.2 | 25        | TCP       | Permit |
| Mail IMAP (entrée)     | WAN       | 5  | *              | *        | 172.28.62.2 | 143       | TCP       | Permit |
| Mail SMTP (sortie)     | DMZ → WAN | 6  | 172.28.62.2    | *        | *           | 25        | TCP       | Permit |
| DNS sortant (DMZ)      | DMZ → WAN | 7  | 172.28.62.0/24 | *        | *           | 53        | TCP/UDP   | Permit |
| Accès Internet LAN     | LAN       | 8  | 172.28.32.0/19 | *        | *           | *         | ANY       | Permit |
| Bloc tout non autorisé | WAN       | 9  | *              | *        | *           | *         | ANY       | Deny   |


## Stormshield n°2

| Catégorie                      | Interface | N° | Source         | Port Src | Destination  | Port Dest | Protocole | Action |
| ------------------------------ | --------- | -- | -------------- | -------- | ------------ | --------- | --------- | ------ |
| DNS (entrée secondaire)        | WAN       | 1  | *              | *        | 172.28.62.11 | 53        | TCP/UDP   | Permit |
| Reverse Proxy HTTP secondaire  | WAN       | 2  | *              | *        | 172.28.62.10 | 80        | TCP       | Permit |
| Reverse Proxy HTTPS secondaire | WAN       | 3  | *              | *        | 172.28.62.10 | 443       | TCP       | Permit |
| DNS sortant                    | DMZ → WAN | 4  | 172.28.62.0/24 | *        | *            | 53        | ANY       | Permit |
| Accès Internet LAN             | LAN       | 5  | 172.28.32.0/19 | *        | *            | *         | ANY       | Permit |
| Bloc tout non autorisé         | WAN       | 6  | *              | *        | *            | *         | ANY       | Deny   |

## OPNsense

| Catégorie                        | Interface | N° | Source         | Port Src | Destination    | Port Dest | Protocole | Action | Commentaire                |
| -------------------------------- | --------- | -- | -------------- | -------- | -------------- | --------- | --------- | ------ | -------------------------- |
| DNS → DNS interne 1              | WAN       | 1  | 172.28.62.0/24 | *        | 172.28.33.4    | 53        | TCP/UDP   | Permit | —                          |
| DNS → DNS interne 2              | WAN       | 2  | 172.28.62.0/24 | *        | 172.28.33.5    | 53        | TCP/UDP   | Permit | —                          |
| Web DMZ → SQL                    | WAN       | 3  | 172.28.62.3    | *        | 192.168.28.10  | 3306      | TCP       | Permit | —                          |
| REVERSE PROXY 1 → DOCKER (GLPI)  | WAN       | 4  | 172.28.62.5    | *        | 172.28.33.8    | 2000      | TCP       | Permit | ⚠️ Spécifier la passerelle ```
<div class="annotate" markdown>

:bulb:(1)

</div>
1. ghrf

---|
| REVERSE PROXY 2 → DOCKER (GLPI)  | WAN       | 5  | 172.28.62.10   | *        | 172.28.33.8    | 2000      | TCP       | Permit | ⚠️ Spécifier la passerelle |
| SMTP DMZpub → LAN                | WAN       | 6  | 172.28.62.2    | *        | 172.28.35.0/24 | 25        | TCP       | Permit | —                          |
| IMAP DMZpub → LAN                | WAN       | 7  | 172.28.62.2    | *        | 172.28.35.0/24 | 143       | TCP       | Permit | —                          |
| Bloc tout non autorisé WAN       | WAN       | 8  | *              | *        | *              | *         | ANY       | Deny   | Règle de blocage           |
| Internet LAN                     | LAN       | 1  | 172.28.32.0/19 | *        | *              | *         | ANY       | Permit | —                          |
| Bloc tout non autorisé LAN       | LAN       | 2  | *              | *        | *              | *         | ANY       | Deny   | —                          |
| Bloc tout non autorisé DMZ(priv) | DMZpriv   | 1  | *              | *        | *              | *         | ANY       | Deny   | Règle par défaut           |

