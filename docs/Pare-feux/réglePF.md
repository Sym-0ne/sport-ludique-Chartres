# Régle des Pare-Feu.

---

## Stormshield n°1

### Règles de filtrage principales

| ID | État | Action  | Source            | Destination                                                             | Port / Service | Protocole | Inspection | Commentaire             |
| -- | ---- | ------- | ----------------- | ----------------------------------------------------------------------- | -------------- | --------- | ---------- | ----------------------- |
| 1  | ON   | Bloquer | Network_guest     | Network_out, Network_lan, Network_Wifi_SLAM, Vlan-clients, Vlan-DMZ-PRV | Any            | -         | IPS        | Isolation Wifi-Guest    |
| 2  | ON   | Bloquer | Network_Wifi-SLAM | Network_out, Vlan-clients, Vlan-DMZ-PRV, Network_guest                  | Any            | -         | IPS        | Isolation Wifi-SLAM     |
| 3  | ON   | Passer  | Any               | DNS-Autorité-Primaire                                                   | DNS            | -         | IPS        | Autorisation DNS externe|
| 4  | ON   | Passer  | Any               | Reverse-Proxy-Primaire                                                  | HTTP / HTTPS   | TCP       | IPS        | Accès reverse proxy     |
| 5  | ON   | Passer  | Any               | Server-Mail                                                             | SMTP / IMAP    | TCP       | IPS        | Accès serveur mail      |
| 6  | ON   | Passer  | Server-Mail       | Any                                                                     | SMTP           | TCP       | IPS        | Envoi mails sortants    |

### Section Wifi-SLAM

| ID | État | Action     | Source                                  | Destination           | Port / Service | Protocole | Inspection   | Commentaire                 |
| -- | ---- | ---------- | --------------------------------------- | --------------------- | -------------- | --------- | ------------ | --------------------------- |
| 7  | ON   | Passer     | Network_Mana-AP                         | CHA-RADIUS            | RADIUS         | -         | IPS          | Authentification borne WiFi |
| 8  | ON   | Passer     | Network_Wifi-SLAM (interface Wifi-SLAM) | DNS_Resolver_Primaire | DNS            | -         | IPS          | Résolution DNS              |
| 9  | ON   | Passer     | Network_Wifi-SLAM (interface Wifi-SLAM) | Internet              | Any            | -         | IPS          | Accès Internet Wifi-SLAM    |

### Section Wifi_Guest

| ID | État | Action                     | Source                                     | Destination | Port / Service | Protocole | Inspection   | Commentaire                   |
| -- | ---- | -------------------------- | ------------------------------------------ | ----------- | -------------- | --------- | ------------ | ----------------------------- |
| 12 | ON   | Portail d’authentification | unknown @ Network_guest (interface: guest) | Internet    | HTTP / HTTPS   | -         | IPS          | Authentification utilisateurs |
| 13 | ON   | Déchiffrer                 | any @ Network_guest (Auth. par : Invité)   | Internet    | ssl_srv        | -         | IPS          | Filtrage SSL CHA-PROXY        |
| 14 | ON   | Passer                     | any @ Network_guest via Proxy SSL          | Internet    | ssl_srv        | -         | IPS (IPS_01) | Navigation Internet invités   |

### Section Acces_Internet

| ID | État | Action     | Source                          | Destination | Port / Service | Protocole | Inspection   | Commentaire                |
| -- | ---- | ---------- | ------------------------------- | ----------- | -------------- | --------- | ------------ | -------------------------- |
| 15 | ON   | Déchiffrer | Réseau-interne (interface: lan) | Internet    | ssl_srv        | -         | IPS          | Filtrage SSL CHA-PROXY     |
| 16 | ON   | Passer     | Réseau-interne via Proxy SSL    | Internet    | ssl_srv        | -         | IPS (IPS_01) | Navigation Internet LAN    |
| 17 | ON   | Passer     | Any                             | Any         | Any            | -         | IPS          | Règle globale autorisation |

---

## Stormshield n°2

| ID | État | Action  | Source         | Destination              | Port / Service | Protocole | Inspection | Commentaire        |
| -- | ---- | ------- | -------------- | ------------------------ | -------------- | --------- | ---------- | ------------------ |
| 1  | ON   | Passer  | Any            | Dns-autorité-secondaire  | DNS            | -         | IPS        | Accès DNS externe  |
| 2  | ON   | Passer  | Any            | Reverse-Proxy-secondaire | HTTP / HTTPS   | TCP       | IPS        | Accès Reverse Proxy|
| 3  | ON   | Passer  | Network_dmz1   | Any                      | DNS            | -         | IPS        | DNS sortant        |
| 4  | ON   | Passer  | Réseau-Interne | Any                      | Any            | -         | IPS        | Accès Internet LAN |
| 5  | ON   | Bloquer | Any            | Any                      | Any            | -         | IPS        | Bloc tout WAN      |

---

## OPNsense

### DMZPUB

| ID | Protocole    | Source          | Port Source | Destination    | Port Destination | Passerelle | Description                              |
| -- | ------------ | --------------- | ----------- | -------------- | ---------------- | ---------- | ---------------------------------------- |
| 1  | IPv4 TCP/UDP | 172.28.62.0/24  | *           | 172.28.33.4    | 53 (DNS)         | *          | DMZ vers DNS interne principal           |
| 2  | IPv4 TCP/UDP | 172.28.62.0/24  | *           | 172.28.33.5    | 53 (DNS)         | *          | DMZ vers DNS interne secondaire          |
| 3  | IPv4 TCP/UDP | 172.28.32.0/24  | *           | 172.28.33.4    | 53 (DNS)         | *          | Réseau interne vers DNS principal        |
| 4  | IPv4 TCP/UDP | 192.168.20.0/24 | *           | 172.28.33.4    | 53 (DNS)         | *          | Wif_Guest accès DNS Interne principal    |
| 5  | IPv4 TCP     | 172.28.62.3     | *           | 192.168.28.10  | 3306 (MySQL)     | *          | Accès base de données                    |
| 6  | IPv4 TCP     | 172.28.62.5     | *           | 172.28.33.8    | *                | SWCORE     | Reverse Proxy principal vers GLPI        |
| 7  | IPv4 TCP     | 172.28.62.10    | *           | 172.28.33.8    | *                | SWCORE     | Reverse Proxy secondaire vers GLPI       |
| 8  | IPv4 TCP     | 172.28.62.2     | *           | 172.28.35.0/24 | 25 (SMTP)        | *          | Envoi de mails                           |
| 9  | IPv4 TCP     | 172.28.62.2     | *           | 172.28.35.0/24 | 143 (IMAP)       | *          | Accès messagerie                         |
| 10 | IPv4 UDP     | 192.168.99.1    | *           | 172.28.33.7    | 1812 (RADIUS)    | *          | Borne Wifi vers serveur Radius           |

### DMZPRV

| ID | Protocole    | Source          | Port Source | Destination    | Port Destination | Passerelle | Description                              |
| -- | ------------ | --------------- | ----------- | -------------- | ---------------- | ---------- | ---------------------------------------- |
| 1  | IPv4 *       | *               | *           | *              | *                | *          | Bloc all (la BBD ne peut pas sortir)     |

### LAN

| ID | Protocole | Source         | Port Source | Destination | Port Destination | Passerelle | Description                        |
| -- | --------- | -------------- | ----------- | ----------- | ---------------- | ---------- | ---------------------------------- |
| 1  | IPv4 *    | 172.28.32.0/19 | *           | *           | *                | *          | LAN vers DMZ                       |

---


