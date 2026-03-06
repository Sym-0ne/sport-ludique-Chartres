# Mise en place de Rsyslog pour Firewall vers Graylog

## Objectif

L’objectif est de configurer toutes les Pare-feu de l'infrastructure afin qu’elles envoient leurs logs vers le serveur Graylog via UDP.<br>
Cela permet de centraliser la collecte et la visualisation des logs, d’identifier rapidement les problèmes et de suivre les activités des équipements.<br>

## Input

Si l'input n'est pas encore créé vous pouvez allez voir [ici](https://sym-0ne.github.io/sport-ludique-Chartres/Serveurs/Logs/InterfaceWebLog/) à l'étape 3.

## 1 Configuration du pare-feu Stormshield

### 1.1 Accéder aux paramètres de logs :

Dans l’interface Stormshield SNS, aller dans **Configuration → Notifications → Logs - Syslog**.

### 1.2 Ajouter un serveur Syslog : 

Ajouter un serveur avec les paramètres :

```
Adresse du serveur	IP de Graylog
Port	514
Protocole	UDP
Format	Syslog
```

### 1.3 Choisir les logs à envoyer :

Dans **Configuration → Logs**, activer l’envoi des événements :

```
Firewall
VPN
Authentication
System
```

Ensuite il ne faut pas oublier **d'appliquer** la configuration, après ça le pare-feu Stormshield enverra maintenant ses logs vers Graylog.

## 2 Configuration du pare-feu OPNsense

### 2.1 Accéder aux paramètres Syslog :

Dans l’interface OPNsense aller dans **System → Settings → Logging → Targets**

### 2.2 Ajouter un serveur Syslog :

Ajouter une nouvelle cible :

```
Transport	UDP
IP Address	IP du serveur Graylog
Port	514
Facility	local0
```

### 3.3 Choisir les logs à envoyer :

Sélectionner les catégories :

```
Firewall
DHCP
VPN
System
Authentication
```

Enfin il ne faut pas oublier de sauvergarder :

```
Save
```

Maintenant les logs seront envoyés vers Graylog.