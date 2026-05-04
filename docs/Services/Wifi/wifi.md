# Configuration de la Borne et du DHCP Stormshield

## Objectif

---

Pouvoir se connecter et administrer la borne Wifi.

---

## 1. Configuration du serveur DHCP sur Stormshield

Pour que la Borne puissent accéder au réseau, elle doit recevoir une adresse IP.

Il faut se connecter à l’interface web du Stormshield puis aller dans **Configuration → Réseau → Services → DHCP Server**.

Ensuite activer le DHCP sur l’interface in, sur le réseau de Mana de la borne :

```
Plage IP	192.168.99.100 – 192.168.99.200
Masque	255.255.255.0
Passerelle	192.168.99.254
DNS	172.28.33.4
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

Une fois connecter nous pourrons configurer notre Wi-fi Guest et SLAM.