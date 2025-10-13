# UFW 
Cette documentation donne les commandes basiques et l'installation d'UFW (Uncomplicated Fire Wall)

UFW (Uncomplicated Firewall) est un pare-feu simple pour Linux, basé sur iptables.
Il sert à contrôler le trafic réseau entrant et sortant grâce à des règles faciles à gérer.

<ins>Points clés :</ins>

- Facile à utiliser : commandes simples (allow, deny, status).

- Filtrage par port, protocole et IP.

- Gestion des interfaces multiples.

- Règles persistantes et journalisation.

- Objectif : sécuriser un serveur ou un réseau en autorisant uniquement le trafic nécessaire.

## 1. Instalation 📦

Mettez a jour votre system
```
sudo apt update 
sudo apt upgrade
```

Installez UFW 
```
sudo apt install ufw -y
```

## 2. Configuration basique 🔧

Voici les commandes basiques d'UFW :

### Activation/ Désactivation 🟢/🔴

UFW comme tout les pare-feux peut être activer ou désactiver, lors de son instalation il est désactiver par default
```
sudo ufw enable     #active le pare-feux
sudo ufw disable    #désactive le pare-feux
sudo ufw status     #Donne l'état du pare-feux (actif/inactif)
```
### Règles par défaut 📄

UFW compte 4 règles de filtrage par default qui servent a tout bloquer/autoriser sur le trafic entrant ou sortant :
```
sudo ufw default deny incoming   # Bloque tout le trafic entrant par défaut
sudo ufw default allow incoming  # Autorise tout le trafic entrant par défaut
sudo ufw default allow outgoing  # Autorise tout le trafic sortant par défaut
sudo ufw default deny outgoing   # Bloque tout le trafic sortant (si serveur très sécurisé)
```
### Règles filtrantes :broom:

Chaque règle prend en compte plusieurs paramètres, autoriser, refuser, le port, l'IP... 

Voici une commande type d'UFW 

```
sudo ufw <allow/deny> <in/out> on <interface> from <all/ip/réseau> to <all/ip/réseau> port <n°de port> proto <protocole ip>
```
Chaque règle peut être adapté en fonction des besoins. 
