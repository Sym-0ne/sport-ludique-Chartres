# FailOver avec le Protocole HSRP

Voici une présentation rapide du protocole HSRP avec des exemples de configuration utilisant des adresses IP de la plage 172.28.63.0/24, avec une adresse VIP (Virtual IP) de 172.28.63.254 :

Supposons que vous ayez deux routeurs Cisco, R1 et R2, et que vous souhaitiez configurer HSRP entre eux pour la haute disponibilité.

## Configuration de R1 :
```
interface GigabitEthernet0/0
 ip address 172.28.63.2 255.255.255.0
 standby 1 ip 172.28.63.10
 standby 1 priority 110
 standby 1 preempt
```

### Dans cette configuration :

L'interface GigabitEthernet0/0 de R1 est configurée avec l'adresse IP 172.28.63.2.<br>

```"standby 1 ip 172.28.63.10"``` définit l'adresse VIP (Virtual IP) HSRP à **172.28.x.254.**<br>
```"standby 1 priority 110"``` définit la priorité de ce routeur à 110 (par défaut est 100).<br>
```"standby 1 preempt"``` permet à R1 de reprendre automatiquement le rôle de routeur actif s'il redevient disponible après une panne.<br>


## Configuration de R2 :
```
interface GigabitEthernet0/0
 ip address 172.28.63.3 255.255.255.0
 standby 1 ip 172.28.63.10
 standby 1 priority 100
 standby 1 preempt
```

### Dans cette configuration :

L'interface GigabitEthernet0/0 de R2 est configurée avec l'adresse IP 172.28.63.3.<br>
"standby 1 ip 172.28.63.10" définit également l'adresse VIP HSRP à 172.28.63.10, qui est la même que celle configurée sur R1.<br>
"standby 1 priority 100" définit la priorité de ce routeur à 100.<br>
"standby 1 preempt" permet à R2 de prendre le relais en tant que routeur actif si R1 devient indisponible.<br>

Ainsi, avec cette configuration, R1 sera le routeur actif tant qu'il est disponible. En cas de panne de R1 ou de sa connexion, R2 prendra automatiquement le relais en tant que routeur actif, assurant ainsi une haute disponibilité pour la VIP 172.28.63.10