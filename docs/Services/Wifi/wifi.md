# Configuration de la Borne et du DHCP Stormshield

## Objectif

---
Permettre aux SLAM de pouvoir accèder à notre réseaux via le Wifi afin qu'il accède à leurs serveurs que nous hebergeons.
--

## 1. Configuration du serveur DHCP sur Stormshield

Pour que les clients Wi-Fi puissent accéder au réseau, ils doivent recevoir une adresse IP.

Il faut se connecter à l’interface web du Stormshield puis aller dans **Configuration → Réseau → Services → DHCP Server**.

Ensuite activer le DHCP sur l’interface in, sur le réseau Guest :

```
Plage IP	192.168.20.1 – 192.168.20.253
Masque	255.255.255.0
Passerelle	192.168.20.254
DNS	172.28.62.1
```

Enfin **sauvegarder la configuration** et allez vérifier dans **les logs** quel adresse IP a récuperer la borne Wi-fi.

## 2. Configuration de la borne Wi-Fi

Pour réinitialiser la borne vous pouvez suivre cette [documentation](https://arubanetworking.hpe.com/techdocs/hardware/aps/ap220/ig/AP-22X%20Installation%20Guide%20Rev%2002_FR.pdf).

La borne Aruba IAP-103 fonctionne en mode autonome.

Premièrement il faut brancher la borne au réseau et il sera alimenter vu que la borne est Poe.

Ensuite la borne récupèrera une IP via DHCP et après on pourra accéder à son interface web :

```
https://IP_DE_LA_BORNE
```

Une fois connecter nous pourrons configurer notre Wi-fi Guest.